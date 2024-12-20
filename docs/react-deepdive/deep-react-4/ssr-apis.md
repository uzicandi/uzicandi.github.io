---
title: '4-2. 서버 사이드 렌더링을 위한 리액트 API'
description: ''
date: 2024-09-06
update: 2024-09-06
tags:
  - React
series: '모던 리액트 Deep Dive'
---

- 브라우저 window 환경이 아닌 Node.js와 같은 서버 환경에서만 실행할 수 있다.
- react-dom/server.js : 서버에서 렌더링하기 위한 다양한 메서드 제공
- 2022년 8월 기준으로 server.node.js에 있는 함수를 export하고 있음

### 1. renderToString

- 리액트 컴포넌트를 렌더링해 HTML 문자열로 반환하는 함수
- 초기 렌더링에서 뛰어난 성능

### 2. renderToStaticMarkup

- renderToString과 유사함
- 추가적인 DOM 속성을 만들지 않는다.
- useEffect 같은 브라우저 API를 절대로 실행할 수 없음
- hydrate를 수행하면 서버와 클라이언트의 내용이 맞지 않다는 에러가 발생한다.
- renderToStaticMarkup의 결과물은 hydrate를 수행하지 않는다는 가정하에 순수한 HTML만 반환한다.
- 블로그 글이나 상품의 약관정보 같이 아무 브라우저 액션이 없는 정적인 내용만 필요한 경우에 유용함

### 3. renderToNodeStream

- renderToString과 결과물이 완전히 동일
- Node.js 환경에 의존하고 있어 브라우저에서 실행할 수 없음 (위의 두 api는 가능)
- 스트림은 큰 데이터를 다룰 때 데이터를 chunk해 조금씩 가져오는 방식을 채택함 -> 서버의 부담을 던다
- 대부분의 SSR 프레임워크가 채택하는 방식

### 4. renderToStaticNodeStream

hydrate할 필요가 없는 순수 HTML 결과물이 필요할 때 사용하는 메서드

### 5. hydrate

- renderToString, renderToNodeStream으로 생성된 HTML콘텐츠에 자바스크립트 핸들러나 이벤트를 붙이는 역할
- renderToString의 결과물은 단순히 서버에서 렌더링한 HTML 결과물로 사용자에게 보여줄 수 있지만, 사용자가 페이지와 인터랙션하는 것은 불가능함
- hydrate는 정적으로 생성된 HTML에 이벤트와 핸들러를 붙여 완전한 웹페이지 결과물을 만듦
  - render는 CRA에서만 실행되는 비슷한 역할

```cs
ReactDOM.hydrate(<App/>, element)
```

- 이미 렌더링된 HTML이 있다는 가정하에 작업이 수행된다.
- 렌더링된 HTML을 기준으로 이벤트를 붙이는 작업만 실행
