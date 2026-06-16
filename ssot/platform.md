---
id: platform.sdi
kind: Platform
title: Scenario-Driven Implementation (SDI)
purpose: "자연어 GWT 시나리오를 1등 시민으로 두고, 다중 LLM 에이전트가 구현·검증·회귀 점검을 자율 수행하게 해 빌더가 PC 앞에 묶이지 않고도 개발이 진행되는 LLM 시대 BDD 후속 도구."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
  - persona.subagent
value: "빌더가 자연어로 시나리오를 작성하고 플랜을 승인하면, 에이전트들이 자율적으로 구현·검증·회귀를 수행한다. 매 라운드마다 이전 시나리오 전체를 자동 회귀 재실행하고, 사람 확인이 필요한 지점에서만 게이트를 열어 통보하며, 빌더는 언제든 circuit breaker로 즉시 통제권을 되찾을 수 있다."
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
implementedIn:
  - "sdi-plugin/plugin/"
  - "sdi-plugin/crates/"
  - "sdi-plugin/docs/PRD.md"
  - "sdi-plugin/CLAUDE.md"
---

## 무엇인가

SDI(Scenario-Driven Implementation)는 TDD(1990년대) → BDD(2000년대)의 LLM 시대 후속 도구다. 차이는 검증 단위에 있다: TDD는 테스트 코드, BDD는 Gherkin step definition, SDI는 자연어 Given/When/Then 시나리오를 직접 LLM이 읽고 실행 가이드로 사용한다. 컴파일 단계와 step definition 유지보수 비용이 사라진다.

본체는 Claude Code 플러그인이다. 플러그인 셸(`plugin/`)과 Rust 워크스페이스(`crates/`)가 같은 저장소의 두 시각이다 — `sdi` CLI와 `sdid` 데몬이 SDI의 실질적인 런타임이고, 플러그인 셸은 훅·MCP·슬래시 명령·에이전트 정의를 담은 얇은 연결 계층이다.

SDI는 선행 도구 Clawket v3.0(약 1개월 운영)의 직계 후속이다. Clawket이 태스크 중심·자유 evidence·수동 회귀였다면, SDI는 시나리오 중심·GWT 강제·자동 회귀다. Clawket에서 검증된 "로컬 SQLite + 데몬 + MCP" 아키텍처를 계승하되, 1등 시민을 Task에서 Scenario로 바꾸고 다중 에이전트 협업을 본체로 세웠다.

## 누구를 위한 것인가

SDI는 세 가지 역할을 위해 설계되었다.

**1인 빌더**(`persona.solo-builder`)는 자연어로 시나리오를 제시하고, 플랜을 승인하고, 중요한 결정에만 개입한다. PC 앞을 벗어나도 개발이 진행된다. 이상함을 느끼면 circuit breaker 단 한 번의 액션으로 즉시 통제권을 되찾는다.

**코딩 에이전트**(`persona.coding-agent`, Claude Code 메인 세션)는 오케스트레이터로서 플랜을 분해하고, 적합한 협업 패턴을 선택하고, 전문 서브에이전트들에게 위임한다. D21 위임 게이트가 코드 직접 수정을 구조적으로 불가능하게 만든다.

**전문 서브에이전트들**(`persona.subagent`)은 각자의 전문 영역(코드 구현·테스트·결정 협상·패턴 선택·회귀 재실행·파괴 분석·롤백 실행 등)에서 단일 작업을 완수한다. AgentNote 블랙보드와 AgentSpec 등록을 통해 서로가 누구인지 알고 협업한다.

## 서비스 영역

SDI가 다루는 영역은 다음과 같다:

- **시나리오 관리** (`domain.scenario-management`): GWT 형식 강제, 상태 전이, 라운드별 판정 누적, 의존관계 DAG, 자원 클레임.
- **요구사항** (`domain.requirements`): 플랜에 귀속된 자연어 요구사항의 SNAPSHOT-ONLY 관리.
- **태스크 분해** (`domain.task-decomposition`): LLM이 시나리오에서 런타임 태스크를 자율 생성. 빌더가 직접 만들지 않음.
- **플랜 관리** (`domain.planning`): 플랜 승인 게이트(시나리오 ≥1, GWT 유효), active 전환, 자율성 정책 스코프.
- **라운드 실행** (`domain.round-execution`): R1 신규 개발, R2+ strict-regression 자동 회귀. 같은 엔진.
- **결정 협상** (`domain.decision-negotiation`): M3 4단계 협상(제안→비평→합의/이견), append-only ADR, 되돌림 계획.
- **협업 패턴** (`domain.collaboration-patterns`): Workflow·Graph·Swarm·Agents-as-Tools 4패턴 + direct 안티패턴 마커. 7번째 1등 시민.
- **자율성 정책** (`domain.autonomy`): 스코프별 L3/L4/L5 설정, circuit breaker, L5 unlock 추가 조건.
- **파괴 검토** (`domain.disruption-review`): needs-review 기본 정책, impacted vs failing 구분.
- **지식 RAG** (`domain.knowledge-rag`): 온디바이스 하이브리드 검색(FTS5 + sqlite-vec), 세션 간 컨텍스트 복원.
- **에이전트 조율** (`domain.agent-coordination`): AgentNote 블랙보드, 핸드오프, AgentSpec 자기조직화.
- **프로젝트 관리** (`domain.project-management`): 최상위 컨테이너, cwd 매핑, enabled 토글.
- **플러그인 런타임** (`domain.plugin-runtime`): Claude Code 훅·MCP·슬래시 명령·대시보드 SPA. XDG 경로 불변.
- **거버넌스 감사** (`domain.governance-audit`): 모든 게이트 이벤트·bypass·자율 적용의 감사 로그.

## 접근 매트릭스 (역할 → 서비스)

| 역할 | 시나리오·플랜 | 라운드·태스크 | 결정·패턴 | 자율성 | 감사·bypass |
|---|---|---|---|---|---|
| 1인 빌더 | 시나리오 작성·확인, 플랜 승인 | 결과 검토, 라운드 확인 | dissensus 판단, L4 강제 결정 확인 | 정책 설정, circuit breaker | 감사 이력 조회, bypass arm |
| 코딩 에이전트(메인) | 시나리오·플랜 조회(read-only) | 라운드 시작, 태스크 분해·위임 | 패턴 선택 위임, 협상 오케스트레이션 | 정책 조회 | — |
| 서브에이전트 | 시나리오 판정 기록 | 태스크 구현·완료·증거 제출 | 결정 제안·비평·합의 작성 | — | bypass 소비(1회) |

메인 세션은 변경 도구가 훅에서 차단된다(D21). 서브에이전트는 `agent_id`가 있어야 게이트를 통과한다.

## 기술 스택 (제품 맥락)

SDI는 사용자가 Rust 툴체인 없이 사용할 수 있도록 사전 빌드 바이너리로 배포된다.

- **플러그인 배포**: Claude Code 플러그인 마켓플레이스. 사전 빌드 `sdi` + `sdid` 바이너리(macOS + Linux × x86_64 + aarch64).
- **CLI (`sdi`)**: Rust 단일 바이너리. 모든 작업 관리 명령 + MCP stdio 서버(`sdi mcp`). 워크스페이스 `crates/cli/`.
- **데몬 (`sdid`)**: Rust(axum + rusqlite). SQLite 단일 writer, HTTP API + SSE + 유닉스 소켓. tower-http ServeDir로 대시보드 SPA 직접 서빙. `crates/daemon/`.
- **도메인 모델**: `crates/core/` — 7개 1등 시민 엔티티 + AgentNote·AgentSpec.
- **저장소**: `crates/db/` — SQLite + FTS5 키워드 검색. sqlite-vec 벡터 검색은 구현 진행 중. XDG 경로에 저장, 외부 전송 없음.
- **대시보드 SPA**: Vite/React 19/Tailwind 4. `plugin/web/`. 자율성 패널·결정 타임라인·AgentNote 블랙보드·패턴 뷰.
- **플러그인 셸**: `plugin/` — 훅·MCP·슬래시 명령·에이전트 정의만 담는 얇은 셸.
- **Rust 워크스페이스**: `crates/` — cli, daemon, mcp, core, db 5개 크레이트. workspace version 0.6.1.

모든 데이터가 로컬에 머문다. 임베딩 모델도 온디바이스로 실행된다.

## 미확정 (OPEN)
- [ ] OPEN: 배포 전략(dist 브랜치 vs GitHub Releases 바이너리 직접 소비)이 아직 결정되지 않음.
- [ ] OPEN: sdi-desktop(Tauri 2 add-on)과 sdi-plugin의 버전 동기화 정책 미확정.
