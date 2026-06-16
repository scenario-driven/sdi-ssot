---
id: decision.desktop-as-addon
kind: Decision
title: sdi-desktop은 sdid를 sidecar로 소비하는 add-on이며 sdi-plugin에 역의존하지 않는다
purpose: "데스크탑 GUI 앱(Tauri)을 sdi-plugin과 같은 repo에 넣을지, 별도 add-on repo로 분리할지"
definition: "sdi-desktop은 별도 GitHub repo(scenario-driven/sdi-desktop)의 Tauri 2 shell add-on이다. sdid(데몬 바이너리)를 sidecar로 spawn하고 sdi-plugin/plugin/web/dist 번들을 소비한다. 의존은 단방향이며 sdi-plugin은 sdi-desktop에 역의존하지 않는다."
relatesTo:
  - { to: domain.plugin-runtime, type: impacts, note: "sdi-plugin의 런타임 모델은 desktop 없이도 완결된다는 사실이 이 결정으로 확인된다" }
  - { to: platform.sdi, type: impacts, note: "SDI 제품의 배포 형태에서 desktop은 선택적 add-on임이 이 결정으로 정의된다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI 대시보드는 daemon이 HTTP로 서빙하는 CSR SPA다. 브라우저에서 접근하거나, Tauri로 감싸 네이티브 앱처럼 열거나 두 방법이 있다. Tauri를 같은 repo에 넣으면 빌드 파이프라인이 복잡해지고 플러그인 배포 크기가 늘어난다. Tauri를 별도 repo에 두면 의존 방향이 단방향으로 명확해진다.

SDI의 핵심 가치는 Claude Code 플러그인이 본체라는 점이다. desktop 앱은 편의를 더하는 add-on이지, 없으면 제품이 성립하지 않는 필수 구성 요소가 아니다.

## 결정 (Decision)

sdi-desktop을 `scenario-driven/sdi-desktop` 별도 repo의 Tauri 2 shell로 분리한다. 의존 방향은 단방향이다.

- `sdi-desktop`이 소비하는 것: sdid 바이너리(sidecar로 spawn), `sdi-plugin/plugin/web/dist`의 SPA 번들(포함 또는 링크)
- `sdi-plugin`은 `sdi-desktop`에 대해 아무것도 알지 않는다. 역의존 없음.

사용자는 desktop 앱 없이 브라우저에서 `http://localhost:<port>/`로 대시보드에 접근하거나, Tauri 앱을 설치해 네이티브 창으로 열 수 있다. 두 방법 모두 지원되며 sdi-plugin 자체는 어느 쪽도 강제하지 않는다.

## 근거와 결과 (Consequences)

단방향 의존으로 분리하면 다음이 따라온다.

- sdi-plugin의 배포 파이프라인이 단순해진다: Tauri 빌드 없이 Rust + Node.js로만 구성된다.
- sdi-desktop의 릴리즈 일정이 sdi-plugin과 독립된다: SPA나 daemon API 계약이 바뀌면 sdi-desktop이 따라가는 방향이지, 반대 방향으로 sdi-plugin을 제약하지 않는다.
- 인터페이스 계약(sdid 바이너리 API + web dist 번들 경로)은 별도 artifact로 관리되어 두 repo가 같은 계약을 신뢰한다.

sdi-web이라는 별도 대시보드 repo가 한때 존재했으나 sdi-plugin/plugin/web/으로 흡수 통합되었다(decision.sdi-web-retired 참조). sdi-desktop은 이와 달리 desktop-specific 목적을 가져 계속 별도 repo로 유지된다.

<!-- provenance: scenario-driven/CLAUDE.md "sdi-desktop/ → github.com/scenario-driven/sdi-desktop (add-on — Tauri 2 shell. sdid sidecar spawn + sdi-plugin/plugin/web/dist 번들 소비. 단방향 소비자.)". scenario-driven/CLAUDE.md "repo 간 의존은 단방향: sdi-desktop → { sdid (sidecar), sdi-plugin/plugin/web/dist (bundle) }. sdi-plugin 은 sdi-desktop 에 역의존하지 않는다." -->
