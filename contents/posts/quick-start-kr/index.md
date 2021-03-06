---
title: "π 2. λΉ λ₯΄κ² μμνκΈ°"
description:
date: 2021-04-02
update: 2021-04-02
tags:
  - hoodie
  - quick-start
series: "gatsby-starter-hoodie λ‘ λΈλ‘κ·Έ μμνκΈ°"
---

μλ λ¨κ³λ₯Ό λ°λΌμ μ¬λ¬λΆμ λΈλ‘κ·Έλ₯Ό μμνμΈμ. κ΅μ₯ν μ¬μμ π.

## 1. Gatsby μ¬μ΄νΈ μμ±

> μ»΄ν¨ν°μ **node.js** μ **gatsby-cli** κ° μ€μΉλμ΄ μμ΄μΌν©λλ€.

```
$ npx gatsby new my-hoodie-blog https://github.com/devHudi/gatsby-starter-hoodie
```

## 2. κ°λ° μλ² μμ

```
$ cd my-hoodie-blog
$ npm run start
```

μ΄μ  localhost:8000 μΌλ‘ μ¬λ¬λΆμ λΈλ‘κ·Έλ₯Ό μ μν  μ μμ΅λλ€.

## 3. Github λ ν¬μ§ν λ¦¬ μμ±

Utterance λκΈ μμ ―μ **Github μ΄μ μμ€ν** κΈ°λ°μλλ€. λ°λΌμ κ° λΈλ‘κ·Έ λ³ Github λ ν¬μ§ν λ¦¬κ° νμν©λλ€. λν μ¬λ¬λΆμ΄ Github Pages νΉμ Netlify λ‘ λΈλ‘κ·Έλ₯Ό λ°°ν¬νκΈΈ μνλ€λ©΄, Github λ ν¬μ§ν λ¦¬λ νμμλλ€.

λ§μ½ Github λ ν¬μ§ν λ¦¬λ₯Ό μμ±νλ λ²μ λͺ¨λ₯Έλ€λ©΄, [Github κ³΅μ λ¬Έμ](https://docs.github.com/en/github/getting-started-with-github/create-a-repo) λ₯Ό μ°Έμ‘°νμΈμ.

### μκ²© λ ν¬μ§ν λ¦¬ λ±λ‘

```
git remote add origin https://github.com/{YOUR_GITHUB_NAME}/{YOUR_REPOSITORY_NAME}
```

## 4. blog-config.js μμ±

```javascript
module.exports = {
  title: "MY BLOG",
  description: "Hello, This is my blog",
  author: "YOUR NAME",
  siteUrl: "https://myblog.com",
  links: {
    github: "https://github.com",
    facebook: "https://www.facebook.com",
    instagram: "https://www.instagram.com",
    etc: "https://www.google.com/",
  },
  utterances: {
    repo: "{YOUR_GITHUB_NAME}/{YOUR_REPOSITORY_NAME}",
    type: "pathname",
  },
}
```

gatsby-starter-hoodie λ `blog-config.js` λΌλ μ€μ  νμΌμ μ κ³΅ν©λλ€. μ΄ νμΌμμ λΈλ‘κ·Έ μ λ³΄, μμ±μ νλ‘ν, Utterance μ€μ  λ±μ μμ±ν  μ μμ΅λλ€. μ¬λ¬λΆ λΈλ‘κ·Έ μ€μ μ λ§κ² `blog-config.js` λ₯Ό μ€μ νμΈμ. νμ§λ§, `utterances.type` μμ±μ μμ νμ§ μλ κ²μ κΆμ₯ν©λλ€.

### νλ‘ν μ΄λ―Έμ§ λ³κ²½

`static/profile.png` μ μμΉν μ΄λ―Έμ§ νμΌμ μνλ μ΄λ―Έμ§ νμΌλ‘ κ΅μ²΄νμΈμ. λ§μ½ νμΌλͺμ λ³κ²½νκ³  μΆλ€λ©΄, `src/components/Bio.jsx` μ μμ€μ½λλ₯Ό μμ ν΄μΌν©λλ€.

## 5. ν¬μ€νΈ μΆκ°

λ§ν¬λ€μ΄ ν¬μ€νΈλ `contents/posts` κ²½λ‘μ μμΉν΄μμ΅λλ€. ν΄λΉ κ²½λ‘μμ κΈμ μμ±ν  μ μμ΅λλ€. [μ¬κΈ°λ₯Ό ν΄λ¦­νμ¬](https://devHudi.github.io/gatsby-starter-hoodie/writing-guide) λ μμΈν κΈ μμ± λ°©λ²μ νμΈνμΈμ.

## 6. λΈλ‘κ·Έ λ°°ν¬νκΈ°

### 6-1 Netlify λ₯Ό ν΅ν΄

[A Step-by-Step Guide: Gatsby on Netlify](https://www.netlify.com/blog/2016/02/24/a-step-by-step-guide-gatsby-on-netlify/) λ¬Έμλ₯Ό μ°Έμ‘°νμ¬, Netlify λ₯Ό Github λ ν¬μ§ν λ¦¬μ μ°κ²°ν  μ μμ΅λλ€. μ΄ κ³Όμ μ μ΄λ ΅μ§ μμ΅λλ€.

Github λ ν¬μ§ν λ¦¬μ μ°κ²°μ΄ λμλ€λ©΄, Github λ ν¬μ§ν λ¦¬μ λ³κ²½μ¬ν­μ΄ λ°μν  λ λ§λ€ μλμΌλ‘ μ¬λ¬λΆμ λΈλ‘κ·Έμ λ°°ν¬λ©λλ€.

### 6-2. Github Pages λ₯Ό ν΅ν΄

#### μν© 1

λ ν¬μ§ν λ¦¬ μ΄λ¦μ΄ `{YOUR_GITHUB_NAME}.github.io` ννμΌ κ²½μ°, μλ λͺλ Ήμ΄λ₯Ό μ€νν΄μ£ΌμΈμ.

```
$ npm run deploy-gh
```

#### μν© 2

λ ν¬μ§ν λ¦¬ μ΄λ¦μ΄ `{YOUR_GITHUB_NAME}.github.io` ννκ° μλ κ²½μ°, μλ λͺλ Ήμ΄λ₯Ό μ€νν΄μ£ΌμΈμ.

```
$ npm run deploy-gh-prefix-paths
```

λ§μ½ μμ κ°μ κ²½μ° `gatsby-config.js` μμ `pathPrefix` λ₯Ό μ¬λ¬λΆμ λ ν¬μ§ν λ¦¬ μ΄λ¦μΌλ‘ λ°κΏμΌν©λλ€.

### 6-3. λ€λ₯Έ νλ«νΌ

```
$ npm run build
```

μ λͺλ Ήμ΄λ‘ Gastby μΉμ¬μ΄νΈλ₯Ό λΉλν  μ μμ΅λλ€. λΉλ κ²°κ³Όλ¬Όμ `/public` μ μ μ₯λ©λλ€. `/public` λλ ν λ¦¬λ₯Ό μ¬λ¬λΆμ΄ μ¬μ©νλ νλ«νΌμ λ°°ν¬ λͺλ Ήμ ν΅ν΄ λ°°ν¬ν΄μ£ΌμΈμ.

## 7. μ»€μ€ν°λ§μ΄μ§

### νλ‘μ νΈ κ΅¬μ‘°

μλ νλ‘μ νΈ κ΅¬μ‘°λ₯Ό μ°Έκ³ νμ¬ μ»€μ€ν°λ§μ΄μ§ ν  μ μμ΅λλ€ π.

```
βββ node_modules
βββ contents
βΒ Β  βββ posts // your articles are here
βββ public // build outputs are here
βββ src
    βββ assets
    βΒ Β  βββ theme // theme config is here
    βββ components
    βΒ Β  βββ Article
    β    Β Β  βββ Body
    β        Β Β  βββ StyledMarkdown
    β            Β Β  βββ index.jsx // markdown styles are here
    β   ...
    βββ fonts // webfonts are here
    βββ hooks
    βββ images
    βββ pages // page components are here
    βββ reducers
    βββ templates // post components are here
    βββ utils
```
