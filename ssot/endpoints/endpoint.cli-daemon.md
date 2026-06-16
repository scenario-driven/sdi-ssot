---
id: endpoint.cli-daemon
kind: Endpoint
title: "CLI `sdi daemon`"
definition: "SDI 데몬(sdid) 프로세스 생애주기 관리 — 시작·중지·상태·재시작"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/ops.rs"
relatesTo:
  - to: "domain.plugin-runtime"
    type: "belongs-to"
  - to: "concept.project"
    type: "reads"
governedBy: []
impacts: []
consumedBy: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

`sdi daemon` 은 SDI 시스템의 핵심 백그라운드 프로세스인 `sdid`(sdi daemon)의 생애주기를 관리하는 CLI 명령 그룹이다. `sdid`는 HTTP API 서버, SQLite 데이터베이스 관리, MCP 서버, 대시보드 SPA 서빙을 모두 담당하는 단일 바이너리 프로세스다. SDI 시스템의 모든 기능은 `sdid`가 실행 중이어야 정상 동작한다.

LM-8 불변식에 따라 `sdid`가 사용하는 런타임 파일(포트 번호, PID, 소켓 경로)은 반드시 `~/.cache/sdi/` 경로에 기록된다. 이 경로는 Claude Code 플러그인 디렉토리와 완전히 분리되어야 하며, 사용자 데이터 경로(`~/.local/share/sdi/`)와도 구분된다.

## 요청 / 응답

주요 서브커맨드 동작은 다음과 같다.

**시작(start)**: `sdid` 바이너리를 백그라운드 프로세스로 시작한다. 이미 실행 중인 프로세스가 있으면 중복 시작을 방지하고 기존 프로세스 정보를 반환한다. 시작 성공 시 데몬이 수신 중인 포트 번호와 PID를 `~/.cache/sdi/sdid.port` 및 `~/.cache/sdi/sdid.pid`에 기록한다.

**중지(stop)**: 실행 중인 `sdid` 프로세스에 종료 신호를 보내고, 프로세스가 정상 종료되면 런타임 파일을 정리한다.

**상태(status)**: 현재 `sdid` 프로세스의 실행 상태(실행 중/중지됨), 포트 번호, PID, 가동 시간을 조회한다. `/health` 엔드포인트에 요청을 보내 데몬 응답 여부도 함께 확인한다.

**재시작(restart)**: 실행 중인 데몬을 중지하고 즉시 다시 시작한다. 설정 변경이나 비정상 동작이 감지되었을 때 사용한다.

**이벤트 스트림 구독(watch)**: 데몬이 발행하는 SSE(Server-Sent Events) 이벤트 스트림을 구독한다. 라운드 결과 기록, 태스크 상태 전이, 결정 추가 등 실시간 이벤트를 스트리밍 방식으로 수신한다.

## 권한 / 제약

- LM-8 불변식: `sdid` 런타임 파일은 반드시 `~/.cache/sdi/` 에 위치해야 한다. 이 경로를 다른 위치로 변경하는 것은 SDI의 사용자 데이터 격리 불변식 위반이다.
- `start` 시 동일 포트를 사용하는 다른 프로세스가 있으면 시작이 실패한다.
- `sdid` 바이너리는 GitHub Releases 빌드 산출물로 배포된다. 사용자 머신에 Rust 툴체인이 없어도 사용 가능해야 한다.
- `watch`는 장기 실행 명령이므로 Ctrl+C로 구독을 해제한다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/ops.rs` 파일 구조와 SDI LM-8 불변식 및 플러그인 아키텍처 문서를 근거로 추론되었다. `watch` 서브커맨드의 SSE 방식과 런타임 파일 경로 규칙은 SDI 설계 명세 기반 추론이다.

## 미확정 (OPEN)

- `start` 명령 시 `sdid` 바이너리 경로를 자동 탐색하는 전략(PATH 환경변수, 고정 경로, 설정 파일)이 코드 레벨에서 확인되지 않았다.
- `restart`가 graceful shutdown(진행 중 요청 완료 후 종료)인지 즉시 강제 종료인지 확인이 필요하다.
- `watch` 구독 시 이벤트 종류 필터링 기능의 존재 여부가 불확실하다.
- 데몬이 비정상 종료되었을 때 PID/포트 파일이 잔존하는 경우의 복구 전략(자동 정리 여부)이 명확하지 않다.
