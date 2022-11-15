# Replacing render with createRoot

> 원본 글  
> https://github.com/reactwg/react-18/discussions/5

**목차**

- [Replacing render with createRoot](#replacing-render-with-createroot)
  - [개요](#개요)
  - [What is a root?](#what-is-a-root)
  - [What are the differences?](#what-are-the-differences)
  - [What about hydration?](#what-about-hydration)
    - [Before:](#before)
    - [After:](#after)
  - [What about the render callback?](#what-about-the-render-callback)
    - [useEffect or Ref callback ?](#useeffect-or-ref-callback-)
  - [Why ship both?](#why-ship-both)

## 개요

**React 18**은 두 가지 **Root API**를 지원하는데, 하나는 **Legacy API** 나머지 하나는 18에 새롭게 추가된 Root API이다.

- **Legacy root API**

  `ReactDOM.render`로 불리는 기존에 존재하던 **Root API**이다. 이 메서드를 사용하면 이제 Root가 “**legacy**” 모드로 동작하며 **React 17**과 동일하게 동작한다. **React 18**이 공식 릴리지가 된 상황이니 이제 **React 18**을 사용하고 있는 상황에서 위 메서드를 사용하게 되면 경고가 발생한다. (해당 메서드는 `deprecated`가 됐으며 새로운 Root API로 교체하라고)

- **New root API**

  `ReactDOM.createRoot`로 불리는 새로운 **Root API**이다. 이 메서드를 사용하면 Root가 **React 18**로 동작하게 만들며, **React 18**의 변경점과 새롭게 추가된 동시성 기능들을 사용할 수 있도록 만들어 준다.

> ✋ **주의**  
> 새로운 Root API는 `react-dom/client`로부터 import한다.

```jsx
import * as ReactDOMClient from 'react-dom/client';

ReactDOMClient.createRoot(/*...*/);

ReactDOMClient.hydrateRoot(/*...*/);
```

## What is a root?

리액트에서 "**root**"는 렌더링할 트리를 추적하는데 사용되는 최상위 자료 구조의 포인터다. (렌더링 트리의 진입점 역할을 한다고 보면 될 것 같다. `Webpack` 설정에서 `entry`가 의존성 트리의 진입점이듯이)

**Legacy API**에서 `root`는 사용자에게 감춰져 있는 상태였다. 왜냐하면 `root`를 통해 DOM 노드를 거쳐 DOM 요소에 접근할 수 있었기 때문에 절대 사용자에게 노출되선 안됐었다.

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Intial render.
ReactDOM.render(<App tab="home" />, container);

// During an update, React would access
// the root of the DOM element
ReactDOM.render(<App tab="profile" />, container);
```

새로운 **Root API**에선 `Root` 객체를 만든 뒤에 해당 객체가 `render` 메서드를 호출한다.

```jsx
import * as ReactDOMClient from 'react-dom/client';
import App from 'App';

const container = document.getElementById('app');

// Create a root.
const root = ReactDOMClient.createRoot(container);

// Initial render: Render an element to the root.
root.render(<App tab="home" />);

// During an update, there's no need to pass the container again.
root.render(<App tab="profile" />);
```

## What are the differences?

**New Root API**를 만든 데에는 몇 가지 이유가 있다.

첫 번째로, 기존 **Legacy Root API**가 **DOM** 업데이트를 진행할 때 갖고 있던 인간 공학(ergonomics)적인 몇 가지 부분을 수정할 수 있었다. (아마 기존 **Legacy Root API**가 **DOM** 업데이트를 진행하는 방식이 인간의 시점에서 설계된 부분이 있는 것 같다. 그 부분을 조금 더 컴퓨터 사고로의 전환이 이루어 진게 아닐까)

위에서 볼 수 있듯 **Legacy Root API**에서는 렌더링을 진행할 때 항상 `container`를 파라미터로 전달해야 했다. 심지어 **DOM**에 아무런 변화가 없었음에도 말이다. 이는 우리가 **DOM** 노드에 `root`를 저장할 필요가 없음을 알 수 있게 해준다.

두 번째로, **New Root API**로 전환함에 따라 `hydrate` 메서드를 제거하는 대신 이를 `root`의 옵션 중 하나로 만들어 줄 수 있게 됐다. `hydrate`가 `root`의 옵션으로 넘어가게 되면서 이러한 `partial hydration`에는 필요성이 사라진 `render` 메서드의 콜백 함수 파라미터 또한 제거됐다.

## What about hydration?

기존의 `hydrate` 함수는 `hydrateRoot` **API**로 변경됐다.

### Before:

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// Render with hydration.
ReactDOM.hydrate(<App tab="home" />, container);
```

### After:

```jsx
import * as ReactDOMClient from 'react-dom/client';

import App from 'App';

const container = document.getElementById('app');

// Create *and* render a root with hydration.
const root = ReactDOMClient.hydarateRoot(container, <App tab="home" />);
// Unlike with createRoot, you don't need a seperate root.render() call here
```

`createRoot`와는 다르게, **`hydrateRoot`**는 초기 JSX 요소를 두 번째 인자값으로 전달해줘야 한다. 두 번째 인자값으로 초기 **JSX** 요소를 전달해줘야 하는 이유는 클라이언트 사이드의 첫 렌더링이 특별하며 클라이언트 측 DOM 트리와 서버 측 DOM 트리를 일치시킬 필요가 있기 때문이다..

만약 `hydration` 이후에 다시 root를 업데이트하길 원한다면, 변수에 root를 저장한 뒤에 `createRoot`처럼 root 객체의 `render` 메서드와 함께 다음과 같이 호출하면 된다.

```jsx
import * as ReactDOMClient from 'react-dom/client';
import App from 'App';

const container = document.getElementById('app');

// Create *and* render a root with hydration.
const root = ReactDOMClient.hydrateRoot(container, <App tab="home" />);

// You can later update it
root.render(<App tab="profile" />);
```

## What about the render callback?

**Legacy Root API**에선, `render` 메서드에 컴포넌트가 렌더링되거나 업데이트 될 때 호출될 콜백 함수를 인자값으로 전달할 수 있었다.

```jsx
import * as ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

ReactDOM.render(container, <App tab="home" />, function () {
  // Called after initial render or any update.
  console.log('rendered');
});
```

새로운 **Root API**에선, 해당 콜백이 제거됐다.

부분적인 `hydration`과 점진적인 서버 사이드 렌더링은 콜백 함수가 호출되는 타이밍을 사용자의 생각과는 다르게 동작하게 만들기 때문이다. 이러한 혼란을 피하기 위해서 `requestIdleCallback`, `setTimeout` 또는 `root`에서의 `ref` callback 사용을 추천한다.

```jsx
import * as ReactDOMClient from 'react-dom/client';

function App() {
  return (
    <div>
      <h1>Hello World</h1>
    </div>
  );
}

const rootElement = document.getElementById('root');
```

그래서 위 방법 대신에 다음과 같이 사용할 수 있다.

```jsx
import * as ReactDOMClient from 'react-dom/client';

function App({ callback }) {
  // Callback will be called when the div is first created.
  return (
    <div ref={callback}>
      <h1>Hello World</h1>
    </div>
  );
}

const rootElement = document.getElementById('root');

const root = ReactDOMClient.createRoot(rootElement);
root.render(<App callback={() => console.log('rerendered')} />);
```

### useEffect or Ref callback ?

> Ref will fire synchronously when the `div` is added to the DOM. Effects fire with a small delay (after the browser paint). So this depends on what exactly you want to do in this callback.

- DOM에 특정 태그가 추가될 때 실행되는 것이 **`Ref callback`**
- 브라우저의 `paint` 작업이 완료된 후 실행되는 것이 **`useEffect`**

따라서, 콜백 함수를 통해 무엇을 하고 싶은지에 따라 원하는 로직을 작성하면 될 것 같다.

## Why ship both?

**React 18**에서 **Legacy Root API**를 여전히 지원하는 이유는 다음과 같다.

- **Smooth upgrade**

  **React** 버전을 17 이하에서 18로 업그레이드 시키면서 바로 앱에 문제가 발생하는 일을 만들고 싶지 않기 때문이다. 바로 문제가 발생하지 않는 대신 경고창을 발생시켜 새로운 **Root API**로의 전환을 추천하려고 한다.

- **Experimentation**

  몇몇의 앱들은 **Legacy Root API**와 **New Root API**의 퍼포먼스 차이를 프로덕션 단에서 실험해볼 가능성도 있기 때문에 두 가지 **Root API**를 모두 지원하는 방향으로 결정
