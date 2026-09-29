# 프로젝트 문서 안내

프로젝트 문서는 핵심 규격서(`core`)와, 이를 작업 단위로 나누어 구현·검증하는 워크플로우 문서(`workflow`)로 구분한다.

## 문서 유형과 호칭

문서는 아래 공식 호칭으로 부른다. "가이드", "기준 가이드", "작업 문서", "단계 문서", "계획서" 같은 혼용 표현은 쓰지 않는다.

| 호칭 | 약칭 | 위치 | 성격·내용 |
|---|---|---|---|
| 핵심 규격서 | URS·DS | `core/USER_REQUIREMENTS_SPEC.md`, `core/DESIGN_SPEC.md` | 프로젝트 전체 범위의 기준선. 사용자 요구사항 규격서(URS)와 설계 규격서(DS)로 구성하며 `REQ-*`/`DES-*` ID를 정의한다. |
| 공통 지침 | - | `workflow/common.md`, `workflow/<영역>/common.md` | 전역 또는 영역 전체의 업무 원칙·책임·작업 순서. |
| 작업 규격서 | WPS | `workflow/<영역>/NN-TOPIC.md` | 핵심 규격서를 영역·작업 단위로 나눈 규격. 대상 ID, 선행 조건, 범위, 완료 기준, 검증 방법을 정한다. |
| 작업 인계서 | HANDOFF | `workflow/<영역>/NN-TOPIC-HANDOFF.md` | 작업 규격서의 수행 결과. 실제 변경·결정·검증·제한·다음 작업을 기록한다. |
| 검증 보고서 | VERIFY | `workflow/<영역>/NN-TOPIC-VERIFY.md` | `work-validation` 스킬이 작업 규격서·작업 인계서를 실제 구현·테스트와 대조한 결과. 재검증은 회차를 추가한다. |
| 검증 종합 보고서 | VSR | `workflow/<영역>/VERIFY-SUMMARY.md` | 영역의 검증 보고서 판정을 모은 요약. |
| 요구사항 추적표 | RTM | `workflow/frontend/07-TRACEABILITY.md` | 핵심 규격서 ID를 작업 규격서·구현·증거에 연결한다. |

위계: 핵심 규격서 → 공통 지침 → 작업 규격서 → 작업 인계서 → 검증 보고서 → 검증 종합 보고서. 규격서는 "무엇이어야 하는가"를, 인계서·보고서는 "무엇을 했고 결과가 어떤가"를 기록한다.

## 폴더 구조

```text
docs/
  README.md                         프로젝트 문서 안내
  core/
    USER_REQUIREMENTS_SPEC.md        핵심 규격서 — 사용자 요구사항 규격서 (URS)
    DESIGN_SPEC.md                   핵심 규격서 — 설계 규격서 (DS)
  workflow/
    common.md                        전역 공통 지침
    frontend/
      common.md                      프런트엔드 공통 지침·실행 순서
      01-FOUNDATION.md               작업 규격서
      01-FOUNDATION-HANDOFF.md       작업 인계서
      01-FOUNDATION-VERIFY.md        검증 보고서
      02-WORKSPACE.md
      02-WORKSPACE-HANDOFF.md
      02-WORKSPACE-VERIFY.md
      03-OBJECTS-IMAGES.md
      03-OBJECTS-IMAGES-HANDOFF.md
      03-OBJECTS-IMAGES-VERIFY.md
      04-WIRING.md
      05-INTEGRATIONS.md
      06-VALIDATION.md
      07-TRACEABILITY.md             요구사항 추적표
      VERIFY-SUMMARY.md              검증 종합 보고서
    backend/
      common.md                      백엔드 공통 지침·실행 순서
      01-DATABASE.md                 작업 규격서 — DB 저장 계약·구현·검증
```

## 읽는 순서

1. 핵심 규격서 — [사용자 요구사항 규격서 (URS)](core/USER_REQUIREMENTS_SPEC.md)와 [설계 규격서 (DS)](core/DESIGN_SPEC.md): 사용자 동작·필수 기능과 UI/UX·객체·배선 설계의 기준.
2. [전역 공통 지침](workflow/common.md): 제품 배경, 업무 분장, 개발 원칙과 공통 인계 기준.
3. 담당 영역의 [Frontend 공통](workflow/frontend/common.md) 또는 [Backend 공통](workflow/backend/common.md): 영역별 책임과 작업 순서.
4. 해당 영역의 작업 규격서와 선행 작업의 작업 인계서·검증 보고서: 구체적인 범위·계약·완료 기준·검증 결과. 프런트엔드 규격별 대응은 [요구사항 추적표](workflow/frontend/07-TRACEABILITY.md)에서 확인한다.

## 작성과 관리 규칙

- `core/`는 핵심 규격서를 관리한다. 워크플로우는 핵심 규격서를 기준으로 작업을 나눈다.
- 공통 지침 중 `workflow/common.md`는 전역 업무 원칙, `workflow/<영역>/common.md`는 해당 영역 전체의 업무 범위를 다룬다.
- 작업 규격서는 `00-work.md`와 같은 번호·이름 형식인 `workflow/<영역>/NN-TOPIC.md`로 작성한다. 기존 프런트엔드 작업 번호와 파일명은 유지한다.
- 작업 규격서에는 대상 REQ/DES ID, 선행 조건, 작업 범위, 완료 기준과 검증 방법을 기록한다. 규격에 없는 제안과 미정 사항은 확정 요구와 구분한다.
- 작업 인계서는 같은 번호의 `NN-TOPIC-HANDOFF.md`로 연결하고 실제 변경·검증·제한·다음 작업을 기록한다.
- 검증 보고서는 `work-validation` 스킬이 같은 번호의 `NN-TOPIC-VERIFY.md`로 작성한다. 이전 회차는 수정하지 않는다.
- 기존 제품 계획과 과거 구현 기록은 현재 핵심 규격서의 확정 범위나 최신 구현 상태로 간주하지 않는다.
- 도메인·로컬 저장·API·인프라의 작업 규격서는 아직 작성되지 않았다. 해당 업무 착수 시 영역별 공통 지침과 작업 규격서를 추가하고 이 안내에 연결한다.

## 관련 문서

- [BE-01 작업 규격서](workflow/backend/01-DATABASE.md): 저장 계약, 권한, 마이그레이션과 검증 기록.
- [백엔드 실행 안내](../backend/README.md): 설치, 환경설정, 실행과 검증 명령.
- [참고 프로젝트 분석](../ref/ARCHITECTURE_ANALYSIS.md): 기존 EasyCable의 기능·구조 분석.

현재 도메인 타입은 `src/domain/`, CI 설정은 `.github/workflows/ci.yml`을 확인한다. 문서에는 현재 구현과 향후 계획을 구분하며, 기능 구현 시 실제 경로·계약·검증 방법도 함께 갱신한다.
