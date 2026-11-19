# 외부 자료 출처 (저장소 미포함)

저작권이 있는 제3자 자료는 저장소에 넣지 않았다. 아래 링크에서 받아 같은 경로에 두면 SPEC.md / REPORT.md의 그림이 표시된다.

## 데이터시트 (`refs/`)
| 로컬 파일 | 출처 |
|---|---|
| refs/P9243-GB.pdf | https://www.renesas.cn/document/dst/p9243-gb-datasheet |
| refs/P9242-R3.pdf | https://www.renesas.cn/document/dst/p9242-r3rb-datasheet |
| refs/MP28167-A.pdf | https://www.monolithicpower.com/en/mp28167-a.html (사본: https://docs.rs-online.com/04d0/A700000013716514.pdf) |
| refs/DGD0590A.pdf | https://diodes.com/datasheet/download/DGD0590A.pdf |

## 분해 사진 (`figures/0*.jpg`, `figures/1*.jpg`)
출처: ChargerLab, "Teardown of Samsung 15W Faster Wireless Charger Duo (EP-P5400)", 2023
https://chargerlab.com/teardown-of-samsung-15w-faster-wireless-charger-duo-ep-p5400

| 로컬 파일 | 원본 이미지 |
|---|---|
| figures/01_inside_pcba_fan.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910034094.jpeg |
| figures/02_pcb_front.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910035238.jpeg |
| figures/04_abov_mcu.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910035646.jpeg |
| figures/05_idtp9236_a.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910035839.jpeg |
| figures/07_mp28167a.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910040777.jpeg |
| figures/09_hbridge.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910041682.jpeg |
| figures/10_npo_res_caps.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910041753.jpeg |
| figures/11_g761.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910043982.jpeg |
| figures/12_dgd0590a.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910044470.jpeg |
| figures/13_coils.jpg | https://www.chargerlab.com/wp-content/uploads/2023/04/2023041910045052.jpeg |

## 데이터시트 그림 캡처 (`figures/2x_P92xx`, `3x_MP28167A`, `35_DGD0590A`, `36_P9243`)
위 데이터시트의 해당 페이지·그림을 캡처한 것 (파일명에 그림 번호 표기).
| 로컬 파일 | 원문 위치 |
|---|---|
| 20_P9243_fig3_xy_efficiency_map.png | P9243-GB p.12, Fig.3 |
| 21_P9243_typical_application.png | P9243-GB p.1 |
| 22_P9243_fig15_block_diagram.png | P9243-GB p.19, Fig.15 |
| 23_P9243_fig16_wpc_phases.png | P9243-GB p.21, Fig.16 |
| 24_P9243_fig27_q_factor.png | P9243-GB p.31, Fig.27 |
| 25_P9243_fig28_schematic.png | P9243-GB p.33, Fig.28 |
| 26_P9243_fig11_demod_waveform.png | P9243-GB p.16, Fig.11 |
| 27_P9242_fig18_lc_resonant.png | P9242-R3 p.20, Fig.18 |
| 30_MP28167A_fig2_3_topology.png | MP28167-A p.18, Fig.2–3 |
| 32_MP28167A_fig15_feedback.png | MP28167-A p.28, Fig.15 |
| 33_MP28167A_fig16_pcb_layout.png | MP28167-A p.29, Fig.16 |
| 35_DGD0590A_p2_block.png | DGD0590A p.2, Functional Block Diagram |
| 36_P9243_pinout.png | P9243-GB p.5, Fig.1 |

## 논문
- J. Domajnko, M. Milanovič, N. Prosen, "3D Platform for Coupling Coefficient Evaluation of an Inductive Power Transfer Systems", *Sensors* 22(4):1445, 2022. https://doi.org/10.3390/s22041445

## 저장소에 포함된 그림
`figures/4x_*`, `5x_*`, `6x_*`는 데이터시트 값과 명시한 가정으로 직접 계산·작성한 그림이다.
