# Keeping browser interactive during hydration

> 원본 글  
> https://github.com/reactwg/react-18/discussions/38

**목차**

- [Keeping browser interactive during hydration](#keeping-browser-interactive-during-hydration)
  - [질문](#질문)
  - [How can React keep the browser responsive?](#how-can-react-keep-the-browser-responsive)
  - [What happens when you set parent state before a sibling gets hydrated?](#what-happens-when-you-set-parent-state-before-a-sibling-gets-hydrated)

## 질문

> In React 18, hydrating content inside Suspense boundaries happens with tiny gaps in which the browser can handle events. Thanks to this, the click is handled immediately, and the browser doesn’t appear stuck during long hydration on a low-end device. For example, this lets the user navigate away from the page they’re no longer interested in.

브라우저가 `hydration`을 진행 중인 상황에서도 어떻게 상호 작용 가능한가요? 일정한 간격을 두고 일괄적으로 `hydration`을 하는 방식인가요?

추가적으로 아래의 시나리오는 어떻게 처리되나요?

1. `A`라는 컴포넌트(`hydrate` 되서 클릭이 가능한)가 있고 여전히 `hydrating` 중인 `B`라는 컴포넌트(`A` 컴포넌트와 형제 요소인)가 있는 경우

   `A` 컴포넌트를 클릭하여 `B` 컴포넌트도 구독하고 있는 어떤 상태값 `X`(외부 저장소에 저장된)를 업데이트하는 상황에서는 어떤 일이 발생하나요? **React**가 `B` 컴포넌트가 `hydrate` 될 때까지 `event`를 `dispatch` 하지 않나요?

2. `A` 컴포넌트와 `B` 컴포넌트가 형제 요소가 아니라면 어떻게 되나요?

질문 자체는 `hydration`에 관한 것이지만 답변은 여러 기능에 대해서도 동일한 답변이 될 것이므로 조금 더 일반적인 질문인 "어떻게 브라우저를 상호 작용 가능하게 유지하는가"에 대해 답변하고자 한다.

그래서 답변을 두 가지로 분리할 것이다.

- **React**는 어떻게 브라우저를 `responsive` 하게 유지할 수 있을까?
- 형제 요소가 hydrate 되기 전에 상위 요소의 상태값이 업데이트되면 무슨 일이 일어날까?

## How can React keep the browser responsive?

이론적으로 **React**의 렌더링 알고리즘은 다음과 같이 생각할 수 있다.

```jsx
while (hasMoreComponents) {
  renderNextComponent();
}
```

([Source code](https://github.com/facebook/react/blob/5aa0c5671fdddc46092d46420fff84a82df558ac/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1473-L1475))

여기서 "**render**"란 컴포넌트 함수를 호출하고 큐에 **DOM** 변경 사항을 집어 넣는 것을 말한다. 호출할 컴포넌트가 더 이상 남아있지 않을 때(상태 업데이트에 의해 영향받는 전체 트리를 모두 지나왔을 때), 실제로 **DOM**을 업데이트한다.

이것이 **synchronous render**다.

**React 18**에서 "**concurrent**" **render**라는 새로운 개념이 생겼다. 이는 대략적으로 다음과 같이 생겼다.

```jsx
while (hasMoreComponents && !hasOtherStuffToDo) {
  renderNextComponent();
}
```

([Source code](https://github.com/facebook/react/blob/5aa0c5671fdddc46092d46420fff84a82df558ac/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L1560-L1562))

**concurrent render**는 컴포넌트 함수를 호출할 때마다 중요한 작업이 있는지 확인한다. 브라우저가 이벤트를 처리하고 필요한 경우 화면을 그릴 수 있도록 주기적으로 이 플래그를 `true`로 설정한다. 그리고 나서 만약 긴급하게 처리해야할 무언가가 더 이상 없다면 이전 루프로 돌아가 렌더링을 다시 진행한다. 다시 말해, **concurrent render**는 방해할 수 있다(`interruptible`). 예를 들어, 만약 이러한 간격(`gap`) 동안 상태를 다시 설정하면 이전 업데이트로 인한 렌더링을 마무리하려고 하지 않는다. 처음부터 다시 시작한다.

> 정리하자면 **concurrent render**는 렌더링을 위해서 컴포넌트 함수를 호출하는 과정 속에서 렌더링 이외에 추가적인 작업이 있는지 주기적으로 확인하는데 이때 플래그가 `true`가 되면 진행하고 있던 렌더링을 멈추고 해당 작업을 우선적으로 처리한다는 것이다.
>
> 만약 이 작업에서 또 다른 상태 업데이트가 발생할 경우 이전에 진행하고 있던 렌더링 과정을 더 이상 마무리하지 않고 새롭게 처음부터 다시 렌더링을 시작한는 뜻인 것 같다.
>
> 물론 실제 구현에 완벽히 들어맞는다고 할 순 없겠지만 비슷한 정도는 되는 것 같다.  
> _Note: the reality is a bit more nuanced but this is a decent approximation._

기본적으로 첫 번째 알고리즘(non-interruptible)을 사용한다. 그리고 다음에 나오는 것들에 대해선 두 번째 알고리즘(interruptible)이 동작한다.

- Updates from `[startTransition](https://github.com/reactwg/react-18/discussions/41)` (which is the main opt-in mechanism to this feature, to be documented soon)
- Retries from `<Suspense>` (`<Suspense>` 경계에서 `hydration` 중에 발생하는 일)
- Some other cases we'll document

So this is how it works. 이미 실행된 **JavaScript** 코드를 "**pause**"하거나 "**interrupt**" 할 수 없다. 이는 불가능하다. 하지만 다음 컴포넌트를 호출하기 전에 **render pass**를 멈추고 대신 다른 일을 하게끔 선택할 수는 있다(예를 들어, 기존 **render pass** 대신 다른 업데이트를 렌더링하거나 브라우저가 `paint` 하도록).

**React에서 각 개별 컴포넌트는 일반적으로 렌더링하는데 시간이 거의 소요되지 않는다**(`< 1ms`). 이는 컴포넌트의 렌더링 시간에 해당 컴포넌트의 **하위 요소들을 포함하지 않기** 때문이다. **JSX**는 `lazy`하기에 `<Button />`은 `Button`을 `Button()`과 같이 호출하지 않는다. 그래서 꽤나 무거운 트리를 갖고 있을지라도 각 개별 레이어는 충분히 작다(전체적으로는 크고 무거운 트리를 갖고 있을지라도 레이어별로 떼어서 보면 작다). 이것이 **React**가 이미 실행 중인 코드를 멈출 순 없지만 실제로는 멈출 수 있는 것처럼 보이는 이유다. 그리고 이것이 가능하다는 이야기는 이 사이에 브라우저가 이벤트를 처리할 수 있다는 뜻이다.

이것이 **React**가 **interrupt execution** 하는 것이 가능한 것에 대한 일반적인 답변이었다.

Now onto your more specific question.

## What happens when you set parent state before a sibling gets hydrated?

> `A`라는 컴포넌트(`hydrate` 되서 클릭이 가능한)가 있고 여전히 `hydrating` 중인 `B`라는 컴포넌트(`A` 컴포넌트와 형제 요소인)가 있는 경우

`hydration`의 세분화는 개별 컴포넌트가 아닌 `<Suspense>` 경계에 의해서 결정된다는 사실을 알아야 한다. 그렇기에 **React**가 컴포넌트를 하나하나 호출할지라도 해당 컴포넌트들은 독립적으로 `hydrate` 되지 않는다. You can think of it as hydrating in "**levels**". 단일 "**level**"은 `Suspense` 경계에서 시작하고 경계 내에 있는 모든 컴포넌트들을 포함한다(다음 "**level**"이 되는 `Suspense`로 래핑한 요소를 제외하고). "**level**"은 기술적인 개념이 아니라 단지 여기서 사용하는 비유일 뿐이다.

다시 질문으로 돌아와서, 개별 컴포넌트 `A`와 `B`에 대해서가 아니라 `A`와 `B`의 서브트리에 대해서 이야기하고 있다고 해보자.

> `A` 컴포넌트를 클릭하여 `B` 컴포넌트도 구독하고 있는 어떤 상태값 `X`(외부 저장소에 저장된)를 업데이트하는 상황에서는 어떤 일이 발생하나요? **React**가 `B` 컴포넌트가 `hydrate` 될 때까지 `event`를 `dispatch` 하지 않나요?

`A` 컴포넌트가 자신의 로컬 `X` 상태값을 업데이트한 경우 `B` 컴포넌트가 `A`의 하위 요소가 아니므로 `B`에게 영향을 주지 않는다.

`A` 컴포넌트가 자신의 상위 요소의 상태값인 `X`를 업데이트한 경우 이는 `B` 컴포넌트가 어디에 있느냐에 따라서 달라진다.

1. 만약 `B`가 진짜로 `A`의 형제 요소라면 이는 `A`와 동일한 가장 가까운 `Suspense` 경계 내에 있음을 의미한다. 이 경우 특별한 동작은 없다.

   왜냐하면 **동일한 가장 가까운 `Suspense` 경계 내에 있는 컴포넌트들은 항상 같이 자기들에게 연결될 있는 이벤트 핸들러를 가져오기 때문이다.**

   그렇기에 이 경우엔 두 컴포넌트 모두 `hydrate` 되지 않거나(해당 컴포넌트들이 `hydrate` 될 때까지 큐에 들어간다) 둘 다 `hydrate` 된다. 이것은 **DOM** 변경 작업과 유사하다. **React**는 컴포넌트들을 하나하나 지나치지만 **DOM** 업데이트는 일괄적(batch)으로 이루어진다(이를 "**commits**"라고 부른다). `hydration`의 경우 **committing**은 이벤트 핸들러를 연결하는 것을 말한다.

2. `B`가 `A`의 직접적인 형제 요소가 아니긴 한데 `<A /> <Suspense fallback={<Spinner />}><B /></Suspense>` 와 같을 때 `A`가 특정 상태값을 업데이트하고 `B`가 아직 `hydrate` 되지 않았다면 상황이 재밌게 돌아간다. 예를 들어 살펴보자.

   댓글 섹션과 같이 서버로부터 받은 **HTML**이 있고 이는 아직 `hydrate` 되지 않았다고 해보자. 그리고 나서 여기에서 업데이트를 수행한다(예를 들어, 다크 모드 스위치 같은 것을 사용하여). 이때, **React**는 **server HTML**을 `Suspense`의 `fallback`으로 대체할 것이다. 왜냐하면 **server HTML**이 `stale` 상태가 되었기 때문이다.

   예를 들어 이미 어두운 테마로 전환한 후에는 밝은 테마를 사용하는 댓글을 보고 싶지 않을 것이다. In general, this means that top-level updates during the app startup, *especially*
    updating context, are not ideal. 이는 사용자에게 더 많은 `fallback`을 보게끔 만든다. 그러나 만약 예를 들어, 로딩 중에는 최상단 **Context**를 변경하지 않는다거나 `B` 컴포넌트 상위 어딘가에 `memo()`를 두는 것과 같이 일부 업데이트가 서브트리에 영향을 미치지 않는다는 것을 **React**에게 "**증명**"할 수 있다면 **React**는 **server content**를 계속 유지할 수 있다. 왜냐하면 이제 이것이 안전하다는 것을 알기 때문이다.
