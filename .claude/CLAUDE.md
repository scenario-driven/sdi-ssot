# sdi-ssot — SSOT 데이터 레포 (Claude 작업 컨텍스트)

이 레포는 **Scenario-Driven Implementation (SDI) 제품의 SSOT 데이터(콘텐츠)만** 담는다. 본문은 코드 레포(`sdi-plugin` 본체 · `sdi-desktop` add-on)에서 역추론한 자연어 문서이며, confidence 대부분 `inferred`(담당자 검증 대기). 도구는 별도 레포 — 작성 스킬·MCP 는 `ssot-studio/ssot-plugin`, 뷰어(웹)는 `ssot-studio/ssot-web`. 데이터/도구는 분리되어 있다 — 이 레포에 스킬 코드를 복사하지 않는다.

## 멀티레포 좌표 (provenance / --root)

이 레포는 `scenario-driven` 워크스페이스의 형제 레포들을 가리킨다. `implementedIn` provenance 경로는 워크스페이스 루트(`scenario-driven/`) 기준 상대경로로 적는다 — 예: `sdi-plugin/...`, `sdi-desktop/...`. 검증 시 `--root ../..`(이 레포의 `ssot/` 에서 두 단계 위 = `scenario-driven/`)로 공통 조상을 지정한다.

## 이 레포에서 노드를 수정할 때 (Claude 필독)

- 노드 = `ssot/<kind>/<slug>.md`, **frontmatter 4축 + 자기완결 본문**.
- 수정은 **ssot 스킬 방법론**을 따른다 — 단일 정의: `ssot-studio/ssot-plugin` 의 `skills/ssot/reference/methodology.md` (없으면 글로벌 `~/.claude/skills/ssot/reference/methodology.md`).
- 핵심 규약: 자기완결(본문이 원본), 4축, authored/mirrored(mirrored는 source에서만 수정), 빈칸 일급(`confidence: unverified`/`OPEN`), 끊긴 엣지 금지.
- **코드 추론으로 채운 내용은 `confidence: inferred`** 로 둔다. 사실 검증 없이 `high` 로 올리지 않는다. 담당자 확인이 끝난 노드만 `high`.
- **수정 후 반드시 검증**: `build-graph.mjs ./ssot` → `verify.mjs ./ssot 90 --root ../..` 로 dangling/schema/drift 0 확인 (스크립트는 ssot 플러그인 `skills/ssot/scripts/`).
- `_catalog.json` 은 생성물 — `.md` 수정 후 `build-graph` 로 재생성해 함께 커밋(공유 인덱스).
- 변경은 PR/push 로. (커밋은 사용자 지시 시에만)

## 하지 말 것
- 노드 본문을 "다른 문서 참조"로 때우기 (자기완결 위반).
- 추론을 사실처럼 단정 (코드 근거 없는 내용은 `unverified`/`OPEN` 로 둔다).
- 허위 엣지로 측면을 억지로 채우기 (정당한 빈칸은 그대로 둔다).
