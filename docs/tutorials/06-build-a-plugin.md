# 6. 직접 개발하기: 새 할당기와 BT 행동

**완료 기준:** 알고리즘 인터페이스, 레지스트리, 환경 연결 위치와 최소 회귀 테스트를 찾고 새 코드를 추가합니다.

## A. 새 할당 알고리즘 추가하기

각 알고리즘은 `swarm_aircombat/assignment/` 아래 **개별 파일**에 구현합니다.
예를 들어 `novel.py`를 추가하고 기존 플러그인과 같은 callable 계약을 따릅니다.

```python
from ..core.aircraft import AircraftState
from .cost import AssignmentConfig


def novel(agents: list[AircraftState],
          targets: list[AircraftState],
          alive: list[bool],
          health: list[float],
          current: list[int | None],
          cfg: AssignmentConfig) -> list[int | None]:
    # 각 agent에 대해 표적 index를 반환. 표적이 없으면 None.
    ...
```

반환 목록은 입력 기체 수와 같아야 합니다. 할당 값은 표적 인덱스 또는 `None`이어야 합니다.
기존 모델을 재사용하고 싶으면 `cost.py`의 `_cost_matrix`/`_slots` 또는 `consensus.py`의 경매 유틸을 살펴보세요.
새 알고리즘의 핵심은 계속 새 파일 안에 둡니다.

그 다음 `swarm_aircombat/assignment/registry.py`에서 새 구현을 가져와 `ASSIGNERS`에 이름을 추가합니다.
환경에는 이미 레지스트리를 거치는 연결부가 있으므로, 이 확장 방식은 새 YAML 선택을 즉시 가능하게 합니다.

```python
from .novel import novel

ASSIGNERS = {
    # 기존 이름들...
    "novel": novel,
}
```

`combat.yaml`의 주석에 선택지를 업데이트하고 다음처럼 지정합니다.

```yaml
team:
  blue:
    assigner: novel
```

## B. 먼저 검증 테스트 작성하기

권장 테스트 확인 항목:

- 입력 에이전트마다 반환값이 하나씩 있는가?
- 죽은 표적을 반환하지 않는가?
- `None` 반환과 센싱 범위 제한을 의도대로 처리하는가?
- 가능한 통신 그래프에서 capacity 제약을 지키는가?
- 플러그인 등록 이름으로 `registry.load("novel")`이 구현을 반환하는가?
- 이 알고리즘 선택이 반대편 팀 설정에는 영향을 주지 않는가?

테스트 파일은 `tests/test_distributed_assignment.py`의 구조를 참고하고, 짧은 반복 확인은:

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 .venv/bin/python -m pytest -q \
  tests/test_distributed_assignment.py tests/test_swarm.py
```

## C. BT는 세부 계층이 다르다

군집 행동 트리 실행 흐름은 `swarm_aircombat/bt/swarm_tree.py`,
노드 계약은 `swarm_aircombat/bt/core.py`에 있습니다.
새 Condition/Action은 Blackboard 입력과 `Status` 반환 규칙을 명확히 해야 합니다.

```python
from .core import Action, Status

def my_action(bb):
    # bb에서 필요한 상태를 읽고 bb.decision을 설정
    return Status.SUCCESS

node = Action("my_action", my_action)
```

단, 현재 XML loader가 임의의 Python 노드 클래스를 외부에서 자동 발견하지는 않습니다.
Python 군집 트리 변경은 `build_tree()`를 고치고 분기 테스트를 추가하는 쪽이 간단합니다.
XML 트리 파일을 추가/변경할 때는 `xml_tree.py`의 지원 태그와 파라미터 검증을 따라야 합니다.

새 BT 규칙을 추가할 때 테스트로 확인할 것:

1. 참/거짓 Condition에서 기대한 `Status`를 반환하는가?
2. `Selector` 우선순위가 기존 안전 분기를 깨지 않는가?
3. Action이 설정하는 결정이 환경이 이해하는 형식인가?
4. 해당 분기가 선택됐다는 `bb.trace` 또는 리플레이 증거가 있는가?
5. 전투 성능 주장은 별도의 다판 평가로 뒷받침되는가?

## 코드 변경의 최소 끝맺음

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 .venv/bin/python -m pytest -q
git diff --check
```

새 코드를 추가했다면 문서의 구조표, YAML의 선택 목록, 테스트 이름과 결과 재현법도 함께 갱신합니다.

다음: [평가 결과를 올바르게 읽고 확장하기](07-evaluate-and-extend.md)
