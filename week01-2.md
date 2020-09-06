## 우아한 테크러닝 1주차 - 2 💻

### JavaScript 변수

```js
var a;
a = 10;

let b = 10;
const c = 10;

function foo() {}
let funcFoo = foo;
```
var, let, const 키워드로 변수를 선언할 수 있고 선언과 할당을 동시에 할 수도 있다. 타입은 원시 타입과 객체 타입(참조 타입?)으로 나뉘는데 이 부분은 간단하기에 생략.

---

### JavaScript 함수
- 함수 선언문(Function Declaration)
```js
function foo() {
  // logic
}
```

- 함수 표현식(Function Expression)
```js
const func = function() {
  // logic
};
```

- IIFE(Immediately Invoked Function Expression, 즉시 실행되는 함수 표현식)
```js
(function () {})();
```

reset 역할과 같이 한 번만 실행되고 메모리에 저장될 필요 없는 함수의 경우는 익명으로 선언과 동시에 실행되고 사라지게 사용하곤 한다.

> JavaScript 는 `일급 객체`이며 일급 객체는 아래와 같은 특징을 지니고 있다고 한다.
> - 모든 일급 객체는 함수의 실질적인 매개변수가 될 수 있다.
> - 모든 일급 객체는 함수의 반환값이 될 수 있다.
> - 모든 일급 객체는 할당의 대상이 될 수 있다.
> - 모든 일급 객체는 비교 연산(==, equal)을 적용할 수 있다.

- 화살표 함수(Arrow Function, 람다식 한 줄 함수)
```js
const foo = x => {
  // logic
};
const bar = (x, y) => x * y;
```
한 개의 인자일 경우는 () 가 생략이 가능하고 별다른 로직 없이 바로 리턴할 경우는 {} 로 감쌀 필요가 없으며 return 키워드 자체가 생략이 가능하다.(화살표 함수의 큰 특징은 자신의 this, arguments 등을 바인딩 하지 않는다.)

---

### 식과 문의 특징

`세미콜론(;)`의 작성 여부를 판단하는 기준을 식, 문으로 잡는다. 간단하게 정리하면 식은 끝에 세미콜론을 붙여주고 문은 붙이지 않는다. 식에는 값, 연산식, 리터럴 객체, 함수 호출(return 값이 있거나 없으면 undefined 를 return 하므로 값) 등이 있다. 문에는 반복문, 조건문 등이 있다.

```js
const a = 'a';
const foo = () => {};
const a = 1 + 3;
foo();
```

```js
if (a) {}
while() {}
for () {}
```

---

### new 연산자, class 문법
```js
const Foo = function() {
  this.name = 'kwon';
}

const foo = new Foo();

clase Bar {
  constructor() {
    this.name = 'kwon';
  }
}

const bar = new Bar();
```

new 연산자는 빈 객체를 만들어 함수에게 전달, 인스턴스 객체를 return 한다. new 키워드로 생성된 객체의 타입(prototype 속성이 객체의 프로토타입 체인 어딘가에 존재하는지)을 알고 싶으면 `instanceof` 를 활용하면 된다.

위 Foo, Bar 는 똑같은 결과를 나타낸다. 하지만 Foo 는 new 키워드 없이 그냥 호출이 가능하고 Bar 는 반드시 new 키워드와 함께 사용이 가능하다. new 키워드를 강제하기를 원한다면 class 로 만드는 것이 좋다.

---

### this, 실행 컨텍스트
```js
const person = {
  name: 'kwon',
  getName() {
    return this.name;
  },
}

console.log(person.getName()); // "kwon"

const man = person.getName;

console.log(man()); // ""
```

man 이 호출되는 시점의 실행 컨텍스트는 전역 객체(window)를 가리키므로 name 이 존재하지 않는다. 이때 this 를 고정하기 위한 방법은 `call`, `apply`, `bind` 등의 함수를 사용할 수 있다.

---

### 클로저(closure)
```js
function makePerson() {
  let age = 10;
  return {
    getAge() {
      return age;
    },
    setAge(x) {
      age = x > 1 && x < 130 ? x : age;
    },
  };
}

p = makePerson();
console.log(p.getAge()); // 10
p.setAge(500);
console.log(p.getAge()); // 10
p.setAge(20);
console.log(p.getAge()); // 20
```

클로저는 독립적인 (자유) 변수를 가리키는 함수이다. 또는, 클로저 안에 정의된 함수는 만들어진 환경을 기억한다. `p.age` 와 같이 변수에 직접 접근은 할 수 없지만 `p.getAge` 함수로 값을 받을 순 있다는 의미이다.

---

### 비동기 Promise, async & await

```js
setTimeout(function () {
  console.log("foo");
  setTimeout(function () {
    console.log("bar");
  }, 2000);
}, 1000);
// (...after 1 second)
// foo
// (...after 2 seconds)
// bar
```

위 `setTimeout` 의 코드를 보면 비동기 로직 2 depth를 가지며 코드가 더 복잡해지면 depth 가 더 깊어지며 그 것을 `콜백 지옥` 이라고 부른다. 이를 해결하기 위해 나온 것이 `Promise`.

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('응답1');
  }, 1000);

  reject();
});

const p2 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('응답2');
  }, 1000);

  reject();
});

p1.then(p2())
  .then(function (r) {
    console.log(r);
  })
  .catch(function(err) {
    console.err(err);
  });

// (...after 1 second)
// 응답1
// (...after 1 second)
// 응답2
```

`Promise` 에서 `then` 의 콜백 함수는 resolve 로 호출되며 catch 는 reject 로 호출된다. Promise 를 더 간결하고 가독성이 좋게 만들어 주는 게 `async - await` 패턴이다.

```js
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

async function main() {
  console.log('1');
  try {
    const result = await delay(3000);
  } catch (err) {
    console.error(err);
  }
  console.log('2');
}
// 1
// (...after 3 seconds)
// 2
```

`Promise`, `resolve`, `reject` 를 `async`, `try`, `catch` 구문으로 정확하게 같은 기능을 할 수 있다. 가독성 면에서 `async - await` 키워드를 사용하는 것이 가장 좋아보인다.

---

### JavaScript 로 Redux 만들어보기

리액트 앱은 크기에 따라 수 없이 깊은 트리 형태로 이뤄져 있다. 많은 UI 컴포넌트들이 서로 data 를 주고받을 때 `Redux` 와 같은 global store 형태의 패턴을 사용하지 않으면 props, props .. 깊이만큼 전달을 반복해야 한다. 어떤 이는 리덕스를 사용하고 있다면 `data 가 최소 2 군데 이상에서 사용되어 진다면 무조건 store 에 담으라`고 말하기도 한다. 

<!-- Wip: 2020.09.06 작성 중, 혼자 다시 타이핑 후 update 예정 -->

- `redux.js`
- `index.js`

### 레퍼런스
- [일급 객체(first-class-object)](https://jcsoohwancho.github.io/2019-10-18-1%EA%B8%89-%EA%B0%9D%EC%B2%B4(first-class-object)%EC%9D%B4%EB%9E%80/)
- [수강생 김민수님의 정리](https://github.com/textuel/Woowa_Tech_Learning_React_Typescript/blob/master/ms/week_1/Thursday.md)
