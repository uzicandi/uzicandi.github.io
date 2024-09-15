---
title: "01. [JS] 동등비교, 함수, 클래스, 클로저"
description:
date: 2024-07-29
update: 2024-07-29
tags:
  - React
series: "모던 리액트 Deep Dive"
---

```cs
💡 [TODO]
+ 자바스크립트에 존재하는 데이터 타입은 무엇인지
+ 이 데이터 타입은 어떻게 저장되며 이 값의 비교는 어떻게 수행되는지를 알아본다.
이를 통해,
+ 함수 컴포넌트에서 사용되는 훅의 의존성 배열의 비교
+ 렌더링 방지를 넘어선 useMemo와 useCallback의 필요성
+ 렌더링 최적화를 위해서 필요한 React.memo를 올바르게 작동시키기 위해 고려해야 할 것을 이해한다.
```

## 1.1 자바스크립트의 동등 비교

---

**의존성 배열(dependencies)**

- 보통은 리액트에서 제공하는 `eslint-react-config`에 선언돼 있는 `react-hooks/exhaustive-deps`의 도움을 받아 해당 배열을 채움
- 리액트 컴포넌트의 렌더링이 일어나는 이유 중 하나가 바로 **props의 동등 비교**에 따른 결과다.
- **props의 동등 비교**는 객체의 얕은 비교를 기반으로 이뤄진다.

**자바스크립트의 동등 비교를 기반으로 하는 작업**

- 가상 DOM과 실제 DOM의 비교
- 리액트 컴포넌트가 렌더링할지를 판단하는 방법
- 변수나 함수의 메모이제이션

### (1) 자바스크립트의 데이터 타입

- `원시 타입`: boolean, null, undefined, number, string, symbol, bigint
  - 객체가 아닌 모든 타입. 객체가 아니므로 메서드를 갖지 않는다.
- `객체 타입`: object
  - 참조를 전달한다고 해서 참조 타입(reference type)으로도 불린다.

### (2) 값을 저장하는 방식의 차이

원시타입과 객체타입의 가장 큰 차이점은 바로 `값을 저장하는 방식`이다.

- 원시타입
  - 불변 형태의 값으로 저장된다.
  - 변수 할당 시점에 메모리 영역을 차지하고 저장한다.
- 객체타입
  - 동일한 내용을 가지고 있지만 비교하면 다르게 나옴
  - 객체는 값을 저장하는게 아니라 참조를 저장한다.
  - 저장하는 순간 다른 참조를 바라보기 때문에 false를 반환하게 된다.
  - 즉, 객체간의 비교는 내부 값이 같더라도 결과는 대부분 true가 아닐 수 있다는 것을 인지

### (3) Object.is - 자바스크립트의 또 다른 비교 공식

- 인수 두 개가 동일한지 확인하고 반환하는 메서드
- `== vs Object.is` :
  - `==` 는 같음을 비교하기 전에 양쪽이 같은 타입이 아니면 비교할 수 있도록 강제로 형변환함
  - `Object.is` 는 `===` 와 동일하게 타입까지 체크
- `=== vs Object.is` : 이것도 차이가 있으며, Object.is 가 더 디테일하게 체크하여 개발자가 기대하는 방식
- `Object.is`를 사용한다 하더라도 객체 비교에는 별 차이가 없다. 객체 비교는 앞서 이야기한 **객체 비교 원리**와 동등함

### (4) 리액트에서의 동등 비교

리액트에서는 Object.is를 사용한다. 이는 ES6의 기능이기 때문에 폴리필을 함께 사용한다.

- Object.is를 기반으로 동등비교를 하는 `shallowEqual`이라는 함수를 만들어 사용한다.
- 리액트에서의 비교는 `Object.is`로 먼저 비교를 수행한 다음, `Object.is`에서 수행하지 못하는 비교, 즉 `객체 간 얕은 비교`를 **한 번 더 수행하는 것**을 알 수 있다.
- JSX props는 객체이고, 여기에 있는 props만 일차적으로 비교한다. 이를 기준으로 렌더링을 수행한다.

```cs
+ 객체비교의 불완전성은 자바스크립트만의 특징
+ 이러한 언어적 한계로 얕은 비교만을 이용해 비교를 수행하는 리액트
```

## 1.2 함수

---

### (1) 함수를 정의하는 4가지 방법

- **함수선언문**
  - `function`으로 선언
  - 변수 선언과 다르게 명시적으로 함수를 구별하고 싶을 때
  - 함수가 선언된 위치에 상관없이 어디서든 호출
- **함수 표현식**

  - `const, let`등 변수로 표현
  - 실제함수를 어디서 선언했는지 확인하고 싶을 때
  - 관리해야하는 스코프가 길어질 경우

- **Function**
  - 생성자 방식으로 함수를 만들면 함수의 클로저 또한 생성되지 않음
  - `const add = new Function('a', 'b', 'return a + b')`
- **화살표 함수**
  - ES6에서 추가된 생성 방식
  - 화살표 함수와 일반 함수의 가장 큰 차이점은 `this 바인딩`이다.

`this`란, **자신이 속한 객체나 자신이 생성할 인스턴스를 가리키는 값**이다.

- 함수가 일반함수로 호출된다면, 내부의 this는 `전역 객체`를 가리키게 된다.
- 이와 달리 화살표 함수는 함수 자체의 바인딩을 갖지 않는다.
  - **따라서 화살표 함수 내부에서 this를 참조하면 상위 스코프의 this를 그대로 따르게 된다.**

```cs
class Component extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      counter = 1,
    }
  }

  functionCountUp() {
    console.log(this) // undefined
    this.setState((prev) => ({ counter: prev.counter + 1}))
  }

  ArrowFunctionCountUp = () => {
    console.log(this) // class Component
    this.setState((prev) => ({ counter: prev.counter + 1}))
  }

  render() {
    return (
      <div>
        {/* Cannot read properties of undefined (reading 'setState') */}
        <button onClick={this.functionCountUp}>일반 함수</button>
        {/* 정상작동 */}
        <button onClick={this.ArrowFunctionCountUp}>일반 함수</button>
      </div>
    )
  }
}
```

### (2) 리액트에서 자주 쓰이는 형태의 함수

**즉시 실행 함수(IIFE)**

- 단 한 번만 호출되고, 다시금 호출할 수 없다. 따라서 이름을 붙이지 않는다.
- 글로벌 스코프를 오염시키지 않는 독립적인 함수 스코프를 운용할 수 있다는 장점

**고차함수(HOF)**

- 자바스크립트의 함수가 일급 객체라는 특징을 활용
- **함수를 인수로 받거나 결과로 새로운 함수를 반환**시킬 수 있다.

```cs
// 함수를 매개변수로 받는 대표적인 고차함수 (Array.prototype.map)
const doubleArray = [1,2,3].map((item) => item * 2)
doubleArray // [2,4,6]

// 함수를 반환하는 고차함수의 예
const add = function(a) {
  // a가 존재하는 클로저를 생성
  return function(b) {
    // b를 인수로 받는 또다른 함수 생성
    return a + b
  }
}
```

### (3) 함수를 만들 때 주의해야 할 사항

**함수의 side-effect를 최대한 억제하라**

- side-effect: 함수 내의 작동으로 인해 **함수가 아닌 함수 외부에 영향**을 끼치는 것.
- useEffect의 작동 최소화 등

**가능한 한 함수를 작게 만들어라**

- max-lines-per-function (ESLint)를 사용
- 함수는 하나의 일을 하나만 잘하면 된다. -> 재사용성 높이는 방법

**누구나 이해할 수 있는 이름을 붙여라**

## 1.3 클래스

---

- 자바스크립트의 프로토타입 기반으로 작동하는 클래스의 원리

```cs
+ 클래스 컴포넌트에 어떻게 생명주기를 구현할 수 있었는지
+ 왜 클래스 컴포넌트 생성을 위해 React.Component나 React.PureComponent를 상속하는지
+ 메서드가 화살표 함수와 일반 함수일 때 어떤 차이가 있는지를 이해할 수 있다.
```

### (1) 클래스란 무엇인가

- 특정한 객체를 만들기 위한 일종의 템플릿과 같은 개념

**constructor**

- 객체를 생성하는데 사용하는 특수한 메서드
- 하나만 존재할 수 있으며, 여러개를 사용하면 에러 발생함

**프로퍼티**

- 클래스로 인스턴스를 생성할 때 내부에 정의할 수 있는 속성 값

```cs
class Car {
  constructor(name) {
    // 값을 받으면 내부에 프로퍼티로 할당된다.
    this.name = name
  }
}

const myCar = new Car('자동차');
```

**getter, setter**

```cs
class Car {
  constructor(name) {
    this.name = name;
  }

  get firstCharacter() {
    return this.name[0]
  }

  set firstCharacter(char) {
    this.name = [char, ...this.name.slice(1)].join('')
  }
}

const myCar = new Car('자동차')

myCar.firstCharacter // 차
console.log(myCar.firstCharacter);
```

**인스턴스 메서드**

- 자바스크립트의 prototype에 선언되어 프로토타입 메서드로 불리기도 한다.

```cs
class Car {
  constructor(name) {
    this.name = name
  }

  // 인스턴스 메서드 정의
  hello() {
    console.log(`안녕하세요. ${this.name}입니다.`)
  }
}

const myCar = new Car('자동차');
myCar.hello(); // 인스턴스 메서드 선언

Object.getPrototypeOf(myCar); // {constructor: f, hello: f}
Object.getPrototypeOf(myCar) === Car.prototype // true
myCar.__proto__ === Car.prototype // true
```

- `프로토타입 체이닝`: 직접 객체에서 선언하지 않았음에도 **프로토타입에 있는 메서드를 찾아서 실행을 도와주는 것**
- 특정 속성을 찾을 때 자기 자신부터 시작해서 최상위 객체인 Object까지 훑는다.
- 이런 특성 덕분에 생성한 객체에서도 직접 선언하지 않은, 클래스에서 선언한 `hello()`메서드를 호출할 수 있고,
- 이 메서드 내부에서 `this`에 접근해 사용할 수 있게 된다.

**정적 메서드**

- 인스턴스를 생성하지 않고도 사용할 수 있다.
- 객체를 생성하지 않고도 여러곳에서 재사용이 가능하다.
- 유틸함수를 정적 메서드로 많이 활용하는 편이다.

```cs
class Car {
  static hello() {
    console.log('안녕하세요')
  }
}

const myCar = new Car()
myCar.hello() // TypeError
Car.hello() // 안녕하세요
```

**상속**

- `extends`로 상속받아서 자식 클래스에서 상위 클래스를 확장함

```cs
class Car {
  constructor(name) {
    this.name = name
  }

  hook() {
    console.log(`${this.name} 경적을 울립니다`)
  }
}

class Truck extends Car {
  constructor(name) {
    // 부모 클래스의 constructor, 즉 Car의 constructor를 호출한다.
    super(name)
  }

  load() {
    console.log('짐을 싣습니다.')
  }
}

const myCar = new Car('자동차')
myCar.honk() // 자동차 경적을 울립니다!

const truck = new Truck('트럭')
truck.honk() // 트럭 경적을 울립니다!
truck.load() // 짐을 싣습니다!
```

### (2) 클래스와 함수의 관계

- 클래스는 ES6에서 나온 개념, 이전에는 프로토타입을 활용해 클래스의 작동 방식을 동일하게 구현함
- 자바스크립트의 프로토타입을 활용.
- 객체지향 언어를 사용하던 다른 프로그래머 입장에서의 문법적 설탕

## 1.4 클로저

---

```cs
- 클래스 컴포넌트 - Class, Prototype, this
- 함수 컴포넌트 - Closure
  - 함수컴포넌트의 구조와 작동방식, 훅의 원리, 의존성 배열
```

**리액트에서의 클로저**

```cs
function Componet() {
  const [state, useState] = useState()

  function handleClick() {
    // useState 호출은 위에서 끝났지만
    // setState는 계속 내부의 최신값(prev)를 알고있다.
    // 이는 클로저를 활용했기 때문에 가능하다.
    setState((prev) => prev + 1)
  }
}
```

- 클로저는 생성될 때마다 그 선언적 환경을 기억해야 하므로 추가로 비용이 발생한다.
  - 클로저가 선언된 순간 내부함수는 외부 함수의 선언적인 환경을 기억하고 있어야 하므로
