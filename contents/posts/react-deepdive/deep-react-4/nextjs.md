---
title: "4-3. Next.js 톺아보기"
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
