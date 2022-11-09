# Redux Style Guide

> 원본 글  
> https://redux.js.org/style-guide/

**목차**

- [Redux Style Guide](#redux-style-guide)
  - [개요](#개요)
  - [Rule Categories](#rule-categories)
    - [Priority A: Essential](#priority-a-essential)
    - [Priority B: Strongly Recommended](#priority-b-strongly-recommended)
    - [Priority C: Recommended](#priority-c-recommended)
  - [Priority A Rules: `Eseential`](#priority-a-rules-eseential)
    - [#1 Do Not Mutate State](#1-do-not-mutate-state)
    - [#2 Reducers Must Not Have Side Effects](#2-reducers-must-not-have-side-effects)
    - [#3 Do Not Put Non-Serializable Values in State or Actions](#3-do-not-put-non-serializable-values-in-state-or-actions)
    - [#4 Only One Redux Store Per App](#4-only-one-redux-store-per-app)
  - [Priority B Rules: Strongly Recommended](#priority-b-rules-strongly-recommended)
    - [#1 Use Redux Toolkit for Writing Redux Logic](#1-use-redux-toolkit-for-writing-redux-logic)
    - [#2 Use Immer for Writing Immutable Updates](#2-use-immer-for-writing-immutable-updates)
    - [#3 Structure Files as Feature Folders with Single-File Logic](#3-structure-files-as-feature-folders-with-single-file-logic)
    - [#4 Put as Much Logic as Possible in Reducers](#4-put-as-much-logic-as-possible-in-reducers)
    - [#5 Reducers Should Own the State Shape](#5-reducers-should-own-the-state-shape)
    - [#6 Name State Slices Based On the Stored Data](#6-name-state-slices-based-on-the-stored-data)
    - [#7 Organize State Structure Based on Data Types, Not Components](#7-organize-state-structure-based-on-data-types-not-components)
    - [#8 Treat Reducers as State Machines](#8-treat-reducers-as-state-machines)
    - [#9 Normalize Complex Nested/Relational State](#9-normalize-complex-nestedrelational-state)
    - [#10 Keep State Minimal and Derive Additional Values](#10-keep-state-minimal-and-derive-additional-values)
    - [#11 Model Actions as Events, Not Setters](#11-model-actions-as-events-not-setters)
    - [#12 Write Meaningful Action Names](#12-write-meaningful-action-names)
    - [#13 Allow Many Reducers to Response to the Same Action](#13-allow-many-reducers-to-response-to-the-same-action)
    - [#14 Avoid Dispatching Many Actions Sequentially](#14-avoid-dispatching-many-actions-sequentially)
    - [#15 Evaluate Where Each Piece of State Should Live](#15-evaluate-where-each-piece-of-state-should-live)
    - [#16 Use the React-Redux Hooks API](#16-use-the-react-redux-hooks-api)
    - [#17 Connect More Components to Read Data from the Store](#17-connect-more-components-to-read-data-from-the-store)
    - [#18 Use the Object Shorthand Form of `mapDispatch` with `connect`](#18-use-the-object-shorthand-form-of-mapdispatch-with-connect)
    - [#19 Call useSelector Multiple Times in Function Components](#19-call-useselector-multiple-times-in-function-components)
    - [#20 Use Static Typing](#20-use-static-typing)
    - [#21 Use the Redux DevTools Extension for Debugging](#21-use-the-redux-devtools-extension-for-debugging)
    - [#22 Use Plain JavaScript Objects for State](#22-use-plain-javascript-objects-for-state)
  - [Priority C Rules: `Recommended`](#priority-c-rules-recommended)
    - [#1 Write Action Types as `domain/eventName`](#1-write-action-types-as-domaineventname)
    - [#2 Write Actions Using the Flux Standard Action Convention](#2-write-actions-using-the-flux-standard-action-convention)
    - [#3 Use Action Creators](#3-use-action-creators)
    - [#4 Use Thunks for Async Logic](#4-use-thunks-for-async-logic)
    - [#5 Move Complex Logic Outside Components](#5-move-complex-logic-outside-components)
    - [#6 Use Selector Functions to Read from Store State](#6-use-selector-functions-to-read-from-store-state)
    - [#7 Name Selector Functions as `selectThing`](#7-name-selector-functions-as-selectthing)
    - [#8 Avoid Putting Form State In Redux](#8-avoid-putting-form-state-in-redux)

## 개요

This is the official style guide for writing Redux code. **It lists our recommended patterns, best practices, and suggested approaches for writing Redux applications.**

Redux를 사용하는 방법은 많으며, 사용법에 대한 하나의 올바른 방법은 존재하지 않는다.

그러나, 시간과 경험을 바탕으로 몇몇 주제의 경우 특정 접근 방법이 다른 접근 방법보다 더 효과적이라는 것을 알 수 있었다. 또한 많은 개발자들이 이러한 접근 방법을 결정하는 것에 피로도를 줄이기 위해서 공식적인 지침을 제공해달라고 요청했다.

이를 염두에 두고 **에러**, **bikeshedding** 그리고 **안티 패턴**을 피할 수 있도록 몇 가지 권장 사항들을 작성했다. 또한, 팀 환경이 다양하고 프로젝트 마다 설정이 다르다는 것을 알기에 모든 경우에 적합한 스타일 가이드를 제공할 순 없다. 이러한 권장 사항들을 따르는 것이 베스트겠지만 시간을 내서 각자의 상황과 필요에 맞춰 해당 사항들을 따를지 말지 결정하자.

## Rule Categories

권장 사항은 세 가지 카테고리로 나뉜다.

### [Priority A: Essential](#priority-a-rules-eseential)

`Eseential` 카테고리로 분류된 사항은 에러를 방지하기에 무슨 수를 쓰더라도 이를 공부하고 준수해야 한다. 예외는 존재할지 몰라도 매우 흔하지 않으며 **JavaScript**와 **Redux**에 상당한 지식이 있는 사람만 가능하다.

### [Priority B: Strongly Recommended](#priority-b-rules-strongly-recommended)

`Strongly Recommended` 카테고리로 분류된 사항은 대부분의 프로젝트에서 가독성과(또는) 개발자 경험을 향상시킬 수 있게 한다. 해당 사항들을 준수하지 않아도 코드는 계속 실행되지만 이런 경우는 드물고 지키지 않는 합당한 이유가 있어야 한다. **Follow these rules whenever it is reasonably possible**.

### [Priority C: Recommended](#priority-c-rules-recommended)

Recommended 카테고리로 분류된 사항의 경우 동등하게 좋은 옵션이 여러 개 있는 경우 일관성 있는 선택을 할 수 있게 도와주는 역할을 한다. 이러한 사항들은 선택 가능한 옵션에 대해 설명해주고 **suggest a default choice**. That means you can feel free to make a different choice in your own codebase, as long as you're consistent and have a good reason. Please do have a good reason though!

## Priority A Rules: `Eseential`

### #1 Do Not Mutate State

`state`를 변형하는 것(**Mutating state**)은 컴포넌트가 적절한 리렌더링에 실패하는 것을 포함한 **Redux**를 사용하는 애플리케이션의 가장 흔한 버그이며, 이는 또한 **Redux DevTools**의 시간여행 디버깅에서도 문제를 일으킨다. `reducer` 내부와 다른 모든 애플리케이션 코드에서 state 값의 실제 변형은 항상 피해야 한다.

**redux-immutable-state-invariant**와 같은 라이브러리를 사용해서 개발 중에 `state`의 변형을 잡아내거나 우발적인 `state`의 변형을 피하기 위해 **Immer**를 사용할 수도 있다.

> ✋ **주의**
>
> 실제 값의 복사본을 변경하는 것은 괜찮다. 값을 복사해서 이를 변경하는 것은 불변 업데이트 로직을 작성하는 일반적인 방법이기 때문이다. 또한, 만약 불변 업데이트를 위한 **Immer** 라이브러리를 사용하고 있다면 실제 데이터는 변경되지 않기 때문에 가변 업데이트 로직을 작성하는 것이 가능하다. **Immer**는 변화를 추적하여 내부적으로 안전하게 값의 불변 업데이트를 발생시킨다.

### #2 Reducers Must Not Have Side Effects

`Reducer` 함수들은 오직 `reducer`의 `state`와 `action` 인자들에 의존해야 하며, 이러한 인자값들에 기반하여 계산하거나 새로운 `state` 값을 반환해야만 한다. 이러한 함수들은 어떠한 종류의 비동기 로직(AJAX 호출, timeouts, promises)도 실행해서는 안되며, 무작위 값(`Date.now()`, `Math.random()`)들을 생성해서도 안된다. 또한 `reducer` 외부에서 변수들을 변경하거나 해당 함수들이 실행될 때 `reducer` 함수의 스코프 밖에 있는 무언가에 영향을 줘서도 안된다.

> ✋ **주의**
>
> 동일한 규칙을 따르는 한 `reducer`가 라이브러리 또는 유틸 함수에서 `import` 해온 자체 외부에 정의된 다른 함수를 호출하도록 하는 것이 허용된다.
>
> 이 규칙의 목적은 `reducer`가 호출될 때 예측 가능하게 동작하는 것을 보장하기 위해서다. 예를 들어, 만약 우리가 시간여행 디버깅을 하고 있는 경우, `reducer` 함수가 현재 `state` 값을 생성하기 위해 이전 `action`들을 여러 번 호출할 수도 있다. 이때 만약 `reducer`가 사이드 이펙트를 가지고 있다면, 디버깅 과정 중에 이러한 사이드 이펙트가 발생할 것이고 이는 애플리케이션이 예측 불가능하게 동작하는 결과를 초래할 것이다.
>
> 이 규칙에는 애매한 부분(gray area)이 있다. 엄밀히 말하면 `console.log(state)`와 같은 코드는 사이드 이펙트지만 실제로는 애플리케이션이 작동하는 방식에 영향을 주지 않는다.

### #3 Do Not Put Non-Serializable Values in State or Actions

**Redux** 스토어 상태 또는 **dispatch**된 `actions` 내부에 `Promises`, `Symbols`, `Maps`/`Sets`, 함수들 또는 클래스 인스턴스와 같은 직렬화되지 않은 값들을 담는 것을 피해야 한다. 이는 **Redux DevTools**를 통한 디버깅과 같은 기능들이 예상대로 동작하도록 만들어 준다. 또한, UI도 예상한 대로 업데이트될 것을 보장해준다.

> 📌  **예외**
>
> 만약 `action`이 `reducer`에 도달하기 전에 미들웨어에 의해서 인터셉트되거나 멈춘다면 `action` 내부에 직렬화되지 않은 값들을 담는 것이 가능하다. 이러한 미들웨어의 예는 대표적으로 `redux-thunk`나 `redux-promise`가 있다.

### #4 Only One Redux Store Per App

일반적인 Redux 애플리케이션이라면 애플리케이션 전체에서 사용되는 단 하나의 Redux 스토어 인스턴스만 가지고 있어야 한다. 이는 일반적으로 `store.js`와 같은 분리된 파일에서 정의된다.

이상적으로는 어떠한 애플리케이션 로직도 스토어를 직접적으로 `import` 하지 않는다. 이는 `<Provider>`를 통해 **React** 컴포넌트 트리에 전달되거나 `thunk`와 같은 미들웨어를 통해 간접적으로 참조되어야 한다. 아주 드문게, 다른 로직 내부에 이를 `import` 할 수도 있겠지만 이는 최후의 수단이 되어야만 한다.

## Priority B Rules: Strongly Recommended

### #1 Use Redux Toolkit for Writing Redux Logic

**[Redux Toolkit](https://redux.js.org/redux-toolkit/overview) is our recommended toolset for using Redux.** **Redux Toolkit**은 변형을 잡아내고 **Redux DevTools Extension**을 사용할 수 있도록 스토어를 설정하고 `Immer`를 통한 불변 업데이트 로직 작성을 간단하게 만들어 주는 등 이러한 다양한 기능들이 포함된 **best practices**를 기반으로 하는 함수들을 가지고 있다.

You are not required to use RTK with Redux, and you are free to use other approaches if desired, but **using RTK will simplify your logic and ensure that your application is set up with good defaults.**

### #2 Use Immer for Writing Immutable Updates

불변 업데이트 로직을 일일히 작성하는 것은 어렵고 빈번하게 에러를 발생시킨다. Immer는 우리가 가변 업데이트 로직 사용하여 간단하게 불변 업데이트를 할 수 있게 만들어 주며 심지어 state를 freeze(동결) 처리하여 개발 중에 애플리케이션 아무 곳에서나 해당 state의 변형을 잡아낼 수 있도록 해준다. **We recommend using Immer for writing immutable update logic, preferably as part of [Redux Toolkit](https://redux.js.org/redux-toolkit/overview)**.

### #3 Structure Files as Feature Folders with Single-File Logic

**Redux** 자체에서는 현재 애플리케이션의 폴더와 파일 구조가 어떻게 되어 있는지 신경쓰지 않는다. 그러나, 일반적으로 기능에 관련된 로직들이 한 곳에 위치해야 코드를 유지보수 하기가 쉬워진다.

그렇기에 대부분의 애플리케이션을 **`feature folder`** 접근 방법(기능을 위한 모든 파일이 동일한 폴더에 위치)을 사용하여 파일을 구조화하는 것을 추천한다. 이 방법을 이용할 경우, 기능을 위한 **Redux** 로직은 하나의 **`slice`** 파일에 작성되어야 하며 이는 가급적이면 **Redux Toolkit**의 `createSlice` API를 사용해야 한다 (이 방법은 `ducks` 패턴으로도 알려져 있다). 이전에는 `actions` 및 `reducers`에 대해 별도의 폴더가 있는 **`folder-by-type`** 접근 방법을 자주 사용했지만 관련된 로직을 함께 유지할 경우 해당 코드를 더 쉽게 찾고 업데이트할 수 있다.

> **Detailed Explanation: Example Folder Structure**
>
> - `/src`
>   - `index.tsx`: Entry point file that renders the React component tree
>   - `/app`
>     - `store.ts`: store setup
>     - `rootReducer.ts`: root reducer (optional)
>     - `App.tsx`: root React component
>   - `/common`: hooks, generic components, utils, etc
>   - `/features`: contains all "feature folders"
>     - `/todos`: a single feature folder
>       - `todosSlice.ts`: Redux reducer logic and associated actions
>       - `Todos.tsx`: a React component
>
> `/app` contains app-wide setup and layout that depends on all the other folders.
>
> `/common` contains truly generic and reusable utilities and components.
>
> `/features` has folders that contain all functionality related to a specific feature. In this example, `todosSlice.ts` is a "duck"-style file that contains a call to RTK's `createSlice()` function, and exports the slice reducer and action creators.

### #4 Put as Much Logic as Possible in Reducers

가능한 한, 준비하고 `action`을 `dispatch`하는 코드(**click handler**와 같은)가 아닌 적절한 `reducer`에 새로운 `state`를 계산하는 로직을 최대한 많이 넣어라. 이를 통해 더 많은 애플리케이션 로직을 쉽게 테스트할 수 있고 시간여행 디버깅을 효과적으로 사용할 수 있으며 변형이나 버그를 일으킬 수 있는 일반적인 실수를 피할 수 있다.

새로운 `state`의 일부 또는 전체가 먼저 계산되어야 하는(고유 ID 생성과 같은) 몇 가지 케이스가 있을 수 있지만 이는 최소한으로 유지해야만 한다.

> Detailed Explanation
>
> **Redux** core는 실제로 `reducer` 내부 또는 `action` 생성 로직 내부에서 새로운 state 값이 계산되는지 계산되지 않는지 신경쓰지 않는다. 예를 들어, Todo 애플리케이션에서 "**toggle todo**" `action`에 대한 로직이 `todos` 배열에 대한 불변 업데이트가 필요하다고 해보자. `action`은 **todo ID** 만을 포함하고 `reducer` 내부에서 새로운 배열을 계산하는 것은 문제가 없다.

```jsx
// Click handler:
const onTodoClicked = (id) => {
  dispatch({ type: "todos/toggleTodo", payload: { id })
}

// Reducer:
case "todos/toggleTodo": {
  return state.map(todo => {
    if (todo.id !== action.payload.id) return todo;

    return { ...todo, completed: !todo.completed };
  })
}
```

> 다음과 같이 새로운 배열을 먼저 계산하고 `action`에 새로운 배열 전체를 전달할 수도 있다.

```jsx
// Click handler:
const onTodoClicked = id => {
  const newTodos = todos.map(todo => {
    if (todo.id !== id) return todo

    return { ...todo, completed: !todo.completed }
  })

  dispatch({ type: 'todos/toggleTodo', payload: { todos: newTodos } })
}

// Reducer:
case "todos/toggleTodo":
  return action.payload.todos;
```

> 위와 같이 할 수 있지만 다음과 같은 이유들로 인해 `reducer` 내부에서 로직을 실행하는 것이 더 선호된다.
>
> ﹒ **Reducer**는 순수 함수기 때문에 항상 테스트하기 쉽다. - 그저 `const result = reducer(testState, action)`과 같이 호출하면 되고 `result` 변수에 기대한 값이 들어가 있는지 확인해보면 된다. 그래서 `reducer`에 더 많은 로직을 둘 수록 해당 로직들을 테스트하기 더 쉬워진다.
>
> ﹒ **Redux** `state`는 항상 불변 업데이트를 지켜야만 한다. 대부분의 **Redux** 사용자들은 `reducer` 내부에서는 해당 규칙을 지켜야만 하는 것을 인지하고 있지만 `reducer` 외부에서 계산된 새로운 `state`는 이러한 규칙을 지켰는지 명백하게 알 수는 없다.
>
> ﹒ 만약 **Redux Toolkit** 또는 **Immer**를 사용하고 있다면, `reducer` 내에서 불변 업데이트 로직을 훨씬 더 작성하기 쉽고 **Immer**가 `state`를 `freeze` 처리하여 우발적인 변형을 잡아준다.
>
> ﹒ 시간여행 디버깅은 `dispatch`된 `action`을 **undo**하거나 다른 작업을 수행하거나 `action`을 **redo** 함으로써 동작한다. 게다가, `reudcer`의 **hot-reloading**은 일반적으로 존재하는 `action`들을 새로운 `reducer`와 다시 실행하는 것을 포함한다. 만약 올바른 `action`을 실행했는데 `redcuer`에 문제가 있는 경우 버그를 수정하기 위해 `reducer`를 수정하고 해당 `reducer`를 **hot-reload** 하면 올바른 `state`를 곧바로 얻을 수 있다. 만약 `action`이 잘못 됐다면, `action`이 `dispatch` 되는 일련의 과정을 다시 실행해야 한다. 그래서 `reducer` 내부에 로직이 많을 수록 디버깅하기 쉽다.
>
> ⇒ `action`은 정상인데 `reducer`에 문제가 있는 경우 `reducer`를 수정하여 동일한 `action`에 대한 결과값의 `state`를 곧바로 얻을 수 있는데 `action`에 로직이 있어서 `action`을 수정해야 하는 경우라면 해당 `action`을 `dispatch`하는 일련의 과정을 동일하게 밟아야만 한다는 뜻
>
> ﹒마지막으로 `reducer` 내부에 로직을 둔다는 것은 업데이트 로직이 애플리케이션 코드의 다른 부분에 무작위로 흩어져 있지 않다는 것 즉, 업데이트 로직이 어디 있는지 단번에 알 수 있다는 것을 의미한다.

### #5 Reducers Should Own the State Shape

**Redux**의 `root state`는 단일 `root reducer` 함수가 소유하고 계산해야 한다. 유지보수를 위해 `root reducer`는 **key/value** `slices`로 분할되며, 각 `slice reducers`는 초기값을 제공하고 `slice`의 상태에 대한 업데이트를 계산한다.

게다가, `slice reducer`는 계산된 상태의 일부로 반환되는 다른 값들을 제어해야만 한다.(In addition, slice reducers should exercise control over what other values are returned as part of the calculated state.) `return action.payload` 또는 `return { …state, …action.payload }`와 같은 **blind spreads/returns** 사용을 최소화해야 한다. 왜냐하면 이런 **blind spreads/returns**은 `dispatch`된 `action`의 `payload`에 의존하게 되고 `reducer`는 `state`가 어떻게 이루어졌는지에 대해 알기를 포기하는 꼴이 되기 때문이다. 이는 `action`의 `payload`가 올바르지 않을 경우 버그를 발생시킬 수 있다.

> ✋ **주의**
>
> "**spread return**"을 사용한 `reducer`는 각 개별 필드에 대해 별도의 `action` 타입을 작성하는 것이 시간이 많이 걸리고 이를 통한 이점이 거의 없는 예를 들어 `form`의 데이터를 편집하는 것과 같은 시나리오에는 적합한 선택일 수 있다.

> **Detailed Explanation**
>
> 다음과 같은 현재 사용자에 대한 reducer가 있다고 해보자.

```jsx
const initialState = {
  firstName: null,
  lastName: null,
  age: null,
};

export default usersReducer = (state = intialState, action) {
  switch(action.type) {
    case "users/userLoggedIn": {
      return action.payload;
    }
    default:
      return state;
  }
}
```

> 위 예제에서, `redcuer`는 `action.payload`가 올바른 형식의 객체일 거라고 완전하게 가정하고 있다.
>
> 그러나 `action` 내부에 `user` 객체 대신 `todo` 객체가 `dispatch` 됐다고 상상해보자.

```jsx
dispatch({
  type: 'users/userLoggedIn',
  payload: {
    id: 42,
    text: 'Buy milk',
  },
});
```

> `reducer`는 맹목적으로 `todo`를 반환할테니 이제 애플리케이션이 스토어에서 사용자 정보를 읽으려고 할 때 문제가 생길 것이다.
>
> 이러한 문제는 `redcucer`가 `action.payload`가 실제로 올바른 필드를 가지고 있는지 확인하기 위한 몇 가지 유효성 검사가 있거나 이름으로 올바른 필드를 읽으려고 시도하는 경우 적어도 부분적으로 수정될 수 있을 지도 모른다. 하지만 이는 더 많은 코드를 작성하게 만든다. 즉 애플리케이션을 안전하게 만들기 위해서 기존보다 더 많은 코드를 만들어내는 결과를 만들어 낸다.
>
> 정적 타입의 사용은 이러한 종류의 코드를 더 안전하고 수용 가능하게 만들어 줄 수 있다. 만약 `reducer`가 `action`의 타입을 `PayloadAction<User>`로 알고 있게 된다면 `return action.payload` 구문의 실행이 조금은 더 안전해진다.

### #6 Name State Slices Based On the Stored Data

As mentioned in [Reducers Should Own the State Shape](https://redux.js.org/style-guide/#reducers-should-own-the-state-shape) , `reducer` 로직 분리에 대한 일반적인 접근법은 `state`의 `slices`에 기반한다. 이에 따라, `combineReducers`는 이러한 `slice reducer`들을 거대한 `reducer`로 합치기 위한 표준 함수이다.

`combineReducers`에 전달되는 객체의 `key` 이름들은 `combineReducers`가 반환하는 `state` 객체의 `key` 이름들을 정의할 것이다. 내부에 보관된 데이터의 이름을 따서 이러한 `key`의 이름을 지어야 하며 `key` 이름에 `reducer`라는 단어의 사용은 피해야 한다. 객체는 `{ usersRedcuer: {}, postReducer: {}}` 가 아닌 `{ users: {}, posts: {}}`가 되어야만 한다.

> **Detailed Explanation**
>
> ES6의 객체 리터럴 단축 표현은 객체에서 `key` 이름과 `value`를 한 번에 정의하는 것을 쉽게 만들어 준다.

```jsx
const data = 42;
const obj = { data };
// same as: { data: data }
```

> `combineReducers`는 `reducer` 함수로 가득찬 객체를 인자로 전달받게 되고 해당 객체의 `key` 이름과 동일한 `state` 객체를 만들어 사용한다. 이는 함수 객체의 `key` 이름이 `state` 객체의 `key` 이름들을 정의한다는 말이다.
>
> 이로 인해 변수 이름에 "**reducer**"를 사용하는 `reducer`를 가져온 다음 객체 리터럴 단축 표현을 사용하여 `combineReducers`에 전달하는 흔한 실수가 발생한다.

```jsx
import userReducer from 'features/users/usersSlice';

const rootReducer = combineReducers({
  usersReducer,
});
```

> 이때, 객체 리터럴 단축 표현의 사용은 `{ userReducer: userReducer }`과 같은 객체를 만들어 낸다. 이렇게 되면 "**reducer**"가 이제 `state`의 `key` 이름에 들어가게 되는데 이는 불필요하고 쓸모없는 이름이다.
>
> 대신 내부 데이터에만 관련된 key 이름을 정의해라. 명확성을 위해 명시적 `key`: `value` 구문을 사용하는 것이 좋다. 이는 코드의 양을 조금 더 늘리겠지만 가장 이해할 수 있는 코드와 `state` 정의를 만들어 낸다.

```jsx
import userReducer from 'features/users/userSlice';
import postReducer from 'features/posts/postSlice';

const rootReducer = combineReducers({
  users: userReducer,
  posts: postsReducer,
});
```

### #7 Organize State Structure Based on Data Types, Not Components

**Root state slices**는 **UI**에 있는 특정 컴포넌트가 아니라 애플리케이션의 주요 데이터 타입 또는 기능 영역을 기반으로 정의되고 네이밍되어야만 한다. 왜냐하면 UI에 있는 컴포넌트들과 **Redux** `store`의 데이터 간의 명확한 1:1 상관 관계가 없고 많은 컴포넌트들은 동일한 데이터에 접근할 지도 모르기 때문이다. `state` 트리를 애플리케이션의 모든 곳에서나 `state`의 일부를 읽기 위하여 접근할 수 있는 일종의 전역 데이터베이스라고 생각하면 된다.

예를 들어, 블로깅 애플리케이션은 로그인한 사람, 작성자 및 게시물에 대한 정보, 활성화된 화면에 대한 정보를 추적해야 할 수 있다. 여기서 좋은 `state` 구조는 `{ auth, posts, users, ui }`와 같을 수 있다. 안좋은 `state` 구조는 `{ loginScreen, usersList, postsList }`와 같을 수 있다.

### #8 Treat Reducers as State Machines

많은 **Redux**의 `reducer`들은 `unconditionally`하게 작성되어 있다. 이러한 `reducer`들은 현재 `state`에 대한 어떠한 로직도 기반으로 하지 않고 `dispatch`된 `action`만 보고 새로운 `state` 값을 계산한다. 이는 몇몇의 `action`들의 경우 애플리케이션의 로직에 따라 어떨 때는 개념적으로 **유효하지 않을 수도** 있기에 버그를 만들어낼 수 있다. 예를 들어, **"request successded"** `action`은 `state`가 이미 **"loading"** 되어 있는 경우에만 새로운 값을 계산해야 하며, **"update this item"** `action`은 `item`이 **"being edited"**라고 되어 있는 경우에만 `dispatch` 되어야 한다.

이 문제를 해결하려면 `reducer`를 `**state machines**`로만 취급해야 한다. `action`이 `unconditionally`(무조건적으로) 계산되는 것이 아니라 현재 `state`와 `dispatch`된 `action`의 조합을 보고 새로운 `state` 값을 계산할지 말지 여부를 결정한다.

> **Detailed Explanation**
>
> **Server state**는 **React Query**를 통해 관리할 것이기 때문에 `finite state machine`을 이용한 데이터 패칭 상태 및 응답 데이터 관리에 관한 설명은 생략

### #9 Normalize Complex Nested/Relational State

**Server state**는 **React Query**를 통해 관리할 것이기 때문에 복잡한 `cache` 데이터 관리에 관한 설명은 생략

### #10 Keep State Minimal and Derive Additional Values

가능하면 **Redux** `store` 내에 실제 데이터를 최소한으로 유지하고 필요에 따라 해당 `state`를 이용하여 추가적인 값을 파생하여 사용하자. 여기엔 목록 필터링이나 합산과 같은 작업이 포함된다. 예를 들어, **todo** 애플리케이션에서 `state`에 **todo** 객체들의 원본 목록은 유지하고 `state` 외부에서 `state`가 업데이트될 때마다 필터링된 **todo** 목록을 파생한다. 비슷하게 모든 **todo**가 완료됐는지 확인하거나 몇 개의 **todo**가 남아 있는지 `store` 밖에서 계산될 수도 있다.

이러한 방식의 장점은 다음과 같다.

- 실제 `state`는 읽기 쉬워진다.
- 추가적인 값들을 계산하거나 나머지 데이터와 동기화된 상태로 유지하는데 필요한 로직이 적어진다.
- 원본 `state`는 대체되지 않고 여전히 동일한 참조값을 가지고 있다.

데이터의 파생은 파생될 데이터의 계산 로직을 캡슐화할 수 있는 `selector` 함수에서 주로 이루어 진다. 퍼포먼스를 향상시키기 위해서, 이러한 `selector`들은 `reselect`나 `proxy-memoize`와 같은 라이브러리들을 사용하여 캐시된 이전 값을 메모이제이션할 수도 있다.

### #11 Model Actions as Events, Not Setters

**Redux**는 `action.type` 필드의 내용이 무엇인지 신경쓰지 않는다. - 해당 필드는 단지 정의되어 있어야할 뿐이다. `action type`들을 현재 시제(`users/update`), 과거 시제(`users/updated`), 이벤트 묘사(`upload/progress`) 또는 **setter** (`users/setUserName`)와 같이 작성하는 것에 문제는 없다. 애플리케이션에서 주어진 작업이 의미하는 바와 이러한 작업을 모델링하는 방법을 결정하는 것은 사용자의 몫이다.

그러나 **"setter"**보다는 **"describing events that occurred"**로서 `action`을 대하는 것을 추천한다. `action`을 이벤트로서 대하면 일반적으로 더 의미 있는 `action` 이름, `dispatch`된 `action` 수의 총 양 감소, 더 의미있는 `action` 로그 기록들이 생성된다. **"setters"**로서 `action`를 작성하는 것은 종종 너무 많은 개별 `action` 타입, 너무 많은 `dispatch` 그리고 의미없는 action 로그들을 만들어 낸다.

> **Detailed Explanation**
>
> 우리가 레스토랑 애플리케이션을 가지고 있다고 상상해보자. 어떤 사람이 피자와 콜라 한 병을 주문했고 이에 대한 `action`을 다음과 같이 `dispatch`한 상황이다.

```jsx
{ type: 'food/orderAdded', payload: { pizza: 1, coke: 1 } }
```

> 또는 다음과 같이 `dispatch`할 수도 있다.

```jsx
{
  type: 'orders/setPizzazOrdered',
  payload: {
    amount: getState().orders.pizza + 1,
  }
}

{
  type: 'orders/setCokesOrdered',
  payload: {
    amount: getState().orders.coke + 1,
  }
}
```

> 첫 번째 예제는 `event`다. "Hey, someone ordered a pizza and a pop, deal with it somehow".
>
> 두 번째 예제는 `setter`다. "I *know* there are fields for 'pizzas ordered' and 'pops ordered', and I am commanding you to set their current values to these numbers".
>
> `event` 접근 방법은 오직 필요한 단일 `action`만 `disptach`되면 되고 더 유연하다. 이는 얼마나 많은 `pizza`가 이미 주문되었는지 상관하지 않는다. 만약 요리할 준비가 되어있지 않은 상태라면 해당 주문은 무시될 것이다.
>
> `setter` 접근 방법은 클라이언트 코드에서 `state`의 실제 구조가 어떻게 이루어 졌는지 그리고 올바른 값이 무엇인지 알아야만 했으며 `transaction`을 마무리하기 위해서 다수의 `action`을 `dispatch` 해야만 했다.

### #12 Write Meaningful Action Names

`action.type` 필드는 두 가지 주요 목적을 지니고 있다.

- `Reducer` 로직은 새로운 `state`를 계산하기 위해서 `action`을 처리해야 하는지 확인하기 위해 `action type`을 확인한다.
- `action type`은 **Redux DevTools** 로그 기록에 읽을 수 있게 보여진다.

Per [Model Actions as "Events"](https://redux.js.org/style-guide/#model-actions-as-events-not-setters), type 필드의 실제 내용물이 무엇인지 Redux는 관심이 없다. 그러나 `type` 값은 개발자 즉, 우리에게는 상관이 있다. action은 의미있고 유익하고 기술적인 type 필드로 작성되어야만 한다. 이상적으로 dispatch된 action type들을 읽을 수 있어야 하며, 각 action의 내용물을 확인하지 않고도 애플리케이션에서 무슨 일이 일어났는지에 대해 이해할 수 있어야 한다. `SET_DATA` 또는 `UPDATE_STORE`와 같은 매우 일반적인 `action` 이름들을 피해야 하는데 이러한 이름들은 무슨 일이 일어났는지에 대한 의미 있는 정보를 제공하지 못하기 때문이다.

### #13 Allow Many Reducers to Response to the Same Action

**Redux** `reducer` 로직은 여러 개의 더 작은 `reducer`로 분할되어 각각 독립적으로 `state` 트리에서 자신이 해당하는 부분을 업데이트하고서 모두 함께 다시 구성되어 `root reducer` 함수를 형성하도록 되어 있다. `action`이 `dispatch` 됐을 때, 이는 `reducer` 전체, 일부 또는 그 어떤 `reducer`에 의해 처리되지 않을 수도 있다.

As part of this, you are encouraged to **have many reducer functions all handle the same action separately** if possible(많은 `reducer` 함수들이 모두 동일한 `action`을 개별적으로 처리). 경험에 따르면 대부분의 `action`들은 일반적으로 단일 `reducer` 함수에 의해 처리되기 때문에 괜찮다. 그러나 `events`로서 `action`을 모델링하고 많은 `reducer`가 이러한 `action`에 응답하도록 하면 일반적으로 애플리케이션의 **codebase**를 더 잘 확장할 수 있고 의미 있는 업데이트 하나를 수행하기 위해 다수의 `action`을 `dispatch` 해야 하는 횟수를 최소화할 수 있다.

### #14 Avoid Dispatching Many Actions Sequentially

개념적으로 더 큰 `transaction`을 처리하기 위해 연속적으로 많은 `action`을 `dispatch` 하지 마라. 이는 가능한 방법이긴 하지만 상대적으로 높은 비용의 **UI** 업데이트를 여럿 만들어 낼 수 있으며, 업데이트 발생하는 중간의 `state`는 애플리케이션 로직에 의해서 잘못된 값을 가지고 있을 가능성을 내포하게 된다. 모든 적잘한 `state` 업데이트를 한 번에 가져오는 단일 `event` 타입의 `action`을 `dispatch`하는 하거나 마지막에 단일 **UI** 업데이트로 다수의 `action`이 `dispatch`되는 것을 배칭하여 작업할 수 있는(일괄 처리할 수 있는) 애드온의 사용을 고려하자.

> 얼마나 많은 `action`을 연속적으로 `dispatch` 할 수 있는지에 대해선 제한은 없다. 그러나 각 `dispatch`된 `action`은 `store`의 모든 **subscription callback**의 실행을 만들어내고(일반적으로 **Redux**가 연결된 하나 또는 그 이상의 UI 컴포넌트) UI 업데이트를 만들어낼 것이다. React 이벤트 핸들러 큐에 저장된 **UI** 업데이트가 단일 **React** 렌더링 단계에서 배칭으로 일괄 처리되는 반면에 큐 외부에 존재하는 이벤트 핸들러의 업데이트는 일괄 처리되지 않는다. 이는 대부분의 비동기 함수, timeout callback 그리고 **React**가 아닌 코드들로부터 발생하는 `dispatch`도 포함된다. (**React** v18부터는 비동기 로직도 배칭 처리되게 업데이트된 상태) 이러한 경우 각 `dispatch`는 `dispatch`가 완료되기 전에 완전한 동기식 **React** 렌더링 단계를 만들어 내기 때문에 퍼포먼스를 저하시킨다.
>
> 추가적으로 개념적으로는 거대한 `transaction` 스타일의 업데이트의 일부인 다수의 `dispatch`들은 유효하지 않은 `state`일 수도 있는 `intermediate state`가 발생한다. 예를 들어, 만약 `UPDATE_A`, `UPDATE_B` 그리고 `UPDATE_C` `action`들이 연속적으로 `dispatch` 됐을 때 `a`, `b` 그리고 `c`가 모두 함께 업데이트 됐을 것이라고 기대하지만 처음 두 `dispatch`된 후의 `state`는 하나 또는 두개만 업데이트되었기 때문에 사실상 불완전하다고 볼 수 있다.
>
> 만약 다수의 `dispatch`가 정말로 필요하다면, 해당 업데이트들을 배칭하는 방법(일괄 처리)을 고려하자.
>
> [https://react-redux.js.org/api/batch](https://react-redux.js.org/api/batch)에서 확인한 결과 **React** v18부터는 이러한 배칭 처리를 추가적으로 하지 않아도 된다.

### #15 Evaluate Where Each Piece of State Should Live

**Redux**의 3가지 원리에 따르면 애플리케이션 전체의 `state`는 단일 `tree`에 저장되어야 한다. 이는 과해석된 감이 있다. 말 그대로 애플리케이션 전체의 모든 값들을 `Redux` `store`에 보관해야 한다는 말이 아니다. 대신 **전역** 그리고 **앱 전체**에 걸쳐서 사용되는 모든 값들을 찾기 위한 단일 공간이어야 한다. 일반적으로 **local** 값들은 제일 가까운 **UI** 컴포넌트에서 유지되어야 한다.

이렇기에 **Redux** `store`에 있어야할 `state`와 **Component**에 있어야할 `state`를 결정하는 것 오롯이 개발자의 몫이다.

### #16 Use the React-Redux Hooks API

**React** 컴포넌트에서 **Redux** `store`와 상호 작용하기 위한 기본적인 방법으로 **React-Redux Hooks API**를 사용해라. 이전 `connect` **API**가 여전히 잘 동작하고 계속해서 지원할 것이긴 하지만, **hooks** **API**가 일반적으로 더 사용하기 쉽다. **hooks**는 간접 참조가 적고 코드의 양도 적으며 `connect`보다 **TypeScript**와 쓰기도 더 간단하다.

**hooks API**는 퍼포먼스와 데이터 흐름 측면에서 `connect`와는 다른 절충안들 소개하지만 이제 기본적으로는 **hooks API**를 추천한다.

> **Detailed Explanation**
>
> The [classic `connect` API](https://react-redux.js.org/api/connect) is a [Higher Order Component](https://reactjs.org/docs/higher-order-components.html). It generates a new wrapper component that subscribes to the store, renders your own component, and passes down data from the store and action creators as props.
>
> This is a deliberate level of indirection, and allows you to write "presentational"-style components that receive all their values as props, without being specifically dependent on Redux.
>
> The introduction of hooks has changed how most React developers write their components. While the "container/presentational" concept is still valid, hooks push you to write components that are responsible for requesting their own data internally by calling an appropriate hook. This leads to different approaches in how we write and test components and logic.
>
> The indirection of `connect` has always made it a bit difficult for some users to follow the data flow. In addition, `connect`'s complexity has made it very difficult to type correctly with TypeScript, due to the multiple overloads, optional parameters, merging of props from `mapState` / `mapDispatch` / parent component, and binding of action creators and thunks.
>
> `useSelector` and `useDispatch` eliminate the indirection, so it's much more clear how your own component is interacting with Redux. Since `useSelector` just accepts a single selector, it's much easier to define with TypeScript, and the same goes for `useDispatch`.

### #17 Connect More Components to Read Data from the Store

더 많은 **UI** 컴포넌트들을 **Redux** `store`에 구독하고 세분화된 단계로 데이터를 읽는 것을 선호해라. 이는 일반적으로 더 나은 **UI** 퍼포먼스로 이어지며, 주어진 `state`에 변경이 발생할 때 더 적은 수의 컴포넌트들이 렌더링해야 하기 때문이다.

예를 들어, `<UserList>` 컴포넌트에 연결하여 사용자 배열 전체를 읽는 것보다 `<UserList>`가 사용자 ID 목록을 검색하고 목록의 아이템을 `<UserListItem userId={userId}>`로 렌더링하고 `<UserListItem>`에 연결되어 해당하는 사용자 정보를 스토어로부터 추출해내는 방법이 낫다.

### #18 Use the Object Shorthand Form of `mapDispatch` with `connect`

The `mapDispatch` argument to `connect` can be defined as either a function that receives `dispatch` as an argument, or an object containing action creators. **We recommend always using [the "object shorthand" form of `mapDispatch`](https://react-redux.js.org/using-react-redux/connect-mapdispatch#defining-mapdispatchtoprops-as-an-object)**, as it simplifies the code considerably. There is almost never a real need to write `mapDispatch` as a function.

### #19 Call useSelector Multiple Times in Function Components

`useSelector` **hook**을 사용하여 데이터를 검색할 때, 객체 내에 다수의 결과물을 반환하는 거대한 단일 `useSelector` 대신에 작은 양의 데이터를 검색하는 `useSelector`를 여러 번 호출하는 것을 선호해라. `mapState`와 달리 `useSelector`는 객체 반환을 필요로 하지 않으며, `selector` 함수가 더 작은 값을 읽도록 하면 `state` 변경 시 해당 `state`를 구독하고 있는 컴포넌트가 렌더링될 가능성이 줄어든다.

그러나, 적절한 세분화의 균형을 찾아야 한다. 만약 단일 컴포넌트가 `state`의 `slice` 내부에 있는 모든 필드가 필요한 경우 각 개별 필드에 대한 개별 `selector` 대신 전체 `slice`를 반환하는 것이 나을 수도 있다.

### #20 Use Static Typing

순수 **JavaScript**보다는 **TypeScript**나 **Flow**와 같은 정적 타입 시스템을 사용하라. 타입 시스템은 많은 양의 흔한 실수들을 잡아줄 것이고 코드의 문서화 향상 그리고 궁극적으로 유지보수성의 향상을 가져온다. **Redux**와 **React-Redux**는 원래 순수 **JavaScript**를 염두에 두고 만들어 졌지만 **TypeScript**와 **Flow**랑도 잘 동작한다. **Redux Toolkit**은 **TypeScript**로 작성되었으며 최소한 양의 추가적인 타입 정의와 함께 적절한 **type safety**를 제공하기 하려고 디자인되었다.

### #21 Use the Redux DevTools Extension for Debugging[](https://redux.js.org/style-guide/#use-the-redux-devtools-extension-for-debugging)

**Configure your Redux store to enable [debugging with the Redux DevTools Extension](https://github.com/reduxjs/redux-devtools/tree/main/extension)**. It allows you to view:

- The history log of dispatched actions
- The contents of each action
- The final state after an action was dispatched
- The diff in the state after an action
- The [function stack trace showing the code where the action was actually dispatched](https://github.com/reduxjs/redux-devtools/blob/main/extension/docs/Features/Trace.md)

In addition, the DevTools allows you to do "time-travel debugging", stepping back and forth in the action history to see the entire app state and UI at different points in time.

**Redux was specifically designed to enable this kind of debugging, and the DevTools are one of the most powerful reasons to use Redux**.

### #22 Use Plain JavaScript Objects for State

**Immutable.js**와 같은 특화된 라이브러리보다는 `state tree`에 순수 **JavaScript** 객체와 배열의 사용을 선호해라. **Immutable.js**를 사용하여 얻은 잠재적인 이득이 존재할테지만, 쉬운 참조 비교와 같이 일반적으로 언급되는 대부분의 경우들은 특별한 라이브러리가 필요하지 않는 불변 업데이트들이다. 이는 번들 사이즈를 더 작게 유지하거나 데이터 타입 전환의 복잡도를 줄여 준다.

As mentioned above, we specifically recommend using Immer if you want to simplify immutable update logic, specifically as part of Redux Toolkit.

> **Detailed Explanation**
>
> Immutable.js has been semi-frequently used in Redux apps since the beginning. There are several common reasons stated for using Immutable.js:
>
> - Performance improvements from cheap reference comparisons
> - Performance improvements from making updates thanks to specialized data structures
> - Prevention of accidental mutations
> - Easier nested updates via APIs like `setIn()`
>
> There are some valid aspects to those reasons, but in practice, the benefits aren't as good as stated, and there's multiple negatives to using it:
>
> - Cheap reference comparisons are a property of any immutable updates, not just Immutable.js
> - Accidental mutations can be prevented via other mechanisms, such as using Immer (which eliminates accident-prone manual copying logic, and deep-freezes state in development by default) or `redux-immutable-state-invariant` (which checks state for mutations)
> - Immer allows simpler update logic overall, eliminating the need for `setIn()`
> - Immutable.js has a very large bundle size
> - The API is fairly complex
> - The API "infects" your application's code. All logic must know whether it's dealing with plain JS objects or Immutable objects
> - Converting from Immutable objects to plain JS objects is relatively expensive, and always produces completely new deep object references
> - Lack of ongoing maintenance to the library
>
> The strongest remaining reason to use Immutable.js is fast updates of *very* large objects (tens of thousands of keys). Most applications won't deal with objects that large.
>
> Overall, Immutable.js adds too much overhead for too little practical benefit. Immer is a much better option.

## Priority C Rules: `Recommended`

### #1 Write Action Types as `domain/eventName`

원래 **Redux** 문서와 예에서는 일반적으로 `"ADD_TODO"`, `"INCREMENT"`와 같이 `SCREAMING_SNAKE_CASE` 컨벤션을 사용하여 `action type`을 정의했었다. 이는 대부분의 프로그래밍 언어에서 상수를 선언하는 일반적이 컨벤션이다. 이 방법의 단점이라고 한다면 대문자로 되어 있기 때문에 읽기 어려울 수도 있다는 점이다.

다른 커뮤니티에서는 다른 컨벤션을 택했었는데 이는 `action`이 관련 있는 `feature` 또는 `domain`과 함께 특정 `action type`을 명시하는 방법이다. The NgRx community typically uses a pattern like `"[Domain] Action Type"`, such as `"[Login Page] Login"`. Other patterns like `"domain:action"` have been used as well.

**Redux Toolkit**의 `createSlice` 함수는 현재 `action type`을 `todos/addTodo`와 같은 `domain/action`으로 생성한다. 그렇기에 앞으로 가독성을 위해 `domain/action`을 사용하는 것이 좋다.

### #2 Write Actions Using the Flux Standard Action Convention

원래 **Flux Architecture** 문서에 명시되어 있기로는 `action` 객체엔 `type` 필드는 반드시 있어야 한다는 것 외에 작업의 필드에 어떤 종류의 필드 또는 명명 규칙을 사용해야 하는지에 대한 추가 지침은 제공하지 않았다. 일관성을 제공하기 위해 Andrew Clark은 **Redux** 개발 초기에 **"Flux Standard Actions"**라는 규칙을 만들었다. 요약하면, FSA 협약에 따르면 `action`는

- 언제나 `payload` 필드 안에 데이터를 전달해야 한다.
- 추가적인 정보가 있다면 이는 `meta` 필드가 가지고 있는다.
- 어떤 종류의 실패를 나타내는 `action`을 가리키는 오류 필드가 있을 수 있다.

Many libraries in the Redux ecosystem have adopted the FSA convention, and Redux Toolkit generates action creators that match the FSA format.

일관성을 위해 FSA 포맷의 `action`의 사용을 선호해라.

> ✋  주의
>
> FSA 스펙에 따르면 "**error**" `action`은 `error: true`를 설정하고 `action`의 "유효한" 형식과 동일한 `action type`을 사용해야 한다. 실제로 대부분의 개발자는 "**success**" 및 "**error**" 사례에 대해 별도의 `action type`을 작성한다.

### #3 Use Action Creators

"**Action creator**" 함수는 **original Flux Architecture** 접근 방법과 함께 시작됐다. **Redux**에선 `action creator`가 엄격하게 필요하진 않다. 컴포넌트와 다른 로직에서 항상 인라인으로 작성된 `action` 객체와 함께 `dispatch({ type: "some/action" })`을 호출하면 된다.

그러나, `action creator`는 일관성을 제공한다. 특히나 `action`의 내용을 채우기 위해 어떤 종류의 준비나 추가적인 로직이 필요한 경우 더 그러하다. (고유 ID 생성과 같은)

`action`을 `dispatch` 하기 위해 `action creator`의 사용을 선호해라. 그러나, `action creator`를 손으로 작성하기 보다는 **Redux** **Toolkit**의 `createSlice` 함수하여 `action createor`와 `action type`을 자동으로 생성하는 것을 추천한다.

### #4 Use Thunks for Async Logic

Redux was designed to be extensible, and the middleware API was specifically created to allow different forms of async logic to be plugged into the Redux store. That way, users wouldn't be forced to learn a specific library like RxJS if it wasn't appropriate for their needs.

This led to a wide variety of Redux async middleware addons being created, and that in turn has caused confusion and questions over which async middleware should be used.

**We recommend [using the Redux Thunk middleware by default](https://github.com/reduxjs/redux-thunk)**, as it is sufficient for most typical use cases (such as basic AJAX data fetching). In addition, use of the `async/await` syntax in thunks makes them easier to read.

If you have truly complex async workflows that involve things like cancellation, debouncing, running logic after a given action was dispatched, or "background-thread"-type behavior, then consider adding more powerful async middleware like Redux-Saga or Redux-Observable.

### #5 Move Complex Logic Outside Components

We have traditionally suggested keeping as much logic as possible outside components. That was partly due to encouraging the "container/presentational" pattern, where many components simply accept data as props and display UI accordingly, but also because dealing with async logic in class component lifecycle methods can become difficult to maintain.

**We still encourage moving complex synchronous or async logic outside components, usually into thunks**. This is especially true if the logic needs to read from the store state.

However, **the use of React hooks does make it somewhat easier to manage logic like data fetching directly inside a component**, and this may replace the need for thunks in some cases.

### #6 Use Selector Functions to Read from Store State

**"Selector functions"**는 **Redux** `store state`로부터 값을 읽는 것을 캡슐화하고 추가 데이터를 파생시키에 아주 강력한 도구이다. 게다가, **Reselect**와 같은 라이브러리는 `input`이 변경되었을 때만 결과를 다시 계산하는 메모이제이션 기능을 가진 `selector` 함수를 생성할 수 있으며, 이는 퍼포먼스 최적화 측면에서 아주 중요하다.

가능하면 `store state`를 읽기 위해 **memoized selector functions**의 사용을 강력히 추천하며 **Reselect**를 통해 이러한 `selector`를 생성하는 것 또한 추천한다.

하지만, `state`의 모든 필드에 `selector` 함수를 작성해야 한다고 생각하지는 않아도 된다. 해당 필드들에 얼마나 자주 접근하고 업데이트 하는지 애플리케이션에 이러한 `selector`를 제공함으로써 얻은 실질적인 이득은 얼마나 되는지를 기반으로 균형점을 잘 찾아야 한다.

### #7 Name Selector Functions as `selectThing`

`selector` 함수에 **prefix** 네이밍으로 `select`이라는 단어를 추천한다. 선택하고자 하는 값의 설명과 결합하면 되는데 예를 들어 `selectTodos`, `selectVisibleTodos` 그리고 `selectTodoById`가 있을 수 있다.

### #8 Avoid Putting Form State In Redux

대부분의 `form state`는 **Redux** 내로 가선 안된다. 대부분의 use-case에서 데이터는 진짜 전역 데이터가 아니거나 캐시될 필요가 없거나 여러 컴포넌트에서 한번에 사용되지도 않는다. 게다가 `form`을 **Redux**에 연결하는 것은 종종 모든 단일 변경 이벤트에 대해 `action`을 `dispatch`하는 것과 관련되어 퍼포먼스 오버헤드가 발생하고 실질적인 이점을 제공하지 않는다. (You probably don't need to time-travel backwards one character from `name: "Mark"` to `name: "Mar"`.)

데이터가 궁극적으로 **Redux**에서 끝나더라도 `form` 데이터를 로컬 컴포넌트 `state`로 유지하고 사용자가 `form`을 `submit` 했을 때만 **Redux** `store`를 업데이트 하는 `action`을 `dispatch`하는 것이 낫다.

There are use cases when keeping form state in Redux does actually make sense, such as WYSIWYG live previews of edited item attributes. But, in most cases, this isn't necessary.
