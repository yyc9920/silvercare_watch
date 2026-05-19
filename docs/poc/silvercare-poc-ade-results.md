# SilverCare Watch · PoC AI 위임 작업 수행 결과 (A · D · E)

> **조사 기준일**: 2026-05-18
> **대상**: `silvercare-poc-ai-tasks-ade.md` — AI 위임 가능 작업 A(부품 조사) · D(회로 참고) · E(조달·구매) 실행 결과
> **방법**: 제조사 공식 데이터시트·앱노트·유통사 페이지 웹 조사. 출처 URL 명시.
> **원칙**: 미확인 수치는 추정하지 않고 "검토 필요"로 표기. 모든 채택 판단은 개발자 영역.

---

## 0. 핵심 요약 (Executive Summary)

### 완료 상태

| 항목 | 산출물 | 상태 |
|------|--------|------|
| A | 부품 5종 레지스터/파라미터 요약표 | ✅ 완료 (미확인 10건 플래그) |
| D | 레퍼런스 회로 4건 + 확인 포인트 | ✅ 완료 (1.5" e-ink 규격 부재 확정) |
| E | BOM + 조달 경로 + 예산 범위 | ✅ 완료 (BG95-M2 단가 미확정) |

### 즉시 의사결정 필요 (개발자 영역)

| # | 이슈 | 영향 | 권장 |
|---|------|------|------|
| 1 | **1.5" 240×240 e-ink 상용 부재** | UI 설계·BOM 전체 | **1.54" 200×200 (SSD1681)** 로 확정. UI 처음부터 200×200 설계 |
| 2 | **LSM6DSOTR 재고 0 / 24주 리드타임** | 일정 치명 | 대체품 LSM6DSOXTR / LSM6DS3TR-C 재고 확인 후 결정 |
| 3 | **BG95-M2 단가 미확정** ($13~37 범위) | 예산 ±$70 | Quectel 영업 견적 또는 Mouser 직접 조회 |
| 4 | **380mAh LiPo 정확 셀 미유통** | 소형화 검증 | PoC는 Adafruit 400mAh 대체, 양산 시 EEMB/PKCELL 직발주 |
| 5 | **조달 경로 선택** | 예산 $115~900 | **브레이크아웃 + PPG만 EVSYS** 권장 ≈ $270~290 (근거 E-1-C) |
| 6 | **MAX86141 브레이크아웃 부재** | 알고리즘 검증 가능성 | MAX86141 단독 브레이크아웃 시중 無. 알고리즘 검증 필수면 PPG는 MAX86140EVSYS 필수 (브레이크아웃 PPG = MAX30101, 광학·알고리즘 비호환) |

### 예산 범위

| 시나리오 | USD | 비고 |
|---------|-----|------|
| 브레이크아웃 경로 (PPG=MAX30101) | $115~130 | 최저가. MAX86141 알고리즘 검증 불가 |
| **브레이크아웃 + PPG만 EVSYS (권장)** | **$270~290** | DK + EVSYS(PPG) + 브레이크아웃 IMU/셀룰러/e-ink. 알고리즘 패리티 유지 |
| 커스텀 부품 경로 (qty 3~5, 베어 IC) | $250~350 | BG95-M2 확정가 포함. 납땜·HW디버깅 공수 큼 |
| 평가보드 경로 (3보드 2세트) | $750~900 | 빠른 검증, 고비용 |

> 관세·부가세·국내 배송비 미포함. A-5 전력 수명 13일은 **LTE-M TX 전류 미포함 이론치** — Phase 4 실측 필수.

---

# A. 부품 조사 및 데이터시트 수집

> 핵심 부품 데이터시트·앱노트에서 PoC 필요 파라미터 추출. 미확인 수치 명시.

## A-1. MAX86141 (Analog Devices — PPG AFE) 레지스터 요약

19-bit ADC, 128-word FIFO, SPI. VDD_ANA 1.8V / VLED 3.1~5.5V.

### LED 전류 설정

| 주소 | 레지스터 | 비트 | 기능 |
|------|----------|------|------|
| 0x23~0x29 | LED1~6_DRV / PILOT_PA | [7:0] | 펄스 진폭 8-bit, RGE 범위에 따라 스케일 |
| 0x2A | LED1_RGE / LED2_RGE | [1:0]/[3:2] | 00=31mA, 01=62mA, 10=93mA, 11=124mA |

전류 계산: `I_LED = (LEDx_PA / 255) × LED_RGE_MAX`

### 샘플레이트 / 평균

| 주소 | 레지스터 | 비트 | 값 |
|------|----------|------|-----|
| 0x11 | PPG_CONFIG_1 | [1:0] PPG_TINT | 00=14.8µs · 01=29.4µs · 10=58.7µs · 11=117.3µs |
| 0x11 | PPG_CONFIG_1 | [3:2] PPG1_ADC_RGE | 00=4.0µA · 01=8.0µA · 10=16µA · 11=32µA |
| 0x12 | PPG_CONFIG_2 | [2:0] SMP_AVE | 1/2/4/8/16/32/64/128 |
| 0x12 | PPG_CONFIG_2 | [7:3] PPG_SR | 샘플레이트 (0x00=25, 0x03=100, 0x05=400 SPS …) |
| 0x13 | PPG_CONFIG_3 | [7:6] LED_SETLNG | 0/4/8/12µs |

### FIFO

| 주소 | 레지스터 | 기능 |
|------|----------|------|
| 0x04/0x05 | FIFO_WR/RD_PTR | 쓰기/읽기 포인터 |
| 0x07 | FIFO_DATA_COUNT | 저장 샘플 수 |
| 0x08 | FIFO_DATA | 버스트 읽기 |
| 0x09 | FIFO_CONFIG_1 [6:0] FIFO_A_FULL | 인터럽트 임계값 |
| 0x0A | FIFO_CONFIG_2 [1] FIFO_RO | 롤오버 허용 |

### 모드 / 시퀀스

| 주소 | 레지스터 | 기능 |
|------|----------|------|
| 0x20~0x22 | LED_SEQ_1~3 | 슬롯별 LED 소스 (0x1=Green/HR, 0x2=Red, 0x3=IR/SpO2, 0x9=Ambient) |
| 0x0D | SYSTEM_CTRL [0]RESET [1]SHDN | 소프트 리셋 / 셧다운(0.6µA) |

**소비전류**: 100 SPS ~32µA(AFE, LED 별도) · 셧다운 0.6µA

**출처**: [ADI DS](https://www.analog.com/media/en/technical-documentation/data-sheets/max86140-max86141.pdf) · [Arduino lib regmap](https://github.com/moothyknight/MAX86141_Arduino) · [Nordic DevZone](https://devzone.nordicsemi.com/f/nordic-q-a/109399/max86141-spi-communication-with-nrf5340)

## A-2. LSM6DSO (ST — 6축 IMU) 낙상/무활동 파라미터

Always-On 가속도+자이로, 내장 낙상·웨이크업 엔진. 1.71~3.6V.

| 주소 | 레지스터 | 비트 | 값 |
|------|----------|------|-----|
| 0x10 | CTRL1_XL | [7:4] ODR_XL | 0001=12.5Hz … 0110=416Hz(낙상 권장) … 1010=6.66kHz |
| 0x10 | CTRL1_XL | [3:2] FS_XL | 00=±2g · 10=±4g · 11=±8g · 01=±16g |
| 0x5D | FREE_FALL | [2:0] FF_THS | 000=156 · 001=219 · 010=250 · 011=312mg · 100=344 · 101=406 · 110=469 · **111=미확인** |
| 0x5D | FREE_FALL | [7:3] FF_DUR | 낙상 지속시간 (ODR 주기 단위) |
| 0x5B | WAKE_UP_THS | [5:0] WK_THS | 1 LSB = FS_XL/64 |
| 0x5E/0x5F | MD1_CFG/MD2_CFG | [3]FF [5]WU | 낙상/웨이크업 → INT1/INT2 라우팅 |

ST 공식 낙상 예제: ODR=416Hz, FS=±2g, FF_THS=312mg, FF_DUR=6 (~14.4ms).
**소비전류**: 가속도+자이로 0.55mA · 가속도 단독 12.5Hz ~25µA · 파워다운 <5µA

**출처**: [ST DS](https://www.st.com/resource/en/datasheet/lsm6dso.pdf) · [AN5192](https://www.st.com/resource/en/application_note/an5192-lsm6dso-alwayson-3axis-accelerometer-and-3axis-gyroscope-stmicroelectronics.pdf) · [C driver regmap](https://github.com/stm32duino/LSM6DSO/blob/main/src/lsm6dso_reg.h) · [free-fall 예제](https://github.com/STMicroelectronics/STMems_Standard_C_drivers/blob/master/lsm6dso_STdC/examples/lsm6dso_free_fall.c)

## A-3. Quectel BG95-M2 AT 명령 (LTE-M / PSM / SMS)

Cat-M1/Cat-NB2/EGPRS 멀티모드. VBAT 2.6~4.8V.

| 명령 | 기능 | 핵심 |
|------|------|------|
| `AT+CFUN=1` | 모뎀 전체 기능 | 0=최소 · 1=전체 · 4=비행 |
| `AT+CEREG=2` | EPS 등록 URC | 응답 `,9` = eMTC(LTE-M) 등록 |
| `AT+CPSMS` | PSM (3GPP) | `=1,,,"<T3412>","<T3324>"` 8-bit 이진 타이머 |
| `AT+QPSMS` | PSM (Quectel) | `=1,"<TAU>","<Active>"` |
| `AT+QCFG="psm/enter"` | 즉시 PSM 진입 | |
| `AT+CMGF=1` → `AT+CMGS="+82..."` | SMS 텍스트 전송 | 본문 후 CTRL+Z(0x26) |

PSM 타이머 단위(bit[7:5]): 000=10분 · 001=1시간 · 010=10시간 · 101=1분(T3324) · 110=2초.
**주의**: PSM 진입 중 SMS 수신 불가 → 중요 수신 시 eDRX 병용. 펌웨어 PSM 지원 버전 확인.

**출처**: [BG95 AT Manual V2.0](https://sixfab.com/wp-content/uploads/2023/05/Quectel_BG95BG77BG600L_Series_AT_Commands_Manual_V2.0.pdf) · [QCFG Manual](https://sixfab.com/wp-content/uploads/2023/05/Quectel_BG95BG77BG600L_Series_QCFG_AT_Commands_Manual_V2.0.pdf)

## A-4. nRF52840 DK (PCA10056) 핀 맵

SPI0~3, I2C0/I2C1 가용. 유연한 핀 매핑.

**충돌 고정 핀**: LED1~4=P0.13~16 · BTN1~4=P0.11/12/24/25 · UART0=P0.05~08 · XTAL=P0.00/01

| SPI | MOSI | MISO | SCK | 비고 |
|-----|------|------|-----|------|
| SPI0 | P0.26 | P0.29 | P0.27 | I2C0 핀 공유 |
| SPI2 | P0.20 | P0.21 | P0.19 | 독립 — 외부 센서 권장 |
| SPI3 | P1.13 | P1.14 | P1.15 | Arduino 헤더 — 외부 센서 권장 |

| I2C | SDA | SCL | 비고 |
|-----|-----|-----|------|
| I2C0 | P0.26 | P0.27 | LSM6DSO 권장 (SPI0 동시 사용 불가) |
| I2C1 | P0.30 | P0.31 | SPI1 핀 공유 |

**PoC 권장 배선**: MAX86141 → SPI3(P1.13/14/15)+CS P0.04+INT P0.03 · LSM6DSO → I2C0(P0.26/27)+INT1 P0.28 · BG95-M2 → UART1(TX P1.02 / RX P1.01)
⚠️ P0.26/27 = 저주파 I/O — I2C OK, SPI(MHz) 불가.

**출처**: [Zephyr DTS](https://github.com/zephyrproject-rtos/zephyr/blob/main/boards/nordic/nrf52840dk/nrf52840dk_nrf52840.dts) · [Pinctrl DTSI](https://github.com/zephyrproject-rtos/zephyr/blob/main/boards/nordic/nrf52840dk/nrf52840dk_nrf52840-pinctrl.dtsi) · [Zephyr 문서](https://docs.zephyrproject.org/latest/boards/nordic/nrf52840dk/doc/index.html)

## A-5. 전력 예산 — 이론 시뮬레이션 (380mAh LiPo + MCP73831)

> ⚠️ **이론 추정치.** 시스템 손실·레귤레이터 효율·기생 성분 미반영. 실측은 Phase 4 개발자.

### MCP73831 충전 설계

`I_CHARGE(mA) = 1000 / R_PROG(kΩ)` — Microchip DS20001984H

| R_PROG | I_CHARGE | 비고 |
|--------|----------|------|
| 2 kΩ | 500 mA | ~1.3C (380mAh) |
| 2.7 kΩ | ~370 mA | **권장 ~0.97C** |
| 10 kΩ | 100 mA | 저속 |

VDD 3.75~6.0V · 충전전압 4.20V±0.75% · 완료 후 대기 <2µA

### 모드별 이론 전류

| 모드 | 구성 | 합계 |
|------|------|------|
| PPG 연속 감지 | nRF52840 ~1.5mA + MAX86141 ~32µA + LSM6DSO ~0.55mA + BG95 대기 3~9mA | **~6~12mA** |
| 딥슬립 | nRF52840 ~2.35µA + MAX86141 0.6µA + LSM6DSO <5µA + BG95 PSM ~3µA | **~11µA** |
| LTE-M TX | nRF52840 ~1.5mA + **BG95-M2 TX 미확인** | **미확인** |

### 이론 수명 (TX 미포함)

```
평균 ≈ 0.10×12mA + 0.85×0.011mA ≈ 1.21mA
수명 ≈ 380mAh / 1.21mA ≈ 314h ≈ 약 13일
```
> LTE-M TX 전류 포함 시 단축. Phase 4 실측 후 재계산 필수.

**출처**: [MCP73831 DS](https://ww1.microchip.com/downloads/en/DeviceDoc/MCP73831-Family-Data-Sheet-DS20001984H.pdf) · [nRF52840 PS v1.11](https://docs.nordicsemi.com/bundle/ps_nrf52840/page/keyfeatures_html5.html)

---

# D. 회로도·PCB 참고 자료 수집

## D-1. MAX86141 + nRF52840 SPI 연결

| 항목 | 값 |
|------|-----|
| VDD_ANA/DIG | 1.8V · VLED 3.1~5.5V |
| SPI | 4MHz max · Mode 0 (CPOL=0,CPHA=0) · 3바이트/24클록 프레임 |
| REFIN | 1µF → GND_ANA |

검증된 핀 예시(Nordic DevZone): SCLK P0.26 · SDI P0.29 · SDO P0.30 · CSB P0.31 + INT GPIO.
⚠️ **레벨 시프터**: nRF52840 VDD=1.8V면 불필요. 3.3V GPIO 운용 시 TXB0304 등 필요 (DevZone에 MISO 신호 손실 사례).

**확인 포인트**: VDD_ANA/DIG에 100nF+1µF 인접 배치 · VLED/VDD 레일 분리 · REFIN 1µF · SPI Mode 0 · INT 풀업 확인 · 3.3V면 레벨 시프터.

**출처**: [ADI DS](https://www.analog.com/media/en/technical-documentation/data-sheets/max86140-max86141.pdf) · [EVSYS DS (Mouser)](https://www.mouser.com/datasheet/2/256/MAX86140EVSYS-1139679.pdf) · [DevZone SPI Q&A](https://devzone.nordicsemi.com/f/nordic-q-a/73646/spi-communication-between-nrf52840dk-master-and-max86140-slave-no-signal-in-miso-line) · [GitHub MakerLabLPI/Max86141](https://github.com/MakerLabLPI/Max86141)

## D-2. Quectel BG95 전원 시퀀스 + 안테나 매칭

| 항목 | 규격 (HW Design V1.5) |
|------|------------------------|
| VBAT | 3.4~4.4V (typ 3.8V) · BB 트레이스 ≥0.5mm · RF ≥2.0mm |
| 디커플링 | 100µF 저ESR + (100nF+33pF+10pF) MLCC, VBAT 핀 최근접 |
| PWRKEY ON | Low 500~1000ms (VBAT 안정 후 ≥30ms) |
| PWRKEY OFF | Low 650~1500ms · 내부 풀업 470kΩ · 출력 1.5V |
| RF | 50Ω 제어 · π형 매칭(R1=0Ω, C1/C2 NM 출하, 튜닝 시 마운트) |

**확인 포인트**: 100µF VBAT 핀 2mm 이내 · PWRKEY는 MCU GPIO→NMOS(2N7002) · π 패드 예비 · 메인/GNSS 안테나 간격 ≥5mm · 50Ω 트레이스.

**출처**: [HW Design V1.5](https://images.quectel.com/python/2023/04/Quectel_BG95_Series_Hardware_Design_V1.5.pdf) · [Reference Design V1.3](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_BG95_Series_Reference_Design_V1.3-1.pdf)

## D-3. 배터리 충전 회로 (MCP73831)

| 소자 | 값 |
|------|-----|
| C_IN / C_BAT | ≥4.7µF X7R 10V+ |
| R_PROG | 2kΩ→500mA (공식 I=1000/R_PROG) |
| R_STAT | 470Ω |
| D_IN | Schottky (BAT54 등) 역전류 방지 |

입력 3.75~6.0V · 정전압 4.20V 고정 · SOT-23-5 / DFN-8.
**USB-C**: CC1/CC2 각 5.1kΩ→GND 필수 · VBUS TVS(PRTR5V0U2X) 권장.

**확인 포인트**: R_PROG 1% 정밀 · 4.7µF X7R · USB-C CC저항 필수 · STAT→MCU GPIO 모니터 · SOT-23 발열(500mA 주의) · 배터리 보호 IC(DW01A+FS8205) 추가 권장.

**출처**: [MCP73831 DS DS20001984H](https://ww1.microchip.com/downloads/en/DeviceDoc/MCP73831-Family-Data-Sheet-DS20001984H.pdf) · [Eval Board DS51596A](https://ww1.microchip.com/downloads/en/devicedoc/51596a.pdf)

## D-4. e-Ink 패널 SPI 인터페이스

### ⚠️ 1.5" 240×240 = 상용 규격 부재

가장 근접 입수 가능: **1.54" 200×200** (Waveshare GDEH0154D67/SSD1681 또는 GDEW0154M09/JD79653A).
→ **PoC 권장: Waveshare 1.54" 200×200 (SSD1681)**. 240×240 필요 시 1.3"/1.6" 검토하나 공급 제한적.

| 핀 | 기능 |
|----|------|
| VCC | 3.3V (모듈 부스트 내장) |
| DIN/CLK | SPI MOSI / 클록 |
| CS / DC / RST | 선택 / 데이터·커맨드 / 리셋 |
| BUSY | High=갱신중(SSD1681) — JD79653A는 극성 반전 ⚠️ |

SPI Mode 0 · 2.3~3.6V · 고전압은 드라이버 IC 내부 DC-DC 생성(외부 부스트 불필요).

**확인 포인트**: VCC 100nF · BUSY High 중 SPI 금지 · RST ≥10ms Low · DC 전환 · **구매 전 SSD1681 vs JD79653A 확인(BUSY 극성)** · UI 200×200 기준 설계.

**출처**: [Waveshare 1.54" 모듈](https://www.waveshare.com/1.54inch-e-paper-module.htm) · [Wiki](https://www.waveshare.com/wiki/1.54inch_e-Paper_Module_Manual) · [Good Display GDEH0154D67](https://www.good-display.com/product/1.54-inch-e-paper-display-module-partial-refresh-E-ink-screen,-GDEH0154D67-208.html) · [Adafruit #4196](https://www.adafruit.com/product/4196) · [GxEPD2](https://github.com/ZinggJM/GxEPD2)

---

# E. 조달 및 구매 목록

> 조회 2026-05-18 · 1 USD ≈ 1,370 KRW (참고). 가격 변동성 — 실구매 시 재확인.

## E-1. 커스텀 부품 경로 BOM (qty 2~5)

| 부품번호 | 설명 | 수량 | 단가(USD) | 합계(qty5) | 리드타임 | 출처 |
|----------|------|------|-----------|-----------|----------|------|
| MAX86141ENP+T | PPG AFE, 테이프릴 | 5 | $10.57→$9.57 | ~$47.85 | 10주(공장 55k) | [Digi-Key](https://www.digikey.com/en/products/detail/analog-devices-inc-maxim-integrated/MAX86141ENP-T/7804058) |
| LSM6DSOTR | 6축 IMU | 5 | $4.17→$3.73 | ~$18.65 | **24주 ⚠️ 재고0** | [Digi-Key](https://www.digikey.com/en/products/detail/stmicroelectronics/LSM6DSOTR/9586579) |
| BG95-M2 | LTE-M/NB-IoT 모듈 | 3 | **$13~37 조회필요** | 조회필요 | 조회필요 | [Mouser](https://www.mouser.com/ProductDetail/Quectel/BG95-M2) |
| MDBT50Q-1MV2 | nRF52840 BLE 모듈 | 3 | $6.15 | ~$18.45 | 14일 (+$25 배송) | [Digi-Key MP](https://www.digikey.com/en/products/detail/raytac/MDBT50Q-1MV2/13677591) |
| MCP73831T-2ACI/OT | LiPo 충전 IC | 5 | $0.76 | ~$3.80 | 4주(재고 38k) | [Digi-Key](https://www.digikey.com/en/products/detail/microchip-technology/MCP73831T-2ACI-OT/964301) |
| LiPo 400mAh 3.7V | 보호회로 내장 (380mAh 미유통 대체) | 3 | $6.95 | ~$20.85 | 즉시 | [Adafruit #3898](https://www.adafruit.com/product/3898) |
| Waveshare 1.54" e-Paper | 200×200 흑백 (1.5" 240² 부재 대체) | 2 | $13.29~14.99 | ~$26.58~29.98 | 즉시 | [Waveshare](https://www.waveshare.com/1.54inch-e-paper-module.htm) |
| 패시브 일식 | 디커플링·저항·인덕터 0402/0603 | 일식 | $5~15 | $5~15 | Mouser/LCSC |

**소계 (커스텀 경로 qty 3~5)**: **$212~285** (BG95-M2 확정가 별도)

**주요 대체**:
- **380mAh LiPo** — Mouser/Digi-Key 미취급. PoC는 Adafruit 400mAh. 양산 EEMB/PKCELL 직발주(MOQ 50+).
- **1.5" 240×240 e-ink** — 상용 부재. Waveshare 1.54" 200×200 대체 확정 권장.
- **LSM6DSOTR** — 재고0/24주. 대안 LSM6DSOXTR / LSM6DS3TR-C.

## E-1-C. 브레이크아웃 모듈 경로 (메인 DK + 모듈, 권장)

> 베어 IC 납땜·커스텀 PCB 없이, nRF52840 DK 헤더에 **제조사 브레이크아웃 모듈**을 점퍼/Qwiic로 직결. Phase 1~3 신호·통신 검증에 충분. 조회일 2026-05-18.

| 부품 | 보드 모델 | 탑재 칩 | 인터페이스 | 단가(USD) | 출처 |
|------|-----------|---------|-----------|-----------|------|
| MCU (메인) | Nordic nRF52840 DK | nRF52840 | — | $48.95 | [Digi-Key](https://www.digikey.com/en/products/detail/nordic-semiconductor-asa/NRF52840-DK/8593726) |
| IMU | Adafruit LSM6DSOX (ADA-4438) | **LSM6DSOX** | I2C/SPI (STEMMA QT) | $11.95 | [Digi-Key](https://www.digikey.com/en/products/detail/adafruit-industries-llc/4438/11497501) |
| LTE-M | Waveshare BG95-M3 Zero | **BG95-M3** | UART + USB-C, Nano SIM | ~$55~60 | [Waveshare](https://www.waveshare.com/bg95-m3-zero.htm) / [Amazon](https://www.amazon.com/dp/B0D4V8MSB8) |
| e-ink | Waveshare 1.54" e-Paper V2 | 200×200 | SPI | ~$10~11 | [Waveshare](https://www.waveshare.com/1.54inch-e-paper-module.htm) / [LCSC C359939](https://lcsc.com/product-detail/Modules_Waveshare_C359939.html) |
| PPG (저가, **비호환**) | SparkFun SEN-16474 | **MAX30101** | I2C (Qwiic) | $34.39 | [SparkFun](https://www.sparkfun.com/sparkfun-photodetector-breakout-max30101-qwiic.html) |
| PPG (**알고리즘 패리티**) | MAX86140EVSYS# | **MAX86141** | USB-UART + GUI | $158.21 | [Digi-Key](https://www.digikey.com/en/products/detail/analog-devices-inc-maxim-integrated/MAX86140EVSYS/7597515) |

**총액 추산**
- 브레이크아웃만 (PPG=SEN-16474 MAX30101): DK $48.95 + IMU $11.95 + LTE-M $58 + e-ink $11 + PPG $34.39 ≈ **$165** (부품 1벌). PPG 알고리즘 검증 불가.
- **브레이크아웃 + PPG만 EVSYS (권장)**: 위에서 PPG를 MAX86140EVSYS($158)로 교체 ≈ **$270~290**. MAX86141 알고리즘 패리티 유지.

**주의**
- ⚠️ **칩 패리티 — MAX30101 ≠ MAX86141**: 현재 구매 가능한 모든 PPG 브레이크아웃은 MAX30101 탑재. MAX86141 단독 브레이크아웃은 시중 부재(SparkFun SEN-16656 단종). 광학 설계·알고리즘 상이 → **PoC 목표가 MAX86141 알고리즘 검증이면 PPG는 MAX86140EVSYS 필수**. 검증 대상이 "맥파 소실 감지 가능성" 수준이면 MAX30101로 1차 탐색 후 EVSYS 전환도 가능.
- **레벨 시프터 불필요**: SparkFun PPG 브레이크아웃·Waveshare e-ink 모듈은 온보드 레귤레이터/레벨 변환 내장. LSM6DSO(X) 브레이크아웃도 3.3V 직결. nRF52840 DK 3.3V GPIO 그대로 사용 — D-1의 TXB0304 외부 시프터 생략.
- **BG95 변형**: Waveshare 보드 = BG95-**M3**(글로벌). 타겟 M2 아님. Cat-M1 기능·AT 커맨드 동등, 검증 가능. 최종 부품은 M2.
- **LSM6DSOX vs DSO**: ADA-4438 = DSOX(ML 코어 추가, 상위 호환). PoC 수준 차이 없음.
- 가격·재고 2026-05-18 기준. Adafruit 직접몰 LSM6DSOX 품절 → Digi-Key 경유.

## E-2. 평가보드 경로

| 보드 | 부품번호 | 단가 | 재고 | 출처 |
|------|----------|------|------|------|
| Nordic nRF52840 DK | NRF52840-DK | $48.95 | 661 | [Digi-Key](https://www.digikey.com/en/products/detail/nordic-semiconductor-asa/NRF52840-DK/8593726) |
| Quectel BG95 EVB | BG95M3-QPYTHON-EVB | $156.95 | 발주 | [Digi-Key](https://www.digikey.com/en/products/detail/quectel/BG95M3-QPYTHON-EVB/24349311) |
| MAX86140/41 Eval | MAX86140EVSYS# | $158.21 | 13 | [Digi-Key](https://www.digikey.com/en/products/detail/analog-devices-inc-maxim-integrated/MAX86140EVSYS/7597515) |

**1세트 = $364.11 · 2세트 ≈ $728**
> BG95 EVB는 M3 탑재(M2와 핀/AT 호환, 검증 가능). e-ink는 nRF52840 DK SPI 직결 — 별도 보드 불필요.

## E-3. e-ink 1.54" 소량 구매 옵션

| 공급처 | 모델 | 단가 | MOQ | 출처 |
|--------|------|------|-----|------|
| Waveshare 공식 | 1.54" B&W 200×200 | $13.29~14.99 | 1 | [waveshare.com](https://www.waveshare.com/1.54inch-e-paper-module.htm) |
| Amazon | Waveshare 1.54" V2 | 조회필요(~$18) | 1 | [Amazon B0728BJTZC](https://www.amazon.com/dp/B0728BJTZC) |
| Good Display | GDEY0154D67 (SSD1681) | 문의 견적 | 미정 | [good-display.com](https://www.good-display.com/product/1.54-inch-e-paper-display-module-partial-refresh-E-ink-screen-panel,-GDEH0154D67-208.html) |

> 권장: PoC 소량은 **Waveshare 공식/Amazon** 최속. GDEW0154M09 = EOL → GDEY0154D67/Waveshare 현행.

## 총 예산 범위

| 시나리오 | USD |
|---------|-----|
| 브레이크아웃만 (PPG=MAX30101, 알고리즘 비호환) | $115~165 |
| **브레이크아웃 + PPG만 EVSYS (권장)** | **$270~290** |
| 커스텀 부품 경로 (qty 3~5, 베어 IC) | $250~350 |
| 평가보드 경로 (3보드 2세트) | $750~900 |

---

# 검토 필요 항목 (개발자 직접 확인 — 위임 불가)

> 원본 PDF 접근 실패·시장 부재·시점 의존으로 미확정. PoC 착수 전 개발자 확인.

## A — 데이터시트 미확인 (10건)

| # | 항목 | 출처 |
|---|------|------|
| A-i | MAX86141 PPG_SR 전체 인코딩표 (0x0A~0x13) | ADI DS Table reg 0x12 |
| A-ii | MAX86141 LED_RGE 레지스터(0x2A/0x2B) 상세 비트 | ADI DS |
| A-iii | LSM6DSO FF_THS `111b` 7번째 임계값 | ST DS Table 87 |
| A-iv | LSM6DSO ODR_XL 전체 비트표 | ST DS Table 19 |
| A-v | **BG95-M2 LTE-M TX 전류 (peak/avg)** — 전력 예산 핵심 | [Quectel HW V1.5](https://images.quectel.com/python/2023/04/Quectel_BG95_Series_Hardware_Design_V1.5.pdf) Table 41/45 |
| A-vi | BG95-M2 PSM 슬립 전류 정확값 | Quectel 사양서 |
| A-vii | nRF52840 System OFF 슬립 전류 | Nordic PS v1.11 Table 45 |
| A-viii | nRF52840 Low-Frequency I/O 핀 목록 | Nordic Product Spec |
| A-ix | MCP73831 충전 중 자체 소비전류(IQ) | Microchip DS |
| A-x | BG95-M2 QPSMS 타이머 이진 인코딩 전체표 | Quectel PSM App Note |

## D — 회로 미확인 / 규격 부재

| # | 항목 | 상태 |
|---|------|------|
| D-i | MAX86141 EVSYS 공식 회로도 PDF · §Layout (디커플링 배치) | ⚠️ 직접 다운로드 필요 |
| D-ii | BG95 Reference Design 회로 부품값 · RF π C1/C2 | ⚠️ 안테나 임피던스 실측 후 결정 |
| D-iii | MCP73831 Eval Board USB 커넥터 (원본 Micro-B, USB-C 시 CC저항 추가) | ⚠️ 미확인 |
| D-iv | **1.5" 240×240 e-ink** | ❌ 시장 부재 → 1.54" 200×200 확정 |

## E — 가격/조달 미확정

| # | 항목 | 우선순위 |
|---|------|----------|
| E-i | **BG95-M2 확정 단가** (Quectel 견적/Mouser 직조회) | 높음 |
| E-ii | **LSM6DSOTR 대체품** (LSM6DSOXTR/LSM6DS3TR-C 재고) | 높음 |
| E-iii | 380mAh 정확 셀 (EEMB/PKCELL, PoC는 400mAh 대체) | 중간 |
| E-iv | MDBT50Q-1MV2 별도 배송비 $25 (2개+ 합리화) | 중간 |
| E-v | Good Display / JLCPCB e-ink 단가 직확인 | 낮음 |

---

*본 문서는 `silvercare-poc-ai-tasks-ade.md` 정의 작업의 웹 조사 수행 결과(2026-05-18)다.*
*모든 수치는 출처 확인용 초안이며, 채택·발주·실측은 개발자/사업책임자 결정 영역이다.*
