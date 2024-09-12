---
title: "24. 클로저"
description:
date: 2024-08-23
update: 2024-08-23
tags:
  - Javascript
series: "모던 자바스크립트 Deep Dive"
---

- 자바스크립트는 렉시컬 스코프를 따르는 프로그래밍 언어이다.
- 함수가 중첩되어 있으면 중첩함수는 외부함수의 변수에 접근할 수 있다.

### 24.1 렉시컬 스코프

---

> 자바스크립트 엔진은 함수를 **어디서 호출했는지**가 아니라
> `함수를 어디에 정의했는지`에 따라 `상위스코프`를 결정한다. 이를 `렉시컬 스코프(정적스코프)`라 한다.

### 24.2 함수 객체의 내부 슬롯 [[Environment]]

---

- 함수는 자신의 내부 슬롯 [[Environment]]에 자신이 정의된 환경, 즉 **상위 스코프의 참조**를 저장한다.
- 이는 현재 실행 중인 실행 컨텍스트의 렉시컬 환경을 가리킨다.

#### 함수 코드 평가 순서

```cs
1. 함수 실행 컨텍스트 생성
2. 함수 렉시컬 환경 생성
2.1. 함수 환경 레코드 생성
2.2. this 바인딩
2.3. 외부 렉시컬 환경에 대한 참조 결정
```

### 24.3 클로저와 렉시컬 환경

---

```cs
const x = 1;

// 1️⃣
function outer() {
  const x = 10;
  const inner = function() { console.log(x); }; // 2️⃣
  return inner;
}

const innerFunc = outer(); // 3️⃣
innerFunc(); // 4️⃣
```

- 1️⃣ outer 함수가 평가되어 함수 객체를 생성할 때 현재 실행중인 실행컨텍스트의 렉시컬 환경을 [[Environment]] 내부 슬롯에 상위 스코프로서 저장한다.
- 2️⃣ 이후 중첩함수 inner가 평가된다. (inner함수는 함수표현식으로 정의했기 때문에 런타임에 평가된다.) 이때 중첩함수 Inner는 자신의 [[Environment]] 내부 슬롯에 현재 실행중인 실행 컨텍스트의 렉시컬 환경, 즉 outer 함수의 렉시컬 환경을 상위 스코프로서 저장한다.
- 3️⃣ outer함수를 호출하면 outer함수는 중첩함수 inner를 반환하고 생명주기를 마감한다.
  - 즉, outer 함수의 실행이 종료되면 함수의 실행컨텍스트는 스택에서 제거된다.
  - 이때 outer함수의 렉시컬 환경까지 소멸하는 것은 아니다.
  - outer 함수의 렉시컬 환경은 inner함수의 [[Environment]] 내부 슬롯에 의해 참조되고 있고, inner함수는 전역변수 innerFunc에 의해 참조되고 있으므로 가비지 컬렉션의 대상이 되지 않기 때문이다. 가비지 컬렉션은 누군가 참조하는 메모리 공간을 함부로 해제하지 않는다.
- 4️⃣ 이때 outer함수의 실행컨텍스트가 제거되었으므로, outer 변수의 지역변수 x에 접근하는 방법이 없어보인다.
  - 하지만 4️⃣를 실행하면, 생명주기가 종료되었음에도 지역변수 x에 접근할 수 있다.
  - inner함수를 호출하면, inner함수의 실행컨텍스트가 생성되고, 실행컨텍스트 스택에 푸시된다.
  - 그리고 렉시컬 환경의 외부 렉시컬 환경에 대한 참조는 Inner 함수 객체의 [[Environment]] 내부 슬롯에 저장되어 있는 참조값이 할당된다.

이처럼 **외부 함수보다 중첩함수가 더 오래 유지되는 경우**, **중첩함수는 이미 생명주기가 종료한 외부함수의 변수를 참조할 수 있다.**
이러한 중첩함수를 `클로저`라고 부른다.

---

> 클로저는 함수와 그 **함수가 선언된 렉시컬 환경**과의 조합이다.

- **함수가 선언된 렉시컬 환경**: 함수가 정의된 위치의 스코프, 즉 `상위 스코프`를 의미하는 `실행 컨텍스트의 렉시컬 환경`을 말한다.

- 자바스크립트의 모든 함수는 자신의 상위 스코프를 기억한다.
- 모든 함수가 기억하는 상위 스코프는 함수를 어디서 호출하든 상관없이 유지된다.
- 따라서 함수를 어디서 호출하든 상관없이 함수는 언제나 자신이 기억하는 상위 스코프의 식별자를 참조할 수 있으며 식별자에 바인딩된 값을 변경할 수도 있다.

```cs
// 클로저에 해당하는 경우
1. 중첩함수이고
2. 해당 함수가 상위 함수보다 오래 남아 있으며
3. 상위 함수의 식별자를 참조하고 있는경우
```

- 클로저는 상위 스코프를 기억해야 하므로 불필요한 메모리 점유를 걱정할 수도 있다.
- 하지만 상위 스코프의 식별자 중에서 기억해야 할 식별자만 기억하도록 최적화 되어있다.

### 24.4 클로저의 활용

---

> 클로저를 `상태(state)`를 안전하게 변경하고 유지하기 위해 사용한다.
> 즉, 상태가 의도치 않게 변경되지 않도록 **상태를 안전하게 은닉**하고 **특정 함수에게만 상태를 허용**하기 위해 사용한다.

```cs
// ❌ 안전하지 않은 함수
// 1. num 변수는 누구나 접근할 수 있고 변경할 수 있다.
// 따라서 num은 increase 함수만이 변경할 수 있어야 한다.

let num = 0;

const increase = function () {
  return ++num;
};

console.log(increase()); // 1
console.log(increase()); // 2
console.log(increase()); // 3
```

```cs
// ✅
// 1. num 변수에 접근이 불가하게 한다.
// 2. 클로저를 이용해 num이 이전 num을 가지고 있도록 한다. (함수 호출시 마다 초기화 되지 않도록)

const increase = (function () {
  let num = 0;

  // 클로저
  return function () {
    return ++num;
  }
}());

console.log(increase()); // 1
console.log(increase()); // 2
console.log(increase()); // 3
```

- 상태가 의도치않게 변경되지 않도록 안전하게 은닉(information hiding)하고
- 특정 함수에게만 상태 변경을 허용하여 상태를 안전하게 변경하고 유지하기 위해 사용한다.

```cs
// 💡 count를 증가/감소 시킬 수 있도록 발전시켜보자
const counter = (function() {
  let num = 0;

  // 클로저인 메서드를 갖는 객체를 반환한다.
  // 객체 리터럴은 스코프를 만들지 않는다.
  // 따라서 아래 메서드들의 상위 스코프는 즉시실행함수의 렉시컬 환경이다.
  return {
    // num: 0, 프로퍼티는 public하므로 은닉되지 않는다.
    increase() { // 상위 스코프는 해당 메서드가 평가되는 시점에 실행중인 즉시실행함수 실행컨텍스트의 렉시컬환경이다.
      return ++num;
    },
    decrease() {
      return num > 0 ? --num : 0;
    }
  }
}());

console.log(increase()); // 1
console.log(increase()); // 2
console.log(decrease()); // 1
console.log(decrease()); // 0
```

**함수형 프로그래밍에서 클로저를 활용하는 예제**

- `외부상태 변경`이나 `mutable 데이터를 피하고` 불변성을 지향하는 함수형 프로그래밍에서 적극적으로 사용된다.

```cs
// ✅ 보조함수(aux)를 인수로 전달받고 함수를 반환하는 고차함수
// ✅ 이 함수는 카운트 상태를 유지하기 위한 자유변수 counter를 기억하는 클로저를 반환한다.
function makeCounter(aux) {
  // 카운트 상태를 유지하기 위한 자유변수
  let counter = 0;

  // makeCounter의 렉시컬환경을 상위 스코프로서 기억하는 클로저
  return function() {
    // 인수로 전달받은 aux 보조함수에 상태 변경을 위임한다.
    counter = aux(counter);
    return counter;
  };
}

// 보조함수
function increase(n) {
  return ++n;
}
function decrease(n) {
  return --n;
}

// 함수로 함수를 생성한다.
// makeCounter 함수는 보조함수로 인수를 전달받아 함수를 반환한다.
const increaser = makeCounter(increase);
console.log(increase()); // 1
console.log(increase()); // 2

// ❌ increase 함수와는 별개의 독립된 렉시컬 환경을 갖기 때문에 카운터 상태가 연동하지 않는다.
const decreaser = makeCounter(decrease);
console.log(decrease()); // -1
console.log(decrease()); // -2
```

- 독립된 카운터가 아니라, 연동하여 증감이 가능한 카운터를 만들려면 **렉시컬 환경을 공유하는 클로저**를 만들어야 한다.
- 이를 위해서는 `makeCounter` 함수를 두 번 호출하지 말아야 한다.

```cs
// 함수를 반환하는 고차함수
// 이 함수는 카운트 상태를 유지하기 위한 자유 변수 counter를 기억하는 클로저를 반환한다.
const counter = (function () {
  // 카운트 상태를 유지하기 위한 자유 변수
  let counter = 0;

  // 함수를 인수로 전달받는 클로저를 반환
  return function (aux) {
    // 인수로 전달받은 보조 함수에 상태 변경을 위임한다.
    counter = aux(counter);
    return counter;
  };
}());

// 보조함수
function increase(n) {
  return ++n;
}
function decrease(n) {
  return --n;
}

console.log(counter(increase)); // 1
console.log(counter(increase)); // 2
console.log(counter(decrease)); // 1
console.log(counter(decrease)); // 0
```

### 24.6 자주 발생하는 실수

---

```cs
// ❌ 클로저를 사용할 때 자주 발생할 수 있는 실수
var funcs = [];

for(var i = 0; i < 3; i++) {
    funcs[i] = function () { return i; };
}

for (var j = 0; j < funcs.length; j++) {
    console.log(funcs[j]());
}
// 3 3 3
```

함수가 funcs 배열의 요소로 추가된다. 그리고 두번째 for문의 코드 블록 내에서 funcs 배열의 요소로 추가된 함수를 순차적으로 호출한다.

- 이때 funcs 배열의 요소로 추가된 3개의 함수가 0,1,2를 반환할 것으로 기대하지만, 아쉽게도 그렇지는 않다.
- for문의 변수 선언문에서 var 키워드로 선언한 `i 변수`는 블록 레벨 스코프가 아닌 `함수 레벨 스코프`를 갖기 때문에 `전역 변수`다.
- 전역 변수 i에는 순차적으로 0,1,2가 순차적으로 할당되지만, 함수를 호출하면 전역변수 i를 호출하여 i의 값이 3이된다.

```cs
var funcs = [];

for (var i = 0; i < 3; i++) {
  // 1️⃣ 즉시 실행함수는 전역 변수 i에 "현재" 할당 되어 있는 값을 인수로 받아 할당함.
  funcs[i] = (function (id) {
    return function() {
      return id;
    };
  }(i)); // id는 중첩함수에 묶여 있는 자유변수가 되어 그 값이 유지됨
}

for (var j = 0; j < funcs.length; j++) {
    console.log(funcs[j]());
}

// 0 1 2
```

- 이는 함수레벨 스코프 특성으로 인해 for문의 변수 선언문에서 var 키워드로 선언한 변수가 전역 변수가 되기 때문에 발생하는 현상
- let을 사용하면 번거로움이 해결된다.

```cs
// ✅ let을 사용하면 for문의 코드블록이 반복 실행될 때 마다 새로운 렉시컬 환경이 생성된다.
// 이는 반복문의 코드 블록 내부에서 함수를 정의할 때 의미가 있다.
const funcs = [];

for (let i = 0; i < 3; i++) {
  funcs[i] = function() { return i; };
}

for (let i = 0; i < funcs.length; i++) {
  console.log(funcs[i]()); // 0 1 2
}
```

- 함수형 프로그래밍 기법인 고차함수를 사용하는 방법이 있다.
- 이 방법은 변수와 반복문의 사용을 억제할 수 있기 때문에 오류를 줄이고 가독성을 좋게 만든다.

```cs
// 요소가 3개인 배열을 생성하고 배열의 인덱스를 반환하는 함수를 요소로 추가한다.
// 배열의 요소로 추가된 함수들은 모두 클로저다.
const funcs = Array.from(new Array(3), (_, i) => () => i); // (3) [f,f,f]

// 배열의 요소로 추가된 함수들을 순차적으로 호출한다.
funcs.forEach(f => console.log(f())); // 0 1 2
```
