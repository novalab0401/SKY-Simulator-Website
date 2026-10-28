# 3. Behavior Tree로 규칙 표현하기

**완료 기준:** Sequence와 Selector의 중단 규칙을 설명하고, XML 트리의 변경이 지원되는 범위인지 확인합니다.

## BT가 하는 일

Behavior Tree는 매 결정 스텝에 조건을 확인하고 행동을 고르는 트리입니다.
이 프로젝트에는 자체 경량 엔진이 있으며 BT가 직접 항공기 물리를 계산하거나 임무 할당을 하지는 않습니다.

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

## 군집 BT 한눈에 보기

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
- 그렇지 않으면 아군과 가까운 기체의 고도를 분리한 뒤 할당된 표적을 추격합니다.
- 표적이 없으면 선회 대기합니다.

`rl_pursue`는 정책 액션을 사용하라는 결정을 Blackboard에 남깁니다.
경계 복귀와 대기 분기는 정책 액션을 사용하지 않고 직접 명령을 지정할 수 있습니다.
각 노드는 `bb.trace`에 기록되며 시각화에서 활성 분기를 확인할 수 있습니다.

## XML BT는 어느 정도 지원하나

`swarm_aircombat/bt/trees/bfm.xml`은 XML 파일럿 트리의 한 예입니다.
이 프로젝트의 `xml_tree.py`가 지원하는 노드와 로컬 동작만 실행합니다.
따라서 임의의 BehaviorTree.CPP XML과 사용자 정의 노드를 모두 실행하는 범용 BT.CPP 런타임은 아닙니다.

트리에서 `DECO_BFMCheck`, `MatchUpPlaneTask` 등의 이름을 볼 수 있지만, 이름이 외부 구현을 그대로 포함한다는 의미는 아닙니다.
이 프로젝트는 필요한 기하 기반 분류와 동작을 자체 구현합니다.

## 안전한 첫 실습

먼저 새 BT 행동을 작성하지 말고, 기존 트리를 다른 스펙으로 바꿔 실행합니다.

```yaml
team:
  blue:
    brain: lead
```

그다음 `bfm.xml`로 되돌리고 실행합니다. “모듈이 로드되는 것”과 “전투 성능이 좋은 것”은 다른 검증 질문입니다.
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
