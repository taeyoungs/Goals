# getStaticPaths

**목차**

- [getStaticPaths](#getstaticpaths)
  - [개요](#개요)
  - [When should I use getStaticPaths?](#when-should-i-use-getstaticpaths)
  - [When does getStaticPaths run](#when-does-getstaticpaths-run)
  - [Where can I use getStaticPaths](#where-can-i-use-getstaticpaths)
  - [Runs on every request in development](#runs-on-every-request-in-development)

## 개요

만약 페이지가 `동적 라우팅`과 `getStaticPaths`를 사용하고 있다면 정적으로 생성될 경로들의 목록을 정의할 필요가 있다.

동적 라우팅을 사용하고 있는 페이지에서 `getStaticPaths`(Static Site Generation)라고 불리는 함수를 `export` 할 때 `Next.js`는 `getStaticPaths`에 의해 지정된 모든 경로를 정적으로 `pre-rendering` 할 것이다.

```jsx
export async function getStaticPaths() {
  return {
    paths: [
      { params: { ... } }
    ],
    fallback: true // false or 'blocking'
  }
}
```

`getStaticPaths`는 반드시 `getStaticProps`와 같이 사용해야 한다. 그리고 `getServerSideProps`와는 같이 사용할 수 없다.

## When should I use getStaticPaths?

동적 라우팅을 사용하고 있는 페이지를 정적으로 `pre-rendering` 하고 싶을 때 `getStaticPaths`를 사용해야 한다. 그리고 추가적으로

- 데이터를 **headless CMS**로부터 가져오는 경우
- 데이터를 데이터베이스로부터 가져오는 경우
- 데이터를 파일시스템에서 가져오는 경우
- 사용자별 데이터가 아닌 상태에서 캐리 처리된 경우
- 페이지가 **SEO**를 위해 **pre-rendering** 되어야만 하고 매우 빨라야 하는 경우
  `getStaticProps`는 `HTML`와 `JSON` 파일을 생성하며 이 둘은 모두 `CDN`에 의해 캐시 처리될 수 있다.

## When does getStaticPaths run

`getStaticPaths`는 오직 `production` 빌드 시에만 실행될 것이며 런타임에는 호출되지 않을 것이다. `getStaticPaths` 또한 `getServerSideProps`와 같이 클라이언트 사이드 번들 파일 내에서는 제거된 상태가 된다.

- `getStaticProps`는 빌드 중에 반환된 모든 경로에 대해 실행된다.
- `getStaticProps`는 `fallback: true`를 사용할 경우 백그라운드 상에서 실행된다.
- `getStaticProps`는 `fallback: blocking`을 사용할 경우 첫 렌더링 전에 호출된다.

## Where can I use getStaticPaths

`getStaticPaths`는 오직 `getStaticProps`도 사용하고 있는 동적 경로에서만 `export` 될 수 있다. 페이지 컴포넌트가 아닌 파일에서 `export` 할 수 없다.

`getStaticPaths`는 독립된 함수 형태로 `export` 해야만 한다. 페이지 컴포넌트 내에 속성으로 추가할 경우 동작하지 않을 것이다.

## Runs on every request in development

개발 환경에서 `getStaticPaths`는 매 요청마다 호출된다.
