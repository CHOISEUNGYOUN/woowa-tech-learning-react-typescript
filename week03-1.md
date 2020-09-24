## 우아한 테크러닝 3주차 - 1 💻

## Redux & Redux-saga

### reducer 는 동기 방식

- `index.js`
```js
import { createStore } from './redux';

function api(url, cb) {
  setTimeout(() => {
    cb({ type: '응답', data: [] });
  }, 2000);
}

function reducer(state = { counter: 0 }, action) {
  switch(action.type) {
    case: 'inc':
      return {
        ...state,
        counter: state.counter + 1,
      };
    case 'fetch-user':
      api('/api/v1/user/1', users => {
        return { ...state, ...users }
      });
      break;
    default:
      return { ...state };
  }
}

const store = createStore(reducer);

store.subscribe(() => {
  console.log(store.getState());
});

store.dispatch({
  type: 'fetch-user',
});
```

위 'fetch-user' 예시는 비동기, 리듀서에 return 할 게 아무것도 없다.(api 의 콜백함수 내에 상태를 갖고 있기 때문에)

리듀서는 의도하던, 의도하지 않던 동기적으로 작동하게 되어 있다. 리듀서는 `순수함수`(외부와 아무런 dependency, 내부에 아무런 side-effect 가 없는, [멱등성](https://ko.wikipedia.org/wiki/%EB%A9%B1%EB%93%B1%EB%B2%95%EC%B9%99#:~:text=%EB%A9%B1%EB%93%B1%EB%B2%95%EC%B9%99(%E5%86%AA%E7%AD%89%E6%B3%95%E5%89%87,%EC%95%8A%EB%8A%94%20%EC%84%B1%EC%A7%88%EC%9D%84%20%EC%9D%98%EB%AF%B8%ED%95%9C%EB%8B%A4.))) 여야만 한다.

---

### Redux Middleware

순수하지 않은 작업은 뭘까? 실행될 때 마다 결과가 일정하지 않는 작업, 대표적으로 비동기 작업(api 호출)

`"비동기 처리가 있는 side-effect 작업은 밖에서 해. 내가 놀이터를 제공해줄게."` 의 역할의 미들웨어를 만들어보자.

- `index.js`
```js
import { createStore } from './redux';

function reducer(state = { counter: 0 }, action) {
  switch(action.type) {
    case: 'inc':
      return {
        ...state,
        counter: state.counter + 1,
      };
    default:
      return { ...state };
  }
}

const store = createStore(reducer);

store.subscribe(() => {
  console.log(store.getState());
});

// 클로저 example
const myMiddleware = store => dispatch => action => {
  dispatch(action);
}

function yourMiddleware(store) {
  return function(dispatch) {
    return function(action) {
      dispatch(action);
    }
  }
}

function ourMiddleware(store, dispatch, action) {
  dispatch(action);
}

myMiddleware(store)(store.dispatch)({ type: 'inc' });
ourMiddleware(store, store.dispatch, { type: 'inc' });

```

리덕스 미들웨어 원리에 대해서는 [리덕스 공식 문서에 미들웨어에 관한 부분](https://dobbit.github.io/redux/advanced/Middleware.html)에 대해 자세히 나와있다. 강의에서 훑은 내용을 천천히 꼭 다시 읽어보자.

- `index.js`
```js
const add1 = function(a, b) {
  return a + b;
}

const add2 = function(a) {
  return function(b) {
    return a + b;
  }
}

add1(10, 20); // 30
add2(10)(20); // 30
```

`add1` 은 사용자가 과정 중간에 개입할 여지가 전혀 없다. `add2` 는 외부 함수 호출 이후 함수를 반환하기에 과정 중간에 개입할 여지가 있다.

```js
const next = add2(10);
// ... do something
next(20);
```

- 위 예시가 몽키패칭을 할 수 있는 구조.(함수 합성, 함수 조합, 함수 Wrapping?)
- 인자 두 개를 함수 두 개로 쪼개는, 인자 세 개를 함수 세 개로 쪼개는 방식을 `커링(currying)` 이라고 부른다.

> 미들웨어가 중간중간 개입해서 원하는 형태로 조작할 수 있게 만드는 개념 `몽키패칭` & `커링` & `미들웨어 체이닝`

---

### 미들웨어 & 플러그인 특징

라이브러리 본질은 바뀌지 않고 조금 응용된 추가 기능을 사용하고 싶을 때가 있다. 그럴 때 사용하는 게 `플러그인(Plugin)` 과 `미들웨어(Middleware)`.

- `데이터(data) -> 처리기 -> 출력(output)` 의 흐름(파이프라인, Pipeline) 중 `처리` 단계에 꽂아(?) 동작시키는 소프트웨어를 `미들웨어` 라고 부른다.
- 두 개의 차이는 플러그인은 선택적으로 사용되며, 미들웨어는 데이터(액션)가 사용하는 미들웨어 소프트웨어를 전부 거쳐간다.
- 미들웨어 소프트웨어는 순차적으로 액션이 흐른다. 그렇기에 의존성 있는 미들웨어는 순서를 보장해주는 게 중요하다.

---

### 기존 Redux 에 Middleware 추가한 전체 예시 코드
- `redux-middleware.js`
```js
export function createStore(reducer, middlewares = []) {
  let state;
  const listeners = [];
  const publish = () => {
    listeners.forEach(({ subscriber, context }) => {
      subscriber.call(context);
    });
  };

  const dispatch = (action) => {
    state = reducer(state, action);
    publish();
  };

  const subscribe = (subscriber, context = null) => {
    listeners.push({
      subscriber,
      context
    });
  };

  const getState = () => ({ ...state });
  const store = {
    dispatch,
    getState,
    subscribe
  };

  middlewares = Array.from(middlewares).reverse();
  let lastDispatch = store.dispatch;

  middlewares.forEach((middleware) => {
    lastDispatch = middleware(store)(lastDispatch);
  });

  return { ...store, dispatch: lastDispatch };
}

export const actionCreator = (type, payload = {}) => ({
  type,
  payload: { ...payload }
});

```

- `index.js`
```js
import { createStore, actionCreator } from "./redux-middleware";

function reducer(state = {}, { type, payload }) {
  switch (type) {
    case "init":
      return {
        ...state,
        count: payload.count
      };
    case "inc":
      return {
        ...state,
        count: state.count + 1
      };
    case "reset":
      return {
        ...state,
        count: 0
      };
    default:
      return { ...state };
  }
}

const logger = (store) => (next) => (action) => {
  console.log("logger: ", action.type);
  next(action);
};

const monitor = (store) => (next) => (action) => {
  setTimeout(() => {
    console.log("monitor: ", action.type);
    next(action);
  }, 2000);
};

const store = createStore(reducer, [logger, monitor]);

store.subscribe(() => {
  console.log(store.getState());
});

store.dispatch({
  type: "init",
  payload: {
    count: 1
  }
});

store.dispatch({
  type: "inc"
});

const Reset = () => store.dispatch(actionCreator("reset"));
const Increment = () => store.dispatch(actionCreator("inc"));

Increment();
Reset();
Increment();
```

---

### 수업 내용 외 코멘트, Q&A 기타

1. 다음 시간(17일, 목요일)엔 CRA(create-react-app)를 사용하지 않고 웹팩 설정까지 포함해서 진행할 예정이다.

2. [몽키패칭(Monkey Patch)](https://kangmin517.tistory.com/m/55?category=775720)이란 실행 중인 런타임 상태에서 소스를 직접 바꾸는 것.

3. 소프트웨어에서 한 개일 때랑 두 개일 때의 의미는 정말 많이 다르다. 늘 한 개를 만드는 건 쉽고 간단하다. 하지만 두 개를 만든다는 건 두 개의 의미가 있는 게 아니라 3, 4 .. 수 없이 많아질 수 있다는 의미.

4. 리덕스 간단한 플로우
```
1) 스토어를 통해서 dispatch 함수를 활용하여 action 을 Redux 에 전달한다.
2) 리덕스는 그 액션을 reducer 에 전달.
3) 전달 받은 action, payload 기반으로 reducer 는 상태를 return, 그 값을 기반으로 store 가 업데이트.
4) store 를 UI 컴포넌트가 구독(subscribe), 변경이 일어날 때마다 통지가 되어 UI 컴포넌트가 리렌더링 된다.
```

### 레퍼런스
- []()
