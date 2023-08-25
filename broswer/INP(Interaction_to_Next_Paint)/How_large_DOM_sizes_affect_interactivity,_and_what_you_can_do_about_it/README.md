# How large DOM sizes affect interactivity, and what you can do about it

> 원본 글  
> https://web.dev/dom-size-and-interactivity/

**목차**

- [How large DOM sizes affect interactivity, and what you can do about it](#how-large-dom-sizes-affect-interactivity-and-what-you-can-do-about-it)
  - [개요](#개요)
  - [When is a page's DOM too large?](#when-is-a-pages-dom-too-large)
  - [How do large DOMs affect page performance?](#how-do-large-doms-affect-page-performance)
  - [How do I measure DOM size?](#how-do-i-measure-dom-size)
  - [How can I measure the number of DOM elements affected by an interaction?](#how-can-i-measure-the-number-of-dom-elements-affected-by-an-interaction)
  - [How can I reduce DOM size?](#how-can-i-reduce-dom-size)
  - [Other strategies to consider](#other-strategies-to-consider)
    - [Consider an additive approach](#consider-an-additive-approach)
    - [Limit CSS selector complexity](#limit-css-selector-complexity)
    - [Use the content-visibility property](#use-the-content-visibility-property)
  - [Conclusion](#conclusion)

## 개요

큰 DOM 크기는 생각보다 상호 작용에 많은 영향을 미칩니다. 이 가이드에서는 그 이유와 할 수 있는 일을 설명합니다.

웹 페이지를 만들면 해당 페이지에는 DOM(Document Object Model)이 생성됩니다. DOM은 페이지의 HTML 구조를 나타내며 JavaScript와 CSS가 페이지의 구조와 콘텐츠에 액세스할 수 있도록 합니다.

하지만 문제는 DOM의 크기가 페이지를 빠르고 효율적으로 렌더링하는 브라우저의 기능에 영향을 미친다는 점입니다. 일반적으로 DOM이 클수록 페이지를 처음 렌더링하고 페이지 수명 주기 후반에 렌더링을 업데이트하는 데 더 많은 비용이 듭니다.

이는 DOM을 수정하거나 업데이트하는 상호 작용이 페이지의 빠른 응답 기능에 영향을 미치는 값비싼 레이아웃 작업을 유발할 때 DOM이 매우 큰 페이지에서 문제가 됩니다. 값비싼 레이아웃 작업은 페이지의 다음 페인트에 대한 상호 작용(INP)에 영향을 줄 수 있으므로 페이지가 사용자 상호 작용에 빠르게 반응하도록 하려면 DOM 크기가 필요한 만큼만 커지도록 하는 것이 중요합니다.

## When is a page's DOM too large?

> **주요 용어**
>
> DOM 엘리먼트와 DOM 노드의 차이점을 아는 것이 중요합니다. DOM 엘리먼트는 DOM 트리의 특정 HTML 엘리먼트를 의미합니다. DOM 노드는 DOM 엘리먼트라는 용어와 의미가 겹치지만 주석, 공백, 텍스트를 포함하도록 정의가 확장되었습니다. Lighthouse DOM 크기 감사는 DOM 노드를 의미하지만, 이 가이드에서는 가능한 한 노드보다는 DOM 엘리먼트를 참조할 것입니다.

Lighthouse에 따르면 페이지의 DOM 크기가 1,400개 노드를 초과하면 과도한 크기라고 합니다. 페이지의 DOM이 800개 노드를 초과하면 Lighthouse에서 경고를 표시하기 시작합니다. 다음 HTML을 예로 들어 보겠습니다:

```html
<ul>
  <li>List item one.</li>
  <li>List item two.</li>
  <li>List item three.</li>
</ul>
```

위 코드에는 `<ul>` 요소와 3개의 `<li>` 자식 요소 등 4개의 DOM 요소가 있습니다. 웹 페이지에는 이보다 훨씬 더 많은 노드가 있을 것이므로 페이지의 DOM을 최대한 작게 만든 후 렌더링 작업을 최적화하기 위한 다른 전략뿐만 아니라 DOM 크기를 유지하기 위해 할 수 있는 일을 이해하는 것이 중요합니다.

## How do large DOMs affect page performance?

큰 DOM은 몇 가지 방식으로 페이지 성능에 영향을 줍니다:

1. 페이지의 초기 렌더링 중입니다. 페이지에 CSS를 적용하면 CSS Object Model(CSSOM)이라는 DOM과 유사한 구조가 만들어집니다. CSS Selector의 구체성이 증가함에 따라 CSSOM은 더욱 복잡해지고 웹 페이지를 화면에 그리는 데 필요한 레이아웃, 스타일링, 합성 및 페인트 작업을 실행하는 데 더 많은 시간이 필요합니다. 이렇게 추가된 작업은 페이지 로드 초기에 발생하는 인터랙션의 인터랙션 지연 시간을 증가시킵니다.

2. 인터랙션이 요소 삽입이나 삭제 또는 DOM 콘텐츠 및 스타일 수정을 통해 DOM을 수정하면 해당 업데이트를 렌더링하는 데 필요한 레이아웃, 스타일링, 합성 및 페인트 작업으로 인해 비용이 매우 많이 들 수 있습니다. 페이지의 초기 렌더링에서와 마찬가지로, 상호 작용의 결과로 HTML 요소가 DOM에 삽입될 때 CSS Selector 특이성이 증가하면 렌더링 작업이 추가될 수 있습니다.

3. JavaScript가 DOM을 쿼리할 때 DOM 요소에 대한 참조는 메모리에 저장됩니다. 예를 들어, 문서에서 모든 `<div>` 요소를 선택하기 위해 `document.querySelectorAll`을 호출하는 경우 결과가 많은 수의 DOM 요소를 반환하는 경우 메모리 비용이 상당할 수 있습니다.

<img src="images/1.avif" alt="Performance profiler에 표시된 긴 작업 예시" width="600" />

> Chrome 개발자 도구의 Performance profiler에 표시된 긴 작업. 표시된 긴 작업은 JavaScript를 통해 큰 DOM에 DOM 요소를 삽입하기 때문에 발생합니다.

이 모든 것이 상호작용에 영향을 미칠 수 있지만, 위 목록의 두 번째 항목이 특히 중요합니다. 상호 작용으로 인해 DOM이 변경되는 경우 페이지의 INP가 저하될 수 있는 많은 작업이 시작될 수 있습니다.

## How do I measure DOM size?

두 가지 방법으로 DOM 크기를 측정할 수 있습니다. 첫 번째 방법은 Lighthouse를 사용합니다. 감사를 실행하면 현재 페이지의 DOM에 대한 통계가 '진단(Diagnostics)' 제목 아래의 '과도한 DOM 크기 방지' 감사에 표시됩니다. 이 섹션에서는 총 DOM 요소 수, 가장 많은 자식 요소를 포함하는 DOM 요소, 가장 깊은 DOM 요소를 확인할 수 있습니다.

더 간단한 방법은 주요 브라우저의 개발자 도구에서 JavaScript 콘솔을 사용하는 것입니다. 페이지가 로드된 후 콘솔에서 다음 코드를 사용하여 DOM에 있는 HTML 요소의 총 개수를 확인할 수 있습니다:

```javascript
document.querySelectorAll('*').length;
```

> 위의 코드 스니펫에는 DOM에 있는 HTML 요소의 수만 포함되어 있습니다. DOM의 모든 노드는 포함되지 않습니다.

DOM 크기 업데이트를 실시간으로 확인하려면 성능 모니터(performance monitoring) 도구를 사용할 수도 있습니다. 이 도구를 사용하면 레이아웃 및 스타일링 작업(및 기타 성능 측면)을 현재 DOM 크기와 연관시킬 수 있습니다.

<img src="images/2.avif" alt="Chrome 개발자 도구의 성능 모니터(performance monitoring) 예시" width="600" />

> Chrome 개발자 도구의 성능 모니터(performance monitoring)입니다. 이 보기에서는 페이지의 현재 DOM 노드 수가 초당 수행되는 레이아웃 작업 및 스타일 재계산과 함께 차트에 표시됩니다.

DOM 크기가 Lighthouse DOM 크기의 경고 임계값에 근접하거나 완전히 실패한 경우 다음 단계는 DOM 크기를 줄여 페이지가 사용자 상호 작용에 응답하는 기능을 개선하여 웹사이트의 INP를 개선할 수 있는 방법을 알아내는 것입니다.

## How can I measure the number of DOM elements affected by an interaction?

실험실에서 페이지의 DOM 크기와 관련이 있다고 의심되는 느린 상호 작용을 프로파일링하는 경우, 프로파일러에서 'Recalculate Style'이라고 표시된 활동을 선택하고 하단 패널에서 컨텍스트 데이터를 관찰하여 영향을 받은 DOM 요소의 수를 파악할 수 있습니다.

<img src="images/3.avif" alt="profiler에서 볼 수 있는 Recalculate Style 작업 예시" width="600" />

> Recalculate Style 작업의 결과로 DOM에서 영향을 받는 요소의 수를 관찰합니다. 인터랙션 트랙에서 음영 처리된 부분은 인터랙션 지속 시간 중 200ms를 초과한 부분을 나타내며, 이는 INP에 대해 지정된 "양호(good)" 임계값입니다.

위 스크린샷에서 작업의 Recalculate Style을 선택하면 영향을 받는 요소의 수가 표시되는 것을 확인할 수 있습니다. 위의 스크린샷은 DOM 요소가 많은 페이지의 렌더링 작업에 대한 DOM 크기의 영향에 대한 극단적인 사례를 보여 주지만, 이 진단 정보는 어떤 경우에도 DOM의 크기가 상호 작용에 대한 응답으로 다음 프레임이 페인트되는 데 걸리는 시간을 제한하는 요소인지 확인하는 데 유용합니다.

## How can I reduce DOM size?

웹사이트의 HTML에 불필요한 마크업이 있는지 감사하는 것 외에도 DOM 크기를 줄이는 주요 방법은 DOM 깊이를 줄이는 것입니다. 브라우저 개발자 도구의 요소 탭에 다음과 같은 마크업이 표시되는 경우 DOM이 불필요하게 깊다는 신호 중 하나입니다:

```html
<div>
  <div>
    <div>
      <div>
        <!-- Contents -->
      </div>
    </div>
  </div>
</div>
```

이와 같은 패턴을 발견하면 DOM 구조를 평평하게 만들어 패턴을 단순화할 수 있습니다. 이렇게 하면 DOM 요소의 수가 줄어들고 페이지 스타일을 단순화할 수 있습니다.

DOM 깊이는 사용하는 프레임워크의 문제일 수도 있습니다. 특히 JSX에 의존하는 컴포넌트 기반 프레임워크의 경우 상위 컨테이너에 여러 컴포넌트를 중첩해야 합니다.

그러나 많은 프레임워크에서는 `Fragment`이라는 것을 사용하여 컴포넌트 중첩을 피할 수 있습니다. `Fragment`을 기능으로 제공하는 컴포넌트 기반 프레임워크에는 다음과 같은 것들이 있습니다(이에 국한되지 않음):

- [React](https://legacy.reactjs.org/docs/fragments.html)
- [Preact](https://preactjs.com/guide/v10/components/#fragments)
- [Vue](https://v3-migration.vuejs.org/new/fragments.html)
- [Svelte](https://svelte.dev/tutorial/svelte-fragment)

원하는 프레임워크에서 조각을 사용하면 DOM 깊이를 줄일 수 있습니다. DOM 구조를 평평하게 만드는 것이 스타일링에 미치는 영향이 걱정된다면 `flexbox`나 `grid`와 같은 최신(그리고 더 빠른) 레이아웃 모드를 사용하면 도움이 될 수 있습니다.

## Other strategies to consider

DOM 트리를 평평하게 만들고 불필요한 HTML 요소를 제거하여 DOM을 최대한 작게 유지하더라도 사용자 상호 작용에 따라 변경되기 때문에 여전히 상당히 커서 많은 렌더링 작업이 시작될 수 있습니다. 이러한 상황에 처한 경우 렌더링 작업을 제한하기 위해 고려할 수 있는 몇 가지 다른 전략이 있습니다.

### Consider an additive approach

페이지가 처음 렌더링될 때 페이지의 많은 부분이 사용자에게 처음에 표시되지 않는 위치에 있을 수 있습니다. 이 경우 시작 시 DOM의 해당 부분을 생략하여 HTML을 지연 로드하고, 사용자가 페이지의 초기 숨겨진 부분을 필요로 하는 부분과 상호 작용할 때 추가할 수 있습니다.

이 접근 방식은 초기 로딩 중은 물론 이후에도 유용합니다. 초기 페이지 로드 시에는 렌더링 작업을 미리 덜 수행하므로 초기 HTML 페이로드가 더 가벼워지고 렌더링 속도가 빨라집니다. 따라서 이 중요한 시기의 인터랙션이 메인 스레드의 관심을 끌기 위한 경쟁을 줄이면서 실행될 수 있는 기회가 더 많아집니다.

페이지의 많은 부분이 로드 시 처음에 숨겨져 있는 경우 렌더링 작업을 다시 트리거하는 다른 상호 작용의 속도를 높일 수도 있습니다. 하지만 다른 인터랙션이 DOM에 더 많이 추가되면 페이지 수명 주기 동안 DOM이 증가함에 따라 렌더링 작업도 증가합니다.

시간이 지남에 따라 DOM에 추가하는 것은 까다로울 수 있으며 나름의 장단점이 있습니다. 이 경로를 사용하는 경우 사용자 상호 작용에 대한 응답으로 페이지에 추가하려는 HTML을 채울 데이터를 가져오기 위해 네트워크 요청을 할 가능성이 높습니다. 기내 네트워크 요청은 INP에 포함되지 않지만 체감 지연 시간을 증가시킬 수 있습니다. 가능하면 로딩 스피너나 기타 표시기를 통해 데이터를 가져오는 중임을 표시하여 사용자가 무슨 일이 일어나고 있음을 알 수 있도록 하세요.

### Limit CSS selector complexity

브라우저는 CSS에서 Selector를 구문 분석할 때 DOM 트리를 탐색하여 해당 선택기가 현재 레이아웃에 어떻게 적용되는지, 그리고 적용되는지 이해해야 합니다. 이러한 Selector가 복잡할수록 페이지의 초기 렌더링을 수행하기 위해 브라우저가 수행해야 하는 작업이 많아질 뿐만 아니라 상호 작용의 결과로 페이지가 변경되는 경우 스타일 재계산(Recalculate Style) 및 레이아웃 작업도 증가합니다.

> https://web.dev/reduce-the-scope-and-complexity-of-style-calculations/

### Use the content-visibility property

CSS는 `content-visibility` 속성을 제공하는데, 이는 화면 밖 DOM 요소를 느리게 렌더링하는 효과적인 방법입니다. 요소가 뷰포트에 접근하면 필요에 따라 렌더링됩니다. `content-visibility`의 이점은 초기 페이지 렌더링에서 상당한 양의 렌더링 작업을 줄일 뿐만 아니라 사용자 상호 작용의 결과로 페이지 DOM이 변경될 때 오프스크린 요소에 대한 렌더링 작업을 생략할 수 있다는 점입니다.

> https://web.dev/content-visibility/

## Conclusion

DOM 크기를 꼭 필요한 만큼만 줄이는 것은 웹사이트의 INP를 최적화하는 좋은 방법입니다. 이렇게 하면 DOM이 업데이트될 때 브라우저가 레이아웃 및 렌더링 작업을 수행하는 데 걸리는 시간을 줄일 수 있습니다. DOM 크기를 의미 있게 줄일 수 없더라도 렌더링 작업을 DOM 하위 트리로 분리하는 데 사용할 수 있는 몇 가지 기술(예: CSS 포함 및 `content-visibility` CSS 속성)이 있습니다.

렌더링 작업이 최소화되는 환경을 조성하고 상호 작용에 대한 응답으로 페이지가 수행하는 렌더링 작업의 양을 줄이면 결과적으로 웹사이트가 사용자와 상호 작용할 때 더 빠르게 반응하는 느낌을 받을 수 있습니다. 즉, 웹사이트의 INP가 낮아지고 이는 곧 더 나은 사용자 경험으로 이어집니다.
