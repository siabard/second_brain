
# 간단한 React 화면
간단히 React로만 구성해보자.

```tsx

import React, { useState } from 'react';

const App = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [errors, setErrors] = useState<{email: string, password: string}>({
    email: "",
    password: "",
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();

    setErrors({email: "", password: ""});
    if(!email.includes("@")) {
      setErrors({...errors, email: "Email must contain @"});
      return;
    }

    if(password.length < 8) {
      setErrors({...errors, password: "Password must e at least 8 chars"});
      return;
    }

    // submit form..
  };

  return (
    <form className="..." onSubmit={handleSubmit}>
      <input type="text" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} />
      {errors.email && <div className="text-red-500">{errors.email}</div>}
      <input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} />
      {errors.password && <div className="text-red-500">{errors.password}</div>}
      <button type="submit">{"Submit"}</button>
    </form>
  );
};
  
```

위와 같이 기존 React 컴포넌트만으로도 오류를 처리할 수 있다. 

# React-hook-form

간단한 Form의 경우 React 만으로도 처리가 가능하지만, 폼의 크기가 커기고, 처리가 복잡해지게되면 별도의 라이브러리를 사용하는 것이 좋다. 

```tsx
import { useForm } from 'react-hook-form';

type FormFields = {
  email: string;
  password: string;
}

const App = () => {
  const {register} = useForm<FormFields>();
  
  return (
    <form className="tutorial gap-2">
      <input {...register("email"} type="text" placeholder="Email" />
      <input {...register("password")} type="password" placeholder="Password" />
      <button type="submit">Submit</button>
    </form>
  );
};

export default App;
```

각 `input` 필드에 `{...register()}`를 통해서 지정한 필드를 react-hook-form 과 연동한 것을 볼 수 있다. 이제 각 필드의 값이 변경되면, react-hook-form 으로 생성된 훅으로 해당 값이나 이벤트가 전달될 것이다.  

## form에 submit 함수 연결하기

```tsx
import { SubmitHandler, useForm } from 'react-hook-form';

...
const App = () => {
  const { handleSubmit, register } = useForm<FormFields>();
  
  const onSubmit: SubmitHandler<FormFields> = (data) => {
    // submit 된 값은 data로 온다.
  };

  return (
    <form className="..." onSubmit={handleSubmit(onSubmit)} >
      ...
    </fomr>
  );

};
```

Form에서 Submit 이벤트를 처리하기위해 `handleSubmit`에 처리할 함수를 파라미터로 넘겨준다. 기본 submit 이벤트를 대신하며, 데이터가 제대로 입력되었다면 입력된 Form data를 처리하게 된다.


## validation
`register` 함수는 두번째 파라미터로 입력필드의 옵션 설정을 받는다. 예를 들어 `email` 필드가 필수여야하고, 특정한 패턴을 갖는다면..

```tsx
<input {...reigster("email", {
  required: true,
  pattern: /^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$/,
})} />
```
처럼 설정해주면 된다.

최소 길이 지정이 필요하면

```tsx
<input {...register("password", {
  required: true,
  minLength: 8,
})} />
```

정합성 검사를 별도로 하고 싶다면..

```tsx
<input {...register("email"), {
  required: true,
  validate: (value) => value.includes("@")
}} />
```

처럼 `validate` 에 `true`, `false`를 반환하는 함수를 정의하면 된다.

## FormState 에 접근하기

form의 현재 상태 (오류, dirty 상태) 등은 `formState` 라는 내부 상태값에 들어있다. 

```tsx

const {
  register,
  handleSubmit,
  formState
} = useForm<FormFields>();
```

`formState`의 특정값에만 접근하고 싶다면, 구조분해를 이용하여 가져온다.

```tsx
const {
  register,
  handleSubmit,
  formState: { errors },
} = useForm<FormFields>();
```

이 값은 다음과 같이 사용할 수 있다.
```tsx
<input {...reigster("email", {
  required: "Email is not valid",
  pattern: /^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$/,
})} />
{ errors.email && <div>{errors.email.message}</div>}
```

위에서 에러 메세지는 `register`에서 `required` 에 설정된 값을 그대로 쓸 수도 있다.  즉 validate 검사를 하는 모든 항목은 반환값이 true 가 아니라면, 해당 값을 errors의 message 값으로 사용하게 된다.

위의 값을 더 명확하게 하고 싶다면


```tsx
<input {...reigster("email", {
  required: {
    value: true,
    message: "Email is not entered",
  },
  pattern: {
    value: /^[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,4}$/,
    message: "Email is not valid",
  },
})} />
{ errors.email && <div>{errors.email.message}</div>}
```
식으로 `value`와 `message` 를 사용해서 더 세세하게 나누어 설정할 수 있다.

## 비동기 방식의 submit 결과 제어
submit을 하고 비동기 방식의 다른 작업이 필요(API 호출등)한 경우, 진행 사항을 파악해야할 필요가 있다. `formState` 에는 `isSubmitting` 이라는 진행중 여부를 알려주는 값이 있다.

```tsx
const {
  register,
  handleSubmit,
  setError,
  formState: { errors, isSubmitting },  
} = useForm<FormFields>();

const onSubmit: SubmitHandler<FormFields> = async (data) => {
  try {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    // 일부러 오류를 발생시킴
    throw new Error();
    // async 작업 후 처리
    console.log(data);
  } catch(e) {
    setError("email", {
      "message": "Email update error"
    });
  }
  
};

return (
...
  <button disabled={isSubmitting} type="submit">
    { isSubmitting ? "Loading..." : "Submit" }
  </button>
);
```

 오류가 발생하면 `setError` 를 이용해 관련 항목을 설정한다. `setError`에 첫번째 파라미터는 에러가 어떤 필드에 관한 것인지 지정한다. 만약 form 전체에 대한 문제라면 `root`라고 설정한다.

```tsx
const {
  register,
  handleSubmit,
  setError,
  formState: { errors, isSubmitting },  
} = useForm<FormFields>();

const onSubmit: SubmitHandler<FormFields> = async (data) => {
  try {
    await new Promise((resolve) => setTimeout(resolve, 1000));
    // 일부러 오류를 발생시킴
    throw new Error();
    // async 작업 후 처리
    console.log(data);
  } catch(e) {
    setError("root", {
      "message": "Email update error"
    });
  }
  
};

return (
...
  <button disabled={isSubmitting} type="submit">
    { isSubmitting ? "Loading..." : "Submit" }
  </button>
  { errors.root && (
    <div>{errors.root.message}</div>
  )}
);
```

## Zod 를 이용한 스키마 관리

TypeScript 프로젝트이고 React-Form-Hook 을 사용한다면, Zod가 괜찮은 스키마 테스트 툴이다. 기존에 설정한 `FormFields` 타입을 zod 를 통해 정의하면 다음과 같다.

```tsx
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

type FormFields = z.infer<typeof schema>();
```

React-Form-hook 에 등록하려면 resolver 가 필요하다.

```tsx
import { zodResolver } from "@hookform/resolvers/zod";
...

const {
  register,
  handleSubmit,
  setError,
  formState: { errors, isSubmitting },  
} = useForm<FormFields>({
  defaultValues: {},
  resolver: zodResolver(schema),
});
```

위와 같이 설정이 끝났더면, 기존의 `register` 에 붙였던 커스텀 Validator들은 더이상 필요없다.

```tsx
<input {...reigster("email")} />
{ errors.email && <div>{errors.email.message}</div>}
```

위와 같이 하면 특정 값이 들어왔을 때 해당 유효성 검증을 zod 를 통해서 할 수 있다.

### Zod와 Yup

Zod는 TypeScript 프로젝트에 맞춰 구축되었으며, 모든 타입체크를 실제로 진행한다. 반면에 Yup은 유효성 검증 결과를 문자열 메세지로 보관은 하지만, 타입에러를 내주지는 않는다.
타입스크립트 프로젝트를 진행한다면 Zod쪽이 더 괜찮아보인다.