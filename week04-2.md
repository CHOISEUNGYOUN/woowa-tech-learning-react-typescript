## 우아한 테크러닝 4주차 - 2 💻

### Route

- `PrivateRoute`
```ts
import * as React from 'react';
import { IStoreState, IAuthentication } from '../store';
import {
  Route,
  Redirect,
  RouteProps,
  RouteComponentProps
} from 'react-router-dom';
import { connect } from 'react-redux';

type RoutePageComponent =
  | React.ComponentType<RouteComponentProps<any>>
  | React.ComponentType<any>;

interface IProps {
  page: RoutePageComponent;
}

interface IStateToProps {
  authentication: IAuthentication;
}

const mapStateToProps = (
  state: IStoreState,
  ownProps: IProps
): IProps & IStateToProps => ({
  ...state,
  ...ownProps
});

const PrivateRouter: React.FC<IProps & IStateToProps & RouteProps> = props => {
  const Page: RoutePageComponent = props.page;
  const { authentication } = props;

  return (
    <Route
      {...props}
      render={props => {
        if (authentication) {
          return <Page {...props} />;
        } else {
          return (
            <Redirect
              to={{
                pathname: '/login',
                state: { from: props.location }
              }}
            />
          );
        }
      }}
    >
      {props.children}
    </Route>
  );
};

export default connect(mapStateToProps)(PrivateRouter);

```

Route 는 `render` 와 `children` 형태를 둘 다 지원한다. 그렇기에 아래와 같이 `PrivateRoute` 를 만들 수 있다.

---

### AuthContainer

- `AuthContainer`
```ts
import * as React from 'react';
import { connect } from 'react-redux';
import { IStoreState } from '../store';
import { requestLogout, openNotificationCenter } from '../actions';

const AuthWrapper: React.FC = props => {
  const children = React.Children.map(
    props.children,
    (child: React.ReactElement, index: number) => {
      return React.cloneElement(child, { ...props });
    }
  );

  return <React.Fragment>{children}</React.Fragment>;
};

export const AuthContainer = connect(
  (state: IStoreState) => ({
    authentication: state.authentication
  }),
  dispatch => ({
    requestLogout: () => dispatch(requestLogout()),
    openNotificationCenter: () => dispatch(openNotificationCenter())
  })
)(AuthWrapper);
```

하위 컴포넌트들을 map 을 돌리면서 위에서 받은 prpos 를 cloneElement 를 통해 주입을 시켜주는 일종의 factory 와 유사한 역할. 반복해서 똑같이 내려가는 정보들(Auth 와 관련된)을 숨길 수 있다. 프로그래밍 테크닉, 명시성이나 그런 부분에선 조금 부족한 패턴. 네이밍은 다시 하는 게 낫지 않을까! 비즈니스 로직은 없고 단순 데이터 주입하는 컨테이너 예시이다.

---

### MobX

MobX 의 메소드에 대해 간단히 예시 타이핑. install `mobx`, `mobx-react`. mobx-react 의 v5.~ 은 리액트 훅을 지원하지 않는다.(`mobx-react-lite` 는 hook 을 지원하는 mobx-react. v6.~ 는 가능!)

- `index.tsx`
```tsx
import React from 'react';
import { render } from 'react-dom';
import { observable, action } from 'mobx';
import App from './App';

// @~ 문법 : 어노테이션, tsconfig.json 에 experimentalDecorators 를 true 로 선언해줘야 함.
class Cart {
  @observable data = 1;
  @observable counter = 1;

  @action // action 은 논리적인 작업 단위를 묶을 수 있는 메소드. Redux 의 액션이랑 유사.
  myAction = () => {
    this.data++;
    this.counter += 2;
  }
}

const cart = new Cart();

const rootElement = document.getElementById('root');
const myAction = action(() => {
  cart.data++;
  cart.counter += 2;
});

render(<App />, rootElement);

cart.myAction();
```

- `App.tsx`;
```tsx
import React from 'react';
import { inject, observer } from 'mobx-react';

interface AppProps {
  data: number;
  counter: number;
}

@inject('cart')
@observer
export default class App extends React.Component<AppProps> {
  render() {
    return (
      <div className="App">
        <h1>
          데이터 : {this.props.data} vs {this.props.counter}
        </h1>
      </div>
    );
  }
}
```

단순 뼈대 타이핑이었고 외에 실제로 사용되는 메소드들은 더 많고 공부할 필요가 있다. 리덕스와 달리 MobX 는 경험이 없어서 타이핑 따라가는 것도 무리일 것 같아서 스톱. blueprint 에 대한 간략한 설명으로 4주 과정 마무리 :)

---

### 수업 내용 외 코멘트, Q&A 기타

1. 컴파일 타임 : 개발자에 의해 개발언어도 소스코드가 작성된 후 컴파일 과정을 통해 컴퓨터가 인식할 수 있는 기계어 코드로 변환되어 실행 가능한 프로그램이 되는 과정.

2. 런타임 : 컴파일 과정을 마친 응용 프로그램이 사용자에 의해서 실행되어지면서 메모리에 탑재되고 실행시키고 있는 시간.

3. 성능은 크게 세 가지. 렌더링, 네트워크, 메모리 누수 찾기. 용량 최적화 등. 이 부분을 신경쓰면 좋다. 페이지 렌더링은 앱마다 비즈니스 로직, 불필요한 렌더링은 없는지? 체크. 브라우저 네트워크 탭에서 waterfall 이 튀는 애가 있는지(?) 검색 해보자.

4. 싱글톤 방식은 객체를 한 번만 생성해서 사용하는 방식. MobX 에서 싱글톤 패턴을 많이 사용한다고 함.

### 레퍼런스
- [레퍼런스 코드](https://codesandbox.io/s/navigation-08-live-forked-os0ku)
