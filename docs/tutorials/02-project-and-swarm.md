# 2. 프로젝트 구조와 군집전

**완료 기준:** 한 시뮬레이션 스텝에서 환경, 할당, BT, 조종, 물리가 어떤 순서로 연결되는지 파일을 따라갑니다.

## N:M은 무엇인가

N:M은 blue N대와 red M대가 싸우는 설정입니다. 각 항공기는 한 번에 추격할 표적을 할당받습니다.
“군집”이라는 말이 곧 모든 구성요소가 분산되어 있다는 뜻은 아닙니다.

현재 분산형 알고리즘은 할당 정보를 제한된 이웃 통신으로 교환합니다. 월드 물리와 평가기는 한 프로세스에서 전체 환경을 관리합니다.
즉, **분산 임무 할당을 시뮬레이션하는 연구용 환경**이지 분산 물리 시뮬레이터나 실기체 통신 스택이 아닙니다.

## 역할의 세 층

| 층 | 질문 | 책임 |
|---|---|---|
| Mission / Assigner | “누가 누구를 맡을까?” | 살아 있는 적 표적 중 추격 대상 선택 |
| Behavior / BT | “지금 어떤 행동이 우선인가?” | 경계 복귀, 아군 분리, 추격, 대기 같은 명시적 규칙 |
| Maneuver / Brain | “그 표적에 어떻게 접근할까?” | 관측에 따라 방향·속도·고도 행동 선택 |
| World / Backend | “명령 뒤에 무슨 일이 일어났나?” | 항공기 상태, 무장, 피해, 종료와 점수 계산 |

할당기는 항공기 제어를 하지 않고, BT는 승패를 직접 계산하지 않습니다.
이 경계 덕분에 같은 조종기를 다른 규모에서 시험하거나, 조종기를 유지한 채 할당 알고리즘만 비교할 수 있습니다.

## 코드 따라가기

```text
combat.yaml
   │ 설정 파싱
   ▼
scripts/run_swarm.py
   │ SwarmConfig + AssignmentConfig 생성
   ▼
swarm_aircombat/envs/swarm_pursuit.py
   ├── assignment/registry.py → 할당 알고리즘
   ├── bt/swarm_tree.py       → safety/deconflict/pursue/loiter
   ├── bt/brain.py            → 조종기 생성
   ├── rl/state.py            → RL 관측 벡터
   └── core/aircraft.py       → 비행 명령을 물리 상태로 적용
```

찾기 시작점:

- 환경과 step 순서: `swarm_aircombat/envs/swarm_pursuit.py`
- 편대 설정과 플러그인 선택: `SwarmConfig`, `assignment/registry.py`
- BT 엔진과 군집 트리: `swarm_aircombat/bt/core.py`, `swarm_aircombat/bt/swarm_tree.py`
- 파일럿을 선택하는 공통 경로: `swarm_aircombat/bt/brain.py`
- 관측·보상: `swarm_aircombat/rl/state.py`, `swarm_aircombat/rl/reward.py`
- 학습·평가·실행 스크립트: `scripts/train.py`, `scripts/compare.py`, `scripts/run_swarm.py`

## 4v20에서 “분산”과 “중앙” 구분하기

`hungarian`은 팀의 비용 행렬 전체를 계산해 전역 최소 비용 배정을 구하는 **중앙 기준선**입니다.
`cbba`, `first_claimed`, `grape`는 각 기체가 감지 범위 안의 표적 비용을 만들고 통신 범위 내 이웃과 정보 교환을 하는 **분산형 적응 알고리즘**입니다.
다만 이 환경이 에피소드 기록을 수집하는 과정과 물리 업데이트는 중앙 실행입니다.

## 실습

1. `scripts/run_swarm.py`에서 `combat.yaml`을 어떻게 읽고 `SwarmConfig`로 바꾸는지 확인합니다.
2. `envs/swarm_pursuit.py`의 초기화에서 `_blue_assign`과 Brain 목록이 생성되는 부분을 읽습니다.
3. 파일럿의 `info`가 어디서 만들어지고 `State`가 어떻게 숫자 벡터로 바꾸는지 추적합니다.
4. [용어 사전](../reference/glossary.md)에서 모르는 개념을 확인합니다.

다음: [Behavior Tree로 규칙 표현하기](03-behavior-trees.md)
