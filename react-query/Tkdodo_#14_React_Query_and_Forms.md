# #14 React Query and Forms

> 원본 글  
> https://tkdodo.eu/blog/react-query-and-forms

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
- [#13 Offline React Query](https://github.com/taeyoungs/Goals/blob/main/react-query/Tkdodo_%2313_Offline_React_Query.md)
- #14 React Query and Forms (현재)

**목차**

- [#14 React Query and Forms](#14-react-query-and-forms)
  - [개요](#개요)
  - [Server State vs. Client State](#server-state-vs-client-state)
  - [The simple approach](#the-simple-approach)
    - [Data might be undefined](#data-might-be-undefined)
    - [No background updates](#no-background-updates)
  - [Keeping background updates on](#keeping-background-updates-on)
    - [You need controlled fields](#you-need-controlled-fields)
    - [Deriving state might be difficult](#deriving-state-might-be-difficult)
  - [Tips and Tricks](#tips-and-tricks)
    - [Double submit prevention](#double-submit-prevention)
    - [Invalidate and reset after mutation](#invalidate-and-reset-after-mutation)

## 개요

> **들어가기에 앞서**: 현재 포스팅에서는 **React Hook Form**이라는 라이브러리를 쓴다는 점을 알아두자. **React Hook Form** 라이브러리를 사용하는 이유는 그저 필자가 생각하기에 좋은 라이브러리라고 생각하기 때문이다. 그렇다고 해서 포스팅에서 보여줄 패턴이 React Hook Form에서만 동작한다는 것은 아니다. 컨셉 자체는 어떤 폼 라이브러리에서도 사용 가능하며 폼 라이브러리 없이도 사용 가능하다.

**Form**은 데이터를 업데이트하기 위한 주요 수단으로써 많은 웹 애플리케이션에서 아주 중요한 부분이다. 우리는 **React Query**를 데이터 패칭을 위해 사용할 뿐만 아니라 데이터를 변경하기 위해서도 사용한다. 따라서, 우리가 사랑하는 비동기 상태 관리자와 **Form**을 어떻게든 통합해야 한다.

좋은 소식은 현실적으로 **Form**에 대해서는 따로 해줄 것이 없다는 것이다. **Form**은 여전히 우리가 특정 데이터를 보여주기 위해 렌더링하는 **HTML** 요소들을 모아둔 것일 뿐이다. 그러나 특정 데이터를 우리가 변경하려고 할 때 서버 상태와 클라이언트 상태 간의 경계가 흐려지면서 이러한 변경이 복잡해질 수 있다.

## Server State vs. Client State

요약하자면 **서버 상태**는 우리(클라이언트)가 가지고 있지 않은 상태고 이 상태들은 대개 비동기이며 우리가 마지막으로 패칭해왔을 때의 데이터 형태 Snapshot만을 볼 수 있다.

**클라이언트 상태**는 프론트엔드가 전적으로 관리하고 있는 상태고 대개 동기적이며 우리는 항상 클라이언트 상태의 정확한 값을 알고 있다.

사람들의 목록을 보여줄 때 이러한 목록은 의심할 여지 없이 서버 상태다. 그러나 몇 가지 데이터를 업데이트할 목적으로 사용자 정보를 클릭했을 때 해당 사용자의 상세 정보를 **Form**에 보여주면 어떻게 될까? 그러면 이제 서버 상태는 클라이언트 상태가 된걸까? 이건 하이브리드 상태일까?

## The simple approach

필자는 이미 `props`에서 `state`로 또는 **React Query**의 상태값을 복사하여 로컬 상태로 집어 넣는 것과 같이 하나의 상태 관리자에서 다른 상태 관리자로 상태값을 복사하는 방식을 좋아하지 않는다고 언급한 적이 있다.

필자는 **Form**이 위 방식의 예외가 될 수 있다고 생각한다. 만약 이 방식을 의도적으로 사용하고 있고 이 방식의 **tradeoff**에 대해서 알고 있다면 말이다(결국 모든 게 **tradeoff**다). 사용자 Form을 렌더링할 때 우리는 서버 상태를 **Form**의 초기 상태로만 취급받길 원할 것이다. `firstName`과 `lastName` 정보를 패칭해 가져오고 이 정보들을 **Form** 상태값에 집어 넣은 뒤 이를 사용자가 업데이트 하도록 놔두는 것처럼 말이다.

다음 예제를 살펴보자.

```jsx
function PersonDetail({ id }) {
  const { data } = useQuery(['person', id], () => fetchPreson(id));
  const { register, handleSubmit } = useForm();
  const { mutate } = useMutation((values) => updatePerson(values));

  if (data) {
    return (
      <form onSubmit={handleSubmit(mutate)}>
        <div>
          <label htmlFor="firstName">First Name</label>
          <input {...register('firstName')} defaultValue={data.firstName} />
        </div>
        <div>
          <label htmlFor="lastName">Last Name</label>
          <input {...register('lastName')} defaultValue={data.lastName} />
        </div>
        <input type="submit" />
      </form>
    );
  }

  return 'loading...';
}
```

위 예제는 매우 잘 동작한다. 그렇다면 위 방식을 사용함으로써 감수해야하는 **tradeoff**는 무엇일까?

### Data might be undefined

이미 알고 있겠지만 `useForm` 또한 전체 **Form**에 대해 사용할 수 있는 `defaultValues` 속성을 가지고 있으며 이는 거대한 **Form**에 사용하기 좋다. 그러나 우리는 훅을 조건부로 호출할 수 없고 첫 번째 렌더링 주기에서 데이터가 `undefined`이기 때문에(데이터 패칭을 처음 하기 때문에) 우리는 예제와 동일한 컴포넌트에서 `useForm`의 `defaultValues` 속성을 사용할 수 없다.

```jsx
const { data } = useQuery(['person', id], () => fetchPerson(id));
// 🚨 this will initialize our form with undefined
const { register, handleSubmit } = useForm({ defulatValues: data });
```

`useState`로 복사하거나 제어되지 않는 **Form**(참고로 **React Hook Form**이 내부적으로 `uncontrolled` 방식)을 사용할 때도 동일한 문제를 겪는다. 가장 좋은 해결책은 **Form**을 다른 컴포넌트로 분리하는 것이다.

```jsx
function PersonDetail({ id }) {
  const { data } = useQuery(['person', id], () => fetchPerson(id));
  const { mutate } = useMutation((values) => updatePerson(values));

  if (data) {
    return <PersonForm person={data} onSubmit={mutate} />;
  }

  return 'loading...';
}

function PersonForm({ person, onSubmit }) {
  const { register, handleSubmit } = useForm({ defaultValues: person });

  return (
    <form onSubmit={handleSubmit(mutate)}>
      <div>
        <label htmlFor="firstName">First Name</label>
        <input {...register('firstName')} />
      </div>
      <div>
        <label htmlFor="lastName">Last Name</label>
        <input {...register('lastName')} />
      </div>
      <input type="submit" />
    </form>
  );
}
```

위 방식은 데이터 패칭하는 부분을 분리하기 때문에 나쁘지 않다. 필자는 개인적으로 위와 같은 분리 방법을 좋아하진 않지만 위 예제는 여기서 마무리하겠다.

### No background updates

**React Query**는 서버 상태로 **UI**를 최신 상태로 유지한다. 우리가 서버 상태를 어딘가에 복사하는 순간 **React Query**는 제 할 일을 더 이상 하지 못한다.

즉, **React Query**가 패칭해온 서버 상태를 관리하게 두지 않고 해당 상태를 어딘가에 복사하는 순간 해당 상태값은 **React Query**의 관리 영역을 벗어나며 최신 상태를 유지하지 못한다는 뜻이다. 만약 어떠한 이유로 **Background refetch**가 일어난 경우 새로운 데이터는 가져오겠지만 우리의 **Form** 상태값은 해당 데이터로 업데이트되지 않을 것이다(임의로 복사한 데이터, 즉 **React Query**가 관리하지 않는 영역).

이는 우리가 해당 **Form** 상태(프로필 페이지의 **Form**과 같이 특정 사용자가 자신만 볼 수 있는 페이지)를 가지고 작업하는 유일한 사람들이라면 문제가 되지 않을 것이다. 이러한 경우 최소한 `query`의 `statleTime`을 더 높게 설정하여 **Background refetch**가 발생하지 않게 해야한다. 결국 화면에 업데이트를 반영하지 않을 거라면 서버에 추가적으로 데이터 패칭을 할 필요가 있을까?

```jsx
// ✅ opt out of background updates
const { data } = useQuery(['person', id], () => fetchPerson(id), {
  statleTime: Infinity,
});
```

위와 같은 접근 방식은 더 거대한 **Form**이나 협업 환경에서 문제가 될 수 있다. 더 거대한 **Form**의 경우 사용자가 해당 **Form**을 채우는데 더 오랜 시간이 걸릴 것이다. 만약 다수의 사람들이 동일한 **Form**을 가지고 작업하는 경우 화면에 부분적으로 오래된 데이터가 보여지기 때문에 마지막으로 업데이트한 사람이 다른 사람이 변경한 값을 덮어쓸 수도 있다.

이제 React Hook Form이 사용자에 의해 변경된 필드를 감지하여 서버에게 오직 변경이 발생한 필드들만 보낼 것이다(see [the example here](https://codesandbox.io/s/react-hook-form-submit-only-dirty-fields-ol5d2)). 그러나 이는 여전히 다른 사용자가 마지막으로 업데이트한 값을 우리에게 보여주지 않는다. 만약 우리가 다른 사람이 특정 필드를 변경했다는 사실을 알았다면 입력값을 변경하려고 했을 지도 모른다.

그렇다면 우리가 **Form**을 수정하는 동안 백그라운드 업데이트를 반영하기 위해선 무엇을 해야할까?

## Keeping background updates on

접근 방법 중 하나는 상태값을 엄격하게 분리하는 것이다. 서버 상태는 **React Query**가 계속 관리하고 오직 사용자가 변경한 값만 클라이언트 상태값으로 관리한다. 이때 우리가 사용자에게 보여주는 건 두 가지 상태값에서 파생된 값이다. 만약 사용자가 필드를 변경한 경우 클라이언트 상태를 보여주고 그게 아니라면 서버 상태를 보여준다.

```jsx
function PersonDetail({ id }) {
  const { data } = useQuery(['person', id], () => fetchPerson(id));
  const { control, handleSubmit } = useForm();
  const { mutate } = useMutation((values) => updatePerson(values));

  if (data) {
    return (
      <form onSubmit={handleSubmit(mutate)}>
        <div>
          <label htmlFor="firstName">First Name</label>
          <Controller
            name="firstName"
            control={control}
            render={({ field }) => (
              // ✅ derive state from field value (client state).
              // and data (server state)
              <input {...field} value={field.value ?? data.firstName} />
            )}
          />
        </div>
        <div>
          <label htmlFor="lastName">Last Name</label>
          <Controller
            name="lasttName"
            control={control}
            render={({ field }) => <input {...field} value={field.value ?? data.lastName} />}
          />
        </div>
        <input type="submit" />
      </form>
    );
  }

  return 'loading...';
}
```

이 접근 방식을 사용하면 변경되지 않은 필드와 여전히 관련이 있기 때문에 백그라운드 업데이트를 계속 유지할 수 있다. 이러면 더 이상 **Form**을 처음 렌더링할 때 `initialState`에 바인딩되지 않는다. 늘 그렇듯 여기에도 몇 가지 주의사항이 있다.

### You need controlled fields

필자가 아는 한 `uncontrolled` 필드로 위 접근 방법을 적용할 수 있는 좋은 방법이 없다. 그렇기에 위 예제에선 `controlled` 필드를 사용했다.

### Deriving state might be difficult

이 접근 방법은 **nullish coalesce**(**nullish** 병합)을 사용하여 서버 상태로 쉽게 돌아갈 수 있는 얕은 **Form**에는 적합하지만 중첩된 객체를 가지고 진행하는 병합엔 꽤나 어려울 수도 있다. 또한 백그라운드에서 **Form**의 값들을 변경하는 것은 의심스러운 사용자 경험이 될 수도 있다. 따라서 조금 더 나은 방법은 서버 상태와 동기화되지 않은 값(사용자가 변경한 값)을 강조 표시하고 사용자가 결정하도록 놔두는 것이다.

어떤 방법을 선택하든 각 접근 방법이 가져오는 장단점을 알아야 한다.

## Tips and Tricks

**Form**을 설정하기 위한 두 가지 주요 방법 외에도 **React Query**와 **Form**을 통합하기 위한 작지만 중요한 트릭이 있다.

### Double submit prevention

**Form**이 두 번 `submit`되는 것을 방지하기 위해 `useMutation`에서 반환되는 `isLoading` 속성을 사용할 수 있다. 이는 `mutation`이 실행되는 동안 `true`일 것이다. **Form** 자체를 비활성화하려면 기본 `submit` 버튼을 비활성화 하기만 하면 된다.

```jsx
const { mutate, isLoading } = useMutation((values) => updatePerson(values));
<input type="submit" disabled={isLoading} />;
```

### Invalidate and reset after mutation

**Form** `submission` 후에 바로 다른 페이지로 리다이렉트할 게 아니라면 `invalidation` 완료 후 **Form**을 초기화하는 것이 좋을 수도 있다. [Mastering Mutations](https://tkdodo.eu/blog/mastering-mutations-in-react-query#some-callbacks-might-not-fire)에서 언급한 대로 아마 `mutate`의 `onSuccess` 콜백에서 `invalidation` 하고 싶을 것이다. 서버 상태를 다시 가져오려면 `undefined`로 초기화하기만 하면 되므로 상태를 분리하는 경우에도 가장 잘 동작한다.

```jsx
function PersonDetail({ id }) {
  const queryClient = useQueryClient();
  const { data } = useQuery(['person', id], () => fetchPerson(id));
  const { control, handleSubmit, reset } = useForm();
  const { mutate } = useMutation(updatePerson, {
    // ✅ return Promise from invalidation
    // so that it will be awaited
    onSuccess: () => queryClient.invalidateQueries(['person', id]),
  });

  if (data) {
    return (
      <form
        onSubmit={handleSubmit((values) =>
          // ✅ rest client state back to undefined
          mutate(value, { onSuccess: () => reset() })
        )}
      >
        <div>
          <label htmlFor="firstName">First Name</label>
          <Controller
            name="firstName"
            control={control}
            render={({ field }) => <input {...field} value={field.value ?? data.firstName} />}
          />
        </div>
        <input type="submit" />
      </form>
    );
  }

  return 'loading ...';
}
```

서버 상태와 클라이언트 상태를 나눠서 관리하고 있으니 `mutate`을 호출한 다음 `useMutation`의 `onSuccess`에 `invalidation` 작업을 넣어놓고 이를 기다리게 만든다. 이러면 `mutate` 콜백 함수 쪽에서 넣어놓은 `onSuccess`가 `invalidation` 작업이 완료된 후 실행되기 때문에 자연스레 업데이트된 서버 상태를 사용자가 볼 수 있게 된다(**Form**을 `reset` 메서드로 초기화했으니 서버 상태가 보인다).
