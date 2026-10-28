# 5. 분산 표적 할당 알고리즘

**완료 기준:** 5개 할당기 중 하나를 `combat.yaml`에서 바꾸고, 동일한 시드 배치로 비교하는 방법을 실행합니다.

## 할당기는 비행 조종기가 아니다

할당기는 기체별 표적 인덱스를 반환합니다. 같은 입력을 받더라도 알고리즘별 선택은 달라질 수 있습니다.
heading/speed/altitude 행동은 Brain/BT 쪽 책임입니다.

| YAML 이름 | 구현 파일 | 동작 모델 |
|---|---|---|
| `hungarian` | `swarm_aircombat/assignment/hungarian.py` | 전체 비용행렬 중앙 최적화 기준선 |
| `greedy` | `swarm_aircombat/assignment/greedy.py` | 가장 싼 남은 기체-슬롯 쌍을 순차 선택 |
| `first_claimed` | `swarm_aircombat/assignment/first_claimed.py` | 로컬 제안과 결정적 우선순위 기반 분산 할당 |
| `cbba` | `swarm_aircombat/assignment/cbba.py` | 비용을 입찰값으로 사용하고 승리 정보를 이웃과 교환하는 적응 |
| `grape` | `swarm_aircombat/assignment/grape.py` | CBBA 시작 배정 후 로컬 정보로 편대 개선을 시도 |

공통 비용 모델과 `AssignmentConfig`는 `assignment/cost.py`에 있습니다.
분산 경매의 이웃 그래프와 메시지 전달 유틸은 `assignment/consensus.py`, 이름 등록은 `assignment/registry.py`입니다.

이 코드는 각 알고리즘의 아이디어를 현재 문제에 맞춘 **시뮬레이터 내 적응**입니다.
논문 구현이나 다른 저장소의 알고리즘과 완전히 동일하다고 간주하면 안 됩니다.

## 설정에서 양 팀의 선택을 각각 바꾸기

```yaml
team:
  blue:
    assigner: cbba
  red:
    assigner: hungarian
assignment:
  communication_radius: 10000.0
  sensing_radius: 25000.0
  consensus_rounds: null
```

`communication_radius`는 누가 메시지를 주고받을 수 있는지, `sensing_radius`는 각 기체가 어떤 표적을 볼 수 있는지 제한합니다.
`consensus_rounds: null`은 팀 규모에 비례한 기본 합의 라운드를 사용합니다.
통신 그래프가 끊기면 서로 다른 연결 컴포넌트의 기체가 동일 표적을 선택할 수 있습니다.
전역 복구 단계를 숨겨 넣어 분산처럼 보이게 하지 않습니다.

## 같은 초기 배치에서 여러 알고리즘 비교

```bash
python scripts/compare_assigners.py \
  --n-blue 4 --n-red 20 \
  --red-behavior mixed --red-assigner hungarian \
  --policy bfm.xml \
  --assigners hungarian greedy first_claimed cbba grape \
  --episodes 30 --seed0 10000 \
  --out-dir docs/results/tutorial-assigners
```

같은 에피소드 seed를 각 알고리즘에 재사용하므로 비교 조건이 짝지어집니다.
seed마다 시작 위치는 달라지지만, 특정 seed의 모든 알고리즘은 같은 시작 배치를 사용합니다.
결과 디렉터리에는 JSON/CSV, 초기 배정 시각화와 알고리즘별 리플레이를 저장할 수 있습니다.

실험 결과를 읽을 때 주의할 점:

1. 한 알고리즘만 바꿨는지 설정을 확인합니다.
2. 승률뿐 아니라 적 격추, 아군 손실, 충돌, 할당 변화, 미할당 비율을 함께 봅니다.
3. 모든 승률이 0이면 승률로 알고리즘을 순위 매길 수 없습니다.
4. 30판은 탐색 비교이지 보편적인 우열 증명이 아닙니다.
5. 원자료와 명령줄을 결과 묶음에 보존합니다.

## 연습 문제

- Greedy와 Hungarian이 다른 결정을 할 수 있는 작은 비용 행렬을 찾아 설명합니다.
- 감지 반경을 줄이고 `unassigned_rate`와 중복 할당 변화를 관찰합니다.
- 한 seed의 서로 다른 배정이 곧 전투에서 더 나은 결과를 뜻하는지 토론합니다.

다음: [새 알고리즘과 BT를 직접 개발하기](06-build-a-plugin.md)
