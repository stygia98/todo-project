# 스타일링 가이드 — `todo-frontend`

**Tailwind CSS 4 + shadcn/ui(new-york)** 기준이다.

> ⚠️ **디자인 규칙의 단일 출처는 [`CLAUDE.md`](../../CLAUDE.md) 8장이다.** 이 가이드가 그 문서와 어긋나면 `CLAUDE.md`를 따른다.

---

## 0. 디자인 방향

**심플·모던. 장식보다 여백과 타이포그래피로 위계를 만든다.**

| 원칙 | 내용 |
|---|---|
| 면 구분 | **그림자 대신 1px border**(`#E5E5E5`). 그림자는 모달·드롭다운에만 |
| 라운드 | 카드 `rounded-xl`, 버튼·인풋 `rounded-lg` |
| 액센트 | **단일 컬러 하나만** (`#4F46E5`). 색을 늘려 위계를 만들지 않는다 |
| 타이포 | 본문 15px / 항목 제목 16px semibold / 캡션 13px |
| 레이아웃 | 컨테이너 `max-w-3xl`, 패딩 모바일 16px · 데스크톱 24px |

---

## 1. Tailwind CSS 4는 CSS-first다

**`tailwind.config.js`를 만들지 않는다.** v3 방식으로 작성하면 설정이 아예 먹지 않는다.

```css
/* src/app/globals.css */
@import "tailwindcss";

@theme {
  --color-bg: #FAFAFA;
  --color-card: #FFFFFF;
  --color-fg: #171717;
  --color-muted: #737373;
  --color-border: #E5E5E5;
  --color-accent: #4F46E5;

  /* 우선순위 */
  --color-priority-high: #EF4444;
  --color-priority-medium: #F59E0B;
  --color-priority-low: #10B981;
}
```

토큰은 그대로 유틸리티가 된다 — `--color-bg` → `bg-bg`, `text-fg`, `border-border`.

---

## 2. 다크 모드는 **미디어쿼리**로 한다

```css
/* @theme는 위(1번)에서 라이트 값으로 한 번만 선언한다.
   다크는 생성된 커스텀 프로퍼티를 :root에서 덮어쓴다. */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg: #0A0A0A;
    --color-card: #171717;
    --color-fg: #FAFAFA;
    --color-border: #262626;
  }
}
```

### ⚠️ `@theme`을 `@media` 안에 넣지 않는다

**Tailwind v4에서 `@theme`은 반드시 최상위에 있어야 한다.** 공식 문서가 명시한다 — *"Theme variables must be defined at the top level and cannot be nested under other selectors or media queries."*

```css
/* ❌ 동작하지 않는다 */
@media (prefers-color-scheme: dark) {
  @theme { --color-bg: #0A0A0A; }
}
```

`@theme`은 **유틸리티 클래스를 생성하는 지시어**이지 단순한 변수 선언이 아니다. 생성된 유틸리티(`bg-bg` 등)는 `var(--color-bg)`를 참조하므로, **다크에서는 그 커스텀 프로퍼티만 `:root`에서 덮어쓰면** 클래스를 다시 만들지 않고도 색이 바뀐다.

```css
@theme {
  --color-bg: #FAFAFA;        /* 라이트 = 기본값. 유틸리티는 여기서 생성된다 */
}

@media (prefers-color-scheme: dark) {
  :root { --color-bg: #0A0A0A; }   /* 값만 교체 */
}
```

### ⚠️ `class` 전략을 쓰지 않는다

**테마 토글 UI는 MVP 범위 밖**이다(`UX-08`은 P2). 토글이 없는데 `class` 전략을 쓰면 두 가지가 동시에 문제가 된다.

1. **클래스를 붙일 주체가 없어 다크 모드가 아예 동작하지 않는다.**
2. 억지로 동작시키려면 서버 렌더 시점에 클래스가 없어 라이트로 그려졌다가 클라이언트에서 다크로 바뀌는 **깜빡임(FOUC)**이 생기고, 이를 막으려면 `<head>`에 인라인 스크립트를 넣어야 한다.

미디어쿼리는 CSS만으로 처리되어 하이드레이션 문제가 아예 없다.

```css
/* ❌ 스캐폴딩이 넣어둔 것 — Phase 6에서 제거한다 */
@custom-variant dark (&:is(.dark *));
```

- **`next-themes`를 설치하지 않는다.** 스택에 없고, 토글이 없으므로 쓸 데도 없다.
- 나중에 토글을 추가할 때 `class` 전략으로 전환한다. 그때가 되면 FOUC 대응이 함께 필요하다.

### 다크 대응 스타일 작성법

토큰이 이미 양쪽으로 정의되어 있으므로 **컴포넌트에서는 `dark:` 접두어를 거의 쓰지 않는다.**

```tsx
// ✅ 토큰만 쓰면 양쪽 테마가 자동으로 처리된다
<div className="bg-card text-fg border border-border rounded-xl p-4">

// ❌ 하드코딩 + dark: 남발 — 토큰 체계를 무너뜨린다
<div className="bg-white dark:bg-neutral-900 text-black dark:text-white">
```

---

## 3. shadcn/ui

```bash
npx shadcn@latest add button input label checkbox select calendar popover
```

> ⚠️ **npm은 shadcn/ui + React 19에서 peer dependency 충돌이 난다.** `npx shadcn add`는 내부적으로 설치를 수행하므로 여기서도 막힌다. **`.npmrc`에 `legacy-peer-deps=true`를 두어** 이후의 모든 npm 동작이 같은 설정을 따르게 하는 편이 안전하다. 직접 설치할 때는 `npm install --legacy-peer-deps`를 쓴다. (pnpm·yarn·bun은 해당 없음)

| 설정 | 값 |
|---|---|
| 스타일 | **`new-york`** |
| 설치 플래그 | npm이면 `--legacy-peer-deps` (pnpm·yarn·bun은 불필요) |
| 토스트 | **`sonner`** — `toast` 컴포넌트는 deprecated다 |

### ⚠️ `form` 컴포넌트를 추가하지 않는다

`form`만 `react-hook-form` 위에 만들어져 있어, 추가하면 스택에 없는 라이브러리가 의존성으로 딸려 들어온다. 폼은 `label` + `input`을 직접 조합한다 → [`forms.md`](./forms.md)

### 커스터마이징은 `cn()`으로 병합한다

```tsx
import { cn } from "@/lib/utils";

<Button className={cn("w-full", isDanger && "bg-priority-high")} />
```

`components/ui/`의 파일을 직접 고치는 것은 **가능하지만 최소한으로 한다.** 고쳤다면 이후 `npx shadcn add`로 덮어쓰지 않도록 주의한다.

---

## 4. 폰트 — Pretendard

**Google Fonts에 없다.** `next/font/google`로 부르면 실패한다. `.woff2`를 `src/app/fonts/`에 넣고 `next/font/local`로 로드한다.

```tsx
import localFont from "next/font/local";

const pretendard = localFont({
  src: "./fonts/PretendardVariable.woff2",
  variable: "--font-pretendard",
  display: "swap",
  weight: "45 920",
});
```

```css
@theme {
  --font-sans: var(--font-pretendard), system-ui, sans-serif;
}
```

가변 폰트 하나면 충분하다. 굵기별 파일을 따로 넣지 않는다.

---

## 5. 우선순위 뱃지

색은 셋으로 고정한다. 색만으로 정보를 전달하지 않도록 **텍스트를 함께** 둔다(접근성).

```tsx
const priorityStyle = {
  HIGH:   "bg-priority-high/10 text-priority-high",
  MEDIUM: "bg-priority-medium/10 text-priority-medium",
  LOW:    "bg-priority-low/10 text-priority-low",
} as const;

const priorityLabel = { HIGH: "높음", MEDIUM: "보통", LOW: "낮음" } as const;
```

**완료 항목**은 제목에 취소선 + 흐린 색상을 준다 (`PRD.md` 5.5).

```tsx
<span className={cn("text-base font-semibold", todo.completed && "line-through text-muted")}>
  {todo.title}
</span>
```

---

## 6. 반응형

**360px ~ 1920px에서 레이아웃이 깨지지 않아야 한다** (`UX-05`).

```tsx
// 모바일 우선. 기본값이 모바일이고 sm: 이상에서 확장한다
<main className="mx-auto max-w-3xl px-4 py-6 sm:px-6">
```

| 지점 | 확인할 것 |
|---|---|
| 360px | 목록 항목의 제목·뱃지·마감일·삭제 버튼이 겹치지 않는가. 페이지네이션이 `3 / 12` 형태로 축약되는가 |
| 1920px | `max-w-3xl`로 중앙 정렬되어 과도하게 늘어나지 않는가 |

긴 제목은 넘치지 않게 처리한다.

```tsx
<span className="truncate">{todo.title}</span>
```

---

## 7. 애니메이션 — Motion

```ts
// import는 항상 motion/react에서 한다. framer-motion이 아니다
import { motion, AnimatePresence, useReducedMotion } from "motion/react";
```

| 대상 | 동작 |
|---|---|
| 목록 등장 | `opacity 0→1` + `y 8→0`, stagger 30ms |
| 삭제 | `opacity→0` + `height→0`, `AnimatePresence` |
| 토글 | scale 스프링 |

- **200ms 이내, 과한 모션 금지.**
- `useReducedMotion`으로 `prefers-reduced-motion`을 존중한다.

```tsx
const shouldReduceMotion = useReducedMotion();
<motion.li
  initial={shouldReduceMotion ? false : { opacity: 0, y: 8 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.2 }}
/>
```

> `tw-animate-css`가 스캐폴딩에 포함되어 있다. shadcn 컴포넌트의 기본 트랜지션용이므로 그대로 두되, 목록·토글 애니메이션은 Motion으로 통일한다.

---

## ❌ 금지사항

```
❌ tailwind.config.js              — v4는 CSS-first
❌ @custom-variant dark (class 전략) — 토글이 없어 동작하지 않는다
❌ next-themes                      — 스택에 없다
❌ npx shadcn add form              — react-hook-form이 딸려 온다
❌ npx shadcn add toast             — deprecated. sonner를 쓴다
❌ bg-[#4F46E5] 같은 임의값 하드코딩  — @theme 토큰을 쓴다
❌ 그림자로 카드 구분                 — 1px border를 쓴다
❌ 액센트 컬러 추가                   — 단일 컬러 하나만
```

---

## ✅ 스타일링 체크리스트

- [ ] `tailwind.config.js`가 없고 `globals.css`의 `@theme`에 토큰이 있다
- [ ] 라이트/다크 토큰이 양쪽 다 정의되어 있다
- [ ] `@custom-variant dark`가 없고 `@media (prefers-color-scheme: dark)`를 쓴다
- [ ] **OS 다크 설정을 바꾸면 클래스 조작 없이 테마가 전환된다**
- [ ] `components.json`의 스타일이 `new-york`이다
- [ ] 색상이 전부 토큰이다 (임의값 하드코딩 없음)
- [ ] 360px에서 레이아웃이 깨지지 않는다
- [ ] 완료 항목에 취소선 + 흐린 색상이 적용된다
- [ ] 애니메이션이 200ms 이내이고 `prefers-reduced-motion`을 존중한다
- [ ] 애니메이션 import가 전부 `motion/react`에서 이루어진다
