# todo-project

개인용 Todo List 풀스택 웹 서비스의 **문서 저장소**다. 로그인한 사용자가 자신의 할 일을 리치 텍스트로 작성하고, 완료·삭제를 즉각적인 반응으로 관리하는 것이 핵심 가치다.

## 저장소 구조 (폴리레포)

이 프로젝트는 **3개의 독립된 Git 저장소**로 관리된다. 모노레포가 아니다 — 아래 두 저장소는 각자 별도의 GitHub 저장소이며, 로컬에서는 이 저장소 하위에 나란히 위치하되 `.gitignore`로 추적에서 제외된다.

| 저장소 | 담당 | 링크 |
|---|---|---|
| `todo-project` (이 저장소) | 문서: `CLAUDE.md`, `docs/PRD.md`, `docs/ROADMAP.md` | — |
| [`todo-backend`](https://github.com/stygia98/todo-backend) | Spring Boot REST API | `todo-backend/` |
| [`todo-frontend`](https://github.com/stygia98/todo-frontend) | Next.js 프론트엔드 | `todo-frontend/` |

## 문서 안내

| 문서 | 내용 |
|---|---|
| [`CLAUDE.md`](./CLAUDE.md) | **기술 규칙의 단일 기준.** 데이터 모델·API 명세·인증 설계·코딩 컨벤션 등 전체 스펙의 정본 |
| [`docs/PRD.md`](./docs/PRD.md) | 제품 요구사항 — 무엇을 만드는가 |
| [`docs/ROADMAP.md`](./docs/ROADMAP.md) | 개발 로드맵 — 어떤 순서로 만드는가, **완료 판정의 정본**(Phase별 DoD와 검증 기록) |
| `docs/guides/` | 참고 자료. 스펙이 아니며 충돌 시 `CLAUDE.md`가 우선한다 |

각 하위 저장소(`todo-backend`, `todo-frontend`)에도 자체 `CLAUDE.md`가 있으며, 해당 저장소에서만 필요한 규칙(빌드 명령, 실행 방법 등)을 담는다. 전체 스펙은 이 저장소의 `CLAUDE.md`를 정본으로 삼는다.

## 기술 스택 요약

- **백엔드**: Spring Boot 4.x · JDK 21 · Spring Data JPA · Spring Security + JWT · PostgreSQL
- **프론트엔드**: Next.js 15 (App Router) · React 19 · TypeScript · Tailwind CSS 4 · React Query
- **인프라**: AWS Amplify(프론트) · EC2(백엔드) · RDS(PostgreSQL) — Docker는 사용하지 않는다

자세한 내용은 [`CLAUDE.md`](./CLAUDE.md)를 참조한다.
