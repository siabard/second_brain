
React 에서 Global State를 관리하는 다양한 방법 중 하나가 Zustand이다. Zustand는 일반적인 `useState` 훅처럼 쉽게 사용이 가능하다..

# Store의 정의
Zustand에서 State는 Store라는 묶음으로 관리된다. 해당 Store에는 저장할 값 뿐 아니라, State를 변경할 수 있는 메서드도 함께 보관하는 것이 일반적이다.

```typescript
import { create } from 'zustand';

type CounterStore = {
  count: number;
  increment: () => void;
  decrement: () => void;
  incrementAsync: () => Promise(void);
}
```

async 함수의 경우 `Promise`를 반환하는 것에 유의하자.

```typescript
export const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => {
    set((state) => ({count: state.count + 1}))
  },
  decrement: () => {
    set((state) => ({count: state.count - 1}))
  },
  incrementAsync: async () => {
    await new Promise((resolve) => setTimeout(resolve, 1000)); // async로 동작할 API 호출 부분 등
    set((state) => ({count: state.count + 1}))
  }
}));
```

해당 코드를 React 컴포넌트에서 이용하는 방법은 아래와 같다.

```typescript
import { useCounterStore } from '../store/counter';

...

const count = useCounterStore((state) => state.count);
const increment = useCounterStore((state) => state.increment);

```