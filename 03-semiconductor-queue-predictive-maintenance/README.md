# 큐타임 위반 예방 자동 라우팅 & O-ring 열화 예지보전 시스템

### PLC · SCADA · DB · 데이터분석 4-Layer 통합 스마트팩토리 시스템

> **반도체 공정의 Q-Time 초과와 Chamber O-ring 열화 문제를 대상으로
> 현장제어 → SCADA → DB → 데이터분석까지 연결한 4-Layer 통합 시스템**

> 🚧 **진행 상태**: 마무리 단계 — PLC 최종 동작 및 병렬 시퀀스 검증 진행 중

* **기간**: 2026.08 ~ 진행 중
* **참여 형태**: 팀 프로젝트 (4인)
* **역할**: 팀장
* **주요 기여**: Q-Time 위반 예방 자동 라우팅 아이디어 및 PLC 시퀀스 최초 제안 · 전체 아키텍처 구체화 · 역할 배분 및 일정관리 · MPS 하드웨어 재구성 · PLC/Servo 배선 · iFIX 설비 개요도 작화 · 3D Printing 부품 설계 총괄
* **주요 기술**: MELSEC PLC, MR-J5 Servo, GOT HMI, iFIX, OPC-UA, SQLite, Python, pandas, LSTM, 3D Printing

---

## 시스템 구성

<p align="center">
  <img src="images/03-architecture.png" alt="4-Layer 시스템 아키텍처" width="90%">
</p>

| Layer                  | 구성 요소                            | 역할                                           |
| ---------------------- | -------------------------------- | -------------------------------------------- |
| **1. Field / Control** | MELSEC PLC, MR-J5 Servo, GOT HMI | Lot·압력·Q-Time 데이터 처리, 위험등급 판정, 우선 이송 및 현장 제어 |
| **2. SCADA**           | iFIX, OPC-UA                     | Q-Time Heatmap, O-ring 상태, KPI 및 설비 상태 시각화   |
| **3. Database**        | SQLite                           | Lot·시간대·Chamber별 공정 이력 저장                    |
| **4. Analysis**        | Python, pandas, LSTM             | 병목 구간 및 압력 변화 분석, 열화 패턴 분석 및 보전 주기 도출        |

* **Q-Time 제어 흐름**

  `Lot 투입 → Q-Time Monitoring → 위험도 판정 → 공급 Interlock → 위험 Lot 우선 배출`

* **O-ring 분석 흐름**

  `Chamber 압력 데이터 → SQLite → Python/pandas → LSTM 시계열 분석 → 열화 패턴 및 보전 시점 분석`

---

<details>
<summary><b>01. 프로젝트 개요 및 문제 정의</b></summary>

<br>

반도체 공정에서는 특정 공정 사이의 **Q-Time을 초과할 경우 Lot 폐기 또는 품질 저하가 발생할 수 있고**, Chamber 내부의 O-ring 열화를 적절한 시점에 감지하지 못하면 압력 이상과 웨이퍼 불량으로 이어질 수 있습니다.

이러한 문제를 단순 모니터링에 그치지 않고 **현장에서 자동으로 판단·대응하고, 발생 이력을 상위 시스템에서 분석할 수 있는 구조**로 구현하는 것을 목표로 프로젝트를 시작했습니다.

프로젝트 초기 단계에서 제가 먼저 **Q-Time이 임박한 Lot을 자동으로 식별하고 일반 Lot보다 우선 이송·배출하는 자동 라우팅 아이디어**를 제안했습니다.

이를 실제 PLC 제어로 구현하기 위해

`Lot 투입 → 시간 측정 → 위험도 판정 → 공급 제어 → 우선 배출`

순서의 PLC 로직 흐름을 순서도로 설계하고, 어떤 조건과 데이터를 Relay 및 Device에 전달할지 구체화했습니다.

이후 전체 시스템을 다음 네 계층으로 확장했습니다.

1. **Field / Control** — PLC·Servo 기반 실시간 설비 제어
2. **SCADA** — iFIX 기반 상태 및 Q-Time 시각화
3. **Database** — 공정 이력 저장
4. **Analysis** — 공정 병목 및 O-ring 열화 패턴 분석

### 프로젝트 핵심 과제

* Q-Time 임박 Lot을 식별해 **일반 Lot보다 우선 이송하는 자동 Routing Logic 구현**
* 여러 웨이퍼가 동시에 처리되는 상황에서도 **공급·적재·배출 Sequence가 충돌하지 않도록 Interlock 설계**
* Chamber 압력 데이터를 활용한 **O-ring 열화 상태 Monitoring**
* PLC부터 분석 계층까지 이어지는 **4-Layer Smart Factory Pipeline 구성**

</details>

---

<details>
<summary><b>02. 사용 기술 / 툴</b></summary>

<br>

| 구분                  | 기술 / 툴                       |
| ------------------- | ---------------------------- |
| PLC                 | MELSEC PLC                   |
| Servo               | MR-J5 Servo Motor            |
| HMI                 | GOT HMI                      |
| SCADA               | iFIX                         |
| Communication       | OPC-UA                       |
| Database            | SQLite                       |
| Data Analysis       | Python, pandas               |
| Predictive Analysis | LSTM                         |
| Hardware            | MPS 설비, Cylinder, Conveyor   |
| Mechanical          | 3D Printing 부품 설계·제작         |
| Project Management  | 주 단위 팀 공유 문서 및 진행상황 Tracking |

</details>

---

<details>
<summary><b>03. 주요 구현 내용</b></summary>

<br>

### 3-1. Q-Time 위반 예방 자동 Routing Logic 기획

프로젝트 착수 단계에서 **Q-Time이 임박한 Lot을 자동으로 우선 처리하는 Routing 방식**을 제안했습니다.

단순히 Q-Time 초과 시 경보를 발생시키는 것이 아니라, PLC가 각 Lot의 잔여 시간을 비교하고 위험 Lot이 발생하면 새로운 공급을 제한한 뒤 해당 Lot을 우선적으로 배출하는 구조를 목표로 했습니다.

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
```

해당 아이디어를 PLC 로직 순서도로 먼저 구체화한 뒤 팀원들에게 공유했고, 이를 기반으로 Field Control · SCADA · DB · 분석 계층의 작업 범위를 나누었습니다.

---

### 3-2. 팀장 역할 및 프로젝트 일정관리

아이디어 제안 이후 전체 구현에 필요한 세부 작업을 기능 단위로 분해하고 팀원별 역할을 배분했습니다.

주 단위 공유 문서를 활용해

* 지난주 완료 작업
* 현재 진행 상황
* 이번주 목표
* 팀원 간 선행 / 후행 작업 의존성

을 지속적으로 Tracking했습니다.

특히 PLC Logic, MPS Hardware, Servo, SCADA 화면 등이 서로 독립적으로 진행될 수 없는 구조였기 때문에 **한 파트의 변경 사항이 다른 파트에 미치는 영향을 확인하며 일정과 작업 순서를 조율**했습니다.

---

### 3-3. PLC · SCADA · DB · 분석 4-Layer Architecture 설계

전체 시스템을 하나의 장비 제어가 아닌 **4-Layer Pipeline**으로 구성했습니다.

#### Layer 1 — Field / Control

MELSEC PLC에서

* Lot별 Q-Time
* Chamber 압력
* 설비 Sensor
* 위험 등급

등을 처리하고, MR-J5 Servo와 Conveyor를 제어해 위험 Lot을 우선 이송하도록 설계했습니다.

현장 작업자는 GOT HMI를 통해 상태와 Alarm을 확인할 수 있도록 구성했습니다.

#### Layer 2 — SCADA

PLC 데이터를 OPC-UA로 iFIX와 연결해

* Q-Time Heatmap
* O-ring 상태 Gauge
* 설비 상태
* 주요 KPI

등을 상위 Monitoring 화면에서 확인할 수 있도록 구성했습니다.

#### Layer 3 — Database

공정 데이터를 SQLite에 저장해

* Lot별
* 시간대별
* Chamber별

이력을 이후 분석에 사용할 수 있도록 구성했습니다.

#### Layer 4 — Analysis

Python과 pandas를 활용해 공정 병목 구간을 분석하고, Chamber 압력 변화 데이터를 기반으로 LSTM 시계열 분석을 적용해 **O-ring 열화 패턴과 보전 시점을 분석하는 구조**를 설계했습니다.

---

### 3-4. MPS Hardware 재구성

기존 MPS 설비를 프로젝트 목적에 맞게 수정하기 위해 Chamber 설치 공간을 확보하고 Cylinder 및 Sensor 위치를 재조정했습니다.

<p align="center">
  <img src="images/03-hardware.png" alt="MPS 하드웨어 재조립" width="90%">
</p>

주요 작업은 다음과 같습니다.

* Chamber 설치를 위한 분배·공급 Cylinder 위치 변경
* MPS 구조 재조립
* 미사용 Sensor 배선 정리
* Solenoid 상태 진단
* Servo 및 PLC 배선
* Camera-PLC 연동 상태 확인
* Conveyor 구동 회로 구성

기존 설비를 그대로 사용하는 것이 아니라 **제어 Sequence에 맞게 Hardware 구조 자체를 수정한 뒤 PLC Logic과 다시 연동**했습니다.

---

### 3-5. iFIX 설비 전체 개요도 작화

SCADA에서 Q-Time Heatmap과 O-ring 상태 등을 표시하기 위한 기반 화면으로 MPS 전체 설비를 iFIX에서 직접 작화했습니다.

실제 MPS의

* Cylinder
* Servo 결합부
* Conveyor
* Vision Gate
* 좌·우 Station

배치를 반영해 현장 구조와 SCADA 화면이 직관적으로 대응되도록 구성했습니다.

#### 상세 시점

<p align="center">
  <img src="images/03-ifix-detail.png" alt="iFIX 상세 시점" width="90%">
</p>

Cylinder와 Servo가 결합되는 주요 제어부를 확대해 현장 동작 상태를 쉽게 파악할 수 있도록 구성했습니다.

#### 전체 개요 시점

<p align="center">
  <img src="images/03-ifix-overview.png" alt="iFIX 전체 설비 개요" width="90%">
</p>

좌·우 MPS Station과 Conveyor, Gate를 포함한 전체 공정 배치를 한 화면에서 확인할 수 있도록 구성했습니다.

---

### 3-6. 3D Printing 기반 설비 부품 재설계

Hardware 구조 변경으로 기존 부품을 그대로 사용할 수 없는 부분은 3D Printing으로 직접 재설계했습니다.

<p align="center">
  <img src="images/03-3dprint-parts.png" alt="3D 프린팅 부품" width="90%">
</p>

주요 개선 사항은 다음과 같습니다.

* 길이가 맞지 않게 된 분배 막대 재설계
* 한 번에 출력하기 어려운 부품을 **조립식 구조로 분할**
* Chamber-O-ring 접합부 구조 개선
* 출력 후 실제 설비에 장착해 간섭 여부 확인
* 문제 발생 시 치수 변경 후 재출력

설계 → 출력 → 조립 → 동작 확인 과정을 **3회 이상 반복**하며 실제 설비에서 사용할 수 있는 구조로 개선했습니다.

---

### 3-7. PLC · Conveyor 연동

PLC 출력으로 Conveyor를 직접 제어하기 위해 **24V → 12V Converter**를 적용하고 배선 및 전압 Setting 절차를 구성했습니다.

이후 Ladder Logic에서 Conveyor Start / Stop 조건을 구성해 다른 공정 Sequence와 연동했습니다.

```text
PLC Output
    ↓
24V → 12V Converter
    ↓
Conveyor Driver
    ↓
Start / Stop
```

</details>

---

<details>
<summary><b>04. 트러블슈팅 ⭐</b></summary>

<br>

### Trouble 01. 병렬 동작 중 Q-Time 임박 Lot과 일반 Lot의 Sequence 충돌

#### Problem

여러 웨이퍼를 병렬로 처리하는 과정에서 **배출해야 할 위험 웨이퍼가 발생하면 신규 공급 동작을 멈추도록 Interlock**을 구성했습니다.

하지만 기존 로직에서는 Q-Time 잔여 시간이 약 **18초 수준에 도달했을 때 공급을 정지**하도록 설정되어 있어, 이미 다른 웨이퍼의 공급 또는 적재 Sequence가 진행 중인 경우 문제가 발생했습니다.

예를 들어,

```text
Wafer A → 적재 진행 중
Wafer B → Q-Time 임박
Wafer C → 이미 공급 진행 중
```

상태에서 Wafer B의 우선 배출 조건이 발생하면 **기존 공급·적재 Sequence와 배출 Sequence가 동시에 영향을 받으면서 웨이퍼 처리 순서가 꼬이는 현상**이 발생했습니다.

그 결과 위험 Lot을 먼저 배출하기 전에 다른 웨이퍼 동작이 계속 진행되면서 오히려 Q-Time을 초과할 가능성이 생겼습니다.

#### Analysis

초기에는 단순히

`Q-Time 임박 → 공급 Stop`

조건만 적용했지만, 실제 설비에서는 정지 명령이 발생한 순간에도 이미 시작된 Cylinder·Conveyor·Servo Sequence가 즉시 모두 종료되는 것이 아니었습니다.

즉,

```text
Q-Time 위험 판단 시점
        ↓
신규 공급 정지
        ↓
현재 진행 중인 적재 Sequence 완료
        ↓
위험 Lot 배출 Sequence 진입
```

사이에 필요한 시간이 존재했습니다.

따라서 Q-Time 임계값 자체보다 **배출 판단 시점과 이미 실행 중인 Sequence가 종료되는 시점 사이의 시간 여유가 부족한 것**이 핵심 원인이라고 판단했습니다.

#### Solution

현재 PLC Logic을 다시 확인하며 다음 방향으로 Interlock 조건을 조정하고 있습니다.

##### 1. 공급 차단 시점 선행

기존 약 **18초 잔여 시점**에서 공급을 차단하는 방식보다 더 이른 시점에 신규 공급을 제한하도록 Safety Margin을 확보하고 있습니다.

예를 들어 약 **25초 수준 등 더 여유 있는 시점부터 공급을 차단하는 조건을 후보로 두고**, 실제 설비 Cycle Time을 기준으로 적절한 임계값을 검증하고 있습니다.

##### 2. 배출 Sequence와 일반 Sequence의 상호 Interlock

위험 Lot 배출 Flag가 활성화되면

* 신규 Wafer 공급 제한
* 일반 적재 Sequence 진입 제한
* 현재 진행 중인 필수 동작 종료 확인
* 위험 Lot 배출 Sequence 우선 실행

순으로 동작하도록 Sequence 간 Interlock을 보완하고 있습니다.

##### 3. 실제 Cycle Time 기준 임계값 조정

단순히 Q-Time 숫자만 기준으로 설정하지 않고,

`공급 소요 시간 + 적재 완료 시간 + 배출 준비 시간 + Safety Margin`

을 고려해 **위험 Lot 판정 시점을 앞당기는 방향으로 반복 검증**하고 있습니다.

#### Expected / Verification

최종적으로는 Q-Time 임박 Lot이 발생했을 때 새로운 Lot이 공정에 추가로 진입하지 않고, 기존 필수 동작이 정리된 이후 **위험 Lot 배출 Sequence가 다른 Sequence와 충돌 없이 우선 수행되는 구조**를 목표로 PLC 최종 검증을 진행하고 있습니다.

> **Learned**
> 병렬 자동화 설비에서는 개별 Sequence가 각각 정상 동작하는 것만으로 충분하지 않고, **두 Sequence가 동시에 실행될 수 있는 모든 상태를 고려한 상호 Interlock 설계가 필요하다는 점**을 확인했습니다.
> 또한 Q-Time 제어에서는 단순히 제한 시간 직전에 대응하는 것이 아니라 **실제 설비의 물리적 Cycle Time까지 역산해 제어 시점을 선행해야 한다는 점**을 배웠습니다.

---

### Trouble 02. Servo Alarm 및 Limit Sensor 신호 불량

#### Problem

MR-J5 Servo 구동 과정에서 Alarm이 발생하거나 동작 도중 Servo가 정지하는 문제가 발생했습니다.

#### Analysis

PLC Logic 문제와 Hardware 문제를 분리해 확인한 결과,

* Servo 일부 배선 미결선
* Limit Sensor 신호 불량

을 각각 확인했습니다.

특히 Servo 동작 중 특정 위치에서 반복적으로 정지하는 현상을 확인해 Limit Sensor 입력 상태를 추적했고, Sensor 자체의 신호가 정상적으로 전달되지 않는 것을 확인했습니다.

#### Solution

* Servo 배선도를 기준으로 미결선 구간 확인 및 재배선
* Limit Sensor 신호 확인
* 불량 Sensor 교체
* PLC 입력 Device에서 Sensor 상태 재검증
* Servo 반복 구동을 통한 정상 동작 확인

#### Result

배선 및 Sensor 문제를 수정해 Servo가 Sequence에 따라 정상적으로 동작하도록 개선했습니다.

> **Learned**
> PLC에서 Servo Alarm이 발생하더라도 Logic부터 수정하기보다 **전원 → 배선 → Sensor → PLC Input → Logic 순으로 계층을 분리해 확인하는 것이 문제를 빠르게 좁히는 방법**임을 배웠습니다.

---

### Trouble 03. Sensor 간섭으로 인한 PLC 입력 오동작

#### Problem

MPS I/O Sensor Plate에서 실제 동작과 관계없는 입력이 간헐적으로 발생해 PLC Sequence가 예상하지 않은 조건으로 진입하는 문제가 있었습니다.

#### Analysis

Online Monitoring을 통해 Sensor 입력을 하나씩 확인한 결과, 현재 공정에서 사용하지 않는 Sensor가 주변 동작에 반응하면서 **불필요한 PLC Input을 발생시키는 현상**을 확인했습니다.

#### Solution

현재 공정에서 사용하지 않는 Sensor를 식별하고 제거한 뒤, 필요한 Sensor만으로 입력 조건을 다시 구성했습니다.

#### Result

불필요한 입력 간섭이 제거되면서 PLC Sequence가 의도한 Sensor 조건에 따라 동작하도록 안정화했습니다.

---

### Trouble 04. Hardware 구조 변경에 따른 부품 간섭 및 치수 불일치

#### Problem

Chamber 설치를 위해 Cylinder 위치를 변경하면서 기존 구조에서는 없었던 부품 간 간섭이 발생했고, 기존 분배 막대의 길이도 변경된 구조와 맞지 않게 되었습니다.

또한 일부 부품은 크기 때문에 한 번에 3D Printing하기 어려웠습니다.

#### Analysis

단순히 기존 부품의 치수만 늘리는 방식으로는

* 출력 가능한 크기
* 실제 조립성
* Chamber와 O-ring 접합
* Cylinder 동작 범위

를 동시에 만족하기 어렵다고 판단했습니다.

#### Solution

* 간섭이 발생하는 구조 자체를 변경
* 분배 막대 치수 재설계
* 대형 부품을 **조립식 구조로 분할**
* Chamber-O-ring 접합부 형상 개선
* 출력 → 장착 → 간섭 확인 → 치수 수정 과정을 반복

#### Result

3회 이상의 반복 설계·출력을 통해 실제 MPS 설비에 장착 가능한 형태로 구조를 개선했습니다.

> **Learned**
> 자동화 설비 제작에서는 Software Logic뿐 아니라 **기구물의 치수·조립성·Sensor 위치까지 제어 Sequence와 함께 고려해야 한다는 점**을 경험했습니다.

---

### Trouble 05. Camera-PLC 연동 시 Barcode 인식 불량

#### Problem

Vision Camera와 PLC를 연결하는 과정에서 Barcode 인식 결과가 정상적으로 PLC Sequence에 반영되지 않는 문제가 발생했습니다.

#### Analysis

Camera 자체 인식 상태와 PLC 입력 상태를 분리해 확인하며 연동 구간을 점검했습니다.

#### Solution

Camera 설정 및 PLC 연동 조건을 다시 확인해 Barcode 인식 결과가 정상적으로 전달되도록 수정했습니다.

#### Result

Camera 판정 결과를 PLC Sequence에서 정상적으로 사용할 수 있도록 연동 상태를 복구했습니다.

</details>

---

<details>
<summary><b>05. 결과 및 성과</b></summary>

<br>

* **Q-Time 위반 예방 자동 Routing 아이디어 및 PLC Logic 순서도 최초 제안**
* 제안한 아이디어를 기반으로 4인 팀의 세부 구현 항목 및 역할 배분
* **Field Control - SCADA - DB - Analysis 4-Layer Architecture 설계**
* MPS Hardware 구조 변경 및 PLC · Servo 배선 수행
* **iFIX 설비 전체 개요도 및 상세 시점 직접 작화**
* MR-J5 Servo · Sensor · PLC · Camera 연동 관련 Hardware Troubleshooting 수행
* **3D Printing 부품 3회 이상 반복 설계·제작**
* PLC-Conveyor 구동용 24V → 12V 전원 구성 및 제어 연동
* 병렬 공정에서 **Q-Time 위험 Lot과 일반 Lot 간 Sequence 충돌 문제를 발견하고 Interlock 및 선행 제어 시점 개선 중**
* 주 단위 일정관리 및 작업 의존성 조율을 통한 **4인 팀 프로젝트 리딩**
* 현재 PLC 병렬 Sequence 및 최종 동작 검증 단계 진행 중

</details>

---

<details>
<summary><b>06. 회고 / 배운 점</b></summary>

<br>

이번 프로젝트에서는 정해진 기능을 구현하는 역할을 넘어, **문제 정의부터 제어 아이디어를 제안하고 이를 실제 시스템 구조와 팀 단위 작업으로 구체화하는 경험**을 했습니다.

Q-Time 위반 문제를 해결하기 위해 자동 Routing Sequence를 먼저 제안하고 PLC Logic의 흐름을 설계한 뒤, 이를 Field Control · SCADA · DB · Analysis로 확장하면서 하나의 아이디어를 실제 Smart Factory Architecture로 발전시키는 과정을 경험했습니다.

특히 병렬 동작 과정에서 Q-Time 임박 Lot과 일반 Lot의 Sequence가 충돌하는 문제를 확인하면서, 자동화 시스템에서는 각 Logic이 개별적으로 정상 동작하는 것보다 **여러 Sequence가 동시에 동작할 때 발생할 수 있는 상태 조합과 상호 Interlock을 설계하는 것이 중요하다**는 점을 배웠습니다.

또한 Q-Time과 같이 시간 제한이 있는 제어에서는 단순히 기준 시간이 되었을 때 동작시키는 것이 아니라, **설비가 실제로 정지하고 다른 Sequence로 전환되는 데 필요한 물리적 Cycle Time을 역산해 선행 제어해야 한다는 점**도 확인했습니다.

Hardware 측면에서는 Servo 배선, Sensor 간섭, Cylinder 위치 변경, 3D Printing 부품 치수 문제 등 매뉴얼대로 해결되지 않는 문제를 반복적으로 경험했습니다. 이를 전기·제어·기구 문제로 나누어 원인을 좁히고 수정하면서 **Software와 Hardware를 분리해서 보지 않고 하나의 시스템으로 분석하는 문제해결 방식**을 익혔습니다.

팀장으로서는 초기 아이디어와 PLC Sequence를 팀원들에게 설명하고 작업 단위로 분해해 역할을 배분했으며, 주 단위로 작업 진행 상황과 의존성을 관리했습니다. 이를 통해 직접 구현하는 역량뿐 아니라 **전체 프로젝트의 기술적 방향을 설계하고 팀원 간 작업을 연결하는 System Integration 및 Project Leading 경험**을 쌓았습니다.

</details>



