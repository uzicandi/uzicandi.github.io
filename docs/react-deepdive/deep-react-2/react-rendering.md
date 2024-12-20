---
title: '2.2 리액트 렌더링 과정'
description: ''
date: 2024-09-02
update: 2024-09-02
tags:
  - React
series: '모던 리액트 Deep Dive'
---

### 1. 리액트의 렌더링이란?

---

- 리액트에서의 렌더링이란 리액트 애플리케이션 트리 안에 있는 모든 컴포넌트들이
- 현재 자신들이 가지고 있는 props와 state의 값을 기반으로 어떻게 UI를 구성하고
- 이를 바탕으로 어떤 DOM 결과를 브라우저에 제공할 것인지 계산하는 일련의 과정을 함
- 컴포넌트가 props와 state와 같은 상태값을 가지고 있지 않다면 오직 컴포넌트가 반환하는 JSX값에 기반에 렌더링이 일어나게 된다.

### 2. 리액트의 렌더링이 일어나는 이유

---

> 렌더링 과정보다 중요한 것은 렌더링이 **언제** 발생하느냐

#### 리액트 렌더링 발생 시나리오

```cs
1. 최초 렌더링: 사용자가 처음 애플리케이션에 진입하면 당연히 렌더링해야 할 결과물이 필요하다.
리액트는 브라우저에 이 정보를 제공하기 위해 최초 렌더링을 수행한다.

2. 리렌더링: 처음 애플리케이션에 진입했을 때 최초 렌더링이 발생한 이후로 발생하는 모든 렌더링을 의미.

[ 리렌더링이 발생하는 경우 ]
- 클래스 컴포넌트의 setState가 실행되는 경우: state의 변화는 컴포넌트 상태의 변화를 의미.
클래스 컴포넌트에서는 state의 변화를 setState 호출을 통해 수행하므로 리렌더링 발생

- 클래스 컴포넌트의 forceUpdate가 실행되는 경우:
클래스 컴포넌트에서 렌더링을 수행하는 것은 인스턴스 메서드인 [render]다.

- 함수 컴포넌트의 useState()의 두번째 배열요소인 [setter]가 실행되는 경우

- 함수 컴포넌트의 useReducer()의 두번째 배열요소인 [dispatch]가 실행되는 경우

- 컴포넌트의 key props가 변경되는 경우
key는 리렌더링이 발생하는 동안 sibling elements사이에서 동일한 요소를 식별하는 값임
리렌더링을 최소화 하기 위해, current 와 workInProgress 트리 사이에서 어떤 컴포넌트가 변경이 있었는지 구별해야 함.
key가 없다면 단순히 파이버 내부의 sibling 인덱스만을 기준으로 판단하게 됨

- props가 변경되는 경우
부모로부터 전달받는 값인 props가 달라지면 이를 사용하는 자식 컴포넌트에서도 변경이 필요하므로 리렌더링이 일어남

- 부모 컴포넌트가 렌더링될 경우
자식컴포넌트도 무조건 리렌더링이 일어난다.
```

### 3. 리액트의 렌더링 프로세스

---

일반적으로 렌더링 결과물은 JSX 문법으로 구성돼있고, 이것이 자바스크립트로 컴파일 되면서 React.createElement()를 호출하는 구문으로 변환된다.

```cs
function Hello() {
  return (
    <TestComponent a={35} b="yceffort">
    안녕하세요
    </TestComponent>
  )
}
```

```cs
// 위 JSX 문법은 다음과 같은 React.createElement를 호출해서 변환된다.
function Hello() {
  return React.createElement(
    TestComponent,
    { a: 35, b: 'yceffort' },
    '안녕하세요'
  )
}
```

결과물은 다음과 같다

```cs
{ type: TestComponent, props: {a:35, b:'yceffort', children: '안녕하세요'} }
```

렌더링 프로세스가 실행되면서 각 컴포넌트의 렌더링 결과물을 수집한 다음, 가상 DOM과 비교해 실제 DOM에 반영하기 위한 변경사항을 하나하나 수집한다.
이렇게 계산하는 과정을 리액트의 재조정이라고 한다. 이러한 재조정 과정이 끝나면 모든 변경사항을 하나의 동시 시퀀스로 DOM에 적용해 변경된 결과물이 보이게 된다.

### 4. 렌더와 커밋

---

**렌더 단계(Render Phase)**

컴포넌트를 렌더링 하고 변경 사항을 계산하는 모든 작업

- 렌더링 프로세스에서 컴포넌트를 실행해 (render() 또는 return) 이 결과와 이전 가상 DOM을 비교하는 과정을 거쳐 변경이 필요한 컴포넌트를 체크하는 단계
  여기서 비교하는 것은 `type, props, key`다.

**커밋 단계(Commit Phase)**

- 렌더 단계의 변경사항을 실제 DOM에 적용해 사용자에게 보여주는 과정
- 이 단계가 끝나야 브라우저의 렌더링이 발생함

**렌더 -> 커밋은 동기적으로 실행됨**

- 이는 렌더링 과정이 길어지면 성능저하로 이어짐.

**비동기 렌더링 시나리오의 효과**

- 특정 컴포넌트 렌더링 작업이 무거워 상대적으로 빠르게 보여줄 수 있는거라도 변경해서 보여줄 수 있다면?
- **의도된 우선순위로 렌더링해 최적화할 수 있는 비동기 렌더링**(동시성 렌더링, 리액트18 도입)

### 5. 메모이제이션

---

**memo를 일단 그냥 다 적용하는 방법 주로 채택**

- 단점: CPU, 메모리를 사용해 이전 렌더링 결과물을 저장해둬야 함
- 하지만 어차피 리액트는 재조정 알고리즘이기에 이전 결과물은 어떻게든 저장해두고 있음.
- 결국 props에 대한 얕은 비교가 발생하는게 다임.

**memo를 하지 않았을 때의 문제가 더 크다**

- 렌더링을 함으로써 생기는 비용
- 컴포넌트 내부의 복잡한 로직 재실행
- 위 두 가지 모두가 모든 자식 컴포넌트에서 반복해서 일어남

**useMemo, useCallback**

```cs
function useMath(number: number) {
  const [double, setDouble] = useState(0)
  const [triple, setTriple] = useState(0)

  useEffect(() => {
    setDouble(number * 2)
    setTriple(number * 3)
  }, [number])

  return { double, triple }
}

export default function App() {
  const [counter, setCounter] = useState(0)
  const value = useMath(10)

  useEffect(() => {
    console.log(value.double, value.triple)
  }, [value])

  function handleClick() {
    setCounter((prev) => prev + 1)
  }

  return (
    <>
      <h1>{counter}</h1>
      <button onClick={handleClick}>+</button>
    </>
  )
}
```

- handleClick으로 강제로 렌더링을 일으키면 value값이 실제로 변하지 않아도 계속해서 console.log가 출력된다.
- 함수 컴포넌트인 App이 호출되면서 useMath가 계속해서 호출되고, 객체 내부의 값 같지만 참조가 변경되기 때문

```cs
function useMath(number: number) {
  const [double, setDouble] = useState(0)
  const [triple, setTriple] = useState(0)

  useEffect(() => {
    setDouble(number * 2)
    setTriple(number * 3)
  }, [number])

  return useMemo(() => ({ double, triple }), [double, triple])
}
```

- 따라서 useMath에 메모이제이션을 하면 컴포넌트 자신의 리렌더링 뿐만 아니라
- 이를 사용하는 쪽에서도 변하지 않는 고정값을 사용할 수 있음
