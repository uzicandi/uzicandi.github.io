---
title: '7. 연산자(Operator)'
description: ''
date: 2024-08-04
update: 2024-08-04
tags:
  - Javascript
series: '모던 자바스크립트 Deep Dive'
---

### 암묵적 타입 변환(=타입 강제 변환)

---

```cs
// number + string : 문자열 연결 연산자
"1" + 2; // '12'
1 + "2"; // '12'

// boolean + number
// true = 1, false = 0으로 타입변환
1 + true; // 2
1 + false; // 1

// number + null
// null은 0으로 타입변환한다.
1 + null; // 1

// number + undefined
1 + undefined; // NaN
```

### 동등비교 vs 일치비교

---

- `동등비교(loose equality)`
  - `==`
  - `느슨한 비교`: 좌항과 우항의 피연산자를 비교할 때, 먼저 `암묵적 타입 변환을 통해` 타입을 일치시킨 후, 값이 같은지 비교

```c
ex) 동등비교
5 == 5; // true

5 == "5"; // true
```

- `일치비교(strict equality)`
  - `===`
  - `엄격한 비교`: 타입도 같고, 값도 같은지 비교, 즉, 암묵적 타입 변환을 하지 않고 값을 비교

```c
5 == "5"; // false
```

### NaN, +0 & -0, Object.is() 매서드

---

> NaN은 자신과 일치하지 않는 유일한 값이다.

```c
NaN === NaN; // false
```

NaN인지 확인 필요시 -> 빌트인 함수 `isNaN(value)`를 사용한다.

> +0, -0이 존재한다. 다만 이 둘을 비교하면 true를 반환한다.

```c
✍🏻 Object.is 메서드

ES6에서 도입된 Object.is 메서드는 "예측 가능한 정확한 비교 결과를 반환한다."
그 외에는 일치 비교 연산자(===)와 동일하게 동작한다.

+0 === -0 // true
Object.is(-0, +0) // false

NaN === NaN; // false
Object.is(NaN, NaN) // true
```

### typeof 연산자

> typeof 연산자는 피연산자의 데이터 타입을 문자열로 반환한다.

- 총 7가지 문자열 형태로 반환

  - string, number, boolean, undefined, symbol, object, function

- null을 반환하는 경우는 없다

```c
// EX

typeof ""; // "string"
typeof 1; // "number"
typeof NaN; // "number"
typeof true; // "boolean"
typeof undefined; // "undefined"
typeof Symbol(); // "symbol"
typeof null; // "object" << 🔎 JS의 버그!
typeof []; // "object"
typeof {}; // "object"
typeof new Date(); // "object"
typeof /test/gi; // "object" << 🔎 ( 정규표현식 )
typeof function () {}; // "function"

```

```cs
// 💡 null 타입인지 확인할 때는 "일치 연산자(===)" 를 사용할 것
const foo = null;

typeof foo === null; // false
foo === null; // true
```

```cs
// 💡 선언하지 않은(undeclared) 식별자(= 변수)에 대해서 typeof 연산시, ReferenceError가 아닌 "undefined 를 반환"한다.
typeof undeclared; // undefined
```

### 지수연산자

- ES7에서 도입된 지수연산자

```c
2 ** 2; // 4
2 ** 0; // 1
2 ** -2; // 0.25
```

- 지수 연산자 도입되기 이전에는 Math.pow(x, y) 메서드를 사용했다.

```c
Math.pow(2, 2); // 4
Math.pow(2, 0); // 1
Math.pow(2, -2); // 0.25
```

- 음수를 거듭제곱 할 때는, 제곱할 수(= 밑)를 괄호 로 묶어야 한다.

```c
-5 ** 2;  // SyntaxError: Unary operator used immediately before exponentiation expression.
(-5) ** 2;  // 25
```

### 그외 연산자 간단 소개

| 연산자     | 개요                                                        |
| ---------- | ----------------------------------------------------------- |
| ?.         | 옵셔널 체이닝 연산자                                        |
| ??         | null 병합 연산자                                            |
| delete     | 프로퍼티 삭제                                               |
| new        | 생성자 함수를 호출할 때 사용하여 인스턴스 생성              |
| instanceof | 좌변의 객체가 우변의 생성자 함수와 연결된 인스턴스인지 판별 |
| in         | 프로퍼티 존재 확인                                          |
