---
id: endpoint.cli-sdi-status
kind: Endpoint
title: "CLI `sdi status` (sdi-status)"
definition: "읽기 전용 대시보드 — 데몬 상태·프로젝트·활성 플랜·라운드·진행 중 태스크 요약"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/aggregate.rs"
  - "sdi-plugin/plugin/commands/sdi-status.md"
relatesTo:
  - to: "concept.project"
    type: "reads"
  - to: "concept.plan"
    type: "reads"
  - to: "concept.task"
    type: "reads"
  - to: "domain.project-management"
    type: "belongs-to"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`sdi status`(또는 `sdi-status` 슬래시 커맨드)는 SDI 데몬과 현재 프로젝트의 전반적인 상태를 한 화면에 요약하여 보여주는 읽기 전용 대시보드 CLI 명령이다. 에이전트와 사람 모두 작업 시작 전 현재 컨텍스트를 빠르게 파악하는 용도로 사용한다.

MCP 레이어를 거치지 않고 HTTP로 데몬(`sdid`)에 직접 접근한다. 읽기 전용 조회이므로 별도 인증 없이 로컬 데몬 포트로 직접 요청을 보낸다. `resolve_project_id()` 헬퍼를 사용하여 현재 작업 디렉토리(cwd)를 기준으로 귀속된 프로젝트를 자동 탐지한다.

## 요청 / 응답

단일 대시보드 뷰를 출력한다. 출력 내용은 다음 섹션으로 구성된다.

**데몬 헬스**: `sdid` 프로세스의 실행 상태, 포트 번호, 응답 지연 시간을 표시한다. 데몬이 응답하지 않으면 연결 오류와 재시작 안내 명령을 함께 표시한다.

**프로젝트 정보**: cwd에서 자동 탐지된 현재 프로젝트 이름, ID, 등록 경로를 표시한다. cwd가 등록된 프로젝트에 속하지 않으면 미등록 경고와 등록 방법을 안내한다.

**활성 플랜 요약**: 현재 active 상태인 플랜의 제목, 승인 일시, 귀속된 시나리오 총 수와 확정/미확정 분류를 표시한다. 활성 플랜이 없으면 플랜 생성 또는 승인을 안내한다.

**활성 라운드 요약**: 현재 active 라운드의 번호, 시나리오 검증 현황(통과/실패/미검증 수)을 표시한다.

**진행 중 태스크**: in_progress 상태인 태스크의 목록과 각 태스크에 연결된 시나리오 수, 담당 에이전트를 요약하여 표시한다.

**최근 활동**: 최근 결정, 노트 추가, 라운드 결과 기록 등 주요 이벤트를 타임라인 형태로 표시한다.

## 권한 / 제약

- 읽기 전용이다. 어떤 데이터도 변경하지 않는다.
- MCP를 거치지 않고 HTTP로 데몬에 직접 접근하므로, 데몬이 실행 중이어야 동작한다. 데몬이 꺼져 있으면 데몬 헬스 섹션만 오류 상태로 표시하고 나머지는 빈 상태로 출력한다.
- `resolve_project_id()`는 cwd를 기준으로 부모 디렉토리를 탐색하여 등록된 프로젝트를 찾는다. 탐색 깊이에 제한이 있을 수 있다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/aggregate.rs` 및 `sdi-plugin/plugin/commands/sdi-status.md` 파일 구조를 근거로 추론되었다. `resolve_project_id()` 헬퍼 함수명과 MCP 우회 방식은 SDI 플러그인 설계 문서 기반 추론이다.

## 미확정 (OPEN)

- `resolve_project_id()`의 cwd 기반 프로젝트 탐색 알고리즘의 구체적 탐색 깊이 한계와 ambiguous 경우 처리 방식이 확인되지 않았다.
- 출력 형식이 사람 가독 텍스트만인지 `--json` 플래그로 기계 가독 JSON도 지원하는지 불확실하다.
- 복수의 활성 플랜이 동일 프로젝트에 존재하는 경우(만약 허용된다면) 표시 방식이 명확하지 않다.
- Claude Code 슬래시 커맨드 `/sdi-status`와 CLI `sdi status` 명령이 정확히 동일한 출력을 생성하는지, 아니면 슬래시 커맨드가 추가적인 Claude 컨텍스트를 포함하는지 확인이 필요하다.
