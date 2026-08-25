# 산업용 제어·모니터링 시스템 2종 구축 (iFIX SCADA 창고관리 + Cognex 비전 품질검사 자동화)

## 개요
대한상공회의소 산업자동화 실습 과정에서 성격이 다른 두 가지 제어·모니터링 시스템을 구축한 프로젝트입니다.
1. PLC·HMI·SCADA(iFIX)를 통합한 스마트 창고관리 시스템 구축
2. Cognex 비전 카메라와 PLC·아두이노를 연동해 기준 미달 물품을 자동 배출하는 품질검사 자동화 시스템 구축

- **기간**: [YYYY.MM ~ YYYY.MM]
- **참여 형태**: [개인 / 팀 - 인원수]
- **담당 역할**: [본인 역할]

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
![시스템 구성도](프젝1이미지/01-architecture_2.png)

**SCADA 데이터 흐름**: PLC → Kepserver → PowerTool → Database

**비전검사 자동화 흐름**: Cognex 비전 카메라 + VS Code(C#)  → 아두이노 → PLC → MPS 자동 배출

## 주요 구현 내용

### 1) PLC 래더로직 설계 및 HMI 화면 구현
GXWorks2로 스마트 창고관리 시스템의 PLC 래더 로직을 설계하고, GTDesigner3로 현장 조작·모니터링용 HMI 화면을 구현했습니다. SCADA 통합에 앞서 필드 계층(PLC)에서 제어 로직과 인터록을 먼저 완성한 뒤, 이를 기반으로 HMI·iFIX와 연동하는 순서로 시스템을 구축했습니다.

**래더로직 설계**

[MPS 창고관리 설비의 입출고 컨베이어, 위치 감지 센서, 스토퍼 실린더, 랙 적재 상태 등]을 제어 대상으로 삼아 기능 단위로 로직을 구성했습니다.

***주요 로직***
| 기능 블록 | 주요 내용 |
|---|---|
| 기동/정지 인터록 | [비상정지·초기 안전 조건이 모두 충족될 때만 기동을 허용하는 로직] |
| 입출고 시퀀스 제어 | [센서 감지 순서에 따라 컨베이어·스토퍼 등을 순차 동작시키는 로직] |
| 위치/재고 판별 | [랙별 적재 유무를 센서로 판별해 저장 위치를 결정하는 로직] |


**기동/정지 인터록**
<p align="center">
  <img src="프젝1이미지/PLC1.png" width="48%" />
  <img src="프젝1이미지/PLC1-1.png" width="48%" />
</p>

**입출고 시퀀스 제어**
![입출고 시퀀스 제어](프젝1이미지/PLC2.png)

**위치/재고 판별**
<p align="center">
  <img src="프젝1이미지/PLC3.png" width="48%" />
  <img src="프젝1이미지/PLC4.png" width="48%" />
</p>
  
<details>
<summary>전체 래더로직 더 보기</summary>

발표 자료에서 정리했던 전체 로직은 PDF로 첨부해 두었습니다 → [PLC 래더로직 전체 PDF](docs/01-plc-ladder-full.pdf)

</details>

**HMI 화면 구현**

GTDesigner3로 MAIN · Servo Control · Equipment Control 3개 화면을 구성하고, 화면 상단 버튼으로 서로 전환할 수 있도록 했습니다.

| ![MAIN](프젝1이미지/HMI1.png) | ![Servo Control](프젝1이미지/HMI2.png) | ![Equipment Control](프젝1이미지/HMI3.png) |
|:---:|:---:|:---:|
| 서보/설비 제어 진입 및 현재 날짜·시각 표시 | 원점 복귀·JOG·에러 리셋 조작 및 상태(에러/경고/위치/속도) 모니터링 | 가공 설정·비상정지/해제·창고 적재 램프·작업 수량 카운트 |


### 2) iFIX 기반 SCADA 창고관리 시스템
PLC·HMI 구현을 마친 뒤, Kepserver에서 Modbus TCP, OPC, OPC UA 통신 드라이버를 각각 직접 설정해 3종 프로토콜 통신을 구현했고, 이 중 미쯔비시 PLC는 실제 장비로 태그값을 가져와 확인했습니다. MQTT·MySQL 기반 데이터 처리도 학습해 적용했습니다.

**Kepserver 통신 드라이버 및 태그 설정**

KEPServerEX에서 미쯔비시 PLC 디바이스 영역별로 OPC 그룹을 구성하고 PLC 메모리를 OPC 태그로 노출시킨 뒤, iFIX I/O 드라이버에서 해당 OPC 항목을 참조하는 중간 태그를 구성해 PLC ↔ Kepserver ↔ iFIX 통신 경로를 연결했습니다. 이후 iFIX DB Manager에서 실제 SCADA 화면에 쓰일 태그에 한글 설명을 붙여 최종 태그 데이터베이스를 완성했습니다.

<details>
<summary>태그 설정 화면 더 보기</summary>

**KEPServerEX OPC 그룹/아이템 구성**
![KEPServerEX PowerTool](프젝1이미지/KEP1.png)

**iFIX I/O 드라이버 태그 (KEPServer OPC 항목 매핑)**
![iFIX I/O 드라이버 태그](프젝1이미지/KEP3.png)

**iFIX DB Manager 최종 태그 목록**
![iFIX DB Manager](프젝1이미지/KEP3.png)

</details>

iFIX에 SCADA 화면을 구성해 DO 태그로 MPS를 원격 구동하려 했으나, 태그 설정이 정확함에도 실제 구동이 되지 않는 문제가 발생했습니다. PLC → Kepserver → PowerTool → Database로 이어지는 데이터 흐름을 역으로 추적한 결과, Database에는 DO 값이 Write되지만 PLC까지 신호가 전달되지 않았고, PowerTool 단계에서 Write 칸에는 값이 들어가지만 Read 값이 비어있는 것을 확인했습니다. DO 태그만으로는 PowerTool의 Read가 이루어지지 않아 이후 Kepserver·PLC로 신호가 전달되지 않는다는 원인을 파악하고, Database에 DO와 짝을 이루는 DI 태그를 추가 생성해 Write→Read 사이클을 완성시킴으로써 문제를 해결했습니다. 이후 HDA 기능으로 원하는 기간의 태그 이력을 조회할 수 있도록 구성했습니다.

### iFIX 화면 구성
iFIX SCADA도 HMI와 유사하게 MAIN·Equipment Control·Servo Control 제어 화면과 HDA 기반 이력 조회(Trend) 화면으로 구성했습니다. 상단 공통 헤더에 화면 전환 버튼과 현재 날짜·시각, Auto/Manual 모드 전환 스위치를 배치했습니다.

| ![MAIN](프젝1이미지/iFIX1.png) | ![Equipment Control](프젝1이미지/iFIX2.png) |
|:---:|:---:|
| 공정 모식도 + 축별 Jog 수동 제어 + Servo Position 실시간 표시 | 창고 적재 램프(층별/좌우) + 상태 램프 + 작업 수량(실시간 태그값) 모니터링 |
| ![Servo Control](프젝1이미지/iFIX3.png) | ![Trend](프젝1이미지/iFIX4.png) |
| JOG·원점 복귀·에러 리셋 조작 및 위치/속도/에러 상태 표시 | HDA로 태그 이력을 시계열 그래프로 조회 |


### 3) Cognex 비전 기반 품질검사 자동화
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
