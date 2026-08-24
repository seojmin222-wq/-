# 산업용 제어·모니터링 시스템 2종 구축 (iFIX SCADA 창고관리 + Cognex 비전 품질검사 자동화)

## 개요
대한상공회의소 산업자동화 실습 과정에서 성격이 다른 두 가지 제어·모니터링 시스템을 구축한 프로젝트입니다.
1. PLC·HMI·SCADA(iFIX)를 통합한 스마트 창고관리 시스템 구축
2. Cognex 비전 카메라와 PLC·아두이노를 연동해 기준 미달 물품을 자동 배출하는 품질검사 자동화 시스템 구축

- **기간**: [2026.03 - 2026.04 / 2026.07 - 2026.08]
- **참여 형태**: [개인]


## 배경 및 문제 정의
PLC 로직만으로는 현장 데이터를 실시간으로 모니터링하고 이력을 관리하는 데 한계가 있었고, 품질검사 공정은 육안 검사에 의존하고 있어 자동화가 필요했습니다. 이 두 가지 문제를 각각 SCADA 통합과 비전 기반 자동화로 해결하는 것을 목표로 프로젝트를 진행했습니다.

## 사용 기술 / 툴
- PLC: GXWorks2 (래더 로직 설계), 미쯔비시 PLC
- HMI: GTDesigner3
- SCADA: iFIX
- 통신: Modbus TCP, OPC, OPC UA, MQTT, HDA (KEPServerEX)
- DB: MySQL
- 비전 검사: Cognex Vision Camera
- 개발 환경: VS Code (C#)
- 하드웨어 연동: Arduino, MPS 설비

## 시스템 구성

![시스템 구성도](images/01-architecture.png)

**SCADA 데이터 흐름**: PLC → Kepserver → PowerTool → Database

**비전검사 자동화 흐름**: Cognex 비전 카메라 → VS Code(C#) 판정 로직 → 아두이노/PLC → MPS 자동 배출

## 주요 구현 내용

### 1) iFIX 기반 SCADA 창고관리 시스템
GXWorks2로 PLC 래더 로직을 설계하고, GTDesigner3로 HMI 화면을 구현했습니다. Kepserver에서 Modbus TCP, OPC, OPC UA 통신 드라이버를 각각 직접 설정해 3종 프로토콜 통신을 구현했고, 이 중 미쯔비시 PLC는 실제 장비로 태그값을 가져와 확인했습니다. MQTT·MySQL 기반 데이터 처리도 학습해 적용했습니다.

iFIX에 SCADA 화면을 구성해 DO 태그로 MPS를 원격 구동하려 했으나, 태그 설정이 정확함에도 실제 구동이 되지 않는 문제가 발생했습니다. PLC → Kepserver → PowerTool → Database로 이어지는 데이터 흐름을 역으로 추적한 결과, Database에는 DO 값이 Write되지만 PLC까지 신호가 전달되지 않았고, PowerTool 단계에서 Write 칸에는 값이 들어가지만 Read 값이 비어있는 것을 확인했습니다. DO 태그만으로는 PowerTool의 Read가 이루어지지 않아 이후 Kepserver·PLC로 신호가 전달되지 않는다는 원인을 파악하고, Database에 DO와 짝을 이루는 DI 태그를 추가 생성해 Write→Read 사이클을 완성시킴으로써 문제를 해결했습니다. 이후 HDA 기능으로 원하는 기간의 태그 이력을 조회할 수 있도록 구성했습니다.

![iFIX SCADA 화면](images/01-ifix-scada.png)

### 2) Cognex 비전 기반 품질검사 자동화
PC제어 수업에서 Cognex 비전 카메라를 VS Code(C#) 환경과 연동해 물품을 실시간으로 촬영·판정하는 로직을 구현했습니다. 판정 결과를 아두이노·PLC와 연동해 기준 미달 물품이 감지되면 자동으로 배출되도록 로직을 구성했습니다. 아두이노, PLC, VS Code, Cognex 카메라를 소프트웨어·하드웨어 전 영역에서 연동해 C#으로 MPS 설비를 직접 구동, 비전 판정부터 물리적 배출까지 이어지는 하나의 자동화 루프를 완성했습니다.

![Cognex 비전 검사 화면](images/01-vision-inspection.png)

## 결과 및 성과
- Kepserver에서 Modbus TCP·OPC·OPC UA 통신 드라이버 3종을 직접 구현하고, 미쯔비시 PLC 실제 태그값 연동을 확인
- PLC-HMI-SCADA 전 계층 통합 시스템 완성, iFIX 화면에서 MPS 원격 구동 및 기간별 이력 조회 정상 동작 확인
- 아두이노·PLC·VS Code·Cognex 카메라 4개 장비를 SW·HW 전 영역에서 통합 연동
- 기준 미달 물품 실시간 판정 및 자동 배출 로직 정상 동작 확인

## 회고 / 배운 점
데이터 모니터링 중심의 SCADA 시스템과 실시간 비전 기반 품질검사 시스템이라는 서로 다른 성격의 산업자동화 시스템을 모두 구축하며, 필드-제어-모니터링 계층을 통합하는 역량과 래더 로직, SCADA 태그, C# 등 이기종 장비·언어를 하나의 제어 루프로 엮는 시스템 통합 역량을 확보했습니다.

특히 MPS 원격 구동이 되지 않는 문제를 해결하는 과정에서, PLC(필드 계층) → Kepserver(통신 드라이버 계층) → PowerTool(미들웨어 계층) → Database(데이터 계층)로 이어지는 전체 데이터 흐름을 직접 역추적해야 했습니다. 이 과정에서 각 계층이 서로 다른 프로그램과 방식으로 데이터를 주고받는다는 점, 그리고 그중 한 계층에서라도 Read/Write 사이클이 끊기면 겉으로는 설정이 맞아 보여도 전체 시스템이 동작하지 않는다는 점을 직접 체감했습니다. 이 경험을 통해 SCADA 시스템을 단순히 "태그 하나"의 문제가 아니라, 여러 프로그램과 계층을 관통하는 하나의 데이터 파이프라인으로 바라보는 시각을 갖게 됐습니다.
