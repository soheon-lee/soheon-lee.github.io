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

## Common JS - (module.exports & require)

가장 처음 알아야 하는 방법이다.

Node.js 모듈 시스템에서는, 파일 한 개를 모듈 한 개로 생각한다. 따라서, 파일 하나를 모듈로 불러도 무방하다. '모듈이 파일이구나' 라고 생각하고 접근하도록 하자.

### 변수를 내보내고(export), 다른 모듈에서 불러오기(require)

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

   const sayHello = (name) => {
     return greeting + name;
   };
   console.log(sayHello('이워크'));
   ```

4. 터미널에서 `ex2` 파일을 실행시킨다.

   실행

   ```sh
   node ex2_sayHello.js
   ```

   결과

   ```sh
   greeting: Hello! Nice to meet you,
   Hello! Nice to meet you, 이워크
   ```

우리의 예상 그대로, `ex1` 모듈에서 내보낸 `greeting` 변수를 아무 무리 없이 `ex2` 모듈에서 사용할 수 있다.

`require()` 문법은 기본적으로 자바스크립트 파일을 읽고, 해당 파일을 한 번 실행시킨 후, `export` 객체를 반환하는 과정을 진행한다. 위 예시에서 반환된 대상은 `greeting`이라는 변수이다. 여기서 한가지 짚고 넘어가야 할 점이 있다. require 하는 파일 입장에서는 불러오는 모듈 `ex1_greeting`을 그 어떤 이름으로 불러와도 상관이 없다.

`ex1` 모듈을 sentence라는 이름으로 불러와보자. `ex2_sayHello.js` 파일에서 가장 첫 줄과 세번째 줄을 다음 코드와 같이 변경해도 정확하게 동일한 기능을 수행한다.

```js
// ./ex2_sayHello.js

const sentence = require('./ex1_greetings');
const sayHello = (name) => {
  return sentence + name;
};

console.log(sayHello('이워크'));
```

`ex1` 모듈에서 export 하는 대상이 정해져있고, `require()` 메소드는 반한된 export객체를 받기 때문에, 받는 변수의 이름은 상관이 없다. 납득하기 어렵지 않다.

### 한 모듈에서 여러 객체 내보내기

그렇다면 `ex1_greetings.js` 모듈 내부에 변수 또는 함수가 여러개인 경우에는 어떨까? 아래처럼 `module.exports`를 여러번 사용하면 될까?

```js
// ex1_greetings.js

const greeting = 'Hello! Nice to meet you, ';
const greeting2 = 'I am very happy to see you again, ';

module.exports = greeting;
module.exports = greeting2;
```

이렇게 할 수는 없다. `module.exports`라는 객체에 재할당 되었으므로, 덮어씌워졌다고 할 수 있다. `greeting2`만 사용할 수 있다.
한가지

아래와 같이 하나의 객체로 내보내야한다.

```js
module.exports = {
  greeting,
  greeting2,
};
```

`ex1` 모듈에서 이렇게 내보낸다면, 불러와서 사용하는 `ex2`파일에서도 사용법이 달라져야 한다. 우선 `ex2` 코드를 수정하지 않고, 이전 코드 그대로 한 번 실행해보자.

```js
// ./ex2_sayHello.js

const sentence = require('./ex1_greetings');
console.log('sentence: ', sentence);

const sayHello = (name) => {
  return sentence + name;
};

console.log(sayHello('이워크'));
```

실행

```sh
node ex2_sayHello.js
```

결과

```js
greeting: {
  greeting: 'Hello! Nice to meet you, ',
  greeting2: 'I am very happy to see you again, '
}
[object Object]이워크
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

사용하기에 어렵지는 않다. 매번 `sentence`라는 객체를 언급하기가 번거롭다면, 모듈을 불러올 때에도 **구조분해할당**을 활용하기도 한다. 단, 이때는 `greeting` 이나 `greeting2` 와 같은 이름이 동일하게 유지되어야 한다. 납득하기 어렵지 않다.

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

이 방법은 편하지만, 간혹 서로 다른 모듈에서 같은 이름의 변수와 함수가 있을 수 있기에 때에 따라 보다 정확한 변수 사용을 위해서는 require한 모듈의 이름을 정확하게 명시해주는 것이 좋을 때도 있다. _진리의 케바케_

## 그 외

### require은 일단 파일을 한 번 실행시킨다

위에서 한 번 언급했는데, require() 함수는 파일을 불러오면서 기본적으로 한 번 실행한다. 따라서 `ex1` 모듈에

```js
console.log('안녕, 나는 ex1 파일이야');
```

라는 코드가 있다면 `ex2` 모듈에서 require하자마자 콘솔로그가 실행된다.

---

여기까지, Node.js가 가장 기본적으로 사용하는 CommonJS 문법 사용법이다. 모듈을 생성하고, 변수 또는 함수를 내보내거나 (export) 불러올 때 (require) 사용한다.

`module.exports` 문법과 그냥 `export` 문법의 차이점에서는 다음 글에서 짚어보도록 하겠다.

```toc

```
