---
title: "4-3. Next.js 톺아보기 [wip]"
description:
date: 2024-09-07
update: 2024-09-07
tags:
  - React
series: "모던 리액트 Deep Dive"
---

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
