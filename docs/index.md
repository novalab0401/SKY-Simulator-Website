# SKY Simulator Lab

## 코드를 읽고, 직접 바꾸고, 실험으로 확인하기

이 사이트는 군집 공중전 프로젝트를 처음 접하는 개발자를 위한 실습형 안내서입니다.
Behavior Tree(BT), 강화학습(RL), 다대다(N:M) 전투와 분산 표적 할당을 기초부터 익힌 다음,
직접 설정과 알고리즘을 변경하고 테스트로 검증합니다. 실제 코드와 결과는
[SKY-Simulator GitHub 저장소](https://github.com/novalab0401/SKY-Simulator)에서 확인하세요.

> **이 프로젝트를 한 문장으로:** 표적 할당은 “누가 누구를 맡을지” 정하고, BT는 “지금 어떤 행동이 필요한지” 판단하며, 조종 정책은 “어떻게 기동할지” 선택합니다. point-mass/JSBSim 물리 모델이 명령을 실행합니다.

## 추천 학습 경로

| 순서 | 배우는 것 | 결과물 |
|---|---|---|
| 1. [첫 비행](tutorials/01-first-flight.md) | 설치, `combat.yaml`, 실행, 결과 보기 | 내 설정으로 4v20 실행 |
| 2. [프로젝트와 군집전](tutorials/02-project-and-swarm.md) | N:M, 분산형의 의미, 코드 지도 | 실행 흐름을 파일 경로로 설명 |
| 3. [Behavior Tree](tutorials/03-behavior-trees.md) | Selector, Sequence, Condition, Action | 기존 트리 분기와 XML 변경 확인 |
| 4. [강화학습](tutorials/04-reinforcement-learning.md) | 관측, 행동, 보상, PPO | 학습과 체크포인트 평가 |
| 5. [분산 표적 할당](tutorials/05-assignment-algorithms.md) | Hungarian부터 CBBA/GRAPE까지 | 설정에서 알고리즘 교체 비교 |
| 6. [직접 개발하기](tutorials/06-build-a-plugin.md) | 새 할당기와 BT 행동 연결 | 새 모듈과 테스트 작성 |
| 7. [검증과 확장](tutorials/07-evaluate-and-extend.md) | 공정한 비교, 결과 해석, 실험 기록 | 재현 가능한 결과 묶음 |

각 장은 **먼저 실행 → 코드에서 확인 → 한 가지 바꾸기 → 테스트/결과로 확인** 순서입니다.
수치가 기대와 다를 때도 실패가 아니라 관측 결과로 기록하고, 승률만으로 알고리즘을 섣불리 평가하지 않습니다.

## 지금 프로젝트가 하는 것과 하지 않는 것

- N:M 공중전, XML/Python BT 파일럿, 규칙 기반 파일럿, PPO 정책을 실행합니다.
- 표적 할당 알고리즘은 설정에서 바꿉니다. `combat.yaml`의 기본 시나리오는 아군 4대 대 적기 20대입니다.
- CBBA/GRAPE는 이 환경의 제한된 단일 표적 할당 문제에 맞춘 변형입니다. 외부 논문/저장소 구현과 동일하다고 주장하지 않습니다.
- 분산은 임무 할당의 정보·결정 구조에 적용됩니다. 현재 시뮬레이터의 물리 업데이트와 점수 집계까지 분산 실행하는 것은 아닙니다.
- 체크인된 `models/pursuit_mixed.zip`은 현재 `Discrete(27)` 행동 공간과 호환되지 않는 구형 체크포인트입니다. 새 실험에서는 재학습한 모델을 사용합니다.

## 사이트를 로컬에서 보기

```bash
pip install -e ".[docs]"
mkdocs serve
```

브라우저에서 `http://127.0.0.1:8000`을 엽니다. 정적 사이트를 빌드하려면 `mkdocs build --strict`를 실행합니다.
게시하려면 저장소의 GitHub Pages 설정을 활성화한 뒤 `mkdocs gh-deploy`를 실행할 수 있습니다.

## 바로 시작

프로젝트 루트에서:

```bash
python scripts/run_swarm.py
```

첫 장에서는 이 명령이 무엇을 실행하는지, `combat.yaml`에서 아군·적군·Brain·할당기를 어떻게 선택하는지 차례대로 확인합니다.
