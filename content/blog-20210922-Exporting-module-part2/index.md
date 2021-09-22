---
emoji: 👑
title: Node.js & Module - PART2 (import/export .mjs file)
date: '2021-09-22 19:00:00'
author: 이워크
tags: node express react wecodefullstack 위코드풀스택 위코드 wecode module.exports export require import
categories: 개발블로그
---

> Node.js에서 모듈을 내보내고(export) 불러와서(import) 사용하는 방법을 다루는 글입니다. 네파트로 나뉘어있습니다.

- PART1 - module.exports (feat. CommonJS)
- PART2 - export/import with .mjs (feat. ES6)
- PART3 - { type: module }
- PART4 - export/import with babel (feat. prisma client)

## ES Modules - (import/export)

PART1에서 사용했던 문법을 import/export 문법으로 변경해보자. 파일 구성은 아래와 같다.

- ex1_greetings.js : 인사말을 변수로 저장해둔 모듈
  ```js
  // ./ex1_greetings.js
  const greeting = 'Hello! Nice to meet you, ';
  module.exports = greeting;
  ```
- ex2_sayHello.js : 인사하는 함수를 선언하고 실행할 모듈

  ```js
  // ./ex2_sayHello.js

  const greeting = require('./ex1_greetings');
  const sayHello = (name) => {
    return greeting + name;
  };

  console.log(sayHello('이워크'));
  ```

### require 대신 import 문법으로 가져오기

`ex2` 모듈에 적용하고 싶은 ES 모듈 import 문법은 아래와 같다.

```js
// ./ex2_sayHello.js

import greeting from './ex1_greetings';

const sayHello = (name) => {
  return greeting + name;
};

console.log(sayHello('이워크'));
```

이대로 실행해보면, 다음과 같은 에러를 만난다.

```js
(node:51915) Warning: To load an ES module, set "type": "module" in the package.json or use the .mjs extension.
(Use `node --trace-warnings ...` to show where the warning was created)
/Users/soheonlee/Development/JS_module_tutorial/ex2_sayHello.js:2
import greeting from "./ex1_greeting.js";
^^^^^^

SyntaxError: Cannot use import statement outside a module
```

```toc

```
