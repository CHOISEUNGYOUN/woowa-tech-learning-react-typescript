## 우아한 테크러닝 2주차 - 2 💻

## 컴포넌트 디자인 및 비동기

### Session 만들어보기

- `index.js`

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

const rootElement = document.getElementById('root');

const sessionList = [
  { id: 1, title: '1회차: Overview' },
  { id: 2, title: '2회차: Redux 만들기' },
  { id: 3, title: '3회차: React 만들기' },
  { id: 4, title: '4회차: 컴포넌트 디자인 및 비동기' },
];

ReactDOM.render(
  <React.StrictMode>
    <App store={{sessionList}} />
  </React.StrictMode>,
  rootElement
);
```

- 클래스형 `App.js`

`Arrow Function` 은 lexical scope 를 따르기에 this 를 bind 해줄 필요가 없다.(this 가 고정된다.)

```js
export default class App extends React.Component {
  constructor(props) {
    super(props);

    this.onToggleDisplayOrder = this.onToggleDisplayOrder.bind(this);
    this.toggleDisplayOrder = this.toggleDisplayOrder.bind(this);

    this.state = {
      displayOrder: 'ASC',
    }
  }

  onToggleDisplayOrder() {
    this.setState({
      displayOrder: displayOrder === 'ASC' ? 'DESC : 'ASC',
    })
  }

  toggleDisplayOrder = () => {
    this.setState({
      displayOrder: displayOrder === 'ASC' ? 'DESC' : 'ASC',
    })
  }

  render() {
    return (
      <div>
        여기여기
        <button onClick={this.toggleDisplayOrder}>정렬</button>
      </div>
    )
  }
}
```

---

- 함수형 `App.js`

`상태(state)`는 클래스형 컴포넌트에서만 사용 가능했다. 하지만 hook(useState)이 나온 이후엔 함수형 컴포넌트도 상태를 가질 수 있다.

```js
import React, { useState } from 'react';

const SessionItem = ({ title }) => <li>{title}</li>;

const App = props => {
  const [displayOrder, toggleDisplayOrder] = useState('ASC');
  const { sessionList } = props.store;
  const orderedSessionList = sessionList.map((session, i) => {
    ...session,
    order: i,
  })

  const onToggleDisplayOrder = () => {
    toggleDisplayORder(displayOrder === 'ASC' ? 'DESC' : 'ASC');
  }

  return (
    <div>
      <header>
        <h1>React & TypeScript</h1>
      </header>
      <p>전체 세션 갯수: 4개 {displayOrder}</p>
      <button onClick={onToggleDisplayOrder}>재정렬</button>
      <ul>
        {sessionList.map(session => <SessionItem title={session.title} />)}
      </ul>
    </div>
  )
}

export default App;
```

> Session 만들어보기는 준비가 조금 덜 되어 이 정도로 간단하게 타이핑 끝내고, 다음 시간에 자료를 마련 후 다른 방향으로 진행 예정.

### 제너레이터(Generator)와 비동기(Asynchronous)

- `Promise`
```js
const p = new Promise(function(rseolve, reject) {
  // 
  setTimeout(() => {
    resolve('1');
  }, 1000);
})

p.then(function(r) {
  console.log(r); // 1
})
```

- `제너레이터 함수(코루틴)`

함수는 인자(입력값)를 받고 무언가 계산을 한 뒤에 값을 return 하는 것이다. 함수인데 return 을 여러번 할 수 있게하면 어떨까? 마지막 return 지점부터 다시 시작(반복)되는 컨셉이다.

`yield` 는 제너레이터 함수의 return 과 유사하다고 생각하자. 함수를 종료시키진 않지만 정지시키고 바깥으로 내보낸다? 정도로 이해. `next` 메소드를 사용하면 다음 번 `yield` 가 있는 곳까지 실행한다. 제너레이터 함수의 특징은 return 으로 종결되지 않으니 값을 쥐고 있다는 점!

> `코루틴`이란 caller 가 함수를 call 하고, 함수가 caller 에게 값을 return 하면서 종료하는 것에 더해
> return 하는 대신 suspend(혹은 yield)하면 caller 가 나중에 resume 하여 중단된 지점부터 실행을 이어갈 수 있다.

```js
function* makeNumber() {
  let num = 1;

  while(true) {
    yield num++;
  }
}

const i = makeNumber();

console.log(i.next()); // { value: 1, done: false }
console.log(i.next()); // { value: 2, done: false }

const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

function* main() {
  console.log('시작');
  yield delay(3000);
  console.log('3초 뒤');
}

const iter = main();
const { value } = iter.next(); // value: Promise
value.then(() => {
  iter.next();
})
```

- `비동기 async await`

제너레이터 함수와 유사하다. `async` 키워드와 함께 `yield` 키워드가 await 가 된 정도.

async await 는 Promise 에 최적화 되어있는 함수. 제너레이터는 Promise 비동기 뿐 아니라 다양한 케이스에서 사용이 가능하기에 응용 범위가 더 넓다. 포인트는 `async` 내부도 제너레이터로 이뤄져있다는 점!

```js
const delay = ms => new Promise(resolve => setTimeout(resolve, ms));

async function main() {
  console.log('시작');
  await delay(3000);
  console.log('3초 뒤');
}
```

### 수업 내용 외 코멘트, Q&A 기타

1. 지식을 배우는 데에 가장 효과적인 방법은 누군가에게 쏟아내는(알려주는) 것이라고 생각함.

2. 아주 기본적인 `소프트 스킬(Soft Skill)` 중 커뮤니케이션에 있어서 가장 중요한 건 `약속(프로토콜, Protocol)`이라고 생각한다. 함께 일하다보면 그게 맞지 않는 경우가 많은데 의식조차 못하는 사람도 대다수. 정보를 주고받는다는 건 `이해의 수준(레이어, Layer)`이 일치(유사, 동등)해야한다. 비개발자에게 개발 언어를 사용하면 이해하는 것이 어렵 듯 상대에 따라 단어 선택도 바꿔야할 필요가 있다. `상대(Target Layer)` 를 잘 이해(의식)하고 커뮤니케이션을 할 수 있길, 커뮤니케이션뿐 아니라 어떤 분야던.

3. 학습도 물론 마찬가지. 교육 과정이나 강의 등을 고를 때를 예로 들면 유명하고 인기 있는(핫한) 게 좋은 것이 아니라 `내가 이해할 수 있는 영역인지?`, `정말 필요한지?` 가 중요하다고 받아들여진다. 고급 테크닉만 쫓는다고 좋은 건 아니다. 주니어 시절에 갖추는 본질과 기본이 중요하다.(편식 No! 차곡차곡!)

4. 한 방향으로 너무 반복되면 습관이 되고 몸에 붙게 된다. 그걸 떼어내기가 쉽지 않으므로 의욕이 넘칠 땐 위험할 수 있으니 조금 돌아보고 낮추는 것도 필요.

5. `제너레이터(Generator)`는 비동기를 동기적으로 사용하기 위해 만든 것이 아니라 단지 `코루틴`이라는 매커니즘을 활용해 만든 것. 만들고 보니 `비동기를 동기처럼 사용할 수 있는 방식으로 사용이 가능하겠다.`하여 사용하는 것이지 단지 그 목적으로 만들어진 것이 아니다. 뭐가 우선인지 컨셉을 이해하는 게 정말 중요하다.

### 레퍼런스
- []()
