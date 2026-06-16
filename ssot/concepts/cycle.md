---
id: concept.cycle
kind: Concept
title: Cycle(사이클) — 폐기됨
definition: "Clawket v3에 있던 작업 실행 주기 엔티티. SDI에서는 Round(라운드)로 대체되었다(D4). Cycle과 Round는 이름만 다른 것이 아니라 의미도 다르다 — Round는 시나리오 검증을 중심으로 재정의되었다."
relatesTo:
  - { to: concept.round, type: relates-to, note: "SDI에서 Cycle의 역할을 대체한 것이 Round다. 단순 이름 변경이 아니라 의미 재정의를 동반한다." }
  - { to: domain.round-execution, type: belongs-to }
implementedIn: []
owner: TBD
lifecycle: deprecated
confidence: inferred
lastVerified: ""
---

## 정의

Cycle(사이클)은 Clawket v3에서 태스크를 활성화하는 시간 단위였다. 특정 기간 또는 스프린트 안에서 어떤 태스크들을 실행할지를 정하는 컨테이너였으며, active Cycle에 배정된 태스크만 시작할 수 있었다.

SDI에서는 Cycle 개념이 Round(라운드)로 대체되었다(D4). 단순한 이름 변경이 아니라 의미의 재정의다.

- **Clawket의 Cycle**: 기간 기반 스프린트 단위. 태스크를 선택해 배정.
- **SDI의 Round**: 시나리오 검증 중심의 실행 단위. 시나리오들이 태스크로 분해되어 검증.

SDI 코드베이스에는 Cycle 관련 소스 파일이 존재하지 않는다. DB 마이그레이션에도 cycles 테이블이 없다.

## 엔티티 (DB)

SDI에 존재하지 않음.

## API 표면

SDI에 존재하지 않음.

## 불변식

해당 없음.

## 구현 위치 (provenance)

SDI에 구현 없음. 폐기된 개념.

## 미확정 (OPEN)

없음. 이 개념은 SDI에서 Round로 대체되었다(D4).
