### 1. 웹팩 엔트리/아웃풋 실습

https://jeonghwan-kim.github.io/series/2019/12/09/frontend-dev-env-npm.html

자동차 - 자동주행 차량
주행방향을 오른쪽에서 왼쪽으로 바꿔야된다

개발을 빨리 시작할 수 있음
개발 세팅을 변경해야된다면?
CRA, View CLI 가 업데이트 될 때까지 기다릴 수도.

엔지니어라면 사용하는 도구를 자유자재로 쓰는것도 중요
대표적인 도구.

### 1. 배경

- 모듈: 문법 수준에서 모듈을 지원하기 시작한 것은 ES2015부터다
- import/export 구문이 없었던 모듈 이전 상황을 살펴보는 것이 웹팩 등장 배경을 설명

```cs
// math.js
function sum(a,b) {
  return a + b
} // 전역 공간에 sum이 노출
```

```cs
// app.js
sum(1,2) // 3
```

- 이 코드의 문제점: 하나의 HTML 파일 안에서 로딩해야만 실행된다.
- math.js가 로딩되면 app.js는 이름 공간에서 'sum'을 찾은 뒤 이 함수를 실행한다.
- 문제는 'sum'이 전역 공간에 노출된다는 것. 다른 파일에서도 'sum'이란 이름을 사용하면 충돌한다.

### 1.1 IIFE 방식의 모듈

이러한 문제를 예방하기 위해 `스코프`를 사용한다. 함수 스코프를 만들어 외부에서 안으로 접근하지 못하도록 공간을 격리하는 것이다. 스코프 안에서는 자신만의 이름 공간이 존재하므로 스코프 외부와 이름 충돌을 막을 수 있다.

```cs
// math.js

var math = math || {} // math 네임스페이스

;(function () {
  function sum(a, b) {
    return a + b
  }

  math.sum = sum // 네임 스페이스에 추가
})

```

### 1.2 다양한 모듈 스펙

이런 방식으로 자바스크립트 모듈을 구현하는 대표적인 명세가 AMD와 CommonJS다.

CommonJS는 자바스크립트를 사용하는 모든 환경에서 모듈을 하는 것이 목표다.
exports 키워드로 모듈을 만들고 require()함수로 불러 들이는 방식이다.
대표적으로 서버 사이드 플랫폼인 Node.js에서 이를 사용한다.

```cs
// math.js

exports function sum(a,b) { return a + b }
```

```cs
// app.js
const math = require("./math.js")
math.sum(1,2) // 3
```

- AMD는 비동기로 로딩되는 환경에서 모듈을 사용하는 것이 목표다. 주로 브라우저 환경이다.
- UMD는 AMD 기반으로 CommonJS 방식까지 지원하는 통합형태다.

이렇게 각 커뮤니티에서 각자의 스펙을 제안하다가 ES2015에서 표준 모듈시스템을 내놓았다.
지금은 바벨과 웹팩을 이용해 모듈시스템을 사용하는 것이 일반적이다.
ES2015 모듈 시스템은 export 구문으로 모듈을 만들고, import 구문으로 가져올 수 있다.

### 1.3 브라우저의 모듈 지원

모든 브라우저에서 모듈을 사용하지는 않는다.
-> 브라우저에 무관하게 모듈을 사용하고 싶을때... Webpack!

```cs
// index.html

<script type="module" src="app.js"></script>
```

### 2. 엔트리/아웃풋

`웹팩`은 여러개 파일을 하나의 파일로 합쳐주는 번들러(bundler)다.
하나의 시작점으로 부터 의존적인 모듈을 전부 찾아내서 하나의 결과물을 만들어낸다.
app.js 부터 시작해 math.js 파일을 찾은 뒤 하나의 파일로 만드는 방식이다.

간단히 웹팩으로 번들링 작업을 해보자.
번들 작업을 하는 webpack 패키지와 웹팩 터미널 도구인 webpack-cli를 설치한다.
설치 완료하면 node_modules/.bin 폴더에 실행가능한 명령어가 몇 개 생긴다.
webpack과 webpack-cli가 있는데 둘 중 하나를 실행하면 된다.
`--help`옵션으로 사용방법을 확인해보자.

- `--mode`, `--entry`, `--output` 세 개 옵션만 사용하면 코드를 묶을 수 있다.

- --mode는 웹팩 실행 모드는 의미하는데 개발 버전인 development를 지정한다
- --entry는 시작점 경로를 지정하는 옵션이다
- --output은 번들링 결과물을 위치할 경로다

위 명령어를 실행하면 dist/main.js에 번들된 결과가 저장된다.

- 이 옵션은 웹팩 설정파일의 경로를 지정할 수 있는데 기본 파일명이 webpack.config.js 혹은 webpackfile.js다. `webpack.config.js` 파일을 만들어 방금 터미널에서 사용한 옵션을 코드로 구성해보자.

### 3. 로더

- 웹팩은 **모든 파일을 모듈로 바라본다**. 자바스크립트로 만든 모듈 뿐만 아니라 **스타일시트, 이미지, 폰트까지도 전부 모듈로 보기 때문에 import 구문을 사용하면 자바스크립트 코드 안으로 가져올 수 있다**.
- 이것이 가능한 이유는 웹팩의 로더 덕분이다.
- 로더는 타입스크립트 같은 다른 언어를 자바스크립트 문법으로 변환해주거나 이미지를 **data URL**형식의 문자열로 변환한다. 뿐만아니라 CSS 파일을 **자바스크립트에서 직접 로딩**할 수 있도록 해준다.

### 3.2 커스텀 로더 만들기

- 로더가 동작하는지 확인하는 용도로 로그만 찍고 곧장 content를 돌려 준다.
- 로더를 사용하려면 웹팩 설정파일의 module 객체에 추가한다.

```cs
// myloader.js
module.exports = function myloader(content) {
  console.log("myloader가 동작함")
  return content
}
```

```cs
// webpack.config.js:
module: {
  rules: [{
    test: /\.js$/, // .js 확장자로 끝나는 모든 파일
    use: [path.resolve('./myloader.js')] // 방금 만든 로더를 적용한다
  }],
}
```

- module.rules 배열에 모듈을 추가하는데 test와 use로 구성된 객체를 전달한다.
- test에는 로딩을 적용할 파일을 지정한다. 파일명 뿐만아니라 파일 패턴을 정규표현식으로 지정할 수 있는데, 위 코드는 .js확장자를 갖는 모든 파일을 처리하겠다는 의미다.
- use에는 이 패턴에 해당하는 파일에 적용할 로더를 설정하는 부분이다. 방금만든 myloader 함수의 경로를 지정한다.

### 4. 자주 사용하는 로더

- 로더의 동작원리를 살펴보았으니 이번에는 몇몇 자주 사용하는 로더를 소개하겠다.

### 4.1 css-loader

웹팩은 모든 것을 모듈로 바라보기 때문에 자바스크립트 뿐만 아니라 스타일 시트로 import 구문으로 불러올 수 있다.

```cs
// app.js

import "./style.css"
```

`$ npm install -D css-loader`

```cs
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/, // .css 확장자로 끝나는 모든 파일
        use: ["css-loader"], // css-loader를 적용한다
      },
    ],
  },
}
```

### 4.2 style-loader

모듈로 변경된 스타일시트는 돔에 추가되어야만 브라우져가 해석할 수 있다. css-loader로 처리하면 자바스크립트 코드로만 변경되었을 뿐 돔에 적용되지 않았기 때문에 스타일이 적용되지 않았다.

`style-loader`는 자바스크립트로 변경된 스타일을 동적으로 돔에 추가하는 로더이다.
CSS를 번들링 하기 위해서는 css-loader와 style-loader를 함께 사용한다.

npm - nodejs : 개발환경 전반에서

- npm이 프로젝트를 관리하는 방식

webpack:

- 핵심개념
- 로더

babel

- 동작 원리
- 최신 ecma script로

babel을 웹팩으로 통합하는 방법

eslint: 협업할 때 꼭 써야 하는 도구

prettier:

webpack 심화

- api 서버 연동,
- 핫 로딩 설정
- 번들 결과 최적화

---

웹팩이 필요한 이유?

- JS의 모듈
  - import, export 키워드를 지원했었는데
  - webpack이 등장하기 이전에는 어떤식으로 모듈을 만들어 썼는지?
- 배경
  - script를 호출할 때 전역 스코프가 오염된다는 단점이 있음

```ts
sum(1, 2)
```

- 전역스코프가 오염된다는 문제가 생김
- math.js의 sum이라는 함수는 이 math 모듈 안에서만 유효한 것이 아니라
  이 어플리케이션이 돌아가는 어느곳에서나 sum 함수를 접근할 수 있음
- 어플리케이션이 예측할 수 없게되고 런타임 에러를 발생하게 됨

```
window.sum
으로 호출하면 확인할 수 있음
```

- IIFE: 즉시 실행 함수 표현
- 이 함수를 실행하자 마자 정의하는 것.
- 함수 외부에서 접근할 수 없기 때문에 전역 스코프가 오염되는 문제를 예방할 수 있음

```
(function () {
  statements
})();
```

```
var math = math || {};

(function() {
  function sum(a,b) {
    return a+b;
  }

  math.sum = sum;
})()

```

math 객체를 통해서 sum에 접근할 수 있음.

---

### 1.2 다양한 모듈 스펙

이러한 방식으로 JS 모듈을 구현하는 대표적인 명세가 AMD와 CommonJS다.

CommonJS는 자바스크립트를 사용하는 모든 환경에서 모듈을 하는 것이 목표다

```
// math.js
exports function sum(a,b) {return a+b};
```

```
const sum = require('./math.js');
sum(1,2);
```

- 자바스크립트 모듈을 구현하는 대표적인 명세: AMD, CommonJS
- CommonJS: 자바스크립트를 사용하는 모든 환경에서 모듈을 하는 것이 목표.
  exports 키워드로 모듈을 만들고 require() 함수로 불러들이는 방식.
  대표적으로 서버사이드 플랫폼인 Node.js에서 이를 사용한다.

CommonJS는 자바스크립트를 사용하는 모든 환경에서 모듈을 하는 것이 목표

- require() 함수로 불러 들이는 방식
- exports 키워드로

- AMD:
- UMD: AMD 기반으로 CommonJS 방식까지 지원하는 통합 형태

바벨이나 웹팩을 이용해서 표준 모듈시스템을 사용하는게 기본

### 1.3 모듈시스템

- `<script type="module" />` 이라고 하면 모듈 시스템 쓰는것
- 웹팩이 이런 모듈시스템을 어떻게 처리하는지
