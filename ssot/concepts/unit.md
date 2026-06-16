---
id: concept.unit
kind: Concept
title: Unit(유닛) — 폐기됨
definition: "Clawket v3에 있던 태스크 그룹핑 엔티티. SDI에서는 제거되었다(D4). SDI는 Unit 대신 시나리오 태그를 사용해 시나리오를 분류한다. 본 개념 노드는 Clawket→SDI 마이그레이션 맥락과 '없는 개념'임을 명확히 하기 위해 보존된다."
relatesTo:
  - { to: concept.scenario, type: relates-to, note: "SDI에서 Unit의 역할을 대체한 것은 시나리오의 tags 필드다." }
  - { to: domain.task-decomposition, type: belongs-to }
implementedIn: []
owner: TBD
lifecycle: deprecated
confidence: inferred
lastVerified: ""
---

## 정의

Unit은 Clawket v3에서 태스크를 논리적으로 묶는 그룹핑 컨테이너였다. Plan → Unit → Task의 계층 구조에서 Unit은 중간 단계로, 단순 그룹핑 엔티티이며 자체 상태 기계는 없었다.

SDI에서는 Unit 개념이 제거되었다(결정 D4). 시나리오가 first-class citizen이 된 SDI에서 그룹핑은 시나리오의 `tags` 필드로 충분하다고 판단했기 때문이다. SDI의 계층 구조는 Project → Plan → Round → Task이며 Unit 계층이 없다.

SDI 코드베이스(`crates/core/src/`)에는 Unit 관련 소스 파일이 존재하지 않는다. DB 마이그레이션에도 units 테이블이 없다.

## 엔티티 (DB)

SDI에 존재하지 않음. DB에 units 테이블 없음.

## API 표면

SDI에 존재하지 않음.

## 불변식

해당 없음.

## 구현 위치 (provenance)

SDI에 구현 없음. 폐기된 개념.

## 미확정 (OPEN)

없음. 이 개념은 SDI에서 명시적으로 제거되었다(D4).
