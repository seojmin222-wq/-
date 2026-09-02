# C# · Arduino · Cognex 기반 PC 제어 및 비전 자동화 시스템

## 프로젝트 개요

C# Windows Forms 기반의 PC 제어 프로그램을 제작하고 **Arduino, Cognex In-Sight 비전 카메라, MPS 자동화 장비를 Serial 및 Ethernet 통신으로 연동**했습니다.

PC 프로그램에서 장비의 Digital I/O 상태를 확인하고 출력 장치를 직접 제어할 수 있도록 구성했으며, Cognex SDK를 활용하여 비전 카메라 영상과 검사 결과를 PC 제어 화면에 통합했습니다.

또한 Modbus TCP를 이용해 PC와 MPS 장비 간 데이터를 주고받으며

**PC Control → Controller → Vision → Equipment**

로 이어지는 산업용 PC 제어 시스템의 통신 구조를 실습했습니다.

---

## 사용 기술

| 구분         | 사용 기술                 |
| ---------- | --------------------- |
| PC Control | C# / Windows Forms    |
| MCU        | Arduino               |
| Vision     | Cognex In-Sight       |
| Vision SDK | Cognex In-Sight SDK   |
| Serial     | SerialPort            |
| Network    | Ethernet / TCP/IP     |
| Protocol   | Modbus TCP            |
| Library    | EasyModbus            |
| Control    | Digital I/O           |
| Equipment  | MPS Automation System |

---

## 시스템 구성

```text
                  ┌──────────────────┐
                  │ C# Windows Forms │
                  │    PC Control    │
                  └────────┬─────────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
           Serial       Ethernet      Modbus TCP
             │             │             │
             ▼             ▼             ▼
         ┌────────┐   ┌──────────┐   ┌────────┐
         │Arduino │   │ Cognex   │   │  MPS   │
         │        │   │ In-Sight │   │        │
         └───┬────┘   └────┬─────┘   └───┬────┘
             │              │             │
             ▼              ▼             ▼
       Digital I/O     Vision Result   Equipment
                                         Control
```

---

# 1. Arduino 기반 산업용 Digital I/O 제어

산업용 Arduino와 DC 24V 입·출력 장치를 연결하여 실제 외부 신호를 처리했습니다.

### 주요 구현 내용

* Digital INPUT
* Digital OUTPUT
* Sink / Source 방식 확인
* Lamp 제어
* Buzzer 제어
* External Switch 입력
* Conveyor 제어

```text
External Input
      ↓
Arduino
      ↓
Control Logic
      ↓
Digital Output
      ↓
Lamp / Buzzer / Conveyor
```

기본적인 ON/OFF 제어에서 시작해 Start 신호에 따라 여러 출력 장치가 순차적으로 동작하는 제어까지 확장했습니다.

<!-- Arduino I/O 배선 이미지 삽입 -->

---

# 2. C# Windows Forms 기반 PC 제어 화면

Visual Studio Windows Forms를 이용하여 외부 장비를 제어하고 상태를 확인할 수 있는 PC 프로그램을 제작했습니다.

### 구현 기능

* COM Port 선택
* Connect / Disconnect
* Start / Stop
* Auto Control
* Digital Input 상태 표시
* Digital Output 제어
* 통신 연결 상태 표시
* Button 상태에 따른 색상 변경

```text
Operator
   ↓
C# Windows Forms
   ↓
Communication
   ↓
Arduino / Equipment
```

장비의 상태를 Serial Monitor에서만 확인하는 것이 아니라 **작업자가 PC 화면을 통해 장비를 조작하고 상태를 확인할 수 있도록 구성**했습니다.

<!-- C# PC 제어 화면 이미지 삽입 -->

---

# 3. C# ↔ Arduino Serial 통신

C#의 `SerialPort`를 이용하여 PC와 Arduino 간 데이터 통신을 구성했습니다.

```text
C# PC Program
      │
      │ Serial
      ▼
   Arduino
      │
 ┌────┴────┐
 ▼         ▼
Input    Output
```

사용자가 PC 프로그램에서 COM Port를 선택하여 Arduino와 연결하고, Arduino의 입력 데이터를 PC로 전달하거나 PC에서 출력 명령을 전송할 수 있도록 구성했습니다.

---

# 4. Cognex In-Sight 비전 검사

Cognex In-Sight 카메라를 Ethernet으로 연결하고 In-Sight Explorer에서 검사 환경을 구성했습니다.

### 주요 구현 내용

* Camera Network 연결
* IP 설정
* Image Acquisition
* 검사 영역 설정
* 부품 검색
* 부품 검사
* QR Code 인식
* 검사 Result 설정
* Input / Output Data 설정

```text
Inspection Target
        ↓
Cognex In-Sight
        ↓
Image Acquisition
        ↓
Vision Tool
        ↓
Inspection Result
```

이미지를 단순히 확인하는 데 그치지 않고 **검사 결과를 외부 제어 시스템에서 사용할 수 있는 데이터로 설정**했습니다.

<!-- In-Sight Explorer 검사 화면 이미지 삽입 -->

---

# 5. Cognex SDK 기반 C# 연동

Cognex SDK를 Windows Forms 프로젝트에 적용하여 비전 카메라를 자체 PC 제어 프로그램과 연결했습니다.

사용한 주요 Library는 다음과 같습니다.

```csharp
using Cognex.InSight;
using Cognex.InSight.Cell;
using Cognex.InSight.Controls;
using Cognex.InSight.Extensions;
using Cognex.InSight.Graphic;
using Cognex.InSight.Sensor;
```

이를 통해 별도의 In-Sight Explorer 화면만 사용하는 방식에서 확장하여 **C# 프로그램에서 직접 Cognex 카메라를 연결하고 제어하는 구조**를 실습했습니다.

---

# 6. PC 제어화면에 Cognex 영상 표시

Cognex Display Control을 Windows Forms 화면에 적용하여 PC 프로그램 내부에서 카메라 영상을 확인할 수 있도록 구성했습니다.

```text
┌───────────────────────────────┐
│       C# PC Control           │
│                               │
│ Equipment Status              │
│ Digital I/O                   │
│                               │
│ ┌───────────────────────────┐ │
│ │ Cognex Camera Live View   │ │
│ └───────────────────────────┘ │
│                               │
│ Inspection Result             │
└───────────────────────────────┘
```

이를 통해 하나의 화면에서

* 장비 상태
* Digital I/O
* 비전 카메라 영상
* 검사 결과

를 함께 확인할 수 있도록 구성했습니다.

<!-- Cognex 영상이 표시된 PC Control 화면 이미지 삽입 -->

---

# 7. Camera Trigger 제어

비전 검사를 두 가지 방식으로 시작할 수 있도록 Trigger 제어를 실습했습니다.

### Software Trigger

PC 프로그램의 Trigger 버튼을 이용해 Cognex 카메라의 이미지 취득을 실행했습니다.

```text
C# Trigger Button
       ↓
Cognex Camera
       ↓
Image Acquisition
       ↓
Inspection
```

### External Trigger

외부 버튼 입력을 이용해서도 카메라 Trigger가 동작하도록 구성했습니다.

```text
External Button
       ↓
Controller Input
       ↓
Trigger Signal
       ↓
Cognex Camera
```

이를 통해 PC 명령뿐만 아니라 실제 설비의 외부 입력을 이용한 검사 시작 구조를 경험했습니다.

---

# 8. Cognex 검사 결과 PC 수신

카메라에서 생성된 검사 데이터를 C# PC 프로그램에서 확인할 수 있도록 연동했습니다.

```text
Cognex Inspection
        ↓
     Result
        ↓
C# PC Control
        ↓
Result Display
```

이를 통해 카메라 내부의 검사 결과를 외부 프로그램으로 전달하고 **상위 제어 프로그램에서 결과값을 활용하는 구조**를 실습했습니다.

---

# 9. Modbus TCP 기반 PC ↔ MPS 통신

C# 프로그램과 MPS 자동화 장비 사이의 Ethernet 통신을 위해 Modbus TCP를 적용했습니다.

C#에서는 `EasyModbus` Library를 사용했습니다.

```csharp
ModbusClient modbus = new ModbusClient();
```

### 구현 기능

* IP Address / Port 설정
* Connect / Disconnect
* Discrete Input Read
* Coil Write
* Holding Register Read
* Holding Register Write
* Auto 명령
* Start / Stop 명령

```text
C# PC Control
      │
   Modbus TCP
      │
      ▼
     MPS
```

---

# 10. PC에서 MPS I/O 모니터링 및 제어

MPS 장비의 Digital Input 상태를 Modbus TCP로 읽어 PC 화면에 표시하고, Digital Output은 PC에서 직접 제어할 수 있도록 구성했습니다.

```text
MPS Input
    ↓
Modbus TCP
    ↓
C# Program
    ↓
Input Display

C# Output Command
    ↓
Modbus TCP
    ↓
MPS Output
```

Input은 CheckBox 형태로 상태를 확인하고, Output은 PC 프로그램을 통해 ON/OFF 할 수 있도록 구성했습니다.

---

# 11. Holding Register 기반 Auto / Start / Stop

Modbus Holding Register 값을 이용하여 장비의 운전 명령을 전달했습니다.

```text
C# AUTO Button
       ↓
Holding Register D0
       ↓
Controller
       ↓
AUTO Mode

C# START Button
       ↓
Holding Register D1
       ↓
Controller
       ↓
Sequence Start
```

PC에서 Auto와 Start를 선택하면 Controller에서 Register 값을 확인하여 작성된 자동 운전 시퀀스를 실행하도록 구성했습니다.

---

# 12. Vision 검사와 장비 제어 연계

Cognex의 검사 결과를 이후 장비 동작에서 활용하는 자동화 구조를 실습했습니다.

```text
제품 감지
   ↓
Camera Trigger
   ↓
Image Acquisition
   ↓
Vision Inspection
   ↓
OK / NG Result
   ↓
PC / Controller
   ↓
MPS 후속 동작
```

이를 통해 Cognex 카메라를 독립적인 검사 장비로 사용하는 것이 아니라 **자동화 시스템의 후속 동작을 결정하는 데이터 입력 장치로 활용하는 구조**를 이해했습니다.

---

# Troubleshooting

## 1. Arduino Input 신호 미입력

### 문제

외부 입력 신호를 연결했지만 Arduino Input LED가 점등되지 않았습니다.

### 확인 과정

프로그램부터 수정하지 않고 입력 신호가 전달되는 경로를 순서대로 확인했습니다.

```text
Power
  ↓
COM
  ↓
External Input
  ↓
Arduino Input
```

### 원인

산업용 Input 회로의 **COM(+24V) 연결이 누락**되어 있었습니다.

### 해결

COM 단자에 +24V를 연결하여 입력 회로를 완성했고 Arduino Input LED 및 입력 데이터가 정상적으로 들어오는 것을 확인했습니다.

### 배운 점

제어 프로그램의 문제처럼 보이더라도 먼저 **전원 → COM → 배선 → Controller** 순서로 물리적인 신호 경로를 확인해야 한다는 점을 경험했습니다.

---

## 2. Arduino Input은 정상이나 PC 화면에 표시되지 않는 문제

### 문제

Arduino에서는 외부 Input 신호가 정상적으로 확인됐지만 C# 프로그램의 Input Display에는 상태 변화가 나타나지 않았습니다.

### 확인 과정

```text
Physical Input      정상
      ↓
Arduino Input       정상
      ↓
Data Processing     확인
      ↓
Serial Data
      ↓
PC Display
```

물리적인 입력까지 정상인 것을 확인한 뒤 Arduino의 데이터 처리 부분을 점검했습니다.

### 원인

Digital Input 배열을 읽는 함수가 코드 조건에 의해 실행되지 않아 입력 데이터가 PC로 정상 전달되지 않고 있었습니다.

### 해결

해당 조건을 수정하여 Input 배열을 정상적으로 읽도록 변경했고 PC 제어 화면에서도 입력 상태가 표시되는 것을 확인했습니다.

### 배운 점

통신 문제 발생 시 하나의 영역만 반복해서 확인하는 것이 아니라

**Physical I/O → Controller → Data Processing → Communication → PC Display**

순서로 데이터 흐름을 나누어 확인하면 문제 발생 위치를 효율적으로 특정할 수 있다는 점을 경험했습니다.

---

# 전체 데이터 흐름

```text
External Input
      ↓
   Arduino
      ↕
 Serial Communication
      ↕
┌─────────────────────┐
│   C# Windows Forms  │
│     PC Control      │
└─────────┬───────────┘
          │
     ┌────┴─────────────┐
     │                  │
     ▼                  ▼
Cognex In-Sight      Modbus TCP
     │                  │
     ▼                  ▼
Vision Inspection      MPS
     │                  │
     ▼                  ▼
Inspection Result   Equipment Control
```

---

# 결과

C# Windows Forms 기반 PC 제어 프로그램을 중심으로 Arduino, Cognex In-Sight, MPS 자동화 장비를 Serial 및 Ethernet 통신으로 연결했습니다.

단순 개별 장비 사용에서 끝나지 않고

**Physical I/O → Controller → Communication → PC Control → Vision → Equipment**

으로 이어지는 데이터 흐름을 단계별로 실습했습니다.

### 핵심 경험

* C# Windows Forms
* PC Control UI
* Arduino Digital I/O
* Industrial 24V I/O
* Serial Communication
* Cognex In-Sight
* Cognex In-Sight SDK
* Camera Live View
* Software / External Trigger
* Vision Result Data
* Ethernet / TCP/IP
* Modbus TCP
* EasyModbus
* MPS Control
* I/O / Communication Troubleshooting

