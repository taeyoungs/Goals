# React Hook Form with Yup, TypeScript

**Yup** 라이브러리 다음 릴리즈 버전이 올라오지 않는 이상 **TypeScript**와 같이 사용할 경우 **Yup**을 통해 만든 `Schema` 객체만으로 **React Hook Form**의 타입 추론이 정상적으로 이루어지지 않는다.

따라서, **React Hook Form**의 타입 추론이 자연스럽게 이루어지도록 `useForm`을 한번 래핑한 커스텀 훅을 만들어 이를 이용

```jsx
import { yupResolver } from '@hookform/resolvers/yup';
import { useForm, UseFormProps, UseFormReturn } from 'react-hook-form';
import * as Yup from 'yup';

/**
 * This function is type inference ready and will auto-validate the useForm with the proper values.
 *
 * If you don't already have the schema or use a dynamic schema, consider useFormWithSchemaBuilder()
 *
 * @param schema - A valid you schema
 * @param useFormProps
 * @returns
 */
export function useFormWithSchema<T extends Yup.AnyObjectSchema>(
  schema: T,
  useFormProps?: UseFormProps<Yup.Asserts<T>>,
): UseFormReturn<Yup.Asserts<T>> {
  return useForm({ ...useFormProps, resolver: yupResolver(schema) });
}

/**
 * Use this to prevent importing yup library in your components, if you don't already have the schema, and if you want to have a dynamic schema values.
 * This is rebuild on each render.
 * This function is type inference ready and will auto-validate the useForm with the proper values.
 *
 * If you have already the schema or use a static schema, consider useFormWithSchema()
 *
 * @param schemaBuilder - Should return a validation schema
 * @param useFormProps Do not provide "resolver" value as it will be ignored.
 * @returns
 */
export function useFormWithSchemaBuilder<T extends Yup.AnyObjectSchema>(
  schemaBuilder: (yup: typeof Yup) => T,
  useFormProps?: UseFormProps<Yup.Asserts<T>>,
): UseFormReturn<Yup.Asserts<T>> {
  return useForm({ ...useFormProps, resolver: yupResolver(schemaBuilder(Yup)) });
}
```
