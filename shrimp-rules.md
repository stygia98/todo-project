# 개발 규칙 (shrimp-rules.md)

> **대상**: 이 저장소에서 작업하는 Coding Agent 전용. 사람을 위한 튜토리얼이 아니다.
> **최종 검증**: 2026-08-28 · 실제 파일 스캔 기준
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

## 3. 현재 상태 스냅샷 (2026-08-28 검증)

작업 착수 전 이 표와 실제 상태가 같은지 확인한다. 다르면 이 절을 먼저 갱신한다.

| 저장소 | 브랜치 | 커밋 | 원격 | 비고 |
|---|---|---|---|---|
| `todo-project` | `main` (unborn) | 0 | `origin` (푸시 전) | 전 파일 untracked. `develop`은 첫 커밋 후 생성 |
| `todo-backend` | `main` (unborn) | 0 | `origin` (푸시 전) | 전 파일 untracked. `develop`은 첫 커밋 후 생성 |
| `todo-frontend` | `main` (unborn) | 0 | `origin` (푸시 전) | 전 파일 untracked. `develop`은 첫 커밋 후 생성 |

**세 저장소가 완전히 같은 출발선이다.** `todo-frontend`에 있던 커밋 1건(`Initial commit from Create Next App`)과 `develop` 브랜치는 **2026-08-28에 제거했다**(3.2 참조). 작업 트리 파일은 그대로다.

⚠️ **커밋이 0건인 저장소에는 `develop`을 만들 수 없다.** 브랜치는 커밋을 가리키는 포인터이므로 가리킬 대상이 없다. **세 저장소 모두** 첫 커밋 직후 `git branch develop main`을 실행한다.

**구현된 소스 (이것이 전부다)**

```
todo-backend/src/main/java/com/example/TodoBackendApplication.java   # @SpringBootApplication 뿐
todo-backend/src/main/resources/application.properties               # yml로 교체 대상
todo-backend/src/test/java/com/example/TodoBackendApplicationTests.java

todo-frontend/src/app/{layout.tsx,page.tsx,globals.css}              # create-next-app 기본값
todo-frontend/src/components/ui/button.tsx                           # shadcn 버튼 1개
todo-frontend/src/lib/utils.ts                                       # cn()
```

`domain/`, `service/`, `controller/`, `dto/`, `config/`, `exception/` 패키지는 **아직 없다.**
`src/hooks/`, `src/types/`, `src/components/{common,todo}/`도 **아직 없다.**

**진행 Phase**: 0(저장소 초기화) · 1(백엔드 스캐폴딩) · 6(프론트 스캐폴딩) 모두 🟡 진행중.

### 3.1 미해결 부채 — 착수 전 반드시 확인

| # | 항목 | 현재 상태 | 조치 위치 |
|---|---|---|---|
| 1 | 세 저장소 첫 커밋·푸시 | 커밋 **0/0/0건**. 원격 `origin`은 3개 모두 **연결됨**(`git@github.com:stygia98/{todo_project,todo-backend,todo-frontend}.git`)이나 **푸시 전**이다. 첫 커밋 → 푸시 → `develop` 분기가 남았다 | Phase 0 |
| 2 | **DB 비밀번호 평문** | `application.properties`에 `spring.datasource.password`가 하드코딩되어 있고, 이 파일은 아직 **커밋 전**이다 | Phase 1 · yml + 환경변수로 이전한 뒤 커밋 |
| 3 | **DB 대상 불일치** | 현재: DB `postgres` + `currentSchema=todolist_db`. `CLAUDE.md` 12·13장: **DB 자체가 `todolist_db`**(`createdb todolist_db`) | Phase 1 · **영속 규칙은 6.4절**(이 행을 지워도 규칙은 남는다) |
| 4 | 백엔드 설정 분리 | `application.properties` 하나뿐. `application.yml` + `-local` + `-prod` 미생성 | Phase 1 |
| 5 | 백엔드 `CLAUDE.md` | **없다.** 프론트에는 있다 | Phase 1 |
| 6 | 백엔드 `.env.example` | 파일이 아직 없다 (무시 규칙은 준비됨) | Phase 1 |
| 7 | 프론트 다크 모드 | `globals.css`가 `@custom-variant dark (&:is(.dark *))` + `.dark {}` = **`class` 전략.** 스펙은 `@media (prefers-color-scheme: dark)` | Phase 6 |
| 8 | 프론트 디자인 토큰 | shadcn 기본 neutral(oklch). `CLAUDE.md` 8장 팔레트 미적용 | Phase 6 |
| 9 | `components.json` | `"style": "radix-nova"`. 스펙은 `new-york` | Phase 6 |
| 10 | 프론트 미설치 | `motion`, `@tiptap/react`, `@tiptap/starter-kit`, `@tanstack/react-query`, `dompurify`, `date-fns`, `sonner` | Phase 6 |
| 11 | 프론트 `.env.example` | 파일이 아직 없다 (무시 규칙은 준비됨) | Phase 6 |
| 12 | Pretendard 폰트 | `src/app/fonts/` 없음 | Phase 6 |

### 3.2 해소된 부채 (2026-08-28)

| 항목 | 조치 |
|---|---|
| 루트 `.metadata/` 미무시 | 루트 `.gitignore`에 `.metadata/` 추가. 스테이징 대상이 **563개 → 21개**로 줄었다 |
| 루트 `node_modules/` 미무시 | 재발 방지용으로 추가 |
| shrimp 태스크 상태 유입 | `shrimp_data/`를 루트 `.gitignore`에 추가 |
| `.claude/commmands/` 오타 | `.claude/commands/`로 정정. `/git commit` 커맨드가 등록된다 |
| shrimp `DATA_DIR` 오지정 | `.mcp.json`을 `D:\claude\todo-project\shrimp_data`로 변경 |
| 브랜치명 `master` | 세 저장소 모두 `main` |
| `todo-frontend`만 커밋 1건 | 세 저장소의 출발선을 맞추기 위해 **커밋과 `develop`을 제거**해 unborn `main`·0커밋으로 초기화했다. 커밋에만 있던 `AGENTS.md`(Next 16 자동 생성물, 의도적 삭제)와 `app/globals.css`(→ `src/app/globals.css`로 이동) 외에 **소실된 파일은 없다** |
| 원격 미연결 | 세 저장소 모두 `origin` 연결 완료 (푸시는 첫 커밋 이후) |
| `.env` 무시 규칙 | **세 저장소 모두** `.env*` + `!.env.example` 적용. `.env`는 무시되고 `.env.example`은 추적 가능함을 실파일로 검증 |

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

`CLAUDE.md`(**v1.8**) · `docs/PRD.md`(**v1.8**) · `docs/ROADMAP.md`(**v1.9**)는 상단에 **버전·최종 수정일**을 가진다.
내용을 고치면 **그 문서의 버전·수정일을 함께 올린다.** 세 문서의 버전은 서로 독립이며 일치할 필요가 없다.

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

### 6.5 실행 명령 (Windows)

| 셸 | 명령 |
|---|---|
| PowerShell / cmd | `.\mvnw.cmd spring-boot:run` · `.\mvnw.cmd test` |
| Git Bash | `./mvnw spring-boot:run` · `./mvnw test` |

`todo-backend/.gitattributes`(`/mvnw text eol=lf`)는 **이미 있다. 삭제하지 않는다.**

### 6.6 계층 규칙

- 컨트롤러는 **엔티티를 반환하지 않는다.** 항상 DTO(`record`).
- 엔티티에 `@Setter` 금지. 변경은 `updateCompleted(boolean)`·`softDelete()` 같은 의미 있는 메서드로.
- `@ManyToOne`은 **반드시 `fetch = FetchType.LAZY`** 명시(기본값 EAGER).
- 물리 삭제 금지. `deleted_at` 기록 + 모든 조회에 `deleted_at IS NULL`.

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

### 7.5 `globals.css` 정리 대상 (Phase 6)

현재 파일에 있는 다음 두 가지를 **걷어낸다.**

```css
@custom-variant dark (&:is(.dark *));   /* 제거 — class 전략 */
.dark { ... }                            /* → @media (prefers-color-scheme: dark) { :root { ... } } */
```

`@theme`은 **최상위에만** 둔다. `@media` 안에 중첩하지 않는다. 라이트 값을 `@theme`으로 선언해 유틸리티를 만들고, 다크에서는 `:root`에서 **값만 덮어쓴다.**

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

---

## 8. Git 규칙

- **커밋 전 반드시 사용자 승인을 받는다.**
- 커밋 메시지는 **한글**, prefix는 `feat: / fix: / refactor: / test: / docs: / chore:`
- 브랜치: `main`(배포 가능) ← `develop` ← `feature/{작업명}`
  - 세 저장소 모두 unborn `main`(0커밋)이다. **세 저장소 모두 첫 커밋 직후 `git branch develop main`**을 실행해야 한다(3절 참조).
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
