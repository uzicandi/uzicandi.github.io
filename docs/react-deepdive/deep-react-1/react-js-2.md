---
title: '1.2. [JS] 이벤트루프와 비동기통신, JS 문법, 타입스크립트 '
description: ''
date: 2024-07-29
update: 2024-07-29
tags:
  - React
series: '모던 리액트 Deep Dive'
---

## 1.5 이벤트 루프와 비동기 통신의 이해

---

- 싱글스레드에서 작동하는 자바스크립트. 한 번에 하나의 작업만 동기 방식으로 처리할 수 있다.
  - 무조건 응답을 받은 이후에 다른 작업을 처리할 수 있음
  - 비동기는 병렬방식으로 처리하는 것
- 비동기 통신을 도와주는 것이 `이벤트 루프`다.

### (1) 싱글 스레드 자바스크립트

- 프로세스: 프로그램을 구동해 프로그램의 상태가 메모리상에서 실행되는 작업 단위
- 스레드: 하나의 프로세스에서 여러개의 스레드를 만들 수 있고, 스레드끼리는 메모리를 공유할 수 있어 여러개의 작업을 동시에 수행할 수 있다.

```cs
// 💡 자바스크립트는 동기로 작동하기 때문에 아래 코드가 순서대로 실행될 것 같지만 실제로는 1,4,2,3의 순서로 나타난다.
console.log(1)

setTimeout(() => {
  console.log(2)
}, 0)

setTimeout(() => {
  console.log(3)
}, 100)

console.log(4)
```

### (2) 이벤트 루프란?

- 자바스크립트 런타임 외부에서 `비동기 실행`을 돕기 위해 만들어진 장치 (ex. V8)

**Call Stack 과 이벤트 루프**

- 콜스택: 자바스크립트에서 수행해야 할 코드나 함수를 순차적으로 담아두는 스택
- 이벤트루프: 콜스택이 비어있는지 여부를 확인하는 것. 실행할 것이 남아있는지 확인한다.

```cs
✅ 동기작업
// foo를 호출 -> bar, baz를 순차적으로 호출하는 구조
// 호출 스택에 쌓이고 비워지게 된다.
function bar() {
  console.log('bar') // 4. 호출스택에 들어가고, 실행 후 다음 코드로 넘어간다 (콜스택에 foo(), bar() 존재)
} // 5. bar()에 실행할 것이 없으므로 호출스택에서 제거된다.

function baz() {
  console.log('baz') // 7. 호출스택에 들어간다.
} // 8. baz()에 실행할 것이 없으므로 호출스택에서 제거된다.

function foo() {
  console.log('foo') // 2. 호출스택에 들어가고, 실행 후 다음 코드로 넘어간다 (아직 콜스택에 foo()존재)
  bar() // 3. 호출스택에 들어간다
  baz() // 6. 호출스택에 들어간다.
} // 9. foo()에 실행할 것이 없으므로 호출스택에서 제거된다.

foo() // 1. 호출스택에 먼저 들어간다

// 10. 호출스택이 완전히 비워졌다.
```

```cs
✅ 비동기 작업
function bar() {
  console.log('bar') // 10. 호출스택에 넣음
} // 5. bar()에 실행할 것이 없으므로 호출스택에서 제거된다.

function baz() {
  console.log('baz') // 5. 호출스택에 들어간다.
} // 6. baz()에 실행할 것이 없으므로 호출스택에서 제거된다.

function foo() {
  console.log('foo') // 2. 호출스택에 들어가고, 실행 후 다음 코드로 넘어간다 (아직 콜스택에 foo()존재)
  setTimeout(bar(), 0) // 3. 호출스택에 들어간다.
  ✅ -> 타이머 이벤트가 실행되며 태스크 큐로 들어가고, 그대신 바로 스택에서 제거된다.
  baz() // 4. 호출스택에 들어간다.
} // 7. foo()에 실행할 것이 없으므로 호출스택에서 제거된다.
// 8. 호출스택이 완전히 비워졌다.
// 9. ✅ 이벤트 루프가 호출스택이 비워진건 확인했음, 태스크큐에는 setTimeout이 남아있어 bar()를 호출스택에 넣음

foo() // 1. 호출스택에 먼저 들어간다

```

- 이 코드를 보면 `setTimeout`이 **정확히 0초 뒤에 실행된다는 것을 보장하지 못한다**는 것을 이해하게 된다.
- `태스크 큐`: 실행해야 할 태스크의 집합
  - 이벤트 루프는 이러한 태스크 큐를 한 개 이상 가지고 있다.
  - 태스크 큐는 자료구조의 queue(FIFO-First In First Out)가 아니고 **set형태**를 띠고 있다.
  - 실행해야 할 태스크 : **비동기 함수의 콜백 함수, 이벤트 핸들러**
- 콜스택이 비었다면 태스크 큐에 대기 중인 작업이 있는지 확인하고, 이 작업을 실행 가능한 오래 된 것 부터 순차적으로 꺼내와서 실행한다.
- `n초뒤 setTimeout`, `fetch`를 기반으로 실행되는 네트워크 요청은 누가 실행할까?
  - 메인스레드가 아닌 태스크 큐가 할당되는 **별도의 스레드**에서 수행된다.
  - 별도의 스레드에서 태스크 큐에 작업을 할당해 처리하는 것은 `브라우저나 Node.js`의 역할

### (3) 태스크 큐와 마이크로 태스크 큐

- 이벤트 루프는 하나의 마이크로 태스크 큐를 갖고 있고, 기존의 태스크 큐와는 다른 태스크를 처리한다.
  - `Promise`
  - 마이크로 태스크 큐는 기존 태스크 큐보다 우선권을 갖는다.
  - 즉, setTimeout, setInterval은 Promise보다 늦게 실행됨

```cs
function foo() {
  console.log('foo')
}

function bar() {
  console.log('bar')
}

function baz() {
  console.log('baz')
}

setTimeout(foo, 0)
Promise.resolve().then(bar).then(baz)

// bar -> baz -> foo 순으로 실행된다.
```

**렌더링은 언제 실행될까?**

- 1. 태스크 큐 실행 전 마이크로 태스크 큐를 실행한다.
- 2. 마이크로 태스크 큐와 태스크 큐 사이에서 렌더링이 일어난다.

```cs
<html>
  <body>
    <ul>
      <li>동기 코드: <button id="sync">0</button></li>
      <li>태스크 : <button id="macrotask">0</button></li>
      <li>마이크로 태스크: <button id="microtask">0</button></li>
    </ul>
    <button id="macro_micro">모두 동시 실행</button>
  </body>
  <script>
    const button = document.getElementById('run');
    const sync = document.getElementById('sync');
    const macrotask = document.getElementById('macrotask');
    const microtask = document.getElementById('microtask');
    const macro_micro = document.getElementById('macro_micro');

    // 동기 코드로 버튼에 1부터 렌더링
    // ✅ 숫자가 올라가기 전까지는 렌더링이 일어나지 않다가 for문이 끝나야 렌더링이 일어남
    sync.addEventListener('click', () => {
      for(let i = 0; i < 100000; i++) {
        sync.innerHTML = i;
      }
    });

    // setTimeout으로 태스크큐에 작업을 넣어서 1부터 렌더링
    // ✅ 태스크 큐에는 모든 setTimeout 콜백이 큐에 들어가기 전까지 잠깐의 대기시간을 갖다가 1부터 순차적으로 렌더링
    macrotask.addEventListener('click', function() {
      for(let i = 0; i < 100000; i++) {
        setTimeout(() => {
          macrotask.innerHTML = i;
        }, 0);
      }
    });

    // queueMicrotask으로 마이크로태스크 큐에 작업을 넣어서 1부터 렌더링
    // ✅ 마이크로 태스크 큐는 동기코드와 마찬가지로 렌더링이 전혀 일어나지 않다가 다 끝난 이후에야 한번에 렌더링이 일어남
    microtask.addEventListener('click', function() {
      for(let i = 0; i < 100000; i++) {
        queueMicrotask(() => {
          microtask.innerHTML = i;
        });
      }
    });

    macro_micro.addEventListener('click', () => {
      for(let i = 0; i < 100000; i++) {
        sync.innerHTML = i;

        setTimeout(() => {
          macrotask.innerHTML = i;
        }, 0);

        queueMicrotask(() => {
          microtask.innerHTML = i;
        });
      }
    });
  </script>
</html>
```

**브라우저에 다음 리페인트 전에 콜백 함수 호출을 가능하게 하는 requestAnimationFrame으로도 확인할 수 있다**

```cs
// a c d b 순서대로 출력
// 브라우저에 렌더링 = 마이크로 태스크 큐와 태스크 큐사이에서 일어남
console.log('a')

setTimeout(() => {
  console.log('b')
}, 0)

Promise.resolve().then(() => {
  console.log('c')
})

window.requestAnimationFrame(() => {
  console.log('d')
})
```

## 1.6 리액트에서 자주 사용하는 자바스크립트 문법

---

- 바벨이 어떻게 최신 코드를 트랜스파일하는지, 그리고 그 결과 어떤 코드가 생성되는지 이해하면 향후 애플리케이션을 디버깅하는데 도움이 된다.

### (1) 구조 분해 할당

- 주로 어떠한 객체나 배열에서 선언문 없이 즉시 분해해 변수를 선언하고 할당하고 싶을 때 사용한다.

`배열 구조 분해 할당`

```cs
const array = [1,2,3,4,5];

const [first, second, third, ...arrayRest] = array // 1, 2, 3, [4,5]
```

`객체 구조 분해 할당`

객체에서 값을 꺼내온 뒤 할당하는 것.
배열 구조 분해 할당과는 달리 **객체는 객체 내부 이름으로** 꺼내온다.

```cs
const object = {a:1, b:2, c:3, d:4, e:5}
const {a, b, c, ...rest} = object // 1, 2, 3, {d: 4, e:5}

// 새로운 이름으로 다시 할당도 가능
const {a: first, b: second} = object

// 배열과 마찬가지로 기본값을 주는 것도 가능
const {a = 10, b = 10, c = 10} = object;
```

```cs
// 변수에 있는 값으로 꺼내오는 계산된 속성 이름 방식도 가능
const key = 'a'
const object = {
  a: 1,
  b: 1,
}

const {[key]: a} = object
```

### (2) 전개 구문

`배열 전개 구문`

- 과거에는 배열간에 합성을 하려면 push(), concat(), splice() 등의 메서드를 사용해야 했다.

```cs
const arr1 = ['a', 'b'];
const arr2 = [...arr1, 'c', 'd', 'e'];
```

이런 특징을 활용하면 기존 배열에 영향을 미치지 않고 배열을 복사하는 것도 가능하다.

```cs
const arr1 = ['a', 'b'];
const arr2 = [...arr1];  // 값만 복사되고 참조는 다르다.
arr1 === arr2 // false

const arr1 = ['a', 'b'];
const arr2 = arr1 // 내용이 아닌 참조를 복사한다.
arr1 === arr2 // true
```

`객체 전개 구문`

```cs
const obj1 = {a:1};
const obj2 = {b:2};
const newObj = { ...obj1, ...obj2 }
// {"a": 1, "b":2}
```

```cs
const arr1 = ['a','b'];

const arr2 = [...arr1, 'c','d','e'];
// 코드가 바벨에서 어떻게 변환되는지
const arr2 = [].concat(arr1, ['c', 'd', 'e'])
```

- 객체는 비교적 간단하나, 객체는 객체의 속성값 및 설명자 확인, 심벌 체크 등 때문에 트랜스파일된 코드가 커진다.
- 따라서 상대적으로 번들링이 커지기 때문에 사용할 때 주의해야 한다.

### (3) Array 프로토타입의 메서드 (map, filter, reduce, forEach)

- ES5에서부터 사용한 문법이기 때문에 별도의 트랜스파일이나 폴리필이 없어도 부담없이 사용할 수 있다.

**Array.prototype.map**

```cs
const arr = [1, 2, 3, 4, 5];
const doubledArr = arr.map((item) => item * 2)
```

리액트에서는 주로 특정 배열을 기반으로 요소를 반환할 때 많이 사용된다

```cs
const Elements = arr.map((item) => {
  return <Fragment key={item}>{item}</Fragment>
})
```

**Array.prototype.filter**

- 콜백함수를 인수로 받는데, 이 콜백 함수에서 truthy 조건을 만족하는 경우에만 해당 원소를 반환한다.
- 기존 배열에 대해 어떤 조건을 만족하는 새로운 배열을 반환할때

```cs
const arr = [1, 2, 3, 4, 5];
const evenArr = arr.filter((item) => item % 2 === 0)
// [2, 4]
```

**Array.prototype.reduce**

- 초기값에 누적해 결과를 반환한다.

```cs
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce((result, item) => {
  return result + item
},0) // 15
```

```cs
// 짝수만 100을 곱해 반환하는 함수의 예제
const arr = [1, 2, 3, 4, 5];

// [200, 400]
const result1 = arr.filter((item) => item % 2 === 0).map((item) => item * 100)

const result2 = arr.reduce((result, item) => {
  if(item % 2 === 0) {
    result.push(item * 100)
  }
  return result
}, [])
```

**Array.prototype.forEach**

콜백함수를 받아 배열을 순회하면서 그 콜백 함수를 실행하기만 하는 메서드

- return이 있어도 순회를 멈추지 않는다.
- 0(n)만큼 실행되므로 코드 작성과 실행시에 반드시 최적화할 가능성이 있는지 검토해보자

```cs
function run() {
  const arr = [1,2,3];
  arr.forEach((item) => {
    console.log(item);
    if(item === 1) {
      console.log('finished')
      return
    }
  })
} // 1, finished, 2, 3
```

## 1.7 타입스크립트

---

- 타입체크를 정적으로 런타임이 아닌 빌드(트랜스파일) 타임에 수행할 수 있게 해준다.
- 런타임까지 가지 않더라도 **코드를 빌드하는 시점**에 이미 에러가 발생할 가능성이 있는 코드를 확인할 수 있다.
- 타입스크립트는 함수의 반환타입, 배열, enum 등 기존에는 사용하기 어려웠던 타입 관련 작업들을 손쉽게 처리할 수 있다.

### (1) 리액트 코드를 효과적으로 작성하기 위한 타입스크립트 사용법

**any 대신 unknown을 사용하자**

**타입가드를 적극 활용하자**

- instanceof와 typeof: 지정한 인스턴스가 특정 클래스의 인스턴스인지 확인할 수 있는 연산자

- in: 특정 객체에 키가 존재하는지 확인하는 용도

**제네릭**

- 함수나 클래스 내부에서 단일 타입이 아닌 다양한 타입에 대응할 수 있도록 도와주는 도구

**인덱스 시그니처**

- `[key: string]`: 키에 원하는 타입을 부여할 수 있다.
- 키는 동적으로 선언되는 경우는 최대한 지양해야 하고, 객체의 타입도 필요에 따라 좁혀야 한다.

### (2) 타입스크립트 전환가이드
