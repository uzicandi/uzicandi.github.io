---
title: '2.1 JSX가 트랜스파일을 거쳐 자바스크립트 코드로 변환되는 과정'
description: ''
date: 2024-09-01
update: 2024-09-01
tags:
  - React
series: '모던 리액트 Deep Dive'
---

## 1. JSX란?

---

- 리액트가 등장하면서 페이스북에서 임의로 만든 문법이나 리액트에서만 사용하는 것은 아니다
- JSX가 포함된 코드를 아무런 처리없이 그대로 실행하면 에러가 난다.
- JSX는 반드시 트랜스파일러를 거쳐야 비로소 자바스크립트 런타임이 이해할 수 있는 의미 있는 자바스크립트 코드로 변환된다.
- 설계 목적 : 다양한 트랜스파일러에서 다양한 속성을 가진 트리구조를 토큰화해 ECMAScript로 변환하는 데 초점을 두고 있다.
  - 즉, JSX 내부에 트리 구조로 표현하고 싶은 다양한 것들을 작성해 두고, 이 JSX를 트랜스파일이라는 과정을 거쳐 자바스크립트가 이해할 수 있는 코드로 변경하는 것이 목표

## 2. JSX는 어떻게 자바스크립트에서 변환될까?

---

> JSX가 트랜스파일을 거쳐 자바스크립트 코드로 변환되는 과정

자바스크립트에서 JSX가 변환되는 방식을 알려면

- 리액트에서 JSX를 변환하는 `@babel/plugin-transform-react-jsx` 플러그인을 알아야 한다.
- 이 플러그인은 JSX 구문을 자바스크립트가 이해할 수 있는 형태로 변환한다.

```cs
import * as Babel from '@babel/standalone'

Babel.registerPlugin(
  '@babel/plugin-transform-react-jsx',
  require('@babel/plugin-transform-react-jsx'),
)

const BABEL_CONFIG = {
  presets: [],
  plugins: [
    [
      '@babel/plugin-transform-react-jsx',
      {
        throwIfNamespace: false,
        runtime: 'automatic',
        importSource: 'custom-jsx-library'
      }
    ]
  ]
}

const SOURCE_CODE = `const ComponentA = <A>안녕하세요</A>`
// code 변수에 트랜스파일된 결과가 담긴다.
const { code } = Babel.transform(SOURCE_CODE, BABEL_CONFIG)
```

- JSXElement를 첫 번째 인수로 선언해 요소를 정리한다.
- 옵셔널인 JSXChildren, JSXAttributes, JSXStrings는 이후 인수로 넘겨주어 처리한다.

이 점을 활요하면 JSXElement를 렌더링해야 할 때 굳이 요소 전체를 감싸지 않더라도 처리할 수 있다.

- 즉, JSXElement만 다르고, 나머지는 동일한 상황에서 중복코드를 최소화 할 수 있다.

```cs
import { createElement, PropsWithChildren } from 'react'

// props 여부에 따라 children 요소만 달라지는 경우
// 굳이 번거롭게 전체 내용을 삼항 연산자로 처리할 필요가 없다.
// 이 경우 불필요한 코드 중복이 일어난다.

function TextOrHeading({ isHeading, children }: PropsWithChildren<{ isHeading: boolean }>) {
  return isHeading ? (
    <h1 className="text">{children}</h1>
  ) : (
    <span className="text">{children}</span>
  )
}

// JSX가 변환되는 특성을 활용하면 다음과 같이 간결하게 처리할 수 있다.
// JSX 반환값은 결국 React.createElement로 귀결된다는 사실을 알면 이렇게 리팩터링 가능

import { createElement } from 'react'

function TextOrHeading({ isHeading, children }: PropsWithChildren<{ isHeading: boolean }>) {
  return createElement(
    isHeading ? 'h1' : 'span',
    { className: 'text' },
    children
  )
}
```

#### 실제로 사용했었던 곳

```cs
import type { FC, PropsWithChildren } from 'react';
import * as React from 'react';
import { Fragment, createElement, useMemo } from 'react';
import { transform } from '@babel/standalone';
import type { SafeFunction } from '@class101/utils-type';

type JsxRendererProps = {
  source: string;
  functions: Record<string, SafeFunction>;
};

export const JsxRenderer: FC<PropsWithChildren<JsxRendererProps>> = ({ source, functions: functionsProp }) => {
  const functions = {
    jsx: createElement,
    fragment: Fragment,
    ...functionsProp,
  };

  const code = useMemo(
    () =>
      transform(`const Block = ${source}`, {
        presets: [
          [
            'react',
            {
              pragma: 'jsx',
              pragmaFrag: 'fragment',
            },
          ],
        ],
      }).code,
    [source]
  );

  // eslint-disable-next-line no-new-func
  const fn = new Function('React', ...Object.keys(functions), `${code}; return Block`);

  return <div id="document-root">{fn(React, ...Object.values(functions))}</div>;
};
```
