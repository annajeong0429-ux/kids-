# Phase R — 아토피 공개 데이터 조사

| 항목 | 내용 |
|---|---|
| 작성일 | 2026-11-11 |
| 작성 | planning-agent |
| 상태 | **검토 전** |
| 근거 | D-25 (공개 데이터로 소아 아토피 병변 분할·정량화 모델 재구현, 먼저 데이터 조사), D-26 (Google Colab 학습, 데이터·가중치 커밋 금지) |
| 관련 | PRD 과제 H0~H4·H-CLS, 지표 K-01~K-04 (`docs/01_prd/PRD.md`), 원래 데이터 구조 F-INFO-04 (`docs/decisions.md`) |

> 표기 규칙: ❓ = 직접 확인하지 못한 값 (6장 질문으로 올림). 출처는 각 칸에 `[약어]` 링크로 달고 전체 목록은 문서 끝 "출처 목록"에 둔다. 데이터 파일은 하나도 내려받지 않았다 (논문 PDF·README·라이선스·메타데이터 API 응답만 열람).

---

## 1. 요약

- **소아 아토피 + 병변 마스크 + EASI 계열 징후 점수를 모두 갖춘 "공개" 데이터는 찾지 못했다.** 세 가지를 모두 갖춘 데이터는 SWET(영국 임상시험, 소아 287명, 3-클래스 마스크, 7징후 0~3)뿐인데, 논문이 "동의 제한으로 공개 불가"라고 명시하고 저자 문의만 열어 두었다 [EZ1] [EZ2].
- 즉시 받을 수 있는 아토피 이미지 데이터(SCIN·DermaCon-IN·Fitzpatrick17k)와 신청형 데이터(PASSION)는 모두 **진단명 라벨만** 있고 마스크·중증도 라벨이 없다.
- 따라서 **추천 = SCIN(아토피·질환 분류용) + ISIC 2018 Task 1(분할 파이프라인용)**, **예비 = PASSION**이며, PRD의 H2(면적↔중증도)·H3(4징후)는 **공개 데이터만으로는 재현 불가**다. H2·H3를 하려면 SWET 접근 문의(RQ-02)가 사실상 유일한 길이다.

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
| 논문 본문 (데이터 가용성 문구 확인) | arXiv·medRxiv·Europe PMC 전문, JID Innovations PDF (Imperial 저장소) |
| 이미지 제공처 이용약관 | DermNet NZ image licence, 유료 데이터셋 안내 |

### 2-3. 표에서 뺀 것 (존재는 확인했으나 공개 데이터가 아님)
| 이름 | 내용 | 뺀 이유 |
|---|---|---|
| Bang et al. 2021 (서울성모병원) | 아토피 임상 사진 8,000장(224×224 병변 크롭), EASI 4징후 0~3 라벨 [BG1] | 논문에 데이터 가용성 문구 없음, 공개 경로 확인 못 함 [BG1] |
| EIR (Eczema Image Resource) | 습진 7개 범주 2,039장 [EI1] | 다운로드 경로 확인 못 함 |
| Roboflow "3. Atopic Dermatitis" 인스턴스 분할 | 검색 결과상 52장 | 페이지 접속 불가(403)로 라이선스·출처 확인 못 함 |
| SD-198 | 198개 질환 6,584장 임상 사진 | 아토피/습진 클래스 포함 여부·라이선스 확인 못 함 |

---

## 3. 후보 비교

### 3-1. 비교표

후보 9개 = 아토피 관련 7개 + 분할 파이프라인 대안 2개(HAM10000, ISIC 2018 Task 1).

| # | 후보 | ① 아토피 장수 / 소아 비율 | ② 병변 마스크 → H1·H2 | ③ 중증도 라벨 → H2·H3 | ④ 라이선스·절차 | ⑤ 촬영 방식 / 피부톤 | ⑥ Colab 수령 / 용량 |
|---|---|---|---|---|---|---|---|
| 1 | **SWET** (EczemaNet2에 사용) | 1,345장 / 소아 287명, 6개월~16세 (**100% 소아**) [EZ1] | **있음** — 배경·비아토피 피부·아토피 피부 **3-클래스**, 피부과 전문의 4명 [EZ1] → H1·H2 가능 | **있음** — 7징후(건조·홍반·찰상·균열·삼출·태선화·부종) 0~3, 대면 평가 [EZ1]. 구진은 별도 항목 아님 | **공개 불가.** "cannot be shared publicly owing to restrictions on the participants' consent" [EZ1] / 접근은 Tanaka 박사 문의 [EZ2] | 디지털 카메라, 임상 스태프 촬영 [EZ1] / Fitzpatrick I–II 1,031장, III–IV 182장, V–VI 132장 [EZ1] | 해당 없음 (승인 전 수령 불가) [EZ2] / 용량 ❓ |
| 2 | **Legit.Health-AD** (ASCORAD 논문) | 604장 + 테스트 367장 + 어두운 피부 112장 / AD 세트의 31%가 18세 미만 [LH1] | **있음(논문 기준)** — 3명 마스크 평균 후 임계값으로 이진 마스크 [LH1]. **공개 여부 ❓** | **있음(논문 기준)** — SCORAD 6징후(홍반·부종/구진·삼출/가피·찰상·태선화·건조) 0~3 [LH1]. **공개 여부 ❓** | 데이터 가용성 문구는 이미지 출처(온라인 아틀라스 6곳)만 안내, 라벨·마스크 공유 언급 없음 [LH1]. 코드 저장소에도 데이터 접근 안내 없음 [LH2]. 출처 중 DermNet NZ는 AI 학습 금지 [DN1] | 온라인 피부과 아틀라스 이미지 [LH1] / AD 세트·테스트 세트는 밝은 피부(Fitzpatrick I–III), 별도 세트 IV–VI [LH1] | 라벨 입수 경로 ❓ [LH2] / 용량 ❓ |
| 3 | **SCIN** (Google) | 전체 5,033건·10,408장 [SC2]. 데모 노트북 출력상 Eczema 라벨 1,211(피부과 라벨)·1,056(가중 라벨) — 단위(건/장) ❓ [SC5] / **소아 0%** — 18세 이상만 참여, 아동 가능 이미지는 자동 필터로 제거 [SC2] | **없음** [SC1] [SC4] → H1·H2 불가 | **없음** (진단명·감별진단 가중치·신뢰도 1~5만) [SC4] [SC2] → H2·H3 불가 | SCIN Data Use License. 복제·공유 허용, "You may not make any attempt to re-identify or re-link any of the individual data subjects." [SC3]. 신청 절차 없음 [SC1] | 미국 구글 검색 사용자의 자발적 기증(모바일 검색 광고로 모집), 촬영 기기 명시 없음 ❓ [SC2] / 피부과 추정 FST I 7.6%·II 40.2%·III 32.8%·IV 13.6%·V 5.3%·VI 0.6% [SC2], Monk 톤 라벨 포함 [SC4] | GCS 버킷 `dx-scin-public-data`, Colab에서 `auth.authenticate_user()` 후 `google.cloud.storage`로 접근 [SC5] / 용량 ❓ |
| 4 | **PASSION** (MICCAI 2024) | 전체 1,653명·4,901장, Eczema 군 414건(논문 표 1을 본 조사자가 합산 — 원표 재확인 ❓), 장수 ❓ [PA2] / 소아 중심(동아프리카 세트는 소아만), 소아 비율 수치 ❓ [PA2] | **없음** [PA2] → H1·H2 불가 | **없음** [PA2] (진단 44종 → 4군, 농가진 동반 여부 이진 라벨, 연령·성별·부위) → H2·H3 불가 | PASSION data public license: "You may use the dataset for non-commercial purposes only.", "You must not repost the dataset on any other data hosting platforms or websites." [PA1] / Google Form 신청 [PA1], 심사 기간 ❓ | 현지 휴대폰(원격진료 환경) [PA2] / Fitzpatrick IV–VI 중심, 결과표에 III도 있음 [PA2] | 승인 후 안내 링크 ❓ [PA1] / 용량 ❓ |
| 5 | **DermaCon-IN** (NeurIPS 2025) | 전체 5,450장·3,002명, 습진/피부염 하위군 포함, 장수 ❓ [DC1] / 전 연령, 소아 비율 ❓ [DC1] | **없음** (일부 마스크는 향후 공개 예정) [DC1] → H1·H2 불가 | **없음** [DC1] → H2·H3 불가 | **CC BY-NC-SA 4.0** [DC2]. 파일 비제한(restricted=false)이나 fileAccessRequest=true — 실제로 신청이 필요한지 ❓ [DC2] | 스마트폰(108MP 안드로이드, 48MP 아이폰 등) 외래 조명 [DC1] / Fitzpatrick IV–VI, MST 4–9 [DC1] | Harvard Dataverse (DOI 10.7910/DVN/W7OUZM) [DC2] / 이미지 zip 2개 약 1.26 GB + 2.34 GB [DC2] |
| 6 | **Fitzpatrick17k** | 전체 16,577장·114개 질환 [FZ2]. **아토피/습진 클래스 포함 여부·장수 ❓** ("dry skin eczema"는 제외 목록에 있음) [FZ2] / 소아 비율 ❓ | **없음** (논문은 YCbCr 자동 피부 마스크만 사용) [FZ2] → H1·H2 불가 | **없음** [FZ2] → H2·H3 불가 | **CC BY-NC-SA 3.0** [FZ1]. 이미지는 저장소에 없음 — CSV의 원본 URL(일부 깨짐) 또는 요청 양식 [FZ1] | 온라인 아틀라스(DermaAmin 12,672·Atlas Dermatologico 3,905) 임상 사진 [FZ2] / FST 1–2 7,755·3–4 6,089·5–6 2,168·미상 565 [FZ2] | CSV URL 순회 또는 요청 링크 [FZ1] / 용량 ❓ |
| 7 | **DermNet (Kaggle·HF 미러)** | 23개 클래스 중 "Atopic Dermatitis Photos" 클래스 존재 [DN3], 장수 ❓ / 소아 비율 ❓ | 없음 [DN3] | 없음 [DN3] | **사용 불가 판단.** DermNet 이용약관이 "...for the purpose of training or testing artificial intelligence models" 사용을 금지 [DN1]. HF 미러는 라이선스 표기 없음 [DN3]. 공식 유료 데이터셋은 이미지당 과금, 최소 1,000장 [DN2] | DermNet 아틀라스 이미지 ❓ / 피부톤 ❓ | HF 미러 parquet 1.47 GB [DN3] — **약관상 사용하지 않음** |
| 8 | **HAM10000** (+ 병변 마스크) — 분할 대안 | 아토피 **0** (색소성 병변 7개 범주) [HM1] / 소아 비율 ❓ | **있음** — 10,015장 전체 이진 병변 마스크(피부과 전문의 1명) [HM1] → **분할 파이프라인만** 가능 | 없음 [HM1] | **CC BY-NC 4.0**, 신청 없음 [HM1]. 인용 요청: Sci Data 2018, 마스크는 Nature Medicine 2020 [HM1] | **더모스코피** [HM1] / 피부톤 분포 ❓ | Harvard Dataverse (DOI 10.7910/DVN/DBW86T) [HM1] / 이미지 1.37 GB + 1.40 GB, 마스크 10.8 MB [HM1] |
| 9 | **ISIC 2018 Task 1** — 분할 대안 | 아토피 **0** [IS3] / 소아 비율 ❓ | **있음** — 학습 2,594장(마스크 포함), 검증 100·테스트 1,000장 [IS3], 이진 PNG(0/255, 단일 연결 영역) [IS2] → **분할 파이프라인만** 가능 | 없음 [IS2] | Task 1: **CC-0** 표기 [IS1], 신청 없음 | **더모스코피** [IS2] / 피부톤 분포 ❓ | `https://isic-archive.s3.amazonaws.com/challenges/2018/<파일명>` 직접 다운로드 [IS1] / 학습 입력 10.4 GB + 정답 26 MB, 검증 228 MB, 테스트 2.2 GB [IS1] |

### 3-2. 후보별 상세

#### (1) SWET — Softened Water Eczema Trial 이미지 (EczemaNet2)
- **우리 원래 설계와 가장 가깝다.** 소아(6개월~16세) 287명 1,345장, 마스크가 **배경·비아토피 피부·아토피 피부 3-클래스**로 F-INFO-04의 "피부/배경, 정상/병변" 구조 및 PRD H1의 3-클래스 설계와 같다 [EZ1].
- 징후 점수: 임상 스태프가 대면으로 7징후를 0~3으로 기록했고, 논문은 이를 영역별 EASI(0~12)·TISS·SASSAD로 집계해 예측했다 [EZ1]. PRD의 4징후 중 **홍반·찰상·태선화는 이름이 같고, 구진은 별도 항목이 없으며 "부종(edema)"이 있다.** EASI 원 정의의 해당 항목이 "부종/구진"이라 대응 가능성은 있으나, SWET 기록 정의와 같은지 ❓.
- 한계: 같은 그룹 연구에서 전문의 간 아토피 영역 일치도가 낮다고 보고 [EZ3] → 마스크 라벨 노이즈가 크다. IGA 라벨 여부 ❓.
- 접근: "For privacy concerns, the ... (SWET) dataset that we used in EczemaNet2 is not shareable." / "if you do need to access SWET, please contact Dr Reiko Tanaka" [EZ2]. 코드는 GPLv3 [EZ2].

#### (2) Legit.Health-AD (ASCORAD)
- 이진 병변 마스크 + SCORAD 6징후(0~3) — EASI 4징후(홍반·구진(부종/구진)·찰상·태선화)를 모두 포함 [LH1]. 분할은 U-Net(ResNet-34), 징후는 EfficientNet-B0 멀티출력 — 우리 원래 설계와 유사 [LH1].
- 그러나 이미지는 6개 공개 아틀라스에서 모은 것이고, 데이터 가용성 문구는 **원본 이미지 위치만** 안내하며 마스크·점수 공유는 언급하지 않는다 [LH1]. 저장소는 폴더 구조(`lesion_segmentation/ground_truth_masks`, `visual_sign_assessment`)만 보여 준다 [LH2]. 출처 중 DermNet NZ는 AI 학습 금지 조항이 있다 [DN1] → 라벨을 받아도 이미지 이용 조건이 출처별로 다름 ❓.

#### (3) SCIN
- 공개·무신청·명확한 라이선스·습진 사례가 가장 많은 공개 세트(데모 출력에서 Eczema가 1위 라벨) [SC5].
- **성인 전용**이 결정적 한계: "Participation was limited to individuals above 18 and in the US." [SC2]. 소아 아토피 목표(D-25)와 대상이 다르다.
- 라벨: 피부과 전문의 최대 3명의 감별진단 + 신뢰도, 가중 라벨, 추정 Fitzpatrick, Monk 톤, 부위·증상·기간 [SC4]. 알려진 이슈: 중복 이미지 15개(42회), 라벨 없는 gradable 48건, 누락 이미지 1장 [SC1].
- 라이선스는 CC 형식의 자체 라이선스로, 공유 시 저작권·라이선스 고지 유지, 수정 시 표시 [SC3]. 연구/상업 구분 조항은 확인 범위에서 없음 — 법적 해석은 ❓.

#### (4) PASSION
- 소아 중심·흔한 소아 질환(습진·진균·옴·농가진)·휴대폰 촬영 → 대상·촬영 환경은 우리 서비스 시나리오에 가깝다 [PA1] [PA2]. 농가진 라벨은 감염성 군(K-04 감염성 재현율) 연습에 쓸 수 있다.
- 피부톤이 어두운 쪽(IV–VI)에 몰려 있어 한국 소아와 분포가 다를 수 있음 (한국 소아 분포 자료는 이번 조사 범위 밖 ❓).
- 비상업·재게시 금지·직접 다운로드 링크 공유 금지 [PA1]. 저장소(Public, D-10)에 링크·파일을 올리지 않도록 주의.

#### (5) DermaCon-IN
- 공개(CC BY-NC-SA 4.0)·스마트폰·245개 진단 + 개념 라벨 [DC1] [DC2]. 습진 장수·연령 분포는 메타데이터(`Skin_Metadata.tab`)에 있을 것으로 보이나 내려받지 않아 ❓ [DC2].
- 버전: 논문 v1은 2,993명, v2는 3,002명으로 표기가 다름 [DC1] — Dataverse는 v4.0 (2026-01-11) [DC2].

#### (6) Fitzpatrick17k
- 피부톤 라벨이 있는 대표 공개 세트지만 아토피 클래스 존재를 확인하지 못했다 [FZ2] ❓. 이미지 원본 링크 일부가 깨져 있다 [FZ1].

#### (7) DermNet 계열
- Kaggle·HF에 "Atopic Dermatitis Photos" 클래스를 포함한 23-클래스 미러가 돌지만, 원 제공처가 AI 학습·테스트 목적 이용을 명시적으로 금지한다 [DN1]. **학습 목적이라도 사용하지 않는 것을 제안.**

#### (8) HAM10000 + 마스크 / (9) ISIC 2018 Task 1 — 분할 파이프라인 대안
- 둘 다 더모스코피·단일 색소 병변이라 **아토피(경계 모호·산재형, 임상 사진)와 도메인이 크게 다르다.** 여기서 얻은 DSC는 K-01(아토피 병변 DSC ≥ 0.75)의 근거가 되지 못한다.
- 마스크는 **이진(병변/배경)** 이라 PRD H1의 3-클래스(배경·피부·병변) 중 "피부" 클래스가 없다.
- ISIC 2018 Task 1: 공식 분할(2,594/100/1,000)·공식 지표(임계 Jaccard, 0.65 미만은 0점) [IS2] [IS3]·CC-0 [IS1] → 파이프라인 검증·재현에 가장 깔끔.
- HAM10000: 마스크 10,015장으로 더 많고 용량이 작음(이미지 약 2.8 GB) [HM1], 단 CC BY-NC 4.0. 원래 CLAUDE.md에 "파이프라인 사전 검증용 HAM10000"으로 적혀 있던 후보.

---

## 4. PRD 과제별 재현 가능성 (추천 데이터 기준)

추천 = **SCIN + ISIC 2018 Task 1**, 예비 = **PASSION + ISIC 2018 Task 1**. 참고로 SWET를 받는 경우도 한 열로 둔다.

| 과제 | 필요한 것 | 추천 (SCIN + ISIC T1) | 예비 (PASSION + ISIC T1) | 참고: SWET (접근 승인 시) |
|---|---|---|---|---|
| H0 데이터 구조 확인 | 스키마·분포 확인 | **재현 가능** — 공개 스키마 [SC4]·ISIC 파일 구성 [IS1]으로 Colab EDA | **재현 가능** — 단 승인 후 [PA1] | 재현 가능 (승인 후) |
| H1 병변 분할 (3-클래스, K-01) | 아토피 병변 마스크 | **부분 가능** — ISIC로 분할 파이프라인(이진)을 만들고 DSC·IoU 측정 가능. 아토피 병변 DSC(K-01)·3-클래스는 불가 (SCIN 마스크 없음) | **부분 가능** — 추천과 같음 | **재현 가능** — 3-클래스 마스크 [EZ1] |
| H2 면적 비율 ↔ 중증도 (K-02) | 아토피 마스크 + 중증도 등급 | **불가** — 마스크·중증도 라벨 모두 없음 | **불가** | **부분 가능** — 마스크·징후 점수 있음. IGA 같은 전체 등급 유무 ❓, 영역별 EASI 등으로 대체 검토 필요 |
| H3 4징후 추정 (K-03) | 홍반·구진·찰상·태선화 0~3 | **불가** | **불가** | **부분 가능** — 홍반·찰상·태선화 있음, 구진은 "부종"으로 대응 가능 여부 ❓ [EZ1] |
| H-CLS 질환군 분류 (K-04) | 다질환 진단 라벨 | **부분 가능** — 피부과 감별진단 라벨로 다질환 분류 가능 [SC4]. PRD의 4군(염증성·선천성·감염성·기타) 체계와 라벨이 달라 매핑 필요, 성인 데이터 | **부분 가능** — 습진·진균·옴·기타 4군 + 농가진 라벨 [PA2]. PRD 4군과 다름 | 불가 (아토피 단일 질환) |
| H4 스마트폰 일반화 | 스마트폰 사진 + 마스크 | **불가** (DSC 하락폭 측정에 마스크 필요). SCIN 촬영 기기도 ❓ [SC2] | **불가** (마스크 없음). 휴대폰 촬영 분류 성능은 볼 수 있음 [PA2] | 불가 (디지털 카메라만) [EZ1] |

---

## 5. 결론

### 5-1. 추천: SCIN (+ 분할은 ISIC 2018 Task 1)
- **이유**
  1. 즉시 사용 가능: 신청 절차 없음, 공식 Colab 접근 코드 제공 [SC1] [SC5].
  2. 라이선스가 원문으로 명확하고 재식별 금지 외 학습 제약이 적다 [SC3].
  3. 공개 세트 중 습진 사례가 가장 많이 확인됨(데모 출력 기준) [SC5], 다질환 라벨로 H-CLS 연습 가능.
  4. 분할은 CC-0·공식 분할·공식 지표가 있는 ISIC 2018 Task 1으로 EfficientNet-B0 인코더 + U-Net 파이프라인을 먼저 완성 [IS1] [IS2].
- **한계 (팀장 판단 필요)**: 성인만 → "소아" 아토피 재현이 아니다. H2·H3 불가.

### 5-2. 예비: PASSION
- 소아 중심·휴대폰 촬영·농가진(감염성) 라벨 → 대상이 목표에 더 가깝다 [PA1] [PA2].
- 추천이 아닌 이유: 신청·승인 필요(기간 ❓), 비상업·재게시 금지 조건, 피부톤이 IV–VI에 집중, 습진 장수가 적다(414건, 재확인 ❓).

### 5-3. 마스크·중증도 라벨이 없을 때의 선택지
| 안 | 내용 | 장점 | 단점 |
|---|---|---|---|
| A (제안) | **ISIC 2018 Task 1로 분할 파이프라인**을 만들고, 아토피 데이터(SCIN 또는 PASSION)는 **분류에만** 사용. 인코더 공유 멀티태스크 구조는 유지하되 헤드별로 다른 데이터로 학습 | 공개 데이터만으로 즉시 시작, 코드 구조(공유 인코더·분할 헤드·분류 헤드)는 원래 설계 재현 | 아토피 분할·면적·4징후 지표(K-01~K-03)는 측정 불가 — "구조 재현"까지만 |
| B | A + **SWET 접근 문의** (RQ-02). 승인되면 H1(3-클래스)·H2·H3를 SWET로 교체 | 원래 목표(소아 아토피 분할·정량화)에 가장 근접 | 승인 불확실, 동의 제한으로 거절 가능성 [EZ1], 응답 시점 ❓ |
| C | A + Legit.Health-AD 라벨 문의 (RQ-04) | 마스크 + EASI 4징후 포함 징후 점수 [LH1] | 라벨 공개 여부 ❓, 원 이미지 출처 일부가 AI 학습 금지 [DN1] |
| D | 아토피 이미지 일부를 팀장(간호사 경력)이 직접 마스크·징후 라벨링 | 라벨 확보 | 공개 데이터 라이선스상 파생 라벨 배포 조건 확인 필요, 비전문의 라벨 신뢰도·시간 비용. **범위 확대이므로 이 문서에서는 제안만** — 채택 시 backlog·회의 안건 |

---

## 6. 팀장 확인 필요

| ID | 질문 | 왜 필요한가 | 선택지 |
|---|---|---|---|
| RQ-01 | 추천안(SCIN + ISIC 2018 Task 1, 예비 PASSION)과 5-3의 **안 A**를 승인하는가 | 공개 데이터만으로는 H2·H3가 불가 → D-25의 "정량화" 범위를 "구조 재현 + 분류"로 줄일지 결정 필요 | A) 승인 / B) 안 B·C 병행 / C) 다른 조합 |
| RQ-02 | **SWET 접근 문의**(Imperial College, Tanaka 박사 [EZ2])를 할 것인가 | 소아·3-클래스 마스크·징후 점수를 모두 가진 유일한 데이터. 개인 학습 목적 요청이 받아들여질지, 응답 시점 ❓ | A) 문의함 (팀장 명의 이메일) / B) 안 함 |
| RQ-03 | **PASSION 신청**을 할 것인가 (Google Form, 비상업·재게시 금지 [PA1]) | 예비안 사용 전제. 심사 기간 ❓ | A) 지금 신청 / B) 추천안이 막힐 때만 |
| RQ-04 | Legit.Health-AD **마스크·징후 라벨 공개 여부**를 저자에게 문의할 것인가 | 논문·저장소 어디에도 라벨 입수 경로가 없음 ❓ [LH1] [LH2] | A) 문의 / B) 안 함 |
| RQ-05 | SCIN이 **성인 전용**(18세 이상 [SC2])인 점을 받아들이는가 | 목표가 "소아" 아토피이므로 대상 불일치 | A) 수용(학습 목적) / B) PASSION을 주 데이터로 승격 |
| RQ-06 | ❓ 수치 확인을 위해 **메타데이터 표 파일만**(이미지 제외: SCIN `scin_cases.csv`·`scin_labels.csv`, DermaCon-IN `Skin_Metadata.tab`, Fitzpatrick17k CSV) Colab에서 읽어 집계해도 되는가 | 이번 작업은 "데이터 내려받기 금지"라 습진 장수·연령·용량을 확인 못 함 | A) 허용 / B) 승인 이후 Phase에서 |
| RQ-07 | SCIN 접근은 Colab에서 **Google 계정 인증**이 필요하다 [SC5]. 팀장 개인 Google 계정 사용이 괜찮은가 | 접근 방식 확정 | A) 개인 계정 / B) 별도 계정 |
| RQ-08 | 분할을 **이진(병변/배경)** 으로 시작해도 되는가 | 공개 마스크(ISIC·HAM)는 이진뿐, PRD H1은 3-클래스 | A) 이진으로 시작, SWET 확보 시 3-클래스 / B) 3-클래스 고수(피부 클래스는 별도 방법 필요) |
| RQ-09 | 공개 데이터 기준 **목표 지표**(K-01~K-04 대체)를 언제 정할 것인가 | K-01(DSC ≥ 0.75)은 대회 데이터 공식 베이스라인 기준이라 ISIC 더모스코피에 그대로 적용하는 것은 의미가 다름. PRD 수정은 PRD 담당 영역 | A) 다음 Phase 계획서에서 / B) 지금 회의에서 |
| RQ-10 | DermNet 계열(Kaggle·HF 미러)을 **사용하지 않는** 것에 동의하는가 | 원 제공처 약관이 AI 학습·테스트를 금지 [DN1] | A) 사용 안 함 (제안) / B) 다른 판단 |

### 6-1. 확인하지 못한 항목 (❓ 목록)
| # | 항목 | 해당 후보 | 관련 RQ |
|---|---|---|---|
| 1 | SWET 데이터 용량 | SWET | RQ-02 |
| 2 | SWET "부종" 항목이 EASI의 구진 항목과 같은 정의인지 | SWET | RQ-02 |
| 3 | SWET IGA 등 전체 중증도 등급 유무 | SWET | RQ-02 |
| 4 | Legit.Health-AD 마스크·징후 라벨 공개 여부·입수 경로 | Legit.Health-AD | RQ-04 |
| 5 | Legit.Health-AD 출처 이미지별 이용 조건 | Legit.Health-AD | RQ-04 |
| 6 | Legit.Health-AD 용량 | Legit.Health-AD | RQ-04 |
| 7 | SCIN 습진 라벨 집계 단위(건/장) | SCIN | RQ-06 |
| 8 | SCIN 촬영 기기 (스마트폰 여부) | SCIN | RQ-06 |
| 9 | SCIN 용량 | SCIN | RQ-06 |
| 10 | SCIN 라이선스의 상업/비상업 해석 | SCIN | RQ-01 |
| 11 | PASSION 습진 장수(건수 414는 합산값 재확인) | PASSION | RQ-03 |
| 12 | PASSION 소아 비율 | PASSION | RQ-03 |
| 13 | PASSION 심사 기간 | PASSION | RQ-03 |
| 14 | PASSION 수령 방식·용량 | PASSION | RQ-03 |
| 15 | DermaCon-IN 습진 장수 | DermaCon-IN | RQ-06 |
| 16 | DermaCon-IN 소아 비율 | DermaCon-IN | RQ-06 |
| 17 | DermaCon-IN 접근 신청 필요 여부 | DermaCon-IN | RQ-06 |
| 18 | Fitzpatrick17k 아토피/습진 클래스 유무·장수 | Fitzpatrick17k | RQ-06 |
| 19 | Fitzpatrick17k 소아 비율·용량 | Fitzpatrick17k | RQ-06 |
| 20 | DermNet 미러 아토피 장수·소아 비율·피부톤 | DermNet | (사용 안 함 제안, RQ-10) |
| 21 | HAM10000 소아 비율·피부톤 분포 | HAM10000 | — (분할 파이프라인용이라 진행 영향 작음) |
| 22 | ISIC 2018 Task 1 소아 비율·피부톤 분포 | ISIC 2018 T1 | — (위와 같음) |
| 23 | 한국 소아 피부톤 분포와 후보 데이터 분포 비교 | 공통 | 조사 범위 밖 |

> 참고: Colab 무료 세션의 디스크 여유는 이번에 확인하지 않았다. ISIC 2018 Task 1 학습 입력 zip이 10.4 GB라 [IS1] 압축 해제까지 고려하면 Google Drive 보관 또는 축소본 생성이 필요할 수 있다 — 다음 Phase 계획서에서 확인한다.

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
[EZ3]: https://www.medrxiv.org/content/10.1101/2022.03.03.22271780v1.full
[LH1]: https://pmc.ncbi.nlm.nih.gov/articles/PMC9382656/
[LH2]: https://github.com/Legit-Health/ASCORAD
[DN1]: https://dermnetnz.org/image-licence
[DN2]: https://dermnetnz.org/dermatology-image-dataset
[DN3]: https://huggingface.co/datasets/WahajRaza/Dermnet
[IS1]: https://challenge.isic-archive.com/data/
[IS2]: https://challenge.isic-archive.com/landing/2018/45/
[IS3]: https://arxiv.org/abs/1902.03368
[HM1]: https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/DBW86T
[BG1]: https://pmc.ncbi.nlm.nih.gov/articles/PMC7961024/
[EI1]: https://doi.org/10.1016/j.compbiomed.2022.105807

| 약어 | 내용 |
|---|---|
| SC1~SC5 | SCIN 저장소 · 논문(arXiv 2402.18545) · 라이선스 · 스키마 · 데모 노트북 |
| PA1~PA2 | PASSION 프로젝트 사이트 · 논문(arXiv 2411.04584) |
| DC1~DC2 | DermaCon-IN 논문(arXiv 2506.06099v2) · Harvard Dataverse |
| FZ1~FZ2 | Fitzpatrick17k 저장소 · 논문(arXiv 2104.09957) |
| EZ1~EZ3 | Attar et al. 2023 JID Innov (EczemaNet2, 본문 PDF 열람 — 데이터 가용성·윤리 문구) · EczemaNet2 저장소 · Hurault et al. 2022 (전문의 간 일치도) |
| LH1~LH2 | Medela et al. 2022 ASCORAD 논문 · 저장소 |
| DN1~DN3 | DermNet NZ 이미지 라이선스(25항) · 유료 데이터셋 안내 · HF 미러 |
| IS1~IS3 | ISIC Challenge 데이터 페이지 · 2018 Task 1 안내 · ISIC 2018 논문(arXiv 1902.03368) |
| HM1 | HAM10000 Harvard Dataverse |
| BG1 | Bang et al. 2021 Sci Rep (Europe PMC 전문 열람) |
| EI1 | Rasheed et al. 2022 Comput Biol Med (EIR) |
