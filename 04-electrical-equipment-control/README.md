# 반도체장비 제어 전문가 과정 – 시퀀스 제어회로 4종 설계·배선

### 급수 · 컨베이어 · 온도제어 · 이중 과부하 보호 복합 시퀀스 구현

> **Relay · Timer · Counter · Sensor · EOCR 등 산업용 제어기기를 조합해
> 자동/수동 시퀀스 회로를 직접 설계·배선하고 최종 복합 시퀀스 평가를 통과한 제어회로 프로젝트**

* **기간**: 2026.04 ~ 2026.05
* **참여 형태**: 개인 실습 및 최종평가
* **담당 역할**: 회로 도면 설계 · 제어기기 선정 · 실제 결선 · 동작 검증 전 과정 수행
* **주요 기술**: Sequence Control, Relay, Timer, Counter, FLS, EOCR, Limit Switch, Temperature Controller

---

## 시스템 구성

| 구분                 | 핵심 제어 기기                   | 구현 동작                                         |
| ------------------ | -------------------------- | --------------------------------------------- |
| **급수 자동/수동 제어**    | FLS, Timer, MC             | 수위 감지 기반 자동 급수 / Timer 기반 수동 순차 기동            |
| **Conveyor 자동화**   | Proximity Sensor, CNT      | 제품 수량 계수 후 Conveyor → 포장 공정 자동 전환             |
| **온도 자동조절**        | TC, Thermocouple           | 설정 온도 도달 시 순환 → 배기 Motor 자동 전환                |
| **공통 보호회로**        | EOCR, Flicker Relay        | 과전류 발생 시 설비 정지 및 Lamp·Buzzer 경보               |
| **최종 복합 Sequence** | EOCR, LS, Timer, Relay, MC | Limit Sensor 입력에 따른 Motor·Lamp 반복 점멸 Sequence |

---

<details>
<summary><b>01. 프로젝트 개요 및 문제 정의</b></summary>

<br>

대한상공회의소 **반도체장비 제어 전문가 과정**에서 산업 현장에서 사용하는 Relay, Timer, Counter, Sensor 등의 동작 원리를 이해하고, 이를 조합한 자동화 Sequence 회로를 직접 설계·배선했습니다.

단순히 주어진 회로를 결선하는 것이 아니라 각 제어기기의 접점 특성과 동작 조건을 이해한 뒤,

**입력 조건 → Relay Logic → Timer / Counter → Output → Protection**

으로 이어지는 전체 Sequence를 도면으로 설계하고 실제 제어반에 구현하는 것이 주요 과제였습니다.

특히 자동화 설비에서는 정상 동작뿐 아니라 Over Current 등 이상 상황에서도 설비를 안전하게 정지시켜야 하기 때문에 **EOCR 및 Interlock을 포함한 보호회로 설계**도 함께 적용했습니다.

최종평가에서는 개별 실습에서 학습한 내용을 종합해 **2개의 EOCR, Limit Switch, Timer, Relay, Magnetic Contactor를 결합한 복합 자동 Sequence**를 직접 설계·배선했습니다.

</details>

---

<details>
<summary><b>02. 사용 기술 / 제어 기기</b></summary>

<br>

| 구분                 | 제어 기기                            | 역할                         |
| ------------------ | -------------------------------- | -------------------------- |
| Relay Logic        | Relay(X), Magnetic Contactor(MC) | 조건 Logic 및 Motor 출력 제어     |
| 유지회로               | 자기유지 회로                          | 입력 해제 후에도 운전 상태 유지         |
| Timer              | T                                | 설정 시간 이후 출력 및 Sequence 전환  |
| Counter            | CNT                              | Sensor 입력 횟수 계수            |
| Level Control      | FLS                              | 수위 감지 및 Pump 자동 제어         |
| Temperature        | TC, Thermocouple                 | 온도 검출 및 설정값 비교             |
| Position Detection | Limit Switch                     | 기계 위치 및 동작 완료 감지           |
| Object Detection   | Proximity Sensor                 | 제품 통과 감지                   |
| Protection         | EOCR                             | Motor Over Current 감지 및 보호 |
| Alarm              | Flicker Relay                    | Lamp / Buzzer 점멸 경보        |

### 작업 범위

`회로 요구사항 분석 → Sequence 설계 → 도면 작성 → 실제 결선 → 동작 확인 → 오류 수정`

</details>

---

<details>
<summary><b>03. 주요 구현 내용</b></summary>

<br>

### 3-1. FLS 기반 급수 자동 / 수동 제어

Selector Switch를 이용해 **AUTO / MANUAL Mode를 전환할 수 있는 급수 제어회로**를 설계했습니다.

자동 Mode에서는 FLS(Floatless Level Switch)가 수위를 감지해 설정 조건에 따라 MC1을 자동으로 동작시키도록 구성했습니다.

수동 Mode에서는 Push Button과 Timer를 이용해 작업자가 직접 운전을 시작하면 **MC1 → MC2가 시간차를 두고 순차적으로 기동**하도록 설계했습니다.

#### 제어 흐름

```text
AUTO Mode
FLS 수위 감지
     ↓
수위 조건 판정
     ↓
MC1 자동 기동
```

```text
MANUAL Mode
PB1 입력
   ↓
MC1 기동
   ↓
Timer 동작
   ↓
MC2 순차 기동
```

<!-- 여기에 급수 제어 회로 사진 추가 -->

<p align="center">
  <img src="이미지경로" width="80%" alt="급수 자동 수동 제어회로">
</p>

---

### 3-2. Proximity Sensor + Counter 기반 Conveyor 자동화

Proximity Sensor로 Conveyor를 통과하는 제품을 감지하고 **Counter(CNT)를 이용해 제품 수량을 계수하는 자동화 회로**를 구성했습니다.

설정된 수량에 도달하기 전에는 Conveyor Motor인 IM1이 동작하며, Counter가 목표 수량을 감지하면 기존 Conveyor 동작을 종료하고 **포장 공정용 IM2로 자동 전환**되도록 Sequence를 설계했습니다.

#### 제어 흐름

```text
제품 진입
   ↓
Proximity Sensor
   ↓
CNT +1
   ↓
설정 수량 확인
   ↓
IM1 정지
   ↓
IM2 자동 기동
```

센서 입력을 단순 Motor ON/OFF 신호로 사용하는 것이 아니라 **Counter와 연계해 생산 수량을 기준으로 다음 공정을 자동 전환하는 Sequence**를 구현했습니다.

<!-- 여기에 컨베이어 회로 사진 추가 -->

<p align="center">
  <img src="이미지경로" width="80%" alt="컨베이어 자동화 회로">
</p>

---

### 3-3. Temperature Controller 기반 자동 온도조절

Thermocouple을 이용해 온도를 측정하고 Temperature Controller의 설정값과 비교해 Motor 동작을 자동 전환하는 회로를 구성했습니다.

정상 온도 범위에서는 순환 Motor를 운전하고, 설정 온도에 도달하면 접점 상태를 전환해 **순환 Motor → 배기 Motor**로 자동 변경되도록 설계했습니다.

#### 제어 흐름

```text
Thermocouple
      ↓
현재 온도 측정
      ↓
TC 설정값 비교
      ↓
설정 온도 미만 → 순환 Motor
설정 온도 도달 → 배기 Motor
```

이를 통해 작업자가 직접 Motor를 조작하지 않아도 온도 조건에 따라 설비가 자동으로 상태를 변경하는 **Feedback 기반 Sequence Control**을 구현했습니다.

<!-- 여기에 온도제어 회로 사진 추가 -->

<p align="center">
  <img src="이미지경로" width="80%" alt="온도 자동조절 회로">
</p>

---

### 3-4. EOCR + Flicker Relay 기반 공통 보호회로

각 실습 회로에는 정상 운전 Logic뿐 아니라 **Motor Over Current 발생 시 설비를 안전하게 정지시키는 EOCR 보호회로**를 함께 적용했습니다.

EOCR이 이상 전류를 감지하면 Magnetic Contactor의 운전 회로를 차단해 Motor를 정지시키고, 동시에 Flicker Relay를 이용해 **Lamp와 Buzzer가 반복적으로 동작하는 Alarm 회로**를 구성했습니다.

#### 보호 흐름

```text
Over Current 발생
       ↓
EOCR 동작
       ↓
Motor 출력 차단
       ↓
설비 안전 정지
       ↓
Flicker Relay
       ↓
Lamp + Buzzer 경보
```

정상 Sequence와 Protection Logic을 별도로 구성하는 것이 아니라 **운전회로 안에 보호 조건을 함께 포함시키는 방식**으로 설계했습니다.

<!-- 여기에 EOCR 보호회로 사진 추가 -->

<p align="center">
  <img src="이미지경로" width="80%" alt="EOCR 보호회로">
</p>

---

### 3-5. 최종평가 – 이중 EOCR 기반 복합 자동 Sequence

**2026.05.08** 최종평가에서는 앞선 실습에서 사용한 제어기기를 종합해 보다 복잡한 Sequence 회로를 설계·배선했습니다.

#### 사용 기기

* EOCR1 / EOCR2
* LS1 / LS2
* Timer T1 / T2
* Relay X1 / X2
* Magnetic Contactor MC1 / MC2
* Yellow Lamp / Red Lamp

#### 동작 조건

```text
LS1 ON
   ↓
Timer Sequence
   ↓
MC1 + Yellow Lamp
   ↓
설정 시간 간격 반복 점멸
```

```text
LS2 ON
   ↓
Timer Sequence
   ↓
MC2 + Red Lamp
   ↓
설정 시간 간격 반복 점멸
```

두 개의 Motor 회로에 각각 EOCR을 적용해 **이중 Overload Protection**을 구성했으며, Limit Switch 입력과 Timer·Relay 접점을 조합해 두 Sequence가 요구조건에 맞춰 반복 동작하도록 구현했습니다.

#### 회로 설계 및 실제 결선

<p align="center">
  <img src="CT1.png" width="48%" alt="최종평가 회로 1">
  <img src="CT2.png" width="48%" alt="최종평가 회로 2">
</p>

<p align="center">
  <img src="CT3.png" width="90%" alt="최종평가 실제 배선 작업">
</p>

</details>

---

<details>
<summary><b>04. 설계 · 배선 검증 포인트</b></summary>

<br>

### 1. Contact 동작 상태를 기준으로 Sequence 검증

Relay와 Timer를 사용할 때는 단순히 배선 위치를 외우는 것이 아니라 **각 단계에서 NO / NC Contact가 어떤 상태로 변화하는지**를 기준으로 전체 Sequence를 확인했습니다.

```text
Input
 ↓
Relay Coil
 ↓
Contact 상태 변화
 ↓
Timer / Counter
 ↓
MC Output
```

회로가 예상과 다르게 동작하는 경우 입력부터 출력까지 접점 상태를 순서대로 확인해 원인을 좁혔습니다.

---

### 2. 자동 / 수동 Mode 회로 분리

자동·수동 겸용 회로에서는 두 Mode의 출력 조건이 동시에 성립하지 않도록 Selector Switch 조건을 분리했습니다.

이를 통해 자동운전 중 수동 입력이 Motor Sequence에 영향을 주거나 반대로 수동운전 조건이 자동 Logic에 영향을 주는 것을 방지했습니다.

---

### 3. Protection Logic 우선 적용

Motor 운전 조건이 충족되더라도 EOCR 등 Protection Contact가 정상 상태일 때만 Magnetic Contactor가 동작하도록 구성했습니다.

```text
운전 조건
   +
Protection 정상
   ↓
MC 동작 허용
```

이를 통해 제어 Sequence보다 **Safety Condition이 우선하도록 회로를 설계하는 습관**을 익혔습니다.

---

### 4. 도면과 실제 배선 비교 검증

회로도상에서는 정상으로 보이더라도 실제 배선 과정에서는 단자 번호나 접점 위치를 잘못 연결할 수 있기 때문에,

`도면 → 단자 번호 → 실제 배선 → 동작`

순서로 하나씩 대조하며 검증했습니다.

이를 통해 회로 설계 능력뿐 아니라 실제 현장에서 필요한 **Drawing Reading 및 Wiring Verification 경험**을 함께 쌓았습니다.

</details>

---

<details>
<summary><b>05. 결과 및 성과</b></summary>

<br>

* **급수 · Conveyor · 온도제어 · Protection 등 4종 이상의 Sequence 회로 직접 설계 및 배선**
* FLS 기반 **급수 AUTO / MANUAL 제어 구현**
* Proximity Sensor + Counter 기반 **제품 수량별 Conveyor 자동 전환 구현**
* Temperature Controller 기반 **자동 온도조절 Sequence 구현**
* EOCR + Flicker Relay 기반 **Motor Over Current Protection 및 Alarm 회로 구현**
* Relay · Timer · Counter · Limit Switch 등 다양한 제어기기 조합 경험
* **EOCR1/2 · LS1/2 · Timer1/2 · Relay · MC 기반 최종 복합 Sequence 평가 통과**
* 회로 설계부터 실제 결선 및 동작 검증까지 전 과정 수행
* **반도체장비 제어 전문가 과정 이수**

</details>

---

<details>
<summary><b>06. 회고 / 배운 점</b></summary>

<br>

이번 과정을 통해 Relay, Timer, Counter, Sensor와 같은 개별 제어기기의 기능을 아는 것과 **여러 기기를 조합해 하나의 Sequence를 설계하는 것은 다른 문제**라는 점을 체감했습니다.

각 기기의 Coil과 NO / NC Contact가 동작하면서 다음 조건에 어떤 영향을 주는지를 순서대로 이해해야 복합 Sequence를 안정적으로 구성할 수 있었습니다.

특히 Motor를 동작시키는 Logic만 구현하는 것이 아니라 EOCR과 Interlock 같은 Protection Circuit을 항상 함께 적용하면서 **설비 제어에서는 정상 동작보다 이상 상황에서 안전하게 정지시키는 조건이 우선되어야 한다는 설계 원칙**을 익혔습니다.

또한 회로도가 논리적으로 맞더라도 실제 결선 과정에서 단자나 접점을 잘못 연결하면 설비가 정상 동작하지 않기 때문에, 도면과 실제 배선을 하나씩 비교하며 원인을 찾는 과정을 반복했습니다.

이를 통해 **Sequence Drawing 설계와 실제 Wiring 능력을 함께 갖춰야 현장 장비의 동작 원인을 정확히 분석하고 Troubleshooting할 수 있다는 점**을 배웠습니다.

</details>
