## 우아한 테크러닝 3주차 - 2 💻

오늘은 웹팩 기초 설정 & 테크러닝 2기 때 자료 기반으로 React, TypeScript, Redux, Redux-Saga, Mobx 등 코드 기반으로 수업을 진행 예정이다.

### 웹팩(Webpack)

`create-react-app` 은 Zero Configuration 개발 환경 설정 방법이다. 나온 배경은 설정이 정말 어렵고 깨지기도 하고 잘 되던 게 갑자기 안되고..

`웹팩(webpack)` 은 그야말로 플랫폼적인 역할. 관용적으로 `webpack.config.js` 로 명명 후 사용, 웹팩은 node 의 실행 파일이다.(그렇기에 `require` 문법을 사용한다.) config 객체를 export 하며 그걸 node 가 받아서 설정대로 어플리케이션을 빌드한다.

```js
const webpack = require('webpack');
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const { TsconfigPathsPlugin } = require('tsconfig-paths-webpack-plugin');
```

config 속성의 entry 에 기재해준 위치로부터 내부로 들어가면서 build 한다. `core-js` 라이브러리는 하위호환 지원을 위한 polyfill 을 위해 사용한다.

```js
const config = {
  // ..
  entry: {
    main: ['core-js', './src/index.tsx'],
  }
  // ..
}
```

미들웨어인 `loader` 에 전달한다. 대표적인 loader 가 babel-loader, file-loader.. 등 굉장히 많은 loader 를 rules 배열 내에 객체로 정의한다. loader의 각 역할에 맞춰 트랜스파일링 한 뒤 return 해준다.

loader 의 프로세스가 끝난 다음 그 결과물을 `plugin` 에 넘겨준다. plugins 배열 내에는 환경 변수 주입, html 생성, 압축, 암호화, uglyfy .. 등의 후처리를 한다. loader 는 컨버팅(Converting), plugin 은 컨버팅된 소스를 기반으로 후처리해준다. loader 와 plugin 은 각자 한 모듈당 한 역할만 해야하기에 종류가 ~더럽게~ 많다. babel 내에도 각종 언어를 babel 용 plugin 을 활용해 트랜스파일링 하는 역할. plugin 이 모이면 preset.

```js
const config = {
  // ..
  module: {
    rules: [
      {
        test: /\.(ts|js)x?$/,
        include: path.resolve('src'),
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        loader: 'file-loader',
        options: {
          name: '[name].[ext]',
          outputPath: './images/',
        },
      },
    ],
  },
  plugins: [
    new webpack.SourceMapDevToolPlugin({}),
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/index.html'),
    }),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    }),
  ],
  // ..
}
```

컴파일과 트랜스파일링의 차이는 결과물에 있다. 컴파일은 기계가 아는 언어로 변환해주는 것, 트랜스파일링은 변환 후 결과물도 사람이 아는 언어로 변환되는 것. [rollup](https://rollupjs.org/guide/en/) 이나 [Parcel](https://parceljs.org/) 도 웹팩과 같은 기능을 제공하는 라이브러리다. 웹팩 공식문서에서 Get Started 를 바닐라 자바스크립트에서 하나하나 만들어보는 것이 좋다.

- `webpack.config.js` 예제 코드 전체
```js
/**
 * https://webpack.js.org/contribute/writers-guide/
 */
const webpack = require('webpack');
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const { TsconfigPathsPlugin } = require('tsconfig-paths-webpack-plugin');

const config = {
  context: path.resolve(__dirname, '.'),
  
  entry: {
    main: ['core-js', `./src/index.tsx`],
  },

  output: {
    path: path.resolve(__dirname, './build'),
    filename: '[name].js',
  },

  devtool: 'source-map',

  devServer: {
    contentBase: path.resolve('./build'),
    hot: true,
    port: 9000,
  },

  resolve: {
    modules: [
      path.resolve(__dirname, './src'),
      path.resolve(__dirname, './node_modules'),
    ],
    extensions: ['.ts', '.tsx', '.js', '.jsx', '.json'],
    alias: {
      react: require.resolve('react'),
    },
    plugins: [
      new TsconfigPathsPlugin({
        configFile: path.resolve(__dirname, './tsconfig.json'),
      }),
    ],
  },

  module: {
    rules: [
      {
        test: /\.(ts|js)x?$/,
        include: path.resolve('src'),
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
        },
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        loader: 'file-loader',
        options: {
          name: '[name].[ext]',
          outputPath: './images/',
        },
      },
    ],
  },

  plugins: [
    new webpack.SourceMapDevToolPlugin({}),
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: path.resolve(__dirname, 'public/index.html'),
    }),
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    }),
  ],

  optimization: {
    minimize: false,
  },      
};

module.exports = config;
```

### React & TS Boilerplate 리뷰하기

`typesafe-actions` 모듈을 사용하면 액션의 타입을 만들어줄 수 있다. createAction 으로 첫 번째 인자 `@fetch/success` 와 같은 형태로 상수를 전달. 골뱅이는 뽀대(컨벤션의 일종)다..* `getType` 을 활용해 상수를 만드는 것을 생략할 수 있다. (`redux-toolkit` 도 유사한 기능을 한다고 함)

- `rootSaga example`

```js
function* rootSaga() {
  const response = yield { action: 'api/users' };
  yield { id: "@inc" };
  yield { id: "@put" };
}

const root = rootSaga();

// ...
```

TODO: [codesandbox 예제 코드](https://codesandbox.io/s/ordermonitor04-forked-llkcw?file=/src/index.tsx) 를 기반으로 진행했으며 내부 코드 정리 타이핑은 추후 복습 예정. 

### 수업 내용 외 코멘트, Q&A 기타

1. 코드를 비동기(async, yield)를 활용해 동기처럼 보이게 짠 것일 뿐 실제 동기 실행이 되는 것은 아니다. 그 두 개념을 연결해서 생각하지 말자. 동기와 비동기는 완전히 다르기에 섞으면 안되는 개념.

2. 코드를 훑을 때 뭔가 대단한 게 있을 것 같지만 그렇지 않다. 코드 그 자체로 해석하고 두려워하지마!

3. Saga 는 UI 컴포넌트와 비즈니스 로직을 완전히 분리할 수 있어서 선호하는 사람이 많다.

### 레퍼런스
- [예시 코드]()
