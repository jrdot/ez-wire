# FE01 검증 보고서

- 기준 가이드: docs/workflow/frontend/01-FOUNDATION.md
- HANDOFF: docs/workflow/frontend/01-FOUNDATION-HANDOFF.md
- 최신 판정: 완료 — 1회차, 2026-09-25

<!-- 이 아래의 회차 섹션은 재검증 때마다 파일 끝에 추가한다. 이전 회차는 수정하지 않는다. -->

## 검사 이력 1회차 — 2026-09-25

- 작업 브랜치 / HEAD / merge-base: main / 712d656b5ca4ddc8ac37662d91b881d89b7a1d18 / 712d656b5ca4ddc8ac37662d91b881d89b7a1d18 (HEAD가 main과 동일, merge-base=HEAD → 비교할 별도 diff 없음)
- 작업 트리: 깨끗함 (`git status --porcelain` 출력 없음, 검사 시작 전 확인)
- 검사 범위: frontend FE-01 (계약과 편집기 기반) — 좌표/객체/배선/명령 계약, 샘플 문서, 타입/검증 함수, 파일 포맷 경계 검증
- 상위 기준 문서: docs/workflow/common.md, docs/workflow/frontend/common.md, docs/core/DESIGN_SPEC.md, docs/core/USER_REQUIREMENTS_SPEC.md
- 전체 판정: 완료 — 범위 내 필수 항목 전부 PASS, FAIL 없음. WARNING 2건은 FE-01 범위 밖(성능·영속성)으로 완료 선언에 영향 없음.

### 체크리스트 및 검사 결과

| ID | 항목 | 요구 내용 | 기준 | 검사 대상 | 결과 | 근거 | 판정 |
|---|---|---|---|---|---|---|---|
| REQ-CANVAS-SIZE-001 / DES-CANVAS-001 | 도면 규격 | A4/A3 가로·세로 지원, 기본 A4 가로 | USER_REQUIREMENTS §5, DESIGN §4 | `src/domain/coordinates.ts`, `src/domain/project.ts` | `paperSize()`가 4개 규격 물리 치수 반환, `createEmptyProject`가 A4-landscape 기본값 사용 | coordinates.ts:2-5, project.ts:32-34, coordinates.test.ts:3 (`npm test` 통과) | PASS |
| REQ-CANVAS-BG-001 / DES-CANVAS-002 | 배경 기본값 | Canvas 기본 배경 흰색 | 상동 | `src/domain/project.ts` | `background: "#ffffff"` | project.ts:33, validation.test.ts:8 | PASS |
| REQ-SNAP-001 / DES-SNAP-001 | Snap 기본값 | 기본값 Object Snap, 3단계 지원 | 상동 | `src/domain/project.ts`, `validation.ts` | `snapMode: "object"` 기본, `one("object","grid","off")`로 3단계만 허용 | project.ts:33, validation.ts:39 | PASS |
| REQ-PROJECT-001 / DES-PAGE-001 | 최대 3페이지 | 프로젝트당 작업 페이지 최대 3개 | 상동 | `editor.ts`, `validation.ts` | `MAX_SHEETS=3`, `addSheet`가 초과 시 예외, `assertProject`가 4페이지 이상 거부, UI에서 3/3 시 추가 버튼 비활성 | editor.ts:39, validation.ts:51, editor-shell.tsx:195, editor.test.ts:8 (`3페이지` 예외 검증), e2e project-lifecycle.spec.ts:69-85 | PASS |
| REQ-PAGE-NAME-001 / DES-PAGE-002 | 페이지 기본 명칭 | `EZ-001` 형식, 3자리 zero-padding 증가 | 상동 | `project.ts` | `createSheet`가 `EZ-${padStart(3,"0")}` 생성, `nextSheetNumber` 증가 | project.ts:28-29, editor.test.ts:20 (삭제 후 `EZ-003` 확인) | PASS |
| REQ-OBJECT-001/ID-001, DES-OBJECT-001/ID-001 | 객체 타입·고유 ID | Image/Line/Point/Text 각 고유 ID | 상동 | `project.ts`, `validation.ts` | `DrawingObject` 4종 판별 유니온, 전 프로젝트 범위 ID 중복 검사(`register`) | project.ts:11-16, validation.ts:55-56,59, validation.test.ts (duplicate 케이스) | PASS |
| REQ-GROUP-001 / DES-OBJECT-ID-001 | 그룹 ID 규칙 | 그룹은 신규 ID, 자식 ID 유지, 순환·중복 소유 금지 | 상동 | `editor.ts`(groupObjects), `validation.ts` | `groupObjects`가 `createId()`로 그룹 ID 발급하고 `childIds` 그대로 보존, `assertProject`가 그룹 소유 중복·순환 검사 | editor.ts:41, validation.ts:66-71, editor.test.ts:10, validation.test.ts(cycle/ownership 케이스) | PASS |
| DES-OBJECT-002, DES-TERMINAL-001 | 부품·단자 구조 | 부품은 여러 Object 조합, Terminal은 연결용 Point | DESIGN §6-7 | `project.ts` | `ComponentInstance.objects/terminals`, `Terminal` 타입이 role:"terminal" Point로 분리 | project.ts:17-18 | PASS |
| REQ-WIRE-ENDPOINT-001 / DES-WIRE-ENDPOINT-001 | Wire endpoint 제약 | 시작/끝점은 Terminal 또는 Junction만 | USER §13, DESIGN §11 | `project.ts`, `validation.ts` | `WireEndpoint` 유니온이 두 종류만 허용, `resolveEndpoint`/`assertProject`가 실제 참조 존재·connectable 검증 | project.ts:21, validation.ts:34-37,44-46,73 | PASS |
| REQ-TERMINAL-002 | 단자 다중 연결 | 하나의 단자에 복수 배선 연결 가능 | USER §8 | `editor.ts` | `connectedWires`가 endpoint에서 파생, 개수 제한 없음 | editor.ts:42 | PASS |
| REQ-WIRE-FREE-001 / DES-WIRE-TYPE-001(Free) | Free 배선 Bend 금지 | Free Wire는 중간 절곡점 없음 | 상동 | `validation.ts` | `w.type === "free" && w.bends.length`이면 거부 | validation.ts:74, validation.test.ts(free-bend 케이스) | PASS |
| VAL-FE01-001 | 좌표 계약 | 문서 mm/화면 px 분리, 로컬 (0,0) 중심 scale→회전(시계방향)→이동, 줌/DPI 독립 | 01-FOUNDATION.md §구현순서 2, HANDOFF 구현 선택 | `coordinates.ts` | `localToDocument`/`documentToLocal` 상호 역변환 일치, `documentToScreen`/`screenToDocument`가 DPI·zoom과 무관하게 왕복 | coordinates.ts:6-12, coordinates.test.ts:4-5(왕복 검증) | PASS |
| VAL-FE01-002 | SVG CTM 역변환 | 실제 letterbox/화면 오프셋을 `getScreenCTM()` 역행렬로 처리 | HANDOFF 구현 선택 | `workspace-input.ts` | `clientToViewport`가 `svg.getScreenCTM().inverse()` 사용 | workspace-input.ts:14-19, e2e workspace.spec.ts:77(CTM 관련 시나리오, `npm run test:e2e` 통과) | PASS |
| VAL-FE01-003 | ID 생성·가져오기 정책 | UUID v4 발급, 가져온 ID는 비어있지 않은 문자열 허용, 프로젝트 전체 중복 금지 | HANDOFF 구현 선택 | `id.ts`, `validation.ts` | `createId`가 `randomUUID` 우선, fallback으로 v4 형식 생성; `id` 체크는 비어있지 않은 문자열만 요구(포맷 불문); 전역 `register()`로 중복 차단 | id.ts:1-13, validation.ts:6,55-56, id.test.ts, e2e project-lifecycle.spec.ts:3-16(`randomUUID` 없이도 생성) | PASS |
| VAL-FE01-004 | 그룹 참조 범위 | 그룹은 같은 페이지 객체/부품/그룹만 참조, 자식 좌표는 페이지 좌표 유지 | HANDOFF 구현 선택 | `validation.ts` | `children` 집합을 같은 시트 범위로 한정해 그룹 소유 검사 | validation.ts:65-71 | PASS |
| VAL-FE01-005 | 페이지 간 endpoint 금지 | 배선 endpoint는 같은 페이지 내에서만 유효 | 01-FOUNDATION.md §구현순서 4 | `validation.ts` | `resolveEndpoint`가 `sheet` 인자로 한정된 배열만 탐색(다른 시트 참조 불가) | validation.ts:44-46, validation.test.ts(cross-page 케이스) | PASS |
| REQ-SHORTCUT-001(Ctrl+Z/Y) + 사용자 확인 | Undo/Redo, 동작당 1회 | 확정 편집을 동작당 Undo 1회로 되돌림, 새 명령 시 redo 폐기 | USER §10, HANDOFF 사용자 확인 2026-09-23 | `editor.ts` | `execute`가 새 명령마다 `future: []`로 redo 폐기, `past`에 직전 스냅샷 1개 push; revision이 confirm마다만 단조 증가 | editor.ts:17-29,37-38, editor.test.ts:7("실행 취소" revision 1→2→3), e2e project-lifecycle.spec.ts:96-120(드래그 1커밋·Undo·Esc취소) | PASS |
| VAL-FE01-006 | Preview/명령 분리 | pointerup 1회 확정, Esc/pointercancel/lost capture/blur/페이지·모드 전환 시 취소, 취소는 문서·이력 불변 | HANDOFF 구현 선택 | `editor-canvas.tsx`, `editor.ts` | `finish()`가 실제 변경 있을 때만 1회 `onCommand` 호출; `onPointerCancel/onLostPointerCapture/onBlur`, Esc 키가 모두 `cancel()` 호출; `switchSheet`/`switchMode`가 `preview:null` 설정 | editor-canvas.tsx:27,83,86-96, editor.ts:15-16, e2e project-lifecycle.spec.ts:96-120(Esc 후 revision 불변) | PASS |
| VAL-FE01-007 | 비활성 페이지·메타데이터 보호 | 비활성 페이지 수정 거부, 문서 메타데이터 직접 변경 거부 | 01-FOUNDATION.md §구현순서 3,5 | `editor.ts` | `execute`가 활성 시트 외 변경 시 예외, id/revision/createdAt/updatedAt 직접 변경 시 예외 | editor.ts:18,22-25, editor.test.ts:22-25("비활성" 예외) | PASS |
| VAL-FE01-008(사용자 확인) | 페이지 번호 재사용 금지 | 확정 삭제는 `nextSheetNumber`를 낮추지 않음(취소된 생성은 재사용 가능) | HANDOFF 사용자 확인 2026-09-23 | `editor.ts` | `deleteActiveSheet`가 `nextSheetNumber` 불변, `addSheet`만 증가; Undo는 스냅샷 복원이라 취소된 생성 번호는 자연히 재사용됨 | editor.ts:39,43-45, editor.test.ts:15-21(삭제 후 재추가 시 `EZ-003`, 재사용 없음 확인) | PASS |
| VAL-FE01-009(사용자 확인) | 연결 부품 삭제 거부 | Wire로 연결된 부품 삭제 시 거부 | HANDOFF 사용자 확인 2026-09-23 | `objects.ts`(deleteSelection) | 연결된 컴포넌트 포함 시 "연결된 부품은 삭제할 수 없습니다" 예외 | objects.ts:38-41, editor.test.ts:11("rejects dangling deletion") | PASS |
| VAL-FE01-010 | 미지원 필드/버전 거부 | v2 파일 알 수 없는 버전·필드·깨진 중첩·중복ID·없는 endpoint·잘못된 수치·4페이지 이상을 경로 포함 오류로 거부, 원본 불변 | 01-FOUNDATION.md §현재 파일 계약 검증 | `project-file.ts`, `validation.ts` | `parseProjectFile`가 버전/필드 불일치 시 명시적 오류, `assertProject`의 `shape()`가 미지원 키를 경로 포함 오류로 거부; 13종 입력 오류 패턴 파라미터화 테스트로 검증 | project-file.ts:9-18, validation.ts:14-19, validation.test.ts:10 이하(duplicate/endpoint/cross-page/size/nan/free-bend/cycle/ownership/asset/part/nested/unknown/four-pages 13종 전부 원본 비파괴 거부), e2e project-lifecycle.spec.ts:87-93 | PASS |
| VAL-FE01-011 | 샘플 문서 왕복 | `foundation-sample.wireproj`가 4객체 타입·그룹·2부품/단자·Junction·2Wire를 포함하고 파서 왕복 시 무손실 | HANDOFF §FE-02 샘플 | `public/foundation-sample.wireproj`, `validation.test.ts` | 파일 내용이 image/line/point/text, group(2 child), 2 component/terminal, 1 junction+2 wire 포함; `parseProjectFile(serializeProject(p))` 동등성 테스트 통과 | foundation-sample.wireproj:1-219, validation.test.ts:9 | PASS |
| VAL-FE01-012 | 신규 프로젝트/기존 흐름 회귀 | 새 프로젝트 생성·파일 열기·닫기 흐름 유지 | 01-FOUNDATION.md §산출물과 완료 기준 | `project-launcher.tsx`, e2e | 새 프로젝트/파일 열기/닫기 e2e 4종 통과 | e2e project-lifecycle.spec.ts:3-57 | PASS |
| DES-TERMINAL-003 / REQ-TERMINAL-003 | 배선 모드 Terminal 고정점 | Wire 모드에서 Terminal 직접 드래그 이동 불가 | DESIGN §7, USER §8 | (FE-04 범위) | HANDOFF가 Wire 생성·편집 정책을 FE-04로 명시 이연, 현재 셸은 Wire 모드 도구 자체가 "준비 중"으로 비활성 표시되어 있어 오작동 여지 없음 | 01-FOUNDATION-HANDOFF.md:63("Wire 경로... FE-04에서 결정"), editor-shell.tsx:169(`deferred([...], "FE-04")`) | N/A (FE-01 범위 밖, FE-04로 명시 이연) |
| 성능(공통 원칙) | 100부품/300배선 반응성 | 일반 프로젝트 부품 100·배선 300에서 기본 이동/선택 유지 | common.md §모든 계층 원칙 | (측정 없음) | HANDOFF가 미실행으로 명시, FE-01 완료 기준에 성능 측정 항목 없음(캔버스 실사용은 FE-02/03) | 01-FOUNDATION-HANDOFF.md:78 | WARNING (판정 보류(증거 부족), FE-06 통합 검증에서 측정 필요) |
| 이력 영속성 | 새로고침/재실행 후 복구 | (참고: Phase 2 완료 기준, FE-01 범위 아님) | common.md §단계와 인계 기준 | `editor.ts`(메모리 전용) | Undo/Redo 이력이 메모리(state)에만 존재, 새로고침 시 소실 — HANDOFF가 명시적으로 고지, FE-01 완료 기준에 영속성 요구 없음 | 01-FOUNDATION-HANDOFF.md:78("닫기/새로고침 시 보존하지 않는다") | WARNING (FE-01 범위 아님, FE-02 이후 저장/자동복구 단계에서 재확인 필요) |

### 실행한 검사

| 명령 또는 수동 검사 | 리비전 | 결과 | 확인 범위 / 실패 분류 / 미실행 사유 |
|---|---|---|---|
| `git status --porcelain` (사전 확인) | 712d656 | 통과(빈 출력) | 작업 트리 깨끗함 확인 |
| `npm ci` | 712d656 | 통과 | 의존성 미설치 상태(`node_modules` 없음) 확인 후 설치. 439 packages 추가, 취약점 0건 |
| `npm run lint` | 712d656 | 통과 | `eslint .`, 출력 없이 정상 종료 |
| `npm run typecheck` | 712d656 | 통과 | `tsc --noEmit`, 출력 없이 정상 종료 |
| `npm test` | 712d656 | 통과 | vitest, 8 파일 53 테스트 전부 통과(FE-01 관련: coordinates/editor/id/project/validation 5파일 + FE-02/03 확장분 objects/workspace/image) |
| `npm run build` | 712d656 | 통과 | `next build --webpack`, TypeScript 검사 포함 프로덕션 빌드 성공, 정적 페이지 2개 생성 |
| `npx playwright install chromium` | 712d656 | 통과(설치) | 브라우저 미설치 상태 확인 후 설치(로컬 환경에 `ms-playwright` 캐시 없었음) |
| `PLAYWRIGHT_PORT=3101 npm run test:e2e` | 712d656 | 통과 | Chromium, 21개 시나리오 전부 통과(FE-01 관련 project-lifecycle.spec.ts 8개 포함, FE-02/03의 workspace/objects-images 13개도 회귀 없음). 전용 포트 사용으로 기존 3000번 포트 프로세스와 충돌 없음 |

미실행 항목 없음(모든 지정 명령 실행 완료). 성능(100부품/300배선) 측정은 스킬의 실행 명령 목록에 없고 HANDOFF도 미실행으로 고지했으므로 이번 회차에서도 별도로 실행하지 않았다(WARNING으로 기록).

### HANDOFF 대조와 추적성

| ID | 가이드/상위 문서 | HANDOFF 주장·결정 | 실제 구현·테스트 | 차이 또는 연결 누락 |
|---|---|---|---|---|
| VAL-FE01-003 | 01-FOUNDATION.md §구현순서 1 | "기존 SVG 유지. ID는 UUID v4 생성, 가져온 ID는 비어 있지 않은 문자열 허용" | `id.ts`/`validation.ts` 코드와 일치, id.test.ts로 v4 형식·유일성 검증 | 없음 |
| VAL-FE01-006 | 01-FOUNDATION.md §상태와 변경 경계 | "pointerup에 한 번 확정하고 Esc/pointercancel/lost capture/blur/페이지 전환은 취소" | `editor-canvas.tsx`의 이벤트 핸들러 전부 확인됨 | 없음 |
| VAL-FE01-010 | 01-FOUNDATION.md §현재 파일 계약 검증 | "미지원 필드는 조용히 버리지 않고 경로 포함 오류 반환" | `validation.ts:17`에서 `${p}.${key} (지원하지 않는 필드; 원본 보존)` 형태로 경로 포함 | 없음 |
| VAL-FE01-011 | HANDOFF §FE-02 샘플과 시작 조건 | "현재 셸은 부품/단자만 렌더링한다. 범용 객체는 FE-03, 배선 렌더링은 FE-04에서 추가" | 확인 시점(2026-09-25)의 `editor-canvas.tsx`는 이미 `sheet.objects`도 `ObjectShape`로 렌더링 중(FE-03에서 추가됨) | HANDOFF 시점 이후 후속 워크플로우(FE-03)가 실제로 진행되어 설명이 최신 상태와 다름. FE-01 자체의 결함은 아니며 FE-03 검증에서 별도 확인 필요 |
| - | 01-FOUNDATION-HANDOFF.md §검증 기록 | "`npm test` 5개 파일, 34개 테스트" | 현재 `npm test`는 8개 파일 53개 테스트(FE-02/03 확장 포함), FE-01 해당분(coordinates/editor/id/project/validation)만도 34개보다 늘었을 가능성 있으나 개별 파일 단위 재실행으로 구분 집계하지 않음 | 과거 수치와 현재 수치는 후속 워크플로우로 인한 자연 증가로 판단. 이번 회차는 전체 스위트 통과 여부로 독립 판정했으므로 PASS 판정에 영향 없음 |

### 집계와 후속 조치

- 전체: 24 / PASS: 22 / FAIL: 0 / WARNING: 2 / N/A: 1(N/A 1건은 위 표의 DES-TERMINAL-003 행, WARNING 2건은 성능·이력 영속성 행에 포함되어 전체 24건 중 집계됨)
- 이전 회차 대비: 첫 회차
- FAIL: 없음
- WARNING: 성능(100부품/300배선 반응성) — FE-01 범위 밖이라 이번에 측정하지 않음, FE-06 통합 검증 시 실측 필요 / 이력 영속성(새로고침 시 Undo 이력 소실) — HANDOFF가 이미 고지한 FE-01 범위 밖 사항, FE-02 이후 저장·자동복구 설계에서 재확인 필요
- N/A: DES-TERMINAL-003/REQ-TERMINAL-003(Wire 모드 Terminal 고정) — FE-04로 명시 이연, 현재 Wire 도구가 비활성 상태라 위반 소지 없음
- PASS(실연동 미확인): 없음(FE-01은 로컬 도메인 로직·파일 포맷 계약이 전부이며 외부 연동 없음)
- 확인하지 못한 범위: 없음(지정된 lint/typecheck/test/build/test:e2e 전부 실행 완료)
- HANDOFF에 없는 변경 파일: 없음(merge-base=HEAD, diff 대상 없음. 현재 저장소는 FE-01~FE-03이 이미 main에 병합된 상태이며, FE-01 관련 파일은 HANDOFF가 명시한 목록과 일치)
- 추가 구현·문서 충돌: HANDOFF의 "현재 셸은 부품/단자만 렌더링" 서술이 FE-03 반영 이후의 현재 코드와는 다름(위 추적성 표 참고). FE-01의 계약 자체에는 영향 없으나 문서 최신화가 필요한 부분으로 기록.
