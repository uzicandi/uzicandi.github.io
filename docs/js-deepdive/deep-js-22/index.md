---
title: '22. this'
description: ''
date: 2024-08-21
update: 2024-08-21
tags:
  - Javascript
series: '모던 자바스크립트 Deep Dive'
---

### 22.1 this 키워드

---

- `객체`는 상태를 나타내는 `프로퍼티`와 동작을 나타내는 `메서드`를 하나의 논리적인 단위로 묶은 복합적인 자료구조다.
- `메서드`는 자신이 속한 객체의 상태, 즉 **프로퍼티를 참조하고 변경할 수 있어야** 한다.
  이때 **메서드가 자신이 속한 객체의 프로퍼티를 참조**하려면, **자신이 속한 객체를 가리키는 식별자를 참조**할 수 있어야 한다.

### this를 왜 사용해야 할까?

> this 는 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수(self-referencing variable) 이다.

- this 를 통해 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있다.
- this 는 자바스크립트 엔진에 의해 암묵적으로 생성된다.

```cs
[ this 바인딩 ]
+ 바인딩(binding) : 식별자와 값을 연결하는 과정
+ this 바인딩 : this와 this가 가리킬 객체를 바인딩하는 것
```

#### `객체 리터럴`의 메서드 내부에서 this는 `메서드를 호출한 객체`다.

```cs
// 객체 리터럴
const circle = {
  radius: 5,
  getDiameter() {
    // 여기서 this는 메서드를 호출한 객체
    return 2 * this.radius;
  }
};

console.log(circle.getDiameter());  // 10
```

#### `생성자 함수` 내부의 this는 `생성자 함수가 생성할 인스턴스`다.

```cs
// 생성자 함수
function Circle(radius) {
  // 여기서 this는 생성자 함수 Circle이 생성할 인스턴스
  this.radius = radius;
}

Circle.prototype.getDiameter = function () {
  // 여기서 this는 생성자 함수가 생성할 인스턴스
  return 2 * this.radius;
};

// 인스턴스
const circle = new Circle(5);
console.log(circle.getDiameter()); // 10
```

> this 가 가리키는 값, 즉 this 바인딩 은 함수 호출 방식에 의해 동적으로 결정된다.

- 자바나 C++ 같은 클래스 기반 언어에서 this는 언제나 `클래스가 생성하는 인스턴스`를 가리킨다.
- 하지만 `자바스크립트의 this`는 함수가 호출되는 방식에 따라 this에 바인딩될 값, 즉 **this 바인딩이 동적으로 결정된다.**
- 또한 `strict mode`는 this 바인딩에 영향을 준다.
  - this는 코드 어디서도 참조가 가능하다. `전역`에서도 `함수내부`에서도 참조할 수 있다.
  - 하지만 this의 본질은 객체의 프로퍼티나 메서드를 참조하기 위한 **자기 참조 변수의 개념**이다.
    - 따라서 객체가 아닌 일반 함수 내부의 this는 필요가 없으므로, strict mode에서 this를 참조할 경우, `undefined`를 반환한다.

### 22.2 함수 호출방식과 this 바인딩

---

함수를 호출하는 방식은 크게 다음과 같다.

- `일반 함수 호출`
- `메서드 호출`
- `생성자 함수 호출`
- `Function.prototype.apply / call / bind 메서드에 의한 간접 호출`

#### 일반 함수 호출

> 기본적으로 this에는 전역 객체(global object)가 바인딩된다.

- 전역 함수는 물론, 중첩 함수를 일반 함수로 호출된 모든 함수(중첩함수, 콜백함수 포함) 내부의 this에는 전역 객체가 바인딩
- 단, strict mode에서 this를 참조할 경우는 undefined가 바인딩

```cs
function foo() {
  console.log(`전역 함수 foo의 this : ${this}`);
  // 전역 함수 foo의 this : [object global]

  function bar() {
    console.log(`중첩 함수 bar의 this : ${this}`);
    // 중첩 함수 bar의 this : [object global]
  }

  bar();
}
foo();
```

```cs
// strict mode 적용
function foo() {
  "use strict";

  console.log(`전역 함수 foo의 this : ${this}`);    // 전역 함수 foo의 this : undefined

  function bar() {
    console.log(`중첩 함수 bar의 this : ${this}`);  // 중첩 함수 bar의 this : undefined
  }

  bar();
}
foo();
```

```cs
// 전역 변수 value (전역 객체 프로퍼티)
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log(`foo 메서드의 this : ${this}`);
    // foo 메서드의 this : [object Object]
    console.log(`foo 메서드의 this가 가리키는 객체의 value : ${this.value}`);
    // foo 메서드의 this가 가리키는 객체의 value : 100

    // 메서드 내, 정의한 중첩함수

  }
}
```

```cs
// 전역 변수 value ( 전역 객체 프로퍼티 )
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log(`foo 메서드의 this : ${this}`); // foo 메서드의 this : [object Object]
    console.log(`foo 메서드의 this가 가리키는 객체의 value : ${this.value}`); // foo 메서드의 this가 가리키는 객체의 value : 100

    // 메서드 내, 정의한 중첩 함수
    function bar() {
      console.log(`메서드 내 중첩 함수 bar의 this : ${this}`); // 메서드 내 중첩 함수 bar의 this : [object global]
      console.log(`중첩 함수 bar의 this가 가리키는 객체의 value : ${this.value}`); // 중첩 함수 bar의 this가 가리키는 객체의 value : 1
    }

    // 메서드 내 중첩 함수를 일반 함수 처럼 호출
    bar();
  },
};

obj.foo();
```

```cs
// 전역 변수 value ( 전역 객체 프로퍼티 )
var value = 1;

const obj = {
  value: 100,
  foo() {
    console.log(`foo 메서드의 this : ${this}`); // foo 메서드의 this : [object Object]
    console.log(`foo 메서드의 this가 가리키는 객체의 value : ${this.value}`); // foo 메서드의 this가 가리키는 객체의 value : 100

    // 메서드 내 콜백 함수
    setTimeout(function () {
      console.log(`callback 함수의 this : ${this}`); // callback 함수의 this : [object global]
      console.log(`callback 함수의 this가 가리키는 객체의 value : ${this.value}`); // callback 함수의 this가 가리키는 객체의 value : 1
    }, 100);
  },
};

obj.foo();
```

- 중첩 함수나 콜백 함수의 경우, 용도가 일반적으로 외부 함수를 돕는 헬퍼 함수의 역할을 하므로, this가 가리키는 객체가 전역 객체일 경우에는 헬퍼 함수의 역할을 하기 힘들다.

- 메서드 내부의 중첩 함수나 콜백 함수의 this 바인딩을 메서드의 this 바인딩과 일치시키기 위한 방법은 다음과 같다.
  - 메서드의 this 바인딩할 객체를 `변수에 할당하는 방법`
  - `Function.prototype.apply` or `[Function.prototype.call]` `(http://Function.prototype.call)`에 의한 바인딩

```cs
var value = 1;

const obj = {
  value: 100,
  foo() {
    // 콜백 함수에 바인딩할 obj 객체를 가리키는 this를 변수 that에 할당
    const that = this;

    setTimeout(function () {
      console.log(`callback 함수의 this : ${that}`); // callback 함수의 this : [object Object]
      console.log(`callback 함수의 this가 가리키는 객체의 value : ${that.value}`); // callback 함수의 this가 가리키는 객체의 value : 100
    }, 100);
  },
};

obj.foo();
```

```cs
var value = 1;

const obj = {
  value: 100,
  foo() {

    setTimeout(
      function () {
        console.log(`callback 함수의 this : ${this}`); // callback 함수의 this : [object Object]
        console.log(`callback 함수의 this가 가리키는 객체의 value : ${this.value}`); // callback 함수의 this가 가리키는 객체의 value : 100
      }.bind(this),
      // Function.prototype.bind 메서드로 obj 객체를 가리키는 this를 콜백함수에 명시적으로 바인딩
      100,
    );
  },
};

obj.foo();
```

#### 메서드 호출

> 기본적으로 메서드 내부의 `this` 에는 메서드를 호출한 `객체` 가 바인딩된다.

- 즉, 메서드를 호출할 때 메서드 앞에 마침표(.) 연산자 앞에 기술한 객체가 바인딩
  - 주의할 점은 메서드 내부의 this는 `메서드를 소유한 객체`가 아닌, `메서드를 호출한 객체에 바인딩` 된다는 점

```cs
const person = {
  name: 'LEE',
  getName() {
    // 메서드 내부의 this는 메서드를 호출한 객체에 바인딩
    // getName 메서드는 person 객체에 포함된 것이 아닌, 독립적으로 존재하는 별도의 객체
    return this.name;
  }
};

console.log(person.getName()); // LEE
```

- 메서드는 특정 객체에 포함된 것이 아닌, `독립적으로 존재하는 별도의 객체`
  - 즉, 다른 객체의 프로퍼티에 할당이 가능
  - 일반 변수에 할당하여 일반 함수로 호출도 가능

```cs
const person = {
  name: 'Lee',
  getName() {
    // 메서드 내부의 this는 메서드를 호출한 객체에 바인딩
    // getName 메서드는 person 객체에 포함된 것이 아닌, 독립적으로 존재하는 별도의 객체
    return this.name;
  }
};

const anotherPErson = {
  name: 'Kim"
};

// getName 메서드를 anotherPerson 객체의 메서드로 할당 (getName 메서드는 독립적인 객체이기 때문)
anotherPerson.getName = person.getName;

console.log(anotherPerson.getName());  // KIM

const getName = person.getName;

// getName을 호출한 객체는 이 시점에서는 전역 객체다.
// 전역 객체에 프로퍼티에는 name 이라는 프로퍼티가 존재하지 않다.
// 참조 시, 자바스크립트 엔진이 암묵적으로 undefined 로 초기화한다.
console.log(getName());  // undefined
```

- 프로토타입 메서드 내부에서 사용된 this도 일반 메서드와 마찬가지로 해당 메서드를 호출한 객체에 바인딩

```cs
// 생성자 함수
function Person(name) {
  this.name = name;
}

// 프로토타입에 getName 메서드 할당
Person.prototype.getName = function () {
  return this.name;
};

// me 인스턴스 생성
const me = new Person("Lee");
console.log(me.getName()); // Lee

Person.prototype.name = "KIM";
console.log(Person.prototype.getName()); // KIM
```

#### 생성자 함수 호출

> 생성자 함수 내부의 this에는 `함수가 생성할 인스턴스`가 바인딩 된다.

- 생성자 함수의 역할은 객체(인스턴스)를 생성하는 함수

```cs
// 생성자 함수
function Person(name) {
  this.name = name;
  this.getName = function () {
    return `안녕하세요. 저는 ${this.name}입니다.`;
  };
}

const person1 = new Person("Lee");
const person2 = new Person("KIM");

console.log(person1.getName()); // 안녕하세요. 저는 Lee입니다.
console.log(person2.getName()); // 안녕하세요. 저는 KIM입니다.
```

### Function.prototype.apply / call / bind 메서드에 의한 간접 호출

---

> `apply, call, bind` 는 `Function.prototype`의 메서드이다.

- 즉, 모든 함수가 이 메서드들을 상속 받아 사용 가능

  - Function.prototype.apply(this로 사용할 객체, arguments 리스트(배열 or 유사 배열객체))
  - Function.prototype.call(this로 사용할 객체, arguments 인수 리스트(,로 구분하여 전달))
  - Function.prototype.bind(this로 사용할 객체)

- `apply, call`
  - 본질적인 기능은 `함수를 호출하는 것`
  - 함수를 호출하면서 `첫번째 인수`로 전달한 특정 객체를 호출한 함수의 `this에 바인딩`
  - `두번째 인수`를 `함수에 전달하는 방식`만 다를 뿐 동일하게 작동
  - 대표적으로, `arguments 객체`와 같은 유사 배열 객체에 `배열 메서드를 사용하는 경우` 효과적

```cs
function convertArgsToArray() {
  console.log(arguments); // [Arguments] { '0': 1, '1': 2, '2': 3 }

  // 유사 배열 객체인 arguments 객체는 배열이 아니다.
  // 그런 arguments 객체에 대해, 배열 메서드인 Array.prototype.slice 메서드를
  // Function.prototype.call 메서드로 arguments 객체에 바인딩한다.
  // 유사 배열 객체 arguments 객체임에도 배열 메서드인 slice 메서드를 적용하여 반환된 새로운 배열 객체를 반환할 수 있게 된다.

  const arr = Array.prototype.slice.call(arguments);

  console.log(arr); // [ 1, 2, 3 ]

  return arr;
}
convertArgsToArray(1, 2, 3);

```

- `bind`
  - 본질적인 기능은 함수를 호출하진 않고 `this로 사용할 객체만 전달`
  - 대표적으로, `메서드의 this`와 `메서드 내부의 중첩 함수 or 콜백 함수의 this`가 불일치하는 문제 해결에 효과적

```cs
const person = {
  name: "Lee",
  foo(callback) {
    // bind 를 적용하지 않는다면, foo 메서드 내부에 콜백 함수에 정의된 this는 전역 객체(window 또는 global)를 가리킨다.
    // 전역 객체에는 name 프로퍼티가 없기 때문에, 원래는 undefined 를 출력하는 것이 맞다.
    // 하지만, Function.prototype.bind 메서드로 콜백 함수의 주체를 person 객체로 동적 바인딩 해주었다.
    // 때문에 person 객체의 name 프로퍼티에 접근할 수 있게 되었다.
    setTimeout(callback.bind(this), 100);
  },
};

person.foo(function () {
  console.log(`안녕하세요. ${this.name}입니다.`); // 안녕하세요. Lee입니다.
});
```

### 정리

| 함수 호출방식                                             | this 바인딩                                                           |
| --------------------------------------------------------- | --------------------------------------------------------------------- |
| 일반 함수 호출                                            | 전역 객체                                                             |
| 메서드 호출                                               | 메서드를 호출한 객체                                                  |
| 생성자 함수 호출                                          | 생성자 함수가 (미래에) 생성할 인스턴스                                |
| Function.prototype.apply/call/bind 메서드에 의한 간접호출 | Function.prototype.apply/call/bind 메서드에 첫번째 인수로 전달한 객체 |
