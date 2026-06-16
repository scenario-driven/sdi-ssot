---
id: concept.knowledge-entry
kind: Concept
title: KnowledgeEntry(지식 항목)
definition: "프로젝트에 연결된 자유 형식 지식 저장소 항목. scope에 따라 rag(MCP 서버로 노출)/reference(로컬 전용)/archive(사용 중단)로 분류된다. 종류(kind)는 자유 문자열(예: decision, runbook, note, adr)이다."
relatesTo:
  - { to: concept.project, type: belongs-to, note: "지식 항목은 프로젝트에 속한다." }
  - { to: domain.knowledge-rag, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/knowledge.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

KnowledgeEntry(지식 항목)는 프로젝트와 연결된 자유 형식 지식을 보관하는 저장소다. SDI의 지식 항목은 세 가지 scope으로 분류된다.

- **rag(기본값)**: MCP 서버를 통해 외부로 노출된다. 에이전트들이 RAG(검색 증강 생성) 방식으로 조회하는 지식이다.
- **reference**: 사용자가 소유한 문서로 로컬에만 존재한다. MCP 노출 없음.
- **archive**: 사용 중단된 항목. 조회에서 제외되지만 삭제하지 않고 보존한다.

kind 필드는 자유 문자열이다. 예를 들어 "decision"(의사결정 기록), "runbook"(운영 절차), "note"(메모), "adr"(아키텍처 결정 기록) 등 프로젝트가 원하는 대로 분류할 수 있다.

각 항목은 제목, 마크다운 본문, 태그 목록, 출처 참조(file:line, URL, 커밋 SHA 등)를 갖는다.

## 엔티티 (DB)

지식 항목 한 건은 다음 정보를 보존한다.

- **소속**: 자신이 속한 프로젝트 참조.
- **분류**: scope(rag/reference/archive), kind(자유 문자열).
- **내용**: 제목, 마크다운 본문, 태그 목록.
- **출처**: source_ref(선택적 — 파일:라인, URL, 커밋 SHA 등).
- **타임스탬프**: 생성, 수정 시각.

## API 표면

지식 항목은 MCP 서버의 `search_knowledge` 도구로 조회된다(scope=rag인 항목만). CLI로도 관리할 수 있다.

## 불변식

- **MCP 노출 범위**: scope=rag인 항목만 MCP 서버를 통해 노출된다. reference와 archive는 로컬 전용.

## 구현 위치 (provenance)

KnowledgeEntry 도메인 구조체는 `sdi-plugin/crates/core/src/knowledge.rs`에 있다. DB 스키마는 마이그레이션 001에서 정의된다.

## 미확정 (OPEN)

- [ ] OPEN: FTS5(전문 텍스트 검색)가 knowledge 테이블에도 적용되는지, 또는 단순 LIKE 쿼리인지 확인 필요.
