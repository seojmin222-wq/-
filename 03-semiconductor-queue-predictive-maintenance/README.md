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
