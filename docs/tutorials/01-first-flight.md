# 1. 첫 비행: 설정을 바꾸고 시뮬레이터 실행

**완료 기준:** 전투 시나리오를 실행하고, 어떤 파일럿이 조종했는지와 항공기가 어떻게 움직였는지 리플레이에서 찾아봅니다.

## 1) 환경 준비

저장소 루트에서 Python 환경을 만들고 기본·학습·개발 의존성을 설치합니다.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -e ".[train,dev]"
```

Windows PowerShell은 `.venv\Scripts\Activate.ps1`로 환경을 활성화합니다.
처음에는 JSBSim 없이 기본 `pointmass` 백엔드로 시작합니다.

## 2) 설정에서 전투 시나리오 찾기

열어볼 파일은 저장소 루트의 `combat.yaml`입니다.

```yaml
team:
  blue:
    count: 4
    assigner: cbba
    brain: bfm.xml
  red:
    count: 20
    assigner: cbba
    behavior: mixed
simulation:
  backend: pointmass
evaluation:
  episodes: 30
  render: true
```

| 키 | 역할 |
|---|---|
| `team.blue.count`, `team.red.count` | 아군/적군 항공기 수 (N:M 전투) |
| `team.*.brain` | 사용할 파일럿: 규칙 기반, BT/XML BT 또는 호환 PPO 체크포인트 |
| `team.red.behavior` | `brain`이 비어 있을 때 사용할 적기 규칙 기반 행동 |
| `team.blue.brains` | 선택적 기체별 고정 blue 파일럿 목록 |
| `simulation.backend` | 학습 기본 `pointmass` 또는 선택형 JSBSim F-16 모델 |
| `team.*.assigner`, `assignment` | 여러 기체의 표적 배분 (첫 학습에서는 생략 가능) |
| `evaluation` | 실행 판수, 리플레이 표시 여부와 HTML 출력 경로 |

`config.xml`은 기존 1v1 `run.py` 설정으로 유지합니다. 구형 `<swarm>` XML도 호환되므로
`python scripts/run_swarm.py --config config.xml`로 실행할 수 있습니다.
새 군집 설정은 YAML에서 관리합니다.

## 3) 기본 설정 실행

```bash
python scripts/run_swarm.py
```

설정대로 에피소드를 실행합니다. `evaluation.render: true`이면 실행 결과를 살펴볼
인터랙티브 HTML 리플레이도 만듭니다.
리플레이를 생략하고 빠르게 설정만 시험하려면 `combat.yaml`에서 다음처럼 바꾸세요.

```yaml
evaluation:
  episodes: 1
  render: false
```

다른 설정 파일도 지정할 수 있습니다.

```bash
python scripts/run_swarm.py --config path/to/my-combat.yaml
```

## 4) 파일럿 하나만 바꾸기

먼저 같은 시나리오에서 파일럿을 바꿔 봅니다. 예를 들어:

```yaml
team:
  blue:
    brain: lead
```

`lead`는 코드로 작성한 규칙 기반 추격 파일럿입니다. 이를 `bfm.xml`로 바꾸면
Behavior Tree 기반 파일럿이 됩니다. 두 번 실행한 뒤 리플레이에서 선회·고도 변화와
표적 접근이 어떻게 달라지는지 찾아보세요. 한 판의 승패만으로 어느 파일럿이
더 뛰어나다고 결론 내리지는 않습니다.

## 5) 표적 배분은 보조 계층

`team.*.assigner`는 “누가 어느 적을 맡을지” 정합니다. “그 적을 향해 어떻게 비행할지”를
결정하는 파일럿과는 다른 기능입니다. 첫 실습에서는 기본값을 그대로 두고,
표적 배분 실험은 [선택 확장 5장](05-assignment-algorithms.md)에서 진행합니다.

## 6) 결과 확인

- 터미널의 설정 요약에서 blue/red의 수, Brain, 할당기가 맞는지 봅니다.
- `evaluation.output`에 리플레이가 기록됐는지 확인합니다.
- 리플레이에서 파일럿 결정과 항공기 움직임을 함께 봅니다. 명령 변화가 곧바로 좋은 전투 성과를 뜻하는 것은 아닙니다.

## 실습

1. `team.blue.brain`을 `lead`와 `bfm.xml`로 각각 실행합니다.
2. 두 리플레이에서 항공기의 기동과 BT 실행 경로가 어떻게 다른지 적습니다.
3. 어떤 설정이 비행 행동을, 어떤 설정이 추격할 상대를 바꾸는지 설명합니다.

다음: [프로젝트 구조와 군집전의 의미](02-project-and-swarm.md)
