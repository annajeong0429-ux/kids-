# 진행 상태 (STATUS.md)

최종 갱신: 2026-10-01

| 항목 | 내용 |
|---|---|
| 현재 Phase | Phase 0 — 저장소·에이전트 팀 세팅 (마무리 단계) |
| 상태 | 진행 중 |
| 다음 Phase | Phase 1 PRD 작성 (`prd-agent`) — 입력: `docs/00_source/` 2종 + `docs/decisions.md` |

## 본선 주요 일정
| 일정 | 내용 |
|---|---|
| ~10/23 전후 | 안심존 분석 결과물 **반출 신청** (심의 1~2주) |
| 11/6 16:00 | 데이터 시각화 결과보고서 · 발표자료 제출 |
| 상시 | 안심존 방문 + 증빙자료 확보 (7회 이상 = 방문 점수 만점) |

## 완료
- git 저장소 초기화, 원격 `annajeong0429-ux/kids-` 연결, `main`·`dev`·`develop` 생성 (D-09)
- 예선 PDF 2종 `docs/00_source/` 이동
- CLAUDE.md 2장(프로젝트 개요·본선 제출물)·9-1(브랜치 역할) 기입
- `.claude/agents/` 에이전트 5종 정의
- `docs/decisions.md` D-01~D-13, F-INFO-01~03 기록
- Codex CLI 0.159.3 설치·ChatGPT 로그인, Codex 플러그인 `/codex:setup` ready (리뷰 게이트 꺼짐)
- GitHub ruleset `protect-main-dev-develop` 적용 (main·dev·develop: 삭제 금지, force push 금지, PR 필수)
- PR #1 Merge (팀장)
- Claude Code deny 규칙 추가 (D-13)

## 대기 중
- 팀장: Q-01 남은 확인 (가중치 파일 반출 심의 대상 여부, 공개 웹앱 배포 허용 여부)
- 팀장: Q-04 남은 확인 (11/6 전 웹앱 개발 병행 여부)
- CI 워크플로: 기술 스택 확정(Phase 2) 후 제안 → 승인 후 ruleset에 status check 추가

## 열려 있는 PR
- `chore/phase0-deny-rules` → `develop` (deny 규칙, 결정·STATUS 갱신)
