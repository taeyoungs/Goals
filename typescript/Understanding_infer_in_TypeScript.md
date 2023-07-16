# Understanding infer in TypeScript

> 원본 글  
> https://blog.logrocket.com/understanding-infer-typescript/

**목차**

- [Understanding infer in TypeScript](#understanding-infer-in-typescript)
  - [개요](#개요)
  - [The no-value `never` type](#the-no-value-never-type)
  - [Using conditional types in TypeScript](#using-conditional-types-in-typescript)
    - [The `extends` keyword](#the-extends-keyword)
    - [Conditional types and unions](#conditional-types-and-unions)
    - [Conditional types and functions](#conditional-types-and-functions)
  - [Using `infer` in TypeScript](#using-infer-in-typescript)
    - [React `prop` types](#react-prop-types)
  - [`Infer` keyword use cases](#infer-keyword-use-cases)
    - [Function’s first argument:](#functions-first-argument)
    - [Function’s second argument:](#functions-second-argument)
    - [Promise return type](#promise-return-type)
    - [Array type](#array-type)
  - [Conclusion](#conclusion)

## 개요

누구나 한 번쯤은 라이브러리를 잘못 입력한 적이 있을 것입니다. 다음 서드파티 함수를 예로 들어보겠습니다:

```typescript
function describePerson(person: {
  name: string;
  age: number;
  hobbies: [string, string]; // tuple
}) {
  return `${person.name} is ${person.age} years old and love ${person.hobbies.join(' and  ')}.`;
}
```

라이브러리에서 `describePerson`의 `person` 인자에 대한 standalone 타입을 제공하지 않는 경우, 미리 변수를 `person` 인자로 정의하면 TypeScript에서 올바르게 추론하지 못합니다.

```typescript
const alex = {
  name: 'Alex',
  age: 20,
  hobbies: ['walking', 'cooking'], // type string[] != [string, string]
};

/* Type string[] is not assignable to type [string, string] */
describePerson(alex);
```

TypeScript는 `alex`의 타입을 `{ name: string; age: number; hobbies: string[] }`로 추론하고 이를 `describePerson`의 인수로 사용할 수 없습니다.

그리고 설사 그렇다 하더라도, 적절한 자동 완성 기능을 갖추기 위해 `alex` 객체 자체에 대한 타입 검사가 있으면 좋을 것입니다. TypeScript의 `infer` 키워드 덕분에 이를 쉽게 달성할 수 있습니다.

```typescript
const alex: GetFirstArgumentOfAnyFunction<typeof describePerson> = {
  name: 'Alex',
  age: 20,
  hobbies: ['walking', 'cooking'],
};

describePerson(alex); /* No TypeScript errors */
```

TypeScript에서 `infer` 키워드와 조건부 타입을 사용하면 타입을 가져와서 나중에 사용할 수 있도록 해당 타입을 분리할 수 있습니다.

## The no-value `never` type

TypeScript에서 `never`는 "`no value`" 타입으로 취급되지 않습니다. 종종 막 다른 타입으로 사용되는 것을 볼 수 있습니다. TypeScript에서 `string | never`와 같은 유니온 타입은 `never`을 버리고 `string`으로 평가됩니다.

이를 이해하기 위해 `string`과 `never`를 수학적 집합으로 생각하면 되는데, `string`은 모든 문자열 값을 포함하는 집합이고 `never`는 아무 값도 포함하지 않는 집합(`∅ set`)입니다. 이러한 두 집합의 합은 분명히 전자의 집합입니다.

이와 대조적으로, `string | any`은 `any`로 평가됩니다. 다시 말하지만, 이것은 `string` 집합과 모든 집합을 포함하는 범용 집합(Universal Set, `U`) 사이의 결합으로 생각할 수 있으며, 놀랍게도 이 결합은 그 자체로 평가됩니다.

다른 타입과 결합하면 사라지기 때문에 `never`가 탈출구로 사용되는 이유를 설명합니다.

## Using conditional types in TypeScript

조건부 타입은 특정 제약 조건을 만족하는지 여부에 따라 타입을 수정합니다. JavaScript의 삼항식과 유사하게 작동합니다.

### The `extends` keyword

TypeScript에서 제약 조건(constraint)은 `extends` 키워드를 사용하여 표현됩니다. `T extends K`는 `T` 타입의 값도 `K` 타입이라고 가정해도 안전하다는 의미입니다. 예를 들어, `0 extends number`는 `var zero: number = 0`이 type safe하기 때문입니다.

따라서 제약 조건이 충족되는지 여부를 검사하고 다른 유형을 반환하는 제네릭(generic)을 가질 수 있습니다.

`StringFromType`은 수신한 원시 타입(primitive type)을 기반으로 리터럴 문자열(literal string)을 반환합니다:

```typescript
type StringFromType<T> = T extends string ? 'string' : never;

type lorem = StringFromType<'lorem ipsum'>; // 'string'
type ten = StringFromType<10>; // never
```

`StringFromType` 제네릭의 더 많은 경우를 다루기 위해 JavaScript에서 삼항 연산자를 중첩하는 것처럼 더 많은 조건을 체인으로 연결할 수 있습니다.

```typescript
type StringFromType<T> = T extends string
  ? 'string'
  : T extends boolean
  ? 'boolean'
  : T extends Error
  ? 'error'
  : never;

type lorem = StringFromType<'lorem ipsum'>; // 'string'
type isActive = StringFromType<false>; // 'boolean'
type unassignable = StringFromType<TypeError>; // 'error'
```

### Conditional types and unions

제약 조건으로 유니온을 확장하는 경우 TypeScript는 유니온의 각 멤버를 반복하고 자체 유니온을 반환합니다:

```typescript
type NullableString = string | null | undefined;

type NonNullable<T> = T extends null | undefined ? never : T; // Built-in type, FYI(= 참고로)

type CondUnionType = NonNullable<NullableString>; // evalutes to `string`
```

TypeScript는 `string | null | undefined`이라는 유니온을 한 번에 한 타입씩 반복하여 `T extends null | undefined`이라는 제약 조건을 테스트합니다.

다음과 같은 예시 코드라고 생각하면 됩니다:

```typescript
type stringLoop = string extends null | undefined ? never : string; // string

type nullLoop = null extends null | undefined ? never : null; // never

type undefinedLoop = undefined extends null | undefined ? never : undefined; // never

type ReturnUnion = stringLoop | nullLoop | undefinedLoop; // string
```

`ReturnUnion`은 `string | never | never`의 결합이므로 `string`으로 평가됩니다(위 설명 참조).

확장 유니온을 제네릭으로 추상화하면 TypeScript에서 내장 `Extract` 및 `Exclude` 유틸리티 유형을 어떻게 만들 수 있는지 확인할 수 있습니다:

```typescript
type Extract<T, U> = T extends U ? T : never;
type Exclude<T, U> = T extends U ? never : T;
```

### Conditional types and functions

타입이 특정 함수 모양을 확장하는지 확인하려면 `Function` 타입을 사용해서는 안 됩니다. 대신 다음 signature를 사용하여 가능한 모든 함수를 확장할 수 있습니다:

```typescript
type AllFunctions = (…args: any[]) => any
```

`...args: any[]`는 zero와 그 이상의 인수를 포함하며 `=> any`는 모든 반환 타입을 포함합니다.

## Using `infer` in TypeScript

`infer` 키워드는 조건부 타입을 보완하며 `extends` 절 외부에서는 사용할 수 없습니다. `Infer`를 사용하면 제약 조건 내에서 참조하거나 반환할 변수를 정의할 수 있습니다.

예를 들어, TypeScript 내장 `ReturnType` 유틸리티를 살펴보겠습니다. 함수 타입을 받아 반환 타입을 제공합니다:

```typescript
type a = ReturnType<() => void>; // void
type b = ReturnType<() => string | number>; // string | number
type c = ReturnType<() => any>; // any
```

먼저 타입 인자(`T`)가 함수인지 여부를 검사하고, 검사 과정에서 반환 타입을 변수로 만들어 `R`을 유추한 후 검사에 성공하면 반환하는 방식으로 이를 수행합니다:

```typescript
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

앞서 언급했듯이 이 기능은 주로 사용할 수 없는 타입에 액세스하고 이를 사용할 때 유용합니다.

### React `prop` types

React에서는 종종 `prop` 타입에 접근해야 할 때가 있습니다. 이를 위해 React는 `ComponentProps`라는 `infer` 키워드로 구동되는 `prop` 타입에 접근하기 위한 유틸리티 타입을 제공합니다.

```typescript
type ComponentProps<T extends keyof JSX.IntrinsicElements | JSXElementConstructor<any>> =
  T extends JSXElementConstructor<infer P>
    ? P
    : T extends keyof JSX.IntrinsicElements
    ? JSX.IntrinsicElements[T]
    : {};
```

타입 인수가 React 컴포넌트인지 확인한 후, 프로퍼티를 유추하여 반환합니다. 실패하면 타입 인수가 `IntrinsicElements`(`div`, `button` 등)인지 확인하고 해당 프로퍼티를 반환합니다. 모두 실패하면 `{}`를 반환하는데, 이는 TypeScript에서 "**null이 아닌 모든 값**"을 의미합니다.

## `Infer` keyword use cases

`infer` 키워드를 사용하는 것은 종종 타입을 언래핑하는 것으로 설명됩니다. 다음은 `infer` 키워드의 몇 가지 일반적인 사용 예입니다.

### Function’s first argument:

이것이 첫 번째 예제의 솔루션입니다:

```typescript
type GetFirstArgumentOfAnyFunction<T> = T extends (
  first: infer FirstArgument,
  ...args: any[]
) => any
  ? FirstArgument
  : never;

type t = GetFirstArgumentOfAnyFunction<(name: string, age: number) => void>; // string
```

### Function’s second argument:

```typescript
type GetSecondArgumentOfAnyFunction<T> = T extends (
  first: any,
  second: infer SecondArgument,
  ...args: any[]
) => any
  ? SecondArgument
  : never;

type t = GetSecondArgumentOfAnyFunction<(name: string, age: number) => void>; // number
```

### Promise return type

```typescript
type PromiseReturnType<T> = T extends Promise<infer Return> ? Return : T;

type t = PromiseReturnType<Promise<string>>; // string
```

### Array type

```typescript
type ArrayType<T> = T extends (infer Item)[] ? Item : T;

type t = ArrayType<[string, number]>; // string | number
```

## Conclusion

`infer` 키워드는 서드파티 TypeScript 코드로 작업하는 동안 타입을 언래핑하고 저장할 수 있는 강력한 도구입니다. 이 글에서는 `never` 키워드, `extends` 키워드, 유니온 및 함수 signature를 사용하여 강력한 조건부 타입을 작성하는 다양한 측면에 대해 설명했습니다.
