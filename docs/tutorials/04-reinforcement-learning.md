# 4. 강화학습: 관측에서 행동과 보상까지

**완료 기준:** 관측·행동·보상의 역할을 설명하고, 이산 setpoint 정책과 연속 조종 정책의 차이를 확인해 작은 학습 실험을 재현합니다.

## 강화학습의 작은 고리

1. 환경이 교전 상태를 관측 벡터로 바꿉니다.
2. 정책이 행동을 선택합니다.
3. 환경과 물리 모델이 기체·무장 상태를 갱신합니다.
4. 정책은 보상과 다음 관측을 받습니다.
5. PPO 또는 SAC가 경험을 이용해 정책 파라미터를 학습합니다.

관측 정의는 `swarm_aircombat/rl/state.py`에 있습니다. 기본 `State`는 배정 표적의
상대 기하와 전투 상황 등을 25개 채널로 변환합니다. 연속 제어의
`ContinuousState`는 여기에 pitch 자세, 비행경로각, p/q/r 각속도 다섯 채널을 더합니다.
이는 조종면 입력에 필요한 자세·회전 피드백을 정책에 제공합니다.

## 먼저 기존 이산 정책 학습하기

기본 동작은 `Discrete(27)`입니다. 방향 변화 3개 × 속도 변화 3개 × 고도 변화 3개 중
하나를 고르고, 오토파일럿이 목표점을 따라갑니다. 정책이 조종면을 직접 움직이지
않으므로 이산 행동의 의미를 익히고 PPO 학습 경로를 확인하기 좋습니다.

```bash
python -m pip install -e ".[train]"
python scripts/train.py --red-behavior mixed --n-blue 4 --n-red 2 \
  --steps 100000 --worlds 4 --seed 0 --name tutorial
```

`--steps`는 병렬 수집한 에이전트 샘플 수입니다. 결과는
`runs/tutorial/s0/`에 저장됩니다. `best.zip`은 검증에서 가장 높은 점수를 받은
스냅샷이고, `last.zip`은 마지막 가중치입니다. `progress.csv`와 `result.json`에는
평가와 설정이 들어 있습니다.

## 연속 조종 정책: JSBSim에서 조종축 직접 제어

기체가 받는 직접 제어를 학습하고 싶다면 선택 의존성을 설치합니다.

```bash
python -m pip install -e ".[jsbsim,train]"
```

그리고 `Box(4,)` 연속 행동, JSBSim 백엔드, PPO를 이용해 짧은 실험부터 실행합니다.

```bash
python scripts/train.py --control-mode continuous --backend jsbsim \
  --algorithm PPO --red-behavior circular --n-blue 1 --n-red 1 \
  --steps 4096 --worlds 1 --val-every 2048 --val-episodes 1 \
  --seed 0 --name continuous-smoke
```

각 행동은 `[-1, 1]` 범위의 `[roll, pitch, rudder, throttle]`입니다. Roll과 rudder의
부호는 좌/우, pitch의 양수는 기수 올림, throttle의 `-1/+1`은 유휴/최대 추력을
뜻합니다. 환경은 이 명령을 F-16 모델에 직접 전달하며 기존 heading/speed/altitude
오토파일럿 경로는 거치지 않습니다. 이것이 원하는 기동을 자동으로 학습한다는
보장은 없습니다. 관측·보상·상대·초기 조건에 따라 성능을 평가해야 합니다.

SAC는 Box 연속 행동을 위한 두 번째 알고리즘입니다.

```bash
python scripts/train.py --control-mode continuous --backend jsbsim \
  --algorithm SAC --red-behavior circular --steps 10000 \
  --worlds 1 --val-every 5000 --val-episodes 1 --seed 0 --name sac-smoke
```

PPO와 SAC 모두 테스트할 수 있는 학습 구현입니다. 작은 smoke run은 연결을 확인할
뿐 전술 성능 평가가 아닙니다. 기본 point-mass 환경은 연속 조종면을 지원하지 않아
`control_mode: continuous`에는 JSBSim이 필요합니다. 현재 BT/XML 예제도 직접 조종면
행동을 출력하지 않으며, 이산 setpoint 행동과 연속 행동은 서로 다른 인터페이스입니다.

## 평가하고 체크포인트 불러오기

학습이 끝났다고 성능이 좋아졌다는 뜻은 아닙니다. 여러 고정 시드와 상대 설정에서
검증하세요.

```bash
python scripts/compare.py --red-behavior circular \
  --policies lead runs/tutorial/s0/best.zip --episodes 30

python scripts/compare.py --backend jsbsim --control-mode continuous \
  --policies sac:runs/sac-smoke/s0/best.zip --episodes 10
```

SAC 체크포인트는 알고리즘을 명시해 `sac:<경로>`로 불러옵니다. PPO 체크포인트는
일반 경로 또는 `ppo:<경로>`를 사용할 수 있습니다. 연속 정책 외에도 이전
이산 setpoint Brain은 호환 목적으로 연속 환경에서 실행할 수 있지만, 이는 같은
행동 인터페이스를 비교하는 실험은 아닙니다.

!!! warning "체크포인트 호환성과 모델 범위"
    이산 `Discrete(27)`과 연속 `Box(4,)` 체크포인트는 서로 대체할 수 없습니다.
    정책 로더는 행동 공간을 확인합니다. 예전 `Discrete(9)` 체크포인트도 현재
    행동 인코딩과 호환되지 않습니다. 체크포인트는 학습 알고리즘, 관측·행동 계약,
    시뮬레이터 설정 및 seed와 함께 보관하세요.

## 보상은 무엇을 가르치나

`swarm_aircombat/rl/reward.py`의 shaping, 피해·격추, WEZ, 충돌, 시간 및 종료 보상은
서로 다른 신호입니다. Shaping 점수가 높아졌다고 승리율도 좋아졌다고 볼 수는
없습니다. 승패뿐 아니라 격추·손실·충돌·교전 시간과 리플레이를 함께 확인하세요.

## 규칙·BT와 RL의 차이

규칙 기반 Brain은 사람이 코드를 작성해 행동을 결정합니다. BT는 조건과 행동의
우선순위를 트리로 구성합니다. PPO와 SAC는 관측·보상과 경험으로 정책 파라미터를
학습합니다. 서로 조합할 수 있지만 하나의 필수 파이프라인은 아니며, 현재 BT/XML
Action이 연속 조종면 입력을 출력하는 것은 아닙니다.

## 재현성 실습

실험마다 알고리즘, 제어 모드, 백엔드, 팀 규모, 상대 Brain/behavior, 보상 설정,
seed, timestep, 평가 seed와 체크포인트를 함께 기록합니다. [실험 설계 안내](07-evaluate-and-extend.md)처럼
학습 seed와 held-out 평가 seed를 구분하고 에피소드별 결과를 남기세요.
Windows는 별도로 JSBSim 설치, 단일 에피소드 smoke test와 PPO/SAC 짧은 학습을
실행하기 전까지 검증 완료로 간주하지 않습니다.

다음: [학습 결과를 해석하고 작은 실험 설계하기](07-evaluate-and-extend.md)
