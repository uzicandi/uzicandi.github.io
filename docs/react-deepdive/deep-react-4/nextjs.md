---
title: '4-3. Next.js 톺아보기 [wip]'
description: ''
date: 2024-09-07
update: 2024-09-07
tags:
  - React
series: '모던 리액트 Deep Dive'
---

서버사이드 렌더링을 위해 만들어진 리액트 기반 프레임워크

#### Server-Routing과 Client-Routing의 차이

Next.js는 SSR을 수행하지만 동시에 SPA처럼 Client-Routing 또한 수행한다.

- SSR을 비롯한 사전 렌더링을 지원하기 때문에 최초 페이지 렌더링이 서버에서 수행된다.

```cs
export default function Hello() {
  console.log(typeof window === 'undefined' ? '서버' : '클라이언트')
  return <>hello</>
}
```

#### next.config.js

- next 프로젝트의 환경 설정을 담당하며, next.js를 자유자재로 다루려면 반드시 알아야 하는 파일

```cs
/** @type {import('next').NextConfig} */ <- 타입스크립트의 도움을 받기위해 추가된 코드
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true // 바벨보다 빠른 번들링, 컴파일을 빠르게 수행하도록 하는 도구 (swc)
};
export default nextConfig;
```

#### `pages/_app.tsx`

- `_app.tsx` 그리고 내부에 있는 `default export`로 내보낸 함수는 애플리케이션의 전체 페이지의 시작점이다.
- 페이지의 시작점이라는 특징 때문에 웹 애플리케이션에서 공통으로 설정해야 하는 것들을 여기에서 실행할 수 있다.
  - 에러 바운더리를 사용해 전역 에러 처리
  - reset.css같은 전역 CSS 선언
  - 모든 페이지에 공통으로 사용 또는 제공해야하는 데이터 제공
- `_app.tsx`의 render() 내부에 console.log()를 추가해서 메세지를 기록햊보자
  - 페이지 새로고침 -> 해당 고르가 브라우저 콘솔창이 아닌 터미널에 실행 (SSR)
  - 페이지 전환하면 -> 브라우저에서 로깅 (CSR)

#### `pages/_documents.tsx`

- `_app.tsx`: 애플리케이션 페이지 전체를 초기화 하는 곳.
  - Next 설정과 관련된 코드를 모아둔다.
  - 서버, 클라이언트 모두에서 렌더링 될 수 있다.
- `_documents.tsx` : 애플리케이션의 HTML을 초기화 하는 곳
  - 서버에서만 렌더링

```cs
1.<html><body>에 DOM 속성을 추가하고 싶은 경우 사용
2. _app.tsx는 렌더링이나 라우팅에 따라 서버나 클라이언트에서 실행될 수 있지만
_documents.tsx는 무조건 서버에서 실행된다.
따라서 이 파일에서 onClick과 같은 이벤트 핸들러를 추가하는 것은 불가능하다.
이벤트를 추가하는 것은 클라이언트에서 실행되는 hydrate의 몫임
3. next/document에서 제공하는 head, next/head에서 제공하는 head가 있음
- next/document는 _document.tsx에서만 사용할 수 있음
- next/head는 페이지에서 사용할 수 있음, SEO에 필요한 정보나 title등을 담을 수 있음
4. getServerSideProps, getStaticProps같은 서버에서 사용가능한 데이터 불러오기 함수는 여기에서 사용할 수 없음
```

```cs
import { Html, Head, Main, NextScript } from 'next/document'

export default function Document() {
  return (
    <Html lang="ko">
      <Head />
      <body className="body">
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

```cs
예제 프로젝트의 구성
- /pages/index.tsx : 웹사이트의 루트, localhost:3000과 같은 루트 주소
- /pages/hello.tsx : /pages가 생략되고, 파일명이 주소가 된다. localhost:3000/hello
- /pages/hello/world.tsx: localhost:3000/hello/world
- /pages/hello/[greeting].tsx: []는 어떠한 문자도 올 수 있다는 뜻. greeting이라는 변수에 담긴 주소가 오게 된다.
- /pages/hi/[...props].tsx : /hi를 제외한 하위의 모든 주소가 여기로 온다. props라는 변수에 배열로 오게됨
```

```cs
- /pages/hi/[...props].tsx

import { useRouter } from 'next/router';

// 클라이언트에서 값을 가져오는 법
export default function HiAll({ props: severProps }: {props: string[]}) {
  const { query: {props} } = useRouter();
}

// 서버에서 값을 가져오는 법
export const getServerSideProps = (context: NextPageContext) => {
  const { query: { props }} = context
}

// /hi/my/name/is = ['hi','my','name','is']
```

#### a 태그와 next/link

- `a 태그`: 모든 리소스를 처음부터 다시 받는다.
  - 서버에서 렌더링 수행하고 클라이언트에서 Hydrate 하는 과정에서 한번더 실행됨
- `next/link`: 페이지 이동에 필요한 내용만 받음
  - SSR이 아닌 클라이언트에서 필요한 자바스크립트만 불러온 뒤 라우팅하는 CSR로 실행함
- `<a>` 대신 `<Link>`를 사용한다
- `window.location.push` 대신 `router.push`를 사용한다.

#### getServerSideProps의 여부

- getServerSideProps가 있는 채로 빌드하면 Server side runtime 체크가 되어 있다.
- 없는 경우 빌드 크기도 약간 줄고, 정적 페이지로 분류됨
  - 서버에서 실행하지 않아도 되는 페이지로 분류하고
  - typeof window의 처리를 object로 바꾼 다음, 빌드 시점에 트리쉐이킹 해버린다.

#### `/pages/api/hello.ts`

- 서버의 API를 정의하는 폴더
- 이 경우 `/api/hello`로 호출할 수 있음
- 서버에서만 실행되므로 window, document 코드는 불가
- 서버에서 내려주는 데이터를 조합해 BFF 형태로 활용하거나, 풀스택 애플리케이션을 구축하고 싶을 때, CORS 문제를 우회하기 위해 사용

### Data Fetching

SSR 지원을 위한 데이터 불러오기 전략

- `pages/` 폴더에 있는 라우팅이 있는 파일에서만 사용할 수 있다.
- 예약어로 지정되어 반드시 정해진 함수명으로 export를 사용해 함수를 파일 외부로 내보내야 한다.
- **서버에서 미리 필요한 페이지를 만들어서 제공**하거나 해당 페이지에 요청이 있을 때마다 서버에서 데이터를 조회해서 **미리 페이지를 만들어서 제공**할 수 있다.

**getStaticPaths와 getStaticProps**

- CMS(Contents Management System)나 블로그, 게시판과 같이 사용자와 관계없이 정적으로 결정된 페이지를 보여주고자 할 때 사용되는 함수
- getStaticProps와 getStaticPaths는 반드시 함께 있어야 사용할 수 있음

---

https://nextjs.org/docs/getting-started/installation
이거 보고 따라해보기

#### 1. Top-level folders

최상위 폴더는 애플리케이션의 코드와 정적 자산을 구성하는데 사용됩니다.

```cs
- app : 앱 라우터
- pages : 페이지 라우터
- public : 제공될 정적 자산
- src : 선택적인 애플리케이션 소스 폴더
```

#### 2. Top-level files

최상위 파일은 애플리케이션 구성, 종속성 관리, 미들웨어 실행, 모니터링 도구 통합, 환경 변수 정의에 사용됩니다.

```cs
- next.config.js : Next.js에 대한 구성 파일
- package.json : 프로젝트 종속성 및 스크립트
- instrumentation.ts : OpenTelemetry 및 계측 파일
- middleware.ts	: Next.js 요청 미들웨어
```

#### 3. app Routing Conventions

다음 파일 규칙은 app라우터 에서 경로를 정의하고 메타데이터를 처리하는 데 사용됩니다 .

```cs
[ Routing Files ]
- layout
- page
- loading
- not-found
- error
- global-error
- route : API endpoint
- template : Re-rendered layout
- default : Parallel route fallback page
```

```cs
[ Nested Routes ]
- folder: Route segment
- folder/folder: Nested route segment
```

```cs
[ Dynamic Routes ]
- [folder] : Dynamic route segment
- [...folder] :	Catch-all (포괄적인) route segment
- [[...folder]]	: Optional catch-all (선택가능한 포괄적인) route segment
```

```cs
[ Route Groups and Private Folders : 경로 그룹 및 개인 폴더]
- (folder) : Group routes without affecting routing, 라우팅에 영향을 주지 않는 그룹 경로
- _folder :	Opt folder and all child segments out of routing, 라우팅에서 폴더 및 모든 자식 세그먼트를 제외합니다.
```

```cs
[ Parallel and Intercepted Routes : 평행 및 차단 경로 ]
- @folder	: Named slot
- (.)folder :	Intercept same level
- (..)folder : Intercept one level above
```

```cs
[ Metadata File Conventions ]
- App Icons
- Open Graph and Twitter Images
- SEO
```

### App Router

---

The Next.js App Router introduces a new model for building applications using React's latest features such as Server Components, Streaming with Suspense, and Server Actions.

### Building Your Application

---

```cs
1. Routing : learn the fundamental of routing for front-end applications
2. Data Fetching : how to fetch, cache, revalidate and mutate data with next.js
3. Rendering: Learn the differences between Next.js rendering environments, strategies, and runtime
4. Caching: An overview of caching mechanisms in Next.js
5. Styling: Len the different ways you can style your Next.js application
6. Optimizing: Optimize your Next.js application for best performance and user experience.
7. Configuring: Learn how to configure your Next.js application
8. Testing: how to setup Next.js with four commonly used testing tools - Cypress, Playwright, Vitest, and Jest
9. Authentication: how to implement authentication in your Next.js application
10. Deploying: how to deploy your Next.js app to production, either managed or self-hosted
11. Upgrading: how to upgrade to the latest versions of Next.js
12. Examples: Examples of popular Next.js UI patters and use cases.
```
