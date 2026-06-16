---
id: concept.project
kind: Concept
title: Project(프로젝트)
definition: "SDI의 최상위 컨테이너. 모든 플랜·정책·지식·에이전트 노트가 프로젝트에 속한다. 고유 key(티켓 접두사), slug, 하나 이상의 anchored cwd(작업 디렉터리)를 가지며, enabled=false로 SDI 훅 레이어를 일시 비활성화할 수 있다."
relatesTo:
  - { to: concept.plan, type: relates-to, note: "플랜들은 프로젝트에 속한다." }
  - { to: concept.autonomy-policy, type: relates-to, note: "AutonomyPolicy들은 프로젝트에 속한다." }
  - { to: concept.knowledge-entry, type: relates-to, note: "지식 항목들은 프로젝트에 속한다." }
  - { to: domain.project-management, type: belongs-to }
implementedIn:
  - sdi-plugin/crates/core/src/project.rs
  - sdi-plugin/crates/db/src/migrations/001_core.sql
  - sdi-plugin/crates/db/src/migrations/008_project_metadata.sql
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 정의

Project(프로젝트)는 SDI에서 가장 높은 레벨의 조직 단위다. 개발자의 코드 저장소 또는 작업 공간 하나가 프로젝트 하나에 대응하는 것이 일반적이다.

프로젝트는 다음 핵심 속성을 갖는다.

- **key**: 짧은 티켓 접두사(예: "SDI"). 프로젝트 식별자이며 티켓 번호(예: SDI-1)의 앞부분이 된다. 전역적으로 유일해야 한다.
- **name**: 사람이 읽는 이름.
- **slug**: URL-safe 슬러그(name에서 자동 생성). 전역적으로 유일하다.
- **cwds**: 이 프로젝트에 연결된 작업 디렉터리 경로 목록. Claude Code가 이 경로들 중 하나에서 실행되면 이 프로젝트가 "현재 프로젝트"로 감지된다.
- **enabled**: true(기본값)이면 SDI 훅 레이어가 작동한다. false이면 해당 프로젝트의 모든 mutating gate를 건너뛴다 — 사용자가 SDI 없이 Claude Code를 운영하고 싶을 때 사용한다.
- **wiki_paths**: 대시보드 WikiView의 문서 트리 루트 경로 목록. 기본값은 `["docs"]`.
- **description**: 대시보드 프로젝트 설정에 표시되는 자유 서술.

프로젝트를 삭제하면 소속된 모든 플랜과 그 하위 항목이 cascade 삭제된다. 단, in_progress 태스크가 있으면 기본적으로 삭제를 거부하며 `?force=true`로 강제할 수 있다.

## 엔티티 (DB)

프로젝트 한 건은 다음 정보를 보존한다.

- **식별**: id(PROJ-<slug>), key(전역 유일), name, slug(전역 유일).
- **연결점**: cwds(JSON 배열).
- **제어**: enabled(boolean), wiki_paths(JSON 배열).
- **설명**: description(optional).
- **타임스탬프**: 생성, 수정 시각.

## API 표면

프로젝트는 `/sdi project` 명령 또는 대시보드의 프로젝트 설정에서 관리된다. `PUT /projects/:id`는 PATCH 방식(부분 업데이트)이다.

## 불변식

- **key 전역 유일**: 다른 프로젝트와 중복되는 key는 허용하지 않는다.
- **slug 전역 유일**: slug도 전역적으로 유일해야 한다.
- **삭제 시 in_progress 태스크 보호**: in_progress 태스크가 있는 플랜을 가진 프로젝트는 `?force=true` 없이 삭제 불가.

## 구현 위치 (provenance)

Project 구조체와 기본값(`default_wiki_paths`)은 `sdi-plugin/crates/core/src/project.rs`에 있다. 핵심 스키마는 마이그레이션 001, description/enabled/wiki_paths_json 컬럼은 마이그레이션 008에서 추가되었다.

## 미확정 (OPEN)

- [ ] OPEN: enabled=false 시 훅이 bypass되는 구체적 조건 — `project-disabled` audit 로그가 생성된다는 주석은 있으나 실제 audit 로그 경로 확인 필요.
