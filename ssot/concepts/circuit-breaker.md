---
id: concept.circuit-breaker
kind: Concept
title: Circuit Breaker(회로 차단기)
definition: "D18 결정. 단일 UI 동작으로 모든 자율성 모드를 즉시 L3으로 강등하는 긴급 제동 장치. 발동 중에는 진행 중인 결정도 다음 게이트 시점에 L3으로 적용된다. 해제될 때까지 유지된다."
relatesTo:
  - { to: concept.autonomy-policy, type: relates-to, note: "회로 차단기가 발동되면 모든 AutonomyPolicy의 유효 모드가 L3이 된다." }
  - { to: concept.decision, type: relates-to, note: "회로 차단기 발동 중 결정 자동 적용(L4/L5)이 차단된다." }
  - { to: domain.autonomy, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/autonomy_policy.rs
  - sdi-plugin/plugin
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Circuit Breaker(회로 차단기)는 사용자가 에이전트의 자율성을 즉시 최대로 제한할 수 있는 비상 제동 장치다. SDI는 기본적으로 L5(자동 적용+사후 통보) 모드로 운영될 수 있는데, 예상치 못한 자동 결정이 연속으로 이루어질 때 사용자가 개입해 모든 결정을 다시 인간 확인 필수(L3)로 전환할 수 있다.

회로 차단기의 핵심 성질은 **즉시성**이다. 단일 UI 동작(한 번의 사용자 행위)으로 발동되며, 발동 즉시 모든 활성 AutonomyPolicy의 유효 모드가 L3으로 강등된다. 진행 중인 결정도 다음 게이트 체크 시점에 L3을 적용받는다.

회로 차단기는 에이전트 간 통신 채널(M1~M5)에는 영향을 미치지 않는다(D19). 에이전트들이 서로 메시지를 주고받고 작업을 분담하는 것은 계속되지만, 최종 결정을 적용하는 단계에서 반드시 인간 확인을 거쳐야 한다.

## 엔티티 (DB)

회로 차단기 상태는 AutonomyPolicy 엔티티와 별개로 관리되는 시스템 레벨 플래그다. DB에 별도 테이블이 있는지, 아니면 daemon 메모리 상태로 관리되는지 구체적 구현은 확인 필요.

## API 표면

회로 차단기는 UI 또는 CLI의 단일 동작으로 발동한다. AutonomyMode의 `demoted()` 메서드가 L3을 반환하는 것이 차단기 발동의 코드 표현이다.

## 불변식

- **즉시 강등(D18)**: 발동 즉시 모든 autonomy mode → L3.
- **통신 무영향(D19)**: 에이전트 간 통신 채널에는 영향 없음.

## 구현 위치 (provenance)

`AutonomyMode::demoted()` 메서드(항상 L3 반환)는 `sdi-plugin/crates/core/src/autonomy_policy.rs`에 있다. UI/훅 레벨의 회로 차단기 발동 로직은 `sdi-plugin/plugin/` 아래 훅 파일들에 있다.

## 미확정 (OPEN)

- [ ] OPEN: 회로 차단기 상태를 영구 저장하는지(daemon 재시작 후에도 유지) 아니면 메모리 상태인지 확인 필요.
- [ ] OPEN: 회로 차단기 해제 방법 및 권한 확인 필요.
