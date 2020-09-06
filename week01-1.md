## 우아한 테크러닝 1주차 - 1

### 과정을 진행하면서 자주 들어가게 될 도구 및 문서

1. [TypeScript Playground](https://www.typescriptlang.org/play?#code/Q)
2. [CodeSandbox](https://codesandbox.io/)
3. [React](https://reactjs.org/)
4. [Redux](https://redux.js.org/)
5. [MobX](https://mobx.js.org/README.html)
6. [Redux-Saga](https://redux-saga.js.org/)
7. [Blueprint](https://blueprintjs.com/)
8. [Testing Library](https://testing-library.com/)
9. [Babel Playground](https://babeljs.io/repl)

### 강의 내용

#### TypeScript 에서 타입을 선언하는 방법

1. 타입 추론을 통해 자동으로 타입이 지정되는 경우 & 명시적으로 타입을 지정하는 경우
```ts
let foo: number = 10;
let bar = 10;
foo = false; // error
```
bar 의 타입을 별도로 foo 처럼 number 로 지정하지 않아도 자동으로 타입이 할당된다.

2. 타입 별칭(Type Alias)
```ts
let age: number = 30;

type Age = number;
let abc: Age = 28;
```

age, abc 변수는 둘 다 number 라는 타입을 갖지만 같은 number 일지라도 가독성의 차이는 있다. type alias 방식을 사용하는 주된 이유 중 하나.

3. Object 에 타입을 지정하는 방법(type, Interface)
```ts
// type
type Age = number;

type Foo = {
  age: Age;
  name: string;
}

const foo: Foo = {
  age: 30,
  name: "Kwon",
};
```
```ts
// interface
type Age = number;
interface Bar {
  age: Age;
  name: string;
}
const bar: Bar = {
  age: 30,
  name: "Kwon",
};
```

`type` 과 `interface` 는 거의 같은 기능을 한다. 확장에 대한 몇 가지 기능을 제외하고는 동일하게 동작한다고 하는데 명확하게 언제 어떤 의미로 사용하는지는 추후 강의를 통해 알아보기로.

#### Babel 을 사용하면 실제로 코드가 어떻게 변화하는지

- 변환 전 App 컴포넌트
```ts
interface AppProps {
  title: string;
}

function App(props: AppProps) {
  return <h1 id="header">{props.title}</h1>;
}
```
- babel 로 변환 후 App 컴포넌트
```js
"use strict";

function App(props) {
  return /*#__PURE__*/React.createElement("h1", {
    id: "header"
  }, props.title);
}
```

이 코드 변환되는 걸 보고 뒤통수를 맞은 듯 했다. 그렇게 아무 생각 없이 사용하던 `props` 가 `createElement` 메서드의 2 번째 인자로 들어가는 것이었다니, 리액트로 현업 개발을 하면서 이런 원리 자체에 대해 관심이 없던 과거를 반성해본다.
```js
React.createElement(
  type,
  [props],
  [...children]
)
```

> 참고 : [리액트 공식문서 - createElement()](https://ko.reactjs.org/docs/react-api.html#createelement)
