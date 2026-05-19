## E. 조달 및 구매 목록 작성

> **조회 기준일**: 2026-05-18  
> **환율 참고**: 1 USD ≈ 1,370 KRW (작성 시점 참고용; 실구매 시 재확인 필요)

---

### E-1. PoC BOM 초안 (수량 2–5 기준)

#### 경로 분기

| 경로 | 설명 |
|------|------|
| **평가보드 경로** | 개별 IC 대신 평가보드·모듈 사용. 납땜 없이 빠른 기능 검증. 비용 높음 |
| **커스텀 부품 경로** | 상용 IC·모듈 직구매 후 PoC PCB에 실장. 비용 낮음, 설계 공수 필요 |

---

#### E-1-A. 커스텀 부품 경로 BOM

| 부품번호 | 설명 | 수량 | 단가(USD) | 합계(USD, qty 5) | 출처 | 리드타임 | 조회일 |
|----------|------|------|-----------|-----------------|------|---------|--------|
| MAX86141ENP+T | 광학 PPG AFE (손목 HR/SpO₂), WLCSP 제거 후 리테일 테이프릴 | 5 | $10.57 (qty 1) / $9.57 (qty 5) | ~$47.85 | [Digi-Key](https://www.digikey.com/en/products/detail/analog-devices-inc-maxim-integrated/MAX86141ENP-T/7804058) | 10주 (공장 55,000개 재고) | 2026-05-18 |
| MAX86141ENP+ | 광학 PPG AFE, 튜브 패키지 (DIP-like 8핀 WLCSP) | 5 | ~$16.39 (qty 1) | ~$81.95 | [Mouser](https://www.mouser.com/ProductDetail/Analog-Devices-Maxim-Integrated/MAX86141ENP+) | 재고 12,907개 | 2026-05-18 |
| LSM6DSOTR | 6축 IMU (가속도+자이로), TFLGA 14핀 | 5 | $4.17 (qty 1) / $3.73 (qty 5) | ~$18.65 | [Digi-Key](https://www.digikey.com/en/products/detail/stmicroelectronics/LSM6DSOTR/9586579) | 24주 ⚠️ (재고 0, 백오더) | 2026-05-18 |
| BG95M2LA-64-SGNS | Quectel BG95-M2 LTE-M/NB-IoT/EGPRS+GNSS, SMT 23.6×19.9mm | 3 | AUD $20.89 (≈USD $13–14 예상) | 조회 필요 | [Elecom AUS](https://www.elecomes.com/products/bg95m2la-64-sgns-quectel-bg95m2-multimode-cat-m1-nb2-nb-iot-module) / [Mouser](https://www.mouser.com/ProductDetail/Quectel/BG95-M2) | 조회 필요 | 2026-05-18 |
| MDBT50Q-1MV2 | Raytac nRF52840 BLE 모듈 (1MB Flash, 256KB RAM, U.FL 안테나) | 3 | $6.15 (qty 1) | ~$18.45 | [Digi-Key Marketplace](https://www.digikey.com/en/products/detail/raytac/MDBT50Q-1MV2/13677591) | 14일 (재고 50개, 별도 $25 배송비) | 2026-05-18 |
| MCP73831T-2ACI/OT | Li-Ion/LiPo 충전 IC, 500mA SOT-23-5 | 5 | $0.76 (qty 1) | ~$3.80 | [Digi-Key](https://www.digikey.com/en/products/detail/microchip-technology/MCP73831T-2ACI-OT/964301) | 4주 (재고 38,368개) | 2026-05-18 |
| LiPo 400mAh 3.7V | 36×17×7.8mm, JST-PH, 보호회로 내장 (380mAh 미재고 → 400mAh 대체) | 3 | $6.95 (qty 1) | ~$20.85 | [Adafruit #3898](https://www.adafruit.com/product/3898) | 즉시 출하 (재고 있음) | 2026-05-18 |
| Waveshare 1.54" e-Paper | 200×200 흑백 e-ink 모듈, SPI, 3.3V (1.5" 240×240 미존재 → 1.54" 200×200 대체) | 2 | ~$13.29–$14.99 | ~$26.58–$29.98 | [Waveshare 공식](https://www.waveshare.com/1.54inch-e-paper-module.htm) / [Amazon B0728BJTZC](https://www.amazon.com/dp/B0728BJTZC) | 즉시 출하 | 2026-05-18 |
| 패시브 부품 일식 (R, C, L) | 100nF/10µF 디커플링, 저항류, 인덕터 (0402/0603) | 일식 | 소계 $5–15 예상 | $5–15 | Mouser / Digi-Key / LCSC | — | — |

> **주요 대체 사항**:
> - **380mAh LiPo 셀**: Mouser/Digi-Key 미취급, 정확한 380mAh는 EEMB/PKCELL 직발주 또는 Alibaba (MOQ 50개 이상). PoC 규모에서는 Adafruit 400mAh ($6.95, 재고 확보)를 권장.
> - **1.5" 240×240 e-ink**: 해당 규격 상용 모듈 존재하지 않음. 가장 근접한 옵션은 Waveshare 1.54" 200×200 (HINK-E0154A05). 정원형 240×240 원형 e-ink는 JDI/E Ink 직접 수주 필요 (PoC 단계 비현실).
> - **BG95-M2 가격**: Mouser 페이지 접근 불가 (봇 차단). AliExpress 엔지니어링 샘플 $7.65–$9.00/개, 정상 유통가는 $13–37 범위로 확인됨.
> - **LSM6DSOTR**: Digi-Key 재고 0, 24주 리드타임. 대안: LSM6DSOXTR (IMU+온도, Digi-Key 재고 조회 권장) 또는 LSM6DS3TR-C.

---

#### E-1-B. 소계 추산 (커스텀 부품 경로, qty 3–5)

| 항목 | 금액(USD) |
|------|----------|
| MAX86141 × 5 | ~$48 |
| LSM6DSOTR × 5 | ~$19 (또는 대체품) |
| BG95-M2 × 3 | 조회 필요 (추산 $40–110) |
| MDBT50Q-1MV2 × 3 + 배송 | ~$43 (배송 포함) |
| MCP73831T × 5 | ~$4 |
| LiPo 400mAh × 3 | ~$21 |
| e-ink 모듈 × 2 | ~$27–$30 |
| 패시브 부품 | ~$10 |
| **소계** | **$212–$285** |

---

### E-2. 평가보드 경로 — Dev Board Combo 조달

| 보드 | 부품번호 | 단가(USD) | 재고 | 출처 | 리드타임 | 조회일 |
|------|---------|----------|------|------|---------|--------|
| Nordic nRF52840 DK | NRF52840-DK | $48.95 | 661개 재고 | [Digi-Key 1490-1072-ND](https://www.digikey.com/en/products/detail/nordic-semiconductor-asa/NRF52840-DK/8593726) | 16주 (공장), 재고분 즉시 | 2026-05-18 |
| Quectel BG95 EVB (QuecPython) | BG95M3-QPYTHON-EVB | $156.95 | 미재고 (주문 시 제조사 발주) | [Digi-Key 2958-BG95M3-QPYTHON-EVB-ND](https://www.digikey.com/en/products/detail/quectel/BG95M3-QPYTHON-EVB/24349311) | 3주 | 2026-05-18 |
| MAX86140/41 Eval System | MAX86140EVSYS# | $158.21 | 13개 재고 (공장 24개) | [Digi-Key MAX86140EVSYS#-ND](https://www.digikey.com/en/products/detail/analog-devices-inc-maxim-integrated/MAX86140EVSYS/7597515) | 10주 (공장), 재고분 즉시 | 2026-05-18 |

> **평가보드 소계 (1세트)**: $48.95 + $156.95 + $158.21 = **$364.11**  
> PoC 2세트 구성 시 약 **$728**

#### 평가보드 경로 참고 사항

- **BG95M3-QPYTHON-EVB**: BG95-M3 탑재 (M2 미탑재). BG95-M2와 핀/AT커맨드 호환이므로 검증 용도로 사용 가능. Quectel 공식 QuecPython 개발 환경 지원.
- **MAX86140EVSYS**: MAX86140/MAX86141 양쪽 평가 가능. USB-UART 브릿지 내장, 공식 GUI 소프트웨어 제공.
- **nRF52840 DK**: Nordic 공식 SDK(nRF5 SDK/Zephyr RTOS) 전 기능 지원. SWD 디버거 내장.
- **Waveshare e-ink용 별도 평가환경**: 위 nRF52840 DK에서 직접 SPI 연결 가능 (별도 eval 보드 불필요).

---

### E-3. e-ink 1.5" 모듈 소량 구매 옵션

> **주의**: 정확한 1.5" 240×240 흑백 e-ink 모듈은 상용 유통 제품 존재하지 않음. 가장 근접한 대체 규격: **1.54" 200×200** (대각선 1.54인치, 해상도 200×200, 184ppi).

#### 구매 옵션 비교

| 공급처 | 모델명 | 해상도 | 단가(USD) | MOQ | 배송 | 출처 | 조회일 |
|--------|--------|--------|----------|-----|------|------|--------|
| Waveshare 공식 | 1.54inch e-Paper Module (B&W) | 200×200 | ~$13.29–$14.99 | 1개 | DHL/우편 ($5–15) | [waveshare.com](https://www.waveshare.com/1.54inch-e-paper-module.htm) | 2026-05-18 |
| Amazon (Waveshare) | Waveshare 1.54" V2 B0728BJTZC | 200×200 | 조회 필요 (이전 $18 수준) | 1개 | Prime (미국) | [Amazon](https://www.amazon.com/dp/B0728BJTZC) | 2026-05-18 |
| Good Display | GDEY0154D67 (GDEH0154D67 후속) | 200×200 | 조회 필요 ($0 표시, 직접 문의) | 미정 | DHL/우편 | [good-display.com](https://www.good-display.com/product/1.54-inch-e-paper-display-module-partial-refresh-E-ink-screen-panel,-GDEH0154D67-208.html) | 2026-05-18 |
| buy-lcd.com | GDEW0154M09 + 어댑터 보드 | 200×200 | 조회 필요 | 2개 | DHL | [buy-lcd.com](https://www.buy-lcd.com/products/154-inch-e-paper-display-high-resolution-200x200-partial-refresh-fast-speed-with-adapter-board-gdew0154m09despi-c02) | 2026-05-18 |
| LCSC / JLCPCB | Waveshare 1.54" (C359939) | 200×200 | 조회 필요 | 조회 필요 | EMS/DHL | [JLCPCB](https://jlcpcb.com/partdetail/Waveshare-1_54inch_e_PaperModule/C359939) | 2026-05-18 |

> **권장**: PoC 소량(2–5개)은 Waveshare 공식 또는 Amazon 경유가 가장 빠르고 확실. GDEW0154M09는 EOL(단종) 확인됨 — GDEY0154D67 또는 Waveshare 현행 모델 사용.

#### e-ink 1.54" 기술 비교

| 항목 | Waveshare 1.54" | Good Display GDEY0154D67 |
|------|----------------|-------------------------|
| 해상도 | 200×200 | 200×200 |
| 인터페이스 | SPI | SPI |
| 부분 리프레시 | 지원 (V2) | 지원 |
| 컨트롤러 | UC8151 / IL3897 | SSD1681 |
| 모듈(PCB 포함) | 있음 | 모듈 버전 있음 |
| 소량 구매 | 용이 (1개부터) | 문의 필요 |

---

### 검토 필요 항목

| # | 항목 | 내용 | 우선순위 |
|---|------|------|---------|
| 1 | **BG95-M2 확정 단가** | Mouser/Digi-Key 직접 접속 또는 Quectel 영업팀 견적 요청 필요. 추산 $13–37/개 | 높음 |
| 2 | **LSM6DSOTR 대체품 선정** | 24주 리드타임 — LSM6DSOXTR 또는 LSM6DS3TR-C로 대체 여부 결정 | 높음 |
| 3 | **LiPo 380mAh 정확한 셀** | EEMB/PKCELL 직발주 시 MOQ 50개 이상, 샘플 요청 가능. PoC는 Adafruit 400mAh 대체 권장 | 중간 |
| 4 | **Good Display 가격 확인** | GDEY0154D67 직접 문의 (info@good-display.com). 웹 $0 표시는 문의 견적 방식 | 낮음 |
| 5 | **JLCPCB e-ink 파트 가격** | C359939 JLCPCB 내 단가 및 재고 직접 확인 필요 | 낮음 |
| 6 | **MDBT50Q-1MV2 배송비 $25** | Digi-Key Marketplace 제품으로 별도 $25 고정 배송비. 2개 이상 구매 시 단가 합리화 | 중간 |
| 7 | **nRF52840 IC 직접 구매 여부** | 모듈(MDBT50Q) vs 베어 IC(Nordic nRF52840) 선택. PoC에서는 모듈 권장 | 낮음 |

---

### 총 예산 범위 요약

| 시나리오 | 예상 총액(USD) | 비고 |
|---------|--------------|------|
| **커스텀 부품 경로** (qty 3–5, 배송 포함) | $250–$350 | BG95-M2 확정 단가 포함 시 |
| **평가보드 경로** (2세트) | $750–$900 | 보드 $728 + 개별 부품 + 배송 |
| **혼합 경로** (평가보드 1세트 + 커스텀 부품) | $600–$750 | 권장 PoC 구성 |

> 모든 가격은 USD 기준, 관세 및 국내 배송비 미포함. 한국 반입 시 관세·부가세 별도 발생.
