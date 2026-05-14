# learning-babel-webpack

**모던 자바스크립트 Deep Dive: 자바스크립트의 기본 개념과 동작 원리** 49장을 직접 실습한 프로젝트

React나 TypeScript 없이 순수 JavaScript만 사용해서, 번들러와 트랜스파일러가 각각 어떤 역할을 하는지 집중적으로 학습했다.

## 기술 스택

- **Webpack 5**: 모듈 번들러
- **Babel**: 트랜스파일러
  - `@babel/preset-env`: 최신 JS 문법을 구형 브라우저 대응 문법으로 변환
  - `@babel/polyfill`~~(deprecated)~~: `Promise`, `Object.assign`, `Array.from` 등 런타임 내장 객체 지원

## 프로젝트 구조

```
learning-babel-webpack/
├── src/
│   └── js/
│       ├── main.js   # 진입점, 다양한 최신 JS 문법 사용
│       └── lib.js    # ES 모듈, private class field, 거듭제곱 연산자 등
├── dist/             # 번들 결과물
├── babel.config.json
└── webpack.config.js
```

## 실습 포인트

### 트랜스파일러 vs 번들러
- **Babel**(트랜스파일러): 문법을 변환한다. `lib.js`의 private class field(`#private`), rest/spread, `**` 연산자 같은 최신 문법을 구형 브라우저가 이해하는 ES5 코드로 바꿔준다.
- **Webpack**(번들러): 모듈을 묶는다. `main.js`에서 `lib.js`를 `import`하는 것을 추적해 `dist/js/bundle.js` 하나로 합쳐준다.

둘은 완전히 다른 일을 한다. Webpack이 파일을 수집할 때 각 파일의 변환을 Babel에게 위임하는 구조다.

### `@babel/polyfill`이 entry 배열 앞에 있는 이유

```js
entry: ["@babel/polyfill", "./src/js/main.js"]
```

Babel은 문법만 변환한다. `Promise`, `Array.from` 같은 내장 객체는 구형 브라우저에 아예 존재하지 않기 때문에 문법 변환만으로는 해결이 안 된다. `@babel/polyfill`은 이 기능들을 직접 구현해서 전역에 추가해주고, 다른 코드보다 먼저 실행되어야 하므로 entry 배열의 맨 앞에 위치한다.

> **deprecated**: 현재는 `@babel/polyfill` 대신 `core-js`와 `@babel/preset-env`의 `useBuiltIns` 옵션을 함께 사용하는 방식을 권장한다.
>
> ```json
> // babel.config.json
> {
>   "presets": [
>     ["@babel/preset-env", {
>       "useBuiltIns": "usage",
>       "corejs": 3 // polyfill의 실제 구현체인 core-js의 버전
>     }]
>   ]
> }
> ```
>
> - `useBuiltIns: "usage"`: 코드에서 실제로 사용한 기능만 골라서 polyfill을 주입한다. 전체를 전역에 추가하는 `@babel/polyfill`과 달리 번들 크기를 크게 줄일 수 있다.
> - `useBuiltIns: "entry"`: entry 파일에 `import "core-js"`를 직접 추가하면, 타겟 브라우저 기준으로 필요한 polyfill 전체를 주입한다.
>
> 실제로 이 프로젝트(`Promise`, `Object.assign`, `Array.from` 사용)에서 방식을 바꿨을 때:
>
> | | `@babel/polyfill` | `useBuiltIns: "usage"` |
> |---|---|---|
> | 파일 크기 | 401 KB | 7.9 KB |
> | 줄 수 | 10,077줄 | 164줄 |

### source-map

```js
devtool: "source-map"
```

번들된 코드는 원본 코드와 완전히 달라 보여서 디버깅이 불가능하다. source-map을 생성하면 브라우저 개발자 도구에서 원본 파일 기준으로 오류 위치를 볼 수 있다.

## 실행 방법

```bash
npm install
npm run build   # webpack -w (watch 모드, 저장할 때마다 재번들링)
```
