# FE03 검증 보고서

- 기준 가이드: docs/workflow/frontend/03-OBJECTS-IMAGES.md
- HANDOFF: docs/workflow/frontend/03-OBJECTS-IMAGES-HANDOFF.md
- 최신 판정: 미완료(배경 제거 대기) — 1회차, 2026-09-25

<!-- 이 아래의 회차 섹션은 재검증 때마다 파일 끝에 추가한다. 이전 회차는 수정하지 않는다. -->

## 검사 이력 1회차 — 2026-09-25

- 작업 브랜치 / HEAD / merge-base: main / 712d656b5ca4ddc8ac37662d91b881d89b7a1d18 / 712d656b5ca4ddc8ac37662d91b881d89b7a1d18 (HEAD가 이미 main, merge-base와 동일)
- 작업 트리: 깨끗함 (사전 배치 작업 01/02 검증 산출물 `01-FOUNDATION-VERIFY.md`, `02-WORKSPACE-VERIFY.md`만 untracked, 지시에 따라 정상으로 간주하고 진행함. 그 외 변경/미추적 파일 없음)
- 검사 범위: FE-03 객체·부품·이미지 편집 (docs/workflow/frontend/03-OBJECTS-IMAGES.md, HANDOFF 동일 폴더)
- 상위 기준 문서: docs/workflow/common.md, docs/workflow/frontend/common.md, docs/core/USER_REQUIREMENTS_SPEC.md(§7~11), docs/core/DESIGN_SPEC.md(§6~9, §17), docs/workflow/frontend/07-TRACEABILITY.md
- 전체 판정: 미완료 — REQ-IMAGE-001/DES-IMAGE-001의 필수 최소 기능인 "배경 제거"가 전혀 구현되지 않았고(버튼 비활성만 존재), 이는 사용자 결정(외부 AI 연동 확정 대기)으로 가이드·HANDOFF·common.md·추적표에 일관되게 기록되어 있어 WARNING으로 판정. 그 외 범위 내 항목은 실행 검사(lint/typecheck/test/build/e2e 전부 통과, 53 unit + 21 e2e)와 코드 대조로 PASS.

### 체크리스트 및 검사 결과

| ID | 항목 | 요구 내용 | 기준 | 검사 대상 | 결과 | 근거 | 판정 |
|---|---|---|---|---|---|---|---|
| REQ-OBJECT-001 / DES-OBJECT-001,002 | 기본 4객체·복합 부품 | Image/Line/Point/Text 렌더링, Line/Point는 Wire/Terminal의 별칭이 아님, 부품은 객체 조합 | 가이드 §객체와 선택-1, 완료기준 | `src/domain/objects.ts`(newObject), `src/features/editor/object-shape.tsx`, `editor-canvas.tsx`, `editor-shell.tsx`(addObject/좌측 생성 버튼) | 4타입 모두 독립 렌더링 확인. 일반 Line은 도형, 일반 Point는 표시 객체이며 Wire/Terminal 생성 없음(코드·HANDOFF 서술 일치). fixture(`public/objects-images-fixture.wireproj`)에 독립 image/text/point/line 4종 모두 존재(직접 파싱 확인) | fixture 파싱 결과, `objects.test.ts` 전체 통과 | PASS |
| REQ-OBJECT-ID-001 / REQ-GROUP-001 / DES-OBJECT-ID-001 | 고유 ID·그룹 ID 발급, 자식 ID 유지 | 그룹 생성 시 새 그룹 ID, 자식 ID 불변 | 가이드 §객체와 선택-4 | `src/domain/objects.ts`(rootId/expandSelection), `src/domain/editor.ts`(groupObjects) | `objects.test.ts` "groups preserve child IDs and rotate all descendants..." 테스트로 그룹 ID 신규 발급·자식 ID 불변·공동 회전·Undo 1회 확인 | `npm test` 통과(53/53), 해당 케이스 통과 | PASS |
| REQ-SELECT-001 / DES-SELECT-001 | Click 단일, Ctrl/Shift+Click 토글, 겹침 허용, z-order | 클릭 선택 정책 | 가이드 §객체와 선택-2 | `editor-canvas.tsx` begin()/onPointerDown | 코드상 단일/토글 분기 확인. z-order는 `sheet.components` 배열 후 `sheet.objects` 배열 순서로 렌더링(뒤 배열이 위) — HANDOFF 서술과 일치. e2e "Ctrl/Shift toggle, ordinary click selects one..." 통과 | e2e 통과(21/21 중 포함) | PASS |
| VAL-FE03-004 | 이동/삭제/속성변경은 명령, 회전은 Drag 자유/Shift 45° 스냅, 공통 변환 함수 | 가이드 §객체와 선택-3, REQ-ROTATE-001/002 | `objects.ts`(rotationAngle, transformSelection, transformCommand) | `rotationAngle`이 단일 절대각·그룹은 변화량 스냅 구분 구현. e2e "rotation drag is free, Shift snaps, Escape restores..." 통과 | `objects.test.ts` "free angles retain precision and shift snaps to 45 degrees" 통과, e2e 통과 | PASS |
| REQ-SIDEBAR-R-001 / VAL-FE03-005 | 우측 속성 패널: 선택 타입별 표시, 미선택/단일/다중/그룹 구분, 혼합 상태 표시(제안) | 가이드 §객체와 선택-5 | `entity-properties.tsx`, `editor-shell.tsx`(inspector 영역) | 단일 객체/부품/다중/그룹 4상태 UI 분기 확인, 다중·그룹은 "속성 혼합" 문구와 일괄 이동/회전/배율 폼 제공 | 코드 읽음, e2e 그룹/다중 선택 텍스트 검증("다중 선택","그룹 선택") 통과 | PASS |
| REQ-SHORTCUT-001 | Ctrl+C/X/V, Ctrl+Z/Y, Delete, Ctrl+A, Esc, 입력창 보호, 내부/외부 붙여넣기 구분, 하위ID 재매핑, cut은 복사 확보 후 삭제, 1동작 1 Undo | 가이드 §단축키 | `editor-shell.tsx`(keyHandler/copyHandler/pasteHandler), `workspace-input.ts`(isProtectedTarget), `objects.ts`(copySelection/pasteSelection) | 모든 단축키 분기 및 isProtectedTarget 가드 확인. 마커 `application/x-ezwire-objects`, 이미지 파일 우선 처리, 일반 텍스트는 객체로 해석 안 함. `pasteSelection`이 소유 ID 전체 재매핑 | `objects.test.ts`(재매핑 케이스), e2e "four objects, multi selection, group, clipboard graph and shortcuts protect text input" 통과 | PASS |
| REQ-LIBRARY-001 | 개인 라이브러리 추가/목록/수정/DnD 배치 | 가이드 §라이브러리와 Terminal | `editor-shell.tsx`(partDraft, DnD), `part-editor.tsx` | 등록/목록/수정/DnD 전 과정 코드·e2e로 확인 | e2e "library registration, terminal editing, DnD and definition isolation" 통과 | PASS |
| REQ-PUBLIC-LIBRARY-001 | 공용부품 사전 등록 목록 읽기 어댑터, API 없으면 샘플 명시 | 가이드 §라이브러리와 Terminal | `src/adapters/library.ts`(samplePublicParts) | `status:'sample'`, 메시지 "샘플 목록 · 공용 서비스 미연동"으로 명시 표시. 실제 서비스 미연동 | 코드 확인, HANDOFF·common.md·추적표 일관 기록 | PASS. 주석: 실연동 미확인(mock/샘플 기준) |
| VAL-FE03-009 | 부품 정의/인스턴스 구분, 정의 수정은 이후 배치에만 반영, 사용자 편집이 원본 안바꿈 | 가이드 §라이브러리와 Terminal | `editor-shell.tsx`(saveImage: 정의는 project.parts만 갱신, 인스턴스 편집은 instanceId 경로로 컴포넌트만 갱신) | 두 경로 분리 확인, `objects.test.ts` "multiple pasted instances have unique terminals and independent definitions"에서 정의 변경이 이미 배치된 인스턴스 라벨에 영향 없음 검증 | 테스트 통과 | PASS |
| DES-TERMINAL-001,002,003 / REQ-TERMINAL-001,002,003 | Terminal ID/로컬XY/이름/표시/연결가능, 배선모드 직접 드래그 금지, 부품/단자 편집 UI에서 위치 수정 | 가이드 §라이브러리와 Terminal | `entity-properties.tsx`(TerminalFields), `editor-canvas.tsx`(Wire 모드 시 begin 무시 로직 부재이나 별도 hit-test) | TerminalFields로 이름/x/y/visible/connectable 수정 및 단자 추가/삭제 제공. Wire 모드에서 캔버스 직접 드래그 불가 e2e로 검증 | e2e "library registration..." 테스트에서 Wire 모드 단자 드래그 시도 후 transform 불변 확인 | PASS |
| VAL-FE03-011 | 부품 이동/회전/크기 변경 시 단자 월드 위치도 변환 | 가이드 §라이브러리와 Terminal 마지막 항목 | `objects.ts`(endpointPosition, localToDocument) | 회전·배율 적용 컴포넌트의 단자 월드 좌표 계산 검증 | `objects.test.ts` "transformed terminals use scale, rotation and translation..." 통과 | PASS |
| DES-IMAGE-002 / REQ-CLIPBOARD-001 | Paste 이미지가 부품 추가 흐름과 자동 연결 | 가이드 §이미지 입력과 편집 | `editor-shell.tsx`(pasteHandler → uploadImage) | 클립보드 이미지 파일 감지 시 이미지 초안으로 즉시 진입 | e2e "image paste starts draft, cancel and decode failure never mutate document" 통과 | PASS |
| REQ-IMAGE-001a / DES-IMAGE-001a | 이미지 자르기(Crop) | 가이드 §이미지 입력과 편집, 완료기준 | `src/domain/image-edit.ts`(editPartGeometry), `src/adapters/image.ts`(edit), `image-editor.tsx` | 실제 Canvas 기반 픽셀 크롭. 단자가 Crop 영역 밖이면 거부(숨김 단자 포함) | `objects.test.ts` image geometry 스위트, e2e "image upload, actual crop/resize pixels..."에서 실제 픽셀 데이터(getImageData)로 결과 좌표·색상 검증 | PASS |
| REQ-IMAGE-001b / DES-IMAGE-001b | 크기 조정(Resize) | 가이드 §이미지 입력과 편집, 완료기준 | 위와 동일 | Resize 시 단자 좌표 비율 재계산, 물리 mm 비율 유지 | 위 e2e에서 Resize 후 실측 픽셀 144×100, 단자 좌표(10,20) 정확히 검증 | PASS |
| REQ-IMAGE-001c / DES-IMAGE-001c | 배경 제거 | REQ-IMAGE-001 최소 기능 3종 중 하나(자르기/배경 제거/크기 조정), 가이드 완료기준 "mock이면 완료로 표시하지 않음" | `src/adapters/image.ts`(backgroundRemoval={status:'pending'}), `editor-shell.tsx`/`image-editor.tsx`(버튼 disabled) | 실제 배경 제거 기능 미구현. 버튼은 비활성 상태로만 존재하며 가짜 투명화·단색 대체 없음(요구사항이 금지한 행위도 하지 않음) | 코드 확인(`backgroundRemoval.status==='pending'`), `image.test.ts`에서 상태값만 확인 | WARNING — REQ-IMAGE-001 필수 최소 기능이 완전히 비어 있음. 단, 가이드 자체 "구현 결과" 절, HANDOFF, common.md FE-03 인계, 07-TRACEABILITY.md(REQ-IMAGE-001/DES-IMAGE-001 행, "서비스" 결정 묶음)에 "사용자 지시로 외부 AI 연동 확정까지 대기"로 일관되게 기록되어 있고 완료로 허위 주장하지 않음. 승인 주체가 이 워크플로우 문서 체계 내부 기록에 한정되어 외부 독립 승인 증거는 확인 못함(판정 보류 요소) |
| VAL-FE03-014 | 원본/편집결과 구분, 취소 시 고아 자산 미생성 | 가이드 §이미지 입력과 편집 | `image-editor.tsx`(apply/onCancel), `editor-shell.tsx`(saveImage) | Crop/Resize 적용은 로컬 draft에만 반영, 저장(등록) 시에만 커밋. 취소 시 project.assets에 반영 안 됨 | e2e "image paste starts draft, cancel..."에서 취소 후 revision 0 유지 확인 | PASS |
| VAL-FE03-015 | 임시 blob URL 미저장, 프로젝트 전환 중 이전 결과 삽입 금지 | 가이드 §이미지 입력과 편집 | `editor-shell.tsx`(publish 시 imageTask abort, unmount cleanup), `image.ts`(data URL 반환, blob URL 미사용) | 어댑터가 data URL만 반환(`canvas.toDataURL`), 지연 응답이 새 프로젝트에 삽입되지 않음 | e2e "pending image read cannot insert into a replacement project" 통과 | PASS |
| VAL-FE03-016 | MIME/디코딩 실패, 미지원 포맷(GIF/SVG), 클립보드 접근 실패, 취소, 비동기 중 재입력 처리 | 가이드 §이미지 입력과 편집 | `image.ts`(decode 검증), `image.test.ts` | PNG/JPEG/WebP만 허용, 12MiB/8192px/1600만픽셀 제한, AbortError 처리 | `image.test.ts` 통과, e2e 디코딩 실패 케이스("broken.png") 통과 | PASS |
| VAL-FE03-017 | 여러 인스턴스 단자 ID 비충돌, 이동/회전 후 단자 좌표 정확 | 가이드 완료기준 | `objects.ts`(pasteSelection ID 재매핑) | 붙여넣기 2회 후 전체 단자 ID 유일성 확인 | `objects.test.ts` "multiple pasted instances have unique terminals..." 통과 | PASS |
| VAL-FE03-018 | 도메인 테스트(ID 재매핑·그룹변환·회전스냅·이력) + UI/E2E(등록→배치→선택→편집→Undo/Redo, 폼 입력 중 단축키 보호) | 가이드 완료기준 | `objects.test.ts`, `image.test.ts`, `e2e/objects-images.spec.ts` | 전 항목 실제 실행 확인 | `npm test` 53/53, `npm run test:e2e` 21/21 (FE-03 관련 8개 포함) | PASS |

### 실행한 검사

| 명령 또는 수동 검사 | 리비전 | 결과 | 확인 범위 / 실패 분류 / 미실행 사유 |
|---|---|---|---|
| `npm run lint` | 712d656 | 통과 | eslint 출력 없음(경고/오류 0) |
| `npm run typecheck` | 712d656 | 통과 | tsc --noEmit 오류 없음 |
| `npm test` | 712d656 | 통과 | vitest 8파일 53개 전체 통과, HANDOFF 수치와 일치 |
| `npm run build` | 712d656 | 통과 | `next build --webpack` 성공, 정적 페이지 생성 완료(패키지 잠금파일 위치 경고만 존재, 빌드 실패 아님) |
| `PLAYWRIGHT_PORT=3103 npm run test:e2e -- --reporter=line` | 712d656 | 통과 | Chromium 21개 전체 통과(objects-images.spec.ts 8개, project-lifecycle.spec.ts 8개, workspace.spec.ts 5개), HANDOFF 수치와 일치. Firefox/WebKit/모바일 프로젝트는 `playwright.config.ts`에 정의되어 있지 않아 실행 대상이 아님(HANDOFF "미실행: Firefox/WebKit/모바일"과 일치, 구성 파일상 애초에 프로젝트 미정의) |

### HANDOFF 대조와 추적성

| ID | 가이드/상위 문서 | HANDOFF 주장·결정 | 실제 구현·테스트 | 차이 또는 연결 누락 |
|---|---|---|---|---|
| REQ-IMAGE-001c | 가이드 §이미지 입력과 편집, REQ-IMAGE-001 | "배경 제거는 사용자 지시로 외부 AI 연동 확정까지 대기"이며 완료로 보고하지 않음 | `backgroundRemoval.status='pending'`, UI 버튼 비활성, 실제 alpha 처리 없음 | 없음. HANDOFF·가이드 구현결과절·common.md·07-TRACEABILITY.md가 모두 동일하게 기록. 다만 사용자 결정의 출처가 이 워크플로우 문서 체계 자체이며, 외부(사람) 승인 로그 등 별도 증거는 이번 검사 범위에서 확인 불가 |
| REQ-PUBLIC-LIBRARY-001 | 가이드 §라이브러리와 Terminal | "공용부품은 샘플 읽기 어댑터, 실제 서비스 미연동" | `samplePublicParts` 구현, status='sample' | 없음, 일치 |
| VAL-FE03-검증수치 | HANDOFF §검증 결과 | "npm test 8파일 53개", "Chromium E2E 21개 통과" | 재실행 결과 동일(53/53, 21/21) | 없음, 과거 기록이 아니라 이번 실행으로 재확인 |
| VAL-FE03-변경파일 | HANDOFF §구현 내용·파일 | 파일 목록 13개 항목(objects.ts 등) | 전 파일 존재 확인, `git diff --stat` merge-base 대비 변경 없음(HEAD가 이미 main과 동일 커밋) | 없음. HANDOFF에 없는 변경 파일 발견되지 않음 |
| VAL-FE03-그룹해제 | 가이드 §객체와 선택-4 "그룹 해제 UI를 제공한다면 검증" | "그룹 해제 UI는 제공하지 않는다" | 코드에 ungroup 기능 없음 확인 | 조건부 요구("제공한다면")이므로 미제공은 위반 아님 — N/A |

### 집계와 후속 조치

- 전체: 19 / PASS: 18 / FAIL: 0 / WARNING: 1 / N/A: 0 (HANDOFF 대조 표의 그룹해제 조항은 별도 N/A로 부기, 체크리스트 집계에는 미포함)
- 이전 회차 대비: 첫 회차
- FAIL: 없음
- WARNING: REQ-IMAGE-001c/DES-IMAGE-001c(배경 제거 미구현) — REQ-IMAGE-001의 필수 최소 기능 중 하나가 전혀 구현되지 않음. 조치: 외부 AI 배경 제거 서비스 선정·연동 확정 후 FE-03 또는 FE-05에서 재구현·재검증 필요. 현재 상태로는 REQ-IMAGE-001 전체 완료 선언 불가
- N/A: VAL-FE03-그룹해제(그룹 해제 UI, 가이드상 조건부 요구이며 미제공 자체는 위반 아님)
- PASS(실연동 미확인): REQ-PUBLIC-LIBRARY-001(공용부품 샘플 어댑터)
- 확인하지 못한 범위: 배경 제거 "사용자 지시" 결정의 외부(워크플로우 문서 밖) 승인 근거, Firefox/WebKit/모바일 브라우저 검증(설정 파일에 프로젝트 자체가 없어 범위 밖), 100부품·300배선 규모 성능
- HANDOFF에 없는 변경 파일: 없음 (`git diff --stat` merge-base 대비 차이 없음, HEAD가 main과 동일 커밋)
- 추가 구현·문서 충돌: 없음. 가이드·HANDOFF·common.md·07-TRACEABILITY.md의 FE-03 관련 기술이 서로 일치함
