---
title: "Next.js fetch와 Data Cache"
date: 2025-10-22
---

Next.js는 표준 Web API인 `fetch`에 프레임워크 수준의 캐시와 재검증 기능을 더해, 캐시 정책을 선언적으로 설정할 수 있도록 했습니다.<br>
이 포스트에서는 먼저 캐시가 무엇인지, 어떤 문제를 해결하는지 살펴보고, 최근 버전에서 기본 캐시 전략이 어떻게 변경되었는지도 함께 알아봤습니다.
<br><br>

## 캐시

캐싱은 자주 쓰는 데이터를 가까운 곳에 잠깐 저장해 다음 접근을 빠르게 만드는 기술입니다. 응답 속도 향상이 핵심 목표이며, 웹 관점에서는 원본 서버 리퀘스트 수 감소로 부하와 비용 절감의 효과를 얻습니다.

### 브라우저 캐시 vs. 서버 캐시

브라우저 캐시와 서버 캐시는 서로 다른 레이어입니다. 목적에 맞게 함께 설계해 병행할 때 가장 효과적입니다.

- 브라우저 캐시
  - 유저가 방문한 리소스를 헤더가 허용하는 범위에서 로컬에 저장
  - 페이지 재방문 시 네트워크 왕복 없이 로딩 시간을 단축

- 서버 캐시
  - 원본 서버의 데이터를 서버 측에 임시 저장
  - 유저의 요청이 들어오면 캐시 서버에서 먼저 데이터를 찾아보고 데이터가 있으면 이를 제공
  - 데이터가 없거나 만료된 경우 원본 서버에서 데이터를 가져와 캐시에 저장하고 유저에게 제공
  - 이 포스트에서 다루는 Next.js의 Data Cache는 서버 앱 레벨 캐시입니다

## Next.js의 캐싱 메커니즘

각 캐싱 메커니즘과 목적에 대한 간략한 개요는 다음과 같습니다.

| Mechanism           | What                       | Where    | Purpose                                           | Duration                        |
| ------------------- | -------------------------- | -------- | ------------------------------------------------- | ------------------------------- |
| Request Memoization | Return values of functions | Server   | Re-use data in a React Component tree             | Per-request lifecycle           |
| **Data Cache**      | **데이터**                 | **서버** | **유저 요청 및 배포 전반에 걸쳐 데이터를 저장함** | Persistent (can be revalidated) |
| Full Route Cache    | HTML and RSC payload       | Server   | Reduce rendering cost and improve performance     | Persistent (can be revalidated) |
| Router Cache        | RSC payload                | Client   | Reduce server requests on navigation              | User session or time-based      |

**Data Cache**

- `fetch`로 요청한 데이터 그 자체를 캐싱
- A의 리퀘스트 뒤 B의 리퀘스트에서, A의 리퀘스트 덕에 캐싱된 데이터를 가져올 수 있습니다

<img src='/images/nextjs-fetch/caching-overview.avif' width="800">

## `fetch`

모든 데이터 요청은 웹 표준 API인 `fetch`를 기반으로 이뤄집니다. Next.js는 서버에서 실행되는 `fetch`에 Data Cache 기능을 연결해 옵션으로 캐시 전략을 선언할 수 있게 했습니다.

### History

- Next.js 12 이하 (Pages Router): `fetch` 자체에 Data Cache 없음
- App Router 초창기: 서버 `fetch`가 Data Cache와 기본 연동되는 상황이 많았음
- Next.js 14-15 (최근 버전): 기본적으로 `fetch` 응답이 자동 캐시되지 않음
  - 정적 렌더링 시 결과가 캐시에 저장될 수 있음
  - 동적 렌더링 시 매 요청 `fetch`

### 기본 동작: `auto no cache`

- 개발 환경에서는 매 리퀘스트마다 네트워크로 다시 가져옵니다
- 프로덕션 환경에서는 정적으로 프리렌더가 가능한 페이지는 빌드 때 한 번 가져오고, 그 결과를 캐싱합니다
- 동적 API가 섞여있으면 (쿠키나 헤더 의존 등) 프로덕션에서도 매 리퀘스트마다 다시 가져옵니다

### fetch 옵션별 작동 정리

`options.cache`: 리퀘스트가 Data Cache와 상호작용하는 방식을 구성합니다

```ts
fetch(`https://...`, { cache: "force-cache" | "no-store" });
```

- `cache: 'no-store'`: 항상 원본을 호출하며 Data Cache를 사용하지 않습니다
- `cache: 'force-cache'`: Data Cache를 조회하고 적용하며, 없거나 만료면 원본에서 가져오고 캐시를 업데이트합니다

`options.next.revalidate`: 리소스의 캐시 수명을 설정할 수 있습니다

```ts
fetch(`https://...`, { next: { revalidate: false | 0 | number } });
```

- `false`: 사실상 무기한인 캐시입니다 (공간 정책으로 관리됩니다)
- `0`: 캐시하지 않습니다
- `number`: 초 단위의 캐시 최대 수명을 지정합니다
- 캐시 갱신 과정:
  1. 최초 요청에는 미리 정적으로 캐시해둔 데이터를 보여줍니다
  2. 이 캐시된 초기 요청은 `revalidate`에 선언된 값만큼 유지됩니다
  3. 만약 해당 시간이 지나도 일단은 캐시된 데이터를 보여줍니다
  4. Next.js는 캐시된 데이터를 보여주는 한편, 시간이 경과했으므로 백그라운드에서 다시 데이터를 불러옵니다
  5. 4번의 작업이 성공적으로 끝나면 캐시된 데이터를 갱신하고, 그렇지 않다면 과거 데이터를 보여줍니다

```ts
export const revalidate = 3600

export default async function Page() {
  const res = await fetch('https://api.vercel.app/blog', {
    next: { revalidate: 60 },
  })
  const posts = await res.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

- 위 예시처럼 루트에 `revalidate`를 값과 선언해두면 하위에 있는 모든 라우팅에서는 페이지를 그 값 간격으로 갱신해 새로 렌더링합니다
- 위 예시처럼 라우트 기본 `revalidate`보다 더 낮은 값을 개별 `fetch()`에 지정하면 해당 라우트 전체의 재검증 주기가 그 값으로 단축됩니다
- 같은 라우터 안에서 같은 URL에 서로 다른 값을 쓰면 더 낮은 값이 적용됩니다
- `{ revalidate: 3600, cache: 'no-store' }` 같은 충돌 옵션이 있으면 둘 다 무시됩니다

`options.next.tags`: 리소스의 캐시 태그를 구성합니다

```ts
fetch(`https://...`, { next: { tags: ["collection"] } });
```

- 핸들러에서 `revalidateTag`를 써서 연관 데이터를 일괄적으로 무효화할 수 있습니다
- 태그의 최대 길이는 256자이고 최대 태그 항목 수는 128개입니다

```ts
export default async function Page() {
  const res = await fetch('https://api.vercel.app/blog', {
    next: { revalidate: 60, tags: ['blog']},
  })
  const posts = await res.json()
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  )
}
```

```ts
'use server'

import { revalidateTag } from 'next/cache'

export async function publishPost(formData: FormData) {
  // 새 포스트 publish
  // 해당 태그로 묶인 캐시 전부 무효화 -> 다음 요청에서 최신 데이터
  ...
  revalidateTag('blog')
}
```

<br><br>

_출처:<br>
[1] Next.js 공식 문서 (["Caching in Next.js"](https://nextjs.org/docs/app/guides/caching), ["fetch"](https://nextjs.org/docs/app/api-reference/functions/fetch))<br>
[2] 모던 리액트 Deep Dive, 김용찬 저, 위키북스, 2024, p.740-747<br>
[3] 코드잇 FESI 11기 강의자료_
