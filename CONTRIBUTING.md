# 기여 가이드 — sdi-ssot

이 레포는 **Scenario-Driven Implementation (SDI)의 SSOT 데이터(콘텐츠)** 입니다. 노드 `.md`를 수정해 PR을 올리면, CI가 무결성을 검사한 뒤 머지합니다. 본문은 `sdi-plugin`(플러그인·Rust CLI/데몬/MCP·대시보드 SPA)·`sdi-desktop`(Tauri add-on) 코드에서 역추론한 자연어 문서로, 현재 confidence 대부분 `inferred` — **담당자 검증으로 `high` 로 승급**하는 것이 기여의 핵심입니다. 도구는 별도 레포입니다 — 작성 스킬·MCP 는 [`ssot-studio/ssot-plugin`](https://github.com/ssot-studio/ssot-plugin), 뷰어(웹)는 [`ssot-studio/ssot-web`](https://github.com/ssot-studio/ssot-web).

## 무엇을 고치나

```
ssot/
├── platform.md · personas/ · domains/ · concepts/ · capabilities/
├── components/ · integrations/ · invariants/ · decisions/
├── screens/ · endpoints/        ← 노드 1개 = .md 1개
└── _catalog.json                ← 그래프 인덱스(생성물, 커밋됨)
```

노드 = **frontmatter(4축 측면 + 관계 엣지) + 본문(자기완결 원본)**. 본문은 요약이 아니라 진실의 원본 — *코드가 사라져도 노드만으로 재현 가능*해야 합니다.

## 수정하는 법 (3가지, 모두 결과는 `.md`+PR)

**① ssot 스킬 (권장)** — Claude Code에 `ssot-studio/ssot-plugin` 플러그인 설치 후:
```
/ssot add Concept <주제>   # 코드 정독+대화로 노드 작성
/ssot fill                 # 빈 노드 채움
/ssot verify               # 무결성 검사
```

**② 직접 편집** — `.md`를 직접 고침(아래 규약 준수).

**③ 뷰어 위젯** — 뷰어 웹(dev)에서 노드를 보다가 위젯으로 수정 요청 → 로컬 Claude Code가 편집.

## 규약 (PR 전 필수)

- **자기완결**: 본문에 원본 데이터를 담는다(다른 문서 "참조"로 때우지 않음).
- **4축**: ① 비즈니스·제품 ② 도메인 개념·관계·불변식 ③ 시스템·경계·영향 ④ 거버넌스·근거·생명주기.
- **authority**: `authored`(원본=이 레포) / `mirrored`(원본=코드 레포 등 — **직접 편집 금지**, source에서 고치고 sync).
- **confidence**: `high`(사람 검증) / `inferred`(코드 추론) / `unverified`(빈칸). 빈칸은 에러가 아니라 일급 상태(`TBD`/`OPEN`). 코드 추론으로 채운 노드는 `inferred` 로 두고, 담당자 확인 후 `high` 로 올린다.
- **끊긴 엣지 금지**: 가리키는 노드가 없으면 만들거나 연결 보류.

## PR 전 자가 검증

```bash
# ssot 플러그인 경로 기준 (예: ssot-plugin 클론 위치)
SSOT=<ssot-plugin>/skills/ssot/scripts
node $SSOT/build-graph.mjs ./ssot
node $SSOT/verify.mjs   ./ssot 90 --root ../..   # 공통조상 = scenario-driven 워크스페이스
# dangling 0 · schema 0 · drift 0 이어야 머지 가능 (CI 가 동일 검사)
```

상세 방법론: `ssot-studio/ssot-plugin` 의 `skills/ssot/reference/methodology.md`.
