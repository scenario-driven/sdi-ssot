---
id: endpoint.cli-doctor
kind: Endpoint
title: "CLI `sdi doctor`"
definition: "설치 상태·데몬 연결·스킬 파일 존재 여부 진단 및 자동 복구 안내"
realizedBy: []
implementedIn:
  - "sdi-plugin/crates/cli/src/commands/ops.rs"
relatesTo:
  - to: "domain.plugin-runtime"
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

`sdi doctor` 는 SDI 시스템의 설치 상태와 런타임 환경을 진단하고, 발견된 문제에 대한 수정 방법을 안내하는 CLI 명령이다. 설치 후 초기 확인, 업그레이드 후 환경 검증, 비정상 동작 트러블슈팅 시나리오에서 첫 번째로 실행하는 자기진단 도구다.

Claude Code 플러그인의 `runSessionStart()` 내부 `ensureInstalled()` 함수가 자동으로 수행하는 동일한 체크 항목을 사람이 읽을 수 있는 형태로 출력한다. 즉, `sdi doctor`가 정상이면 Claude Code 세션 시작 시 SDI 플러그인이 문제 없이 초기화된다는 것을 의미한다.

## 요청 / 응답

단일 진단 뷰를 출력한다. 각 체크 항목은 통과(OK)/경고(WARN)/실패(FAIL) 상태로 표시되며, 실패 항목에는 권장 수정 명령이 함께 제시된다.

진단 항목은 다음과 같다.

**바이너리 탐지**: `sdi` CLI 및 `sdid` 데몬 바이너리가 PATH 또는 예상 위치에 존재하는지 확인한다. 발견되면 버전과 경로를 표시하고, 발견되지 않으면 GitHub Releases에서 다운로드하는 방법을 안내한다.

**스킬 파일 존재 확인**: Claude Code 플러그인이 의존하는 4개의 SKILL.md 파일이 정해진 위치에 존재하는지 확인한다. 누락된 파일이 있으면 생성 또는 복원 방법을 안내한다.

**데몬 연결 확인**: `~/.cache/sdi/sdid.port` 파일을 읽어 포트를 확인한 후, 해당 포트의 `/health` 엔드포인트에 요청을 보내 데몬이 응답하는지 확인한다. 응답하지 않으면 `sdi daemon start` 명령을 안내한다.

**캐시 경로 권한 확인**: `~/.cache/sdi/` 경로가 존재하고 현재 사용자가 읽기/쓰기 권한을 가지고 있는지 확인한다. LM-8 불변식에 따른 경로 접근 가능 여부를 보장한다.

## 권한 / 제약

- `sdi doctor`는 읽기 전용 진단 명령이다. 문제를 자동으로 수정하지 않고 수정 방법만 안내한다.
- 데몬 연결 확인은 `~/.cache/sdi/sdid.port`에서 포트를 읽어 로컬에만 요청을 보낸다. 외부 네트워크 요청을 포함하지 않는다.
- 4개의 SKILL.md 파일 위치와 내용은 SDI 플러그인 버전에 따라 변경될 수 있다.

## provenance

이 엔드포인트 정의는 `sdi-plugin/crates/cli/src/commands/ops.rs` 파일 구조와 SDI 플러그인 런타임 설계 문서를 근거로 추론되었다. `ensureInstalled()` 함수명과 점검 항목 4개 구성은 SDI 설계 의도 기반 추론이며, 코드에서 직접 확인된 것이 아니다.

## 미확정 (OPEN)

- 정확히 4개의 SKILL.md 파일이 무엇인지(파일명과 위치)가 코드 레벨에서 확인되지 않았다.
- `sdi doctor`가 일부 문제를 자동 수정하는 `--fix` 플래그를 지원하는지 불확실하다.
- `ensureInstalled()`와 `sdi doctor`의 점검 항목이 완전히 동일한지, 아니면 `sdi doctor`가 추가 항목을 더 포함하는지 확인이 필요하다.
- 진단 출력 형식이 `--json` 플래그로 기계 가독 JSON을 지원하는지 불확실하다(CI 파이프라인에서의 활용 가능성).
