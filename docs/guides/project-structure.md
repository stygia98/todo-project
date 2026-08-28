# 프로젝트 구조 가이드

이 문서는 **폴리레포 3개 저장소**의 폴더 구조와 네이밍 컨벤션을 정의한다.

> ⚠️ **구조의 단일 출처는 [`CLAUDE.md`](../../CLAUDE.md) 2장이다.** 이 가이드가 그 문서와 어긋나면 `CLAUDE.md`를 따른다.

---

## 0. 모노레포가 아니다

**독립된 Git 저장소 3개**다. 하나의 저장소에 워크스페이스로 묶여 있지 않다.

```
todo-project/                    # [저장소 1] 문서
├── .git/
├── .gitignore                   # todo-backend/, todo-frontend/ 제외
├── CLAUDE.md                    # 기술 규칙. 루트에 둔다
├── docs/
│   ├── PRD.md
│   ├── ROADMAP.md
│   └── guides/                  # 참고 자료. 스펙 아님
│
├── todo-backend/                # [저장소 2] 독립 .git/
└── todo-frontend/               # [저장소 3] 독립 .git/
```

### ⚠️ 부모 저장소가 하위 폴더를 추적하면 안 된다

부모가 Git 저장소이면서 하위 폴더도 Git 저장소이므로, **제외하지 않으면 Git이 하위 폴더를 gitlink로 커밋**해 버린다. 클론했을 때 빈 폴더만 남는다.

```gitignore
# todo-project/.gitignore
todo-backend/
todo-frontend/
node_modules/

.DS_Store
*.log
```

### ⚠️ `CLAUDE.md`만 루트에 둔다

Claude Code는 현재 디렉토리에서 **상위로 거슬러 올라가며 `CLAUDE.md`를 찾아 로드**한다. 이 파일을 `docs/`로 옮기면 `todo-backend/`에서 작업할 때 부모 규칙이 로드되지 않는다. 나머지 문서는 전부 `docs/` 아래에 둔다.

각 하위 저장소에도 자체 `CLAUDE.md`를 두되, 첫 줄에서 부모를 임포트한다.

```markdown
@../CLAUDE.md

# todo-frontend
> 전체 스펙의 정본은 위에서 임포트한 부모 문서다.
> 이 문서에는 이 저장소에서만 필요한 규칙만 적는다.
```

### 커밋 규칙

- **백엔드와 프론트엔드의 커밋을 섞지 않는다.** 각 저장소에서 개별 커밋한다.
- **API 계약(`CLAUDE.md` 5장)이 바뀌면 문서 저장소를 먼저 수정**한 뒤 백엔드 → 프론트엔드 순으로 반영한다.
- 브랜치: `main`(배포 가능) ← `develop` ← `feature/{작업명}`
- 커밋 메시지 접두어: `feat:` `fix:` `refactor:` `test:` `docs:` `chore:` — 본문은 **한글**

---

## 1. `todo-frontend` 구조

```
todo-frontend/
├── CLAUDE.md                    # @../CLAUDE.md 임포트 + 저장소 전용 규칙
├── .env.example                 # 값 비운 채 키만 커밋
├── components.json              # shadcn/ui 설정 (style: new-york)
├── next.config.ts
├── eslint.config.mjs            # FlatCompat 방식 (Next 15)
├── public/                      # ⚠️ public/static을 만들지 않는다 (Amplify 예약)
└── src/                         # ⚠️ 소스는 전부 여기 아래
    ├── app/
    │   ├── layout.tsx           # 루트. 유일한 서버 컴포넌트
    │   ├── providers.tsx        # "use client" — React Query, Toaster
    │   ├── globals.css          # Tailwind 4 @theme 토큰
    │   ├── fonts/               # PretendardVariable.woff2
    │   ├── (auth)/
    │   │   ├── login/page.tsx
    │   │   └── signup/page.tsx
    │   ├── oauth/callback/page.tsx
    │   └── (main)/
    │       ├── layout.tsx       # 라우트 보호 + 공통 헤더
    │       └── todos/
    │           ├── page.tsx
    │           ├── new/page.tsx
    │           └── [id]/page.tsx
    ├── components/
    │   ├── ui/                  # shadcn/ui가 생성한 것만. 직접 만든 것을 섞지 않는다
    │   ├── common/              # Pagination, EmptyState, ErrorState, Skeleton, AppHeader
    │   └── todo/                # TodoList, TodoItem, TodoForm, TodoEditor
    ├── hooks/                   # useAuth, useTodos, useTodo
    ├── lib/                     # apiClient, queryClient, queryKeys, sanitize, validation, errorMessages, utils
    └── types/                   # 백엔드 DTO와 이름을 맞춘다
```

### 폴더별 판단 기준

| 폴더 | 넣는 것 | 넣지 않는 것 |
|---|---|---|
| `components/ui/` | `npx shadcn add`가 생성한 파일 | 직접 만든 컴포넌트 |
| `components/common/` | 도메인과 무관하게 재사용되는 것 | Todo를 아는 컴포넌트 |
| `components/todo/` | Todo 도메인을 아는 컴포넌트 | 범용 UI |
| `hooks/` | React Query 래퍼, 인증 상태 | 순수 함수 |
| `lib/` | 순수 함수, 싱글턴 클라이언트 | React 훅 |

**판단이 애매하면 `components/todo/`에 둔다.** 재사용처가 실제로 생겼을 때 `common/`으로 올린다. 쓰이지도 않는 범용 컴포넌트를 미리 만들지 않는다.

### 경로 별칭

```json
// tsconfig.json
{ "compilerOptions": { "paths": { "@/*": ["./src/*"] } } }
```

```ts
// ✅
import { apiClient } from "@/lib/apiClient";
import { TodoItem } from "@/components/todo/TodoItem";

// ❌ 상대 경로 거슬러 올라가기
import { apiClient } from "../../../lib/apiClient";
```

> `components.json`의 경로 설정도 `src/` 기준이어야 한다. 스캐폴딩 이후 `src/`로 옮겼다면 `components.json`의 `css`·`aliases`를 함께 고쳤는지 확인한다.

---

## 2. `todo-backend` 구조

```
todo-backend/
├── CLAUDE.md
├── .gitattributes               # mvnw text eol=lf (Git Bash bad interpreter 방지)
├── .env.example
├── pom.xml
├── mvnw / mvnw.cmd
└── src/
    ├── main/java/com/example/
    │   ├── domain/              # 엔티티, Repository
    │   ├── service/             # 비즈니스 로직, HtmlSanitizer
    │   ├── controller/          # REST API
    │   ├── dto/                 # 요청/응답 record
    │   ├── config/              # Security, JWT, Swagger, CORS
    │   └── exception/           # BusinessException, ErrorCode, GlobalExceptionHandler
    ├── main/resources/
    │   ├── application.yml           # 공통 + spring.profiles.active: local
    │   ├── application-local.yml
    │   ├── application-prod.yml
    │   └── db/                       # seed-dev.sql, seed-perf.sql, schema.sql
    └── test/
        ├── java/com/example/
        └── resources/application-test.yml
```

### 계층 방향

```
controller → service → domain(repository)
     ↓          ↓
    dto        dto
```

- **역방향 의존을 만들지 않는다.** `domain`이 `service`나 `dto`를 알면 안 된다.
- **엔티티를 컨트롤러에서 직접 반환하지 않는다.** 항상 DTO로 변환한다.
- 엔티티에 `@Setter`를 두지 않는다. 변경은 의미 있는 메서드로 한다 (`updateCompleted(boolean)`, `softDelete()`).

> ⚠️ **설정 파일은 `.yml`로 통일한다.** `application.properties`와 `application.yml`이 공존하면 `.properties`가 우선해 `.yml`이 조용히 무시된다. 스캐폴딩이 만든 `.properties`는 삭제한다.

---

## 3. 네이밍 컨벤션

### 프론트엔드

| 대상 | 규칙 | 예 |
|---|---|---|
| 컴포넌트 파일 | `PascalCase.tsx` | `TodoItem.tsx`, `Pagination.tsx` |
| 훅 파일 | `useCamelCase.ts` | `useTodos.ts`, `useAuth.ts` |
| 유틸 파일 | `camelCase.ts` | `apiClient.ts`, `errorMessages.ts` |
| 라우트 폴더 | `kebab-case` 또는 App Router 예약형 | `oauth/callback`, `(auth)`, `[id]` |
| 타입 | 백엔드 DTO와 **이름을 맞춘다** | `TodoResponse`, `PageResponse<T>` |

`components/ui/`만 예외다. shadcn이 `kebab-case.tsx`로 생성하므로 **그대로 둔다.** 손으로 바꾸면 이후 `npx shadcn add`가 중복 파일을 만든다.

### 백엔드

| 대상 | 규칙 |
|---|---|
| 클래스 | `PascalCase` |
| 메서드·변수 | `camelCase` |
| 상수 | `UPPER_SNAKE_CASE` |
| DTO | `TodoCreateRequest`, `TodoResponse` — **record** |

### 공통

- **모든 주석과 문서는 한글**로 작성한다.
- 변수명·함수명은 영어(코드 표준 준수).

---

## ❌ 금지사항

```
❌ src/app/api/**          — Next에 API를 만들지 않는다. 백엔드는 todo-backend뿐이다
❌ src/middleware.ts        — localStorage를 읽지 못한다
❌ public/static/           — Amplify 예약 경로
❌ tailwind.config.js       — Tailwind 4는 CSS-first. globals.css의 @theme를 쓴다
❌ 4단계 이상 중첩          — components/todo/list/item/badge/... 처럼 깊게 파지 않는다
❌ utils/, helpers/, common/utils/ 남발  — lib/ 하나로 충분한 규모다
```

---

## ✅ 구조 체크리스트

- [ ] 부모 저장소 `git status`에 `todo-backend/`·`todo-frontend/`·`node_modules/`가 나타나지 않는다
- [ ] 프론트 소스가 전부 `src/` 아래에 있다
- [ ] `src/middleware.ts`가 없다
- [ ] `public/static/`이 없다
- [ ] `application.properties`가 없고 `.yml` 3개가 있다
- [ ] 각 저장소에 `CLAUDE.md`와 `.env.example`이 있다
- [ ] `.gitignore`에 `.env*` + `!.env.example` 예외 줄이 있다
