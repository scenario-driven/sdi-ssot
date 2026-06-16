---
id: invariant.daemon-single-instance
kind: Invariant
title: 한 XDG 홈에서 sdid 데몬은 단 하나만 실행된다
definition: "동일한 XDG 데이터 경로(~/.local/share/sdi/)를 사용하는 sdid 인스턴스는 동시에 하나만 실행될 수 있다. 두 번째 인스턴스 시작 시도는 기존 인스턴스가 살아있으면 거부된다."
governs:
  - domain.project-management
  - domain.plugin-runtime
  - concept.project
  - concept.plan
implementedIn:
  - "sdi-plugin/crates/daemon/src/main.rs"
  - "sdi-plugin/crates/db/src/paths.rs"
decidedBy: []
crossesBoundary: false
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 제약

`sdid`는 SQLite 단일 DB 파일(`~/.local/share/sdi/sdi.db`)을 상태의 단일 진실 공급원으로 사용한다. 이 설계에서 두 개의 데몬이 같은 DB를 동시에 쓰면 SQLite의 ACID 보장에도 불구하고 고수준 의사결정 일관성이 깨질 수 있다. 예를 들어 Round 활성화, Task 상태 전환, Scenario claim 할당 같은 연속 작업이 두 인스턴스에서 인터리빙되면 Race 상태가 발생한다.

단일 인스턴스 강제는 `~/.cache/sdi/` 아래의 PID 파일 또는 sokcet 파일을 통해 구현된다. 기존 인스턴스가 살아있는지 확인(liveness probe)한 뒤 새 인스턴스가 시작되고, 기존이 살아있으면 새 인스턴스는 종료된다.

D29(Multi-session resource claims)는 여러 Claude Code 세션이 같은 `sdid`에 접속하는 시나리오를 지원한다. 복수 세션이 허용되지만 복수 데몬은 허용되지 않는다. 데몬이 하나여야 세션 간 resource claim 충돌을 데몬이 라우터로서 중재할 수 있다.

## 깨지면 무슨 일이 일어나나

두 데몬이 동시에 실행되면 두 인스턴스가 각자 독립적인 상태를 관리하는 것처럼 보이지만 실제로는 같은 SQLite 파일을 공유한다. Round 활성화 체크, Task done 전환, Scenario claim 할당 같은 read-then-write 패턴의 작업이 두 인스턴스에서 동시에 실행되면 체크가 통과한 뒤 다른 인스턴스가 먼저 쓰는 race가 발생한다. 더 심각한 문제는 두 인스턴스가 각자 다른 포트/소켓을 점유해 Claude Code 훅이 어느 데몬에 접속할지 모르게 되는 것이다. 이 경우 PreToolUse 활성 태스크 게이트가 잘못된 데몬에 질의하여 부정확한 차단/허용 판단을 내린다.

## 코드에서 어떻게 강제되나

`sdid` 시작 시 `~/.cache/sdi/`의 PID/소켓 파일을 확인하여 기존 프로세스가 살아있는지 probe한다. 기존 인스턴스가 응답하면 새 인스턴스는 에러 메시지를 출력하고 종료한다. 응답이 없으면(기존 인스턴스가 비정상 종료된 경우) stale 파일을 정리하고 새 인스턴스가 시작된다. Clawket의 `daemon-flock-single-instance` 불변식에서 계승된 패턴이다.

## 미확정 (OPEN)
- [ ] OPEN: decidedBy(단일 데몬 인스턴스 강제 정책의 SDI 측 결정 엔트리) 연결 필요
