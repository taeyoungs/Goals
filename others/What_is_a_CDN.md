# What is a CDN? An Unbiased Guide to Content Delivery Networks

> 원본 글  
> https://calibreapp.com/blog/content-delivery-networks-guide?ck_subscriber_id=1780635791

**목차**

- [What is a CDN? An Unbiased Guide to Content Delivery Networks](#what-is-a-cdn-an-unbiased-guide-to-content-delivery-networks)
  - [개요](#개요)
  - [What is a CDN?](#what-is-a-cdn)
  - [How do CDNs work?](#how-do-cdns-work)
  - [What are the benefits of CDNs?](#what-are-the-benefits-of-cdns)
  - [What to consider when choosing a CDN](#what-to-consider-when-choosing-a-cdn)
  - [How to introduce a new CDN](#how-to-introduce-a-new-cdn)
  - [How to test CDN performance](#how-to-test-cdn-performance)

## 개요

콘텐츠 전송 네트워크(CDN)는 사이트의 성능과 안정성을 개선하는 가장 좋은 방법 중 하나입니다. 페이지 속도를 높이고, 서비스 거부 공격(denial of service attacks)을 방어하고, 콘텐츠를 최적화하여 더 빠르게 전송할 수 있습니다.

CDN을 찾고 있다면 선택할 수 있는 옵션이 많습니다. 웹사이트에 적합한 것을 선택하는 것은 중요한 결정이므로 충분한 조사를 해야 합니다. 하지만 한 가지 문제가 있습니다. CDN에 대한 검색 결과의 최상위에는 모두 CDN을 판매하는 회사의 가이드가 표시됩니다. 영업사원이 작성한 블로그 게시물을 어떻게 신뢰할 수 있을까요?

다행히도 당사는 CDN을 판매하지는 않지만 웹 성능 전문가로서 CDN에 대해 한두 가지 정도는 확실히 알고 있습니다.

CDN을 선택할 때 정보에 입각한 결정을 내릴 수 있도록 이 가이드를 작성했습니다. CDN이 무엇인지, 사이트에 CDN을 도입하면 어떤 이점이 있는지, CDN을 선택하고 테스트하는 방법을 간략하게 설명합니다. 이 리소스를 통해 사이트의 요구 사항에 완벽하게 맞는 CDN을 찾을 수 있습니다.

## What is a CDN?

CDN은 콘텐츠를 호스팅하고 방문자에게 빠르게 전달할 수 있는 전 세계에 분산된 서버 네트워크입니다. CDN은 지연 시간을 줄이고 안정성을 개선하며 콘텐츠를 최적화하여 더 빠르게 전송할 수 있도록 메인 호스팅 서버를 지원하는 데 도움이 됩니다.

웹사이트를 글로벌 택배 회사라고 생각하세요. 매일 전 세계의 사람들이 여러분의 웹사이트 콘텐츠를 원하고, 이를 신속하게 전달받기를 원합니다. 웹 콘텐츠 배송을 하나의 대규모 창고에만 의존한다면 다음과 같은 여러 가지 문제에 노출될 수 있습니다:

- 바쁜 날에는 물류창고에 과부하가 걸릴 수 있습니다.
- 한 번의 장애로 전체 네트워크가 중단될 수 있습니다.
- 멀리 떨어진 고객에게는 배송 시간이 더 오래 걸립니다.

CDN은 전 세계에 소규모의 로컬 창고를 배치한 것과 같습니다. 로컬 창고는 중앙 창고(호스트 서버)에서 콘텐츠를 받아 방문자가 콘텐츠를 주문할 때까지 저장합니다. 이렇게 하면 CDN 서버 위치가 전 세계에 분산되어 있어 한 서버가 다운되더라도 다른 서버가 그 공백을 메울 수 있으므로 콘텐츠를 안정적이고 빠르게 전송할 수 있습니다.

## How do CDNs work?

CDN은 전 세계 전략적 위치에 서버를 보유하고 있기 때문에 콘텐츠를 더 빠르게 전송할 수 있습니다. 글로벌 서버 네트워크가 유리한 이유를 이해하려면 CDN 없이 인터넷에서 데이터가 어떻게 이동하는지 생각해 보세요.

덴버의 서버에서 뉴질랜드의 가정으로 데이터를 전송하려면 샌프란시스코, 시드니, 오클랜드의 주요 허브를 통과한 후 뉴질랜드 현지 인터넷 서비스 제공업체(ISP)를 통해 라우팅되어야 합니다. 이 모든 라우팅에는 시간이 걸립니다. 특정 경로가 손상되었거나 사용량이 많은 경우 다른 경로를 통해 트래픽을 라우팅해야 하므로 시간이 더 오래 걸릴 수 있습니다.

CDN은 콘텐츠를 전 세계에 미리 저장하여 이러한 문제를 해결합니다. 예를 들어, 요청이 있을 때 덴버에서 데이터를 가져오는 대신 CDN은 시드니의 서버에 데이터를 저장하여 뉴질랜드에 더 빨리 도달할 수 있도록 합니다.

## What are the benefits of CDNs?

CDN은 콘텐츠 전송 시간을 단축하고 지연 시간을 줄이며 [Time to First Byte](https://calibreapp.com/blog/time-to-first-byte)(TTFB)을 향상하는 데만 도움이 되는 것이 아닙니다. 다음과 같은 이점도 있습니다:

- **원본 서버의 부하를 줄입니다**(Reducing the load on your origin server:) 강력한 서버도 한계가 있습니다. 대규모 세일을 진행하면 모든 트래픽이 오리진 서버를 빠르게 압도할 수 있습니다. CDN은 이러한 트래픽을 여러 서버로 분산하므로 한 서버에 과부하가 걸리지 않습니다.
- **파일 크기 축소**(Shrinking file sizes): 대부분의 CDN은 콘텐츠 파일을 자동으로 최적화하여 사이트 방문자에게 데이터를 더 빠르게 전송합니다.
- **보안 개선**(Improving security): DDoS 공격은 대량의 봇 트래픽으로 서버를 압도하여 사이트를 다운시키려고 시도합니다. 한 서버가 다운되면 다른 서버가 이를 대신할 수 있으므로 CDN은 이러한 위협을 완화하는 데 도움이 됩니다.
- **이미지 및 동영상 최적화**(Optimising images and videos): 전문 CDN을 사용하면 다양한 종류의 콘텐츠를 더 쉽게 호스팅하고 전송할 수 있습니다. 이러한 CDN은 파일 유형을 자동으로 변경하는 등의 전략을 사용하여 콘텐츠를 최대한 빠르고 안정적으로 전송합니다.

모든 CDN이 동일한 혜택을 제공하는 것은 아니므로 잠재적 제공업체를 조사할 때 이러한 혜택을 염두에 두세요.

## What to consider when choosing a CDN

아마존 웹 서비스(AWS)나 구글 클라우드와 같은 거대 업체부터 틈새 프리미엄 서비스까지 다양한 CDN 제공업체가 있습니다. 각 CDN에는 장단점이 있으며, 이 기술과 웹 성능을 처음 접하는 경우 압도적일 수 있습니다.

이러한 변수를 기반으로 각 CDN을 평가하여 자신에게 가장 적합한 CDN을 찾아보세요.

- **가격**(Price): 일부 CDN은 무료인 반면, 트래픽 수에 따라 약간의 요금이 부과되는 서비스도 있습니다. 비용을 지불할 수 없는 경우가 아니라면, 유료 CDN이 거의 항상 최선의 선택입니다.
- **서버 위치**(Server locations): 지리적으로 분산된 잠재 고객을 보유한 기업은 CDN의 이점을 가장 많이 누릴 수 있습니다. 멜버른 거주자를 대상으로 서비스를 제공하는 지역 기업이라면 런던에 있는 CDN 서버는 도움이 되지 않습니다. 고객과 가까운 곳에 서버가 있는 CDN 서비스를 선택해야 합니다.
- **직관적인 인터페이스**(Intuitive interface): CDN은 단순히 "설정하고 잊어버리는" 서비스가 아닙니다. CDN 설정을 최적화하거나 오래된 콘텐츠를 쉽게 제거할 수 있어야 합니다. 뛰어난 인터페이스가 있으면 이 작업이 훨씬 쉬워집니다.
- **고객 지원 수준**(Level of customer support): 완벽한 CDN은 없으며, 문제 해결을 위해 고객 지원팀에 문의해야 할 때가 있을 수 있습니다. 특히 팀에 서버와 CDN에 대해 잘 아는 사람이 없는 경우, 적시에 우수한 고객 서비스를 제공하는 것으로 입증된 CDN에 투자하는 것은 비용 대비 가치가 충분합니다.
- **추가 기능**(Extra features): 앞서 언급했듯이 일부 CDN은 틈새 웹사이트가 필요한 브랜드에 특화되어 있습니다. 이미지 CDN은 특히 이미지가 많은 웹사이트에 적합하고, 비디오 CDN은 비디오 스트리밍에 의존하는 사이트에 적합합니다. 사이트에 특정 요구 사항이 있는 경우, 해당 요구 사항을 충족하는 CDN이 있는지 확인하세요.

"최고의" CDN 서비스는 없으며, 비즈니스의 고유한 사이트 요구 사항에 따라 도구를 선택해야 합니다. 위의 기준을 사용하여 사이트에 적합한 몇 가지 좋은 경쟁 업체를 찾아보세요.

## How to introduce a new CDN

사이트에 CDN을 추가하는 것은 단순한 기본 설치 그 이상입니다. CDN의 설정 프로세스를 실행한 후에는 제대로 작동하는지 확인해야 합니다. 고객에게 사이트를 게시하기 전에 통제된 환경에서 사이트를 감사할 시간을 따로 확보하세요.

여기에는 다음과 같은 확인사항이 포함됩니다:

- 모든 이미지, 동영상 및 기타 미디어를 포함한 모든 페이지가 제대로 로드됩니다.
- 모든 타사 스크립트가 로드되고 예상대로 작동합니다.
- 브라우저 개발자 콘솔에 오류가 없습니다. CORS 문제, 실패한 요청 또는 기타 오류 메시지가 있는지 확인합니다.

CDN이 제대로 작동하고 있는지 확인했다면 이제 사이트 성능에 눈에 띄는 차이를 가져오는지 확인해야 합니다.

## How to test CDN performance

CDN이 성능에 영향을 미쳐서는 안 되지만, 기대했던 만큼 눈에 띄는 차이를 만들지 못하는 경우도 있습니다. CDN을 활성화 및 비활성화한 상태에서 웹 트래픽 속도를 확인하여 CDN의 영향을 테스트하세요.

특히 Time to First Byte(TTFB)과 Largest Contentful Paint(LCP)을 측정하고 싶을 것입니다. TTFB는 **서버가 첫 바이트의 데이터를 전송하는 데 걸리는 시간을 측정**하고, LCP는 **페이지에서 가장 눈에 띄는 요소가 로드되는 데 걸리는 시간을 측정**합니다. 좋은 CDN은 다양한 위치에서 두 지표의 속도를 모두 높여야 합니다. 이렇게 하면 CDN이 국내뿐 아니라 전 세계 사이트 방문자를 위해 제대로 작동하고 있다는 것을 알 수 있습니다.

약간의 계획만 세우면, CDN이 TTFB 및 LCP에 미치는 영향을 당사의 Core Web Vitals 검사기나 Google의 Pagespeed Insights와 같은 간단한 무료 테스터를 사용하여 테스트할 수 있습니다. 이러한 도구를 사용하면 CDN을 활성화하기 전에 기본 테스트를 수행한 다음 CDN을 활성화한 후 한 달 정도 후에 다시 테스트할 수 있습니다. 이러한 도구는 매월 업데이트되는 Google 크롬 사용자로부터 가져온 데이터 세트인 Google의 CRuX 데이터에 의존하므로 한 달 정도 기다려야 합니다. 안타깝게도 이러한 도구로는 특정 위치를 테스트할 수 없습니다.

여러 위치에 걸쳐 보다 세분화된 테스트를 원한다면 Calibre 테스트 플랫폼을 사용하세요. 이 플랫폼을 사용하면 원하는 만큼 자주 사이트를 테스트하여 CDN의 즉각적인 영향을 확인할 수 있습니다. 또한 특정 위치에서 페이지 속도를 테스트하여 CDN이 진정으로 전 세계에 영향을 미치는지 확인할 수 있습니다.