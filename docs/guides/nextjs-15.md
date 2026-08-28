# Next.js 15 개발 지침 — `todo-frontend`

이 문서는 이 프로젝트의 프론트엔드(`todo-frontend`, **Next.js 15.5.24**)를 개발할 때 따르는 규칙이다.

> ⚠️ **스택과 규칙의 단일 출처는 [`CLAUDE.md`](../../CLAUDE.md) 3장·9장이다.** 이 가이드가 그 문서와 어긋나면 `CLAUDE.md`를 따른다.
> 이 문서는 "Next.js를 이 프로젝트에서 어떻게 쓰는가"만 다룬다. 무엇을 만드는지는 [`PRD.md`](../PRD.md), 언제 만드는지는 [`ROADMAP.md`](../ROADMAP.md)를 본다.
> **미설치 라이브러리를 `import` 하지 않는다.** `todo-frontend/package.json`을 먼저 확인할 것.

---

## 0. 왜 16이 아니라 15인가

**AWS Amplify Hosting의 compute(SSR) 지원 범위가 Next.js 12~15이기 때문이다.** 16으로 올리면 Phase 11 배포가 불가능해진다.

- `next`와 `eslint-config-next`의 **메이저를 항상 함께 맞춘다.** 하나만 올리면 ESLint 설정이 깨진다.
- App Router · React 19 · Tailwind CSS 4는 모두 15에서 정상 동작하므로, 15를 쓴다고 이 프로젝트가 잃는 기능은 없다.
- 이 저장소는 한 번 Next 16으로 생성됐다가 15로 되돌린 이력이 있다(`ROADMAP.md` 정합성 점검 표). **다시 16으로 올리지 않는다.**

---

## 🚀 필수 규칙 (엄격 준수)

### 1. 이 프로젝트는 "클라이언트 컴포넌트 우선"이다

일반적인 App Router 가이드는 "Server Components 우선"을 권한다. **이 프로젝트는 반대다.** 일반론을 그대로 적용하면 안 된다.

이유는 두 가지다.

1. 인증 토큰이 **localStorage**에 있다. 서버에는 토큰이 없으므로 서버에서 사용자 데이터를 가져올 수 없다.
2. 서버 상태를 **React Query**로 가져온다. 데이터 페칭이 전부 브라우저에서 일어난다.

```tsx
// ✅ 이 프로젝트의 페이지는 전부 클라이언트 컴포넌트다
// src/app/(main)/todos/page.tsx
"use client";

export default function TodosPage() {
  // React Query로 브라우저에서 페칭한다
  const { data, isLoading } = useTodos({ page, completed, keyword });
  ...
}
```

```tsx
// ❌ 금지: 서버 컴포넌트에서 Todo를 미리 가져오려는 시도
// 서버에는 localStorage가 없어 토큰을 읽을 수 없다. 항상 401이 난다.
export default async function TodosPage() {
  const todos = await fetch(`${API}/api/v1/todos`); // 인증 헤더를 만들 수 없다
  ...
}
```

**서버 컴포넌트로 남기는 것은 루트 `src/app/layout.tsx` 하나뿐이다.** Provider(React Query, sonner 등)는 별도 클라이언트 컴포넌트로 분리해 루트 레이아웃이 감싼다.

```tsx
// src/app/layout.tsx — 서버 컴포넌트로 유지
import { Providers } from "./providers";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

```tsx
// src/app/providers.tsx — 클라이언트 경계는 여기서 시작한다
"use client";

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());
  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <Toaster />
    </QueryClientProvider>
  );
}
```

> ⚠️ **루트 레이아웃의 타입은 명시적으로 적는다.** `LayoutProps<"/">` 같은 전역 타입은 **Next 16 전용**이라 15에서 컴파일에 실패한다. 이 저장소가 실제로 겪은 문제다.

---

### 2. `params` · `searchParams`는 Promise다 (Next 15 변경점)

Next 15부터 `params`와 `searchParams`가 **Promise**로 바뀌었다. 서버 컴포넌트에서는 `await`, 클라이언트 컴포넌트에서는 React의 `use()`로 푼다.

```tsx
// 참고: 서버 컴포넌트라면 이렇게 쓴다 (이 프로젝트에는 해당 페이지가 없다)
export default async function Page(props: { params: Promise<{ id: string }> }) {
  const { id } = await props.params;
  ...
}
```

```tsx
// 참고: 클라이언트 컴포넌트에서 props로 받는다면 use()로 푼다
"use client";
import { use } from "react";

export default function Page(props: { params: Promise<{ id: string }> }) {
  const { id } = use(props.params);
  ...
}
```

**다만 이 프로젝트의 `/todos/[id]`는 훅을 쓴다.** 페이지가 이미 클라이언트 컴포넌트이므로 `useParams()`가 더 단순하고, Promise를 다룰 일이 없다.

```tsx
// ✅ 이 프로젝트의 방식
// src/app/(main)/todos/[id]/page.tsx
"use client";
import { useParams } from "next/navigation";

export default function TodoDetailPage() {
  const { id } = useParams<{ id: string }>();
  const { data, isError } = useTodo(id);
  ...
}
```

> 구버전 예제처럼 `params.id`를 **동기로 접근하지 않는다.** 15에서는 경고와 함께 동작하지만 16에서 완전히 제거되며, 이 프로젝트에서는 애초에 `useParams()`를 쓰므로 그럴 일이 없다.

---

### 3. `useSearchParams`는 `<Suspense>` 경계가 **필수**다

**이것을 빠뜨리면 `npm run dev`는 통과하고 `npm run build`에서 실패한다.** 개발 중에는 드러나지 않아 늦게 발견되는 대표적인 함정이다.

정적 프리렌더 단계에서 `useSearchParams`는 가장 가까운 `<Suspense>` 경계까지를 클라이언트 렌더로 미룬다. 경계가 없으면 프리렌더 자체가 막힌다.

해당하는 곳이 **둘**이다.

| 경로 | 읽는 값 |
|---|---|
| `/oauth/callback` | `?token=` |
| `/todos` | `?page=`, `?completed=`, `?keyword=` |

두 페이지 모두 **실제 로직을 내부 컴포넌트로 빼고** 바깥에서 감싼다.

```tsx
// ✅ src/app/(main)/todos/page.tsx
"use client";
import { Suspense } from "react";

function TodosPageInner() {
  const searchParams = useSearchParams(); // 여기서만 읽는다
  ...
}

export default function TodosPage() {
  return (
    <Suspense fallback={<TodoListSkeleton />}>
      <TodosPageInner />
    </Suspense>
  );
}
```

```tsx
// ❌ 금지: 페이지 본체에서 바로 읽기 — 빌드에서 실패한다
"use client";
export default function TodosPage() {
  const searchParams = useSearchParams();
  ...
}
```

> `fallback`은 빈 요소가 아니라 **실제 스켈레톤**을 넣는다. `UX-01`(로딩 중 스켈레톤)이 이 경계에서 함께 충족된다.

---

### 4. 목록 상태는 `useState`가 아니라 URL 쿼리에 둔다

```
/todos?page=2&completed=false&keyword=회의
```

검색어·완료 필터·페이지 번호를 URL에 두면 새로고침해도 상태가 유지되고, 뒤로가기가 자연스럽고, 링크 공유가 된다. React Query 키(`['todos', { page, size, completed, keyword }]`)가 URL 상태와 1:1로 대응해 캐시 대상이 명확해진다.

**이 선택 때문에 위 3번의 Suspense 경계가 선택이 아니라 필수가 된다.**

```tsx
// 쿼리 갱신은 replace로 (히스토리를 오염시키지 않는다)
const router = useRouter();
const pathname = usePathname();
const searchParams = useSearchParams();

function setQuery(next: Record<string, string | undefined>) {
  const params = new URLSearchParams(searchParams.toString());
  for (const [key, value] of Object.entries(next)) {
    if (value === undefined || value === "") params.delete(key);
    else params.set(key, value);
  }
  router.replace(`${pathname}?${params.toString()}`);
}
```

---

### 5. 라우트 구조는 그룹으로 나눈다

```
src/app/
├── layout.tsx              # 루트. 서버 컴포넌트로 유지
├── providers.tsx           # "use client" — Provider 모음
├── fonts/                  # Pretendard .woff2
├── (auth)/
│   ├── login/page.tsx      # "use client"
│   └── signup/page.tsx     # "use client"
├── oauth/callback/page.tsx # "use client" + <Suspense>
└── (main)/
    ├── layout.tsx          # "use client" — 라우트 보호가 여기 있다
    └── todos/
        ├── page.tsx        # "use client" + <Suspense>
        ├── new/page.tsx    # "use client"
        └── [id]/page.tsx   # "use client"
```

- `(auth)`와 `(main)`은 **괄호 그룹**이라 URL에 나타나지 않는다. `/login`, `/todos` 그대로다.
- `/oauth/callback`은 인증 처리 중 화면이므로 **`(main)` 밖**에 둔다. 보호 레이아웃 안에 넣으면 토큰을 저장하기도 전에 미인증으로 판정돼 `/login`으로 튕긴다.

---

### 6. 라우트 보호는 `(main)/layout.tsx`에서 한다 — `middleware.ts`를 만들지 않는다

**토큰이 localStorage에 있으므로 middleware로 보호할 수 없다.** middleware는 서버에서 실행되어 쿠키만 읽는다. 관행대로 만들면 토큰을 읽지 못해 무한 리다이렉트가 나거나 보호가 아예 걸리지 않는다.

```tsx
// ✅ src/app/(main)/layout.tsx
"use client";

export default function MainLayout({ children }: { children: React.ReactNode }) {
  const { status } = useAuth();       // "loading" | "authenticated" | "unauthenticated"
  const router = useRouter();

  useEffect(() => {
    if (status === "unauthenticated") router.replace("/login");
  }, [status, router]);

  // 판정 전에는 스켈레톤. 서버 렌더 결과와 어긋나지 않게 마운트 이후에만 인증 상태를 읽는다
  if (status !== "authenticated") return <PageSkeleton />;

  return (
    <>
      <AppHeader />
      {children}
    </>
  );
}
```

> ⚠️ `useAuth`는 **토큰 존재 여부가 아니라 `exp`를 봐야 한다.** 문자열이 있는지만 검사하면 만료 토큰이 판정을 통과해, 401 왕복 동안 보호 화면이 노출된다(`AUTH-07` 위반). 상세는 `CLAUDE.md` 9장.

---

### 7. Pretendard는 `next/font/local`로 로드한다

**Pretendard는 Google Fonts에 없다.** `next/font/google`로 부르면 실패한다.

```tsx
// src/app/layout.tsx
import localFont from "next/font/local";

const pretendard = localFont({
  src: "./fonts/PretendardVariable.woff2",
  variable: "--font-pretendard",
  display: "swap",
  weight: "45 920",              // 가변 폰트 범위
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko" className={pretendard.variable}>
      <body>{children}</body>
    </html>
  );
}
```

가변 폰트(`PretendardVariable.woff2`) 하나면 충분하다. 굵기별 파일을 따로 넣지 않는다.

---

## ✅ 이 프로젝트의 설정 파일

### `next.config.ts`

```ts
import path from "node:path";
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // 폴리레포이므로 이 저장소를 워크스페이스 루트로 못박는다.
  outputFileTracingRoot: path.join(__dirname),

  // distDir은 설정하지 않는다. Amplify는 빌드 출력이 .next여야 한다 (CLAUDE.md 3장)
};

export default nextConfig;
```

- **`distDir`을 설정하지 않는다.** 출력은 `.next`여야 Amplify가 인식한다.
- `outputFileTracingRoot`는 부모 폴더의 `package-lock.json` 때문에 Next가 워크스페이스 루트를 잘못 추론하던 문제의 대응이다. 원인 파일은 제거됐지만, 재발 시 조용히 어긋나므로 설정은 남긴다.

### `eslint.config.mjs`

**Next 15의 `eslint-config-next`는 flat config를 직접 내보내지 않는다.** `FlatCompat`으로 감싸야 한다. Next 16 방식으로 쓰면 동작하지 않는다.

```js
import { dirname } from "path";
import { fileURLToPath } from "url";
import { FlatCompat } from "@eslint/eslintrc";

const compat = new FlatCompat({ baseDirectory: dirname(fileURLToPath(import.meta.url)) });

export default [...compat.extends("next/core-web-vitals", "next/typescript")];
```

### Tailwind CSS 4

**v4는 CSS-first 설정이다.** `tailwind.config.js`를 만들지 않는다.

```css
/* src/app/globals.css */
@import "tailwindcss";

@theme {
  --color-bg: #FAFAFA;
  --color-card: #FFFFFF;
  --color-fg: #171717;
  /* ... */
}

/* 다크 모드는 미디어쿼리로 처리한다. class 전략을 쓰지 않는다.
   ⚠️ @theme은 최상위에만 올 수 있다. @media 안에 중첩하면 동작하지 않으므로,
      생성된 커스텀 프로퍼티를 :root에서 덮어쓴다. */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0A0A0A;
    --color-card: #171717;
    --color-fg: #FAFAFA;
  }
}
```

> 자세한 내용은 [`styling-guide.md`](./styling-guide.md).

> ⚠️ 스캐폴딩 시점의 `globals.css`에는 `@custom-variant dark (&:is(.dark *))`가 들어 있다. 이것이 **`class` 전략**이다. 테마 토글 UI가 범위 밖이라 클래스를 붙일 주체가 없어, 이대로 두면 다크 모드가 아예 동작하지 않는다. Phase 6에서 미디어쿼리로 교체한다.

---

## ❌ 금지 사항

### Pages Router

```
❌ pages/           — _app.tsx, _document.tsx, getServerSideProps, getStaticProps
✅ src/app/         — App Router만 쓴다
```

### 이 프로젝트에서 쓰지 않는 Next 기능

아래는 Next가 제공하지만 **이 프로젝트에서는 쓰지 않는다.** 튜토리얼에서 보고 끌어오지 않는다.

| 기능 | 쓰지 않는 이유 |
|---|---|
| **Server Actions** (`"use server"`) | API가 별도 Spring 서버다. Next는 API를 제공하지 않는다 |
| **Route Handlers** (`app/api/**`) | 위와 같다. 백엔드는 `todo-backend`뿐이다 |
| `middleware.ts` | localStorage를 읽지 못한다 (6번 참조) |
| `notFound()` / `unauthorized()` / `forbidden()` | 클라이언트 데이터 페칭이라 동작하지 않는다. 404는 **전용 화면 + "목록으로 가기" 버튼**으로 직접 그린다 |
| Parallel Routes (`@slot`) · Intercepting Routes (`(.)`) | 필요한 화면이 없다 |
| ISR · `revalidateTag` · `fetch` 캐시 옵션 | 서버 페칭이 없다. 캐시는 React Query가 담당한다 |
| `experimental.typedRoutes` | 라우트가 6개뿐이라 실익이 없다. 스펙에 없는 설정을 켜지 않는다 |
| `after()` | 서버 실행 경로가 없다 |
| `next/image` 최적화 | 이미지 업로드가 **비목표**다 (`PRD.md` 1장) |

### `public/static` 경로

**만들지 않는다.** Amplify가 배포용으로 예약한 경로다. 정적 파일은 `public/` 바로 아래나 `public/assets/`에 둔다.

### 불필요한 `"use client"`

이 프로젝트는 클라이언트 우선이지만, **그렇다고 전부 붙이라는 뜻은 아니다.** 상태도 이벤트 핸들러도 없는 표시 전용 컴포넌트는 붙이지 않는다. 부모가 이미 클라이언트 경계 안이면 자식은 자동으로 클라이언트에서 실행된다.

```tsx
// ❌ 불필요
"use client";
export function PriorityBadge({ priority }: { priority: Priority }) {
  return <span className={styles[priority]}>{label[priority]}</span>;
}

// ✅ 지시어 없이 둔다. 클라이언트 부모 안에서 그대로 동작한다
export function PriorityBadge({ priority }: { priority: Priority }) {
  return <span className={styles[priority]}>{label[priority]}</span>;
}
```

### `dangerouslySetInnerHTML`

**이 앱에는 사용 지점이 없다.** 상세 화면이 "진입 즉시 편집"이라 본문 HTML은 오직 Tiptap 에디터로만 들어간다. 렌더 단계 XSS 방어는 **`editor.commands.setContent()` 직전의 `lib/sanitize.ts` 호출**이 유일한 지점이다.

```ts
// ✅ 서버에서 받은 본문을 에디터에 주입하기 직전에 정화한다
editor.commands.setContent(sanitizeHtml(todo.content));
```

목록 미리보기 등으로 `dangerouslySetInnerHTML`을 쓰는 화면이 새로 생기면, 그곳도 반드시 같은 함수를 거친다.

---

## 코드 품질 체크리스트

`todo-frontend/package.json`에 정의된 스크립트는 넷뿐이다. **없는 스크립트를 부르지 않는다.**

```bash
npm run dev      # 개발 서버 (http://localhost:3000)
npm run build    # 빌드 — Suspense 경계 누락은 여기서만 드러난다
npm run start    # 빌드 결과 실행
npm run lint     # ESLint
```

작업을 마치기 전 **반드시 `npm run build`를 돌린다.** `npm run dev`만으로는 다음을 검출하지 못한다.

- `useSearchParams`의 `<Suspense>` 경계 누락
- 프리렌더 단계에서만 나는 타입·직렬화 오류

패키지를 설치할 때는 shadcn/ui와 React 19의 peer dependency 충돌 때문에 플래그가 필요하다.

```bash
npm install --legacy-peer-deps
```

---

## 버전 관리 유의사항

- `next`는 **15.5.24**, `eslint-config-next`도 **동일 메이저**로 고정한다. `CLAUDE.md` 3장에서 확정된 사항이므로 재조사하거나 임의로 올리지 않는다.
- Node.js는 **20 이상**(권장 22). Amplify Hosting이 20·22·24만 지원한다.
- 애니메이션 패키지는 **`motion`**이다. `framer-motion`은 개명 전 deprecated 별칭이며, import는 반드시 **`motion/react`**에서 한다.
- **`@tiptap/extension-link`를 설치하지 않는다.** v3 StarterKit에 `Link`가 포함되어 있어 중복 등록이 된다.

메이저 업그레이드가 필요해지면 이 문서와 `CLAUDE.md` 3장을 **함께** 갱신한다.
