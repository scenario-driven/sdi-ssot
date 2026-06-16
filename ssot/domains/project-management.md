---
id: domain.project-management
kind: Domain
title: 프로젝트 관리 (Project Management)
purpose: "SDI의 최상위 작업 컨테이너인 Project를 관리하고, 작업 디렉터리(cwd)와 프로젝트를 연결해 훅 게이트가 올바른 프로젝트에 적용되도록 한다."
definition: "Project 엔티티의 생성·설정·삭제를 담당하는 경계. 프로젝트는 플랜·라운드·시나리오·결정의 최상위 컨테이너다. 작업 디렉터리(cwds)와 연결되어 훅 게이트가 올바른 프로젝트 컨텍스트를 인식하게 한다. enabled=false면 해당 프로젝트의 모든 훅 게이트가 비활성화된다."
servesPersona:
  - persona.solo-builder
  - persona.coding-agent
relatesTo:
  - to: domain.planning
    type: relates-to
    note: 플랜은 반드시 하나의 프로젝트에 귀속된다.
  - to: domain.plugin-runtime
    type: relates-to
    note: 훅이 현재 cwd를 프로젝트에 매핑해 올바른 훅 게이트를 적용한다.
  - to: concept.project
    type: relates-to
    note: 이 도메인의 핵심 개념이다.
  - to: concept.ticket-number
    type: relates-to
    note: 프로젝트의 key가 티켓 번호 prefix다(예: SDI-001).
governedBy: []
realizedBy: []
impacts:
  - domain.planning
  - domain.plugin-runtime
implementedIn:
  - "sdi-plugin/crates/core/src/project.rs"
  - "sdi-plugin/crates/daemon/src/router/project.rs"
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

## 목적

이 도메인은 SDI 전체 데이터의 최상위 조직 단위를 관리한다. 모든 플랜·시나리오·결정·라운드는 하나의 프로젝트 아래에 있다. 더 중요하게는, 현재 빌더가 작업 중인 디렉터리(cwd)가 어떤 프로젝트에 속하는지를 매핑해, 훅 게이트가 올바른 프로젝트 컨텍스트에서 작동하게 한다.

## 경계와 핵심 개념

이 도메인이 다루는 것:

- **프로젝트 등록**: 이름·키(티켓 prefix)·작업 디렉터리를 설정해 프로젝트를 만든다. 키는 전역 유일이고 변경 불가하다(변경하려면 새 프로젝트 생성).

- **cwd 매핑**: 프로젝트의 `cwds` 목록에 작업 디렉터리를 등록한다. 훅이 현재 cwd를 이 목록과 대조해 올바른 프로젝트를 찾는다.

- **enabled 플래그**: `enabled=false`로 설정하면 해당 프로젝트 cwd에서의 모든 훅 게이트(D21 위임, 활성 태스크, D29 클레임)가 비활성화된다. 프로젝트를 "SDI 없이" 잠시 작업해야 할 때 사용한다. 비활성화 모든 게이트 통과는 `audit=project-disabled`로 기록된다.

- **wiki_paths**: 대시보드 WikiView에서 보여줄 문서 디렉터리. 기본값은 `["docs"]`.

- **삭제와 cascade**: `DELETE /projects/:id`는 프로젝트에 귀속된 모든 데이터를 cascade 삭제한다. in_progress 태스크가 있으면 기본적으로 거부(`?force=true`로 override).

포함되지 않는 것: 플랜 내용(domain.planning), 훅 게이트 로직 자체(domain.plugin-runtime), 시나리오 관리(domain.scenario-management).

## 기능

- 프로젝트를 등록하고 cwd를 연결한다.
- 프로젝트 설정(이름·설명·wiki_paths·enabled)을 갱신한다.
- 현재 cwd로 프로젝트를 조회한다(훅 컨텍스트 해석).
- 프로젝트와 모든 귀속 데이터를 삭제한다.

## 시스템 흐름

빌더가 새 사이드 프로젝트를 시작할 때 `sdi project create`로 등록하고 작업 디렉터리를 지정한다. 이후 해당 디렉터리에서 Claude Code를 사용하면 훅이 이 등록을 조회해 올바른 훅 게이트를 적용한다. 프로젝트에 플랜을 만들고 시나리오를 추가하고 라운드를 실행하는 모든 작업이 이 프로젝트 컨텍스트 아래에서 이루어진다.

## 다른 도메인과의 관계

프로젝트는 SDI 데이터 모델의 루트다. 모든 1등 시민 엔티티(플랜·요구사항·결정·시나리오·라운드·자율성 정책·협업 패턴)가 프로젝트에 귀속된다. 플러그인 런타임(domain.plugin-runtime)이 cwd→프로젝트 매핑을 사용해 훅을 올바르게 적용한다.

## 미확정 (OPEN)
- [ ] OPEN: 하나의 cwd가 여러 프로젝트에 등록될 수 있는지, 아니면 1:1 제약이 있는지 코드 확인 필요.
