# Why React Context is Not a "State Management" Tool (and Why It Doesn't Replace Redux)

> 원본 글  
> https://blog.isquaredsoftware.com/2021/01/context-redux-differences/

**목차**

- [Why React Context is Not a "State Management" Tool (and Why It Doesn't Replace Redux)](#why-react-context-is-not-a-state-management-tool-and-why-it-doesnt-replace-redux)
  - [개요](#개요)
  - [요약](#요약)
    - [Are Context and Redux the same thing?](#are-context-and-redux-the-same-thing)
    - [Is Context a "state manangement" tool?](#is-context-a-state-manangement-tool)
    - [Are Context and `useReducer` a replacement for Redux?](#are-context-and-usereducer-a-replacement-for-redux)
    - [When should I use Context?](#when-should-i-use-context)
    - [When should I use Context and `useReducer`?](#when-should-i-use-context-and-usereducer)
    - [When should I use Redux instead?](#when-should-i-use-redux-instead)
- [Understanding Context and Redux](#understanding-context-and-redux)
  - [What is React Context?](#what-is-react-context)
    - [Using Context](#using-context)
    - [Purpose and Use Cases for Context](#purpose-and-use-cases-for-context)
  - [What is Redux?](#what-is-redux)
    - [Redux and React](#redux-and-react)
    - [Purposes and Use Cases for (React-)Redux](#purposes-and-use-cases-for-react-redux)
  - [Why Context is Not "State Management"](#why-context-is-not-state-management)
  - [Comparing Context and Redux](#comparing-context-and-redux)
    - [Context](#context)
    - [React+Redux](#reactredux)
  - [Context and `useReducer`](#context-and-usereducer)
  - [Choosing the Right Tool](#choosing-the-right-tool)
    - [Use Case Summary](#use-case-summary)
  - [Recommendations](#recommendations)
  - [Final Thoughts](#final-thoughts)

## 개요

"**Context vs Redux**"는 지금의 **React Context API**가 릴리즈된 이래로 **React** 커뮤니티 내에서 가장 넓게 토론되고 있는 주제 중 하나다. 안타깝게도 이 논쟁은 두 가지 도구(Context API와 Redux)의 목적과 use-case에 대한 혼란에서 비롯됐다. 필자는 [Redux - Not Dead Yet!](https://blog.isquaredsoftware.com/2018/03/redux-not-dead-yet/), [React, Redux, and Context Behavior](https://blog.isquaredsoftware.com/2020/01/blogged-answers-react-redux-and-context-behavior/), [A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/), and [When (and when not) to Reach for Redux](https://changelog.com/posts/when-and-when-not-to-reach-for-redux)와 같은 포스팅을 포함하여 인터넷 전반에 걸쳐 수백번도 더 **Context**와 **Redux**에 대한 다양한 질문에 답해왔다. 하지만 여전히 이 혼란은 계속되고 심지어 더 악화되고 있다.

이 주제에 대한 질문이 널리 퍼져 있기 때문에 이러한 질문들에 대한 최종 답변을 현재 포스팅에 적고자 한다. 이 포스팅에선 **Context**와 **Redux**가 실제로 무엇이며 사용 방법은 무엇인지 그리고 이들은 얼마나 다르고 어떻게 사용해야 하는지에 대해 알아볼 것이다.

## 요약

### Are Context and Redux the same thing?

아니다. 이 둘은 다른 일을 하는 다른 도구들이이며 서로 다른 목적을 위해 사용해야 한다.

### Is Context a "state manangement" tool?

아니다. **Context**는 일종의 의존성 주입이다. 이는 `transport` 메커니즘이며 무언가를 "**관리**"하지 않는다. 모든 "상태 관리는 일반적으로 `useState`/`useReducer`를 통하여 우리와 우리의 코드를 통해서 이루어진다.

### Are Context and `useReducer` a replacement for Redux?

그럴 수 없다. 이 둘의 조합은 **Redux**와 비슷한 점 그리고 겹치는 부분이 있긴 하지만 기본적으로 할 수 있는 일에서 큰 차이점이 존재한다.

### When should I use Context?

특정 값을 컴포넌트의 각 레벨을 통하여 `props`를 아래로 전달하는 것 없이 **React** 컴포넌트 트리의 특정 부분에서 접근 가능하도록 만들고 싶을 때 언제든지 사용하면 된다.

### When should I use Context and `useReducer`?

애플리케이션의 특정 섹션 내에서 적당히 복잡한 **React** 컴포넌트 상태 관리가 필요할 때 사용하면 된다.

### When should I use Redux instead?

**Redux**는 다음과 같은 케이스에서 가장 유용하다.

- 애플리케이션 내 많은 곳에서 거대한 양의 애플리케이션 상태를 필요로 할 때
- 애플리케이션의 상태가 시간이 지남에 따라 빈번하게 업데이트되는 경우
- 상태를 업데이트하는 로직이이 복잡한 경우
- 애플리케이션이 중간 또는 큰 규모의 코드베이스를 가지고 있고 많은 사람들이 같이 이를 작업하는 경우
- 언제, 왜, 그리고 어떻게 애플리케이션의 상태가 업데이트가 되는지 이해하고 싶고 시간에 따른 상태 변화를 시각화하고 싶을 때
- 사이드 이펙트, persistence 그리고 데이터 직렬화에 대한 더 강력한 기능이 필요할 때

# Understanding Context and Redux

어떤 도구가 됐건 간에 정확하게 사용하기 위해선 다음과 같은 사항을 이해하는 것이 중요하다.

- 해당 도구의 목적이 무엇인지
- 해당 도구는 어떤 문제를 해결하고자 하는지
- 해당 도구가 처음 만들어진 때와 이유는 무엇인지

여기에 더해 현재 애플리케이션에서 해결하고자 하는 문제가 무엇인지 이해하고 해당 문제를 가장 잘 해결할 수 있는 도구를 선택하는 것이 중요하다. 도구를 선택한 이유가 단지 누군가 해당 도구를 해서 또는 해당 도구가 유명해서가 아니라 특정 상황에 이 도구가 가장 적합하기 때문이여야 한다.

"**Context vs Redux**"에 대한 혼란의 대부분은 이 도구들이 실제로 무엇을 하며 어떤 문제를 해결하고자 하는지 몰라서이다. 그래서 해당 도구들을 실제로 언제 사용하는지 알기 위해선 먼저 도구들이 무엇을 하고 어떤 문제를 해결하고자 하는지에 대해 명확하게 정의해야 한다.

## What is React Context?

Let's start by looking at [the actual description of Context from the React docs](https://reactjs.org/docs/context.html):

> **Context**는 `props`를 모든 단계에 일일히 내려줄 필요 없이 컴포넌트 트리를 통하여 데이터를 전달하기 위한 방법을 제공한다.
>
> 일반적인 **React** 애플리케이션에선, 데이터는 `props`를 통해 하향식(부모에서 자식)으로 전달해야 하지만 애플리케이션 내 많은 컴포넌트들이 필요로 하는 특정 타입의 `props`(예를 들어, **locale**, **preference**, **UI theme**)의 경우 번거로울 수 있다. **Context**는 트리의 모든 단계를 통하여 prop를 명시적으로 내려주지 않고도 컴포넌트 간에 특정 값을 공유할 수 있는 방법을 제공한다.

위 내용을 봤을 때 어디에서 "**관리**"한다라는 말은 존재하지 않는다. 오직 값을 "**전달**"하고 "**공유**"한다는 말 밖에 없다.

[지금의 React Context API(React.createContext())는 React 16.3에 처음 릴리즈됐다](https://reactjs.org/blog/2018/03/29/react-v-16-3.html). 이는 이전 버전에서 사용이 가능했으나 설계 결함을 가지고 있었던 [Legacy Context API](https://reactjs.org/docs/legacy-context.html)를 대체했다. **Legacy Context**의 가장 큰 문제는 만약 컴포넌트가 `shouldComponentUpdate`를 통하여 렌더링을 건너뛰었을 경우 **Context**를 통해 전달된 값이 업데이트가 되지 않을 수도 있다는 점이었다. 많은 컴포넌트들이 성능 최적화를 위해 `shouldComponentUpdate`에 의존하고 있었기 때문에 **plain** 데이터를 전달하는데 **Legacy Context**를 사용할 이유가 없었다. `createContext()`는 이런 문제를 해결하기 위해 설계가 됐기에 컴포넌트가 중간에 렌더링을 건너뛰었을지라도 값에 대한 모든 업데이트가 하위 컴포넌트에서 확인할 수 있었다.

### Using Context

애플리케이션에서 **React Context**를 사용하려면 몇 가지 단계가 필요하다.

1. `const MyContext = React.createContext()`를 호출하여 **Context** 객체 인스턴스를 생성해야 한다.
2. 상위 컴포넌트에서 `<MyContext.Provider value={someValue}>`를 렌더링 해야 한다. 이렇게 하면 Context에 단일 데이터 조각(some single piece of data)이 포함된다. 여기서 데이터는 어떤 것(`string`, `number`, `object`, `array`, `class instance`, `event emitter` 등)도 될 수 있다.
3. 그리고 나서 **Provider** 내에 중첩된 컴포넌트 중 아무 컴포넌트에서나 `const theContextValue = useContext(MyContext)`를 호출하면 된다.

상위 컴포넌트가 리렌더링될 때마다 새로운 참조값이 **Context Provider**의 `value`로 전달되며 **Provider** 내에 있으면서 **Context**로부터 값을 읽고 있는 컴포넌트라면 강제로 리렌더링된다.

대개 일반적으로 **Context**를 통해 전달할 `value`에는 다음과 같은 **React** 컴포넌트 상태값을 전달한다.

```jsx
function ParentComponent() {
  const [counter, setCounter] = useState(0);

  // Create an object containing both the value and the setter
  const contextValue = { counter, setCounter };

  return (
    <MyContext.Provider value={contextValue}>
      <SomeChildComponent />
    </MyContext.Provider>
  );
}
```

이러면 하위 컴포넌트에서 `useConext`를 호출하여 값을 읽을 수 있게 된다.

```jsx
function NestedChildComponent() {
  const { counter, setCounter } = useContext(MyContext);

  // do something with the counter value and setter
}
```

### Purpose and Use Cases for Context

위를 참고해 봤을 때 **Context**는 실제로 어떤 것도 "관리"하지 않는다는 것을 알 수 있다. 대신에 **Context**는 일종의 파이프 또는 웜홀이다. `<MyContext.Provider>`를 사용하여 파이프의 최상단에 무언가를 집어 넣으면 그게 무엇이든 반대편으로 튀어나올 때까지 파이프를 타고 내려간다. 여기서 반대편은 `useContext(MyProvider)`를 통하여 요청하는 다른 컴포넌트를 말한다.

따라서, **Context**을 사용하는 주요 목적 중 하나는 **prop-drilling**을 피하기 위해서다. 특정 값을 필요로 하는 컴포넌트 트리의 모든 레벨(`deps`)을 통하여 명시적으로 `prop`을 내리는 대신 `<MyContext.Provider>` 내에 중첩된 모든 컴포넌트들은 해당 값이 필요할 때 `useContext(MyContext)`를 호출하여 값을 가져오면 된다. 이는 추가적인 **prop-passing** 로직을 작성하지 않아도 되게 만들어 주기 때문에 코드를 조금 더 간단하게 만든다.

이론적으로 이는 "**의존성 주입**"의 형태다. 하위 컴포넌트에서 특정 타입의 값이 필요하다는 것은 알고 있지만 해당 값을 자체적으로 생성하거나 설정하려고 하지 않는다. 대신에 런타임에 몇몇의 상위 컴포넌트에서 해당 값을 내려준다고 가정한다.

## What is Redux?

For comparison, let's look at the description from [the "Redux Essentials" tutorial in the Redux docs](https://redux.js.org/tutorials/essentials/part-1-overview-concepts):

> **Redux**는 "**actions**"라고 불리는 이벤트를 사용하여 애플리케이션의 상태를 관리하고 업데이트하기 위한 패턴이자 라이브러리다. 이는 상태값이 예측 가능한 방식으로만 업데이트 되어야 한다는 규칙을 사용하여 애플리케이션 전체에 걸쳐 사용되는 상태값들의 중앙 저장소 역할을 한다.
>
> **Redux**는 "전역" 상태를 관리할 수 있게 도와준다. 여기서 말하는 상태란 애플리케이션 내 많은 부분에서 사용되는 값을 말한다.
>
> **Redux**에서 제공하는 패턴과 도구는 애플리케이션 내 상태값을 언제, 어디서, 왜, 그리고 어떻게 업데이트 해야할 지 이해하기 쉽도록 만들어주고 이러한 업데이트 사항이 발생했을 때 애플리케이션 로직이 어떻게 동작해야 하는지 알려준다.

위 설명을 정리하면 다음과 같다.

- "**상태 관리**"
- **Redux**의 목적은 시간이 지남에 따라 상태값이 어떻게 변하는지 이해하도록 돕는 것이다.

**Redux**는 원래 **React**가 출시된 후 일년이 지난 2014년에 페이스북에서 제안한 패턴인 "**Flux 아키텍처**"의 구현체였다. 해당 발표가 있고 난 후 커뮤니티에서는 **Flux** 아키텍처에 영감을 받아서 **Flux**에 대한 다양한 접근 방법으로 수 십개의 라이브러리가 만들어 졌다. 2015년에 나온 **Redux**가 "**Flux 전쟁**"에서 일찍이 이긴 이유는 당시에 사람들이 겪고 있는 문제를 해결하기에 제일 적합한 디자인을 갖고 있었고 **React**와 아주 잘 동작했기 때문이다.

구조적으로 **Redux**는 함수형 프로그래밍 원칙을 사용하여 예측 가능한 "**reducer**" 함수로 가능한 한 많은 코드를 작성할 수 있도록 도와주며 "**어떤 이벤트가 발생했는지**"에 대한 아이디어를 "**해당 이벤트가 발생할 때 상태가 업데이트되는 방법**"을 결정하는 로직에서 분리한다. **Redux**는 사이드 이펙트를 다루는 것을 포함하여 Redux 스토어의 능력을 확장하는 데 미들웨어를 사용한다.

**Redux**는 또한 시간의 흐름에 따른 상태값과 `action`의 변화를 기록으로 볼 수 있게 해주는 **Redux DevTools**를 가지고 있다.

### Redux and React

**Redux** 자체는 **UI**에 구애받지 않는다. **Redux**는 모든 **UI** 레이어(**React**, **Vue**, **Angular**, **Vanilla JS** 등) 또는 UI이 없이도 사용할 수 있다.

즉, **Redux**는 대개 일반적으로 React와 사용된다는 것이다. **React-Redux** 라이브러리는 **Redux** 상태값으로부터 값을 읽고 `action`을 `dispatch` 하여 **Redux** 스토어와 **React** 컴포넌트가 상호 작용할 수 있도록 하는 공식적인 UI 바인딩 레이어다. 따라서, 대부분의 사람들이 **Redux**를 언급할 때, 이들이 실제로 말하고자 하는건 "**Redux 스토어와 React-Redux 라이브러리를 함께 사용**"하는 것을 의미한다.

**React-Redux** 라이브러리는 애플리케이션 내 모든 **React** 컴포넌트가 **Redux** 스토어와 통신할 수 있게 만든다. 이는 **React-Redux**가 내부적으로 Context를 사용하기 때문에 가능한 것이다. 하지만 여기서 중요한 점은 **React-Redux**가 **Context**를 통하여 **Redux** 스토어의 인스턴스를 전달할 뿐이지 현재 상태값을 전달하는 것이 아니라는 점이다. 이는 실제로 위에서 언급한 것처럼 의존성 주입을 위해 **Context**를 사용하는 예일 뿐이다. 우리는 **Redux**에 연결된 **React** 컴포넌트들이 **Redux** 스토어와 통신해야 한다는 것을 알고 있지만 컴포넌트를 정의할 때 **Redux** 스토어의 존재를 모르거나 신경쓰지 않는다. 실제로 **Redux** 스토어는 **React-Redux** `<Provider>` 컴포넌트를 사용하여 런타임에 트리에 주입된다.

이렇기 때문에 **React-Redux**는 특히 **React-Redux**가 내부적으로 **Context**를 사용하기 때문에 **prop-drilling**을 피하는 데 사용될 수 있다. `<MyContext.Provider>`에 새로운 값을 명시적으로 집어 넣는 것 대신에 **Redux** 스토어에 해당 데이터를 집어 넣고 이를 어디에서든 접근할 수도 있다.

### Purposes and Use Cases for (React-)Redux

The primary reason to use Redux is captured in the description from the Redux docs:

> **Redux**에서 제공하는 패턴과 도구는 애플리케이션 내 상태값을 언제, 어디서, 왜, 그리고 어떻게 업데이트 해야할 지 이해하기 쉽도록 만들어주고 이러한 업데이트 사항이 발생했을 때 애플리케이션 로직이 어떻게 동작해야 하는지 알려준다.

Redux를 사용하길 원하는 위 주요 이유를 제외하고도 몇 가지 이유들이 더 존재한다. "**prop-drilling을 피하기**"도 몇 가지 이유 중 하나에 해당된다. 많은 사람들이 일찍이 Redux를 선택한 이유, 특히 **prop-drilling**을 피하기 위해 **Redux**를 선택한 이유는 **React**의 **Legacy Context**는 제대로 동작하지 않았지만 **React-Redux**는 올바르게 동작했기 때문이다.

Other valid reasons to use Redux include:

- 상태 관리 로직을 완전히 UI 레이어와 분리하고 싶어하는 경우
- 다른 UI 레이어 간에 상태 관리 로직을 공유하고 싶은 경우(예를 들어, AngularJS에서 React로 마이그레이션 하는 경우)
- `action`이 `dispatch` 될 때 추가적인 로직을 더할 수 있는 **Redux** 미들웨어를 사용하는 경우
- **Redux** 상태값의 일부분을 유지하고 싶은 경우
- Enabling bug reports that can be replayed by developers
- 개발 중에 빠르게 로직과 UI를 디버깅하고 싶은 경우

Dan Abramov listed a number of these use cases when he wrote his post [You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367), all the way back in 2016.

## Why Context is Not "State Management"

"**State**"란 애플리케이션의 동작을 묘사한 모든 데이터를 뜻한다. 여기서 **State**를 "**server state**", "**communications state**" 그리고 "**location state**"와 같은 카테고리로 나눌 수 있다. 하지만 중요한 것은 저장, 읽기, 업데이트 및 사용 중인 데이터가 있다는 것이다.

[David Khourshid, author of the XState library and an expert on state machines, said](https://twitter.com/DavidKPiano/status/1347723896800346112):

> 상태 관리란 시간에 흐름에 따라 상태가 변경되는 방식을 의미한다.
>
> **_"State management is how state changes over time."_**

이를 기반으로 우리는 "**상태 관리**"란 다음과 같은 의미를 내포한다고 말할 수 있다.

- 초기값을 저장하고
- 현재 저장된 값을 읽으며
- 해당 값을 업데이트하는

일반적으로 현재 값이 변경되었을 때 알림을 받는 방법도 있다.

**React**의 `useState`와 `useRedcuer` 훅은 상태 관리의 좋은 예제이다.

두 가지 훅 모두 다음과 같은 일을 할 수 있다.

- 훅을 호출하여 초기값을 저장한다.
- 훅을 호출하여 현재 상태값을 읽는다.
- `setState` 또는 `dispatch` 함수를 호출하여 상태값을 업데이트한다.
- 컴포넌트가 리렌더링 됐기 때문에 상태값이 업데이트 됐다는 것을 알 수 있다.

마찬가지로 Redux와 MobX도 상태 관리를 할 수 있다.

- **Redux**는 **root reducer**를 호출하여 초기값을 저장하고 `store.getState()`를 통해 현재 상태앖을 읽으며 `store.dispatch(action)`을 통해 상태값을 업데이트 한다. 그리고 `store.subscribe(listener)`를 통하여 스토어가 업데이트 됐다는 것을 리스너들에게 알린다.
- **MobX**는 `store` 클래스에 필드 값을 할당하여 초기값을 저장하고 해당 스토어의 필드에 접근하여 현재 상태값을 읽으며 필드값들을 재할당함으로써 값을 업데이트한다. 그리고 `autorun()`과 `computed()`를 통하여 이러한 변경 사항들을 알린다.

**React-Query**, **SWR**, **Apollo** 그리고 **Urql**과 같은 서버 캐싱 도구들도 "**상태 관리**"의 정의에 적합하다고 말할 수도 있다. 이 도구들은 패칭된 데이터를 기반으로 초기 값을 저장하고 각 도구들이 제공하는 훅을 통하여 현재 상태값을 반환받으며 "**server mutation**"을 통하여 업데이트를 한다. 그리고 컴포넌트 리렌더링을 통하여 변경 사항을 알린다.

**React** **Context**는 이러한 기준을 충족하지 못한다. 그러므로 **Context**는 "상태 관리" 도구가 아니다!

위에서 설명했듯이 **Context**는 **Context** 자체로는 아무것도 "저장"하지 않는다. `<MyContext.Provider>`를 렌더링하는 상위 컴포넌트는 **Context**에 전달할 값이 무엇인지 결정하는 역할을 하며 일반적으로 전달하는 값은 **React** 컴포넌트의 상태값을 기반으로 한다. 실질적인 "**상태 관리**"는 `useState`/`useReducer` 훅에서 발생한다.

[As David Khourshid also said](https://twitter.com/DavidKPiano/status/1347723896800346112):

> **Context**는 어딘가에 이미 존재하는 상태값을 다른 컴포넌트들과 공유하기 위한 방법이다.
>
> **Context**는 상태 관리와 거의 관련이 없다.

Or, [as a recent tweet put it](https://twitter.com/CONNERtheBUSH/status/1347736489875140608):

> **Context**는 추상화된 상태값 보다는 숨겨진 `props`에 가깝다.
>
> **_I guess Context is more like hidden props than abstracted state._**

이렇게 생각해보자. 우리는 정확히 동일한 `useState`/`useReducer` 코드를 작성할 수 있었지만 컴포넌트 트리를 통하여 데이터와 업데이트 함수를 **prop-drilling** 했었다. 애플리케이션의 실제 동작은 전반적으로 동일했을 것이다. 모든 Context는 우리를 위해 우리가 **prop-drilling**을 건너뛸 수 있게 만들어 준다.

## Comparing Context and Redux

Context와 React+Redux가 실제로 가지고 있는 능력이 무엇인지 다시 짚어보자.

### Context

- 아무것도 저장 또는 "**관리**"하지 못한다.
- React 컴포넌트에서만 동작한다.
- 단일 값을 내려주되 해당 값은 무엇이든(원시값, 객체, 클래스 등) 될 수 있다.
- 내려준 단일 값을 읽을 수 있게 해준다.
- **prop-drilling**을 피할 수 있게 해준다.
- Provider와 Consumer의 현재 Context 값을 React DevTools에서 볼 수 있지만 해당 값이 시간에 흐름에 따라 어떻게 변화했는지에 대한 기록을 볼 순 없다.
- **Context** 값이 변경됐을 때 이를 구독하고 있던 컴포넌트들이 업데이트되는데 이 업데이트를 건너뛸 수 있는 방법은 없다.
- 사이드 이펙트를 위한 메커니즘을 포함하고 있지 않다. 이는 순전히 컴포넌트를 렌더링하기 위한 것이다.

### React+Redux

- 단일 값(일반적으로 해당 값은 객체다)을 저장하고 관리한다.
- React 컴포넌트 밖을 포함한 어떤 UI 레이어와도 동작한다.
- 단일 값을 읽을 수 있다.
- **prop-drilling**을 피할 수 있다.
- action을 dispatch하고 reducer의 실행을 통하여 값을 업데이트 할 수 있다.
- dispatch된 모든 action과 시간의 흐름에 따른 상태값에 대한 기록을 DevTools를 통해 살펴볼 수 있다.
- 미들웨어를 사용하면 애플리케이션 코드에서 사이드 이펙트를 `trigger` 할 수 있다.
- 컴포넌트들이 스토어의 업데이트 사항을 구독하고 스토어 상태값에서 특정 데이터만을 추출할 수 있도록 하며 오직 추출한 값이 변경됐을 때만 해당 컴포넌트들이 리렌더링된다.

따라서, 위 도구들은 다른 능력을 가진 다른 도구다. 위 도구들의 유일한 공통점이라고 한다면 "**prop-drilling**을 피하는데 사용될 수 있다는 점"뿐이다.

## Context and `useReducer`

"**Context vs Redux**" 논쟁의 한가지 문제점은 사람들이 실제로 말하는 바는 "**나는 useReducer를 사용하여 상태값을 관리하고 Context를 통해 해당 값을 내려주고 있다**"라는 것이다. 하지만 사람들은 절대 상태값을 명시적으로 언급하지 않는다. 그저 "**Context를 사용하고 있다**" 라고만 말한다. 이것이 필자가 보는 혼란의 원인이며 이러한 혼란이 **Context**가 "**상태를 관리한다**"라는 생각을 지속적으로 하게 만든다고 느껴서 정말 안타깝게 생각한다.

따라서, **Context** + `useReducer` 조합에 대해 구체적으로 알아보자. **Context** + `useReducer`는 **Redux** + **React-Redux**와 매우 유사하게 보인다. 두 조합 모두 가지고 있는 것들은 다음과 같다.

- 저장된 값
- `reducer` 함수
- `action`을 `dispatch`
- 중첩된 컴포넌트들에게 값을 내려주고 해당 값을 읽을 방법을 제공

하지만 여전히 **Context** + `useReducer`와 **Redux** + **React-Redux**의 능력과 동작에는 몇 가지 명확힌 차이점이 존재한다. I covered the key points in my posts [React, Redux, and Context Behavior](https://blog.isquaredsoftware.com/2020/01/blogged-answers-react-redux-and-context-behavior/) and [A (Mostly) Complete Guide to React Rendering Behavior](https://blog.isquaredsoftware.com/2020/05/blogged-answers-a-mostly-complete-guide-to-react-rendering-behavior/). Summarizing here:

- **Context** + `useReducer`는 **Context**를 통하여 현재 상태값을 전달하는 것에 의존한다. **React-Redux**는 **Context**를 통하여 현재 **Redux** 스토어 인스턴스를 전달한다.
- 이말인 즉슨 `useReducer`가 새로운 상태값을 생성할 때 해당 **Context**를 구독하고 있는 모든 컴포넌트들이 상태값의 일부를 사용하고 있을지라도 강제로 리렌더링을 발생시킨다. 이는 상태값의 크기, 해당 데이터를 얼마나 많은 컴포넌트가 구독하고 있는지 그리고 얼마나 해당 컴포넌트들이 자주 리렌더링되는 지에 따라 퍼포먼스 이슈로 이어질 수 있다. **React-Redux**는 스토어 상태값의 일부만도 구독할 수 있으며 해당 값들이 변경됐을 때만 리렌더링이 발생시킨다.

In addition, there's some other important differences as well:

- **Context** + `useRedcuer`는 **React**의 기능이기 때문에 **React**를 벗어나면 사용할 수가 없다. 그러나 **Redux** 스토어는 모든 **UI**와 독립적이기에 **React**에서 분리해서 사용할 수도 있다.
- React DevTools는 현재 Context의 값을 볼 수 있게 해주지만 시간에 흐름에 따른 값의 변화에 대한 기록은 볼 수가 없다. **Redux DevTools**는 `dispatch`된 모든 `action`, 각 `action`의 컨텐츠, `action`이 처리되고 난 후의 상태값의 상태 그리고 시간에 흐름에 따라 각 상태값 간의 차이를 알 수 있게 해준다.
- `useReducer`는 미들웨어가 없다. 사이드 이펙트에 관한 처리를 하려면 `useEffect`와 `useReducer`의 조합을 사용해야 한다. 미들웨어와 유사한 무언가로 `useReducer`를 래핑한 것도 본적이 있으나 이것들을 **Redux** 미들웨어의 기능과 능력에 비교했을 때 너무 제한 사항이 많았다.

It's worth repeating [what Sebastian Markbage (React core team architect) said about the uses for Context](https://github.com/facebook/react/issues/14110#issuecomment-448074060):

> 개인적으로 **Context**를 요약하자면 **Context**는 `locale`/`theme`과 같이 자주 업데이트되지 않는 요소와 함께 사용할 수 있다. 이전 **Context**를 사용하던 방식(예를 들어, 정적 값을 위해서 사용하거나 구독을 통한 업데이트 전파를 위해)을 그대로 사용하는 것도 좋다. 이는 Flux와 같은 상태 전파를 대체하기엔 충분하지 않다.

불필요한 리렌더링을 줄이고 스코프 문제를 해결하기 위해 서로 다른 **chunks of state**에 대해 여러 별도의 **Context**를 설정하도록 권장하는 포스팅이 많이 있다. Some of those also suggest [adding your own "context selector components"](https://saul-mirone.github.io/performance-optimization-in-react-context/), which require a mixture of `React.memo()`, `useMemo()`, and [carefully splitting things up so there's two separate contexts for each segment of state](https://kentcdodds.com/blog/how-to-use-react-context-effectively) (one for the data, and one for the updater functions). 물론 이런 식으로도 코드를 작성할 순 있겠지만 이 시점에서 이런 코드는 그저 **React-Redux**를 다시 만들고 있는 모양새가 될 뿐이다.

따라서, **Context** + `useReducer`는 얼핏 보기에 **Redux** + **React-Redux**와 유사하지만... 둘은 완전히 동일하지 않으며 **Redux**를 대체할 수 없다!

## Choosing the Right Tool

앞서 말했듯이 문제를 해결하는 올바른 도구를 올바르게 선택하려면 도구가 어떤 문제를 해결하는지 이해하고 어떤 문제가 있는지 아는 것이 중요하다.

### Use Case Summary

Let's recap the use cases for each of these:

- **Context**
  - **prop-drilling** 없이 중첩된 컴포넌트에 값을 전달하고 싶은 경우
- `useReducer`
  - `reducer` 함수를 사용한 적당히 복잡한 **React** 컴포넌트 상태를 관리하고 싶은 경우
- **Context** + `useReducer`
  - `reducer` 함수를 사용한 적당히 복잡한 **React** 컴포넌트 상태를 관리하며 **prop-drilling** 없이 중첩된 컴포넌트에 상태값을 전달하고 싶은 경우
- **Redux**
  - `reducer` 함수를 사용한 적당한 수준부터 복잡한 수준의 상태 관리
  - 시간 경과에 따라 상태값이 언제, 왜, 어떻게 변경되었는지 추적하고 싶은 경우
  - UI 레이어에서 상태 관리 로직을 완전히 분리하고 싶은 경우
  - 서로 다른 UI 레이어에서 상태 관리 로직을 공유하고 싶은 경우
  - `action`이 `dispatch` 됐을 때 **Redux** 미들웨어를 사용하여 로직을 추가하고 싶은 경우
  - Redux 상태의 일부를 유지하고 싶은 경우
  - Enabling bug reports that can be replayed by developers
  - 개발 중에 로직과 UI에 대한 빠른 디버깅
- **Redux** + **React-Redux**
  - **Redux**의 모든 **use-case**에 더해 **React** 컴포넌트에서 **Redux** 스토어와 상호 작용하고 싶은 경우

다시 한번 말하지만 이 둘은 다른 문제를 해결하기 위한 다른 도구들이다.

## Recommendations

So, how do you decide whether to use Context, Context + `useReducer`, or Redux + React-Redux?

You need to determine which of these tools best matches the set of problems that you're trying to solve!

- 만약 오직 prop-drilling을 피하고 싶은 거라면 Context를 사용하자.
- 만약 적당히 복잡한 **React** 컴포넌트 상태값을 가지고 있거나 외부 라이브러리를 사용하고 싶지 않은 경우 **Context** + `useReducer` 조합을 사용하자.
- 만약 시간의 흐름에 따른 상태 변화를 더 잘 추적하고 싶고 상태가 업데이트될 때 특정 컴포넌트만 리렌더링 시킬 필요가 있으며 사이드 이펙트 관리에 대한 더 강력한 능력이 필요한 경우 또는 비슷한 문제를 가지고 있을 때 **Redux** + **React-Redux** 조합을 사용하자.

개인적인 의견으로는 만약 애플리케이션에서 상태 관련 **Context**가 2-3개를 넘어가면 **React-Redux**의 `weaker` 버전을 다시 만들고 있는 상황이기에 **Redux** 사용하는 방식으로 전환해야 한다고 생각한다.

또 다른 걱정 중 하나는 "Redux를 사용하는 것이 너무 많은 boilerplate를 만들어 낸다"라는 것이다. "modern Redux"는 이전 Redux 보다 훨씬 배우기도 쉽고 사용하기도 쉽기 때문에 이러한 걱정들은 더 이상 큰 의미가 없다. [Our official Redux Toolkit package eliminates those "boilerplate" concerns](https://redux-toolkit.js.org/), and [the React-Redux hooks API simplifies using Redux in your React components](https://redux.js.org/tutorials/fundamentals/part-5-ui-react#reading-state-from-the-store-with-useselector). As [one user recently told me](https://www.reddit.com/r/javascript/comments/kr0ce6/future_of_state_management_in_react_with_xstate/gi9af94/):

> 프로덕션 애플리케이션의 프론트엔드 중 하나에서 기존 **Context**와 훅을 **RTK**로 전환했었다. That thing processes a little over $1B/year. Fantastic stuff in the toolkit. The RTK is the polish that helped me convince the rest of the teams to buy into the refactor. 리팩토링을 하면서 `boilerplate` 분석을 진행했었는데 이는 실제로 **Context**를 사용한 `dispatch` 패턴보다 **RTK**를 사용하는 것이 더 적은 `boilerplate`를 만들어 낸다는 사실을 알게 됐다. 데이터 처리가 얼마나 쉬운지는 말할 필요도 없다.

**RTK**와 **React-Redux**를 `dependency` 목록에 추가한다는 것이 **React**에 빌트인으로 내장된 **Context** + `useReducer` 조합을 사용하는 것보다 애플리케이션 번들 사이즈를 크게 만든다는 사실을 알고 있다. But, the tradeoffs are worth it. 더 나은 상태 추적 기능, 더 간단하고 예측 가능한 로직, 그리고 향상된 컴포넌트 렌더링 퍼포먼스

RTK와 React-Redux를 사용한다고 Context와 useReducer를 사용하지 못하는게 아니라는 것을 알아야 한다. Redux, Context 그리고 useReducer를 동시에 함께 사용할 수도 있다. We specifically encourage [putting "global state" in Redux and "local state" in React components](https://redux.js.org/tutorials/essentials/part-2-app-structure#component-state-and-forms), and [carefully deciding whether each piece of state should live in Redux or component state](https://redux.js.org/style-guide/style-guide#evaluate-where-each-piece-of-state-should-live). 따라서, 동일한 애플리케이션 내에서 동시에 전역 상태를 위해 **Redux**를 사용하고 로컬 상태를 위해서 `useReducer` + **Context** 조합을 사용하며 일부 반 정적인 값들을 대상으로 **Context**를 사용할 수도 있다.

모든 애플리케이션에서 Redux를 사용해야 하거나 Redux가 항상 최선의 선택이라는 것이 아니다. There's many nuances to this discussion. Redux는 유효한 선택이지며 Redux를 선택해야 하는 이유는 많다. and the tradeoffs for choosing Redux are a net win more often than many people think.

마지막으로 **Context**와 **Redux**만이 고려할 수 있는 유일한 선택지는 아니다. **Context**와 **Redux** 외에도 다른 방식으로 상태 관리 문제를 해결하려고 하는 많은 도구들이 존재한다. **MobX**는 데이터 의존성을 자동으로 업데이트 하기 위해 **OOP**와 `observable`을 사용하는 상태 관리 도구다. **Jotai**, **Recoil**, 그리고 **Zustand**는 더 가벼운 상태 업데이트 접근 방식을 제공한다. **React Query**, **SWR**, **Apollo** 그리고 **Urql**과 같은 데이터 패칭 라이브러리는 모두 캐싱된 서버 상태 작업을 위한 공통 패턴을 단순화하는 추상화를 제공한다. 다시 말하지만 이것들은 다른 목적과 **use-case**를 가진 다른 도구들이며 각자의 **use-case**를 기반으로 각 도구들을 평가할 필요가 있다.

## Final Thoughts

I realize that this post won't stop the seemingly never-ending debate over "Context vs Redux?!?!?!?!?". There's too many people out there, too many conflicting ideas, and too much miscommunication and misinformation.

Having said that, I hope that this post has clarified what these tools actually *do*, how they're different, and when you should actually consider using them. (And maybe, just *maybe*, some folks will read this article and not feel the need to post the same question that's been asked a million times already...)
