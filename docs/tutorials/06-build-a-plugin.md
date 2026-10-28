# 선택 확장 B. 사용자 플러그인과 직접 개발

**선행 권장:** 단일 기체 조종의 규칙·BT·강화학습 학습 경로를 먼저 살펴보고,
여러 기체의 임무 할당을 확장하고 싶을 때 진행하세요.

**완료 기준:** 중앙형 또는 에이전트별 할당 플러그인 계약을 이해하고, 작은 예제를 설정에서 로드해 테스트합니다.

## 먼저 경계 이해하기

현재 외부 플러그인 계약은 **표적 할당기**를 위한 것입니다. `brain`에 제3자 파일럿을
바로 연결하는 일반 플러그인 계약은 아닙니다. 규칙/BT 조종을 직접 수정하려면
`swarm_aircombat/bt/brain.py`, `swarm_aircombat/bt/swarm_tree.py`와
`swarm_aircombat/bt/trees/`를 살펴보세요.

할당 알고리즘의 점수·입찰·합의 방식은 작성자가 정합니다. 시뮬레이터는 입력과
결과 형식을 제공하고, 반환된 상대 표적 인덱스가 유효한지 확인합니다.

## 중앙형 플러그인

중앙형 알고리즘은 전체 편대 문제를 한 번에 받고 각 편대 기체에 대응하는 적 표적
인덱스 또는 `None`을 반환합니다.

`combat.yaml`에서 Python 파일 경로는 YAML 파일이 있는 디렉터리를 기준으로 합니다.

```yaml
team:
  blue:
    assigner:
      type: central
      callable: my_assignment.py:assign
      config: {}
```

```python
def assign(problem, config):
    alive_targets = [
        j for j, alive in enumerate(problem.target_alive) if alive
    ]
    target = alive_targets[0] if alive_targets else None
    return [target for _ in problem.agents]
```

`problem`에는 `agents`, `targets`, `target_alive`, `target_health`,
`current_targets`, `config`가 있습니다. 반환값은 friendly aircraft마다 하나씩
있어야 하며, target index는 opposing team의 인덱스입니다. `None`은 표적 미할당을 뜻합니다.

## 에이전트별 분산형 플러그인

이 방식은 각 기체마다 `Plugin(agent)` 객체를 만들고 재할당 시점마다
`decide(blackboard)`를 호출합니다.

```yaml
team:
  red:
    assigner:
      type: agent
      class: my_allocator.py:MyAllocator
      config: {}
```

```python
class MyAllocator:
    def __init__(self, agent):
        self.agent = agent

    def decide(self, blackboard):
        tasks = blackboard["local_tasks_info"]
        if not tasks:
            return None

        chosen = min(
            tasks,
            key=lambda task: self.agent.position.distance_to(task.position),
        )
        self.agent.message_to_share = {"agent_id": self.agent.agent_id}
        return chosen.task_id
```

blackboard에는 `agent_id`, `local_tasks_info`, `local_agents_info`,
`current_task_id`, `current_target_id`, `assignment_config`가 들어 있습니다.
`local_tasks_info`는 해당 기체가 감지한 살아 있는 표적 목록이며 `task_id`는 상대 편대
내 인덱스입니다. 에이전트 컨텍스트의 `messages_received`에서 통신 이웃의 이전
할당 주기 메시지를 읽고, 다음 메시지는 `message_to_share`에 기록합니다.

플러그인 인스턴스는 에피소드마다 새로 생성됩니다. 메시지는 재할당 주기마다 전달되는
동기식 스냅샷으로, 지연·패킷 손실 모델은 없습니다. 이 계약은 알고리즘을 연결하는
기능이지 논문 충실도를 확인해주는 검증 도구가 아닙니다.

## 무엇을 테스트할까

- 플러그인 참조가 로드되는가?
- 중앙형은 기체 수와 같은 길이의 valid target index/`None` 목록을 반환하는가?
- 에이전트형은 감지하지 못한 표적을 반환하지 않는가?
- 메시지 없는 경우, 죽은 표적, 통신이 끊긴 경우를 의도대로 처리하는가?
- 알고리즘 규칙을 작은 손계산 예제와 테스트로 확인했는가?

```bash
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 .venv/bin/python -m pytest -q \
  tests/test_distributed_assignment.py tests/test_swarm.py
```

테스트 통과는 계약과 작성한 사례에서 코드가 기대대로 동작한다는 근거입니다.
그 자체가 알고리즘의 우수성, 현실 전투 효과 또는 논문 재현을 증명하지는 않습니다.

주 조종 학습으로 돌아가기: [Behavior Tree](03-behavior-trees.md) ·
[강화학습](04-reinforcement-learning.md)
