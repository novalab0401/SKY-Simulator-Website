# 용어 사전

| 용어 | 뜻 |
|---|---|
| Agent | PettingZoo 환경에서 학습 행동을 받는 기체. 이 프로젝트는 한 번에 한 팀만 trainable roster로 둡니다. |
| Assigner | 편대 기체마다 추격 표적을 정하는 함수 |
| Behavior Tree (BT) | Condition/Action/Composite 노드의 성공·실패로 행동을 선택하는 계층 구조 |
| Blackboard | 현재 기체·표적 등 BT 노드가 함께 읽고 결정을 기록하는 스텝별 데이터 |
| Brain | `info -> Discrete(27) action` 인터페이스를 제공하는 파일럿. 규칙, BT, XML tree, PPO 등이 등록됩니다. |
| CBBA-style | 입찰 효용과 이웃 consensus를 적용한 이 프로젝트의 단일 표적 할당 적응 |
| Episode | reset부터 승패/시간 제한 종료까지의 시뮬레이션 한 판 |
| Hungarian | 중앙에서 전체 비용 행렬을 최소화하는 할당 기준 알고리즘 |
| Observation | 정책 입력으로 바뀐 현재 교전 상태 |
| PPO | 정책과 가치 함수를 업데이트하는 on-policy 강화학습 알고리즘 |
| Reward | 한 스텝의 행동이 목표에 얼마나 기여했는지 표현하는 학습 신호 |
| Seed | 난수 생성 초기값. 같은 시드의 paired episode는 같은 초기 배치를 재사용합니다. |
| Selector | 자식 중 첫 `FAILURE`가 아닌 결과를 반환하는 BT 합성 노드 |
| Sequence | 모든 자식이 `SUCCESS`여야 성공하며 처음 실패한 자식에서 멈추는 합성 노드 |
| WEZ | 현재 모델이 사격 조건으로 사용하는 무장 교전 영역 |
| Paired comparison | 동일 초기 시드·조건에서 비교 대상만 바꾸는 실험 설계 |
| 분산 할당 | 기체별 감지·이웃 통신에 기반해 임무 할당을 모의하는 계층. 중앙 물리 시뮬레이션과 구분합니다. |
