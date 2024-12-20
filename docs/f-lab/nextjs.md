---
title: 'SSR, SSG, CSR, ISR'
description: ''
date: 2024-10-03
update: 2024-10-03
tags:
  - NextJS
series: 'NextJS'
---

NextJS는 SSR 프레임워크이지만, SSR만을 위해서 사용하는 것은 아니다.
기본적으로 리액트 프레임워크이기 때문에 라우터, 빌드, 설정을 쉽게 할 수 있기 때문에 사용한다.

- Next.js에서는 처음만 SSR이고 이후에는 index.json 파일로 가져온다
  `_next/data/~index.json`
- SSR로 가져오고 이후에는 CSR로 동작한다.

### Hydration

- React에서 SSR을 하게 되면 html와 react component를 연결하는 작업
- 서버에서 그렸을 때, 클라이언트에서 그렸을 때 = hydration mismatch
  - window가 있는지 없는지로 분리하는 경우에 주로 발생
  - 서버에서 안그리도록 만드는 경우로 해결하기도
  - dynamic으로 해서 NoSSR로 함

### SSR, SSG의 차이

- index.html을 만들어주는데 시점이 다름
- SSR: 요청마다 (유저 정보에 따라 다르게 보여줘야 하는 경우에 유용)
  - `next start`
- SSG: 빌드시점마다 만듦 (동적으로 하진 못함. 정적사이트에 유리 - 블로그)
  - `next export`
  - 따라서 SSG로 빌드해서 FP+LCP를 개선할 수 있다.
  - 처음 페이지는 정적콘텐츠로 보여주고 이후에 data fetching 되면 CSR로 사용하도록

### Client-side Rendering (CSR)

- 최소한의 HTML 페이지와 JavaScript를 다운로드한다.
- 자바스크립트는 DOM을 업데이트 하고 페이지를 렌더한다.
- 애플리케이션이 처음 로드되면, full page를 보여주기 전에 약간의 딜레이가 있다. 이는 자바스크립트가 완전히 다운로드되고, 파싱되고, 실행되기 전이기 때문이다.

- 페이지가 처음 로드되면, 다른 페이지로 navigate하는데 빠르다. full page를 새로고침하지 않고 리렌더하여 필요한 데이터만 fetch하기 때문이다.

```cs
import React, { useState, useEffect } from 'react'

export function Page() {
  const [data, setData] = useState(null)

  useEffect(() => {
    const fetchData = async () => {
      const response = await fetch('https://api.example.com/data')
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      const result = await response.json()
      setData(result)
    }

    fetchData().catch((e) => {
      // handle the error as needed
      console.error('An error occurred while fetching the data: ', e)
    })
  }, [])

  return <p>{data ? `Your data: ${data}` : 'Loading...'}</p>
  // Loading... 이후에 데이터가 페치되면 리렌더해서 보여준다.
}
```

- data-fetching 퍼포먼스, 캐싱, Optimistic updates를 위해 SWR을 추천한다.

```cs
import useSWR from 'swr'

export function Page() {
  const { data, error, isLoading } = useSWR(
    'https://api.example.com/data',
    fetcher
  )

  if (error) return <p>Failed to load.</p>
  if (isLoading) return <p>Loading...</p>

  return <p>Your Data: {data}</p>
}
```

- SEO에 좋지 않다. 검색 엔진 크롤러는 자바스크립트를 실행하지 않아 초기의 비어있거나 로드 중인 상태만 볼 수 있다.
- 또한 인터넷 연결이나 기기가 느린 사용자는 모든 Javascript가 로드되어 실행되어야 페이지를 볼 수 있으므로 성능 이슈를 겪을 수 있다.

### Server-side Rendering (SSR)

- SSR 혹은 Dynamic Rendering
- 사용자가 페이지를 요청하면 서버에서 페이지를 동적으로 생성한다.
- 생성된 HTML을 클라이언트에 전송한다.
- 브라우저는 받은 HTML을 렌더링하고, 필요한 경우 Javascript를 실행하여 페이지를 인터랙티브하게 만든다
- 서버사이드 렌더링을 사용하기 위해서는 `getServerSideProps` async function을 export 해야한다.
- 예를 들어, 페이지에서 자주 없데이트 되는 데이터를 미리 렌더링 해야 하는 경우 아래와 같이 데이터를 가져와서 전달하는 코드를 작성한다.

```cs
export default function Page({ data }) {
  // Render data...
}

// This gets called on every request
export async function getServerSideProps() {
  // Fetch data from external API
  const res = await fetch(`https://.../data`)
  const data = await res.json()

  // Pass data to the page via props
  return { props: { data } }
}
```

- `getServerSideProps`와 `getStaticProps`은 비슷하지만 빌드시점이 아니라 모든 요청에서 실행된다는 점이 다르다

### Static Site Generation (SSG)

- HTML을 빌드타임에 생성한다.
- 즉, production 환경에서 `next build`를 실행시킬 때 HTML 페이지를 생성한다.
- data가 있든 없든 정적으로 페이지를 생성한다.

#### Static Generation with data

어떤 페이지는 렌더링 전에 외부 데이터 페칭을 요청한다.

- 페이지 content가 외부 데이터에 의존할 때: `getStaticProps`
- 페이지 paths가 외부 데이터에 의존할 때: `getStaticPaths`

**1. content를 외부 데이터에 의존하는 경우**

- `getStaticProps`를 빌드시 호출하여, 사전 렌더링 시 페치된 데이터를 페이지에 전달한다.

```cs
export default function Blog({ post }) {
  return (
    <ul>
      {
        posts.map((post) => (
          <li>{post.title}</li>
        ))
      }
    </ul>
  )
}
```

```cs
export default function Blog({ posts }) {
  // Render posts...
}

// This function gets called at build time
export async function getStaticProps() {
  // Call an external API endpoint to get posts
  const res = await fetch('https://.../posts')
  const posts = await res.json()

  // By returning { props: { posts } }, the Blog component
  // will receive `posts` as a prop at build time
  return {
    props: {
      posts,
    },
  }
}
```

**2. 외부 데이터에 따라 달라지는 페이지 경로**

- `pages/posts/[id].js`로 동적 라우팅을 할 수 있음

### SSG는 언제 사용하는가?

- 빌드 시점에 모든 페이지가 생성된다.
- 생성된 정적 파일들(HTML, CSS, JavaScript)은 CDN 등에 배포된다.
- 사용자 요청 시 미리 생성된 파일을 제공한다.

따라서

- 빠른 로딩 속도
- 보안성 향상
- 호스팅 비용 절감
- SEO 최적화

에 도움이 된다.

### Incremental Static Regeneration (ISR)

- 정적으로 생성된 페이지를 주기적으로 또는 필요에 따라 재생성하는 방식
- 정적 사이트 생성(SSG)과 서버 사이드 렌더링(SSR)의 장점을 결합한 하이브리드 접근 방식

#### 작동방식

- 초기에는 SSG처럼 정적페이지를 생성
- 설정된 시간 간격이나 특정 조건에 따라 백그라운드에서 페이지를 재생성
- 새 요청이 들어오면 캐시된 버전을 제공하고, 동시에 새 버전을 생성

#### 장점

- SSG의 빠른 성능과 SSR의 데이터 최신성을 모두 제공
- 서버 부하 감소 (SSR에 비해)
- 대규모 사이트에서도 정적 생성의 이점을 누릴 수 있음
- 콘텐츠 업데이트가 자주 일어나는 사이트에 적합 (ex. 뉴스기사 업데이트)

#### 단점

- 구현 복잡도가 증가할 수 있음

```cs
export async function getStaticProps() {
  // ... 데이터 페칭 로직
  return {
    props: { /* ... */ },
    revalidate: 60 // 60초마다 페이지 재생성
  }
}
```

```cs
export const getStaticPaths: GetStaticPaths = async () => {
  const posts = await fetch('https://api.vercel.app/blog').then((res) =>
    res.json()
  )
  const paths = posts.map((post: Post) => ({
    params: { id: String(post.id) },
  }))

  // 빌드 시 이 경로만 사전 렌더링.
  // 경로가 존재하지 않는 경우 요청 시 { fallback: 'blocking' }은 페이지를 서버에서 렌더링
  return { paths, fallback: false }
}
```
