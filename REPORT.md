# EP-P5400 무선충전기 분석 보고서 (발표 대본용 자료)

> 이 보고서는 발표자료 슬라이드 순서와 1:1로 대응한다. 각 절 제목의 `[S#]`는 슬라이드 번호.
> 각 절은 **핵심 메시지 → 근거·수치 → 설명 포인트 → 예상 질문** 순서로 정리했다. 대본 작성 시 "핵심 메시지"를 첫 문장으로 쓰면 된다.
> 표기: **[확인]** 분해·데이터시트로 확인 / **[추정]** 동일 계열 데이터시트로 추론 / **[가정]** 계산을 위해 둔 값.

---

## [S1] 표지
- 제목: Samsung EP-P5400 Super Fast Wireless Charger Duo — 반도체 관점 분석
- 부제: 충전 효율과 발열은 어디서 결정되는가

## [S2] 분석 흐름
Step 0 제품 → Step 1 설계 과제(효율·발열) → **메인 블록: 효율–발열 결합 루프** → Step 2 블록 다이어그램 → Step 3 칩 → Step 4 회로(H-bridge, LC 공진) → **MOSFET 소자 심화** → 열관리 → Step 5 기업 → Step 6 기술 트리

---

## [S3] Step 0. 제품 개요
**핵심 메시지:** 15 W 폰 충전과 웨어러블 충전을 동시에 하는 2-패드 Qi 충전기이며, 팬을 내장했다.

| 항목 | 내용 | 근거 |
|---|---|---|
| 출력 | 좌측 폰 최대 15 W / 우측 Watch·Buds 전용 | Samsung 제품 페이지 [확인] |
| 입력 | USB-C 9 V 2.77 A (25 W TA 동봉) | 라벨 [확인] |
| 냉각 | Toshiba 팬 5 V 0.31 A | 분해 [확인] |
| 내부 | PCBA 1장 + 코일 3개(폰 1, 워치 외곽 1, 버즈 내측 1) + 팬 | 분해 [확인] |

- 시각자료: `figures/01_inside_pcba_fan.jpg`, `figures/13_coils.jpg`
- 설명 포인트: 우측 패드는 워치(외곽 큰 코일)와 버즈(내측 작은 코일)를 서로 다른 코일로 처리한다 → 기기별 코일 크기·인덕턴스가 다르기 때문.

## [S4] Step 1. 설계 과제 — 효율과 발열
**핵심 메시지:** 폰 충전 시 입력 19.56 W 중 최소 4.5 W가 열이 된다. 효율과 발열은 같은 문제의 양면이다.

| 조건 | 패드 입력 전력 (ChargerLab 실측) |
|---|---|
| 대기 | 0.65 W |
| Galaxy S22+ | 19.56 W |
| AirPods 3 | 2.93 W |
| S22+ + AirPods 3 동시 | 17.75 W |

- 효율 계산: η = P<sub>out</sub>/P<sub>in</sub> ≈ 15 W / 19.56 W ≈ **77 % (상한)** — 폰 수신전력이 15 W라고 가정 [가정]
- 손실: 19.56 − 15 ≈ **4.5 W 이상** → 전부 열
- 흥미로운 점: 동시 충전(17.75 W)이 폰 단독(19.56 W)보다 낮다 → 총 입력전력 한계(약 20 W) 안에서 MCU가 **전력을 분배**하는 것으로 해석 [추정]
- 시각자료: `figures/43_measured_input_power.png`
- 예상 질문: "실제 수신 전력은?" → 측정 자료 없음. 77 %는 상한이며 실제는 더 낮을 수 있음.

---

## [S5] System Level · Main Block — 효율–발열 결합 루프
**핵심 메시지:** 효율과 발열은 따로 있는 두 문제가 아니라 하나의 루프다. 이 발표는 이 루프를 메인 블록으로 삼아 모든 회로를 "루프의 어느 지점을 막는가"로 설명한다.

루프 구성
1. **① 전력 손실**: P<sub>in</sub> − P<sub>out</sub> ≥ 4.5 W. 전력 경로의 모든 단(USB-C·MCU, DC-DC, H-bridge, 코일, 폰 정류)에서 생긴 손실이 모인다.
2. **② 온도 상승**: 손실은 모두 열이 되어 충전기와 폰을 함께 가열한다.
3. **③ 폰 충전 전력↓**: 폰은 배터리 보호를 위해 스스로 충전 전력을 낮춘다(스로틀링) → 15 W 유지 실패.
4. **악순환 (②→①)**: 온도↑ → 구리 저항(+0.39 %/°C)·R<sub>DS(on)</sub>↑ → 손실↑ → 온도↑ (양의 피드백)

대책 (루프를 끊는 지점)
| 대책 | 막는 지점 | 관련 블록 |
|---|---|---|
| ZVS · V<sub>bridge</sub> 최적화 | ① 손실 자체를 줄임 | H-bridge, MP28167-A, IDTP9236 |
| 팬 + G761 | ② 열을 빼내 온도↓ (악순환 차단) | 팬, G761 |
| 팬 · NTC · FOD | ③ 폰 표면 냉각, 과열·이물질 차단 | 팬, NTC, IDTP9236 |

- 설명 포인트: ZVS·V<sub>bridge</sub> 최적화는 효율 대책이면서 발열 대책이고, 팬은 발열 대책이면서 효율(15 W 유지) 대책이다.
- 시각자료: 발표자료 5번 슬라이드 (시스템 레벨 다이어그램), SPEC.md §2-0

## [S6] Step 2. 시스템 블록 다이어그램
**핵심 메시지:** 전력은 USB-C → DC-DC(전압 조절) → H-bridge(DC→AC) → LC 공진·코일(자기결합) → 폰 순서로 흐르고, 폰은 역방향으로 "더/덜 보내라" 신호를 보낸다.

전력 경로 [확인/추정]
1. **USB-C 9 V** → ABOV MCU가 PD/AFC 협상으로 9 V 확보 (PD3.0 인증 칩)
2. **MP28167-A (Buck-Boost)** → H-bridge 입력전압 V<sub>bridge</sub> 생성 (폰 채널)
3. **Full-bridge (VS3618AE ×4)** → 127.7 kHz 교류 생성 [추정]
4. **C<sub>p</sub> (NPO ×5) + L<sub>p</sub> (폰 코일)** → 직렬 공진, 자기장 송출
5. 우측 패드: Buck(6.8 µH) → Half-bridge (DGD0590A 구동) → 워치/버즈 코일
6. **IDTP9236 ×2**: 패드별 Qi 송신 제어 (게이트 구동, 통신 복조, FOD)

## [S7] 전력 변환 단별 손실
| 단 | 블록 | 손실 메커니즘 |
|---|---|---|
| ① | 입력·보호 | 커넥터/트레이스 I²R — 9 V 입력으로 전류를 줄여 완화 |
| ② | DC-DC | 스위칭손실, 인덕터 DCR, R<sub>DS(on)</sub> — 데이터시트상 ~95 % 효율 영역 |
| ③ | H-bridge | 도통손실 + 스위칭손실 (ZVS 여부가 핵심) |
| ④ | 코일·자기결합 | 코일 ESR, 누설자속, 와전류, **거리·정렬에 따른 k 감소** |
| ⑤ | 폰 내부 정류 | 정류기·LDO 손실 |

설명 포인트: S23에서 계산하듯 ③ MOSFET 손실은 수백 mW 수준 → 4.5 W 손실의 대부분은 ②④⑤, 특히 **④ 자기결합**이 지배적이다.

## [S8] 피드백 루프
**핵심 메시지:** 폰이 측정한 전압 오차를 Control Error Packet(CEP)으로 보내면, 송신기는 브리지 전압을 올리거나 내린다.

- Rx→Tx 통신: **ASK** (폰이 부하를 변조 → 송신 코일 전압/전류 진폭 변화), 2 kbps [확인: P9242 datasheet]
- Tx→Rx 통신: **FSK** (동작 주파수 미세 변조)
- 패킷: CEP(8-bit 부호 있는 오차), RPP(수신 전력 → FOD용), EPT(전송 종료)
- 제어 변수: 고정주파수(127.7 kHz) 모드에서는 **V<sub>bridge</sub>**를 조절 (P9243-GB §8) [추정: EP-P5400에 적용]
- 루프 속도: 새 목표 전압 수렴까지 최대 ~90 ms (TI 데이터시트)
- 열 루프: NTC 온도 → 과열 시 전력 제한, G761 → 팬 속도
- 시각자료: `figures/26_P9243_fig11_demod_waveform.png` (ASK 복조 신호)

---

## [S9] Step 3. 실제 PCB
- 시각자료: `figures/02_pcb_front.jpg`
- 위치 설명: 우상단 USB-C → 그 아래 ABOV MCU, 중앙 2R2 인덕터(MP28167-A용 2.2 µH), 좌·우 IDTP9236 2개, 하단 NPO 커패시터 열(공진), 좌측 팬 커넥터

## [S10] 사용 칩 목록
| 칩 | 제조사 | 역할 | 핵심 사양 |
|---|---|---|---|
| IDTP9236 ×2 | Renesas (구 IDT) | Qi 송신 SoC | Cortex-M0 + FET 드라이버 + 복조기, 외부 Flash 펌웨어 |
| MP28167-A | MPS | 4-switch Buck-Boost | 2.8–22 V 입력, 3 A, I²C V<sub>REF</sub> 0.8 mV 분해능, QFN 3×3 |
| VS3618AE | Vergiga (Vanguard) | H-bridge MOSFET | 30 V, 50 A, 6.4 mΩ@10 V, ~11 mΩ@4.5 V, PDFN3333 |
| DGD0590A | Diodes | 하프브리지 게이트 드라이버 | 40 V 부트스트랩, 5 V 게이트, 지연 16/12 ns |
| ABOV 94B428 | ABOV | 시스템 MCU | USB PD3.0, 패드 간 전력 분배 |
| G761 | GMT | 팬 컨트롤러 | 개루프/폐루프 |
| S3FW6MA | Samsung | 커스텀 (기능 미공개) | – |

## [S11] IDTP9236은 어떤 칩인가 (P9243-GB로 추정)
**핵심 메시지:** 공개 데이터시트가 없지만, 동일 계열 P9243-GB와 5가지 특징이 일치한다.

| P9243-GB 특징 | EP-P5400에서 관찰 |
|---|---|
| "Samsung Fast Charge" 지원, 15 W | 삼성 15 W 충전기 |
| 외부 Flash 펌웨어 | 칩마다 외부 메모리 1개 |
| 127.7 kHz 고정 → 외부 오실레이터 | 8 MHz 크리스털 |
| 외부 강압 레귤레이터로 V<sub>bridge</sub> 제어 | MP28167-A |
| MP-A11 코일 6.3 µH / 500 nF | NPO 공진 커패시터 5개 |

- SoC 관점: **Cortex-M0 CPU + 아날로그 프론트엔드(전류센스앰프, 복조기, ADC) + 전원(LDO, 내부 Buck) + 고전류 게이트 드라이버**를 한 다이에 집적 → 혼성신호(mixed-signal) SoC
- 시각자료: `figures/22_P9243_fig15_block_diagram.png`
- 예상 질문: "다이 레이아웃은?" → 공개 자료 없음. 기능 블록도로 대체.

## [S12] MP28167-A — 4-switch Buck-Boost
**핵심 메시지:** 입력이 5 V든 9 V든 필요한 브리지 전압을 만들기 위해, 강압·승압을 모두 하는 4-스위치 구조를 쓴다.
- V<sub>IN</sub> > V<sub>OUT</sub>: Buck 모드 (SWA/SWB 스위칭, SWD 상시 ON)
- V<sub>IN</sub> < V<sub>OUT</sub>: Boost 모드 (SWC/SWD 스위칭, SWA 상시 ON)
- V<sub>IN</sub> ≈ V<sub>OUT</sub>: Buck-Boost (4개 모두 스위칭)
- 제어: Buck은 COT(정 온타임), Boost는 정 오프타임 → 빠른 과도응답
- I²C로 기준전압을 0.8 mV 단위 조정 → **송신 전력의 미세 조정 수단** [추정: 제어 주체는 미확인]
- 시각자료: `figures/30_MP28167A_fig2_3_topology.png`

---

## [S13] Step 4-1. MOSFET H-Bridge
**핵심 메시지:** MOSFET 4개가 대각선 쌍으로 번갈아 켜지며 DC를 127.7 kHz 교류로 바꾼다.
- Q1·Q4 ON → 코일에 +V, Q2·Q3 ON → −V
- Full-bridge: 같은 V<sub>bridge</sub>에서 Half-bridge 대비 전압 진폭 2배 → 15 W 폰 채널
- Half-bridge: 저전력 웨어러블 채널 (DGD0590A가 상·하 MOSFET 구동)
- 데드타임: 상·하 동시 ON(관통전류, shoot-through) 방지
- 시각자료: `figures/09_hbridge.jpg` (실물), `figures/27_P9242_fig18_lc_resonant.png` (회로)

## [S14] Step 4-2. LC 공진과 주파수 응답
**핵심 메시지:** 공진점에서 코일 전류가 최대가 되며, 부하가 무거울수록 공진 곡선이 완만해진다.
- f<sub>0</sub> = 1/(2π√(LC)) = 1/(2π√(6.3 µH × 500 nF)) ≈ **89.7 kHz** (MP-A11 레퍼런스 값) [추정]
- 동작점 127.7 kHz는 f<sub>0</sub>보다 높다 → 다음 슬라이드(위상)에서 이유 설명
- 공진 커패시터 전압 최대 ~70 V → 100 V C0G/NPO 필요 (온도·전압에 따른 정전용량 변화 거의 없음 → 공진점 고정)
- 시각자료: `figures/40_concept_lc_frequency_response.png` (데이터시트 L/C 값으로 계산)

## [S15] 페이즈 매칭과 ZVS
**핵심 메시지:** 공진점보다 약간 높은 주파수(유도성 영역)에서 동작하면 전류가 전압보다 뒤져서, MOSFET이 전압 0 상태에서 켜진다(ZVS).
- f < f<sub>0</sub>: 용량성 → 전류가 앞섬 → 하드 스위칭, 바디다이오드 역회복 손실
- f > f<sub>0</sub>: 유도성 → 데드타임 동안 코일 전류가 스위치 노드 C<sub>oss</sub>를 충·방전 → 턴온 시 V<sub>DS</sub> ≈ 0
- 시각자료: `figures/41_concept_phase_zvs.png`

## [S16] 결합계수와 정렬 — 효율의 지배 요인
**핵심 메시지:** 코일이 멀어지거나 어긋나면 결합계수 k가 줄고, 효율이 급락한다.
- 상호유도: V<sub>rx</sub> = jωM·I<sub>tx</sub>, k = M/√(L<sub>1</sub>L<sub>2</sub>)
- 최대 링크 효율: η<sub>max</sub> = k²Q<sub>1</sub>Q<sub>2</sub> / (1+√(1+k²Q<sub>1</sub>Q<sub>2</sub>))²
- 논문 실측 (Domajnko 2022): k = 0.73 (10 mm) → 0.15 (58 mm), 25 mm 오정렬 시 k 60 % 이상 감소
- 데이터시트 실측 (P9243-GB Fig.3): 중심 82 % → 가장자리 75 %, 범위 밖은 충전 불가
- 임피던스 매칭: 폰 쪽 임피던스가 Tx에 (ωM)²/Z<sub>rx</sub>로 반사됨 → k가 변하면 반사 임피던스가 변해 최적점에서 벗어남 → 피드백으로 보정
- 시각자료: `figures/42_concept_efficiency_vs_k.png`, `figures/20_P9243_fig3_xy_efficiency_map.png`

## [S17] 안전: 이물질 감지(FOD)
**핵심 메시지:** 동전·열쇠 같은 금속이 올라가면 와전류로 발열하므로, 송신기는 Q-factor와 전력 손실을 감시해 차단한다.
- 1단계 (전송 전): 코일 Q-factor 측정 → 기준보다 낮으면 금속으로 판단
- 2단계 (전송 중): P<sub>tx</sub> − P<sub>rx</sub>(RPP) > 임계값 → 차단
- 발열 관점: 손실 전력이 기기가 아닌 금속에서 열로 소모되는 상황을 막는 것
- 시각자료: `figures/24_P9243_fig27_q_factor.png`

---

## [S18] 심화: MOSFET 소자 관점 (섹션 표지)
**핵심 메시지:** H-bridge의 MOSFET 4개가 충전기 전체 효율에 어떤 영향을 주는지 소자 구조부터 본다.

## [S19] Planar vs Trench MOSFET
**핵심 메시지:** VS3618AE 같은 저전압 파워 MOSFET은 채널을 수직으로 세운 트렌치 구조로 단위면적당 저항을 줄인다.
- Planar: 채널이 표면에 수평 → 전류가 JFET 영역(두 P-body 사이 좁은 목)을 통과 → R<sub>JFET</sub> 존재, 셀 피치 축소 한계
- Trench: 게이트를 실리콘에 파묻어 채널이 트렌치 벽을 따라 수직 형성 → JFET 영역 제거, 셀 밀도↑ → **R<sub>DS(on)</sub>·면적 감소**
- R<sub>DS(on)</sub> = R<sub>ch</sub> + R<sub>acc</sub> + (R<sub>JFET</sub>) + R<sub>drift</sub> + R<sub>sub</sub> (+ 패키지)
- 30 V급에서는 drift층이 얇아 **채널·기판·패키지 저항 비중이 큼** → 셀 밀도를 높이는 트렌치가 유리
- 반도체공학개론 연결: MOS 반전층, 문턱전압 V<sub>th</sub>, 이동도, 공핍층 폭과 항복전압의 트레이드오프
- 시각자료: `figures/50_mosfet_planar_vs_trench.png` (개념도)

## [S20] 출력특성과 R<sub>DS(on)</sub>
**핵심 메시지:** 스위치로 쓰는 MOSFET은 선형(triode) 영역에서 작은 저항처럼 동작하며, 이때의 저항이 R<sub>DS(on)</sub>이다.
- 선형영역: I<sub>D</sub> = k<sub>n</sub>[(V<sub>GS</sub>−V<sub>th</sub>)V<sub>DS</sub> − V<sub>DS</sub>²/2] ≈ k<sub>n</sub>(V<sub>GS</sub>−V<sub>th</sub>)V<sub>DS</sub>
- 따라서 R<sub>DS(on)</sub> ≈ 1 / [k<sub>n</sub>(V<sub>GS</sub>−V<sub>th</sub>)] → **게이트 전압이 높을수록 R 감소**
- VS3618AE: 6.4 mΩ @ V<sub>GS</sub>=10 V, ~11 mΩ @ 4.5 V → DGD0590A의 5 V 게이트 구동에서는 ~11 mΩ 영역 [확인: 분해기·유통사 자료]
- "Logic-level" MOSFET을 쓰는 이유: 드라이버가 5 V만 공급하므로 낮은 V<sub>th</sub>(~2.5 V)가 필요
- 시각자료: `figures/51_mosfet_output_characteristics.png` (square-law 개념도)

## [S21] 게이트 전하와 구동 회로
**핵심 메시지:** MOSFET을 켜는 것은 게이트 커패시터를 충전하는 일이며, 밀러 구간이 스위칭 속도를 결정한다.
- Q<sub>gs</sub>: V<sub>th</sub>까지 C<sub>iss</sub> 충전 / Q<sub>gd</sub>(Miller plateau): V<sub>DS</sub>가 바뀌는 동안 V<sub>GS</sub> 정체 → 이 구간 길이 = 스위칭 손실 구간
- 게이트 구동 손실: P<sub>g</sub> = Q<sub>g</sub>·V<sub>GS</sub>·f
- 트레이드오프: 다이를 키우면 R<sub>DS(on)</sub>↓(도통손실↓) 하지만 Q<sub>g</sub>↑(스위칭·구동 손실↑) → 주파수에 맞는 최적 다이 크기 존재
- 하이사이드 구동: 소스가 떠 있으므로 **부트스트랩 커패시터**로 V<sub>GS</sub> 생성 (DGD0590A 내장 부트스트랩 다이오드, 최대 40 V)
- DGD0590A 전파지연 16/12 ns, 상승/하강 27/17 ns (3 nF 부하) — 127.7 kHz 주기(7.83 µs)의 1 % 미만
- 시각자료: `figures/52_mosfet_gate_charge.png`, `figures/35_DGD0590A_p2_block.png`

## [S22] ZVS 동작 파형
**핵심 메시지:** 데드타임 동안 코일 전류가 스위치 노드 전압을 반대쪽 레일까지 옮겨놓기 때문에, 다음 MOSFET은 전압 0에서 켜진다.
- ① Q1 OFF → ② 데드타임: 유도성 전류가 Q1의 C<sub>oss</sub> 충전, Q2의 C<sub>oss</sub> 방전 → V<sub>SW</sub> 0 V로 하강, Q2 바디다이오드 도통 → ③ Q2 ON (V<sub>DS</sub>≈0 → 턴온 손실 0)
- ZVS가 없으면: 턴온 시 ½C<sub>oss</sub>V² 에너지가 매 주기 채널에서 소모 + V·I 중첩 손실
- 시각자료: `figures/53_mosfet_zvs_waveform.png` (개념도)

## [S23] MOSFET 손실 추정 — 결론
**핵심 메시지:** H-bridge MOSFET 손실은 약 0.3–0.5 W로, 전체 손실 4.5 W의 일부에 불과하다. 효율의 승부처는 소자보다 자기결합이다.

계산 [가정: V<sub>bridge</sub>=12 V, I<sub>rms</sub>=2 A, I<sub>pk</sub>=2.8 A, R<sub>DS(on)</sub>=11 mΩ, Q<sub>g</sub>=10 nC, C<sub>oss</sub>=400 pF, t<sub>r</sub>+t<sub>f</sub>=44 ns, f=127.7 kHz]

| 손실 | 식 | Hard switching | ZVS |
|---|---|---|---|
| 도통 | 4 × I<sub>rms</sub>²·R·0.5 | 88 mW | 88 mW |
| 게이트 구동 | 4 × Q<sub>g</sub>·V<sub>GS</sub>·f | 26 mW | 26 mW |
| 스위칭 | 턴온 + 턴오프 + C<sub>oss</sub> | 392 mW | 189 mW (턴오프만) |
| **합계** | | **~506 mW** | **~302 mW** |

- 해석: ZVS로 MOSFET 손실 약 40 % 감소. 하지만 4.5 W 중 MOSFET은 ~7–11 % → 나머지는 DC-DC(~1 W), 코일·자기결합, 폰 내부 정류
- 시각자료: `figures/54_mosfet_loss_estimate.png`
- 예상 질문: "가정값 근거?" → R<sub>DS(on)</sub>은 분해기 자료, Q<sub>g</sub>·C<sub>oss</sub>는 30 V PDFN3333급 일반값 (VS3618AE 원문 데이터시트 미확보)

---

## [S24] 열관리 — 팬과 회로
**핵심 메시지:** 4.5 W의 열을 팬(능동)과 회로 설계(손실 저감·보호)로 함께 관리한다.
- 능동 냉각: Toshiba 팬 5 V 0.31 A(최대 1.55 W), **G761**이 타코 신호 기반 폐루프 속도 제어 → 필요할 때만 회전(소음·전력 절감)
- 열 → 효율 악순환: 온도↑ → 구리 저항↑(+0.39 %/°C), MOSFET R<sub>DS(on)</sub>↑(양의 온도계수) → 손실↑ → 온도↑
- 폰 쪽 영향: 배터리 온도가 오르면 폰이 스스로 충전 전력을 낮춤 → **충전 속도 저하** → 팬이 폰 표면까지 식혀 15 W 유지에 기여
- 회로적 관리: 9 V 입력(전류↓), Buck-Boost로 V<sub>bridge</sub> 최적화, ZVS, NPO 커패시터(저 ESR), NTC 과열 차단, FOD
- 시각자료: `figures/11_g761.jpg`

## [S25] Step 5·6. 기업과 기술 트리
| 기업 | 칩 | 분야 |
|---|---|---|
| Samsung | 제품 설계, S3FW6MA | 시스템·SoC |
| Renesas (IDT 인수, 2019) | IDTP9236 | 무선전력 혼성신호 SoC |
| MPS | MP28167-A | 전력관리 IC |
| ABOV | MCU | 한국 MCU 팹리스 |
| Vergiga, Diodes, GMT | MOSFET, 드라이버, 팬 IC | 디스크리트·아날로그 |

기술 트리: 전력전자(DC-DC, 인버터, ZVS) · 반도체소자(트렌치 MOSFET, 혼성신호 SoC) · 전자기학(상호유도, 결합계수, 와전류) · 회로설계(공진, 피드백 제어, 임피던스 매칭)

## [S26] 결론
1. 효율 상한 ~77 %, 손실 ≥4.5 W는 전부 열이 된다.
2. 손실의 승부처는 **코일 정렬·결합(k)**이며, 소자(MOSFET) 손실은 ZVS로 수백 mW까지 줄였다.
3. 고정주파수 + 가변 V<sub>bridge</sub>(Buck-Boost) 구조, CEP 피드백, FOD로 효율과 안전을 함께 관리한다.
4. 팬은 충전기와 폰의 온도를 낮춰 폰이 충전 전력을 낮추지 않게 하고, 15 W를 유지시킨다.

## [S27] 참고문헌
1. ChargerLab, "Teardown of Samsung 15W Faster Wireless Charger Duo (EP-P5400)", 2023. https://chargerlab.com/teardown-of-samsung-15w-faster-wireless-charger-duo-ep-p5400
2. Renesas, P9243-GB Datasheet, 2020. (refs/P9243-GB.pdf)
3. IDT/Renesas, P9242-R3/RB Datasheet, 2019. (refs/P9242-R3.pdf)
4. MPS, MP28167-A Datasheet Rev 1.0, 2020. (refs/MP28167-A.pdf)
5. Diodes Inc., DGD0590A Datasheet DS41728, 2021. (refs/DGD0590A.pdf)
6. J. Domajnko, M. Milanovič, N. Prosen, "3D Platform for Coupling Coefficient Evaluation of an Inductive Power Transfer Systems", *Sensors* 22(4):1445, 2022. doi:10.3390/s22041445
7. NXP AN11695 (Qi 송신기 공진 탱크·주파수 제어), TI bq51013B 데이터시트 (WPC 피드백 루프 속도)
8. B. J. Baliga, *Fundamentals of Power Semiconductor Devices*, Springer (트렌치 MOSFET R<sub>DS(on)</sub> 구성)
