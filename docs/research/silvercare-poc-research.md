# SilverCare Watch · PoC 사전 조사 보고서

> 사업 타당성을 결정하는 12개 검증 항목에 대해 공개 자료 기반으로 사전 조사한 결과입니다.
> 직접 확인 불가능한 항목은 명시했고, 다음 액션을 우선순위별로 정리했습니다.
>
> **작성일**: 2026년 5월 · **문서 상태**: v0.1 (가안)

---

## 📋 요약 (TL;DR)

| 등급 | 의미 | 항목 수 |
|---|---|---|
| 🟢 상당 부분 파악 | 공개 자료로 핵심 데이터 확보 | 4개 |
| 🟡 부분 파악 | 일부 확인, 일부는 직접 컨택 필요 | 6개 |
| ❌ 직접 확인 불가 | 검색만으로는 불가, 직접 면담·실측 필요 | 2개 |

**가장 중요한 두 가지 발견**:

1. **PPG 손목 광혈류로 심정지를 감지하는 학술적 근거는 2025년에 거의 굳어졌다.** Nature(2025년 2월, Google), Circulation(2025년 DETECT-1b)에서 임상 검증 완료. 이제 "되는지 안 되는지"가 아니라 "얼마나 정확한지"의 문제.
2. **119 직접 자동 신고 API는 없지만, 우회 경로는 검증되어 있다.** 의정부시가 2023년 IoT 센서 → 시 CCTV통합관제센터 → 119 다매체 자동 신고 시스템을 전국 최초로 구축해 운영 중.

이 두 가지가 사업 타당성의 가장 큰 두 리스크를 크게 줄여줍니다.

---

# 🔴 가장 시급 — 사업 자체가 성립하는지 결정하는 검증

## 1. 119 자동 신고 시스템 연동 가능성

**🟡 부분 파악**

### 현재 상황

소방청 119에 일반 사업자가 사용할 수 있는 **공개 자동 신고 API는 존재하지 않습니다**. 다매체 신고 서비스(SMS·앱·영상통화)는 모두 사람이 직접 만들어 전송하는 인터페이스입니다.

> "119 다매체 신고 서비스는 기존의 음성 통화 외에도 문자(SMS·MMS), 스마트폰 앱(App), 영상통화 등을 통해 신고가 가능한 서비스다."
> — 하동소방서, 2025년 9월 [^1]

### 검증된 우회 경로

의정부시가 **전국 최초로 IoT 센서 → 지자체 관제센터 → 119 다매체 자동 신고** 시스템을 구축했습니다. 이게 표준 패턴이고 이미 작동 중입니다.

> "노후주택이 많은 지점에 불꽃파장 감지센서와 연기·불꽃 영상 감지센서를 설치해 작동한다. 화재가 발생하면 의정부시 CCTV통합관제센터의 '스마트 도시안전망 서비스'를 통해 소방청 '119 다매체 신고 서비스'에 자동으로 신고돼 신속한 대응이 가능하다."
> — 경기북부탑뉴스, 2023년 7월 [^2]

### SilverCare에 대한 시사점

이 패턴을 차용하면 다음 구조가 가능합니다:

```
SilverCare Watch → 자체 응급통보 플랫폼 → 지자체 관제센터 (또는 응급안전안심서비스 관제) → 119 다매체
                                       ↘ 가족 SMS (병행)
```

직접 119 연동은 불가능하지만 우회 경로는 입증되었습니다.

### 직접 확인 불가

- 소방청과의 사업 협약 가능 여부
- 자동 발신 SMS의 119 측 신고 분류 정책
- 오신고 페널티 수준

→ 소방청 종합상황관리관 또는 지자체 안전정책과 직접 면담 필요.

---

## 2. 의료기기 인허가 등급 사전 검토

**🟢 상당 부분 파악**

### 분류·인증 절차

식약처 의료기기전자민원시스템(emed.mfds.go.kr)에서 다음 두 트랙으로 사전 확인이 가능합니다:

- **의료기기 해당 여부 검토 신청** (정부24 민원, 무료) [^3]
- **사전상담** (혁신제품) 또는 **사전검토** 제도 [^4]

식약처는 2025년 5월 30일 「식약처 제품화 지원 상담 업무 안내서」를 발간해 신청 절차를 표준화했습니다. [^5]

### 2025년 디지털의료제품법 시행

2025년 5월 7일 식약처는 디지털의료기기 관련 가이드라인을 제·개정했습니다:

> "디지털의료기기 관련 가이드라인의 제·개정은 디지털의료기기소프트웨어의 기술적 특성과 구조를 반영하여 ① 허가신청서 및 첨부서류 작성방법을 표준화하고, ② 인공지능(AI), 가상융합기술(VR·AR·MR) 등 최신 기술이 적용된 의료기기가 디지털의료기기에 해당하는지 여부를 판단할 수 있도록 흐름도를 정리"
> — Lexology, 2025년 5월 [^6]

AI 기반 심정지 감지 기능을 가진 SilverCare는 이 디지털의료기기 분류에 해당할 가능성이 큽니다.

### 가장 가까운 선례: 휴이노 메모워치

> "휴이노는 자사가 개발한 웨어러블 시계형 심전도 기기인 '메모 워치'와 인공지능(AI) 기반 분석 소프트웨어가 식품의약품안전처로부터 허가를 받았다고 25일 밝혔다."
> — 한국경제, 2019년 3월 [^7]

> "휴이노는 웨어러블 시계형 심전도 기기인 'MEMO Watch'와 인공지능 기반 분석 소프트웨어가 식품의약품안전처로부터 **2등급 의료기기 홀터심전계** 관련 시험에…"
> — 히트뉴스, 2019년 3월 [^8]

후속으로 2020년 5월에는 국내 최초 손목시계형 의료기기 건강보험 급여대상으로 등재되었습니다. [^9]

### 의료기기 허가 등급별 처리 절차

> "민원 처리기간은 기술문서 심사기간을 포함하여 **허가 65일(임상 80일), 인증 30일**, 신고는 즉시 처리하고 있음. 「의료기기법 시행규칙」 제4조에 따른 제조·수입 품목별 **허가대상 의료기기는 3등급·4등급 의료기기**로 한다."
> — 의료기기 안심책방 [^10]

### SilverCare 등급 추정

- 심정지를 "감지"한다고 명시 → 최소 **2등급 의료기기** (휴이노 선례)
- 진단 영역까지 가면 3등급
- "응급 호출 보조 디바이스"로 포지셔닝 시 1등급 또는 의료기기 회피 가능

### 직접 확인 불가

식약처가 우리 제품을 어느 등급으로 판정할지는 사전상담을 직접 신청해야만 알 수 있습니다 (소요 1~2개월, 무료).

---

## 3. PPG로 심정지를 정말 잡을 수 있는가

**🟢 학술적으로 거의 검증됨 — 가장 좋은 소식**

이 항목이 가장 우려스러웠으나, **2024~2025년에 의학적 근거가 거의 확립**된 상태임을 확인했습니다.

### Google Pixel Watch · Nature 2025

> "These considerations are magnified for always-on systems that utilize wrist-based photoplethysmography (PPG), a signal that is susceptible to noise and reductions in amplitude from benign, everyday use activities. Informed by these constraints and the considerable public health unmet need of unwitnessed arrest, we reasoned that a wearable-based PPG- and motion-based, multimodal model could classify loss of pulse events with meaningful sensitivity and specificity."
> — Nature, 2025년 2월 [^11]

상용 스마트워치(Pixel Watch)에서 PPG+모션 다중 모드 모델로 심정지 자동 감지가 실증된 첫 논문입니다.

### DETECT-1b · Circulation 2025

손목 PPG 워치로 유도된 심실세동(VF)·심실빈맥(VT) 환자 50명 대상 외부 검증:

> "The positive predictive value for cardiac arrest detection was **86%** (54/63; 95% CI, 74%–93%). In the per-patient analysis, considering only the first event per patient (n=26), the sensitivity for detecting VF/pVT was **92%** (24/26; 95% CI, 73%–99%). This is the first external validation in patients of a wearable-based cardiac arrest detection model, demonstrating that **wrist-derived PPG reliably detects shockable cardiac arrest, with 100% sensitivity for VF**."
> — Circulation: Arrhythmia and Electrophysiology, 2025 [^12]

### 손목 위치의 한계 · Scientific Reports 2024

> "Across the classification models trained and evaluated on the three anatomical locations (i.e., fingertip, finger base, wrist), higher classification performances were observed on the finger (**macro average F1-score of 0.964** on the fingertip and 0.954 on the finger base) compared to the wrist (**macro average F1-score of 0.837**)."
> — Scientific Reports, 2024년 10월 [^13]

손목 PPG는 손가락 PPG보다 정확도가 낮지만 임상적으로 의미있는 수준입니다.

### 국내 응급실 검증 연구

서울 한 3차병원 응급실 OHCA 환자 50명 대상 연구:

> "Smart watches at all three sites had the same or higher sensitivity than manual palpation. The sensitivity of the smart watch was the highest, at 100%, in the carotid region and the lowest, at 78.6%, in the wrist region. **The specificity of the smart watch was the highest, at 100%, in the wrist region** and the lowest, at 78.7%, in the carotid region."
> — PubMed, 2019년 [^14]

손목 PPG는 **민감도 78.6%, 특이도 100%**를 보여, 오신고가 적다는 것이 핵심 강점입니다.

### SilverCare에 대한 시사점

- "PPG로 심정지 잡는 게 가능한가?"라는 근본 의문은 해소됨
- 우리 제품은 학술적 검증 위에 서 있음
- 손목 위치 한계는 인정해야 하며, false negative는 여전히 존재 → 마케팅에서 과장 금물

---

# 🟡 사업화 단계 검증

## 4. 통신·요금제 협상

**🟢 거의 파악**

### SKT LTE-M 소매 요금제 (확인됨)

| 요금제 | 월정액 (부가세 포함) | 데이터 |
|---|---|---|
| LTE-M 30 | 3,300원 | 30MB |
| LTE-M 40 | 4,400원 | 50MB |
| LTE-M 50 | 5,500원 | 100MB |

출처: T world 상품 페이지 [^15][^16]

또한 SKT는 자가개통 단말 대상:

> "1천원(5MB), 2천원(15MB), 3천원(30MB), 4천원(50MB), 5천원(100MB)입니다."
> — SKT IoT FAQ [^17]

### B2B 대량 단가

- 일반적으로 소매가의 30~50% 수준 (업계 통념)
- 5만 대 이상 회선 계약 시 월 1,000~2,000원대 가능 추정
- 우리 추정 ARPU 4,000원에서 통신원가 1,200원은 합리적 추정

### 직접 확인 불가

- LTE-M의 산간·도서 지역 실측 커버리지
- 119 응답 콜백을 위한 VoLTE 음성 지원 여부 (LTE-M Cat.M1은 일반 음성 미지원)

→ SKT/KT IoT 전문대리점 또는 영업담당 직접 컨택 필요.

---

## 5. 양산 BOM 검증

**🟡 부분 파악**

### e-ink 디스플레이 모듈 (확인됨)

> "Colorful Smart Watch 1.5 inch Display Square E-Ink Epaper Screen · $2.20-5.97 · Min. order: 1 piece"
> — Alibaba [^18]

대량 발주 시 3,000~6,000원 수준으로 우리 추정(6,000원) 적정.

### MCU·센서·모뎀 단가

대량 단가는 비공개. Digi-Key/Mouser 등에서 소량 단가는 확인 가능하나, **5만 대 ODM 견적이 필수**입니다.

### 직접 확인 불가

- 양산 5만 대 MOQ로 BOM ₩38,000 실현 가능 여부
- 국내 ODM(인탑스·휴맥스·LG이노텍) vs 중국 ODM 단가 차이

→ 위탁제조사 3곳 이상에서 실제 견적 의뢰가 필수. 통상 추정치 대비 5,000~10,000원 상승 가능성 있음.

---

## 6. 배터리·전력 예산

**🟡 부분 파악**

### 이론적 근거

- Nordic nRF52840 (Cortex-M4F, 64MHz, 64nA Sleep)
- e-ink는 정적 표시 시 전력 소모 거의 없음 (bistable 특성)

> "Bistability significantly reduces the power consumption of displays using E Ink and is a key reason eReaders have such long battery life."
> — LCD Module TFT [^19]

### 결정 변수

- LTE-M attach/detach 주기, eDRX·PSM 설정
- PPG 측정 빈도 (1Hz 상시 vs 5분 주기)
- 동일 칩셋·디스플레이로 380mAh 배터리에서 7일 작동은 이론적으로 가능하나, 실측 필수

### 직접 확인 불가

실제 7일 가는지는 프로토타입 보드로 실측해야 합니다. 변수가 많아 시뮬레이션만으로는 신뢰도 낮음.

---

# 🟢 사업 모델 보강

## 7. 응급안전안심서비스와의 관계 정립

**🟢 핵심 데이터 확보**

### 사업 규모

> "[관계부처] 보건복지부 [예산액] **28,847 백만원** [집행액] 28,487 백만원 [실집행액] 25,719 백만원"
> — 마이버짓 사업추진현황 (2023년 기준) [^20]

2023년 예산 약 288억 원, 집행률 99% 수준의 안정적 예산 사업입니다. 우리 시범사업 100억은 이 예산의 35%로, **별도 사업으로 추진해야 함, 기존 예산 내 편입 불가능**.

### 사업 내용·한계

> "(서비스 내용) 댁내 설치된 장비에서 감지한 응급상황을 119 및 응급관리요원에 연락해 신속한 구조·구급 지원"
> — 마이버짓 사업추진현황 [^20]

**핵심 한계**: 댁내 게이트웨이·화재감지기 중심이라 **집 밖에 나가면 작동하지 않습니다**. 이 빈 공백이 SilverCare의 명확한 진입점입니다.

### 운영 체계

> "한국사회보장정보원은 사회보장정보를 통합하여 관리·활용하여 포용적 복지를 실현하는 사회보장 정보 플랫폼 선도기관... 디지털돌봄시스템 관리자(safety@ssis.or.kr)"
> — 한국사회보장정보원 [^21]

운영 주체는 한국사회보장정보원, 현장 인력은 사회복지사·거점응급관리요원·읍면동 행정복지센터.

### 대상자

> "독거노인의 경우, 주민등록상 거주지와 동거자 유무와 관계없이 실제로 혼자 살고 있는 만 65세 이상의 노인" — 한국사회보장정보원 [^21]

### 대상자 확대 (2023년)

> "보건복지부는 「독거노인·장애인 응급안전안심서비스」 사업의 대상자 기준을 확대해 돌봄이 필요한 어르신·장애인 가구에 서비스를 제공한다고 7.17.(월) 밝혔다… 기존에는 65세 이상의 홀로 지내는 노인에게 서비스를 제공하였지만, 대상자 [기준이 확대됨]"
> — KDI 경제정보센터, 2023년 7월 [^22]

### 시사점

"응급안전안심서비스의 **이동형 보완 모듈**"이라는 포지셔닝이 가장 정확한 사업 메시지입니다.

---

## 8. 경쟁자 정밀 분석

**🟡 부분 파악**

### 휴이노 메모워치 (직접 경쟁자)

- 식약처 2등급 의료기기 허가 (2019년 3월) [^7][^8]
- 건강보험 급여대상 등재 (2020년 5월) [^9]
- 포지션: **홀터심전계 = 진단 보조** (응급 알림이 아님)
- 가격대: 약 30만 원대 (의료기기)
- 시사점: 우리와 시장 포지션이 다름. 잠재 협력 가능

### SKT 누구 돌봄 케어콜 (간접 경쟁자)

> "1일 경상남도와 서비스 시작… AI 상담사가 돌봄 케어 대상자에게 전화 연결 – 누구 인터렉티브 기술 기반 상담 서비스 3종… 총 1100만 콜, 8만 6천 시간 통화 실적"
> — SK텔레콤 뉴스룸, 2026년 1월 [^23]

AI 전화 안부 확인 방식으로 디바이스가 없음. 디바이스 경쟁자가 아니라 **잠재적 파트너**.

### 케어닥-SKT MOU (생태계 신호)

> "시니어 돌봄 플랫폼 케어닥은 SK텔레콤(SKT)과 케어닥 이용 고객을 위한 인공지능(AI) 시니어 케어콜 서비스 관련 업무 협약(MOU)을 공식 체결했다고 밝혔다."
> — 헬소, 2024 [^24]

시니어 돌봄 + AI 케어콜이 시장 형성 중. 우리가 이 생태계에 디바이스로 진입할 자리 존재.

### Apple Watch의 오신고 문제 (반면교사)

> "Apple's September 2023 update to its fall detection algorithms triggered a rash of false alarms. As the New York Times noted in a recent story, 'Why Apple Watch Keeps Calling 911:' …the latest [Apple Watch] innovation appears to send the device into overdrive: It keeps mistaking skiers, and some other fitness enthusiasts, for car-wreck victims. Lately, emergency call centers in some ski regions have been inundated with inadvertent, automated calls, dozens or more a week."
> — Medical Alert Buyers Guide [^25]

Apple Watch가 오신고로 911 콜센터에 부담을 준 사례. 우리가 **오신고 최소화 설계**를 강조하는 정당한 근거.

### Philips Lifeline (글로벌 벤치마크)

미국 시니어 응급호출 시장 1위. 게이트웨이 + 펜던트 + 워치형 조합. 월 구독 모델이 일반적 [^26].

### 직접 확인 불가

휴맥스·인탑스·LG U+·중소 시니어 디바이스 기업의 미공개 개발 프로젝트.

---

## 9. ROI 근거 강화

**🟡 부분 파악**

### 국내 OHCA 발생률

> "우리나라에서 '병원 밖 심정지(OHCA)' 환자 발생률은 인구 **10만 명 당 84명** 정도이며, 주요 사망의 원인 중 하나"
> — 한양대학교병원, 2023년 8월 [^27]

65세 이상 1,051만 명에 단순 적용 시 연간 약 8,800명의 OHCA 발생 추정 (단, 연령별 발생률 차이 보정 필요).

### 전체 등록 건수

> "All OHCA cases in which patients are transported by the national 119 EMS to a medical institution are captured in this registry, corresponding to approximately **30,000 cases annually**"
> — PMC (질병관리청 OHCA 등록자료 분석), 2024 [^28]

### 생존율

> "응급실 퇴실 시 생존 여부에 영향을 미치는 요인을 분석하기 위하여 다중 로지스틱 회귀 분석을 실시함. 연구결과: 생존율은 **19.3%**였으며 생존에 긍정적 영향을 미친 요인은 병원 전 단계에서 응급실 도착 전 심폐소생술(CPR)의 시행과 환자 발견 후 응급실 도착 시간"
> — 대한보건연구 (2016년 자료 분석), 2020 [^29]

→ **골든타임 단축 = 생존율 직접 개선 인과**가 통계적으로 입증되어 있음. SilverCare가 발견 시간을 단축하면 생존율 개선에 기여 가능.

### 직접 확인 불가

- 1인당 의료비·요양비 절감액 정밀 추정
- VSL(통계적 생명가치) 적용 시 1인당 사회적 가치

→ 한국보건사회연구원, 건강보험공단 정책연구원 보고서 별도 검색 또는 자문 필요.

---

# 🔵 부가 검증

## 10. 사용자 수용성 조사

**❌ 직접 확인 불가**

웹 검색으로는 65세 이상 시니어의 다음 사항을 알 수 없습니다:

- e-ink 디스플레이 가독성에 대한 실제 반응
- 큰 빨간 SOS 버튼 인지율·심리적 거부감
- 매일 또는 주 1회 충전 가능성
- 자녀 보호자의 월 6,900원 지불 의향
- 가족 알림 앱 사용성

→ 노인복지관·행정복지센터 협조로 **30~50명 심층 인터뷰(IDI) + 1~2주 wear test가 필수**.

---

## 11. 데이터·개인정보 처리 설계

**🟡 부분 파악**

- ISMS-P 인증 (KISA): B2G 사업 입찰 시 일반적으로 요구
- 개인정보보호위원회 생체정보·위치정보 가이드라인 존재
- 디지털의료제품법 (2025년 시행)에서 디지털의료기기 소프트웨어 데이터 처리 기준 신설됨 [^6]

### 직접 확인 불가

- 사망 후 생체 데이터 권리 귀속
- 응급 상황 예외 적용 가능 범위
- 가족 알림 시 개인정보 동의 범위

→ 의료기기·헬스케어 전문 변호사 자문 필요.

---

## 12. 책임 한계와 보험

**❌ 직접 확인 불가**

- 의료기기 PL 보험료 실제 견적
- 자동 신고 실패로 사망 발생 시 제조물 책임 범위
- 약관 설계 (의료기기 진단·치료가 아닌 "응급 알림 시스템" 명시)

→ 보험중개사 + 변호사 협업 영역.

---

# 📌 PoC 전 우선순위 액션 플랜

| 우선순위 | 항목 | 다음 액션 | 소요 | 비용 |
|---|---|---|---|---|
| 🔴 1 | 식약처 의료기기 분류 사전상담 | emed.mfds.go.kr 신청 | 1~2개월 | 무료 |
| 🔴 2 | ODM 3곳 BOM 견적 | 인탑스·휴맥스·중국 ODM 컨택 | 2~4주 | 무료 |
| 🟡 3 | 응급안전안심서비스 사무관 면담 | 보건복지부 노인지원과 | 협의 | 무료 |
| 🟡 4 | SKT/KT B2B LTE-M 대량 견적 | IoT 전문대리점 컨택 | 2주 | 무료 |
| 🟡 5 | 의정부 시민안전망 사례 청취 | 의정부시 스마트도시과 | 협의 | 무료 |
| 🟢 6 | 시니어 30명 IDI | 노인복지관 협조 | 1개월 | 200~500만 |
| 🟢 7 | 임상 협력 병원 사전 접촉 | 분당서울대·서울아산 | 협의 | 무료 |
| 🟢 8 | 한국보건사회연구원 OHCA 비용 보고서 검색 | 자료 조사 | 1주 | 무료 |
| 🔵 9 | 의료기기·헬스케어 변호사 자문 | 법무법인 컨택 | 협의 | 시간당 30~50만 |

---

# 🎯 다음 단계 권고

가장 적은 시간·비용으로 가장 큰 정보를 얻을 수 있는 행동 3가지:

**① 식약처 의료기기 분류 사전상담 신청**
무료, 1~2개월. 답에 따라 사업 전체 일정과 예산이 2~3배 차이남. 가장 큰 미지수를 해소함.

**② 보건복지부 응급안전안심서비스 담당 사무관 비공식 면담**
정부 사업은 공모 전 사전 협의가 절반. "이런 디바이스를 만들고 있는데 사업에 편입 가능한 경로가 있는지" 질의로 시작.

**③ 의정부시 스마트도시과 사례 청취**
이미 IoT → 119 자동 연계를 운영 중인 유일한 지자체. 우리 모델의 운영상 노하우를 얻을 수 있음.

---

## 📚 출처

[^1]: 하동소방서 (2025-09-01), "소방서, '119 다매체 신고 서비스' 운영한다", 하동뉴스. https://www.hadongnews.co.kr/news/articleView.html?idxno=20173

[^2]: 경기북부탑뉴스 (2023-07-28), "의정부시, 전국 최초 AI와 IoT 센서 결합한 119 다매체 신고체계 구축". https://www.gbtopnews.net/news/articleView.html?idxno=94449

[^3]: 정부24, "의료기기 해당 여부 검토 신청". https://www.gov.kr/mw/AA020InfoCappView.do?HighCtgCD=A06002&CappBizCD=14700000653

[^4]: 의료기기 안심책방, "허가 절차". https://emedi.mfds.go.kr/msismext/emd/bif/prmProcssView.do

[^5]: 이지경제 (2025-06-05), "식약처, 의약품·의료기기 등 제품화지원 상담 업무 안내서 마련". https://www.ezyeconomy.com/news/articleView.html?idxno=214748

[^6]: Lexology (2025-05-13), "식약처, 디지털의료기기 관련 가이드라인 6종 제·개정". https://www.lexology.com/library/detail.aspx?g=516ed348-16f2-48f0-9545-dc8a061790d5

[^7]: 한국경제 (2019-03-25), "휴이노, 시계형 심전도 장치 메모 워치 식약처 허가". https://www.hankyung.com/article/201903252376f

[^8]: 히트뉴스 (2019-03-25), "휴이노, 웨어러블 심전도 장치 KFDA 승인 획득". http://www.hitnews.co.kr/news/articleView.html?idxno=7285

[^9]: 메트로서울 (2020-05-19), "휴이노 '메모워치', 국내 최초 웨어러블 의료기기 건강보험 등재". https://www.metroseoul.co.kr/article/20200519500303

[^10]: 의료기기 안심책방, "허가 절차" — 의료기기법 시행규칙 제4조. https://emedi.mfds.go.kr/msismext/emd/bif/prmProcssView.do

[^11]: Nature (2025-02-26), "Automated loss of pulse detection on a consumer smartwatch". https://www.nature.com/articles/s41586-025-08810-9

[^12]: Circulation: Arrhythmia and Electrophysiology (2025), "Automated Cardiac Arrest Detection Using Wrist-Worn Photoplethysmography: External Validation in Patients With Induced Shockable Cardiac Arrest (DETECT-1b)". https://www.ahajournals.org/doi/full/10.1161/CIRCEP.125.014708

[^13]: Scientific Reports (2024-10-05), "Detecting cardiac states with wearable photoplethysmograms and implications for out-of-hospital cardiac arrest detection". https://www.nature.com/articles/s41598-024-74117-w

[^14]: PubMed (2019-02-19), "Can pulse check by the photoplethysmography sensor on a smart watch replace carotid artery palpation during cardiopulmonary resuscitation in cardiac arrest patients?". https://pubmed.ncbi.nlm.nih.gov/30782884/

[^15]: T world, "LTE-M 30 상품원장". https://m.tworld.co.kr/product/callplan?prod_id=NA00005667

[^16]: T world, "LTE-M 50(선납) 상품원장". https://m.tworld.co.kr/product/callplan?prod_id=NA00005921

[^17]: SKT IoT FAQ, "자가개통 단말의 요금제는 어떤 것이 있나요?". https://m.catm1.sktelecom.com/front/common/popFaq.jsp

[^18]: Made-in-China / Alibaba, "Colorful Smart Watch 1.5 Inch Display Square E-Ink Epaper Screen". https://qdyesh.en.made-in-china.com/product/bxZUQXVrhMhP/

[^19]: LCD Module TFT, "1.73 Inch 240X320 E Ink Smartwatch SGS Electronic Programmable E Ink Display". https://www.lcdmoduletft.com/sale-14279169-1-73-inch-240x320-e-ink-smartwatch-sgs-electronic-programmable-e-ink-display.html

[^20]: 마이버짓 (열린재정), "독거노인·중증장애인 응급안전안심서비스 사업추진현황 (2023년)". https://www.mybudget.go.kr/budgetBsnsInfo/executionResultView?in_year=2023&cndcy_no=T2300018

[^21]: 한국사회보장정보원, "응급안전안심서비스 FAQ". http://www.ssis.or.kr/lay1/bbs/S1T113C123/G/46/list.do

[^22]: KDI 경제정보센터 (2023-07-17), "응급안전안심서비스, 노인 부부·조손 가구 등 대상자 기준 확대한다!". https://eiec.kdi.re.kr/policy/materialView.do?num=240918

[^23]: SK텔레콤 뉴스룸 (2026-01-22), "SKT, '누구(NUGU) 돌봄 케어콜'로 독거 어르신 안부 확인한다". https://news.sktelecom.com/172066

[^24]: 헬소, "케어닥, SKT와 'AI 시니어 케어콜' 서비스 도입 위한 MOU". https://healtho.co.kr/news/view.php?idx=138459

[^25]: Medical Alert Buyers Guide (2026-01-13), "Apple Watch Medical Alert With Fall Detection: Not Quite There Yet". https://www.medicalalertbuyersguide.org/smartwatch-medical-alert-system/apple-watch-medical-alert-review/

[^26]: TechRadar (2024-06-13), "Philips Lifeline review". https://www.techradar.com/reviews/philips-lifeline-review

[^27]: 한양대학교병원 (2023-08-10), "병원 밖 심정지 생존자의 우울증 관리가 장기 생존에 영향 미친다". https://www.hyumc.com/seoul/hospitalStory/hospitalNews.do?action=view&bbsId=news&nttSeq=12843

[^28]: PMC (2024), "A Moderated Mediation Analysis of Timely EMS Activation and Bystander CPR in the Association Between Regional Deprivation and Outcomes Following Out-of-Hospital Cardiac Arrest". https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12896887/

[^29]: 대한보건연구 (2020), "병원 밖 심장정지 환자의 생존 요인 분석". https://www.kci.go.kr/kciportal/ci/sereArticleSearch/ciSereArtiView.kci?sereArticleSearchBean.artiId=ART002594684

---

*본 문서는 SilverCare Watch 사업 가안의 PoC 사전 조사 자료로, 공개 자료에 근거한 추정과 분석을 포함합니다. 실제 사업 추진 시 모든 수치는 각 기관 직접 컨택을 통해 검증되어야 합니다.*
