## 시스템 구성
![시스템 구성도](프젝1이미지/01-architecture_2.png)

...

**대표 rung 캡처**

![PLC 래더로직 - 기동 인터록](프젝1이미지/PLC1.png)
> [비상정지 B접점과 안전센서 조건이 모두 충족될 때만 기동 코일이 여자되도록 구성한 인터록 로직입니다.]

![PLC 래더로직 - 입출고 시퀀스](프젝1이미지/PLC1-1.png)
> [위치 센서 신호에 따라 컨베이어 모터와 스토퍼 실린더를 순차 제어하는 로직입니다.]

**HMI 화면 구현**

<p align="center">
  <img src="프젝1이미지/HMI1.png" width="32%" />
  <img src="프젝1이미지/HMI2.png" width="32%" />
  <img src="프젝1이미지/HMI3.png" width="32%" />
</p>

...

## iFIX 기반 SCADA 창고관리 시스템

Kepserver에서 Modbus TCP, OPC, OPC UA 통신 드라이버를 각각 직접 설정했습니다.

<p align="center">
  <img src="프젝1이미지/KEP1.png" width="32%" />
  <img src="프젝1이미지/KEP2.png" width="32%" />
  <img src="프젝1이미지/KEP3.png" width="32%" />
</p>

...

![iFIX SCADA 화면](프젝1이미지/iFIX1.png)

<details>
<summary>iFIX 화면 더 보기</summary>

![iFIX SCADA 화면 2](프젝1이미지/iFIX2.png)
![iFIX SCADA 화면 3](프젝1이미지/iFIX3.png)
![iFIX SCADA 화면 4](프젝1이미지/iFIX4.png)

</details>
