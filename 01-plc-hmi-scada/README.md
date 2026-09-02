# PLC · HMI · SCADA 기반 자동화 제어 및 모니터링 시스템

### Mitsubishi PLC 제어 · GT Designer3 HMI · X-SCADA 상위 모니터링

> **센서·액추에이터 제어부터 자동 운전 시퀀스, HMI 운전 화면, Servo 위치제어, OPC UA 기반 SCADA 연동까지 구현하며
> Field Device → PLC → HMI → SCADA로 이어지는 산업자동화 시스템의 제어 및 데이터 흐름을 학습한 프로젝트**

* **PLC**: Mitsubishi PLC / GX Works2
* **HMI**: GT Designer3
* **SCADA**: X-SCADA
* **통신**: Ethernet / OPC UA
* **Middleware**: KEPServerEX
* **주요 제어 대상**: Sensor / Solenoid / Conveyor / MPS / Servo Motor

---

## 시스템 구성

<p align="center">
  <img src="01-architecture_3.png" alt="시스템 구성도" width="90%">
</p>

* **제어 흐름**
  `Sensor / Switch → PLC Sequence Logic → Actuator`

* **운전·모니터링 흐름**
  `Operator ↔ HMI ↔ PLC`

* **상위 데이터 흐름**
  `PLC → KEPServerEX → OPC UA → X-SCADA`

---

<details>
<summary><b>01. 프로젝트 개요</b></summary>

<br>

Mitsubishi PLC를 기반으로 센서·액추에이터의 **Digital I/O 제어와 자동 운전 시퀀스**를 구성하고, GT Designer3 HMI와 X-SCADA를 연동하여 설비의 **운전·제어·상태 모니터링 기능**을 구현했습니다.

PLC 프로그램 작성에 그치지 않고 HMI에서 작업자가 장비를 직접 조작하고 상태를 확인할 수 있도록 화면을 구성했으며, SCADA에서는 PLC 데이터를 수집하여 현장 설비의 상태를 상위 PC에서 모니터링할 수 있도록 연동했습니다.

또한 MPS 설비와 Servo Motor를 활용하여 센서 입력에 따른 순차 제어와 위치제어를 실습하며 PLC 프로그램이 실제 물리 장비의 동작으로 연결되는 과정을 확인했습니다.

이를 통해

```text
Field Device
     ↓
    PLC
     ↓
    HMI
     ↓
KEPServerEX
     ↓
   OPC UA
     ↓
  X-SCADA
```

로 이어지는 **산업자동화 시스템의 제어 계층과 데이터 흐름**을 학습했습니다.

</details>

---

<details>
<summary><b>02. 사용 기술 / 툴</b></summary>

<br>

| 구분             | 기술 / 툴                             |
| -------------- | ---------------------------------- |
| PLC            | Mitsubishi PLC / GX Works2         |
| HMI            | GT Designer3                       |
| SCADA          | X-SCADA                            |
| Communication  | Ethernet / OPC UA                  |
| OPC Server     | KEPServerEX                        |
| Control        | Digital I/O / Sequence Control     |
| Equipment      | Sensor / Solenoid / Conveyor / MPS |
| Motion Control | Servo Motor / Positioning Module   |

</details>

---

<details>
<summary><b>03. 주요 구현 내용</b></summary>

<br>

### 3-1. PLC 기반 Digital I/O 제어

GX Works2를 이용하여 센서와 스위치의 입력 상태를 확인하고, 조건에 따라 솔레노이드·램프·컨베이어 등의 출력 장치를 제어하는 PLC 프로그램을 구성했습니다.

#### 주요 구현 기능

* A접점 / B접점
* 자기유지 회로
* 인터록 회로
* 자기유지 + 인터록 구성
* Timer / Counter
* SET / RST
* Rising Pulse
* 연속동작 / 단속동작
* Digital Input / Output 제어

```text
Sensor Input
     ↓
PLC Input Device
     ↓
Sequence Logic
     ↓
PLC Output Device
     ↓
Actuator
```

PLC 내부 Device의 상태와 실제 장비의 동작을 함께 확인하며 **래더 프로그램의 논리 신호가 물리적인 I/O 동작으로 연결되는 구조**를 확인했습니다.

<!-- PLC 기본 I/O 및 래더로직 이미지 삽입 -->

---

### 3-2. 자동 운전 시퀀스 및 MPS 제어

센서 입력과 장비의 동작 완료 조건을 이용하여 여러 출력 장치가 순차적으로 동작하는 자동 운전 시퀀스를 구성했습니다.

```text
START
  ↓
초기 조건 확인
  ↓
Sensor Input
  ↓
1단계 장비 동작
  ↓
완료 조건 확인
  ↓
2단계 장비 동작
  ↓
Cycle Complete
```

Timer, Counter, SET/RST 및 센서 입력을 조합해 단순 ON/OFF 제어에서 **조건 기반 순차 제어 방식**으로 프로그램을 확장했습니다.

MPS 장비를 이용한 실습에서는 각 공정의 센서 입력과 장비 상태를 다음 동작의 조건으로 사용하여 하나의 공정이 완료된 후 다음 공정으로 넘어가도록 시퀀스를 구성했습니다.

<!-- 자동 운전 시퀀스 이미지 삽입 -->

<!-- MPS 실제 장비 및 PLC 프로그램 이미지 삽입 -->

---

### 3-3. Auto / Manual 운전 구성

설비 점검과 자동 생산 운전을 구분하기 위해 **Manual / Auto Mode**를 구성했습니다.

#### Manual Mode

작업자가 개별 출력 장치를 직접 조작하여 설비의 I/O와 장비 상태를 점검할 수 있도록 구성했습니다.

```text
Manual Command
      ↓
     PLC
      ↓
Individual Output
      ↓
 Equipment
```

#### Auto Mode

초기 운전 조건을 확인한 후 Start 신호가 입력되면 작성한 시퀀스에 따라 설비가 자동으로 동작하도록 구성했습니다.

```text
AUTO
 ↓
START
 ↓
Condition Check
 ↓
Sequence Logic
 ↓
Equipment Operation
```

이를 통해 실제 자동화 설비에서 사용하는 **점검용 Manual 운전과 생산용 Auto 운전의 역할 차이**를 학습했습니다.

---

### 3-4. GT Designer3 기반 HMI 구성

GT Designer3를 이용하여 PLC와 연동되는 설비 운전 화면을 제작했습니다.

작업자가 PLC 프로그램을 직접 확인하지 않아도 **설비 상태를 확인하고 필요한 제어 명령을 입력할 수 있도록 HMI 화면을 구성**했습니다.

#### 구현 기능

* Button / Switch
* Lamp
* Alternate Switch
* Numerical Display
* Numerical Input
* 날짜 / 시간 표시
* Screen 전환
* 문자 표시 및 변환
* Language 전환
* Graph 표시
* Alarm History 표시

```text
Operator
   ↓
HMI Button / Input
   ↓
PLC Device
   ↓
Sequence Logic
   ↓
Equipment
```

반대로 실제 장비와 PLC의 상태는 다음 경로로 HMI에 표시했습니다.

```text
Equipment Status
      ↓
 PLC Device
      ↓
HMI Lamp / Text
      ↓
Numerical Display
```

HMI에서 값을 입력하면 연결된 PLC Device에 해당 데이터가 반영되고, PLC 내부 데이터가 변경되면 HMI의 Lamp·Text·Numerical Display도 함께 변경되는 **HMI ↔ PLC 양방향 데이터 연동**을 확인했습니다.

<!-- HMI 메인 화면 이미지 삽입 -->

<!-- HMI Button / Lamp / Numerical Input 이미지 삽입 -->

---

### 3-5. HMI 설비 상태 모니터링

PLC 내부 Device와 HMI Object를 연결하여 현장 설비의 상태를 HMI 화면에서 실시간으로 확인할 수 있도록 구성했습니다.

#### 주요 모니터링 항목

| 항목             | 모니터링 내용          |
| -------------- | ---------------- |
| Sensor Input   | 센서 입력 ON/OFF 상태  |
| Output         | 액추에이터 출력 상태      |
| Operation Mode | Auto / Manual 상태 |
| Equipment      | 설비 운전 상태         |
| Set Value      | 작업자가 입력한 설정값     |
| Current Value  | PLC 내부 현재 데이터    |
| Alarm          | 설비 이상 및 경고 상태    |
| Process        | 공정 진행 단계         |

```text
PLC Device
     ↓
HMI Object
     ↓
Lamp / Text / Value / Graph
```

이를 통해 PLC의 Bit 및 Word 데이터를 작업자가 쉽게 확인할 수 있는 **시각적인 운전·모니터링 정보로 변환**했습니다.

---

### 3-6. Servo Motor 위치제어

PLC와 위치결정 모듈, Servo Amplifier, Servo Motor를 연결하여 위치제어를 실습했습니다.

#### 주요 구현 기능

* JOG 운전
* 원점 복귀
* 기계 원점 복귀
* 고속 원점 복귀
* 위치 데이터 설정
* 축 현재 위치 확인
* 축 속도 확인
* 축 상태 확인
* Servo Error 확인
* 위치결정 시퀀스 구성

```text
PLC
 ↓
Positioning Module
 ↓
Servo Amplifier
 ↓
Servo Motor
 ↓
Position Control
```

특히 Incremental Encoder 방식에서는 장비 전원을 다시 인가하면 현재 절대 위치를 알 수 없기 때문에 **자동 운전 전 원점 복귀 과정이 필요하다는 Servo 위치제어의 기본 구조**를 확인했습니다.

<!-- Servo 및 위치제어 프로그램 이미지 삽입 -->

---

### 3-7. X-SCADA 기반 상위 모니터링

현장의 PLC 데이터를 상위 PC에서 확인할 수 있도록 X-SCADA와 KEPServerEX를 이용한 데이터 연동 환경을 구성했습니다.

```text
PLC
 ↓
Communication
 ↓
KEPServerEX
 ↓
OPC UA
 ↓
X-SCADA
 ↓
Monitoring
```

KEPServerEX에서 PLC Device를 OPC Tag로 구성하고, X-SCADA에서 해당 데이터를 연동하여 현장 설비의 상태를 상위 시스템에서 확인할 수 있도록 구성했습니다.

#### 주요 구현 내용

* PLC 데이터 수집
* KEPServerEX Tag 구성
* OPC UA 통신
* SCADA Text / Value 표시
* 설비 상태 모니터링
* Alarm / Event 데이터 활용

이를 통해 PLC 내부 데이터가 Middleware와 통신 프로토콜을 거쳐 상위 SCADA 시스템까지 전달되는 데이터 흐름을 확인했습니다.

<details>
<summary><b>📎 KEPServerEX / SCADA 설정 화면 보기</b></summary>

<br>

<!-- KEPServerEX Tag 설정 이미지 삽입 -->

<!-- OPC UA 통신 설정 이미지 삽입 -->

<!-- X-SCADA 모니터링 화면 이미지 삽입 -->

</details>

<br>

---

### 3-8. PLC · HMI · SCADA 전체 데이터 흐름

전체 시스템은 현장 장비, 제어 계층, 운전 계층, 상위 모니터링 계층이 연결되는 구조로 구성했습니다.

```text
            [Field Device]
             Sensor / SW
                  │
                  ▼
             ┌─────────┐
             │   PLC   │
             └────┬────┘
                  │
        ┌─────────┼─────────┐
        │                   │
        ▼                   ▼
   [Actuator]             [HMI]
 Motor / Cylinder            │
 Conveyor                    │
                             ▼
                       Operator Control

             PLC Data
                  │
                  ▼
            KEPServerEX
                  │
               OPC UA
                  │
                  ▼
              X-SCADA
                  │
                  ▼
             Monitoring
```

이를 통해 단순 PLC 프로그램 작성이 아니라 **Field Device → PLC → HMI → OPC Server → SCADA로 이어지는 산업자동화 시스템의 계층 구조와 각 계층의 역할**을 경험했습니다.

</details>

---

<details>
<summary><b>04. 트러블슈팅 ⭐</b></summary>

<br>

### Trouble 01. HMI Alternate 버튼과 PLC 자기유지 로직 충돌

#### Problem

HMI에서 하나의 버튼으로 ON/OFF 상태를 전환하기 위해 **Alternate Switch**를 적용했지만, PLC 프로그램에서도 동일 Device에 자기유지 로직을 구성하면서 버튼을 눌렀을 때 원하는 방식으로 상태가 전환되지 않는 문제가 발생했습니다.

#### Analysis

HMI와 PLC 로직을 각각 확인한 결과 동일한 상태를 두 영역에서 동시에 유지하고 있었습니다.

```text
HMI Alternate 기능
        +
PLC 자기유지 기능
        ↓
두 영역에서 동시에 상태 유지
        ↓
ON / OFF 제어 충돌
```

즉, Device Address 자체의 문제가 아니라 **상태를 유지하는 제어 주체가 HMI와 PLC 양쪽에 중복되어 있다는 점**이 원인이었습니다.

#### Solution

HMI의 Alternate 기능을 사용하는 경우 PLC에서는 해당 Device의 상태를 별도로 자기유지하지 않고, HMI에서 전달된 Device 값을 기준으로 출력 장치가 동작하도록 로직을 단순화했습니다.

```text
HMI Alternate
      ↓
PLC Device
      ↓
Output Condition
      ↓
Equipment
```

이를 통해 하나의 상태에 대한 관리 주체를 HMI로 명확하게 구분했습니다.

#### Result

HMI 버튼을 누를 때마다 ON/OFF 상태가 정상적으로 전환되었고, PLC 출력도 해당 상태에 따라 정상적으로 동작하는 것을 확인했습니다.

> **Learned**
> HMI와 PLC를 연동할 때는 Device Address만 일치시키는 것이 아니라 **상태를 HMI와 PLC 중 어느 영역에서 관리할지까지 고려해 제어 구조를 설계해야 한다는 점**을 배웠습니다.

</details>

---

<details>
<summary><b>05. 결과 및 성과</b></summary>

<br>

* Mitsubishi PLC 기반 **Digital I/O 제어 로직 구현**
* Timer / Counter / SET-RST / 인터록을 이용한 **자동 운전 시퀀스 구성**
* **Auto / Manual Mode**를 구분한 설비 운전 로직 구현
* Sensor / Solenoid / Conveyor 등 **실제 Field Device 연동**
* MPS 장비를 이용한 **순차 공정 제어**
* Positioning Module과 Servo Motor를 이용한 **JOG 및 위치제어 실습**
* GT Designer3 기반 **HMI 운전·모니터링 화면 구현**
* **HMI ↔ PLC 양방향 데이터 연동**
* Lamp / Numerical Display / Graph / Alarm History 등 HMI 기능 구현
* KEPServerEX를 이용한 **PLC Device → OPC Tag 변환**
* **OPC UA 기반 X-SCADA 데이터 연동**
* 현장 PLC 데이터를 상위 PC에서 확인할 수 있는 **SCADA 모니터링 환경 구성**
* **Field Device → PLC → HMI → OPC → SCADA** 전체 데이터 흐름 이해

</details>

---

<details>
<summary><b>06. 회고 / 배운 점</b></summary>

<br>

이번 실습을 통해 PLC 프로그램은 단순히 래더로직을 작성하는 작업이 아니라, **실제 센서 입력과 액추에이터 동작을 조건과 순서에 맞게 연결하는 설비 제어의 중심 역할**이라는 점을 이해했습니다.

특히 자동 운전 시퀀스를 구성하면서 한 단계의 동작 완료 조건이 다음 단계의 시작 조건이 되는 구조를 구현했고, Manual / Auto Mode를 구분하면서 실제 산업 설비에서 점검 운전과 자동 생산 운전이 어떻게 분리되는지도 확인했습니다.

GT Designer3 HMI를 PLC와 연결하면서는 PLC 내부의 Bit·Word Device가 작업자 관점에서는 버튼, 램프, 수치, 알람과 같은 **운전 인터페이스로 변환되는 과정**을 경험했습니다.

또한 HMI Alternate Switch와 PLC 자기유지 로직이 충돌했던 문제를 해결하면서 단순히 Device Address를 연결하는 것보다 **하나의 신호를 어느 계층에서 생성하고 유지할 것인지 제어 주체를 명확하게 정의하는 것이 중요하다는 점**을 배웠습니다.

Servo 위치제어에서는 원점 복귀와 현재 위치 관리의 중요성을 확인했으며, KEPServerEX와 OPC UA를 이용한 X-SCADA 연동을 통해 PLC의 현장 데이터가 상위 모니터링 시스템까지 전달되는 구조를 직접 구성했습니다.

결과적으로 개별 PLC 기능에 대한 실습을 넘어

**Field Device → PLC → HMI → Communication → SCADA**

로 이어지는 전체 자동화 시스템을 하나의 데이터 및 제어 흐름으로 바라보는 관점을 익혔습니다.

</details>
