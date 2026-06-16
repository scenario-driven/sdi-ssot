---
id: decision.sdi-web-retired
kind: Decision
title: sdi-web 별도 repo의 대시보드 SPA는 sdi-plugin/plugin/web/으로 흡수 통합되었다
purpose: "대시보드 SPA를 별도 repo(sdi-web)로 유지할지, sdi-plugin repo 안으로 통합할지"
definition: "sdi-web(scenario-driven/sdi-web)은 더 이상 유지보수되지 않으며 제거 대상이다. 대시보드 SPA는 sdi-plugin/plugin/web/으로 흡수 통합되었고, 새 변경은 sdi-plugin 트리에서만 일어난다."
relatesTo:
  - { to: domain.plugin-runtime, type: impacts, note: "웹 UI가 플러그인 repo 안에 통합되어 배포 단위가 하나가 된다" }
  - { to: platform.sdi, type: impacts, note: "SDI 제품의 물리적 구성 방식이 이 결정으로 단순화된다" }
supersedes: []
owner: TBD
lifecycle: active
confidence: inferred
lastVerified: ""
---

> Decision은 append-only다. 결정이 바뀌면 이 파일을 고치지 말고 새 Decision을 만들어 `supersedes`로 잇는다.

## 맥락 (Context)

SDI 초기 설계에서는 대시보드 SPA를 `scenario-driven/sdi-web`이라는 별도 repo로 관리했다. 그러나 sdi-plugin이 Claude Code 플러그인이 본체라는 설계 원칙(decision.plugin-is-the-product)에서, daemon이 SPA를 직접 서빙하는 구조(decision.daemon-serves-spa)로 굳어지면서, SPA가 플러그인 디렉토리 안에 있어야 배포가 원자적으로 이루어진다는 것이 명확해졌다.

별도 repo를 유지하면 sdi-plugin 변경 시 sdi-web도 맞춰 변경해야 하는 cross-repo 조율 비용이 생긴다. 또한 플러그인 배포 시 sdi-plugin 하나만 가져오면 되는데, SPA가 별도 repo에 있으면 두 repo를 조율하는 추가 파이프라인이 필요하다.

## 결정 (Decision)

`scenario-driven/sdi-web` repo를 더 이상 유지보수하지 않는다. 대시보드 SPA(Vite + React 19 + Tailwind 4)는 `sdi-plugin/plugin/web/`으로 이전·통합되었다. 새 변경은 sdi-plugin 트리에서만 일어난다. 흡수 시점의 출처는 `sdi-plugin/plugin/web/SNAPSHOT.json`에 기록되어 있다. sdi-web 업스트림 repo는 제거 대상이다.

## 근거와 결과 (Consequences)

통합으로 얻는 이점은 두 가지다.

첫째, 배포 원자성: `plugin/web/dist`가 플러그인 디렉토리 안에 있으므로 플러그인 설치 하나로 API + 웹 UI + 바이너리가 모두 배포된다.

둘째, 개발 단순성: SPA와 daemon API가 같은 PR에서 함께 변경되어 인터페이스 불일치가 코드 리뷰 단계에서 즉시 보인다. cross-repo 버전 맞춤 비용이 없어진다.

sdi-desktop(별도 Tauri add-on repo)은 이 통합의 영향을 받지 않는다. sdi-desktop은 `sdi-plugin/plugin/web/dist` 번들을 소비하는 단방향 소비자 구조를 유지한다.

<!-- provenance: scenario-driven/CLAUDE.md "> sdi-web (retired) — 대시보드 SPA 는 sdi-plugin/plugin/web/ 으로 흡수되었다. 업스트림 github.com/scenario-driven/sdi-web 는 더 이상 유지보수되지 않으며 제거 대상이다. ... 흡수 시점의 출처는 plugin/web/SNAPSHOT.json 에 박혀 있다." -->
