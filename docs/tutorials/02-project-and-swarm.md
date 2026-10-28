# 2. 전투 시뮬레이터 구조

**완료 기준:** 전투 한 스텝에서 에이전트의 결정이 비행 명령과 항공기 상태 갱신으로 이어지는 흐름을 설명합니다.

## N:M은 무엇인가

N:M은 blue N대와 red M대가 싸우는 설정입니다. “군집”이라는 말이 모든 구성요소가
분산되어 있다는 뜻은 아닙니다. 표적 할당을 위한 분산형 선택지도 있지만, 물리
업데이트와 전투 결과는 시뮬레이터 한 곳에서 계산합니다.

1v1 결투에는 편대 표적 할당기가 없습니다. 현재 SKY의 N:M 환경은 각 기체에 표적을
정하는 별도 assigner를 사용하고, 사용자가 설정에서 그 구현을 바꿀 수 있습니다.
이것은 현재 코드의 모듈 경계이며, 모든 전투가 반드시 따라야 하는 구조는 아닙니다.

## 하나의 필수 파이프라인이 아니라 가능한 조합

규칙 기반 조종, Behavior Tree(BT), 강화학습(PPO)을 항상 층층이 연결해야 하는 것은
아닙니다. 이들은 에이전트 행동을 만드는 방법이며, 하나만 사용하거나 필요에 맞게
조합할 수 있습니다.

| 구성 방식 | 예 | SKY에서 가능한 형태 |
|---|---|---|
| 규칙 기반 파일럿 | 기하 조건에 따라 선회·속도·고도를 직접 결정 | `lead` 같은 코드형 Brain |
| Behavior Tree | 조건과 우선순위로 어떤 행동을 할지 선택 | BT Action이 직접 행동을 내거나 선택한 Brain을 호출 |
| 학습 정책 | 관측을 받아 행동을 선택 | PPO 또는 연속 행동용 SAC 정책을 조종기(Brain)로 사용 |
| 사용자 조합 | 상태에 따라 규칙·BT·정책을 결합 | 사용 목적에 맞춰 의사결정 흐름 구성 |

BT는 사람이 작성한 규칙을 구조화해 실행하는 방법이지, 이 프로젝트에서 자동으로
학습되는 알고리즘은 아닙니다. PPO는 별도의 학습 과정에서 정책을 업데이트합니다.
현재 SKY의 N:M 구현은 할당기와 파일럿을 별도 인터페이스로 제공하고 환경이
assigner를 호출합니다. 사용자는 한 정책이 표적 선택과 기동을 함께 결정하는 설계도
할 수 있으며, BT가 할당을 맡아야 한다는 규칙은 없습니다.

## 코드 따라가기

```text
combat.yaml
   │ 설정 파싱
   ▼
scripts/run_swarm.py
   │
   ▼
swarm_aircombat/envs/swarm_pursuit.py
   ├── bt/brain.py            → 파일럿/Brain 선택
   ├── bt/swarm_tree.py       → N:M에서 쓰는 BT 규칙 구성 예시
   ├── rl/state.py            → PPO를 쓸 때 현재 상태를 관측으로 변환
   ├── core/aircraft.py       → 비행 명령으로 항공기 상태 갱신
   ├── core/engagement.py     → 사격 조건과 피해 갱신
   └── assignment/            → N:M 표적 배분 (필요할 때만)
```

찾기 시작점:

- 환경과 step 순서: `swarm_aircombat/envs/swarm_pursuit.py`
- 편대 설정과 조종기 선택: `SwarmConfig`, `bt/brain.py`
- BT 엔진과 군집 트리: `swarm_aircombat/bt/core.py`, `swarm_aircombat/bt/swarm_tree.py`
- 관측·보상: `swarm_aircombat/rl/state.py`, `swarm_aircombat/rl/reward.py`
- 항공기·교전 모델: `swarm_aircombat/core/aircraft.py`, `swarm_aircombat/core/engagement.py`
- 학습·평가·실행 스크립트: `scripts/train.py`, `scripts/compare.py`, `scripts/run_swarm.py`

## 항공기에게 전달하는 두 가지 조종 계약

- 기본 `discrete` 모드에서 에이전트는 27개 heading/speed/altitude setpoint 조합 중
  하나를 선택합니다. point-mass와 JSBSim의 자동조종기가 해당 목표를 따라갑니다.
- 선택형 `continuous` 모드에서는 JSBSim만 사용하며 정책이
  `[roll, pitch, rudder, throttle]` 네 값을 `[-1, 1]` 범위로 직접 보냅니다.
  이 모드의 관측에는 자세와 p/q/r 각속도 피드백이 추가됩니다.
- 기존 XML BT 예제의 Action은 여전히 setpoint를 출력합니다. BT로 연속 조종축을
  직접 만들려면 그 동작 인터페이스를 별도로 구현해야 합니다.

## 항공기 모델은 얼마나 현실적인가?

기본 point-mass 모델은 빠른 학습 반복을 위한 간략화된 운동 모델입니다. JSBSim
옵션은 F-16 모델과 자동조종기를 사용하지만, 선택형 모델을 쓴다는 사실이 현실
전투 재현이나 전술 검증을 의미하지는 않습니다. 두 설정에서 같은 행동이 어떻게
다르게 움직이는지 학습 주제로 삼을 수 있습니다.

## 현재 구현에서 표적 할당은 어디에 있는가?

N:M 환경은 `_reassign_side`에서 별도 assigner를 호출해 기체별 표적을 정합니다.
현재 BT는 그 할당 결과를 입력으로 받아 규칙을 실행하며, 표적 할당을 하지 않습니다.
이것은 SKY 코드의 구현 방식이지, BT로 표적 선택을 할 수 없다는 뜻은 아닙니다.

## N:M에서 “분산”과 “중앙” 할당 구분하기

할당기를 사용한다면 `hungarian`은 팀의 비용 행렬 전체를 계산하는 중앙 기준선입니다.
`cbba`, `first_claimed`, `grape`는 각 기체의 감지 범위와 통신 범위 내 이웃 정보를
사용하는 분산형 적응입니다. 이런 알고리즘 비교는 편대 표적 배분을 공부하고 싶을 때
선택할 수 있으며, 단일 기체 조종이나 BT/RL의 선행 단계는 아닙니다.

## 실습

1. `scripts/run_swarm.py`에서 `combat.yaml`을 어떻게 읽고 `SwarmConfig`로 바꾸는지 확인합니다.
2. `bt/brain.py`에서 규칙·BT·PPO가 파일럿 선택지로 연결되는 부분을 읽습니다.
3. 관심이 있다면 `envs/swarm_pursuit.py`의 `_reassign_side`에서 N:M 할당이 호출되는 경우를 확인합니다.
4. [용어 사전](../reference/glossary.md)에서 모르는 개념을 확인합니다.

다음: [규칙 기반 조종과 Behavior Tree](03-behavior-trees.md)
