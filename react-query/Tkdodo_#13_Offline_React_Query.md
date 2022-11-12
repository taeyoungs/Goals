# #13 Offline React Query

> 원본 글  
> https://tkdodo.eu/blog/offline-react-query

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- [#3 React Query Render Optimizations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%233_React_Query_Render_Optimizations.md)
- [#4 Status Checks in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%234_Status_Checks_in_React_Query.md)
- [#6 React Query and TypeScript](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%236_React_Query_and_TypeScript.md)
- [#7 Using WebSockets with React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%237_Using_WebSockets_with_React_Query.md)
- [#8 Effective React Query Keys](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238_Effective_React_Query_Keys.md)
- [#8a Leveraging the Query Function Context](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%238a_Leveraging_the_Query_Function_Context.md)
- [#9 Placeholder and Initial Data in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%239_Placeholder_and_Initial_Data_in_React_Query.md.md)
- [#10 React Query as a State Manager](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2310_React_Query_as_a_State_Manager.md)
- [#11 React Query Error Handling](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2311_React_Query_Error_Handling.md)
- [#12 Mastering Mutations in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2312_Mastering_Mutations_in_React_Query.md)
- #13 Offline React Query (현재)
- [#14 React Query and Forms](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2314_React_Query_and_Forms.md)
- [#15 React Query FAQs](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2315_React_Query_FAQs.md)

**목차**

- [#13 Offline React Query](#13-offline-react-query)
  - [개요](#개요)
  - [Issues in v3](#issues-in-v3)
    - [1) no data in the cache](#1-no-data-in-the-cache)
    - [2) no retries](#2-no-retries)
    - [3) quries that don’t need the network](#3-quries-that-dont-need-the-network)
  - [The new NetworkMode](#the-new-networkmode)
    - [fetchStatus](#fetchstatus)
    - [always](#always)
    - [offlineFirst](#offlinefirst)
  - [What does all of this mean for me, exactly?](#what-does-all-of-this-mean-for-me-exactly)

## 개요

여러 번 말하지만 **React Query**는 비동기 상태 매니저다. 우리가 `resolve`된 또는 `reject`된 `Promise`를 라이브러리에게 넘겨주는 한 라이브러리는 이상적으로 동작할 것이다. `Promise`가 어디로부터 온 건지는 상관없다.

`Promise`를 만들어 내는 방법에는 여러 가지가 있지만 지금까지 가장 많은 **use-case**는 데이터 패칭이다. 데이터 패칭은 네트워크 연결을 필요로 하지만 가끔 특히나 네트워크 연결이 불안정할 수 있는 모바일 기기에서는 네트워크 연결 없이도 애플리케이션이 잘 동작해야 한다.

## Issues in v3

**React Query**는 오프라인 시나리오를 처리하는 데 적합하다. 왜냐하면 **React Query**는 캐싱 레이어를 제공하기 때문에 캐시 데이터가 저장되어 있는 한 네트워크 연결이 없을지라도 계속해서 사용이 가능할 수 있을 것이다.

이제 **React Query v3**에서 예상하지 못했던 몇몇의 엣지 케이스 시나리오를 알아보자. 설명을 위해서 **React Query** [공식 홈페이지에 있는 예제](https://tanstack.com/query/v4/docs/examples/react/basic?from=reactQueryV3&original=https://react-query-v3.tanstack.com/examples/basic)인 기본적인 포스팅 목록과 포스팅 상세 페이지를 사용할 것이다.

### 1) no data in the cache

앞서 언급했듯이 **React Query v3**는 캐시 데이터가 존재하는 한 정상적으로 동작한다. 문제가 생기는 엣지 케이스 시나리오는 다음과 같다.

- 네트워크 연결이 잘 되어 있는 상태에서 포스팅 목록 페이지로 이동한다.
- 인터넷 연결이 끊기고 포스팅을 클릭한다.

![출처: [https://tkdodo.eu/blog/offline-react-query](https://tkdodo.eu/blog/offline-react-query)](https://tkdodo.eu/blog/loading-forever.gif)

> 출처: [https://tkdodo.eu/blog/offline-react-query](https://tkdodo.eu/blog/offline-react-query)

네트워크가 다시 연결될 때까지 `query`의 상태가 로딩 상태로 유지된다. 또한 브라우저 개발자 도구에서 네트워크 요청이 실패한 것을 볼 수 있다. 이는 **React Query**가 항상 첫 번째 요청을 실패하고 네트워크 연결이 끊긴 경우에 재시도를 일시 정지하기 때문이다.

더군다나 **React Query DevTools**는 `query`가 데이터 패칭 중이라고 보여주지만 이는 사실이 아니다. `query`는 실제로 일시 정시된 상태지만 **React Query v3**에는 현재 상태를 나타내는 개념이 없다. - it's a hidden implementation detail.

### 2) no retries

비슷하게 만약 위 시나리오에서 재시도 기능을 모두 꺼둘 경우 `query`는 곧장 에러 상태가 되며 이를 막을 방법은 없다.

![출처: [https://tkdodo.eu/blog/offline-react-query](https://tkdodo.eu/blog/offline-react-query)](https://tkdodo.eu/blog/network-error.gif)

> 출처: [https://tkdodo.eu/blog/offline-react-query](https://tkdodo.eu/blog/offline-react-query)

네트워크 연결이 되어 있지 않은 경우 `query`를 일시 정지하기 위해서 재시도 기능이 필요한 이유는 무엇일까?

(Why do I need *retries* for my query to *pause* if I have no network connection 🤷‍♂️?)

### 3) quries that don’t need the network

동작하기 위해서 네트워크 연결이 필요하지 않은 `query`들(예를 들어, Web Worker에서 비용이 큰 비동기 작업하기 때문에)은 다른 이유로 실패할 경우 네트워크가 다시 연결될 때까지 일시 정지될 것이다. 또한 해당 `query`들은 네트워크 연결이 없는 경우 기능들이 완전히 사용 불가능하기 때문에 윈도우가 다시 포커징된다 해도 실행되지 않을 것이다.

요약하자면 여기엔 두 가지 주요한 이슈가 있다.

1. **React Query**는 네트워크 연결이 필요하지 않을 수도 있다. (Case 3)
2. **React Query**가 `query`를 실행하지 않아야 하는데 실행할 수도 있다. (Case 1, 2)

## The new NetworkMode

**React Query** 팀은 **v4**에서 새로운 `networkMode` 설정으로 이 문제를 전체적으로 해결하려고 했다. `networkMode` 설정으로 분명하게 `online query`와 `offline query`를 구별할 수 있다. 간단히 말해서, 이 설정으로 네트워크가 연결된 경우에만 `query`가 실행될 것이라고 가정한다.

그렇다면 네트워크 연결이 필요없는 `query`를 실행하려는 경우엔 어떻게 될까? `query`는 새로운 `paused` 상태가 될 것이다. 이 `paused` 상태는 `query`가 가질 수 있는 메인 상태들(`idle`, `loading`, `success` 또는 `error`)의 보조 상태다. 왜냐하면 네트워크 연결은 언제든지 끊어질 수 있기 때문이다.

이는 `success` 상태와 `paused` 상태를 모두 가질 수 있다는 것을 뜻한다. 예를 들어 데이터 패칭 한 번 성공적으로 이루어 졌지만 **Background refetch**는 `paused` 상태일 수 있다.

또는 `query`가 처음 `mount` 되는 경우 `loading` 상태와 `paused` 상태를 모두 가질 수도 있다.

### fetchStatus

**React Query**엔 `query`가 실행 중임을 나타내는 `isFetching` 플래그가 항상 있다. 새로운 `paused` 상태와 비슷하게 `query`는 `success` 상태와 `fetching` 상태를 모두 가질 수도 있고 `error` 상태와 `fetching` 상태를 모두 가질 수도 있다. **Background refetch**는 가능한 한 많은 상태를 제공한다. (👋 state machines)

`fetching`과 `pasued`는 상호 배타적, 즉 서로 겹치지 않기 때문에 이 두 개를 `useQuery`에서 반환되는 새로운 `fetchStatus`로 결합했다.

| 이름     | 설명                                                               |
| -------- | ------------------------------------------------------------------ |
| fetching | query가 정말 실행 중인 상태를 말한다. - 네트워크 요청 중           |
| paused   | query가 실행되지 않는다. - 네트워크가 다시 연결될 때까지 일시 정지 |
| idle     | query가 현재 실행 중인 상태가 아니다.                              |

일반적으로 `query`의 `status`는 데이터에 대한 정보를 알려준다. `success`는 데이터가 항상 있음을 의미하고 `loading`은 데이터가 아직 없음을 의미한다. I thought about renaming the *loading* state to *pending*, but alas, this probably would have been "too breaking". 😅

반면에 `fetchStatus`는 `queryFn`에 대한 정보(`queryFn`이 실행 중인지 아닌지)를 알려준다. `isFetching`과 `isPaused` 플래그는 `fetchStatus`로부터 파생된다.

위 사례(Case 1)가 **v4**에서는 어떻게 보이는지 살펴보자. **React Query DevTools**에 새로운 네트워크 모드 토글 버튼이 생겼다. 이 버튼으로 네트워크 실제로 끊어지지 않았음에도 네트워크 연결이 끊어진 것처럼 만들 수 있다. 이는 테스팅 목적으로 **React Query**가 네트워크가 없다고 믿게 만든다.

![출처: [https://tkdodo.eu/blog/offline-react-query](https://tkdodo.eu/blog/offline-react-query)](https://tkdodo.eu/blog/paused.gif)

> 출처: [https://tkdodo.eu/blog/offline-react-query](https://tkdodo.eu/blog/offline-react-query)

새로운 보라색 상태 뱃지를 통해 `query`의 상태(`paused`)를 명확하게 볼 수 있다. 또한 네트워크가 다시 연결되면 첫 번째 네트워크 요청이 이루어 진다.

> **v4**에서 바로 에러 상태가 되거나 영원히 로딩 상태가 되는 것이 아니라 네트워크가 연결된 순간 `query`를 다시 실행시켜 네트워크 요청을 정상적으로 발생시키는 것을 알 수 있다.

### always

`always` 모드에선 **React Query**는 네트워크 연결에 대해 전혀 신경쓰지 않는다. `query`는 언제나 실행될 것이고 `paused` 상태가 절대 되지 않는다. 이 기능은 **React Query**를 데이터 패칭이 아닌 다른 무언가로 사용하는 경우 가장 유용하다.

### offlineFirst

`offlineFirst` 모드는 **v3**에서 **React Query**가 동작하는 방식과 가장 비슷하다. 첫 번째 요청은 항상 발생하며 재시도는 일시 정지된다. 이 모드는 **React Query** 위에 브라우저 캐시와 같은 추가적인 캐싱 레이어를 사용하는 경우에 유용하다.

Github repo API 예시를 가져와보자. 해당 API는 아래와 같은 응답 헤더를 보낸다.

```
cache-control: public, max-age=60, s-maxage=60
```

위 헤더가 의미하는 바는 다음 60초 동안 해당 리소스를 다시 요청하면 브라우저 캐시에서 응답이 온다는 말이다. 여기서 멋진 점은 오프라인 상태에서도 작동한다는 것이다. Service workers, e.g. for [offline first PWAs](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Offline_Service_workers), work in a similar way by intercepting the network request and delivering cached responses if they are available.

위와 같은 동작들은 원하는 대로 동작하지 않을 수도 있는데 네트워크 연결이 끊어져 있는 경우 **React Query**가 기본 네트워크 옵션인 `online` 모드가 그러하듯 네트워크 요청을 발생시키지 않을 수도 있기 때문이다. 데이터 패칭 요청을 가로채기 위해선 이러한 동작들은 반드시 발생해야 한다. 따라서, 만약 추가적인 캐싱 레이어를 갖기 원한다면 반드시 `offlineFirst` `networkMode`를 하자.

만약 첫 번째 요청 발생해서 캐시를 히트한다면 `query`는 `success` 상태로 변하게 될 것이고 우리는 곧 데이터를 가져올 수 있을 것이다. 캐시를 히트하지 못한 경우 네트워크 에러가 발생할 가능성이 높다. 따라서 **React Query**는 재시도를 일시 정지하고 `query`를 `paused` 상태로 변경할 것이다. It's the best of both worlds. 🙌

## What does all of this mean for me, exactly?

이러한 작업들을 원하지 않는다면 아무런 의미도 없다. 새로운 `fetchStatus`는 무시하고 `isLoading`만 계속해서 체크할 수도 있다. **React Query**는 이전과 같이 동작할 것이다. (위 Case 2의 경우 네트워크 에러를 볼 수 없을 것이기에 더 잘 동작할 것이다)

그러나 만약 애플리케이션이 네트워크 연결이 없는 상황에서도 대처할 수 있도록 만드는 것이 우선 순위라면 `fetchStatus`를 꺼내서 이에 따라 동작하게 만드는 방법도 있다.

What you do with that new status is up to you. I'm excited to see which ux people will build on top of this. 🚀
