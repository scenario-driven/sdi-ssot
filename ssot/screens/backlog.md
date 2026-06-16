---
id: screen.backlog
kind: Screen
title: Backlog 뷰 (플랜 전체 태스크 목록)
purpose: "소로 빌더가 프로젝트 내 모든 플랜의 비완료 태스크를 한 화면에서 검색·필터링하며 파악하고, 개별 태스크로 이동한다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/views/BacklogView.tsx
consumesApi: []
relatesTo:
  - { to: concept.plan, type: reads, note: "플랜 단위로 그룹화해 태스크를 표시한다" }
  - { to: concept.task, type: reads, note: "비완료 태스크(todo/in_progress/blocked)를 상태·키워드 필터로 조회한다" }
  - { to: screen.detail-drawer, type: leads-to, note: "태스크 클릭 시 태스크 디테일 드로어를 연다" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

프로젝트 안의 모든 플랜에 걸쳐 아직 처리되지 않은 태스크를 플랜별로 묶어 보여주는 전체 백로그 뷰다. 라운드 단위로 보는 Board와 달리, 플랜 수준에서 비완료 태스크를 한 번에 볼 수 있다. 키워드 검색과 상태 필터를 조합해 원하는 태스크를 빠르게 찾고 클릭해 상세로 이동한다.

## UI 요소 / 입력 필드

- **키워드 검색 입력**: 태스크 설명 또는 short_code에서 입력 문자열을 포함하는 항목만 표시한다.
- **상태 필터 드롭다운**: All / Todo / In progress / Blocked / Done / Cancelled 중 선택한 상태만 표시한다.
- **플랜별 태스크 목록**: 각 플랜의 short_code·제목을 헤더로 두고, 그 아래 필터링된 태스크를 줄 목록으로 표시한다. 각 태스크 줄에는 상태 글리프(○·◐·⊘·●·✕), short_code, 설명, 상태 텍스트가 표시된다. 줄 클릭 시 해당 태스크의 디테일 드로어가 열린다.

## 표시 데이터 / 호출 API

화면 진입 시 프로젝트의 모든 플랜을 조회한 뒤, 각 플랜의 백로그 태스크를 데몬에서 가져온다. 데몬이 플랜별 백로그 엔드포인트를 지원하면 그것을 쓰고, 지원하지 않으면 라운드 목록을 가져와 각 라운드의 태스크를 병합한 뒤 done·cancelled를 제외해 백로그를 구성한다. 키워드·상태 필터는 인메모리에서 적용된다.

## 상태 / 엣지케이스

- **에러**: 에러 메시지를 빨간 텍스트로 표시한다.
- **플랜 없음**: "No plans in this project." 안내를 표시한다.
- **필터 결과 없음**: 해당 플랜 아래 "No tasks matching the current filter." 안내를 이탤릭으로 표시한다.
- **초기 로딩 중 태스크 영역**: rows가 비어있으면 플랜 목록 자체를 표시하지 않고 빈 상태를 보여준다.

## 미확정 (OPEN)
- [ ] OPEN: 완료(done)·취소(cancelled) 태스크도 백로그에 포함할 수 있도록 상태 필터가 동작하는지(코드에는 'All'에 포함됨) 확인 필요.
