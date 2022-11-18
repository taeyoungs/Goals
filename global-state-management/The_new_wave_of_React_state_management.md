# The new wave of React state management

> 원본 글  
> https://frontendmastery.com/posts/the-new-wave-of-react-state-management/?ck_subscriber_id=1780635791

**목차**

- [The new wave of React state management](#the-new-wave-of-react-state-management)
  - [개요](#개요)
- [The problems global state management libraries need to solve](#the-problems-global-state-management-libraries-need-to-solve)
    - [1. 컴포넌트 트리 어디에서도 저장된 상태값을 읽을 수 있는 기능](#1-컴포넌트-트리-어디에서도-저장된-상태값을-읽을-수-있는-기능)
    - [2. 저장된 상태값을 작성(업데이트 혹은 생성)할 수 있는 기능](#2-저장된-상태값을-작성업데이트-혹은-생성할-수-있는-기능)
    - [3. 렌더링을 최적화하기 위한 mechanisms을 제공해야 한다.](#3-렌더링을-최적화하기-위한-mechanisms을-제공해야-한다)
    - [4. 메모리 사용에 대한 최적화 mechanisms을 제공해야 한다.](#4-메모리-사용에-대한-최적화-mechanisms을-제공해야-한다)
  - [More problems to solve](#more-problems-to-solve)
    - [concurrent mode(이하 동시성 모드)와의 호환성 문제](#concurrent-mode이하-동시성-모드와의-호환성-문제)
    - [Serialization of data](#serialization-of-data)
    - [The context loss problem](#the-context-loss-problem)
    - [The stale props problem](#the-stale-props-problem)
    - [The zombie child problem.](#the-zombie-child-problem)
- [A brief history of the state management ecosystem](#a-brief-history-of-the-state-management-ecosystem)
  - [The original rise of Redux](#the-original-rise-of-redux)
    - [Issues in smaller apps](#issues-in-smaller-apps)
    - [Issues in larger apps](#issues-in-larger-apps)
  - [The de-emphasis of Redux](#the-de-emphasis-of-redux)
  - [The pendulum swing to simpler approaches](#the-pendulum-swing-to-simpler-approaches)
    - [1. Re-inventing Redux](#1-re-inventing-redux)
    - [2. Optimizing runtime performance](#2-optimizing-runtime-performance)
  - [The rise of purpose built libraries to solve the remote state management problem](#the-rise-of-purpose-built-libraries-to-solve-the-remote-state-management-problem)
- [The new wave of global state management libraries and patterns](#the-new-wave-of-global-state-management-libraries-and-patterns)
    - [The rise of bottom up patterns](#the-rise-of-bottom-up-patterns)
  - [How modern libraries address the core problems of state management](#how-modern-libraries-address-the-core-problems-of-state-management)
    - [Ability to read stored state from anywhere within a subtree](#ability-to-read-stored-state-from-anywhere-within-a-subtree)
    - [Ability to write and update stored state](#ability-to-write-and-update-stored-state)
    - [Runtime performance re-render optimizations](#runtime-performance-re-render-optimizations)
    - [Memory optimizations](#memory-optimizations)
- [Concluding thoughts](#concluding-thoughts)

## 개요

**React** 애플리케이션의 규모와 복잡성이 증가함에 따라 애플리케이션 내에서 공유하고 있는 전역 상태를 관리하는 일은 어려운 일이다. 일반적인 조언은 정말로 전역 상태 관리가 필요할 때만 사용하라는 것이다.

이 포스팅에서는 전역 상태 관리 라이브러리들이 해결해야하는 핵심 문제들에 대해 자세히 다뤄볼 것이다.

이러한 근본적인 문제점들을 이해하면 상태 관리 접근법들의 새로운 흐름이 만들어낸 방법들을 평가하는 데 도움이 될 것이다. 대부분의 경우, 로컬 상태 관리에서 시작하여 필요할 때 확장하는 것이 낫다.

**React**는 애플리케이션 전역에서 공유하는 상태에 대해 어떤 명확한 가이드라인도 제공하고 있지 않다. 때문에 **React** 생태계는 이러한 전역 상태 관리 문제를 해결하기 위해 수 많은 접근법과 라이브러리들을 수집해왔다.

이런 상황은 전역 상태 관리 문제를 해결하기 위해 어떤 라이브러리나 패턴을 선택해야 할지 헷갈리도록 혼란을 일으킬 수 있다.

이럴 때 일반적인 접근 방법은 가장 유명한 것을 사용하거나 채택하는 것이다. 이러한 접근 방법으로 인해 이전에는 많은 애플리케이션에서 **Redux**가 필요하지 않았음에도 **Redux**를 선택해서 사용하고 있었다.

상태 관리 라이브러리들이 갖는 문제점을 이해하면 많은 상태 관리 라이브러리들이 어째서 서로 다른 접근법을 가지게 됐는지 이해할 수 있다.

이러한 접근법들은 문제들에 대해 다른 절충안을 만들어내기 때문에 상태 관리에 대한 수 많은 개념 모델, 패턴 그리고 다양한 **API**들을 이끌어 냈다.

우리는 이 포스팅에서 **Recoil**, **Jotai**, **Zustand**, **Valtio**와 같은 라이브러리에서 찾을 수 있는 상태 관리의 모던한 접근 방식 및 패턴에 대해 알아보고 **React Tracked**나 **React Query**와 같은 다른 라이브러리가 어떻게 변화하는지 그리고 어떤 식으로 끊임없이 진화하는 환경에 부합하는지 살펴볼 것이다.

# The problems global state management libraries need to solve

### 1. 컴포넌트 트리 어디에서도 저장된 상태값을 읽을 수 있는 기능

> 이는 상태 관리 라이브러리의 가장 기본이 되는 기능이다.

이러한 기능은 개발자들이 그들이 관리하고자 하는 상태를 메모리에서 유지할 수 있도록 해주고 `prop drilling` 이슈를 피할 수 있게 해준다. **React** 생태계 초기에는 이러한 기능을 얻기 위하여 불필요하더라도 **Redux**를 이용하곤 했다.

실전에서 상태를 실제로 저장해야 할 때엔 두 가지 주요한 접근 방식 있다.

첫 번째는 **React 런타임 내에 상태를 저장하는 방법**이다. 이 말인 즉슨 공유하고자 하는 값들을 전파하기 위해 **React**에서 제공하는 `useState`, `useRef` 그리고 `useReducer`를 **React Context**를 결합하여 활용하는 것을 의미한다. 이 방법의 주요 과제는 리렌더링 최적화이다.

두 번째는 **React의 라이프 사이클 밖에서 즉, 모듈 상태(module state)로 상태값을 저장하는 방법**이다. 모듈 상태를 이용하면 싱글톤 패턴과 유사한 형태로 상태를 저장할 수 있다. 이 방법은 상태값이 변경될 때 리렌더링이 발생해야함을 알리는 **subscription**을 이용하기 때문에 리렌더링을 최적화하기 더 쉽다. 하지만 메모리 내에 단일 값이기 때문에 다른 **subtree**를 위한 다른 상태값을 가질 수 없다.

### 2. 저장된 상태값을 작성(업데이트 혹은 생성)할 수 있는 기능

> 라이브러리는 스토어에 저장된 데이터를 읽고 쓸 수 있는 직관적인 API를 제공해야만 한다.

An intuitive API is often one that fits ones existing mental models. 따라서, 라이브러리를 사용하는 사람이 누구냐에 따라 다소 주관적일 수 있다.

이러한 `mental model`들의 충돌은 라이브러리 채택에 있어서 마찰을 일으킬 수 있고 러닝 커브를 높이는 원인이 되기도 한다. **React** 내에서 대표적인 `mental model`의 충돌은 `mutable state`와 `immutable state`의 대립이다.

**React**의 상태 함수로서의 UI 모델은 참조적으로 평등하고 불변성을 지키는 업데이트에 의존하는 개념에 적합하기 때문에 상태값이 변경되는 것을 감지하여 올바르게 리렌더링을 진행할 수 있다. 그에 반해 **JavaScript**는 `mutable language`이다.

우리는 **React**를 사용할 때 `reference equality`과 같은 것들을 염두해두어야 한다. 이는 이러한 개념이 익숙하지 않거나 **React**에 대해 어느 정도 학습이 이루어지지 않은 **JavaScript** 개발자에게는 혼란의 원인이 될 수 있다.

**Redux**는 이러한 모델을 따르며 **Redux** 내에서 이루어 지는 모든 상태 업데이트는 불변성을 지키는 업데이트가 되어야만 한다. There are trade-offs with choices like this. 이 경우 일반적인 불만은 `mutable style update`에 사용되는 업데이트를 만들어 내기 위해 작성해야 하는 **boilerplate**의 양이다.

> **Redux**를 사용하면 불변성을 지키는 상태 업데이트를 만들어야 하기 때문에 기존 **JavaScript** 흐름과는 조금 다르게 코드를 작성해야 하는데(ex. 객체 내에 특정 속성 하나만 업데이트 불가) 여기서 꽤나 많은 양의 추가 코드가 발생

> 그렇기에 **Redux Toolkit**는 **Immer** 아래에서 동작한다.

이것이 개발자에게 `mutable style code`를 작성할 수 있게 해주는 **Immer**와 같은 라이브러리가 인기가 많은 이유이다(even if under the hood updates are immutable).

전역 상태 관리 라이브러리의 새로운 흐름 중에서는 **차세대 리덕스**라고 불리며 개발자들에게 `mutable style` API를 사용할 수 있게 해주는 **Valtio**가 있다.

### 3. 렌더링을 최적화하기 위한 mechanisms을 제공해야 한다.

> 상태 함수로서의 React UI 모델은 놀라울 정도로 단순하고 생산적이다.

그러나 상태를 업데이트할 때 발생하는 `reconciliation`의 과정은 매우 비용이 높다. 그리고 `reconciliation`는 종종 대규모 애플리케이션에서 런타임 퍼포먼스를 저하시킨다.

이 모델에서 전역 상태 관리 라이브러리는 상태가 업데이트될 때 리렌더링할 시기를 감지하고 필요한 항목만 리렌더링해야 한다.

이 과정을 최적화시키는 것은 상태 관리 라이브러리가 풀어야할 가장 큰 문제 중 하나이다.

이 문제에 자주 사용되는 두 가지 주요 접근 방법이 있다. 첫 번째로 위 과정의 최적화를 해당 라이브러리를 사용하는 개발자에게 맡기는 방법이 있다.

이러한 수동 최적화의 예로는 `selector` 함수를 통하여 저장된 상태의 일부를 구독하는 것이다. `selector`를 통해 상태를 읽고 있는 컴포넌트들은 특정 상태의 일부가 업데이트될 때만 리렌더링이 발생할 것이다.

두 번째는 라이브러리 사용자들이 이러한 수동 최적화를 고려하지 않도록 라이브러리가 자동으로 이러한 최적화를 다뤄주는 방법이다.

**Valtio**는 이러한 라이브러리 예 중 하나로 `Proxy` 아래에서 자동으로 업데이트 시기를 추적하고 컴포넌트 리렌더링을 관리한다.

### 4. 메모리 사용에 대한 최적화 mechanisms을 제공해야 한다.

매우 큰 규모의 프론트엔드 애플리케이션의 경우 메모리 사용을 제대로 관리하지 않으면 나중에 문제가 될 수 있다.

특히나 매우 큰 규모의 애플리케이션을 저사양 스펙의 기기로 접근하려는 고객이 존재하는 경우 더 문제가 될 소지가 있다.

상태를 저장하기 위해 **React**의 라이프 사이클에 연결한다는 말은 컴포넌트가 `unmount`될 때 자동으로 가비지 컬렉팅되는 것을 이용하겠다는 말이다.

단일 전역 스토어의 패턴으로 알려진 **Redux**와 같은 라이브러리의 경우 이를 직접 관리해야 한다. 이러한 라이브러리는 데이터가 자동으로 가비지 컬렉팅되지 않도록 데이터의 참조를 계속해서 유지할 것이기 때문이다.

마찬가지로, **React** 런타임 외부의 상태를 모듈 상태로 저장하는 상태 관리 라이브러리를 사용한다는 것은 특정 컴포넌트에 연결되지 않고 수동으로 관리해야 할 수도 있음을 의미한다.

> **React**의 라이프 사이클 내에서 상태를 관리하는 것이 아니기 때문에 이러한 가비지 컬렉팅이 이루어지도록 신경써야 할 수도 있다는 것을 의미

## More problems to solve

> 위에서 설명한 기본적인 문제 외에도, **React**와 통합할 때 고려해야할 일반적인 문제들이 더 있다.

### concurrent mode(이하 동시성 모드)와의 호환성 문제

**동시성 모드**는 **React**가 렌더링 단계에서 하던 일을 잠시 멈추고 우선 순위를 바꿀 수 있게 해준다. 이전에는 이 과정이 완전히 동기적이었다.

무엇이 됐든 동시성을 도입하면 엣지 케이스가 발생한다. 상태 관리 라이브러리의 경우 렌더링 단계에서 읽은 값이 변경되면 컴포넌트들이 외부 저장소로부터 다른 값을 읽을 가능성이 생긴다.

이는 `tearing`이라고 알려진 문제이다. 이 문제로 인해 React 팀은 `useSyncExternalStore` 훅을 만들었고 이를 통해 라이브러리 개발자들이 문제를 해결할 수 있도록 했다.

### Serialization of data

완전히 직렬화할 수 있는 상태를 유지하여 저장소 어딘가에 애플리케이션의 상태값을 저장하고 복원할 수 있게 된다면 이는 어디선가 유용하게 사용할 수도 있을 것이다. 일부 라이브러리들에서는 이러한 직렬화를 핸들링 해주고 있지만 다른 라이브러리들에서는 직렬화를 위해 추가적인 작업이 필요할 수도 있다.

### The context loss problem

이 문제는 애플리케이션에서 다수의 React 렌더러를 함께 사용할 때 발생한다. 예를 들어 애플리케이션에서 `react-dom`과 `react-three-fiber`와 같은 라이브러리를 모두 활용하는 경우 **React**는 분리된 두 context를 `reconcile` 할 수 없다.

> [2018년에 React 깃허브 이슈](https://github.com/facebook/react/issues/13332)로 이미 올라와 있는 문제이다. 중첩 렌더러를 사용 중인 경우 하나의 렌더러에서 다른 하나의 렌더러의 컨텍스트를 읽을 수 없는 문제이다.

### The stale props problem

훅은 클래스 컴포넌트가 가지고 있던 많은 이슈들을 해결했다. 하지만 클래스 컴포넌트 대신 함수 컴포넌트를 사용하게 되면서 받아들인 클로저 개념은 다른 문제들을 야기했다.

대표적인 문제 중 하나는 현재 렌더링 사이클에서 클로저 내부의 데이터는 더 이상 최신 상태가 아니라는 것이다. 최신 값이 아닌 화면에 렌더링되는 데이터로 이어진다. (Leading to the data that is rendered out to the screen not being the latest value.) 이는 상태를 계산하기 위해 `props`에 의존하는 `selector` 함수를 이용하는 경우 문제가 될 수 있다.

### The zombie child problem.

이는 부모 컴포넌트가 `mount` 되기 전에 상태값이 업데이트되는 경우 자식 컴포넌트가 부모 컴포넌트보다 먼저 자신을 `mount`하고 저장소에 연결함으로 인해 불일치를 일으킬 수 있는 **Redux**의 오래된 이슈를 말한다.

# A brief history of the state management ecosystem

지금까지 전역 상태 관리 라이브러리들이 고려할 필요가 있는 다양한 문제들과 엣지 케이스들에 대해 살펴봤다.

**React** 상태 관리에 대한 모든 현대적 접근 방식을 더 잘 이해하기 위해. 과거의 고통이 오늘날 우리가 "**Best Pratice**"라고 부르는 교훈으로 이어진 방법을 알아보기 위해 걸어온 길을 되돌아볼 필요가 있다.

이러한 **Best Pratice**들은 주로 다양한 시행착오를 통해 발견됐었다. And from finding that certain solutions don’t end up scaling well.

처음부터, **React**의 태그라인은 `Model` `View` `Controller` 중에 `View`였다.

그렇기에 **React**는 상태를 어떻게 구조화하고 관리할지에 대해 아무런 정보가 없었다. 이는 프론트엔드 애플리케이션 개발에 있어서 가장 복잡한 부분을 다룰 때 개발자들이 이를 해결하기 위해 혼자 씨름해야 했음을 의미했다.

페이스북 내부에서는 "`Flux`"라고 불리는 패턴이 사용되었는데, 이는 **React**의 "`always re-render`"하는 모델과 일치하는 단방향 데이터 흐름과 예측 가능한 업데이트에 적합했다.

`Flux` 패턴은 **React**의 `mental model`에 잘 들어맞았기 때문에 **React** 생태계 초기에 이 패턴이 자리잡게 되었다.

## The original rise of Redux

**Redux**는 당시 널리 채택된 **Flux** 패턴으로 구현된 라이브러리 중 하나였다.

**Redux**는 다른 **Flux** 구현체들이 일반적으로 취하고 있는 스토어의 형태와 다르게 `Elm architecture`에서 영감을 받은 단일 스토어의 형태를 취하고 있었다.

새롭게 프로젝트를 시작하는 경우에 상태 관리 라이브러리로 **Redux**를 선택한다고 해고당하는 일이 발생하진 않을 것이다. 더군다나 **Redux**는 `undo`와 `redo`를 구현한 기능을 사용할 수 있고 이를 통해 시간에 구애받지 않고 디버깅이 가능하다.

> **Redux Devtools**를 이용하면 특정 `action`이 실행됐던 시점으로 돌아갈 수도 있고 `action`을 처음부터 다시 실행시키고 이를 지켜볼 수 있는 등 시간 여행과 같은 디버깅을 할 수 있다.

**Redux**의 전반적인 모델링은 여전히 단순하면서 훌륭하다. 특히 `Backbone`과 같이 이전에 선행했던 이전 세대의 MVC 스타일 프레임워크들과 비교하면 더욱 더 그렇다.

> 불변 데이터 컨셉으로 무결성을 보장하고 side-effect 없이 상태를 업데이트하도록 강제한 것은 분명 Redux의 장점이다. 하지만 이를 위해 디스패쳐를 이용한 유연한 업데이트 방식을 포기하고, 중앙집중형 스토어를 구현함으로써 뷰가 데이터에 접근하는 방식을 무척 번거롭게 만든 것도 사실이다.

**Redux**는 여전히 특정 애플리케이션에서의 실제 사용 사례가 있는 훌륭한 상태 관리 라이브러리이다. 다만, 시간이 지남에 따라 **Redux**에 대한 몇 가지 일반적인 불만 사항이 표면화되었고 이러한 불만 사항들이 커뮤니티에서 더 많은 것을 알게 되면서 사람들이 **Redux**를 점점 선호하지 않게 되었다.

### Issues in smaller apps

많은 애플리케이션들이 초기에 **Redux**를 사용하여 첫 번째 문제(위에서 언급한 `prop-drilling`과 같은 문제들)를 해결했다. 트리의 어느 곳에서나 저장된 상태에 접근하여 해당 데이터를 여러 수준으로 업데이트하기 위해 데이터와 함수 모두를 `prop-drilling`하는 고통을 피할 수 있었다.

다만, 사용자와의 상호 작용이 별로 없고 `Endpoint`에 대한 데이터 패칭이 별로 없는 간단한 애플리케이션에서 **Redux**는 다소 과한 경향이 있었다.

### Issues in larger apps

시간이 지남에 따라 규모가 작았던 애플리케이션은 점점 규모가 커진다. 규모가 커진 프론트엔드 애플리케이션에서는 다양한 타입의 상태를 갖게 되기 마련인데 이는 각각 다른 하위 문제들을 만들어 낸다.

> `로컬 UI 상태`, `원격 서버 캐시 상태` 그리고 `전역 상태` 등 이외에도 다양한 타입의 상태가 존재할 수 있다.

예를 들어, `로컬 UI 상태`의 경우 상태를 업데이트하기 위해 데이터나 메서드를 `prop-drilling` 하는 것은 규모가 커질 수록 상대적으로 더 빠르게 문제가 되는 경우들이 있다. To solve this, using **[component composition patterns](https://frontendmastery.com/posts/building-future-facing-frontend-architectures)** in combination with **[lifting state up](https://reactjs.org/docs/lifting-state-up.html)** can get you pretty far.

> **`lifting state up`**: 컴포넌트들의 공통 조상(부모)로 상태를 끌어올리는 것

`원격 서버 캐시 상태`의 경우 `중복 요청 제거`, `재시도`, `Polling`, `Mutation 핸들링` 등 다양한 문제들이 존재한다.

애플리케이션이 커짐에 따라 다양한 타입의 상태가 생겨날텐데 **Redux**는 이러한 타입에 상관없이 모든 타입의 상태를 하나의 스토어 안에 집어넣으려고 하는 경향이 있다.

이러한 경향은 하나의 큰 단일 저장소에 모든 상태를 저장하게 만드는데 이는 런타임 성능 최적화의 두 번째 문제를 악화시키곤 했다.

**Redux**는 전역 공유 상태를 일반적으로 핸들링하기 위해 지속적으로 이러한 많은 하위 문제들을 해결해왔다. (또는 종종 방치했다)

이로 인해 UI와 원격 엔터티 상태 사이의 모든 것을 한 곳에서 관리하는 거대한 중앙집중형 저장소가 발생한다.

이러한 구조는 규모가 커지면 커질 수록 매우 관리하기 어려워지며 특히 프론트엔드 개발자들이 빠르게 출시를 해야하는 팀인 경우에 더욱 더 어려워진다. Where working on decoupled independent complex components becomes necessary.

## The de-emphasis of Redux

이러한 고충이 더 많이 발생함에 따라 시간이 지남에 따라 새 프로젝트를 시작할 때 **Redux**를 기본값으로 사용하는 것이 권장되지 않게 되었다.

실제로 많은 웹 애플리케이션은 주로 프론트엔드를 원격 상태 데이터(서버 데이터)와 동기화해야 하는 `CRUD`(`create`, `read`, `update` and `delete`) 스타일의 애플리케이션이다.

**즉, 시간을 주로 쏟는 주요 문제점들은 원격 서버 캐시 문제들이다.** 이러한 문제에는 서버 상태를 가져오고 캐시하고 동기화하는 방법들이 포함된다.

또한 `handling race conditions`, `invalidating and refetching of stale data`, `de-duplicating requests`, `재요청`, `refetching on component re-focus`, 그리고 일반적으로 **Redux**와 관련된 **boilerplate**에 비해 원격 데이터의 변형이 용이하다는 등의 많은 문제를 포함한다.

이러한 **boilerplate**는 불필요하고 과하게 복잡하다. 특히 `redux-saga`나 `redux-observable`과 같은 미들웨어 라이브러리를 **Redux**와 결합할 때 더욱 더 그러하다.

> 실제로 두 라이브러리와 **Redux**를 결합해서 사용해본 경험을 토대로 하면 **boilerplate**의 양은 생각 이상으로 많고 쉽게 사용하지 못할 만큼 꽤나 복잡하다.

이 도구들은 이러한 유형의 애플리케이션에서 너무 과하다. 클라이언트의 `fetching`과 `mutations` 측면에서 과할 뿐만 아니라 비교적 간단한 작업에 사용되는 모델의 복잡성에서도 마찬가지이다.

## The pendulum swing to simpler approaches

**hooks**와 **Context API**가 새롭게 생겨났다. 한동안 **Redux**와 같은 무거운 추상화에서 새로운 **hooks** API로 native context를 활용하는 방향으로 돌아갔다. 여기엔 `useState` 또는 `useReducer`와 결합된 간단한 `useContext`가 포함된다.

이 방식은 작은 애플리케이션에서는 괜찮은 접근 방식이었다. 그렇기에 많은 소규모 애플리케이션들이 이 방식을 통해 문제를 해결했다. 문제는 역시나 애플리케이션의 규모가 커지면서 발생했고 문제들은 다음과 같다.

### 1. Re-inventing Redux

종종 우리가 이전에 정의한 많은 문제에 빠지기도 한다. 그리고 그것들을 해결하지 않거나, 특정 엣지 케이스를 해결하는 데 전념하는 라이브러리와 비교하여 기존 문제들을 제대로 해결하지 못하게 된다. Leading many feeling the need to the promote the idea that **[React context has nothing to do with state management](https://blog.isquaredsoftware.com/2021/01/context-redux-differences/)**.

### 2. Optimizing runtime performance

또 다른 주요 문제는 리렌더링 최적화 문제이다. native context(Context API)를 사용할 때 규모가 커짐에 따라 원하는 바를 이루기 어려울 수도 있다.

이 문제의 해결을 돕기 위해 설계된 `useContextSelector`와 같은 최신 유저 라이브러리 라이브러리에 주목할 필요가 있다. With the React team starting to look at **[addressing this pain point automatically in the future as part of React](https://github.com/reactjs/rfcs/pull/119#issuecomment-586390430)**.

> I’m pretty convinced that the big problem we see today doesn't have to do with the propagation mechanism of the Context. The main problem that happens in practice is that we end up rendering beyond the component reading the Context because that component creates a new object (e.g. JSX element) that isn't referentially identical to the one before. So this would be solved if everything was auto-memoized.
>
> However, since we're probably not going to have predictable auto-memoization in a reasonable time frame. It's probably better to introduce this as a stop-gap.
>
> The nice thing is that this could be implemented either with the lazy propagation, or as a late bailout which would be closer to the final variant.
>
> Regardless of implementation, in terms of API, I think we should implement Context selectors. I think we can just use the second argument to `useContext` and get rid of changed bits though.
>
> 링크에서 설명된 컨트리뷰터의 말을 정리하면 이러한 리렌더링 최적화 문제가 Context API의 설계 문제는 아니라는 것이다. 참조에서 발생하는 문제이며 `memoization`이 모든 부분에서 이루어지는 순간 자연스레 해결될 수 있다고 보는 것 같다.
>
> 다만, 지금 당장 해결할 수 있는 문제는 아니기에 `useContextSelector`와 같은 커스텀 훅들이 필요할 수 있다고 말하고 있다.

## The rise of purpose built libraries to solve the remote state management problem

**CRUD** 스타일 애플리케이션인 대부분의 웹 애플리케이션의 경우 로컬 상태를 원격 상태 관리 라이브러리와 결합하면 훨씬 더 많은 이점을 얻을 수 있다.

이런 원격 상태 관리 라이브러리의 요즘 트렌드는 **React Query**, **SWR**, **Apollo** 그리고 **Relay**를 예로 들 수 있다. 또한 **Redux Toolkit**, **RTK Query**와 함께 개량된 Redux도 하나의 예시이다.

이러한 라이브러리들은 **Redux**만을 사용하여 구현하기에는 너무 번거로운 원격 데이터 문제를 해결하기 위해 만들어졌다.

이러한 라이브러리들은 **SPA**들에게 좋은 `abstractions`다. 다만, 이런 라이브러리들은 데이터를 `fetching`하고 `mutation`하는 데에 **JavaScript** 측면에서 오버헤드를 발생시킨다.

> 실제로 **React Query v3**에선 `notifyOnProps: "tracked"`라는 좋은 옵션이 있지만 오버헤드 방지를 위해 기본적으로 false 처리되어있다. 오버헤드 문제를 개선한 것인지 **v4**에서는 기본 옵션으로 바뀌었다. (물론 옵션은 해제 가능하다)

그리고 웹 빌더들의 커뮤니티에게서 **JavaScript**의 실제 비용은 점점 더 중요해지고 있다.

**Remix**와 같은 최신 메타 프레임워크에서 이러한 문제를 해결했다는 사실을 주목하자. Remix에서는 전용 라이브러리를 다운로드할 필요 없이 server-first 데이터 로딩과 선언적 mutation에 대한 추상화를 제공함으로써 이러한 문제를 해결했다.

> By providing abstractions for server-first data loading and declarative mutations that don’t require downloading a dedicated library.
>
> **Remix** 문서나 컨셉을 이해하지 않는 이상 무슨 말인지 정확하게 해석할 수 없을듯

Extending the “UI as a function of state” concept **[beyond the just the client](https://remix.run/blog/remix-data-flow)** to include the backend remote state data.

> [https://remix.run/blog/remix-data-flow](https://remix.run/blog/remix-data-flow) 참조

# The new wave of global state management libraries and patterns

대규모 애플리케이션의 경우 원격 서버 상태와 구별되는 공유 전역 상태를 유지해야 하는 경우가 많다.

### The rise of bottom up patterns

우리가 이전에 살펴본 Redux와 같은 상태 관리 솔루션은 `하향식`이라고 볼 수 있다. 이런 `하향식`은 시간이 지남에 따라 컴포넌트 최상단에서 모든 상태를 가지고 있으려는 경향이 있다. 상태값은 트리 상단에 있고 하위 컴포넌트가 `selector`를 통해 필요한 상태를 끌어오는 구조이다.

In **[Building future facing frontend architectures](https://frontendmastery.com/posts/building-future-facing-frontend-architectures)** we saw the usefulness of the bottom-up view to constructing components with composition patterns.

> 위 포스팅을 보게 되면 합성 패턴과 함께 상향식으로 짜여진 컴포넌트 구조의 유용성을 알 수 있다.

Hooks both afford and promote the same principle of composable pieces put together to form a larger whole.

> 무슨 말인지 이해가 가질 않는다….

**hooks**를 이용하면 거대한 전역 스토어를 통해서도 단일 상태 관리 접근법에서 변화를 만들어 낼 수 있다.

> With hooks we can mark a shift from monolithic state management approaches

**hooks**를 통해 접근하는 더 작은 상태 조각(원자 단위)에 중점을 둔 상향식 `micro` 상태 관리를 지향한다.

> Hooks both afford and promote the same principle of composable pieces put together to form a larger whole. With hooks we can mark a shift from monolithic state management approaches with a giant global store. Towards a bottom-up “micro” state management with an emphasis of smaller state slices consumed via hooks.
>
> 직역하면 당최 무슨 말인지 이해가 안가는데 의역을 많이 섞어보면
>
> 훅을 기반으로 하는 상태 관리 접근법들은 아주 작은 단위에 중점을 둔 상향식 `micro` 상태 관리를 지향하며 훅을 이용한다면 거대한 전역 스토어도 `monolithic`한 상태 관리가 가능하다는 말인 것 같다.

**Recoil**과 **Jotai**와 같은 유명한 라이브러리들이 `atomic` 상태 컨셉을 바탕으로 이러한 상향식 접근 방식을 보여준다.

`atom`은 작지만 완전한 상태 단위이다. `atom`들은 새로운 파생 상태를 형성하기 위해 서로 연결될 수 있는 작은 상태의 조각들이다. 이러한 `atom`들을 마침내 그래프를 형성한다.

이 모델을 사용하면 상태를 상향식으로 점진적으로 구축할 수 있다. 그리고 업데이트된 그래프의 `atom`만 `invalidate`하여 리렌더링을 최적화한다.

> 상향식 컴포넌트 구조에 대한 이해가 필요할듯

이것은 사용자가 구독하고 불필요한 리렌더링을 피하려고 하는 하나의 큰 단일 상태 스토어의 형태와 대조적이다.

## How modern libraries address the core problems of state management

다음은 새로운 흐름에 해당하는 라이브러리들이 상태 관리의 주요 문제들을 해결하기 위해 각 문제들을 대하는 서로 다른 접근법들을 간단하게 요약한 것이다. 여기서 말하는 주요 문제들이란 이 포스팅의 처음에 정의한 문제들과 동일하다.

### Ability to read stored state from anywhere within a subtree

| Library     | Description     | Simplified API example                                                                            |
| ----------- | --------------- | ------------------------------------------------------------------------------------------------- |
| React-Redux | React lifecycle | `useSelector(state ⇒ state.foo)`                                                                  |
| Recoil      | React lifecycle | `const todos = atom({ key: ‘todos’, default: [] })`<br />`const todoList = useRecoilValue(todos)` |
| Jotai       | React lifecycle | `const countAtom = atom(0)`<br />`const [count, setCount] = useAtom(countAtom)`                   |
| Valtio      | Module state    | `const state = proxy({ count: 0 })`<br />`const snap = useSnapshot(state)`<br />`state.count++`   |

### Ability to write and update stored state

| Library     | Update API    |
| ----------- | ------------- |
| React-Redux | Immutable     |
| Recoil      | Immutable     |
| Jotai       | Immutable     |
| Zustand     | Immutable     |
| Valtio      | Mutable style |

### Runtime performance re-render optimizations

**Manual optimizations**는 주로 특정 상태의 일부를 구독하기 위한 `selector` 함수의 생성을 의미한다. 이 방법의 **장점**은 사용자가 구독 방법을 세밀하게 제어하고 해당 상태를 구독하는 구성 요소가 리렌더링되는 방법을 최적화할 수 있다는 것이다. 이 방법의 **단점**은 이 프로세스가 수동 프로세스이므로 오류가 발생하기 쉽다는 것이며 API의 일부가 되어서는 안 되는 불필요한 오버헤드가 필요하다고 주장할 수 있다.

**Automatic optimizations**은 라이브러리가 리렌더링 과정을 필요한 곳에 자동으로 사용자를 위해 최적화해주는 것을 말한다. 이 방법의 장점은 우선 사용하기 쉽고 사용자가 수동 최적화에 대해 걱정할 필요 없이 기능 개발에만 초점을 맞출 수 있다는 점이다. 이 방법의 단점은 사용자 입장에선 이러한 최적화 프로세스가 블랙박스라는 것이고 수동으로 최적화하기 위한 장치가 없다면 너무 마술처럼 느껴질 수도 있다는 것이다.

| Library     | Description                                |
| ----------- | ------------------------------------------ |
| React-Redux | Manual via selectors                       |
| Recoil      | Semi-manual through subscriptions to atoms |
| Jotai       | Semi-manual through subscriptions to atoms |
| Zustand     | Manual via selectors                       |
| Valtio      | Automatic via Proxy snapshots              |

### Memory optimizations

메모리 최적화는 매우 큰 애플리케이션에서만 문제가 되는 경향이 있다. 여기서 가장 중요한 부분은 라이브러리가 모듈 수준에서 또는 **React** 런타임 내에서 상태를 저장하는지 여부에 달려 있으며 스토어를 어떻게 구성하느냐에 따라 다르다.

거대한 `monolithic` 스토어에 비해 작고 독립된 스토어가 갖는 이점은 구독한 컴포넌트들이 `unmount` 될 때 자동으로 가비지 컬렉팅될 수 있다는 점이다. 이에 반해 거대한 `monolithic` 스토어는 적절한 메모리 관리가 없을 경우 메모리 누수가 발생할 가능성이 있다.

| Library | Description                                                                      |
| ------- | -------------------------------------------------------------------------------- |
| Redux   | Needs to be managed manually                                                     |
| Recoil  | Automatic - as of v0.3.0                                                         |
| Jotai   | Automatic - atoms are stored as keys in a WeakMap under the hood                 |
| Zustand | Semi-automatic - API’s are available to aid in manually unsubscribing components |
| Valtio  | Semi-automatic - Garbage collected when subscribing components unmount           |

# Concluding thoughts

전역 상태 관리 라이브러리 중 무엇이 최고인지에 대해선 정답은 없다. 왜냐하면 많은 부분이 특정 애플리케이션이 필요로 하거나 누가 만드냐에 따라 달려있기 때문이다.

상태 관리 라이브러리들이 해결해야 하는 변하지 않는 문제들을 이해하면 현재 또는 미래에 개발될 라이브러리들을 비교하고 평가하는데 도움이 될 것이다.

Going into depth on specific implementations is outside the scope of this article. If you’re interested to dig deeper I can recommend **[Daishi Kato’s](https://twitter.com/dai_shi)** **[React state management book](https://github.com/PacktPublishing/Micro-State-Management-with-React-Hooks)**, which is a good resource that delves deeper into specific side-by-side comparisons of some of the newer libraries and approaches mentioned in this post.
