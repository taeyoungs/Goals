# Cross-Origin Read Blocking for Web Developers

> 원본 글  
> https://www.chromium.org/Home/chromium-security/corb-for-developers/

**목차**

- [Cross-Origin Read Blocking for Web Developers](#cross-origin-read-blocking-for-web-developers)
  - [개요](#개요)
  - [How can I ensure CORB protects resources on my website?](#how-can-i-ensure-corb-protects-resources-on-my-website)
    - [For HTML, JSON, and XML resources:](#for-html-json-and-xml-resources)
    - [For other resource types (e.g. PDF, ZIP, PNG)](#for-other-resource-types-eg-pdf-zip-png)
  - [What should I do about CORB warnings reported by Chrome?](#what-should-i-do-about-corb-warnings-reported-by-chrome)

## 개요

**Cross-Origin Read Blocking** (**CORB**)은 **side-channel** 공격들(**Spectre**를 포함한)의 위협을 완화하는데 도움을 주는 새로운 웹 플랫폼 보안 기능이다.

> **Spectre**(이하 스펙터)는 프로그램을 속여 해당 프로그램의 메모리 공간 내 임의의 주소에 접근하도록 하는 취약점이다. 공격자는 이 취약점을 이용하여 데이터에 접근할 수 있고, 잠재적으로 민감한 데이터를 얻는데 활용할 수도 있다.
> 단일한 방어책이 존재하는 여타 취약점과는 달리, 스펙터 백서에는 다양한 종류의 변종 취약점들이 소개되고 있는데, 이들은 모두 예측적 실행 (특히 **분기 예측**) 중에 발생하는 주변 효과(사이드 이펙트)를 이용한다. 예측적 실행은 현대 마이크로프로세서에서 메모리 접근 지연시간을 감추어 전반적인 실행 속도를 향상시키는데 통용되는 일반적인 기술이다. 스펙터는 동시에 공개된 멜트다운 버그와도 관련성이 있지만, 멜트다운과는 달리 특정 프로세서의 구체적인 특징을 전제로 두지 않아 좀 더 일반적이다. (멜트다운의 경우 메모리 관리/보호 체계에 대한 특징을 전제로 한다.)

> **분기 예측**이란 다음 실행될 [조건문](https://ko.wikipedia.org/wiki/%EC%A1%B0%EA%B1%B4%EB%AC%B8)이 어떤 곳으로 분기할 것인지를 확실히 알게 되기 전에 미리 추측하는 **CPU** 기술이다. **분기 예측기**(branch predictor)는 분기 예측을 수행하는 [디지털 회로](https://ko.wikipedia.org/wiki/%EB%94%94%EC%A7%80%ED%84%B8_%ED%9A%8C%EB%A1%9C)를 가리킨다. 분기 예측을 수행하는 목적은, [명령어 파이프라인](https://ko.wikipedia.org/wiki/%EB%AA%85%EB%A0%B9%EC%96%B4_%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8)이 일시적으로 정지되지 않도록 하는 것이다. 최신 [x86](https://ko.wikipedia.org/wiki/X86) 아키텍처와 같이 많은 수의 파이프라인을 사용하는 구조에서 분기 예측기는 전체적인 성능을 크게 향상시킬 수 있다.

**CORB**는 민감한 정보를 가지고 있거나 기존 웹 기능에 필요하지 않은 경우 브라우저가 특정 **cross-origin** 네트워크 응답을 웹 페이지에 전달하지 못하도록 설계됐다. 예를 들어, `<script>` 또는 `<img>` 태그에서 요청한 **cross-origin** `text/html` 응답을 차단하고 대신 빈 응답으로 바꾼다. 이는 [Site Isolation](https://www.chromium.org/Home/chromium-security/site-isolation)에 포함된 보호 기능의 중요한 부분 중 하나이다.

이 문서는 웹 개발자가 **CORB**에 대응하여 취해야 할 조치를 알도록 하는 것을 돕는데 초점이 맞춰져 있다. **CORB**에 대해 더 많은 것을 알고 싶다면 다음 문서들을 살펴보자.

- [https://chromium.googlesource.com/chromium/src/+/HEAD/services/network/cross_origin_read_blocking_explainer.md](https://chromium.googlesource.com/chromium/src/+/HEAD/services/network/cross_origin_read_blocking_explainer.md)
- [https://fetch.spec.whatwg.org/#corb](https://fetch.spec.whatwg.org/#corb)
- [https://developer.chrome.com/blog/site-isolation/](https://developer.chrome.com/blog/site-isolation/)

## How can I ensure CORB protects resources on my website?

웹사이트의 민감한 리소스(e.g. 페이지들 또는 특정 사용자의 정보를 담고 있는 **JSON** 파일 또는 **CSRF Token**이 포함된 페이지)가 다른 웹 출처로 누출되지 않도록 하려면 다음 단계를 수행하여 해당 리소스를 모든 사이트에서 포함할 수 있는 리소스와 구별하자. (e.g. 이미지들, **JavaScript** 라이브러리들)

### For HTML, JSON, and XML resources:

이러한 리소스가 아래 목록의 올바른 "**Content-Type**" 응답 헤더와 "**X-Content-Type-Options: nosniff**" 응답 헤더와 함께 제공되는지 확인한다. 이러한 헤더들은 **Chrome**으로 하여금 리소스의 내용에 의존하지 않고 보호가 필요한 리소스들을 식별할 수 있게 한다.

- **HTML MIME type** - `text/html`
- **XML MIME type** - `text/html`, `application/xml` 또는 서브 타입이 `+xml`로 끝나는 **MIME type**
- **JSON MIME type** - `text/json`, `application/json` 또는 서브 타입이 `+json`로 끝나는 **MIME type**

민감한 리소스에는 **multipart [range request](https://developer.mozilla.org/en-US/docs/Web/HTTP/Range_requests)**를 지원하지 않는 것을 추천한다. 왜냐하면 해당 기능을 지원하게 될 경우 **MIME type**을 `multipart/byteranges`로 변경하게 되며 이는 **Chrome**이 사용자를 보호하기 힘들게 만들기 때문이다. 일반적으로 **range request**는 문제가 되지 않으며 `nosniff` 경우와 유사하게 처리된다.

위의 사례 외에도 **Chrome**은 위의 **MIME** **type**으로 레이블이 지정된 응답과 `nosniff` 헤더가 없는 응답을 보호하기 위해 최선을 다하지만 여기에는 한계가 있다. 웹에 있는 많은 **JavaScript** 파일에는 불행히도 이러한 **MIME type** 중 일부를 사용하여 레이블이 지정되어 있으며 **Chrome**에서 해당 파일들에 대한 액세스를 막을 경우 기존 웹사이트가 중단된다. 따라서 `nosniff` 헤더가 없으면 **Chrome**은 파일을 보호할지 여부를 결정하기 전에 먼저 파일의 시작 부분을 보고 **HTML**, **XML** 또는 **JSON**인지 확인한다. 만약 이를 확인할 수 없다면, **cross-site** 페이지의 프로세스에서 응답을 수신하도록 허용한다. 이는 기존 사이트와의 호환성을 유지하면서 일부 제한된 보호를 추가하는 최선의 접근 방식이다. 우리는 웹 개발자들에게 브라우저가 `confirmation sniffing` 접근 방식에 의존하지 않으면서 리소스를 보호하기 위해 `nosniff` 헤더를 포함할 것을 권장한다.

### For other resource types (e.g. PDF, ZIP, PNG)

위 리소스들에 대해서는 이러한 리소스가 추측할 수 없는 **CSRF Token**을 포함하는 요청에 대한 응답으로만 제공되는지 확인하자. (위의 **HTML**, **JSON** 또는 **XML** 단계를 통해 보호되는 리소스를 통해 배포되어야 한다)

> 추측할 수 없는 **CSRF Token**을 가지고 있다는 건 **CSRF** 공격을 방지하기 위한 **Token**이 포함되어 있다는 것이니 그러한 **Token**을 포함하고 있는 요청에 대해서만 응답으로 제공해야 한다는 뜻이다.
>
> 또한 `nosniff` 헤더를 포함하여 보호되고 있는 리소스를 통해 배포가 되어야 한다.

## What should I do about CORB warnings reported by Chrome?

**CORB**가 **HTTP** 응답을 막았을 때, **Chrome** 개발자 도구 콘솔에 다음과 같은 경고 메시지가 발생한다.

> Cross-Origin Read Blocking (CORB) blocked cross-origin response https://www.example.com/example.html with MIME type text/html. See [https://www.chromestatus.com/feature/5629709824032768](https://www.chromestatus.com/feature/5629709824032768) for more details.

대부분의 경우 차단된 응답은 웹 페이지의 동작에 영향을 미치지 않아야 하며 **CORB** 오류 메시지는 무시해도 된다. 예를 들어, 차단된 응답의 **body**가 이미 비어 있거나 응답을 처리할 수 없는 컨텍스트로 응답이 전달될 때 경고가 발생할 수 있다. (e.g. `<img>` 태그로 전달되는 404 오류 페이지와 같은 **HTML** 문서)

드문 경우지만 **CORB** 경고 메시지는 특정 응답이 차단될 때 웹 사이트의 동작을 방해할 수 있는 웹 사이트의 문제를 나타낼 수 있다. 예를 들어, "**X-Content-Type-Options: nosniff**" **Response** 헤더와 잘못된 "**Content-Type**" **Response** 헤더가 함께 제공되는 **Response**인 경우 차단될 수 있다. 예시를 하나 들면 "**Content-Type: text/html**"이면서 `nosniff`로 잘못 표시된 실제 이미지의 경우 차단될 수 있다. 이러한 상황이 발생하여 페이지 동작을 방해하는 경우 웹사이트에 알리고 응답에 대한 "**Content-Type**" 헤더를 수정하도록 요청하는 것이 좋다.

웹 개발자 본인의 의도와는 다르게 크롬과 같은 브라우저에서 **CORB** 경고가 발생하고 있다면 **Content Type**과 `nosniff` 헤더를 정상적으로 사용하고 있는지 확인하자.
