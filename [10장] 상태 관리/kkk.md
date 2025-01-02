```jsx
10.1 상태관리
10.1.1 상태
					지역 상태 / 전역 상태 / 서버 상태
10.1.2 상태를 잘 관리하기 위한 가이드
					시간이 지나도 변하지 않는다면 상태가 아니다
					파생된 값은 상태가 아니다
					useState vs useReduces, 어떤 것을 사용해야 할까
10.1.3 전역 상태 관리와 상태 관리 라이브러리
					컨텍스트 API
10.2 상태 관리 라이브러리
10.2.1 MobX
10.2.2 Redux
10.2.3 Recoil
10.2.4 Zustand
```

# 10.1 상태 관리

# 10.1.1 상태 (state)

> 상태는 렌더링에 영향을 줄 수 있는 동적인 데이터 값
렌더링 결과에 영향을 주는 정보를 담은 순수 자바스크립트 객체
> 

키워드 : 동적 데이터, 렌더링 결과에 영향

**상태종류**

지역 상태, 전역 상태, 서버 상태

**외부상태 라이브러리**

리덕스, MobX, 리코일

## 지역 상태 (local state)

- 컴포넌트 내부에서 사용되는 상태 (ex 체크박스의 체크 여부, 폼의 입력값 등)
- useState 훅 주로 사용, useReducer 같은 훅도 사용

## 전역 상태 (global state)

- 앱 전체에서 공유하는 상태
- prop drilling 문제 회피 용도로 사용 가능

## 서버 상태 (server state)

- 외부 서버에 저장해야 하는 상태 (ex 사용자 정보, 글 목록 등)
- UI 상태와 결합해 관리
- 로딩 여부, 에러 상태 등 포함

# 10.1.2 상태를 잘 관리하기 위한 가이드

상태가 업데이트되면 리렌더링이 발생 > 상태의 개수 최소화하는 것이 좋다.

가능하다면 stateless 컴포넌트를 활용하는 것이 좋다

❗**어떤 값을 상태로 정의할 때 고려할 2가지**

- 시간이 지나도 변하지 않는다면 상태가 아니다
- 파생된 값은 상태가 아니다.

## ❗시간이 지나도 변하지 않는다면 상태가 아니다

시간이 지나도 변하지 않는 값이라면, **객체 참조 동일성을 유지하는 방법**을 고려해라.

**개요**

1. 컴포넌트가 마운트될 때만 스토어 객체 인스턴스 생성, 컴포넌트가 언마운트될 때까지 해당 참조가 변하지 않는다.
2. 이를 상수 변수에 저장해 사용할 경우
    1. 렌더링될 때마다 새로운 객체 인스턴스 생성, 컨텍스트나 props 등으로 전달했을 때 매번 다른 객체로 인식 불필요한 리렌더링 발생.
3. 컴포넌트 라이프사이클 내에서 마운트될 때 인스턴스가 생성되고, 렌더링될 때마다 동일한 객체 참조가 유지되도록 구현해야 한다.
    1. how? 
        1. useState의 초깃값만 지정하는 방법
        2. useRef를 사용하는 방법

> **결론 : 객체 참조 동일성을 유지하려면 useRef를 사용하는 것이 적절하다**
> 

객체의 참조동일성을 유지하는 방법에는 메모이제이션이 있다. useMemo를 활용해 컴포넌트가 마운트될 때만 객체 인스턴스를 생성하고 이후 렌더링에서는 이전 인스턴스를 재활용할 수 있도록 구현할 수 있다.

그러나 리액트 공식 문서에서는 useMemo를 통한 메모이제이션은 의미상으로 보장된 것이 아니기 때문에 오로지 성능 향상을 위한 용도로만 사용해야 한다고 명시되어 있다. 즉 메모이제이션을 사용하지 않아도 돌아가는 코드를 작성하고 이후 성능 향상을 위해 메모이제이션을 추가하는 것이 적절한 접근 방식이다.

> 메모이제이션은 올바르게 동작하는 코드 위에, 성능 향상을 목적으로 추가되어야 한다.
> 

**다른방법 2가지**

- useState의 초깃값만 지정하는 방법
- useRef를 사용하는 방법

### ✅ useState의 초깃값만 지정하기

초깃값만 지정해 모든 렌더링 과정에서 객체 참조를 동일하게 유지할 수 있다.

```tsx
import {useState} from 'react'

const [store, setStore] = useState(newStore()); // 1번 즉시 실행
const [store] = useState(() => new Store()); // 2번 지연 초기화 방식
```

1번 방법은 컴포넌트가 매 렌더링마다 newStore()를 호출한다. store 값이 변경되지 않아도 함수가 실행되어 객체 인스턴스를 새로 생성한다.

2번 방법은 초기값을 계산하는 콜백 함수를 useState에 전달한다. 리액트는 이 콜백을 단 한 번 호출해 초기값을 생성하고 이후에는 렌더링이 반복되어도 동일한 초기값을 유지한다.

**한계**

useState를 사용하는 것은 기술적으로는 잘 동작하나, 의미론적으로 봤을 때는 좋은 방법이 아니다. 

useState는 원래 **렌더링과 관련된 상태**를 관리하기 위한 도구인데 여기서의 목적은 객체의 참조를 유지하려는 것이기 때문에 기존 목적과 다르게 사용되고 있다.

### ✅ useRef를 사용하는 방법

> useRef()와 { current : … } 객체를 직접 생성하는 방법간의 유일한 차이는 useRef는 매번 렌더링할 때마다 동일한 ref 객체를 제공한다는 것이다.
> 

동일한 객체 참조 유지가 목적이라면 useRef가 가장 적합하다. 

```tsx
import {useRef} from 'react';

const store = useRef<Store>(null); // store 객체 생성, 초기값 null

if (!store.current) {
	// 최초 렌더링, 즉 마운트될 때만 이 조건문 실행. 이후 실행되지 않음
  store.current = new Store(); // 새로운 Store 인스턴스를 생성하여 store.current에 할당
}
```

useRef는 기술적으로 `useState ({ chileren : initialValue})[0]` 과 동일하다.

그러나 상태는 렌더링에 영향을 주며 변화하는 동적인 값을 의미한다. 

그런 관점에서 객체 참조 동일성을 유지하기 위해 useState에 초깃값만 할당하는 것은 적절하지 않다. 그러므로 객체 참조를 할 때는 useRef 사용을 권장한다.

## ❗파생된 값은 상태가 아니다

부모에게서 전달받을 수 있는 props이거나 기존 상태에서 계산될 수 있는 값은 상태가 아니다. 

> SSOT(single source of truth)는 어떠한 데이터도 단 하나의 출처에서 생성하고 수정해야 한다는 원칙을 의미하는 방법론이다.
> 

리액트 앱에서 상태를 정의할 때도 이를 고려해야 한다. 다른 값에서 파생된 값을 상태로 관리하게 되면 기존 출처와는 다른 새로운 출처에서 관리하게 되는 것이므로 해당 데이터의 정확성과 일관성을 보장하기 어렵다.

> 부모에게서 props로 전달받으면 상태가 아니다.
> 

### **예제1 : 상태 끌어올리기로 SSOT 지키기**

초기 이메일 값을 부모 컴포넌트로부터 받아 input value로 렌더링하고 이후에는 사용자가 입력한 값을 input 태그의 value로 렌더링한다. 어떤 문제가 있을까?

```tsx
import React, { useState } from 'react';

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
      <input type='text' value={email} onChange={onChangeEmail} />
    </div>
  );
};
```

**문제**

initialEmail이 변경되어도 useState로 초깃값이 세팅된 email은 변경되지 않는다. 

- useState의 초깃값으로 설정한 값은 컴포넌트가 마운트될 때 한 번만 email 상태의 값으로 설정되며 이후에는 독자적으로 관리된다.

그렇다면 useEffect로 렌더링마다 업데이트해주기? 그러면 사용자가 값을 변경한 뒤(onChangeEmail) props가 변경되면 입력이 무시될 수도 있다.

useEffect를 사용한 동기화 작업은 리액트 외부 데이터(localStorage 등)와 동기화할 때만 사용해야 한다. 내부에 존재하는 데이터를 상태와 동기화하는데 사용하면 안된다. 왜? 내부 상태를 useEffect로 동기화하면 개발자가 추적하기 어려운 오류가 발생할 수 있기 때문에 피해야 한다!

```tsx
import {useState, useEffect} from 'react';

const [email, setEmail] = useState(initialEmail);

// 이러면 안돼!!!!!
useEffect(() => {
  setEmail(initialEmail);
}, [initialEmail]);
```

**어떻게 해결?**

> 현재 상황 요약 : 부모에게 받은 props를 자식 컴포넌트에서 input의 초기값으로 사용 중이다. useState의 초기값으로 부모 props 값을 줬는데 부모에서 받은 props가 업데이트되면 해당 state도 업데이트되어야 하는데 이 state는 사용자 입력으로도 업데이트된다.
> 

현재 email 상태의 출처는 2가지인데 prop으로 받는 값과 useState로 생성한 email state이다. 

문제를 해결하려면 출처를 한 곳으로 통일해줘야 한다. (위의 useEffect를 사용해서 내부적으로 동기화시키는 방법은 여전히 2개의 출처를 유지하므로 적절하지 않다! 출처를 통일하는 방법으로 접근)

이 때 사용하는 방법 **“상태 끌어올리기(lifting state up) 기법” > 이를 통해 SSOT를 지키도록 해야한다!**

이는 상위 컴포넌트에서 상태를 관리하도록 해준다. 상태를 부모 컴포넌트로 옮긴다. 이러면 자식 컴포넌트에서 useState로 관리하지 않게 된다.

```tsx
import React, { useState } from 'react';

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
      <input type='text' value={email} onChange={onChangeEmail} />
    </div>
  );
};
```

### **예제2 : 여러 출처를 하나의 출처로 합치기**

아이템 목록과 선택된 아이템 목록을 가지고 있는 코드다. 아이템 목록이 변경될 때마다 선택된 아이템 목록을 가져오기 위해 useEffect로 동기화작업을 한다. 코드의 문제는?

```tsx
import {useState, useEffect} from 'react';

const [items, setItems] = useState<Item[]>([]);
const [selectedItems, setSelectedItems] = useState<Item[]>([]);

useEffect(() => {
  setSelectedItems(items.filter((item) = > item.isSelected));
}, [items]);
```

**성능 문제는?**

- items의 값이 바뀌며 렌더링 발생
- items의 값 변경을 감지하고 selectedItems 값을 변경하며 리렌더링 발생

**문제**

1. items와 selectedItems가 동기화되지 않을 수 있다. 
2. 새로운 상태로 정의하면서 출처가 여러 개가 되었다. 동기화 문제 발생 가능

**❓여러 출처를 하나의 출처로 합치려면 어떻게 해야 할까?**

1. 초간단 방법 : 상태로 정의하지 않고 계산된 값을 자바스크립트 변수로 담는다. 그러면 items가 변경될 때마다 컴포넌트가 새로 렌더링되고 이 때마다 selectedItems를 다시 계산한다.
    
    ```tsx
    import {useState} from 'react';
    
    const [items, setItems] = useState<Item[]>([]);
    const selectedItems = items.filter((item) = > item.isSelected);
    ```
    
    **이 방법의 성능 문제는?**
    
    재렌더링 횟수를 줄일 수 있다. 다만 렌더링마다 계산을 수행해야 하므로 렌더링 횟수는 줄지만 계산 비용은 추가된다. 계산 비용이 클 경우에는 useMemo를 사용해 items가 변경될 때만 계산을 수행하고 결과를 메모이제이션하여 성능을 개선할 수 있다.
    
    ```tsx
    import {useState, useMemo} from 'react';
    
    const [items, setItems] = useState<Item[]>([]);
    const selectedItems = useMemo(() => veryExpensiveCalculation(items), [items]);
    ```
    

## ☑️ useState vs useReducer, 어떤 것을 사용해야 할까

useReducer 사용을 권장하는 2가지 경우

- 다수의 하위 필드를 포함하고 있는 복잡한 상태 로직을 다룰 때
- 다음 상태가 이전 상태에 의존적일 때

예시 : 배민 리뷰 리스트를 필터링해 보여주기 위한 쿼리를 상태로 저장해야 한다. 이 쿼리는 복잡하고 날짜, 점수, 키워드, 페이지네이션(페이지, 사이즈) 등 많은 하위 필드를 가진다.

```tsx
// 날짜 범위 기준 - 오늘, 1주일, 1개월
type DateRangePreset = 'TODAY' | 'LAST_WEEK' | 'LAST_MONTH';
type ReviewRatingString = '1' | '2' | '3' | '4' | '5';
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

이런 복잡한 데이터 구조를 useState로 다룰 때 문제

- 상태 업데이트마다 오류 가능성이 증가한다 - 다른 필드 수정 가능성 등 오류 가능성
- 특정 업데이트 규칙이 있다면 useState로는 한계가 있다.

useReducer는 “무엇을” 변경할지와 “어떻게“ 변경할지를 분리한다. dispatch를 통해 어떤 작업을 할지를 액션으로 넘기고 reducer 함수 내에서 상태를 업데이트하는 방식을 정의한다. 

⏩ 복잡한 상태 로직을 숨기고 안전성을 높인다. 

```tsx
import React, { useReducer } from 'react';

// Action 정의
type Action =
| { payload: ReviewFilter; type: 'filter'; }
| { payload: number; type: 'navigate'; }
| { payload: number; type: 'resize'; };
// Reducer 정의
const reducer: React.Reducer<State, Action> = (state, action) => {
  switch (action.type) {
    case 'filter':
      return {
        filter: action.payload,
        page: 0,
        size: state.size,
      };
    case 'navigate':
      return {
        filter: state.filter,
        page: action.payload,
        size: state.size,
      };
    case 'resize':
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
dispatch({ payload: filter, type: 'filter' });
dispatch({ payload: page, type: 'navigate' });
dispatch({ payload: size, type: 'resize' });
```

이외에도 boolean 상태를 토글하는 액션만 사용하는 경우에도 useState 대신 useReducer를 사용할 수 있다.

```tsx
import { useReducer } from 'react';

//Before
const [fold, setFold] = useState(true);

const toggleFold = () => {
  setFold((prev) => !prev);
};

// After
const [fold, toggleFold] = useReducer((v) => !v, true);
```

# 10.1.3 전역 상태 관리와 상태 관리 라이브러리

> 사애는 사용하는 곳과 최대한 가까워야 하며 사용 범위를 제한해야 한다.
> 

**전역 상태 사용법 2가지**

1. 리액트 컨텍스트 API + useState 또는 useReducer
2. 외부 상태 라이브러리

## ✅ 컨텍스트 API(Context API)

전역적으로 공유해야 하는 데이터는 컨텍스트로 제공하고 해당 컨텍스트를 “구독”한 컴포넌트에서만 데이터를 읽을 수 있다. 

문제 : 부모 컴포넌트에서 자식 컴포넌트에게 프롭을 넘겨줄 때 모든 자식마다 프롭을 인자로 받지 않고 해당 부모의 자식들은 인자로 넘겨주지 않아도 해당 프롭을 사용할 수 있도록 하고 싶다.

```tsx
// 현재 구현된 것 - TabGroup 컴포넌트뿐 아니라 모든 Tab 컴포넌트에도 type prop을 전달
<TabGroup type='sub'>
  <Tab name='텝 레이블 1' type='sub'>
    <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2' type='sub'>
    <div>123</div>
  </Tab>
</TabGroup>

// 원하는 것 - TabGroup 컴포넌트에만 전달
<TabGroup type='sub'>
  <Tab name='텝 레이블 1'>
   <div>123</div>
  </Tab>
  <Tab name='텝 레이블 2'>
    <div>123</div>
  </Tab>
</TabGroup>
```

**어떻게?**

상위 컴포넌트 구현 부에 컨텍스트 프로바이더 context propvider를 넣어주고 하위 컴포넌트에서 해당 컨텍스트를 구독해 데이터를 읽어오도록 한다.

```tsx
import {FC} from 'react';

const TabGroup: FC<TabGroupProps> = (props) => {
  const { type = 'tab', ...otherProps } = useTabGroupState(props);
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

**컨텍스트 API 유틸리티 함수로 정의하기**

유틸리티 함수로 정의해서 더 간단하게 컨텍스트와 훅을 생성할수 있다.

```tsx
// Example
interface StateInterface {}
const [context, useContext] = createContext<StateInterface>();
```

```tsx
import React from 'react';

type Consumer<C> = () => C;

export interface ContextInterface<S> {
  state: S;
}

export function createContext<S, C = ContextInterface<S>>(): readonly [React.FC<C>, Consumer<C>] {
  const context = React.createContext<Nullable<C>>(null);

  const Provider: React.FC<C> = ({ children, ...otherProps }) => {
    return (
      <context.Provider value= {otherProps as C}>{children}</context.Provider>
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
```

**컨텍스트 API + useState 또는 useReducer**

컨텍스트 API는 엄밀히 말해 전역 상태를 관리하기 위한 솔루션이라기보다 여러 컴포넌트 간에 값을 공유하는 솔루션에 가깝다. 그러나 useState나 useReducer 같이 지역 상태를 관리하기 위한 api와 결합해 여러 컴포넌트 사이에서 상태를 공유하기 위한 방법으로 사용될 수 있다.

```tsx
import { useReducer } from 'react';

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

**컨텍스트 API의 한계**

대규모 애플리케이션이나 성능이 중요한 애플리케이션에서 권장되지 않는 방법이다. 이는 컨텍스트 프로바이더의 props로 주입된 값이나 참조가 변경될 때마다 해당 컨텍스트를 구독하는 모든 컴포넌트가 재렌더링되기 때문이다.

# 10.2 상태 관리 라이브러리

## ✅ 외부 상태 라이브러리 : MobX

객체 지향 프로그래밍과 반응형 프로그래밍 패러다임의 영향을 받은 라이브러리다. 

상태 변경 로직을 단순하게 작성할 수 있고, 복잡한 업데이트 로직을 라이브러리에 위임할 수 있다.

객체 지향 스타일로 코드를 작성한다면 편리할 것

**한계**

데이터가 언제, 어떻게 변하는지 추적하기 어렵기 때문에 트러블슈팅에 어려움을 겪을 수 있다.

- 예제
    
    ```tsx
    import { observer } from 'mobx-react-lite';
    import { makeAutoObservable } from 'mobx';
    
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
    
    ReactDOM.render(<CartView cart= {myCart} />, document.body);
    ```
    

## ✅ 외부 상태 라이브러리 : Redux

함수형 프로그래밍의 영향을 받은 라이브러리다. 특정 UI 프레임워크에 종속되지 않아 독립적으로 상태 관리 라이브러리를 사용할 수 있다. 상태 변경 추적에 최적화되어 있어 원인 파악에 용이한다.

**한계**

단순한 상태 설정에도 많은 보일러 플레이트가 필요하다. 사용 난이도가 높다.

- 예제
    
    ```tsx
    import { createStore } from 'redux';
    
    function counter(state = 0, action) {
      switch (action.type) {
        case 'PLUS':
          return state + 1;
        case 'MINUS':
          return state - 1;
        default:
          return state;
      }
    }
    
    let store = createStore(counter);
    
    store.subscribe(() => console.log(store.getState()));
    
    store.dispatch({ type: 'PLUS' });
    // 1
    store.dispatch({ type: 'PLUS' });
    // 2
    store.dispatch({ type: 'MINUS' });
    // 1
    ```
    

## ✅ 외부 상태 라이브러리 : Recoil

상태를 저장할 수 있는 Atom과 해당 상태를 변형할 수 있는 순수 함수 셀렉터를 통해 상태를 관리하는 라이브러리다. 리덕스에 비해 보일러플레이트가 적고 난이도가 쉬워 배우기 쉽다. 리코일 상태를 공유하기 위해 컴포넌트들은 RecoilRoot 하위에 위치해야 한다.

**한계**

아직 실험적인 상태라 다양한 요구사항에 대해 충분한 검증이 이뤄지지 않았다.

- 예제
    
    ```tsx
    import React from 'react';
    import { RecoilRoot } from 'recoil';
    import { TextInput } from './';
    
    function App() {
      return (
        <RecoilRoot>
          <TextInput />
        </RecoilRoot>
      );
    }
    ```
    
    ```tsx
    import { atom } from 'recoil';
    
    export const textState = atom({
      key: 'textState', // unique ID (with respect to other atoms/selectors)
      default: '', // default value (aka initial value)
    });
    
    import { useRecoilState } from 'recoil';
    import { textState } from './';
    
    export function TextInput() {
      const [text, setText] = useRecoilState(textState);
      const onChange = (event) => {
        setText(event.target.value);
      };
      return (
        <div>
          <input type='text' value= {text} onChange= {onChange} />
          <br />
          Echo: {text}
        </div>
      );
    }
    
    setInterval(() => {
      myCart.increase();
    }, 1000);
    ```
    

## ✅ 외부 상태 라이브러리 : Zustand

**“flux 패턴”**을 사용해 많은 보일러플레이트를 가지지 않는 훅 기반의 편리한 api 모듈을 제공한다. 클로저를 활용해 스토어 내부 상태를 관리함으로써 특정 라이브러리에 종속되지 않는 특징이 있다.

- flux 패턴
    
    MVC 패턴은 양방향으로 데이터가 흐르므로 규모가 커지면 매우 복잡해진다. 이를 해결하기 위해 등장한 flux 패턴은 단방향 데이터 흐름을 가진다.
    
    사용자 입력을 기반으로 액션을 만들고 액션을 디스패처로 전달해 스토어의 데이터를 변경. 이를 뷰에 반영한다.
    
    > action >>> dispatcher >>> model >>> view
    > 
    
    뷰로부터 새로운 데이터 변경이 생기면 처음부터 다시 이 순서를 실행한다. 
    

상태와 상태를 변경하는 액션을 정의하고 반환된 훅을 어느 컴포넌트에서나 import하여 원하는 대로 사용할 수 있다.

- 예제
    
    ```tsx
    import { create } from 'zustand';
    
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
    

**우형에서는 어떤 상태 관리 라이브러리를 사용할까?**

- 상태 관리 라이브러리를 선정하는 기준으로 보일러 플레이트가 적고, 코드 작성에 너무 많은 요소가 들어가지 않는 것을 정했다. 리코일이 좋았다. 러닝 커브 낮음
- 리코일이 좋았다 의견 다수

❔왜 프로젝트에 상태 관리 라이브러리가 필요하다고 생각했나요? 앞으로도 계속 상태 관리 라이브러리가 필요할까요?

- 컴포넌트 구조를 설계하고 만들면 계단식 형태의 트리가 생긴다. 이 리액트적인 동작구조와 데이터의 흐름이 항상 일치하지 않는다. 이를 해소하기 위해 상태 관리 라이브러리가 있다고 생각
- 리액트는 ui 그리는 것에 관심이 있는 거고 데이터를 컨트롤하는 건 관심이 없으니 상태 관리 라이브러리 필요.
- prop 드릴링 피하는 게 크다. 여러 컴포넌트에서 같은 상태를 쓸 때 또 필요하고, 하지만 모든 리액트 프로젝트에서 상태 관리가 필요하다고 생각하지는 않는다.