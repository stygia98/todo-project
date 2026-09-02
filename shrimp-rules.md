# 개발 규칙 (shrimp-rules.md)

> **대상**: 이 저장소에서 작업하는 Coding Agent 전용. 사람을 위한 튜토리얼이 아니다.
> **최종 검증**: 2026-09-02 · 실제 파일 스캔 기준
> **이 문서의 지위**: **작업 절차 규칙**이다. **기술 스펙의 정본이 아니다.**
> 스펙이 필요하면 `CLAUDE.md`를 읽는다. 이 문서는 "무엇을, 어느 파일을, 어떤 순서로 함께 고치는가"만 정의한다.

---

## 1. 정본 우선순위

충돌하면 **위쪽이 이긴다.**

| 순위 | 문서 | 담당 범위 |
|---|---|---|
| 1 | `CLAUDE.md` (루트) | 기술 규칙. 버전·API 계약·보안·컨벤션의 단일 기준 |
| 2 | `docs/PRD.md` | 기능 요구사항(ID 체계), 화면별 요구사항, 에러 문구 매핑 |
| 3 | `docs/ROADMAP.md` | Phase 순서 + **완료 판정(DoD)의 정본** |
| 4 | `shrimp-rules.md` (이 문서) | 작업 절차, 파일 연동, 금지 사항 |
| 5 | `docs/guides/*.md` | 참고 자료. **스펙 아님** |

**규칙**
- 이 문서가 `CLAUDE.md`와 충돌하면 → `CLAUDE.md`를 따르고 **이 문서를 고친다.**
- ⚠️ **이 순위는 「스펙·완료 판정」에만 적용된다. 「저장소의 현재 상태」에 대해서는 실제 파일이 정본이다.**
  문서(3절 스냅샷 포함)와 실제가 다르면 **먼저 실제를 확인하고 문서 쪽을 고친다.** 순위가 낮다는 이유로 더 최신인 사실을 지우지 않는다.
  예: 브랜치가 이미 `main`인데 어떤 문서가 `master`라고 적혀 있으면, `git branch -m`을 다시 실행하는 것이 아니라 **그 문서를 고친다.**
- `docs/guides/`가 `CLAUDE.md`와 충돌하면 → `CLAUDE.md`를 따르고 **가이드를 고친다.**
- `docs/guides/`에 `CLAUDE.md` 규칙을 **복사하지 않는다.** 링크로 참조한다.

---

## 2. 저장소 라우팅 — 어디를 고칠 것인가

**폴리레포다. 독립된 Git 저장소가 3개다. 모노레포가 아니다.**

| 변경 내용 | 저장소 | 경로 |
|---|---|---|
| 스펙·요구사항·로드맵·가이드 | `todo-project` (루트) | `CLAUDE.md`, `docs/**` |
| Spring Boot API, 엔티티, 보안, 테스트 | `todo-backend` | `todo-backend/**` |
| Next.js 화면, 훅, 컴포넌트 | `todo-frontend` | `todo-frontend/**` |

**필수**
- Git 명령은 **저장소를 명시**한다: `git -C todo-backend status` / `git -C todo-frontend status`
- 루트에서 `git add .`를 실행하지 않는다. 루트 `.gitignore`가 하위 두 폴더를 제외하고 있으나, 실수로 규칙을 지우면 gitlink로 커밋되어 클론 시 빈 폴더만 남는다.
- **한 커밋에 두 저장소의 변경을 섞지 않는다.**

---

## 3. 현재 상태 스냅샷 (2026-09-02 검증)

작업 착수 전 이 표와 실제 상태가 같은지 확인한다. 다르면 이 절을 먼저 갱신한다.

| 저장소 | 브랜치 | 커밋 | 원격 | 비고 |
|---|---|---|---|---|
| `todo-project` | `main` (`69894a2`) · `develop` (`38400ea`) | main 20 | `main`·`develop` 푸시 완료 | 태그 `v0.0.0`. `docs/ROADMAP.md`·`shrimp-rules.md` 미커밋(이번 Phase 8 문서 동기화 작업분) |
| `todo-backend` | `main` (`6f7aca6`) · `develop` (`de9b0f7`) | main 19 | `main`·`develop` 푸시 완료 | 태그 `v0.0.0`~`v0.5.0`. 미커밋 변경 없음(clean) |
| `todo-frontend` | `main` (`61a14d5`) · `develop` (`24cc490`) | main 7 | `main`·`develop` 푸시 완료 | 태그 `v0.0.0`·`v0.6.0`·`v0.7.0`. Phase 8 작업분 미커밋(아래 참조) |

**세 저장소 모두 `develop` 분기 + `origin` 푸시 완료이며, `master`는 없다.**
태그는 전부 annotated이며 원격에 peeled ref(`^{}`)와 함께 존재한다. `v0.5.0`은 `6f7aca6`(= `todo-backend main`), `v0.6.0`·`v0.7.0`은 각각 `todo-frontend main`의 Phase 6·Phase 7 병합 커밋을 가리킨다.
`main`이 `develop`보다 병합 커밋 하나만큼 앞선 구조는 Phase 1에서 확립된 패턴이며 오류가 아니다.

⚠️ **`todo-frontend`의 Phase 8 작업분은 이 스냅샷 시점에 아직 커밋되지 않았다.**
`main` 브랜치 작업 트리에 **신규/수정 다수**로 존재한다(`hooks/useTodos.ts`·`useLeaveConfirm.ts`, `hooks/useAuth.ts` 수정,
`components/todo/`(TodoEditor·TodoForm·FormSkeleton·TodoItem·TodoList), `components/ui/calendar.tsx`·`checkbox.tsx`·`popover.tsx`·`select.tsx`,
`app/(main)/todos/page.tsx` 전면 재작성, `app/(main)/todos/new/`·`app/(main)/todos/[id]/` 신규, `lib/queryKeys.ts` 수정,
`package.json`·`package-lock.json`(Tiptap 확장·shadcn 컴포넌트 의존성 추가)). 커밋·병합·태그 `v0.8.0`은 사용자 승인 후 별도 태스크에서 수행한다.
아래 소스 목록은 **작업 트리 기준**이다.

원격은 `git@github.com:stygia98/{todo-project,todo-backend,todo-frontend}.git`이다(언더스코어 이름은 301 리다이렉트로만 남아 있다).

⚠️ **앞으로의 작업은 `feature/{작업명}` → `develop` → `main` 순서다.** `main`에 직접 커밋하지 않는다.

**구현된 소스 (이것이 전부다)**

```
todo-backend/src/main/java/com/example/TodoBackendApplication.java   # @SpringBootApplication + @EnableJpaAuditing

# --- config (Phase 1: 2개 / Phase 3: 4개 추가 / Phase 5: 3개 추가) ---
todo-backend/src/main/java/com/example/config/SecurityConfig.java    # [P3 확장][P5 수정] csrf.disable, STATELESS, permitAll, CORS, exceptionHandling, oauth2Login 배선
todo-backend/src/main/java/com/example/config/OpenApiConfig.java     # bearerAuth 스킴
todo-backend/src/main/java/com/example/config/JwtTokenProvider.java  # [P3] 발급·검증. signWith(key, Jwts.SIG.HS256)
todo-backend/src/main/java/com/example/config/JwtAuthenticationFilter.java     # [P3] sub → id 조회, deleted_at IS NULL
todo-backend/src/main/java/com/example/config/JwtAuthenticationEntryPoint.java # [P3] 필터 단계 401을 ApiResponse 포맷으로
todo-backend/src/main/java/com/example/config/JwtAccessDeniedHandler.java      # [P3] 필터 단계 403을 ApiResponse 포맷으로
todo-backend/src/main/java/com/example/config/CustomOAuth2UserService.java    # [P5 신규] OidcUserService 확장. 신규가입/기존조회/충돌거부 3분기
todo-backend/src/main/java/com/example/config/OAuth2SuccessHandler.java       # [P5 신규] JWT 발급 후 {FRONTEND_URL}/oauth/callback?token= 302
todo-backend/src/main/java/com/example/config/OAuth2FailureHandler.java       # [P5 신규] email_conflict → /login?error=email_conflict 302

# --- domain (Phase 2 기본 + Phase 4 확장) ---
todo-backend/src/main/java/com/example/domain/BaseEntity.java        # @MappedSuperclass, 감사 필드(Instant)
todo-backend/src/main/java/com/example/domain/User.java              # users
todo-backend/src/main/java/com/example/domain/Todo.java              # todos, @Index(idx_todos_user_deleted), user는 LAZY. Phase 4에서도 미수정
todo-backend/src/main/java/com/example/domain/AuthProvider.java      # LOCAL / GOOGLE
todo-backend/src/main/java/com/example/domain/Priority.java          # HIGH / MEDIUM / LOW
todo-backend/src/main/java/com/example/domain/UserRepository.java    # deleted_at IS NULL 조건 포함
todo-backend/src/main/java/com/example/domain/TodoRepository.java    # [P4 수정] JpaSpecificationExecutor 확장. 기존 2메서드 유지
todo-backend/src/main/java/com/example/domain/TodoSpecifications.java # [P4 신규] ownedBy/completedIs/titleContains. @Query 대신 채택한 이유는 6.7 참조

# --- Phase 3 신규 패키지 ---
todo-backend/src/main/java/com/example/controller/AuthController.java  # signup / login / me. 로그아웃 API 없음
todo-backend/src/main/java/com/example/service/AuthService.java        # 로그인 실패는 단일 ErrorCode
todo-backend/src/main/java/com/example/dto/ApiResponse.java            # {success, data, error} 공통 봉투
todo-backend/src/main/java/com/example/dto/SignupRequest.java
todo-backend/src/main/java/com/example/dto/LoginRequest.java           # @NotBlank만. @Email을 걸지 않는다
todo-backend/src/main/java/com/example/dto/TokenResponse.java
todo-backend/src/main/java/com/example/dto/UserResponse.java
todo-backend/src/main/java/com/example/exception/ErrorCode.java        # [P4 수정] NOT_FOUND·METHOD_NOT_ALLOWED 추가로 8종
todo-backend/src/main/java/com/example/exception/BusinessException.java
todo-backend/src/main/java/com/example/exception/GlobalExceptionHandler.java  # [P4 수정] NoResourceFoundException·HttpRequestMethodNotSupportedException 핸들러 추가
todo-backend/src/main/java/com/example/validation/MaxByteLength.java          # 스펙 구조도에 없는 신설 패키지
todo-backend/src/main/java/com/example/validation/MaxByteLengthValidator.java # BCrypt 72바이트 검증

# --- Phase 4 신규 패키지 ---
todo-backend/src/main/java/com/example/controller/TodoController.java  # 6엔드포인트. 클래스에 @SecurityRequirement
todo-backend/src/main/java/com/example/service/TodoService.java        # 소유권 검증(findOwned), 정렬 화이트리스트(sanitizeSort)
todo-backend/src/main/java/com/example/service/HtmlSanitizer.java      # Jsoup Safelist. addEnforcedAttribute로 rel/target 강제
todo-backend/src/main/java/com/example/dto/PageResponse.java           # Page<E> → DTO 변환. of(page, mapper)
todo-backend/src/main/java/com/example/dto/TodoCreateRequest.java
todo-backend/src/main/java/com/example/dto/TodoUpdateRequest.java      # completed 없음(의도)
todo-backend/src/main/java/com/example/dto/TodoToggleRequest.java      # completed는 래퍼 Boolean + @NotNull
todo-backend/src/main/java/com/example/dto/TodoResponse.java           # user 정보 없음(의도)
todo-backend/src/main/resources/db/seed-dev.sql                        # 계정1 + Todo 100건. 기능 확인용
todo-backend/src/main/resources/db/seed-perf.sql                       # 같은 계정에 Todo 10000건. 성능 측정 전용

todo-backend/src/main/resources/{application,application-local,application-prod}.yml   # [P5 수정] application.yml에 security.oauth2.client.registration.google 추가
todo-backend/src/test/java/com/example/TodoBackendApplicationTests.java    # contextLoads
todo-backend/src/test/java/com/example/domain/UserRepositoryTest.java      # @DataJpaTest 5건
todo-backend/src/test/java/com/example/domain/TodoRepositoryTest.java      # [P4 수정] 11건. search 테스트 3건이 Specification 방식으로 교체됨
todo-backend/src/test/java/com/example/controller/AuthControllerTest.java  # [P3] @SpringBootTest + MockMvc 12건
todo-backend/src/test/java/com/example/controller/TodoControllerTest.java  # [P4 신규] 21건. 14장 4~7번 + DoD 전용 케이스
todo-backend/src/test/java/com/example/service/TodoServiceTest.java        # [P4 신규] 8건. 정렬 화이트리스트·소유권·PUT/toggle/delete·정화
todo-backend/src/test/java/com/example/config/CustomOAuth2UserServiceTest.java  # [P5 신규] 5건. processOidcUser 직접 호출(같은 패키지), MockMvc 미사용
todo-backend/src/test/resources/application-test.yml                       # todolist_test, create-drop + [P3] jwt·app 픽스처

todo-frontend/src/app/layout.tsx                                     # [P6 수정] Geist → Pretendard(next/font/local), Providers로 children 감쌈
todo-frontend/src/app/page.tsx                                       # create-next-app 기본값. 최종 스펙엔 없는 라우트(정리 대상은 아님)
todo-frontend/src/app/globals.css                                    # [P6 수정] CLAUDE.md 8장 팔레트로 전면 교체, 다크모드 class→media 전환
todo-frontend/src/app/dod-verify/page.tsx                            # [P6 신규] DoD 브라우저 검증용 데모 페이지. 사용자 확인 후 존치(Phase 7~9 재사용 가능)
todo-frontend/src/app/fonts/PretendardVariable.woff2                 # [P6 신규] Pretendard 공식 릴리스 v1.3.9에서 추출
todo-frontend/src/components/ui/button.tsx                           # [P6 수정] shadcn style radix-nova → new-york로 재생성
todo-frontend/src/components/ui/skeleton.tsx                         # [P6 신규] shadcn 공식 스켈레톤 프리미티브
todo-frontend/src/components/common/Pagination.tsx                   # [P6 신규] currentPage는 1-based(화면 표시용). totalPages<=1이면 null
todo-frontend/src/components/common/EmptyState.tsx                   # [P6 신규]
todo-frontend/src/components/common/ErrorState.tsx                   # [P6 신규] onRetry 필수 prop
todo-frontend/src/components/common/Skeleton.tsx                     # [P6 신규] 항목형, ui/skeleton.tsx 조합
todo-frontend/src/components/common/Header.tsx                       # [P6 신규] 공통 헤더 껍데기. nickname/onLogout은 Phase 7에서 useAuth 연결
todo-frontend/src/components/providers/QueryProvider.tsx             # [P6 신규] React Query Provider
todo-frontend/src/components/providers/Providers.tsx                 # [P6 신규] QueryProvider + sonner Toaster(next-themes 없이 theme="system")
todo-frontend/src/lib/utils.ts                                       # cn()
todo-frontend/src/lib/apiClient.ts                                   # [P6 신규] apiFetch, 토큰 관리, 401 처리, NetworkError/ApiRequestError 구분
todo-frontend/src/lib/errorMessages.ts                                # [P6 신규] PRD 5.1 매핑
todo-frontend/src/lib/validation.ts                                  # [P6 신규] 이메일/비밀번호(바이트)/닉네임/제목/본문 검증
todo-frontend/src/lib/sanitize.ts                                    # [P6 신규] DOMPurify 래퍼, 서버 HtmlSanitizer와 동일 화이트리스트
todo-frontend/src/lib/queryKeys.ts                                   # [P6 신규] todosKey/todoKey/authMeKey
todo-frontend/src/types/api.ts                                       # [P6 신규] ApiResponse, ApiError, PageResponse
todo-frontend/src/types/user.ts                                      # [P6 신규] UserResponse(id/email/nickname만), TokenResponse(필드명 token)
todo-frontend/src/types/todo.ts                                      # [P6 신규] Priority, TodoResponse, TodoCreateRequest, TodoUpdateRequest(completed 없음), TodoToggleRequest

todo-frontend/src/lib/authService.ts                                 # [P7 신규] login→POST /auth/login, signup→POST /auth/signup(계획서 지침의 엔드포인트가 서로 뒤바뀐 오타를 실소스 대조로 정정)
todo-frontend/src/hooks/useAuth.ts                                   # [P7 신규] isExpired(exp 디코드)로 게이팅한 /auth/me 조회 + login/logout. Context 없이 authMeKey 캐시를 그대로 전역 상태로 씀
todo-frontend/src/components/ui/input.tsx                            # [P7 신규] shadcn 추가(react-hook-form 등 딸려오지 않음 확인)
todo-frontend/src/components/ui/label.tsx                            # [P7 신규] shadcn 추가. 기존 설치된 radix-ui 패키지의 Label 프리미티브 사용
todo-frontend/src/app/(main)/layout.tsx                               # [P7 신규] 라우트 보호 전담. isPending/isAuthenticated/user/logout을 useAuth에서 받아 Skeleton→null(리다이렉트)→Header+children 순으로 렌더
todo-frontend/src/app/(main)/todos/page.tsx                           # [P7 신규] 라우트 보호 검증용 플레이스홀더. 실제 목록 UI는 Phase 8
todo-frontend/src/app/(auth)/login/page.tsx                           # [P7 신규] useSearchParams(?error=email_conflict)를 Suspense로 감쌈. submitLogin을 폼 제출·재시도 버튼이 공유
todo-frontend/src/app/(auth)/signup/page.tsx                          # [P7 신규] touched는 blur가 아니라 onChange 최초 호출에서 true — 타이핑 중 실시간 검증
todo-frontend/src/app/oauth/callback/page.tsx                         # [P7 신규] (auth) 그룹이 아니라 oauth/ 최상위(CLAUDE.md 2장 구조도). hasHandledRef로 중복 처리 가드

todo-frontend/src/hooks/useTodos.ts                                   # [P8 신규] useTodos/useCreateTodo/useUpdateTodo/useToggleTodo/useDeleteTodo(낙관적 삭제)/useUpdateTodosQuery
todo-frontend/src/hooks/useLeaveConfirm.ts                            # [P8 신규] beforeunload+popstate(더미 history 1개)+confirmAndLeave 3계층
todo-frontend/src/hooks/useAuth.ts                                    # [P8 수정] 하이드레이션 불일치 수정(hasMounted 게이트로 isPending도 강제 true)
todo-frontend/src/lib/queryKeys.ts                                    # [P8 수정] TodosQueryParams를 export로 변경
todo-frontend/src/components/ui/checkbox.tsx                          # [P8 신규] shadcn 추가. TodoItem 완료 토글에 사용
todo-frontend/src/components/ui/select.tsx                            # [P8 신규] shadcn 추가. TodoForm 우선순위 선택
todo-frontend/src/components/ui/popover.tsx                           # [P8 신규] shadcn 추가. TodoForm 마감일 피커
todo-frontend/src/components/ui/calendar.tsx                          # [P8 신규] shadcn 추가. date-fns 기반, popover와 조합
todo-frontend/src/components/todo/TodoEditor.tsx                      # [P8 신규] Tiptap 통합. StarterKit heading[2,3]/strike:false/horizontalRule:false/underline:false, link은 내장 옵션. 툴바 버튼 onMouseDown preventDefault(포커스 유지)
todo-frontend/src/components/todo/TodoForm.tsx                        # [P8 신규] title/priority/dueDate/content 4필드. onSubmit: Promise<boolean>. 완료 체크박스 없음(의도). 성공 시에만 dirty 기준선(ref) 갱신
todo-frontend/src/components/todo/FormSkeleton.tsx                    # [P8 신규] 폼 형태 스켈레톤
todo-frontend/src/components/todo/TodoItem.tsx                        # [P8 신규] 체크박스+제목(완료 시 취소선)+우선순위 뱃지+마감일+삭제
todo-frontend/src/components/todo/TodoList.tsx                        # [P8 신규] TodoItem 목록 래퍼
todo-frontend/src/app/(main)/todos/page.tsx                           # [P8 전면 재작성] URL 쿼리 기반 검색·필터·페이지, <Suspense>, 4가지 화면 상태 분기. Phase 7 플레이스홀더를 대체
todo-frontend/src/app/(main)/todos/new/page.tsx                       # [P8 신규] TodoForm + useLeaveConfirm 연결
todo-frontend/src/app/(main)/todos/[id]/page.tsx                      # [P8 신규] 단건 조회+404 전용 화면+삭제. key={data?"loaded":"loading"}로 강제 리마운트
```

백엔드 테스트는 **총 63건**이다(`AuthControllerTest` 12 + `TodoControllerTest` 21 + `TodoRepositoryTest` 11 +
`TodoServiceTest` 8 + `UserRepositoryTest` 5 + `CustomOAuth2UserServiceTest` 5 + `contextLoads` 1).
`AuthControllerTest`의 12건은 14장 "통합 테스트 1~3번"(시나리오 3개)을, `TodoControllerTest`의 21건 중
15건은 "통합 테스트 4~7번"(시나리오 4개)을 `@Nested` 묶음으로 구현한 것이다. **묶음 수(3, 4)가 시나리오 수이지
`@Test` 메서드 수가 아니다.** `TodoControllerTest`의 나머지 6건(PUT/toggle 2·입력검증 3·N+1 1)은
14장이 요구하지 않는 DoD 전용 추가 케이스다. `CustomOAuth2UserServiceTest`의 5건은 14장 "통합 테스트 8번"
(시나리오 1개, 서비스 단위 테스트로 작성)에 nickname 폴백·절삭 등 DoD 전용 케이스를 더한 것이다.

`domain/`·`config/`·`controller/`·`service/`·`dto/`·`exception/`·`validation/`에 모두 클래스가 있다.
**Phase 5 로 백엔드 구현 목록이 완결됐다.**

프론트는 **Phase 6 으로 스캐폴딩 목록이 완결됐다.** `src/types/`·`src/lib/`·`src/components/{ui,common,providers}/`가
모두 채워졌다. Phase 7에서 `src/hooks/`(`useAuth`)가 신설됐다. **Phase 8 로 `src/components/todo/`(TodoEditor·TodoForm·
FormSkeleton·TodoItem·TodoList)와 `useTodos`·`useLeaveConfirm`이 채워지며 프론트 화면 구현이 완결됐다.** 남은 것은
Phase 9(인터랙션 다듬기 — 토글 낙관적 업데이트 정교화·연타 대비·sonner 토스트·Motion 애니메이션)뿐이다.

⚠️ **`validation/` 패키지는 `CLAUDE.md` 2장 구조도에 없다.** 임의 추가가 아니라, 4장이 요구한
`@MaxByteLength` 커스텀 validator를 둘 곳이 필요해 신설했다. 구조도는 예시이지 금지 목록이 아니며,
검증 애노테이션을 `dto/`에 섞으면 DTO와 재사용 가능한 제약이 뒤엉킨다. **스펙 변경이 아니므로 `CLAUDE.md`를 고치지 않았다.**

**진행 Phase**: 0(저장소 초기화) **✅** · 1(백엔드 스캐폴딩) **✅ DoD 9항목 전수 통과** · 2(도메인 & DB) **✅ DoD 6항목 전수 통과(2026-08-31 재검증)** · 3(인증) **✅ DoD 15항목 전수 통과(2026-09-01). 단 DoD 12는 `curl` 대체 검증** · 4(Todo API) **✅ DoD 16항목 전수 통과(2026-09-01). 단 Swagger 확인은 `curl` 대체 검증** · 5(구글 OAuth2) **✅ DoD 5항목 전수 통과(2026-09-02). 라이브 브라우저 검증. 병합·태그 `v0.5.0` 완료** · 6(프론트 스캐폴딩) **✅ DoD 15항목 중 14항목 전수 통과, 1항목(`"use client"`) N/A(2026-09-02). 병합·태그 `v0.6.0` 완료** · 7(인증 화면) **✅ DoD 17항목(계획상 "16항목") 전수 통과(2026-09-02). 구글 로그인은 JWT 직접 주입으로 검증(백엔드 리다이렉트는 Phase 5에서 라이브 검증됨). 병합·태그 `v0.7.0` 완료** · 8(Todo 화면) **✅ DoD 26항목(계획상 "27항목") 전수 통과(2026-09-02). 360px 실측은 도구 한계로 555px 대체 검증**. 다음은 Phase 8 커밋·병합·태그 `v0.8.0`(사용자 승인 대기) 후 Phase 9(인터랙션 다듬기).

### 3.1 미해결 부채 — 착수 전 반드시 확인

Phase 6 완료로 이전에 있던 6항목이 전부 해소됐다(3.2 참조). 현재 남은 항목은 없다.

### 3.2 해소된 부채 (2026-08-28 · 2026-09-01 · 2026-09-02)

| 항목 | 조치 |
|---|---|
| 루트 `.metadata/` 미무시 | 루트 `.gitignore`에 `.metadata/` 추가. 스테이징 대상이 **563개 → 21개**로 줄었다 |
| 루트 `node_modules/` 미무시 | 재발 방지용으로 추가 |
| **없는 URL·잘못된 메서드가 404·405가 아니라 500** | Phase 4 착수 전 실측으로 재현·확대 확인 후 사용자 승인 받아 해소(2026-09-01). `GlobalExceptionHandler`의 catch-all `Exception` 핸들러가 `NoResourceFoundException`과 `HttpRequestMethodNotSupportedException`을 함께 삼켜 둘 다 500 `INTERNAL_ERROR`로 나가고 있었다 — 원인이 하나인 두 증상이라 함께 처리했다. `CLAUDE.md` 11장에 `NOT_FOUND`(404)·`METHOD_NOT_ALLOWED`(405) 행 추가(v1.10), `ErrorCode`에 두 상수 추가, catch-all 위에 전용 핸들러 2개 추가. **미인증 상태에서는 재현되지 않는다** — Security 필터가 먼저 401로 막는다. 기존 26건 회귀 없음, 실기동 curl로 재현·수정 모두 확인 |
| shrimp 태스크 상태 유입 | `shrimp_data/`를 루트 `.gitignore`에 추가 |
| `.claude/commmands/` 오타 | `.claude/commands/`로 정정. `/git commit` 커맨드가 등록된다 |
| shrimp `DATA_DIR` 오지정 | `.mcp.json`을 `D:\claude\todo-project\shrimp_data`로 변경 |
| 브랜치명 `master` | 세 저장소 모두 `main` |
| `todo-frontend`만 커밋 1건 | 세 저장소의 출발선을 맞추기 위해 **커밋과 `develop`을 제거**해 unborn `main`·0커밋으로 초기화했다. 커밋에만 있던 `AGENTS.md`(Next 16 자동 생성물, 의도적 삭제)와 `app/globals.css`(→ `src/app/globals.css`로 이동) 외에 **소실된 파일은 없다** |
| 원격 미연결 | 세 저장소 모두 `origin` 연결 완료 |
| Phase 1 전체 | ✅ **완료(2026-08-28).** DoD 9항목 전수 통과. `.properties` → yml 3분할, 평문 비밀번호를 `${DB_PASSWORD}`로 이전(히스토리 미유입 확인), DB 대상을 `todolist_db` 직결로 정정, 패키지 6개 + `SecurityConfig`·`OpenApiConfig`, `.env.example`·저장소 `CLAUDE.md` 작성. 기동 로그에 `local` 프로파일 확인, Swagger UI 200 + Authorize 버튼 렌더 확인 |
| Phase 0 전체 | ✅ **완료.** 첫 커밋 → `develop` 분기 → `main`·`develop` 푸시 → 태그 `v0.0.0`. ROADMAP Phase 0 **DoD 9항목 전부 통과** |
| 저장소 이름 | GitHub 3개 모두 `todo-` 하이픈으로 통일(`todo-project`/`todo-backend`/`todo-frontend`). 로컬 remote URL도 일치. 언더스코어 이름은 301 리다이렉트로만 남음 |
| `.env` 무시 규칙 | **세 저장소 모두** `.env*` + `!.env.example` 적용. `.env`는 무시되고 `.env.example`은 추적 가능함을 실파일로 검증 |
| Phase 6 전체(프론트 다크모드·디자인 토큰·`components.json` style·미설치 패키지·`.env.example`·Pretendard 폰트) | ✅ **완료(2026-09-02).** DoD 15항목 중 14항목 전수 통과, 1항목(`page.tsx`에 `"use client"`)은 검증 대상 부재로 N/A(사용자 확인). `globals.css`를 CLAUDE.md 8장 팔레트로 전면 교체하고 `.dark`/`@custom-variant dark`를 제거해 `@media (prefers-color-scheme: dark)`로 전환, `components.json` style을 `new-york`으로, `motion`·`sonner`·`date-fns`·`dompurify`·`@tanstack/react-query`·`@tiptap/react`·`@tiptap/starter-kit` 설치, Pretendard 가변 폰트를 `next/font/local`로 로드, `apiClient`·`errorMessages`·`validation`·`sanitize`·쿼리 키 상수화·공용 컴포넌트 4종·`src/types/`·`.env.example` 작성. 병합·태그 `v0.6.0` 완료 |
| Phase 7 전체(로그인·회원가입·구글 콜백·라우트 보호) | ✅ **완료(2026-09-02).** DoD 17항목(계획상 "16항목", 실제로는 17개 — Phase 6의 "14항목" 오기와 같은 패턴) 전수 통과. `authService`·`useAuth`(exp 디코드 게이팅, Context 없이 authMeKey 캐시가 전역 상태), `(main)` 보호 레이아웃, `/login`·`/signup`·`/oauth/callback` 작성. 백엔드 실제 종료 후 네트워크 실패 문구 확인, 만료 토큰 주입 시 `/auth/me` 요청 자체가 발생하지 않음을 네트워크 로그로 확인, 키보드만으로 가입 완주까지 실기동 검증. **병합·태그 `v0.7.0` 완료** |
| Phase 8 전체(Todo 목록·검색·필터·페이지네이션·Tiptap 편집·이탈 확인) | ✅ **완료(2026-09-02).** DoD 26항목(계획상 "27항목", 실제로는 26개 — Phase 6·7과 같은 집계 오기 패턴) 전수 통과. `useTodos`(React Query, 낙관적 삭제)·`useLeaveConfirm`(3계층: beforeunload+버튼+popstate)·`TodoEditor`(Tiptap, StarterKit 커스텀)·`TodoForm`(공용, `onSubmit: Promise<boolean>`)·목록/신규/상세 3라우트 작성. `db/seed-dev.sql`(100건) 적용해 페이지네이션·정렬 실측, 리치 콘텐츠·XSS 페이로드를 DB에 직접 주입해 서식 유지·sanitize 실증, React Query 기본 `retry:3`을 감안해 5회 연속 실패로 에러 상태 재현. 360px 정밀 실측은 `resize_window` 도구 한계로 555px 대체 검증. 병합·태그 `v0.8.0`은 사용자 승인 대기 |

---

## 4. 작업 시작 전 필수 절차

**모든 작업에서 이 순서를 지킨다.**

1. `docs/ROADMAP.md`의 **「진행 현황」 표**에서 현재 Phase를 확인한다.
2. 해당 Phase의 **작업 목록과 DoD**를 읽는다. DoD가 완료 판정의 정본이다.
3. 요구사항 ID(`AUTH-xx` / `TODO-xx` / `UX-xx`)가 있으면 `docs/ROADMAP.md`의 **「요구사항 ↔ Phase 추적표」**에서 구현/검증 Phase를 확인한다.
4. `CLAUDE.md`에서 해당 영역의 장을 읽는다. (4장 데이터 · 5장 API · 6장 인증/XSS · 8장 UI · 9장 상태 · 11장 에러 · 12장 환경변수)
5. **그 Phase의 범위만 구현하고 멈춘다.** 실행/검증 결과를 보고한 뒤 다음 Phase로 넘어가지 않는다.

**병렬 허용**: Phase 6(프론트 스캐폴딩)은 실서버를 호출하지 않으므로 백엔드 Phase 1~5와 **병렬 진행 가능**하다. 그 외 순서는 지킨다.

---

## 5. 다중 파일 동시 수정 규칙 (핵심)

> 아래 각 항목은 **한 곳만 고치면 조용히 깨진다.** 반드시 나열된 파일을 함께 수정한다.

### 5.1 서식 태그 집합 — **4곳 동시**

허용 태그: `p br strong em h2 h3 ul ol li a code pre blockquote`

| # | 파일 | 반영 지점 |
|---|---|---|
| 1 | `todo-frontend/src/components/todo/TodoEditor.tsx` | 툴바 버튼 목록 |
| 2 | 같은 파일 | `StarterKit.configure({ ... })` (끄는 확장 포함) |
| 3 | `todo-backend/src/main/java/com/example/service/HtmlSanitizer.java` | Jsoup `Safelist` |
| 4 | `todo-frontend/src/lib/sanitize.ts` | DOMPurify `ALLOWED_TAGS` / `ALLOWED_ATTR` |

**해서는 안 되는 것** — 툴바에 버튼만 추가하고 나머지 3곳을 두는 것. 사용자가 입력한 서식이 저장 후 **무음으로 사라진다.**

**해야 하는 것** — 태그를 하나 늘리거나 줄일 때 위 4곳 + `CLAUDE.md` 8장 툴바 목록까지 **5곳**을 함께 고친다.

### 5.2 API 계약 변경 — **순서 고정**

```
1) CLAUDE.md 5장 (API 명세)        ← 먼저 고친다
2) todo-backend   dto/ → controller/ → service/
3) todo-frontend  src/types/ → src/lib/apiClient.ts → 호출부
```

**역순 금지.** 백엔드나 프론트에서 계약을 임의로 바꾸지 않는다.

### 5.3 입력값 제약 변경 — **3곳**

| 파일 | 형태 |
|---|---|
| `CLAUDE.md` 4장 「입력값 제약」 표 | 정본 |
| `todo-backend/.../dto/*Request.java` | Bean Validation 애노테이션 |
| `todo-frontend/src/lib/validation.ts` | 수동 검증 함수 |

⚠️ 비밀번호 상한은 **문자 수가 아니라 UTF-8 바이트(72)**다. 세 곳 모두 바이트로 센다.
- 프론트: `new TextEncoder().encode(v).length`
- 백엔드: 커스텀 `@MaxByteLength(72)` validator → 위반 시 **400 `INVALID_INPUT`**

⚠️ **안전장치를 네 번째 파일에 함께 건다** — `todo-backend/.../exception/GlobalExceptionHandler.java`.
`BCryptPasswordEncoder`는 72바이트 초과분을 조용히 버리지 않고 **`IllegalArgumentException`을 던진다**(CVE-2025-22228 대응).
validator를 우회한 경로가 생기면 그대로 **500 `INTERNAL_ERROR`**가 나간다. `IllegalArgumentException` → **400** 매핑을 걸어둔다.

### 5.4 에러 코드 추가/변경 — **4곳**

| 파일 | 형태 |
|---|---|
| `CLAUDE.md` 11장 표 | 코드 ↔ HTTP 상태 |
| `docs/PRD.md` 5.1 「에러 문구 매핑」 | 사용자 문구 |
| `todo-backend/.../exception/ErrorCode.java` | enum |
| `todo-frontend/src/lib/errorMessages.ts` | 코드 → 문구 단일 변환 함수 |

**화면 컴포넌트에 에러 문구를 직접 쓰지 않는다.** 반드시 `errorMessages.ts`를 거친다.

⚠️ **`PRD.md` 5.1 표에는 `error.code`가 없는 행이 하나 있다 — 「네트워크 실패」.**
`apiClient`의 에러 정규화 단계에서 이 케이스를 **별도 구분자로 남겨야** `errorMessages.ts`가 "연결에 실패했습니다." + 재시도 버튼을 낼 수 있다.
코드 매핑만 만들면 이 행이 사문화된다. 검증은 **서버를 내린 상태로** 한다.

### 5.5 요구사항 추가 — **선행 2곳**

```
docs/PRD.md 3장 (ID 부여)
  └→ docs/ROADMAP.md 「요구사항 ↔ Phase 추적표」에 행 추가
       └→ 해당 Phase의 작업 목록 + DoD에 기술
            └→ 구현
```

**추적표에 행이 없는 P0는 구현되지 않는다.** 표를 건너뛰고 구현하지 않는다.

### 5.6 Phase 완료 시 — **2곳**

- `docs/ROADMAP.md` 「진행 현황」 표의 상태를 ⬜/🟡 → ✅ 로 갱신
- 해당 저장소에 태그 `v0.{Phase번호}.0` (Phase 10 통과 시 세 저장소 모두 `v1.0.0`)

### 5.7 디자인 토큰 변경 — **2곳**

`CLAUDE.md` 8장 컬러 표 ↔ `todo-frontend/src/app/globals.css`의 `@theme` + `@media (prefers-color-scheme: dark)`

### 5.8 환경 변수 추가 — **3곳**

| 백엔드 | 프론트 |
|---|---|
| `CLAUDE.md` 12장 | `CLAUDE.md` 12장 |
| `todo-backend/.env.example` | `todo-frontend/.env.example` |
| `application-{local,prod}.yml` 바인딩 | `NEXT_PUBLIC_` 접두사 + 사용부 |

⚠️ `FRONTEND_URL`(단일 URL)과 `CORS_ALLOWED_ORIGINS`(쉼표 목록)를 **하나로 합치지 않는다.**

### 5.9 문서 수정 시

`CLAUDE.md`(**v1.10**) · `docs/PRD.md`(**v1.8**) · `docs/ROADMAP.md`(**v2.5**)는 상단에 **버전·최종 수정일**을 가진다.
내용을 고치면 **그 문서의 버전·수정일을 함께 올린다.** 세 문서의 버전은 서로 독립이며 일치할 필요가 없다.

⚠️ **이 줄의 버전 번호도 함께 고친다.** 대상 문서만 올리고 여기를 두면 규칙 문서가 거짓이 된다.

---

## 6. 백엔드 규칙 (`todo-backend`)

### 6.1 `pom.xml` 아티팩트 명명 — Boot 3 이름 금지

이 프로젝트의 `pom.xml`은 **기능별로 분리된 스타터 명명**을 쓴다. 새 의존성을 추가할 때 **기존 항목의 명명 규칙을 그대로 따른다.**

| 이 저장소에서 쓰는 이름 | 대신 쓰면 안 되는 이름 |
|---|---|
| `spring-boot-starter-webmvc` | ~~`spring-boot-starter-web`~~ |
| `spring-boot-starter-security-oauth2-client` | ~~`spring-boot-starter-oauth2-client`~~ |
| `spring-boot-starter-webmvc-test`, `-security-test`, `-validation-test`, `-data-jpa-test`, `-security-oauth2-client-test` | ~~`spring-boot-starter-test`(단일)~~ |

**해서는 안 되는 것** — 인터넷 예제의 Boot 3 스타터명을 그대로 붙여넣는 것.

### 6.2 버전 핀 — 변경 금지

| 항목 | 값 | 위치 |
|---|---|---|
| Spring Boot | `4.1.1` | `<parent>` |
| Java | `21` | `<java.version>` |
| SpringDoc | `3.1.0` | `<springdoc.version>` |
| Jsoup | `1.23.2` | `<jsoup.version>` |
| jjwt | `0.12.6` × 3 아티팩트 | 직접 명시 |

- **jjwt는 3개가 정상이다.** `jjwt-impl`·`jjwt-jackson`의 `<scope>runtime</scope>`은 의도된 설정이다. "정리"하면 기동 시 `ClassNotFoundException`이 난다.
- jjwt는 **0.12 문법**이다. `Jwts.parser().verifyWith(key).build()` / `Jwts.builder().signWith(key)`. 0.11 예제(`setSigningKey`, `SignatureAlgorithm.HS256`)를 옮기면 컴파일에 실패한다.
- SpringDoc은 **범위로 두지 않는다.** Boot 마이너 버전과 1:1 대응이다.

### 6.3 Lombok 애노테이션 프로세서

`maven-compiler-plugin`에 `default-compile`·`default-testCompile` 두 execution의 `annotationProcessorPaths` 설정이 **이미 있다. 삭제하거나 단순화하지 않는다.**

### 6.4 설정 파일

- **`application.properties`와 `application.yml`을 동시에 두지 않는다.** `.properties`가 우선 적용되어 `.yml`이 조용히 무시된다.
- Phase 1에서 `.properties`를 **삭제**하고 `application.yml` + `application-local.yml` + `application-prod.yml`로 교체한다.
- `application.yml`에 `spring.profiles.active: local` 기본값을 둔다.
- 테스트 설정은 `src/test/resources/application-test.yml`.
- **DB는 `todolist_db`(테스트 `todolist_test`) 데이터베이스 *자체*를 쓴다.** `postgres` DB에 `?currentSchema=todolist_db`로 붙지 않는다 (`CLAUDE.md` 12·13장은 `createdb todolist_db` 전제).
  `spring.datasource.url`은 `jdbc:postgresql://localhost:5432/todolist_db` 형태다.
- **`application.yml`(공통)에 `spring.jpa.properties.hibernate.jdbc.time_zone: UTC`를 둔다.** 10절 20번 참조.
- ⚠️ **설정을 바꿨는데 동작이 그대로면 `./mvnw clean`을 먼저 실행한다.** Maven은 리소스를 `target/classes`로 복사할 뿐 **낡은 파일을 지우지 않는다.** Phase 1에서 `application.properties`를 소스에서 삭제했을 때 `target/classes`에 평문 비밀번호를 담은 복사본이 남아 있었다. 그대로 두면 런타임에 그 `.properties`가 로드되어 `.yml`을 덮어쓴다 — **소스에는 없는데 동작은 옛 설정인** 상태가 된다.

### 6.5 실행 명령 (Windows)

| 셸 | 명령 |
|---|---|
| PowerShell / cmd | `.\mvnw.cmd spring-boot:run` · `.\mvnw.cmd test` |
| Git Bash | `./mvnw spring-boot:run` · `./mvnw test` |

`todo-backend/.gitattributes`(`/mvnw text eol=lf`)는 **이미 있다. 삭제하지 않는다.**

**테스트·기동에는 환경변수 주입이 필요하다.** `application-test.yml`과 `application.yml`이 기본값 없이 참조하기 때문이다.

| 목적 | 필요한 환경변수 |
|---|---|
| `mvnw test` | `DB_PASSWORD` |
| `mvnw spring-boot:run` | `DB_USERNAME`, `DB_PASSWORD`, `JWT_SECRET`(raw UTF-8 32자 이상), `FRONTEND_URL`, `CORS_ALLOWED_ORIGINS` |

⚠️ **`JAVA_HOME`과 PATH의 JDK가 다르다** (2026-09-01 실측).

| 경로 | 버전 |
|---|---|
| `JAVA_HOME` (`C:\SpringBootProject\zulu21`) | **21.0.12.1** ← 프로젝트 요구 버전 |
| PATH의 `java` / `javac` | **17.0.19** |

`mvnw`는 `JAVA_HOME`을 우선 쓰므로 **Maven 빌드는 정상**이다. 그러나 Maven 밖에서 `javac`를 직접 호출하면
`wrong version 65.0, should be 61.0` 오류가 난다. `CLAUDE.md` 13장이 경고한 상황이 실제로 존재하는 환경이므로,
빌드가 되는데 다른 도구만 실패하면 이것부터 의심한다.

⚠️ **8080 포트를 무관한 타 프로젝트가 점유하고 있을 수 있다** (2026-09-01 기준 `com.hi.mallapi.MallapiApplication`).
남의 프로세스를 종료하지 말고 `SERVER_PORT=8081` 등으로 우회한다. 포트 번호는 DoD 판정에 영향을 주지 않는다.

### 6.6 계층 규칙

- 컨트롤러는 **엔티티를 반환하지 않는다.** 항상 DTO(`record`).
- 엔티티에 `@Setter` 금지. 변경은 `updateCompleted(boolean)`·`softDelete()` 같은 의미 있는 메서드로.
- `@ManyToOne`은 **반드시 `fetch = FetchType.LAZY`** 명시(기본값 EAGER).
- 물리 삭제 금지. `deleted_at` 기록 + 모든 조회에 `deleted_at IS NULL`.
- **감사 필드(`created_at`·`updated_at`)와 `deleted_at`은 `Instant`다. `LocalDateTime`으로 바꾸지 않는다.** 아래 두 이유 때문이며, 설정만으로는 대체되지 않는다.
  1. **UTC 저장이 설정만으로 보장되지 않는다.** `hibernate.jdbc.time_zone: UTC`는 JDBC 바인딩 계층 설정인데, `@CreatedDate` 값을 실제로 만드는 주체는 Spring Data의 시각 공급자이고 그 기본 구현은 **시스템 기본 타임존**(개발 환경은 KST +09:00)을 쓴다. `LocalDateTime`은 타임존 정보가 없어 이 KST 벽시계 값이 이후 UTC로 재해석될 근거가 없다 → 운영(UTC)과 9시간 어긋난다.
  2. **`CLAUDE.md` 5장의 직렬화 형식을 만족할 수 없다.** `2026-08-28T04:30:00Z`의 `Z`는 UTC를 뜻하는데 `LocalDateTime`은 타임존이 없어 이 접미사를 붙일 수 없다.

  `Instant`는 정의상 UTC 기준 시점이라 공급자의 타임존과 무관하고, PostgreSQL에서 `timestamptz`로 생성된다. `due_date`만은 시각이 없어 타임존 영향을 받지 않으므로 `LocalDate`를 그대로 쓴다.
- **`completed`·`priority`의 기본값은 DB `DEFAULT` 절이 아니라 `Todo` 생성자에 있다.** 실제 생성된 컬럼의 `column_default`는 비어 있다(2026-08-31 확인). JPA 경로는 무해하지만 **JPA를 우회한 직접 INSERT(Phase 4의 `seed-dev.sql`·`seed-perf.sql`)는 두 컬럼을 반드시 명시해야 한다.** 누락하면 NOT NULL 위반이다.

### 6.7 Phase 3에서 확정된 규칙

아래 넷은 **잘못 써도 컴파일이 통과하거나 테스트가 통과해버리는** 종류라 규칙으로 고정한다.

- **`ObjectMapper`는 `tools.jackson.databind` 쪽이며 `new`하지 않고 주입받는다.** Boot 4의 직렬화 엔진은 Jackson 3(`tools.jackson`)인데, `springdoc`과 `jjwt-jackson`이 Jackson 2(`com.fasterxml.jackson`)를 compile scope로 함께 끌고 온다. **잘못된 쪽을 import해도 컴파일은 통과한다.** 직접 `new`하면 Boot가 구성한 설정을 잃는다. `JwtAuthenticationEntryPoint`·`JwtAccessDeniedHandler`가 이 방식이며, 테스트도 같은 빈을 주입받아 프로덕션과 타입을 맞춘다.
- **로그인 실패는 원인과 무관하게 `ErrorCode.UNAUTHORIZED` 하나만 던진다.** 미가입·소셜 전용 계정·비밀번호 불일치를 구분해 응답하면 계정 존재 여부가 노출된다(`PRD.md` 5.1). **"메시지를 둘 만든 뒤 같게 맞추는" 방식을 쓰지 않는다.** 한쪽만 수정되면 조용히 갈라진다. 단일 코드로 모아 **애초에 갈릴 수 없는 구조**로 둔다.
- **JWT 서명은 `signWith(key, Jwts.SIG.HS256)`으로 알고리즘을 명시한다.** 인자 없는 `signWith(key)`를 쓰면 jjwt가 **키 길이로 알고리즘을 추론**한다(32~47B→HS256, 48~63B→HS384, 64B+→HS512). 즉 코드가 아니라 `JWT_SECRET`의 길이가 알고리즘을 결정하게 되어, 환경마다 시크릿 길이가 다르면 로컬과 운영이 갈린다. Phase 3 DoD 검증에서 49바이트 시크릿이 실제로 `alg=HS384`를 발급해 발견했다. 상수는 0.11의 `SignatureAlgorithm` enum이 아니라 **0.12의 `Jwts.SIG`**를 쓴다.
- **Boot 4는 테스트 애노테이션 패키지가 재편됐다.** 인터넷 예제를 그대로 옮기면 컴파일에 실패한다. jar를 열어 확인한 실측값이다.

  | 애노테이션 | Boot 4 경로 |
  |---|---|
  | `@SpringBootTest` | `org.springframework.boot.test.context` (**기존과 동일**) |
  | `@AutoConfigureMockMvc` | `org.springframework.boot.webmvc.test.autoconfigure` |
  | `@DataJpaTest` | `org.springframework.boot.data.jpa.test.autoconfigure` |
  | `@AutoConfigureTestDatabase` | `org.springframework.boot.jdbc.test.autoconfigure` |

  `@SpringBootTest`만 경로가 그대로라 **둘이 갈린다는 점**이 함정이다.

- **`application-test.yml`에 `jwt.secret`·`app.frontend-url`·`app.cors.allowed-origins`를 리터럴로 둔다.** Phase 3부터 `JwtTokenProvider`가 생성자에서, `SecurityConfig`가 필드에서 이 값을 읽어 `@SpringBootTest` 기동에 필수가 됐다(Phase 2까지 통과한 이유는 플레이스홀더를 읽는 빈이 없어 해석 자체가 일어나지 않아서다). **`DB_PASSWORD`는 실제 자격증명이므로 여기에 기본값을 심지 않는다.** 테스트 실행에는 환경변수 주입이 필요하다.

### 6.8 Phase 4에서 확정된 규칙

- **완료 필터·제목 검색 조합 조회는 JPQL `@Query`가 아니라 `Specification`을 쓴다.** `TodoRepository`는 처음 `@Query("... :completed IS NULL OR t.completed = :completed ... :keyword IS NULL OR LOWER(t.title) LIKE ...")` 형태로 만들었다가 폐기했다. PostgreSQL은 named parameter의 타입을 **그 SQL 텍스트가 처음 실행되는 시점의 문맥**으로 추론하는데, `keyword=null`이 최초 바인딩이면 문맥이 없어 `bytea`로 기본 처리돼 `lower(bytea)` 함수 없음 또는 `bytea→boolean` 캐스팅 불가 오류가 났다. `TodoRepositoryTest`(다른 값으로 먼저 실행해 타입이 캐시된 뒤라 우연히 통과)와 `TodoServiceTest`(별도 컨텍스트라 콜드 스타트, 실패)의 결과가 갈려 발견했다. **JPQL 안에서 `CAST(:param AS ...)`을 추가해도 소용없다** — pgjdbc가 파라미터를 바인딩하는 시점의 와이어 타입 자체가 문제라 SQL 안의 형변환으로는 고쳐지지 않는다. `domain/TodoSpecifications`(ownedBy·completedIs·titleContains)로 전환해 해결했다 — Criteria API는 메타모델에서 컬럼 타입을 미리 알고 명시적으로 바인딩하므로 이 문제 자체가 없다. 스택 추가 없이 `JpaSpecificationExecutor`(Spring Data JPA 기본 기능)만 썼다.
- **동적 조회 관련 테스트는 반드시 콜드 스타트(단독 실행)로도 재확인한다.** 위 사례처럼 "다른 테스트가 먼저 실행돼 우연히 통과"하는 경우가 실재한다. `-Dtest=클래스명`으로 단독 실행했을 때도 통과해야 안전하다고 판단한다.
- **정렬 화이트리스트는 서비스 진입부에서 처리하고 `Pageable`을 그대로 리포지토리에 넘기지 않는다.** `TodoService.sanitizeSort`가 `createdAt`·`dueDate` 밖의 정렬 프로퍼티를 걸러 `createdAt desc`로 대체한다. 컨트롤러에 두면 서비스를 직접 호출하는 테스트가 보호받지 못하고, 리포지토리에 그대로 넘기면 없는 프로퍼티로 500이 난다.
- **HtmlSanitizer의 `rel`·`target` 강제는 `Safelist.addEnforcedAttribute`로 하고, 정화 후 재파싱하지 않는다.** `addAttributes`는 통과만 허용할 뿐 값을 채우지 않는다 — 사용자가 `rel="opener"`를 직접 넣으면 그대로 통과한다. `addEnforcedAttribute(tag, key, value)`로 정화 한 번에 값을 강제한다. 정화 후 HTML을 다시 파싱해 속성을 수동 주입하는 방식은 정화 보장이 흐려지고 `Document.OutputSettings.prettyPrint(false)`를 재적용하는 것을 놓치기 쉬워 쓰지 않는다.
- **N+1 부재는 원리적 논증(LAZY 연관관계 + 응답 DTO에 필드 없음)이 아니라 Hibernate Statistics 실측으로 검증한다.** `EntityManagerFactory.unwrap(SessionFactory.class).getStatistics()`로 쿼리 실행 횟수를 잰다. **절대값이 아니라 항목 수 증가 전후의 차이를 비교한다** — `JwtAuthenticationFilter`가 매 요청마다 사용자 조회 쿼리 1건을 추가해 절대값이 매직 넘버가 되기 쉽다.

### 6.9 Phase 5에서 확정된 규칙

- **구글의 기본 scope에는 `openid`가 포함되므로 이 흐름은 OAuth2가 아니라 OIDC다.** `OAuth2UserService`가 아니라 **`OidcUserService`를 상속**해야 `userInfoEndpoint().oidcUserService(...)`로 등록할 수 있다. 일반 `OAuth2UserService`로 만들면 등록 지점(`.userInfoEndpoint().userService(...)`)부터 다르고, `OidcUser`가 제공하는 ID 토큰 클레임(email·name)에 접근할 수 없다.
- **`CustomOAuth2UserService.processOidcUser`를 `loadUser`에서 분리해 package-private으로 둔다.** `loadUser`는 네트워크(구글 서버)와 통신하는 `super.loadUser()` 호출을 포함해 `MockMvc`나 순수 단위 테스트로 끝까지 검증할 수 없다. 판정 로직(신규가입/기존조회/충돌거부 3분기)만 `processOidcUser`로 떼어내면, 테스트가 `OidcIdToken`에 클레임을 직접 심어 만든 `DefaultOidcUser`를 이 메서드에 바로 넣어 네트워크 없이 검증할 수 있다. `CustomOAuth2UserServiceTest`가 이 방식이다(14장 통합 테스트 8번).
- **계정 충돌은 `OAuth2AuthenticationException(new OAuth2Error("email_conflict"), message)`로 던진다.** 커스텀 `RuntimeException`을 던지면 Spring Security의 `oauth2Login` 필터가 이를 잡지 못해 500으로 새 버린다. `OAuth2AuthenticationException`이어야 `AuthenticationFailureHandler`(`OAuth2FailureHandler`)로 흐르고, `getError().getErrorCode()`로 실패 종류를 구분해 `email_conflict`와 그 외(`oauth_failed`)를 갈라 302 리다이렉트할 수 있다.
- **로컬 계정 존재 여부로 인한 충돌 검증은 순서에 주의한다.** 같은 이메일로 GOOGLE 계정을 먼저 만든 상태에서는 로컬 회원가입 자체가 `EMAIL_DUPLICATED`로 막혀 충돌 시나리오(로컬 선점 + 구글 시도)를 재현할 수 없다. 라이브 검증 시 GOOGLE 계정을 먼저 정리한 뒤 로컬 가입 → 구글 로그인 재시도 순서로 진행한다.

---

## 7. 프론트엔드 규칙 (`todo-frontend`)

### 7.1 설치 명령

```bash
npm install --legacy-peer-deps          # shadcn/ui + React 19 peer 충돌 회피
npx shadcn@latest add <component>       # 컴포넌트 추가
```

**`npx shadcn add form`을 실행하지 않는다.** `react-hook-form`과 `@hookform/resolvers`가 의존성으로 함께 설치된다. 폼은 `label` + `input`을 직접 조합한다.

### 7.2 설치 금지 목록

| 패키지 | 이유 |
|---|---|
| `react-hook-form`, `zod`, `@hookform/resolvers` | 폼 라이브러리 미도입 확정 |
| `next-themes` | 다크 모드 토글은 범위 밖 |
| `framer-motion` | deprecated 별칭. **`motion`**을 쓰고 `motion/react`에서 import |
| `@tiptap/extension-link` | v3 StarterKit에 포함. 중복 등록됨 |
| `next` 16 이상 | Amplify SSR 지원 범위가 12~15 |

`next`와 `eslint-config-next`는 **메이저를 항상 함께 맞춘다.**

### 7.3 파일 배치 금지

| 금지 | 이유 |
|---|---|
| `src/middleware.ts` | 토큰이 localStorage에 있어 middleware가 읽지 못한다. 라우트 보호는 `(main)` 클라이언트 레이아웃 |
| `public/static/` | Amplify 예약 경로 |
| `tailwind.config.js` | Tailwind 4는 CSS-first. `globals.css`의 `@theme` |
| `next.config.ts`의 `distDir` | 빌드 출력은 `.next`여야 한다 |

`next.config.ts`의 `outputFileTracingRoot`는 **유지한다.** 워크스페이스 루트 오인 재발 방지용이다.

### 7.4 렌더링 경계

- `page.tsx`에는 **`"use client"`**를 붙인다. 루트 `layout.tsx`만 서버 컴포넌트.
- `useSearchParams`를 쓰는 컴포넌트는 **`<Suspense>`로 감싼다.** 대상은 `/oauth/callback`과 `/todos` 둘. 누락하면 `npm run dev`는 통과하고 **`npm run build`에서 실패**한다.

### 7.5 `globals.css` 다크모드 전략 (Phase 6에서 정리 완료)

Phase 6에서 아래 두 가지를 걷어냈다. **다시 넣지 않는다.**

```css
@custom-variant dark (&:is(.dark *));   /* 제거됨 — class 전략 */
.dark { ... }                            /* → @media (prefers-color-scheme: dark) { :root { ... } } 로 대체됨 */
```

`@theme`은 **최상위에만** 둔다. `@media` 안에 중첩하지 않는다. 라이트 값을 `@theme`으로 선언해 유틸리티를 만들고, 다크에서는 `:root`에서 **값만 덮어쓴다.**

⚠️ **`@custom-variant dark`를 지우면 커스텀 토큰뿐 아니라 Tailwind의 `dark:` 유틸리티 전체가 media 전략으로 바뀐다.** shadcn이 생성한 컴포넌트(예: `button.tsx`의 `dark:bg-input/30`)도 예외가 아니다 — 별도 손질 없이 한 줄 삭제만으로 앱 전체의 다크 모드 체계가 일관되게 전환된다(CSSOM으로 실측 확인, 2026-09-02).

### 7.6 정화 적용 지점

이 앱에는 **`dangerouslySetInnerHTML`이 없다.** DOMPurify의 유일한 적용 지점은 다음이다.

```ts
editor.commands.setContent(sanitizeHtml(todo.content));   // setContent 직전
```

### 7.7 `useAuth`는 토큰 **존재**가 아니라 `exp`를 본다

`localStorage`에 문자열이 있는지만 검사하면 **만료 토큰이 인증 판정을 통과한다.**
그러면 `(main)` 레이아웃이 인증으로 판정 → **보호 화면 렌더** → API 호출 → 401 → 자동 로그아웃 순서가 되어,
그 왕복 동안 보호된 화면이 사용자에게 노출된다. `AUTH-07`("토큰이 없거나 **만료된** 상태") 위반이다.

```ts
// 서명 검증은 서버가 한다. 프론트는 만료 시각만 읽으면 된다. 라이브러리를 추가하지 않는다.
const { exp } = JSON.parse(atob(token.split(".")[1]));
```

- 만료로 판정되면 **즉시 토큰을 폐기**하고 미인증으로 처리한다. 파싱 실패도 만료로 취급한다.
- ⚠️ `JWT_EXPIRATION`이 24시간이라 **개발 중에는 재현되지 않는다.** 검증은 `exp`를 과거로 조작한 토큰으로 한다.

### 7.8 낙관적 업데이트 — 멱등성만으로는 부족하다

toggle을 목표 상태 전송(멱등)으로 설계한 것과, **요청 순서가 뒤바뀌어도 최종 상태가 같은 것**은 다른 성질이다.
React Query v5의 mutation은 기본적으로 **병렬 실행**되므로 연타 시 화면이 사용자 의도와 반대로 되돌아가는 깜빡임이 생긴다.

- 항목별 직렬화(`useMutation({ scope: { id: "todo-toggle-" + todoId } })`) **또는** `isMutating` 가드로 마지막 것만 재조회한다 (`CLAUDE.md` 9장).
- ⚠️ **마지막 항목 삭제로 페이지가 비어 이전 페이지로 이동할 때, 이동은 `onSuccess`에서 한다. `onMutate`가 아니다.**
  `onMutate`에서 옮기면 쿼리 키가 바뀌어, 실패 시 `onError`의 롤백이 **사용자가 보고 있지 않은 캐시에 적용된다** — `TODO-13` 위반이다.

### 7.9 이탈 확인(`TODO-16`)은 **3계층**이다 — 한 줄짜리 작업이 아니다

App Router에는 `router.events`도, 공식 내비게이션 차단 API도 **없다.** `beforeunload` 하나만 넣으면 **앱 내부 이동이 전혀 막히지 않는다.**

| 이탈 경로 | 방어 수단 |
|---|---|
| 새로고침 · 탭 닫기 · 주소창 이동 | `beforeunload` |
| 페이지 내 "취소"·"목록으로" 버튼 | **버튼 자체 핸들러**에서 확인 후 `router.push` |
| 브라우저 뒤로가기 | `popstate` 리스너 + 취소 시 `history.pushState`로 복원 |

- `next-navigation-guard` 등 **서드파티를 도입하지 않는다** (`CLAUDE.md` 3장 스택에 없다).
- ⚠️ **Tiptap `dirty` 판정의 초기 스냅샷은 `setContent()` 직후의 `editor.getHTML()`로 잡는다.**
  에디터가 HTML을 정규화하므로 서버가 준 문자열과 그대로 비교하면 **아무것도 고치지 않아도 dirty가 된다.**
- 저장 성공 직후 가드를 **반드시 해제**한다. 저장하고 나가는데 확인창이 뜨면 안 된다.

### 7.10 Phase 6에서 확정된 규칙

- **shadcn `style`을 바꾼 뒤 기존 컴포넌트는 수동 편집이 아니라 `npx shadcn add <name> --overwrite`로 재생성한다.** `components.json`의 `style` 값만 바꿔서는 이미 생성된 파일(`button.tsx` 등)의 실제 마크업이 그대로 남는다 — `radix-nova`와 `new-york`은 클래스 구조(`data-slot`·`group/button`·`oklch color-mix` 유무 등)가 달라 값만 바꿔서는 재현되지 않는다. CLI로 다시 받아야 실제로 전환된다.
- **`sonner`는 shadcn 레지스트리의 래퍼(`npx shadcn add sonner`)를 쓰지 않는다.** 그 래퍼는 `next-themes`에 의존하는데, 이 프로젝트는 다크 모드 토글이 없어(`CLAUDE.md` 8장) `next-themes`로 관리할 테마 상태 자체가 없다. `sonner` 패키지 자체의 `<Toaster theme="system" />`만으로 OS 설정을 따라가며, `CLAUDE.md` 3장이 확정한 스택에 라이브러리를 추가하지 않는다.
- **Next.js App Router는 `_`로 시작하는 폴더를 라우팅에서 제외한다(private folder 컨벤션).** 임시 검증 라우트를 `src/app/__verify/`처럼 만들면 실제로 페이지가 컴파일돼도 **404가 난다.** 임시 라우트가 필요하면 `_` 없는 이름(`dod-verify` 등)을 쓴다.
- **React Server Components 경계는 "이 파일에 `use client`가 있는가"가 아니라 "가장 가까운 클라이언트 경계 조상이 있는가"로 정해진다.** `Header.tsx`·`Button.tsx`처럼 `"use client"`가 없는 컴포넌트라도, 이들을 렌더하는 페이지가 클라이언트 컴포넌트면 트리 전체가 클라이언트 번들에 포함돼 이벤트 핸들러가 정상 동작한다. 반대로 서버 컴포넌트에서 이런 컴포넌트에 `onClick` 같은 함수 prop을 내려보내면 "Event handlers cannot be passed to Client Component props" 런타임 에러가 난다 — 이 프로젝트의 모든 실제 `page.tsx`는 `"use client"`이므로(7.4) 실전에서는 문제가 안 되지만, 검증용 임시 페이지를 만들 때 빠뜨리기 쉽다.
- **`globals.css`의 `@theme inline` 블록에서 `--font-sans: var(--font-sans)`처럼 자기참조로 두면 실제로는 아무 폰트도 지정하지 않은 것과 같다.** Pretendard를 `next/font/local`로 로드했어도 `--font-sans`가 로드한 변수(`var(--font-pretendard)`)를 가리키지 않으면 `font-sans` 유틸리티가 폴백 스택으로 조용히 떨어진다. 폰트를 바꿀 때는 로드 지점(`layout.tsx`의 `localFont({ variable })`)과 참조 지점(`globals.css`의 `--font-sans`)을 항상 함께 확인한다.
- **`localStorage`에 접근하는 함수는 전부 `typeof window === "undefined"` 가드를 넣는다.** `apiClient.ts`의 `getStoredToken`에는 가드가 있었지만 `setStoredToken`·`clearStoredToken`에는 빠져 있어, 브라우저 밖(Node 등)에서 401 처리 경로를 타면 `ReferenceError: window is not defined`로 죽었다. `tsc`는 `window`가 `dom` lib에 포함된 유효한 타입이라 이 결함을 잡지 못한다 — 반드시 실제 실행(Node의 네이티브 TS 실행 또는 브라우저)으로 검증한다.

### 7.11 Phase 7에서 확정된 규칙

- **`app/oauth/callback/`은 `(auth)` 그룹이 아니라 `src/app/` 바로 아래(최상위)에 둔다.** `CLAUDE.md` 2장 구조도가 정본이며, 계획 단계에서 생성된 태스크 지침 텍스트가 이와 다르게(`(auth)/oauth/callback`) 적혀 있어도 지침 쪽이 오타다. **지침 텍스트와 구조도가 갈리면 구조도를 따른다** — 이 프로젝트에 `(auth)` 전용 레이아웃이 없어 실질적 라우팅 차이는 없지만, 저장소 구조 일관성을 위해 위치를 지킨다.
- **`useAuth().login`이 호출할 엔드포인트를 계획 텍스트만 보고 옮기지 않는다.** 태스크 계획 단계의 지침이 "`login`이 `/auth/signup`을, `signup`이 `/auth/login`을 호출한다"처럼 **엔드포인트가 서로 뒤바뀐 채** 적혀 있었던 사례가 있다. 두 함수 모두 `TokenResponse`를 반환하는 동일한 타입 시그니처라 `tsc`가 잡지 못한다 — **항상 백엔드 실소스(`AuthController.java`)와 대조한다.**
- **`/oauth/callback`처럼 `router.replace()`로 쿼리를 지운 뒤 `router.push()`로 다시 이동하는 페이지는, 그 사이 `useSearchParams()` 변경으로 effect가 재실행될 수 있음을 전제로 설계한다.** `replace`가 URL을 바꾸면 컴포넌트가 새 `searchParams`로 리렌더되어 effect가 다시 돈다 — 이때 `token`이 이미 지워진 상태라 "토큰 없음 → `/login`"으로 잘못 튕기지 않도록, `useRef` 가드로 effect 본문이 실질적으로 한 번만 실행되게 한다.
- **`useState` + 수동 검증 폼에서 `touched`(터치 여부)는 `onBlur`가 아니라 `onChange` 최초 호출에서 켠다.** 실시간 검증(`PRD.md` 5.3)의 취지가 "타이핑하는 동안 즉시 안내"이므로, blur 기반으로 하면 사용자가 다른 필드로 포커스를 옮기기 전까지 안내가 뜨지 않는다. 첫 렌더(빈 칸)에는 에러를 숨기는 목적은 "아직 한 번도 입력하지 않은 필드"로 판정해도 동일하게 달성된다.
- **`npm run build` 실행 직후 `npm run dev` 서버가 응답 불능 상태(포트는 LISTENING인데 요청이 타임아웃)가 되는 현상이 이 환경에서 반복 재현된다** (Phase 7에서 2회, Phase 8에서 1회 재현). 원인은 특정하지 못했다(Turbopack 캐시 충돌로 추정되나 미확인). **`npm run build`를 돌린 뒤에는 항상 `netstat`으로 기존 dev 서버 PID를 확인해 `Stop-Process -Force`로 종료하고 재기동한다** — 재시작 없이 이어서 브라우저 검증을 시도하면 모든 요청이 타임아웃돼 원인 파악에 시간을 허비한다.

### 7.12 Phase 8에서 확정된 규칙

- **`useState(initialValue)`는 마운트 시 1회만 평가된다.** `TodoForm`처럼 "로딩 중(undefined) → 데이터 도착" 순서로 prop이 바뀌는 컴포넌트에서 초기값을 `useState`로 잡으면, 데이터가 나중에 도착해도 이미 확정된 초기 렌더 값(빈 폼)이 리셋되지 않는다. 부모(`app/(main)/todos/[id]/page.tsx`)가 **`key={data ? "loaded" : "loading"}`로 `key`를 바꿔 강제 리마운트**시켜야 해결된다 — `useEffect`로 값을 동기화하는 방식은 dirty 판정용 기준선(ref)까지 함께 어긋나므로 채택하지 않았다.
- **Tiptap 툴바 버튼은 `onMouseDown={(e) => e.preventDefault()}`를 반드시 건다.** 버튼 클릭이 브라우저 기본 포커스 이동을 일으켜 에디터 selection이 풀리고, 그 직후 입력이 선택돼 있던 텍스트를 지워버린다(ProseMirror 계열 에디터의 표준 관행). `onClick`에만 로직을 넣고 이 처리를 빠뜨리면 툴바 버튼을 누른 뒤 첫 타이핑에서만 재현되는, 원인을 찾기 어려운 버그가 된다.
- **`TodoForm`의 `onSubmit`은 `Promise<void>`가 아니라 `Promise<boolean>`이어야 한다.** 페이지 핸들러(`new`/`[id]`)가 에러를 잡아 `submitError` state로만 표시하고 예외를 다시 던지지 않으므로(폼 입력을 유지해야 하는 `TODO-13` 요구 때문에 reject 방식을 쓸 수 없다), `TodoForm`이 저장 성공 여부를 알 방법은 반환값뿐이다. 성공(`true`)일 때만 dirty 기준선(ref)을 갱신한다 — 실패 시 갱신하면 에러가 난 상태에서도 이탈 확인이 꺼져버린다.
- **에러 상태 UI를 자동화로 재현할 때 React Query 기본 `retry:3`을 감안한다.** `new QueryClient()`를 옵션 없이 쓰면 쿼리가 실패해도 최대 3회까지 자동 재시도하므로, 네트워크 요청을 1회만 실패시키는 테스트는 사용자에게 에러가 보이지 않고 조용히 성공한다(이것 자체는 견고함의 증거이지 버그가 아니다). `ErrorState` 렌더를 실제로 확인하려면 최소 4회 연속 실패를 유도해야 한다.

---

## 8. Git 규칙

- **커밋 전 반드시 사용자 승인을 받는다.**
- 커밋 메시지는 **한글**, prefix는 `feat: / fix: / refactor: / test: / docs: / chore:`
- 브랜치: `main`(배포 가능) ← `develop` ← `feature/{작업명}`
  - 세 저장소 모두 `main` + `develop`이 서 있고 원격 추적까지 설정됐다. **이제부터 `main`·`develop`에 직접 커밋하지 않고 `feature/{작업명}`을 판다.**
- 저장소별로 **개별 커밋**한다.
- 첫 커밋 전 `git add -An .`로 대상 파일 수를 확인한다. **문서 저장소의 예상값은 21건**이다. 수백~수천 건이면 무시 규칙이 빠진 것이다.

---

## 9. AI 판단 기준 (결정 트리)

### 9.1 라이브러리 버전을 정해야 할 때

```
CLAUDE.md 3장 「버전 관련 확정 사항」에 있는가?
├─ 있다  → 그대로 쓴다. 재조사 금지. 다른 버전으로 바꾸지 않는다.
└─ 없다  → 3장 스택 표에 있는가?
           ├─ 있다 → 그 스택을 쓰고, 정확한 버전을 확인해 명시한다.
           └─ 없다 → 임의로 도입하지 않는다. 사용자에게 제안하고 대기한다.
```

### 9.2 스펙에 없는 기능이 필요해 보일 때

```
구현하지 않는다
  └→ 사용자에게 제안한다
       └→ 승인되면 PRD 3장 + ROADMAP 추적표에 먼저 반영한 뒤 구현한다.
```

### 9.3 문서끼리 어긋날 때

```
CLAUDE.md 를 따른다
  └→ 어긋난 문서(PRD / ROADMAP / guides / 이 문서)를 고친다
       └→ 버전·최종 수정일을 올린다
```

### 9.4 요구사항 해석이 갈릴 때

```
서로 다른 해석이 서로 다른 산출물을 만드는가?
├─ 예     → 사용자에게 질문한다. 임의로 진행하지 않는다.
└─ 아니오 → 판단해서 진행하고, 가정한 내용을 보고에 명시한다.
```

### 9.5 어느 Phase의 일인지 애매할 때

`docs/ROADMAP.md`의 **「요구사항 ↔ Phase 추적표」**를 본다. 표에 없으면 9.2로 간다.

---

## 10. 절대 금지

1. **Docker 사용** — 범위 밖이다. 따라서 **Testcontainers도 금지**다. 테스트 DB는 로컬 PostgreSQL `todolist_test`를 쓴다.
2. **테스트 DB로 H2 사용** — `LOWER(...) LIKE` 등 PostgreSQL 문법을 쓰므로 동작이 갈린다.
3. **`@DataJpaTest`에 `@AutoConfigureTestDatabase(replace = NONE)` + `@ActiveProfiles("test")` 누락** — 임베디드 DB로 교체를 시도한다.
4. **`@EnableJpaAuditing`을 `@Configuration` 클래스에 배치** — `@DataJpaTest`가 로드하지 않아 `created_at`이 null이 된다. 메인 애플리케이션 클래스에 붙인다.
5. **`TodoUpdateRequest`에 `completed` 포함** — 완료 상태는 오직 `PATCH /toggle`로만 바꾼다.
6. **toggle을 서버에서 뒤집기(`completed = !completed`)** — 바디로 받은 목표 상태를 그대로 저장한다.
7. **Spring `Page` 객체를 그대로 반환** — `PageResponse<T>`로 변환해 `ApiResponse.data` 안에 담는다.
8. **`SecurityFilterChain`에서 CSRF 비활성화·STATELESS 세션 누락** — `POST /auth/signup`부터 403으로 막힌다.
9. **`authorizeRequests()` 사용** — 제거된 API다. `authorizeHttpRequests()`를 쓴다.
10. **`WebSecurityConfigurerAdapter` 사용**
11. **`permitAll`에서 Swagger 경로 누락** — Phase 1 DoD가 조용히 회귀한다.
12. **Security 필터 단계 401/403을 `GlobalExceptionHandler`에 맡기기** — 컨트롤러에 도달하지 않는다. `AuthenticationEntryPoint` + `AccessDeniedHandler`를 별도 구현한다.
13. **JWT Secret 하드코딩**, **DB 비밀번호 하드코딩**
14. **`JWT_SECRET`을 Base64로 디코드해서 사용** — raw UTF-8 32자 이상을 그대로 키로 쓴다.
15. **`SerializationFeature.WRITE_DATES_AS_TIMESTAMPS` 참조** — Jackson 3에서 이동했다. 컴파일에 실패한다. 무설정으로 둔다.
16. **소유권 불일치를 403으로 응답** — **404**를 반환한다(존재 여부 노출 방지).
17. **`TodoResponse`에 사용자 정보 포함** — 목록에서 N+1이 난다.
18. **정규식으로 HTML 정화** — Jsoup Safelist / DOMPurify를 쓴다.
19. **`Jsoup.clean` 호출 시 `OutputSettings.prettyPrint(false)` 누락** — `pre` 블록의 공백·줄바꿈이 망가진다.
20. **`hibernate.jdbc.time_zone: UTC` 누락** — 로컬은 KST(+09:00), RDS는 UTC다. 설정하지 않으면 **시각이 9시간 어긋난 채 배포 후에야 드러난다.** 저장·비교는 전부 UTC, 표시만 브라우저 로컬(`date-fns`). `due_date`는 `LocalDate`라 무관하고 `created_at`·`updated_at`·`deleted_at`이 대상이다.
21. **`sort` 쿼리 파라미터를 `Pageable`에 그대로 전달** — 허용 필드는 **`createdAt`·`dueDate` 둘뿐**이며, 그 외 값은 **기본값(`createdAt,desc`)으로 대체**한다. 없는 프로퍼티가 들어가면 500이 난다.
22. **한 번에 전체 Phase 생성** — Phase 단위로 만들고 보고 후 멈춘다.
23. **`.metadata/` 편집** — Eclipse 워크스페이스다. 무시 대상이지 작업 대상이 아니다.

---

## 11. 도구·환경 주의사항

| 항목 | 상태 | 지침 |
|---|---|---|
| **shrimp-task-manager `DATA_DIR`** | `D:\claude\todo-project\shrimp_data` (2026-08-28 정정) | 이 폴더는 `.gitignore` 대상이다. 커밋하지 않는다. **`.mcp.json` 변경은 MCP 서버 재연결 후에 적용된다** |
| **이전 태스크 데이터** | 정정 전 `D:\claude\nextjs-supabase-app\shrimp_data`에 쌓였을 수 있다 | 이 프로젝트의 태스크가 거기 있다면 옮기거나 새로 만든다. 자동으로 이전되지 않는다 |
| **`nextjs-supabase-fullstack-developer` 에이전트** | `.claude/agents/`에 있으나 **Supabase는 이 프로젝트 스택에 없다** | 사용하지 않는다 |
| **`docs/guides/`** | 2026-08-28에 이 프로젝트 기준으로 전면 재작성됨 | 참고 자료로만 쓴다. 충돌 시 `CLAUDE.md` |
| **개발 OS** | Windows 10 | POSIX 명령은 Git Bash 기준이다. PowerShell에서는 6.5의 Windows 명령을 쓴다 |
| **PostgreSQL CLI** | `createdb`가 PATH에 없을 수 있다 | `& "C:\Program Files\PostgreSQL\17\bin\createdb" -U postgres todolist_db` · `-U postgres` 누락 금지 |
| **JDK** | 여러 개 설치된 환경일 수 있다 | `JAVA_HOME`과 PATH가 모두 21을 가리키는지 확인한다 |
| **`git check-ignore -v`** | **`-v`는 부정 규칙(`!`)이 매칭돼도 그 줄을 출력하고 exit 0을 반환한다** | 무시 여부 판정에는 **`-v`를 빼고** `git check-ignore .env.example`을 쓴다(exit 1이면 추적 가능). `ROADMAP.md` **v1.9**에서 Phase 0·1 DoD를 이 형태로 정정했다 |
| **`claude-in-chrome`의 `resize_window`** | 이 실행 환경(추정 원격/가상 디스플레이, devicePixelRatio 0.9)에서 실제 `window.innerWidth`를 바꾸지 못한다(360/233/150 요청 모두 555px 고정, Phase 8에서 확인) | 360px 등 특정 브레이크포인트 정밀 검증이 필요하면 **현재 폭에서의 실측(가로 스크롤 여부 등) + 반응형 분기 코드를 근거로 대체**하고, DoD 문서에 한계를 그대로 기록한다. 조용히 통과시키지 않는다 |

---

## 12. 이 문서의 갱신

다음 경우 **이 문서를 함께 고친다.**

- 3절 스냅샷과 실제 저장소 상태가 달라졌을 때 (특히 Phase 완료·부채 해소 시 3.1 표의 행을 지운다)
  > ⚠️ **행을 지우기 전에, 그 행이 「사실」인지 「규칙」인지 가른다.** "지금 어긋나 있다"는 사실은 지워도 되지만,
  > "이렇게 해야 한다"는 판단(예: 3.1 #3의 DB 분리, #7의 다크 모드 전략)은 **6·7절의 영속 규칙으로 먼저 옮긴 뒤** 지운다.
  > 옮기지 않고 지우면 근거가 사라져 나중에 조용히 되돌아간다.
- 새로운 다중 파일 연동 관계가 생겼을 때 (5절에 항목 추가)
- `CLAUDE.md`가 개정되어 이 문서의 규칙과 어긋날 때
- 새 금지 사항이 확정됐을 때 (10절에 추가)

**하지 않을 것**: `CLAUDE.md`·`PRD.md`·`ROADMAP.md`의 내용을 이 문서에 복사하는 것. 복사하면 두 곳이 갈라지고, 갈라진 순간 이 문서는 신뢰할 수 없어진다. **참조하고 링크한다.**
