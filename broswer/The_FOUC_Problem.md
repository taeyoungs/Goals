# The FOUC Problem

> 원본 글  
> https://webkit.org/blog/66/the-fouc-problem/

**목차**

- [The FOUC Problem](#the-fouc-problem)
  - [Stall](#stall)
  - [Don’t Stall](#dont-stall)
  - [On-Demand Stall](#on-demand-stall)
  - [Why does this matter?](#why-does-this-matter)

FOUC는 스타일이 지정되지 않은 콘텐츠가 보이는 현상을 의미합니다. 이 상황은 웹 브라우저가 웹 페이지의 콘텐츠를 스타일 정보 없이 표시할 때마다 발생합니다. 브라우저의 엔진 설계 방식과 웹 사이트 작성자가 사이트를 설계할 때 내린 흥미로운 가정에 따라 브라우저가 FOUC 현상을 만들어 내는 방법이 크게 달라지기 때문에 흥미로운 기술적 문제입니다.

브라우저는 웹 페이지를 로드할 때 먼저 웹 사이트에서 HTML 파일을 가져옵니다. 해당 HTML 파일이 웹 사이트에서 다운로드되면 브라우저 엔진은 해당 파일의 콘텐츠를 HTML Parser에 공급하기 시작합니다. 브라우저 엔진은 파일에서 `element`와 `node`를 발견하면 웹 페이지의 구조를 나타내는 트리(DOM 트리)를 구축합니다. 구문 분석(파싱) 중 어느 시점에서 엔진은 `<style>` 블록 내부 또는 `<link>` 요소에 있는 스타일시트 로딩 지시문에 도달하게 됩니다.

이 시점에서 상황이 흥미로워집니다. 이제 브라우저 엔진은 선택을 해야 합니다. 원격 `stylesheet`가 로드되기를 기다리는 동안 구문 분석(파싱) 프로세스를 지연시킬 수도 있고, 용감하게 앞으로 나아갈 수도 있습니다. 각 옵션을 살펴보겠습니다.

## Stall

브라우저 엔진이 구문 분석을 중단하면 이제 전체 페이지의 처리가 보류됩니다. 네트워크 계층(일반적으로 다른 스레드에 있음)이 여전히 엔진에 데이터를 제공할 수 있지만 이 데이터는 기본적으로 처리되지 않은 채 대기열에 대기합니다. 사실상 전체 엔진은 이 `stylesheet`가 완료되기를 기다리는 동안 아무것도 하지 않습니다. `Gecko`에는 이러한 동작이 있습니다.

이 접근 방식의 한 가지 주요 단점은 `stylesheet`와 `script`를 동시에 로드할 수 없다는 것입니다. 웹 페이지에 `link` 요소 뒤에 `script` 요소가 있는 경우 엔진은 `stylesheet`가 완전히 로드될 때까지 `script` 요소에 대한 로딩을 시작하지 않습니다. `stylesheet` 자체에서 다른 `stylesheet`를 가져오려는 경우 상황은 더욱 악화됩니다. HTML 파일의 지시어로 표시되는 전체 `stylesheet` 트리가 로드될 때까지 엔진이 계속 지연되기 때문입니다.

이 선택은 `stylesheet`를 `script`와 동일하게 동작하게 하고 웹 페이지 `head`의 어떤 부분에서도 로드 병렬화가 발생하지 않도록 효과적으로 보장합니다. 좋지 않게 들리는데 왜 그럴까요? 근본적인 이유는 웹 작성자가 기본적으로 `script`를 통해 액세스하는 **스타일 정보가 최신 상태일 것이라고 가정하는 코드를 이미 작성했기 때문**입니다.

예를 들어 `link` 요소를 따르는 `script`를 생각해 보겠습니다. 해당 `script`가 요소의 `offset` 높이를 요청하고 해당 높이가 CSS 파일에 지정되어 있다면 웹 페이지 작성자는 올바른 높이가 표시될 것으로 기대합니다. 이 동작은 표준의 일부가 아니지만 웹 디자이너는 암묵적으로 직렬로 로딩한다라는 가정을 `script`에 구축해 왔습니다(파일을 병렬로 읽는 것이 아닌 직렬, 즉 순차적으로 읽는 것을 가정한다는 뜻인 것 같다).

어떻게 이런 일이 발생했을까요? 기존 엔진이 이런 식으로 작동했기 때문이기도 하지만, `script` 요소의 **동기적 특성** 때문이기도 합니다. `script`가 **이미 구문 분석(파싱)을 지연시키기 때문에** 작성자는 `stylesheet`도 구문 분석을 지연시킬 것이라는 **당연한 가정을 한 것입니다**.

평가: 👎

## Don’t Stall

`WebKit`은 그 반대 동작을 수행하며 `stylesheet` 지시문이 발생한 후에도 페이지를 계속 구문 분석하여 `stylesheet` 및 `script` 로드가 병렬화될 수 있도록 합니다. 이렇게 하면 모든 것을 훨씬 더 일찍 표시할 준비가 될 수 있습니다.

이 동작의 문제점은 `script`가 **정확한 레이아웃/스타일 정보가 있어야 응답할 수 있는 속성에 액세스하려고 시도할 때 어떻게 해야 하는가**입니다. 이런 상황이 발생했을 때 Shipping Safari의 현재 동작은 다음과 같습니다. 아직 스타일시트가 없더라도 가지고 있는 것을 레이아웃합니다. 또한 이를 표시합니다. 즉, `stylesheet`가 로드되기 전에 `script`가 `scrollHeight` 또는 `offsetWidth`와 같은 속성에 액세스하려고 시도할 때마다 **FOUC**가 표시됩니다.

최소한 스타일이 지정되지 않은 콘텐츠의 렌더링이 억제되도록 WebKit nightlies 업데이트의 기능을 약간 개선했습니다. 그러나 최신 정보를 제공하는 스타일시트가 아직 로드되지 않았기 때문에 페이지에서 잘못된 것으로 간주할 수 있는 정보는 여전히 제공될 것입니다.

이 접근 방식은 스타일/레이아웃 프로퍼티에 액세스하지 않는 경우 `stalling` 접근 방식보다 속도가 더 빠릅니다. 그러나 **스타일이 지정되지 않은 렌더링과 최종 스타일이 지정된 렌더링 사이에서 엔진이 수행해야 하는 왜곡이 얼마나 심한지에 따라** 이러한 프로퍼티에 액세스하는 경우 `stalling` 접근 방식보다 속도가 떨어질 수 있습니다.

평가: 👎

## On-Demand Stall

이 문제에 대한 이상적인 기술적 해결책은 두 가지 접근 방식을 혼합한 `on-demand stalling`입니다. 이 접근 방식에서는 엔진이 여전히 로드를 병렬 처리하고 구문 분석을 중단하지 않습니다. 하지만 레이아웃이나 스타일 프로퍼티에 액세스하면 자바스크립트 엔진이 일시 중단되고 보류 중인 `stylesheet`가 로드될 때까지 기다립니다. 모든 `stylesheet`가 로드되면 `script`는 중단된 부분부터 다시 시작합니다.

이 접근 방식의 장애물은 `script` 실행 도중에 **중단했다가 나중에 다시 시작할 수 있어야 한다는 점**입니다. 기술적으로 열등한 해결책은 **모든** `script`의 실행을 중단하는 것일 수 있지만, **대부분의 웹 사이트에서 스크립트 요소가 압도적으로 많다는 점을 고려할 때 이러한 접근 방식은 항상 중단하는 것보다 훨씬 낫지 않습니다**.

평가: 👍

## Why does this matter?

**FOUC** 문제는 일반적으로 사소한 문제일 수 있습니다. 하지만 Google 애드센스의 등장으로 인해 **FOUC**는 Safari의 전염병이 되었습니다. 이러한 Google 광고는 인라인 스크립트를 실행할 뿐만 아니라 페이지에서 사용되지도 않는 레이아웃 정보에 액세스하기 때문에 **FOUC** 문제는 생각보다 훨씬 더 심각합니다.

엔진이 정답을 제공하기 위해서는 레이아웃과 스타일이 최신 상태인지 확인해야 하므로 페이지 로딩 주기 동안 스타일/레이아웃 정보에 반복적으로 액세스하는 것은 잘못된 코딩입니다. 엔진이 `script`에 정답을 제공하기 위해 전체 페이지의 정확한 스냅샷을 계산하는 동안 웹 페이지 표시를 지연시키는 병목 현상으로 `offsetWidth`와 같은 속성의 모든 사용을 생각해 보세요. `script`가 **이러한 속성에 반복적으로 액세스하면서 액세스 사이에 다른 레이아웃/스타일을 여러 번 변경하면 모든 최신 브라우저에서 웹 사이트의 로드 속도가 저하될 수 있습니다**.