## A. 부품 조사 및 데이터시트 수집

> **작성일**: 2026-05-18  
> **목적**: SilverCare Watch PoC — 손목 PPG 심정지 감지 + LTE-M 알림 시스템 부품 검증  
> **주의**: 본 문서는 공개된 데이터시트, 애플리케이션 노트, 제조사 공식 문서를 기반으로 작성되었습니다.  
> 미확인 또는 원본 PDF에서 직접 추출하지 못한 수치는 명시적으로 표시하였습니다.

---

### A-1: MAX86141 (Analog Devices/Maxim — PPG AFE) 레지스터 요약

**개요**: MAX86141은 최대 2채널 광학 데이터 수집 시스템(PPG AFE)이며, 19-bit ADC, 128-word FIFO, SPI 인터페이스를 내장합니다. 공급전압: VDD_ANA 1.8V / VBATT(LED) 3.1V~5.5V.

#### A-1-1. LED 전류 설정 레지스터

| 주소 (hex) | 레지스터명 | 비트 필드 | 기능 설명 |
|-----------|-----------|----------|----------|
| 0x23 | LED1_DRV (LED1_PA) | [7:0] | LED1 펄스 진폭 설정. 8-bit 값, LED 진폭 범위(REG_LED_RANGE, 0x2A)에 따라 전류 스케일 결정 |
| 0x24 | LED2_DRV (LED2_PA) | [7:0] | LED2 펄스 진폭 설정 (동일 구조) |
| 0x25 | LED3_DRV (LED3_PA) | [7:0] | LED3 펄스 진폭 설정 |
| 0x26 | LED4_DRV (LED4_PA) | [7:0] | LED4 (외부 MUX 사용 시) |
| 0x27 | LED5_DRV (LED5_PA) | [7:0] | LED5 |
| 0x28 | LED6_DRV (LED6_PA) | [7:0] | LED6 |
| 0x29 | LED_PILOT_PA | [7:0] | 파일럿 LED 전류 설정 |
| 0x2A | LED1_RGE / LED2_RGE | [1:0] / [3:2] | LED 최대 전류 범위: 00=31mA, 01=62mA, 10=93mA, 11=124mA |

> **전류 계산**: `I_LED = (LEDx_PA / 255) × LED_RGE_MAX`  
> 예: LED1_PA=0x20 (32d), RGE=10b (93mA) → 약 11.6mA

#### A-1-2. 샘플레이트 / 평균 설정 레지스터

| 주소 (hex) | 레지스터명 | 비트 필드 | 값/기능 |
|-----------|-----------|----------|--------|
| 0x11 | PPG_CONFIG_1 | [1:0] PPG_TINT | 적분 시간: 00=14.8μs, 01=29.4μs, 10=58.7μs, 11=117.3μs |
| 0x11 | PPG_CONFIG_1 | [3:2] PPG1_ADC_RGE | ADC 풀스케일: 00=4.0μA, 01=8.0μA, 10=16μA, 11=32μA |
| 0x11 | PPG_CONFIG_1 | [5:4] PPG2_ADC_RGE | (MAX86141 2채널용, 동일 인코딩) |
| 0x12 | PPG_CONFIG_2 | [2:0] SMP_AVE | 평균 샘플 수: 000=1, 001=2, 010=4, 011=8, 100=16, 101=32, 110=64, 111=128 |
| 0x12 | PPG_CONFIG_2 | [7:3] PPG_SR | 샘플레이트 선택 (5-bit, 표 참조) |

**PPG_SR 주요 값 (register 0x12 bits[7:3]):**

| PPG_SR 값 | 샘플레이트 (SPS) | 노출/샘플 수 |
|----------|----------------|------------|
| 0x00 | 25 | 1 |
| 0x01 | 50 | 1 |
| 0x02 | 84 | 1 |
| 0x03 | 100 | 1 |
| 0x04 | 200 | 1 |
| 0x05 | 400 | 1 |
| 0x06 | 25 | 2 |
| 0x07 | 50 | 2 |
| 0x08 | 84 | 2 |
| 0x09 | 100 | 2 |
| 0x0A~0x13 | 8~4096 | 1 |

> **참고**: 선택한 PPG_TINT 및 노출 횟수와 호환되지 않는 SR 값을 설정하면, 디바이스가 지원 가능한 최대 SR로 자동 조정됨. 레지스터를 읽어 실제 SR을 확인 권장.

| 주소 (hex) | 레지스터명 | 비트 필드 | 기능 |
|-----------|-----------|----------|------|
| 0x13 | PPG_CONFIG_3 | [7:6] LED_SETLNG | LED 세틀링 시간: 00=0μs, 01=4μs, 10=8μs, 11=12μs |

#### A-1-3. FIFO 설정 레지스터

| 주소 (hex) | 레지스터명 | 비트 필드 | 기능 |
|-----------|-----------|----------|------|
| 0x04 | FIFO_WR_PTR | [6:0] | FIFO 쓰기 포인터 (읽기 전용) |
| 0x05 | FIFO_RD_PTR | [6:0] | FIFO 읽기 포인터 (쓰기 가능, 포인터 리셋 용) |
| 0x06 | OVF_COUNTER | [6:0] | 오버플로우 카운터 |
| 0x07 | FIFO_DATA_COUNT | [7:0] | FIFO에 저장된 샘플 수 |
| 0x08 | FIFO_DATA | [23:0] | FIFO 데이터 출력 레지스터 (SPI 버스트 읽기) |
| 0x09 | FIFO_CONFIG_1 | [6:0] FIFO_A_FULL | FIFO 인터럽트 임계값 (나머지 빈 슬롯 수): 0=128, 1=127, ... |
| 0x0A | FIFO_CONFIG_2 | [1] FIFO_RO | 롤오버 허용: 1=롤오버 허용 (오래된 데이터 덮어씀), 0=정지 |

#### A-1-4. PPG 모드 / 시퀀스 설정 레지스터

| 주소 (hex) | 레지스터명 | 비트 필드 | 기능 |
|-----------|-----------|----------|------|
| 0x20 | LED_SEQ_1 | [3:0] LEDC1, [7:4] LEDC2 | 시간 슬롯 1,2의 LED 소스 선택 |
| 0x21 | LED_SEQ_2 | [3:0] LEDC3, [7:4] LEDC4 | 시간 슬롯 3,4 |
| 0x22 | LED_SEQ_3 | [3:0] LEDC5, [7:4] LEDC6 | 시간 슬롯 5,6 |
| 0x0D | SYSTEM_CTRL | [0] RESET | 소프트 리셋 (1 = 리셋 실행) |
| 0x0D | SYSTEM_CTRL | [1] SHDN | 셧다운: 1=저전력 셧다운 모드, 0=정상 동작 |

**LEDC 비트 인코딩 (PPG 모드 선택):**

| LEDC 값 | 슬롯 기능 |
|--------|---------|
| 0x0 | 없음 (비활성) |
| 0x1 | LED1 (Green — 심박수 HR 모드) |
| 0x2 | LED2 (Red — SpO2 용) |
| 0x3 | LED3 (IR — SpO2 용) |
| 0x9 | 직접 주변광 측정 (Ambient) |

> **SpO2 구성 예**: LEDC1=LED2(Red), LEDC2=LED3(IR), Ambient 슬롯 추가  
> **HR 구성 예**: LEDC1=LED1(Green) 만 활성화

**소비 전류 (데이터시트 Typical 값):**

| 동작 모드 | 전류 (Typ.) |
|---------|-----------|
| PPG 활성 25 SPS (단일 LED) | ~8.5 μA (LEDs off 시) |
| PPG 활성 100 SPS | ~32 μA |
| PPG 활성 256 SPS | ~80 μA |
| 셧다운 (SHDN=1) | 0.6 μA (Typ.) |

#### 출처 (A-1)

- Analog Devices 공식 데이터시트: https://www.analog.com/media/en/technical-documentation/data-sheets/max86140-max86141.pdf
- GitHub Arduino 라이브러리 (레지스터 맵 참조): https://github.com/moothyknight/MAX86141_Arduino/blob/master/libraries/MAX86141/MAX86141.h
- Nordic DevZone SPI 연동 예제: https://devzone.nordicsemi.com/f/nordic-q-a/109399/max86141-spi-communication-with-nrf5340
- Analog Devices 제품 페이지: https://www.analog.com/en/products/max86141.html

---

### A-2: LSM6DSO (STMicroelectronics IMU) — 낙상/무활동 관련 파라미터

**개요**: LSM6DSO는 Always-On 3축 가속도계 + 3축 자이로스코프 IMU. I2C/SPI 인터페이스, 내장 낙상감지·웨이크업·활동/비활동 인터럽트 엔진 탑재. 공급전압: 1.71V~3.6V.

#### A-2-1. 가속도계 풀스케일 및 ODR

| 레지스터 주소 | 레지스터명 | 비트 필드 | 값 / 기능 |
|------------|----------|----------|---------|
| 0x10 | CTRL1_XL | [7:4] ODR_XL[3:0] | 출력 데이터레이트 선택 (4-bit, 표 참조) |
| 0x10 | CTRL1_XL | [3:2] FS_XL[1:0] | 풀스케일 가속도 범위 선택 |
| 0x10 | CTRL1_XL | [1] LPF2_XL_EN | 2차 로우패스 필터 활성화 |

**FS_XL 인코딩:**

| FS_XL[1:0] | 풀스케일 범위 |
|-----------|------------|
| 00 | ±2g |
| 10 | ±4g |
| 11 | ±8g |
| 01 | ±16g |

**ODR_XL 인코딩 (주요 값):**

| ODR_XL[3:0] | 출력 데이터레이트 | 비고 |
|------------|----------------|------|
| 0000 | Power-down (0Hz) | 가속도계 비활성화 |
| 0001 | 12.5 Hz | 저전력 |
| 0010 | 26 Hz | |
| 0011 | 52 Hz | |
| 0100 | 104 Hz | |
| 0101 | 208 Hz | |
| 0110 | 416 Hz | 낙상/웨이크업 감지 권장 ODR |
| 0111 | 833 Hz | |
| 1000 | 1.66 kHz | |
| 1001 | 3.33 kHz | |
| 1010 | 6.66 kHz | |

#### A-2-2. 낙상 감지 레지스터 (FREE_FALL)

| 레지스터 주소 | 레지스터명 | 비트 필드 | 값 / 기능 |
|------------|----------|----------|---------|
| 0x5D | FREE_FALL | [2:0] FF_THS[2:0] | 낙상 임계값 (3-bit, 아래 표 참조) |
| 0x5D | FREE_FALL | [7:3] FF_DUR[4:0] | 낙상 지속시간 (단위: ODR 주기, 최소 인식 기간) |

**FF_THS 임계값 인코딩:**

| FF_THS[2:0] | 임계값 |
|-----------|-------|
| 000 | 156 mg |
| 001 | 219 mg |
| 010 | 250 mg |
| 011 | 312 mg (ST 예제 기본값: `LSM6DSO_FF_TSH_312mg`) |
| 100 | 344 mg |
| 101 | 406 mg |
| 110 | 469 mg |
| 111 | 미확인 — 원본 데이터시트 확인 필요 |

> ST 공식 낙상 감지 예제 설정: ODR=416Hz, FS=±2g, FF_THS=312mg, FF_DUR=6 (약 14.4ms @ 416Hz)

#### A-2-3. 웨이크업 / 활동-비활동 인터럽트 레지스터

| 레지스터 주소 | 레지스터명 | 비트 필드 | 값 / 기능 |
|------------|----------|----------|---------|
| 0x5B | WAKE_UP_THS | [5:0] WK_THS[5:0] | 웨이크업 임계값 (6-bit). 1 LSB = FS_XL / 64. 기본값: 0x02 |
| 0x5B | WAKE_UP_THS | [6] USR_OFF_ON_WU | 사용자 오프셋을 웨이크업 계산에 포함 여부 |
| 0x5B | WAKE_UP_THS | [7] SINGLE_DOUBLE_TAP | 싱글/더블 탭 인터럽트 선택 |
| 0x5C | WAKE_UP_DUR | [3:0] WAKE_DUR[1:0] / FF_DUR [7:5] | 웨이크업 지속시간 및 낙상 FF_DUR 상위 비트 |
| 0x56 | TAP_CFG0 | [7] INT_CLR_ON_READ | 인터럽트 클리어 방식 |
| 0x56 | TAP_CFG0 | [3] SLOPE_FDS | 경사 필터 활성화 (웨이크업용) |
| 0x58 | TAP_CFG2 | [7] INTERRUPTS_ENABLE | 기능 인터럽트 전역 활성화 |
| 0x5E | MD1_CFG | [3] INT1_FF | 낙상 이벤트 → INT1 핀 라우팅 |
| 0x5E | MD1_CFG | [5] INT1_WU | 웨이크업 이벤트 → INT1 핀 라우팅 |
| 0x5F | MD2_CFG | [3] INT2_FF | 낙상 이벤트 → INT2 핀 라우팅 |
| 0x5F | MD2_CFG | [5] INT2_WU | 웨이크업 이벤트 → INT2 핀 라우팅 |

> **무활동(No-Motion) 감지**: LSM6DSO는 별도의 activity/inactivity 엔진을 가짐. WAKE_UP_THS 동일 레지스터로 모션 감지 → 무모션 시 저전력 모드 자동 전환 가능. 상세 설정은 AN5192 애플리케이션 노트 참조.

**소비 전류:**

| 모드 | 전류 (Typ.) |
|-----|-----------|
| 가속도계 + 자이로 고성능 콤보 | 0.55 mA |
| 가속도계 단독 저전력 12.5Hz | ~25 μA |
| 파워다운 | <5 μA |

#### 출처 (A-2)

- ST 공식 데이터시트: https://www.st.com/resource/en/datasheet/lsm6dso.pdf
- ST 애플리케이션 노트 AN5192: https://www.st.com/resource/en/application_note/an5192-lsm6dso-alwayson-3axis-accelerometer-and-3axis-gyroscope-stmicroelectronics.pdf
- ST 공식 C 드라이버 (레지스터 헤더): https://github.com/stm32duino/LSM6DSO/blob/main/src/lsm6dso_reg.h
- ST 낙상 감지 예제: https://github.com/STMicroelectronics/STMems_Standard_C_drivers/blob/master/lsm6dso_STdC/examples/lsm6dso_free_fall.c
- DeepWiki 웨이크업 감지: https://deepwiki.com/stm32duino/LSM6DSO/4.2-wake-up-detection

---

### A-3: Quectel BG95-M2 AT 명령 참조 — LTE-M 접속 / PSM / SMS

**개요**: BG95-M2는 LTE Cat-M1 / Cat-NB2 / EGPRS 멀티모드 모듈. 공급전압(VBAT): 2.6V~4.8V (정격 3.3V Typ.).

#### A-3-1. 주요 AT 명령 일람

| AT 명령 | 기능 | 주요 구문 | 비고 |
|--------|------|---------|------|
| `AT+CFUN` | 모뎀 기능 전환 | `AT+CFUN=<fun>` | 0=최소기능, 1=전체기능, 4=비행모드 |
| `AT+CEREG` | EPS 네트워크 등록 상태 | `AT+CEREG=<n>` | 0=URC 비활성, 1=URC활성, 2=위치정보포함URC |
| `AT+CPSMS` | PSM 설정 (3GPP 표준) | 구문 참조 (아래) | |
| `AT+QPSMS` | PSM 설정 (Quectel 확장) | 구문 참조 (아래) | |
| `AT+QCFG="psm/enter"` | 즉시 PSM 진입 | `AT+QCFG="psm/enter"` | |
| `AT+QCFG="psm/urc",1` | PSM 진입 URC 알림 활성화 | | |
| `AT+CMGF` | SMS 메시지 포맷 | `AT+CMGF=<mode>` | 0=PDU 모드, 1=텍스트 모드 |
| `AT+CMGS` | SMS 전송 | 구문 참조 (아래) | |
| `AT+QNWINFO` | 네트워크 정보 조회 | `AT+QNWINFO` | 예: +QNWINFO: "eMTC","20408","LTE BAND 20",6400 |

#### A-3-2. 상세 명령 구문

**LTE-M 네트워크 접속 절차:**
```
AT+CFUN=1                    // 모뎀 전체 기능 활성화
AT+CEREG=2                   // 네트워크 등록 URC 활성화 (위치정보 포함)
AT+CEREG?                    // 현재 등록 상태 쿼리
// 응답: +CEREG: 2,1,"XXXX","XXXXXXXX",9  → 9=eMTC(LTE-M) 등록 완료
```

**PSM 설정 (AT+CPSMS — 3GPP TS 27.007 표준):**
```
AT+CPSMS=<mode>[,<Requested_Periodic-RAU>[,<Requested_GPRS-READY-timer>
          [,<Requested_Periodic-TAU>[,<Requested_Active-Time>]]]]

// 파라미터:
// <mode>:   0=비활성화, 1=활성화, 2=비활성화+파라미터 삭제
// <Requested_Periodic-TAU>: T3412 타이머 (8-bit 이진 문자열)
// <Requested_Active-Time>:  T3324 타이머 (8-bit 이진 문자열)

// 타이머 인코딩 (8-bit 이진 문자열):
// 비트[7:5]: 단위 선택
//   000 = 10분 단위 (T3412용)
//   001 = 1시간 단위
//   010 = 10시간 단위
//   101 = 1분 단위 (T3324용)
//   110 = 2초 단위
// 비트[4:0]: 타이머 값 (2진수)

// 예: PSM 활성화, TAU=1시간(001 00001), Active=2분(101 00010)
AT+CPSMS=1,,,"00100001","00101010"
```

**PSM 설정 (AT+QPSMS — Quectel 확장):**
```
AT+QPSMS=<mode>[,"<Requested_Periodic-TAU>"[,"<Requested_Active-Time>"]]

// 예: PSM 활성화, TAU=4분, Active=15분
AT+QPSMS=1,"00000100","00001111"

AT+QPSMS?      // 현재 PSM 설정 조회
AT+QPSMS=0     // PSM 비활성화
```

**SMS 전송:**
```
// 1단계: 텍스트 모드 설정
AT+CMGF=1

// 2단계: SMS 전송 (텍스트 모드)
AT+CMGS="+821012345678"
> [메시지 내용 입력]
[CTRL+Z]  // 전송 완료 (ASCII 26)

// 응답: +CMGS: <mr>  → mr=메시지 참조 번호
// OK
```

**중요 참고사항:**
- PSM은 네트워크 지원 필요 (LTE-M 네트워크에서 PSM negotiation 진행)
- PSM 진입 중에는 SMS 수신 불가. 중요 수신이 필요한 경우 eDRX 병용 고려
- BG95-M2 펌웨어가 PSM을 지원하는 버전인지 확인 필요 (구형 펌웨어 미지원 사례 있음)

#### 출처 (A-3)

- Quectel BG95 AT Commands Manual V2.0: https://sixfab.com/wp-content/uploads/2023/05/Quectel_BG95BG77BG600L_Series_AT_Commands_Manual_V2.0.pdf
- Quectel QCFG AT Commands Manual: https://sixfab.com/wp-content/uploads/2023/05/Quectel_BG95BG77BG600L_Series_QCFG_AT_Commands_Manual_V2.0.pdf
- IoT Creators BG95 문서: https://docs.iotcreators.com/docs/quectel-bg95-at-commands
- Quectel 포럼 PSM 예제: https://forums.quectel.com/t/bg95-m3-cant-enter-psm-mode/7455

---

### A-4: Nordic nRF52840 DK (PCA10056) 핀 맵

**개요**: nRF52840 DK(PCA10056)는 Arduino Uno R3 호환 헤더(P3~P6)와 확장 GPIO 헤더를 제공합니다. SPI는 4개 인스턴스(SPI0~SPI3), I2C(TWI)는 2개 인스턴스(I2C0/I2C1) 사용 가능. 모든 GPIO 핀에 SPI/I2C 기능 배정 가능(유연한 핀 매핑).

#### A-4-1. 온보드 고정 핀 (충돌 주의)

| 기능 | GPIO 핀 | 비고 |
|-----|---------|------|
| LED1 (green) | P0.13 | 사용 불가 (LED 고정) |
| LED2 (green) | P0.14 | 사용 불가 |
| LED3 (green) | P0.15 | 사용 불가 |
| LED4 (green) | P0.16 | 사용 불가 |
| BUTTON1 (SW1) | P0.11 | 사용 불가 (버튼 고정) |
| BUTTON2 (SW2) | P0.12 | 사용 불가 |
| BUTTON3 (SW3) | P0.24 | 사용 불가 |
| BUTTON4 (SW4) | P0.25 | 사용 불가 |
| UART0 TX | P0.06 | 인터페이스 MCU 연결 (디버그 UART) |
| UART0 RX | P0.08 | 인터페이스 MCU 연결 |
| UART0 RTS | P0.05 | 인터페이스 MCU 연결 |
| UART0 CTS | P0.07 | 인터페이스 MCU 연결 |
| 32.768kHz XTAL | P0.00, P0.01 | 커넥터에 노출 안 됨, GPIO 사용 불가 |

#### A-4-2. SPI 인스턴스별 기본 핀 배정 (Zephyr pinctrl 기준)

| SPI 인스턴스 | MOSI | MISO | SCK | 용도 |
|------------|------|------|-----|-----|
| SPI0 | P0.26 | P0.29 | P0.27 | ※ I2C0와 핀 공유 → 동시 사용 불가 |
| SPI1 | P0.30 | P1.08 | P0.31 | ※ I2C1과 핀 공유 → 동시 사용 불가 |
| SPI2 | P0.20 | P0.21 | P0.19 | 독립 사용 가능 |
| SPI3 (Arduino) | P1.13 | P1.14 | P1.15 | Arduino ICSP 헤더 (D11/D12/D13) |

> **외부 SPI 센서 (MAX86141) 권장 인스턴스**: SPI2 (P0.19/P0.20/P0.21) 또는 SPI3 (P1.13/P1.14/P1.15 — Arduino 헤더)  
> CS 핀은 임의의 빈 GPIO 배정 가능 (예: P0.03, P0.04)

#### A-4-3. I2C 인스턴스별 기본 핀 배정 (Zephyr pinctrl 기준)

| I2C 인스턴스 | SDA | SCL | 용도 |
|------------|-----|-----|-----|
| I2C0 (Arduino) | P0.26 | P0.27 | Arduino 헤더 A4/A5 위치 |
| I2C1 | P0.30 | P0.31 | ※ SPI1과 핀 공유 → 동시 사용 불가 |

> **외부 I2C 센서 (LSM6DSO) 권장**: I2C0 (P0.26 SDA, P0.27 SCL)  
> **주의**: I2C0 사용 시 SPI0 동시 활성화 불가

#### A-4-4. 전형적인 PoC 핀 배정 제안

| 신호 | GPIO 핀 | 연결 부품 |
|-----|---------|---------|
| SPI SCK (MAX86141) | P1.15 (SPI3) | MAX86141 SCLK |
| SPI MOSI (MAX86141) | P1.13 (SPI3) | MAX86141 MOSI |
| SPI MISO (MAX86141) | P1.14 (SPI3) | MAX86141 MISO |
| SPI CS (MAX86141) | P0.04 (임의 GPIO) | MAX86141 /CS |
| I2C SDA (LSM6DSO) | P0.26 (I2C0) | LSM6DSO SDA |
| I2C SCL (LSM6DSO) | P0.27 (I2C0) | LSM6DSO SCL |
| INT (MAX86141 인터럽트) | P0.03 (임의 GPIO) | MAX86141 INT |
| INT1 (LSM6DSO 낙상 인터럽트) | P0.28 (임의 GPIO) | LSM6DSO INT1 |
| UART TX → BG95-M2 | P1.02 (UART1) | BG95-M2 RXD |
| UART RX ← BG95-M2 | P1.01 (UART1) | BG95-M2 TXD |

> **P0.26/P0.27는 "Low Frequency I/O" 핀**: I2C(400kHz 이하)에는 사용 가능, SPI(수 MHz)에는 사용 불가.  
> **저주파 I/O 핀 목록**: P0.00~P0.03 및 일부 — 원본 Product Specification Table 참조 필요

#### 출처 (A-4)

- Zephyr DTS (nRF52840DK 핀 배정): https://github.com/zephyrproject-rtos/zephyr/blob/main/boards/nordic/nrf52840dk/nrf52840dk_nrf52840.dts
- Zephyr Pinctrl DTSI (상세 핀): https://github.com/zephyrproject-rtos/zephyr/blob/main/boards/nordic/nrf52840dk/nrf52840dk_nrf52840-pinctrl.dtsi
- TinyGo PCA10056 핀 정의: https://tinygo.org/docs/reference/microcontrollers/machine/pca10056/
- Zephyr nRF52840DK 문서: https://docs.zephyrproject.org/latest/boards/nordic/nrf52840dk/doc/index.html
- Nordic DevZone SPI/I2C 핀 논의: https://devzone.nordicsemi.com/f/nordic-q-a/75110/nrf52840-spi-and-i2c-pin-mapping

---

### A-5: 전력 예산 이론 시뮬레이션 — 380mAh LiPo + MCP73831

> **면책 고지**: 이 표는 **이론적 추정치**입니다. 실제 전력 측정은 Phase 4 (개발자 실측 단계)에서 수행되어야 합니다. 아래 수치는 각 부품 데이터시트의 Typical 값을 기반으로 하며, 시스템 통합 손실, PCB 기생 성분, 전압 레귤레이터 효율은 반영되지 않았습니다.

#### A-5-1. MCP73831 충전기 설계

**공식:**

```
I_CHARGE (mA) = 1000 / R_PROG (kΩ)
```

> 출처: Microchip MCP73831 데이터시트 DS20001984H

**PROG 저항값 vs 충전 전류:**

| R_PROG | 충전 전류 | 비고 |
|--------|---------|------|
| 2 kΩ | 500 mA | 380mAh 배터리 기준 약 1C 충전 |
| 2.7 kΩ | ~370 mA | ~0.97C (권장: 0.5~1C 범위) |
| 4 kΩ | 250 mA | ~0.66C (안전 충전 속도) |
| 10 kΩ | 100 mA | 저속 충전 (~0.26C) |
| 67 kΩ | ~15 mA | 최소 충전 전류 |

> **380mAh LiPo 권장**: R_PROG = 2.7kΩ (충전 전류 ~370mA ≈ 0.97C)  
> MCP73831 공급전압(VDD): 3.75V~6.0V (정격 5V USB)  
> 충전 종료 전류: 설정 충전 전류의 5%~20% (디폴트 ~7.5%)  
> 대기 전류 (충전 완료 후): <2μA (Typ.)

#### A-5-2. 모드별 이론적 전류 예산

**전제조건:**
- 배터리: 380mAh, 3.7V LiPo
- MCU: nRF52840 (1.8V/3.3V 동작)
- PPG AFE: MAX86141 (1.8V VDD, LED용 VBATT 3.3V)
- IMU: LSM6DSO (1.8V~3.3V)
- 모뎀: BG95-M2 (VBAT 3.3V~4.2V, 전형적 3.8V)

| 동작 모드 | 구성 부품 | 부품별 전류 (Typ.) | 합계 추정 | 출처 |
|---------|---------|-----------------|---------|------|
| **PPG 연속 감지** | nRF52840 (활성, BLE off) | ~1.5mA | | Nordic PS v1.11 |
| | MAX86141 (100 SPS, LED ~15mA@3.3V) | ~32μA (AFE) + LED전류 | | ADI MAX86141 DS |
| | LSM6DSO (가속도계 104Hz) | ~0.55mA | | ST LSM6DSO DS |
| | BG95-M2 (대기, PSM off) | ~3~9mA | | Quectel 포럼 |
| | **합계 (PPG+MCU+IMU, 모뎀 대기)** | | **~6~12 mA** | |
| **MCU 슬립** | nRF52840 System ON, RAM 유지 | ~2.35μA | | Nordic DevZone |
| | MAX86141 SHDN=1 | 0.6μA | | ADI MAX86141 DS |
| | LSM6DSO 파워다운 | <5μA | | ST LSM6DSO DS |
| | BG95-M2 PSM 모드 | ~3μA | | Quectel 사양서 |
| | **합계 (딥슬립)** | | **~11μA** | |
| **LTE-M 활성 TX** | nRF52840 (활성) | ~1.5mA | | Nordic PS v1.11 |
| | BG95-M2 Cat-M1 TX 평균 | 미확인 — 원본 데이터시트 확인 필요 | | Quectel HW Guide Table 45 접근 실패 |
| | BG95-M2 Cat-M1 TX 피크 | 미확인 — 원본 데이터시트 확인 필요 | | Quectel HW Guide Table 45 접근 실패 |
| | MAX86141 SHDN 중 | 0.6μA | | |
| | **합계 (TX 활성, 부분 추정)** | | **미확인 포함** | |

> **BG95-M2 TX 전류 참고 (미확인 추정)**: Quectel 유사 모듈(BG96 LTE-M) 기준 평균 TX ≈ 160~180mA @ 3.3V, 피크 ≈ 490mA 수준이나, BG95-M2 원본 사양서 Table 45 직접 확인 필요.

#### A-5-3. 이론 배터리 수명 추정

**사용 시나리오 가정 (심장 모니터링 모드):**

| 구간 | 시간 비율 | 예상 전류 |
|-----|---------|---------|
| PPG 연속 감지 | 10% (6분/시간) | ~12mA |
| MCU/센서 딥슬립 | 85% (51분/시간) | ~11μA |
| LTE-M TX (알림) | 5% (3분/시간, 미확인) | 미확인 |

**딥슬립 + PPG만 고려한 간략 추정:**
```
평균 전류 ≈ (0.10 × 12mA) + (0.85 × 0.011mA)
           = 1.2mA + 0.0094mA ≈ 1.21mA

이론 수명 ≈ 380mAh / 1.21mA ≈ 314시간 ≈ 약 13일
```

> **중요**: LTE-M TX 전류 미포함 추정치. 실제 알림 빈도 및 TX 전류 포함 시 수명 단축됨. Phase 4 실측 후 재계산 필요.

#### A-5-4. MCP73831 주요 전기적 특성

| 파라미터 | 값 | 출처 |
|--------|---|------|
| 입력 전압 범위 (VDD) | 3.75V ~ 6.0V | Microchip DS20001984H |
| 최대 충전 전류 | 500mA | 공식: IREG = 1000 / RPROG(kΩ) |
| 대기 전류 (입력 전원 없음) | <2μA | Microchip 데이터시트 |
| 충전 종료 전류 | 충전 전류의 5%~20% | DS20001984H |
| 배터리 충전 전압 | 4.20V ±0.75% | DS20001984H |

#### 출처 (A-5)

- Microchip MCP73831 데이터시트: https://ww1.microchip.com/downloads/en/DeviceDoc/MCP73831-Family-Data-Sheet-DS20001984H.pdf
- MCP73831 기술 정보: https://www.ultralibrarian.com/2021/11/04/mcp73831-datasheet-a-linear-charge-management-controller-ulc/
- nRF52840 슬립 전류 논의: https://devzone.nordicsemi.com/f/nordic-q-a/111667/nrf52840-power-consumption-with-sleep-off-mode-and-sleep-on-mode
- nRF52840 Product Specification v1.11: https://docs.nordicsemi.com/bundle/ps_nrf52840/page/keyfeatures_html5.html
- Quectel BG95 제품 페이지: https://www.quectel.com/product/lpwa-bg95-cat-m1-cat-nb2-egprs-series/

---

## 검토 필요 항목

아래 항목은 이번 조사에서 직접 확인되지 않아 **원본 데이터시트/공식 문서 직접 확인**이 필요합니다.

1. **MAX86141 PPG_SR 전체 테이블**: 0x0A~0x13 범위의 8~4096 SPS 전체 인코딩 표 — 원본 PDF p.XX (Table Register 0x12) 직접 확인 필요. 출처: [ADI DS](https://www.analog.com/media/en/technical-documentation/data-sheets/max86140-max86141.pdf)

2. **MAX86141 LED_RGE 레지스터 (0x2A, 0x2B)**: LED 최대 전류 범위 설정 상세 비트 인코딩 확인 필요 (31/62/93/124mA 선택 관련 전체 레지스터 구조)

3. **LSM6DSO FF_THS 인코딩 111b 값**: 7번째 임계값 (000~110은 확인, 111 미확인) — 원본 DS Table 87 확인 필요

4. **LSM6DSO ODR_XL 전체 인코딩 표**: 공식 데이터시트 Table 19 (CTRL1_XL 레지스터) — Mbed/Zephyr 헤더에서 구체적 비트값 미추출

5. **BG95-M2 LTE-M TX 전류 (peak/average)**: Quectel BG95 Series Hardware Design V1.5 Table 41/45의 Cat-M1 송신 전류 (평균 및 피크 mA) — PDF 접근 실패로 미확인. 출처: [Quectel HD V1.5](https://images.quectel.com/python/2023/04/Quectel_BG95_Series_Hardware_Design_V1.5.pdf)

6. **BG95-M2 PSM 슬립 전류 정확한 수치**: 포럼에서 "약 3μA" 언급되나 공식 사양서 표 직접 확인 필요

7. **nRF52840 System OFF 슬립 전류**: 포럼에서 2.35μA(System ON) 언급되나, Nordic PS v1.11 공식 Table 45 (Electrical Specifications) 직접 확인 필요

8. **nRF52840 Low Frequency I/O 핀 목록**: Product Specification의 정확한 LF-only 핀 목록 (P0.00~P0.xx 범위) — PoC 핀 선택 전 필수 확인

9. **MCP73831 공급 전류 (충전 중 자체 소비)**: 데이터시트에서 IQ (quiescent current during charging) 수치 직접 추출 필요

10. **BG95-M2 AT+QPSMS 타이머 이진 인코딩 전체 표**: PSM Application Note (BG95&BG77&BG600L_Series_PSM_Application_Note) 직접 접근 후 T3412/T3324 전체 단위 표 확인 필요
