```jsx
9.1 리액트 훅
9.1.1 useState
9.1.2 의존성 배열을 사용하는 훅
9.1.3 useRef

9.2 커스텀 훅
9.2.1 나만의 훅 만들기
9.2.2 타입스크립트로 커스텀 훅 강화하기
```

# 9.1 리액트 훅

클래스 컴포넌트에서는 하나의 생명주기에서만 상태 업데이트에 대한 로직을 실행시킬 수 있다. 즉 여러 상태를 하나의 함수에서 처리하다보니 디버깅이 어렵고 여러 문제가 생길 여지가 많았다.

```tsx
componentDidMount() {
  this.props.updateCurrentPage(routeName);
  this.didFocusSubscription = this.props.navigation.addListener('focus', () => {/*
add focus handler to navigation */});
  this.didBlurSubscription = this.props.navigation.addListener('blur', () => {/* add
blur handler to navigation */});
}

componentWillUnmount() {
  if (this.didFocusSubscription != null) {
    this.didFocusSubscription();
  }
  if (this.didBlurSubscription != null) {
    this.didBlurSubscription();
  }
  if (this._screenCloseTimer != null) {
    clearTimeout(this._screenCloseTimer);
    this._screenCloseTimer = null;
  }
}

componentDidUpdate(prevProps) {
  if (this.props.currentPage != routeName) return;

  if (this.props.errorResponse != prevProps.errorResponse) {/* handle error response
*/}
  else if (this.props.logoutResponse != prevProps.logoutResponse) {/* handle logout
response */}
  else if (this.props.navigateByType != prevProps.navigateByType) {/* handle
navigateByType change */}

  // Handle other prop changes here
}
```

리액트 훅이 도입되면서 더 작은 단위로 코드를 분할해 작성할 수 있게 되었다. 

# 9.1.1 useState

리액트 함수 컴포넌트에서 상태를 관리하기 위해 useState 훅을 활용할 수 있다.

**useState 타입 정의**

```tsx
function useState<S>(
  initialState: S | (() => S)
  ): [S, Dispatch<SetStateAction<S>>];
  
type Dispatch<A> = (value: A) => void;
type SetStateAction<S> = S | ((prevState: S) => S); // 유니온
// 새로운 상태 타입 S,  또는 이전 상태 값을 받아 새로운 상태 리턴 (불변성 보장)
```

☑️ **리턴 튜플**

`[S, Dispatch<SetStateAction<S>]`

1. 제네릭 S 타입
2. 상태를 업데이트할 수 있는 디스패치 타입 함수

## ✅ useState에 타입스크립트 적용하기

문제 : memberList에 새로운 멤버 객체를 추가할 때 문제 발생. 기존 배열 요소에는 없는 “agee”라는 잘못된 속성이 포함된 객체가 추가되었다. 이로 인해 sumAge 변수가 NaN이 되는 사이드 이펙트가 발생한다. 이러한 문제 때문에 기능을 추가하거나 수정할 때 해당 컴포넌트에서 다루는 상태 타입을 모른다면 치명적인 에러가 발생할 수 있다. 

그러나? 타입스크립트를 사용하면 이런 에러를 사전에 방지할 수 있다! 

```tsx
import { useState } from "react";

const MemberList = () => {
  const [memberList, setMemberList] = useState([
    {
      name: "KingBaedal",
      age: 10,
      },
    {
      name: "MayBaedal",
      age: 9,
    },
  ]);
  
  // 🚨 addMember 함수를 호출하면 sumAge는 NaN이 된다
  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);
  
  const addMember = () => {
    setMemberList([
      ...memberList,
      {
        name: "DokgoBaedal",
        agee: 11,
      },
    ]);
  };
};
```

setMembeerList의 호출 부분에서 추가하려는 새 객체의 타입을 확인해서 컴파일 타임에 타입에러가 발생하도록 할 수 있다.

```tsx
import { useState } from "react";

interface Member {
  name: string;
  age: number;
}

const MemberList = () => {
  const [memberList, setMemberList] = useState<Member[]>([]);
  
  // member의 타입이 Member 타입으로 보장된다
  const sumAge = memberList.reduce((sum, member) => sum + member.age, 0);
  
  const addMember = () => {
  // 🚨 Error: Type ‘Member | { name: string; agee: number; }’
  // is not assignable to type ‘Member’
    setMemberList([
      ...memberList,
      {
        name: "DokgoBaedal",
        agee: 11,
      },
    ]);
  };

  return (
    // ...
  );
};
```

# 9.1.2 의존성 배열을 사용하는 훅

## ☑️ useEffect는 뭐지

```tsx
// useEffect 타입 정의
function useEffect(effect: EffectCallback, deps?: DependencyList): void;
type DependencyList = ReadonlyArray<any>;
type EffectCallback = () => void | Destructor;
```

EffectCallback은 Destructor을 반환하거나 void, 아무것도 반환하지 않는 함수이다. Promise 타입은 반환하지 않는다. useEffect의 콜백함수에는 비동기 함수가 들어갈 수 없다. useEffect에서 비동기 함수를 호출할 수 있다면 경쟁  상태(race condition)을 불러일으킬 수 있기 때문이다.

> **“경쟁 상태”**란 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제다. 이러한 상황에서 실행 순서나 타이밍을 예측할 수 없게 되어 프로그램 동작이 원하지 않는 방향으로 흐를 수 있다.
> 

두번째 인자인 deps는 옵셔널하게 제공되고 effect가 수행되기 위한 조건을 나열한다. (deps 배열의 원소가 변경되면 실행한다 등) 다만 deps의 원소로 숫자나 문자열 같은 타입스크립트 기본 자료형이 아닌 객체나 배열을 넣을 때는 주의해야 한다.

**deps 얕은 비교**

useEffect는 deps가 변경되었는지를 얕은 비교로만 판단하기 때문에, 실제 객체 값이 바뀌지 않았더라도 객체의 참조 값이 변경되면 콜백 함수가 실행된다. 즉 아래 예시처럼 부모에서 받은 인자를 직접 deps로 작성한 경우 원치 않는 렌더링이 반복될 수 있다. 이를 방지하기 위해 실제로 사용하는 값을 deps에서 사용해야 한다.

> **“얕은 비교”**는 객체나 배열과 같은 복합 데이터 타입을 비교할 때 내부의 각 요소나 속성을 재귀적으로 비교하지 않고, 해당 값들의 참조나 기본 타입 값만을 간단하게 비교하는 것을 말한다.
> 

```tsx
type SomeObject = {
  name: string;
  id: string;
};

interface LabelProps {
  value: SomeObject;
}

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
    // value.name과 value.id를 사용해서 작업한다
  }, [value]);
  // 부모에게 받은 props를 의존성 배열에 넣으면 안된다. 왜?
  // ...
};
```

❓**부모에게 받은 props를 의존성 배열에 넣으면 안된다. 왜?**

의존성 배열에서는 얕은 비교를 하므로 내용물이 아닌 참조 주소값이 변하면 그때마다 재렌더링을 한다. 의도치 않은 렌더링이 많이 발생할 수 있다. useMemo로 참조를 고정하거나 실제로 사용하는 값을 의존성 배열에 넣는 것이 적절하다.

```tsx
const { id, name } = value;

useEffect(() => {
  // value.name과 value.id 대신 name, id를 직접 사용한다
}, [id, name]);
```

**destructor**

해당 함수는 컴포넌트가 마운트 해제될 때 실행하는 함수인데 이 말은 어느 정도만 맞다.

deps가 빈 배열이라면 useEffect의 콜백 함수는 컴포넌트가 처음 렌더링될 때만 실행되고 이 때의 destructor=클린업 함수는 컴포넌트가 마운트 해제될 때 실행된다.

deps 배열이 존재한다면 배열의 값이 변경될 때마다 클린업 함수가 실행된다.

> “클린업 함수”는 useEffect나 useLayoutEffect와 같은 리액트 훅에서 사용되며, 컴포넌트가 해제되기 전에 정리=클린업 작업을 수행하기 위한 함수이다.
> 

## ☑️ useLayoutEffect는 뭐지

```tsx
// useLayoutEffect의 타입 정의 - useEffect와 동일, 역할의 차이
type DependencyList = ReadonlyArray<any>;

function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;
```

useEffect는 앞의 생명주기 함수 componentDidUpdate와 다르게, 레이아웃 배치와 화면 렌더링이 모두 완료된 후에 실행된다.

**useEffect의 실행순서 문제, useLayoutEffect의 사용목적**

아래 코드를 실행하면 출력이 name이 처음에는 공백이었다가 이후에 name이 들어가서 렌더링될 것이다. 이런 상황에서 useLayoutEffect를 사용할 수 있다.

**useLayoutEffect를 사용하면 화면에 해당 컴포넌트가 그려지기 전에 콜백함수를 실행한다. 첫번째 렌더링 때 빈 이름이 뜨지 않는다.**

```tsx
const [name, setName] = useState("");

useEffect(() => {
  // 매우 긴 시간이 흐른 뒤 아래의 setName()을 실행한다고 생각하자
  setName("배달이");
}, []);

return (
  <div>
    {`안녕하세요, ${name}님!`}
  </div>
);
```

## ☑️ useMemo와 useCallback

둘 다 이전에 생성된 값 또는 함수를 기억하며, 동일한 값과 함수를 반복해서 생성하지 않도록 해주는 후이다. 어떤 값을 계산하는데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 유용하게 사용할 수 있다.

```tsx
type DependencyList = ReadonlyArray<any>;

function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(callback: T, deps: DependencyList): T;
```

두 훅 모두 제네릭을 지원하기 위해 T 타입을 선언한다. useCallback은 함수 타입만 와야 해서 제네릭을 제한하고 있다. 

두 훅 모두 deps 배열을 가지고 있으며 해당 의존성이 변경되면 값을 다시 계산한다. 이전처럼 얕은 비교를 수행하기 때문에 주의해야 한다.

불필요한 곳까지 사용해서 과도하게 메모이제이션하면 컴포넌트의 성능 향상이 보장되지 않을 수 있다.

> “메모이제이션”은 이전에 계산한 값을 저장함으로써 같은 입력에 대한 연산을 다시 수행하지 않도록 최적화하는 기술이다.
> 

# 9.1.3 useRef

input 요소에 포커스를 설정하거나 특정 컴포넌트의 위치로 스크롤을 하는 등 DOM을 직접 선택해야 하는 경우에 사용한다.

**예시: 버튼 누르면 ref에 저장된 input DOM에 포커스를 설정**

useRef의 제네릭 타입에 null도 함께 유니온해주지 않았다. 그런데 초기값으로는 null이 잘 들어간다. 왜?

```tsx
import { useRef } from "react";

const MyComponent = () => {
  const ref = useRef<HTMLInputElement>(null);
  
  const onClick = () => {
    ref.current?.focus();
  };
  
  return (
    <>
      <button onClick= {onClick}>ref에 포커스!</button>
      <input ref= {ref} />
    </>
  );
};

export default MyComponent;
```

**useRef 타입 정의 3가지**

useRef는 세 종류의 타입 정의를 가진다. 인자 타입에 따라 반환되는 타입이 다르다.

```tsx
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
}

interface RefObject<T> {
  readonly current: T | null; // 읽기만 가능
}
```

useRef는 `MutableRefObject` 또는 `RefObject`를 반환한다.

MutableRefObject의 current는 값을 변경할 수 있다.

**1번 타입**

초기값을 반드시 제공해야 하며 초기값의 타입은 제네릭 T이다. 반환 타입은 뮤터블로 current 값을 변경할 수 있다.

```tsx
// function useRef<T>(initialValue: T): MutableRefObject<T>;
const ref = useRef<HTMLInputElement | null>(null);
```

이렇게 넣어주었다면 첫 번째 타입 정의를 따른다. 이 때 current는 변경할 수 있는 값이 되어 ref.current의 값이 바뀌는 사이드 이펙트가 발생할 수 있다.

**2번 타입**

초기값으로 null을 허용하는 경우 리턴 타입은 레프객체다. 반환된 current는 리드온니다.

```tsx
// function useRef<T>(initialValue: T | null): RefObject<T>;
const ref = useRef<HTMLInputElement>(null);
```

**3번 타입**

초기값을 제공하지 않는 경우다. 리턴 타입은 뮤터블. current는 undefined로 초기화되고 이후 변경가능하다.

```tsx
function useRef<T = undefined>(): MutableRefObject<T | undefined>;
```

## ☑️ 자식 컴포넌트에 ref 전달하기

리액트 컴포넌트에 ref를 전달할 수도 있다. 그러나 이 때 ref를 일반적인 props 넘겨주는 방식으로 전달하면 브라우저에서 경고 메시지를 띄운다.

```tsx
import { useRef } from "react";

const Component = () => {
  const ref = useRef<HTMLInputElement>(null);
  return <MyInput ref= {ref} />;
};

interface Props {
  ref: RefObject<HTMLInputElement>;
}

/**
  * 🚨 Warning: MyInput: `ref` is not a prop. Trying to access it will result in
  `undefined` being returned
  * If you need to access the same value within the child component, you should pass
  it as a different prop
  */
const MyInput = ({ ref }: Props) => {
  return <input ref= {ref} />;
};
```

ref라는 속성의 이름은 리액트에서 “DOM 요소 접근”이라는 특수 목적으로 사용되기 때문에 props를 넘겨주는 방식으로 전달할 수 없다. prop으로 전달하려면 “forwardRef”를 사용해야 한다.

> ref가 아닌 inputRef 등의 다른 이름을 사용한다면 forwardRef를 사용하지 않아도 된다.
> 

forwardRef의 두번째 인자에 ref를 넣어 자식 컴포넌트로 ref를 전달할 수 있다.

```tsx
interface Props {
  name: string;
}

const MyInput = forwardRef<HTMLInputElement, Props>((props, ref) => {
  return (
    <div>
      <label>{props.name}</label>
      <input ref= {ref} />
    </div>
  );
});
```

**forwardRef 타입정의**

```tsx
function forwardRef<T, P = {}>(
  render: ForwardRefRenderFunction<T, P>
): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;
```

**forwardRef에 인자로 넘겨주는 콜백함수의 타입 정의**

두 개의 타입 매개변수 T, P를 받는다.

P는 일반적인 리액트 컴포넌트에서 자식 컴포넌트로 넘겨주는 props의 타입을

T는 ref로 전달하려는 요소의 타입이다.

ref의 타입이 T를 래핑한 형태인 ForwardedRef<T>이다.

```tsx
interface ForwardRefRenderFunction<T, P = {}> {
  (props: P, ref: ForwardedRef<T>): ReactElement | null;
  displayName?: string | undefined;
  defaultProps?: never | undefined;
  propTypes?: never | undefined;
}
```

**ForwardedRef 타입 정의**

useRef의 반환타입은 뮤터블이나 레프만 된다고 했었다. 그러나 ForwardedRef에는 뮤터블만 가능하다. 뮤터블과 레프 중 뮤터블이 더 넓은 범위의 타입을 가지기 때문에 이후 레프에 둘 중 어느 것이 와도 수용 가능하다.

```tsx
type ForwardedRef<T> =
| ((instance: T | null) => void)
| MutableRefObject<T | null>
| null;
```

## ☑️ useImperativeHandle는 뭔가

ForwardRefRenderFunction과 함께 쓸 수 있는 훅이다. 이 훅을 활용하면 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스터마이징된 메서드를 호출할 수 있게 된다. 이에 따라 자식 컴포넌트는 내부 상태나 로직을 관리하면서 부모 컴포넌트의 결합도도 낮출 수 있다.

```tsx
// <form> 태그의 submit 함수만 따로 뽑아와서 정의한다
type CreateFormHandle = Pick<HTMLFormElement, "submit">;

type CreateFormProps = {
  defaultValues?: CreateFormValue;
};

const JobCreateForm: React.ForwardRefRenderFunction<CreateFormHandle,
  CreateFormProps> = (props, ref) => {
  // useImperativeHandle Hook을 사용해서 submit 함수를 커스터마이징한다
  useImperativeHandle(ref, () => ({
    submit: () => {
      /* submit 작업을 진행 */
    }
  }));
  
  // ...
}
```

자식 컴포넌트에서는 ref와 정의된 CreateFormHandle를 통해 부모 컴포넌트에서 호출할 수 있는 함수를 생성하고, 부모 컴포넌트에서는 다음처럼 current.submit()을 사용하여 자식 컴포넌트의 특정 메서드를 실행할 수 있게 된다.

```tsx
const CreatePage: React.FC = () => {
  // `CreateFormHandle` 형태를 가진 자식의 ref를 불러온다
  const refForm = useRef<CreateFormHandle>(null);
  
  const handleSubmitButtonClick = () => {
    // 불러온 ref의 타입에 따라 자식 컴포넌트에서 정의한 함수에 접근할 수 있다
    refForm.current?.submit();
  };
  
  // ...
};
```

## ☑️ useRef의 여러 가지 특정

useRef는 자식 컴포넌트를 저장하는 변수로 활용할 수 있을 뿐만 아니라 다른 방식으로도 유용하게 사용할 수 있다.

- useRef로 관리되는 변수는 값이 바뀌어도 컴포넌트의 재렌더링이 발생하지 않는다.
- 리액트 컴포넌트의 상태는 상태 변경 함수를 호출하고, 렌더링된 이후에 업데이트된 상태를 조회할 수 있다. 반면 useRef로 관리되는 변수는 값을 설정한 후 즉시 조회할 수 있다.

isAutoPlayPause는 현재 자동 재생이 일시 정지 되었는지 확인하는 ref다. 이 변수는 렌더링에 영향을 미치지 않으며, 값이 변경되더라도 다시 렌더링을 기다릴 필요 없이 사용할 수 있어야 한다. isAutoPlayPause.current에 null이 아닌 값을 할당해서 변수처럼 활용할 수 있다.

```tsx
type BannerProps = {
  autoplay: boolean;
};

const Banner: React.FC<BannerProps> = ({ autoplay }) => {
  const isAutoPlayPause = useRef(false);
  
  if (autoplay) {
    // keepAutoPlay 같이 isAutoPlay가 변하자마자 사용해야 할 때 쓸 수 있다
    const keepAutoPlay = !touchPoints[0] && !isAutoPlayPause.current;

    // ...
  }
  return (
    <>
      {autoplay && (
        <button
          aria-label="자동 재생 일시 정지"
          // isAutoPlayPause는 사실 렌더링에는 영향을 미치지 않고 로직에만 영향을 주므로 상태
          로 사용해서 불필요한 렌더링을 유발할 필요가 없다
          onClick={() => { isAutoPlayPause.current = true }}
        />
      )}
    </>
  );
};

const Label: React.FC<LabelProps> = ({ value }) => {
  useEffect(() => {
    // value.name과 value.id를 사용해서 작업한다
  }, [value]);
  
  // ...
};
```

<aside>
🔬

**훅의 규칙**

리액트 훅을 안전하게 사용하기 위해 다음 2가지 규칙을 지켜야 한다. 리액트는 이러한 규칙을 준수할 수 있도록 도와주는 lint 플러그인도 제공한다.

첫째, 훅은 항상 최상위 레벨에서 호출되어야 한다. 다시 말해 조건문, 반복문, 중첩 함수, 클래스 등의 내부에서는 훅을 호출하지 않아야 한다. 

반환문으로 함수 컴포넌트가 종료되거나, 조건문 또는 변수에 따라 반복문 등으로 훅의 호출 여부가 결정되어서는 안된다.

이렇게 해야 useState나 useEffect가 여러 번 호출되더라도 훅의 상태를 바르게 유지할 수 있다.

둘째, 훅은 항상 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 한다.

이 2가지 규칙을 지키면 컴포넌트의 모든 상태 관련 로직을 좀 더 명확하게 드러낼 수  있다. 이러한 규칙이 필요한 이유는 리액트에서 훅은 호출 순서에 의존하기 때문이다. 모든 컴포넌트 렌더링에서 훅의 순서가 항상 동일하게 유지되어야 하며, 이를 통해 항상 동일한 컴포넌트 렌더링이 보장된다.

</aside>

# 9.2 커스텀 훅

# 9.2.1 나만의 훅 만들기

커스텀 훅의 이름은 반드시 use로 시작해야 한다. 

**useInput 커스텀 훅**

```tsx
import { useState } from "react";

const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);
  
  const onChange = (e) => {
    setValue(e.target.value);
  };

  return { value, onChange };
};
```

**사용 예제**

```tsx
const MyComponent = () => {
  const { value, onChange } = useInput("");
  
  return (
    <div>
      <h1>{value}</h1>
      <input onChange= {onChange} value= {text} />
    </div>
  );
};

export default App;
```

# 9.2.2 타입스크립트로 커스텀 훅 강화하기

위의 예시를 타입스크립트로 작성

**인자의 타입을 지정하지 않아 에러 발생**

```tsx
import { useState, useCallback } from "react";

// 🚨 Parameter ‘initialValue’ implicitly has an ‘any’ type.ts(7006)
const useInput = (initialValue) => {
  const [value, setValue] = useState(initialValue);

  // 🚨 Parameter ‘e’ implicitly has an ‘any’ type.ts(7006)
  const onChange = useCallback((e) => {
    setValue(e.target.value);
  }, []);

  return { value, onChange };
};

export default useInput;
```

**에러 해결**

```tsx
import { useState, useCallback, ChangeEvent } from "react";

// ✅ initialValue에 string 타입을 정의
const useInput = (initialValue: string) => {
  const [value, setValue] = useState(initialValue);

  // ✅ 이벤트 객체인 e에 ChangeEvent<HTMLInputElement> 타입을 정의
  const onChange = useCallback((e: ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  }, []);

  return { value, onChange };
};

export default useInput;
```