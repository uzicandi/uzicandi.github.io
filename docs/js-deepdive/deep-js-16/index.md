---
title: '16. 프로퍼티 어트리뷰트(Property Attribute)'
description: ''
date: 2024-08-13
update: 2024-08-13
tags:
  - Javascript
series: '모던 자바스크립트 Deep Dive'
---

### 16.1 내부 슬롯(Internal Slot)과 내부 메서드(Internal Method)

---

- 자바스크립트 엔진의 구현 알고리즘을 설명하기 위해 ECMAScript 사양에서 사용하는 `의사 프로퍼티(pseudo property)`와 `의사 메서드(pseudo method)`다.
- `내부 슬롯 & 내부 메서드` 는 자바스크립트 엔진에서 실제로 동작은 하지만 개발자가 직접 접근할 수 있도록 외부로 공개된 객체의 프로퍼티는 아니다.
- 즉, 자바스크립트 엔진의 `내부 로직` 이므로 자바스크립트로 직접 접근하거나 호출할 수 있는 방법을 제공하진 않는다.
- 단, 일부 내부 슬롯 & 메서드에 대해서는 접근할 수 있는 수단을 제공한다.
  - 예를 들어 `[[Prototype]]` 내부 슬롯은 `__proto__` 를 통해 간접 접근 가능

```cs
const o = {};

// 내부 슬롯은 자바스크립트 엔진의 내부 로직이므로 직접 접근할 수 없다.
o.[[Prototype]] // Uncaught SyntaxError: Unexpected token '['
// 단, 일부 내부 슬롯과 내부 메서드에 한하여 간접적으로 접근할 수 있는 수단을 제공하기는 한다.
o.__proto__ // Object.prototype
```

### 16.2 Property Attribute & Property Descriptor Object

---

### Property Attribute

> 자바스크립트 엔진은 프로퍼티를 생성할 때 `프로퍼티의 상태`를 나타내는 `프로퍼티 어트리뷰트`를 기본값으로 자동 정의한다.

- 프로퍼티 상태

  - `프로퍼티의 값(value)`
  - `값의 갱신 가능 여부(writable)`
  - `열거 가능 여부(enumerable)`
  - `재정의 가능 여부(configurable)`

- 프로퍼티 어트리뷰트에 직접 접근할 수 없지만 `Object.getOwnPropertyDescriptor(객체의 참조, property key)` 메서드를 사용하여 간접적으로 확인할 수는 있다.

```cs
const person = {
  name: 'Lee'
};

// 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체를 반환한다.
console.log(Object.getOwnPropertyDescriptor(person, 'name'));
// {value: 'Lee', writable: true, enumerable: true, configurable: true}

```

- 객체: `객체의 참조(Reference)`를 전달
- 프로퍼티 키: `문자열 or 심벌`을 전달

### Property Descriptor Object

> 프로퍼티 어트리뷰트 정보를 제공하는 객체

- 존재하지 않는 프로퍼티나 상속받은 프로퍼티에 대한 프로퍼티 디스크립터를 요구하면 -> undefined 반환
- 기본적으로 하나의 프로퍼티에 대해 프로퍼티 디스크립터 객체를 반환

  - ES8에 모든 프로퍼티의 `프로퍼티 어트리뷰트 정보`를 제공하는 프로퍼티 디스크립터 객체를 반환할 수 있게 되었다.

  ```cs
  const person = {
    name: 'Lee'
  };

  // 프로퍼티 동적 생성
  person.age = 20;

  // 모든 프로퍼티의 프로퍼티 어트리뷰트 정보를 제공하는 프로퍼티 디스크립터 객체를 반환한다.
  console.log(Object.getOwnPropertyDescriptors(person));

  /*
  {
    name: {value: 'Lee', writable: true, enumerable: true, configurable: true},
    age: {value: 20, writable: true, enumerable: true, configurable: true}
  }
  */
  ```

### 16.3 데이터 프로퍼티와 접근자 프로퍼티

---

- 데이터 프로퍼티(data property)
  - 키와 값으로 구성된 일반적인 프로퍼티.
- 접근자 프로퍼티(accessor property)
  - 자체적으로는 값을 갖지 않고 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 호출되는 접근자 함수로 구성된 프로퍼티다.

#### 1. 데이터 프로퍼티

다음과 같은 프로퍼티 어트리뷰트를 갖는다. 자바스크립트 엔진이 프로퍼티를 생성할 때 기본적으로 자동 정의 된다.

| 프로퍼티 어트리뷰트 | 프로퍼티 디스크립터 객체의 프로퍼티 | 설명                                                                                                                                                                                                                                                            |
| ------------------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [[Value]]           | value                               | <li> 프로퍼티 키를 통해 프로퍼티 값에 접근하면 반환 되는 값 </li> <li>프로퍼티 키를 통해 프로퍼티 값을 변경하면 [[Value]]에 값을 재할당 한다. 이때 프로퍼티가 없으면 프로퍼티를 동적 생성하고 생성된 프로퍼티의 [[Value]]에 값을 저장한다.</li>                 |
| [[Writable]]        | writable                            | <li>프로퍼티 값 변경 가능 여부 → boolean </li> <li>[[writable]]의 값이 false인 경우 해당 프로퍼티의 [[Value]]값을 변경할 수 없는 읽기 전용 프로퍼티가 된다.</li>                                                                                                |
| [[Enumerable]]      | enumerable                          | <li>프로퍼티의 열거 가능 여부 → boolean </li> <li> false인 경우, `for...in` 문이나 `Object.keys` 메서드 등으로 열거할 수 없다</li>                                                                                                                              |
| [[Configurable]]    | configurable                        | <li>프로퍼티의 재정의 가능 여부 → boolean </li> <li> [[Configurable]]의 값이 false인 경우 해당 프로퍼티의 삭제, 프로퍼티 어트리뷰트 값의 변경이 금지된다. 단, [[Writable]]이 true인 경우 [[value]]의 변경과 [[writable]]을 false로 변경하는 것은 허용된다.</li> |

- 프로퍼티 생성 시
  - [[Value]] 값은 프로퍼티 값으로 초기화
  - [[Writable]], [[Enumerable]], [[Configurable]] 는 모두 true 로 초기화
  - 프로퍼티를 동적 생성 해도 마찬가지로 적용

#### 2. 접근자 프로퍼티

- 자체적으로는 값을 갖지 않고
- 다른 데이터 프로퍼티의 값을 읽거나 저장할 때 사용하는 `접근자 함수`로 구성된 프로퍼티다.
- getter/setter 함수라고도 부른다.

| 프로퍼티 어트리뷰트 | 프로퍼티 디스크립터 객체의 프로퍼티 | 설명                                                                     |
| ------------------- | ----------------------------------- | ------------------------------------------------------------------------ |
| [[Get]]             | get                                 | 데이터 프로퍼티의 값을 읽을 때 호출되는 접근자 함수 → getter 함수 호출   |
| [[Set]]             | set                                 | 데이터 프로퍼티의 값을 저장할 때 호출되는 접근자 함수 → setter 함수 호출 |
| [[Enumerable]]      | enumerable                          | 데이터 프로퍼티의 [[Enumerable]] 와 같다                                 |
| [[Configurable]]    | configurable                        | 데이터 프로퍼티의 [[Configurable]] 와 같다                               |

```cs
const person = {
  // 데이터 프로퍼티
  firstName: 'Jiwoo',
  lastName: 'Kim'

  // getter/setter 함수의 이름 fullName이 접근자 프로퍼티다.
  // property attribute [[value]]를 가지지 않으며 다만 데이터 프로퍼티의 값을 읽거나 저장할 때 관여할 뿐이다.
  // getter 함수
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  },

  // setter 함수
  set fullName(name) {
    // 배열 디스트럭쳐링 할당: 31.1 배열 디스트럭처링 할당" 참고
    [this.firstName, this.lastName] = name.split('');
  }
};

// 1️⃣ 데이터 프로퍼티를 통한 프로퍼티 값의 참조.
console.log(person.firstName + ' ' + person.lastName); // Jiwoo Kim

// 2️⃣ 접근자 프로퍼티를 통한 프로퍼티 값의 저장
// 접근자 프로퍼티 fullName에 값을 저장하면 setter함수가 호출된다.
person.fullName = 'Lina Kim';
console.log(person); // {firstName: 'Lina', lastName: 'Kim'}

// 접근자 프로퍼티를 통한 프로퍼티 값의 참조
// 접근자 프로퍼티 fullName에 접근하면 getter 함수가 호출된다.
console.log(person.fullName); // Lina Kim

// firstName은 데이터 프로퍼티다.
// 데이터 프로퍼티는 [[Value]], [[Writable]], [[Configurable]]
// 프로퍼티 어트리뷰트를 갖는다.
let descriptor = Object.getOwnPropertyDescriptor(person, 'firstName');
console.log(descriptor);
// {value: 'Lina', writable: true, enumerable: true, configurable: true}

// fullName은 접근자 프로퍼티다.
// 접근자 프로퍼티는 [[Get]], [[Set]], [[Enumerable]], [[Configurable]] 프로퍼티 어트리뷰트를 갖는다.
descriptor = Object.getOwnPropertyDescriptor(person, 'fullName');
console.log(descriptor);
// {get: f, set: f, enumerable: true, configurable: true}
```

**접근자 프로퍼티 fullName으로 프로퍼티 값에 접근하면 내부적으로 [[Get]] 내부 메서드가 호출되어 다음과 같이 동작한다.**

```cs
1️⃣ 프로퍼티 키가 유효한지 확인한다. 프로퍼티 키는 문자열 또는 심벌이어야 한다.
2️⃣ 프로토타입 체인에서 프로퍼티를 검색한다. person 객체에 fullName 프로퍼티가 존재한다.
3️⃣ 검색된 fullName 프로퍼티가 데이터 프로퍼티인지 접근자 프로퍼티인지 확인한다.
fullName 프로퍼티는 접근자 프로퍼티다.
4️⃣ 접근자 프로퍼티 fullName의 프로퍼티 어트리뷰트 [[Get]]의 값, 즉 getter 함수를 호출하여 그 결과를 반환한다.
```

```cs
[ 💡 Prototype ]

'프로토타입'은 어떤 객체의 상위(부모) 객체의 역할을 하는 객체다.
프로토타입은 하위(자식) 객체에게 자신의 프로퍼티와 메서드를 상속한다.
이를 상속받은 하위 객체는 자신의 프로퍼티 또는 메서드인 것 처럼 자유롭게 사용할 수 있다.

'프로토타입 체인'은 프로토타입이 단방향 링크드리스트 형태로 연결되어 있는 상속구조를 말한다.
객체의 프로퍼티나 메서드에 접근하려고 할 때 해당 객체에 접근하려는 프로퍼티 또는 메서드가 없다면
프로토타입 체인을 따라 프로토타입의 프로퍼티나 메서드를 차례대로 검색한다.
```

**접근자 프로퍼티와 데이터 프로퍼티를 구별하는 방법**

```cs
// 일반 객체의 __proto__는 접근자 프로퍼티다.
Object.getOwnPropertyDescriptor(Object.prototype, '__proto__');
// {get: f, set: f, enumerable: false, configurable: true}

// 함수 객체의 prototype은 데이터 프로퍼티다.
Object.getOwnPropertyDescriptor(function() {}, 'prototype');
// {value: {...}, writable: true, enumerable: false, configurable: true}
```

### 16.4 프로퍼티 정의

---

> 새로운 프로퍼티를 추가하면서 property attribute를 명시적으로 정의하거나,
> 기존 프로퍼티의 property attribute를 재정의하는 것을 말한다.

- 프로퍼티 값을 갱신(writable) 가능하도록 할 것인지
- 프로퍼티를 열거(enumerable) 가능하도록 할 것인지
- 프로퍼티를 재정의(configurable) 가능하도록 할 것인지
  - 객체의 프로퍼티가 어떻게 동작해야 하는지를 명확히 정의할 수 있다.

`Object.defineProperty` 메서드를 사용하면 프로퍼티의 attribute를 정의할 수 있다.

- 한번에 하나의 프로퍼티만 정의할 수있다.
- `Object.defineProperties`를 사용하면 여러 개의 프로퍼티를 한 번에 정의할 수 있다.

- 인수로는 `객체의 참조`, `데이터 프로퍼티의 key`인 문자열, `property descriptor object`를 전달

```cs
const person = {};

// 데이터 프로퍼티 정의
Object.defineProperty(person, 'firstName', {
  value: 'Jiwoo',
  writable: true,
  enumerable: true,
  configurable: true
});

Object.defineProperty(person, 'lastName', {
  value: 'Kim'
});

let descriptor = Object.getOwnPropertyDescriptor(person, 'firstName');
console.log('firstName', descriptor);

// 디스크립터 객체의 프로퍼티를 누락시키면 undefined, false가 기본값이다.
descriptor = Object.getOwnPropertyDescriptor(person, 'lastName');
console.log('lastName', descriptor);

```

### 16.5 객체 변경 방지

---

- 객체는 변경 가능한 값이므로 재할당 없이 직접 변경할 수 있다.
  - 프로퍼티를 추가, 삭제, 값 갱신 모두 할 수 있다.
  - `object.defineProperty` or `object.defineProperties` 메서드를 사용하여 프로퍼티 어트리뷰트를 재정의할 수도 있다.

> 그러나 객체를 변경하지 말아야할 경우가 존재하고, 이를 지원하는 메서드를 제공한다.

- 객체 변경 방지 메서드 들은 금지하는 강도가 다르다.
- `strict mode`에서 **허용하지 않는 프로퍼티 제어 동작**에 대해서는 `에러를 발생시킴`
- `strict mode`가 아닌 경우에서는 해당 명령을 무시한다.

| 구분           | 메서드                   | 프로퍼티 추가 | 프로퍼티 삭제 | 프로퍼티 값 읽기 | 프로퍼티 값 쓰기 | 프로퍼티 어트리뷰트 재정의 |
| -------------- | ------------------------ | ------------- | ------------- | ---------------- | ---------------- | -------------------------- |
| 객체 확장 금지 | Object.preventExtensions | X             | O             | O                | O                | O                          |
| 객체 밀봉      | Object.isSealed          | X             | X             | O                | O                | X                          |
| 객체 동결      | Object.freeze            | X             | X             | O                | X                | X                          |

#### 1. 객체 확장 금지

확장이 금지된 객체는 프로퍼티 추가가 금지된다.

```cs
Object.preventExtensions(객체);

// 확장 가능한 객체인지 판단 여부 메서드 -> boolean
Object.isExtensible(객체);

```

#### 2. 객체 밀봉

밀봉된 객체는 읽기와 쓰기만 가능하다.

```cs
Object.seal(객체);

// 밀봉된 객체인지 판단 여부 메서드 -> boolean
Object.isSealed(객체);

```

#### 3. 객체 동결

동결된 객체는 읽기만 가능하다

```cs
Object.freeze(객체);

// 동결된 객체인지 판단 여부 메서드 -> boolean
Object.isFrozen(객체);
```

#### 4. 불변 객체

위의 변경방지 메서들은 얕은 변경 방지(shallow only)로 직속 프로퍼티만 변경이 방지되고 중첩 객체까지는 영향을 주지 못한다.

그래서 Object.freeze로 객체를 동결해도 중첩 객체까지 동결할 수는 없다.

따라서 중첩객체까지 동결하는 읽기 전용의 불변 객체를 구현하려면 객체를 값으로 갖는 모든 프로퍼티에 대해 재귀적으로 Object.freeze 메서드를 호출해야 한다.

```cs
function deepFreeze(target) {
  // 객체가 아니거나 동결된 객체는 무시하고 객체이고 동결되지 않은 객체만 동결한다.
  if(target && typeof target === 'object' && !Object.isFrozen(target)) {
    Object.freeze(target);
    /*
      모든 프로퍼티를 순회하며 재귀적으로 동결한다.
      Object.keys 메서드는 객체 자신의 열거 가능한 프로퍼티 키를 배열로 반환한다.
    */
    Object.keys(target).forEach(key => deepFreeze(target[key]));
  }
  return target;
}
```
