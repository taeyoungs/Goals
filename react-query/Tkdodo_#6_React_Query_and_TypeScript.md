# #6 React Query and TypeScript

**Series**

- [#1 Practical React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%231_Practical_React_Query.md)
- [#2 React Query Data Transformations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%232_React_Query_Data_Transformations.md)
- [#3 React Query Render Optimizations](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%233_React_Query_Render_Optimizations.md)
- [#4 Status Checks in React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%234_Status_Checks_in_React_Query.md)
- #6 React Query and TypeScript (현재)

**목차**

- [#6 React Query and TypeScript](#6-react-query-and-typescript)
    - [개요](#개요)
  - [Generics](#generics)
  - [The four Generics](#the-four-generics)
  - [Type Inference (타입 추론)](#type-inference-타입-추론)
  - [Partial Type Argument Inference](#partial-type-argument-inference)
  - [Infer all the things](#infer-all-the-things)
  - [What about error?](#what-about-error)
  - [Type Narrowing](#type-narrowing)
  - [Type safety with the enabled option](#type-safety-with-the-enabled-option)
    - [non-null assertion operator란?](#non-null-assertion-operator란)
  - [Optimistic Updates](#optimistic-updates)
  - [useInfiniteQuery](#useinfinitequery)
  - [Typing the default query function](#typing-the-default-query-function)

> 원본 글  
> https://tkdodo.eu/blog/react-query-and-type-script

### 개요

타입스크립트는 이제 프론트엔드 커뮤니티에서 기본처럼 되어가고 있다. 많은 개발자들이 라이브러리가 타입스크립트로 작성되기를 바라며 자바스크립트로 작성되어 있다고 해도 최소한 잘 정리된 타입 정의를 제공해주길 바라고 있다. 필자에겐 만약 라이브러리가 타입스크립트로 작성되었다면, 타입 정의가 최고의 문서가 된다. 구현을 직접 반영하기 때문에 이러한 타입 정의는 결코 틀리지 않는다. 필자는 자주 API 문서를 읽기 전에 타입 정의부터 살펴보곤 한다.

**React Query**는 초기에 자바스크립트로 작성되었지만 v2에서 타입스크립트로 다시 작성되었다. This means that right now, there is very good support for TypeScript consumers.

그러나 **React Query**가 동적이고 unopinionated함에 따라 타입스크립트와 함께 사용하는데는 몇 가지 "문제점"이 있다. 이제 이러한 문제점을 하나하나 살펴보도록 하겠다.

## Generics

**React Query**는 제네릭을 많이 사용한다. 이는 라이브러리가 실제로 데이터를 가져오지 않기 때문에 필요하기도 하고 API 반환값으로 오는 데이터의 타입에 대해 라이브러리가 알 수 없기 때문이다.

**React Query** 공식 문서의 [타입스크립트 섹션](https://react-query.tanstack.com/typescript)은 광범위하지 않으며 이는 **useQuery**를 호출할 때 제네릭을 통하여 가져오는 데이터의 타입에 대해서 명시적으로 지정해야 한다는 것을 말해준다.

```tsx
function useGroups() {
  return useQuery<Group[], Error>('groups', fetchGroups);
}
```

시간이 지남에 따라, **React Query**는 **useQuery** 훅에 제네릭을 계속해서 추가했는데(현재 4개의 제네릭이 존재) 이는 **useQuery** 훅에 더 많은 기능이 추가되었기 때문이다. 위 코드가 동작할 때 우리의 커스텀 훅은 `data` 속성은 `Group[] | undefined` 타입일 것이고 `error` 속성의 타입은 `Error | undefined`이 될 것이라고 말해줄 것이다. 그러나 두 개의 제네릭이 추가적으로 필요한 상황이라면 이는 다르게 동작할 것이다.

## The four Generics

다음은 **useQuery** 훅에 현재 정의된 타입 정의이다.

```tsx
export function useQuery<
  TQueryFnData = unknown,
  TError = unknown,
  TData = TQueryFnData,
  TQueryKey extends QueryKey = QueryKey
>
```

- `TQueryFnData`: **queryFn**의 반환 타입. 위 예제에서는 `Group[]`이 이에 해당된다.
- `TError`: **queryFn**에서 발생할 수 있는 에러 타입. 위 예제에서는 **Error**가 이에 해당된다.
- `TData`: **Data** 속성이 궁극적으로 갖게 될 타입. **select** 옵션을 활용할 때만 해당 **Generic**을 사용하는 것에 의미가 생긴다. **select** 옵션을 사용한다라는 것은 **queryFn**이 반환하는 것과 데이터 속성이 달라짐을 의미하기 때문이다.
- `TQueryKey`: **queryKey**의 타입이며 **queryFn**에 **queryKey**를 전달하는 상황에 의미가 있는 **Generic**이다.

위에서 볼 수 있듯이 모든 **Generic**은 기본값을 가지며 우리가 해당 **Generic**에 타입을 제공하지 않는 경우 **TypeScript**는 해당 기본값을 사용할 것이다. 이는 **JavaScript**에서 파라미터에 기본값을 설정해주는 것과 거의 비슷하다.

```tsx
function multiply(a, b = 2) {
  return a * b;
}

multiply(10); // 20
multiply(10, 3); // 30
```

## Type Inference (타입 추론)

**TypeScript**는 스스로 어떤 타입이 되어야 하는지 추론(또는 파악)하게끔 만들어 줄 때 가장 잘 동작한다. 코드를 더 쉽게 작성할 수 있을 뿐만 아니라(모든 타입을 명시해줘야 하는 건 아니기 때문에) 더 읽기 쉬워진다. 많은 경우에 코드의 모양새는 **JavaScript**와 비슷하게 작성할 수 있다. 타입 추론의 예제들은 다음과 같다.

```tsx
const num = Math.random() + 5; // number

// 🚀 both greeting and the result of greet will be string
function greet(greeting = 'ciao') {
  return `${greeting}, ${getName()}`;
}
```

**Generic**의 경우, 일반적으로 코드의 사용하는 방식에서 타입을 유추할 수 있으며 이러한 점은 상당히 유용하다. 우리는 **Generic**을 명시적으로 제공할 수도 있지만 대부분의 경우 필요하지 않을 수 있다.

```tsx
function identity<T>(value: T): T {
  return value;
}

// 🚨 no need to provide the generic
let result = identity<number>(23);

// 🟡 or to annotate the result
let result: number = identity(23);

// 😎 infers correctly to `string`
let result = identity('react-query');
```

## Partial Type Argument Inference

> 부분 타입 추론 기능은 아직 TypeScript에 정식으로 구현되지 않았다. ⇒ [링크](https://github.com/microsoft/TypeScript/issues/26242)

이 말은 무엇이냐 ! 만약 하나의 **Generic**을 제공하고 싶을 경우 나머지 **Generic**도 모두 제공해야만 함을 의미한다. **React Query**에서는 **Generic**들에 대해서 기본값을 제공하고 있기 때문에 이러한 나머지 **Generic**을 사용하고 있다는 사실을 모를 수 있다. 이렇게 때문에 에러 메시지의 내용이 당황스러울 수 있다. 실제로 역효과가 나는 예시를 한번 살펴보자.

```tsx
function useGroupCount() {
  return useQuery<Group[], Error>('groups', fetchGroups, {
    select: (groups) => groups.length,
    // 🚨 Type '(groups: Group[]) => number' is not assignable to type
    // '(data: Group[]) => Group[]'.
    // Type 'number' is not assignable to type 'Group[]'.ts(2322)
  });
}
```

위 에러는 3번째 **Generic**를 제공하지 않은 상황에서 `select` 옵션을 사용하여 `number`를 반환하고 있기 때문에 발생하고 있다. 3번째 **Generic**을 제공하고 있지 않았기 때문에 **Data**의 타입은 기본값 즉, **queryFn**의 반환 타입인 `Group[]`가 될 것이기 때문이다. 이를 쉽게 고칠 수 있는 방법 중 하나는 3번째 **Generic**을 제공하는 것이다.

```tsx
function useGroupCount() {
  // fixed it
  return useQuery<Group[], Error, number>('groups', fetchGroups, {
    select: (groups) => groups.length,
  });
}
```

**TypeScript**에 부분 타입 추론 기능이 추가되지 않는 한 우리는 우리가 가진 것만으로 이러한 상황을 해결해야 한다.

그렇다면 대안은 무엇이 있을까?

## Infer all the things

어떠한 **Generic**도 전달하지 않고 **TypeScript**에게 우리가 하고자 하는 것들을 파악하도록 해보자. 이를 가능하게 하기 위해서 우리는 **queryFn**이 반환 타입을 잘 갖게끔 만들어줄 필요가 있다. 물론, 만약 명확한 반환 타입 없이 함수를 인라인으로 작성할 경우 `any` 타입을 갖게 될 것이다. 이는 **axios** 또는 **fetch**가 그렇게 줄 것이기 때문이다.

```tsx
function useGroups() {
  // 🚨 data will be `any` here
  return useQuery('groups', () => axios.get('groups').then((response) => response.data));
}
```

만약 **API layer**를 **query**와 분리된 상태로 유지하고 싶다면 어쨋든 `any`를 피하기 위해 타입 정의를 추가해줄 필요가 있다. 그렇게만 한다면 **React Query**가 나머지 타입에 대해선 추론을 할 수 있다.

```tsx
function fetchGroups(): Promise<Group[]> {
  return axios.get('groups').then((response) => response.data);
}

// ✅ data will be `Group[] | undefined` here
function useGroups() {
  return useQuery('groups', fetchGroups);
}

// ✅ data will be `number | undefined` here
function useGroupCount() {
  return useQuery('groups', fetchGroups, {
    select: (groups) => groups.length,
  });
}
```

이 방법의 장점은

- 더 이상 특정 **Generic**들을 명시적으로 전달할 필요가 없다.
- 3번째 (`select`)와 4번째 (`QueryKey`) **Generic**이 필요한 경우에도 잘 동작한다.
- 더 많은 Generic이 추가되어도 계속해서 동작한다.
- 코드가 덜 혼란스럽고 좀 더 **JavaScript**스럽다.

## What about error?

그럼 에러는 어떻게 ? 라고 생각했을 것이다.

> 진짜다. Tkdodo 그는 신인가? 🤣

기본적으로 어떠한 **Generic**도 없다면 `error`는 `unknown`으로 추론될 것이다. 왜 `Error` 타입이 아닌걸까? 이는 어찌보면 에러같이 보일 수도 있을 것이다. 하지만 이는 에러가 아니며 의도된 것이다. 왜나하면 **JavaScript**에서는 무엇이든 **throw** 할 수 있기 때문이다. **throw** 하는 것이 `Error` 타입이어야만 하는 것이 아니다.

```tsx
throw 5;
throw undefined;
throw Symbol('foo');
```

**React Query**는 `Promise`를 반환하는 함수를 담당하지 않기 때문에 어떤 유형의 오류가 발생할지 알 수도 없다. 그래서 `unknown`으로 추론되는 것이 맞다. 앞으로 언젠가 여러 **Generic**이 있는 함수를 호출할 때 **TypeScript**에서 일부 **Generic**에 대한 생략 허용한다면, 이를 더 잘 핸들링할 수 있을 지 모른다. (관련 [이슈](https://github.com/microsoft/TypeScript/issues/10571)) 하지만 아직은 지원하지 않는 기능이기 때문에 만약 우리가 에러를 가지고 작업을 하고 싶고 **Generic**을 전달하는 것에 의존하고 싶지 않다면 `instanceof`를 통해 타입을 좁힐 수는 있다.

```tsx
const groups = useGroups();

if (groups.error) {
  // 🚨 this doesn't work because: Object is of type 'unknown'.ts(2571)
  return <div>An error occuerred: {groups.error.message}</div>;
}

// ✅ the instanceOf check narrows to type `Error`
if (groups.error instanceof Error) {
  return <div>An error occuered: {groups.error.message}</div>;
}
```

어쨋든 우리는 에러 여부를 검사할 수 있는 무언가가 필요한 상황이기 때문에 `instanceof` 검사는 나쁘지 않은 생각이다. 그리고 이를 통해 런타임에서 `error` 객체가 `message` 속성을 가지고 있다는 것을 확실히 할 수 있다. 이는 **TypeScript**가 4.4 버전 릴리즈를 위해 계획한 것과도 일치한다. v4.4 부터는 새로운 컴파일러 플래그 `useUnknownInCatchVariables`를 등장하는데 이 플래그를 사용할 경우 위와 같은 상황에서 **TypeScript**가 변수를 추론할 때 `any` 대신 `unknown`이 됩니다.

## Type Narrowing

필자는 **React Query**를 사용할 때 드물게 구조 분해 할당을 사용한다. 우선, `data`나 `error` 같은 네이밍은 보편적으로 사용되기 때문에 아마 이러한 네이밍을 변경하게 될 것이다. 전체 객체를 유지하면(구조 분해 할당을 하지 않으면) 데이터의 타입 또는 오류가 발생한 곳에 대한 정보를 유지할 수 있을 것이다. 또한 상태 필드 또는 `boolean` 상태값 중 하나를 사용할 때 **TypeScript**가 타입을 좁히는 데 도움이 된다.

```tsx
const { data, isSuccess } = useGroups();

if (isSuccess) {
  // 🚨 data will still be `Group[] | undefined` here
}

const groupsQuery = useGroups();

if (groupsQuery.isSuccess) {
  // ✅ groupsQuery.data will now be `Group[]`
}
```

이 상황에서 React Query가 해줄 수 있는 건 없다. **TypeScript**는 원래 이렇게 동작하기 때문이다.

> TypeScript does refinement on the types of individual symbols. Once you split them apart, it can't keep track of the relationship any more. Doing this in general would be computationally hard. It can also be hard for people.

> **TypeScript** v4.6부터 위 예제는 더 이상 이슈가 아니다. 구조 분해 할당에도 **TypeScript**는 정상적으로 타입 추론을 이어간다.

## Type safety with the enabled option

필자는 앞선 포스팅 시리즈에서도 언급한 바가 있듯이 `enabled` 옵션을 정말 좋아한다. 그러나 만약 `dependent query`를 위해 `enabled` 옵션을 사용하고 싶거나 특정 파라미터들이 아직 정의되지 않은 상태에서 `query`를 비활성화하고 싶은 경우 `enabled` 옵션의 타입은 까다로워질 수 있다.

```tsx
function fetchGroup(id: number): Promise<Group> {
  return axios.get(`group/${id}`).then((response) => response.data)
}

function useGroup(id: number | undefined) {
  return useQuery(['group', id], () => fetchGroup(id), {
    enabled: Boolean(id),
  // 🚨 Argument of type 'number | undefined' is not assignable to parameter of type 'number'.
  //  Type 'undefined' is not assignable to type 'number'.ts(2345)
}}
```

위 상황만 놓고 봤을 때 **TypeScript**는 정상적으로 타입 추론을 하고 있다. `id`는 `undefined`일 수 있기 때문이다. `enabled` 옵션에게 타입을 좁혀주는 기능은 존재하지 않는다. `enabled` 옵션은 우회하는 방법도 있긴 하다. 예를 들어, **useQuery**에서 반환하는 속성 중 `refetch` 메서드를 호출하는 것이다. 위 경우에는 `id`는 아마 `undefined`가 맞을 것이다.

필자가 찾은 최적의 방법은 다음과 같다. 만약 **non-null assertion operator**를 좋아하지 않는다면 `id`가 `undefined`될 수 있다는 것을 받아들이고 `queryFn`에서 `Promise`를 `reject` 해야한다. 이는 조금의 중복이 발생하긴 해도 명시적이고 안전하다.

---

### non-null assertion operator란?

A new `!` post-fix expression operator may be used to assert that its operand is non-null and non-undefined in contexts where the type checker is unable to conclude that fact. Specifically, the operation `x!` produces a value of the type of `x` with `null` and `undefined` excluded. Similar to type assertions of the forms `<T>x` and `x as T`, the `!` non-null assertion operator is simply removed in the emitted JavaScript code.

```tsx
// Compiled with --strictNullChecks
function validateEntity(e?: Entity) {
  // Throw exception if e is null or invalid entity
}
function processEntity(e?: Entity) {
  validateEntity(e);
  let s = e!.name; // Assert that e is non-null and access name
}
```

---

```tsx
function fetchGroup(id: number | undefined): Promise<Group> {
  // ✅ check id at runtime because it can be `undefined`
  return typeof id === 'undefined'
    ? Promise.reject(new Error('Invalid id'))
    : axios.get(`group/${id}`).then((response) => response.data);
}

function useGroup(id: number | undefined) {
  return useQuery(['group', id], () => fetchGroup(id), {
    enabled: Boolean(id),
  });
}
```

## Optimistic Updates

> **TypeScript**에서 낙관적 업데이트를 가져가는 건 쉽지 않기 때문에, 이를 다루는 포괄적인 [예제](https://tanstack.com/query/v4/docs/examples/react/optimistic-updates?from=reactQueryV3&original=https://react-query-v3.tanstack.com/examples/optimistic-updates)를 문서에 추가하기로 결정

**TypeScript** v4.7에서 객체와 메서드에 대한 타입 추론 기능 향상이 이루어졌기 때문에 해당 사항은 더 이상 이슈가 아니며 추가적인 작업 없이도 낙관적 업데이트에 대한 정확한 타입 추론이 가능해진 상황이다.

## useInfiniteQuery

대부분의 경우 **useInfiniteQuery**를 작성하는 것은 **useQuery**를 작성할 때와 별 차이가 없다. 한 가지 눈에 띄는 문제는 **queryFn**에 전달되는 `pageParam` 값이 `any`로 추론된다는 것이다. 라이브러리에서 확실하게 개선될 수 있지만 `any`인 한 명시적으로 타입을 정의하는 것이 가장 좋다.

```tsx
type GroupResponse = { next?: number, groups: Group[] }

const queryInfo = useInfiniteQuery(
  'groups',
  // ✋ explicitly type pageParam to override `any`
  {{ pageParam = 0 }: { pageParam: GroupResponse['next']) => fetchGroups(groups, pageParam),
  {
    getNextPageParam: (lastGroup) => lastGroup.next,
  }
)
```

만약 `fetchGroups`가 `GroupResponse`를 반환한다면 `lastGroup`은 깔끔하게 타입 추론이 될 것이고 우리는 `pageParam`에 명시해놓은 타입과 동일한 타입을 사용할 수 있다.

## Typing the default query function

필자 개인적으로는 `defaultQueryFn` 사용하진 않지만, 많은 사람들이 사용하고 있다는 것을 안다. 이 방법은 전달된 **queryKey**를 활용하여 request URL을 직접 작성하는 깔끔한 방법이다. 만약 `queryClient`를 생성할 때 함수를 인라인으로 생성하면 전달된 **QueryFunctionContext**의 타입도 자동으로 유추된다. **TypeScript**는 인라인으로 코드를 작성할 때 더 잘 동작한다.

```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: async ({ queryKey: [url] }) => {
        const { data } = await axios.get(`${baseURL}/${url}`);

        return data;
      },
    },
  },
});
```

위 코드는 동작하지만 `url`의 타입은 `unknown`으로 추론된다. 왜냐하면 전체 **queryKey**는 `unknown` 배열이기 때문이다. **useQuery**를 호출할 때 **queryKeys**가 어떻게 구성될지 전혀 보장되지 않으므로 `queryClient`를 생성할 때 **React Query**가 할 수 있는 일은 매우 다양하다. 이는 매우 동적인 기능이다. 이게 마냥 나쁜 기능이 아닌 것이 동적인 기능이기 때문에 방어적으로 작업해야 하고 런타임 검사로 타입을 좁혀 작업해야 하기 때문이다. 예를 들어

```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      queryFn: async ({ queryKey: [url] }) => {
        // ✅ narrow the type of url to string so that we can work with it
        if (typeof url === 'string') {
          const { data } = await axios.get(`${baseUrl}/${url.toLowerCase()}`);

          return data;
        }
        throw new Error('Invalid QueryKey');
      },
    },
  },
});
```

필자는 이러한 점이 `unknown`이 `any`와 비교했을 때 더 훌륭한지(그리고 덜 사용되는지)를 잘 보여준다고 생각한다.
