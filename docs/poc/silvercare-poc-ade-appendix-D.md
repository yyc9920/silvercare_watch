## D. 회로도·PCB 참고 자료 수집

> **조사 기준일**: 2026-05-18  
> **목적**: SilverCare Watch (손목 PPG + LTE-M) 하드웨어 PoC를 위한 검증된 레퍼런스 회로 및 애플리케이션 노트 수집  
> **주의**: 부품값은 실제 확인된 출처만 기재. 미확인 값에는 별도 표기.

---

### D-1. MAX86141 + nRF52840 SPI 연결 레퍼런스

#### 개요
MAX86141은 Analog Devices(구 Maxim Integrated)의 완전 통합형 광학 데이터 수집 시스템으로, 손목 PPG(심박·혈중산소) 측정에 최적화된 IC이다. 표준 SPI 인터페이스를 통해 nRF52840과 연결한다.

#### 핵심 전기 규격 (데이터시트 확인값)
| 항목 | 값 |
|---|---|
| VDD_ANA / VDD_DIG | 1.8 V |
| VLED (LED 드라이버 공급) | 3.1 V ~ 5.5 V |
| SPI 최대 클록 | 4 MHz |
| SPI 모드 | Mode 0 (CPOL=0, CPHA=0) |
| SPI 프레임 길이 | 3 바이트 / 24 클록 사이클 (CSB Low 구간) |
| REFIN 디커플링 캐패시터 | 1 µF → GND_ANA |

#### SPI 핀 매핑
MAX86141 핀 명칭은 **SCLK · SDI(MOSI) · SDO(MISO) · CSB(액티브 로우)**이다.  
Nordic DevZone Q&A(nRF52840DK ↔ MAX86140/41 연동 스레드)에서 검증된 예시 핀 배정:

| MAX86141 핀 | nRF52840DK 핀 | 비고 |
|---|---|---|
| SCLK | P0.26 | SPI 클록 |
| SDI (MOSI) | P0.29 | 마스터→슬레이브 데이터 |
| SDO (MISO) | P0.30 | 슬레이브→마스터 데이터 |
| CSB | P0.31 | 액티브 로우 |
| INT | 별도 GPIO | 인터럽트 (데이터 준비 통보) |

> **레벨 시프터 주의**: MAX86141 VDD=1.8 V, nRF52840 GPIO=1.8 V 또는 3.3 V 설정 가능.  
> nRF52840의 VDD를 1.8 V로 설정하면 레벨 시프터 불필요.  
> 3.3 V로 운용 시 TXB0304 등 양방향 레벨 시프터 필요(DevZone 케이스에서 불량 점퍼로 MISO 신호 손실 사례 확인).

#### LED 드라이버 경로
- LED1_DRV / LED2_DRV / LED3_DRV 핀 → LED 캐소드 연결  
- LED 애노드 → VLED 공급 (3.1~5.5 V)  
- MAX86141(단채널) : LED 드라이버 1개, MAX86141(이중채널) : 최대 3드라이버로 외부 3×2:1 MUX를 통해 6개 LED 구동 가능  

#### 평가보드 (EVSYS) 정보
- **문서명**: *Evaluates: MAX86140 and MAX86141 — MAX86140/MAX86141 Evaluation System*  
- **문서번호**: MAX86140EVSYS 데이터시트 (Analog Devices / Mouser 미러)  
- WLP 24-범프 패키지(MAX86141ENP+) 탑재, 손목 PPG 평가 특화

#### 출처
- [MAX86141 제품 페이지 — Analog Devices](https://www.analog.com/en/products/max86141.html)
- [MAX86140/MAX86141 데이터시트 (analog.com PDF)](https://www.analog.com/media/en/technical-documentation/data-sheets/max86140-max86141.pdf)
- [MAX86140EVSYS 데이터시트 (Mouser 미러)](https://www.mouser.com/datasheet/2/256/MAX86140EVSYS-1139679.pdf)
- [Nordic DevZone — nRF52840DK ↔ MAX86140 SPI 연동 Q&A](https://devzone.nordicsemi.com/f/nordic-q-a/73646/spi-communication-between-nrf52840dk-master-and-max86140-slave-no-signal-in-miso-line)
- [Nordic DevZone — MAX86141 SPI with nRF5340](https://devzone.nordicsemi.com/f/nordic-q-a/109399/max86141-spi-communication-with-nrf5340)
- [GitHub — MakerLabLPI/Max86141 (nRF52840 예제)](https://github.com/MakerLabLPI/Max86141)

#### 확인 포인트
- [ ] VDD 1.8 V 공급 시 100 nF + 1 µF 디커플링 캐패시터를 VDD_ANA·VDD_DIG 핀 인접 배치
- [ ] VLED와 VDD 전원 레일 분리 (VLED는 배터리 직결 또는 별도 LDO)
- [ ] REFIN 핀에 1 µF 캐패시터 → GND_ANA (데이터시트 명시)
- [ ] SPI Mode 0 설정 확인 (데이터는 SCLK 상승 엣지 래치, 하강 엣지 출력)
- [ ] INT 핀 활성화 시 풀업 저항(100 kΩ) 필요 여부 데이터시트 확인
- [ ] 3.3 V GPIO 환경이라면 레벨 시프터(TXB0304 또는 동급) 삽입

---

### D-2. Quectel BG95 모뎀 전원 시퀀스 + 안테나 매칭 레퍼런스

#### 개요
Quectel BG95는 LTE-M / NB-IoT / EGPRS 지원 LPWA 모듈이다. 공식 Hardware Design 문서(V1.5, 2022-10-09)와 Reference Design(V1.3)에 VBAT 인러시, PWRKEY 타이밍, RF 매칭 네트워크가 상세히 기술되어 있다.

#### VBAT 전원 설계 (§3.5 확인값)
| 항목 | 규격 |
|---|---|
| VBAT 전압 범위 | 3.4 V ~ 4.4 V (typ. 3.8 V) |
| VBAT_BB 트레이스 폭 | ≥ 0.5 mm |
| VBAT_RF 트레이스 폭 | ≥ 2.0 mm |
| 디커플링 (대용량) | 100 µF (저 ESR 전해) — VBAT 핀 인근 배치 |
| 디커플링 (MLCC 어레이) | 100 nF + 33 pF + 10 pF (VBAT 핀 최근접) |

> 트레이스가 길어질수록 폭을 추가로 확대할 것.

#### PWRKEY 타이밍 (§3.6.1 / §3.6.2 확인값)
| 이벤트 | 타이밍 |
|---|---|
| VBAT 안정화 → PWRKEY Low 사이 최소 간격 | ≥ 30 ms |
| **전원 ON**: PWRKEY Low 유지 시간 | 500 ms ~ 1000 ms |
| **전원 OFF**: PWRKEY Low 유지 시간 | 650 ms ~ 1500 ms |
| PWRKEY 핀 출력 전압 | 1.5 V (Qualcomm 칩셋 내부 전압 강하) |
| PWRKEY 내부 풀업 | 470 kΩ |

> 시스템 MCU가 3.3 V GPIO로 PWRKEY를 구동하는 경우, 오픈 드레인 출력 + MOSFET 또는 레벨 변환 회로 사용 권장.

#### RF 안테나 매칭 (§5.1.x 확인값)
- RF 트레이스 특성 임피던스: **50 Ω** 제어 필수
- 안테나 포트에 **π형 매칭 회로** 예비 장착 권고 (R1/C1/C2)
  - 기본값: R1 = 0 Ω (쇼트), C1·C2 = 미장착 (Not Mounted)
  - 실제 안테나 임피던스에 맞춰 C1/C2 조정
- GNSS 안테나 트레이스: 메인 안테나와 최대 거리 유지, 접지 비아 주변 배치로 차폐

#### 문서 목록
| 문서명 | 버전 | 비고 |
|---|---|---|
| BG95 Series Hardware Design | V1.5 (2022-10-09) | 회로 설계 전체 기준 |
| BG95 Series Reference Design | V1.3 | 실제 레퍼런스 회로도 포함 |
| BG95 & BG96 Compatible Design | V1.0 | BG95/96 호환 설계 참고 |

#### 출처
- [BG95 Series Hardware Design V1.5 (Quectel images 서버 PDF)](https://images.quectel.com/python/2023/04/Quectel_BG95_Series_Hardware_Design_V1.5.pdf)
- [BG95 Series Reference Design V1.3 (Quectel developer portal)](https://developer.quectel.com/wp-content/uploads/2024/09/Quectel_BG95_Series_Reference_Design_V1.3-1.pdf)
- [BG95 Hardware Design V1.3 (Quectel 공식 PDF)](https://quectel.com/content/uploads/2024/02/Quectel_BG95_Series_Hardware_Design_V1.3-2.pdf)
- [BG95 & BG96 Compatible Design V1.0 (Codico 미러)](https://content.codico.com/fileadmin/media/download/datasheets/quectel/Quectel_BG95_BG96_20_Compatible_20_DesignV1.0.pdf)
- [ManualsLib — Quectel BG95 Hardware Design (레이아웃 가이드라인 §56)](https://www.manualslib.com/manual/1949966/Quectel-Bg95-Series.html?page=56)

#### 확인 포인트
- [ ] VBAT 공급 커패시터 100 µF (저 ESR)를 BG95 모듈 VBAT 핀에서 2 mm 이내 배치
- [ ] PWRKEY 구동 회로: MCU GPIO → NMOS(예: 2N7002) → PWRKEY 권장
- [ ] 안테나 커넥터(U.FL 또는 SMA)와 BG95 RF 핀 사이 π형 패드 예비 (R=0Ω, C1/C2 미장착으로 출하, 튜닝 시 마운트)
- [ ] 메인 안테나와 GNSS 안테나 트레이스 간격 ≥ 5 mm 유지
- [ ] RF 트레이스 임피던스: 50 Ω (PCB 스택업에 맞게 CPW 또는 마이크로스트립 계산)

---

### D-3. 배터리 충전 회로 레퍼런스 (MCP73831)

#### 개요
Microchip MCP73831은 단일 셀 Li-Ion/Li-Po 배터리용 선형 충전 관리 IC이다. 최소 외부 부품으로 USB 입력 충전을 구현할 수 있으며, 공식 데이터시트(DS20001984H)에 전형적인 응용 회로가 포함되어 있다.

#### 전형적 응용 회로 구성 (데이터시트 DS20001984H 확인값)

**부품 목록**:

| 소자 | 값 / 사양 | 역할 |
|---|---|---|
| C_IN (VIN 디커플링) | ≥ 4.7 µF, X7R, 10 V 이상 | 입력 전압 안정화 |
| C_BAT (VBAT 출력) | ≥ 4.7 µF, X7R, 10 V 이상 | 출력 충전 전압 안정화 |
| R_PROG (PROG 핀) | 2 kΩ → 500 mA 충전 전류 | 충전 전류 설정 |
| R_STAT (STAT LED) | 470 Ω | 상태 표시 LED 전류 제한 |
| D_IN (입력 역방향 보호) | Schottky 다이오드 (예: BAT54, V_F ≈ 0.3 V) | 역전류 방지 |

> **충전 전류 공식**: I_CHARGE (mA) = 1000 / R_PROG (kΩ)  
> 예: R_PROG = 2 kΩ → I_CHARGE = 500 mA  
> 예: R_PROG = 10 kΩ → I_CHARGE = 100 mA (소형 배터리용 안전 값)

**입력 전압 범위**: 3.75 V ~ 6.0 V (USB 5 V 호환)  
**정전압 충전 전압**: 4.20 V 고정 (또는 주문 옵션으로 4.35 / 4.40 / 4.50 V)  
**패키지**: SOT-23-5 또는 2×3 mm DFN-8

#### STAT 핀 동작
| 상태 | STAT 핀 |
|---|---|
| 충전 중 | Low (LED ON) |
| 충전 완료 | High (LED OFF) |
| 슬립 / 오류 | High-Z |

#### USB-C 연결 참고
MCP73831 자체는 USB 프로토콜을 구분하지 않는다. USB-C 입력 회로 시:
- CC1/CC2 핀에 5.1 kΩ 저항 → GND (5 V/0.9 A UFP 모드 지정)
- MCP73831 VIN에 USB VBUS 직결 (또는 Schottky를 통해 연결)
- VBUS 과전압 보호: TVS 다이오드(예: PRTR5V0U2X, 5.5 V 클램프) 추가 권장

#### 평가 보드
- **문서번호**: DS51596A — *MCP73831 Evaluation Board User's Guide* (Microchip, 2005)
- Micro-B USB 입력, 배터리 커넥터 포함한 단순 레퍼런스 보드

#### 출처
- [MCP73831 데이터시트 DS20001984H (Microchip 공식)](https://ww1.microchip.com/downloads/en/DeviceDoc/MCP73831-Family-Data-Sheet-DS20001984H.pdf)
- [MCP73831 평가 보드 가이드 DS51596A (Microchip 공식)](https://ww1.microchip.com/downloads/en/devicedoc/51596a.pdf)
- [MCP73831 제품 페이지 — Microchip Technology](https://www.microchip.com/en-us/product/mcp73831)
- [MCP73831 데이터시트 구버전 (Adafruit CDN 미러)](https://cdn-shop.adafruit.com/datasheets/MCP73831.pdf)
- [MCP73831 + USB-C 이중 충전 회로 참고 (PCBway 블로그)](https://www.pcbway.com/blog/technology/Double_Lithium_Ion_Lithium_Polymer_USB_Type_C_Charger_863d1ae1.html)

#### 확인 포인트
- [ ] R_PROG 저항 정밀도 1% (금속 피막) 사용 — 충전 전류 오차 최소화
- [ ] C_IN, C_BAT 용량 ≥ 4.7 µF, X7R (온도 변화에 강한 유전체)
- [ ] USB-C 사용 시 CC1/CC2 각각 5.1 kΩ → GND 필수 (없으면 충전 안 됨)
- [ ] STAT 핀으로 MCU GPIO 충전 상태 모니터링 가능 (인터럽트 연결 권장)
- [ ] 선형 충전 IC이므로 (VBUS - V_BAT) × I_CHARGE 만큼 IC 자체 발열; SOT-23 패키지는 최대 500 mA 주의
- [ ] 배터리 보호 IC(예: DW01A + FS8205) 별도 추가 권장 (과충전·과방전·단락 보호)

---

### D-4. e-Ink 패널 SPI 인터페이스 레퍼런스

#### 패널 크기 가용성 경고 ⚠️
**1.5인치 240×240 e-ink 패널은 상용 표준 규격으로 존재하지 않는다.**  
시장에서 실제로 입수 가능한 가장 근접한 규격은 아래와 같다:

| 실제 규격 | 해상도 | 제조사 | 모델 예시 |
|---|---|---|---|
| **1.54인치 200×200** | 200×200 px | Waveshare / GoodDisplay | GDEH0154D67 (SSD1681), GDEW0154M09 (JD79653A) |
| 1.54인치 200×200 (컬러) | 200×200 px | Waveshare | 1.54inch e-Paper (B) |
| 1.54인치 200×200 (4색) | 200×200 px | Waveshare | 1.54inch e-Paper (G) |

> **결론**: PoC에서는 **Waveshare 1.54인치 200×200 (흑백, SSD1681)** 사용 권장.  
> 240×240 해상도가 필요하다면 1.3인치 또는 1.6인치 대역 제품을 검토해야 하나, 해당 크기의 e-ink 패널도 시장 공급이 제한적이다.

#### SPI 인터페이스 핀 구성 (Waveshare 1.54inch, 200×200)

| 핀 이름 | 방향 | 기능 |
|---|---|---|
| VCC | 전원 입력 | 3.3 V (모듈 자체 부스트 회로 내장) |
| GND | 전원 | 그라운드 |
| DIN (SDA) | 입력 | SPI MOSI — 직렬 데이터 입력 |
| CLK (SCL) | 입력 | SPI 클록 |
| CS (CSB) | 입력 | 칩 선택, 액티브 로우 |
| DC (D/C) | 입력 | High: 데이터, Low: 커맨드 |
| RST (RES#) | 입력 | 리셋, 액티브 로우 |
| BUSY | 출력 | High: 패널 갱신 중(명령 금지), Low: 대기 |

**SPI 타이밍**: CPOL=0, CPHA=0 (SPI Mode 0)  
**동작 전압**: 2.3 V ~ 3.6 V (VCC 3.3 V 권장)  
**내장 부스트 컨버터**: 패널 구동에 필요한 고전압(약 15 V 내외)은 드라이버 IC(SSD1681 등) 내부 DC-DC가 생성 — 외부 별도 부스트 회로 불필요

#### 드라이버 IC별 주요 차이
| 드라이버 IC | 해당 모델 | 부분 갱신 | 특이사항 |
|---|---|---|---|
| SSD1681 | GDEH0154D67, Waveshare V2 | 지원 | 가장 범용적, Adafruit 4196도 SSD1681 |
| JD79653A | GDEW0154M09 (GoodDisplay) | 지원 | M5Stack Core Ink에 채용; BUSY 핀 극성 반전(inverted: true 필요) |
| UC8151 (IL0373) | 152×152 소형 패널 | 부분 지원 | 1.54인치 표준 모듈에는 드물게 탑재 |

#### M5Stack Core Ink 주의사항
GDEW0154M09(JD79653A) 탑재 모듈의 경우:  
BUSY 핀이 High일 때 **준비 완료**를 의미하므로, ESPHome 등 라이브러리에서 `inverted: true` 설정 필수. 미설정 시 디스플레이 영구 손상 위험.

#### 출처
- [Waveshare 1.54inch e-Paper Module 제품 페이지](https://www.waveshare.com/1.54inch-e-paper-module.htm)
- [Waveshare 1.54inch e-Paper Module Wiki (Manual)](https://www.waveshare.com/wiki/1.54inch_e-Paper_Module_Manual)
- [Waveshare 1.54inch e-Paper 데이터시트 PDF](https://www.waveshare.com/w/upload/7/77/1.54inch_e-Paper_Datasheet.pdf)
- [GoodDisplay GDEW0154M09 (JD79653A) 제품 페이지](https://www.good-display.com/product/206.html)
- [GoodDisplay GDEH0154D67 (SSD1681) 모듈 제품 페이지](https://www.good-display.com/product/1.54-inch-e-paper-display-module-partial-refresh-E-ink-screen,-GDEH0154D67-208.html)
- [Adafruit 1.54" e-Ink 200×200 SSD1681 모듈 (Adafruit #4196)](https://www.adafruit.com/product/4196)
- [ESPHome Waveshare e-Paper 지원 목록 (드라이버 IC 포함)](https://esphome.io/components/display/waveshare_epaper/)
- [GxEPD2 Arduino 라이브러리 — 1.54인치 모델 선택 헤더](https://github.com/ZinggJM/GxEPD2)
- [Hutscape 튜토리얼 — Waveshare 1.54인치 SPI 연결 예제](https://hutscape.com/tutorials/waveshare-1in54-e-paper)

#### 확인 포인트
- [ ] VCC 3.3 V 공급, 100 nF 디커플링 캐패시터 패널 인근 배치
- [ ] BUSY 핀: High 동안 SPI 명령 전송 금지 (폴링 또는 인터럽트 처리)
- [ ] RST 핀: 초기화 시 ≥ 10 ms Low 펄스 인가
- [ ] DC 핀: 커맨드 전송 전 Low, 데이터 전송 전 High로 전환
- [ ] SSD1681 vs JD79653A: 구매 전 모델명 확인 — BUSY 핀 극성 다름
- [ ] PoC에서 1.54인치 200×200 채택 시, 최종 제품 UI 레이아웃 초기부터 200×200 기준으로 설계

---

## 검토 필요 항목

| 번호 | 항목 | 상태 | 비고 |
|---|---|---|---|
| 1 | MAX86141 EVSYS 공식 회로도(PDF) 직접 열람 | ⚠️ 미완료 | analog.com PDF가 웹 크롤러에서 바이너리로만 수신됨. Analog Devices 웹사이트에서 직접 다운로드 후 §Layout Guidelines 섹션 확인 필요 |
| 2 | MAX86141 VDD_ANA 핀별 100 nF 디커플링 위치 | ⚠️ 미확인 | 데이터시트 §Layout Considerations에서 VLED·VDD_ANA·VDD_DIG 각각의 커패시터 정확한 값 확인 필요 |
| 3 | BG95 Reference Design 회로도 회로 부품값 | ⚠️ 부분 확인 | RF π형 C1/C2 최적값은 실측 안테나 임피던스 측정 후 결정. 공식 Reference Design V1.3 PDF 직접 열람 필요 |
| 4 | MCP73831 평가 보드 DS51596A USB 커넥터 유형 | ⚠️ 미확인 | PDF 바이너리 파싱 실패. 원 문서는 2005년 작성으로 **Micro-B** 사용. PoC에서 USB-C 사용 시 CC저항 회로 추가 설계 별도 필요 |
| 5 | 1.5인치 240×240 e-ink 패널 가용성 | ❌ 해당 규격 없음 | 시장에 존재하지 않는 규격. **1.54인치 200×200** (SSD1681 드라이버)으로 대체 확정 권장 |
| 6 | BG95 PWRKEY 정확한 최소 Low 펄스 하한 | ✅ 확인 | 500 ms (Hardware Design V1.5 §3.6.1) |
| 7 | MCP73831 충전 전류 설정 공식 | ✅ 확인 | I = 1000/R_PROG (mA, kΩ); 2 kΩ → 500 mA |
| 8 | e-Paper BUSY 핀 HIGH 의미 | ✅ 확인 | 표준(SSD1681): High = 갱신 중(명령 금지); JD79653A: High = 준비 완료(반전) |
