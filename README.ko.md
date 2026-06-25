# Agent Model Optimizer

AI 코딩 에이전트를 위한 **작업별 모델 라우팅** 설정 스킬입니다.

**Cursor Agent Skill**로 동작하며, 사용 중인 도구(Cursor, Claude Code, Codex, Windsurf)에 맞는 설정 파일을 생성합니다. 탐색·셸은 빠른 모델, 리뷰·디버깅은 균형 모델, 보안·아키텍처는 고성능 모델로 위임하도록 구성합니다.

**지원 도구:** Cursor · Claude Code · Codex · Windsurf

## 하는 일

- **작업 유형**을 **능력 tier**(`fast` ~ `high-thinking`)로 매핑 (`routing-matrix.md`가 기준)
- tier를 도구별 모델 ID로 변환 (Claude `haiku`/`sonnet`/`opus`, Cursor `composer-2.5-fast` 등)
- **사용 중인 도구의 파일만** 생성
- 기본 에이전트: `explorer`, `reviewer`, `tester` (+ 선택: `security-auditor`, `architect`, `docs-writer`)
- 기존 설정 충돌 시 **병합 / 덮어쓰기 / 건너뛰기 / model 필드만 갱신** 중 선택

## 하지 않는 말

절약 효과와 품질은 워크플로·서브에이전트 호출 빈도·작업 난이도에 따라 달라집니다. **작업에 맞는 모델을 쓰도록 돕는 것**이지, 특정 비용 절감이나 품질을 보장하지 않습니다.

## 도구별 생성 파일

| 도구 | 생성 경로 |
|------|-----------|
| **Cursor** | `.cursor/rules/model-routing.mdc`, `.cursor/agents/*.md` |
| **Claude Code** | `.claude/agents/*.md`, `CLAUDE.md` (라우팅 섹션) |
| **Codex** | `AGENTS.md` (에이전트 섹션) |
| **Windsurf** | `.windsurfrules` (라우팅 가이드) |

## 시작하기

1. 이 폴더를 `~/.cursor/skills/agent-model-optimizer/`에 복사합니다.
2. Cursor에서 요청합니다:
   > *"agent-model-optimizer 스킬로 이 프로젝트 모델 라우팅 설정해줘"*
3. 도구(가능하면 자동 감지), 스코프(프로젝트/사용자), 프로파일을 선택합니다:
   - `budget` — 한 단계 낮은 tier 선호
   - `balanced` — 기본값 (권장)
   - `quality` — 한 단계 높은 tier 선호
4. 기존 설정이 있으면 충돌 정책을 선택합니다.
5. 생성된 라우팅 요약 표를 확인합니다.

## 기본 라우팅

| 에이전트 | 작업 | tier |
|----------|------|------|
| `explorer` | 코드베이스 탐색 | fast |
| `reviewer` | 코드 리뷰 | balanced-thinking |
| `tester` | 테스트 작성·수정 | balanced |
| `security-auditor` | 보안 감사 | high-thinking |
| `architect` | 아키텍처·마이그레이션 | high-thinking |
| `docs-writer` | 문서 작성 | fast |

Cursor 내장 서브에이전트(`explore`, `shell`, `bugbot`)는 생성된 rule에 사용 시점이 정리됩니다. 모델은 제품 기본값을 따릅니다.

## 커스터마이즈

| 목적 | 수정 파일 |
|------|-----------|
| 작업별 tier 변경 | `references/routing-matrix.md` |
| 모델 ID 업데이트 | `references/model-catalog.md` |
| 에이전트 프롬프트·구조 변경 | `templates/<tool>/` |

스킬 저장소를 수정한 뒤 스킬을 다시 실행하면 프로젝트 설정을 재생성할 수 있습니다.

## 문서

- [DESIGN.md](./DESIGN.md) — 전체 설계 (영문)
- [README.md](./README.md) — 영문 개요

## License

TBD
