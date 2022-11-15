# useEffect 완벽 가이드

> 원본 글  
> https://overreacted.io/ko/a-complete-guide-to-useeffect/

**목차**

- [useEffect 완벽 가이드](#useeffect-완벽-가이드)
  - [요약본](#요약본)
    - [🤔 **질문: `useEffect` 로 `componentDidMount` 동작을 흉내내려면 어떻게 하지?**](#질문useeffect로componentdidmount동작을-흉내내려면-어떻게-하지)
    - [🤔 **질문: `useEffect` 안에서 데이터 페칭은 어떻게 해야할까? 두번째 인자로 오는 배열(`[]`) 은 뭐지?**](#질문useeffect안에서-데이터-페칭은-어떻게-해야할까-두번째-인자로-오는-배열-은-뭐지)
    - [🤔 **질문: 이펙트를 일으키는 의존성 배열에 함수를 명시해도 되는걸까?**](#질문-이펙트를-일으키는-의존성-배열에-함수를-명시해도-되는걸까)
    - [🤔 **질문: 왜 가끔씩 데이터 페칭이 무한루프에 빠지는걸까?**](#질문-왜-가끔씩-데이터-페칭이-무한루프에-빠지는걸까)
    - [🤔 **질문: 왜 가끔씩 이펙트 안에서 이전 state나 prop 값을 참조할까?**](#질문-왜-가끔씩-이펙트-안에서-이전-state나-prop-값을-참조할까)
  - [본론](#본론)
    - [우리는 `useEffect`를 왜 사용할까?](#우리는-useeffect를-왜-사용할까)
  - [모든 랜더링은 고유의 `Prop`과 `State`가 있다](#모든-랜더링은-고유의-prop과-state가-있다)
  - [모든 랜더링은 고유의 이벤트 핸들러를 가진다](#모든-랜더링은-고유의-이벤트-핸들러를-가진다)
  - [모든 랜더링은 고유의 이펙트를 가진다](#모든-랜더링은-고유의-이펙트를-가진다)
  - [모든 랜더링은 고유의… 모든 것을 가지고 있다](#모든-랜더링은-고유의-모든-것을-가지고-있다)
  - [그러면 클린업(cleanup)은 뭐지?](#그러면-클린업cleanup은-뭐지)
  - [리액트에게 이펙트를 비교하는 법을 가르치기](#리액트에게-이펙트를-비교하는-법을-가르치기)
  - [리액트에게 의존성으로 거짓말하지 마라](#리액트에게-의존성으로-거짓말하지-마라)
  - [의존성으로 거짓말을 하면 생기는 일](#의존성으로-거짓말을-하면-생기는-일)
  - [의존성을 솔직하게 적는 두 가지 방법](#의존성을-솔직하게-적는-두-가지-방법)
    - [1. deps](#1-deps)
    - [2. 의존성 제거](#2-의존성-제거)
  - [이펙트가 자급자족 하도록 만들기](#이펙트가-자급자족-하도록-만들기)
  - [액션을 업데이트로부터 분리하기](#액션을-업데이트로부터-분리하기)
  - [왜 useReducer가 Hooks의 치트 모드인가](#왜-usereducer가-hooks의-치트-모드인가)
  - [함수를 이펙트 안으로 옮기기](#함수를-이펙트-안으로-옮기기)
  - [하지만 저는 이 함수를 이펙트 안에 넣을 수 없어요](#하지만-저는-이-함수를-이펙트-안에-넣을-수-없어요)
  - [함수도 데이터 흐름의 일부인가?](#함수도-데이터-흐름의-일부인가)
  - [경쟁 상태에 대해](#경쟁-상태에-대해)
  - [진입 장벽을 더 높이기](#진입-장벽을-더-높이기)

## 요약본

### 🤔 **질문: `useEffect` 로 `componentDidMount` 동작을 흉내내려면 어떻게 하지?**

완전히 같진 않지만 `useEffect(fn, [])` 으로 가능하다. `componentDidMount` 와 달리 `prop`과 `state`를 잡아둘 것이다. 그래서 콜백 안에서도 초기 `prop`과 `state`를 확인할 수 있다. 만약 "**최신의**" 상태 등을 원한다면, `ref`에다 담아둘 수는 있다. 하지만 보통 더 간단하게 코드를 구조화하는 방법이 있기 때문에 굳이 이 방법을 쓸 필요도 없습니다. 이펙트를 다루는 멘탈 모델은 `componentDidMount` 나 다른 라이프사이클 메서드와 다르다는 점을 인지해야만 한다. 그리고 어떤 라이프사이클 메서드와 비슷한 동작을 하도록 만드는 방법을 찾으려고 하면 오히려 혼란을 더 키울 뿐이다. 더 생산적으로 접근하기 위해 "**이펙트 기준으로 생각해야(thinking in effects)**" 하며 이 멘탈 모델은 동기화를 구현하는 것에 가깝지 라이프사이클 이벤트에 응답하는 것과는 다르다.

### 🤔 **질문: `useEffect` 안에서 데이터 페칭은 어떻게 해야할까? 두번째 인자로 오는 배열(`[]`) 은 뭐지?**

[이 링크의 글이](https://www.robinwieruch.de/react-hooks-fetch-data/) `useEffect` 를 사용하여 데이터를 불러오는 방법을 파악하는데 좋은 기본서가 된다. 글을 꼭 끝까지 읽어볼 것! 지금 읽고 있는 글처럼 길지 않다. `[]` 는 이펙트에 리액트 데이터 흐름에 관여하는 어떠한 값도 사용하지 않겠다는 뜻이다. 그래서 한 번 적용되어도 안전하다는 뜻이기도 하다. 이 빈 배열은 실제로 값이 사용되어야 할 때 버그를 일으키는 주된 원인 중 하나다. 잘못된 방식으로 의존성 체크를 생략하는 것 보다 의존성을 필요로 하는 상황을 제거하는 몇 가지 전략을(주로 `useReducer`, `useCallback`) 익혀야 할 필요가 있다.

### 🤔 **질문: 이펙트를 일으키는 의존성 배열에 함수를 명시해도 되는걸까?**

추천하는 방법은 `prop`이나 `state`를 반드시 요구하지 않는 함수는 컴포넌트 바깥에 선언해서 호이스팅하고, 이펙트 안에서만 사용되는 함수는 이펙트 함수 내부에 선언하는 것이다. 그러고 나서 만약에 랜더 범위 안에 있는 함수를 이펙트가 사용하고 있다면 (`prop`으로 내려오는 함수 포함해서), 구현부를 `useCallback` 으로 감싸자. 왜 이런걸 신경써야 할까? 함수는 `prop`과 `state`로부터 값을 "**볼 수**" 있다. 그러므로 리액트의 데이터 플로우와 연관이 있다. [자세한 답변](https://reactjs.org/docs/hooks-faq.html#is-it-safe-to-omit-functions-from-the-list-of-dependencies)은 훅 FAQ 부분에 있다.

### 🤔 **질문: 왜 가끔씩 데이터 페칭이 무한루프에 빠지는걸까?**

이펙트 안에서 데이터 페칭을 할 때 두 번째 인자로 의존성 배열을 전달하지 않았을 때 생길 수 있는 문제다. 이게 없으면 이펙트는 매 랜더마다 실행된다. 그리고 `state`를 설정하는 일은 또 다시 이펙트를 실행한다. 의존성 배열에 항상 바뀌는 값을 지정해 두는 경우에도 무한 루프가 생길 수 있다. 하나씩 지워보면서 어느 값이 문제인지 확인할 수도 있지만, 사용하고 있는 의존 값을 지우는 일은(아니면 맹목적으로 `[]` 을 지정하는 것은) 보통 잘못된 해결법이다. 그 대신 문제의 근원을 파악하여 해결해야 한다. 예를 들어 함수가 문제를 일으킬 수 있다. 그렇다면 이펙트 함수 안에 집어넣거나, 함수를 꺼내서 호이스팅 하거나, `useCallback` 으로 감싸서 해결할 수 있다. 객체가 재생성되는 것을 막으려면 `useMemo` 를 비슷한 용도로 사용할 수 있다.

### 🤔 **질문: 왜 가끔씩 이펙트 안에서 이전 state나 prop 값을 참조할까?**

이펙트는 언제나 자신이 정의된 블록 안에서 랜더링이 일어날 때마다 `prop`과 `state`를 "**지켜봅니다**". 이렇게 하면 [버그를 방지할 수 있지만](https://overreacted.io/ko/how-are-function-components-different-from-classes/) 어떤 경우에는 짜증날 수 있다. 그럴 때는 명시적으로 어떤 값을 가변성 `ref`에 넣어서 관리할 수 있다(링크에 있는 글 말미에 설명되어 있다). 혹시 기대한 것과 달리 이전에 랜더링될 때의 `prop`이나 `state`가 보인다면, 아마도 의존성 배열에 값을 지정하는 것을 깜빡했을 것이다. 이 [린트 규칙](https://github.com/facebook/react/issues/14920)을 사용하여 그 값을 파악할 수 있도록 연습해 보자. 며칠 안으로 자연스레 몸에 밸 것이다. 또한 FAQ 문서에서 [이 답변 부분을](https://reactjs.org/docs/hooks-faq.html#why-am-i-seeing-stale-props-or-state-inside-my-function) 읽자.

## 본론

### 우리는 `useEffect`를 왜 사용할까?

특정 랜더링 시점의 `prop`와 `state`가 필요하기 때문이다. 상태가 업데이트 된 후의 `prop` 그리고 `state`를 기반으로 무언가를 실행하고 싶기 때문에 !

## 모든 랜더링은 고유의 `Prop`과 `State`가 있다

이펙트는 매 랜더링 후 실행된다. 즉, DOM 요소가 모두 그려진 후에 이펙트가 실행된다는 것이다.
그리고 개념적으로 컴포넌트 결과물의 일부로서 특정 랜더링 시점의 `prop`과 `state`를 **본다**

> **`state`를 업데이트할 때마다, 리액트는 컴포넌트를 호출한다.**

호출하는 과정 자체는 React가 호스트 객체부터 하위 컴포넌트까지 차례대로 쭉 새로운 상태를 기반으로 컴포넌트를 호출하여 기존 렌더링 상태를 버리고 새로운 렌더링을 진행하는 거지만 ..

다시 돌아와 여기서 중요한 점은 여느 특정 렌더링 시 그 안에 있는 `state`가 시간이 지난다고 변하는 게 아니고 컴포넌트가 다시 호출되고, 각각의 렌더링마다 격리된 고유의 `state` 값을 **보는** 것이다.

## 모든 랜더링은 고유의 이벤트 핸들러를 가진다

이벤트 핸들러도 동일하다. 특정 클릭 이벤트에 걸려 있는 핸들러에 내부에 있는 상태값(또는 `props`) 또한 매 랜더링마다 독립적인 값으로 존재한다. 즉, 함수 자체는 여러 번 호출되지만(렌더링마다 한번씩), 각각의 렌더링에서 함수 안의 상태값(또는 `props`)은 독립적인 값(특정 렌더링 시의 상태)으로 존재한다는 말이다.

리액트만이 특별이 이렇게 동작하는건 아니다. 보통의 함수도 같은 방식으로 동작한다.

```jsx
function sayHi(person) {
  const name = person.name;
  setTimeout(() => {
    alert('Hello, ' + name);
  }, 3000);
}

let someone = { name: 'Dan' };
sayHi(someone);

someone = { name: 'Yuzhi' };
sayHi(someone);

someone = { name: 'Dominic' };
sayHi(someone);
```

위 예제에서 외부의 `someone` 변수는 여러 번 재할당된다. (리액트 어딘가에서 현재의 컴포넌트 상태가 바뀔 수 있는 것처럼) 하지만 `sayHi` 내부에서, 특정 호출마다 `person`과 엮여있는 `name` 이라는 지역 상수가 존재합니다. **이 상수는 지역 상수이기 때문에, 각각의 함수 호출로부터 분리되어 있다.** **결과적으로 타임아웃 이벤트가 실행될 때마다 각자의 alert은 고유의 `name`을 기억하게 된다.**

특정 렌더링 시 그 내부에서 `props`와 `state`는 영원히 같은 상태로 유지된다. `props`와 `state`가 렌더링으로부터 분리되어 있다면, 이를 사용하는 어떠한 값(이벤트 핸들러를 포함하여)도 분리되어 있는 것이다. 이들도 마찬가지로 특정 렌더링에 **속해 있다.** 따라서 이벤트 핸들러 내부의 비동기 함수라 할지라도 같은 상태값(또는 props)을 **보게** 될 것이다.

## 모든 랜더링은 고유의 이펙트를 가진다

**변화하지 않는** 이펙트 안에서 상태값(또는 `props`)이 임의로 바꾼다는 뜻이 아니라 이펙트 함수 자체가 매 랜더링마다 별도로 존재한다.

각각의 이펙트 버전은 매번 렌더링에 **속한** 상태값(또는 `props`)을 본다.

리액트는 우리가 제공한 이펙트 함수를 기억해 놨다가 DOM의 변화를 처리하고 브라우저가 스크린에 DOM 요소를 그리고 난 뒤 실행한다.

따라서 비록 우리가 하나의 개념으로 이펙트를 이야기하고 있지만 사실 매 렌더링마다 다른 함수라는 뜻이다. 그리고 각각의 이펙트 함수는 그 렌더링에 **속한** `props`와 `state` **본다**.

개념적으로, 이펙트는 렌더링 결과의 일부라 생각할 수도 있다. (엄격하게 이야기하자면 물론 그렇지는 않다)
우리가 형성하고 있는 멘탈 모델 속에서 이펙트 함수는 이벤트 핸들러처럼 특정 렌더링에 속하는 함수라고 생각하면 된다.

## 모든 랜더링은 고유의… 모든 것을 가지고 있다

이제 이펙트는 매 렌더링 후 실행되며, 그리고 개념적으로 컴포넌트 결과물의 일부로서 특정 렌더링 시점의 `props`과 `state`를 **본다**는 것을 알았다.

당연하다고 생각할 수도 있겠지만 클래스 컴포넌트로 만들면 위에서 말한 방식과는 조금 다르게 동작한다. 예를 들어 다음과 같이 코드를 작성했을 경우엔 특정 렌더링 시점의 값이 아닌 언제나 최신의 값을 보게 된다.

```jsx
componentDidUpdate() {
  setTimeout(() => {
     console.log(`You clicked ${this.state.count} times`);
  }, 3000);
}
```

클로저를 사용해서 위 코드를 다음과 같이 변경하면 위에서 논의한대로 특정 렌더링 시점의 값을 볼 수 있게 된다.

```jsx
componentDidUpdate() {
	const count = this.state.count;
  setTimeout(() => {
     console.log(`You clicked ${this.state.count} times`);
  }, 3000);
}
```

이와 같은 문제는 `this` 와 클로저의 문제라기 보다는 리액트 자체에서 `this.state`가 최신 상태를 가리키도록 변경하기 때문이다.

그렇다면 함수형 컴포넌트에서 특정 렌더링 시점이 아닌 매번 최신 상태를 유지하는 값을 만들 수는 없는걸까?

제일 쉬운 방법은 `ref`를 이용하는 방법이다.

```jsx
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);
  useEffect(() => {
    // 변경 가능한 값을 최신으로 설정한다
    latestCount.current = count;
    setTimeout(() => {
      // 변경 가능한 최신의 값을 읽어 들인다
      console.log(`You clicked ${latestCount.current} times`);
    }, 3000);
  });
  // ...
```

위 예제와 같이 리액트로 어떠한 값을 직접 변경하는 것이 꺼림칙해 보일 순 있으나 리액트의 클래스 컴포넌트는 정확하게 위와 같은 식으로 `this.state`를 재할당하고 있다.

미리 잡아둔 `props` 및 `state`와는 달리 특정 콜백에서 `lastestCount.current`의 값을 읽어 들일 때 언제나 같은 값을 보장하지 않는다. 정의된 바에 따라 이 값은 언제나 변경할 수 있다. 그렇기 때문에 이런 사용 방법은 기본 동작이 아니며, 직접 가져다 써야 한다.

## 그러면 클린업(cleanup)은 뭐지?

어떤 이펙트는 클린업 단계를 가질 수도 있다. 본질적으로 클린업의 목적은 구독과 같은 이펙트를 **되돌리는** 것이다.

리액트는 브라우저가 페인트(페이지에 DOM 요소를 그리는 작업) 하고 난 뒤에야 이펙트를 실행한다. 이렇게 하여 대부분의 이펙트가 스크린 업데이트를 가로막지 않기 때문에 앱을 빠르게 만들어 준다. 마찬가지로 이펙트의 클린업도 미뤄진다. 이전 이펙트는 새 `prop`과 함께 리렌더링 되고 난 뒤에 클린업된다.

당연하지만 여전히 모든 이펙트를 매번 랜더링마다 실행하는 것은 효율이 떨어질 수 있다.

## 리액트에게 이펙트를 비교하는 법을 가르치기

우리는 이미 **DOM** 그 자체에 대해서는 배운 상태이다. 매번의 리렌더링마다 **DOM** 전체를 새로 그리는 것이 아니라, 리액트가 실제로 바뀐 부분만 **DOM**을 업데이트한다.

예를 들어, 아래의 컴포넌트를

```jsx
<h1 className="Greeting">Hello, Dan</h1>
```

다음과 같이 바꾼다면

```jsx
<h1 className="Greeting">Hello, Yuzhi</h1>
```

리액트는 두 객체를 비교한다.

```jsx
const oldProps = { className: 'Greeting', children: 'Hello, Dan' };
const newProps = { className: 'Greeting', children: 'Hello, Yuzhi' };
```

각각의 `prop`을 짚어보고 `children`이 바뀌어서 **DOM** 업데이트가 필요하다고 파악했지만 `className`은 그렇지 않다. 그래서 그저 아래의 코드만 호출된다.

```jsx
domNode.innerText = 'Hello, Yuzhi';
// domNode.className은 건드릴 필요가 없다
```

이펙트에도 이런 방법을 적용할 수가 있을까? 이펙트를 적용할 필요가 없다면 다시 실행하지 않는 것이 좋을테니 말이다.

예를 들어 아마 아래의 컴포넌트는 상태 변화 때문에 다시 렌더링 될 것이다.

```jsx
function Greeting({ name }) {
  const [counter, setCounter] = useState(0);

  useEffect(() => {
    document.title = 'Hello, ' + name;
  });

  return (
    <h1 className="Greeting">
      Hello, {name}
      <button onClick={() => setCounter(count + 1)}>Increment</button>
    </h1>
  );
}
```

하지만 이펙트는 상태값을 사용하지 않는다. 이펙트는 `document.title`과 `name prop`을 통기화하지만, `name prop`은 같다. `document.title`을 매번 카운터 값이 바뀔때마다 재할당하는 것은 그다지 이상적으로 보이지 않는다.

그렇다면 리액트가 그냥 이펙트를 비교하면 안되는 걸까?

```jsx
let oldEffect = () => {
  document.title = 'Hello, Dan';
};
let newEffect = () => {
  document.title = 'Hello, Dan';
};
// 리액트가 이 배열을 같은 배열이라고 인식할 수 있을까?
```

그럴 수는 없다. 리액트가 함수를 호출해보지 않고 함수가 어떤 일을 하는지 알아낼 수는 없다. 저 코드는 특정한 값을 포함하고 있는 것이 아니라 `name` prop에 있는 것을 담아왔을 뿐이다.

그래서 **특정한 이펙트가 불필요하게 다시 실행되는 것을 방지하고 싶다면** 의존성 배열(deps)을 `useEffect`의 인자로 전달할 수 있는 것이다.

```jsx
useEffect(() => {
  document.title = 'Hello, ' + name;
}, [name]);
```

이건 마치 우리가 리액트에게 `이봐, 네가 이 함수의 안을 볼 수 없는 것을 알고 있지만 렌더링 스코프에서 name 외의 값은 쓰지 않는다고 약속할게`라고 말하는 것과 같다.

현재와 이전 이펙트 발동 시 이 값들이 같다면 동기화할 것은 없으니 리액트는 이펙트를 스킵할 수 있다.

```jsx
const oldEffect = () => {
  document.title = 'Hello, Dan';
};
const oldDeps = ['Dan'];

const newEffect = () => {
  document.title = 'Hello, Dan';
};
const newDeps = ['Dan'];

// 리액트는 함수 안을 살펴볼 수 없지만, deps를 비교할 수 있다.
// 모든 deps가 같으므로, 새 이펙트를 실행할 필요가 없다.
```

렌더링 사이에 의존성 배열 안에 있는 값이 하나라도 다르다면 이펙트를 스킵할 수 없다. 모든 것을 동기화해야 한다! 👊

## 리액트에게 의존성으로 거짓말하지 마라

의존성에 대해 리액트에게 거짓말을 할 경우 좋지 않은 결과를 가져온다.

```jsx
function SearchResults() {
  async function fetchData() {
    // ...
  }

  useEffect(() => {
    fetchData();
  }, []); // 이게 맞을까요? 항상 그렇진 않지요. 그리고 더 나은 방식으로 코드를 작성하는 방법이 있습니다.

  // ...
}
```

위 예제는 직관적으로 봤을 때 말은 되지만 정확히는 이펙트 훅의 규칙을 어기고 있는 상황이다.

물론 `마운트 될 때만 이펙트를 실행`하고 싶을 수도 있다.

일단 지금은 deps를 지정한다면, 컴포넌트에 있는 모든값 중 그 이펙트에 사용될 값은 반드시 deps 내에 있어야 한다는 것을 기억하자. `props`, `state`, `함수들` 등 컴포넌트 안에 있는 모든 것이 말이다.

위 예제처럼 코드를 작성하게 되면 가끔 문제가 일어난다. 무한 루프에 빠진다거나 소켓이 너무 자주 반응한다거나 .. 해결책을 알아보기 전에 앞서 말한 문제들이 왜 일어나는지에 대해 먼저 살펴보자.

## 의존성으로 거짓말을 하면 생기는 일

만약 `deps`가 이펙트에 사용하는 모든 값을 가지고 있다면, 리액트는 언제 다시 이펙트를 실행해야 할 지 알고 있다. 하지만 만약 이펙트에 `[]`를 넘겨주었다면, 새 이펙트 함수는 실행되지 않을 것이다.

의도에 맞게 사용하면 되는게 아닌가? 라고 생각할 수도 있지만 다음의 예제를 한번 살펴보자.

예를 들어 매 초마다 숫자가 올라가는 카운터를 작성한다고 해보자. 클래스 컴포넌트의 개념을 적용했을 때 우리의 직관은 `인터벌을 한 번만 설정하고, 한 번만 제거하자` 가 된다. 이펙트를 사용한 코드로 방금의 생각을 그대로 옮기게 되면 `deps`에 `[]`를 넣게 될 것이다.

`한번만 실행됐으면 좋겠다.` 라고 말이다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return <h1>{count}</h1>;
}
```

**하지만 위 예제의 숫자는 오직 한 번만 증가한다.**

말이 안되는 결과야! 라고 생각할 수도 있겠지만 의존성 배열이 리액트에게 어떤 랜더링 스코프에서 나온 값 중 이펙트에 쓰이는 것 **전부**를 알려주는 힌트라고 인식한다면 말이 된다. `count` 를 사용하지만 deps 를 `[]` 라고 정의하면서 거짓말을 했다.

첫 번째 랜더링에서 `count` 는 `0`이다. 따라서 첫 번째 랜더링의 이펙트에서 `setCount(count + 1)`는 `setCount(0 + 1)`이라는 뜻이 된다. **deps 를 `[]` 라고 정의했기 때문에 이펙트를 절대 다시 실행하지 않고, 결국 그로 인해 매 초마다 `setCount(0 + 1)` 을 호출하는 것이다.**

```jsx
// 첫 번째 랜더링, state는 0
function Counter() {
  // ...
  useEffect(
    // 첫 번째 랜더링의 이펙트
    () => {
      const id = setInterval(() => {
        setCount(0 + 1); // 언제나 setCount(1)
      }, 1000);
      return () => clearInterval(id);
    },
    [] // 절대 다시 실행하지 않는다
  );
  // ...
}

// 매번 다음 랜더링마다 state는 1이다
function Counter() {
  // ...
  useEffect(
    // 이 이펙트는 언제나 무시될 것
    // 왜냐면 리액트에게 빈 deps를 넘겨주는 거짓말을 했기 때문
    () => {
      const id = setInterval(() => {
        setCount(1 + 1);
      }, 1000);
      return () => clearInterval(id);
    },
    []
  );
  // ...
}
```

위 예제는 리액트에게 이 이펙트는 컴포넌트 안에 있는 값을 쓰지 않는다고 거짓말을 한 상태이다. 실제로는 쓰는데도 말이다.

> **이런 거짓말을 방지하려면 린트를 이용해 이러한 규칙을 강제하자.**

## 의존성을 솔직하게 적는 두 가지 방법

### 1. deps

첫 번째 방법은 컴포넌트 안에 있으면서 이펙트에서 사용되는 모든 값이 의존성 배열 안에 포함되도록 하는 것이다. 이러면 이펙트 내에서 사용되는 모든 값은 매 렌더링 마다 고유한 값을 정상적으로 갖게 될 것이다.

### 2. 의존성 제거

두 번째 방법은 이펙트의 코드를 바꿔서 우리가 원하던 것 보다 자주 바뀌는 값을 요구하지 않도록 만드는 것이다. 의존성에 대해 거짓말을 하자는게 아니라 그냥 의존성을 더 `적게` 넘겨주도록 바꾸는 것이다.

## 이펙트가 자급자족 하도록 만들기

우리는 앞선 예제의 이펙트 deps에서 `count`를 제거하도록 만들고 싶다.

문제를 해결하기 위해 먼저 우리 스스로에게 질문을 해보자. 무엇 때문에 `count`를 쓰고 있는가? 오로지 `setCount`를 위해 사용하는 있지 않은가? 이 경우에 스코프 안에서 count를 쓸 필요가 전혀 없다. 이전 상태를 기준으로 상태 값을 업데이트 하고 싶을 때는, `setState`에 **함수형 업데이터**를 사용하면 되기 때문이다.

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount((c) => c + 1);
  }, 1000);
  return () => clearInterval(id);
}, []);
```

이런 경우는 잘못된 의존관계 라고 생각할 수 있다. `count`는 우리가 `setCount(count + 1)`이라고 썼기 때문에 이펙트 안에서 필요한 의존성이었다. 하지만 진짜로 우리는 `count`를 `count + 1`로 변환하여 리액트에게 `돌려주기` 위해 원했을 뿐이다. 하지만 리액트는 현재의 `count`를 **이미 알고 있다.** 우리가 리액트에게 알려줘야 하는 것은 지금 값이 뭐든 간에 상태 값을 하나 더하라는 것이다.

그게 정확히 `setCount(c ⇒ c + 1)`이 의도하는 것이다. 리액트에게 상태가 어떻게 바뀌어야 하는지 `지침을 보내는 것`이라고 생각할 수도 있다.

방금의 과정으로 **우리는 의존성을 제거하지 않고도 실제로 문제를 해결했다는 것**에 주목해야 한다. 꼼수를 쓰지 않았다. 이펙트는 더 이상 렌더링 스코프에서 `count` 값을 읽어 들이지 않는다.

**`setCount(c => c + 1)` 조차도 그리 좋은 방법은 아니다.** 좀 이상해 보이기도 하고 할 수 있는 일이 굉장히 제한적이다. 예를 들어 서로에게 의존하는 두 상태 값이 있거나 prop 기반으로 다음 상태를 계산할 필요가 있을 때는 도움이 되지 않는다. 다행히도 `setCount(c => c + 1)`은 더 강력한 자매 패턴이 있다. 바로 `useReducer`다.

## 액션을 업데이트로부터 분리하기

앞선 예제를 `count`와 `step` 두 가지 상태 변수를 가지는 것으로 바꿔보자. 인터벌은 `step` 입력값에 따라 `count` 값을 더할 것이다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [step, setStep] = useState(1);

  useEffect(() => {
    const id = setInterval(() => {
      setCount((c) => c + step);
    }, 1000);
    return () => clearInterval(id);
  }, [step]);
  return (
    <>
      <h1>{count}</h1>
      <input value={step} onChange={(e) => setStep(Number(e.target.value))} />
    </>
  );
}
```

이펙트 안에서 `step`을 사용하고 있기 때문에 의존성 배열에 `step`을 추가해둔 상태이다. 이 예제의 현재 동작은 `step`이 변경되면 인터벌을 다시 시작하는 것이다.

하지만 `step`이 바뀐다고 인터벌 시계가 초기화되지 않는 것을 원한다면 어떻게 될까? 이펙트의 의존성 배열에서 `step`을 제거하려면 어떻게 해야할까?

어떤 상태 변수가 다른 상태 변수의 현재 값에 연관되도록 설정하려고 한다면, 두 상태 변수 모두 `useReducer`로 교체해야 할 수도 있다.

`setSomething(something ⇒ ...)` 같은 코드를 작성하고 있다면, 대신 리듀서를 써보는 것을 고려하기 좋은 타이밍이다. 리듀서는 컴포넌트에서 일어나는 `액션`의 표현과 그 반응으로 상태가 어떻게 업데이트되어야 할지를 분리한다.

이펙트 안에서 `step` 의존성을 `dispatch`로 바꿔보자.

```jsx
const [state, dispatch] = useReducer(reducer, intialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' });
  }, 1000);
  return () => clearInterval(id);
}, [dispatch]);
```

`이게 도데체 뭐가 더 좋냐`고 물어볼 수도 있다. 리액트는 컴포넌트가 유지되는한 **`dispatch`** 함수가 항상 같다는 것을 보장한다. 따라서 위의 예제에서 인터벌을 다시 구독할 필요조차 없다.

> 리액트가 `dispatch`, `setState`, `useRef` 컨테이너 값이 항상 고정되어 있다는 것을 보장하니까 의존성 배열에서 뺄 수도 있다. 하지만 명시한다고 해서 나쁠 것은 없다. 👍

이팩트 안에서 상태를 읽는 대신 `무슨 일이 일어났는지` 알려주는 정보를 인코딩하는 `액션`을 디스패치한다. 이렇게 하여 이펙트는 `step` 상태로부터 분리되어 있게 된다. 이펙트는 `어떻게` 상태를 업데이트 할지 신경쓰지 않고, 단지 `무슨 일이 일어났는지` 알려준다. 그리고 리듀서가 업데이트 로직을 모아둔다.

```jsx
const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```

## 왜 useReducer가 Hooks의 치트 모드인가

우리는 앞서 이펙트가 이전 상태를 기준으로 상태를 설정할 필요가 있을 때 어떻게 의존성을 제거하는지 살펴보았다. 하지만 다음 상태를 계산하는데 `props`가 필요하다면 어떨까? 예를 들어 API가 `<Count step={1} />` 라면? 당연하지만 이럴 때 의존성으로 `props.step`을 설정하는 것을 피할 순 없을 것이다.

아니 ! 사실은 피할 수 있다. `리듀서 그 자체`를 컴포넌트 안에 정의하여 `props`를 읽도록 하면 된다.

```jsx
function Counter({ step }) {
  const [count, dispatch] = useReducer(reducer, 0);

  function reducer(state, action) {
    if (action.type === 'tick') {
      return state + step;
    } else {
      throw new Error();
    }
  }

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);

  return <h1>{count}</h1>;
}
```

이 패턴은 몇 가지 최적화를 무효화하기 때문에 어디서나 쓰는 것이 권장되진 않는다. 하지만 필요하다면 이렇게 리듀서 안에서 `props`에 접근할 수 있다.

이 경우조차 렌더링간 `dispatch`의 동일성은 여전히 보장된다. 그래서 원한다면 이펙트의 의존성 배열에서 빼버릴 수도 있다. 이펙트가 재실행되도록 만들지 않을테니까.

아마 이렇게 생각할 수도 있다. **"이게 어떻게 가능하지?" 어떻게 다른 렌더링에 포함된 이펙트 안에서 호출된 리듀서가 "props"를 "알고" 있지?** 답은 `dispatch`를 할 때에 있다. 리액트는 그저 `액션을 기억해 놓는다.`하지만 다음 렌더링 중에 리듀서를 `호출할` 것이다. 이 시점에서 새`props`가 스코프 안으로 들어오고 이펙트 내부와는 상관이 없게 된다.

이래서 `useReducer`를 `Hooks`의 `치트 모드`라고 생각할 수 있는 것이다. 업데이트 로직과 그로 인해 무엇이 일어나는지 서술하는 것을 분리할 수 있도록 만들어준다. 그 다음은 이펙트의 불필요한 의존성을 제거하여 필요할 때보다 더 자주 실행되는 것을 피할 수 있도록 도와준다.

## 함수를 이펙트 안으로 옮기기

리액트를 사용하면서 흔한 실수 중 하나는 함수는 의존성에 포함되면 안된다는 것이다. 예를 들어 다음 코드는 동작하는 것처럼 보인다.

```jsx
function SearchResults() {
  const [data, setData] = useState({ hits: [] });

  async function fetchData() {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=react',
    );
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []); // 이거 괜찮은가?
  // ...
```

일단 위 코드는 동작한다. 하지만 간단히 로컬 함수를 의존성에서 제외하는 해결책은 컴포넌트가 커지면서 모든 경우를 다루고 있는지 보장하기 아주 힘들다는 문제가 있다.

각 함수가 5배 정도는 커져서 코드를 다음과 같은 방식으로 나누었다고 가정해보자.

```jsx
function SearchResults() {
  // 이 함수가 길다고 상상해 봅시다
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=react';
  }

  // 이 함수도 길다고 상상해 봅시다
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```

이제 나중에 이 함수들 중에 하나가 `state`나 `prop`을 사용한다고 생각해보자.

```jsx
function SearchResults() {
  const [query, setQuery] = useState('react');

  // 이 함수가 길다고 상상해 봅시다
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  // 이 함수가 길다고 상상해 봅시다
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []);

  // ...
}
```

만약 이런 함수를 사용하는 어떠한 이펙트에도 `deps`를 업데이트하는 것을 깜빡했다면, 이펙트는 `prop`과 `state`의 변화에 동기화하는데 실패할 것입니다. 이건 좋지 못하다.

다행히도, 이 문제를 해결할 쉬운 방법이 있다. 어떠한 함수를 이펙트 `안에서만` 쓴다면, 그 함수를 직접 이펙트 안으로 옮기는 방법이다.

```jsx
function SearchResults() {
  // ...
  useEffect(() => {
    // 아까의 함수들을 안으로 옮겼어요!
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=react';
    }
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }
    fetchData();
  }, []); // ✅ Deps는 OK
  // ...
}
```

이러면 뭐가 좋아지는가? 우리는 더 이상 `옮겨지는 의존성`에 신경 쓸 필요가 없다. 의존성 배열은 더 이상 거짓말을 하지 않는다. 진짜로 이펙트 안에서 컴포넌트의 범위 바깥에 있는 그 어떠한 것도 `사용하지 않고` 있다.

나중에 `getFetchUrl`을 수정하고 `query` state를 써야한다고 하면, 이펙트 안에 있는 함수만 고치면 되는 것을 쉬이 발견할 수 있다. 거기에 더해 `query`를 이펙트의 의존성으로 추가해야한다.

```jsx
function SearchResults() {
  const [query, setQuery] = useState('react');

  useEffect(() => {
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=' + query;
    }

    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, [query]); // ✅ Deps는 OK
  // ...
}
```

이 의존성을 더하는 것이 단순히 `리액트를 달래는` 것은 아니다. `query`가 바뀔 때 데이터를 다시 패칭하는 것이 `말이 된다.` `useEffect`의 디자인은 사용자가 제품을 사용하다 겪을 때까지 무시하는 대신, 데이터 흐름의 변화를 알아차리고 이펙트가 어떻게 동기화해야할지 선택하도록 강제한다.

## 하지만 저는 이 함수를 이펙트 안에 넣을 수 없어요

때때로 함수를 이펙트 `안에` 옮기고 싶지 않을 수도 있다. 예를 들어 한 컴포넌트에서 여러개의 이펙트가 있는데 같은 함수를 호출할 때, 로직을 복붙하고 싶진 않을 것이다. 아니면 prop 때문이거나.

이런 함수를 이펙트의 의존성으로 정의하지 말아야 할까? 아니 ! 그렇게 생각하지 않는다. 다시 말하지만, **이펙트는 자신의 의존성에 대해 거짓말을 하면 안된다.** 보틍은 더 나은 해결이 있다. 흔한 오해 중 하나가 `함수는 절대 바뀌지 않는다` 이다. 하지만 이 글을 통해 배웠듯, 그 말이 진실로부터 얼마나 멀리 떨어져 있는지 알 수 있다. 사실은 컴포넌트 안에 정의된 함수는 매 렌더링마다 바뀐다!

하지만 그로 인해 문제가 발생한다. 두 이펙트가 `getFetchUrl`을 호출한다고 생각해보자.

```jsx
function SearchResults() {
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // 🔴 빠진 dep: getFetchUrl

  // ...
}
```

이 경우 `getFetchUrl`을 각각의 이펙트 안으로 옮기게 되면 로직을 공유할 수 없으니 그다지 달갑지 않다. 두 이펙트 모두(매 렌더링마다 바뀌는) `getFetchUrl`에 기대고 있으니, 의존성 배열도 쓸모가 없다.

```jsx
function SearchResults() {
  // 🔴 매번 랜더링마다 모든 이펙트를 다시 실행한다
  function getFetchUrl(query) {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // 🚧 Deps는 맞지만 너무 자주 바뀐다

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // 🚧 Deps는 맞지만 너무 자주 바뀐다

  // ...
}
```

구미가 당기는 해결책은 그냥 `getFetchUrl` 함수를 의존성 배열에서 빼 버리는 것이다. 하지만 좋은 방법이라는 생각은 들지 않는다. 이렇게 하면 우리가 언제 이펙트에 의해 다루어질 `필요가 있는` 데이터 흐름에 변화를 `더해야 할지` 알아차리기 어려워진다.

대신 더 간단한 해결책이 두 개 있다.

먼저, 함수가 컴포넌트 스코프 안의 어떠한 것도 사용하지 않는다면, 컴포넌트 외부로 끌어올려두고 이펙트 안에서 자유롭게 사용하면 된다.

```jsx
// ✅ 데이터 흐름에 영향을 받지 않는다
function getFetchUrl(query) {
  return 'https://hn.algolia.com/api/v1/search?query=' + query;
}
function SearchResults() {
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // ✅ Deps는 OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, []); // ✅ Deps는 OK

  // ...
}
```

저 함수는 렌더링 스코프에 포함되어있지 않으며 데이터 흐름에 영향을 받을 수 없기 때문에 `deps`에 명시할 필요가 없다. 우연히라도 `props`나 `state`를 사용할 수 없다.

혹은 `useCallback` 훅으로 감쌀 수도 있다.

```jsx
function SearchResults() {
  // ✅ 여기 정의된 deps가 같다면 항등성을 유지한다
  const getFetchUrl = useCallback((query) => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []); // ✅ 콜백의 deps는 OK
  useEffect(() => {
    const url = getFetchUrl('react');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK

  // ...
}
```

`useCallback`은 의존성 체크에 레이어를 하나 더 더하는 것이다. 달리 말해 문제를 다른 방식으로 해결하는데, **함수의 의존성을 피하기보다 함수 자체가 필요할 때만 바뀔 수 있도록 만드는 것**이다.

이 접근 방식이 왜 유용한지 살펴보자. 아까 예제는 `react`, `redux`라는 두 가지 검색 결과를 보여주었다. 하지만 입력을 받는 부분을 추가하여 임의의 `query`를 검색할 수 있다고 해보자. 그래서 `query`를 인자로 받는 대신, `getFetchUrl`이 지역 상태로부터 이를 읽어들인다.

그렇게 수정하면 즉시 query 의존성이 빠져있다는 사실을 파악하게 된다.

```jsx
function SearchResults() {
  const [query, setQuery] = useState('react');
  const getFetchUrl = useCallback(() => {
    // No query argument
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, []); // 🔴 빠진 의존성: query
  // ...
}
```

`useCallback`의 deps에 `query`를 포함하도록 고치면, `getFetchUrl`을 사용하는 어떠한 이펙트라도 `query`가 바뀔 때마다 다시 실행될 것이다.

```jsx
function SearchResults() {
  const [query, setQuery] = useState('react');

  // ✅ query가 바뀔 때까지 항등성을 유지한다
  const getFetchUrl = useCallback(() => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, [query]); // ✅ 콜백 deps는 OK
  useEffect(() => {
    const url = getFetchUrl();
    // ... 데이터를 불러와서 무언가를 한다 ...
  }, [getFetchUrl]); // ✅ 이펙트의 deps는 OK

  // ...
}
```

`useCallback` 덕분에 `query`가 같다면, `getFetchUrl` 또한 같을 것이며, 이펙트는 다시 실행되지 않을 것이다. 하지만 `query`가 바뀐다면, `getFetchUrl` 또한 바뀌며, 데이터를 다시 패칭할 것이다. 마치 엑셀 스프레드시트에서 어떤 셀을 바꾸면 다른 셀이 자동으로 다시 계산되는 것과 비슷하다.

이것은 그저 데이터 흐름과 동기화에 대한 개념을 받아들인 결과이다. 부모로부터 함수 `prop`을 내려보내는 것 또한 같은 해결책이 적용된다.

```jsx
function Parent() {
  const [query, setQuery] = useState('react');

  // ✅ query가 바뀔 때까지 항등성을 유지한다
  const fetchData = useCallback(() => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + query;
    // ... 데이터를 불러와서 리턴한다 ...
  }, [query]); // ✅ 콜백 deps는 OK
  return <Child fetchData={fetchData} />;
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ 이펙트 deps는 OK

  // ...
}
```

`fetchData`는 오로지 `Parent`의 `query` 상태가 바뀔 때만 변하기 때문에, `Child` 컴포넌트는 앱에 꼭 필요할 때가 아니라면 데이터를 다시 패칭하지 않을 것이다.

## 함수도 데이터 흐름의 일부인가?

흥미롭게도 이 패턴은 클래스 컴포넌트에서 사용하면 제대로 동작하지 않는데, 이게 이펙트와 라이프 사이클 패러다임의 결정적인 차이를 보여준다. 위의 코드를 클래스 컴포넌트로 치환해 봤다고 치자.

```jsx
class Parent extends Component {
  state = {
    query: 'react',
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... 데이터를 불러와서 무언가를 한다 ...
  };
  render() {
    return <Child fetchData={this.fetchData} />;
  }
}

class Child extends Component {
  state = {
    data: null,
  };
  componentDidMount() {
    this.props.fetchData();
  }
  render() {
    // ...
  }
}
```

아마도 이런 생각을 할 수 있다. "`useEffect`는 `componentDidMount`와 `componentDidUpdate`가 섞인 것이다." 하지만 이 로직은 `componentDidUpdate`에선 동작하지 않는다.

```jsx
class Child extends Component {
  state = {
    data: null,
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    // 🔴 이 조건문은 절대 참이 될 수 없다
    if (this.props.fetchData !== prevProps.fetchData) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

당연히도 `fetchData`는 클래스 메서드다. 아니면 클래스 속성이라고 말할 수 있지만 딱히 달라지는 것은 없다. `state`가 바뀌었다고 저 메서드가 달라지지 않는다. 따라서 `this.props.fetchData`는 `prevProps.fetchData`와 같기 때문에 절대 다시 데이터를 패칭하지 않는다. 그렇다면 아까 조건문을 제거하면 어떻게 될까?

```jsx
componentDidUpdate(prevProps) {
  this.props.fetchData();
}
```

음? 뭔가 이상하지 않은가? 위 코드는 처음 의도와 벗어나게 된다. 이렇게 하면 `매번` 다시 렌더링을 할 때마다 데이터를 불러오게 된다. 혹시 특정한 `query`를 바인딩해두면 어떨까?

```jsx
render() {
  return <Child fetchData={this.fetchData.bind(this, this.state.query)} />;
}
```

하지만 이렇게 하면 `query`가 바뀌지 않았는데도 `this.props.fetchData ≠ prevProps.fetchData`는 언제나 `true`가 될 것이다. 결국 매번 데이터를 다시 패칭하게 될 것이다.

진짜 클래스 컴포넌트로 이 문제를 해결하는 방법은 눈 딱 감고 `query` 자체를 `Child` 컴포넌트에 넘기는 것 뿐이다. `Child` 컴포넌트가 `query`를 직접 사용하지 않음에도 불구하고 `query`가 바뀔 때 다시 데이터를 불러오는 로직은 해결할 수 있다.

```jsx
class Parent extends Component {
  state = {
    query: 'react',
  };
  fetchData = () => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + this.state.query;
    // ... 데이터를 불러와서 무언가를 한다 ...
  };
  render() {
    return <Child fetchData={this.fetchData} query={this.state.query} />;
  }
}

class Child extends Component {
  state = {
    data: null,
  };
  componentDidMount() {
    this.props.fetchData();
  }
  componentDidUpdate(prevProps) {
    if (this.props.query !== prevProps.query) {
      this.props.fetchData();
    }
  }
  render() {
    // ...
  }
}
```

클래스 컴포넌트에서, 함수 `prop` 자체는 실제로 데이터 흐름에서 차지하는 부분이 없다. 메소드는 가변성이 있는 `this` 변수에 묶여 있기 때문에 함수의 일관성을 담보할 수 없게 된다.

> 자바스크립트 `this`의 특성대로 메서드 내에서 참조하는 `this`는 해당 메서드를 호출하는 객체에 따라 달라지기 때문에 일관성을 보장할 수 없다.

그러므로 우리가 함수만 필요할 때도 차이를 비교하기 위해 온갖 다른 데이터를 전달해야만 한다. 부모 컴포넌트로부터 내려온 `this.prop.fetchData`가 **어떤 상태에 기대고 있는지, 아니면 그냥 상태가 바뀌기만 한 것인지 알 수가 “없다”.**

`useCallback`을 사용하면, 함수는 명백하게 데이터 흐름에 포함된다. 만약 함수의 입력값이 바뀌면 함수 자체가 바뀌고, 만약 그렇지 않다면 같은 함수로 남아있다고 말할 수 있다. 세밀하게 제공되는 `useCallback` 덕분에 `props.fetchData` 같은 props 변화는 자동적으로 하위 컴포넌트로 전달된다.

비슷하게, `useMemo` 또한 복잡한 객체애 대해 같은 방식의 해결책을 제공한다.

```jsx
function ColorPicker() {
  // color가 진짜로 바뀌지 않는 한
  // Child의 얕은 props 비교를 깨트리지 않는다
  const [color, setColor] = useState('pink');
  const style = useMemo(() => ({ color }), [color]);
  return <Child style={style} />;
}
```

그렇다고 `useCallback`을 어디든지 사용하는 것은 꽤 투박한 방법이라고 말할 수 있다. `useCallback`은 꽤 좋은 돌파구이며 함수가 전달되어 자손 컴포넌트의 이펙트 안에서 호출되는 경우 유용하다. 아니면 자손 컴포넌트의 메모이제이션이 깨지지 않도록 방지할 때도 쓰인다. 하지만 훅 자체가 콜백을 내려보는 것을 피하는 더 좋은 방법을 함께 제공한다.

위의 예제라면 `fetchData`를 이펙트 안에 두거나(커스텀 훅으로 추상화될 수도 있다) 최상위 레벨로 만들어 `import` 하는 것도 하나의 방법이다. 우리는 보통 이펙트를 최대한 단순하게 만들고 싶어할 텐데 그 안에 콜백을 두는 것은 단순하게 만드는데 별로 도움이 되지 않는다.

> 만약에 어떤 `props.onComplete` 콜백이 요청이 보내지고 있을 때 실행된다면?

클래스 컴포넌트의 동작을 흉내낼순 있지만 `경쟁 상태`를 해결할 수 는 없다.

## 경쟁 상태에 대해

클래스로 데이터를 불러오는 전통적인 예제는 다음과 같이 생겼다.

```jsx
class Article extends Component {
  state = {
    article: null,
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

위 예제를 보면 알겠지만 해당 예제는 컴포넌트가 업데이트되는 상황을 다루지 않는다. 그럼 두 번째 클래스 컴포넌트 예제를 한번 봐보자.

```jsx
class Article extends Component {
  state = {
    article: null,
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

아까보다는 나아졌다. 하지만 여전히 문제가 있다. 문제가 발생하는 이유는 순서를 보장할 수 없기 때문이다. 그래서 만약 우리가 `{ id: 10 }`으로 데이터를 요청하고 `{ id: 20 }`으로 바꾸었다면, `{ id: 20 }`의 요청이 먼저 시작된다. 그래서 먼저 시작된 요청이 더 늦게 끝나서 잘못된 상태를 덮어씌울 수 있다.

이를 경쟁 상태라고 하며, (보통 비동기 호출의 결과가 돌아올 때까지 기다린다고 여기며) 위에서 아래로 데이터가 흐르면서 `async` / `await`이 섞여 있는 코드에 흔히 나타난다. 여기서 데이터가 흘러간다는 말은 `props`나 `state`가 `async` 함수 안에서 바뀔 수 있다는 이야기이다.

이펙트가 마법같이 이 문제를 해결해주진 않는다. 거기다 `async` 함수를 이펙트에 직접적으로 전달하면 경고가 나타난다.

취소 기능을 지원하여 클린업 함수에서 바로 비동기 함수를 취소시키거나 `boolean` 값으로 흐름이 멈춰야 하는 타이밍을 조절하는 방법이 있긴 하다.

```jsx
function Article({ id }) {
  const [article, setArticle] = useState(null);

  useEffect(() => {
    let didCancel = false;
    async function fetchData() {
      const article = await API.fetchArticle(id);
      if (!didCancel) {
        setArticle(article);
      }
    }

    fetchData();

    return () => {
      didCancel = true;
    };
  }, [id]);

  // ...
}
```

## 진입 장벽을 더 높이기

클래스 컴포넌트의 라이프사이클 개념으로 생각하면, 사이드 이펙트는 렌더링 결과물과 다르게 동작한다. **UI**를 렌더링하는 것은 `props`와 `state`를 통해 이루어지며 이들의 일관성에 따라 보장받는다. 하지만 사이드 이펙트는 아니다. 이게 흔히 버그가 일어나는 원인이다.

`useEffect`의 개념으로 생각하면, 모든 것들은 기본적으로 동기화된다. 사이드 이펙트는 리액트의 데이터 흐름의 일부이다. 모든 `useEffect` 호출에서, 제대로만 코드를 작성했다면 엣지 케이스를 훨씬 쉽게 다룰 수 있다.

아직까지는 `useEffect` 가 가장 흔히 사용되는 경우는 데이터 패칭이다. 하지만 데이터 패칭은 엄밀히 말해 동기화의 문제는 아니다. 특히나 명백한 것이 주로 이럴 때는 deps가 `[]` 가 되기 때문이다. 도대체 우리는 무엇을 동기화하고 있을까?

장기적인 관점에서 [데이터 불러오는 로직을 해결하기 위해 Suspense가 도입되면](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#react-16x-mid-2019-the-one-with-suspense-for-data-fetching) 서드 파티 라이브러리들이 기본적으로 리액트에게 어떤 비동기적인 행위(코드, 데이터, 이미지 등 모든 것)가 준비될 때까지 랜더링을 잠시 미룰 수 있도록 지원하게 될 것이다.

`Suspense`가 자연스럽게 데이터를 불러오는 경우를 더 많이 커버하게 되면, 예측컨대 `useEffect` 는 더욱 로우 레벨로 내려가 파워유저들이 진정으로 사이드 이펙트를 통해 `props`와 `state`를 동기화하고자 할 필요가 있을 때 사용하는 도구가 될 것이다. 데이터 패칭과 다르게 원래 디자인 된 대로 이런 동기화 케이스를 자연스럽게 다룬다. 하지만 그 때까지는, [여기 있는 것과 같은](https://www.robinwieruch.de/react-hooks-fetch-data/) 커스텀 훅이 재사용 가능한 데이터 페칭 로직을 다루는 좋은 방법이다.
