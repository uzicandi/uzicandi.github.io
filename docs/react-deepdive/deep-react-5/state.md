---
title: '5-1. 상태관리 라이브러리'
description: ''
date: 2024-09-08
update: 2024-09-08
tags:
  - React
series: '모던 리액트 Deep Dive'
---

### ContextAPI

상태관리가 아닌, 주입을 도와주는 기능
렌더링을 막아주는 기능 존재하지 않음

### React Query와 SWR

외부에서 데이터를 불러오는 fetch를 관리하는 데 특화된 라이브러리.
API 호출에 대한 상태를 관리하고 있기 때문에 **HTTP 요청에 특화된 상태관리 라이브러리**

```cs
import React from 'react'
import useSWR from 'swr'

const fetcher = (url) => fetch(url).then((res) => res.json())

export default function App() {
  const {data, error} = useSWR(
    'https://api.github.com/repos/vercel/swr', // 조회할 API 주소
    fetcher // 조회에 사용되는 fetch
  )

  if(error) return 'An error has occured'
  if(!data) return 'Loading...'

  return (
    <div>
      <p>{JSON.stringify(data)}</p>
    </div>
  )
}
```

### Recoil, Zustand, Jotai, Valtio

- 훅을 활용해 작은 크기의 상태를 효율적으로 관리한다는 점
- 기존 상태관리 라이브러리의 아쉬운 점으로 지적받던 전역 상태 관리 패러다임에서 벗어나 개발자가 원하는 만큼의 상태를 지역적으로 관리하는 것을 가능하게 만듦
- 훅을 지원함으로써 함수컴포넌트에서 손쉽게 사용할 수 있음

```cs
// Recoil
const counter = atom({ key: 'count', default: 0 })
const todoList = useRecoilValue(counter)
```

```cs
// jotai
const countAtom = atom(0)
const [count, setCount] = useAtom(countAtom)
```

```cs
// zustand
const useCounterStore = create((set) => ({
  count: 0,
  increase: () => set((state) => ({ count: state.count + 1 }))
}))
const count = useCounterStore((state) => state.count)
```

- **Recoil, Jotai**는 Context와 Provider, 훅을 기반으로 가능한 작은 상태를 효율적으로 관리
- **Zustand**는 리덕스와 비슷하게 하나의 큰 스토어를 기반으로 상태를 관리
  - Recoil, Jotai와는 다르게 하나의 큰 스토어는 Context가 아니라 스토어가 가지는 클로저를 기반으로 생성
  - 이 스토어의 상태가 변경되면 이 상태를 구독하고 있는 컴포넌트에 전파에 리렌더링을 알리는 방식

이 라이브러리들의 목적? 내부는 어떻게 상태관리? 각 컴포넌트로 어떻게 전파해 렌더링을 일으키는지?

### Recoil

- Recoil의 상태값은 RecoilRoot로 생성된 Context의 스토어에 저장된다.
- 스토어의 상태값에 접근할 수 있는 함수들이 있으며 이 함수를 활용해 상태값에 접근하거나 상태값을 변경할 수 있다
- 값의 변경이 발생하면 이를 참조하고 있는 하위 컴포넌트에 모두 알린다.

### Jotai

- Recoil의 atom모델에 영감을 받아 만들어진 상태 관리 라이브러리
- **상향식(bottom-up)** 접근법
  - 리덕스 처럼 하나의 큰 상태를 애플리케이션에 내려주는게 아니라
  - 작은 단위의 상태를 위로 전파할 수 있는 구조
- 리액트 Context의 문제점인 **불필요한 리렌더링이 일어난다는 문제를 해결**하고자 설계
- atom을 도입하면서도 API가 간결하다. 사용자는 키를 관리할 필요가 없다.

### Zustand

- 리덕스에 영감을 받아, 하나의 스토어를 중앙 집중형으로 활용해 스토어 내부에서 상태 관리
- 미들웨어 지원, 스토어 데이터를 영구히 보존할 수 있는 persist, 복잡한 객체를 관리하기 쉽게 도와주는 immer, 리덕스와 함께 사용할 수 있는 리덕스 미들웨어 등 여러가지 미들웨어를 제공
