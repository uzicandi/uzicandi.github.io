---
title: '3-3. 사용자 정의 훅과 고차 컴포넌트 중 무엇을 써야 할까?'
description: ''
date: 2024-09-04
update: 2024-09-04
tags:
  - React
series: '모던 리액트 Deep Dive'
---

### 사용자 정의 훅

- 서로 다른 컴포넌트 내부에서 같은 로직을 공유하고자 할 때 주로 사용되는 것
- use로 시작하는 함수 이름을 써야 한다. (아니면 에러 발생)
- ex. useFetch 라는 훅을 만들어서 컴포넌트에 사용

### 고차컴포넌트 (HOC)

- 컴포넌트 자체의 로직을 재사용
- 자바스크립트의 일급 객체, 함수의 특징을 이용함
- ex. React.memo - 렌더링에 앞서 props를 비교해 이전과 같다면 렌더링을 생략하고 이전에 기억해둔 컴포넌트를 반환한다.

### 고차함수 만들어보기

- 함수를 인수로 받거나 결과로 반환하는 함수
- ex. map, forEach, reduce, 리액트의 함수컴포넌트 (setState)

```cs
const setState = (function () {
  // 현재 Index를 클로저로 가둬놔서 이후에도 계속해서 동일한 index에 접근할 수 있도록 한다.
  let currentIndex = index
  return function (value) {
    global.states[currentIndex] = value
    // 컴포넌트를 렌더링한다. 실제로 컴포넌트를 렌더링하는 코드는 생략했다
  }
})()
```

```cs
function add(a) {
  return function (b) {
    return a + b
  }
}

const result = add(1); // 여기서 result는 앞서 반환한 함수를 가리킨다
const result2 = result(2); // 비로소 a와 b를 더한 3이 반환된다
```

### 고차함수를 활용한 리액트 고차컴포넌트 만들어보기

- with로 시작하는 이름을 써야한다. (권장)
- 부수효과를 최소화 해야 한다.
- 사용자 인증 정보에 따라
  - 인증된 사용자에게는 개인화된 컴포넌트를
  - 그렇지 않은 사용자에게는 별도의 공통 컴포넌트를 보여주는 시나리오

```cs
interface LoginProps {
  loginRequired?: boolean;
}

function withLoginComponent<T>(Component: ComponentType<T>) {
  return function (props: T & LoginProps) {
    const { loginRequired, ...restProps } = props;

    if(loginRequired) {
      return <>로그인이 필요합니다.</>
    }

    return <Component {...(restProps as T)} />
  }
}

// 원래 구현하고자 하는 컴포넌트를 만들고, withLoginComponent로 감싸기만 하면 끝이다.
// 로그인 여부, 로그인이 안되면 다른 컴포넌트를 렌더링하는 책임은 모두
// 고차 컴포넌트인 withLoginComponent에 맡길 수 있어 매우 편리하다.
const Component = withLoginComponent((props: { value: string }) => {
  return <h3>{props.value}</h3>
});

export default function App() {
  // 로그인 관련 정보를 가져온다.
  const isLogin = true;
  return <Component value="text" loginRequired={isLogin} />
}
```

### 고차 컴포넌트를 써야하는 경우

- 사용자 정의 훅: 은 단순히 중복 로직을 분리하거나, 특정한 훅의 작동을 취하고 싶은 경우 사용
- 고차컴포넌트 : 함수컴포넌트의 반환값, 즉 렌더링의 결과물에도 영향을 미치는 공통로직이라면 고차 컴포넌트 사용
