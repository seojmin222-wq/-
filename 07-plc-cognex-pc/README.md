# C# · Arduino · Cognex 기반 PC 제어 및 비전 자동화 시스템

### Windows Forms · Cognex In-Sight · Serial · Modbus TCP · MPS 장비 연동

> **C# 기반 PC 제어 프로그램에 Arduino I/O와 Cognex 비전 검사를 통합하고,
> 검사 결과를 MPS 설비의 후속 동작까지 연결한 자동화 제어 시스템**

* **기간**: 2026.07 - 08
* **주요 기술**: C#, Windows Forms, Arduino, Cognex In-Sight, Serial, Ethernet, Modbus TCP, EasyModbus, MPS

---

## 시스템 구성

<p align="center">
  <img src="./프젝7이미지/PC제어1.png" width="90%" alt="PC 제어 및 비전 자동화 시스템 구성도">
</p>

### 제어 흐름

```text
C# Windows Forms
        │
   ┌────┼───────────────┐
   │    │               │
 Serial Ethernet    Modbus TCP
   │    │               │
Arduino Cognex          MPS
   │   In-Sight          │
   │      │              │
I/O   Vision Result   Equipment
                      Control
```

PC 제어 프로그램을 중심으로 **Arduino I/O 제어, Cognex 비전검사, MPS 장비 제어를 하나의 시스템으로 연동**했습니다.

---

<details>
<summary><b>01. 프로젝트 개요 및 목표</b></summary>

<br>

C# Windows Forms 기반의 PC 제어 프로그램을 제작하고 **Arduino, Cognex In-Sight 비전 카메라, MPS 자동화 장비를 Serial 및 Ethernet 통신으로 연동**했습니다.

PC 프로그램에서 장비의 Digital I/O와 운전 상태를 확인하고 직접 제어할 수 있도록 구성했으며, Cognex SDK를 적용하여 **카메라 영상과 검사 결과를 하나의 PC 제어 화면에서 실시간으로 확인**할 수 있도록 구현했습니다.

Cognex In-Sight에서는 **PatMax 패턴 매칭, Blob 검출, QR Code 판독**을 활용하여 제품을 검사했으며, 검사 결과를 Arduino의 입력 신호와 연계해 후속 장비의 자동 시퀀스에 활용했습니다.

이를 통해

**I/O 제어 → 비전 검사 → 결과 판정 → 장비 동작**

으로 이어지는 산업 자동화 시스템의 전체적인 제어 및 데이터 흐름을 구현했습니다.

</details>

---

<details>
<summary><b>02. 사용 기술 / 툴</b></summary>

<br>

| 구분          | 기술 / 툴                              |
| ----------- | ----------------------------------- |
| PC Control  | C# / Windows Forms                  |
| MCU         | Arduino                             |
| Vision      | Cognex In-Sight / In-Sight Explorer |
| Vision SDK  | Cognex In-Sight SDK                 |
| Vision Tool | PatMax / Blob / QR Code             |
| Serial      | SerialPort                          |
| Network     | Ethernet / TCP/IP                   |
| Protocol    | Modbus TCP                          |
| Library     | EasyModbus                          |
| I/O         | Digital Input / Output, DC 24V      |
| Equipment   | MPS Automation System               |

</details>

---

<details>
<summary><b>03. 주요 구현 내용</b></summary>

<br>

<details>
<summary><b>3-1. C# 기반 PC 제어 및 Arduino I/O 연동</b></summary>

<br>

Windows Forms를 이용하여 **장비의 상태를 확인하고 직접 제어할 수 있는 PC 제어 화면**을 구성했습니다.

Arduino와는 Serial 통신으로 연결하여 외부 Digital Input 상태를 PC에서 확인하고, PC에서 전달한 명령에 따라 Lamp, Buzzer, Conveyor 등의 출력을 제어했습니다.

### 주요 구현 기능

* COM Port 선택 및 Connect / Disconnect
* Digital Input 상태 모니터링
* Digital Output ON / OFF 제어
* Start / Stop / Auto 운전
* 연결 상태에 따른 Button 상태 표시
* DC 24V 산업용 I/O 처리
* Lamp / Buzzer / Conveyor 제어

```text
Operator
   ↓
C# Windows Forms
   ↕
Serial Communication
   ↕
Arduino
   ↓
Digital I/O
   ↓
Lamp / Buzzer / Conveyor
```

실제 장비의 입출력 Address를 I/O Table로 정리한 뒤 Arduino와 산업용 I/O Board를 배선하여 **소프트웨어의 I/O 정보와 실제 장비 신호를 일치시켜 제어**했습니다.

<br>

<table>
  <tr>
    <th width="50%">I/O Mapping Table</th>
    <th width="50%">Arduino · I/O Board 실제 배선</th>
  </tr>
  <tr>
    <td align="center">
      <img src="./프젝7이미지/PC제어3.png" width="95%">
    </td>
    <td align="center">
      <img src="./프젝7이미지/PC제어2.png" width="95%">
    </td>
  </tr>
  <tr>
    <td align="center">
      Arduino·MPS 입출력 신호 및 Address Mapping
    </td>
    <td align="center">
      Arduino와 산업용 I/O Board 실제 배선 및 신호 연동
    </td>
  </tr>
</table>

<br>

</details>

<br>

<details>
<summary><b>3-2. Cognex 비전검사 및 C# 프로그램 통합</b></summary>

<br>

Cognex In-Sight 카메라를 Ethernet으로 연결하고 In-Sight Explorer에서 제품 검사 환경을 구성했습니다.

제품의 위치, 형상 및 정보를 판별하기 위해 다음 Vision Tool을 활용했습니다.

| 기능          | 적용 내용                   |
| ----------- | ----------------------- |
| **PatMax**  | 패턴 매칭을 통한 대상 위치 및 형상 확인 |
| **Blob**    | 객체 및 영역 검출              |
| **QR Code** | 제품 정보 판독                |

```text
Inspection Target
        ↓
Cognex In-Sight
        ↓
PatMax / Blob / QR Code
        ↓
Inspection Result
        ↓
C# PC Control
```

Cognex SDK를 Windows Forms 프로젝트에 적용하여 별도의 In-Sight Explorer 화면에서만 결과를 확인하는 것이 아니라 **PC 제어 프로그램 안에서 카메라 영상과 검사 결과를 직접 확인**할 수 있도록 구성했습니다.

사용한 주요 Cognex Library는 다음과 같습니다.

```csharp
using Cognex.InSight;
using Cognex.InSight.Cell;
using Cognex.InSight.Controls;
using Cognex.InSight.Extensions;
using Cognex.InSight.Graphic;
using Cognex.InSight.Sensor;
```

PC 제어 화면에서는 다음 기능을 구현했습니다.

* Cognex Camera 연결
* Camera Live View
* Software Trigger
* 외부 입력 Trigger
* PatMax 좌표값 표시
* Blob 검출 결과 표시
* QR Code 판독값 표시
* Vision 검사 결과 모니터링

Cognex 카메라와 Arduino의 I/O를 실제로 연결하고, PC 프로그램에서는 **카메라 화면·Trigger·검사 결과값을 하나의 화면에서 확인할 수 있도록 통합**했습니다.

<br>

<table>
  <tr>
    <th width="50%">Cognex · Arduino 하드웨어 연동</th>
    <th width="50%">C# 기반 Cognex 비전 제어 화면</th>
  </tr>
  <tr>
    <td align="center">
      <img src="./프젝7이미지/PC제어4.png" width="95%">
    </td>
    <td align="center">
      <img src="./프젝7이미지/PC제어5.png" width="95%">
    </td>
  </tr>
  <tr>
    <td align="center">
      Cognex Vision Camera와 Arduino I/O 실제 배선 및 신호 연동
    </td>
    <td align="center">
      Camera View · Trigger · PatMax · Blob · QR Code 검사값 실시간 표시
    </td>
  </tr>
</table>

<br>

</details>

<br>

<details>
<summary><b>3-3. 비전 판정 결과 기반 MPS 장비 제어</b></summary>

<br>

Cognex에서 생성된 검사 결과가 단순 모니터링 데이터로 끝나지 않고 **실제 MPS 장비의 후속 동작에 활용되도록 자동 시퀀스를 구성**했습니다.

Cognex의 Output 신호를 Arduino의 Digital Input으로 받아 제품 판정 조건에 활용하고, 결과에 따라 Conveyor와 Cylinder가 순차적으로 동작하도록 제어했습니다.

```text
제품 도착
   ↓
Camera Trigger
   ↓
Cognex Vision Inspection
   ↓
PatMax / Blob / QR Code
   ↓
Vision Output
   ↓
Arduino 판정
   ↓
MPS 자동 시퀀스
```

Arduino 자동 시퀀스에서는

* Conveyor를 이용한 제품 이송
* 카메라 도착 위치에서 Conveyor 정지
* Cognex Camera Trigger 출력
* Vision Result 입력 대기
* 판정 결과 확인
* 결과에 따른 후속 Cylinder 및 Conveyor 동작

순으로 설비를 제어했습니다.

또한 C# 프로그램과 MPS 장비 간에는 **Modbus TCP**를 적용하여 장비 데이터를 읽고 제어할 수 있도록 구성했습니다.

```csharp
ModbusClient modbus = new ModbusClient();
```

### Modbus TCP 주요 기능

* IP / Port 설정
* Connect / Disconnect
* Discrete Input Read
* Coil Write
* Holding Register Read / Write
* MPS I/O 상태 모니터링
* Auto / Start / Stop 명령

```text
C# PC Control
      ↕
 Modbus TCP
      ↕
     MPS
      ↓
Equipment Control
```

이를 통해 **PC 제어 → 비전검사 → 판정 → MPS 후속 동작**으로 이어지는 자동화 제어 흐름을 구성했습니다.

<br>

<table>
  <tr>
    <th width="50%">Cognex 판정 기반 Arduino 시퀀스 제어</th>
    <th width="50%">MPS 장비 실제 동작</th>
  </tr>
  <tr>
    <td align="center">
      <img src="./프젝7이미지/PC제어6.png" width="95%">
    </td>
    <td align="center">
      <img src="./프젝7이미지/PC제어7.png" width="95%">
    </td>
  </tr>
  <tr>
    <td align="center">
      Cognex Output 신호를 이용한 검사 결과 판정 및 MPS 자동 시퀀스 제어
    </td>
    <td align="center">
      비전 판정 결과에 따라 Conveyor · Cylinder가 동작하는 실제 MPS 자동화 과정
    </td>
  </tr>
</table>

<br>

</details>

</details>

---

<details>
<summary><b>04. Troubleshooting</b></summary>

<br>

<details>
<summary><b>Trouble 01. Arduino Input 신호 미입력</b></summary>

<br>

### Problem

외부 입력 장치를 연결했지만 Arduino의 Input LED가 점등되지 않았고 입력 신호도 인식되지 않았습니다.

### Analysis

프로그램부터 수정하기보다 실제 입력 신호가 전달되는 경로를 순서대로 확인했습니다.

```text
Power
  ↓
COM
  ↓
Input Signal
  ↓
Arduino
```

배선을 점검한 결과 산업용 Input 회로의 **COM(+24V) 연결이 누락**되어 있었습니다.

### Solution

COM 단자에 +24V를 연결하여 입력 회로를 완성했습니다.

### Result

Arduino Input LED가 정상적으로 점등되고 외부 입력 신호도 정상적으로 읽을 수 있었습니다.

> 프로그램 문제처럼 보이는 경우에도 먼저
> **전원 → COM → 배선 → Controller → Software**
> 순서로 실제 신호 경로를 확인하는 것이 중요하다는 점을 경험했습니다.

</details>

<br>

<details>
<summary><b>Trouble 02. Arduino에서는 입력되지만 PC 화면에 표시되지 않는 문제</b></summary>

<br>

### Problem

Arduino에서는 외부 Input 신호가 정상적으로 확인됐지만 C# PC 제어 화면에서는 Input 상태가 변경되지 않았습니다.

### Analysis

입력부터 PC 화면까지의 데이터 흐름을 단계별로 나누어 확인했습니다.

```text
Physical Input      정상
      ↓
Arduino Input       정상
      ↓
Data Processing     문제 확인
      ↓
Serial Communication
      ↓
PC Display
```

Physical I/O와 Arduino Input이 정상인 것을 확인한 뒤 데이터 처리 코드를 점검했습니다.

Digital Input 배열을 읽는 함수가 코드 조건으로 인해 실행되지 않아 입력 데이터가 PC로 전달되지 않는 것을 확인했습니다.

### Solution

해당 조건을 수정하여 Digital Input 배열을 정상적으로 읽고 PC로 전달하도록 변경했습니다.

### Result

C# PC 제어 화면에서도 외부 Input 상태가 정상적으로 표시되는 것을 확인했습니다.

> 통신 문제를 하나의 영역으로 판단하지 않고
> **Physical I/O → Controller → Data Processing → Communication → PC Display**
> 순으로 구간을 나누어 확인함으로써 문제 발생 위치를 특정했습니다.

</details>

</details>

---

<details>
<summary><b>05. 결과 및 성과</b></summary>

<br>

C# Windows Forms를 중심으로 Arduino, Cognex In-Sight, MPS 장비를 연결하여 **PC 제어·비전검사·장비 동작을 하나의 자동화 시스템으로 구성**했습니다.

### 주요 결과

* C# Windows Forms 기반 **PC 장비 제어 화면 구현**
* Arduino Serial 통신 기반 **Digital I/O 제어**
* 산업용 DC 24V I/O 배선 및 Address Mapping
* Cognex In-Sight와 C# 프로그램 **SDK 연동**
* PC 프로그램 내 **Camera Live View 통합**
* **PatMax · Blob · QR Code** 기반 제품 검사
* Software / External Trigger 구현
* Cognex Vision Result를 Arduino 입력 신호로 활용
* 검사 결과 기반 **MPS 자동 시퀀스 제어**
* EasyModbus 기반 **Modbus TCP 통신**
* MPS Digital I/O 및 Holding Register 제어
* 실제 배선과 소프트웨어 데이터 흐름을 기반으로 한 Troubleshooting

최종적으로

**Physical I/O → PC Control → Vision Inspection → Decision → Equipment Control**

로 이어지는 전체 자동화 시스템의 데이터 및 제어 흐름을 경험했습니다.

</details>

---

<details>
<summary><b>06. 배운 점</b></summary>

<br>

이번 프로젝트를 통해 C#, Arduino, Cognex를 각각 독립적으로 사용하는 것을 넘어 **서로 다른 장비와 통신 방식을 하나의 자동화 시스템으로 통합하는 과정**을 경험했습니다.

특히 Cognex 검사 결과를 단순히 PC 화면에 표시하는 데 그치지 않고 Arduino의 제어 입력으로 활용하여 실제 MPS의 Conveyor와 Cylinder 동작까지 연결하면서 **머신비전의 판정 결과가 실제 장비 제어 신호로 활용되는 구조**를 이해했습니다.

또한 I/O 및 통신 문제를 해결하면서 소프트웨어만 확인하는 것이 아니라

**Physical I/O → Controller → Communication → PC → Vision → Equipment**

전체 신호 흐름을 기준으로 문제를 단계적으로 분리하고 원인을 찾는 방법을 익혔습니다.

</details>
