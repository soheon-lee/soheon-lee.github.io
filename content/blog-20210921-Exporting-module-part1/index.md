---
emoji: 👑
title: Node.js & Module - PART1 (module.exports)
date: '2021-09-21 02:00:00'
author: 이워크
tags: node express react wecodefullstack 위코드풀스택 위코드 wecode module.exports export require import
categories: 개발블로그
---

Node.js에서 모듈을 내보내고(export) 불러와서(import) 사용하는 방법을 다루는 글입니다. 네파트로 나뉘어있습니다.

- PART1 - module.exports (feat. CommonJS)
- PART2 - export/import with .mjs (feat. ES6)
- PART3 - { type: module }
- PART4 - export/import with babel (feat. prisma client)

## 가장 처음 알아야 하는 방법 (module.exports)

Node.js 모듈 시스템에서는, 파일 한 개를 모듈 한 개로 생각한다. 따라서, 파일 하나를 모듈로 불러도 무방하다. '모듈이 파일이구나' 라고 생각하고 접근하도록 하자.

### 변수를 내보내고, 다른 모듈에서 불러오기

1. 테스트를 위해 `JS_module_tutorial` 이라는 디렉토리를 하나 생성했다. 내부에는 파일 두개를 만든다.

   - `ex1_greetings.js` : 인사말을 변수로 저장해둔 모듈
   - `ex2_sayHello.js` : 인사하는 함수를 선언하고 실행할 모듈

2. `ex1` 모듈에서 변수를 선언하고, 모듈 내부에서도 활용할 수 있도록 `module.exports`를 이용해 내보낸다.

   ```js
   // ./ex1_greetings.js
   const greeting = 'Hello! Nice to meet you, ';

   module.exports = greeting;
   ```

3. 선언한 변수를 `ex2` 모듈에서 불러와 사용하고자 한다. 불러온 인사말을 활용하여 함수 `sayHello`를 선언하고, 적당한 argument와 함께 호출한다.

   ```js
   // ./ex2_sayHello.js
   const greeting = require('./ex1_greetings');
   console.log('greeting: ', greeting);
   // "greeting: Hello! Nice to meet you, " 라는 텍스트가 나옴.

   const sayHello = (name) => {
     return greeting + name;
   };
   console.log(sayHello('이워크'));
   // 터미널에 "Hello! Nice to meet you, 이워크'
   ```

4. 터미널에서 `ex2` 파일을 실행시킨다.
   ```
   node ex2_sayHello.js
   ```

자 여기서 짚고 넘어가야 할 포인트는 사실 ex1_greeting파일을 불러올 때 호출 되는 대상이 `greeting` 이라는 변수 뿐이기 때문에, require 하는 파일 입장에서는 그 어떤 이름으로 불러와도 상관이 없다.

`ex2_sayHello.js` 파일에서 가장 첫 줄과 세번째 줄을 다음 코드와 같이 변경해도 정확하게 동일한 기능을 수행한다는 뜻이다.

```js
// ./ex2_sayHello.js

const sentence = require('./ex1_greetings');
const sayHello = (name) => {
  return sentence + name;
};

console.log(sayHello('이워크'));
```

그렇다면 `ex1_greetings.js` 모듈 내부에 변수와 함수가 여러개 있는 경우에는 어떨까?

```js
// ex1_greetings.js

const greeting = 'Hello! Nice to meet you, ';
const greeting2 = 'I am very happy to see you again, ';

module.exports = greeting;
module.exports = greeting2;
```

이렇게 할 수는 없다. (덮어씌워지나?)
한가지 방법은 아래와 같이 하나의 객체로 내보내는 것이다.

```js
module.exports = {
  greeting,
  greeting2,
};
```

이렇게 내보낸다면, 불러와서 사용하는 파일에서도 사용법이 조금 달라져야 한다. 그 전에 코드 수정하지 않고, 그대로 한 번 실행을 해보자.

```js
// ./ex2_sayHello.js

const sentence = require('./ex1_greetings');
console.log('sentence: ', sentence);

const sayHello = (name) => {
  return sentence + name;
};

console.log(sayHello('이워크'));
```

콘솔에 `sentence`를 찍어보니 객체 형태 그대로 불러온 것을 알 수 있다. 따라서, 선언된 문구를 그대로 사용하고 싶다면 객체의 key를 활용해야만 한다.

```js
// ./ex2_sayHello.js

const sentence = require('./ex1_greetings');
console.log('sentence: ', sentence);

const sayHello = (name) => {
  return sentence.greeting + name;
  // 또는
  return sentence.greeting2 + name;
};

console.log(sayHello('이워크'));
```

모듈을 불러올 때에도 **구조분해할당**을 할 수 있다.

```js
const { greeting, greeting2 } = require('./ex1_greetings');
console.log('GREETING: ', greeting);
console.log('GREETING2: ', greeting2);

const sayHello = (name) => {
  return greeting + name;
  // 또는
  return greeting2 + name;
};
```

단! 이때는 `greeting` 이나 `greeting2` 와 같은 이름이 무조건 동일하게 유지되어야 한다.

---

이것이 Node.js가 가장 기본적으로 사용하는 CommonJS 문법 사용법이다. 모듈을 생성하고, 변수 또는 함수를 내보내거나 (export) 불러올 때 (require) 사용한다.

`module.exports` 문법과 그냥 `export` 문법의 차이점에서는 다음 글에서 짚어보도록 하겠다.

```toc

```
