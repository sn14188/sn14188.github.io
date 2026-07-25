---
title: "TanStack Router"
date: 2026-07-25
---

TanStack Router를 프로젝트에 써보긴 했는데 왜 좋은지 명확히 설명하기는 어려워서, 개념과 실제로 장점이 무엇인지 이번 기회에 정리해봤습니다.
<br><br>

## React Router

React Router는 URL 경로와 컴포넌트를 매핑해 화면을 전환해주는 라우팅 라이브러리입니다. React 생태계에서 가장 오래되고 가장 널리 쓰이는 라우터로, SPA를 만들 때 사실상 기본 선택지라 할 수 있습니다.<br>
일반적인 `<a href="/posts">` 링크를 클릭하면 브라우저는 서버에 새 HTML 문서를 요청하고, 페이지가 리로드되는 과정은 다음과 같습니다.

- 현재 실행 중이던 JavaScript 컨텍스트가 통째로 사라지고 React 앱이 처음부터 다시 마운트됩니다
- 번들도 다시 다운로드해 파싱합니다
- 입력 중이던 폼 값이나 열려있던 모달 같은 상태도 전부 초기화됩니다

React Router의 `<Link>` 컴포넌트는 겉보기엔 `<a>`처럼 생겼지만 클릭 시 `event.preventDefault()`로 이 기본 동작을 가로챕니다. 대신 `history.pushState()`로 주소창의 URL만 바꾸고, 그 변화를 감지해서 현재 실행 중인 React 앱 안에서 컴포넌트만 교체해 다시 렌더링합니다.<br>
브라우저 입장에서는 페이지가 이동한 것처럼 보이지만 실제로는 JS 실행 환경과 앱 상태를 유지한 채 필요한 부분만 다시 그리는 것입니다.

### 직접 구현의 한계

로직을 직접 구현하면 다음과 같습니다.

```tsx
// src/useRoute.ts
import { useEffect, useState } from 'react'

function useRoute() {
  const [path, setPath] = useState(window.location.pathname)

  useEffect(() => {
    const onPopState = () => setPath(window.location.pathname)
    window.addEventListener('popstate', onPopState)
    return () => window.removeEventListener('popstate', onPopState)
  }, [])

  const navigate = (to: string) => {
    window.history.pushState({}, '', to)
    setPath(to)
  }

  return { path, navigate }
}
```

이 훅을 실제로 쓰는 쪽은 다음처럼 됩니다.

```tsx
// src/App.tsx
function App() {
  const { path } = useRoute()

  if (path === '/posts') return <PostListPage />
  if (path.startsWith('/posts/')) return <PostPage id={path.split('/')[2]} />

  return <HomePage />
}
```

문제는 라우트가 몇 개만 늘어나도 이런 단순 분기로는 감당이 안 되는 지점들이 나온다는 것입니다.

- 매칭 우선순위:
  - `path.startsWith('/posts/')` 하나로 `/posts/new`, `/posts/123`, `/posts/123/edit`이 전부 걸립니다
  - 더 구체적인 경로를 먼저 검사하도록 순서를 잘 배치해야 하고, 라우트가 늘어날수록 이 순서를 다루기가 까다로워집니다
- 파라미터 추출:
  - `path.split('/')[2]`는 세그먼트가 하나일 때만 통합니다
  - `/posts/:postId/comments/:commentId`처럼 중첩되면 인덱스 계산과 예외 처리까지 직접 핸들해야 합니다
- 중첩 레이아웃:
  - `/posts`와 `/posts/:id`가 같은 사이드바를 공유해야 하는 경우입니다
  - 경로 매칭과 부모 레이아웃 유지를 if문만으로 동시에 표현하기 어렵습니다
- 404 처리:
  - 위 코드의 마지막 `return <HomePage />`는 존재하지 않는 경로를 조용히 홈으로 보냅니다
  - "아무 라우트도 매칭되지 않음"이라는 경우를 별도로 처리해야 제대로 된 404가 됩니다

이 로직을 재사용 가능한 함수로 뽑아내도, 결국 경로 패턴 매칭과 우선순위, 중첩 구조 처리를 하는 미니 라우터를 직접 만드는 셈이 됩니다.<br>
React Router의 `useNavigate()`, `<Link>`, `useParams()`는 이미 이 문제를 다 풀어놓은 편의 계층이라, 평소에는 라이브러리가 이 과정을 대신하고 있다는 걸 의식하지 못한 채로 쓰게 됩니다. 아래 라우트 정의 방식이 바로 이 네 가지를 선언적으로 해결하는 부분입니다.

### 주요 기능

React Router의 주요 기능은 다음과 같습니다.

- 라우트 매칭: `<Route path="..." element={...} />`로 경로와 컴포넌트를 연결합니다
- 중첩 라우팅은 `<Outlet />`으로 부모 라우트 안에 자식 컴포넌트를 렌더링합니다
- 동적 세그먼트: `path="/posts/:postId"`처럼 `:`로 파라미터 자리를 표시하고, `useParams()`로 값을 읽습니다
- 네비게이션: `<Link>`, `<NavLink>`로 선언적으로 이동하거나, `useNavigate()`로 코드에서 직접 페이지를 전환합니다
- 데이터 로딩(v6.4+):
  - `useEffect`로 데이터를 가져오면 컴포넌트가 마운트된 뒤에야 요청이 시작됩니다
  - `loader`를 라우트 정의에 붙이면 라우트가 매칭되는 즉시 페칭이 시작되고, 컴포넌트는 `useLoaderData()`로 이미 준비된 값을 바로 사용합니다

```tsx
// src/main.tsx
import { createBrowserRouter, RouterProvider, useLoaderData } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <RootLayout />,
    children: [
      { index: true, element: <HomePage /> },
      { path: 'posts/new', element: <NewPostPage /> },
      {
        path: 'posts/:postId',
        element: <PostPage />,
        loader: ({ params }) => fetchPost(params.postId),
      },
      { path: '*', element: <NotFoundPage /> },
    ],
  },
])

function PostPage() {
  const post = useLoaderData()
  return <h1>{post.title}</h1>
}

function App() {
  return <RouterProvider router={router} />
}
```

`createBrowserRouter`로 라우트 트리를 배열 형태로 정의하고, `RouterProvider`로 앱을 감싸면 됩니다. 이 배열 하나가 앞서 손으로 짜야 했던 네 가지 문제를 전부 해결합니다.

- 매칭 우선순위: `posts/new`는 배열 순서와 무관하게 `posts/:postId`보다 항상 먼저 매칭됩니다
- 파라미터 추출: `params.postId`에 이미 파싱된 값이 들어옵니다
- 중첩 레이아웃: `children`이 `RootLayout` 안에서 중첩 구조를 표현합니다
- 404 처리: `path: '*'`가 명시적인 캐치올 라우트로 처리합니다

여기서 라우트 경로는 그냥 문자열이라, 이 배열 안에 있는 경로와 실제로 `<Link to="...">`에 적는 경로가 일치하는지는 개발자가 직접 확인해야 합니다.

### 사용 영역

요즘 Next.js가 많이 쓰이지만 React Router의 입지가 좁은 건 아닙니다.

- SPA:
  - SSR·SEO가 필요하지 않은 사내 어드민, 대시보드, 브라우저 확장 프로그램에서는 여전히 Vite와 React Router (또는 TanStack Router) 조합이 기본값입니다
- 프레임워크:
  - 2024년 React Router v7이 Remix의 프레임워크 기능을 (SSR, Vite 기반 번들링 등) 흡수했습니다
  - Next.js와 경쟁하는 프레임워크 레벨 포지션에도 들어와있습니다

## TanStack Router

TanStack Router는 React Query로 잘 알려진 TanStack 팀이 만든 라우팅 라이브러리입니다. React Router가 이미 사실상 표준인데도 별도로 만들어진 이유는 타입 안정성을 핵심 설계 원칙으로 삼았기 때문입니다.<br>
앞서 본 것처럼 React Router의 라우트 경로는 그냥 문자열이라 오타나 존재하지 않는 경로도 런타임이 되어서야 드러나는데, TanStack Router는 이걸 컴파일 타임에 잡아내는 것을 목표로 합니다.

### 파일 기반 라우팅

`src/routes/` 폴더 아래에 파일을 만들면 그 경로 구조가 곧 라우트가 됩니다.

```text
src/routes/
  __root.tsx        # 모든 라우트의 최상위 레이아웃
  index.tsx          # "/"
  posts.index.tsx    # "/posts"
  posts.$postId.tsx  # "/posts/:postId"
```

`@tanstack/router-plugin` Vite 플러그인이 이 폴더를 감시하다가 파일이 추가되거나 바뀔 때마다 `routeTree.gen.ts`라는 파일을 자동으로 생성합니다. 이 파일 안에 전체 라우트 구조가 타입으로 정의되기 때문에, 사람이 라우트 목록을 따로 등록하거나 import할 필요가 없습니다.

```ts
// vite.config.ts
import { tanstackRouter } from '@tanstack/router-plugin/vite'
import react from '@vitejs/plugin-react'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [
    tanstackRouter({ target: 'react', autoCodeSplitting: true }),
    react(),
  ],
})
```

### 타입 세이프 라우팅

각 라우트는 `createFileRoute`로 정의하고, 그 경로 문자열 자체가 타입의 일부가 됩니다.

```tsx
// src/routes/posts.$postId.tsx
import { createFileRoute } from '@tanstack/react-router'

export const Route = createFileRoute('/posts/$postId')({
  loader: ({ params }) => fetchPost(params.postId),
  component: PostPage,
})

function PostPage() {
  const post = Route.useLoaderData()
  return <h1>{post.title}</h1>
}
```

이렇게 정의해두면 앱 어디에서 `Link`를 쓰든 이 라우트 정보를 그대로 참조할 수 있습니다.

```tsx
<Link to="/posts/$postId" params={{ postId: '123' }}>
  게시글 보기
</Link>
```

`to` 값에 오타를 내거나, `postId` 파라미터를 빠뜨리면 그 자리에서 바로 타입 에러가 납니다. 이게 가능한 이유는 라우터 인스턴스를 만들 때 전역으로 타입을 등록해두기 때문입니다.

```tsx
// src/main.tsx
import { RouterProvider, createRouter } from '@tanstack/react-router'
import { routeTree } from './routeTree.gen'

const router = createRouter({ routeTree })

declare module '@tanstack/react-router' {
  interface Register {
    router: typeof router
  }
}
```

`declare module`로 `Register` 인터페이스를 확장해두면, 이 라우터가 앱 전체의 유일한 라우터 타입으로 등록됩니다. 그 덕분에 다른 파일에서 라우트 정의를 일일이 import하지 않아도 `Link`나 `useNavigate`가 전체 라우트 트리를 알고 있는 것처럼 동작합니다.

### Search Params도 타입으로

React Router에서 쿼리스트링은 그냥 문자열입니다. `?page=2`를 읽으려면 `URLSearchParams`를 직접 파싱하고, 숫자로 변환하고, 값이 없거나 이상한 경우까지 직접 처리해야 합니다.<br>
TanStack Router는 `validateSearch`에 스키마를 넘겨서 이 과정을 라우터가 대신 하도록 합니다.

```tsx
// src/routes/posts.index.tsx
import { createFileRoute } from '@tanstack/react-router'
import { z } from 'zod'

const searchSchema = z.object({
  page: z.number().catch(1),
  sort: z.enum(['asc', 'desc']).catch('asc'),
})

export const Route = createFileRoute('/posts/')({
  validateSearch: searchSchema,
  component: PostListPage,
})

function PostListPage() {
  const { page, sort } = Route.useSearch()
}
```

URL이 `/posts?page=abc`처럼 스키마에 안 맞게 들어와도 `.catch()`로 지정한 기본값으로 자동 대체됩니다.

### 장점

- 컴파일 타임 검증: 존재하지 않는 경로로 이동하거나 필수 파라미터를 빠뜨리는 실수를 빌드 단계에서 잡아줍니다
- Search params를 상태로 활용: 파싱과 기본값 처리가 라우터 레벨에서 끝나 페이지네이션이나 필터 상태를 `useState` 대신 URL에 위임할 수 있습니다
- 자동 코드 스플리팅:
  - `autoCodeSplitting: true` 옵션만 켜면 라우트별로 청크가 자동 분리됩니다
  - `React.lazy`를 라우트마다 손으로 붙이지 않아도 됩니다
- 개발자 도구: 매칭된 라우트, 로더 상태, 캐시 상태를 시각적으로 확인할 수 있습니다

## Takeaways

1. 라우팅 라이브러리는 브라우저의 API 위에 얹은 편의 계층이며, `<Link>`나 `useNavigate()`가 이걸 대신 처리해줍니다
2. React Router는 선언적 라우트 테이블로 매칭 우선순위, 파라미터 추출, 중첩 레이아웃, 404 처리를 자동으로 해결하지만 라우트 경로가 문자열이라 오타는 런타임에야 드러납니다
3. TanStack Router는 파일 기반 라우팅과 `validateSearch`를 통해 라우트 경로, 파라미터, search params까지 전부 타입으로 다룹니다
   <br><br>

_출처:<br>
[1] TanStack (["TanStack Router Documentation"](https://tanstack.com/router/latest))<br>
[2] TanStack (["TanStack Router GitHub Repository"](https://github.com/TanStack/router))<br>_
