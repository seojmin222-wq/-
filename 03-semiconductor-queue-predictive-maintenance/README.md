# Q-Time 위반 예방 자동 라우팅 기반 스마트팩토리 시스템

### PLC · SCADA · DB · 데이터분석 4-Layer 통합 자동화 시스템

> **반도체 공정의 Q-Time 초과 문제를 대상으로  
> 현장제어 → SCADA → DB → 데이터분석까지 연결한 4-Layer 통합 시스템**

> **진행 상태**: 마무리 단계 — PLC 최종 동작 및 병렬 시퀀스 검증 진행 중

* **기간**: 2026.08 ~ 진행 중
* **참여 형태**: 팀 프로젝트 (4인)
* **역할**: 팀장
* **주요 기여**: Q-Time 위반 예방 자동 라우팅 아이디어 및 PLC 시퀀스 최초 제안 · 전체 아키텍처 구체화 · 역할 배분 및 일정관리 · MPS 하드웨어 재구성 · PLC/Servo 배선 · iFIX 설비 개요도 작화 · 3D Printing 부품 설계 및 Wafer 이송 구조 개선
* **주요 기술**: MELSEC PLC, MR-J5 Servo, GOT HMI, iFIX, OPC-UA, SQLite, Python, pandas, 3D Printing

---

## 시스템 구성

<p align="center">
  <img src="images/03-architecture.png" alt="4-Layer 시스템 아키텍처" width="90%">
</p>

| Layer | 구성 요소 | 역할 |
| --- | --- | --- |
| **1. Field / Control** | MELSEC PLC, MR-J5 Servo, GOT HMI | Lot·Q-Time 데이터 처리, 위험도 판정, 우선 이송 및 현장 제어 |
| **2. SCADA** | iFIX, OPC-UA | Q-Time Heatmap, KPI 및 설비 상태 시각화 |
| **3. Database** | SQLite | Lot·시간대·공정별 이력 저장 |
| **4. Analysis** | Python, pandas | Q-Time 이력 및 공정 Cycle Time 분석, 병목 구간 확인 |

### Q-Time 제어 흐름

```text
Lot 투입
    ↓
Q-Time Monitoring
    ↓
위험도 판정
    ↓
신규 공급 Interlock
    ↓
위험 Lot 우선 이송 / 배출
    ↓
정상 Sequence 복귀
```

---

<details>
<summary><b>01. 프로젝트 개요 및 문제 정의</b></summary>

<br>

반도체 공정에서는 특정 공정 사이의 **Q-Time을 초과할 경우 Lot 폐기 또는 품질 저하가 발생할 수 있습니다.**

이러한 문제를 단순히 Q-Time 초과 이후 Alarm을 발생시키는 방식으로 처리하는 것이 아니라, **Q-Time이 임박한 Lot을 설비가 사전에 판단하고 일반 Lot보다 먼저 처리하는 자동화 구조**로 구현하는 것을 목표로 프로젝트를 시작했습니다.

프로젝트 초기 단계에서 제가 먼저 **Q-Time이 임박한 Lot을 자동으로 식별하고 일반 Lot보다 우선 이송·배출하는 자동 Routing 아이디어**를 제안했습니다.

이를 실제 PLC 제어로 구현하기 위해

```text
Lot 투입
    ↓
시간 측정
    ↓
위험도 판정
    ↓
신규 공급 제어
    ↓
위험 Lot 우선 배출
```

순서로 PLC Logic 흐름을 구체화하고, 각 단계에서 필요한 Relay 및 Device 조건을 정리했습니다.

이후 하나의 PLC Sequence에 그치지 않고 시스템을 다음 네 계층으로 확장했습니다.

1. **Field / Control** — PLC·Servo 기반 실시간 설비 제어
2. **SCADA** — iFIX 기반 설비 및 Q-Time 상태 시각화
3. **Database** — Lot 및 공정 이력 저장
4. **Analysis** — 공정 이력 및 Cycle Time 기반 병목 분석

### 프로젝트 핵심 과제

* Q-Time 임박 Lot을 식별해 **일반 Lot보다 우선 이송하는 자동 Routing Logic 구현**
* 여러 Wafer가 동시에 처리되는 상황에서도 **공급·적재·배출 Sequence가 충돌하지 않도록 Interlock 설계**
* MPS 구조 변경 이후에도 Wafer가 목표 위치에 안정적으로 도달할 수 있도록 **기구 구조 및 Cylinder 동작 조건 개선**
* PLC부터 SCADA·DB·분석까지 이어지는 **4-Layer Smart Factory Pipeline 구성**

</details>

---

<details>
<summary><b>02. 사용 기술 / 툴</b></summary>

<br>

| 구분 | 기술 / 툴 |
| --- | --- |
| PLC | MELSEC PLC |
| Servo | MR-J5 Servo Motor |
| HMI | GOT HMI |
| SCADA | iFIX |
| Communication | OPC-UA |
| Database | SQLite |
| Data Analysis | Python, pandas |
| Hardware | MPS 설비, Cylinder, Conveyor |
| Mechanical | 3D Printing 부품 설계·제작 |
| Project Management | 주 단위 팀 공유 문서 및 진행상황 Tracking |

</details>

---

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
```

해당 아이디어를 PLC Logic 순서도로 먼저 구체화한 뒤 팀원들에게 공유했고, 이를 기반으로 Field Control · SCADA · DB · 분석 계층의 작업 범위를 나누었습니다.

</details>

<br>

<details>
<summary><b>3-2. 팀장 역할 및 프로젝트 일정관리</b></summary>

<br>

아이디어 제안 이후 전체 구현에 필요한 세부 작업을 기능 단위로 분해하고 팀원별 역할을 배분했습니다.

주 단위 공유 문서를 활용해

* 지난주 완료 작업
* 현재 진행 상황
* 이번주 목표
* 발생한 문제 및 해결 방법
* 팀원 간 선행 / 후행 작업 의존성

을 지속적으로 Tracking했습니다.

특히 PLC Logic, MPS Hardware, Servo, SCADA 화면 등이 서로 독립적으로 진행될 수 없는 구조였기 때문에 **한 파트의 변경 사항이 다른 파트에 미치는 영향을 확인하며 일정과 작업 순서를 조율**했습니다.

이전에 발생했던 문제와 해결 방법 중 다시 발생할 가능성이 높은 내용은 팀원들이 함께 확인할 수 있도록 공유 문서에 정리해 동일한 문제에 반복적으로 시간을 소모하지 않도록 관리했습니다.

</details>

<br>

<details>
<summary><b>3-3. PLC · SCADA · DB · 분석 4-Layer Architecture 설계</b></summary>

<br>

전체 시스템을 하나의 장비 제어가 아닌 **4-Layer Pipeline**으로 구성했습니다.

#### Layer 1 — Field / Control

MELSEC PLC에서

* Lot별 Q-Time
* 설비 Sensor 상태
* 위험 등급
* Sequence 진행 상태

등을 처리하고, MR-J5 Servo와 Cylinder·Conveyor를 제어해 위험 Lot을 우선 이송하도록 설계했습니다.

현장 작업자는 GOT HMI를 통해 설비 상태와 Alarm을 확인할 수 있도록 구성했습니다.

#### Layer 2 — SCADA

PLC 데이터를 OPC-UA로 iFIX와 연결해

* Q-Time Heatmap
* 설비 운전 상태
* Alarm
* 주요 KPI

등을 상위 Monitoring 화면에서 확인할 수 있도록 구성했습니다.

#### Layer 3 — Database

공정 데이터를 SQLite에 저장해

* Lot별
* 시간대별
* 공정별

이력을 이후 분석에 활용할 수 있도록 구성했습니다.

#### Layer 4 — Analysis

Python과 pandas를 활용해 저장된 Lot 및 Q-Time 이력을 가공하고, 공정별 Cycle Time과 대기시간을 비교해 **Q-Time 지연이 반복적으로 발생하는 구간과 병목 구간을 확인할 수 있는 구조**로 설계했습니다.

</details>

<br>

<details>
<summary><b>3-4. MPS Hardware 재구성</b></summary>

<br>

프로젝트 목적에 맞는 공정 구조를 만들기 위해 기존 MPS 설비의 Cylinder, Stopper, Sensor 및 Wafer 이동 구간을 재구성했습니다.

초기에는 Chamber에 O-ring을 적용한 밀폐 구조까지 구현하기 위해 **가공·분배 Stopper의 높이와 위치를 변경**했습니다.

하지만 Stopper 위치가 변경되면서 기존 Wafer 공급·분배 경로와 Cylinder 위치도 함께 달라졌기 때문에 주변 Hardware 구조를 연쇄적으로 수정해야 했습니다.

주요 작업은 다음과 같습니다.

* 가공·분배 Stopper 높이 및 위치 조정
* 변경된 구조에 맞춘 분배부 재설계
* MPS 구조 재조립
* Cylinder 및 Sensor 위치 조정
* 미사용 Sensor 배선 정리
* Solenoid 상태 진단
* Servo 및 PLC 배선
* Camera-PLC 연동 상태 확인
* Conveyor 구동 회로 구성

기존 설비를 그대로 사용하는 것이 아니라 **제어 Sequence와 실제 Wafer 이동 경로에 맞춰 Hardware 구조 자체를 수정한 뒤 PLC Logic과 다시 연동**했습니다.

</details>

<br>

<details>
<summary><b>3-5. iFIX 설비 전체 개요도 작화</b></summary>

<br>

SCADA에서 Q-Time과 설비 상태를 표시하기 위한 기반 화면으로 MPS 전체 설비를 iFIX에서 직접 작화했습니다.

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

좌·우 MPS Station과 Conveyor, Gate를 포함한 전체 공정 구조를 서로 다른 시점에서 확인할 수 있도록 구성했습니다.

<table align="center">
  <tr>
    <td align="center" width="50%">
      <img src="images/full1.png" alt="전체 개요 시점 1" width="95%">
    </td>
    <td align="center" width="50%">
      <img src="images/full2.png" alt="전체 개요 시점 2" width="95%">
    </td>
  </tr>
  <tr>
    <td align="center"><b>전체 개요 시점 1</b></td>
    <td align="center"><b>전체 개요 시점 2</b></td>
  </tr>
</table>

</details>

<br>

<details>
<summary><b>3-6. 3D Printing 기반 설비 구조 재설계</b></summary>

<br>

Hardware 구조 변경으로 기존 부품을 그대로 사용할 수 없는 부분은 3D Printing을 활용해 직접 재설계했습니다.

<p align="center">
  <img src="images/03-3dprint-parts.png" alt="3D 프린팅 부품" width="90%">
</p>

초기에는 Chamber에 O-ring을 적용한 구조를 구현하기 위해 가공·분배 Stopper의 높이와 위치를 변경했습니다.

하지만 구조 변경 이후 기존 분배 부품의 높이와 길이가 맞지 않게 되어 **변경된 설비 치수에 맞춰 분배 Cylinder 측 부품을 새로 설계·출력**했습니다.

또한 부품 크기로 인해 한 번에 출력하기 어려운 구조는 조립식으로 분할했습니다.

주요 작업은 다음과 같습니다.

* 변경된 Stopper 위치에 맞춘 분배부 치수 재설계
* 기존 길이가 맞지 않게 된 분배 막대 재설계
* 한 번에 출력하기 어려운 부품을 **조립식 구조로 분할**
* Chamber 및 Cylinder 결합부 형상 수정
* 출력 후 실제 설비에 장착해 간섭 여부 확인
* 문제 발생 시 치수 변경 후 재출력

설계 → 출력 → 조립 → 동작 확인 과정을 **3회 이상 반복**하며 실제 MPS 설비에 적용할 수 있는 구조로 개선했습니다.

</details>

<br>

<details>
<summary><b>3-7. Wafer 이송 안정화를 위한 받침대 및 Guide 구조 추가</b></summary>

<br>

분배 Cylinder가 Wafer를 밀어 Conveyor로 전달하는 과정에서 높이 차이로 인해 Wafer가 직접 떨어지면 낙하 위치가 일정하지 않고 이동이 불안정했습니다.

이를 개선하기 위해 분배부와 Conveyor 사이에 **Wafer를 받아주는 받침대를 추가**해 Wafer가 직접 낙하하지 않고 자연스럽게 Conveyor Belt로 이동하도록 경로를 구성했습니다.

하지만 받침대만 설치했을 때는 Wafer가 Conveyor 중앙에 항상 정확하게 도착하지 않고 좌·우로 치우치는 현상이 발생했습니다.

이에 받침대 측면에 **Guide 구조를 추가**하여 Wafer의 좌·우 이동 범위를 제한하고 Conveyor 중앙 방향으로 유도하도록 개선했습니다.

```text
분배 Cylinder
      ↓
Wafer 밀어내기
      ↓
받침대를 통한 이동
      ↓
Guide를 통한 방향 보정
      ↓
Conveyor 중앙 진입
```

단순히 Wafer가 Conveyor에 도착하는 것에서 끝내지 않고, **후속 공정에서 반복적으로 동일한 위치를 사용할 수 있도록 이동 경로의 재현성을 높이는 방향으로 구조를 수정**했습니다.

</details>

<br>

<details>
<summary><b>3-8. Cylinder 전·후진 속도 차등 설정</b></summary>

<br>

Wafer가 Chamber에 공급되거나 분배되는 과정에서 Cylinder의 전진 속도가 빠르면 Wafer에 가속이 붙어 최종 정지 위치가 일정하지 않는 현상이 발생했습니다.

Cylinder Stroke가 동일하더라도 Wafer가 빠른 속도로 밀려나면 관성에 의해 계속 움직이기 때문에 실제 도달 위치에 편차가 생겼습니다.

이를 개선하기 위해 Wafer를 직접 밀어내는 **공급 Cylinder와 분배 Cylinder의 전진 속도를 낮춰 천천히 동작하도록 조정**했습니다.

#### 전진 속도 — 저속

* Wafer에 발생하는 관성 최소화
* Chamber 공급 위치 안정화
* 분배 위치 편차 감소
* Guide 및 Stopper 충돌 가능성 감소

반면 Cylinder의 복귀 동작까지 동일하게 느리게 설정하면 전체 Cycle Time이 길어지고 다음 Cylinder의 Sequence와 동작이 겹칠 가능성이 있었습니다.

따라서 Wafer 이동이 완료된 이후의 **후진 동작은 빠르게 설정**했습니다.

#### 후진 속도 — 고속

* 불필요한 Cycle Time 증가 방지
* 다음 Sequence 시작 전 빠른 원위치 복귀
* 후속 Cylinder 동작과의 간섭 방지

```text
Wafer 이송
   ↓
Cylinder 저속 전진
   ↓
Wafer 목표 위치 도달
   ↓
Cylinder 고속 후진
   ↓
다음 Sequence 진행
```

이를 통해 단순히 Cylinder의 동작 여부만 확인하는 것이 아니라 **Wafer의 위치 재현성과 전체 Sequence의 Cycle Time을 함께 고려해 Actuator 속도를 조정**했습니다.

</details>

<br>

<details>
<summary><b>3-9. PLC · Conveyor 연동</b></summary>

<br>

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

<br>

<details>
<summary><b>3-10. Q-Time 기반 전체 자동 운전 Sequence 검증</b></summary>

<br>

PLC Logic과 MPS Hardware, iFIX Monitoring 화면을 연동한 뒤 실제 Wafer를 투입하여 전체 자동 운전 Sequence를 검증했습니다.

전체 동작을 다음 3단계로 나누어 확인했으며, 각 단계별로 **실제 MPS Hardware 동작과 iFIX Monitoring 화면**을 함께 확인했습니다.

```text
[STEP 1] 병렬 Wafer 적재 및 Q-Time Monitoring
                    ↓
[STEP 2] Q-Time 잔여시간 기준 우선 배출
                    ↓
[STEP 3] 배출 후 빈 위치 순차 재적재
                    ↓
             Sequence 반복
```

#### STEP 1. 병렬 Wafer 적재 및 Q-Time Monitoring

여러 Wafer를 순차적으로 공급하면서 Servo와 Cylinder를 이용해 각 위치에 적재하고, **적재가 완료된 Wafer별로 Q-Time 측정을 시작**하도록 구성했습니다.

한 Wafer의 전체 공정이 끝날 때까지 기다린 뒤 다음 Wafer를 투입하는 방식이 아니라, 각 공정의 동작 가능 상태를 확인하면서 **여러 Wafer가 동시에 공정 내에서 처리되는 병렬 Sequence**로 구성했습니다.

iFIX에서는 실제 설비의 Wafer 적재 상태와 함께 각 Wafer별 Q-Time이 정상적으로 증가하는지 확인했습니다.

##### 실제 Hardware 동작

> Wafer가 연속적으로 공급되어 각 위치에 적재되고, 여러 Wafer가 병렬로 처리되는 실제 MPS 동작입니다.

https://github.com/user-attachments/assets/8edfe899-859c-44aa-bd8a-7cd0d4e1a9ec

##### iFIX Monitoring

> 실제 Hardware의 Wafer 적재 상태와 연동하여 각 Wafer의 Q-Time이 측정되는 iFIX Monitoring 화면입니다.

https://github.com/user-attachments/assets/dbd3c299-5736-459a-8db8-66f02bdc4d8c

---

#### STEP 2. Q-Time 잔여시간 기준 Wafer 우선 배출

Wafer가 적재된 이후에는 각 Wafer의 Q-Time을 지속적으로 Monitoring하고, **Q-Time 잔여시간이 적은 Wafer부터 우선적으로 배출**하도록 Sequence를 구성했습니다.

단순히 먼저 적재된 순서로 배출하는 것이 아니라 PLC에서 각 Wafer의 Q-Time 상태를 확인한 뒤, **공정 제한시간 초과 위험이 높은 Wafer를 먼저 선택하여 배출하는 Routing Logic**을 적용했습니다.

위험 Wafer의 배출이 시작되면 신규 공급 및 일반 Sequence와 충돌하지 않도록 Interlock 조건을 적용했습니다.

##### 실제 Hardware 동작

> 적재된 Wafer 중 Q-Time 잔여시간이 적은 Wafer를 우선 선택하여 배출하는 실제 MPS 동작입니다.

https://github.com/user-attachments/assets/256d5afd-f6e5-4ea2-8553-24b0a2ee4325

##### iFIX Monitoring

> Wafer별 Q-Time 상태와 우선 배출 대상의 변화를 확인할 수 있는 iFIX Monitoring 화면입니다.

https://github.com/user-attachments/assets/b050b6d4-b794-49c0-8d60-977e825c9181

---

#### STEP 3. 배출 후 빈 위치 순차 재적재

Q-Time을 기준으로 Wafer가 배출되면 기존 적재 위치에 빈 공간이 발생합니다.

이때 새로운 Wafer를 단순히 마지막 위치에 추가하는 것이 아니라, **현재 비어 있는 적재 위치를 확인한 뒤 1층부터 순차적으로 다시 채우도록** 공급 Sequence를 구성했습니다.

이를 통해 Wafer가 배출된 이후에도 설비가 정지하지 않고,

```text
Wafer 배출
    ↓
빈 위치 확인
    ↓
낮은 층부터 적재 위치 결정
    ↓
신규 Wafer 공급
    ↓
해당 위치 적재
    ↓
Q-Time Monitoring 재시작
```

순서로 자동 운전을 지속하도록 구성했습니다.

##### 실제 Hardware 동작

> Wafer 배출 후 비어 있는 위치를 확인하고, 1층부터 다시 Wafer를 적재하는 실제 MPS 동작입니다.

https://github.com/user-attachments/assets/382351eb-6bda-4968-b9fa-1ba32ec8b9fd

##### iFIX Monitoring

> Wafer 배출로 발생한 빈 위치와 신규 Wafer가 낮은 층부터 다시 적재되는 상태를 확인할 수 있는 iFIX Monitoring 화면입니다.

https://github.com/user-attachments/assets/2c279415-942a-4b5f-a6ff-cd18e8c6c593

---

#### 전체 Sequence

```text
Wafer 공급
    ↓
병렬 적재
    ↓
Wafer별 Q-Time Monitoring
    ↓
Q-Time 잔여시간 비교
    ↓
위험 Wafer 우선 배출
    ↓
빈 적재 위치 확인
    ↓
1층부터 신규 Wafer 재적재
    ↓
Q-Time Monitoring 지속
```

실제 MPS Hardware 동작과 iFIX 화면을 함께 확인하며 **PLC의 내부 Sequence와 실제 설비 동작, 상위 Monitoring 화면의 상태가 동일하게 연동되는지 검증**했습니다.

</details>

<br>

</details>

---

<details>
<summary><b>04. 트러블슈팅</b></summary>

<br>

### Trouble 01. 병렬 동작 중 Q-Time 임박 Lot과 일반 Lot의 Sequence 충돌

#### Problem

여러 Wafer를 병렬로 처리하는 과정에서 **배출해야 할 위험 Wafer가 발생하면 신규 공급 동작을 멈추도록 Interlock**을 구성했습니다.

하지만 기존 Logic에서는 Q-Time 잔여 시간이 약 **18초 수준에 도달했을 때 공급을 정지**하도록 설정되어 있어, 이미 다른 Wafer의 공급 또는 적재 Sequence가 진행 중인 경우 문제가 발생했습니다.

예를 들어,

```text
Wafer A → 적재 진행 중
Wafer B → Q-Time 임박
Wafer C → 이미 공급 진행 중
```

상태에서 Wafer B의 우선 배출 조건이 발생하면 **기존 공급·적재 Sequence와 배출 Sequence가 동시에 영향을 받으면서 Wafer 처리 순서가 꼬이는 현상**이 발생했습니다.

그 결과 위험 Lot을 먼저 배출하기 전에 다른 Wafer 동작이 계속 진행되면서 오히려 Q-Time을 초과할 가능성이 생겼습니다.

#### Analysis

초기에는 단순히

```text
Q-Time 임박 → 공급 Stop
```

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

##### 1. 공급 차단 시점 선행

기존 약 **18초 잔여 시점**에서 공급을 차단하는 방식보다 더 이른 시점에 신규 공급을 제한하도록 Safety Margin을 확보했습니다.

약 **25초 수준 등 더 여유 있는 시점부터 공급을 차단하는 조건을 후보로 두고**, 실제 설비 Cycle Time을 기준으로 적절한 임계값을 검증했습니다.

##### 2. 배출 Sequence와 일반 Sequence의 상호 Interlock

위험 Lot 배출 Flag가 활성화되면

* 신규 Wafer 공급 제한
* 일반 적재 Sequence 진입 제한
* 현재 진행 중인 필수 동작 종료 확인
* 위험 Lot 배출 Sequence 우선 실행

순으로 동작하도록 Sequence 간 Interlock을 보완했습니다.

##### 3. 실제 Cycle Time 기준 임계값 조정

단순히 Q-Time 숫자만 기준으로 설정하지 않고,

```text
공급 소요 시간
+ 적재 완료 시간
+ 배출 준비 시간
+ Safety Margin
```

을 고려해 **위험 Lot 판정 시점을 앞당기는 방향으로 검증**했습니다.

#### Result

Q-Time 임박 Lot이 발생했을 때 신규 Lot 공급을 제한하고, 기존 필수 동작이 완료된 이후 **위험 Lot 배출 Sequence가 다른 Sequence와 충돌하지 않고 우선 수행되도록 Logic을 개선**했습니다.

> **Learned**  
> 병렬 자동화 설비에서는 개별 Sequence가 정상 동작하는 것만으로 충분하지 않고, **여러 Sequence가 동시에 실행될 수 있는 상태를 고려한 상호 Interlock 설계가 필요하다는 점**을 확인했습니다.  
> 또한 Q-Time 제어에서는 제한 시간 직전에 대응하는 것이 아니라 **실제 설비의 물리적 Cycle Time까지 역산해 제어 시점을 선행해야 한다는 점**을 배웠습니다.

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

현재 공정에서 사용하지 않는 Sensor를 식별하고 제거한 뒤 필요한 Sensor만으로 입력 조건을 다시 구성했습니다.

#### Result

불필요한 입력 간섭이 제거되면서 PLC Sequence가 의도한 Sensor 조건에 따라 동작하도록 안정화했습니다.

> **Learned**  
> 자동화 설비에서는 사용하지 않는 Sensor라도 PLC Input에 연결되어 있다면 예상하지 못한 Sequence 조건을 만들 수 있기 때문에, **실제 사용하는 I/O 기준으로 Hardware와 Logic을 함께 정리해야 한다는 점**을 확인했습니다.

---

### Trouble 04. 구조 변경 후 Wafer 이송 위치 불안정

#### Problem

Chamber 구조를 적용하기 위해 가공·분배 Stopper의 높이와 위치를 변경하면서 기존 Wafer 이송 경로가 달라졌습니다.

변경된 구조에 맞춰 분배부를 다시 설계했지만 실제 Wafer를 공급해보니 다음과 같은 문제가 발생했습니다.

* 분배된 Wafer가 Conveyor로 떨어지는 과정에서 이동이 불안정함
* Conveyor 중앙이 아닌 좌·우로 치우쳐 진입함
* 공급·분배 Cylinder가 빠르게 전진할 경우 Wafer의 최종 정지 위치가 일정하지 않음

#### Analysis

Wafer의 이동 과정을 반복해서 확인한 결과 문제를 하나의 원인이 아닌 **기구 구조와 Cylinder 동작 조건이 함께 영향을 주는 문제**로 판단했습니다.

먼저 분배부와 Conveyor 사이에 높이 차이가 있어 Wafer가 직접 떨어질 경우 낙하 위치가 일정하지 않았습니다.

이를 해결하기 위해 받침대를 설치했지만 Wafer의 좌·우 이동을 제한하는 구조가 없어 Conveyor 중앙에서 벗어나는 현상이 남아 있었습니다.

또한 공급·분배 Cylinder의 전진 속도가 빠르면 Wafer가 Cylinder에서 힘을 받은 이후에도 관성에 의해 계속 움직이면서 같은 Stroke에서도 최종 정지 위치에 편차가 발생하는 것을 확인했습니다.

#### Solution

##### 1. Wafer 이동용 받침대 설치

분배부와 Conveyor 사이에 받침대를 추가해 Wafer가 직접 낙하하지 않고 자연스럽게 Conveyor로 이동하도록 경로를 수정했습니다.

##### 2. Guide 구조 추가

받침대만으로는 Wafer가 Conveyor 중앙에 정확하게 진입하지 않아 좌·우 이동을 제한하는 Guide를 추가했습니다.

이를 통해 Wafer가 정해진 경로를 따라 Conveyor 중앙으로 이동하도록 구조를 개선했습니다.

##### 3. 공급·분배 Cylinder 전진 속도 저감

Wafer에 불필요한 가속이 발생하지 않도록 공급 Cylinder와 분배 Cylinder의 **전진 속도를 낮춰 Wafer를 천천히 이송**하도록 조정했습니다.

이를 통해 Chamber 공급 및 분배 과정에서 Wafer가 지나치게 밀려나는 현상을 줄였습니다.

##### 4. Cylinder 후진 속도 고속 설정

전진과 동일하게 후진 속도까지 낮추면 Cycle Time이 증가하고 다음 Cylinder 동작과 겹칠 가능성이 있었습니다.

따라서 Wafer를 직접 움직이는 **전진 동작만 저속으로 설정하고, Wafer 이송이 끝난 후의 후진 동작은 빠르게 설정**해 다음 Sequence가 시작되기 전에 원위치로 복귀하도록 조정했습니다.

#### Result

받침대와 Guide를 통해 Wafer 이동 경로를 제한하고 Cylinder의 전진·후진 속도를 동작 목적에 맞게 다르게 설정함으로써 **Wafer가 Chamber 및 Conveyor의 목표 위치에 보다 안정적으로 공급·분배될 수 있도록 개선**했습니다.

> **Learned**  
> 자동화 설비에서는 Cylinder의 Stroke와 Sensor 조건만 맞는다고 해서 실제 물체가 항상 동일한 위치에 도달하는 것은 아니며, **이송 대상의 관성·낙하 경로·Guide 구조·Cylinder 속도까지 함께 고려해야 위치 재현성을 확보할 수 있다는 점**을 경험했습니다.  
> 또한 Cycle Time을 무조건 줄이는 것보다 **정확도가 필요한 동작은 느리게, 단순 복귀 동작은 빠르게 설정하는 방식으로 각 동작의 목적에 맞게 속도를 조정하는 것이 중요하다는 점**을 배웠습니다.

---

### Trouble 05. 3D Printing 기반 O-ring 밀폐 구조의 한계

#### Problem

Chamber에 O-ring을 적용해 밀폐 구조를 구현하기 위해 O-ring 결합부를 직접 설계하고 3D Printing으로 제작했습니다.

O-ring 압착을 고려해 여러 차례 치수와 결합 구조를 수정했지만 실제 조립 후 테스트에서는 원하는 수준의 기밀을 확보하지 못했습니다.

#### Analysis

초기에는 O-ring 결합부의 치수와 압착량 문제로 판단해 구조를 반복 수정했습니다.

하지만 여러 차례 출력·조립한 결과 단순 치수 문제뿐 아니라 **3D Printing 특유의 적층면과 표면 상태, 조립부의 미세한 공차**로 인해 밀폐 구조에 필요한 균일한 접촉면을 확보하기 어렵다는 점을 확인했습니다.

따라서 동일한 방식으로 치수만 계속 수정하는 것은 실질적인 해결 방법이 아니라고 판단했습니다.

#### Decision

O-ring 적용을 위해 설비 구조와 부품을 여러 차례 수정했지만 현재 제작 방식으로는 안정적인 기밀을 확보하기 어렵다고 판단해 **O-ring 밀폐 기능은 최종 시스템 구현 범위에서 제외**했습니다.

다만 이 과정에서 변경된 설비를 기준으로

* 가공·분배 Stopper 높이 및 위치 조정
* 분배부 재설계
* Wafer 이동 받침대 설치
* Guide 구조 추가
* 공급·분배 Cylinder 속도 최적화

등의 개선 작업을 수행했고 해당 구조는 Wafer 이송 안정화에 활용했습니다.

향후 동일한 밀폐 구조를 다시 구현한다면 Chamber 자체를 3D Printing으로 제작하기보다 **치수 정밀도와 기밀 확보가 가능한 규격 아크릴 Chamber를 사용하고 O-ring 결합부를 별도로 설계하는 방식**으로 개선할 계획입니다.

> **Learned**  
> 3D Modeling에서 원하는 형상을 구현할 수 있는 것과 실제 부품이 요구 성능을 만족하는 것은 별개의 문제라는 점을 경험했습니다.  
> 특히 밀폐 구조에서는 형상뿐 아니라 **제작 방식의 표면 상태·공차·재료 특성까지 고려해 제조 방법을 선정해야 한다는 점**을 배웠습니다.

---

### Trouble 06. Camera-PLC 연동 시 Barcode 인식 불량

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
* 구조 변경에 맞춰 **3D Printing 부품을 3회 이상 반복 설계·제작**
* 분배부와 Conveyor 사이의 **Wafer 이동 받침대 및 Guide 구조 추가**
* 공급·분배 Cylinder의 **전진 저속 / 후진 고속 설정을 통해 위치 재현성과 Cycle Time을 함께 고려**
* PLC-Conveyor 구동용 **24V → 12V 전원 구성 및 제어 연동**
* O-ring 밀폐 구조 구현 과정에서 **3D Printing 방식의 기밀 한계를 확인하고 최종 구현 범위에서 제외**
* 병렬 공정에서 **Q-Time 위험 Lot과 일반 Lot 간 Sequence 충돌 문제를 발견하고 Interlock 및 선행 제어 시점 개선**
* 실제 MPS Hardware와 iFIX 화면을 통해 **병렬 적재 → Q-Time 기반 우선 배출 → 빈 위치 재적재 전체 Sequence 검증**
* 주 단위 일정관리 및 작업 의존성 조율을 통한 **4인 팀 프로젝트 리딩**

</details>

---

<details>
<summary><b>06. 회고 / 배운 점</b></summary>

<br>

이번 프로젝트에서는 정해진 기능을 구현하는 역할을 넘어, **문제 정의부터 제어 아이디어를 제안하고 이를 실제 시스템 구조와 팀 단위 작업으로 구체화하는 경험**을 했습니다.

Q-Time 위반 문제를 해결하기 위해 자동 Routing Sequence를 먼저 제안하고 PLC Logic의 흐름을 설계한 뒤, 이를 Field Control · SCADA · DB · Analysis로 확장하면서 하나의 아이디어를 실제 Smart Factory Architecture로 발전시키는 과정을 경험했습니다.

특히 병렬 동작 과정에서 Q-Time 임박 Lot과 일반 Lot의 Sequence가 충돌하는 문제를 확인하면서 자동화 시스템에서는 각 Logic이 개별적으로 정상 동작하는 것보다 **여러 Sequence가 동시에 실행될 때 발생할 수 있는 상태 조합과 상호 Interlock을 설계하는 것이 중요하다**는 점을 배웠습니다.

또한 Q-Time과 같이 시간 제한이 있는 제어에서는 단순히 기준 시간이 되었을 때 동작시키는 것이 아니라, **설비가 실제로 정지하고 다른 Sequence로 전환되는 데 필요한 물리적 Cycle Time을 역산해 선행 제어해야 한다는 점**도 확인했습니다.

Hardware 측면에서는 Stopper 위치 변경, Cylinder 및 Servo 배선, Sensor 간섭, Wafer 이송 위치 편차, 3D Printing 부품 치수 문제 등 매뉴얼만으로 해결하기 어려운 문제를 반복적으로 경험했습니다.

특히 Wafer 이송 과정에서는 Cylinder의 Stroke만 맞추는 것으로 충분하지 않았습니다. Wafer가 Conveyor로 낙하하는 경로를 확인해 받침대를 추가하고, 중앙에서 벗어나는 문제를 해결하기 위해 Guide 구조를 설계했습니다.

또한 Cylinder의 전진 속도가 빠르면 Wafer에 관성이 발생해 목표 위치가 일정하지 않다는 점을 확인해 **Wafer를 움직이는 전진 동작은 느리게, 다음 Sequence 준비를 위한 후진 동작은 빠르게 설정**했습니다.

이를 통해 자동화 설비에서는 Logic뿐만 아니라 **이송 대상의 물리적 움직임, 기구 구조, Actuator 속도와 전체 Cycle Time을 함께 고려해야 안정적인 Sequence를 만들 수 있다는 점**을 배웠습니다.

O-ring 밀폐 구조의 경우 구현을 위해 부품을 여러 차례 설계하고 실제 설비 구조까지 변경했지만, 3D Printing 방식으로는 요구한 수준의 기밀을 확보하기 어렵다는 것을 확인했습니다.

기능을 완성하는 것에만 집중하기보다 반복 테스트 결과를 기준으로 **현재 제작 방식의 한계를 판단하고 최종 구현 범위에서 제외하는 결정**을 내렸습니다.

이를 통해 설계 형상의 완성도뿐 아니라 실제 요구 성능과 제조 방식의 적합성을 함께 판단해야 한다는 점을 경험했습니다.

팀장으로서는 초기 아이디어와 PLC Sequence를 팀원들에게 설명하고 작업 단위로 분해해 역할을 배분했으며, 주 단위로 작업 진행 상황과 의존성을 관리했습니다.

또한 진행 과정에서 발생한 문제와 해결 방법을 공유 문서에 지속적으로 기록해 팀원들이 동일한 문제를 반복해서 해결하지 않도록 관리했습니다.

이를 통해 직접 구현하는 역량뿐 아니라 **전체 프로젝트의 기술적 방향을 설계하고 Software · Hardware · 팀원 간 작업을 연결하는 System Integration 및 Project Leading 경험**을 쌓았습니다.

</details>
