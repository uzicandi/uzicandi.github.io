### Babel

프론트엔드 코드가 브라우저에 맞게 구현되다 보니까 코드가 일관적이지 않을때가 많다.

- 작년까지만 해도 사파리에서는 `Promise.prototype.finally` 메소드를 사용할 수 없었다.
- 이것에 대한 **폴리필** 코드를 추가하는 것으로 해결
- 프론트엔드 개발에서 크로스 브라우징 이슈는 프론트엔드 개발할때 처음 입문하는 사람한테는 프론트엔드 개발
- 크로스 브라우징으로 인한 문제를 해결하기 위해 바벨이 등장
- ECMA2015+ 로 작동하는 것
  - Ts, JSX처럼 다른 언어로 동작하는 것도 동작하도록 호환성 지켜줌

### 설치 및 기본 동작

- `@babel/core`, `@babel/cli` 설치
- babel로 변환시키기 위한 코드

```
app.js

const alert = msg => window.alert(msg);
// ie 같은 경우에는 인식하지 못함
```

- 바벨을 설치하면 `node_modules/bin`에 바벨 폴더가 생김
- `npx babel app.js`

코드가 이전과 이후의 코드가 똑같다.

바벨은 세 단계로 빌드를 진행한다.

```
바벨이 코드를 변환하는 작업 = 빌드
1. 파싱: 코드를 받아서 토큰별로 분해한다. 그 과정을 파싱이라고 함.
2. 변환: es6 -> es5
3. 출력
```

코드를 읽고 추상구문트리(AST)로 변환하는 단계를 "파싱"이라고 한다.
이것은 빌드작업을 처리하기에 적합한 자료구조인데 컴파일러 이론에 사용되는 개념이다.
추상구문트리를 변경하는 것이 "변환" 단계이다. 실제로 코드를 변경하는 작업을 한다.
변경된 결과물을 "출력"하는 것을 마지막으로 바벨은 작업을 완료한다.

`const babel = code => code`

바벨은 파싱과 출력만 담당하고 변환작업은 다른 녀석이 처리하되 이것을 "플러그인" 이라 부른다.

### 3.1 커스텀 플러그인

결과를 보면 파싱한 것 같고, 변환은 하지 않았음

- 바벨은 Plugin이라는 요소가 있는데, 변환을 담당하는 녀석임
- 웹팩에서 커스텀으로 로더도 만들고, 플러그인도 만들었음
- 바벨도 커스텀 플러그인을 만들 수 있음

```cs
// my-babel-plugin
// 커스텀 플러그인을 만들때는 visitor 객체를 반환해줘야 한다.
// 이 visitor 객체는 바벨이 파싱하여 만든 추상구문트리(AST)에 접근할 수 있는 메소드를 제공한다.
module.exports = function myBabelPlugin() {
  return {
    visitor: {
      Identifier(path) {
        const name = path.node.name; // 어떤 파싱된 결과물에 접근할 수 있음

        // 바벨이 만든 AST 노드를 출력한다.
        console.log('Identifier() name:', name)

        // 변환작업: 코드 문자열을 역순으로 반환한다.
        path.node.name = name
          .split("")
          .reverse()
          .join("");
      }
    }
  }
}
```

- `babel --help` 문서를 보면서 어떻게 플러그인을 쓰는지 봐보겠음
- `--plugin [list]` 옵션을 줘서 바벨로 실행
- `npx babel app.js --plugins './my-babel-plugin`을 전달

Identifier라는 메소드에 정의한 콘솔로그가 찍힘.

```cs
// es6 -> es5 로 변환하는 객체로 변환해보기
module.exports = function myBabelPlugin() {
  return {
    visitor: {
      VariableDeclaration(path) { // 파싱된 결과를 path라는 객체로 받음
        console.log('VariableDeclaration() kind', path.node.kind);
        if(path.node.kind === 'const') {
          path.node.kind = 'var'
        }
      }
    }
  }
}
```

### 플러그인 사용하기

```cs
// @babel/plugin-transform-block-scoping

- npm install @babel/plugin-transform-block-scoping
- npx babel app.js --plugins @babel/plugin-transform-block-scoping
```

브라우저에서 `use strict`로 해서 엄격모드로 실행하는 것이 좋음.

- babel.config.js 설정파일을 제공하고, 설정파일로 옮겨보면

```
module.exports = {
  plugins: []
}

// npx babel app.js
```

- 플러그인을 적용하고 코드를 변환한 다음에 출력

### 커스텀 프리셋

```cs
// 목적에 맞게 여러가지 플러그인을 세트로 모아놓은 것을 프리셋이라고 함
// my-babel-preset.js

module.exports = function myBablePreset() {
  return {
    plugins: [

    ]
  }
}
```

```cs
// babel.config.js

module.exports = {
  presets: [
    './my-babel-presets.js'
  ]
}
```

### 5. 폴리필

- 바벨은 ES6 -> ES5로 변환할 수 있는 것만 빌드한다.
- 그렇지 못한 것들은 "폴리필" 이라고 부르는 코드 조각을 추가해서 해결한다.

env 프리셋은 폴리필을 지정할 수 있는 옵션을 제공한다.

```cs
// babel.config.js:
module.exports = {
  presets: [
    [
      "@babel/preset-env",
      {
        useBuiltIns: "usage", // 폴리필 사용 방식 지정
        corejs: {
          // 폴리필 버전 지정
          version: 2,
        },
      },
    ],
  ],
}
```

### 6. 웹팩으로 통합

- 실무환경에서는 바벨을 직접 사용하는 것보다는 웹팩으로 통합해서 사용하는 것이 일반적이다.
- 로더형태로 제공하는데 babel-loader가 그것이다.

```cs
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader", // 바벨 로더를 추가한다
      },
    ],
  },
}
```

### 7. 정리

- 바벨은 일관적인 방식으로 코딩하면서, 다양한 브라우저에서 돌아가는 어플리케이션을 만들기위한 도구다.
- 바벨의 코어는 파싱과 출력을 담당하고 변환작업은 플러그인이 처리한다.
- 여러개의 플러그인들을 모아놓은 세트를 프리셋이라고 하는데 ES6 환경은 env프리셋을 사용한다.
- 바벨이 변환하지 못하는 코드는 폴리필이라 부르는 코드조각을 불러와 결과물에 로딩해서 해결
- babel-loader로 웹팩과 함께 사용하면 훨씬 단순하고 자동화된 프론트엔드 개발환경
