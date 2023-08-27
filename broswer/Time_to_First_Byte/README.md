# Time to First Byte: What it is and How to Make Improvements

> 원본 글  
> https://calibreapp.com/blog/time-to-first-byte

**목차**

- [Time to First Byte: What it is and How to Make Improvements](#time-to-first-byte-what-it-is-and-how-to-make-improvements)
  - [개요](#개요)
  - [What does TTFB measure?](#what-does-ttfb-measure)
  - [Why should you improve your TTFB?](#why-should-you-improve-your-ttfb)
  - [What is a good TTFB?](#what-is-a-good-ttfb)
  - [4 ways to improve TTFB](#4-ways-to-improve-ttfb)
    - [1. Upgrade your hosting or servers](#1-upgrade-your-hosting-or-servers)
    - [2. Use a Content Delivery Network (CDN)](#2-use-a-content-delivery-network-cdn)
    - [3. Reduce DNS lookup time](#3-reduce-dns-lookup-time)
    - [4. Always default to server-side rendered HTML](#4-always-default-to-server-side-rendered-html)

## 개요

Time to First Byte(TTFB)은 브라우저가 서버에 연결하여 데이터를 다운로드하는 속도를 반영하는 측정 항목입니다. 사이트 전반에서 TTFB 시간을 추적하면 성능에 부정적인 영향을 미치는 서버의 속도 저하 또는 문제를 감지할 수 있습니다.

TTFB는 핵심 웹 필수 지표(Core Web Vital)는 아니지만 모두가 추적해야 하는 중요한 사이트 성능 지표입니다. TTFB가 낮으면 서버가 충분히 빠르게 응답하지 않아 전체 페이지 로딩 프로세스에 연쇄적인 성능 문제가 발생할 수 있음을 나타냅니다. 이러한 시간은 단순히 성가신 문제가 아니라 방문자가 이탈하여 경쟁업체로 이동할 수 있는 직접적인 원인이 될 수 있습니다.

이 가이드에서는 측정 방법, 중요한 이유, 개선 방법 등 Time to First Byte 지표에 대해 알아야 할 모든 것을 알려드립니다. 이 지식을 바탕으로 방문자를 위한 빠른 사이트 환경을 만들 수 있습니다.

## What does TTFB measure?

**TTFB는 방문자가 링크를 클릭하거나 키보드의 엔터 키를 누른 시점과 해당 페이지의 첫 바이트가 다운로드되는 시점 사이의 시간을 측정합니다**.

대부분의 사람들에게 이 과정은 즉각적으로 느껴지겠지만, 그 이면에서는 여러 가지 일이 일어나고 있습니다. 페이지의 TTFB는 다음과 같은 모든 작업의 합계입니다:

- **리디렉션 시간**(Redirect time): URL이 301, 302 등에 의해 이동된 경우와 같이 URL 리디렉션이 발생한 경우 이를 해결하는 데 걸리는 시간입니다.
- **Service worker 시작 시간**(Service worker startup time): 모든 경우에 적용되지는 않습니다. Service worker API는 브라우저에서 실행되며 서버에서 요청이 반환되기 전이나 후에 요청을 수정할 수 있다는 점에서 프록시 서버와 유사합니다.
- **DNS 조회**(DNS lookup): DNS 조회는 칼리버앱닷컴과 같은 URL에 사용하는 단어를 브라우저가 올바른 웹페이지를 다운로드하는 데 사용할 수 있는 IP 주소로 변환합니다.
- **연결 및 TLS 협상**(Connection and TLS negotiation): 데이터를 전송할 수 있도록 브라우저와 서버가 연결을 형성하는 데 걸리는 시간입니다.
- **요청**(The request): 브라우저가 서버에 데이터를 요청하고 서버가 이를 다시 전송하는 과정. TTFB는 첫 번째 바이트가 넘어간 후 측정을 중지합니다.

사람들이 사이트에 액세스할 때 여러 요인에 따라 TTFB가 크게 달라질 수 있다는 점에 유의해야 합니다. 사용자가 서버에서 멀리 떨어져 있을수록 데이터가 물리적으로 사용자에게 도달하는 데 시간이 더 오래 걸립니다. 따라서 시드니와 런던에 있는 사람의 TTFB 시간은 완전히 다를 수 있으며 페이지에서 매우 다른 경험을 할 수 있습니다.

## Why should you improve your TTFB?

TTFB는 페이지 속도의 중추입니다. 좋은 TTFB는 빠른 로딩 시간을 보장하지는 않지만, 나쁜 TTFB는 사이트가 느리게 느껴지기에 충분하며 사람들은 느린 사이트를 좋아하지 않습니다. Google의 연구에 따르면 단 1초만 추가되어도 이탈률이 크게 증가할 수 있습니다. 페이지가 독자에게 깊은 인상을 남기려면 TTFB가 가능한 한 빨라야 합니다.

느린 사이트는 이탈을 증가시킬 뿐만 아니라 Google의 사이트 순위에도 영향을 미칩니다. Google은 검색 엔진에서 페이지 경험을 순위 결정 요소로 고려한다고 밝혔습니다. 사이트가 좋은 경험을 제공하는지 판단하는 요소는 여러 가지가 있는데, 그 중 하나가 Core Web Vitals(부분적으로 TTFB에 의존하는 성능 지표 세트)입니다.

예를 들어, Largest Contentful Paint(LCP)는 페이지의 스크롤 위 부분에서 가장 데이터 집약적인 요소를 로드하는 데 걸리는 시간을 측정하는 Core Web Vitals입니다. 페이지가 서버에서 데이터를 다운로드하는 데 시간이 오래 걸리는 경우(즉, TTFB가 느린 경우) LCP 점수는 더 나빠집니다. Google은 이를 부정적인 페이지 경험 신호로 간주하여 검색 순위에 영향을 미칩니다.

## What is a good TTFB?

TTFB는 빠르면 빠를수록 좋습니다. 대부분의 페이지는 800ms 미만의 TTFB를 목표로 해야 합니다. 800ms를 초과하면 로드 시간이 눈에 띄게 느려질 위험이 있습니다.

<img src="images/1.avif" alt="Time to First Byte 예시" width="600" />

사이트의 모든 페이지에는 서로 다른 TTFB가 있으며 이 점수는 방문자의 위치에 따라 크게 달라진다는 점을 기억하세요. 따라서 다양한 위치에서 사이트의 페이지를 정기적으로 테스트하여 800ms 벤치마크에 미달하는 페이지를 파악하는 것이 중요합니다(자세한 내용은 나중에 설명합니다).

## 4 ways to improve TTFB

TTFB를 낮추려면 첫 번째 바이트가 로드되기 전에 하나 이상의 프로세스에서 귀중한 ms를 줄여야 합니다. 다음은 이를 위한 몇 가지 방법입니다.

### 1. Upgrade your hosting or servers

이 단계는 TTFB를 눈에 띄게 개선하기 위해 할 수 있는 가장 중요한 작업입니다. 더 나은 호스팅이나 서버에 투자하면 연결 시간을 단축하고 브라우저에 데이터를 더 빠르게 전송할 수 있습니다.

경우에 따라서는 서버를 업그레이드하기만 하면 됩니다. 메모리 또는 CPU 제한으로 인해 성능에 제약이 있는 경우, 업그레이드를 통해 더 나은 TTFB 결과를 얻을 수 있습니다.

다른 경우에는 호스팅 제공업체를 변경하는 것이 더 적절한 해결책일 수 있습니다. 하지만 어떤 호스팅 제공업체가 더 나은지 항상 명확하게 알 수 있는 것은 아닙니다. 여러 호스팅 옵션을 평가할 때는 이러한 점을 고려해야 합니다:

- 전용 서버를 제공
- 경쟁사보다 우수한 CPU, 메모리, GPU 사양을 보유
- 사이트 방문자의 위치와 가까운 곳에 서버를 보유

### 2. Use a Content Delivery Network (CDN)

CDN은 전 세계에 분산된 서버 네트워크에 콘텐츠를 캐싱할 수 있기 때문에 TTFB를 개선합니다. 더 많은 서버에 액세스할 수 있으므로 방문자에게 더 빠르게 콘텐츠를 제공할 수 있습니다(방문자와 서버의 물리적 근접성은 데이터가 브라우저에 도달하는 데 걸리는 시간에 영향을 미칩니다).

CDN은 또한 자동으로 도움을 줍니다:

- 원본 서버가 처리해야 하는 요청 수가 줄어들어 부하가 줄어듭니다.
- 파일 크기를 줄여 더 빠른 전송이 가능합니다.
- 더 빠른 속도를 위해 서버 설정을 최적화합니다.

### 3. Reduce DNS lookup time

DNS 조회(DNS lookup)는 사람들이 사용하는 URL을 컴퓨터에서 사용하는 IP 주소로 변환하기 때문에 페이지 로딩 프로세스의 필수적인 부분입니다. 서버가 브라우저에 첫 번째 바이트의 데이터를 다시 보내기 전에 이 확인 작업을 수행해야 합니다. 따라서 DNS 조회 프로세스를 최적화하면 TTFB 시간을 단축하는 데 도움이 됩니다.

DNS 조회 시간을 단축하는 몇 가지 방법은 다음과 같습니다:

- DNS 레코드에 장기 TTL 레코드가 있는지 확인하여 DNS를 최소 24시간 동안 캐시할 수 있도록 합니다.
- 페이지 로드 중 발생할 수 있는 지연을 줄이기 위해 `dns-prefetch`를 사용합니다.
- 페이지 리디렉션을 모두 제거합니다.
- 글꼴 또는 CSS 파일과 같이 일반적으로 사용되는 요소 캐싱합니다.

### 4. Always default to server-side rendered HTML

서버 또는 캐시 설정을 조정하는 것은 이 목록의 다른 옵션보다 훨씬 더 복잡합니다. 하지만 최상의 TTFB 결과를 얻으려면 서버를 최적화하여 데이터를 최대한 빨리 전송해야 합니다.

서버가 더 빠르게 반응하도록 하는 한 가지 방법은 Server Side Rendering(SSR)을 사용하는 것입니다. SSR은 페이지가 서버에서 렌더링된 후 브라우저로 전송되는 것을 의미합니다. 자주 업데이트되는 페이지의 경우 SSR을 활용하여 페이지를 미리 렌더링하고 일정 시간 동안 캐시한 다음 브라우저에서 페이지를 요청할 때 신속하게 전송할 수 있습니다.
