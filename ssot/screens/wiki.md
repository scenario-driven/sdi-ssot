---
id: screen.wiki
kind: Screen
title: Wiki 뷰 (RAG 지식 항목 브라우저)
purpose: "소로 빌더 또는 코딩 에이전트가 프로젝트의 RAG 지식 항목(knowledge entry)을 목록에서 선택해 본문을 읽는다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
  - persona.subagent
realizedBy: []
implementedIn:
  - sdi-plugin/plugin/web/src/views/WikiView.tsx
consumesApi: []
relatesTo:
  - { to: concept.knowledge-entry, type: reads, note: "scope=rag인 지식 항목을 제목·본문·태그로 검색하고 마크다운으로 렌더한다" }
impacts: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 화면 목적

프로젝트에 축적된 RAG 지식 항목(wiki/rag 유형)을 탐색하는 읽기 전용 뷰다. 좌측 패널에서 항목을 검색·선택하면 우측 영역에 해당 항목의 마크다운 본문이 렌더된다. 에이전트가 코드 작업 중 참고해야 할 결정·정책·배경 정보를 빠르게 찾는 데 사용한다.

## UI 요소 / 입력 필드

- **좌측 패널 — 검색 입력**: 제목·본문·태그를 대소문자 구분 없이 검색해 항목 목록을 실시간으로 걸러낸다.
- **좌측 패널 — 항목 목록**: 검색 결과 항목의 제목과 kind를 나열한다. 선택된 항목은 배경색으로 강조된다.
- **우측 영역 — 본문 뷰어**: 선택한 항목의 kind·scope 레이블, 제목, 태그 칩, 마크다운 본문(GFM 지원)을 표시한다.

## 표시 데이터 / 호출 API

화면 진입 시 프로젝트의 scope=rag인 지식 항목 목록을 데몬에서 조회한다. 목록이 있으면 첫 번째 항목을 자동으로 선택해 본문을 표시한다. 검색 필터는 인메모리에서 적용된다.

## 상태 / 엣지케이스

- **에러**: 에러 메시지를 빨간 텍스트로 표시한다.
- **항목 없음**: 좌측 패널에 "No knowledge yet." 안내를 표시한다.
- **검색 결과 없음**: 필터링된 목록이 비어 있으면 동일한 안내를 표시한다.
- **항목 미선택**: 우측 영역에 "Select a knowledge entry to view it." 안내를 표시한다.
- **목록 갱신**: SSE refreshKey 변경 시 목록을 재조회하지만, 이미 선택된 항목 ID는 유지된다(새 목록에 해당 ID가 없으면 선택이 해제된다).

## 미확정 (OPEN)
- [ ] OPEN: scope=wiki 등 다른 scope 항목이 있을 경우 Wiki 뷰에서 scope=rag만 표시하는 정책이 의도된 것인지 확인 필요.
