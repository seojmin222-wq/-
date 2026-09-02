
# PLC · HMI · SCADA 기반 자동화 제어 및 모니터링 시스템

## 프로젝트 개요

Mitsubishi PLC를 기반으로 센서·액추에이터의 I/O 제어와 자동 운전 시퀀스를 구성하고, **GT Designer3 HMI와 SCADA를 연동하여 설비의 운전·제어·모니터링 기능을 구현**했습니다.

PLC 프로그램 작성에 그치지 않고 HMI에서 작업자가 장비를 직접 조작하고 상태를 확인할 수 있도록 화면을 구성했으며, SCADA에서는 PLC 데이터를 수집하여 설비 상태를 상위 시스템에서 모니터링할 수 있도록 연동했습니다.

이를 통해

**Field Device → PLC → HMI → SCADA**

로 이어지는 산업 자동화 시스템의 제어 및 데이터 흐름을 학습했습니다.

---

## 사용 기술

| 구분             | 사용 기술                              |
| -------------- | ---------------------------------- |
| PLC            | Mitsubishi PLC / GX Works2         |
| HMI            | GT Designer3                       |
| SCADA          | X-SCADA                            |
| Communication  | Ethernet / OPC UA                  |
| OPC Server     | KEPServerEX                        |
| Control        | Digital I/O / Sequence Control     |
| Equipment      | Sensor / Solenoid / Conveyor / MPS |
| Motion Control | Servo Motor / Positioning Module   |

---

## 시스템 구성

```text
[Sensor / Switch]
       │
       ▼
┌───────────────┐
│      PLC      │
│ Sequence Logic│
└───────┬───────┘
        │
   ┌────┴─────────────┐
   │                  │
   ▼                  ▼
[Actuator]          [HMI]
Motor /             │
Cylinder /           │
Conveyor             ▼
                  Operator
                  Control
                     │
                     ▼
                 [SCADA]
                     │
                     ▼
              Monitoring /
              Data Display
```

---

# 1. PLC 기반 I/O 제어

센서와 스위치의 입력 상태를 PLC에서 확인하고, 조건에 따라 솔레노이드·램프·컨베이어 등의 출력 장치를 제어했습니다.

### 주요 구현 내용

* A접점 / B접점 활용
* 자기유지 회로
* 인터록 회로
* 자기유지 + 인터록 구성
* Timer / Counter
* SET / RST
* Rising Pulse
* 연속동작 / 단속동작
* Auto / Manual 제어
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

PLC 내부 신호와 실제 장비의 동작을 함께 확인하며 **프로그램 로직과 물리적인 I/O 신호가 연결되는 과정**을 실습했습니다.

<!-- 기존 PLC 프로그램 / I/O 관련 이미지 삽입 -->

---

# 2. 자동 운전 시퀀스 구성

여러 출력 장치가 지정된 조건에 따라 순차적으로 동작하도록 자동 운전 시퀀스를 구성했습니다.

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

Timer, Counter, SET/RST 및 센서 입력을 조합하여 단순 ON/OFF 제어에서 **조건 기반 순차 제어**로 프로그램을 확장했습니다.

MPS 장비를 이용한 실습에서는 센서 입력과 장비 상태에 따라 각 공정이 순차적으로 진행되도록 제어했습니다.

<!-- 기존 자동 시퀀스 / MPS 관련 이미지 삽입 -->

---

# 3. Auto / Manual 운전 구성

설비 점검과 자동 생산 운전을 구분할 수 있도록 Manual / Auto 운전 방식을 구성했습니다.

### Manual Mode

작업자가 개별 출력 장치를 직접 동작시켜 장비 상태와 I/O를 점검할 수 있도록 구성했습니다.

```text
Manual Command
      ↓
PLC
      ↓
Individual Output
      ↓
Equipment
```

### Auto Mode

초기 조건을 확인한 후 Start 신호가 입력되면 작성한 시퀀스에 따라 설비가 자동으로 동작하도록 구성했습니다.

```text
AUTO
 ↓
START
 ↓
Condition Check
 ↓
Sequence
 ↓
Equipment Operation
```

---

# 4. GT Designer3 기반 HMI 구성

GT Designer3를 이용하여 PLC와 연동되는 HMI 운전 화면을 구성했습니다.

작업자가 PLC 프로그램을 직접 확인하지 않아도 **설비 상태를 확인하고 필요한 제어 명령을 입력할 수 있도록 화면을 구성**했습니다.

### 구현 기능

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

Equipment Status
   ↓
PLC Device
   ↓
HMI Lamp / Numerical Display
```

HMI에서 값을 입력하면 PLC Device에 해당 값이 반영되고, PLC 내부 데이터의 변화가 다시 HMI 화면에 표시되는 **양방향 데이터 연동**을 확인했습니다.

<!-- 기존 HMI 메인 화면 이미지 삽입 -->

<!-- 기존 HMI 버튼 / 램프 / 수치입력 이미지 삽입 -->

---

# 5. HMI를 통한 설비 상태 모니터링

PLC 내부 Device와 HMI Object를 연결하여 실제 장비의 상태를 화면에서 확인할 수 있도록 구성했습니다.

### 주요 모니터링 항목

* Sensor Input
* Output 상태
* Auto / Manual 상태
* 운전 상태
* 설정값
* 현재값
* Alarm
* 공정 진행 상태

```text
PLC Device
     ↓
HMI Object
     ↓
Lamp / Text / Value / Graph
```

이를 통해 PLC 내부의 Bit 및 Word 데이터를 작업자가 직관적인 화면으로 확인할 수 있도록 구성했습니다.

---

# 6. Servo 위치제어

PLC와 위치결정 모듈, Servo Amplifier, Servo Motor를 연결하여 위치제어를 실습했습니다.

### 주요 구현 내용

* JOG 운전
* 원점 복귀
* 기계 원점 복귀
* 고속 원점 복귀
* 위치 데이터 설정
* 축 현재 위치 확인
* 축 속도 확인
* 축 상태 및 Error 확인
* 위치결정 시퀀스

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

Incremental Encoder 사용 시 전원 재인가 후 현재 위치를 알 수 없기 때문에 **원점 복귀 과정이 선행되어야 한다는 위치제어의 기본 구조**를 확인했습니다.

<!-- 기존 Servo / 위치제어 이미지 삽입 -->

---

# 7. SCADA 기반 상위 모니터링

PLC 데이터를 상위 PC에서 확인할 수 있도록 SCADA 시스템을 구성했습니다.

X-SCADA에서 PLC 관련 데이터를 화면에 표시하고, OPC UA 기반 데이터 연동을 실습했습니다.

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

SCADA 화면에서는 현장 PLC의 값을 상위 시스템에서 확인할 수 있도록 구성했습니다.

### 주요 구현 내용

* PLC 데이터 수집
* OPC UA 통신
* KEPServerEX Tag 연동
* SCADA Text / Value 표시
* 설비 상태 모니터링
* Alarm / Event 데이터 활용

<!-- 기존 SCADA / KEPServerEX 이미지 삽입 -->

<!-- 기존 SCADA 모니터링 화면 이미지 삽입 -->

---

# 8. PLC · HMI · SCADA 데이터 흐름

전체 시스템의 데이터 흐름은 다음과 같이 구성했습니다.

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

이를 통해 현장의 센서 및 액추에이터부터 PLC 제어, HMI 운전, SCADA 상위 모니터링으로 이어지는 **산업 자동화 시스템의 계층 구조**를 경험했습니다.

---

# Troubleshooting

## HMI Alternate 버튼과 PLC 자기유지 로직 충돌

### 문제

HMI에서 하나의 버튼으로 ON/OFF를 전환하기 위해 Alternate Switch를 적용했지만, PLC 프로그램에서도 자기유지 로직을 사용하면서 원하는 방식으로 상태가 전환되지 않았습니다.

### 원인

```text
HMI Alternate 기능
        +
PLC 자기유지 기능
        ↓
두 영역에서 동시에 상태 유지
        ↓
ON/OFF 제어 충돌
```

HMI와 PLC 양쪽에서 동일한 상태 유지 기능을 처리하고 있어 제어 주체가 중복된 것이 원인이었습니다.

### 해결

HMI의 Alternate 기능을 사용하는 경우 PLC 측에서는 해당 Device를 기준으로 출력이 동작하도록 로직을 단순화하여 상태 관리 주체를 명확하게 구분했습니다.

### 배운 점

HMI와 PLC를 연동할 때는 단순히 Device Address만 맞추는 것이 아니라 **버튼의 동작 방식과 상태를 어느 프로그램에서 관리할지까지 고려하여 설계해야 한다는 점**을 확인했습니다.

---

# 결과

PLC를 이용한 기본 I/O 제어부터 자동 운전 시퀀스, MPS 장비 제어, Servo 위치제어까지 단계적으로 실습하고 이를 GT Designer3 HMI와 연동하여 작업자가 설비를 조작하고 상태를 확인할 수 있는 화면을 구성했습니다.

추가로 OPC UA와 X-SCADA를 이용해 PLC 데이터를 상위 모니터링 시스템까지 전달하면서

**Field Device → PLC → HMI → OPC → SCADA**

로 이어지는 자동화 시스템의 전체 데이터 흐름을 이해했습니다.

### 핵심 경험

* PLC Sequence Programming
* Digital I/O
* Auto / Manual Control
* Sensor / Actuator Control
* MPS Equipment Control
* Servo Position Control
* GT Designer3 HMI
* HMI ↔ PLC 양방향 데이터 연동
* Alarm / Graph / Numerical Display
* OPC UA
* KEPServerEX
* X-SCADA Monitoring
