---
title: 'Next/Link의 기능과 동작원리'
description: ''
date: 2024-10-10
update: 2024-10-10
tags:
  - NextJS
series: 'NextJS'
---

### Next/Link 의 특징

Link 컴포넌트는 **CSR**을 지원하기 위해 설계되었다.

- 빌드 후 `<a>`로 자동 변환된다.
- 따라서 `<a>`의 장점을 갖는다. (SEO 최적화, prefetch 가능, 우클릭 기능 사용 가능 등)
  - **prefetch**: 뷰포트에 Link 컴포넌트가 들어오면 해당 페이지의 JS 파일을 자동으로 미리 로드
- 페이지 렌더링 시점에 이동할 주소가 정해져 있는 경우 사용
- 페이지 이동시 SPA 방식으로 전체 html 중 필요한 부분만 리렌더링 된다.

**따라서 Next/Link는 SPA같은 사용자 경험을 제공하면서도 SSR의 이점을 유지할 수 있다.**

### 추가 특징

- `scroll`: 스크롤 위치 유지
- `prefetch`: 해당 경로가 미리 fetch하여 클라이언트 탐색 성능을 개선한다.

### 동작 과정

```cs
1. 사용자가 Link 컴포넌트를 클릭한다.
2. 브라우저는 JavaScript를 실행하고 클라이언트 측 라우팅 메커니즘이 활성화 된다
3. 클라이언트 측 라우팅은 새로운 페이지의 필요한 컴포넌트 및 데이터를 로드한다.
4. 화면이 부드럽게 전환되면서 새로운 페이지가 렌더링 된다. 이 과정에서는 전체 페이지를 새로 렌더링 하는 것이 아니라 필요한 부분만 업데이트 된다.
5. 사용자는 새로운 페이지로 이동한 것 처럼 보이지만, 실제로는 전체 페이지의 새로고침 없이 부드럽게 화면이 전환된다.
```
