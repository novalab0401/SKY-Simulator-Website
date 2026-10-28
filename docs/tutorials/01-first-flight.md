# 1. 첫 비행: 설정을 바꾸고 시뮬레이터 실행

**완료 기준:** 설정 파일에서 편대 규모와 할당기를 찾아 설명하고, 직접 실행한 뒤 로그에 적용된 선택이 표시되는 것을 확인합니다.

## 1) 환경 준비

저장소 루트에서 Python 환경을 만들고 기본·학습·개발 의존성을 설치합니다.

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -e ".[train,dev]"
```

Windows PowerShell은 `.venv\Scripts\Activate.ps1`로 환경을 활성화합니다.
처음에는 JSBSim 없이 기본 `pointmass` 백엔드로 시작합니다.

## 2) 설정에서 시나리오 찾기

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
| `team.blue.count`, `team.red.count` | 아군/적군 항공기 수 |
| `team.*.assigner` | 해당 편대가 사용할 임무 할당 알고리즘 |
| `team.*.brain` | 사용할 파일럿. `null`이면 red의 `behavior` 사용 |
| `team.blue.brains` | 선택적 기체별 고정 blue 파일럿 목록 |
| `simulation.backend` | `pointmass` 또는 선택적 `jsbsim` |
| `assignment` | 거리 감지, 통신, 비용·합의 설정 |
| `evaluation` | 비교 판수, 화면 표시 여부, HTML 출력 경로 |

`config.xml`은 기존 1v1 `run.py` 설정으로 유지합니다. 구형 `<swarm>` XML도 호환되므로
`python scripts/run_swarm.py --config config.xml`로 실행할 수 있습니다.
새 군집 설정은 YAML에서 관리합니다.

## 3) 기본 설정 실행

```bash
python scripts/run_swarm.py
```

설정대로 30 에피소드를 평가하고 `evaluation.render: true`이면 HTML 리플레이를 만듭니다.
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

## 4) 같은 설정에서 하나만 바꾸기

아군 규모를 4대로, 적군을 20대로 두고 할당기만 바꿔 봅니다.

```yaml
team:
  blue:
    count: 4
    assigner: hungarian
  red:
    count: 20
    assigner: cbba
```

선택 가능한 이름은 `hungarian`, `greedy`, `first_claimed`, `cbba`, `grape`입니다.
잘못된 YAML 키는 설정 로더가 오류로 알려주고, 모르는 알고리즘 이름도 실행 전에 거부합니다.

## 5) 결과 확인

- 터미널의 설정 요약에서 blue/red의 수, Brain, 할당기가 맞는지 봅니다.
- `evaluation.output`에 리플레이가 기록됐는지 확인합니다.
- **이 한 판으로 알고리즘 성능을 결론내리지 않습니다.** 알고리즘 비교는 [7장](07-evaluate-and-extend.md)의 동일 시드 다판 비교를 사용합니다.

## 실습

1. `team.red.behavior`를 `circular`로 바꾸고 1판 실행합니다.
2. `team.blue.brain`을 `lead`로 바꾸고 다시 실행합니다.
3. 어떤 설정은 비행 행동에, 어떤 설정은 누가 누구를 추격할지에 영향을 주는지 설명해 봅니다.

다음: [프로젝트 구조와 군집전의 의미](02-project-and-swarm.md)
