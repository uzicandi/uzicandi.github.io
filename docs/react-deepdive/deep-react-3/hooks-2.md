---
title: '3-2. useMemo, useCallback, useRef, useLayoutEffect'
description: ''
date: 2024-09-04
update: 2024-09-04
tags:
  - React
series: '모던 리액트 Deep Dive'
---

### 1. useMemo

의존성 배열의 값이 변경됐다면 첫번째 인수의 함수를 실행한 후 값을 반환하고, 그 값을 다시 기억해둔다.

### 2. useCallback

- 인수로 넘겨받은 콜백 자체를 기억한다. 즉, 함수를 재생성 하지 않는다.
- 함수의 재생성을 막아 불필요한 리소스 또는 리렌더링을 방지하고 싶을 때 사용한다.

### 3. useRef

useState와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태값을 저장한다

- useRef는 반환값인 객체 내부에 있는 current로 값에 접근 또는 변경할 수 있다.
- useRef는 그 값이 변하더라도 렌더링을 발생시키지 않는다.

**왜 필요할까?**

- 렌더링에 영향을 미치지 않는 고정된 값을 관리하기 위해서
- 그냥 함수 외부에서 값을 선언하는 것과 뭐가 다른가?

```cs
1️⃣ 컴포넌트가 실행되어 렌더링되지 않았음에도 value값이 기본적으로 존재하게 된다.
- 이는 메모리에 불필요한 값을 갖게 하는 악영향

2️⃣ Component가 여러번 생성된다면 각 컴포넌트에서 가리키는 값이 모두 value로 동일하다.
- 컴포넌트 인스턴스 하나 당 하나의 값을 필요로 하는게 일반적이다.

=> 이를 해결하는 useRef
```

- **가장 일반적인 사용 예는 DOM에 바로 접근하고 싶을 때이다.**

```cs
function RefComponent() {
  const inputRef = useRef()

  // 이때는 미처 렌더링이 실행되기 전이라 undefined로 반환
  console.log(inputRef.current)

  useEffect(() => {
    console.log(inputRef.current) // <input type="text"></input>
  }, [inputRef]);

  return <input ref={inputRef} type="text" />
}
```

- **렌더링을 발생시키지 않고 원하는 상태값을 저장할 수 있다.**
- useState의 이전 값을 저장하는 `usePrevious()`같은 훅을 구현할 때 유용

```cs
function usePrevious(value) {
  const ref = useRef()
  useEffect(() => {
    ref.current = value
  }, [value]); // value가 변경되면 그 값을 ref에 넣어둔다.

  return ref.current;
}
```

### 4. useLayoutEffect

useEffect와 동일하나, 모든 DOM의 변경 후에 동기적으로 발생한다.

- useEffect보다 먼저 실행된다.
- DOM은 계산됐지만 이것이 화면에 반영되기 전에 하고 싶은 작업이 있을 때
- DOM요소를 기반으로 한 애니메이션, 스크롤위치를 제어하는 등 화면에 반영되기 전에 하고 싶은 작업에 사용한다면 자연스러운 사용자 경험을 제공할 수 있다.

```cs
1. 리액트가 DOM을 업데이트
2. useLayoutEffect를 실행
3. 브라우저에 변경 사항을 반영
4. useEffect를 실행
```
