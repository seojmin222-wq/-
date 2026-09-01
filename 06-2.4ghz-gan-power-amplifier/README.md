# 2.4GHz GaN RF Power Amplifier 설계 · PCB Layout 및 EM Simulation

### CGH40010F GaN HEMT 기반 RF PA 설계 + Impedance Matching + Harmonic Suppression + Momentum EM 검증

> **2.4GHz ISM 대역에서 동작하는 GaN RF Power Amplifier를 설계하고,**
> **S-Parameter 분석 · Impedance Matching · Harmonic Balance · PCB Layout · Momentum EM Simulation까지 수행한 RF 회로 설계 프로젝트**

- **기간**: 3월 ~ 5월
- **참여 형태**: 5인 팀 프로젝트
- **담당 역할**: PCB Layout 설계 · 소신호 Simulation · Momentum EM Simulation · 제안서 작성
- **주요 기술**: Keysight ADS, ADS Momentum, S-Parameter, Smith Chart, LC Matching, Harmonic Balance, Harmonic Trap, PCB Layout, GaN HEMT

---

## 작업 프로세스

```text
GaN HEMT 특성 분석
        ↓
Bias Point 설정
        ↓
S-Parameter / Stability 분석
        ↓
Input / Output Impedance Matching
        ↓
Large Signal Simulation
        ↓
Harmonic Trap 설계
        ↓
PCB Layout
        ↓
Momentum EM Simulation
        ↓
Schematic - Layout 성능 비교 및 검증
```

### Layout / EM Simulation Process

```text
Schematic Simulation
        ↓
실제 Component Size 반영
        ↓
CGH40010F Layout 직접 구성
        ↓
PCB Routing / GND Via 설계
        ↓
Substrate Parameter 설정
        ↓
Momentum EM Simulation
        ↓
Parasitic 영향 분석
        ↓
Layout 성능 검증
```

---

2.4GHz ISM 대역에서 동작하는 **고출력 · 고효율 GaN RF Power Amplifier를 설계하는 캡스톤 프로젝트**를 수행했습니다.

주 능동소자로 **CGH40010F GaN HEMT**를 사용했으며, Keysight ADS를 이용해 회로 설계부터 PCB Layout 및 Momentum EM Simulation까지 진행했습니다.

RF Power Amplifier는 단순히 높은 출력 전력만 확보하면 되는 것이 아니라

- Input / Output Impedance Matching
- 회로 Stability
- Gain
- Output Power
- Drain Efficiency
- PAE
- Harmonic Suppression

을 동시에 고려해야 했습니다.

특히 RF 회로는 낮은 주파수의 회로와 달리 PCB의

`배선 길이 → Pad 구조 → Via → Substrate → Parasitic L / C`

와 같은 물리적 요소가 회로 성능에 직접 영향을 미치기 때문에, Schematic 단계에서 얻은 결과가 실제 PCB에서도 유지되는지를 추가로 검증해야 했습니다.

이에 따라 회로도 수준의 Simulation에서 끝내지 않고 실제 Component 크기와 PCB 구조를 반영한 **Layout을 설계한 뒤 ADS Momentum을 이용해 EM Simulation까지 수행**했습니다.

프로젝트에서 저는 **PCB Layout 설계와 소신호 Simulation을 중심으로 담당**했으며, 제조사에서 제공되지 않은 CGH40010F의 Layout 구조를 직접 구성해 EM Simulation이 가능하도록 구현했습니다.

---

| 구분 | 기술 / 기능 | 활용 내용 |
|---|---|---|
| RF Design Tool | Keysight ADS | RF 회로 설계 및 Simulation |
| EM Simulation | ADS Momentum | PCB Layout 기반 전자기 해석 |
| Active Device | CGH40010F | GaN HEMT 기반 Power Amplifier |
| Small Signal | S-Parameter | S11 · S21 · S12 · S22 분석 |
| Stability | Stability Factor K | 발진 가능성 및 안정성 검증 |
| Matching | Smith Chart | Input / Output Impedance Matching |
| Matching Network | L / C | 50Ω Impedance Matching |
| Large Signal | Harmonic Balance | 출력 전력 및 비선형 특성 분석 |
| Harmonic | LC Trap Circuit | 2차 · 3차 Harmonic 억제 |
| Layout | PCB Layout | 실제 Component / Routing 구조 구현 |
| Ground | GND Via | RF Ground Path 구현 |
| Verification | EM Simulation | Schematic과 실제 Layout 결과 비교 |

---

### 3-1. CGH40010F GaN HEMT 동작점 및 Stability 분석

2.4GHz 대역에서 Power Amplifier를 안정적으로 동작시키기 위해 먼저 CGH40010F의 동작 조건과 S-Parameter를 분석했습니다.

Bias 조건은

- `VDS = 28V`
- `VGS = -2.7V`
- `IDS ≈ 211mA`

수준으로 설정했습니다.

RF Power Amplifier는 높은 Gain을 갖는 만큼 특정 Source / Load Impedance 조건에서 원하지 않는 발진이 발생할 수 있기 때문에 **Stability Factor K를 이용해 회로 안정성을 검증**했습니다.

2.4GHz에서

```text
K = 2.243
```

을 확인했으며 Input / Output Stability Index 역시 안정 조건을 만족해 목표 주파수에서 안정적으로 동작할 수 있음을 확인했습니다.

---

### 3-2. Smith Chart 기반 Input / Output Impedance Matching

트랜지스터와 RF System의 기준 Impedance인 `50Ω` 사이의 부정합으로 인해 발생하는 반사를 줄이기 위해 Input / Output Matching Network를 설계했습니다.

S-Parameter를 기반으로 트랜지스터의 Impedance 특성을 분석한 뒤 **Smith Chart를 이용해 LC Matching Network를 구성**했습니다.

```text
Transistor Impedance
        ↓
Smith Chart 분석
        ↓
L / C 값 선정
        ↓
50Ω 방향으로 Impedance 이동
        ↓
S11 / S22 Simulation
        ↓
Matching 상태 검증
```

최종적으로 2.3GHz ~ 2.5GHz 구간에서

```text
S11 < -10dB
```

수준의 Input Matching 특성을 확인했습니다.

또한 기준 주파수인 2.4GHz에서 약

```text
S21 ≈ 12.5dB
```

의 Gain을 확인했습니다.

---

### 3-3. Harmonic Balance 및 Harmonic Trap 설계

Power Amplifier는 큰 신호에서 비선형 동작하기 때문에 기본 주파수뿐만 아니라 2차 · 3차 Harmonic 성분이 함께 발생합니다.

이에 따라 ADS Harmonic Balance Simulation을 이용해

- Fundamental
- 2nd Harmonic
- 3rd Harmonic
- Output Power
- PAE
- Drain Efficiency

를 분석했습니다.

필터 적용 전에는 2차 Harmonic에서도 비교적 큰 출력 성분이 나타났기 때문에 **L과 C를 이용한 Harmonic Trap Circuit**을 추가했습니다.

```text
2.4GHz Fundamental
        ↓
Power Amplifier
        ↓
LC Harmonic Trap
        ↓
4.8GHz / 7.2GHz Harmonic 억제
        ↓
Output Spectrum 개선
```

Layout Simulation 기준 2.4GHz에서 가장 큰 출력 성분을 유지하면서, 2차 Harmonic인 `4.8GHz` 성분을 약 **4dBm 이하**로 낮췄습니다.

---

### 3-4. Output Power 및 Efficiency 분석

Input Power를 변화시키면서 Output Power와 Efficiency 변화를 분석했습니다.

Schematic Simulation에서는 Input Power 증가에 따라 Output Power가 증가하다 약 33 ~ 34dBm 이후 포화되는 특성을 확인했습니다.

```text
Input Power 증가
        ↓
Output Power 증가
        ↓
약 33dBm까지 선형적인 증가
        ↓
34dBm 이후 Saturation
        ↓
Gain Compression / Harmonic 증가
```

최종 Layout 기반 Simulation에서는 최대 약

```text
Pout = 42.3dBm
```

의 Output Power를 확인했습니다.

또한

```text
Drain Efficiency = 75.74%
PAE              = 62.2%
```

수준의 결과를 확보했습니다.

---

### 3-5. 실제 Component Size를 반영한 PCB Layout 설계

Schematic Simulation에서는 Component와 배선이 이상적인 회로 요소로 표현되지만 실제 PCB에서는 소자의 크기와 배선 구조 자체가 RF 특성에 영향을 줍니다.

따라서 실제 제작 환경을 고려해

- Capacitor Package
- Inductor Package
- CGH40010F Pad
- Signal Routing
- GND Via
- Substrate
- Metal Thickness

등을 반영한 Layout을 구성했습니다.

특히 Capacitor와 Inductor는 실제 RF Component Data Sheet에 명시된 크기를 기준으로 Layout에 반영해 회로도와 물리적인 PCB 구조 사이의 차이를 줄였습니다.

---

### 3-6. ADS Momentum 기반 EM Simulation

완성한 PCB Layout을 ADS Momentum으로 가져와 전자기 해석을 수행했습니다.

Momentum Simulation에서는 Schematic Simulation에서 직접적으로 고려되지 않는

- 배선의 기생 Inductance
- Pad 사이의 기생 Capacitance
- Via
- PCB Material
- Metal Thickness
- RF Routing

의 영향을 포함할 수 있기 때문에 실제 제작 환경에 더 가까운 특성을 확인할 수 있었습니다.

```text
Schematic
    ↓
Ideal Circuit Simulation
    ↓
PCB Layout
    ↓
Physical Structure 반영
    ↓
Momentum EM Simulation
    ↓
Schematic / Layout 결과 비교
```

이를 통해 단순 회로도 수준의 성능이 아니라 **실제 PCB 구조를 적용했을 때에도 목표 성능을 유지할 수 있는지를 검증**했습니다.

---

## Troubleshooting ⭐

### Trouble 01. CGH40010F에 Layout 정보가 없어 Momentum EM Simulation을 수행할 수 없는 문제

#### Problem

제조사에서 제공한 CGH40010F Model File에는 회로 Simulation에 필요한 정보는 존재했지만 **Momentum에서 사용할 수 있는 실제 Transistor Layout 정보가 포함되어 있지 않았습니다.**

따라서 Schematic 단계에서는 Simulation이 가능했지만 PCB Layout으로 변환해 EM Simulation을 진행하려고 할 때 Transistor의 물리적인 구조를 표현할 수 없는 문제가 발생했습니다.

#### Analysis

처음에는 제공된 Model을 그대로 Layout에 적용할 수 있을 것으로 생각했습니다.

하지만 Momentum은 회로의 전기적인 Model만을 해석하는 것이 아니라 실제

- Metal Pattern
- Pad
- Routing
- Via
- Substrate

구조를 기반으로 전자기장을 계산하기 때문에 Transistor의 물리적인 Layout 정보가 필요했습니다.

따라서 회로 Model과 Layout Model은 동일한 정보가 아니며, **EM Simulation을 위해 별도의 Physical Model을 구성해야 한다고 판단했습니다.**

#### Solution

CGH40010F의 제조사 권장 구조와 Model Parameter를 분석해

- Gate
- Drain
- Source
- Metal Pad

구조를 직접 Layout으로 구현했습니다.

```text
CGH40010F Model 확인
        ↓
Gate / Drain / Source 구조 분석
        ↓
Physical Pad Size 확인
        ↓
Metal Pattern 직접 구성
        ↓
Signal Routing 연결
        ↓
GND Via 구성
        ↓
Momentum Port 설정
        ↓
EM Simulation
```

또한 실제 PCB 환경과 유사하게

- Substrate Material
- Substrate Height
- Metal Layer
- Component Size
- GND Via

를 함께 설정했습니다.

#### Result

CGH40010F가 포함된 전체 PCB를 Momentum 환경에서 해석할 수 있게 되었으며, 회로도 수준에서 끝나지 않고 **실제 PCB 생산 환경을 고려한 EM Simulation까지 수행**할 수 있었습니다.

> **Learned**
> RF 설계에서는 Electrical Model뿐 아니라 **실제 구조를 표현하는 Physical Layout Model까지 직접 이해하고 구성할 수 있어야 한다는 점**을 배웠습니다.

---

### Trouble 02. Schematic과 PCB Layout Simulation의 Output Power 차이

#### Problem

Schematic 단계에서는 최대 Output Power가 약 `43.35dBm`까지 나타났지만 실제 PCB Layout을 적용한 Momentum 기반 분석에서는 약 `42.31dBm` 수준으로 감소했습니다.

```text
Schematic : 약 43.35dBm
Layout    : 약 42.31dBm
```

동일한 Transistor와 Matching Network를 사용했음에도 Simulation 단계에 따라 성능 차이가 발생했습니다.

#### Analysis

Schematic에서는 배선과 Component 연결이 이상적인 회로 요소로 표현되지만 PCB Layout에서는

```text
Routing
+
Pad
+
Via
+
Substrate
        ↓
Parasitic Inductance
+
Parasitic Capacitance
```

가 추가됩니다.

따라서 실제 Layout 구조가 Input / Output Matching Point와 RF 전달 특성을 변화시키면서 Output Power가 감소한다고 판단했습니다.

#### Solution

Momentum EM Simulation을 통해 Layout의 물리적인 영향을 포함하고 다음 요소들을 검토했습니다.

- 불필요한 Metal Routing 제거
- 불필요한 Via 최소화
- Component 간 배선 길이 검토
- Metal Thickness 설정
- Substrate Dielectric 특성 반영
- Component Physical Size 반영

회로도를 단순히 Layout으로 변환하는 것이 아니라 **RF Signal Path를 기준으로 실제 배선 구조를 검토**했습니다.

#### Result

Schematic 대비 일부 Output Power 감소는 발생했지만 실제 Layout 환경에서도

```text
Pout = 42.3dBm
DE   = 75.74%
PAE  = 62.2%
```

수준의 성능을 확보했습니다.

> **Learned**
> RF 회로에서는 Schematic 결과가 그대로 실제 PCB 성능으로 이어지지 않으며, **Layout에서 발생하는 Parasitic 성분까지 포함해 성능을 검증해야 한다는 점**을 체감했습니다.

---

### Trouble 03. Harmonic 성분으로 인해 출력 신호 왜곡이 발생하는 문제

#### Problem

Power Amplifier의 Large Signal Simulation을 진행하면서 기본 주파수인 2.4GHz 이외에도 2차 · 3차 Harmonic 성분이 발생하는 것을 확인했습니다.

특히 Filter 설계 전에는 2차 Harmonic에서 비교적 큰 Power가 나타나 출력 신호 왜곡 가능성이 있었습니다.

#### Analysis

Power Amplifier는 큰 입력 신호에서 Transistor가 비선형적으로 동작하기 때문에

```text
2.4GHz × 2 = 4.8GHz
2.4GHz × 3 = 7.2GHz
```

와 같은 Harmonic 성분이 발생한다고 분석했습니다.

단순히 Matching 성능만 개선하는 것으로는 Harmonic 문제를 해결할 수 없었기 때문에 Output Network에 별도의 Harmonic 억제 구조가 필요했습니다.

#### Solution

특정 Harmonic Frequency에서 낮은 Impedance Path를 형성하도록 **L과 C를 이용한 Harmonic Trap Circuit**을 추가했습니다.

Harmonic Balance Simulation을 반복하면서 Fundamental Power를 유지하면서 Harmonic 성분이 낮아지도록 L / C Parameter를 조정했습니다.

#### Result

Layout Simulation 기준 2차 Harmonic인

```text
4.8GHz ≤ 약 4dBm
```

수준까지 낮춰 2.4GHz 기본파 대비 Harmonic 성분을 억제했습니다.

> **Learned**
> Power Amplifier에서는 단순히 높은 Gain과 Output Power뿐 아니라 **비선형 동작으로 발생하는 Harmonic까지 함께 고려해야 실제 RF 출력 품질을 확보할 수 있다는 점**을 배웠습니다.

---

### Trouble 04. 회로 성능과 실제 PCB 구현 가능성을 동시에 고려해야 하는 문제

#### Problem

초기 설계에서는 Schematic Simulation 결과를 중심으로 Matching과 Efficiency를 조정했지만, Layout으로 변환하면서 실제 Component Size와 Routing 공간을 고려하면 회로도와 동일한 구조를 그대로 구현하기 어려운 부분이 발생했습니다.

RF Circuit의 Electrical Performance만 최적화하면 실제 PCB에서 배선이 불필요하게 길어지거나 Via가 증가할 수 있었습니다.

#### Analysis

RF 회로에서는

```text
Electrical Performance
        +
Physical Layout
        +
PCB Manufacturability
```

를 함께 고려해야 한다고 판단했습니다.

회로도에서 최적의 Component 값만 찾는 것과 실제 PCB상에서 해당 Component를 배치하고 짧은 RF Path를 만드는 것은 서로 다른 문제였습니다.

#### Solution

Layout 설계 시 단순히 Schematic 배치를 그대로 옮기지 않고 RF Signal Path를 기준으로 Component 위치를 조정했습니다.

또한

- Component Package Size
- Gate / Drain 연결 구조
- RF Signal Path
- Ground Path
- GND Via 위치

를 함께 검토하면서 실제 구현 가능한 PCB 구조로 설계했습니다.

#### Result

회로 Simulation뿐만 아니라 Layout 기반 EM Simulation까지 완료해 **Electrical Design과 Physical Design을 연결한 RF 설계 과정**을 경험했습니다.

> **Learned**
> 실제 제품을 고려한 회로 설계에서는 Simulation 결과뿐 아니라 **PCB에서 실제로 구현 가능한 구조인지까지 고려해야 설계가 완성된다는 점**을 배웠습니다.

---

## 결과 및 성과

- CGH40010F GaN HEMT 기반 **2.4GHz RF Power Amplifier 설계**
- `VDS = 28V`, `VGS = -2.7V` 기준 Bias Point 설정
- 2.4GHz에서 **Stability Factor K = 2.243** 확보
- Smith Chart와 LC Network를 이용한 **Input / Output Impedance Matching**
- `2.3GHz ~ 2.5GHz` 구간에서 **S11 < -10dB**
- 2.4GHz에서 약 **12.5dB Gain 확인**
- Harmonic Balance를 이용한 **Large Signal 특성 분석**
- LC Harmonic Trap 적용을 통한 **2차 Harmonic 억제**
- 실제 Component Size · Via · Substrate를 반영한 **PCB Layout 설계**
- 제조사에서 제공되지 않은 **CGH40010F Physical Layout 직접 구현**
- ADS Momentum을 활용한 **Layout 기반 EM Simulation 수행**
- 최종 Output Power **42.3dBm**
- Drain Efficiency **75.74%**
- PAE **62.2%**
- 비교 연구 대비 Output Power **+0.3dBm**
- Drain Efficiency **+11.74%p**
- PAE **+4.6%p**
- Schematic → Layout → EM Simulation으로 이어지는 **RF 설계 전 과정 경험**

---

## Final Performance

| Parameter | Result |
|---|---:|
| Operating Frequency | **2.4GHz** |
| Maximum Output Power | **42.3dBm** |
| Drain Efficiency | **75.74%** |
| PAE | **62.2%** |
| Gain @ 2.4GHz | **약 12.5dB** |
| Stability Factor K | **2.243** |
| Input Matching | **S11 < -10dB @ 2.3 ~ 2.5GHz** |
| 2nd Harmonic | **약 4dBm 이하 @ 4.8GHz** |

---

## 기존 연구와 성능 비교

| Parameter | Reference | 본 프로젝트 | Improvement |
|---|---:|---:|---:|
| Output Power | 42dBm | **42.3dBm** | **+0.3dBm** |
| Drain Efficiency | 64% | **75.74%** | **+11.74%p** |
| PAE | 57.6% | **62.2%** | **+4.6%p** |

---

이번 프로젝트를 통해 RF 회로에서는 **Schematic에서 좋은 Simulation 결과를 얻는 것만으로 설계가 끝나는 것이 아니라는 점**을 배웠습니다.

처음에는 회로도에서 Matching과 Output Power를 확보하면 Layout에서도 비슷한 결과가 나올 것으로 생각했지만, 실제 PCB 구조를 적용하면서 Routing · Pad · Via · Substrate에서 발생하는 Parasitic 성분으로 인해 RF 특성이 달라지는 것을 확인했습니다.

이를 해결하면서 단순히 ADS를 이용해 회로를 Simulation하는 것을 넘어

**Bias 설정 → S-Parameter 분석 → Stability 검증 → Impedance Matching → Harmonic Balance → PCB Layout → Momentum EM Simulation**

으로 이어지는 RF Power Amplifier의 전체 설계 흐름을 경험했습니다.

특히 CGH40010F의 Layout 정보가 제공되지 않는 상황에서 제조사 자료와 Model 정보를 기반으로 Gate · Drain · Source의 Physical Structure를 직접 구성하면서 **주어진 Model에 의존하지 않고 필요한 정보를 분석해 직접 구현하는 문제 해결 경험**을 얻었습니다.

또한 Schematic과 Layout Simulation 결과를 비교하면서 RF 회로에서 Component 값뿐만 아니라 **배선 길이 · Pad · Via · 기판 특성 등 물리적인 구조 자체가 하나의 회로 요소로 작용한다는 점**을 체감했습니다.

Harmonic 문제를 해결하는 과정에서는 단순히 높은 Output Power를 만드는 것이 아니라 Gain · Efficiency · Stability · Harmonic을 함께 고려해야 한다는 점을 배웠습니다.

최종적으로

**소자 특성 분석 → 회로 설계 → Simulation → Troubleshooting → PCB Layout → EM 검증**

까지 이어지는 전체 RF 설계 과정을 경험하며, 회로의 전기적 성능과 실제 구현 가능성을 함께 고려하는 설계 관점을 익혔습니다.
