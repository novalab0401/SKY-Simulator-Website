# 3. 규칙 기반 조종과 Behavior Tree

**완료 기준:** 사람이 작성한 규칙과 BT의 차이를 설명하고, Sequence/Selector 우선순위를 바꿔 실행 경로가 어떻게 달라지는지 확인합니다.

## BT가 하는 일

Behavior Tree는 매 결정 스텝에 조건을 확인하고 행동을 고르는 트리입니다.
이 프로젝트에는 자체 경량 엔진이 있으며 BT가 직접 항공기 물리를 계산하거나 임무 할당을 하지는 않습니다.

정확히는 **현재 N:M 환경의 BT가** 별도 assigner가 정한 표적을 받아 규칙을 실행합니다.
이는 이 시뮬레이터의 연결 방식입니다. BT라는 방법 자체가 표적 선택을 할 수 없다는
뜻은 아닙니다. 사용자 BT의 Condition이나 Action에 표적 선택 규칙을 넣는 설계도 가능합니다.

**Rule-based는 결정 방식을 넓게 부르는 말이고, BT는 규칙을 구성하는 한 가지 구조입니다.**
`lead`처럼 if/else나 함수로 직접 작성한 파일럿도 규칙 기반입니다. BT는 그 규칙들을
Condition과 Action, 그리고 Selector/Sequence 같은 우선순위 구조로 나누어 사람이
읽고 추적하기 쉽게 합니다. BT를 수정하는 것은 규칙 설계 연습이지, BT 엔진이
자동으로 전술을 학습하는 것은 아닙니다.

`swarm_aircombat/bt/core.py`의 상태:

- `SUCCESS`: 노드가 성공했습니다.
- `FAILURE`: 조건이 맞지 않거나 행동을 실행하지 못했습니다.
- `RUNNING`: 동작이 아직 진행 중입니다. 현재 트리의 모든 행동에서 지속 상태를 유지하는 핵심 메커니즘으로 쓰이지는 않습니다.

| 노드 | 순회 규칙 |
|---|---|
| `Condition` | 참이면 `SUCCESS`, 거짓이면 `FAILURE` |
| `Action` | 함수를 실행. 반환이 없으면 `SUCCESS` |
| `Sequence` | 앞에서부터 실행하다 처음으로 `SUCCESS`가 아닌 결과에서 멈춤 |
| `Selector` | 앞에서부터 실행하다 처음으로 `FAILURE`가 아닌 결과에서 멈춤 |

## 전투기 조종에서 규칙이 선택되는 예

`swarm_aircombat/bt/swarm_tree.py`의 트리는 우선순위가 높은 분기부터 확인합니다.

```text
Selector(root)
├── Sequence(avoid_boundary)
│   ├── Condition(near_boundary)
│   └── Action(turn_to_center)
├── Sequence(engage)
│   ├── Action(deconflict)
│   ├── Condition(has_target)
│   └── Action(rl_pursue)
└── Action(loiter)
```

- 경계가 위험하면 방향을 경기장 중앙으로 강제합니다.
- 그렇지 않으면 아군과 가까운 기체의 고도를 분리한 뒤 배정 표적을 향한 기동 정책으로 넘어갑니다.
- 표적이 없으면 선회 대기합니다.

`rl_pursue`는 추격 행동을 정책에 맡기라는 결정을 Blackboard에 남깁니다. 선택한
Brain 설정에 따라 규칙 기반 파일럿이나 PPO 정책이 그 기동을 만들 수 있습니다.
경계 복귀와 대기 분기는 정책 액션을 사용하지 않고 직접 명령을 지정할 수 있습니다.
각 노드는 `bb.trace`에 기록되며 시각화에서 활성 분기를 확인할 수 있습니다.

## 내가 기체에 시킬 수 있는 조종은 무엇인가

현재 파일럿 행동 공간은 `Discrete(27)`입니다. 한 번에 세 가지 목표를 각각 고르고,
그 조합을 기체의 오토파일럿에 전달합니다.

| 축 | 선택지 | 뜻 |
|---|---|---|
| 방향 | 좌회전 `-30°` / 유지 `0°` / 우회전 `+30°` | 현재 heading에 대한 목표 방향 변화 |
| 속도 | `-25` / `0` / `+25 m/s` | 현재 속도에서 감속 / 유지 / 가속 목표 |
| 고도 | 기준 고도 `-300` / `0` / `+300 m` | 기준 고도보다 낮게 / 유지 / 높게 |

예를 들어 “표적 쪽으로 우회전하면서 가속하고, 표적보다 300 m 높게”를 고를 수
있습니다. 세 축의 조합이 27개 행동을 만듭니다. 결정 주기 기본값은 0.5초입니다.
명령은 조종면 입력이 아닙니다. point-mass 모델이나 선택형 JSBSim 백엔드가
heading·속도·고도 목표를 따라가며, 속도·선회·상승률 제한을 적용합니다. 따라서
현재 동작만으로 실제 F-16의 롤, 피치, 요 조종이나 복잡한 기동을 직접 지정할 수는
없습니다. `Break` 같은 이름도 이 행동 조합을 만드는 프로젝트 내부 규칙이지,
실제 기동을 정밀하게 재현한다는 뜻은 아닙니다.

## 기동 의도에서 조종면으로: 첫 BT 기동

BT 액션마다 조종면 값을 직접 지정하는 대신, 기동 액션은 **의도**를 내고 내부
제어기가 이를 JSBSim 입력으로 변환할 수 있습니다.

```text
BT Action -> ManeuverIntent(목표 뱅크, 하중배수, 비행경로각, 스로틀)
          -> JSBSim 내부 제어기 -> 에일러론/엘리베이터/방향타/스로틀
```

첫 예제 `swarm_aircombat/bt/trees/level-flight.xml`은 `LevelFlight` 액션 하나만 가진
BT입니다. 액션은 **목표 뱅크 0°, 하중배수 1g, 비행경로각 0°, 스로틀 0.6**을
요청합니다. 항공기 제어기가 bank/rate, 하중배수와 비행경로각 오차를 사용해
조종면 명령을 만듭니다. 지금은 고정된 목표값인 한 단계짜리 기동입니다.
상태에 따라 목표를 바꾸거나 여러 단계로 이어지는 기동은 아직 추가하지 않았습니다.

이 BT 액션은 N:M 환경의 팀 Brain으로 사용할 수 있고, `backend: jsbsim`,
`control_mode: continuous`가 필요합니다. `training.side` 팀에 설정하면 그 Brain이
외부 정책으로 제출한 intent를 실행하고, 반대편 팀에 설정하면 고정 Brain으로
실행합니다. 예를 들어 다음 설정은 Blue의 LevelFlight BT와 Red의 `lead`를 실행합니다.

```yaml
team:
  blue: {count: 1, brain: level-flight.xml, behavior: null}
  red: {count: 1, brain: lead, behavior: null}
training: {side: blue, control_mode: continuous}
simulation: {backend: jsbsim}
evaluation: {episodes: 1, render: false}
```

`python scripts/run_swarm.py`로 실행합니다. `training.side: red`로 바꾸고 두 Brain을
서로 바꾸면 반대편 조합도 시험할 수 있습니다. 기존 `Pursue`·`Break` XML 액션은
여전히 이산 setpoint를 출력합니다.

확인 테스트:

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests/test_swarm.py -q -k level_flight
```

테스트는 BT의 intent 출력, 처음 기울어진 JSBSim 기체의 날개 수평 복귀와 고도
변화, N:M 환경의 학습 측·고정 측 XML Brain 경로를 확인합니다. 이 테스트는
기초 제어 동작을 점검하는 것이지 공중전 성능이나 실제 전술을 검증하지는 않습니다.

## 정책의 직접 조종면 입력: 연속 제어

JSBSim 백엔드를 선택하고 `training.control_mode: continuous`로 바꾸면 학습 정책의
행동은 `[-1, 1]` 범위의 네 값입니다.

| 순서 | 입력 | 의미 |
|---|---|---|
| 1 | `roll` | 좌·우 롤 명령: `-1` 왼쪽, `+1` 오른쪽 |
| 2 | `pitch` | 피치 명령: `-1` 기수 아래, `+1` 기수 위 |
| 3 | `rudder` | 좌·우 방향타 명령 |
| 4 | `throttle` | `-1` 유휴 추력, `+1` 최대 추력 |

예를 들어 `[0.4, 0.2, 0.0, 0.5]`는 오른쪽 롤, 약한 기수 올림, 방향타 중립,
중간보다 높은 스로틀 입력입니다. 각 값은 직접 조종면 명령이지 “도 10만큼 선회”
같은 정확한 기동 결과가 아닙니다. 실제 자세·속도 변화는 JSBSim F-16 모델에서
계산됩니다.

```yaml
training:
  side: blue
  control_mode: continuous
simulation:
  backend: jsbsim
```

연속 RL 정책은 조종면 입력을 직접 출력합니다. 이와 별도로 위 BT 기동은 의도를
출력하고 내부 제어기가 조종면을 만듭니다. 이 모드에서 기존 `Discrete(27)` 액션도
호환 목적으로 받을 수 있지만 그 경우에는
오토파일럿 setpoint 경로를 탑니다. 현재 XML BT의 `Pursue`·`Break` 등은 이산
setpoint를 만들며 조종면 벡터나 `LevelFlight`와 같은 기동 의도를 출력하지 않습니다.
직접 제어 정책의 상태에는 기존 25개 채널에 자세각, 비행경로각, p/q/r 각속도
피드백이 추가됩니다.

BT에서 쓰는 주요 Action은 다음과 같습니다.

| Action | 현재 동작 |
|---|---|
| `Pursue` | 표적 방향으로 돌고, 거리·접근 속도에 따라 가속/감속하며, 표적 고도에 맞춥니다. |
| `Break` | 위협이 뒤에 가까이 있을 때 옆으로 급선회하고 가속·하강합니다. |
| `MatchUpPlaneTask` | 표적을 추격하면서 멀면 가속하고, 표적이 더 높으면 상승합니다. |
| `EnterTurnCircleTask` | 정면으로 마주치는 가까운 상황에서 옆으로 선회하고 상승합니다. |

현재 모델의 무장은 BT가 발사 버튼을 누르는 방식이 아닙니다. 표적이 사거리
150–1200 m 안에 있고 탄도 선행 조준선의 4° 안에 1초 이상 유지되면 자동으로
피해가 발생합니다. BT는 조준 조건을 만들도록 기동을 선택합니다.

## BT가 읽을 수 있는 상황 정보

XML Condition과 Python BT Action은 매 스텝 전달되는 교전 정보(`info`)를 읽습니다.
거리 단위는 m, 속도는 m/s, 각도는 radian입니다(아래 각도 설명은 이해를 위해
degree로도 함께 표시).

| 변수 | 뜻 | 규칙 예 |
|---|---|---|
| `has_target` | 배정된 표적이 살아 있는지 | 표적이 있을 때만 추격 |
| `range` | 내 기체와 표적 사이 거리 | `range < 1500`이면 방어 분기 확인 |
| `bearing` | 내 기수 기준 표적 방향. 오른쪽이 양수, 범위 `-π..π` | 양수면 오른쪽으로 선회 |
| `ata` | 내 속도 방향과 표적 시선의 각도. `0°`면 내 정면 | 표적이 내 정면에 가까운지 판단 |
| `aa` | 표적 속도 방향과 표적에서 나를 향하는 시선의 각도. `0°`면 내가 표적 뒤쪽, `180°`면 정면 대치 | `aa > 140°`면 위협이 내 뒤쪽인지 확인 |
| `ata_lead` | 표적의 현재 위치가 아니라 예측 위치 기준 조준 각도 | 사격 정렬 상태 확인 |
| `closure` | 서로 가까워지는 속도. 양수면 접근 중 | 너무 빠른 근접에서 감속 |
| `dh` | 표적 고도 - 내 고도 | 양수면 표적이 더 높음 |
| `own_v`, `own_h`, `own_phi` | 내 속도, 고도, 뱅크각 | 속도나 자세에 따라 행동 변경 |
| `target_v`, `target_health`, `rel_heading` | 표적 속도, 체력, 내 기수와 표적 기수의 상대 각도 | 표적 상태에 따른 분기 |
| `in_wez`, `track_time` | 현재 사격 가능 여부와 연속 추적 시간 | 사격 조건에 들어왔는지 확인 |
| `damage`, `damage_taken`, `kill`, `killed`, `collision` | 이번 스텝의 피해·격추·충돌 결과 | 피해를 받은 직후 회피 규칙 |
| `mate_range`, `mate_bearing`, `mates_on_target` | 가장 가까운 아군 거리·방향, 같은 표적을 맡은 아군 수 | 아군과 간격 유지, 표적 집중 확인 |
| `edge`, `t`, `t_max`, `targets_alive`, `allies_alive` | 경기장 가장자리 비율, 현재/최대 시간, 양 팀 잔존 비율 | 경계 복귀, 종료 임박 규칙 |

표적이 없을 때도 `has_target`은 `False`이며 거리·각도 등 표적 관련 수치는 0으로
채워집니다. 그러므로 표적 정보에 의존하는 규칙은 먼저 `has_target`을 확인해야
합니다. XML의 `RangeCheck`는 `range`를 m로 비교하고, `AspectCheck`는 `aa`를
degree 임계값과 비교합니다. 예를 들어 `<AspectCheck UpDown="Greater" Value="140"/>`
은 `aa > 140°`를 뜻합니다.

## 첫 번째 BT 수정: 방어 분기 임계값 바꾸기

`swarm_aircombat/bt/trees/default.xml`의 조건은 “1500 m보다 가깝고, 표적이 내 뒤쪽
140°보다 더 뒤에 있으면 `Break`; 아니면 `Pursue`”입니다.

```xml
<Fallback name="root">
  <Sequence name="defensive">
    <RangeCheck UpDown="Less" Value="1500"/>
    <AspectCheck UpDown="Greater" Value="140"/>
    <Break/>
  </Sequence>
  <Pursue/>
</Fallback>
```

`Value="1500"`을 `2500`으로 바꾸고 같은 시나리오를 실행해 보세요. 방어 분기가
더 먼 거리에서 선택되는지 리플레이의 BT trace로 확인할 수 있습니다. 이건 “더 좋은
전술”이라는 뜻이 아니라, 조건 변경이 실제 분기와 기체 명령에 반영되는지 보는
실습입니다. `xml_tree.py`가 지원하는 조건·Action 이름만 사용할 수 있습니다.

## XML BT는 어느 정도 지원하나

`swarm_aircombat/bt/trees/bfm.xml`은 XML 파일럿 트리의 한 예입니다.
이 프로젝트의 `xml_tree.py`가 지원하는 노드와 로컬 동작만 실행합니다.
따라서 임의의 BehaviorTree.CPP XML과 사용자 정의 노드를 모두 실행하는 범용 BT.CPP 런타임은 아닙니다.

트리에서 `DECO_BFMCheck`, `MatchUpPlaneTask` 등의 이름을 볼 수 있지만, 이름이 외부 구현을 그대로 포함한다는 의미는 아닙니다.
이 프로젝트는 필요한 기하 기반 분류와 동작을 자체 구현합니다.

## 안전한 첫 실습

먼저 같은 시나리오에서 코드형 규칙 파일럿과 BT 파일럿을 번갈아 실행합니다.

```yaml
team:
  blue:
    brain: lead
```

그다음 `bfm.xml`로 실행합니다. `lead`도 규칙 기반이고, `bfm.xml`은 규칙을 트리로
구성한 예입니다. “모듈이 로드되는 것”, “규칙 분기가 의도대로 실행되는 것”, “전투
성과가 좋은 것”은 서로 다른 확인 질문입니다.
BT 실행 성공만으로 BT가 더 잘 싸운다고 결론 내릴 수 없습니다.

## 직접 확인하는 방법

```bash
python scripts/compare.py --red-behavior mixed --policies bfm.xml lead --episodes 10
```

이 명령은 BT와 규칙 기반 `lead` 조종기를 같은 평가 도구로 비교합니다.
초기 비교 결과는 탐색 자료입니다. 충분한 시드·상대·반복 실험 없이 통계적 우열을 주장하지 않습니다.

## 도전 과제

1. `swarm_tree.py`에서 `near_boundary` 조건과 `turn_to_center` 행동을 찾아봅니다.
2. 어떤 조건이 실패하면 Selector가 다음 가지로 넘어가는지 손으로 추적합니다.
3. `bfm.xml`에서 읽는 노드 이름을 `xml_tree.py`의 지원 목록과 대조합니다.
4. 트리를 바꿨다면 스모크 테스트와 결과 리플레이로 실행 경로가 실제 달라지는지 확인합니다.

다음: [강화학습으로 기동 정책 배우기](04-reinforcement-learning.md)
