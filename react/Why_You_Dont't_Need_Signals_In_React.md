# Why You Don't Need Signals in React

> 원본 글  
> https://blog.axlight.com/posts/why-you-dont-need-signals-in-react/?ck_subscriber_id=1780635791

## Introduction

웹 프론트엔드 개발 세계에서 `signal`는 인기 있는 주제가 되었습니다. `signal`의 핵심은 시간에 따른 상태 변화를 표현하는 데 사용됩니다. 일부 개발자들은 React와 함께 `signal`를 사용할 수 있는 가능성에 대해 논의했습니다.

`signal`는 사실 오래된 개념이며, 현대 웹 개발자들이 어떻게 이해하고 있는지는 불확실합니다. 처음에는 `signal`의 특성에 대해 혼란스러웠지만 나중에 `signal`는 크게 두 가지 측면으로 요약할 수 있다는 것을 깨달았습니다.

- **Reactive primitives**
- **Bypassing diffing**

이 블로그 포스트에서는 이 두 가지 측면과 React의 맥락에서의 관련성을 살펴볼 것입니다. `signal`의 이 두 가지 측면에 대해서만 논의할 것이며 누군가에게 더 중요할 수 있는 다른 잠재적 용도는 고려하지 않을 것이라는 점에 유의하세요.

## Reactive primitives

`Reactivity`은 React의 핵심 기능입니다. 컴포넌트는 상태가 변경되면 다시 렌더링됩니다. `useState`를 사용하면 업데이트 시 다시 렌더링을 트리거하는 상태를 정의하여 **Reactive primitives**를 만들 수 있습니다. 이렇게 하면 컴포넌트를 반응형으로 만들 수 있으며 렌더 함수에서 파생된 상태를 정의할 수도 있습니다.

다음은 `useState`의 사용 예시입니다:

```typescript
const Component = () => {
  const [count, setCount] = useState(0);
  const doubleCount = count * 2; // derived state
  // ...
};
```

하지만 `useState`는 로컬 상태만 생성한다는 점에 유의해야 합니다. 따라서 컴포넌트 간에 상태를 공유하기 어려울 수 있으며, 이를 위해 **prop drilling** 또는 **context propagation**를 사용해야 할 수도 있습니다.

전역 상태를 정의하고 사용하는 프로세스를 간소화하려면 **Jotai**와 같은 서드파티 라이브러리가 유용할 수 있습니다. **Jotai**를 사용하면 **prop drilling**이나 **context propagation**에 의존하지 않고도 컴포넌트 간에 상태를 쉽게 공유할 수 있습니다.

**Jotai**로 전역 상태를 정의하려면 `atom`을 사용할 수 있습니다. 이러한 `atom`는 컴포넌트에서 사용할 수 있는 상태 조각의 정의를 나타냅니다. 예를 들어 `primitive atom`를 정의하는 방법은 다음과 같습니다:

```typescript
const countAtom = atom(0);
```

다음과 같이 다른 `atom`에 의존하는 `derived atom`(다른 `atom`으로부터 파생된)를 정의할 수도 있습니다:

```typescript
const doubleCountAtom = atom((get) => get(countAtom) * 2);
```

`atom` 구문은 일반적인 `signal` 구문과 약간 다르게 보일 수 있지만, 멘탈 모델은 매우 유사합니다. 기본 요소를 정의하고 이를 복잡한 상태에 맞게 구성합니다. 데이터 그래프를 표현하는 데 필요한 만큼의 `atom`를 정의할 수 있으므로 애플리케이션에서 복잡한 상태를 쉽게 관리할 수 있습니다.

다음은 컴포넌트에서 **Jotai** `atom`를 사용하는 방법을 보여줍니다:

```typescript
const Component = () => {
  const [count, setCount] = useAtom(countAtom);
  const [doubleCount] = useAtom(doubleCountAtom); // derived state
  // ...
};
```

`useState`와 달리 `useAtom`은 로컬 상태가 아니므로 다른 컴포넌트에서 사용하여 `atom` 상태를 공유할 수 있습니다:

```typescript
const AnotherComponent = () => {
  const [count, setCount] = useAtom(countAtom);
  // ...
};
```

**Reactive primitives**와 관련하여 **Jotai** `atom`가 `signal`과 유사하게 작동한다는 것을 눈치채셨을 것입니다. 하지만 **Jotai**는 React 규칙을 따르는 `useAtom`과 같은 React 훅을 통해 추가적인 이점을 제공합니다. 이러한 훅을 사용하면 **prop drilling**이나 **context propagation** 없이도 컴포넌트 간에 상태를 공유할 수 있으므로 코드가 간소화되고 추론하기가 더 쉬워집니다. 또한 **Jotai**에는 서브트리의 상태를 분리하는 `Provider`가 있는데, 이는 `global signal`로는 불가능합니다.

## Bypassing diffing

React의 또 다른 핵심 기능은 `diffing`이라는 프로세스를 통해 이뤄지는 DOM의 업데이트입니다. React는 UI의 이전 표현과 현재 표현을 비교하여 무엇이 변경되었는지 판단하고 해당 부분만 업데이트하여 더 나은 성능과 반응성이 뛰어난 UI를 제공합니다.

그러나 `diffing`에는 비용이 발생하며, `diffing`을 우회하는 것이 더 효율적인 경우가 있을 수 있습니다. 예를 들어 UI의 한 부분만 업데이트하고 다른 부분은 모두 변경하지 않는 것이 확실하다면 `diffing`을 거치지 않고 직접 UI를 업데이트하는 것이 더 효율적일 수 있습니다.

기술적으로 **Bypass diffing**이 가능하다는 것을 증명하기 위해 **jotai-signal**이라는 실험적인 라이브러리가 있습니다. 내부에 대해 설명하는 블로그 게시물도 있습니다: [Demystifying Create React Signals Internals](https://blog.axlight.com/posts/demystifying-create-react-signals-internals/)

그러나 `diffing`을 우회하는 것은 선언적 프로그래밍이라는 React의 핵심 원칙에 위배됩니다. 성능상의 이유로 `diffing`을 우회하고 싶을 수도 있지만, 그렇게 하면 UI에 불일치가 발생하고 애플리케이션을 추론하기 어렵게 만들 위험이 있습니다.

`diffing`을 우회하기로 결정하기 전에 성능 이점을 철저히 평가하고 잠재적인 위험과 비교하여 평가하는 것이 중요합니다. 또한 애플리케이션의 성능을 최적화할 수 있는 다른 방법이 있는지 고려하는 것도 좋습니다.

일반적으로 UI의 일관성과 예측 가능성을 유지하기 위해 React의 best practice를 따르고 `diffing`을 적절히 사용하는 것이 좋습니다.

## Summary

결론적으로, `signal`은 웹 개발에서 흥미로운 개념이지만, **Jotai**는 React 애플리케이션에서 상태를 관리하는 더 간단하고 직관적인 방법을 제공합니다. **Jotai**를 사용하면 **prop drilling**이나 **context propagation**가 필요 없이 `atom`을 통해 전역 상태를 쉽게 생성하고 사용할 수 있습니다. 아톰은 개념적으로 **Reactive primitives** 측면에서 `signal`와 매우 유사합니다.

기술적으로 React의 `diffing` 알고리즘을 우회하는 것은 가능하지만, 그렇게 하면 선언적 프로그래밍의 원칙에 어긋나며 UI에 불일치가 발생할 수 있습니다. `diffing`을 우회하기로 결정하기 전에 잠재적인 이점과 위험을 철저히 평가하는 것이 중요합니다.

React의 best practice를 따르고 **Jotai**의 강력한 기능을 활용하면 유지 관리가 용이하고 성능이 뛰어난 React 애플리케이션을 만들 수 있습니다.
