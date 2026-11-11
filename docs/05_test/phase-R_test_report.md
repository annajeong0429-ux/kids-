# Phase R 테스트 리포트 — 아토피 공개 데이터 조사

| 항목 | 내용 |
|---|---|
| 작성일 | 2026-11-11 |
| 작성 | test-agent |
| 검사 대상 | `docs/04_analysis/phase-R_dataset_survey.md` (239줄, 커밋 `f0a03a0`, 브랜치 `docs/restart-phase-r`) |
| 기준 문서 | `CLAUDE.md` 4장 ⑤ "데이터 조사 (Phase R)" 행, `docs/decisions.md` D-24~D-27, `docs/01_prd/PRD.md` H0~H4·H-CLS(147~152줄)·K-01~K-04(236~239줄) |
| 독립성 | Codex 리뷰(`phase-R_codex_review.md`)는 읽지 않음 (아직 없음) |
| 데이터 다운로드 | **하지 않음.** 데이터 파일(zip·parquet·이미지·CSV 라벨)은 HEAD 요청만. 논문 본문·README·LICENSE·스키마 문서·Dataverse 메타데이터 API·GitHub 트리 API(파일 목록)·데모 노트북(코드·출력 텍스트)만 열람 |

---

## 1. 결론 요약

| 심각도 | 건수 |
|---|---|
| 🔴 차단 | 0 |
| 🟠 주요 | 1 |
| 🟡 경미 | 6 |
| 💡 제안 | 6 |

- 출처 링크 25개 중 24개가 HTTP 200/202로 열림. 1개(EZ3, medRxiv)는 봇 차단(403)이지만 medRxiv API로 논문 존재는 확인.
- **핵심 후보의 라이선스 원문 인용 6건은 모두 원문과 글자 단위로 일치**(SCIN, PASSION 2건, DermNet 25항, EczemaNet2 README, EczemaNet2 논문 데이터 가용성 문구). ISIC 2018 Task 1 CC-0, HAM10000 CC BY-NC 4.0, DermaCon-IN CC BY-NC-SA 4.0, Fitzpatrick17k CC BY-NC-SA 3.0도 출처와 일치.
- 비교표 9개 후보 모두 비교 기준 6개 열을 갖췄고, 확인 못 한 값은 대체로 ❓로 표시됨. 수치 대조 결과도 대부분 정확함(4장 참조).
- 주요 문제는 1건: **추천 데이터 SCIN의 "Eczema 1,211 / 1,056" 수치를 잘못 해석**함. 이 숫자는 아토피 장수가 아니라 감별진단 목록 안에서 "Eczema"가 나온 횟수(중복 포함)이고, 단위(건/장)도 확인 가능한데 ❓로 남겨 두었다. 이 수치가 추천 이유 3과 PASSION 비교 근거로 쓰였다.
- 차단 이슈는 없음. 추천과 한계(성인 전용, H2·H3 재현 불가)를 RQ-01·RQ-05로 팀장에게 올려 두었으므로, BUG-PR-01을 고친 뒤 회의를 진행해도 된다.

---

## 2. 실행한 명령어와 결과

### 2-1. 출처 링크 유효성 (HEAD, 리다이렉트 추적)
명령:
```bash
grep -oP '^\[[A-Z]+[0-9]\]: \K\S+' docs/04_analysis/phase-R_dataset_survey.md | while read u; do
  curl -sIL -o /dev/null -m 25 -A "Mozilla/5.0" -w "%{http_code} %{url_effective}" "$u"; echo "  <= $u"; done
```
결과: **25개 중 200 = 22, 202 = 2, 403 = 1**

| 약어 | 코드 | 비고 |
|---|---|---|
| SC1~SC5 | 200 ×5 | |
| PA1, PA2 | 200 ×2 | |
| DC1 | 200 | |
| DC2 | 202 | Harvard Dataverse (정상 응답, 메타데이터 API로 내용 확인) |
| FZ1, FZ2 | 200 ×2 | |
| EZ1 | 200 | DOI → `linkinghub.elsevier.com/.../S2667026723000371` |
| EZ2 | 200 | |
| **EZ3** | **403** | medRxiv 봇 차단으로 보임. `api.biorxiv.org` 메타데이터로 제목 "Detecting eczema areas in digital images: an impossible task?" 확인, 정식 출판 DOI `10.1016/j.xjidi.2022.100133` |
| LH1, LH2 | 200 ×2 | |
| DN1~DN3 | 200 ×3 | |
| IS1~IS3 | 200 ×3 | |
| HM1 | 202 | Harvard Dataverse |
| BG1 | 200 | |
| EI1 | 200 | DOI → Elsevier |

추가 HEAD (데이터 파일, 본문 미수신):
```bash
curl -sI https://isic-archive.s3.amazonaws.com/challenges/2018/ISIC2018_Task1_Training_GroundTruth.zip   # HTTP/1.1 200 OK
curl -sI https://isic-archive.s3.amazonaws.com/challenges/2018/ISIC2018_Task1-2_Training_Input.zip        # 200, Content-Length: 11165358566 (= 10.4 GiB)
```
→ 문서의 ISIC 직접 다운로드 경로 형식 [IS1]과 학습 입력 10.4 GB 일치.

### 2-2. 라이선스·수치 대조 (텍스트 페이지·메타데이터만)
| 명령 (요약) | 대조 항목 | 결과 |
|---|---|---|
| `curl raw.githubusercontent.com/google-research-datasets/scin/main/LICENSE` + grep | SC3 재식별 금지 인용, 상업 이용 조항 | 일치 (Section 3 b. Restrictions). "commercial" 단어 없음 → 문서의 "연구/상업 구분 조항 없음" 일치 |
| `curl arxiv.org/html/2402.18545v1` → 태그 제거 후 grep | SC2 "above 18" 인용, 5,033건·10,408장, 아동 이미지 자동 필터, eFST 분포 | 모두 일치. 표 S10 eFST 7.55/40.21/32.76/13.64/5.27/0.57 % → 문서 7.6/40.2/32.8/13.6/5.3/0.6 % |
| `curl scin_demo.ipynb` (노트북 JSON, 출력 텍스트만) | SC5 Eczema 1,211·1,056, GCS 버킷, `auth.authenticate_user` | 숫자·버킷명·인증 코드 일치. **단, 숫자의 의미는 문서 해석과 다름 → BUG-PR-01** |
| `curl scin/main/dataset_schema.md`, `README.md` | SC4 라벨 구조, SC1 알려진 이슈(중복 15/42, 48건, 누락 1장) | 일치 |
| `curl passionderm.github.io` | PA1 인용 2건, Google Form 신청 | 원문과 일치 ("You may use the dataset for non-commercial purposes only.", "You must not repost the dataset on any other data hosting platforms or websites.") |
| `curl arxiv.org/html/2411.04584` | PA2 1,653명·4,901장, 표 1 습진 57+301+56, EAS 소아만, 44종→4군, 농가진 이진 라벨, 휴대폰 촬영 | 모두 일치. 표 1 합산 414 **확인됨**(국가별 합계 384/985/284 = 1,653과도 맞음) |
| `curl dermnetnz.org/image-licence` | DN1 25항 인용 | 일치 ("...for the purpose of training or testing artificial intelligence models...") |
| `curl dermnetnz.org/dermatology-image-dataset` | DN2 이미지당 과금, 최소 1,000장 | 일치 ("Licence fees are charged per image", "1,000 (minimum order quantity)") |
| `curl huggingface.co/api/datasets/WahajRaza/Dermnet` | DN3 23클래스, 1.47 GB, 라이선스 표기 | 일치 (download_size 1,473,388,407 B, license 태그 없음) |
| `curl challenge.isic-archive.com/data/` | IS1 Task 1 CC-0, 용량 | 일치 (10.4GB / 26MB / 228MB / 2.2GB, CC-0) |
| `curl challenge.isic-archive.com/landing/2018/45/` | IS2 이진 PNG 0/255, 단일 연결 영역, 임계 Jaccard 0.65 | 일치 |
| `curl arxiv.org/pdf/1902.03368` → `pdftotext` | IS3 2,594/100/1,000 | 일치 |
| `curl raw.githubusercontent.com/Tanaka-Group/EczemaNet2/master/README.md`, GitHub API | EZ2 인용 2건, GPLv3 | 일치 (spdx GPL-3.0) |
| `curl ebi.ac.uk/europepmc/.../PMC10504536/fullTextXML` | EZ1 데이터 가용성 인용, 1,345장·287명·6개월~16세, 3-클래스 마스크, 7징후 0~3 대면, FST 1,031/182/132 | 모두 일치. 마스크 작성 방식은 BUG-PR-04 |
| `curl ebi.ac.uk/europepmc/.../PMC9382656/fullTextXML` | LH1 604/367/112장, 표 10 <18세 31 %, 이진 마스크(평균 후 임계 155), SCORAD 6징후, 아틀라스 6곳, Fitzpatrick I–III / IV–VI, ResNet-34·EfficientNet-B0 | 모두 일치 |
| GitHub API `repos/Legit-Health/ASCORAD/git/trees/HEAD?recursive=1`, readme | LH2 "폴더 구조만 보여 준다" | 일치 — readme 60~62줄의 폴더 구조, 실제 `data/` 폴더에는 일치도·분포 요약 CSV만 있고 마스크·징후 라벨 없음 |
| Dataverse API `api/datasets/:persistentId/?persistentId=doi:10.7910/DVN/W7OUZM` | DC2 CC BY-NC-SA 4.0, fileAccessRequest, v4.0 2026-01-11, zip 용량 | 일치 (1,263,404,816 B / 2,336,232,934 B, restricted=False, fileAccessRequest=True) |
| Dataverse API `...DVN/DBW86T` | HM1 CC BY-NC 4.0, 용량 | 일치 (1.37 GB / 1.40 GB / 마스크 10.8 MB) |
| `curl github.com/mattgroh/fitzpatrick17k` README, `arxiv.org/pdf/2104.09957` → `pdftotext` | FZ1 CC BY-NC-SA 3.0, 깨진 URL·요청 양식 / FZ2 16,577장·114질환, 아틀라스 12,672·3,905, FST 7,755/6,089/2,168/565, "dry skin eczema" 제외, YCbCr | 모두 일치 |
| `curl arxiv.org/html/2506.06099v2`, `arxiv.org/pdf/2506.06099v1` → `pdftotext` | DC1 5,450장·3,002명, 스마트폰 기종, 마스크 향후 공개, v1 2,993명 | v2 수치 일치. **v1 "2,993명"은 찾지 못함 → BUG-PR-05** |
| `api.biorxiv.org/details/medrxiv/10.1101/2022.03.03.22271780` | EZ3 존재 | 확인 (본문 일치도 수치는 미대조) |

### 2-3. 비밀값·개인정보·데이터 파일 검사
```bash
grep -n -i -E "[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[a-z]{2,}|api[_-]?key|token|secret|password|sk-[A-Za-z0-9]|AKIA|010-[0-9]" docs/04_analysis/phase-R_dataset_survey.md
# → 일치 없음 (exit 1)
find . -path ./.git -prune -o -type f \( -iname "*.pth" -o -iname "*.pt" -o -iname "*.zip" -o -iname "*.parquet" -o -iname "*.csv" -o -iname "*.jpg" -o -iname "*.png" -o -iname "*.npy" -o -iname "*.h5" \) -print
# → 결과 없음
git diff --stat main...HEAD   # 문서 파일(.md)과 .claude/settings.json만 변경
```
→ 실제 데이터·가중치·비밀값 없음. EczemaNet2 README에 있는 연구자 이메일도 문서에 옮기지 않았다(이름 "Tanaka 박사"만 표기). 6-3장 안 D의 "팀장(간호사 경력)"은 이미 공개된 `CLAUDE.md` 11줄에 있는 내용이라 새로운 노출이 아니다.

### 2-4. 미실행
- Colab에서 SCIN·DermaCon-IN 메타데이터 CSV 집계: **미실행** (데이터 내려받기 금지, RQ-06 승인 사항)
- EZ3 본문(전문의 간 일치도 수치) 대조: **미실행** (medRxiv 403)
- EI1(EIR)·BG1(Bang 2021) 본문 대조: **미실행** (표에서 뺀 후보라 우선순위 낮음, 링크 200만 확인)
- Fitzpatrick17k 114개 클래스에 아토피/습진이 있는지: **미실행** (확인하려면 CSV 라벨 파일을 받아야 함)

---

## 3. 이슈 목록

### 🟠 BUG-PR-01 — SCIN "Eczema 1,211 / 1,056" 수치를 아토피 사례 수처럼 해석, 단위 ❓는 확인 가능
- **위치:** 3-1 비교표 3행 ①열(65줄), 3-2 (3) 86줄, 5-1 이유 3(135줄), 5-2 141줄("습진 장수가 적다(414건)"), 6-1 ❓ #7(177줄)
- **확인 방법:**
  1. `scin_demo.ipynb` 20번 셀: 데이터프레임은 `cases_and_labels_df`(사례 1건 = 1행, `scin_cases.csv` + `scin_labels.csv` 결합). 1,211은 `dermatologist_skin_condition_on_label_name` 목록 원소를 `Counter.update()`로 센 값, 1,056은 `weighted_skin_condition_label` 딕셔너리 **키**를 센 값.
  2. `dataset_schema.md`: "There may be duplicate entries in this list. A duplicate entry means more than 1 dermatologist labeller included that condition" — 전문의 최대 3명 × 각 최대 3개의 감별진단 목록.
  3. 같은 문서의 예시: "Nummular eczema"와 "Eczematous dermatitis"가 모두 "Eczema"로 합쳐진다.
- **기대:** ① 단위는 **사례(건)**. ② 1,211은 "감별진단 언급 횟수(같은 사례 중복 포함)"라 사례 수가 아니다. ③ 1,056은 "가중 감별진단 어딘가에 Eczema가 들어 있는 사례 수"이며, 가중치가 작아도(예: 3순위) 포함된다. ④ SCIN의 "Eczema"는 화폐상 습진 등을 합친 상위 범주라 **아토피피부염 수가 아니다**. 이 내용을 문서에 적고 ❓ #7은 "확인됨: 건 단위, 감별진단 포함 기준"으로 바꿔야 한다.
- **실제:** ①열 "아토피 장수 / 소아 비율"에 Eczema 1,211·1,056을 적고 단위만 ❓로 둠. 5-1 이유 3("습진 사례가 가장 많이 확인됨")과 5-2(PASSION 습진 414건이 적다)가 이 수치를 근거로 씀. 그런데 PASSION 414는 **주진단 기준 사례 수**(표 1, 습진 군)이고 SCIN 1,211은 **감별진단 언급 횟수**라 같은 잣대가 아니다. 추천 순위(SCIN > PASSION)의 근거 하나가 과대평가된 비교 위에 서 있다.
- **영향:** 팀장이 RQ-01·RQ-05(SCIN 추천 승인)를 결정할 때 쓰는 핵심 근거. "주진단이 습진인 사례 수"는 RQ-06 승인 후 메타데이터로 다시 세야 하므로, 그 전까지는 "Eczema가 감별진단에 포함된 사례 1,056건(주진단 기준 수는 ❓)"처럼 적는 것이 맞다.
- **담당:** planning-agent

### 🟡 BUG-PR-02 — PRD H0 정의를 바꿔서 "재현 가능"으로 판정
- **위치:** 4장 표 H0 행(120줄)
- **확인 방법:** PRD 147줄 H0 = "데이터 구조 확인 — 안심존 첫 방문 시 PQ-10 항목 확인". PQ-10(PRD 327줄) = ① 아토피 마스크 보유 장수 ② 사전학습 가중치 반입 ③ 중증도 등급 정의 ④ 정상 피부 클래스 처리.
- **기대:** H0을 "공개 데이터 기준으로 다시 정의한 것"임을 밝히거나, PQ-10 항목별로 판정(SCIN: ① 마스크 0, ③ 중증도 라벨 없음 → 확인할 대상 자체가 없음).
- **실제:** "스키마·분포 확인"으로 바꿔 적고 추천·예비·SWET 모두 "재현 가능"으로 표시. 틀린 판정은 아니지만, PRD 정의와 다른 기준을 쓴다는 설명이 없다.
- **담당:** planning-agent

### 🟡 BUG-PR-03 — 표에서 뺀 후보 2개에 출처 링크 없음
- **위치:** 2-3 표 50줄(Roboflow "3. Atopic Dermatitis"), 51줄(SD-198)
- **확인 방법:** 두 행에 `[약어]` 표기가 없고 출처 목록(201~225줄)에도 해당 URL이 없음.
- **기대:** CLAUDE.md 4장 ⑤ "모든 후보에 출처 링크". 제외한 후보라도 존재를 확인한 근거(검색 결과 URL, 논문 DOI)를 단다. Roboflow는 403이었더라도 시도한 URL을 남긴다.
- **실제:** 링크 없음. "검색 결과상 52장"의 출처를 추적할 수 없다.
- **담당:** planning-agent

### 🟡 BUG-PR-04 — SWET "피부과 전문의 4명" 표현이 마스크 작성 방식을 오해하게 함
- **위치:** 3-1 비교표 1행 ②열(63줄), 3-2 (1) 한계(78줄)
- **확인 방법:** EZ1 본문(Europe PMC PMC10504536): "Four dermatologists delineated (segmented) AD regions at the pixel level in the 1,345 images (307 images by BO, 308 images by EE, 308 images by LS, and 422 images by HCW)". 앞쪽에 "Four dermatologists segmented the AD regions from each of the 1,345 photographs"라는 문장도 있지만, 괄호 안 분담 수를 더하면 1,345 → **이미지마다 1명이 그림**.
- **기대:** "전문의 4명이 나눠서, 이미지당 1명" 으로 적는다. 78줄의 "라벨 노이즈" 한계와 직접 관련된 정보다(평균·합의 마스크가 아님. Legit.Health-AD의 3명 평균과도 대비됨).
- **실제:** "피부과 전문의 4명"만 적혀 있어 다수 평가자 합의 마스크로 읽힐 수 있다.
- 덧붙임: 77줄 "영역별 EASI(0~12)"의 범위 0~12는 논문에 적혀 있지 않고 4징후 × 0~3에서 계산한 값으로 보인다. 계산값이면 그렇다고 표시한다.
- **담당:** planning-agent

### 🟡 BUG-PR-05 — DermaCon-IN "논문 v1은 2,993명" 확인 불가
- **위치:** 3-2 (5) 98줄
- **확인 방법:** `arxiv.org/abs/2506.06099v1` 초록과 v1 PDF 전체(`pdftotext`)에서 "2,993"·"2993" 검색 → 없음. v1은 "approximately 3,000 patients"·"3,000 Indian patients"로 적혀 있음. v2 HTML·Dataverse 메타데이터에도 2,993 없음.
- **기대:** "v1은 약 3,000명, v2는 3,002명" 또는 2,993의 출처(표·쪽) 명시.
- **실제:** 출처에서 찾을 수 없는 수치를 [DC1]로 인용. 결정에 미치는 영향은 작음.
- **담당:** planning-agent

### 🟡 BUG-PR-06 — PASSION 피부톤 "IV–VI 중심"이 논문 서술과 다름 (예비안 탈락 사유에 쓰임)
- **위치:** 3-1 비교표 4행 ⑤열(66줄), 3-2 (4) 93줄, 5-2 141줄("피부톤이 IV–VI에 집중")
- **확인 방법:** PA2 본문: "with a majority of type VI on the FST scale in Guinea. EAS and Madagascar have a fairly similar distribution from type III to type V or VI".
- **기대:** "기니는 VI가 다수, 마다가스카르·동아프리카는 III~V/VI" 처럼 논문 문장대로 적는다. "결과표에 III도 있음"이 아니라 본문 분포 서술에 III이 포함된다.
- **실제:** "IV–VI 중심"으로 요약해 예비안 탈락 사유로 씀. 방향(어두운 쪽 치우침)은 맞지만 범위가 다르다.
- **담당:** planning-agent

### 🟡 BUG-PR-07 — DermNet 미러의 "Eczema Photos" 클래스 누락
- **위치:** 3-1 비교표 7행 ①열(69줄), 3-2 (7) 104줄
- **확인 방법:** `huggingface.co/api/datasets/WahajRaza/Dermnet` 클래스 목록: 2 "Atopic Dermatitis Photos", **5 "Eczema Photos"**, 13 "Poison Ivy Photos and other Contact Dermatitis". 분할 train 13,223 / test 2,334.
- **기대:** 아토피 관련 클래스로 "Eczema Photos"도 적는다 (사용하지 않는다는 결론은 그대로).
- **실제:** "Atopic Dermatitis Photos"만 적음. 결론에는 영향 없음.
- **담당:** planning-agent

### 💡 BUG-PR-08 — ❓ 일부는 이미 열람한 출처로 해소 가능
- **위치:** 6-1 ❓ 목록
- **내용:**
  - #11 PASSION 414건: 표 1 국가별 합(57+301+56)이 맞고, 국가별 4군 합계(384/985/284)가 논문의 국가별 사례 수와 일치 → "확인됨(사례 수, 장수는 ❓)"로 바꿀 수 있음. 단 같은 표의 부위별 합(351)과는 다르니 "국가 기준 합"이라고 적는다.
  - #2 SWET 부종↔구진: EZ1 스스로 "regional scores for Eczema Area and Severity Index (erythema, excoriation, lichenification, and edema)"로 부종을 EASI의 4번째 징후로 쓴다. 기록 정의가 같은지는 여전히 ❓이지만 근거로 덧붙일 수 있다.
  - #8 SCIN 촬영 기기: SC2 "selectively to adults in the United States (US) searching for these keywords on a mobile device (to facilitate photo taking)" → 모바일 기기로 검색하는 사람을 모집했다는 근거. 실제 촬영 기기는 여전히 ❓.
- **담당:** planning-agent

### 💡 BUG-PR-09 — ISIC 2018 Task 1 검증·테스트 정답 마스크도 현재 공개됨
- **위치:** 3-1 비교표 9행 ②·⑥열(71줄), 3-2 (8)(9) 109줄
- **확인 방법:** IS1 데이터 페이지 2018 표: Validation Ground Truth 742KB, Test Ground Truth 9MB 다운로드 링크 존재(`ISIC2018_Task1_Validation_GroundTruth.zip`, `ISIC2018_Task1_Test_GroundTruth.zip`). IS3 논문 당시에는 "without ground truth masks"였음.
- **제안:** 검증·테스트 정답이 있다는 점을 적어 두면 D-19의 3분할(학습/임계값 조정/홀드아웃)을 공식 분할로 바로 짤 수 있다 — Phase M 계획에 유용.
- **담당:** planning-agent

### 💡 BUG-PR-10 — 출처 추적성: 실제로 열람한 판본과 링크가 다름
- **위치:** 출처 목록 212·214·224줄, 약어표 233·238줄
- **내용:** EZ1은 "본문 PDF 열람 (Imperial 저장소)"이라 했지만 링크는 Elsevier DOI. 열린 전문은 Europe PMC `PMC10504536`에 있다. BG1은 "Europe PMC 전문 열람"이라 했지만 링크는 NCBI PMC. EZ3은 medRxiv 링크가 403이며, 정식 출판본 DOI `10.1016/j.xjidi.2022.100133`이 있다.
- **제안:** 실제 열람한 판본 URL을 함께 적어 재확인하기 쉽게 한다.
- **담당:** planning-agent

### 💡 BUG-PR-11 — Legit.Health-AD 저장소의 `data/` 폴더 내용을 근거로 추가
- **위치:** 3-1 비교표 2행 ④열(64줄), 3-2 (2) 83줄, RQ-04
- **확인 방법:** GitHub 트리 API로 파일 목록만 조회 → `data/annotation/*metrics.csv`(평가자 일치도), `data/distributions/*`(징후별 분포 요약), `data/lesion-segmentation/*metrics.csv`만 있고 마스크·이미지별 징후 라벨은 없음.
- **제안:** "저장소의 data 폴더에는 일치도·분포 요약만 있음"을 적으면 "라벨 비공개" 판단의 근거가 강해지고 RQ-04 질문이 더 구체적이 된다.
- **담당:** planning-agent

### 💡 BUG-PR-12 — 추천 근거와 D-25 목표의 차이를 RQ에서 한 번 더 분명히
- **위치:** 1장 19줄, 5-1, RQ-01·RQ-05
- **내용:** D-25 목표는 "**소아** 아토피 **병변 분할·정량화**"인데, 추천 SCIN은 성인 전용·마스크 없음·중증도 없음이라 목표의 세 요소 중 어느 것도 직접 채우지 못한다(분할은 ISIC 더모스코피로 대체). 문서는 이를 한계·RQ로 올려 두었으므로 결함은 아니다. 다만 RQ-01 선택지에 "추천을 PASSION(소아·휴대폰)으로 바꾸고 SCIN은 보조"안이 RQ-05에만 나뉘어 있어, 팀장이 한 번에 비교하기 어렵다. BUG-PR-01을 고치면 SCIN의 "사례 수 우위"도 약해지므로 추천 순위를 다시 검토하길 제안한다.
- **담당:** planning-agent

### 💡 BUG-PR-13 — 공통 점검: 진단·처방 오인 문구, 위험 신호 상담 안내
- **결과:** 이 문서는 데이터 조사 문서라 사용자 대상 문구가 없고, 진단·처방으로 오인될 문구도 없다. 해당 없음. 이후 Phase M 문서에서 SCIN의 "감별진단 라벨"을 쓸 때 "진단" 표현을 사용자 화면으로 옮기지 않도록 주의.
- **담당:** planning-agent (참고)

---

## 4. 검사 항목별 결과

### 4-1. 출처 링크 (검사 항목 1)
| 범위 | 결과 |
|---|---|
| 비교표 9행 | 모든 칸에 `[약어]` 있음, 약어는 모두 출처 목록에 정의됨 |
| 2-3 제외 후보 4행 | 2행 링크 없음 → BUG-PR-03 |
| 링크 유효성 | 25개 중 24개 정상(200/202), EZ3 403(존재는 API로 확인) |

### 4-2. 라이선스 원문 인용 (검사 항목 2)
| 후보 | 인용 | 대조 결과 |
|---|---|---|
| SCIN (추천) | "You may not make any attempt to re-identify or re-link any of the individual data subjects." | **일치** (LICENSE Section 3 b) |
| ISIC 2018 T1 (추천) | CC-0 | **일치** (데이터 페이지 2018 표 Task 1 행) |
| PASSION (예비) | "You may use the dataset for non-commercial purposes only." / "You must not repost the dataset on any other data hosting platforms or websites." | **일치** (Key Points 절) |
| SWET | "cannot be shared publicly owing to restrictions on the participants' consent" / README 2문장 | **일치** (EZ1 데이터 가용성, EZ2 README 41줄) |
| DermNet (제외 사유) | "...for the purpose of training or testing artificial intelligence models" (25항) | **일치** (25항 "Prohibited Use of Images for AI Training") |
| HAM10000 / DermaCon-IN / Fitzpatrick17k | CC BY-NC 4.0 / CC BY-NC-SA 4.0 / CC BY-NC-SA 3.0 | **일치** |

### 4-3. 비교 기준 6개 (검사 항목 3)
9개 후보 × 6열(① 아토피 장수·소아 비율 ② 마스크 ③ 중증도 라벨 ④ 라이선스 ⑤ 촬영·피부톤 ⑥ Colab 수령·용량) 모두 채워짐. 빈칸 없음. 확인 못 한 값은 ❓ 표시.

### 4-4. 추측·모순 (검사 항목 4)
- ❓ 미표시 추측: 대조한 범위에서 발견 못 함. 다만 출처에서 찾을 수 없는 수치 1건(BUG-PR-05), 해석 오류 1건(BUG-PR-01), 요약이 원문과 다른 것 2건(BUG-PR-04, BUG-PR-06).
- 문서 안 수치 모순: 없음. 9개 후보 수, 추천·예비 조합, RQ 번호와 ❓ 목록 연결, 용량 합(HAM 약 2.8 GB) 모두 일관됨. 단 SCIN과 PASSION 사례 수를 다른 기준으로 비교한 것은 BUG-PR-01.

### 4-5. H0~H4·H-CLS 재현 가능성 (검사 항목 5)
| 과제 | PRD 정의 | 문서 판정 | 검사 결과 |
|---|---|---|---|
| H0 | 안심존 PQ-10 확인 | 재현 가능 | 기준을 바꿨다는 설명 없음 → BUG-PR-02 |
| H1 | 3-클래스(배경·피부·병변), K-01 아토피 병변 DSC | 추천·예비 부분 가능, SWET 가능 | 타당. ISIC는 이진·더모스코피라 K-01 근거가 안 된다는 설명(107~108줄) 있음 |
| H2 | 면적 비율 = 병변 ÷ (피부+병변), 중증도 등급별 단조성 | 추천·예비 불가, SWET 부분 | 타당. SWET는 3-클래스 마스크가 있어 PRD 공식 계산 가능, 전체 등급(IGA) 유무 ❓ 표시 적절 |
| H3 | 4징후 0~3 회귀 | 추천·예비 불가, SWET 부분 | 타당. 부종↔구진 ❓에 EZ1 근거 추가 가능(BUG-PR-08) |
| H-CLS | 4군(염증성·선천성·감염성·기타) | 추천·예비 부분, SWET 불가 | 타당. 4군 체계와 다른 라벨이라 매핑 필요하다는 설명 있음 |
| H4 | 스마트폰 검증셋 DSC 하락폭 | 모두 불가 | 타당. 마스크가 필요하다는 근거 명확, SWET는 디지털 카메라(EZ1 확인) |

### 4-6. 데이터·비밀값·개인정보 (검사 항목 6)
문제 없음 (2-3 참조).

---

## 5. 다음 단계 권고
1. planning-agent가 BUG-PR-01(🟠)을 고친다 — SCIN 수치 의미·단위 수정, 5-1 이유 3과 5-2 비교 문구 수정, ❓ #7 갱신.
2. 🟡 6건은 같은 수정에서 함께 처리 권장(모두 문구·출처 수준).
3. 수정 후 test-agent 재검사는 BUG-PR-01 해당 줄만 확인하면 된다.

---

## 6. 재검사 (v0.2)

| 항목 | 내용 |
|---|---|
| 재검사일 | 2026-11-11 |
| 대상 | `docs/04_analysis/phase-R_dataset_survey.md` v0.2 (작업 트리 수정본, 267줄, `git diff --stat`: +98 / −70) |
| 요청 | ① BUG-PR-01~13 해결 여부 ② 수정 중 새 문제(특히 5-3 추천 변경 근거, PASSION 414건 단위) ③ BUG-PR-04 덧붙임(EASI 0~12)에 대한 planning-agent 반론 확인 |
| 데이터 다운로드 | 하지 않음 (HEAD 요청, 논문 본문, Crossref·Europe PMC 메타데이터만) |

### 6-1. 실행한 명령어와 결과
| 명령 (요약) | 결과 |
|---|---|
| `curl -sIL -o /dev/null -w "%{http_code}"` 새로 추가·바뀐 링크 6개 (EZ3 DOI, RF1, SD1, Imperial PDF, ar5iv FZ2, Europe PMC BG1) | **200 = 5, 403 = 1** (RF1 Roboflow — 문서에 "접속 403"으로 이미 적혀 있음) |
| `curl api.crossref.org/works/10.1016/j.xjidi.2022.100133` | 제목 "Detecting Eczema Areas in Digital Images: An Impossible Task?" — EZ3 서지 일치 |
| `curl doi.org/10.1007/978-3-319-46466-4_13` 의 `<title>` | "A Benchmark for Automatic Visual Classification of Clinical Skin Disease Images" — SD1(SD-198, ECCV 2016) 일치 |
| Europe PMC `PMC10504536/fullTextXML` 그림 설명 추출 (`<fig>` 태그) | 그림 5 설명: "...for EASI (in [0, 12]), TISS (in [0, 9]), and SASSAD (in [0, 18])" → **반론 맞음** |
| PASSION 논문(`arxiv.org/html/2411.04584`) grep: "ICD-10", "44 conditions", "primary" | "44 conditions grouped into 4 categories: Eczema, Fungal, Scabies ... and Others. The grouping is based on the ICD-10 ontology." / "primary diagnosis" 같은 표현 없음 → BUG-PR-14 |
| 비밀값·이메일 grep (이전과 같은 정규식) | 일치 없음 (exit 1) |
| `git diff --stat` | 조사 문서 1개만 변경, 데이터·가중치 파일 없음 |

미실행: Imperial PDF 본문 대조(HEAD 200·PDF 형식만 확인. 같은 전문을 Europe PMC로 이미 대조), ar5iv 본문 재대조(이전에 arXiv PDF로 대조 완료).

### 6-2. 기존 이슈 처리 결과
| ID | 심각도 | 판정 | 확인 위치 (v0.2) | 메모 |
|---|---|---|---|---|
| BUG-PR-01 | 🟠 | **해결** | 73줄(표 3행 ①), 96~99줄, 157줄, 199줄(❓ #7) | 1,211 = 언급 횟수(중복 포함), 1,056 = 포함 사례 수, 단위 = 사례, "Eczema" = 상위 범주로 정확히 고침. 다만 같은 잣대를 PASSION 쪽에는 적용하지 않아 새 이슈가 생김 → BUG-PR-14 |
| BUG-PR-02 | 🟡 | **해결** | 132줄 H0 판정 기준, 136줄 | PQ-10 ①~④를 공개 데이터에 대입했다고 밝히고, "재현 가능 = 확인할 수 있다"는 뜻도 정의함 |
| BUG-PR-03 | 🟡 | **해결** | 58~59줄, 250~251줄, 266~267줄 | RF1·SD1 링크 추가. RF1은 403이라는 점과 "검색 결과 요약만 확인"이라는 점을 표기 |
| BUG-PR-04 | 🟡 | **해결** | 71줄, 85~87줄 | 이미지당 1명(307·308·308·422) 반영. **덧붙임(EASI 0~12)은 test-agent 오류로 철회** — 6-3 참조 |
| BUG-PR-05 | 🟡 | **해결** | 112줄 | "v1 약 3,000명, v2 3,002명"으로 고치고, 2,993을 지운 이유도 적음 |
| BUG-PR-06 | 🟡 | **해결** | 74줄 ⑤, 107줄, 153줄 | 논문 문장 그대로 인용 |
| BUG-PR-07 | 🟡 | **해결** | 77줄, 118줄 | "Eczema Photos"와 분할 수(train 13,223 / test 2,334) 추가 |
| BUG-PR-08 | 💡 | **해결** | 194줄 #2, 200줄 #8, 203줄 #11 | |
| BUG-PR-09 | 💡 | **해결** | 79줄, 123줄 | D-19 3분할에 대응시킬 수 있다는 점 추가, 확정은 다음 Phase로 미룸 |
| BUG-PR-10 | 💡 | **해결** | 238줄(EZ3 DOI 교체), 253~267줄 "실제 열람 판본" 열 | |
| BUG-PR-11 | 💡 | **해결** | 72줄, 92줄, 182줄 | 근거 출처 표기 방식은 BUG-PR-16 참조 |
| BUG-PR-12 | 💡 | **해결** | 179줄 RQ-01 선택지 A~D 통합, 5-3 | 추천 순위를 다시 검토했고, 결과로 추천 PASSION · 예비 SCIN으로 바뀜 |
| BUG-PR-13 | 💡 | **해결(해당 없음 확인)** | 219줄 | CLAUDE.md 64~66줄(3-1)을 가리키는 참조가 맞음 |

### 6-3. BUG-PR-04 덧붙임에 대한 반론 — 받아들임
- planning-agent 주장: EZ1 그림 5 설명에 "[0, 12]"가 직접 나온다.
- 확인 결과: **맞다.** Figure 5 설명 원문 — "Accuracy of severity prediction evaluated with RMSE. Points show the mean ± SE over 10-fold cross-validation with the test set of 136 photographs for EASI (in [0, 12]), TISS (in [0, 9]), and SASSAD (in [0, 18])."
- 원인: 1차 검사 때 본문 텍스트만 정규식(`0.12`, `0–12`, `range`)으로 찾고 그림 설명의 대괄호 표기 `[0, 12]`를 검색하지 않았다. **test-agent의 검사 누락**이며, BUG-PR-04 덧붙임("범위 0~12는 논문에 적혀 있지 않다")은 **철회**한다. v0.2 85줄의 "논문 그림 5 설명에 표기"는 출처와 일치한다.

### 6-4. 새 이슈

#### 🟠 BUG-PR-14 — PASSION 414건을 "주진단 기준·깨끗한 습진 라벨"로 설명함. SCIN에 적용한 잣대(상위 범주, 아토피만이 아님)를 PASSION에는 적용하지 않음
- **위치:** 27줄(요약 v0.2 변경), 74줄(표 4행 ① "주진단 기준 사례 수"), 99줄, 106줄, 150줄(5-1 이유 2), 157줄, 162줄(5-3)
- **확인 방법:** PA2 본문 — "The cases collected represent **44 conditions grouped into 4 categories**: Eczema, Fungal, Scabies ... and Others. The grouping is based on the **ICD-10 ontology**." 표 1 제목은 "Number of cases per country and body localization". 논문 전체에서 "primary diagnosis"·"main diagnosis" 같은 표현을 찾지 못함. 농가진은 "may occur alone or in combination with other conditions"라 별도 이진 열.
- **단위 판정 (요청 ②):** 414는 **사례(건) 단위가 맞다**(표 1 = cases, 국가별 합 384/985/284 = 1,653과 일치). 이미지 장수는 ❓로 둔 것도 맞다.
- **기대:**
  1. PASSION의 "Eczema"도 **ICD-10 기준으로 여러 진단을 묶은 군**이므로, SCIN과 마찬가지로 **아토피피부염만 센 수가 아니다**. 군 안의 아토피 비율은 ❓로 표시해야 한다.
  2. "주진단 기준"은 논문에 없는 표현이다. 논문대로 "사례별 진단을 ICD-10으로 4군에 묶은 값(사례 수)"이라고 적는다.
  3. 5-1 이유 2("습진 라벨이 … 깨끗하다", "학습 라벨로 바로 쓸 수 있다")와 5-3의 대비는 "사례당 군 라벨이 하나(감별진단 혼합이 아님)"라는 점에서만 성립한다. 두 데이터 모두 아토피 수는 ❓라는 점을 함께 적는다.
- **실제:** SCIN 쪽에는 "상위 범주·아토피만 센 값 아님"을 명시(98줄)했지만, PASSION 414건은 "주진단 기준"으로 "깨끗하다"고 적어 아토피 라벨처럼 읽힌다. 추천 근거 4개 중 하나(이유 2)와 순서 변경 사유(5-3)에 쓰였다.
- **영향:** 추천 변경의 핵심 근거인 "소아"(이유 1)는 출처와 맞으므로 **추천 순서 자체를 뒤집을 사유는 아니다.** 다만 팀장이 RQ-01을 결정할 때 "PASSION = 아토피 414건"으로 오해할 수 있다. BUG-PR-01과 같은 종류의 오류라 같은 심각도로 둔다. ❓ 목록에 "PASSION 습진 군 안의 아토피 비율"(RQ-03) 항목 추가가 필요하다.
- **담당:** planning-agent

#### 🟡 BUG-PR-15 — "D-25 '소아'를 채우는 유일한 접근 가능 후보" 단정
- **위치:** 149줄(5-1 이유 1)
- **확인 방법:** 같은 문서 75줄 DermaCon-IN = "전 연령, 소아 비율 ❓", 공개(CC BY-NC-SA 4.0). 183줄 RQ-05 선택지 B는 DermaCon-IN을 소아 대안 예비로 제시. DC1 본문에 "common in Indian children"이 나옴(소아 포함 가능성). PASSION도 기니·마다가스카르 세트에는 성인이 섞여 있고(PA2 "EAS images including only paediatric cases") 소아 비율은 ❓(204줄 #12).
- **기대:** "확인된 범위에서 소아 비중이 가장 높은 접근 가능 후보(동아프리카 세트는 소아만), DermaCon-IN 소아 비율은 ❓"처럼 ❓와 맞게 적는다.
- **실제:** 소아 비율이 ❓인 후보가 있는데 "유일"이라고 단정함. 문서 안에서 ❓ 표시(75줄, 204줄)와 맞지 않는다.
- **담당:** planning-agent

#### 💡 BUG-PR-16 — 출처 표기에 테스트 리포트를 근거로 인용
- **위치:** 260줄(LH2 "테스트 리포트의 GitHub 트리 API 조회 결과 인용"), 261줄(DN3 "테스트 리포트 조회")
- **내용:** 조사 문서의 출처는 1차 출처여야 추적하기 쉽다. test-agent 조회 결과를 근거로 쓰기보다, 조회한 API URL(`api.github.com/repos/Legit-Health/ASCORAD/git/trees/HEAD?recursive=1`, `huggingface.co/api/datasets/WahajRaza/Dermnet`)을 출처로 적고 planning-agent가 직접 확인했다고 표시하길 제안한다. DN3는 API URL을 이미 적었으므로 "(테스트 리포트 조회)" 표현만 정리하면 된다.
- **담당:** planning-agent

### 6-5. 5-3 추천 변경 근거 대조 (요청 ②)
| 5-3·5-1 주장 | 출처 대조 | 판정 |
|---|---|---|
| SCIN 수치는 언급 횟수·포함 사례 수, 상위 범주 | 노트북 20번 셀, 스키마 문서 | 맞음 |
| PASSION 414 = 주진단 기준 | 논문은 ICD-10 4군 묶음, "주진단" 표현 없음 | **표현 부정확 → BUG-PR-14** |
| PASSION = 소아 중심, 동아프리카 세트는 소아만 | PA2 "EAS images including only paediatric cases", "substantial effort was devoted to including the paediatric population" | 맞음 ("유일" 단정은 BUG-PR-15) |
| 휴대폰 촬영, 농가진 라벨 | PA2 "mobile phones", "binary column 'Impetiginized'" | 맞음 |
| ISIC 분할은 PASSION 승인 없이 바로 시작 가능, 검증/테스트 정답 공개 | IS1 데이터 페이지(CC-0, 정답 zip 링크), HEAD 200 | 맞음 |
| 피부톤 III~VI(기니 VI 다수) | PA2 원문 | 맞음 |
| 즉시성을 중시하면 v0.1 순서로 되돌릴 수 있음(RQ-01 B) | 179줄 RQ-01 선택지 B | 문서 안에서 일관됨 |

결론: 추천을 PASSION으로 바꾼 주된 근거(소아, D-25)는 출처와 맞는다. 보조 근거(이유 2)의 표현만 고치면 된다.

### 6-6. 재검사 집계
| 구분 | 🔴 | 🟠 | 🟡 | 💡 |
|---|---|---|---|---|
| 기존 이슈 미해결·부분 해결 | 0 | 0 | 0 | 0 |
| 새 이슈 (BUG-PR-14~16) | 0 | 1 | 1 | 1 |
| 철회 | — | — | BUG-PR-04 덧붙임(EASI 0~12) | — |

- 기존 13건은 모두 해결됨.
- 남은 열린 이슈는 3건: 🟠 BUG-PR-14, 🟡 BUG-PR-15, 💡 BUG-PR-16. 차단 이슈는 없다.
- BUG-PR-14를 고친 뒤 회의(Phase R)로 넘어가는 것을 권고한다. 다시 검사할 범위는 27·74·106·150·157·162줄과 ❓ 목록 추가 항목뿐이다.

---

## 7. 재검사 2 (v0.3)

| 항목 | 내용 |
|---|---|
| 재검사일 | 2026-11-11 |
| 대상 | `docs/04_analysis/phase-R_dataset_survey.md` v0.3 (작업 트리 수정본, 271줄, `git diff --stat`: +102 / −70) |
| 요청 | ① BUG-PR-14·15·16 해결 여부 ② 수정하면서 새로 생긴 문제 |
| 데이터 다운로드 | 하지 않음 (HEAD 요청, 메타데이터 API 응답만) |

### 7-1. 실행한 명령어와 결과
| 명령 (요약) | 결과 |
|---|---|
| `grep -n "v0.3\|414\|주진단\|ICD-10\|유일\|LH3\|DN4\|테스트 리포트\|아토피 비율"` (조사 문서) | "주진단" **0건**. 414 관련 줄(28·75·100·107·151·163·204·217)은 모두 "ICD-10 기반 Eczema 군 사례 수"로 표기됨 |
| `curl -sIL -o /dev/null -w "%{http_code}"` LH3·DN4 | **200 ×2** (둘 다 `application/json`) |
| GitHub 트리 API에서 `data/` 아래 이미지 파일 중 `distributions/plots` 밖의 것 개수 | **0** → LH3 설명("사진·마스크 PNG 없음, 그래프 PNG만 있음")과 일치 |
| 비밀값·이메일 grep (이전과 같은 정규식) | 일치 없음 (exit 1) |
| `git status --short` | 조사 문서 수정(M), 테스트 리포트(??) 두 개뿐. 데이터·가중치 파일 없음 |

미실행: PASSION 메타데이터의 세부 진단 확인(승인 전이라 불가, 문서도 ❓ #24로 둠).

### 7-2. BUG-PR-14·15·16 처리 결과
| ID | 심각도 | 판정 | 확인 위치 (v0.3) | 메모 |
|---|---|---|---|---|
| BUG-PR-14 | 🟠 | **해결** | 28, 75, 100, 107, 151, 163, 204, 217(❓ #24)줄, 185줄 RQ-06 | "주진단"을 지우고 논문 문장("44 conditions grouped into 4 categories ... ICD-10 ontology")을 인용함. "아토피만 센 수가 아님"을 양쪽 데이터에 똑같이 적용했다. 5-1 이유 2는 "사례당 군 라벨 1개"라는 **라벨 형식** 장점으로 좁혔다. 이 근거는 표 1 국가별 합(1,653)이 전체 사례 수와 같아 사례마다 한 군에만 속한다는 점과 맞는다. 5-3도 "습진 수는 어느 쪽 근거도 되지 못한다"로 고쳐 비교가 대칭이 됐다 |
| BUG-PR-15 | 🟡 | **해결** | 150줄(5-1 이유 1), 163줄 | "확인된 범위에서 소아 비중이 가장 높은 접근 가능 후보"로 고쳤다. 기니·마다가스카르 세트에 성인이 있다는 점, DermaCon-IN 소아 비율 ❓, 비교는 집계 후 확정(RQ-05·RQ-06)도 함께 적어 ❓ 표시와 맞는다 |
| BUG-PR-16 | 💡 | **해결** | 243·247줄(LH3·DN4 추가), 264·265줄 | 1차 출처 API URL을 달고 "planning-agent가 직접 조회"라고 적었다. 수치(클래스 23개, 분할 수, 1,473,388,407 B, 라이선스 태그 없음)는 1차 검사 때 test-agent 조회 결과와 같다. "CSV 내용은 열어 보지 않음(파일 목록 기준 판단)"이라고 한계도 적어 두었다 |

### 7-3. 새 이슈

#### 💡 BUG-PR-17 — 버전 표기와 RQ 연결이 덜 정리됨
- **위치:**
  - 179줄 RQ-01 선택지 A "(v0.2 제안)"
  - 161줄 5-3 제목 "(v0.1 → v0.2)"
  - 185줄 RQ-06 ↔ 217줄 ❓ #24
- **내용:**
  1. 문서 버전은 v0.3인데 RQ-01과 5-3에 v0.2가 남아 있다. 추천 순서는 v0.2에서 바뀌었으므로 틀린 표기는 아니다. 다만 RQ-01 A를 "v0.2~v0.3 제안"처럼 적으면 덜 헷갈린다.
  2. ❓ #24(PASSION 군 안의 아토피 비율)는 관련 RQ를 RQ-03(PASSION 신청)으로 두었다. 그런데 RQ-06 "왜 필요한가"에도 같은 항목이 "(승인 후)"로 들어가 있다. RQ-06은 공개 메타데이터 파일 집계를 허락받는 질문이고, 대상 파일 목록에 PASSION이 없다. 한쪽(RQ-03)으로 모으면 팀장이 결정할 범위가 더 분명해진다.
- **영향:** 결론·근거에는 영향 없음.
- **담당:** planning-agent

### 7-4. 재검사 2 집계
| 구분 | 🔴 | 🟠 | 🟡 | 💡 |
|---|---|---|---|---|
| 기존 이슈 미해결·부분 해결 (BUG-PR-01~16) | 0 | 0 | 0 | 0 |
| 새 이슈 (BUG-PR-17) | 0 | 0 | 0 | 1 |

- **BUG-PR-01~16 모두 해결.** 남은 이슈는 💡 BUG-PR-17 하나이며, 고치지 않아도 다음 단계로 넘어가는 데 지장이 없다.
- 차단·주요 이슈가 없으므로 **Phase R 회의로 넘어가도 된다.** 추천 순서(PASSION + ISIC 2018 T1, 예비 SCIN)의 근거는 출처와 맞고, 남은 판단(H2·H3 범위 축소, SWET 문의, PASSION 신청 시점)은 RQ-01~RQ-06으로 팀장에게 올라가 있다.
