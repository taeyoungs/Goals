# Building future facing frontend architectures

> 원본 글  
> https://frontendmastery.com/posts/building-future-facing-frontend-architectures/

컴포넌트 기반 프론트엔드 아키텍처가 규모에 따라 어떻게 복잡성이 급속도로 증가하는지 그리고 이를 어떻게 하면 피할 수 있는지 자세히 알아보자.

**목차**

- [Building future facing frontend architectures](#building-future-facing-frontend-architectures)
  - [개요](#개요)
  - [The influence of common mental models](#the-influence-of-common-mental-models)
  - [Thinking in components](#thinking-in-components)
    - [What is the one responsibility of this component?](#what-is-the-one-responsibility-of-this-component)
    - [What’s the absolute minimum, but complete, representation of its state?](#whats-the-absolute-minimum-but-complete-representation-of-its-state)
    - [Where should the state live?](#where-should-the-state-live)
  - [Top down vs bottom up](#top-down-vs-bottom-up)
  - [Building top down](#building-top-down)
    - [Where top down goes wrong](#where-top-down-goes-wrong)
  - [The organic growth of monolithic components](#the-organic-growth-of-monolithic-components)
    - [They arise through premature abstraction](#they-arise-through-premature-abstraction)
    - [They prevent code re-use across teams](#they-prevent-code-re-use-across-teams)
    - [They bloat bundle sizes](#they-bloat-bundle-sizes)
    - [They lead to poor runtime performance](#they-lead-to-poor-runtime-performance)
  - [Building bottom up](#building-bottom-up)
    - [How is a bottom up mental model different to top down?](#how-is-a-bottom-up-mental-model-different-to-top-down)
    - [What does a bottom up approach look like?](#what-does-a-bottom-up-approach-look-like)
    - [What’s the catch?](#whats-the-catch)
  - [Strategies for avoiding monolithic components](#strategies-for-avoiding-monolithic-components)
    - [Balancing single responsibility vs DRY.](#balancing-single-responsibility-vs-dry)
    - [Inversion of control](#inversion-of-control)
    - [Open for extension](#open-for-extension)
    - [Leveraging storybook driven development](#leveraging-storybook-driven-development)
    - [Some questions you can ask yourself when building the UI components in isolation that lead to resilient components](#some-questions-you-can-ask-yourself-when-building-the-ui-components-in-isolation-that-lead-to-resilient-components)
    - [Name components based on what they actually do](#name-components-based-on-what-they-actually-do)
    - [Avoid prop names that contain implementation details](#avoid-prop-names-that-contain-implementation-details)
    - [Be cautions of configuration via props](#be-cautions-of-configuration-via-props)
    - [Avoid defining components in the render method](#avoid-defining-components-in-the-render-method)
  - [Breaking down monolithic components](#breaking-down-monolithic-components)
  - [Concluding thoughts](#concluding-thoughts)
  - [**References**](#references)

## 개요

성능이 뛰어나고 변경하기 쉬운 프론트엔드 아키텍처를 구축하는 일은 규모 측면에서 쉬운 일은 아니다.

이 포스팅에선 많은 개발자와 팀이 같이 작업하는 프론트엔드 프로젝트에서 프로젝트의 복잡성이 빠르고 조용하게 증가할 수 있는 주요 요인들에 대해서 알아볼 것이다.

또한 이러한 복잡성에 매몰되지 않을 수 있는 효과적인 방법도 알아볼 것이다. 이러한 복잡성이 문제가 되기 전 그리고 기능을 추가하거나 변경해야 할 때 "이런, 어떻게 이렇게 복잡해질 수가 있지?"라고 생각하는 자신을 발견한 상황 모두에 대해서도 알아본다.

> We’ll also look at effective ways to avoid getting overwhelmed in that complexity. Both before it’s a problem, and after if you find yourself thinking “oh shit, how did this end up getting so complicated?” when you’re tasked to add or change a feature.

프론트엔드 아키텍처는 매우 다양한 측면을 가진 광범위한 주제다. 여기선 특히 변경 사항에 대해 쉽게 대응할 수 있는 유연한 프론트엔드를 제공하는 컴포넌트 코드 구조에 집중할 것이다.

이 포스팅에서 사용되는 예제에는 **React**가 사용되었다. 하지만 예제에서 말하고자 하는 원리 자체는 어떠한 컴포넌트 기반 프레임워크에도 적용할 수 있다.

아주 기초 단계부터 시작할 것이다. 시작은 코드가 작성되기도 전에 코드의 구조가 어떻게 영향을 받는지에 대해서다.

## The influence of common mental models

우리가 가진 **mental model**, 즉 우리가 사물(?)을 어떻게 생각하는지는 결국 우리의 결정에 큰 영향을 끼친다.

대규모 코드베이스에서는 꾸준히 내려지는 많은 의사결정들이 모여 하나의 전체적인 구조를 이룬다.

우리가 팀을 이룰 때, 팀이 가지고 있는 **mental model**을 명시하여 다른 사람들도 해당 **mental model**을 갖도록 만드는 것이 중요하다. 왜냐하면 모든 사람들은 대부분 자신만의 고유의 **mental model**을 가지고 있기 때문이다.

그렇기 때문에 팀은 보통 공유 스타일 가이드와 **prettier**와 같은 도구를 필요로 한다. 그래서 그룹으로서 우리는 어떻게 일관되어야 하는지, 우리가 하려는 건 무엇인지, 우리가 어디로 나아가야 하는지에 대한 공유된 **mental model**을 가져야 한다.

This makes life much easier. 이는 시간이 지남에 따라 모든 사람들이 자신만의 **mental model**의 영향으로 코드 베이스가 유지보수할 수 없게 되어버리는 일을 피할 수 있게 해준다.

출시를 열망하는 많은 개발자들에 의해 빠르게 개발되는 프로젝트를 경험했다면 적절한 가이드라인 없이 일이 얼마나 빨리 통제 불능이 될 수 있는지 봤을 것이다. 이렇게 되면 시간이 지남에 더 많은 코드가 추기되고 런타임 퍼포먼스가 저하되어 프론트엔드는 점점 더 느려질 수 있다.

앞으로 나올 몇 가지 섹션에선 다음의 질문들에 대해 답변하려고 한다.

1. **React**를 사용하고 있는 컴포넌트 기반 모델 프레임워크를 사용하여 프론트엔드 애플리케이션을 개발할 때 가지는 가장 일반적인 **mental model**은 무엇일까?
2. **mental model**이 컴포넌트를 구성하는 방식에 어떻게 영향을 미치는가?
3. 우리가 명시적으로 표현할 **mental model**이 내포하고 있는 **trade-off**엔 무엇이 있으며 이것들은 어떻게 복잡성을 급격하게 증가하게 만들까?

## Thinking in components

React는 가장 유명한 컴포넌트 기반 프론트엔드 프레임워크다. **React**를 처음 시작할 때 처음 읽게 되는 글은 주로 "[**Thinking in react**](https://reactjs.org/docs/thinking-in-react.html)"라는 포스팅이다.

이 포스팅에선 프론트엔드 애플리케이션을 구축할 때 "**the React way**"로 생각하려면 어떻게 생각하는지에 대한 핵심 **mental model**을 제시하고 있다. 해당 포스팅에 나오는 조언들은 모든 컴포넌트 기반 프레임워크에도 적용할 수 있기 때문에 이 포스팅은 확실히 좋은 포스팅이다.

포스팅에 나오는 주요 원칙들은 컴포넌트를 구성해야할 때마다 다음과 같은 질문을 할 수 있도록 만들어준다.

### What is the one responsibility of this component?

좋은 컴포넌트 API 설계는 자연스럽게 **composition pattern**에 대해 중요한 **단일 책임 원칙(single responsibility principle)**을 따른다. 간단한 것을 합치는 것은 쉽다. 그에 반해 새로운 요구사항이 들어오거나 변경됨에 따라 간단했던 것을 계속해서 간단하게 유지하는 것은 쉬운 일이 아니다.

> 쉽게 말해, 좋은 컴포넌트 API 설계를 바탕으로 만들어진 컴포넌트들은 여러 가지 일을 하지 않을 것이다. 컴포넌트 하나가 하나의 책임만을 가지는 형태로 구성이 되어 있을 것이다.
>
> 각 컴포넌트가 하나의 책임만을 가진 상태에서 이 컴포넌트들을 합치는 건 어려운 일이 아니다. 말 그대로 각자의 책임이 명확하고 여러 가지 일이 아닌 하나의 일만을 열심히 하는 작은 컴포넌트들을 합친다는 것은 레고 블럭을 합치듯이 착착 들어맞을테니
>
> 문제는 이렇게 단일 책임 원칙을 따르게 만들어 놓은 컴포넌트 설계가 "**새로운 요구사항이나 기존 요구사항이 변경되더라도 여전히 단일 책임 원칙을 따를 수 있게 만들 수 있냐**"다.

### What’s the absolute minimum, but complete, representation of its state?

이 아이디어가 말하고자 하는 것은 가장 작은 것부터 시작하는 게 더 낫다는 것이다. 가장 작긴 하지만 이는 우리가 만들어 낼 수 있는 다양한 변경으로부터 만들어 지는 상태값의 근간이 되어줄 것이다. 이러면 코드가 유연해지고 간단해지며 근간이 같은 상태값 하나를 업데이트 했는데 이와 같은 근간을 가진 다른 상태값은 업데이트되지 않는 것과 같은 일반적인 데이터 동기화 문제를 피할 수 있다.

> The idea is that it’s better to start with the smallest but complete source of truth for your state, that you can derive variations from.

### Where should the state live?

상태 관리는 현재 포스팅의 범위를 벗어나는 광범위한 주제다. 하지만 일반적으로 상태값을 컴포넌트 내부에 만들 수 있다면 그렇게 해야 한다. 내부적으로 전역 상태에 의존하는 컴포넌트가 많을수록 재사용 가능성이 낮아진다. 이 질문은 어떤 컴포넌트가 어떤 상태값에 의존해야 하는지 식별하는 데 유용하다.

Some more wisdom from the article:

> 컴포넌트는 이상적으로 하나의 일만을 수행해야 한다. 만약 컴포넌트의 규모가 커진다면 해당 컴포넌트를 더 작은 서브 컴포넌트들로 분해해야만 한다.
>
> **_A component should ideally only do one thing. If it ends up growing, it should be decomposed into smaller sub components._**

여기에서 설명하고 있는 원칙들은 간단하고 실전성이 있으며, 복잡성을 해결하는데 효과적이다. 이러한 원칙들은 컴포넌트를 만들 때 가장 일반적인 **mental model**에 근간이 된다.

간단하다는 말이 쉽다는 말은 아니다. In practice this is much easier said than done in the context of large projects with multiple teams and developers.

성공적인 프로젝트는 기본 원칙들을 잘 그리고 일관되게 고수하는 데서 나온다. 그리고 이러한 프로젝트들은 비용이 많이 발생하는 실수를 너무 많이 하지 않는다.

This brings up two questions we’ll explore.

1. 이러한 간단한 원칙들의 적용을 방해하는 상황들은 무엇일까?
2. 그리고 어떻게 하면 이러한 상황들을 최대한 완화할 수 있을까?

아래에서 시간이 지남에 따라 앞서 말했던 간단함(또는 단순성)을 유지하는 것이 실제로 쉽지 않은 이유들에 대해서 알아볼 것이다.

## Top down vs bottom up

컴포넌트들은 **React**와 같은 모던 프레임워크에서 추상화의 핵심 단위이다. 이러한 컴포넌트들의 생성에 대해 생각하는 두 가지 주요 방법이 있다. 다음은 **React**가 말하는 컴포넌트 생성에 대한 사고다.

> **top-down(하향식)** 또는 **bottom-up(상향식)** 방식으로 컴포넌트를 구축할 수 있다. 즉, 계층 구조에서 더 높은 단계에 위치한 컴포넌트를 구축하는 것으로 시작할 수 있다는 말이다. 간단히 말해서, 보통은 하향식으로 진행하는 것이 쉽고 이에 반해 대규모 프로젝트에서는 빌드하면서 상향식으로 테스트를 작성하는 것이 더 쉽다.
>
> **_You can build top-down or bottom-up. That is, you can either start with building the components higher up in the hierarchy. In simpler examples, it’s usually easier to go top-down, and on larger projects, it’s easier to go bottom-up and write tests as you build._**

More solid advice. 언뜻 보면 간단해 보인다. 마치 "단일 책임이 좋다(single responsibility is good)"를 읽는 것처럼 고개를 끄덕이고 넘어가기 쉽다.

그러나 **하향식 mental model**과 **상향식 mental model**의 차이는 겉으로 보이는 것보다 훨씬 더 중요하다. 적정 규모의 프로젝트에 두 **mental model** 중 하나를 컴포넌트를 구축하는 암묵적인 방법으로 선택했을 때 이 선택들은 매우 다른 결과로 이어진다.

## Building top down

위 인용문이 암시하는 것은 간단한 프로젝트에서 하향식 접근방법을 선택함으로써 프로세스 진행을 쉽게 진행하는 것과 큰 규모의 프로젝트에서 더 느리지만 더 확장 가능한 상향식 접근 방법을 선택하는 것 사이의 **trade-off**이다.

하향식은 일반적으로 가장 간단하고 직관적인 접근 방법이다. 필자의 경험상 하향식은 기능을 개발하는 개발자들이 컴포넌트를 구성할 때 가장 흔하게 가져가는 **mental model**이다.

**하향식 접근 방법은 어떤 모습일까?**

일반적으로 구축해야할 디자인이 주어졌을 때 해주는 조언은 "UI 주위에 박스를 그리면 그것이 컴포넌트가 될 것이다"이다.

이러한 형태는 우리가 만들어야 할 최상위 컴포넌트의 기반이 된다. 이 접근 방법을 사용하게 되면 처음부터 조잡한(coarse) 컴포넌트를 만들게 되는 경우가 많다. With what seems like the right boundaries to get started with.

새롭게 구축해야 할 관리자용 대시보드 디자인을 받았다고 해보자. 해당 디자인을 살펴보면서 어떤 컴포넌트를 만들어야 하는지 확인할 것이다.

디자인엔 새로운 사이드바 네비게이션이 있었기에 사이드바 주위에 상자를 그리고 개발자에게 새로운 `<SideNavigation />` 컴포넌트를 만들라고 이야기한다.

이러한 하향식 접근 방법을 따라서 필요한 `props`는 무엇이고 어떻게 렌더링해야 할지 생각해볼 수 있다. 백엔드 API로부터 네비게이션 항목들에 대한 리스트를 받는다고 가정해보자. 현재 하향식 모델을 따르고 있으므로 아래 pseudo code와 같은 초기 설계를 보는 것은 놀랍지 않을 것이다.

```jsx
// get list from API call somewhere up here
// and then transform into a list we pass to our nav component
const navItems = [
  { label: 'Home', to: '/home' },
  { label: 'Dashboards', to: '/dashboards' },
  { label: 'Settings', to: '/settings' },
]

...

<SideNavigation items={navItems} />
```

지금까지의 하향식 접근 방법은 상당히 간단하고 직관적으로 보인다. 우리의 의도는 컴포넌트를 사용하기 쉽고 재사용 가능하도록 만드는 것이며 해당 컴포넌트를 사용하는 사용자들이 렌더링하고자 하는 항목들을 그저 전달하기만 하면 `SideNavigation` 컴포넌트가 그 항목들을 처리해줄 것이다.

하향식 접근 방법에서 알아야할 유의 사항들은 다음과 같다:

1. 처음에 필요하다고 생각되는 컴포넌트로 인식한 최상단 영역에서 컴포넌트를 만들기 시작했다. 디자인에서 박스를 그려 컴포넌트를 표시했다.
2. 이렇게 그린 박스는 사이드 네비게이션 바와 관련된 모든 것을 처리하는 단일 추상화다.
3. 이러한 API는 이를 사용하는 사용자들이 작동하는 데 필요한 데이터를 아래로 내려주고 내부에서 모든 것을 처리한다는 의미에서 종종 "하향식"이라고 표현한다.

   컴포넌트가 백엔드 데이터를 직접적으로 렌더링하는 경우가 많기 때문에 하향식은 렌더링할 컴포넌트에 데이터를 아래로 전달하는 모델에 적합하다.

작은 규모의 프로젝트의 경우 이러한 접근 방법이 반드시 잘못된 것은 아니다. 문제는 대규모의 프로젝트일 경우다. 많은 개발자들이 함께 빠르게 작업하는 대규모의 코드베이스에서 하향식 **mental model**이 어떻게 빠르게 문제가 되는지 이제 살펴볼 것이다.

### Where top down goes wrong

하향식 접근 방법의 사고 방식은 당면한 문제를 먼저 해결하기 위해 특정 추상화에 의존하는 경향이 있다.

이러한 사고 방식은 직관적이다. 이는 컴포넌트를 구축하기 위한 가장 간단한(또는 쉬운) 접근 방법처럼 보인다. 또한 초기에 사용 편의성을 위해 최적화된 API로 이어지는 경우가 많다.

다음은 일반적인 시나리오다.

팀에 소속되어 있고 급하게 개발을 해야하는 프로젝트에 참여하고 있다고 해보자. 개발을 위해 상자를 그리고 스토리를 만든 뒤에 새로운 컴포넌트를 병합했다. 이 상황에 사이드 네비게이션 컴포넌트를 업데이트해야 하는 새로운 요구 사항이 발생했다.

상황이 빠르게 악화되기 시작한다. 이러한 상황은 일반적으로 거대한 모놀리식 컴포넌트의 생성으로 이어진다.

개발자는 변경 사항을 적용하기 위해 스토리를 선택한다. They arrive at the scene, ready to code. They’re in the context of the abstraction and API having already been decided.

Do they:

- **A**
  올바른 추상화인지 아닌지 생각해본다. 올바른 추상화가 아니라면 스토리에 표시된 작업을 수행하기 전에 해당 작업을 분해하여 이를 수행하지 않는다.
- **B**
  속성을 추가하고 해당 속성을 체크하는 간단한 조건 뒤에 새로운 기능을 추가한다. 새로운 `props`가 통과하는 몇 가지 테스트를 작성한다. It works and is tested. And as a bonus it was done fast.

As Sandy Mets puts it:

> 이미 존재하는 코드는 강력한 영향력을 발휘한다. 코드의 존재 자체만으로도 해당 코드가 정확하고 필요하다고 주장한다. 우리는 이러한 코드가 해당 코드에 투여된 우리의 노력을 나타낸다는 것을 알고 있기에 노력의 가치, 즉 이미 존재하는 코드들을 보존하려고 한다. 그리고 안타깝게도 코드가 더 복잡하고 이해할 수 없을 수록, 즉 해당 코드를 만드는 데 투자한 시간이 클 수록 우리는 이러한 코드를 유지해야 한다는 압박감을 더 느낀다는 것이다(the "sunk cost fallacy").

**sunk cost fallacy**(매몰 비용 오류?)는 우리가 자연적으로 손실을 피하는 데 더 민감하기 때문이다. 마감 시간이 촉박하거나 단순히 "the story point is a 1"일 때 우리(또는 동료)는 **A**를 선택하지 못할 확률이 높다.

At scale it’s these rapid culmination of these smaller decisions that add up quickly and start to compound the complexity of our components.

> 규모에 따라 이런 작은 의사결정들이 모이고 모여 컴포넌트를 더 복잡하게 만든다는 뜻인 것 같다. 🤔

불행히도 이제 "**Thinking in React**"에서 설명한 기본 원칙 중 하나를 지키는 데 실패했다. 하기 쉬운 일은 종종 단순함으로 이어지지 않는다. 그리고 우리를 단순함으로 이끌어 주는 건 다른 대안들에 비해 하는 것도 쉽지 않다.

- **Caveat**
  Again context matters here, if you’re in a rush to ship an MVP that was due yesterday, do what you have to do to keep the business or project alive. Technical debt is a trade off and situations call for taking it on. But if you’re working on a product with many teams contributing to it, that has a long term plan, thinking about effective decomposition through continual refactoring is critically important to longevity.

이제 이 일반적인 시나리오를 간단한 사이드바 네비게이션 예제에 적용해보자.

먼저 디자인 변경이 필요하다. 네비게이션 항목에 아이콘을 추가해야 하고 텍스트 사이즈를 다르게 가져야 하며 네비게이션 항목 중 일부는 SPA 페이지 전환이 아닌 링크가 되어야 한다.

In practice UI holds a lot of visual state. we also want to have things like separators, opening links in a new tab, some to have selected default state, and so on and so forth.

사이드바 컴포넌트에 네비게이션 항목들의 리스트를 배열로 내려주고 있기 때문에 네비게이션 항목들의 새로운 타입(새로운 요구사항으로 발생한 타입들)과 기존의 다양한 상태값들을 구별할 수 있는 새로운 속성들(새로운 요구사항 각각에 대한)을 추가해줄 필요가 있다.

So our type for our now might look something like with type corresponding to whether its a link or a regular nav item: **`{ id, to, label, icon, size, type, separator, isSelected }`** etc.

그리고 나서 `<SideNavigation />` 컴포넌트 내부에서 `type` 속성을 체크하여 이를 기반으로 네비게이션 항목을 렌더링할 것이다. 이런 작은 변화에 벌써부터 조금씩 냄새가 나기 시작한다.

여기서 문제는 이러한 **API**가 있는 하향식 컴포넌트 자체다. 하향식 컴포넌트는 **API**에 추가되는 요구사항의 변경점에 대응해야 하며, 전달된 내용을 기반으로 내부적으로 로직을 분기해야 한다.

> **_From little things big things grow_**

몇 주 후에 새로운 기능에 대한 요청사항이 발생했다. 해당 기능에 대한 요구사항은 네비게이션 항목을 클릭하면 해당 항목 아래에 중첩된 하위 네비게이션으로 전환할 수 있어야 하며 뒤로 가기 버튼을 사용하여 다시 메인 네비게이션 목록으로 돌아갈 수 있어야 한다는 것이었다. 또한 네비게이션을 관리하는 관리자가 네비게이션 항목들을 드래그 앤 드롭을 통하여 재정렬할 수도 있어야 한다.

요구사항에 따라 이제 목록을 중첩하고 하위 목록을 상위 목록과 연결하고 일부 항목을 드래그할 수 있는지에 대한 개념이 필요하다.

몇 가지 요구사항이 발생하고 나면 얼마나 컴포넌트가 복잡해지기 시작하는지 알 수 있다.

단순한 **API**를 가지고 비교적 단순한 컴포넌트로 시작한 것이 몇 번의 요구사항 변경으로 빠르게 다른 무언가로 성장해버린다. 일단, 개발자가 시간 내에 작업을 수행할 수 있다고 가정해보자.

이 시점에서 이 컴포넌트를 사용해야 하거나 수정해야 하는 다음 개발자 또는 팀은 복잡한 구성이 필요한 모놀리식 컴포넌트를 다루고 있다(이러한 모놀리식 컴포넌트에 대한 문서화는 제대로 이루어지지 않은 경우가 대부분이다).

> At this point the next developer or team who needs to use or adapt this component is dealing with a monolithic component that requires a complex configuration, that is (let’s be real) most likely poorly documented if at all.

"네비게이션 목록을 내려주고 컴포넌트가 나머지를 처리한다"라는 초기의 의도는 이 시점에서 역효과를 만들어 냈고 지금의 컴포넌트는 변경하기에 느리고 위험하다.

이 시점에서의 일반적인 시나리오는 만들었던 모든 것을 던져버리고 컴포넌트를 처음부터 다시 만드는 것이다. 이제 하향식 접근 방법을 통해 컴포넌트를 구축하는 프로세스에서 해결해야 할 문제와 **use-case**에 대해 이해했다.

## The organic growth of monolithic components

> 처음을 제외하고 모든 것은 하향식으로 구축되어야 한다.
>
> **_Everything should be built top-down, except the first time._**

위에서 본 것처럼 모놀리식 컴포넌트는 너무 많은 일을 하려고 하는 컴포넌트다. 이러한 컴포넌트들은 너무 많은 데이터를 가져가거나 `props`를 통하여 구성에 필요한 너무 많은 옵션들을 가져오며, 너무 많은 상태값을 관리하고 너무 많은 **UI**를 보여준다.

모놀리식 컴포넌트들은 간단한 컴포넌트로 시작하여 위에서 묘사한 것과 같이 자연스럽게 복잡성이 증가하여 시간이 지남에 따라 너무 많은 일들을 하게 된다.

처음에 간단한 컴포넌트로 시작한 것도 새 기능을 구축하기 위해 프로세스를 몇 번 반복(심지어 동일한 스프린트 내에서도)하고 나면 모놀리식 컴포넌트가 될 수 있다.

이런 상황이 팀이 빠르게 개발해야 하는 동일한 코드베이스에서 작업할 때 다수의 컴포넌트에서 발생하게 되면 프론트엔드는 빠르게 변경하기 어려워지고 사용자에게는 더 느려지게 된다.

다음은 모놀리식 컴포넌트가 소리없이 무너질 수 있는 다른 방법들이다.

### They arise through premature abstraction

모놀리식 구성 요소로 이어지는 또 다른 미묘한 문제가 있다. 소프트웨어 개발자로서 초기에 주입되는 몇 가지 일반적인 모델과 관련이 있다. 그 중에서도 특히 **DRY(don’t repeat yourself)** 원칙을 준수하는 것과 관련이 있다.

**DRY** 원칙이 초기에 주입된 상태로 컴포넌트가 구성된 곳에서 작은 양의 중복을 발견하게 되면 "중복이 많이 발생했으니 단일 컴포넌트로 추상화하기 좋겠다"라고 생각하기 쉽다. 그리고 이른 추상화에 돌입해버린다.

모든 것이 **trade-off**긴 하지만 잘못된 추상화보다 추상화가 없는 상태에서 복구하는 것이 더 쉽다. 그리고 아래에서 더 이야기 하겠지만 상향식 모델부터 시작하면 이러한 추상화에 유기적으로 도달할 수 있으므로 모놀리식 컴포넌트의 조기 생성을 피할 수 있다.

### They prevent code re-use across teams

본인이 속한 팀이 필요로 하는 것과 비슷한 것을 다른 팀에서 구현했거나 아니면 작업 중인 경우를 종종 발견하게 될 것이다.

대부분의 경우 다른 팀에서 구현했거나 구현 중인 것이 본인이 원하는 바와 90% 정도 같고 조금은 다를 것이다. 또는 해당 작업물의 전체를 사용하는 것이 아니라 작업물의 일부 만을 재사용하고 싶을 수도 있다.

만약 이러한 작업물이 앞에서 예제로 사용했던 `<SideNavigation />`과 같은 모놀리식 "all or nothing" 컴포넌트라면 작업물을 재사용하기 더 어려울 것이다. 이렇게 되면 다른 사람의 작업물을 리팩토링하거나 분해하는 위험을 감수하는 대신 처음부터 다시 구현하던가 포크해오는 게 더 쉬워져버린다. 이는 나중에 조금만 다르고 동일한 문제를 겪고 있는 다수의 중복된 컴포넌트로 이어진다.

### They bloat bundle sizes

적절한 때에 불러오고 파싱되고 실행하는 코드만을 허용하려면 어떻게 해야할까?

사용자에게 먼저 보여줘야 하기 때문에 더 중요한 컴포넌트들이 있다. 대규모 애플리케이션의 핵심 퍼포먼스 전략은 우선 순위에 기반한 "단계(phases)"에서 비동기로 로딩되는 코드를 조정하는 것이다.

이러한 비동기로 로딩되는 코드를 조정하기 위해 서버에서 렌더링될 컴포넌트를 선택 및 제외할 수 있는 기능을 제공하는 것 뿐만 아니라(이상적으로는 첫 번째 `paint` 단계에서 사용자가 실제로 보게될 컴포넌트들로만 가능한 빨리 서버 사이드 렌더링을 수행하기 때문이다) 가능하다면 코드의 로딩을 연기한다.

모놀리식 컴포넌트는 몸집이 큰 하나의 컴포넌트로써 모든 것을 로딩해야 하기 때문에 위와 같이 비동기로 로딩되는 코드를 조정하기 위한 작업들을 할 수 없게 막는다. 사용자가 진정으로 필요로 할 때 로딩되고 최적화할 수 있는 독립적인 컴포넌트들을 가지는 것보다 사용자들이 실제로 사용하는 만큼의 퍼포먼스 비용만 지불할 수 있게 하는 것이 중요하다.

> Rather than having independent components that can be optimized and only loaded when truly needed by the user. Where consumers only pay the performance price of what they actually use.

### They lead to poor runtime performance

상태값을 UI로 만드는 간단한 함수형 모델을 가진 React와 같은 프레임워크는 매우 생산적이다. 그러나 **virtual DOM**에서 변경 사항을 확인하기 위한 조정(reconciliation) 프로세스는 규모에 따라 비용이 많이 발생한다. 모놀리식 컴포넌트는 상태가 변경될 때 최소한의 부분만 리렌더링 되도록 만들기가 매우 어렵다.

**virtual DOM**을 사용하는 **React**와 같은 프레임워크에서 더 나은 렌더링 퍼포먼스를 달성하기 위한 가장 간단한 방법은 변경되는 컴포넌트들을 변경을 일으키는 컴포넌트들로부터 분리하는 것이다.

이렇게 하면 상태가 변경될 때 오직 필요한 것들만 리렌더링 된다. 만약 Relay와 같은 선언적 데이터 패칭 프레임워크를 사용하는 경우 데이터의 업데이트가 발생했을 때 하위 트리에서 발생하는 높은 비용의 리렌더링을 막기 위해 이러한 기술들이 더욱 더 중요해진다.

일반적으로 모놀리식 컴포넌트와 하향식 접근 방법을 사용하는 경우 컴포넌트를 분리할 지점을 찾기가 힘들고 에러는 발생하기 쉬우며 종종 `memo()`의 과도한 사용으로 이어진다.

## Building bottom up

하향식 접근 방법과 비교했을 때 상향식 접근 방법은 덜 직관적인 것처럼 보이고 초반에는 느릴 수도 있다. 상향식 접근 방법을 사용하면 거대한 하나의 스타일 컴포넌트 대신 API를 재사용할 수 있는 다수의 작은 컴포넌트들이 생겨난다.

그러나 빠르게 개발해야 하는 상황이 아니라면 API를 재사용할 수 있는 컴포넌트를 만들면 일반적으로 훨씬 더 잘 읽히고 테스트하기 용이하며 변경 및 삭제할 수 있는 컴포넌트 구조로 이어진다.

컴포넌트를 어느 수준까지 분해해야 하는지에 대한 명확한 해답은 없다. 핵심은 SPA, 즉 단일 책임 원칙(single responsibility principle)의 사용을 가이드라인으로 삼는 것이다.

### How is a bottom up mental model different to top down?

위에서 나왔던 예제로 돌아가보자. 상향식 접근 방법을 사용해도 여전히 최상단 `<SideNavigation />` 컴포넌트가 만들어질 확률이 높지만 하향식 접근 방법과의 차이점은 컴포넌트를 만드는 과정에 존재한다.

상향식 접근 방법도 최상단 `<SideNavigation />` 컴포넌트의 필요성을 식별하는 것까지는 동일하지만 이 컴포넌트에서 작업이 시작되지 않는다는 것이 바로 차이점이다.

전체적으로 `<SideNavigation />`의 기능을 구성하는 모든 기본 요소를 분류하고 함께 구성할 수 있는 작은 부분을 만드는 것으로 시작한다. 이런 방식으로 시작하는 부분이 약간 덜 직관적이다.

컴포넌트의 전체적인 복잡성은 단일 모놀리식 컴포넌트에 몰려 있는 것이 아닌 많고 작은 단일 책임 컴포넌트로 분산된다.

### What does a bottom up approach look like?

사이드 네비게이션 예제로 다시 돌아가보자. 다음은 예제에 대한 간단한 코드이다.

```jsx
<SideNavigation>
  <NavItem to="/home">Home</NavItem>
  <NavItem to="/settings">Settings</NavItem>
</SideNavigation>
```

간단한 예인 만큼 눈에 띄는 곳은 없다. 중첩 그룹을 지원하는 API의 경우 어떻게 생겼을까?

```jsx
<SideNavigation>
  <Section>
    <NavItem to="/home">Home</NavItem>
    <NavItem to="/projects">Projects</NavItem>
    <Separator />
    <NavItem to="/settings">Settings</NavItem>
    <LinkItem to="/foo">Foo</NavItem>
  </Section>
  <NestedGroup>
    <NestedSection title="My projects">
      <NavItem to="/project-1">Project 1</NavItem>
      <NavItem to="/project-2">Project 2</NavItem>
      <NavItem to="/project-3">Project 3</NavItem>
      <LinkItem to="/foo.com">See documentation</LinkItem>
    </NestedSection>
  </NestedGroup>
</SideNavigation>
```

상향식 접근 방법의 최종 결과물은 직관적이다. 간단한 API의 복잡성이 각 개별 컴포넌트의 뒤로 캡슐화되기에 상향식 접근 방법에는 더 많은 사전 준비가 필요하다. 하지만 이러한 준비 과정들이 이 접근 방법을 더욱 소비성이 높고 적응력이 뛰어난 장기적인 접근 방법으로 만드는 이유이다.

하향식 접근 방법와 비교하여 상향식 접근 방법이 갖는 장점들은 다음과 같다.

1. 컴포넌트를 사용하는 다른 팀들은 실제로 `import`하고 사용하는 컴포넌트들의 비용만을 지불한다(딱 필요한 부분만을 가져와서 사용할 수 있다. 이런건 모놀리식 컴포넌트로는 불가능하다).
2. 코드를 쉽게 분리할 수 있고 사용자에게 즉각적으로 필요한 우선 순위 요소가 아니라면 비동기적으로 로딩할 수도 있다.
3. 업데이트로 인해 변경되는 하위 트리만 리렌더링 되기 때문에 렌더링 퍼포먼스가 더 좋아지고 관리하기도 쉬워진다.
4. 네비게이션 컴포넌트 안에서 특정 책임을 가진 부분을 개별 컴포넌트로 만들고 최적화할 수 있다. 각 컴포넌트들을 독립적으로 작업할 수 있고 최적화할 수 있기 때문에 코드 구조의 관점으로 봤을 때 더 확장에 용이하다.

### What’s the catch?

상향식 접근 방법은 적응력이 더 좋기 때문에 초기에는 느리지만 장기적으로는 빠르다. You can more easily avoid hasty abstractions and instead ride the wave of changes over time until the right abstraction becomes obvious. 이는 모놀리식 컴포넌트의 전파를 막기 위한 가장 좋은 방법이다.

사이드바 네비게이션과 같이 코드베이스 전체에 걸쳐 공유되는 컴포넌트의 경우 상향식 접근 방법을 통해 구축하면 해당 컴포넌트를 사용하는 측에서 분해되어 있는 작은 컴포넌트들을 합치는 데 조금 더 많은 노력이 필요하다. 하지만 앞서 본 것처럼 공유 컴포넌트를 많이 가지고 있는 대규모 프로젝트에서는 이러한 **trade-off**(노력)을 할 가치가 있다.

상향식 접근 방법의 힘은 우리의 **mental model**이 이미 염두에 두고 있는 특정 추상화로 시작하는 것이 아니라 "내가 원하는 것을 달성하기 위해 구성해야 하는 단순한 기본 요소들이 무엇일까"라는 전제에서 시작할 수 있게 해준다는 것이다.

> The power of a bottom-up approach is that your model starts with the premise “what are the simple primitives I can compose together to achieve what I want” versus starting out with a particular abstraction already in mind.

> 애자일 소프트웨어 개발의 가장 중요한 교훈 중 하나는 반복의 가치다. 이는 아키텍처를 포함한 모든 수준의 소프트웨어 개발에 적용된다.
>
> **_One of the most important lessons of Agile software development is the value of iteration; this holds true at all levels of software development, including architecture_**

상향식 접근 방법은 우리가 장기적으로 더 잘 "반복"할 수 있게 해준다.

다음은 컴포넌트 구축을 쉽게 만들어 주는 몇 가지 유용한 원칙들에 대해서 알아본다.

## Strategies for avoiding monolithic components

### Balancing single responsibility vs DRY.

상향식 접근 방법으로 생각한다는 것은 종종 **composition pattern**을 받아들이는 것을 의미한다. 즉, 상향식 접근 방법으로 만들어진 컴포넌트를 사용하는 지점에서 약간의 중복이 있을 수 있음을 의미한다.

**DRY** is the first thing we learn as developers and it feels good to **DRY** up code. 그러나 종종 모든 코드에 **DRY** 원칙을 적용하기 전에 **DRY** 원칙이 필요한지 확인하고 기다리는 것이 좋다.

그러나 이러한 접근 방법을 사용하면 프로젝트가 커지고 요구사항이 변경될 때 "복잡함의 흐름을 극복"할 수 있으며, 추상적인 것들을 사용하여 합리적인 시점에 더 쉽게 소비할 수 있도록 만들어 준다.

> But this approach lets you “ride the wave of complexity” as the project grows and requirements change, and allows abstract things for easier consumption at the time it makes sense to.

### Inversion of control

다음은 이 원칙을 이해하기 위해 `Callback`과 `Promise`의 차이점을 다룬 예제이다.

`Callback`을 사용하면 해당 함수가 어디로 가는지 얼마나 많이 호출되거나 무엇으로 호출되는지 알 필요가 없다.

`Promise`는 코드의 제어권을 함수를 호출한 쪽에 되돌려 로직을 구성하기 시작하고 값이 이미 있는 것처럼 가장할 수 있다.

```jsx
// may not know what onLoaded will do with the callback we pass it
onLoaded((stuff) => {
  doSomething(stuff);
});

// control stays with us to start composing logic as if the
// value was already there
onLoaded.then((stuff) => {
  doSomething(stuff);
});
```

이를 **React**의 맥락에선 컴포넌트 API 설계를 통하여 달성되는 것을 확인할 수 있다.

`children`을 통하여 "슬롯(slots)"을 노출하거나 컴포넌트를 사용하는 측에서 제어의 역전을 유지하는 스타일 `props`를 렌더링할 수 있다.

가끔은 이러한 방식이 컴포넌트를 사용하는 측에 더 많은 일을 하게 만든다는 느낌을 주기 때문에 제어의 역전에 대한 거부감이 생길 수도 있다. 하지만 이것은 미래를 예측할 수 있는 아이디어를 포기하는 것과 컴포넌트를 사용하는 자들에게 유연성을 부여하는 선택에 관한 것이다.

> But this is both about giving up the idea you can predict future, and opting to empower consumers with flexibility.

```jsx
// A "top down" approach to a simple button API
<Button isLoading={loading} />

// with inversion of control
// provide a slot consumers can utilize how they see fit
<Button before={loading ? <LoadingSpinner /> : null} />
```

> 정리를 좀 하자면 컴포넌트의 제어권을 컴포넌트를 사용하는 측에게 부여함으로써 해당 컴포넌트를 사용하는 사용자들이 컴포넌트를 더 유연하게 사용할 수 있게 만들어 주는 것이다.
>
> 위 예제에서 볼 수 있듯이 동일한 버튼 컴포넌트 임에도 불구하고 하향식 접근 방법으로 만들어진 버튼 컴포넌트는 해당 컴포넌트를 사용하는 측에서 버튼의 로딩 스피너를 활성화할지 말지만 선택할 수 있게 하고 있다.
>
> 그에 반해 제어의 역전을 활용한 두 번째 버튼 컴포넌트는 버튼 컴포넌트의 로딩 상태에 대한 모든 것을 사용하는 측에서 결정할 수 있도록 만들어 주고 있다.

두 번째 버튼 컴포넌트는 `<LoadingSpinner />`가 더 이상 버튼 컴포넌트에 의존하지 않기 때문에 요구사항의 변경에 더 유연하고 더 뛰어난 성능을 갖게 된다. 두 번째 버튼 컴포넌트는 조금 더 많은 작업을 필요로 하지만 결국엔 더 유연하고 뛰어난 접근 방식이 된다.

또한 재밌는 점은 `<Button />` 컴포넌트 자체가 내부에서 더 작은 컴포넌트들로 구성될 수도 있다는 점이다. Sometimes a particular abstraction has many different sub behavioral elements underneath that can be made explicit.

예를 들어, `Button`과 `Link` 컴포넌트에 모두 적용할 수 있는 `Pressable`과 같은 것과 `LinkButton`과 같은 것을 결합하여 만들 수 있는 컴포넌트로 더 세분화할 수 있다. •  This finer grained breakdown is usually left for the domain of design system libraries, but worth keeping in mind as product focussed engineers.

### Open for extension

Even when using composition patterns to build bottom up. You’ll still want to export specialized components with a consumable API, but built up from smaller primitives. For flexibility, you can also expose those smaller building blocks that make up that specialized component from your package as well.

Ideally your components do one thing. So in the case of a pre-made abstraction, consumers can take that one thing they need and wrap it to extend with their own functionality. Alternatively they can just take a few primitives that make up that existing abstraction and construct what they need.

> 이 부분은 잘 이해가 되지 않아 원문으로 남겨뒀다. 🥲

### Leveraging storybook driven development

일반적으로 컴포넌트에서 관리되는 수 많은 개별 상태값이 존재한다. 그렇기에 상태 머신 라이브러리(상태 관리 라이브러리를 말하는 것 같다) 점점 인기를 얻고 있다.

We can adopt the models behind their thinking when building out our UI components in isolation with storybook and have stories for each type of possible state the component can be in.

Doing it upfront like this can avoid you realizing that in production you forgot to implement a good error state.

또한 작업 중인 컴포넌트를 만드는 데 필요한 모든 하위 컴포넌트를 식별하는 데 도움이 된다.

### Some questions you can ask yourself when building the UI components in isolation that lead to resilient components

- Is it accessible?
- 로딩 중일 때 어떻게 보이는가?
- 어떤 데이터에 의존하는가?
- 에러는 어떻게 처리하는가?
- 데이터의 일부분만 이용할 수 있는 상황이라면 어떻게 되는가?
- 해당 컴포넌트를 여러 번 `mount` 하면 어떻게 되는가?
  즉, 어떤 종류의 **side effect**가 있고 컴포넌트가 내부 상태를 관리한다면 그 내부 상태가 일관적이라고 기대할 수 있는가?
- **impossible states**를 어떻게 처리하고 해당 상태들 간의 전환은 어떻게 이루어지는가?
  예를 들어, `loading`과 `error`의 `props`가 모두 `true`라면 어떻게 되는가? (이러한 예가 아마도 컴포넌트 API에 대해 다시 생각해볼 수 있는 기회가 될 것이다)
- How composable is it? Thinking about its API.
- Are there any opportunities for delight here? E.g subtle animations done well.

Here are some more common situations to avoid that prevent building resilient components:

### Name components based on what they actually do

단일 책임 원칙으로 다시 돌아오자. 이름에 담긴 의미가 타당하다면 컴포넌트의 이름이 길어지는 것을 두려워 하지 말자.

실제로 컴포넌트가 수행하는 것보다 약간 더 일반적인 컴포넌트의 이름으로 지정하는 것도 좋다. 컴포넌트가 실제로 수행하는 것보다 더 일반적인 이름으로 지정되면 다른 개발자에게 X와 관련된 모든 것을 처리하는 추상화임을 나타낼 수 있기 때문이다.

이러면 자연스럽게 새로운 요구사항이 들어왔을 때 변경해야할 곳이 명백하게 두드러진다. Even when it might not make sense to do so.

### Avoid prop names that contain implementation details

특히 UI style "leaf" components의 경우 더욱 그러하다. 내부 상태 또는 도메인의 특정 항목과 관련된 `isSomething`과 같은 `props`를 추가하는 것은 가급적 피하는 것이 좋다. And then have that component do something different when that prop is passed in.

If you need to do this, it’s clearer if the prop name reflects what it actually does in the context of that component consuming it.

예를 들어, `isSomething`과 같은 `prop`이 결국 `padding`과 같은 것을 제어하게 되면 `prop` 이름에 이러한 사실을 반영해야만 한다.

### Be cautions of configuration via props

제어의 역전으로 다시 돌아가보자.

Components like **`<SideNavigation navItems={items} />`** can work out fine if you know you’ll only have one type of child (and you know for certain this definitely won’t change!) as they can also be typed safely.

그러나 앞서 살펴본 것처럼 빠른 개발을 진행하는 여러 팀과 개발자 간에는 확장하기 어려운 패턴이다. 실제로는 변경 사항에 대해 유연하지 못하고 복잡성이 빠르게 증가하는 경향이 있다.

이는 컴포넌트를 다르거나 추가적인 타입의 자식 요소를 갖도록 확장하려는 경우가 많기 때문이다. 즉, 컴포넌트를 구성하는 옵션이나 `props` 추가하거나 로직을 분기하는 지점을 추가하는 일이 많이 발생하기 때문이다.

컴포넌트를 사용하는 측에서 내려줄 객체를 정리하거나 전달하는 것보다 더 유연한 접근 방법은 내부 자식 컴포넌트를 `export`하고 컴포넌트를 사용하는 측에서 컴포넌트를 구성하고 전달하도록 만드는 것이다.

> Rather than have consumers arrange and pass in objects, a more flexible approach is to export the internal child component as well, and have consumers compose and pass components.

### Avoid defining components in the render method

때로는 컴포넌트 내에 "helper" 컴포넌트가 있는 것이 일반적일 수 있다. 이는 결국 모든 렌더링에서 다시 `mount` 되고 이상한 버그를 만들어 낼 수 있다.

또한 컴포넌트 내에 다수의 내부 render 메서드가 있는 경우 이는 일반적으로 컴포넌트가 모놀리식이 되고 있다는 신호이다. 이런 컴포넌트들은 분리하기에 좋은 후보군들이다.

## Breaking down monolithic components

가능하다면 자주 그리고 일찍 리팩토링하자. Identifying components likely to change and actively decomposing them is a good strategy to bake into your estimates.

프론트엔드가 지나치게 복잡해진 상황이라면 어떻게 해야할까?

이러한 경우 대표적으로 두 가지 옵션이 있다:

1. 컴포넌트를 다시 만들면서 점진적으로 새로운 컴포넌트로 마이그레이션 하는 방법
2. 점진적으로 세분화하는 방법

Going into component refactoring strategies is outside the scope of this guide for now. But there are a bunch of existing battle-tested refactoring patterns you can utilize.

In frameworks like React, “components” are really just functions in disguise. Sp you can replace the word “function” with component in all the existing tried and true refactoring techniques.

To give a few relevant examples:

- **[Remove Flag Argument](https://refactoring.com/catalog/removeFlagArgument.html)**
- **[Replace Conditional with Polymorphism](https://refactoring.com/catalog/replaceConditionalWithPolymorphism.html)**
- **[Pull Up Field](https://refactoring.com/catalog/pullUpField.html)**
- **[Rename Variable](https://refactoring.com/catalog/renameVariable.html)**
- **[Inline Function](https://refactoring.com/catalog/inlineFunction.html)**

## Concluding thoughts

We covered a lot of ground here. Let’s recap the main takeaways from this guide.

1. **The models we have affect the many micro-decisions we make when designing and building frontend components.**

   ⇒ 우리가 가지고 있는 mental model은 프론트엔드 컴포넌트를 구축하고 설계할 때 내리는 미세한 결정들에 영향을 준다.

   Making these explicit is useful because they accumulate pretty rapidly. The accumulation of these decisions ultimately determine what becomes possible - either increasing or reducing the friction to add new features or adopt new architectures that allow us to scale further (not sure about this point or merge it below).

2. **Going top down versus bottom up when constructing components can lead to vastly different outcomes at scale.**

   ⇒ 컴포넌트를 구축할 때 상향식으로 하냐 하향식으로 하냐에 따라 결과물이 크게 다를 수 있다.

   컴포넌트를 구축할 때는 대개 하향식 **mental model**이 가장 직관적이다. UI를 분해할 때 가장 일반적인 방법은 컴포넌트가 되는 기능 영역 주위에 상자를 그리는 것이다. 이러한 기능 분해 과정이 바로 하향식 접근 방법이며 종종 특정 추상화를 사용하여 특수한 컴포넌트의 생성으로 이어진다. 요구사항은 변경되기 마련이다. 이러한 변경이 몇 번 반복되고 나면 컴포넌트들은 빠르게 모놀리식 컴포넌트가 되기 쉽다.

3. **Designing and building top down can lead to monolithic components.**

   ⇒ 하향식 접근 방법을 통한 설계와 구축은 모놀리식 컴포넌트로 이어질 수 있다.

   모놀리식 컴포넌트로 가득한 코드베이스는 느리고 변경 사항에 유연하지 않은 프론트엔드 아키텍처를 초래한다. 모놀리식 컴포넌트는 다음과 같은 이유로 좋지 않다.

   - 변경과 유지보수에 비용이 많이 든다.
   - 컴포넌트를 변경하는데 위험이 따른다.
   - 팀 간에 기존 작업물을 활용하는데 어려움이 따른다.
   - 퍼포먼스가 좋지 않다.
   - 효과적인 code-splitting, 팀 간의 코드 재사용, 코드가 로딩되는 단계의 조정, 렌더링 퍼포먼스 등과 같이 프론트엔드를 지속적으로 확장하는데 중요한 미래 지향적인 기술 및 아키텍처를 선택하려고 할 때 걸림돌이 된다.

4. **We can avoid the creation of monolithic components.**

   ⇒ 성급한 추상화 또는 지속적인 확장으로 이어지는 **mental model**과 상황을 이해함으로써 모놀리식 컴포넌트의 생성을 피할 수 있다.

   **React**는 컴포넌트를 설계할 때 하향식보다는 상향식 **mental model**에 더 효과적으로 들어맞는다. 이는 성급한 추상화를 보다 효과적으로 방지할 수 있게 한다. Such that we can “ride the wave of complexity” and abstract when the time is right.

   > 여기서 “ride the wave of complexity”이라는 부분을 정확히 모르겠다. 이전에 언급된 문단들에서도 확실하지 않아 원문을 남겨놨었다.

   상향식 접근 방법으로 컴포넌트를 구축하면 **composition pattern**을 실현할 가능성이 커진다. 모놀리식 컴포넌트가 실제로 얼마나 많은 비용을 발생시키는지 알고 있으므로 표준 리팩토링 방식을 적용하여 매일 프로덕트 개발의 일환으로 점진적으로 컴포넌트를 분해해 나갈 수 있다.

   > Being aware of how costly monolithic components truly are, we can apply standard refactoring practices to decompose them regularly as part of everyday product development.

## **References**

- **[Difference between Bottom-Up Model and Top-Down Model](https://www.geeksforgeeks.org/difference-between-bottom-up-model-and-top-down-model/)**
- **[The wrong abstraction](https://sandimetz.com/blog/2016/1/20/the-wrong-abstraction)**
- **[Inversion of control](https://kentcdodds.com/blog/inversion-of-control)**
- **[AHA programming](https://kentcdodds.com/blog/aha-programming)**
