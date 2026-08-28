# 개발 가이드

이 폴더의 문서는 **참고 자료**다. **스펙이 아니다.**

> ⚠️ **충돌하면 [`CLAUDE.md`](../../CLAUDE.md)가 이긴다.**
> 규칙의 단일 기준은 `CLAUDE.md`, 무엇을 만드는지는 [`PRD.md`](../PRD.md), 언제 만드는지는 [`ROADMAP.md`](../ROADMAP.md)다.
> 이 폴더의 문서는 그 규칙을 **코드로 옮기는 방법**만 다룬다.

---

## 목록

| 문서 | 내용 |
|---|---|
| [`nextjs-15.md`](./nextjs-15.md) | App Router 사용 규칙. 클라이언트 우선 렌더링, Suspense 경계, 라우트 보호, 쓰지 않는 Next 기능 |
| [`project-structure.md`](./project-structure.md) | 폴리레포 3저장소 구조, 폴더 배치 기준, 네이밍 컨벤션 |
| [`component-patterns.md`](./component-patterns.md) | Server/Client 경계, Props 설계, 상태 컴포넌트 넷, 접근성 |
| [`styling-guide.md`](./styling-guide.md) | Tailwind 4 CSS-first, 디자인 토큰, 다크 모드, shadcn/ui, Motion |
| [`forms.md`](./forms.md) | 라이브러리 없는 폼 처리, 검증, 에러 문구 매핑, 이탈 확인 3계층 |

---

## 이력 (2026-08-28)

이 폴더에는 원래 **다른 프로젝트에서 넘어온 문서 5개**가 있었다. 다음과 같은 이유로 이 프로젝트에 그대로 적용하면 위험했다.

| 문서 | 문제 |
|---|---|
| `nextjs-16.md` | Next.js **16** 기준. 이 프로젝트는 Amplify SSR 지원 범위 때문에 **15로 고정**되어 있다. `proxy.ts`, `cacheComponents` 등 15에 없는 내용 |
| `forms-react-hook-form.md` | `react-hook-form`·`zod`가 **"이미 설치되어 있다"고 서술**했으나 이 프로젝트에는 설치되어 있지 않다. 비밀번호 규칙을 "6자 이상이 전부"로 적어 **UTF-8 72바이트 한계를 누락**. 파일 업로드·자동 저장·CSRF 등 **비목표** 내용 다수 |
| `component-patterns.md` | "Server Components 기본값"을 전제. 이 프로젝트는 토큰이 localStorage에 있어 **클라이언트 우선**이다 |
| `styling-guide.md` | `next-themes` 기반 **테마 토글**을 전제. 토글 UI는 P2로 범위 밖이고 `class` 전략도 쓰지 않는다 |
| `project-structure.md` | **모노레포**를 전제. 이 프로젝트는 폴리레포다 |

다섯 문서 모두 존재하지 않는 **"PRD 1.3 기술 스택"** 절을 정본으로 참조하고, 정의되지 않은 npm 스크립트(`typecheck`, `check-all`, `format:check`)를 기준으로 쓰였다.

**전부 이 프로젝트 기준으로 다시 썼다.** `nextjs-16.md` → `nextjs-15.md`, `forms-react-hook-form.md` → `forms.md`로 이름도 바뀌었다.

---

## 새 가이드를 추가할 때

- **`CLAUDE.md`의 규칙을 복사하지 않는다.** 링크로 참조하고, 여기에는 "어떻게 적용하는가"만 쓴다. 복사하면 두 곳이 갈라진다.
- 스택에 없는 라이브러리를 예제에 쓰지 않는다. 예제 코드가 사실상의 스펙으로 읽힌다.
- 첫 줄에 **정본이 `CLAUDE.md`라는 것**을 밝힌다.
