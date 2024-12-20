Lint는 왜 필요한가?

### 1.1 린트가 필요한 상황

```cs
console.log()
(function() {

})()
```

### ESLint

- 포매팅: 코드를 일관되게
- 코드 품질

```cs
npm i eslint
```

- `.eslintrc.js`를 만들면 es-lint 설정파일을 읽고 린트 실행
- `rules`가 규칙을 만들어둠

```cs
module.exports = {
  rules: {
    'no-unexpected-multiline': 'error'
  }
}
```

unexpected한 줄바꿈이 있다고 함.
