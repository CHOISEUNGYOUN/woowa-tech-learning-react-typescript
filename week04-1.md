## 우아한 테크러닝 4주차 - 1 💻

### Promise

```js
const p1 = new Promise(resolve => {
  setTimeout(() => {
    console.log('@4초 완료');
    resolve('4초 작업 완료');
  }, 4000);
});

const p2 = new Promise(resolve => {
  setTimeout(() => {
    console.log('@2초 완료');
    resolve('2초 작업 완료');
  }, 2000);
});

p1.then(res => {
  console.log(res);
}).then(res => {
  console.log(res);
});
```

`then` 은 실제 내부 비동기와는 상관이 없다. 단순히 resolve 가 실행 되었는지 확인하는 인터페이스이다. 위 p1, p2 는 병렬로 동작.

`Promise` 는 `state machine` 이다. 이 개념에 대해서는 별도로 공부해보자.

```js
const p1 = () => new Promise(resolve => {
  setTimeout(() => {
    console.log('@4초 완료');
    resolve('4초 작업 완료');
  }, 2000);
})

p1().then(res => {
  console.log(res);
})
```

시점을 명확히 알 수 없으니 함수로 만들어서 p1 의 실행 시점을 강제하는 위와 같은 방식(함수로 wrapping)으로 사용한다.

---

### interface vs type alias

기능적으로는 거의 유사하다. 작은 차이는 `interface` 는 상속을 지원하고 `Union` 타입을 지원하지 않는다. 

인터페이스라는 이름에 집중을 해보면 `소통` 이라는 느낌이 있다. 컴포넌트 간 소통이 있다면 `interface` 를 사용. 모든 인터페이스는 `pulic` 이다. 리액트 컴포넌트의 Props 를 정의하는 건 단순한 타입이 아니라 `interface` 에 가깝지 않을까? 반면에 내부에만 한정적일 땐 `type` 을 사용한다. Union 타입이 아닌 경우엔 대부분 `interface` 로 쓰기도 하고.. 정답은 없다.

> 어떤 게 런타임에, 어떤 게 컴파일 타임에 일어나는 일인지 명확하게 알고있지 않으면 버그를 예측하기 힘들다. 첫 시간에 말했 듯 명확하게 구분을 해야한다.

---

### Maybe 컴포넌트

```ts
import React from 'react';

interface MaybeProps {
  test: boolean;
  children: React.ReactNode;
}

const Maybe: React.FC<MaybeProps> = ({ test, children }) => (
  <>{test ? children : null}</>
);

export default Maybe;
```

&& 연산자나 삼항연산자처럼 jsx 태그 내에 더러움을 방지? 해주는 유용한 유틸 컴포넌트.

---

### 수업 내용 외 코멘트, Q&A 기타

1. 언어 수준이 아닌 라이브러리는 공식 문서 1회독은 기본적으로 하는 게 좋다. 언어 만큼 양이 방대하진 않고 전체적인 컨셉을 이해하는 건 중요하고 이런 부분은 지름길을 찾으려고 하지 말자.

2. React vs Vue. 리액트를 사용하는 이유는 개발 생태계가 더 크고 선택의 폭(자유도)이 더 높은 것 같다는 생각도 가지고 있다. 허들과 자유도는 한 끗 차이.

3. axios, fetch? 어떤 차이가 있는가? 브라우저마다 메소드 등의 규약이 다 다를 수 있다. `polyfill(babel, corjs)` 을 사용하기도 하여 ES5 로 변환하여.. 브라우저가 이해할 수 있게 만들어준다. `axios` 는 다양한 브라우저 대응이 되는 핫한 네트워크 라이브러리. `fetch` 는 브라우저 종속.. 놓쳤다..!

4. 원칙적으로 컴포넌트는 어떻게 나눌까? 외부 상태에 의존적인(비즈니스 로직이 포함되어있는) Component, 외부 상태에 의존적이지 않은 Component 로 나눈다. props 만 존재하는 컴포넌트와 비즈니스 로직이 들어가있는 컴포넌트로 이해해도 좋지 않을까. 컨테이너 컴포넌트와 개별 컴포넌트로 나누는 방식.(실제 우리 회사에서도 이에 가까운 것 같다)

5. 프로젝트의 볼륨이 커지면 `코드 스플리팅(웹팩 번들러를 활용해 분할, 그룹핑)` 까지 고려해보는 것도 필요. 혹은 서버사이드 렌더링(SSR). 이때 서버 부하도 고려해서 만들어야 함.

6. 컴포넌트 파일명에 역할을 다 부여(작명)하는 방식으로 사용하곤 한다. 암묵적인 이름보단 명시적인 이름.

7. `mapStateToProps` 는 props 를 통해 state(store의) 를 매핑(주입)해줘. `mapDispatchToProps` 는 props 를 통해 Dispatch 를 매핑(주입)해줘. 지금은 `Redux` 도 hook 을 지원하기에 훨씬 깔끔하게 사용 가능하다.

8. 컴파일 단계는 `설계를 검증하는 단계` 라는 비유.

9. `saga` 의 `takeLatest` 는 연속된 액션이 계속 들어오면 마지막 액션만 취하는 메소드.

---

### 레퍼런스
- [레퍼런스 코드](https://codesandbox.io/s/ordermonitor08-1ttxf)
