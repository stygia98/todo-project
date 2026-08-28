# 컴포넌트 패턴 가이드 — `todo-frontend`

**Next.js 15 + React 19 + TypeScript** 환경에서 이 프로젝트의 컴포넌트를 작성하는 규칙이다.

> ⚠️ **규칙의 단일 출처는 [`CLAUDE.md`](../../CLAUDE.md) 9장·10장이다.** 이 가이드가 그 문서와 어긋나면 `CLAUDE.md`를 따른다.

---

## 0. Server/Client 경계 — 이 프로젝트는 클라이언트 우선이다

일반적인 App Router 가이드는 "Server Components 기본, Client는 최소한"을 권한다. **이 프로젝트는 반대다.**

- 인증 토큰이 **localStorage**에 있어 서버가 인증 헤더를 만들 수 없다.
- 서버 상태를 **React Query**로 가져오므로 데이터 페칭이 전부 브라우저에서 일어난다.

```
src/app/layout.tsx        ← 서버 컴포넌트. 이것 하나뿐이다
  └─ providers.tsx        ← "use client" — 여기서부터 전부 클라이언트
       └─ 모든 페이지·컴포넌트
```

### 그렇다고 전부 `"use client"`를 붙이라는 뜻은 아니다

지시어는 **경계를 여는 선언**이다. 부모가 이미 클라이언트 경계 안이면 자식은 자동으로 클라이언트에서 실행된다.

```tsx
// ✅ 지시어 없이 둔다. 클라이언트 부모 안에서 그대로 동작한다
export function PriorityBadge({ priority }: { priority: Priority }) {
  return <span className={priorityStyle[priority]}>{priorityLabel[priority]}</span>;
}

// ❌ 불필요 — 상태도 이벤트 핸들러도 없다
"use client";
export function PriorityBadge({ priority }: { priority: Priority }) { ... }
```

**붙이는 곳은 정해져 있다.**

| 대상 | `"use client"` |
|---|---|
| 모든 `page.tsx` | 필요 |
| `(main)/layout.tsx`, `providers.tsx` | 필요 |
| 훅을 쓰거나 이벤트를 다루는 컴포넌트 | 필요 |
| 표시 전용 컴포넌트 (뱃지, 스켈레톤 등) | 불필요 |
| 루트 `layout.tsx` | **붙이지 않는다** |

---

## 1. 컴포넌트 배치 기준

| 폴더 | 넣는 것 | 예 |
|---|---|---|
| `components/ui/` | shadcn이 생성한 것만 | `button.tsx`, `input.tsx` |
| `components/common/` | 도메인을 모르는 재사용 컴포넌트 | `Pagination`, `EmptyState`, `ErrorState`, `Skeleton`, `AppHeader` |
| `components/todo/` | Todo 도메인을 아는 컴포넌트 | `TodoList`, `TodoItem`, `TodoForm`, `TodoEditor` |

**판단이 애매하면 `todo/`에 둔다.** 재사용처가 실제로 생겼을 때 `common/`으로 올린다. 쓰이지도 않는 범용 컴포넌트를 미리 만들지 않는다.

---

## 2. Props 설계

### 필요한 것만 받는다

```tsx
// ✅ 이 컴포넌트가 실제로 쓰는 것만
type TodoItemProps = {
  todo: TodoResponse;
  onToggle: (id: number, completed: boolean) => void;
  onDelete: (id: number) => void;
};

// ❌ 컴포넌트가 스스로 페칭 — 목록 전체가 N번 요청하게 된다
type TodoItemProps = { id: number };
```

### 선택 prop이 조용한 결함을 만드는 경우가 있다

`UX-04`는 **재시도 버튼이 있는** 에러 상태를 요구한다. `onRetry`를 선택으로 두면 호출부에서 빠뜨려도 타입 검사를 통과하고, 재시도 버튼 없는 에러 카드가 배포된다.

```tsx
// ✅ 필수로 둬서 누락이 컴파일에서 잡히게 한다
type ErrorStateProps = {
  message: string;
  onRetry: () => void;
};

// ❌ 선택 — 누락이 조용히 통과한다
type ErrorStateProps = {
  message: string;
  onRetry?: () => void;
};
```

### 상태를 어디에 둘지

| 상태 | 위치 |
|---|---|
| 서버 데이터 (Todo 목록·단건, 내 정보) | **React Query** |
| 검색어·완료 필터·페이지 번호 | **URL 쿼리** (`useState` 아님) |
| 입력 중인 폼 값, 열림/닫힘 | `useState` |

목록 상태를 URL에 두는 이유는 새로고침·뒤로가기·링크 공유가 자연스럽게 동작하고, React Query 키가 URL과 1:1로 대응해 캐시 대상이 명확해지기 때문이다.

---

## 3. 이 프로젝트의 핵심 컴포넌트

### `TodoForm` — 두 화면이 재사용한다

`/todos/new`와 `/todos/[id]`가 **같은 컴포넌트**를 쓴다. 초기값 유무와 삭제 버튼 노출로만 구분한다.

```tsx
type TodoFormProps = {
  initialValue?: TodoResponse;   // 있으면 수정, 없으면 생성
  onSubmit: (values: TodoFormValues) => Promise<void>;
  onDelete?: () => void;         // 상세 화면에서만 넘긴다
};
```

- **완료 체크박스를 두지 않는다.** 완료는 목록에서만 바꾼다 (`TODO-10`).
- 상세 화면은 **보기/편집 모드를 나누지 않는다.** 진입 즉시 편집 가능하다.
- 자세한 내용은 [`forms.md`](./forms.md).

### `Pagination` — 공용

```tsx
type PaginationProps = {
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
};
```

- 현재 페이지 주변 5개 + 처음/이전/다음/마지막
- **페이지 수가 1 이하면 아무것도 렌더링하지 않는다**
- 모바일은 `3 / 12` 형태로 축약

### 상태 컴포넌트 넷을 구분한다

| 컴포넌트 | 언제 |
|---|---|
| `Skeleton` | 로딩 중. 목록은 항목형 3개 (`UX-01`) |
| `EmptyState` | 할 일이 하나도 없음 — "아직 할 일이 없어요" + CTA (`UX-02`) |
| `EmptyState` (검색 변형) | 검색·필터 결과 없음 — **문구가 달라야 한다** (`UX-03`) |
| `ErrorState` | 요청 실패 — 재시도 버튼 필수 (`UX-04`) |

> ⚠️ **빈 상태와 검색 결과 없음을 같은 문구로 처리하면 `UX-03` 위반이다.** "할 일이 없다"와 "검색 결과가 없다"는 사용자가 취할 행동이 다르다.

---

## 4. 리스트 렌더링

```tsx
// ✅ key는 항상 서버 id
{todos.map((todo) => (
  <TodoItem key={todo.id} todo={todo} onToggle={handleToggle} onDelete={handleDelete} />
))}

// ❌ index — 낙관적 삭제로 항목이 빠지면 잘못된 항목이 재사용된다
{todos.map((todo, i) => <TodoItem key={i} ... />)}
```

낙관적 업데이트로 항목이 즉시 사라지는 화면이라 **`key`가 특히 중요하다.** index를 쓰면 삭제 애니메이션이 엉뚱한 항목에 걸린다.

`AnimatePresence`로 감쌀 때도 같은 `key`를 유지한다.

```tsx
<AnimatePresence>
  {todos.map((todo) => (
    <motion.li key={todo.id} exit={{ opacity: 0, height: 0 }} />
  ))}
</AnimatePresence>
```

---

## 5. 접근성 (`UX-06`)

- **모든 입력에 `label`을 `htmlFor`/`id`로 연결한다.** placeholder는 label이 아니다.
- 아이콘만 있는 버튼에는 접근 가능한 이름을 준다.

```tsx
// ✅ 삭제 버튼이 아이콘뿐이라면
<button aria-label={`${todo.title} 삭제`}>
  <Trash2 aria-hidden />
</button>
```

- **키보드만으로 가입·로그인·할 일 생성·완료 토글·삭제를 완주할 수 있어야 한다.**
- 체크박스는 `<input type="checkbox">` 또는 shadcn `Checkbox`를 쓴다. `div`에 `onClick`을 달면 키보드로 조작할 수 없다.

---

## 6. 타입

```ts
// src/types/todo.ts — 백엔드 DTO와 이름을 맞춘다
export type Priority = "HIGH" | "MEDIUM" | "LOW";

export type TodoResponse = {
  id: number;
  title: string;
  content: string | null;
  completed: boolean;
  priority: Priority;
  dueDate: string | null;      // yyyy-MM-dd
  createdAt: string;           // ISO-8601 UTC
  updatedAt: string;
};

export type ApiResponse<T> = {
  success: boolean;
  data: T | null;
  error: { code: string; message: string } | null;
};

export type PageResponse<T> = {
  content: T[];
  page: number;
  size: number;
  totalElements: number;
  totalPages: number;
  first: boolean;
  last: boolean;
};
```

- **`any`를 쓰지 않는다.** 불가피하면 `unknown` + 타입 가드.
- 목록 응답은 `ApiResponse<PageResponse<TodoResponse>>`다. `apiClient`가 `data`를 벗겨내므로 컴포넌트는 `PageResponse<TodoResponse>`를 받는다.

---

## 7. 성능 — 지금 하지 않아도 되는 것

MVP 규모(페이지당 10건)에서는 아래가 **불필요하다.** 없는 문제를 최적화하지 않는다.

| 기법 | 판단 |
|---|---|
| `React.memo` / `useMemo` / `useCallback` | 10건 렌더는 측정 가능한 비용이 아니다. **측정 후에 넣는다** |
| 가상화 (react-window 등) | 페이지네이션이 있어 필요 없다. 스택에도 없다 |
| `dynamic()` 지연 로딩 | 화면이 6개뿐이다 |

**예외 하나** — Tiptap 에디터는 무겁다. `/todos` 목록에서는 로드되지 않도록 `TodoEditor`를 `TodoForm` 안에서만 import한다. 목록 컴포넌트가 에디터를 참조하지 않으면 자연히 분리된다.

---

## ❌ 안티패턴

```tsx
// ❌ 서버 컴포넌트에서 인증 데이터 페칭 — 토큰이 없어 항상 401
export default async function TodosPage() {
  const todos = await fetch(`${API}/api/v1/todos`);
}

// ❌ 컴포넌트 안에서 직접 fetch — apiClient를 우회하면 토큰 주입·401 처리·언래핑이 빠진다
useEffect(() => { fetch("/api/v1/todos").then(...) }, []);

// ❌ dangerouslySetInnerHTML — 이 앱에는 사용 지점이 없다
<div dangerouslySetInnerHTML={{ __html: todo.content }} />

// ❌ 목록 필터를 useState로 — 새로고침하면 사라지고 뒤로가기가 깨진다
const [keyword, setKeyword] = useState("");
```

**API 호출은 전부 `src/lib/apiClient.ts`를 거친다.** 토큰 주입, `ApiResponse` 언래핑, 401 자동 로그아웃이 그 안에 있다.

---

## ✅ 컴포넌트 체크리스트

**경계**
- [ ] 루트 `layout.tsx`에 `"use client"`가 없다
- [ ] 모든 `page.tsx`에 `"use client"`가 있다
- [ ] 표시 전용 컴포넌트에 불필요한 지시어가 없다

**설계**
- [ ] `TodoForm`을 `/todos/new`와 `/todos/[id]`가 재사용한다
- [ ] `TodoForm`에 완료 체크박스가 없다
- [ ] `ErrorState.onRetry`가 **필수 prop**이다
- [ ] 빈 상태와 검색 결과 없음의 문구가 다르다
- [ ] `Pagination`이 1페이지 이하에서 렌더링되지 않는다

**타입·데이터**
- [ ] `any`가 없다
- [ ] 타입 이름이 백엔드 DTO와 일치한다
- [ ] 모든 API 호출이 `apiClient`를 거친다
- [ ] 리스트 `key`가 서버 id다

**접근성**
- [ ] 모든 입력에 label이 연결되어 있다
- [ ] 아이콘 버튼에 `aria-label`이 있다
- [ ] 키보드만으로 주요 동작을 완주할 수 있다
