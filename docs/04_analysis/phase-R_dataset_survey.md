# Phase R — 아토피 공개 데이터 조사

| 항목 | 내용 |
|---|---|
| 작성일 | 2026-11-11 |
| 작성 | planning-agent |
| 버전 | **v0.4 — Codex 교차 검토 반영** (`docs/05_test/phase-R_codex_review.md` CX-PR-01~04) |
| 상태 | **검토 전** |
| 근거 | D-25 (공개 데이터로 소아 아토피 병변 분할·정량화 모델 재구현, 먼저 데이터 조사), D-26 (Google Colab 학습, 데이터·가중치 커밋 금지) |
| 관련 | PRD 과제 H0~H4·H-CLS, 지표 K-01~K-04 (`docs/01_prd/PRD.md`), 원래 데이터 구조 F-INFO-04 (`docs/decisions.md`) |

### 변경 이력
| 버전 | 날짜 | 내용 |
|---|---|---|
| v0.1 | 2026-11-11 | 최초 작성 (후보 9개, 추천 SCIN · 예비 PASSION) |
| v0.2 | 2026-11-11 | 테스트 리포트 반영: BUG-PR-01(SCIN Eczema 수치 의미 정정 → **추천·예비 순서 교체: 추천 PASSION · 예비 SCIN**), BUG-PR-02~07(H0 기준 설명, 제외 후보 링크, SWET 마스크 작성자, DermaCon-IN 인원, PASSION 피부톤, DermNet 클래스), 제안 BUG-PR-08~12 반영(13은 해당 없음) |
| v0.3 | 2026-11-11 | 재검사 반영: BUG-PR-14(PASSION 414건을 ICD-10 기반 Eczema **군** 사례 수로 정정, "주진단 기준" 표현 삭제, 아토피 비율 ❓ 추가 — 추천 순서는 유지), BUG-PR-15("소아 유일 후보" 단정 수정), BUG-PR-16(LH2·DN3 1차 출처 API 직접 확인 후 표기) |
| v0.4 | 2026-11-11 | Codex 교차 검토 반영(4건 모두 수용): CX-PR-01(PASSION을 **소아 포함이 확인된 잠정 추천**으로 한정, 순위 확정 조건·재검토 기준 추가), CX-PR-02(SWET H2를 **확인 불가**로 통일, 대체 시 K-02 변경 승인 필요), CX-PR-03(H-CLS를 **다른 분류 과제 연습 가능, K-04 재현은 미확인**으로), CX-PR-04(결론을 이번 조사에서 확인한 후보 기준으로 한정, "유일" 단정 삭제). Codex 질문 3개를 RQ-01·RQ-02에 합치고 RQ-11(연령 기준)·RQ-12(추천 재검토 기준) 추가 |

> 표기 규칙: ❓ = 직접 확인하지 못한 값 (6장 질문으로 올림). 출처는 각 칸에 `[약어]` 링크로 달고 전체 목록은 문서 끝 "출처 목록"에 둔다. 데이터 파일은 하나도 내려받지 않았다 (논문 PDF·README·라이선스·메타데이터 API 응답만 열람).

---

## 1. 요약

- **이번 조사에서는 소아 아토피 + 병변 마스크 + EASI 계열 징후 점수를 모두 갖춘 "공개" 데이터를 찾지 못했다.** 이번에 확인한 후보 중 세 가지를 모두 갖춘 것은 SWET(영국 임상시험, 소아 287명, 3-클래스 마스크, 7징후 0~3)뿐인데, 논문이 "동의 제한으로 공개 불가"라고 명시하고 저자 문의만 열어 두었다 [EZ1] [EZ2].
- 즉시 받을 수 있는 아토피 관련 이미지 데이터(SCIN·DermaCon-IN·Fitzpatrick17k)와 신청형 데이터(PASSION)는 모두 **진단명 라벨만** 있고 마스크·중증도 라벨이 없다.
- 따라서 **잠정 추천 = PASSION(소아 포함이 확인된 후보, 질환군 분류 연습용) + ISIC 2018 Task 1(분할 파이프라인용)**, **예비 = SCIN**이다. 순위는 후보별 소아 습진·아토피 사례 수를 확인한 뒤 확정한다(5-1). **이번 조사에서 이용 조건과 라벨을 확인한 후보만으로는** PRD의 H2(면적↔중증도)·H3(4징후)를 재현할 수 없다. 확인한 후보 중 H2·H3에 쓸 라벨을 가진 것은 SWET(H2에 필요한 중증도 등급은 확인 불가)와 Legit.Health-AD(라벨 공개 여부 ❓)이며, 둘 다 저자 문의가 필요하다(RQ-02·RQ-04). 미확인 후보(2-3)와 조사하지 못한 데이터가 남아 있으므로 이것이 공개 데이터 전체의 부재를 뜻하지는 않는다.
- v0.2 변경: v0.1은 "SCIN 습진 사례가 가장 많다"를 추천 근거로 썼으나, 그 수치(1,211·1,056)는 **감별진단 언급 기준**이고 아토피만 센 값도 아니어서 PASSION의 414건(ICD-10 기반 Eczema 군 사례 수)과 같은 잣대로 비교할 수 없다(3-2 (3)). **두 데이터 모두 "Eczema"는 습진 상위 범주이며 아토피피부염 수는 ❓이다** (v0.3 보완). 사례 수 비교가 빠지면, 소아 포함이 확인된 PASSION이 D-25의 "소아" 조건에 더 맞다고 판단해 순서를 바꿨다(5장). v0.4에서 이 판단을 **잠정**으로 낮췄다 — 후보 간 소아 비중·소아 아토피 사례 수 비교는 입증되지 않았다(CX-PR-01).

---

## 2. 조사 방법

### 2-1. 검색어 (WebSearch, 2026-11-11)
- `atopic dermatitis eczema image dataset segmentation masks publicly available EASI severity`
- `eczema segmentation dataset public download pixel-level masks atopic dermatitis 2024 2025 open dataset Zenodo figshare`
- `atopic dermatitis severity grading image dataset open access IGA labels smartphone children publicly released`
- `SWET eczema images dataset EczemaNet data availability`, `EczemaNet2 JID Innovations 2023 ... data availability`
- `SCIN dataset Google skin condition image network license eczema`
- `Fitzpatrick17k dataset license eczema ...`, `DermNet dataset Kaggle "Atopic Dermatitis Photos" ...`, `DermNet NZ image licence ... machine learning`
- `PASSION dataset pediatric dermatology Sub-Saharan Africa eczema`, `DermaCon-IN dataset ...`, `ASCORAD ... Legit.Health dataset ...`
- `Bang 2021 atopic dermatitis EASI deep neural network ...`, `"Eczema Image Resource" EIR dataset ...`, `SD-198 ...`
- `ISIC 2018 Task 1 ... license`, `HAM10000 lesion segmentation masks Harvard Dataverse license`

### 2-2. 확인한 출처 종류
| 종류 | 예 |
|---|---|
| 데이터셋 공식 저장소·README·라이선스 원문 | GitHub (SCIN, Fitzpatrick17k, EczemaNet2, ASCORAD), PASSION 프로젝트 사이트 |
| 데이터 저장소 메타데이터 API | Harvard Dataverse API (HAM10000, DermaCon-IN) — 파일 목록·용량·라이선스 |
| 대회 공식 페이지 | ISIC Challenge 데이터 페이지·Task 1 안내 |
| 논문 본문 (데이터 가용성 문구 확인) | arXiv·Europe PMC 전문, JID Innovations PDF (Imperial 저장소 판본) — 실제 열람 판본은 출처 목록 약어표에 표기 |
| 이미지 제공처 이용약관 | DermNet NZ image licence, 유료 데이터셋 안내 |

### 2-3. 표에서 뺀 것 (존재는 확인했으나 공개 데이터인지 확인 못 함)
| 이름 | 내용 | 뺀 이유 |
|---|---|---|
| Bang et al. 2021 (서울성모병원) | 아토피 임상 사진 8,000장(224×224 병변 크롭), EASI 4징후 0~3 라벨 [BG1] | 논문에 데이터 가용성 문구 없음, 공개 경로 확인 못 함 [BG1] |
| EIR (Eczema Image Resource) | 습진 7개 범주 2,039장 [EI1] | 다운로드 경로 확인 못 함 [EI1] |
| Roboflow "3. Atopic Dermatitis" 인스턴스 분할 | 웹 검색 결과 요약상 52장 (페이지 직접 확인 못 함) [RF1] | 페이지 접속 시도 결과 403 — 라이선스·이미지 출처 확인 못 함 [RF1] |
| SD-198 | 198개 질환 6,584장 임상 사진 (ECCV 2016) [SD1] | 아토피/습진 클래스 포함 여부·라이선스·현재 다운로드 경로 확인 못 함 [SD1] |

---

## 3. 후보 비교

### 3-1. 비교표

후보 9개 = 아토피 관련 7개 + 분할 파이프라인 대안 2개(HAM10000, ISIC 2018 Task 1).

| # | 후보 | ① 아토피 장수 / 소아 비율 | ② 병변 마스크 → H1·H2 | ③ 중증도 라벨 → H2·H3 | ④ 라이선스·절차 | ⑤ 촬영 방식 / 피부톤 | ⑥ Colab 수령 / 용량 |
|---|---|---|---|---|---|---|---|
| 1 | **SWET** (EczemaNet2에 사용) | 1,345장 / 소아 287명, 6개월~16세 (**100% 소아**) [EZ1] | **있음** — 배경·비아토피 피부·아토피 피부 **3-클래스**. 피부과 전문의 4명이 **나눠서 그림(이미지당 1명**: 307·308·308·422장) [EZ1] → H1 가능 / H2는 중증도 등급 확인 불가(4장) | **있음** — 7징후(건조·홍반·찰상·균열·삼출·태선화·부종) 0~3, 대면 평가 [EZ1]. 구진은 별도 항목 아님 → H3 부분 가능. 사진과 연결된 전체 중증도 등급(IGA 등)은 ❓ → H2 확인 불가 | **공개 불가.** "cannot be shared publicly owing to restrictions on the participants' consent" [EZ1] / 접근은 Tanaka 박사 문의 [EZ2] | 디지털 카메라, 임상 스태프 촬영 [EZ1] / Fitzpatrick I–II 1,031장, III–IV 182장, V–VI 132장 [EZ1] | 해당 없음 (승인 전 수령 불가) [EZ2] / 용량 ❓ |
| 2 | **Legit.Health-AD** (ASCORAD 논문) | 604장 + 테스트 367장 + 어두운 피부 112장 / AD 세트의 31%가 18세 미만 [LH1] | **있음(논문 기준)** — 3명 마스크 평균 후 임계값으로 이진 마스크 [LH1]. **공개 여부 ❓** — 저장소 `data/`에는 일치도·분포 요약 CSV만 있음 [LH2] [LH3] | **있음(논문 기준)** — SCORAD 6징후(홍반·부종/구진·삼출/가피·찰상·태선화·건조) 0~3 [LH1]. **공개 여부 ❓** | 데이터 가용성 문구는 이미지 출처(온라인 아틀라스 6곳)만 안내, 라벨·마스크 공유 언급 없음 [LH1]. 저장소에도 데이터 접근 안내 없음 [LH2]. 출처 중 DermNet NZ는 AI 학습 금지 [DN1] | 온라인 피부과 아틀라스 이미지 [LH1] / AD 세트·테스트 세트는 밝은 피부(Fitzpatrick I–III), 별도 세트 IV–VI [LH1] | 라벨 입수 경로 ❓ [LH2] / 용량 ❓ |
| 3 | **SCIN** (Google) | 전체 5,033건·10,408장 [SC2]. "Eczema"(상위 범주, 아토피만이 아님) — 감별진단 **언급 횟수** 1,211(중복 포함, 사례 수 아님) / 가중 감별진단에 포함된 **사례 수** 1,056건(순위·가중치 무관). 최고 가중치 라벨 기준 사례 수·아토피 비율 ❓ [SC5] [SC4] / **소아 0%** — 18세 이상만 참여, 아동 가능 이미지는 자동 필터로 제거 [SC2] | **없음** [SC1] [SC4] → H1·H2 불가 | **없음** (감별진단·가중치·신뢰도 1~5만) [SC4] [SC2] → H2·H3 불가 | SCIN Data Use License. 복제·공유 허용, "You may not make any attempt to re-identify or re-link any of the individual data subjects." [SC3]. 신청 절차 없음 [SC1] | 모바일 기기로 검색하는 미국 성인을 광고로 모집해 기증받음 — 실제 촬영 기기 ❓ [SC2] / 피부과 추정 FST I 7.6%·II 40.2%·III 32.8%·IV 13.6%·V 5.3%·VI 0.6% [SC2], Monk 톤 라벨 포함 [SC4] | GCS 버킷 `dx-scin-public-data`, Colab에서 `auth.authenticate_user()` 후 `google.cloud.storage`로 접근 [SC5] / 용량 ❓ |
| 4 | **PASSION** (MICCAI 2024) | 전체 1,653명·4,901장. **Eczema 군 414건** — 사례별 진단(44종)을 **ICD-10 기반으로 4군에 묶은 값**(사례 수, 표 1 국가 기준 합 57+301+56 — 국가별 합계가 전체 1,653과 일치). 습진 상위 범주이며 **아토피만 센 수가 아님**, 군 안의 아토피 비율 ❓, 이미지 장수 ❓ [PA2] / 소아 중심(동아프리카 세트는 소아만), 소아 비율 수치 ❓ [PA2] | **없음** [PA2] → H1·H2 불가 | **없음** [PA2] (진단 44종 → 4군, 농가진 동반 여부 이진 라벨, 연령·성별·부위) → H2·H3 불가 | PASSION data public license: "You may use the dataset for non-commercial purposes only.", "You must not repost the dataset on any other data hosting platforms or websites." [PA1] / Google Form 신청 [PA1], 심사 기간 ❓ | 현지 휴대폰(원격진료 환경) [PA2] / 기니는 FST **VI가 다수**, 마다가스카르·동아프리카는 **III~V/VI** 분포 [PA2] | 승인 후 안내 링크 ❓ [PA1] / 용량 ❓ |
| 5 | **DermaCon-IN** (NeurIPS 2025) | 전체 5,450장·3,002명(v2), 습진/피부염 하위군 포함, 장수 ❓ [DC1] / 전 연령, 소아 비율 ❓ [DC1] | **없음** (일부 마스크는 향후 공개 예정) [DC1] → H1·H2 불가 | **없음** [DC1] → H2·H3 불가 | **CC BY-NC-SA 4.0** [DC2]. 파일 비제한(restricted=false)이나 fileAccessRequest=true — 실제로 신청이 필요한지 ❓ [DC2] | 스마트폰(108MP 안드로이드, 48MP 아이폰 등) 외래 조명 [DC1] / Fitzpatrick IV–VI, MST 4–9 [DC1] | Harvard Dataverse (DOI 10.7910/DVN/W7OUZM) [DC2] / 이미지 zip 2개 약 1.26 GB + 2.34 GB [DC2] |
| 6 | **Fitzpatrick17k** | 전체 16,577장·114개 질환 [FZ2]. **아토피/습진 클래스 포함 여부·장수 ❓** ("dry skin eczema"는 제외 목록에 있음) [FZ2] / 소아 비율 ❓ | **없음** (논문은 YCbCr 자동 피부 마스크만 사용) [FZ2] → H1·H2 불가 | **없음** [FZ2] → H2·H3 불가 | **CC BY-NC-SA 3.0** [FZ1]. 이미지는 저장소에 없음 — CSV의 원본 URL(일부 깨짐) 또는 요청 양식 [FZ1] | 온라인 아틀라스(DermaAmin 12,672·Atlas Dermatologico 3,905) 임상 사진 [FZ2] / FST 1–2 7,755·3–4 6,089·5–6 2,168·미상 565 [FZ2] | CSV URL 순회 또는 요청 링크 [FZ1] / 용량 ❓ |
| 7 | **DermNet (Kaggle·HF 미러)** | 23개 클래스 중 아토피 관련 "Atopic Dermatitis Photos"·"Eczema Photos" (참고: "Poison Ivy Photos and other Contact Dermatitis"도 있음) [DN3] [DN4], 클래스별 장수 ❓ / 소아 비율 ❓ | 없음 [DN3] | 없음 [DN3] | **사용 불가 판단.** DermNet 이용약관이 "...for the purpose of training or testing artificial intelligence models" 사용을 금지 [DN1]. HF 미러는 라이선스 표기 없음 [DN3]. 공식 유료 데이터셋은 이미지당 과금, 최소 1,000장 [DN2] | DermNet 아틀라스 이미지 ❓ / 피부톤 ❓ | HF 미러 parquet 1.47 GB (train 13,223 / test 2,334) [DN3] [DN4] — **약관상 사용하지 않음** |
| 8 | **HAM10000** (+ 병변 마스크) — 분할 대안 | 아토피 **0** (색소성 병변 7개 범주) [HM1] / 소아 비율 ❓ | **있음** — 10,015장 전체 이진 병변 마스크(피부과 전문의 1명) [HM1] → **분할 파이프라인만** 가능 | 없음 [HM1] | **CC BY-NC 4.0**, 신청 없음 [HM1]. 인용 요청: Sci Data 2018, 마스크는 Nature Medicine 2020 [HM1] | **더모스코피** [HM1] / 피부톤 분포 ❓ | Harvard Dataverse (DOI 10.7910/DVN/DBW86T) [HM1] / 이미지 1.37 GB + 1.40 GB, 마스크 10.8 MB [HM1] |
| 9 | **ISIC 2018 Task 1** — 분할 대안 | 아토피 **0** [IS3] / 소아 비율 ❓ | **있음** — 학습 2,594장·검증 100장·테스트 1,000장 [IS3]. 현재는 **검증·테스트 정답 마스크도 공개** (Validation GT 742 KB, Test GT 9 MB) [IS1]. 이진 PNG(0/255, 단일 연결 영역) [IS2] → **분할 파이프라인만** 가능 | 없음 [IS2] | Task 1: **CC-0** 표기 [IS1], 신청 없음 | **더모스코피** [IS2] / 피부톤 분포 ❓ | `https://isic-archive.s3.amazonaws.com/challenges/2018/<파일명>` 직접 다운로드 [IS1] / 학습 입력 10.4 GB + 정답 26 MB, 검증 228 MB, 테스트 2.2 GB [IS1] |

### 3-2. 후보별 상세

#### (1) SWET — Softened Water Eczema Trial 이미지 (EczemaNet2)
- **우리 원래 설계와 가장 가깝다.** 소아(6개월~16세) 287명 1,345장, 마스크가 **배경·비아토피 피부·아토피 피부 3-클래스**로 F-INFO-04의 "피부/배경, 정상/병변" 구조 및 PRD H1의 3-클래스 설계와 같다 [EZ1].
- 징후 점수: 임상 스태프가 대면으로 7징후를 0~3으로 기록했고, 논문은 이를 영역별 EASI(0~12, 논문 그림 5 설명에 표기)·TISS·SASSAD로 집계해 예측했다 [EZ1]. 논문은 영역별 EASI를 "erythema, excoriation, lichenification, and edema"로 구성한다고 적어 **부종을 EASI의 네 번째 징후로 사용**했다 [EZ1]. PRD의 4징후 중 홍반·찰상·태선화는 같고 구진 대신 부종이 있는 셈인데, SWET 기록 정의가 EASI의 "부종/구진"과 같은지는 ❓.
- 마스크 작성: 전문의 4명이 이미지를 나눠 맡아 **이미지마다 1명이 그렸다** (307·308·308·422장) [EZ1]. 다수 평가자 합의 마스크가 아니다 (Legit.Health-AD의 3명 평균과 대비).
- 한계: 같은 그룹 연구에서 전문의 간 아토피 영역 일치도가 낮다고 보고 [EZ3] → 이미지당 1명 마스크라 라벨 노이즈가 클 수 있다. IGA 라벨 여부 ❓.
- H2 관련: 마스크와 징후 점수가 있어도 K-02(중증도 **등급별** 면적 비율 분포·단조성)를 하려면 **사진과 연결된 중증도 등급**이 필요하다. 논문에서 확인한 것은 사진 단위로 집계한 영역별 EASI·TISS·SASSAD뿐이고, 전체 등급의 유무·평가 단위(사진/부위/환자)·평가 시점(촬영과 같은 방문인지)·정의는 확인하지 못했다 [EZ1] → **H2 확인 불가**(4장).
- 접근: "For privacy concerns, the ... (SWET) dataset that we used in EczemaNet2 is not shareable." / "if you do need to access SWET, please contact Dr Reiko Tanaka" [EZ2]. 코드는 GPLv3 [EZ2].

#### (2) Legit.Health-AD (ASCORAD)
- 이진 병변 마스크 + SCORAD 6징후(0~3) — EASI 4징후(홍반·구진(부종/구진)·찰상·태선화)를 모두 포함 [LH1]. 분할은 U-Net(ResNet-34), 징후는 EfficientNet-B0 멀티출력 — 우리 원래 설계와 유사 [LH1].
- 그러나 이미지는 6개 공개 아틀라스에서 모은 것이고, 데이터 가용성 문구는 **원본 이미지 위치만** 안내하며 마스크·점수 공유는 언급하지 않는다 [LH1]. 저장소 README는 폴더 구조(`lesion_segmentation/ground_truth_masks`, `visual_sign_assessment`)만 보여 주고, 실제 `data/` 폴더에는 평가자 일치도(`*metrics.csv`)·징후별 분포 요약만 있으며 **마스크·이미지별 징후 라벨은 없다** [LH2] [LH3] → 라벨은 비공개일 가능성이 높다(확정은 ❓, RQ-04). 출처 중 DermNet NZ는 AI 학습 금지 조항이 있다 [DN1] → 라벨을 받아도 이미지 이용 조건이 출처별로 다름 ❓.

#### (3) SCIN
- 공개·무신청·명확한 라이선스 [SC1] [SC3].
- **Eczema 수치의 의미 (v0.2 정정)** [SC5] [SC4]:
  - 데모 노트북은 사례(1건 = 1행) 표에서 두 값을 센다. **1,211** = 전문의(최대 3명)가 낸 감별진단 목록에서 "Eczema"가 나온 **횟수**로, 같은 사례를 여러 전문의가 적으면 중복으로 세므로 **사례 수가 아니다**. **1,056** = 가중 감별진단 딕셔너리에 "Eczema"가 들어 있는 **사례 수**로, 3순위처럼 가중치가 작아도 포함된다.
  - 스키마 예시에서 "Nummular eczema"와 "Eczematous dermatitis"가 모두 "Eczema"로 합쳐진다 [SC4] → SCIN의 "Eczema"는 **습진 상위 범주**이고 **아토피피부염만 센 수가 아니다.**
  - 따라서 "최고 가중치 라벨이 습진인 사례 수"와 그중 아토피 비율은 ❓이며, 메타데이터 집계(RQ-06) 전에는 PASSION 414건(ICD-10 기반 Eczema 군, 사례당 군 라벨 1개)과 **같은 잣대로 비교할 수 없다.** 두 수치 모두 아토피피부염만 센 값이 아니다.
- **성인 전용**이 결정적 한계: "Participation was limited to individuals above 18 and in the US." [SC2]. 소아 아토피 목표(D-25)와 대상이 다르다.
- 라벨: 피부과 전문의 최대 3명의 감별진단 + 신뢰도, 가중 라벨, 추정 Fitzpatrick, Monk 톤, 부위·증상·기간 [SC4]. 알려진 이슈: 중복 이미지 15개(42회), 라벨 없는 gradable 48건, 누락 이미지 1장 [SC1].
- 라이선스는 CC 형식의 자체 라이선스로, 공유 시 저작권·라이선스 고지 유지, 수정 시 표시 [SC3]. 연구/상업 구분 조항은 확인 범위에서 없음 — 법적 해석은 ❓.

#### (4) PASSION
- 소아 중심·흔한 소아 질환(습진·진균·옴·농가진)·휴대폰 촬영 → 대상·촬영 환경이 우리 서비스 시나리오에 가깝다 [PA1] [PA2]. 농가진 라벨은 감염성 질환 분류 연습에 쓸 수 있지만, 다른 질환과 함께 붙는 이진 라벨이라 PRD 감염성 군(7종) 전체의 정답을 대신하지 못한다 → K-04 재현은 미확인(4장).
- 습진 414건은 표 1("Number of cases per country and body localization")의 **국가 기준 합**으로 단위는 사례(건)다. 같은 표의 부위별 합(351)과는 다르다 [PA2]. 논문은 "44 conditions grouped into 4 categories ... The grouping is based on the ICD-10 ontology"라고만 적고 "주진단" 같은 표현은 없다 [PA2] → **Eczema는 여러 습진성 진단을 묶은 군**이며, 그 안의 아토피피부염 비율은 ❓. 이미지 장수도 ❓. 농가진은 다른 질환과 함께 나타날 수 있어 별도 이진 열이다 [PA2].
- 피부톤: "a majority of type VI on the FST scale in Guinea", 마다가스카르·동아프리카는 "from type III to type V or VI" [PA2] → 어두운 쪽으로 치우쳐 있어 한국 소아와 분포가 다를 수 있음 (한국 소아 분포 자료는 이번 조사 범위 밖 ❓).
- 비상업·재게시 금지·직접 다운로드 링크 공유 금지 [PA1]. 저장소(Public, D-10)에 링크·파일을 올리지 않도록 주의.

#### (5) DermaCon-IN
- 공개(CC BY-NC-SA 4.0)·스마트폰·245개 진단 + 개념 라벨 [DC1] [DC2]. 습진 장수·연령 분포는 메타데이터(`Skin_Metadata.tab`)에 있을 것으로 보이나 내려받지 않아 ❓ [DC2].
- 인원 표기: 논문 v1은 "약 3,000명(approximately 3,000 patients)", v2는 3,002명 [DC1]. (v0.1의 "v1은 2,993명"은 출처에서 확인되지 않아 삭제.) Dataverse는 v4.0 (2026-01-11) [DC2].

#### (6) Fitzpatrick17k
- 피부톤 라벨이 있는 대표 공개 세트지만 아토피 클래스 존재를 확인하지 못했다 [FZ2] ❓. 이미지 원본 링크 일부가 깨져 있다 [FZ1].

#### (7) DermNet 계열
- Kaggle·HF에 "Atopic Dermatitis Photos"·"Eczema Photos" 클래스를 포함한 23-클래스 미러가 돌지만 [DN3], 원 제공처가 AI 학습·테스트 목적 이용을 명시적으로 금지한다 [DN1]. **학습 목적이라도 사용하지 않는 것을 제안.**

#### (8) HAM10000 + 마스크 / (9) ISIC 2018 Task 1 — 분할 파이프라인 대안
- 둘 다 더모스코피·단일 색소 병변이라 **아토피(경계 모호·산재형, 임상 사진)와 도메인이 크게 다르다.** 여기서 얻은 DSC는 K-01(아토피 병변 DSC ≥ 0.75)의 근거가 되지 못한다.
- 마스크는 **이진(병변/배경)** 이라 PRD H1의 3-클래스(배경·피부·병변) 중 "피부" 클래스가 없다.
- ISIC 2018 Task 1: 공식 분할(2,594/100/1,000)·공식 지표(임계 Jaccard, 0.65 미만은 0점) [IS2] [IS3]·CC-0 [IS1] → 파이프라인 검증·재현에 가장 깔끔. 논문 당시에는 검증·테스트 정답이 비공개였으나 현재는 데이터 페이지에 정답 zip이 있다 [IS1] → D-19의 3분할(학습 / 임계값 조정 / 홀드아웃)을 공식 분할(학습 / 검증 / 테스트)에 그대로 대응시킬 수 있다 (다음 Phase 계획서에서 확정).
- HAM10000: 마스크 10,015장으로 더 많고 용량이 작음(이미지 약 2.8 GB) [HM1], 단 CC BY-NC 4.0. 원래 CLAUDE.md에 "파이프라인 사전 검증용 HAM10000"으로 적혀 있던 후보.

---

## 4. PRD 과제별 재현 가능성 (추천 데이터 기준)

잠정 추천 = **PASSION + ISIC 2018 Task 1**, 예비 = **SCIN + ISIC 2018 Task 1**. 참고로 SWET를 받는 경우도 한 열로 둔다. 판정은 **이번 조사에서 확인한 정보 기준**이며, 판정 값은 "재현 가능 / 부분 가능 / 불가 / 확인 불가(필요한 정보를 함께 적음)" 넷을 쓴다.

> **H0 판정 기준:** PRD의 H0은 "안심존 첫 방문 시 PQ-10 확인"(① 아토피 마스크 보유 장수 ② 사전학습 가중치 반입 ③ 중증도 등급 정의 ④ 정상 피부 클래스 처리)이다. 공개 데이터에는 안심존이 없으므로, 이 표의 H0은 **PQ-10 항목을 공개 데이터에 대입해 다시 정의한 것**이다: ① 마스크 장수 ② 사전학습 가중치 사용 가능 여부(Colab, 반입 제한 없음) ③ 중증도 라벨 정의 ④ 정상 피부·다른 질환 클래스 구성. "재현 가능"은 이 네 항목을 **확인할 수 있다**는 뜻이지, 확인 결과가 좋다는 뜻이 아니다.

| 과제 | 필요한 것 | 잠정 추천 (PASSION + ISIC T1) | 예비 (SCIN + ISIC T1) | 참고: SWET (접근 승인 시) |
|---|---|---|---|---|
| H0 데이터 구조 확인 (공개 데이터용 재정의) | PQ-10 ①~④ 대입 확인 | **재현 가능 (승인 후)** — ① 습진 마스크 0장 ② 가능 ③ 중증도 라벨 없음 ④ 습진·진균·옴·기타 4군 [PA2] [PA1] / ISIC 마스크 2,594장 [IS3] | **재현 가능** — ① 0장 ② 가능 ③ 없음 ④ 다질환 감별진단 [SC4] / ISIC 동일 | 재현 가능 (승인 후) — ① 1,345장 ③ 7징후 0~3 [EZ1] |
| H1 병변 분할 (3-클래스, K-01) | 아토피 병변 마스크 | **부분 가능** — ISIC로 분할 파이프라인(이진)을 만들고 DSC·IoU 측정 가능. 아토피 병변 DSC(K-01)·3-클래스는 불가 (PASSION 마스크 없음) | **부분 가능** — 추천과 같음 | **재현 가능** — 3-클래스 마스크 [EZ1] |
| H2 면적 비율 ↔ 중증도 (K-02) | 아토피 마스크 + 중증도 등급 | **불가** — 마스크·중증도 라벨 모두 없음 | **불가** | **확인 불가** — 필요한 정보: ① 사진과 연결된 전체 중증도 등급(IGA 등) 유무 ② 평가 단위(사진/부위/환자) ③ 평가 시점(촬영과 같은 방문인지) ④ 등급 정의 [EZ1]. 마스크·징후 점수만으로는 K-02가 성립하지 않음. 영역별 EASI 등 징후 합산 점수로 대체하려면 **K-02 정의 변경이라 별도 팀장 승인 필요** (RQ-02·RQ-09) |
| H3 4징후 추정 (K-03) | 홍반·구진·찰상·태선화 0~3 | **불가** | **불가** | **부분 가능** — 홍반·찰상·태선화 있음, 구진 대신 부종(논문도 EASI 4번째 징후로 사용) — 정의 일치 ❓ [EZ1] |
| H-CLS 질환군 분류 (K-04) | 다질환 진단 라벨 | **다른 분류 과제 연습 가능, K-04 재현은 미확인** — PASSION 4군(Eczema·Fungal·Scabies·Others)은 PRD 4군(염증성·선천성·감염성·기타)과 **바로 매핑되지 않음**: Others를 PRD 군으로 나누려면 사례별 세부 진단이 필요(배포 메타데이터에 있는지 ❓), 농가진 이진 라벨은 감염성 군 정답을 대신하지 못함 [PA2]. 확인할 것: 세부 진단 제공 여부 → 매핑표·미매핑 사례·군별 표본 수 | **다른 분류 과제 연습 가능, K-04 재현은 미확인** — 감별진단 라벨(가중치 포함)이라 사례당 정답 군을 정하는 규칙부터 필요, PRD 4군 매핑표·미매핑 사례·군별 표본 수 미확인 [SC4]. 성인 데이터 | 불가 (아토피 단일 질환) |
| H4 스마트폰 일반화 | 스마트폰 사진 + 마스크 | **불가** (마스크 없음). 휴대폰 촬영 사진의 분류 성능은 볼 수 있음 [PA2] | **불가** (DSC 하락폭 측정에 마스크 필요). 촬영 기기 ❓ [SC2] | 불가 (디지털 카메라만) [EZ1] |

---

## 5. 결론

### 5-1. 잠정 추천: PASSION (+ 분할은 ISIC 2018 Task 1) — 소아 포함이 확인된 잠정 후보
- **이유**
  1. D-25 목표의 "소아"에 대해, **소아 포함이 확인된 접근 가능 후보**다: 소아 포함에 공을 들였고 동아프리카 세트는 소아만 [PA2]. 다만 기니·마다가스카르 세트에는 성인도 있고 전체 소아 비율·소아 습진/아토피 사례 수는 ❓ [PA2]. DermaCon-IN도 공개 데이터이며 소아 비율이 ❓ [DC1], SCIN은 소아 0% [SC2]. 따라서 **"소아 비중이 가장 높다"는 비교는 입증되지 않았고**, 이 추천은 아래 확정 조건을 채울 때까지 잠정이다. SWET는 소아 100%지만 비공개 [EZ1].
  2. 라벨 형식이 **사례당 군 라벨 1개**(Eczema·Fungal·Scabies·Others, ICD-10 기반 묶음)라 감별진단이 섞인 SCIN보다 분류 학습 라벨로 쓰기 쉽다 [PA2] [SC4]. 단 Eczema 군 414건은 습진 상위 범주이고 **아토피 수는 ❓**이다 — SCIN과 마찬가지로 "아토피 라벨"로 쓸 수는 없다.
  3. 휴대폰 촬영(원격진료 환경) + 농가진(감염성) 라벨 → 서비스 시나리오와 감염성 질환 분류 연습에 맞다 [PA2]. 단 PRD 4군과 바로 매핑되지 않아 K-04 재현은 미확인(4장).
  4. 분할은 CC-0·공식 분할·공개된 검증/테스트 정답이 있는 ISIC 2018 Task 1으로 EfficientNet-B0 인코더 + U-Net 파이프라인을 먼저 만든다 [IS1] [IS2]. 이 단계는 PASSION 승인 없이 바로 시작할 수 있으므로, **신청 대기 시간이 일정을 막지 않는다.**
- **한계 (팀장 판단 필요)**: 신청·승인 필요(기간 ❓), 비상업·재게시 금지 [PA1], 피부톤이 III~VI(기니는 VI 다수)로 어두운 쪽에 치우침 [PA2], 이미지 장수 ❓, H2·H3 불가.
- **순위 확정 조건** (모두 채우면 추천 확정, CX-PR-01):
  1. **연령 기준 통일** — "소아"를 몇 세로 볼지 정한다 (RQ-11). 데이터마다 연령 표기가 달라(PASSION 분석은 16세 기준 [PA2], SCIN은 18세 이상만 [SC2]) 같은 기준으로 다시 센다.
  2. **후보별 소아 습진·아토피 사례 수 확인** — PASSION(승인 후 메타데이터), DermaCon-IN(`Skin_Metadata.tab`), SCIN(성인 전용이라 0 확인)에서 통일한 연령 기준의 습진 군 사례 수와 그중 아토피피부염 사례 수를 집계한다 (RQ-06).
  3. **PASSION 세부 진단 제공 여부** — 사례별 세부 진단이 있어야 아토피 구분과 PRD 4군 매핑(매핑표·미매핑 사례·군별 표본 수)이 가능하다.
- **추천 재검토 기준** (하나라도 해당하면 예비 SCIN 또는 DermaCon-IN과 다시 비교, RQ-12):
  1. PASSION 신청이 거절되거나 팀장이 정한 기한 안에 승인되지 않음.
  2. PASSION 메타데이터에 사례별 세부 진단이 없어 아토피피부염 사례를 구분할 수 없음.
  3. 통일한 연령 기준에서 PASSION의 소아 아토피 사례 수가 다른 접근 가능 후보(DermaCon-IN 등)보다 적거나, 팀장이 정한 최소 표본 수에 못 미침. (최소 표본 수·기한 값은 이 문서에서 정하지 않는다 — RQ-12.)

### 5-2. 예비: SCIN
- 즉시 사용 가능(신청 없음, 공식 Colab 접근 코드) [SC1] [SC5], 라이선스가 원문으로 명확하고 재식별 금지 외 학습 제약이 적다 [SC3]. 다질환 감별진단 라벨로 H-CLS 연습 가능 [SC4].
- 추천이 아닌 이유: **성인 전용**이라 소아 목표와 대상이 다르다 [SC2]. 습진 수치는 감별진단 포함 기준이고 아토피만 센 것이 아니라서, 최고 가중치 기준 습진 사례 수는 집계 전까지 ❓ [SC5] [SC4].
- PASSION 신청이 거절되거나 늦어지면 SCIN으로 분류 실험을 시작한다.

### 5-3. 순서를 바꾼 이유 (v0.1 → v0.2)
- v0.1은 SCIN을 추천하며 "습진 사례가 가장 많다"(1,211·1,056)를 근거로, PASSION은 "습진 장수가 적다(414건)"를 탈락 사유로 들었다.
- 테스트 리포트(BUG-PR-01)와 재확인 결과, SCIN 수치는 감별진단 언급 횟수·감별진단 포함 사례 수이고 습진 상위 범주라 **PASSION의 414건(ICD-10 기반 Eczema 군 사례 수, 사례당 군 라벨 1개)과 같은 잣대가 아니다.** 또한 재검사(BUG-PR-14)에서 확인했듯 PASSION 414건도 습진 상위 범주라 **양쪽 모두 아토피 수는 ❓**이다. 따라서 "습진 수"는 어느 쪽의 근거도 되지 못한다. 이 근거를 빼면 SCIN의 남은 장점은 "즉시 사용"과 "명확한 라이선스"이고, PASSION의 장점은 "소아 포함 확인"과 "사례당 군 라벨 1개"이다.
- 목표(D-25)가 **소아** 아토피이고, 분할 파이프라인(ISIC)을 먼저 만드는 동안 신청을 기다릴 수 있어 "즉시 사용"의 이점이 줄어든다. 그래서 PASSION을 추천으로 올렸고, v0.4에서 Codex 검토(CX-PR-01)를 받아 **잠정** 추천으로 한정했다(5-1 확정 조건). 피부톤 범위도 v0.1의 "IV–VI 중심"보다 넓은 III~VI로 정정했다(BUG-PR-06). 팀장이 즉시성을 더 중시하면 v0.1 순서로 되돌릴 수 있다(RQ-01 선택지 B).

### 5-4. 마스크·중증도 라벨이 없을 때의 선택지
| 안 | 내용 | 장점 | 단점 |
|---|---|---|---|
| A (제안) | **ISIC 2018 Task 1로 분할 파이프라인**을 만들고, 아토피 데이터(PASSION, 막히면 SCIN)는 **분류에만** 사용. 인코더 공유 멀티태스크 구조는 유지하되 헤드별로 다른 데이터로 학습 | 이번 조사에서 확인한 공개 데이터만으로 시작 가능, 코드 구조(공유 인코더·분할 헤드·분류 헤드)는 원래 설계 재현 | 아토피 분할·면적·4징후 지표(K-01~K-03)는 측정 불가 — "구조 재현"까지만 |
| B | A + **SWET 접근 문의** (RQ-02). 승인되면 H1(3-클래스)·H3(부분)를 SWET로 교체. H2는 중증도 등급 정보를 확인한 뒤 판단(현재 확인 불가 — 4장) | 이번 조사에서 확인한 후보 중 원래 목표(소아 아토피 분할·정량화)에 가장 근접 | 승인 불확실, 동의 제한으로 거절 가능성 [EZ1], 응답 시점 ❓ |
| C | A + Legit.Health-AD 라벨 문의 (RQ-04) | 마스크 + EASI 4징후 포함 징후 점수 [LH1] | 저장소에 라벨 없음 [LH2] [LH3] → 비공개 가능성 높음, 원 이미지 출처 일부가 AI 학습 금지 [DN1] |
| D | 아토피 이미지 일부를 팀장(간호사 경력)이 직접 마스크·징후 라벨링 | 라벨 확보 | 공개 데이터 라이선스상 파생 라벨 배포 조건 확인 필요, 비전문의 라벨 신뢰도·시간 비용. **범위 확대이므로 이 문서에서는 제안만** — 채택 시 backlog·회의 안건 |

---

## 6. 팀장 확인 필요

| ID | 질문 | 왜 필요한가 | 선택지 |
|---|---|---|---|
| RQ-01 | **잠정 추천 조합과 5-4의 안 A**를 승인하는가. 승인하면 D-25 목표를 "소아 아토피 분할·정량화"에서 **"서로 다른 데이터로 학습하는 분할·분류 구조 재현"으로 바꾸는 것**인가 (Codex 질문 1 합침) | 이번 조사에서 확인한 후보로는 H2·H3가 불가. 추천 데이터는 D-25 세 요소(소아·분할·정량화) 중 "소아"만 부분적으로 채운다. 승인 시 **유지할 과제와 포기할 K-01~K-04를 decisions.md에 명시**해야 함 | A) 잠정 추천 PASSION · 예비 SCIN (v0.4 제안, 5-1 확정 조건 충족 시 확정) / B) 추천 SCIN · 예비 PASSION (v0.1, 즉시성 우선) / C) A 또는 B에 안 B·C(SWET·Legit.Health 문의) 병행 / D) 다른 조합 |
| RQ-02 | **SWET 접근 문의**(Imperial College, Tanaka 박사 [EZ2])를 할 것인가. 문의한다면 접근 가능 여부와 함께 ① 사진과 연결된 전체 중증도 등급 유무 ② 사진과 임상 평가의 연결 방식(평가 단위·시점) ③ 구진·부종 정의까지 물을 것인가 (Codex 질문 2 합침) | 이번 조사에서 확인한 후보 중 소아·3-클래스 마스크·징후 점수를 모두 가진 것은 SWET뿐. ①~③이 없으면 승인되어도 H2는 확인 불가로 남음(4장). 개인 학습 목적 요청이 받아들여질지, 응답 시점 ❓ | A) 문의함 — ①~③ 포함 (팀장 명의 이메일) / B) 문의함 — 접근 여부만 / C) 안 함 |
| RQ-03 | **PASSION 신청**을 지금 할 것인가 (Google Form, 비상업·재게시 금지 [PA1]) | v0.2 추천의 전제. 심사 기간 ❓ — ISIC 분할 작업과 병행하려면 일찍 신청해야 함 | A) 지금 신청 / B) RQ-01 결정 후 |
| RQ-04 | Legit.Health-AD **마스크·이미지별 징후 라벨 공개 여부**를 저자에게 문의할 것인가 | 논문에 공유 언급 없음, 저장소 `data/`에는 일치도·분포 요약만 있음 [LH1] [LH2] [LH3] | A) 문의 / B) 안 함 |
| RQ-05 | 예비 SCIN이 **성인 전용**(18세 이상 [SC2])인 점을 받아들이는가 | PASSION이 막힐 때 소아가 아닌 데이터로 진행하게 됨 | A) 수용(학습 목적) / B) 예비를 DermaCon-IN(전 연령, 소아 비율 ❓)으로 |
| RQ-06 | ❓ 수치 확인을 위해 **메타데이터 표 파일만**(이미지 제외: SCIN `scin_cases.csv`·`scin_labels.csv`, DermaCon-IN `Skin_Metadata.tab`, Fitzpatrick17k CSV) Colab에서 읽어 집계해도 되는가 | SCIN 최고 가중치 라벨 기준 습진 수, PASSION Eczema 군 안의 아토피 비율(승인 후), DermaCon-IN 습진 장수·연령, Fitzpatrick17k 습진 클래스를 확인해야 후보 비교가 같은 잣대가 됨 | A) 허용 / B) 승인 이후 Phase에서 |
| RQ-07 | SCIN 접근은 Colab에서 **Google 계정 인증**이 필요하다 [SC5]. 팀장 개인 Google 계정 사용이 괜찮은가 | 예비안 접근 방식 확정 | A) 개인 계정 / B) 별도 계정 |
| RQ-08 | 분할을 **이진(병변/배경)** 으로 시작해도 되는가 | 공개 마스크(ISIC·HAM)는 이진뿐, PRD H1은 3-클래스 | A) 이진으로 시작, SWET 확보 시 3-클래스 / B) 3-클래스 고수(피부 클래스는 별도 방법 필요) |
| RQ-09 | 공개 데이터 기준 **목표 지표**(K-01~K-04 대체)를 언제 정할 것인가 | K-01(DSC ≥ 0.75)은 대회 데이터 공식 베이스라인 기준이라 ISIC 더모스코피에 그대로 적용하는 것은 의미가 다름. PRD 수정은 PRD 담당 영역 | A) 다음 Phase 계획서에서 / B) 지금 회의에서 |
| RQ-10 | DermNet 계열(Kaggle·HF 미러)을 **사용하지 않는** 것에 동의하는가 | 원 제공처 약관이 AI 학습·테스트를 금지 [DN1] | A) 사용 안 함 (제안) / B) 다른 판단 |
| RQ-11 | 후보 비교에 쓸 **"소아" 연령 기준**을 무엇으로 할 것인가 (CX-PR-01) | 순위 확정 조건 1. 데이터마다 연령 표기가 다름(PASSION 분석 16세 기준 [PA2], SCIN 18세 이상만 [SC2]) | A) PRD 1차 타겟과 같은 만 0~9세 / B) 18세 미만 / C) 데이터가 제공하는 구간에 맞춤(예: 16세 이하) |
| RQ-12 | **추천 재검토 기준**(5-1)의 값 — PASSION 승인 대기 **기한**과 소아 아토피 **최소 표본 수**를 어떻게 정할 것인가 (Codex 질문 3) | PASSION 수령 후 소아 아토피 표본이 부족하거나 세부 진단이 없으면 추천을 다시 검토해야 하는데, 기준 값이 없으면 판단할 수 없음. 값은 이 문서에서 임의로 정하지 않음 | A) 팀장이 값 지정 / B) 다음 Phase 계획서에서 근거(학습에 필요한 표본 수 등)와 함께 제안받기 |

### 6-1. 확인하지 못한 항목 (❓ 목록)
| # | 항목 | 해당 후보 | 관련 RQ |
|---|---|---|---|
| 1 | SWET 데이터 용량 | SWET | RQ-02 |
| 2 | SWET "부종" 기록 정의가 EASI "부종/구진"과 같은지 (논문은 부종을 EASI 4번째 징후로 사용 [EZ1] — 근거는 있으나 정의 일치는 미확인) | SWET | RQ-02 |
| 3 | SWET 사진과 연결된 전체 중증도 등급(IGA 등) 유무, 평가 단위·시점·정의 (H2 판정에 필요) | SWET | RQ-02 |
| 4 | Legit.Health-AD 마스크·징후 라벨 공개 여부·입수 경로 (저장소에는 없음 확인 [LH2] [LH3]) | Legit.Health-AD | RQ-04 |
| 5 | Legit.Health-AD 출처 이미지별 이용 조건 | Legit.Health-AD | RQ-04 |
| 6 | Legit.Health-AD 용량 | Legit.Health-AD | RQ-04 |
| 7 | SCIN 최고 가중치 라벨 기준 습진 사례 수와 그중 아토피 비율. (확인됨: 1,211 = 감별진단 언급 횟수, 1,056 = 감별진단 포함 사례 수, 단위 = 사례 [SC5] [SC4]) | SCIN | RQ-06 |
| 8 | SCIN 실제 촬영 기기 (모집은 모바일 검색 사용자 대상 [SC2]) | SCIN | RQ-06 |
| 9 | SCIN 용량 | SCIN | RQ-06 |
| 10 | SCIN 라이선스의 상업/비상업 해석 | SCIN | RQ-01 |
| 11 | PASSION 습진 **이미지** 장수 (Eczema 군 사례 수 414건은 확인됨 [PA2]) | PASSION | RQ-03 |
| 12 | PASSION 소아 비율 | PASSION | RQ-03 |
| 13 | PASSION 심사 기간 | PASSION | RQ-03 |
| 14 | PASSION 수령 방식·용량 | PASSION | RQ-03 |
| 15 | DermaCon-IN 습진 장수 | DermaCon-IN | RQ-06 |
| 16 | DermaCon-IN 소아 비율 | DermaCon-IN | RQ-06 |
| 17 | DermaCon-IN 접근 신청 필요 여부 | DermaCon-IN | RQ-06 |
| 18 | Fitzpatrick17k 아토피/습진 클래스 유무·장수 | Fitzpatrick17k | RQ-06 |
| 19 | Fitzpatrick17k 소아 비율·용량 | Fitzpatrick17k | RQ-06 |
| 20 | DermNet 미러 클래스별 장수·소아 비율·피부톤 | DermNet | (사용 안 함 제안, RQ-10) |
| 21 | HAM10000 소아 비율·피부톤 분포 | HAM10000 | — (분할 파이프라인용이라 진행 영향 작음) |
| 22 | ISIC 2018 Task 1 소아 비율·피부톤 분포 | ISIC 2018 T1 | — (위와 같음) |
| 23 | 한국 소아 피부톤 분포와 후보 데이터 분포 비교 | 공통 | 조사 범위 밖 |
| 24 | **PASSION Eczema 군(414건) 안의 아토피피부염 비율** — 논문은 44개 진단을 ICD-10 기반 4군으로 묶었다고만 적음 [PA2]. 승인 후 메타데이터의 세부 진단으로 확인 | PASSION | RQ-03 |
| 25 | PASSION 사례별 세부 진단 제공 여부와 PRD 4군 매핑표·미매핑 사례·군별 표본 수 (H-CLS·K-04 판정에 필요) | PASSION | RQ-03 |
| 26 | SCIN 감별진단 라벨의 사례당 정답 군 규칙과 PRD 4군 매핑표·군별 표본 수 | SCIN | RQ-06 |
| 27 | 통일한 연령 기준에서 후보별(PASSION·DermaCon-IN) 소아 습진·아토피피부염 사례 수 (추천 순위 확정 조건) | PASSION·DermaCon-IN | RQ-06·RQ-11 |

> 참고: Colab 무료 세션의 디스크 여유는 이번에 확인하지 않았다. ISIC 2018 Task 1 학습 입력 zip이 10.4 GB라 [IS1] 압축 해제까지 고려하면 Google Drive 보관 또는 축소본 생성이 필요할 수 있다 — 다음 Phase 계획서에서 확인한다.
>
> 참고(BUG-PR-13): 이후 문서에서 SCIN·PASSION의 "진단/감별진단" 라벨을 다룰 때, 이 표현을 사용자 화면 문구로 옮기지 않는다 (CLAUDE.md 3-1).

---

## 출처 목록

[SC1]: https://github.com/google-research-datasets/scin
[SC2]: https://arxiv.org/html/2402.18545v1
[SC3]: https://github.com/google-research-datasets/scin/blob/main/LICENSE
[SC4]: https://github.com/google-research-datasets/scin/blob/main/dataset_schema.md
[SC5]: https://github.com/google-research-datasets/scin/blob/main/scin_demo.ipynb
[PA1]: https://passionderm.github.io/
[PA2]: https://arxiv.org/html/2411.04584
[DC1]: https://arxiv.org/html/2506.06099v2
[DC2]: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/W7OUZM
[FZ1]: https://github.com/mattgroh/fitzpatrick17k
[FZ2]: https://arxiv.org/abs/2104.09957
[EZ1]: https://doi.org/10.1016/j.xjidi.2023.100213
[EZ2]: https://github.com/Tanaka-Group/EczemaNet2
[EZ3]: https://doi.org/10.1016/j.xjidi.2022.100133
[LH1]: https://pmc.ncbi.nlm.nih.gov/articles/PMC9382656/
[LH2]: https://github.com/Legit-Health/ASCORAD
[LH3]: https://api.github.com/repos/Legit-Health/ASCORAD/git/trees/HEAD?recursive=1
[DN1]: https://dermnetnz.org/image-licence
[DN2]: https://dermnetnz.org/dermatology-image-dataset
[DN3]: https://huggingface.co/datasets/WahajRaza/Dermnet
[DN4]: https://huggingface.co/api/datasets/WahajRaza/Dermnet
[IS1]: https://challenge.isic-archive.com/data/
[IS2]: https://challenge.isic-archive.com/landing/2018/45/
[IS3]: https://arxiv.org/abs/1902.03368
[HM1]: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/DBW86T
[BG1]: https://pmc.ncbi.nlm.nih.gov/articles/PMC7961024/
[EI1]: https://doi.org/10.1016/j.compbiomed.2022.105807
[RF1]: https://universe.roboflow.com/ta-oondr/3.-atopic-dermatitis-lastj
[SD1]: https://doi.org/10.1007/978-3-319-46466-4_13

| 약어 | 내용 | 실제 열람 판본 (링크와 다를 때) |
|---|---|---|
| SC1~SC5 | SCIN 저장소 · 논문(arXiv 2402.18545) · 라이선스 · 스키마 · 데모 노트북 | 라이선스·스키마·노트북은 `raw.githubusercontent.com/google-research-datasets/scin/main/` 원문 |
| PA1~PA2 | PASSION 프로젝트 사이트 · 논문(arXiv 2411.04584) | — |
| DC1~DC2 | DermaCon-IN 논문(arXiv 2506.06099v2) · Harvard Dataverse | DC2는 Dataverse API `api/datasets/:persistentId/?persistentId=doi:10.7910/DVN/W7OUZM` |
| FZ1~FZ2 | Fitzpatrick17k 저장소 · 논문(arXiv 2104.09957) | FZ2는 `ar5iv.labs.arxiv.org/html/2104.09957` |
| EZ1~EZ3 | Attar et al. 2023 JID Innov (EczemaNet2) · EczemaNet2 저장소 · Hurault et al. 2022 JID Innov 2:100133 (전문의 간 일치도) | EZ1은 Imperial 저장소 PDF `spiral.imperial.ac.uk/server/api/core/bitstreams/7ae165d2-4892-43e5-ac48-83ae11664ecd/content` (같은 전문: Europe PMC PMC10504536). EZ3은 medRxiv 10.1101/2022.03.03.22271780 검색 결과로 존재 확인, 본문 미열람 |
| LH1~LH3 | Medela et al. 2022 ASCORAD 논문 · 저장소 · 저장소 파일 목록 API | LH1은 Europe PMC `ebi.ac.uk/europepmc/webservices/rest/PMC9382656/fullTextXML`. LH2 `data/` 폴더 내용은 [LH3] GitHub 트리 API를 planning-agent가 직접 조회해 확인 (v0.3): `data/annotation/*metrics.csv`(평가자 일치도), `data/distributions/*`(징후별 분포·예측 요약 CSV와 그래프 PNG), `data/lesion-segmentation/*metrics.csv`만 있고 사진·마스크 PNG·이미지별 징후 라벨 파일은 없음. 각 CSV의 내용은 열어 보지 않음(파일 목록 기준 판단) |
| DN1~DN4 | DermNet NZ 이미지 라이선스(25항) · 유료 데이터셋 안내 · HF 미러 · HF 미러 메타데이터 API | DN3 클래스 23개·분할 수(train 13,223 / test 2,334)·download_size 1,473,388,407 B·라이선스 태그 없음은 [DN4] HF API를 planning-agent가 직접 조회해 확인 (v0.3) |
| IS1~IS3 | ISIC Challenge 데이터 페이지 · 2018 Task 1 안내 · ISIC 2018 논문(arXiv 1902.03368) | IS3은 PDF 본문 |
| HM1 | HAM10000 Harvard Dataverse | Dataverse API `...persistentId=doi:10.7910/DVN/DBW86T` |
| BG1 | Bang et al. 2021 Sci Rep | Europe PMC `ebi.ac.uk/europepmc/webservices/rest/PMC7961024/fullTextXML` |
| EI1 | Rasheed et al. 2022 Comput Biol Med (EIR) | 검색 결과 서지만 확인 |
| RF1 | Roboflow Universe "3. Atopic Dermatitis" | 접속 403, 웹 검색 결과 요약만 확인 |
| SD1 | Sun et al. ECCV 2016 (SD-198) | 검색 결과 서지만 확인 |
