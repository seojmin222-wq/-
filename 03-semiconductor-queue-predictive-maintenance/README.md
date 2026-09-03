<details>
<summary><b>03. 주요 구현 내용</b></summary>

<br>

<details>
<summary><b>3-1. Q-Time 위반 예방 자동 Routing Logic 기획</b></summary>

<br>

프로젝트 착수 단계에서 **Q-Time이 임박한 Lot을 자동으로 우선 처리하는 Routing 방식**을 제안했습니다.

단순히 Q-Time 초과 시 Alarm을 발생시키는 것이 아니라, PLC가 각 Lot의 잔여 시간을 비교하고 위험 Lot이 발생하면 새로운 공급을 제한한 뒤 해당 Lot을 먼저 배출하는 구조를 목표로 했습니다.

#### 기본 제어 흐름

```text
Lot 투입
   ↓
Lot별 Q-Time 측정
   ↓
잔여 시간 비교
   ↓
위험 Lot 판정
   ↓
신규 공급 제한
   ↓
위험 Lot 우선 이송 / 배출
   ↓
정상 Sequence 복귀
