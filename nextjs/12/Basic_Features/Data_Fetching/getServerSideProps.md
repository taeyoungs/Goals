# getServerSideProps

**목차**

- [getServerSideProps](#getserversideprops)
  - [개요](#개요)
  - [When does getServerSideProps run](#when-does-getserversideprops-run)
  - [When should I use getServerSideProps](#when-should-i-use-getserversideprops)
    - [getServerSideProps or API Routes](#getserversideprops-or-api-routes)
  - [Fetching data on the client side](#fetching-data-on-the-client-side)
  - [Using getServerSideProps to fetch data at request time](#using-getserversideprops-to-fetch-data-at-request-time)
  - [Caching with Server-Side Rendering (SSR)](#caching-with-server-side-rendering-ssr)
  - [Does getServerSideProps render an error page](#does-getserversideprops-render-an-error-page)

## 개요

만약 우리가 페이지에서 `getServerSideProps`(Server-side Rendering)라고 불리는 함수를 `export` 한다면, `Next.js`는 `getServerSideProps`에서 반환되는 데이터를 이용하여 매 요청마다 페이지를 `pre-rendering`할 것이다.

```jsx
export async function getServerSideProps(context) {
  return {
    props: {}, // will be passed to the page component as props
  };
}
```

## When does getServerSideProps run

`getServerSideRrops`는 오직 **서버 사이드**에서만 실행되며 절대 브라우저에서 실행되지 않는다. 만약 페이지가 `getServerSideProps`를 사용한다면

- 우리가 이 페이지를 직접 요청했을 때, `getServerSideProps`는 요청 시간에 실행되며 해당 페이지는 `getServerSideProps`에서 반환된 `props`와 함께 `pre-rendering` 될 것이다.
- 우리가 이 페이지를 `next/link` 또는 `next/router`를 통해 클라이언트 사이드 페이지 전환을 통하여 요청했을 경우 `Next.js`는 `getServerSideProps`를 실행하는 서버에 API 요청을 보낸다.

`getServerSideProps`는 페이지를 렌더링하는데 사용될 `JSON`을 반환할 것이다. 이러한 모든 작업은 `Next.js`에 의해 자동으로 관리될 것이기 때문에 우리는 `getServerSideProps`가 정의되어 있는 한 추가적인 작업을 할 필요가 없다.

[next-code-elimination tool](https://next-code-elimination.vercel.app/)을 사용해보면 클라이언트 사이드 번들 파일에서는 Next.js가 해당 함수들을 모두 제거해서 함수들이 존재하지 않는다는 것을 확인할 수 있다.
`getServerSideProps`는 오직 페이지 컴포넌트 파일 내에서 `export` 할 수 있다. 따라서, 페이지 컴포넌트 파일이 아닌 곳에서는 이를 `export` 할 수 없다.

주의해야할 점은 `getServerSideProps` 함수는 반드시 독립된 함수 형태로 `export` 되어야 한다는 것이다. 페이지 컴포넌트 내 속성으로 추가한다면 `getServerSideProps`는 동작하지 않을 것이다.

`getServerSideProps`의 파라미터와 `props`에 대한 자세한 사항은 다음 링크를 참조하자.

[Data Fetching: getServerSideProps | Next.js](https://nextjs.org/docs/api-reference/data-fetching/get-server-side-props)

## When should I use getServerSideProps

우리는 요청 시에 패칭한 데이터를 가지고 렌더링할 필요가 있는 페이지의 경우에만 `getServerSideProps`를 사용해야 한다. 이는 데이터의 특성 또는 요청의 속성들 때문일 수 있다. (예를 들어 request header 속성 중 `authorization` 또는 지리 정보 데이터) `getServerSideProps`를 사용하는 페이지는 매 요청마다 서버 사이드 렌더링될 것이며 오직 request header 속성 중에서 `cache-control` 관련 속성들이 구성이 되어 있을 때만 캐싱 처리될 것이다.

만약 요청하는 동안 데이터를 렌더링할 필요가 없다면, `Client-side Rendering` 또는 `getStaticProps`의 사용을 고려하자.

### getServerSideProps or API Routes

서버에서 데이터를 가져오고 싶을 때 `API Routes`에 도달하고 `getServerSideProps`에서 해당 `API Routes`를 호출하고 싶을 수 있다. 이는 불필요하고 비효율적인 접근법이다. `getServerSideProps`와 `API Routes` 모두 서버에서 돌아가고 있기 때문에 추가 요청을 만들어 낼 것이다.

다음 예를 같이 살펴보자. CMS로부터 특정 데이터를 패칭해오는데 API Route가 사용되고 있는 상황이다. 그러면 해당 `API Route`는 `getServerSideProps`로부터 직접 호출될 것이다. 이는 추가적인 요청을 만들어 내며 퍼포먼스 성능을 저하시킨다. 이렇게 하는 대신에 `API Route` 내에서 실행되는 로직을 `getServerSideProps` 내로 가져오자. 이러면 `getServerSideProps` 내에서 CMS, 데이터베이스 또는 API를 직접 호출할 수 있다.

## Fetching data on the client side

만약 우리의 페이지가 빈번하게 업데이트 되는 데이터를 포함하고 있다면 데이터를 `pre-rendering`할 필요가 없다. 차라리 클라이언트 사이드에서 데이터를 패칭하는 것이 낫다. 이에 대한 예는 사용자별 데이터이다.

- 먼저, 데이터가 없는 페이지를 즉시 보여준다. 페이지의 일부분은 `Static Generation`을 사용하여 `pre-rendering` 될 수 있으며 누락된 데이터에 대한 로딩 상태를 보여줄 수도 있다.
- 그리고 나서 클라이언트에서 데이터를 패칭하고 준비 됐을 때 해당 데이터를 보여준다.

이러한 방식은 예를 들어 유저 대시보드 페이지에서 잘 동작한다. 왜냐하면 대시보드는 `private`하며 사용자별로 페이지가 나뉘어져 있고 `SEO`와는 관련이 없기 대문에 페이지를 `pre-rendering`할 필요가 없다. 데이터는 빈번하게 업데이트되므로 요청 시점에 데이터를 패칭해야 한다.

## Using getServerSideProps to fetch data at request time

다음 예제는 요청 시점에 데이터를 패칭하는 방법과 결과를 `pre-rendering`하는 방법에 대한 것이다.

```jsx
function PAge({ data }) {
  // Render data...
}

// This gets called on every request
export async function getServerSideProps() {
  // Fetch data from external API
  const res = await fetch(`https://.../data`);
  const data = await res.json();

  // Pass data to the page via props
  return { props: { data } }
}

export deafult Page;
```

## Caching with Server-Side Rendering (SSR)

우리는 `getServerSideProps` 내에 header 속성 중 `Cache-Control`을 사용하여 동적인 응답을 캐싱할 수 있다. 예를 들어 다음은 `stale-while-revalidate` 사용 예제이다.

```jsx
// This value is considered fresh for ten seconds (s-maxage=10).
// If a request is repeated within the next 10 seconds, the previously
// cached value will still be fresh. If the request is repeated before 59 seconds,
// the cached value will be stale but still render (stale-while-revalidate=59)
//
// In the background, a revalidation request will be made to populate the cache
// with a fresh value. If you refresh the page, you will see the new value.
export async function getServerSideProps({ req, res }) {
  res.setHeader('Cache-Control', 'public, s-maxage=10, stale-while-revalidate=59');

  return {
    props: {},
  };
}
```

## Does getServerSideProps render an error page

만약 `getServerSideProps` 내에서 에러가 던져진다면 `pages/500.js` 파일이 보여질 것이다. 개발 모드인 동안은 해당 페이지는 사용되지 않을 것이며 대신에 dev overlay가 보여질 것이다.
