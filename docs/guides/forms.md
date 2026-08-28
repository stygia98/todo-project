# 폼 처리 가이드 — `todo-frontend`

이 프로젝트의 폼은 **라이브러리 없이** `useState` + 수동 검증으로 만든다.

> ⚠️ **스택과 규칙의 단일 출처는 [`CLAUDE.md`](../../CLAUDE.md) 3장·4장·9장이다.** 이 가이드가 그 문서와 어긋나면 `CLAUDE.md`를 따른다.
> 검증 규칙의 정본은 `CLAUDE.md` 4장 「입력값 제약」 표다. 이 문서는 그것을 코드로 옮기는 방법만 다룬다.

---

## 0. 왜 `react-hook-form`을 쓰지 않는가

| 이유 | 내용 |
|---|---|
| 규모 | 폼이 **셋**뿐이다 — `/login`(2필드), `/signup`(3필드), `TodoForm`(4필드) |
| 검증 규칙 | `CLAUDE.md` 4장 표로 고정되어 있고 조건부 필드나 동적 배열이 없다 |
| 스택 원칙 | 스펙에 없는 라이브러리를 늘리지 않는다 (`CLAUDE.md` 15장) |

### ⚠️ `npx shadcn add form`을 실행하지 않는다

shadcn/ui의 **`form` 컴포넌트만** `react-hook-form` 위에 만들어져 있다. 이것을 추가하면 `react-hook-form`과 `@hookform/resolvers`가 **의존성으로 함께 설치되어**, 결정이 조용히 뒤집힌다.

```bash
# ❌ 금지 — react-hook-form이 딸려 들어온다
npx shadcn@latest add form

# ✅ 이것들은 안전하다. 그대로 쓴다
npx shadcn@latest add input label button select checkbox calendar popover
```

폼 마크업은 `label` + `input`을 직접 조합한다. 어차피 `UX-06`이 **모든 입력에 label 연결**을 요구하므로 명시적으로 쓰는 편이 낫다.

---

## 1. 기본 형태

```tsx
"use client";

type LoginErrors = Partial<Record<"email" | "password" | "form", string>>;

export function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [errors, setErrors] = useState<LoginErrors>({});
  const { login, isPending } = useAuth();

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();

    // 1) 클라이언트 검증
    const nextErrors = validateLogin({ email, password });
    if (Object.keys(nextErrors).length > 0) {
      setErrors(nextErrors);
      return;
    }

    // 2) 서버 요청 — 실패 시 서버 error.code를 문구로 변환한다
    try {
      await login({ email, password });
    } catch (err) {
      setErrors({ form: toMessage(err) });   // lib/errorMessages.ts
    }
  }

  return (
    <form onSubmit={handleSubmit} noValidate>
      {/* 폼 상단 인라인 에러 — PRD 5.1 에러 문구 매핑 */}
      {errors.form && <p role="alert">{errors.form}</p>}

      <label htmlFor="email">이메일</label>
      <input
        id="email"
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        aria-invalid={!!errors.email}
        aria-describedby={errors.email ? "email-error" : undefined}
      />
      {errors.email && <p id="email-error">{errors.email}</p>}

      {/* password 동일 */}

      <button type="submit" disabled={isPending}>
        {isPending ? "로그인 중…" : "로그인"}
      </button>
    </form>
  );
}
```

**포인트 셋**

- `noValidate`를 붙여 **브라우저 기본 검증 말풍선을 끈다.** 문구를 우리가 통제해야 `PRD.md` 5.1의 에러 매핑을 지킬 수 있다.
- `htmlFor` ↔ `id`로 label을 연결한다 (`UX-06`). placeholder는 label을 대신하지 못한다.
- 에러는 `aria-invalid` + `aria-describedby`로 스크린리더에 전달한다.

---

## 2. 검증 규칙 — `lib/validation.ts`에 모은다

정본은 `CLAUDE.md` 4장 표다. 화면마다 조건을 흩뿌리면 서버와 어긋난다.

```ts
// src/lib/validation.ts

/** 비밀번호 상한은 문자 수가 아니라 UTF-8 바이트다 (BCrypt 72바이트 한계) */
export function utf8Bytes(value: string): number {
  return new TextEncoder().encode(value).length;
}

export function validateSignup(input: SignupInput): SignupErrors {
  const errors: SignupErrors = {};

  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input.email)) {
    errors.email = "올바른 이메일 형식이 아닙니다.";
  } else if (input.email.length > 255) {
    errors.email = "이메일은 255자 이하여야 합니다.";
  }

  if (input.password.length < 6) {
    errors.password = "비밀번호는 6자 이상이어야 합니다.";
  } else if (utf8Bytes(input.password) > 72) {
    errors.password = "비밀번호가 너무 깁니다. (한글은 1자가 3바이트로 계산됩니다)";
  }

  if (input.nickname.length < 1 || input.nickname.length > 50) {
    errors.nickname = "닉네임은 1~50자여야 합니다.";
  }

  return errors;
}
```

### ⚠️ 비밀번호 상한은 "6자 이상"이 전부가 아니다

**`maxLength={64}` 같은 문자 수 제한만 걸면 한글 비밀번호에서 서버가 500을 낸다.**

BCrypt의 한계는 **72바이트**다. UTF-8에서 한글 1자는 3바이트이므로 **한글 25자 = 75바이트**로 이미 한계를 넘는다. 문자 수로 세는 검증은 이 입력을 통과시키고, 서버의 `BCryptPasswordEncoder`가 `IllegalArgumentException`을 던진다.

- `new TextEncoder().encode(v).length`로 **바이트를 센다.**
- 안내 문구에 **"한글은 1자가 3바이트"**를 밝힌다. 밝히지 않으면 사용자가 이유를 알 수 없다.
- 백엔드도 같은 규칙으로 400 `INVALID_INPUT`을 낸다. 프론트 검증은 왕복을 줄이는 것이지 **대체가 아니다.**

### 실시간 검증의 타이밍

`PRD.md` 5.3은 `/signup`에 실시간 검증을 요구한다. 다만 **입력 첫 글자부터 빨간 글씨를 띄우지 않는다.**

```tsx
// 필드를 벗어날 때(blur) 처음 검증하고, 그 뒤부터는 입력마다 갱신한다
const [touched, setTouched] = useState<Record<string, boolean>>({});

<input
  onBlur={() => setTouched((t) => ({ ...t, password: true }))}
  onChange={(e) => {
    setPassword(e.target.value);
    if (touched.password) setErrors(validateSignup({ ...values, password: e.target.value }));
  }}
/>
```

---

## 3. 서버 에러를 문구로 바꾼다 — `lib/errorMessages.ts`

`PRD.md` 5.1의 매핑 표를 **한 곳에** 둔다. 화면마다 문구를 직접 쓰면 표가 사문화된다.

| 서버 `error.code` | 문구 | 표시 위치 |
|---|---|---|
| `INVALID_INPUT` | 서버가 준 필드별 메시지 | 해당 입력 아래 |
| `UNAUTHORIZED` (로그인 시) | "이메일 또는 비밀번호가 올바르지 않습니다." | 폼 상단 |
| `UNAUTHORIZED` (그 외) | 문구 없이 `/login`으로 이동 | — |
| `EMAIL_DUPLICATED` | "이미 사용 중인 이메일입니다." | 이메일 입력 아래 |
| `TODO_NOT_FOUND` | "할 일을 찾을 수 없습니다." | 전체 화면 |
| `INTERNAL_ERROR` | "일시적인 오류가 발생했습니다. 다시 시도해 주세요." | 토스트/에러 카드 |
| 네트워크 실패 | "연결에 실패했습니다." + 재시도 | 에러 카드 |

> ⚠️ **로그인 실패는 계정 존재 여부를 구분해 노출하지 않는다.** 비밀번호 오류와 미가입 이메일에 **같은 문구**를 쓴다. "가입되지 않은 이메일입니다"는 계정 열거(enumeration) 통로가 된다.

`INVALID_INPUT`은 서버가 필드별 메시지를 주므로, 그것을 그대로 필드 아래에 붙인다.

```ts
// 서버가 준 필드 오류를 폼 상태에 병합한다
if (err.code === "INVALID_INPUT" && err.fields) {
  setErrors((prev) => ({ ...prev, ...err.fields }));
}
```

---

## 4. `TodoForm` — 하나를 두 화면이 재사용한다

`/todos/new`와 `/todos/[id]`가 **같은 컴포넌트**를 쓴다. 초기값 유무와 삭제 버튼 노출로만 구분한다.

```tsx
type TodoFormProps = {
  initialValue?: TodoResponse;      // 있으면 수정, 없으면 생성
  onSubmit: (values: TodoFormValues) => Promise<void>;
  onDelete?: () => void;            // 상세 화면에서만 넘긴다
};
```

**규칙 둘**

- **완료 체크박스를 두지 않는다.** 완료 상태는 목록의 체크박스로만 바꾼다. 폼에 넣으면 저장할 때마다 완료 상태를 덮어써, 목록에서 체크한 결과가 되돌아간다(`TODO-10`).
- 저장은 **명시적**이다. 자동 저장하지 않는다.

### 저장 실패 시 폼 내용을 유지한다

```tsx
async function handleSubmit(e: React.FormEvent) {
  e.preventDefault();
  try {
    await onSubmit(values);
    setDirty(false);            // ⚠️ 성공했을 때만 가드를 푼다
    router.push("/todos");
  } catch (err) {
    setFormError(toMessage(err));  // 입력값은 그대로 둔다
  }
}
```

> `TODO-13`·`UX-04`가 여기에 걸린다. **실패했는데 폼을 비우거나 페이지를 떠나면 사용자가 작성한 내용이 사라진다.** 목록의 토글·삭제와 달리 저장은 낙관적 업데이트를 쓰지 않으므로, 이 처리는 폼이 직접 해야 한다.

---

## 5. 이탈 확인 — 3계층으로 구현한다 (`TODO-16`)

**한 줄짜리 작업이 아니다.** App Router에는 Pages Router의 `router.events`가 없고, 공식 내비게이션 차단 API도 없다. `beforeunload` 하나로 끝난다고 가정하면 **앱 내부 이동이 전혀 막히지 않는다.**

| 이탈 경로 | 방어 수단 |
|---|---|
| 새로고침 · 탭 닫기 · 주소창 직접 이동 | `beforeunload` 이벤트 |
| 페이지 내 "취소"·"목록으로" 버튼 | **버튼 자체 핸들러**에서 확인 후 `router.push` |
| 브라우저 뒤로가기 | `popstate` 리스너 + 취소 시 `history.pushState`로 되돌리기 |

```tsx
// 1) 새로고침·탭 닫기
useEffect(() => {
  if (!dirty) return;
  const handler = (e: BeforeUnloadEvent) => e.preventDefault();
  window.addEventListener("beforeunload", handler);
  return () => window.removeEventListener("beforeunload", handler);
}, [dirty]);

// 2) 버튼 핸들러
function handleCancel() {
  if (dirty && !window.confirm("저장하지 않은 변경 사항이 있습니다. 나가시겠습니까?")) return;
  router.push("/todos");
}
```

- **`next-navigation-guard` 같은 서드파티를 도입하지 않는다.** 스택에 없다.
- **저장 직후에는 반드시 가드를 해제한다.** 저장하고 나가는데 확인창이 뜨면 그것도 결함이다.
- `ROADMAP.md`에서 이 항목에 **별도 공수 4~8시간**을 잡아두었다.

### ⚠️ `dirty` 판정에서 Tiptap이 가장 까다롭다

라이브러리의 `formState.isDirty`가 없으므로 초기값과 현재값을 직접 비교한다. 제목·우선순위·마감일은 단순 비교로 끝나지만, **본문은 그렇지 않다.**

Tiptap은 넣어준 HTML을 **자기 스키마로 정규화**한다. 서버가 준 `<p>안녕</p>`이 에디터를 거치면 속성이나 공백이 달라질 수 있다. 서버 원본과 `editor.getHTML()`을 그대로 비교하면 **사용자가 아무것도 고치지 않았는데 dirty로 판정**되어, 저장하고 나가는데도 확인창이 뜬다.

```ts
// ✅ 정규화를 거친 값끼리 비교한다
editor.commands.setContent(sanitizeHtml(todo.content));
const baseline = editor.getHTML();      // ← setContent 직후의 값을 기준으로 삼는다

const isContentDirty = editor.getHTML() !== baseline;
```

```ts
// ❌ 서버 원본과 직접 비교 — 항상 dirty로 나올 수 있다
const isContentDirty = editor.getHTML() !== todo.content;
```

---

## 6. Tiptap 연결

에디터는 폼 상태의 일부이지만 `value`/`onChange`를 가진 일반 입력이 아니다. **에디터 인스턴스를 그대로 두고, 제출 시점에 `getHTML()`로 읽는다.**

```ts
const editor = useEditor({
  extensions: [
    StarterKit.configure({
      heading: { levels: [2, 3] },   // h1, h4~h6 차단
      strike: false,                  // <s> 차단
      horizontalRule: false,          // <hr> 차단
      underline: false,               // <u> 차단 — Ctrl+U까지 함께 꺼진다
      link: {                         // v3 내장 Link. 별도 패키지를 설치하지 않는다
        openOnClick: false,
        HTMLAttributes: { rel: "noopener noreferrer", target: "_blank" },
        protocols: ["http", "https", "mailto"],
      },
    }),
  ],
  immediatelyRender: false,   // SSR 하이드레이션 불일치 방지
});
```

- **서버에서 받은 본문은 `setContent()` 직전에 `lib/sanitize.ts`를 거친다.** 이 앱에는 `dangerouslySetInnerHTML`이 없으므로 여기가 유일한 렌더 방어 지점이다.
- **툴바 · Tiptap 확장 · Jsoup 화이트리스트 · DOMPurify 설정 네 곳이 항상 같은 태그 집합**을 가리켜야 한다. 한 곳을 바꾸면 나머지 셋도 바꾼다.
- 본문 상한은 **50,000자**다. 제출 전에 확인한다.

---

## ❌ 이 프로젝트에서 하지 않는 것

| 항목 | 이유 |
|---|---|
| `react-hook-form` · `zod` · `@hookform/resolvers` | 스택에 없다 (0번) |
| **Server Actions** (`"use server"`) | API가 별도 Spring 서버다. 폼 제출은 `apiClient`로 REST를 호출한다 |
| `useFormStatus` / `useActionState` | 위와 같다. Server Actions 전제 훅이다 |
| **CSRF 토큰** | `Authorization: Bearer` 헤더 인증이라 CSRF 경로가 없다. 백엔드도 `csrf.disable()` 상태다 |
| **파일 업로드 폼** | 파일 첨부가 **비목표**다 (`PRD.md` 1장) |
| **자동 저장** | `TODO-10`이 명시적 저장을 요구한다 |
| **다단계 폼** | 해당 화면이 없다 |

---

## ✅ 폼 체크리스트

**접근성 (`UX-06`)**
- [ ] 모든 입력에 `label`이 `htmlFor`/`id`로 연결되어 있다 (placeholder로 대신하지 않았다)
- [ ] Tab·Enter만으로 가입·로그인·할 일 생성을 완주할 수 있다
- [ ] 에러 메시지가 `aria-describedby`로 입력과 연결되어 있다

**검증**
- [ ] 비밀번호를 **바이트**로 검증한다 (한글 25자로 시험했다)
- [ ] 제목 200자, 본문 50,000자 상한을 지킨다
- [ ] `noValidate`로 브라우저 기본 말풍선을 껐다

**에러 처리**
- [ ] 서버 `error.code`를 `lib/errorMessages.ts` 한 곳에서 문구로 바꾼다
- [ ] 로그인 실패 문구가 미가입/비밀번호 오류에서 **동일**하다
- [ ] 저장 실패 시 폼 내용이 유지된다

**이탈 확인 (`TODO-16`)**
- [ ] 새로고침 · 취소 버튼 · 뒤로가기 **세 경로 모두** 막힌다
- [ ] 저장 직후에는 확인창이 뜨지 않는다
- [ ] 본문을 고치지 않았을 때 dirty로 오판되지 않는다 (Tiptap 정규화)
