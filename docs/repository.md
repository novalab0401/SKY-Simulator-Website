# 시뮬레이터 소스 저장소 안내

튜토리얼 사이트와 Python 시뮬레이터는 **별도 프로젝트**입니다.
사이트는 개념·실습·개발 절차를 설명하고, 실제 구현 코드는 `swarm-aircombat` 저장소에 둡니다.

## 함께 체크아웃하기

사이트와 코드 저장소를 같은 `src/` 폴더에 체크아웃하면 튜토리얼을 따라가기 쉽습니다.

```text
src/
├── swarm-aircombat/
│   ├── combat.yaml
│   ├── scripts/
│   └── swarm_aircombat/
└── swarm-aircombat-site/
    ├── mkdocs.yml
    └── docs/
```

코드 저장소 주소가 아직 공개 설정되어 있지 않으므로, `swarm-aircombat` 저장소는 프로젝트를 제공받은 주소/경로에서 별도로 준비합니다. 사이트 자체는 simulator 코드를 복제하거나 Python 패키지에 포함하지 않습니다.
공개 코드 저장소는 [novalab0401/SKY-Simulator](https://github.com/novalab0401/SKY-Simulator)입니다.

## 실습에서 사용하는 실제 코드 경로

저장소 루트 `swarm-aircombat/` 기준 경로입니다.

| 학습 내용 | 실제 코드 |
|---|---|
| YAML 팀·할당 선택 | [`combat.yaml`](https://github.com/novalab0401/SKY-Simulator/blob/main/combat.yaml), [`scripts/run_swarm.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/scripts/run_swarm.py) |
| 환경, reset, step, 할당·BT 연결 | [`swarm_pursuit.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/envs/swarm_pursuit.py) |
| BT 상태/Sequence/Selector | [`bt/core.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/bt/core.py) |
| N:M 안전·추격 트리 | [`bt/swarm_tree.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/bt/swarm_tree.py) |
| XML 파일럿 정의 | [`bt/trees/`](https://github.com/novalab0401/SKY-Simulator/tree/main/swarm_aircombat/bt/trees) |
| Brain 선택 | [`bt/brain.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/bt/brain.py) |
| Hungarian | [`assignment/hungarian.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/assignment/hungarian.py) |
| Greedy | [`assignment/greedy.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/assignment/greedy.py) |
| First-Claimed | [`assignment/first_claimed.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/assignment/first_claimed.py) |
| CBBA-style | [`assignment/cbba.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/assignment/cbba.py) |
| GRAPE-style | [`assignment/grape.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/assignment/grape.py) |
| 공통 비용과 통신 합의 helper | [`assignment/cost.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/assignment/cost.py), [`consensus.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/assignment/consensus.py) |
| 관측·보상·PPO/SAC 및 학습 연결 | [`rl/state.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/rl/state.py), [`reward.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/rl/reward.py), [`policy.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/rl/policy.py), [`rl/vec.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/rl/vec.py) |
| JSBSim 연속 조종 입력 | [`core/aircraft.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/core/aircraft.py), [`envs/swarm_pursuit.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/swarm_aircombat/envs/swarm_pursuit.py) |
| 회귀 테스트 | [`tests/test_swarm.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/tests/test_swarm.py), [`tests/test_distributed_assignment.py`](https://github.com/novalab0401/SKY-Simulator/blob/main/tests/test_distributed_assignment.py) |

사이트의 예제 명령은 시뮬레이터 저장소 루트에서 실행하세요. Python 환경 설치가 되어 있어야 합니다.

## 코드 링크

튜토리얼에는 실제 코드 파일 경로를 적고, 위 공개 저장소를 코드 기준점으로 연결합니다. 각 GitHub 파일 링크는 현재 기본 브랜치 경로를 사용합니다. 사이트 자체는 별도 저장소로 공개할 예정입니다.
