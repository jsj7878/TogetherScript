# 10. 상태 관리

## 10.1 상태관리

### 1. 상태(State)

공식문서에 따르면 리액트에서의 상태는 **렌더링 결과에 영향을 주는 정보를 담은 순수 자바스크립트 객체**다.

상태는 시간이 지나면서 변할 수 있는 동적인 데이터이며, 값이 변경될 때마다 컴포넌트의 렌더링 결과물에 영향을 준다.

리액트 내부 API인 `Context API`를 사용해서 전역 상태를 관리할 수 있지만, 성능 문제와 상태의 복잡성을 해결하기 위한 여러 라이브러리를 사용한다. 해당 서적에서는 MobX, Redux, Recoil, Zustand 4가지의 상태 관리 라이브러리를 간단히 소개하고 있다.

- 지역 상태: 컴포넌트 내부에서 사용되는 상태로, 보통 체크박스의 체크 여부나 폼의 입력값 등을 지역 상태로 사용한다. `useState`나 `useReducer`를 이용해서 생성, 관리한다.

- 전역 상태: 앱 전체에서 공유하는 상태를 말한다. 여러 개의 컴포넌트가 상태를 공유하며 상태가 변경되면 구독하고 있는 상태들도 따라 업데이트된다. Prop Drilling 문제를 피하기 위해 지역 상태를 해당 컴포넌트들 사이의 전역 상태도 공유하기도 한다.

> 💡 Prop Drilling: props로 데이터를 전달하는 과정에서 해당 데이터가 필요하지 않은 중간 컴포넌트에도 props를 통해 전달되는 과정을 말한다. Prop Drilling이 발생하면 코드가 복잡해질 수 있다.

- 서버 상태: 사용자 정보와 같은 외부 서버에 저장되는 상태들을 말한다. 서버 상태는 지역 상태나 전역 상태와 동일한 방법으로 관리되나 최근에는 react-query, SWR 같은 라이브러리를 통해 관리하기도 한다.

&nbsp;

### 2. 상태를 잘 관리하기 위한 가이드

상태를 잘못 관리하게 되면 애플리케이션의 복잡성을 증가시키고 동작을 예측하기 힘들게 되어 애플리케이션의 관리가 어려워진다. **상태는 렌더링과 직접적으로 연관되어 있기 때문에 상태는 필요한 만큼만 사용해야 한다.** 가능하다면 Stateless 컴포넌트를 활용해야 한다.

---

상태를 정의할 땐 2가지 사항을 고려해야 한다.

> 1. **시간이 지나도 변하지 않으면 상태가 아니다.**

시간이 지나도 변하지 않는 값은 **상태로 관리하지 않고, 객체 참조 동일성을 유지하는 방법을 고려한다**.

컴포넌트가 마운트될 때 스토어 객체 인스턴스가 생성되고, 언마운트될 때까지 참조가 변하지 않는다고 가정하자.

단순히 상수 변수에 저장하여 사용할 수도 있다. -> 렌더링될 때마다 새로운 객체 인스턴스 생성으로 컨텍스트나 props로 전달 시 매번 다른 객체로 인식되어 불필요한 렌더링이이 발생한다.

그럼 컴포넌트가 마운트될 때만 인스턴스 생성되게 하고 렌더링될 때마다 동일한 객체를 참조하게끔 만드려면 어떻게 해야할까? -> `useMemo`를 사용할 수도 있지 않을까? -> 객체 참조 동일성을 유지하기 위한 방법으로는 권장되지 않는다. 리액트 공식 문서에서는 `useMemo`는 성능 향상을 위한 용도로만 사용하기를 권고하고 있다. 리액트에서는 메모리 확보를 위해 이전 메모이제이션 데이터를 삭제될 수 있다고 한다.

이제 2가지의 방법을 고려해볼 수 있다.

1. `useState`의 초깃값만 지정

```js
import { useState } from "react";

const [store] = useState(() => new Store());
```

2. `useRef` 사용

```ts
import { useRef } from "react";

const store = useRef<Store>(null);

if (!store.current) {
  store.current = new Store();
}
```

결론부터 말하자면 `useRef`를 사용하는 것이 가장 적합하다.

`useState(new Store())`로 스토어를 생성하게 되면 객체 인스턴스가 사용되지 않더라도 렌더링마다 인스턴스가 생성되어 애플리케이션 초깃값 설정에 큰 비용이 들 수 있다. 따라서 콜백 함수로 지연 초기화를 해주는 방식을 사용한다.

다만 `useState`를 사용하는 건 처음에 우리가 정의한 의미와 잘 맞지 않아 적절하지 않다. 상태를 시간이 지나면서 변화되어 렌더링에 영향을 주는 데이터라고 정의했지만, 모든 렌더링 과정에서 객체의 참조를 동일하게 유지하고자 하는 것이 현재 목적이기 때문이다.

공식 문서에서도 `useRef`가 해당 목적에 제일 적합한 훅이라고 소개하고 있다. 마찬가지로 `new Store()`를 인자로 직접 사용하면 렌더링마다 불필요한 인스턴스가 생성되므로 위에 소개한 코드처럼 작성하는 것이 좋다.

&nbsp;

> **2. 파생된 값은 상태가 아니다.**

**부모에게서 전달받을 수 있는 props나 기존 상태에서 계산될 수 있는 값은 상태가 아니다.**

리액트 앱에서 상태를 정의할 땐, SSOT(Single Source Of Truth, 어떠한 데이터도 단 하나의 출처에서 생성하고 수정해야 한다는 원칙의 방법론)를 고려해야 한다.

데이터의 정확성과 일관성 보장을 위해, 파생된 값을 상태로 관리하지 않는다.

아래 컴포넌트는 props로 파생된 값을 상태로 관리하고 있다.

```tsx
import React, { useState } from "react";

type UserEmailProps = {
  initialEmail: string;
};

const UserEmail: React.VFC<UserEmailProps> = ({ initialEmail }) => {
  const [email, setEmail] = useState(initialEmail);
  const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(event.target.value);
  };
  return (
    <div>
      <input type="text" value={email} onChange={onChangeEmail} />
    </div>
  );
};
```

이 컴포넌트는 `initialEmail`이 변경되어도 `email` 값이 변경되지 않아 props와 상태를 동기화하기 어렵다.

`useEffect`로 `initialEmail`이 변경되면 prop와 상태를 동기화하도록 코드를 작성할 수도 있겠지만, 사용자가 `email`을 변경한 뒤에 `initialEmail`이 변경되면 사용자의 값을 무시하고 `email`을 `initialEmail` 값으로 설정되는 문제가 발생한다. 따라서 `useEffect`는 리액트 외부 데이터와 동기화할 때만 사용해야 하며 리액트 내부에 존재하는 데이터를 동기화하면 개발자가 추적하기 어려운 오류가 발생할 수 있다.

```ts
// 잘못된 데이터 동기화 예시
import { useState, useEffect } from "react";

const [email, setEmail] = useState(initialEmail);

useEffect(() => {
  setEmail(initialEmail);
}, [initialEmail]);
```

이러한 문제를 해결하기 위해서는 상위 컴포넌트에서 상태를 관리하도록 하는 **상태 끌어올리기 기법을 사용**한다. `UserEmail`에서 관리하던 상태를 부모 컴포넌트로 옮겨서 `email` 데이터의 출처를 props 하나로 통일해야 한다.

```tsx
import React, { useState } from "react";

type UserEmailProps = {
  email: string;
  setEmail: React.Dispatch<React.SetStateAction<string>>;
};

const UserEmail: React.VFC<UserEmailProps> = ({ email, setEmail }) => {
  const onChangeEmail = (event: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(event.target.value);
  };
  return (
    <div>
      <input type="text" value={email} onChange={onChangeEmail} />
    </div>
  );
};
```

두 컴포넌트 간 동일한 데이터를 상태로 갖고 있을 땐, 두 컴포넌트 간 상태를 동기화하는 방법보단, 가까운 공통 부모 컴포넌트로 상태를 끌어올려서 SSOT를 지키는 방법이 데이터의 정확성, 일관성을 보장해준다는 측면에서 더 좋다.

다음으로, 아래 코드는 상태를 새로운 상태로 정의함으로써 단일 출처가 아닌 여러 출처를 가지게 되어 동기화 문제가 발생하고 있다.

```ts
import { useState, useEffect } from "react";

const [items, setItems] = useState<Item[]>([]);
const [selectedItems, setSelectedItems] = useState<Item[]>([]);

useEffect(() => {
  setSelectedItems(items.filter((item) => item.isSelected));
}, [items]);
```

이러한 방식은 `items`와 `selectedItems`의 데이터가 동기화되지 않을 수 있고, 여러 상태가 복잡하게 얽히게 되면 동작 흐름을 파악하기 어렵다. `selectedItems`를 사용해 `items`에서 가져온 데이터가 아닌 임의의 데이터가 들어올 수 있다는 점도 구조적으로 가능하기 때문에 개발자의 의도와 다른 동작을 할 수도 있다.

성능 상의 문제도 있다. `items`가 변경되며 렌더링이 발생하고, `items`의 값이 변경됨을 감지해서 `selectedItems` 값을 변경하며 다시 렌더링이 발생하며 2번의 렌더링이 발생한다.

때문에 여러 출처를 하나의 출처로 합쳐주어야 한다. 간단한 방법은 계산된 값을 변수에 담는 것이다.

```ts
import { useState } from "react";

const [items, setItems] = useState<Item[]>([]);
const selectedItems = items.filter((item) => item.isSelected);
```

다만 이 경우는 매번 렌더링될 때마다 `selectedItems`를 다시 계산하므로 `useMemo`를 사용하면 `items`가 변경될 때만 계산을 수행하게 되어 성능을 개선할 수 있다.

```ts
import { useState, useMemo } from "react";

const [items, setItems] = useState<Item[]>([]);
const selectedItems = useMemo(
  () => items.filter((item) => item.isSelected),
  [items]
);
```

&nbsp;

### useState대신 useReducer를 사용하는 예시

`useReducer`는 다음과 같은 상황에 사용을 권장한다.

1. 다수의 하위 필드를 포함하고 있는 복잡한 상태 로직을 다룰 때
2. 다음 상태가 이전 상태에 의존적일 때

`useReducer`를 사용하면 '무엇을 변경할지', '어떻게 변경할지'를 분리하여 `dispatch`를 통해 어떤 작업을 할지 `action`으로 넘기고 `reducer` 함수 내에서 상태를 업데이트하는 방식을 정의할 수 있다.

리뷰 리스트를 필터링하여 보여주기 위한 쿼리를 상태로 저장하고 싶다. 검색 날짜 범위, 리뷰 점수, 키워드 등 많은 하위 필드를 가지고 있다.

```ts
// 날짜 범위 기준 - 오늘, 1주일, 1개월
type DateRangePreset = "TODAY" | "LAST_WEEK" | "LAST_MONTH";

type ReviewRatingString = "1" | "2" | "3" | "4" | "5";

interface ReviewFilter {
  // 리뷰 날짜 필터링
  startDate: Date;
  endDate: Date;
  dateRangePreset: Nullable<DateRangePreset>;
  // 키워드 필터링
  keywords: string[];
  // 리뷰 점수 필터링
  ratings: ReviewRatingString[];
  // ... 이외 기타 필터링 옵션
}

// Review List Query State
interface State {
  filter: ReviewFilter;
  page: string;
  size: number;
}
```

이러한 데이터 구조에 `useState`로 다루게 되면 상태를 업데이트할 때, `page` 값만 업데이트하고 싶어도 전체 데이터를 다 가져와서 `page` 값을 업데이트하게 되기에 과정에서 `filter`나 `size`와 같은 값까지 수정될 가능성이 있다. 또한 '`size`를 업데이트할 때 `page`를 0으로 설정해야 한다' 등의 업데이트 규칙이 있다면 `useReducer`를 사용하는게 좋다.

```ts
import React, { useReducer } from "react";

// Action 정의
type Action =
  | { payload: ReviewFilter; type: "filter" }
  | { payload: number; type: "navigate" }
  | { payload: number; type: "resize" };

// Reducer 정의
const reducer: React.Reducer<State, Action> = (state, action) => {
  switch (action.type) {
    case "filter":
      return {
        filter: action.payload,
        page: 0,
        size: state.size,
      };
    case "navigate":
      return {
        filter: state.filter,
        page: action.payload,
        size: state.size,
      };
    case "resize":
      return {
        filter: state.filter,
        page: 0,
        size: action.payload,
      };
    default:
      return state;
  }
};

// useReducer 사용
const [state, dispatch] = useReducer(reducer, getDefaultState());

// dispatch 예시
dispatch({ payload: filter, type: "filter" });
dispatch({ payload: page, type: "navigate" });
dispatch({ payload: size, type: "resize" });
```

이외에도 `boolean` 상태를 토글하는 액션만 사용하는 경우 `useReducer`를 사용할 수 있다.

```tsx
import { useReducer } from "react";

// Before
const [fold, setFold] = useState(true);

const toggleFold = () => {
  setFold((prev) => !prev);
};

// After
function Component() {
  const [fold, toggleFold] = useReducer((v) => !v, true);

  return <button onClick={toggleFold}>{fold ? "Expand" : "Collapse"}</button>;
}
```

&nbsp;

### 3. 전역 상태 관리와 상태 관리 라이브러리

**상태는 사용하는 곳과 최대한 가까워야 하며 사용 범위를 제한해야만 한다.**

전역 상태를 정의하는 방법은 크게 두 가지가 있다.

1. 컨텍스트 API + `useState` 혹은 `useReducer`
2. 외부 상태 관리 라이브러리

---

### 컨텍스트 API(Context API)

컨텍스트 API를 사용하면 전역적으로 공유하고 싶은 데이터를 '컨텍스트'로 정의하고 해당 컨텍스트를 구독한 컴포넌트에서만 데이터를 읽게 할 수 있다. UI 테마 정보나 로케일 데이터같이 전역적으로 제공하거나 props를 하위 컴포넌트에 계속해서 전달하고 싶을 때 유용하다.

`TabGroup`이라는 컴포넌트가 있다고 하자. 이 컴포넌트는 내부에 복수의 `Tab` 컴포넌트를 가진다. `type`이라는 prop을 모든 `Tab`에 전달하고 싶은데, `TabGroup`에만 prop을 전달해도 모든 `Tab`이 prop을 공유할 수 있게 하려면, 컨텍스트 API를 사용하면 된다.

```tsx
// TabGroup 컴포넌트뿐 아니라 모든 Tab 컴포넌트에도 type prop을 전달하고 있다.
<TabGroup type='sub'>
  <Tab name='텝 레이블 1' type='sub'>
    <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2' type='sub'>
    <div>123</div>
  </Tab>
</TabGroup>

// TabGroup 컴포넌트에만 전달하게 하고 싶다.
<TabGroup type='sub'>
  <Tab name='텝 레이블 1'>
   <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2'>
    <div>123</div>
  </Tab>
</TabGroup>
```

상위 컴포넌트(`TabGroup`)에 컨텍스트 프로바이더를 넣어주면, 하위 컴포넌트에서 해당 컴포넌트를 구독하여 데이터를 읽어올 수 있다.

```ts
import { FC } from "react";

const TabGroup: FC<TabGroupProps> = (props) => {
  const { type = "tab", ...otherProps } = useTabGroupState(props);
  /* ... 로직 생략 */
  return (
    <TabGroupContext.Provider value={{ ...otherProps, type }}>
      {/* ... */}
    </TabGroupContext.Provider>
  );
};

const Tab: FC<TabProps> = ({ children, name }) => {
  const { type, ...otherProps } = useTabGroupContext();
  return <>{/* ... */}</>;
};
```

아래와 같이 `createContext`라는 유틸리티 함수를 정의해서 사용하면 프로바이더와 해당 컨텍스트를 사용하는 훅을 간편하게 생성해서 사용할 수 있어 유용하다.

```ts
import React from "react";

type Consumer<C> = () => C;

export interface ContextInterface<S> {
  state: S;
}

export function createContext<S, C = ContextInterface<S>>(): readonly [
  React.FC<C>,
  Consumer<C>
] {
  const context = React.createContext<Nullable<C>>(null);

  const Provider: React.FC<C> = ({ children, ...otherProps }) => {
    return (
      <context.Provider value={otherProps as C}>{children}</context.Provider>
    );
  };

  const useContext: Consumer<C> = () => {
    const _context = React.useContext(context);
    if (!_context) {
      throw new Error(ErrorMessage.NOT_FOUND_CONTEXT);
    }
    return _context;
  };

  return [Provider, useContext];
}

// 사용 예시
interface StateInterface {}
const [context, useContext] = createContext<StateInterface>();
```

컨텍스트 API는 엄밀히 말하면 전역 상태를 관리하기 위한 것이 아니라 여러 컴포넌트 간에 값을 공유하기 위한 것에 가깝지만 `useState`나 `useReducer`와 같은 지역 상태 관리 훅과 결합하여 여러 컴포넌트 사이에서 상태를 공유하기 위해 쓰기도 한다.

아래 코드에서는 컨텍스트 API와 지역 상태 관리 훅을 이용해 다수의 컴포넌트 간 상태를 공유하고 있다.

```ts
import { useReducer } from "react";

function App() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <StateProvider.Provider value={{ state, dispatch }}>
      <ComponentA />
      <ComponentB />
    </StateProvider.Provider>
  );
}
```

⚠️ 컨텍스트 프로바이더의 props로 주입된 값이나 참조가 변경될 때마다 해당 컨텍스트를 구독하고 있는 모든 컴포넌트가 리렌더링되므로 성능이 중요한 애플리케이션에서는 권장되지 않는 방법이다. 차라리 외부 상태 관리 라이브러리를 사용하는 편이 좋다.

&nbsp;

## 10.2 상태 관리 라이브러리

상태 관리 라이브러리에 대해서는 책에서 라이브러리에 대해 간략한 설명과 간단한 코드 예제만 보여주고 있어 간단히 요약하고 넘어가겠다.

### 1. MobX

1. 객체 지향 프로그래밍과 반응형 프로그래밍 패러다임의 영향을 받아 객체 지향 스타일로 상태 관리 가능
2. 데이터가 언제, 어떻게 변하는지 추적하기 어려워 트러블슈팅이 힘듬

```ts
import { observer } from "mobx-react-lite";
import { makeAutoObservable } from "mobx";

class Cart {
  itemAmount = 0;

  constructor() {
    makeAutoObservable(this);
  }

  increase() {
    this.itemAmount += 1;
  }

  reset() {
    this.itemAmount = 0;
  }
}

const myCart = new Cart();
const CartView = observer(({ cart }) => (
  <button onClick={() => cart.reset()}>
    amount of cart items: {cart.itemAmount}
  </button>
));

ReactDOM.render(<CartView cart={myCart} />, document.body);
```

&nbsp;

### 2. Redux

1. 함수형 프로그래밍의 영향을 받음
2. 특정 UI 프레임워크에 종속되지 않음
3. 오랜 기간 사용되어 왔기 때문에 레퍼런스가 풍부
4. 상태 변경 추적이 용이
5. 보일러플레이트가 있을 수 밖에 없고, 초기에 학습 곡선이 가파름

```ts
import { createStore } from "redux";

function counter(state = 0, action) {
  switch (action.type) {
    case "PLUS":
      return state + 1;
    case "MINUS":
      return state - 1;
    default:
      return state;
  }
}

let store = createStore(counter);

store.subscribe(() => console.log(store.getState()));

store.dispatch({ type: "PLUS" }); // 1
store.dispatch({ type: "PLUS" }); // 2
store.dispatch({ type: "MINUS" }); // 1
```

&nbsp;

### 3. Recoil

1. 상태를 저장하는 Atom과 상태 변경 순수 함수 Selector로 상태를 관리
2. Redux에 비해 보일러플레이트가 적고 배우기 쉬움

상태를 공유하는 컴포넌트들은 `RecoilRoot`로 감싸주어야 한다.

```tsx
import React from "react";
import { RecoilRoot } from "recoil";
import { TextInput } from "./";

function App() {
  return (
    <RecoilRoot>
      <TextInput />
    </RecoilRoot>
  );
}
```

```tsx
import { atom } from "recoil";

export const textState = atom({
  key: "textState", // unique ID (with respect to other atoms/selectors)
  default: "", // default value (aka initial value)
});

import { useRecoilState } from "recoil";
import { textState } from "./";

export function TextInput() {
  const [text, setText] = useRecoilState(textState);
  const onChange = (event) => {
    setText(event.target.value);
  };
  return (
    <div>
      <input type="text" value={text} onChange={onChange} />
      <br />
      Echo: {text}
    </div>
  );
}

setInterval(() => {
  myCart.increase();
}, 1000);
```

&nbsp;

### 4. Zustand

1. Flux 패턴을 사용하며 보일러플레이트가 적고 훅 기반의 편리한 API 모듈 제공
2. 클로저를 활용하여 스토어 내부 상태를 관리해 특정 라이브러리에 종속되지 않음

```ts
import { create } from "zustand";

const useBearStore = create((set) => ({
  bears: 0,
  increasePopulation: () => set((state) => ({ bears: state.bears + 1 })),
  removeAllBears: () => set({ bears: 0 }),
}));

function BearCounter() {
  const bears = useBearStore((state) => state.bears);

  return <h1>{bears} around here ...</h1>;
}

function Controls() {
  const increasePopulation = useBearStore((state) => state.increasePopulation);
  return <button onClick={increasePopulation}>Plus</button>;
}
```

---

&nbsp;

💡 Flux 패턴: MVC 패턴의 한계를 극복하기 위해 페이스북에서 고안한 단방향 흐름 의 패턴

구성요소:

1. Action: 상태 변경을 위한 이벤트나 액션을 정의(Redux에서는 액션의 식별자 `type`과 실제 동작인 `action`로 구현)
2. Dispatcher: Action을 Model로 전달하는 중앙 허브
3. Model(Store): 애플리케이션의 상태와 로직을 보관
4. View: Model의 상태를 받아 화면에 표시

데이터 흐름:

1. View에서 사용자 입력이나 이벤트 발생
2. View는 Action Creator를 통해 Action 생성
3. Action은 Dispatcher를 통해 Store로 전달
4. Model은 Action에 따라 상태 업데이트
5. 변경된 상태가 View에 반영
