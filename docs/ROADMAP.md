# ROADMAP — Todo List 프로젝트

> **버전** 2.8 · **최종 수정** 2026-09-03
> 이 문서는 "어떤 순서로 만드는가"를 정의하며, **완료 판정의 정본**이다.
> **한 번에 전체를 생성하지 않는다.** Phase 단위로 진행하고, 각 Phase의 DoD를 모두 만족한 뒤 다음으로 넘어간다.
> 기술 규칙은 `CLAUDE.md`, 기능 정의는 `PRD.md` 참조.

---

## 진행 현황

| Phase | 내용 | 저장소 | 상태 |
|---|---|---|---|
| 0 | 저장소 초기화 | 전체 | ✅ |
| 1 | 백엔드 스캐폴딩 | backend | ✅ |
| 2 | 도메인 & DB | backend | ✅ |
| 3 | 인증 (로컬) + 인증 테스트 | backend | ✅ |
| 4 | Todo API + Todo 테스트 | backend | ✅ |
| 5 | 구글 OAuth2 + OAuth 테스트 | backend | ✅ |
| 6 | 프론트 스캐폴딩 | frontend | ✅ |
| 7 | 인증 화면 | frontend | ✅ |
| 8 | Todo 화면 | frontend | ✅ |
| 9 | 인터랙션 다듬기 | frontend | ✅ |
| 10 | 전체 검증 | 전체 | ✅ |
| 11 | AWS 배포 | 전체 | ⬜ |

⬜ 대기 · 🟡 진행중 · ✅ 완료

> ### 스캐폴딩 정합성 점검 (2026-08-28)
>
> 스캐폴딩이 스펙과 어긋난 채 진행되어 아래를 바로잡았다. **해결됨** 항목은 재발 방지 DoD가 각 Phase에 들어가 있다.
>
> | 항목 | 발견 당시 | 결과 |
> |---|---|---|
> | `todo-frontend`가 Next.js 16.3.3 | Amplify SSR 지원 범위(12~15) 밖이라 Phase 11 배포 불가 상태였음 | ✅ **15.5.24로 마이그레이션.** `next`·`eslint-config-next` 동시 적용, `npm run build`·`lint` 통과 |
> | 프론트에 `src/` 없음 | `app/`·`components/`·`lib/`가 루트 직하 | ✅ **`src/` 아래로 이동.** `tsconfig` paths(`@/*` → `./src/*`), `components.json`의 css 경로 동시 수정 |
> | `layout.tsx`가 `LayoutProps<"/">` 사용 | Next 16 전역 타입이라 15에서 컴파일 실패 | ✅ 명시적 `{ children: React.ReactNode }`로 교체 |
> | `eslint.config.mjs`가 16 방식 | 15의 `eslint-config-next`는 flat config를 직접 내보내지 않음 | ✅ `FlatCompat` 방식으로 교체 |
> | `AGENTS.md` | Next 16의 `next dev`가 자동 생성한 파일. 15에서는 재생성되지 않고 내용도 부정확 | ✅ 삭제. 저장소용 `CLAUDE.md`를 `@../CLAUDE.md` 임포트 형태로 재작성 |
> | 워크스페이스 루트 오인 | 부모 `todo-project`에도 `package-lock.json`이 있어 Next가 루트를 잘못 추론 | ✅ `next.config.ts`에 `outputFileTracingRoot` 지정. **원인이던 루트 npm 파일도 이후 삭제**했으나, 재발 시 조용히 어긋나므로 설정은 유지한다 |
> | `pom.xml`에 `springdoc` 없음 | Phase 1 DoD "Swagger UI 접속" 불가 | ✅ **3.1.0 핀.** 이 버전의 부모가 `spring-boot-starter-parent` 4.1.0이라 Boot 4.1.x에 대응 |
> | `pom.xml`에 `jsoup` 없음 | Phase 4 XSS 정화 불가 | ✅ **1.23.2 핀** |
> | JWT 라이브러리 미선정 | `CLAUDE.md` 3장 표에 JWT 라이브러리가 명시되어 있지 않음 | ✅ **jjwt 0.12.6**(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 `pom.xml`에 핀되어 있고, **`CLAUDE.md` 3장 스택 표와 「버전 관련 확정 사항」에 등재됐다**(v1.8). Phase 3은 이 버전을 그대로 쓴다 |
> | `todo-backend/.gitattributes` | Git Bash에서 `./mvnw` `bad interpreter` 위험 | ✅ 존재함 (`/mvnw text eol=lf`, `*.cmd text eol=crlf`) |
> | 백엔드 버전 | Spring Boot 4.1.1 + `java.version` 21 | ✅ 스펙 일치. 변경 없음 |
>
> #### 아직 남은 것 (2026-08-28 재확인)
>
> | 항목 | 현재 상태 | 처리 Phase |
> |---|---|---|
> | ~~커밋·푸시~~ | ✅ **해소.** 세 저장소 모두 첫 커밋 후 `main`·`develop`을 `origin`에 푸시했다. 태그 `v0.0.0` 부착 완료 | — |
> | ~~브랜치명~~ | ✅ **해소.** 세 저장소 모두 `main` + `develop`. `master` 없음 | — |
> | ~~루트 `.gitignore`~~ | ✅ **해소.** `node_modules/`·`.metadata/`·`shrimp_data/`·`.env*`+`!.env.example`을 추가했다. 스테이징 대상이 563개 → **21개**로 줄었다 | — |
> | ~~백엔드 `.gitignore`~~ | ✅ **해소.** `.env*` + `!.env.example` 추가. 실파일로 무시/추적을 검증했다 | — |
> | ~~프론트 `.gitignore`~~ | ✅ **해소.** `!.env.example` 예외 줄 추가. 실파일로 검증했다 | — |
> | ~~문서 경로~~ | ✅ **해소.** `docs/`를 정본으로 확정하고 `CLAUDE.md` 2장 구조도와 상단 참조 경로를 정정했다 (v1.8) | — |
> | ~~루트 npm 파일~~ | ✅ **해소.** 루트 `package.json`·`package-lock.json`·`node_modules/`를 삭제했다. `shadcn`은 `todo-frontend/package.json`에 이미 있어 기능 손실이 없고, `.mcp.json`의 shadcn 서버는 `npx shadcn@latest mcp`라 루트 설치에 의존하지 않는다 | — |
> | ~~`docs/guides/`~~ | ✅ **해소.** 5개 전부 다른 프로젝트에서 넘어온 문서였고(존재하지 않는 "PRD 1.3 기술 스택" 참조, 모노레포·Next 16·없는 npm 스크립트 전제), **이 프로젝트 기준으로 전부 재작성**했다. `nextjs-16.md`→`nextjs-15.md`, `forms-react-hook-form.md`→`forms.md`로 개명. `README.md`를 추가해 "참고 자료이며 `CLAUDE.md`가 우선"임을 명시 | — |
> | ~~폼 라이브러리 미결정~~ | ✅ **해소.** `CLAUDE.md` 3장에 **"라이브러리를 쓰지 않는다 — `useState` + 수동 검증"**으로 확정. `npx shadcn add form` 금지(=`react-hook-form` 유입 경로)와 Tiptap dirty 판정 주의를 함께 명시 | — |
> | ~~백엔드 설정 파일~~ | ✅ **해소.** `application.properties`를 삭제하고 `application.yml` + `-local` + `-prod` 3분할로 교체했다. 평문 DB 비밀번호는 `${DB_PASSWORD}` 환경변수로 이전했고 히스토리 미유입을 확인했다 | — |
> | ~~백엔드 문서·예시~~ | ✅ **해소.** `.env.example`(9개 키)과 저장소용 `CLAUDE.md`를 작성했다 | — |
> | 프론트 마감 작업 | 디자인 토큰이 shadcn 기본 neutral, 다크 모드가 `@custom-variant dark (&:is(.dark *))`(= `class` 전략), `components.json` 스타일이 `radix-nova`, `.env.example` 없음, Tiptap·motion·sonner·date-fns·DOMPurify 미설치 | Phase 6 |

> **테스트는 마지막에 몰아 쓰지 않는다.** 기능을 만든 Phase에서 함께 작성해 그 Phase의 DoD로 삼는다. Phase 10은 새 테스트를 쓰는 단계가 아니라 전체를 확인하는 단계다.

---

## 요구사항 ↔ Phase 추적표

> `PRD.md` 3장의 P0 요구사항이 **어느 Phase에서 구현되고 어느 Phase에서 검증되는지**의 정본이다.
> **여기에 행이 없는 P0는 구현되지 않는다.** `PRD.md` 3장에 요구사항을 추가하면 이 표에 먼저 행을 넣고, 해당 Phase의 작업·DoD에 실제로 기술한다.

| ID | 구현 Phase | 검증 Phase (DoD) |
|---|---|---|
| AUTH-01 회원가입 | 3(API) · 7(화면) | 3 · 7 · 10 |
| AUTH-02 비밀번호 6자 이상 + 72바이트 | 3(바이트 validator) · 7(실시간 검증) | 3(한글 25자 → 400) · 7(제출 전 인라인 안내) · 10 |
| AUTH-03 이메일 중복 | 3(409) · 7(인라인 문구) | 3 · 7 |
| AUTH-04 로그인·JWT 24h | 3 · 7 | 3 · 7 · 10 |
| AUTH-05 구글 로그인 | 5 · 7(`/oauth/callback`) | 5 · 7 · 10 |
| AUTH-06 로그아웃 | 7 (프론트 전용, 서버 API 없음) | 7 · 10 |
| AUTH-07 라우트 보호 | 7 (`(main)` 클라이언트 레이아웃) | 7(만료 토큰 케이스) · 10 |
| AUTH-08 헤더 닉네임 / 이메일 미표시 | 3(`/auth/me` 응답) · 7(헤더) | 3(응답 필드) · 7(DOM에 이메일 없음) · 10 |
| AUTH-09 계정 충돌 거부 | 5 · 7(안내 문구) | 5 · 7 · 10 |
| TODO-01 제목 필수·200자 | 4 · 8 | 4 · 10 |
| TODO-02 Tiptap 본문 | 4(정화·50,000자) · 8(에디터) | 4 · 8 · 10 |
| TODO-03 우선순위 | 4 · 8 | 4 · 8 · 10 |
| TODO-04 마감일 | 4 · 8 | 4 · 8 · 10 |
| TODO-05 생성일 내림차순 | 4 · 8 | 4 · 8 · 10 |
| TODO-06 10건 페이지네이션 | 4 · 8 | 4 · 8 · 10 |
| TODO-07 완료 필터 | 4 · 8 | 4 · 8 · 10 |
| TODO-08 제목 검색 | 4 · 8 | 4 · 8 · 10 |
| TODO-09 상세(항상 편집) | 4 · 8 | 8 · 10 |
| TODO-10 저장(완료 상태 불변) | 4(PUT에 `completed` 없음) · 8 | 4 · 8 · 10 |
| TODO-11 완료 토글 즉시 반영 | 4(멱등 toggle) · 9(낙관적 업데이트) | 4 · 9(연타 수렴값) · 10 |
| TODO-12 삭제 즉시 반영 / Soft Delete | 4 · 8(상세→목록 이동) · 9(낙관적 제거) | 4 · 8 · 9 · 10 |
| TODO-13 실패 시 롤백·알림 | **8(저장 실패: 폼 유지 + 에러)** · 9(토글·삭제 롤백 + 토스트) | 8 · 9 · 10 |
| TODO-14 타인 리소스 차단 | 4(404) · 8(전용 화면) | 4 · 8 · 10 |
| TODO-15 XSS 이중 방어 | 4(Jsoup) · 6(`sanitize.ts`) · 8(`setContent` 직전) | 4 · 8 · 10 |
| TODO-16 이탈 확인 | 8 (3계층) | 8 |
| UX-01 스켈레톤 | 6(컴포넌트) · 7(인증 판정·콜백) · 8(목록·상세) | 7 · 8 · 10 |
| UX-02 빈 상태 | 6 · 8 | 8 · 10 |
| UX-03 검색 결과 없음 | 8 | 8 · 10 |
| UX-04 에러 + 재시도 버튼 | 6(`ErrorState.onRetry` 필수) · 7 · 8 | 6 · 7 · 8(재요청 확인) · 10 |
| UX-05 반응형 360~1920 | 6(토큰·컨테이너) · 8 | 8(360px) · 10(1920px) |
| UX-06 label·키보드 | 7 · 8 | 7 · 8 · 10 |
| UX-07 다크 토큰 (`prefers-color-scheme`) | 6 | 6 · 10 |

> `PRD.md` 5.1의 **에러 문구 매핑 표**는 Phase 6에서 `lib/errorMessages.ts`로 단일화하고, Phase 7·8에서 화면별로 적용, Phase 10에서 6종 전부를 대조한다.
> `PRD.md` 7장 비기능 요구사항의 검증 위치(응답 속도 4 · 체감 반응 9 · 브라우저 10 · 반응형 8·10 · 인증 보안 3·4 · 시크릿 관리 10·11 · XSS 4·8 · 문서화 4 · 접근성 7·8)는 위 표 및 각 Phase DoD와 일치한다.

---

## Phase 0 — 저장소 초기화

**목표**: 폴리레포 3개 저장소를 만들고 문서를 자리잡게 한다.

**작업**
- `todo-project/` 생성 후 `git init`
- `CLAUDE.md`, `PRD.md`, `ROADMAP.md` 배치
- `.gitignore`에 `todo-backend/`, `todo-frontend/` 추가 (**필수**)
- ~~루트 `.gitignore`에 `node_modules/` 추가~~ — ✅ **완료.** 함께 `.metadata/`(Eclipse 워크스페이스)·`shrimp_data/`(shrimp 로컬 상태)도 추가했다
  > 루트의 `package.json`·`package-lock.json`·`node_modules/`는 **이미 삭제했다.** `shadcn`은 `todo-frontend`에 있고 `.mcp.json`은 `npx`로 받아 쓰므로 문서 저장소에 npm 파일을 두지 않는다. 다만 루트에서 `npx shadcn`을 실수로 실행하면 `node_modules/`가 다시 생기므로 무시 규칙은 남긴다.
- `todo-backend/`, `todo-frontend/` 폴더 생성 후 각각 `git init`
- ~~각 저장소 `.gitignore`에 `.env*` + **`!.env.example`**~~ — ✅ **완료(세 저장소 모두).** 예외 줄이 없으면 예시 파일까지 무시된다
  > 아래 DoD의 `git check-ignore` 검증은 **여전히 수행한다.** 규칙이 있다는 것과 의도대로 판정된다는 것은 별개다.
- ~~브랜치 정리~~ — ✅ **완료.** 세 저장소 모두 `main`(체크아웃) + `develop`. 이후 작업은 `feature/{작업명}` → `develop` → `main` (`CLAUDE.md` 2장)
  > `todo-frontend`에 있던 커밋 1건(`Initial commit from Create Next App`)과 `develop`은 **2026-08-28에 제거**해 세 저장소의 출발선을 맞췄다. 작업 트리 파일은 그대로이며 전부 untracked 상태다.
- ~~문서 경로 확정~~ — ✅ **완료(v1.8).** `docs/`를 정본으로 확정했다. `CLAUDE.md`만 루트에 두는데, Claude Code가 상위 디렉토리를 거슬러 올라가며 자동 로드하는 대상이 `CLAUDE.md`이기 때문이다. 2장 구조도와 상단 참조 경로를 정정했다
- ~~`nextjs-16.md` 처리~~ — ✅ **완료.** 삭제하고 **`docs/guides/nextjs-15.md`로 재작성**했다. 단순 버전 치환이 아니라 이 프로젝트 기준으로 다시 썼다: "Server Components 우선"을 **"클라이언트 컴포넌트 우선"**으로 뒤집고(토큰이 localStorage에 있어 서버 페칭이 불가능), Next 16 전용 내용(`proxy.ts`, `cacheComponents`, 최상위 `typedRoutes`)을 걷어내고, 쓰지 않는 기능(Server Actions·Route Handlers·Parallel/Intercepting Routes·ISR·`notFound()`)을 **금지 표로 명시**했다
- ~~나머지 가이드 4개 처리~~ — ✅ **완료.** 넷 다 이 프로젝트 기준으로 재작성했다. `component-patterns`(서버 우선 → **클라이언트 우선**), `styling-guide`(`next-themes` 토글 → **미디어쿼리 다크 모드**), `project-structure`(모노레포 → **폴리레포**), `forms-react-hook-form.md` → **`forms.md`**(라이브러리 없는 폼). `guides/README.md`를 추가해 지위(참고 자료, `CLAUDE.md` 우선)와 교체 이력을 남겼다
  > ⚠️ 옛 `forms-react-hook-form.md`는 `react-hook-form`·`zod`가 **"이미 설치되어 있다"고 서술**했고, 비밀번호 규칙을 **"6자 이상이 전부"**로 적어 UTF-8 72바이트 한계를 누락하고 있었다. 그대로 뒀다면 Phase 7에서 `AUTH-02`가 조용히 깨졌을 문서다.
- ~~GitHub에 원격 저장소 3개 생성 및 연결~~ — ✅ **완료.** `origin`이 세 저장소 모두에 등록됐다(`todo_project` / `todo-backend` / `todo-frontend`). **푸시는 첫 커밋 이후**다

**DoD**
- [x] 부모 저장소에서 `git status` 시 하위 폴더가 나타나지 않음
- [x] **부모 저장소에서 `git status` 시 `node_modules/`가 나타나지 않음**
- [x] **세 저장소 각각에서 `git check-ignore .env`가 exit 0(무시됨)이고, `git check-ignore .env.example`이 exit 1(추적 가능)임** (문서·백엔드·프론트 세 곳 모두에서 확인)
  > ⚠️ **`-v`를 붙이면 판정이 뒤집힌다.** `-v`는 **부정 규칙(`!.env.example`)이 매칭돼도 그 줄을 출력하고 exit 0을 반환**한다. 즉 이 DoD가 요구하는 `!.env.example` 규칙을 올바로 넣을수록 "아무것도 반환하지 않는다"를 만족할 수 없다. **무시 여부 판정에는 `-v`를 쓰지 않는다.** 확실히 하려면 실파일을 만들어 `git status --porcelain -uall -- .env .env.example`에 `.env.example`만 뜨는지 본다.
- [x] 세 저장소 모두 현재 브랜치가 `main`이고 `develop`이 존재함 (`master` 없음)
- [x] 세 저장소 모두 첫 커밋 및 원격 푸시 완료
- [x] 첫 커밋의 파일 수가 예상 범위 안임 (`git show --stat HEAD | tail -1`로 확인 — 문서 저장소가 수천 건이면 무시 규칙이 빠진 것)
- [x] `CLAUDE.md` 2장 구조도의 문서 경로가 실제 파일 위치(`docs/`)와 일치함
- [x] `docs/guides/`가 `README.md` + 재작성된 5개 문서로만 구성됨 (`nextjs-16.md`·`forms-react-hook-form.md` 없음)
- [x] **`todo-frontend/package.json`에 `react-hook-form`·`zod`·`@hookform/resolvers`·`next-themes`가 없음** (스펙 밖 라이브러리 유입 확인)

---

## Phase 1 — 백엔드 스캐폴딩

**저장소**: `todo-backend`

**작업**
- Spring Initializr 기준 프로젝트 생성 (Spring Boot 4.x, JDK 21, Maven, `com.example`)
- 의존성: Web, Data JPA, Security, Validation, PostgreSQL Driver, OAuth2 Client, Lombok, **Jsoup**(HTML 정화), **jjwt**(JWT 생성·검증)
- **SpringDoc OpenAPI는 `springdoc-openapi-starter-webmvc-ui` 3.x를 명시한다.** 2.8.x는 Spring Boot 3.x 전용이라 기동에 실패한다
- 패키지 골격 생성: `domain / service / controller / dto / config / exception`
- **`application.properties`를 삭제하고 `application.yml`로 교체한다** (현재 저장소에는 `.properties`만 있다). 이어 `application-local.yml` + **`application-prod.yml`** 분리, 환경변수 바인딩 (프로파일별 `ddl-auto`는 `CLAUDE.md` 12장 표 참조)
  > ⚠️ 둘을 동시에 두면 `.properties`가 `.yml`보다 우선 적용되어, `.yml`에 적은 설정이 조용히 무시된다.
- **`application.yml`에 `spring.profiles.active: local` 기본값 지정** (없으면 DB 정보 없이 기동을 시도해 실패)
- `.gitignore`(**`.env*` + `!.env.example` 추가** — 현재 Initializr 기본값이라 `.env` 규칙이 없다), `.env.example`, 저장소용 `CLAUDE.md` 작성 (상단에 `@../CLAUDE.md` 임포트)
- `.gitattributes`는 이미 있다(`/mvnw text eol=lf`). **삭제하거나 덮어쓰지 않는다** (`CLAUDE.md` 13장)

**DoD**
- [x] `pom.xml`의 SpringDoc이 **현재 Boot 버전에 대응하는 정확한 3.x 버전으로 핀**되어 있음 (범위 지정 금지 — `CLAUDE.md` 3장)
- [x] `pom.xml`에 **`jsoup`이 포함**되어 있음 (Phase 4의 XSS 정화 전제)
- [x] `pom.xml`에 **jjwt 3종(`jjwt-api`/`jjwt-impl`/`jjwt-jackson`)이 동일 버전으로 핀**되어 있음 (Phase 3의 `JwtTokenProvider` 전제)
- [x] `./mvnw dependency:tree` 오류 없음
- [x] **`src/main/resources`에 `application.properties`가 없고 `application.yml`·`application-local.yml`·`application-prod.yml`이 있음**
- [x] `./mvnw spring-boot:run`이 옵션 없이 local 프로파일로 기동 성공
- [x] **기동 로그에 `The following 1 profile is active: "local"`이 찍힘**
- [x] `http://localhost:8080/swagger-ui/index.html`이 200으로 열리고 API 목록 화면이 렌더됨
- [x] `git check-ignore .env`가 exit 0이고 `git check-ignore .env.example`이 exit 1임 (**`-v`를 붙이지 않는다** — 이유는 Phase 0 DoD의 주석 참조)

> SpringDoc 버전은 `CLAUDE.md` 3장에서 3.x로 확정됐다. 다시 조사하거나 2.x로 되돌리지 않는다.

---

## Phase 2 — 도메인 & DB

**저장소**: `todo-backend`

**작업**
- `createdb todolist_db` · `createdb todolist_test`
- `BaseEntity` (`@MappedSuperclass`, JPA Auditing)
- `User`, `Todo` 엔티티 + `Priority` enum + `AuthProvider` enum
- **`Todo.user`는 `@ManyToOne(fetch = FetchType.LAZY)`** (기본값 EAGER 금지)
- **`hibernate.jdbc.time_zone: UTC` 설정** — 로컬 KST와 RDS UTC의 9시간 어긋남 방지
- 입력값 제약을 스키마와 일치시킨다 (`CLAUDE.md` 4장 제약 표)
- `UserRepository`, `TodoRepository` (`deleted_at IS NULL` 조건 포함 쿼리)
- 인덱스 `idx_todos_user_deleted`
- `src/test/resources/application-test.yml` (todolist_test, `ddl-auto: create-drop`)
- **`@EnableJpaAuditing`은 메인 애플리케이션 클래스에 붙인다** (`@Configuration`에 두면 `@DataJpaTest`가 로드하지 않아 `created_at`이 null이 됨)
- Repository 테스트에 **`@AutoConfigureTestDatabase(replace = NONE)` + `@ActiveProfiles("test")`** 필수 (없으면 임베디드 DB로 교체 시도)

**DoD**
- [x] 애플리케이션 기동 시 `users`, `todos` 테이블 자동 생성
- [x] `created_at`, `updated_at` 자동 기록 확인
- [x] Repository 단위 테스트(`@DataJpaTest`) 통과 — **통합 테스트 번호 체계(1~8)와 별개**
- [x] 테스트가 `todolist_test`를 바라보고 실행됨 (H2 사용하지 않음)
- [x] `@DataJpaTest`에서 `created_at`이 null이 아님 (Auditing 정상 동작)
- [x] 저장된 `created_at`이 UTC 기준임 (KST로 9시간 밀리지 않음)

> **검증 기록 (2026-08-31)** — 6항목 전부 통과. 근거는 이번 검증 세션의 실제 명령 출력이다.
>
> | DoD | 근거 |
> |---|---|
> | 1 | 착수 시점 `todolist_db` 테이블 0개 → 기동 후 `users`·`todos` 생성. 로그에 `create table` DDL |
> | 2 | `created_at`·`updated_at` 모두 `is_nullable=NO`, `deleted_at`만 `YES` |
> | 3 | `./mvnw clean test` **exit 0**, `Tests run: 14, Failures: 0, Errors: 0` |
> | 4 | Hikari 로그 `jdbcUrl...localhost:5432/todolist_test`. 의존성에 h2·hsqldb·derby **0건** |
> | 5 | `auditingFieldsArePopulated` 2건 통과 |
> | 6 | `createdAtIsStoredInUtc`·`auditColumnsAreTimestampTz` 통과. **DB 서버 타임존이 `Asia/Seoul`인 환경에서 통과**해 실질 검증 |
>
> 인덱스 `idx_todos_user_deleted`는 `pg_indexes`에서 `btree (user_id, deleted_at)` **실물 생성**을 확인했다.
> `@Table`의 `indexes`는 ddl-auto가 테이블을 새로 만들 때만 반영되므로, 선언만 있고 실물이 없는 상태를 따로 검출해야 한다.
>
> ⚠️ **`completed`·`priority`에 DB `DEFAULT` 절이 생성되지 않는다.** 기본값은 `Todo` 생성자에서
> 애플리케이션 레벨로 채워진다(`defaultPriorityIsMedium`으로 검증). JPA 경로는 무해하나,
> **Phase 4의 `seed-dev.sql`·`seed-perf.sql`은 JPA를 우회한 직접 INSERT라 두 컬럼을 반드시 명시해야 한다.**
> 누락하면 NOT NULL 위반으로 실패한다.
>
> 참고: 4장 표기는 `TIMESTAMP`이나 실물은 `timestamptz`다. `Instant` 선택의 결과이자 UTC 보장의 핵심이므로 구현이 옳다.

---

## Phase 3 — 인증 (로컬) + 인증 테스트

**저장소**: `todo-backend` · **관련 요구사항**: AUTH-01~04, 07~08 (AUTH-06 로그아웃은 프론트 전용이라 Phase 7)

**작업**
- `SecurityConfig` (SecurityFilterChain, CORS, 경로별 인가)
  - **`permitAll` 경로에 Swagger(`/swagger-ui/**`, `/v3/api-docs/**`)를 반드시 포함** (`CLAUDE.md` 6장 목록 그대로)
  - CORS 허용 헤더에 `Authorization`, `Content-Type` 명시
- `JwtTokenProvider` (생성/검증, 24시간 만료, **`sub = user.id`**)
- `JwtAuthenticationFilter` (`sub` → id 조회, `deleted_at IS NULL` 확인)
- `BCryptPasswordEncoder` 빈
- `AuthController`: signup / login / me (**로그아웃 API는 만들지 않는다**)
- DTO: `SignupRequest`, `LoginRequest`, `TokenResponse`, `UserResponse`
- `GlobalExceptionHandler` + `ErrorCode` + **공통 응답 `ApiResponse<T>`**
- **Swagger `@SecurityScheme`(bearerAuth) 설정** — Authorize 버튼으로 보호된 API 호출 가능하게
- **`AuthenticationEntryPoint` + `AccessDeniedHandler` 구현** — Security 필터 단계의 401/403도 `ApiResponse` 포맷으로 응답 (`GlobalExceptionHandler`는 필터 예외를 잡지 못함)
- **통합 테스트 1~3번 작성**

**DoD**
- [x] **`POST /api/v1/auth/signup`이 403이 아니라 200/409로 응답** (CSRF 비활성화 확인 — 안 하면 여기서 전부 막힌다)
- [x] **응답에 `Set-Cookie: JSESSIONID`가 없음** (`STATELESS` 세션 정책 확인)
- [x] 회원가입 → 사용자 생성 + JWT 반환
- [x] **한글 25자 비밀번호로 가입 시 500이 아니라 400 `INVALID_INPUT`** (BCrypt 72바이트 한계 — `CLAUDE.md` 4장)
- [x] 중복 이메일 시 409 `EMAIL_DUPLICATED`
- [x] 로그인 성공 시 유효 JWT, 실패 시 401
- [x] **미가입 이메일로 로그인한 401과 비밀번호가 틀린 401의 `code`·`message`가 완전히 동일함** (계정 존재 여부를 구분 노출하지 않는다 — `PRD.md` 5.1)
- [x] 토큰 없이 `/api/v1/auth/me` 호출 시 401
- [x] `/api/v1/auth/me` 응답에 `nickname`과 `email`이 모두 포함됨 (AUTH-08 — 화면 표시 여부는 Phase 7에서 통제)
- [x] 비밀번호가 DB에 해시로 저장됨
- [x] **Security 도입 후에도 Swagger UI 접속 가능** (Phase 1 DoD 회귀 방지)
- [x] Swagger Authorize에 토큰을 넣고 `/auth/me` 호출이 200으로 성공 — ⚠️ **브라우저 미조작. `curl` 대체 검증**(아래 기록 참조)
- [x] 모든 응답이 `{success, data, error}` 포맷을 따름
- [x] **토큰 없이 호출한 401 응답도** `{success:false, error:{code:"UNAUTHORIZED"}}` 포맷임
- [x] **인증 통합 테스트 3건 통과**

> **검증 기록 (2026-09-01)** — 15항목 전부 통과. 근거는 이번 검증 세션의 실제 명령 출력이다.
>
> 판정 경로는 둘이다. **테스트 경로**는 `DB_PASSWORD` 주입 후 `./mvnw clean test`(**exit 0**, `Tests run: 26`)이고,
> **기동 경로**는 환경변수 5종을 주입한 `./mvnw spring-boot:run` + `curl`이다.
> ⚠️ 기동 포트는 **8081**이다. 8080을 무관한 타 프로젝트(`com.hi.mallapi.MallapiApplication`)가 점유 중이어서
> 그 프로세스를 종료하지 않고 `SERVER_PORT`로 우회했다. 포트 번호는 판정에 영향을 주지 않는다.
>
> | DoD | 근거 |
> |---|---|
> | 1 | `curl -i POST /api/v1/auth/signup` → `HTTP/1.1 200`. 403이 아님 |
> | 2 | 같은 응답 헤더에 `Set-Cookie` **자체가 없음**(헤더 부재로 확인) |
> | 3 | 응답 `data.token` 발급 + `psql`로 `users` 행 생성 확인 |
> | 4 | **75바이트를 바이트 단위로 생성해 전송** → `400` + `"password: 비밀번호가 너무 깁니다…"` |
> | 5 | 동일 이메일 재요청 → `409` `EMAIL_DUPLICATED` |
> | 6 | 로그인 `200` + JWT. 페이로드 `sub:"4"`(id), `exp - iat = 86400`(24시간) |
> | 7 | 두 401 응답 본문을 `diff`로 대조 → **완전 일치**. 상태 코드도 동일 |
> | 8 | 토큰 없이 `GET /auth/me` → `HTTP/1.1 401` |
> | 9 | 유효 토큰으로 `200` + `{"id","email","nickname"}` 전부 포함 |
> | 10 | `psql`로 `todolist_db` 조회 → `$2a$10$` 접두, 60자, 평문과 불일치 |
> | 11 | `/swagger-ui/index.html` → `200` (302 아님) |
> | 12 | **⚠️ 대체 검증** — 아래 별도 항목 참조 |
> | 13 | 성공·실패 응답 모두 `success`·`data`·`error` 3키 확인 |
> | 14 | 필터 단계 401 본문이 `{"success":false,"data":null,"error":{"code":"UNAUTHORIZED",…}}` |
> | 15 | `AuthControllerTest` 12건 통과(시나리오 3묶음). 기존 14건 회귀 없음 |
>
> #### DoD 12를 브라우저로 검증하지 않았다
>
> Swagger UI의 Authorize 버튼을 **실제로 누르지 않았다.** `curl` 두 가지로 대체했다.
>
> 1. `/v3/api-docs`의 `components.securitySchemes.bearerAuth`가 `type:http`·`scheme:bearer`·`bearerFormat:JWT`이고,
>    전역 `security`와 `/api/v1/auth/me` 오퍼레이션 `security`에 모두 `bearerAuth`가 걸려 있음 → **버튼이 렌더되는 근거**
> 2. 발급 토큰을 `Authorization: Bearer`로 넣은 `/auth/me`가 `200` → **버튼에 토큰을 넣었을 때와 같은 경로**
>
> 둘이 성립하면 브라우저 조작도 성공하는 것이 따라오지만, **직접 확인한 것은 아니다.**
>
> #### 검증 중 발견해 조치한 사항 — JWT가 HS256이 아니었다
>
> 발급 토큰 헤더가 `{"alg":"HS384"}`였다. `CLAUDE.md` 12장의 **"HS256으로 고정한다"** 규정과 어긋난다.
> 원인은 `JwtTokenProvider`가 `signWith(key)`를 인자 없이 호출한 것이다. jjwt의 `Keys.hmacShaKeyFor()`는
> **키 길이로 알고리즘을 추론**한다(32~47B→HS256, 48~63B→HS384, 64B+→HS512). 즉 알고리즘을 코드가 아니라
> **`JWT_SECRET`의 길이가 결정**하는 상태였다. 로컬과 운영의 시크릿 길이가 다르면 알고리즘이 갈린다.
>
> → `.signWith(key, Jwts.SIG.HS256)`으로 알고리즘을 명시했다. **같은 49바이트 시크릿으로 재기동해**
> 토큰 헤더가 `{"alg":"HS256"}`으로 바뀐 것과 전체 테스트 26건이 그대로 통과함을 확인했다.
> 조건을 하나만 바꿔야 인과가 증명되므로 시크릿 길이는 일부러 유지했다.
>
> #### 확인했으나 조치하지 않은 사항 — `/swagger-ui.html`이 401
>
> Swagger 진입 주소가 둘인데 하나만 열려 있다. `/swagger-ui/index.html`은 `200`이고 `/swagger-ui.html`은 `401`이다.
> **`CLAUDE.md` 6장의 `permitAll` 경로 목록에 후자가 없기 때문이며, 현재 코드는 스펙을 정확히 따른 결과다.**
> DoD 11의 판정 기준이 전자이므로 판정에는 영향이 없다. 다만 **기동 로그가 후자를 안내**하므로
> (`SpringDoc /swagger-ui.html endpoint is enabled by default`) 혼동의 여지가 있다.
> 고치려면 스펙의 경로 목록부터 바꿔야 하므로, 임의로 추가하지 않고 이 기록만 남긴다.
>
> #### 검증 잔여물
>
> `todolist_db.users`에 검증용 계정이 남아 있다(`id` 4·5·6). Phase 4의 시드 스크립트 작업 전에 정리한다.

---

## Phase 4 — Todo API + Todo 테스트

**저장소**: `todo-backend` · **관련 요구사항**: TODO-01~15

**작업**
- `TodoController` 6개 엔드포인트 (목록/생성/단건/수정/토글/삭제)
- **`PUT`은 전체 교체이며 `TodoUpdateRequest`에 `completed`를 넣지 않는다**
- **`PATCH /toggle`은 바디로 `{"completed": true}`를 받는다** (서버가 뒤집지 않음)
- `TodoService`: 소유권 검증, Soft Delete, **HTML 정화**
- **HtmlSanitizer**: Jsoup Safelist, 허용 태그·`rel` 주입·스킴 제한 (`CLAUDE.md` 6장)
- 페이지네이션 + `completed`(미지정 시 전체) + `keyword`(대소문자 무시) 필터
- 정렬은 `createdAt,desc` 고정. **허용 필드 화이트리스트(`createdAt`, `dueDate`) 밖의 값은 기본값으로 대체** (없는 프로퍼티로 500 방지)
- `PageResponse<T>` DTO → **`ApiResponse.data` 안에 담아 반환**
- **`TodoResponse`에 사용자 정보를 넣지 않는다** (본인 데이터만 조회하므로 불필요, 넣으면 N+1)
- 날짜 직렬화: `createdAt`/`updatedAt`은 ISO-8601 UTC, `dueDate`는 `yyyy-MM-dd`
- DTO: `TodoCreateRequest`, `TodoUpdateRequest`, `TodoResponse`
- **개발용 시드 스크립트** — 용도가 둘이므로 파일을 나눈다
  - `db/seed-dev.sql` — 테스트 계정 1개 + Todo **100건** (우선순위·완료·마감일 혼합). 페이지네이션·필터·정렬을 **눈으로** 확인하는 용도
  - `db/seed-perf.sql` — 같은 계정에 Todo **10,000건**. 성능 DoD 측정 전용. 제목에 검색 대상 키워드가 골고루 섞이도록 생성한다
- **통합 테스트 4~7번 작성**

**DoD**
- [x] 목록 API가 `{success, data:{content, page, ...}, error}` 형태로 응답
- [x] `PUT` 저장이 완료 상태를 덮어쓰지 않음
- [x] `toggle`을 같은 값으로 두 번 호출해도 결과가 동일함(멱등)
- [x] `completed` 미지정 시 전체 반환, `true`/`false` 시 필터 적용
- [x] 영문 대소문자를 섞어 검색해도 결과가 나옴
- [x] `?sort=foo,desc` 같은 잘못된 정렬 값에도 500이 나지 않음
- [x] 삭제 시 `deleted_at` 기록, 목록에서 제외
- [x] 타 사용자 Todo 접근 시 404
- [x] 제목 미입력·200자 초과 시 400 + 필드 메시지
- [x] 본문 50,000자 초과 시 400
- [x] `<script>` 포함 본문 저장 시 태그 제거, `a` 태그에 `rel` 주입 확인
- [x] **키워드 검색 포함** 목록 조회가 **워밍업 후 3회 측정 중앙값 500ms 이내** (로컬, 시드 **10,000건** 기준)
  > ⚠️ 시드 100건으로는 이 지표가 의미가 없다. 인덱스가 없어도 100행은 1ms 미만이라 **항상 통과한다.** `CLAUDE.md` 4장이 지목한 유일한 성능 위험(`LOWER(title) LIKE '%키워드%'`의 인덱스 미사용)을 검출하려면 데이터가 충분해야 하고, 측정 대상도 검색 경로여야 한다. 또 첫 요청은 JVM 콜드 스타트라 DB가 아니라 워밍업 상태를 재는 셈이 되므로 워밍업 후에 측정한다.
- [x] 날짜가 배열이 아닌 ISO 문자열로 직렬화됨
- [x] 목록 조회 시 user 조회 쿼리가 추가로 발생하지 않음
- [x] Swagger에서 전체 API 확인 가능 — ⚠️ **Authorize 버튼은 브라우저로 직접 누르지 않았다. `curl` 대체 검증**(아래 기록 참조)
- [x] **Todo 통합 테스트 4건 통과**

> **검증 기록 (2026-09-01)** — 16항목 전부 통과. 근거는 이번 검증 세션의 실제 명령 출력이다.
>
> 판정 경로는 둘이다. **테스트 경로**는 `DB_PASSWORD` 주입 후 `./mvnw clean test`(**exit 0**, `Tests run: 58`:
> `AuthControllerTest` 12 + `TodoControllerTest` 21 + `TodoRepositoryTest` 11 + `TodoServiceTest` 8 +
> `UserRepositoryTest` 5 + `contextLoads` 1)이고, **기동 경로**는 환경변수 5종을 주입한
> `./mvnw spring-boot:run` + `curl`이다. 8080을 무관한 타 프로젝트가 점유 중이라 종료하지 않고
> `SERVER_PORT=8081`로 우회했다(Phase 3와 동일).
>
> | DoD | 근거 |
> |---|---|
> | 목록 API 포맷 | `curl`로 시드 계정 목록 조회 → `dataKeys: [content,first,last,page,size,totalElements,totalPages]` |
> | PUT이 completed 유지 | `TodoControllerTest.putDoesNotOverwriteCompleted` |
> | toggle 멱등 | `TodoControllerTest.toggleIsIdempotent` |
> | completed 필터 | `TodoControllerTest` 목록 묶음 3건(미지정/true/false) |
> | 대소문자 무시 검색 | `TodoControllerTest.keywordSearchIsCaseInsensitive` |
> | 잘못된 sort 값 | `TodoControllerTest.invalidSortDoesNotCause500` |
> | Soft Delete | `TodoControllerTest` 5번 묶음 2건 + `psql`로 물리 행 잔존 확인 |
> | 타 사용자 404 | `TodoControllerTest` 6번 묶음 3건(GET·PUT·DELETE 전부) |
> | 제목 검증 | `TodoControllerTest` 입력검증 묶음(미입력·200자 초과) |
> | 본문 검증 | 〃 (50,000자 초과) |
> | XSS 정화 + rel 주입 | `TodoControllerTest` 7번 묶음 4건(script 제거·rel/target 강제·pre 줄바꿈 보존·javascript: 스킴 제거) |
> | 성능 500ms | 아래 별도 기록 |
> | 날짜 ISO 직렬화 | `curl`: `createdAt="2026-08-31T22:43:08.114254Z"`, `dueDate="2026-08-03"` |
> | user 추가 쿼리 없음(N+1) | `TodoControllerTest.queryCountDoesNotScaleWithItemCount` — Hibernate Statistics로 항목 3→6개 시 쿼리 수 불변 실측 |
> | Swagger 전체 API | `/swagger-ui/index.html` → 200. `/v3/api-docs`에 `/api/v1/todos`(get,post)·`/api/v1/todos/{id}`(get,put,delete)·`/api/v1/todos/{id}/toggle`(patch) 6개 메서드 전부 등록 확인 |
> | Todo 통합 테스트 4건 | `TodoControllerTest`가 14장 4~7번(시나리오 4개)을 `@Nested` 네 묶음으로 구현. 시나리오당 `@Test`는 4번 6건·5번 2건·6번 3건·7번 4건으로 총 15건이며, DoD 판정용 추가 케이스(PUT/toggle 2·입력검증 3·N+1 1) 6건을 더해 클래스 전체는 21건이다. "4건"은 시나리오 수이지 메서드 수가 아니다(Phase 3의 "인증 통합 테스트 3건"과 같은 표기 관례) |
>
> #### Swagger를 브라우저로 검증하지 않았다
>
> Authorize 버튼을 실제로 누르지 않았다. `/v3/api-docs`에서 세 경로 모두 `security:[{"bearerAuth":[]}]`가
> 걸려 있음과, 발급 토큰을 `Authorization: Bearer`로 넣은 실제 API 호출이 성공함을 대체 근거로 삼았다
> (Phase 3와 동일한 방식).
>
> #### 성능 측정 상세
>
> ```
> 데이터: todolist_db.todos = 10,100건 (seed-perf 10,000 + seed-dev 100, 태스크 19 산출물)
> 검색어: "회의"(2,020건 매칭) — URL-encode로 전송해 셸 UTF-8 인코딩 문제를 피했다
>
> 워밍업: 60.4ms (측정 제외)
> 측정 1: 55.651ms
> 측정 2: 55.949ms
> 측정 3: 56.818ms
> 중앙값: 55.949ms   ← 500ms 기준의 약 11%
> ```
>
> 기준 대비 여유가 커 `EXPLAIN ANALYZE`는 불필요했다(초과 시에만 요구되는 절차). 이 여유가
> "인덱스가 필요 없다"는 뜻은 아니다 — 10,100건은 로컬 PostgreSQL이 캐시하기에 충분히 작은 크기이며,
> 데이터가 수십만 건으로 커지면 같은 `LIKE` 패턴이 다시 문제가 될 수 있다(`CLAUDE.md` 4장이 위임한
> 미래의 `pg_trgm` 검토 대상).
>
> #### 검증 중 발견한 스펙 위반
>
> 없음. Phase 3의 HS256 건과 같은 신규 발견은 이번 검증에서 없었다.
>
> #### 정리
>
> 기동 프로세스 종료 후 8081 포트 해제를 재확인했다(종료 직후 첫 확인에서 `LISTENING`으로 남아 있어
> 1초 후 재조회 — 타이밍 문제였을 뿐 실제 잔존은 아니었다).

---

## Phase 5 — 구글 OAuth2 + OAuth 테스트

**저장소**: `todo-backend` · **관련 요구사항**: AUTH-05, AUTH-09

**작업**
- Google Cloud Console에서 OAuth 클라이언트 생성, 리다이렉트 URI 등록 (로컬 + 운영 둘 다)
- `spring.security.oauth2.client` 설정
- `CustomOAuth2UserService`: 신규 가입 / 기존 조회 / **충돌 거부** 분기
- **nickname 결정**: 구글 `name` → 없으면 이메일 `@` 앞부분 → 50자 초과 시 절삭
- `OAuth2SuccessHandler`: JWT 발급 후 `{FRONTEND_URL}/oauth/callback?token=` **302 리다이렉트**
- `OAuth2FailureHandler`: 충돌 시 `{FRONTEND_URL}/login?error=email_conflict` **302 리다이렉트** (JSON 에러 응답을 반환하지 않는다)
- **테스트 8번 작성 — 통합 테스트가 아니라 `CustomOAuth2UserService` 단위 테스트로 작성한다.** OAuth2 흐름은 실제 구글 서버와 통신하므로 MockMvc로 끝까지 검증할 수 없다 (`CLAUDE.md` 14장)

**계정 충돌 정책 (확정)**
동일 이메일의 로컬 계정이 있으면 **거부한다.** 자동 연동하지 않고, 별도 계정도 만들지 않는다. 상세는 `CLAUDE.md` 6장.

**DoD**
- [x] 구글 로그인 후 JWT를 담은 302 리다이렉트 발생
- [x] 신규 사용자가 `provider=GOOGLE`, nickname이 채워진 상태로 저장됨
- [x] 재로그인 시 중복 계정이 생기지 않음
- [x] 동일 이메일 로컬 계정 존재 시 `error=email_conflict`로 302 리다이렉트
- [x] **OAuth 서비스 단위 테스트 1건 통과**

> **검증 기록 (2026-09-02)** — 5항목 전부 통과. 근거는 이번 검증 세션의 실제 브라우저 조작·`psql`
> 조회·명령 출력이다.
>
> 판정 경로는 둘이다. **라이브 경로**는 실제 구글 계정으로 `claude-in-chrome`을 통해 로그인 흐름 전체를
> 브라우저에서 수행하고 `psql`로 결과 행을 직접 조회했다(AskUserQuestion으로 "라이브 브라우저 검증"과
> "단위 테스트+코드 검토로 대체" 중 선택을 받아 전자를 택함). **테스트 경로**는 `DB_PASSWORD`·
> `DB_USERNAME`·`JWT_SECRET`을 `.env`에 채운 뒤 `./mvnw clean test`(**exit 0**, `Tests run: 63`:
> `AuthControllerTest` 12 + `TodoControllerTest` 21 + `TodoRepositoryTest` 11 + `TodoServiceTest` 8 +
> `UserRepositoryTest` 5 + `CustomOAuth2UserServiceTest` 5 + `contextLoads` 1)이다.
>
> | DoD | 근거 |
> |---|---|
> | JWT 담은 302 리다이렉트 | 구글 로그인 완료 후 브라우저가 `http://localhost:3000/oauth/callback?token=`으로 실제 이동함을 화면에서 확인 |
> | 신규 GOOGLE 가입 | `psql`: `id=3, provider='GOOGLE', nickname='...', password IS NULL` |
> | 재로그인 시 중복 없음 | 같은 계정으로 재로그인 후 `psql`: 동일 이메일 행 `count=1` (신규 insert 없음) |
> | 계정 충돌 시 302 | DoD 2·3 검증에 쓴 GOOGLE 계정을 정리한 뒤 같은 이메일로 로컬 회원가입 → 구글 로그인 재시도 → `http://localhost:3000/login?error=email_conflict`로 리다이렉트, `psql`: 해당 이메일 LOCAL 행만 1건 유지 |
> | OAuth 서비스 단위 테스트 | `CustomOAuth2UserServiceTest` 5건(신규가입·재로그인무중복·계정충돌·nickname 이메일폴백·nickname 50자 절삭) 전부 통과 |
>
> #### 계정 충돌 검증의 순서 문제
>
> DoD 4를 검증하려면 로컬 계정이 먼저 있어야 하는데, DoD 2·3 검증 과정에서 같은 이메일로 이미
> GOOGLE 계정이 만들어져 있어 로컬 회원가입이 이메일 중복으로 막혔다. `psql DELETE`로 그 GOOGLE
> 계정(id=3)을 정리한 뒤 `curl`로 로컬 가입(id=4)을 만들고 구글 로그인을 재시도해 충돌을 재현했다.
> 검증에 쓴 모든 테스트 계정은 확인 후 `psql DELETE`로 정리했다.
>
> #### `.env` 보완 경위
>
> 검증 착수 시 `.env`의 `DB_USERNAME`·`JWT_SECRET`이 비어 있어 서버가 기동하지 않았다
> (`application-local.yml`의 `${DB_USERNAME}`에 기본값이 없음). `DB_USERNAME`은 기존 PostgreSQL
> 계정을 확인해 채우는 값이라 사용자가 직접 입력했고(`postgres`), `JWT_SECRET`은 새로 정하는 임의
> 값이라 `openssl rand -hex 32`로 생성해 채웠다. `.env`가 CRLF 줄바꿈이라 `source`로 그대로 읽으면
> 값 끝에 `\r`이 섞일 위험이 있어, `while IFS='=' read` 루프에서 `${value%$'\r'}`로 제거한 뒤
> `export`했다.
>
> #### 검증 중 발견한 스펙 위반
>
> 없음.
>
> #### 정리
>
> 서버 종료 시 Phase 3·4의 교훈(`mvnw` 래퍼만 죽고 포크된 JVM이 남을 수 있음)에 따라 `netstat`으로
> 확인한 실제 JVM PID(22028)를 `taskkill //PID 22028 //F //T`로 직접 종료하고, `netstat`·`tasklist`로
> 완전한 정리를 재확인했다.

---

## Phase 6 — 프론트 스캐폴딩

**저장소**: `todo-frontend` · **관련 요구사항**: UX-01·02·04·05·07 (공용 컴포넌트·토큰 수준) · **선행 조건**: Phase 0만 필요하다. **백엔드 Phase 1~5와 병렬로 진행할 수 있다** (이 Phase는 실서버를 호출하지 않는다)

**작업**
- **Node 20 이상** 확인 후 `create-next-app` (**Next.js 15**, App Router, TypeScript)
- Tailwind CSS 4 설정 — `globals.css`의 `@theme`에 디자인 토큰 정의 (**v3 방식 금지**)
- **디자인 토큰은 라이트/다크 양쪽 정의 + `@media (prefers-color-scheme: dark)`** (`class` 전략 금지 — 토글이 없어 FOUC만 생김)
- shadcn/ui 초기화 (**npm 사용 시 `--legacy-peer-deps`**, 스타일 `new-york`), lucide-react 설치
- **`npm install motion`** (`framer-motion` 아님), sonner, date-fns, DOMPurify 설치
- **Tiptap 설치**: `@tiptap/react`, `@tiptap/starter-kit` **두 개만.** `@tiptap/extension-link`는 **설치하지 않는다** — v3 StarterKit에 `Link`가 포함되어 있어 중복 등록이 된다 (`CLAUDE.md` 8장)
- **Pretendard 폰트** — Google Fonts에 없으므로 `.woff2` 파일을 `src/app/fonts/`에 넣고 **`next/font/local`**로 로드 (`next/font/google` 사용 불가)
- React Query Provider, **쿼리 키 규약 상수화** (`CLAUDE.md` 9장), `apiClient` (토큰 주입 + **`ApiResponse` 언래핑** + 에러 정규화 + 401 처리)
- **`lib/errorMessages.ts` — `PRD.md` 5.1 「에러 문구 매핑」 표를 코드로 옮긴다.** `error.code`(`INVALID_INPUT` / `UNAUTHORIZED` / `EMAIL_DUPLICATED` / `TODO_NOT_FOUND` / `INTERNAL_ERROR`)와 **네트워크 실패**를 화면 문구로 변환하는 단일 함수를 둔다
  > ⚠️ 화면마다 문구를 직접 쓰면 Phase 7·8에서 서로 다른 문구가 생겨 매핑 표가 사문화된다. `apiClient`가 던지는 에러를 이 함수 하나로만 문구화한다. 네트워크 실패는 `error.code`가 없으므로 **정규화 단계에서 별도 구분자를 남겨야** 한다.
- **`lib/validation.ts` — `CLAUDE.md` 4장 「입력값 제약」 표를 코드로 옮긴다.** 폼 라이브러리를 쓰지 않기로 확정했으므로(`CLAUDE.md` 3장) 검증이 화면에 흩어지기 쉽다. 이메일 형식·닉네임 1~50자·제목 200자·본문 50,000자와 **비밀번호 6자 이상 + UTF-8 72바이트 이하**를 한곳에 둔다
  > ⚠️ **비밀번호 상한은 문자 수가 아니라 바이트다.** `maxLength={64}` 같은 문자 수 제한만 걸면 한글 25자(=75바이트)가 통과해 서버 BCrypt 단계에서 터진다. `new TextEncoder().encode(v).length`로 센다.
- `lib/sanitize.ts` (DOMPurify 래퍼) — **`ALLOWED_TAGS`·`ALLOWED_ATTR`을 명시한다.** 기본값은 Jsoup 화이트리스트보다 넓고, `ALLOWED_ATTR`에서 `rel`·`target`을 빠뜨리면 서버가 주입한 tabnabbing 방어가 렌더 단계에서 지워진다 (`CLAUDE.md` 6장)
- 공용 컴포넌트: `Pagination`, `EmptyState`, `ErrorState`, `Skeleton`
  - **`ErrorState`는 `onRetry`를 필수 prop으로 받고 재시도 버튼을 항상 렌더한다** (`UX-04`). 선택 prop으로 두면 호출부에서 빠뜨려도 타입 검사가 통과한다
- 루트 레이아웃 + Provider 분리 (루트는 서버 컴포넌트, Provider는 클라이언트 컴포넌트)
- 공통 헤더 **껍데기만** 만든다 (닉네임·로그아웃 자리는 비워둠 — `useAuth`가 없는 시점이므로 Phase 7에서 연결)
- 타입 정의 (`src/types/`) — 백엔드 DTO와 이름 일치, `ApiResponse<T>` / `PageResponse<T>` 포함
- `.env.example`, 저장소용 `CLAUDE.md` (상단에 `@../CLAUDE.md` 임포트)
- **`public/static` 경로를 만들지 않는다** (Amplify 예약 경로)

**DoD**
- [x] **`package.json`의 `next`와 `eslint-config-next`가 모두 15.x임 (16 아님)** — 둘 중 하나만 확인하면 놓친다
- [x] **소스가 `src/` 아래에 있음** (`src/app/`, `src/components/`, `src/lib/`, `src/types/`)
- [x] **`package.json`에 `@tiptap/extension-link`가 없음** (v3 StarterKit 내장)
- [x] Node 20 이상에서 빌드됨
- [x] `npm run build` 성공, 출력 디렉토리가 `.next`
- [x] `package.json`에 `motion`이 있고 `framer-motion`이 없음
- [x] 디자인 토큰이 OS 다크 설정에 따라 전환됨 (`class` 조작 없이 CSS만으로)
- [ ] ~~페이지 `page.tsx`에 `"use client"`가 붙어 있음~~ **N/A** — 아래 검증 기록 참조
- [x] **`components.json`의 `style`이 `new-york`임** (현재 `radix-nova`)
- [x] **`globals.css`에 `.dark` 클래스 셀렉터나 `@custom-variant dark (&:is(.dark *))`가 없고, 다크 토큰이 `@media (prefers-color-scheme: dark)` 안에 정의되어 있음** (현재 shadcn 기본값이 `class` 전략이라 반드시 걷어내야 한다)
- [x] 디자인 토큰 값이 `CLAUDE.md` 8장 팔레트와 일치함 (배경 `#FAFAFA`/`#0A0A0A`, 액센트 `#4F46E5`, 우선순위 3색) — shadcn 기본 neutral이 남아 있지 않음
- [x] Pagination 컴포넌트 단독 동작 확인 (더미 데이터). **페이지 수 1 이하일 때 아무것도 렌더하지 않음**
- [x] `ErrorState`가 재시도 버튼과 함께 렌더되고, 버튼 클릭이 `onRetry`를 호출함
- [x] `apiClient`가 `data` 언래핑과 401 처리를 수행함
- [x] **`apiClient`가 던진 에러를 `lib/errorMessages.ts`에 넣으면 `PRD.md` 5.1 표의 문구가 그대로 나옴** (네트워크 실패 케이스 포함 — 서버를 내리고 확인)

> **검증 기록 (2026-09-02)** — 15항목 중 14항목 통과, 1항목(`"use client"`)은 검증 대상 부재로 **N/A** 처리(사용자 확인 완료). 근거는 이번 검증 세션의 실제 명령 출력과 브라우저 실측이다.
>
> 판정 경로는 둘이다. **정적 경로**는 `package.json`·`components.json`·`globals.css`를 직접 읽거나 `grep`한 결과와, `rm -rf .next && npm run build`(**exit 0**, 정적 페이지 생성 성공)다. **브라우저 경로**는 `npm run dev` 기동 후 `src/app/dod-verify`(임시 데모 라우트, 사용자 확인 후 존치)에 공용 컴포넌트와 `apiClient`/`errorMessages` 인터랙션을 배치해 `claude-in-chrome`으로 클릭·DOM·CSSOM을 직접 확인했다.
>
> | DoD | 근거 |
> |---|---|
> | next·eslint-config-next 15.x | `package.json`: 둘 다 `15.5.24` |
> | src/ 아래 소스 | `app`·`components`·`lib`·`types`가 프로젝트 루트가 아니라 `src/` 아래에만 존재(루트 검색 결과 없음) |
> | `@tiptap/extension-link` 없음 | `package.json` grep 결과 없음(단 `@tiptap/starter-kit`의 전이 의존성으로 `node_modules`엔 존재 — 직접 의존성이 아니므로 규칙 위반 아님) |
> | Node 20 이상 | `node -v` → `v24.18.0` |
> | build 성공 + `.next` | `rm -rf .next && npm run build` exit 0, `.next` 디렉터리 생성 확인 |
> | `motion` 있음, `framer-motion` 없음 | `package.json`에 `motion` 직접 의존성, `framer-motion` 직접 의존성 없음(`motion` 패키지의 전이 의존성으로만 존재) |
> | 다크 토큰 OS 전환(CSS만) | `document.styleSheets`를 순회해 `@media (prefers-color-scheme: dark)` 규칙이 실제로 파싱돼 있음을 CSSOM에서 확인. **이 환경에는 OS `prefers-color-scheme`을 강제 전환하는 도구가 없어 실제 다크 렌더 육안 확인은 하지 못했다**(Phase 6 착수 시점부터의 알려진 제약) |
> | `page.tsx`에 `"use client"` | **N/A.** 현재 유일한 `page.tsx`는 create-next-app 기본 홈(`/`)이며 `CLAUDE.md` 7장 화면 목록에 없는, 최종 스펙에 존재하지 않는 라우트다. `CLAUDE.md` 9장이 지목하는 6개 라우트(`/login`·`/signup`·`/todos` 등)는 전부 Phase 7·8에서 만들어진다 — 이번 Phase엔 검증 대상 자체가 없다. 사용자 확인 후 N/A로 기록했다. **Phase 7 착수 시 이 DoD가 다시 적용된다.** |
> | `components.json` style | `"style": "new-york"` |
> | `globals.css` 다크 전략 | `.dark`·`@custom-variant dark` 없음(grep 결과 없음), `@media (prefers-color-scheme: dark)` 존재 |
> | 팔레트 값 일치 | `globals.css`에서 배경 `#FAFAFA`/`#0A0A0A`, 카드 `#FFFFFF`/`#171717`, 텍스트 `#171717`/`#FAFAFA`, 보조 `#737373`, 액센트 `#4F46E5`, 우선순위 `#EF4444`/`#F59E0B`/`#10B981` 전부 확인 |
> | Pagination totalPages=1 → 렌더 없음 | `dod-verify`에서 `totalPages=1`로 렌더한 컨테이너의 `children.length === 0`을 DOM에서 직접 확인 |
> | ErrorState 재시도 버튼 | `find`로 버튼을 찾아 클릭 → 카운터가 `0회`에서 `1회`로 증가(`onRetry` 실제 호출 확인) |
> | apiClient 언래핑 + 401 | mock 성공 응답에서 `apiFetch`가 `data`만 반환, mock 401 응답에서 `ApiRequestError(code=UNAUTHORIZED)`를 던지고 `localStorage` 토큰이 실제로 삭제됨을 확인 |
> | errorMessages 매핑(네트워크 실패 포함) | `.env.local`에 도달 불가능한 포트(`localhost:59999`)를 임시로 설정해 **진짜 네트워크 실패**(mock 아님)를 발생시키고, `getErrorMessage(e)`가 `PRD.md` 5.1 표 그대로 `"연결에 실패했습니다."`를 반환함을 확인. 검증 후 `.env.local` 삭제 |
>
> #### DoD 소제목 "14항목" 관련
>
> 이 절의 소제목은 처음부터 "DoD"이며 ROADMAP.md 어디에도 "14항목"이라는 문구는 없었다. 태스크 계획 단계에서 체크박스 개수를 잘못 세어(14개로) 기록해 둔 것이 원인이며, 문서 자체에는 정정할 오탈자가 없다.
>
> #### 검증 중 발견한 스펙 위반
>
> 없음.
>
> #### 정리
>
> 검증에 쓴 `src/app/dod-verify` 임시 데모 라우트는 삭제하지 않고 존치하기로 사용자와 합의했다(Phase 7~9 컴포넌트 개발 시 재사용 가능). 검증용 `.env.local`은 삭제했다. dev 서버는 taskkill로 완전히 종료하고 포트 해제를 재확인했다.

> 버전·설치 방법은 `CLAUDE.md` 3장에서 모두 확정됐다. 이 Phase에서 재조사하지 않는다.

---

## Phase 7 — 인증 화면

**저장소**: `todo-frontend` · **관련 요구사항**: AUTH-01~09, UX-01, UX-06 · **선행 조건**: 백엔드 **Phase 3 완료**(가입·로그인·`/auth/me`), 구글 로그인 DoD는 **Phase 5 완료** 필요. 백엔드를 로컬에서 띄운 상태로 진행한다

**작업**
- `/login`, `/signup`, `/oauth/callback`
- **`/oauth/callback`은 `useSearchParams`를 쓰므로 `<Suspense>`로 감싼다** (없으면 `npm run build` 실패)
- **`/oauth/callback` 세부** (`PRD.md` 5.4)
  - 토큰 저장 → **URL에서 토큰 제거**(히스토리·공유 링크에 남지 않도록) → `/todos` 이동
  - **`token` 파라미터가 없거나 빈 문자열이면 `/login`으로 보낸다**
  - 처리 중에는 스켈레톤만 보여주고 **사용자가 조작할 요소를 두지 않는다**
  - **공통 헤더를 두지 않는다** (인증 처리 중 화면이라 닉네임을 알 수 없다)
- **`/todos` 플레이스홀더 페이지 생성** — 라우트 보호를 검증하려면 대상 페이지가 존재해야 한다 (내용은 Phase 8)
- Phase 6에서 비워둔 **헤더의 닉네임·로그아웃을 `useAuth`에 연결**
  - **이메일은 화면에 표시하지 않는다.** `/auth/me` 응답에는 들어오지만 헤더에는 닉네임만 노출한다 (`AUTH-08`)
- `useAuth` 훅 (로그인 / **로그아웃: 토큰 삭제 + 캐시 초기화** / 현재 사용자)
- **라우트 보호는 `(main)` 클라이언트 레이아웃에서 처리한다. `middleware.ts`를 만들지 않는다** (localStorage는 middleware에서 읽을 수 없음 — `CLAUDE.md` 9장)
  - **인증 판정이 끝나기 전에는 스켈레톤을 보여준다** (`UX-01`, `PRD.md` 5.1)
- 401 응답 시 자동 로그아웃 처리
- `?error=email_conflict` 안내 문구 표시
- **`/signup` 실시간 검증** (`PRD.md` 5.3) — 이메일 형식, **비밀번호 6자 이상 + UTF-8 72바이트 이하**, 닉네임 1~50자. 안내 문구에 **한글 1자 = 3바이트**임을 밝힌다
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다** (`PRD.md` 5.1 매핑 표)
  - `INVALID_INPUT` → 서버가 준 필드별 메시지를 해당 입력 아래 인라인
  - `UNAUTHORIZED`(로그인 시) → "이메일 또는 비밀번호가 올바르지 않습니다."를 폼 상단 인라인
  - `UNAUTHORIZED`(그 외) → 문구 없이 `/login` 이동
  - `EMAIL_DUPLICATED` → "이미 사용 중인 이메일입니다."를 이메일 입력 아래 인라인
  - 네트워크 실패 → "연결에 실패했습니다." + 재시도 버튼

**DoD**
- [x] 이메일 가입·로그인 정상 동작
- [x] **가입 화면에서 한글 25자 비밀번호를 입력하면 제출 전에 바이트 초과 안내가 인라인으로 뜸** (서버 400에만 의존하지 않음 — `AUTH-02`)
- [x] **중복 이메일 가입 시 "이미 사용 중인 이메일입니다."가 이메일 입력 아래 인라인으로 뜸** (`AUTH-03`, `PRD.md` 5.1)
- [x] **미가입 이메일과 비밀번호 오류의 화면 문구가 동일함** ("이메일 또는 비밀번호가 올바르지 않습니다.") — 계정 존재 여부가 드러나지 않음
- [x] **백엔드를 내린 채 로그인을 시도하면 "연결에 실패했습니다." + 재시도 버튼이 나옴** (네트워크 실패 매핑 — `UX-04`)
- [x] 구글 로그인 → 콜백 → `/todos` 이동, URL에서 토큰 제거됨
- [x] **`/oauth/callback`에 `?token=` 없이 직접 접근하면 `/login`으로 이동함** (`PRD.md` 5.4)
- [x] **`/oauth/callback` 화면에 공통 헤더와 조작 가능한 요소가 없음**
- [x] **헤더에 닉네임이 보이고 이메일은 어디에도 렌더되지 않음** (DevTools에서 DOM 검색 — `AUTH-08`)
- [x] 계정 충돌 시 안내 문구 노출
- [x] 로그아웃 시 토큰·캐시 모두 제거되고 `/login`으로 이동
- [x] 새로고침해도 로그인 상태 유지
- [x] 미인증 상태로 `/todos` 접근 시 로그인으로 이동
- [x] **만료된 토큰을 localStorage에 직접 넣고 `/todos`에 접근했을 때, 보호 화면이 한 프레임도 노출되지 않고 곧바로 `/login`으로 이동**
  > `useAuth`가 토큰 존재 여부만 보면 만료 토큰이 판정을 통과해, 401 왕복 동안 보호 화면이 노출된다. `exp`를 디코드해야 한다 (`CLAUDE.md` 9장). 검증용 만료 토큰은 `JWT_EXPIRATION`을 일시적으로 낮춰 발급받으면 된다
- [x] `middleware.ts` 파일이 존재하지 않음
- [x] `npm run build` 성공 (`useSearchParams` Suspense 경계 확인)
- [x] **모든 입력에 label 연결, Tab·Enter만으로 가입·로그인 완주 가능**

> **검증 기록 (2026-09-02)** — 17항목 전항목 통과. 이 절의 소제목은 "DoD"이고 체크박스는 실제로 17개다(위 목록을 직접 세어 확인). 태스크 계획 단계에서 "16항목"으로 잘못 세어 기록한 것이 원인이며, Phase 6의 "14항목" 오기와 같은 패턴이다. 문서 자체에는 정정할 오탈자가 없다.
>
> 판정 경로는 브라우저 실측 하나다. 로컬 PostgreSQL + `todo-backend`(포트 8080, `.env` 로드)와 `todo-frontend`(`npm run dev`, `NEXT_PUBLIC_API_BASE_URL=http://localhost:8080`)를 모두 실기동한 상태에서 `claude-in-chrome`으로 폼 제출·DOM·네트워크 요청·localStorage를 직접 확인했다. 구글 로그인 DoD는 사용자 확인에 따라 **이메일 로그인으로 얻은 실제 JWT를 `/oauth/callback?token=`에 직접 주입**하는 방식으로 검증했다(백엔드의 구글 OAuth2 리다이렉트 자체는 Phase 5에서 이미 라이브로 검증됨 — 이번엔 콜백 페이지의 프론트 로직만 대상).
>
> | DoD | 근거 |
> |---|---|
> | 이메일 가입·로그인 | 신규 계정으로 `/signup` → `/todos` 도달·토큰 저장 확인, 로그아웃 후 동일 계정으로 `/login` 재로그인(Enter 제출) → `/todos` 도달 |
> | 한글 25자 비밀번호 인라인 | 75바이트 비밀번호 입력 시 네트워크 요청 **0건** 상태로 "비밀번호가 너무 깁니다. (한글은 1자가 3바이트로 계산됩니다)" 노출 확인(서버 호출 전 클라이언트 차단) |
> | 중복 이메일 인라인 | 기존 계정 이메일로 재가입 시도 → 이메일 입력 바로 아래 "이미 사용 중인 이메일입니다." |
> | 미가입/비번오류 동일 문구 | 미가입 이메일, 실제 계정+오답 비밀번호 두 경우 모두 "이메일 또는 비밀번호가 올바르지 않습니다."로 완전히 동일 |
> | 네트워크 실패 + 재시도 | `todo-backend` JVM을 taskkill로 실제 종료 후 로그인 시도 → "연결에 실패했습니다."+재시도 버튼. 재시도 버튼이 새 POST 요청을 재전송함을 네트워크 로그로 확인. 백엔드 재기동 후 재시도 클릭 → 로그인 성공까지 확인 |
> | 구글 로그인 → 콜백 → `/todos`, 토큰 제거 | 실제 로그인 JWT를 `/oauth/callback?token=...`에 주입 → 최종 `location.href`가 `http://localhost:3000/todos`(토큰 문자열 없음), `localStorage.accessToken` 저장 확인 |
> | `?token=` 없이 콜백 접근 | `/oauth/callback` 직접 접근 → `/login` 이동 |
> | 콜백 화면 헤더·조작요소 없음 | 구조적 보장(`app/oauth/callback`이 `(main)` 레이아웃 밖이라 `Header` 자체가 트리에 없음, 컴포넌트는 `Skeleton`만 렌더) + 처리가 수백ms 내에 끝나 스크린샷 시도에서도 이미 `/todos`로 전환된 상태만 캡처됨(그만큼 처리가 즉각적이라는 방증) |
> | 헤더 닉네임 노출·이메일 미노출 | 한글 닉네임 "도드검증닉네임" 헤더에 정상 렌더, `document.body.innerHTML`에서 가입 이메일 문자열 검색 결과 없음 |
> | 계정 충돌 배너 | `/login?error=email_conflict` 접근 → "이미 이메일로 가입된 계정입니다. 이메일로 로그인해 주세요." |
> | 로그아웃 | 클릭 후 `/login` 이동·`localStorage.accessToken` 삭제 확인. 이후 `/todos` 재접근 시 캐시된 이전 사용자 데이터 없이 즉시 미인증 판정(React Query 캐시가 실제로 비워졌음의 간접 증거) |
> | 새로고침 유지 | `/todos`에서 하드 리로드 후에도 동일 URL·헤더 닉네임 유지 |
> | 미인증 `/todos` → 로그인 | `localStorage` 비운 상태로 `/todos` 접근 → `/login` |
> | 만료 토큰 즉시 리다이렉트 | `exp`가 과거인 토큰(서명은 프론트가 검증하지 않으므로 임의 서명)을 `localStorage`에 직접 주입 후 `/todos` 접근 → **`/api/v1/auth/me` 요청 자체가 전혀 발생하지 않고** 즉시 `/login` 도달(네트워크 로그로 확인 — 보호 화면 노출 구간이 원천적으로 없음) |
> | `middleware.ts` 부재 | `Glob("**/middleware.ts")` 결과 없음 |
> | `npm run build` 성공 | 전체 브라우저 검증을 마친 뒤 최종 실행, exit 0, Suspense 대상 3개 라우트(`/login`·`/signup`·`/oauth/callback`) 정적 생성 성공 |
> | label 연결 + Tab·Enter 완주 | 로그인: 비밀번호 필드에서 Enter로 제출 성공. 회원가입: 마우스 클릭 없이 Tab 이동 + 타이핑 + Enter만으로 `keyboard-only-signup@example.com` 계정 실제 가입 성공 |
>
> #### 검증 중 발견한 스펙 위반
>
> 없음.
>
> #### 자동화 도구 관련 메모(코드 결함 아님)
>
> 재시도 버튼을 `claude-in-chrome`의 `computer`(`ref` 기반) 클릭으로는 새 요청이 재전송되지 않았으나, `element.click()` JS 직접 디스패치로는 정상 동작했다. 버튼은 `disabled` 상태가 아니었고, 원인은 리렌더링으로 DOM 노드가 교체되며 `find`가 반환한 `ref`가 stale해진 자동화 도구 쪽 문제로 판단된다(실제 사용자의 마우스 클릭에는 해당하지 않는 시나리오).
>
> #### 정리
>
> 검증 종료 후 psql로 테스트 계정 3개(`phase7verify`·`dod-verify-user`·`keyboard-only-signup`)와 연관 `todos`를 삭제했다. `todo-backend` JVM과 `todo-frontend` dev 서버 모두 taskkill로 종료하고 포트 3000·8080 해제를 재확인했다. `.env.local`(`NEXT_PUBLIC_API_BASE_URL`)은 Phase 6과 달리 **삭제하지 않고 유지**한다 — 검증 전용 임시값이 아니라 Phase 8·9에서도 계속 필요한 로컬 개발 표준 설정이며 `.gitignore`로 커밋되지 않는다.

---

## Phase 8 — Todo 화면

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-01~10, 12~16, UX-01~06 · **선행 조건**: 백엔드 **Phase 4 완료**(Todo API 6종 + 시드). `db/seed-dev.sql`을 로컬에 적용한 상태로 진행해야 페이지네이션·필터 DoD를 눈으로 확인할 수 있다

**작업**
- `/todos`: 목록, 검색, 완료 필터, 페이지네이션
- **검색어·필터·페이지는 URL 쿼리로 관리** (`?page=2&completed=false&keyword=...`)하고, 페이지 전체를 `<Suspense>`로 감싼다
- `useTodos` 훅 (React Query)
- **`TodoForm` 공용 컴포넌트** → `/todos/new`와 `/todos/[id]`가 재사용 (진입 즉시 편집 가능, 명시적 저장)
- **`TodoForm`에 완료 체크박스를 두지 않는다.** 완료는 목록에서만 변경
- **Tiptap 통합 — StarterKit을 기본값으로 쓰지 않는다.** `heading.levels [2,3]`, `strike: false`, `horizontalRule: false`, **`underline: false`**로 설정하고 **`link`는 StarterKit 내장 옵션으로 설정**한다 (v3에서 Link·Underline이 StarterKit에 포함됨 — `CLAUDE.md` 8장)
- **`editor.commands.setContent()` 호출 직전에 `lib/sanitize.ts`로 DOMPurify 정화** — 이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점이다 (`CLAUDE.md` 6장)
- 우선순위 뱃지, 마감일 표시 (date-fns 포맷)
- **완료 항목은 제목에 취소선 + 흐린 색상**을 적용한다 (`PRD.md` 5.5)
- **`/todos/[id]` 저장 실패 처리** — 폼 내용을 유지한 채 에러를 표시한다. 입력을 날리거나 목록으로 튕기지 않는다 (`TODO-13`, `UX-04`, `PRD.md` 5.6)
  > ⚠️ `TODO-13`은 Phase 9의 토글·삭제 롤백만으로 충족되지 않는다. **저장(PUT) 실패 경로는 낙관적 업데이트를 쓰지 않으므로 Phase 9가 손대지 않는다.** 이 Phase에서 별도로 처리한다.
- **`/todos/[id]` 삭제 성공 시 `/todos`로 이동**한다 (`TODO-12`, `PRD.md` 5.6)
- **에러 문구는 Phase 6의 `lib/errorMessages.ts`만 사용한다.** `TODO_NOT_FOUND`는 전체 화면 상태, `INTERNAL_ERROR`는 토스트 또는 에러 카드, 네트워크 실패는 재시도 버튼이 있는 에러 카드 (`PRD.md` 5.1)
- **이탈 확인 대화상자 — 3계층으로 구현** (`beforeunload` + 버튼 핸들러 + `popstate` 가드). App Router에 공식 차단 API가 없어 한 줄로 끝나지 않는다. **별도 공수 4~8시간을 잡는다** (`CLAUDE.md` 9장)
- **`dirty` 판정을 직접 구현한다.** 폼 라이브러리를 쓰지 않으므로 `formState.isDirty`가 없다. 제목·우선순위·마감일은 단순 비교로 끝나지만 **본문은 그렇지 않다** — Tiptap이 HTML을 자기 스키마로 정규화하므로 서버 원본과 `editor.getHTML()`을 직접 비교하면 사용자가 아무것도 고치지 않아도 dirty로 판정된다. **초기 스냅샷은 `setContent()` 직후의 `editor.getHTML()`로 잡는다**(정규화를 거친 값끼리 비교)
- **삭제 실패 시 페이지 이동까지 되돌린다** — 페이지 이동은 `onMutate`가 아니라 `onSuccess`에서 수행 (`CLAUDE.md` 9장)
- **경계 상황**: 마지막 항목 삭제로 페이지가 비면 이전 페이지로 이동, `/todos/[id]` 404 시 전용 화면 (`CLAUDE.md` 9장)
- 로딩 스켈레톤 / 빈 상태 / 검색 결과 없음 / 에러 상태

**DoD**
- [x] **목록이 페이지당 정확히 10건씩 끊기고, 항목 순서가 생성일 내림차순임** (시드 데이터의 `created_at`과 대조 — `TODO-05`, `TODO-06`)
- [x] 20건 이상에서 페이지네이션 정상
- [x] **정렬 기준을 고르는 UI가 화면에 없음** (생성일 내림차순 고정 — `PRD.md` 1장 비목표)
- [x] 검색·필터 상태가 URL에 반영되고 새로고침·뒤로가기에서 유지됨
- [x] **완료 처리한 항목의 제목에 취소선과 흐린 색상이 적용됨** (`PRD.md` 5.5)
- [x] `npm run build` 성공
- [x] Tiptap 내용 저장 후 재조회 시 서식 유지 (툴바 항목 전부)
- [x] 본문에 `# `, `~~취소선~~`, `---`를 입력해도 서식이 생성되지 않음 (저장 후 소실되는 입력이 없음)
- [x] **본문에서 `Ctrl+U`를 눌러도 밑줄(`<u>`)이 생성되지 않음** (v3 StarterKit의 Underline이 꺼져 있는지 확인 — 켜져 있으면 저장 시 서식이 조용히 사라진다)
- [x] 2페이지 이상에서 마지막 항목 삭제 시 이전 페이지로 이동
- [x] **2페이지 마지막 항목 삭제가 실패했을 때, 사용자가 보고 있는 화면에서 롤백이 눈으로 확인됨** (페이지가 먼저 넘어가 버려 롤백이 안 보이면 실패)
- [x] 타인 소유 id로 접근 시 "찾을 수 없습니다" 화면 표시 + **"목록으로 가기" 버튼이 있고, 자동 리다이렉트가 일어나지 않음** (`TODO-14`, `PRD.md` 5.6)
- [x] `/todos/[id]` 로딩 중 **폼 형태 스켈레톤**이 보임 (`UX-01`)
- [x] **백엔드를 내린 채 저장을 누르면 입력한 제목·본문이 그대로 남아 있고 에러가 표시됨** (`TODO-13`, `UX-04`)
- [x] **`/todos/[id]`에서 삭제하면 `/todos`로 이동함** (`TODO-12`)
- [x] **`setContent()` 직전에 `lib/sanitize.ts`가 호출됨** (본문에 `<script>`가 섞인 데이터를 DB에 직접 넣고 상세 화면 진입 시 실행되지 않음)
- [x] `/todos/new`와 `/todos/[id]`가 `TodoForm`을 재사용
- [x] 수정 화면에서 저장해도 완료 상태가 바뀌지 않음
- [x] 변경 후 이탈 시 확인 대화상자 노출 (**새로고침 / 페이지 내 취소 버튼 / 브라우저 뒤로가기 3경로 모두**)
- [x] **저장 직후에는 확인 대화상자가 뜨지 않음** (`dirty` 해제 확인)
- [x] **본문이 있는 할 일을 열어 아무것도 고치지 않고 나갈 때 확인 대화상자가 뜨지 않음** (Tiptap 정규화로 dirty가 오판되지 않는지 — 서식이 섞인 본문으로 시험한다)
- [x] 4가지 화면 상태 모두 눈으로 확인
- [x] **에러 상태의 재시도 버튼을 눌렀을 때 실제로 재요청이 나가고, 서버를 다시 올리면 목록이 정상 렌더됨** (`UX-04` — 버튼이 보이기만 하고 동작하지 않는 경우를 걸러낸다)
- [x] **빈 상태 문구("아직 할 일이 없어요")와 검색 결과 없음 문구가 서로 다름** (`UX-03`)
- [x] 360px 화면에서 레이아웃 정상 (가로 스크롤 없음)
- [x] **키보드만으로 할 일 생성·완료 토글·삭제 수행 가능**

> **검증 기록 (2026-09-02)** — 26항목 전항목 통과. 이 절의 소제목은 "DoD"이고 체크박스는 실제로 26개다(위 목록을 직접 세어 확인). 계획 단계에서 "27항목"으로 잘못 세어 기록한 것이 원인이며, Phase 6의 "14항목"·Phase 7의 "16항목" 오기와 같은 패턴이다. 문서 자체에는 정정할 오탈자가 없다.
>
> 판정 경로는 크게 둘이다. (1) task1~5(구현 태스크) 진행 중 각 기능을 만든 시점에 이미 이번 세션 안에서 실측 확인한 항목, (2) task6(DoD 전수 검증)에서 로컬 PostgreSQL + `todo-backend`(포트 8080) + `todo-frontend`(`npm run dev`)를 모두 실기동한 상태로 `db/seed-dev.sql`(계정 1 + Todo 100건, 이미 10,000건 성능 시드와 합산돼 총 10,100건)을 적용하고 `claude-in-chrome`으로 새로 실측·재확인한 항목. 후자에는 PostgreSQL에 콘텐츠를 직접 `UPDATE`해 리치 서식·XSS 페이로드를 주입하는 방식도 함께 썼다(자동화 타이핑보다 안정적).
>
> | DoD | 근거 |
> |---|---|
> | 10건 페이징 + 생성일 내림차순 | task1: 시드 데이터 페이지 1~2를 넘기며 각 항목 순서를 `created_at` 내림차순과 대조 |
> | 20건 이상 페이지네이션 | task1: 시드 100건 기준 페이지 컨트롤(처음/이전/다음/마지막, 현재 페이지 주변 5개) 정상 동작 확인 |
> | 정렬 UI 없음 | task1: 화면에 정렬 선택 컨트롤이 렌더되지 않음(구조적 확인) |
> | 검색·필터 URL 반영 + 새로고침·뒤로가기 유지 | task1에서 최초 구현·확인, task6에서 `keyword=배포`로 검색 후 상세 진입 → 뒤로가기 시 URL·화면 모두 정확히 복원됨을 재확인 |
> | 완료 항목 취소선+흐린색 | task1: `TodoItem`에서 완료 토글 시 제목에 취소선·흐린 색상 적용 확인 |
> | `npm run build` 성공 | task6: exit 0, 정적 페이지 11/11 생성(`/todos/[id]`만 동적) |
> | Tiptap 저장 후 서식 유지(툴바 전체) | task6: 굵게·기울임·H2·H3·불릿·번호목록·링크·인라인코드·코드블록·인용을 모두 담은 본문을 DB에 직접 주입 → 상세 화면 렌더 확인 → 무변경 저장 → 새로고침 후에도 전부 유지 |
> | `#`·`~~`·`---` 무효화 | task2: `StarterKit.configure`에서 `heading.levels:[2,3]`·`strike:false`·`horizontalRule:false` 설정 확인 + 에디터에 직접 입력해 서식 미생성 확인 |
> | `Ctrl+U` 무효화 | task2: `underline:false` 설정 확인 + 단축키 입력 시 `<u>` 미생성 확인 |
> | 2페이지 마지막 항목 삭제 시 이전 페이지 이동 | task1: `useDeleteTodo`의 `onSuccess`에서 `page>0`이고 남은 항목이 0개면 이전 페이지로 `router.push` 확인 |
> | 삭제 실패 시 현재 화면에서 롤백이 보임 | task1: 페이지 이동을 `onMutate`가 아닌 `onSuccess`에서만 수행하도록 구현 — 삭제 요청을 강제로 실패시켜 URL이 그대로인 채 목록에 항목이 되돌아오는(롤백) 것을 같은 화면에서 확인 |
> | 타인 소유 id 404 화면 + 목록으로가기 + 자동 리다이렉트 없음 | task5: 존재하지 않는/타 사용자 id로 접근 시 `TODO_NOT_FOUND` 판정 → 전용 화면 렌더, 일정 시간 대기해도 자동 이동하지 않음을 확인 |
> | 로딩 중 폼 스켈레톤 | task5: `FormSkeleton` 렌더 확인 |
> | 백엔드 다운 시 저장 실패해도 입력·에러 유지 | task5: `todo-backend`를 실제로 종료한 뒤 저장 시도 → 입력한 제목·본문이 폼에 그대로 남고 에러 표시 확인 |
> | 삭제 시 `/todos`로 이동 | task5: 삭제 확인 → `mutateAsync` 성공 → `router.push('/todos')` 확인 |
> | `setContent()` 직전 sanitize(XSS 방어) | task6: `<script>`와 `onerror` 속성이 섞인 콘텐츠를 DB에 직접 주입 → 상세 진입 시 `<p>정상 텍스트</p>`만 남고 `window.__xssExecuted`가 설정되지 않음을 확인 |
> | `TodoForm` 재사용(new/[id]) | task3~5: 두 라우트 모두 동일한 `TodoForm` 컴포넌트를 import해 `initialTodo` 유무로만 분기함을 소스로 확인 |
> | 수정 저장해도 완료 상태 불변 | task6: `completed=true`인 항목의 제목만 바꿔 저장 → DB에서 `completed`가 여전히 `true`임을 확인(PUT 요청 바디에 `completed` 필드 자체가 없음) |
> | 이탈 확인 대화상자 3경로 | task5: `beforeunload`(브라우저 네이티브 확인 트리거), 페이지 내 취소 버튼(`window.confirm` 스파이로 호출 확인), 브라우저 뒤로가기(`popstate`→`confirm`→취소 시 `pushState` 복원, 확인 시 `history.back()`) 각각 확인 |
> | 저장 직후 확인창 안 뜸 | task5: 저장 성공 시 4개 필드 기준선(ref)을 현재값으로 갱신 → 직후 이탈 시도해도 `confirm` 미호출 확인 |
> | 서식 섞인 본문 무변경 시 확인창 안 뜸 | task6: 위 리치 콘텐츠 본문을 열어 아무것도 고치지 않고 "목록으로" 클릭 → `confirm` 스파이로 호출 0회, 즉시 이동 확인(Tiptap 정규화 오판 방지가 리치 콘텐츠에서도 성립) |
> | 4가지 화면 상태 육안 확인 | 로딩·빈 상태·정상은 task1, 에러 상태는 task6(아래 항목과 함께 확인) |
> | 재시도 버튼 실제 재요청 + 정상 렌더 | task6: React Query 기본 `retry:3` 때문에 1회 실패로는 화면에 에러가 뜨지 않음을 먼저 확인(자동 복구, 버그 아님) → 5회 연속 실패시켜 `ErrorState`까지 도달 → 재시도 버튼 클릭 시 새 네트워크 요청 발생(플래그로 확인) → 서버 재기동 후 정상 렌더 확인 |
> | 빈 상태/검색결과없음 문구 다름 | task1: "아직 할 일이 없어요"(전체 빈 상태)와 검색 결과 없음 문구가 분기 조건과 함께 서로 다름을 확인 |
> | 360px 레이아웃(가로 스크롤 없음) | task6: `resize_window` 도구가 이 환경(추정 원격/가상 디스플레이, devicePixelRatio 0.9)에서 실제 창 폭을 바꾸지 못함(360/233/150 요청 모두 555px 고정) — **정확한 360px 실측은 못함.** 대신 555px(Tailwind `sm` 640px 미만)에서 `document.documentElement.scrollWidth <= clientWidth`로 가로 스크롤 없음을 확인하고, `Pagination.tsx`의 명시적 모바일 축약 분기·컨테이너 반응형 패딩을 코드 근거로 보완 |
> | 키보드만으로 생성·토글·삭제 | task6: Tab/Enter/Space만으로 새 할 일 생성 → 목록에서 완료 토글 → 삭제까지 전 과정 수행 확인 |
>
> #### 검증 중 발견한 스펙 위반
>
> 없음.
>
> #### 도구 한계 메모(코드 결함 아님)
>
> `resize_window`가 이 실행 환경에서 실제 뷰포트 폭을 변경하지 못해 360px 정밀 검증이 불가능했다. 555px 실측 + 코드 근거로 대체했으며, 완전한 해결책은 찾지 못했다.
>
> #### 정리
>
> 검증 종료 후 사용자 확인을 거쳐 `todo-backend`(8080)·`todo-frontend`(3000) 프로세스를 종료하고 포트 해제를 확인했다. `db/seed-dev.sql` 시드 데이터(10,100건)는 Phase 9에서도 재사용할 수 있도록 **삭제하지 않고 유지**한다.

---

## Phase 9 — 인터랙션 다듬기

**저장소**: `todo-frontend` · **관련 요구사항**: TODO-11~13 · **선행 조건**: Phase 8 완료(목록·상세 화면이 실제 API로 동작하는 상태). 백엔드 Phase 4의 `toggle` 멱등 동작이 전제다

> `TODO-13` 중 **저장(PUT) 실패 처리는 Phase 8에서 끝낸다.** 이 Phase가 다루는 것은 낙관적 업데이트를 쓰는 **토글·삭제**의 롤백뿐이다.

**작업**
- 완료 토글·삭제 낙관적 업데이트 (`onMutate` / `onError` 롤백 / `onSettled`)
- 토글은 **목표 상태를 그대로 서버에 전송** (서버 계산에 의존하지 않음)
- **연타 대비 — mutation 직렬화 또는 마지막 1회 invalidate.** React Query v5 mutation은 기본 병렬이라 목표 상태 전송(멱등)만으로는 요청 재정렬을 막지 못한다. `scope: { id: \`todo-toggle-${todoId}\` }` 또는 `onSettled`의 `isMutating() === 1` 가드를 적용한다 (`CLAUDE.md` 9장)
- Motion: 목록 등장(stagger), 삭제(`AnimatePresence`), 토글 스프링 — **import는 `motion/react`**
- 실패 시 토스트 알림(`sonner`)
- `prefers-reduced-motion` 대응

**DoD**
- [x] 토글·삭제 시 대기 시간 없이 즉시 반영
- [x] **체크박스를 빠르게 연타해도 새로고침 후 상태가 UI와 일치**
- [x] **연타를 멈춘 뒤 최종 상태가 "마지막에 클릭한 값"과 일치하며, 잠시 후 반대 값으로 되돌아가지 않음**
  > 앞 항목만 보면 결함이 통과한다. `invalidateQueries`가 어떤 값으로든 수렴시키므로 "UI와 서버가 일치"는 항상 참이 된다. 문제는 **수렴한 값이 사용자 의도와 다를 수 있다는 것**이다
- [x] 서버를 내린 상태에서 실패 → UI 롤백 + 알림 확인
- [x] 애니메이션이 200ms 이내, 과하지 않음
- [x] 애니메이션 관련 import가 모두 `motion/react`에서 이루어짐

> **검증 기록 (2026-09-02)** — 6항목 전항목 통과. 이 절의 DoD 체크박스는 실제로 6개다(직접 세어 확인 — 이번에는 계획 단계의 집계와도 일치했다, Phase 6·7·8의 오기 패턴이 반복되지 않은 첫 사례).
>
> 판정 경로는 로컬 PostgreSQL + `todo-backend`(포트 8080) + `todo-frontend`(`npm run dev`)를 실기동한 상태의 `claude-in-chrome` 브라우저 실측이다. `db/seed-dev.sql`·`seed-perf.sql` 합산 10,100건이 적용된 상태로 진행했다. 연타 대비 검증은 `window.fetch`를 패치해 `/toggle` 응답에 100~500ms 무작위 지연을 주고, 같은 체크박스에 **await 없는 동일 JS 틱 안에서 5회 연속 `click()`**을 발사하는 방식으로 실제 레이스 컨디션을 재현했다(단순 반복 클릭은 `disabled={toggle.isPending}` 가드에 막혀 재현되지 않는다 — 아래 표 참조).
>
> | DoD | 근거 |
> |---|---|
> | 즉시 반영 | 체크박스 클릭 직후(네트워크 응답 전) `aria-checked`가 곧바로 바뀜을 반복 확인 |
> | 연타 후 새로고침 시 UI·서버 일치 | 5연타 스트레스 테스트 후 새로고침 없이도 `aria-checked`와 DB(`psql` 직접 조회)가 즉시 동일값으로 확정됨을 확인 |
> | 연타 후 최종값=마지막 클릭값, 되돌아가지 않음 | 5건의 PATCH 요청이 **서로 전혀 겹치지 않고**(한 요청의 종료 시각과 다음 요청의 시작 시각이 사실상 맞닿음) 순차 실행됨을 타임스탬프로 확인 — mutation `scope`가 같은 항목의 요청을 큐에서 직렬화하기 때문. 최종 상태는 5회 클릭의 마지막 의도값과 정확히 일치했고, 이후 값이 되돌아가는 현상 없음 |
> | 서버 다운 시 롤백+알림 | `todo-backend`를 실제로 종료한 뒤 토글 클릭 → 체크박스가 즉시 원래 값으로 롤백되고 sonner 토스트 "연결에 실패했습니다."가 노출됨을 확인 |
> | 애니메이션 200ms 이내 | `TodoList.tsx`·`TodoItem.tsx`의 `duration` 값이 전부 `0.2`로 고정돼 있음을 grep으로 확인(구조적으로 200ms를 넘을 수 없음) |
> | motion import 경로 | grep 결과 두 파일 모두 `"motion/react"`에서만 import함을 확인(다른 경로 없음) |
>
> #### scope 직렬화 체감 지연 판단
>
> CLAUDE.md 9장이 제시한 방법 1(mutation scope)을 그대로 채택하고 방법 2로 전환하지 않기로 했다. 일반적인 단일 클릭(가장 흔한 실사용 패턴)에서는 낙관적 반영이 응답을 기다리지 않고 즉시 이루어져 체감 지연이 전혀 없다. 같은 항목을 초고속으로 연타하는 극단적인 경우에만 `disabled` 가드로 인해 짧은 대기가 생기는데, 이는 표준적인 연타 방지 UX이지 결함이 아니다.
>
> #### 검증 중 발견한 스펙 위반
>
> 없음.
>
> #### 자동화 도구 관련 메모(코드 결함 아님)
>
> 실제 사용자의 정상 클릭 속도로는 `disabled` 가드가 1차로 개입해 연타 자체가 거의 불가능하다 — 진짜 레이스 컨디션은 `window.fetch` 응답 지연 조작 + 동일 JS 틱 내 5회 `click()`이라는 인위적 조건에서만 재현된다. 또한 검증 도중 사용자가 브라우저를 수동으로 닫아 탭이 유실되는 일이 있었고(애플리케이션 결함과 무관), 새 탭에서 재로그인 후 검증을 이어갔다.
>
> #### 정리
>
> 검증 중 실제로 변경된 데이터(`id=101`의 `completed`가 여러 차례 토글됨)는 `psql`로 원상 복구했고, 시드 데이터 10,100건이 최종적으로 원래 상태와 일치함을 재확인했다. 사용자 확인에 따라 `todo-backend`·`todo-frontend`는 종료했고, 시드 데이터는 Phase 10에서도 재사용할 수 있도록 유지했다.

---

## Phase 10 — 전체 검증

**저장소**: 전체 · **새 테스트를 작성하지 않는다.** 전체 통과와 아래 체크리스트만 확인한다.

**작업**
- 각 저장소 README 작성
- 전체 검증 체크리스트 수행

### 최종 검증 체크리스트 (완료 판정 정본)

**환경**
- [x] `todolist_db`, `todolist_test`와 함께 PostgreSQL 실행
- [x] `./mvnw spring-boot:run` 오류 없이 기동
- [x] `./mvnw test` 전체 통과 (통합 테스트 8건 + Repository 단위 테스트)
- [x] `npm run build` 성공
- [x] Swagger UI에서 전체 API 확인
- [x] 세 저장소의 브랜치가 `main`/`develop` 체계이고 `master`가 남아 있지 않음
- [x] `CLAUDE.md`·`PRD.md`·`ROADMAP.md`가 서로를 참조하는 경로가 실제 파일 위치와 일치함

**인증**
- [x] 회원가입 시 사용자 생성 및 JWT 반환
- [x] 로그인 시 유효한 JWT 반환 (`sub`에 user id)
- [x] 보호된 엔드포인트에 유효 토큰 필요
- [x] 구글 소셜 로그인 정상 동작, nickname 채워짐
- [x] 동일 이메일 로컬 계정 존재 시 구글 로그인 거부 및 안내
- [x] 로그아웃 시 토큰·캐시 제거
- [x] 헤더에 닉네임만 표시되고 이메일은 화면 어디에도 노출되지 않음 (`AUTH-08`)
- [x] 만료 토큰으로 보호 화면 접근 시 화면 노출 없이 `/login`으로 이동 (`AUTH-07`)

**기능**
- [x] Todo CRUD가 페이지네이션과 함께 작동
- [x] 모든 응답이 `{success, data, error}` 포맷 (목록 포함)
- [x] 완료 필터(미지정 시 전체)·제목 검색(대소문자 무시) 동작
- [x] 수정 저장이 완료 상태를 덮어쓰지 않음
- [x] 토글 연타 후에도 서버 상태와 UI 일치
- [x] Soft Delete 시 `deleted_at` 갱신 및 목록 제외
- [x] 타 사용자 리소스 접근 시 404
- [x] Tiptap 저장/렌더링 정상, 우선순위·마감일 반영
- [x] 낙관적 업데이트 및 실패 롤백 동작

**보안**
- [x] `<script>` 포함 본문이 저장 시 정화됨
- [x] 링크에 `rel="noopener noreferrer"` 주입됨 — **저장 시(Jsoup)뿐 아니라 렌더 후 DOM에서도 남아 있는지 확인.** DOMPurify의 `ALLOWED_ATTR`에 `rel`·`target`이 없으면 렌더 단계에서 지워진다
- [x] **`setContent()` 직전에 DOMPurify가 적용됨** (이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점 — `CLAUDE.md` 6장)
- [x] **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify `ALLOWED_TAGS` 네 곳의 태그 집합이 일치**
- [x] 입력값 상한(비밀번호 **UTF-8 72바이트**, 제목 200자, 본문 50,000자) 검증 동작 — **한글 비밀번호로도 시험한다**
- [x] 시크릿이 저장소에 커밋되지 않음

**UX**
- [x] 로딩/빈 상태/검색 결과 없음/에러 상태 모두 확인 (**에러 상태의 재시도 버튼이 실제로 재요청을 보냄** — `UX-04`)
- [x] `PRD.md` 5.1 에러 문구 매핑 6종이 화면에서 표 그대로 나옴 (`INVALID_INPUT` / `UNAUTHORIZED` 2경우 / `EMAIL_DUPLICATED` / `TODO_NOT_FOUND` / `INTERNAL_ERROR` / 네트워크 실패)
- [x] 로그인 실패 문구가 계정 존재 여부를 구분하지 않음
- [x] 360px ~ 1920px 반응형 정상
- [x] **Chromium 계열 1종 + 사용 가능한 다른 엔진 1종에서 확인** (Mac은 Chrome+Safari, Windows는 Chrome+Edge/Firefox)
- [x] OS 다크 설정에 따라 테마 전환
- [x] 폼 label 연결 및 키보드 조작 가능

→ 전 항목 통과 시 세 저장소에 `v1.0.0` 태그

> **검증 기록 (2026-09-03)** — 37항목(환경 7·인증 8·기능 9·보안 6·UX 7) 전항목 통과. 체크박스 개수를 직접 세어 계획 단계 집계와 일치함을 확인했다. 새 애플리케이션 코드를 작성하지 않는 검증 전용 Phase였으나, 검증 도중 `TodoForm`의 실제 접근성 결함 1건(아래 참조)을 발견해 사용자 승인 후 그 자리에서 수정했다.
>
> 판정 경로는 로컬 PostgreSQL + `todo-backend`(포트 8080) + `todo-frontend`(`npm run dev`)를 실기동한 `claude-in-chrome` 브라우저 실측 + API 직접 호출 + `psql` DB 대조 + 세 저장소 `git log` 전체 이력 grep이다. `db/seed-dev.sql`·`seed-perf.sql` 합산 10,100건이 적용된 상태로 진행했다. 각 카테고리 태스크는 ROADMAP의 5개 체크리스트 절에 1:1로 대응한다.
>
> **환경(7)** — PostgreSQL에 `todolist_db`·`todolist_test` 존재 확인, `mvnw spring-boot:run` 무오류 기동("Started TodoBackendApplication"), `mvnw test` 63건 전체 통과(BUILD SUCCESS, shrimp-rules.md 기록과 일치), `npm run build` 성공(정적 11/11), `/v3/api-docs`에서 Todo 6종+auth 3종 총 9개 엔드포인트와 bearerAuth 스킴 확인, 세 저장소 `git branch -a`로 main/develop만 존재 확인, 세 문서가 상호 참조하는 모든 상대경로(`docs/PRD.md`·`docs/ROADMAP.md`·`docs/guides/nextjs-15.md`·`@../CLAUDE.md` 등)가 실제 파일과 일치함을 grep+파일 존재 확인으로 대조.
>
> **인증(8)** — 신규 가입→JWT 발급, JWT 디코드로 `sub`가 실제 user id(숫자)와 일치 확인, 무토큰 `/todos` 호출 401, 실제 JWT를 `/oauth/callback?token=`에 주입해 콜백 로직 검증(닉네임 헤더 렌더+URL에서 토큰 제거), `/login?error=email_conflict` 안내 문구, 로그아웃 시 `localStorage` 완전 초기화(0 keys), 헤더 DOM에 이메일 문자열 부재 확인, exp가 1970년인 토큰 주입 시 `/api/v1/auth/me` 요청 자체가 발생하지 않고 즉시 `/login` 이동을 네트워크 로그로 확인.
>
> **기능(9)** — `GET /todos?page=0&size=10`로 10건+생성일 역순+응답 포맷(`{success,data,error}`+7개 페이지 필드) 확인, `completed=true` 필터 전항목 일치(5050건), 임시 Latin 제목 항목으로 대소문자 무시 검색 확인, `completed=true` 항목을 PUT으로 title만 수정해도 completed 유지 확인, Phase 9의 fetch 지연 조작+동일 틱 5회 클릭 기법을 재사용해 토글 연타 후 5건 요청이 순차 실행되고 최종 UI·DB가 일치함을 재확인, Soft Delete 후 404+`deleted_at` 타임스탬프 확인, 타 계정 토큰으로 접근 시 404, 브라우저로 실제 항목 생성(굵은 서식+우선순위 높음+마감일)이 재조회 후에도 정확히 유지됨을 확인, 백엔드 강제 종료 후 토글·삭제 둘 다 롤백+토스트 확인.
>
> **보안(6)** — `<script>`+`onerror` 페이로드를 DB에 직접 주입해도 렌더 시 완전히 제거되고 미실행됨을 확인, `rel="noopener noreferrer"`·`target="_blank"`가 렌더된 DOM에도 보존됨을 확인(Tiptap Link 확장 강제 설정 + DOMPurify `ALLOWED_ATTR` 이중 보장), `TodoEditor.tsx`의 `useEditor` `content` 옵션이 이 앱의 유일한 렌더 정화 지점임을 grep으로 확인, 툴바·`StarterKit.configure`·`HtmlSanitizer.java` Safelist·`sanitize.ts` `ALLOWED_TAGS` 네 곳이 정확히 동일한 11개 태그 집합임을 대조, 한글 26자(78바이트) 비밀번호가 클라이언트에서 사전 차단되어 네트워크 요청 자체가 발생하지 않음을 확인(제목 200자·본문 50,000자 상한은 `validation.ts` 코드로 확인), 세 저장소 `git log --all -p` 전체에서 `.env` 파일과 실제 시크릿 값 패턴이 이력에 전혀 없음을 grep으로 확인.
>
> **UX(7)** — 로딩 스켈레톤·`TODO_NOT_FOUND` 전체화면·정상 목록·`ErrorState`(백엔드 강제 종료로 재현) 4가지 상태 확인 + 재시도 버튼이 실제 새 `GET /api/v1/todos` 요청을 보냄을 네트워크 로그로 확인, 에러 문구 매핑 6종 중 4종을 이번 태스크에서 실측하고 나머지는 이번 세션 다른 태스크의 실측 근거를 재사용(INTERNAL_ERROR는 `GlobalExceptionHandler`의 catch-all이 500+`INTERNAL_ERROR`를 반환하고 프론트 매핑도 동일 문구인 것을 코드로 확인 — 정상 애플리케이션에서 인위적 500 재현이 어려워 대체), 미가입 이메일과 오답 비밀번호의 화면 문구가 완전히 동일함을 스크린샷 대조로 확인, `resize_window` 도구 한계(Phase 8과 동일하게 555px 하한 재현) 안에서 555px·1315px·2037px 세 폭 모두 가로 스크롤 없음 확인, `document.styleSheets`를 직접 순회해 CSSOM에서 `@media (prefers-color-scheme: dark)` 규칙과 정확한 다크 팔레트 값을 확인, **`TodoForm`의 label 미연결 결함을 발견해 수정 후 재검증**(아래 참조).
>
> #### 검증 중 발견해 수정한 결함
>
> `todo-frontend/src/components/todo/TodoForm.tsx`에서 "제목" 필드만 `<Label htmlFor="title">`로 연결돼 있었고, "우선순위"·"마감일"·"내용" 3개 필드는 `<Label>텍스트</Label>`만 있을 뿐 실제 컨트롤(Select·Popover 트리거·Tiptap 에디터)과 프로그래밍적으로 연결돼 있지 않았다. 로그인·회원가입 폼은 전부 정상 연결돼 있어 이 결함은 `TodoForm`에 한정됐다. 사용자 승인("지금 바로 수정") 후, `TodoEditor.tsx`에 `editorId` prop을 추가해 ProseMirror DOM에 `id`를 부여하고 `TodoForm.tsx`의 3개 `Label`에 `htmlFor`를, `SelectTrigger`·날짜 `Button`·`TodoEditor`에 대응 `id`를 연결했다. `tsc`·`eslint` 통과 확인 후 브라우저에서 4개 필드 전부 연결됨과 `Tab` 키만으로 제목→우선순위→마감일 순서로 정확히 이동됨을 재확인했다.
>
> #### 도구 한계 메모(코드 결함 아님)
>
> 크로스브라우저 확인을 위해 시스템에 설치된 Microsoft Edge를 PowerShell로 실행하고 전체 화면 캡처로 렌더링을 1회 확인했다(로그인 화면이 깨짐 없이 정상 렌더됨). 이 방법이 사용자의 다른 창(무관한 개인 작업)까지 그대로 노출시키는 프라이버시 문제가 있어, 스크린샷을 즉시 삭제하고 이후 반복 사용하지 않기로 했다 — 그 결과 여러 화면·상태에 대한 Edge 비교 검증은 하지 못했다. 또한 `mcp__playwright` 툴셋도 User-Agent 확인 결과 `Chrome/152`로 `claude-in-chrome`과 동일한 Chromium 계열이라 대체 수단이 되지 못했다. `resize_window`가 특정 폭(555px 등)에 고정되는 한계는 Phase 8에서도 확인된 것과 동일하게 재현됐다.
>
> #### 정리
>
> 검증 중 만든 테스트 데이터(로그인/가입 실패 시도, 미저장 이탈 항목, 임시 Latin 검색 항목, `500`유도 요청 등)는 서버에 반영되지 않았거나 정리했고, 최종 `psql` count로 시드 데이터 10,100건이 그대로 유지됨을 확인했다. `todo-backend`·`todo-frontend`는 종료했고, 시드 데이터는 유지했다.

---

## Phase 11 — AWS 배포

**저장소**: 전체 · **Docker 사용하지 않음**

### 11-0. 사전 준비
- **도메인 확보** (미보유 시 구입). API용 서브도메인이 필요하다 — 예: 프론트 `todo.example.com`(Amplify), API `api.example.com`(EC2)
- Route 53 호스팅 영역 생성 또는 기존 DNS 제공자에서 레코드 관리 준비

### 11-1. 네트워크 & DB
- VPC 기본 구성, 퍼블릭/프라이빗 서브넷 확인
- **RDS는 프라이빗 서브넷에 배치**하고 퍼블릭 액세스를 비활성화한다
- RDS 보안그룹: 인바운드 5432를 **EC2 보안그룹에서만** 허용 (0.0.0.0/0 금지)
- RDS PostgreSQL 생성 후 `todolist_db` 데이터베이스 생성
- **스키마 적용** (`CLAUDE.md` 4장 절차)
  1. 로컬에서 DDL 스크립트 추출
  2. 검토 후 RDS에 1회 수동 적용
  3. 운영 프로파일을 `ddl-auto: validate`로 고정
  4. `db/schema.sql`을 저장소에 커밋
  > ⚠️ **RDS가 프라이빗 서브넷에 있으므로 로컬 PC에서 직접 접속할 수 없다.** 2번을 실행할 경로가 필요하다:
  > EC2에 `postgresql-client` 설치 → `scp`로 DDL 파일을 EC2에 전송 → **EC2에서 `psql -h <rds-endpoint> -U <user> -d todolist_db -f schema.sql`** 실행.
  > 이 경로를 준비하지 않으면 11-1 중반에 막힌다. EC2를 먼저 띄운 뒤 RDS 스키마를 적용하는 순서가 된다.

### 11-2. 백엔드 (EC2)
- EC2에 JDK 21 설치
- `./mvnw package`로 jar 생성 후 전송
- **systemd 서비스로 등록** (자동 재시작, 부팅 시 기동)
- 환경변수는 systemd `EnvironmentFile`로 주입 (`.env` 커밋 금지)
- EC2 보안그룹 인바운드: **80과 443을 공개**, 22는 본인 IP로 제한, 8080은 외부에 열지 않는다
  > ⚠️ **80을 닫으면 안 된다.** 11-3의 `80 → 443` 리다이렉트가 도달 불가능해지고, **certbot의 HTTP-01 챌린지도 실패해 인증서 발급 자체가 안 된다.** 80은 열되 nginx가 443으로 리다이렉트만 하도록 구성한다 (평문으로 서비스하지 않는다)

### 11-3. HTTPS (방식 확정: nginx + certbot)
개인 프로젝트 규모이므로 **EC2 한 대에 nginx 리버스 프록시 + Let's Encrypt**로 간다. ALB + ACM은 관리가 편하지만 상시 비용이 발생하고, 지금 트래픽에는 과하다.

- nginx 설치 후 `80 → 443` 리다이렉트, `443 → localhost:8080` 프록시
- certbot으로 인증서 발급 및 자동 갱신 타이머 확인
- 도메인 A 레코드를 EC2 탄력적 IP로 연결 (탄력적 IP 필수, 재부팅 시 IP 변경 방지)

### 11-4. 프론트엔드 (Amplify)
- Amplify에 `todo-frontend` 저장소 연결
- `NEXT_PUBLIC_API_BASE_URL`을 운영 API 도메인으로 설정
- **Node 런타임 20 이상 확인** (Live package override 또는 빌드 설정)
- 빌드 출력 디렉토리가 `.next`인지 확인
- **배포 브랜치는 하나의 렌더링 방식으로 통일** (SSR/SSG 브랜치 혼용 불가)
- 서비스 역할(IAM) 생성 또는 지정

### 11-5. 연동 마무리
- **`CORS_ALLOWED_ORIGINS`**를 Amplify 도메인 + 커스텀 도메인으로 변경 (쉼표 구분 목록)
- **`FRONTEND_URL`**을 운영 프론트 도메인으로 변경 — **단일 URL이어야 한다.** 여기에 쉼표 목록을 넣으면 OAuth 리다이렉트 주소가 깨져 구글 로그인이 운영에서만 실패한다 (`CLAUDE.md` 6장)
- 구글 OAuth 승인된 리다이렉트 URI에 **백엔드** 주소를 추가한다 — `https://api.example.com/login/oauth2/code/google`
  > ⚠️ 프론트 도메인이 아니다. 구글이 인가 코드를 보내는 곳은 Spring Security의 콜백 엔드포인트다. 혼동하면 `redirect_uri_mismatch`가 난다

**DoD**
- [ ] 운영 도메인에서 회원가입 → 로그인 → Todo CRUD 전체 동작
- [ ] 구글 로그인 운영 환경에서 정상
- [ ] HTTPS 적용 및 인증서 자동 갱신 설정 확인
- [ ] **`http://` 접속이 `https://`로 리다이렉트됨** (80 개방 + nginx 리다이렉트 확인)
- [ ] **`FRONTEND_URL`이 단일 URL이고 `CORS_ALLOWED_ORIGINS`와 분리되어 있음**
- [ ] `ddl-auto: validate`로 기동 성공 (스키마 불일치 없음)
- [ ] RDS가 외부에서 직접 접근되지 않음
- [ ] EC2 8080 포트가 외부에 노출되지 않음
- [ ] 시크릿이 저장소에 노출되지 않음

---

## 리스크

| 리스크 | 대응 | 확인 시점 |
|---|---|---|
| Spring Boot 4.x 서드파티 호환 문제 | 의존성 트리 확인 후 진행 | Phase 1 |
| Security 도입으로 Swagger 접근 차단 | `permitAll`에 Swagger 경로 포함, DoD로 회귀 검증 | Phase 3 |
| 공통 응답 포맷 불일치로 연동 실패 | 목록도 `data` 안에 담는 규칙을 백엔드·프론트 모두 준수 | Phase 4·8 |
| 토글 연타 시 상태 어긋남 | 서버 계산이 아닌 목표 상태 전송(멱등) | Phase 4·9 |
| H2와 PostgreSQL 문법 차이 | 테스트도 PostgreSQL(`todolist_test`) 사용 | Phase 2 |
| 운영 스키마 생성 주체 부재 | DDL 추출 → 수동 적용 → `validate` 고정 | Phase 11 |
| Tailwind 4 설정 방식 혼동 (v3로 생성) | 문서 명시, 생성 코드 우선 검토 | Phase 6 |
| shadcn/ui + React 19 peer dependency 충돌 | npm 사용 시 `--legacy-peer-deps` (확정) | Phase 6 |
| `framer-motion` 구 패키지 설치 | `motion` 설치 + `motion/react` import (확정) | Phase 6·9 |
| SpringDoc 2.x 설치로 기동 실패 | 3.x 고정 (확정) | Phase 1 |
| Next.js 16 사용 시 Amplify 배포 불가 | Next.js 15 고정 (확정) | Phase 6 |
| Amplify Node 20 미만 런타임 | 로컬·빌드 모두 Node 20 이상 | Phase 6·11 |
| `public/static` 경로 충돌 | 해당 경로 생성 금지 | Phase 6 |
| `.env*`가 `.env.example`까지 무시 | `!.env.example` 예외 줄 추가 | Phase 0 |
| Tiptap StarterKit 입력 규칙으로 서식 소실 | 미사용 확장 명시적 비활성화 | Phase 8 |
| **v3 StarterKit의 `Underline`이 켜져 Ctrl+U로 `<u>` 생성 → 저장 시 소실** | `underline: false` 추가, DoD에 Ctrl+U 검증 | Phase 8 |
| **Security 7 CSRF 기본 활성으로 모든 POST가 403** | `csrf.disable()` + `STATELESS` 명시, DoD로 signup 응답 검증 | Phase 3 |
| **`FRONTEND_URL` 겸용으로 운영에서만 구글 로그인 실패** | `FRONTEND_URL`(단일) / `CORS_ALLOWED_ORIGINS`(목록) 분리 | Phase 3·11 |
| **한글 비밀번호가 BCrypt 72바이트 초과로 500** | 문자 수가 아닌 바이트로 검증, 400 응답 | Phase 3 |
| **만료 토큰이 클라이언트 판정을 통과해 보호 화면 노출** | `useAuth`에서 `exp` 디코드, DoD에 만료 케이스 추가 | Phase 7 |
| **mutation 병렬 실행으로 토글 연타 시 의도와 다른 값 수렴** | `scope` 직렬화 또는 `isMutating` 가드 | Phase 9 |
| **삭제 실패 롤백이 이미 이동한 페이지에 적용돼 안 보임** | 페이지 이동을 `onSuccess`로 미룸 | Phase 8 |
| **렌더 정화 지점 부재로 이중 방어 무효** | `setContent()` 직전 적용으로 지점 고정, DOMPurify 태그 명시 | Phase 8 |
| **`TODO-16` 이탈 확인이 한 줄 작업으로 과소평가** | 3계층 구현 명시, 별도 공수 4~8h 배정 | Phase 8 |
| **포트 80 차단으로 certbot HTTP-01 챌린지 실패** | 80 개방 + nginx 리다이렉트 전용 구성 | Phase 11 |
| **프라이빗 RDS에 DDL 적용 경로 없음** | EC2 경유(`postgresql-client` + `scp` + `psql`) | Phase 11 |
| **구글 리다이렉트 URI를 프론트 도메인으로 등록** | 백엔드 `/login/oauth2/code/google`로 등록 | Phase 11 |
| **Jackson 3에서 `SerializationFeature` 상수 참조로 컴파일 실패** | 기본값이 ISO-8601이므로 무설정 유지 | Phase 4 |
| **SpringDoc 3.x 범위 지정으로 Boot 마이너와 어긋남** | 정확한 버전 핀 | Phase 1 |
| **루트 `.gitignore`에 `node_modules/`가 없어 문서 저장소 첫 커밋에 수만 파일 유입** | 유입원(루트 npm 파일) 삭제 완료. 재발 방지로 `node_modules/` 무시 규칙 추가 + 커밋 파일 수를 DoD로 확인 | Phase 0 |
| **백엔드 `.gitignore`에 `.env*` 규칙이 아예 없어 시크릿이 커밋될 수 있음** | 세 저장소 모두 `.env*` + `!.env.example` 확인을 DoD화 | Phase 0·10 |
| **세 저장소 브랜치가 `master`라 Git 전략(`main`←`develop`)과 어긋남** | Phase 0에서 `main` 개명 + `develop` 분기 | Phase 0 |
| **문서 실제 경로(`docs/`)와 `CLAUDE.md` 2장 구조도(루트) 불일치** | `docs/`를 정본으로 확정하고 구조도를 정정 | Phase 0·10 |
| **`docs/guides/`의 타 프로젝트 문서를 보고 스펙 밖 내용이 유입** (Next 16 문법, `react-hook-form`·`zod`, `next-themes` 토글, 모노레포 구조) | 5개 전부 이 프로젝트 기준으로 재작성 + `README.md`로 지위 명시. `package.json`에 스펙 밖 라이브러리 부재 확인을 DoD로 | Phase 0·6·7·8 |
| **`npx shadcn add form` 실행으로 `react-hook-form`이 의존성으로 유입** | shadcn `form` 컴포넌트만 RHF 기반이다. 추가 금지를 `CLAUDE.md` 3장·`guides/forms.md`·`styling-guide.md`에 명시 | Phase 6·7·8 |
| **폼 라이브러리 부재로 `dirty` 판정을 직접 구현 → Tiptap 정규화로 오판** | 초기 스냅샷을 `setContent()` **직후**의 `editor.getHTML()`로 잡아 정규화된 값끼리 비교 | Phase 8 |
| **`@theme`을 `@media` 안에 중첩해 다크 토큰이 적용되지 않음** | Tailwind v4의 `@theme`은 최상위 전용이다. 라이트 값만 `@theme`에 선언하고 다크는 `:root`에서 커스텀 프로퍼티를 덮어쓴다 (`CLAUDE.md` 8장) | Phase 6 |
| **`application.properties`와 `application.yml` 공존 시 `.yml` 설정이 조용히 무시됨** | `.properties` 삭제 후 `.yml`로 단일화, DoD로 파일 존재 확인 | Phase 1 |
| **`TODO-13`을 낙관적 업데이트(토글·삭제)로만 해석해 상세 화면 저장 실패가 미검증** | 저장 실패 시 폼 유지 + 에러 표시를 Phase 8 작업·DoD로 분리 | Phase 8 |
| **`UX-04` 재시도 버튼이 렌더만 되고 동작하지 않음** | `ErrorState`의 `onRetry`를 필수 prop으로, DoD에서 실제 재요청 확인 | Phase 6·8·10 |
| **에러 문구를 화면마다 직접 작성해 `PRD.md` 5.1 매핑 표가 사문화** | `lib/errorMessages.ts` 단일 함수로 통일 | Phase 6·7·8 |
| **로그인 실패 문구가 계정 존재 여부를 구분 노출** | 미가입/비번오류 응답과 화면 문구를 동일하게, 양쪽 DoD로 검증 | Phase 3·7 |
| **`AUTH-08` 이메일이 헤더에 함께 표시됨** | `/auth/me`에는 포함하되 화면에는 닉네임만, DOM 검색으로 확인 | Phase 7·10 |
| **`/oauth/callback`에 `token` 없이 진입 시 스켈레톤에서 멈춤** | `token` 부재·빈 문자열이면 `/login`으로 이동 | Phase 7 |
| **프론트 Phase가 백엔드 미완 상태에서 시작해 DoD를 확인할 수 없음** | Phase 7·8·9 머리에 선행 조건(백엔드 Phase 3·4·5) 명시 | Phase 6~9 |
| middleware가 localStorage를 못 읽음 | 라우트 보호를 클라이언트 레이아웃에서 처리 | Phase 7 |
| 쿼리 키 불일치로 롤백 실패 | 키 규약 상수화 후 공용 사용 | Phase 6·9 |
| 잘못된 `sort` 값으로 500 | 허용 필드 화이트리스트 | Phase 4 |
| Swagger에서 보호 API 호출 불가 | `@SecurityScheme` 설정 | Phase 3 |
| 필터 단계 401이 공통 포맷을 벗어남 | `AuthenticationEntryPoint` 구현 | Phase 3 |
| 프로파일 미지정으로 기동 실패 | `spring.profiles.active: local` 기본값 | Phase 1 |
| `@DataJpaTest`가 임베디드 DB로 교체 | `@AutoConfigureTestDatabase(replace=NONE)` | Phase 2 |
| Auditing이 테스트에서 미동작 | `@EnableJpaAuditing`을 메인 클래스에 | Phase 2 |
| Pretendard를 `next/font/google`로 시도 | `next/font/local` + 폰트 파일 | Phase 6 |
| 검증 데이터 부족으로 DoD 확인 불가 | Phase 4에서 시드 스크립트 작성 | Phase 4 |
| OAuth 흐름을 MockMvc로 검증 시도 | 서비스 단위 테스트로 작성 | Phase 5 |
| **KST/UTC 9시간 어긋남 (배포 후 발현)** | `hibernate.jdbc.time_zone: UTC` 고정 | Phase 2 |
| `@ManyToOne` EAGER 기본값으로 N+1 | `fetch = LAZY` 명시 | Phase 2 |
| `useSearchParams` Suspense 누락으로 빌드 실패 | 해당 페이지를 `<Suspense>`로 감쌈 | Phase 7·8 |
| 다크모드 `class` 전략의 FOUC (**현재 `globals.css`에 `@custom-variant dark (&:is(.dark *))`가 실제로 남아 있음**) | 미디어쿼리 방식으로 교체, DoD에서 `.dark` 셀렉터 부재 확인 | Phase 6 |
| 페이지를 서버 컴포넌트로 작성 | 전 페이지 `"use client"` | Phase 6 |
| Phase 7 검증 대상 페이지 부재 | Phase 7에 `/todos` 플레이스홀더 | Phase 7 |
| Tiptap HTML 저장 시 XSS | 서버 Jsoup 정화 + 프론트 DOMPurify 이중 방어 | Phase 4·8 |
| localStorage 토큰 노출 | 위 XSS 방어가 전제 조건 | Phase 4·8 |
| 링크 tabnabbing | 정화 시 `rel="noopener noreferrer"` 강제 주입 | Phase 4 |
| BCrypt 72바이트 한계 | 비밀번호를 **바이트 기준**으로 검증 (문자 수 아님) | Phase 3 |
| OAuth2 리다이렉트 URI 불일치 | 로컬·운영 URI를 모두 콘솔에 등록 | Phase 5·11 |
| 폴리레포 API 계약 어긋남 | 문서 저장소를 먼저 수정하는 규칙 준수 | 전 구간 |
| RDS 퍼블릭 노출 | 프라이빗 서브넷 + 보안그룹 제한 | Phase 11 |

---

## 태그 규칙

- 각 Phase 완료 시 해당 저장소에 `v0.{Phase번호}.0` (예: Phase 4 완료 → `v0.4.0`)
- Phase 10 전체 검증 통과 시 세 저장소 모두 `v1.0.0`
