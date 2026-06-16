---
id: concept.plan
kind: Concept
title: Plan(플랜)
definition: "SDI의 가장 상위 작업 컨테이너. 프로젝트에 속하며 시나리오·요건·결정·라운드를 포함한다. draft → active → completed 생애주기를 따르며, approve(draft→active) 게이트는 confirmed 시나리오 1개 이상 존재를 요구한다(D8)."
relatesTo:
  - { to: concept.project, type: belongs-to, note: "플랜은 반드시 하나의 프로젝트에 속한다." }
  - { to: concept.scenario, type: relates-to, note: "시나리오들은 플랜에 속한다." }
  - { to: concept.requirement, type: relates-to, note: "요건들은 플랜에 속한다." }
  - { to: concept.decision, type: relates-to, note: "결정들은 플랜에 속한다." }
  - { to: concept.round, type: relates-to, note: "라운드들은 플랜에 속하며 시나리오를 순차적으로 검증한다." }
  - { to: concept.autonomy-policy, type: relates-to, note: "플랜 범위의 자율성 정책이 이 플랜의 에이전트 행동을 제어한다." }
  - { to: concept.collaboration-pattern, type: relates-to, note: "플랜은 produced_via_pattern_id로 어떤 협업 패턴 아래 만들어졌는지 추적된다." }
  - { to: domain.planning, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/plan.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Plan(플랜)은 SDI 안에서 하나의 개발 목표나 기능 묶음에 해당하는 최상위 컨테이너다. 플랜은 "무엇을 만들 것인가(요건·시나리오)"와 "왜 이렇게 결정했는가(결정)"를 보존하고, "실제로 검증이 이루어지는 실행 단위(라운드)"를 포함한다.

플랜의 생애주기는 세 단계다.

- **draft**: 작성 중. 시나리오·요건을 추가·수정할 수 있다. 이 상태에서는 라운드를 시작할 수 없다.
- **active**: 승인(approve)됨. 라운드를 생성하고 에이전트가 실행할 수 있다. 시나리오 추가는 여전히 가능하나 변경이 기존 시나리오에 영향을 주면 DisruptionReview가 열린다.
- **completed**: 완료. active 상태에서만 전환 가능하다.

**approve 게이트(D8)**: draft→active 전환에는 confirmed 상태의 시나리오가 최소 1개 있어야 한다. 태스크 수나 요건 수는 무관하다.

플랜은 낙관적 동시성 제어를 위한 version 필드를 갖는다. 같은 플랜을 동시에 수정하려는 충돌을 감지할 수 있다.

플랜 본문(body)은 마크다운 자유 서술이며 스냅샷 전용이다(D12). 변경 근거는 Decision에 기록한다.

## 엔티티 (DB)

플랜 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 프로젝트 참조.
- **식별**: 프로젝트 내 고유 short_code, 제목.
- **본문**: 마크다운 설명(스냅샷 전용).
- **상태**: 현재 상태(draft/active/completed).
- **버전**: 낙관적 동시성용 단조 증가 정수.
- **패턴 출처(D23)**: produced_via_pattern_id.
- **타임스탬프**: 생성, 수정, 승인(approved_at), 완료(completed_at) 시각.

## API 표면

플랜은 `/sdi plan` 명령(/plan 슬래시 커맨드 포함)으로 생성·수정·승인·완료한다. 승인 시 confirmed 시나리오 존재 여부를 검사한다. 플랜 조회 시 소속 시나리오·요건·결정 목록을 함께 반환하는 옵션이 있다.

## 불변식

- **approve 게이트(D8)**: draft 상태일 때만 approve 가능. confirmed 시나리오 1개 이상 존재 필수.
- **complete 게이트**: active 상태일 때만 complete 가능.
- **스냅샷 전용 본문(D12)**: body에 변경 이력 패턴이 있으면 거부(Requirement와 동일 정신).

## 구현 위치 (provenance)

플랜 도메인 규칙(`check_can_approve`, `check_can_complete`)은 `sdi-plugin/crates/core/src/plan.rs`에 있다. DB 스키마는 마이그레이션 001에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: 플랜 approve 이후 시나리오를 추가할 수 있는지, 그리고 추가 시 자동으로 DisruptionReview가 열리는지 — PRD의 D9(disruption policy)와의 정확한 연동 규칙 확인 필요.
