---
id: concept.project-metadata
kind: Concept
title: ProjectMetadata(프로젝트 메타데이터)
definition: "마이그레이션 008에서 추가된 프로젝트 수명주기·설정 확장. description(자유 서술), enabled(훅 레이어 활성화 제어), wiki_paths_json(문서 트리 루트) 세 컬럼을 포함한다. Project 개념의 부속 데이터."
relatesTo:
  - { to: concept.project, type: belongs-to, note: "ProjectMetadata는 Project 엔티티의 확장 필드들이다. 별도 테이블이 아니라 projects 테이블의 컬럼들." }
  - { to: domain.project-management, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/project.rs
  - sdi-plugin/crates/db/src/migrations/008_project_metadata.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

ProjectMetadata는 SDI 프로젝트의 부가적인 수명주기·설정 정보를 의미한다. 초기 프로젝트 스키마(마이그레이션 001)에는 핵심 식별 정보만 있었고, 마이그레이션 008에서 세 가지 운영 설정이 추가되었다.

세 가지 추가 필드의 의미:

- **description**: 대시보드 프로젝트 설정에 표시되는 자유 서술 설명. `None`(설명 없음)과 빈 문자열(설명을 명시적으로 비움)을 구별한다.
- **enabled**: SDI 훅 레이어 활성화 여부. `true`(기본값)이면 이 프로젝트의 cwd에서 Claude Code가 동작할 때 모든 SDI 게이트(활성 태스크 확인, 위임 게이트, 클레임 차단)가 적용된다. `false`이면 모든 게이트를 건너뛰어 "SDI 없는 Claude Code"로 운영된다.
- **wiki_paths_json**: 대시보드 WikiView가 문서로 표시할 디렉터리 경로 목록. JSON 배열로 저장되며 기본값은 `["docs"]`다.

이 개념은 Project 엔티티에 이미 통합되어 있어 별도의 API 경로가 없다. Project 수정 API(`PUT /projects/:id`)로 함께 관리된다.

## 엔티티 (DB)

projects 테이블의 `description TEXT`, `enabled INTEGER NOT NULL DEFAULT 1`, `wiki_paths_json TEXT NOT NULL DEFAULT '["docs"]'` 세 컬럼으로 구현된다.

## API 표면

Project 생성·수정 API에서 함께 다룬다.

## 불변식

- **enabled 기본값 true**: 새로 생성된 프로젝트는 SDI 게이트가 기본으로 활성화된다.
- **wiki_paths 기본값 ["docs"]**: 문서 트리 루트를 명시하지 않으면 "docs" 디렉터리가 기본값이다.

## 구현 위치 (provenance)

`Project` 구조체의 해당 필드들(`description`, `enabled`, `wiki_paths`)은 `sdi-plugin/crates/core/src/project.rs`에 있다. DB 컬럼 추가는 마이그레이션 008에서 이루어졌다.

## 미확정 (OPEN)

없음.
