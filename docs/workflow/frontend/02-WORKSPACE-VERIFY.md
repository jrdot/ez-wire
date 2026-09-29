# FE02 검증 보고서

- 기준 가이드: docs/workflow/frontend/02-WORKSPACE.md
- HANDOFF: docs/workflow/frontend/02-WORKSPACE-HANDOFF.md
- 최신 판정: 완료 — 1회차, 2026-09-25

<!-- 이 아래의 회차 섹션은 재검증 때마다 파일 끝에 추가한다. 이전 회차는 수정하지 않는다. -->

## 검사 이력 1회차 — 2026-09-25

- 작업 브랜치 / HEAD / merge-base: main / 712d656b5ca4ddc8ac37662d91b881d89b7a1d18 / 712d656b5ca4ddc8ac37662d91b881d89b7a1d18 (HEAD가 main과 동일, merge-base=HEAD → 비교할 별도 diff 없음)
- 작업 트리: 깨끗함 (`git status --porcelain`은 이번 배치 검사 첫 회차(01)가 생성한 `docs/workflow/frontend/01-FOUNDATION-VERIFY.md` 1건만 미추적 상태로 보고, 지시에 따라 검사 중지 사유에서 제외함. 그 외 변경·미추적 파일 없음)
- 검사 범위: frontend FE-02 (레이아웃·페이지·Canvas 탐색) — Header/Sidebar/Canvas/하단 5영역, 3페이지 제한, 좌표·입력·Snap 계약
- 상위 기준 문서: docs/workflow/common.md, docs/workflow/frontend/common.md, docs/core/DESIGN_SPEC.md, docs/core/USER_REQUIREMENTS_SPEC.md
- 전체 판정: 완료 — 범위 내 필수 항목 전부 PASS, FAIL 없음. WARNING 1건(성능 미측정)은 FE-02 완료 기준에 없는 공통 원칙 항목으로 완료 선언에 영향 없음.

### 체크리스트 및 검사 결과

| ID | 항목 | 요구 내용 | 기준 | 검사 대상 | 결과 | 근거 | 판정 |
|---|---|---|---|---|---|---|---|
| REQ-HEADER-001/DES-HEADER-001 | Header 구성 | 소형 로고, 모드 변경, 고정 도구상자 | USER §2, DESIGN §2 | `editor-shell.tsx` | 로고(`brand-compact`)·모드 셀렉터·`mode-toolbox`가 header 내 고정 배치 | editor-shell.tsx:164-170 | PASS |
| REQ-MODE-001/DES-MODE-001 | 3모드 명확 구분 | Mouse/Wire/Image, 현재 모드 강조 | 상동 | `editor-shell.tsx`, `globals.css` | `aria-pressed`로 활성 모드 표시, `[aria-pressed="true"]` teal 강조 CSS | editor-shell.tsx:166, globals.css:329, 시각 확인(workspace-expanded.png/1024-wire.png, Mouse/Wire 강조 확인) | PASS |
| REQ-TOOLBOX-001/DES-TOOLBOX-001 | 도구상자 고정 위치, 모드별 내용 전환 | 위치 불변, 내부만 모드별 변경 | 상동 | `editor-shell.tsx` | 모드별 삼항 분기로 도구만 교체, 위치는 동일 `mode-toolbox` DOM | editor-shell.tsx:168-170, e2e workspace.spec.ts:16-23(`toolbar.boundingBox().x` 모드 전환 후 불변 assert, `npm run test:e2e` 통과) | PASS |
| REQ-SIDEBAR-L-001/002, REQ-LIBRARY-001, REQ-PUBLIC-LIBRARY-001, DES-SIDEBAR-L-001/002 | 좌측 Sidebar 탭·접힘 | 라이브러리/공용부품 탭, 접기 시 Canvas 확장 | USER §3, DESIGN §3 | `editor-shell.tsx` | `role="tablist"`로 2탭 구성, `leftOpen` 토글 시 `--sidebar-left`가 216px↔36px로 전환되어 Canvas 확장 | editor-shell.tsx:180-185, 324, 337, globals.css:337, e2e workspace.spec.ts:38-41(접기 후 `canvas-frame` 폭 300px 이상 증가 assert) | PASS |
| REQ-SIDEBAR-R-001/002, DES-SIDEBAR-R-001/002 | 우측 Sidebar 속성·접힘 | 선택 객체 상세 속성, 접기/펼치기 | USER §4, DESIGN §3 | `editor-shell.tsx` | `inspector-selection`/`EntityProperties`가 좌표·크기·회전·배율 전체 표시, `rightOpen` 토글로 240px↔36px | editor-shell.tsx:201-206, 338, e2e workspace.spec.ts:39-40 | PASS |
| REQ-CANVAS-001 | Canvas DnD 배치 | 라이브러리 부품을 Canvas로 DnD 배치 | USER §5 | `editor-canvas.tsx` | `onDragOver/onDrop`이 `application/x-ezwire-part` payload를 `pointAt`+snap 처리 후 배치 | editor-canvas.tsx:98-100, e2e workspace.spec.ts:98-103(DnD로 2번째 컴포넌트 생성·정확한 좌표 확인) | PASS |
| REQ-CANVAS-SIZE-001/BG-001, DES-CANVAS-001/002 | 4용지·기본값·흰 배경 | A4/A3 가로세로, 기본 A4 가로, 배경 흰색 | 상동, DESIGN §4 | `coordinates.ts`, `project.ts`, `editor-shell.tsx` | `paperSize()` 4규격, 기본 `A4-landscape`+`#ffffff`, `<select>`로 4종 전환 | coordinates.ts:2-3, project.ts:33, editor-shell.tsx:189-191, e2e workspace.spec.ts:29-34(4용지 순회하며 `paper` width/height·fill 확인) | PASS |
| REQ-GRID-001, DES-CANVAS-003 | Grid는 Canvas 좌표 종속 | 화면이 아닌 Canvas와 함께 이동 | USER §5, DESIGN §4 | `editor-canvas.tsx` | Grid `<rect>`가 paper와 동일한 `document-layer` `<g transform>` 내부에 위치, 별도 화면 고정 레이어 없음 | editor-canvas.tsx:101-104 | PASS |
| REQ-PROJECT-001, REQ-PAGE-NAME-001, DES-PAGE-001/002 | 최대 3페이지, EZ-00N 명칭 | 4번째 생성 거부(UI+도메인), EZ-001부터 3자리 증가 | USER §6, DESIGN §5 | `editor.ts`, `project.ts`, `editor-shell.tsx` | `addSheet`가 `MAX_SHEETS` 초과 시 예외, 추가 버튼이 `sheets.length>=3`일 때 disabled, `createSheet`가 `EZ-${3자리}` 생성 | editor.ts:39, project.ts:29, editor-shell.tsx:195, 시각 확인(1/3 표시, workspace-expanded.png) | PASS |
| VAL-FE02-001 | 페이지 전환 시 선택/편집 격리 | 다른 페이지 객체가 선택/편집되지 않음 | 02-WORKSPACE.md §구현 작업 3, §완료 기준 | `editor.ts`(switchSheet) | `switchSheet`가 `selection:[]`, `preview:null`로 초기화, 시트별 `components/objects/groups` 배열 분리 저장 | editor.ts:15, project.ts:29(Sheet 구조), e2e workspace.spec.ts:114-125(페이지 전환 후 컴포넌트 미표시·선택 없음·복귀 시 원상태 확인) | PASS |
| REQ-BOTTOM-001/002, DES-BOTTOM-001~004 | 하단 1줄 요약 + 보조/시스템 기능 진입점 | 선택 요약 1 Line, Zoom/BOM/Table/Grid/Snap/FullScreen, Code/PDF/PNG/Help/System | USER §18, DESIGN §18 | `editor-shell.tsx`, `globals.css` | `selection-summary`가 `white-space:nowrap`+ellipsis, 7개 보조기능·5개 시스템 진입점 모두 렌더링(BOM/Table/Code/PDF/PNG는 `disabled`+"준비 중" 표시로 기능 미연결을 명시) | editor-shell.tsx:160, 210-217, globals.css:367, e2e workspace.spec.ts:36(`whiteSpace==="nowrap"` assert) | PASS(BOM/Table/Code/PDF/PNG는 진입 영역만 확보, 기능 연결은 FE-05 범위로 이연 — 가이드 §5 "기능 구현은 해당 단계에서 연결"과 일치) |
| REQ-NAV-001, DES-NAV-001 | Wheel 상하/Shift+Wheel 좌우/Ctrl·Alt+Wheel 줌 | 02-WORKSPACE.md §입력 동작 계약, USER §12, DESIGN §10 | `workspace-input.ts` | `wheelViewport`가 ctrl/alt→zoom, shift→가로, 기본→세로 순으로 분기, deltaMode 정규화 | workspace-input.ts:7-12, workspace.test.ts:20-25(단위 테스트), e2e workspace.spec.ts:44-60(실제 브라우저 wheel 이벤트로 pan.x/y, zoom anchor 검증) | PASS |
| REQ-PAN-001, DES-PAN-001 | 휠버튼+Drag / Space+Drag Pan | 02-WORKSPACE.md §입력 동작 계약, USER §12, DESIGN §10 | `editor-canvas.tsx` | `onPointerDown`이 `button===1` 또는 `button===0 && space.current`일 때 pan 제스처 시작, Space는 SVG `onKeyDown`(포커스 시에만 수신)에서 세팅 | editor-canvas.tsx:86-95, e2e workspace.spec.ts:61-67(middle/space 두 방식 모두 pan 이동량 assert) | PASS |
| REQ-SNAP-001, DES-SNAP-001/002 | Object/Grid/Off 3단계, 기본 Object, Preview 제공 | USER §5, DESIGN §8 | `project.ts`, `snap.ts`, `editor-canvas.tsx` | 기본 `snapMode:"object"`, `resolveSnap`이 3모드 분기, `snap-preview` `<g>`로 원+십자 Preview 렌더링 | project.ts:33, snap.ts:23-31, editor-canvas.tsx:114, e2e workspace.spec.ts:94-111(Object/Grid/Off 전환하며 preview·확정 좌표 일치 확인) | PASS |
| VAL-FE02-002 | 통합 좌표 변환 함수 재사용 | DnD·객체 이동·hit test가 같은 client↔viewport↔world 변환을 사용, bbox 비례 계산 금지 | 02-WORKSPACE.md §입력 동작 계약 | `workspace-input.ts`, `editor-canvas.tsx` | `clientToViewport`(getScreenCTM 역행렬)+`screenToDocument`를 `pointAt`/`viewportAt` 하나로 캡슐화, pointerdown/move/up, wheel, dragover/drop 전부가 동일 함수 경유. bbox 비례 계산 코드 없음 | workspace-input.ts:14-19, editor-canvas.tsx:48-50, 89-100, e2e workspace.spec.ts:77-112(패널 접힘·줌·팬 후에도 CTM 기반 좌표 일치) | PASS |
| VAL-FE02-003 | 포인터 중심 줌, hit 거리와 문서 좌표 구분 | 포인터 위치를 중심으로 줌, 화면 hit 거리와 문서 좌표를 구분 | 02-WORKSPACE.md §입력 동작 계약 | `coordinates.ts`(zoomAt), `snap.ts`(nearestCandidate) | `zoomAt`이 anchor의 문서 좌표를 보존, `nearestCandidate`가 `pixelsPerMm(=scale)`로 mm 거리를 CSS px로 환산해 10px 임계값 적용(문서 mm ≠ 화면 px 명확히 구분) | coordinates.ts:15-20, snap.ts:19-21, workspace.test.ts:9-19(줌 앵커 고정), workspace.test.ts:26-37(거리 임계값 단위 테스트) | PASS |
| VAL-FE02-004 | 보호 입력 대상 | 속성 입력/대화상자에서 Space·Wheel이 편집기를 오작동시키지 않음 | 02-WORKSPACE.md §입력 동작 계약 | `workspace-input.ts`(isProtectedTarget) | input/textarea/select/button/contenteditable/dialog/[role=dialog] 및 하위 요소를 감지해 wheel·키 핸들러가 조기 반환 | workspace-input.ts:4-6, workspace.test.ts:56-62(6종 보호 대상 단위 테스트), e2e workspace.spec.ts:69-71(System 대화상자 내 wheel/Space/Escape가 문서 CTM을 바꾸지 않음 확인) | PASS |
| VAL-FE02-005 | Snap Off에서도 endpoint 제약 유지 | 위치 Snap이 Off여도 배선 endpoint 유효 대상 제한은 유지 | 02-WORKSPACE.md §입력 동작 계약(굵게 명시) | `snap.ts`(connectionCandidate) | `connectionCandidate`는 `snapMode` 인자를 받지 않고 항상 `endpoint` 보유 후보만 반환, 위치 Snap과 독립 | snap.ts:33-35, workspace.test.ts:38-53("keeps Off endpoints constrained" — snapMode off 상태에서도 `connectionCandidate`가 유효 terminal 반환·무효 지점은 null) | PASS |
| VAL-FE02-006 | 용지 변경 시 객체 보존 | 용지 축소/방향 변경이 객체를 삭제·이동하지 않음, 경계 밖 객체 접근 가능 | 02-WORKSPACE.md §완료 기준 1 | `editor-shell.tsx`(configure), `editor-canvas.tsx` | `configure`는 `project.canvas` 필드만 `Object.assign`, `components/objects` 배열 미접촉. 캔버스 렌더링에 경계 클리핑 없음(문서 좌표 그대로 렌더) | editor-shell.tsx:152-154, editor-canvas.tsx:105-109, e2e workspace.spec.ts:29-33(4용지 전환 후 컴포넌트 transform 불변 assert) | PASS |
| VAL-FE02-007 | 접근성: 이름·활성 상태·포커스 | 탭·버튼의 접근 가능한 이름, 활성 상태, 키보드 포커스 | 02-WORKSPACE.md §구현 작업 본문 | `editor-shell.tsx`, `globals.css` | 탭 `role=tab`+`aria-selected`, 모드/시트 `aria-pressed`, Sidebar 토글 `aria-label`+`aria-expanded`, 전역 `:focus-visible` 아웃라인 | editor-shell.tsx:180, 182, 194, globals.css:328, e2e 전체 시나리오가 `getByRole`/`getByLabel` 접근성 셀렉터로만 동작(간접 검증) | PASS |
| REQ-OVERLAP-001 | 객체·부품 겹침 허용 | 겹침을 막지 않음 | USER §5 | 도메인 전체(`editor.ts`,`objects.ts`) | 이동/배치 명령에 충돌·겹침 검사 코드 없음(`grep overlap/겹침/collision` 결과 없음) — 구조적으로 겹침 차단 없음 | 코드 부재로 확인(적극적 허용 테스트는 없음) | PASS(판정 보류 요소 있음: 명시적 허용 테스트 부재, 차단 로직 부재로 간접 확인) |
| 성능(common.md §모든 계층 원칙) | 100부품·300배선 반응성 | 일반 프로젝트에서 이동·선택이 끊기지 않아야 함 | common.md | (측정 없음) | HANDOFF가 미실행으로 명시, FE-02 완료 기준에도 성능 측정 항목 없음(캔버스 실사용 규모 검증은 FE-06 통합 검증 범위) | 02-WORKSPACE-HANDOFF.md:70("미실행/후속: ... 100부품·300배선 성능") | WARNING(판정 보류(증거 부족), FE-06에서 실측 필요) |

### 실행한 검사

| 명령 또는 수동 검사 | 리비전 | 결과 | 확인 범위 / 실패 분류 / 미실행 사유 |
|---|---|---|---|
| `git status --porcelain` (사전 확인, 검사 시작 전/명령 실행 후 재확인) | 712d656 | 통과 | `01-FOUNDATION-VERIFY.md` 1건만 미추적(지시에 따라 무시), 그 외 변경 없음 확인 |
| `npm run lint` | 712d656 | 통과 | `eslint .`, 출력 없이 정상 종료 |
| `npm run typecheck` | 712d656 | 통과 | `tsc --noEmit`, 출력 없이 정상 종료 |
| `npm test` | 712d656 | 통과 | vitest, 8 파일 53개 테스트 전부 통과(workspace.test.ts 5개 포함) |
| `npm run build` | 712d656 | 통과 | `next build --webpack`, TypeScript 검사 포함 프로덕션 빌드 성공, 정적 페이지 3개(`/`, `/_not-found`) 생성 |
| `PLAYWRIGHT_PORT=3103 npm run test:e2e -- --reporter=line` | 712d656 | 통과 | Chromium, 21개 시나리오 전부 통과(workspace.spec.ts 5개 포함, project-lifecycle/objects-images 회귀 없음). Playwright Chromium은 사전 설치 확인 후 재사용(재설치 불필요) |

미실행 항목 없음(지정된 lint/typecheck/test/build/test:e2e 전부 실행 완료). 성능(100부품/300배선) 측정은 스킬의 실행 명령 목록에 없고 FE-02 완료 기준에도 없으므로 이번 회차에서 실행하지 않았다(위 체크리스트에 WARNING으로 기록).

### HANDOFF 대조와 추적성

| ID | 가이드/상위 문서 | HANDOFF 주장·결정 | 실제 구현·테스트 | 차이 또는 연결 누락 |
|---|---|---|---|---|
| 전체 | 02-WORKSPACE-HANDOFF.md 헤더 | "`02-WORKSPACE`는 main의 FE-01 병합 커밋 `74a86e9`에서 시작했다" | 현재 `git log --all`에는 `74a86e9` 커밋이 존재하지 않음(전체 이력 7개 커밋 중 `3202e15`/`580bf0f`만 소스 변경 커밋이며 메시지가 "1"/"reset") | 저장소가 이후 시점에 스쿼시·리셋된 것으로 보여 HANDOFF가 주장한 병합 기준 커밋을 실제 git 이력으로 재현할 수 없음. FE-02 코드 자체의 존재·동작은 정적/동적 검사로 별도 확인했으므로 이 항목이 PASS 판정에는 영향 없으나, 문서-이력 불일치로 기록 |
| VAL-FE02-005 | 02-WORKSPACE-HANDOFF.md §Snap·확정 명령 | "Off와 중간 분기 후보의 정책은 FE-04 전에 확정한다" | 02-WORKSPACE.md 본문에도 동일하게 FE-04 착수 전 결정 사항으로 명시 | 없음(가이드·HANDOFF 일치, 현재 워크플로우 범위 밖으로 적절히 이연) |
| REQ-BOTTOM-001 | 02-WORKSPACE-HANDOFF.md §UI 토큰 | "BOM/Table/Code/PDF/PNG는 준비 중 및 비활성...저장·출력 완료를 가장하는 동작 없음" | 코드에서 해당 버튼 전부 `disabled`+`title="...에서 구현 예정"` | 없음 |
| 검증 수치 | 02-WORKSPACE-HANDOFF.md §검증·시각 확인 | "`npm test`: 6파일 39개", "e2e 최종 13개 전체 통과" | 현재 `npm test`는 8파일 53개, e2e는 21개(FE-03 확장분 포함) | HANDOFF 작성 시점(FE-02 종료) 이후 FE-03이 진행되며 테스트가 자연 증가. 이번 회차는 현재 전체 스위트 통과 여부로 독립 판정했으므로 PASS 판정에 영향 없음(01-FOUNDATION-VERIFY.md에서도 동일 패턴 기록됨) |

### 집계와 후속 조치

- 전체: 21 / PASS: 20 / FAIL: 0 / WARNING: 1 / N/A: 0
- 이전 회차 대비: 첫 회차
- FAIL: 없음
- WARNING: 성능(100부품/300배선 반응성) — FE-02 완료 기준에 측정 항목 없음, HANDOFF도 미실행으로 명시. FE-06 통합 검증에서 실측 필요
- N/A: 없음(페이지 삭제/이름 변경 UI, 배선 중간 분기 Off 정책 등은 가이드가 "추가하면/FE-04 전 결정"으로 조건부·후속 범위임을 스스로 명시했고 HANDOFF도 미구현을 명확히 고지했으므로 별도 미달성 항목으로 잡지 않음)
- PASS(실연동 미확인): 없음(FE-02는 로컬 도메인·화면 로직이 전부이며 외부 서비스 연동 없음. 공용부품 목록은 샘플 어댑터이나 이는 FE-02 완료 기준의 범위가 아니라 FE-03/05에서 다룸)
- 확인하지 못한 범위: 성능(100부품/300배선) 실측 — 위 WARNING 참고
- HANDOFF에 없는 변경 파일: 없음(merge-base=HEAD, 비교 가능한 diff 없음. 위 추적성 표의 커밋 이력 불일치 참고)
- 추가 구현·문서 충돌: HANDOFF가 주장한 FE-01 병합 기준 커밋(`74a86e9`)이 현재 git 이력에 없음(위 추적성 표 참고). FE-02 자체 구현·검증 결과에는 영향 없으나 저장소 이력 관리 측면에서 별도 확인 필요.
