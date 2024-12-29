# 9. 훅

## 9.1 리액트 훅

리액트 16.8 버전부터 훅이 추가되어 클래스 컴포넌트 뿐만아니라 함수 컴포넌트에서도 상태를 관리할 수 있게 되었다.

클래스 컴포넌트에서는 `componentDidMount`, `componentDidUpdate`와 같은 생명주기 함수에서만 상태 업데이트에 따른 로직을 실행할 수 있었다. 이에 따른 단점이 몇가지 있었는데,

1. 프로젝트 규모가 커지면서 상태를 스토어에 연결하거나 비슷한 로직을 가진 상태 업데이트 및 사이드 이펙트 처리가 불편해졌다.
2. 모든 상태를 하나의 함수 내에서 처리하다 보니 관심사가 뒤섞이게 되었고 상태에 따른 테스트나 잘못 발생한 사이드 이펙트의 디버깅이 어려워졌다.

```ts
// 컴포넌트가 마운트되면 실행되는 메서드
componentDidMount() {
  this.props.updateCurrentPage(routeName);
  // 포커스 구독
  this.didFocusSubscription = this.props.navigation.addListener('focus', () => {/*
add focus handler to navigation */});
  // 블러 구독
  this.didBlurSubscription = this.props.navigation.addListener('blur', () => {/* add
blur handler to navigation */});
}

// componentDidMount에서 정의한 컴포넌트가 언마운트될 때 실행되는 메서드
componentWillUnmount() {
  // 포커스 구독 해제
  if (this.didFocusSubscription != null) {
    this.didFocusSubscription();

  // 블러 구독 해제
  if (this.didBlurSubscription != null) {
    this.didBlurSubscription();
  }
  // 타이머 제거
  if (this._screenCloseTimer != null) {
    clearTimeout(this._screenCloseTimer);
    this._screenCloseTimer = null;
  }
}

// 컴포넌트가 업데이트되면 실행되는 메서드
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

훅이 도입되면서 하나의 함수로 묶어서 관리할 수 없었던 상태 관리를, 비즈니스 로직을 재사용하거나 작은 단위로 코드를 분할하여 테스트하는 게 쉬워졌고, 사이드 이펙트와 상태를 관심사에 맞게 분리하여 구성할 수 있게 되었다.

⚠️ 훅을 사용할 때의 규칙이 있다.

1. 항상 최상위 레벨에서 호출되어야 한다. 조건문, 반복문, 중첩 함수, 클래스 등의 내부에서 훅을 호출하지 않아야 한다.
2. 항상 함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 한다.

이러한 규칙이 필요한 이유는 **훅은 호출 순서에 의존**하기 때문이다. 모든 렌더링에서 훅의 순서가 항상 동일하고 이를 통해 항상 동일한 컴포넌트 렌더링이 보장될 수 있다.

&nbsp;

### 1. useState

`useState` 타입 정의를 살펴보자.

```ts
// S: State
// A: Action
function useState<S>(
  initialState: S | (() => S) // S 타입 혹은 S 타입을 반환하는 함수
): [S, Dispatch<SetStateAction<S>>]; // S와 Dispatch 타입

type Dispatch<A> = (value: A) => void; // A를 받아 void를 반환
type SetStateAction<S> = S | ((prevState: S) => S); // S 혹은 이전 상태 값을 받아 새로운 상태를 반환
```

`useState`에 TS를 적용하면 상태 타입에 대해 엄격하게 검사해 에러를 방지할 수 있다. 아래 코드는 JS로 작성된 문제가 발생하는 코드다.

```js
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
        agee: 11, // 🚨 유효하지 않은 속성
      },
    ]);
  };
};
```

위 코드에서는 유효하지 않은 속성의 객체를 추가함으로써 `sumAge`가 `NaN`가 되는 사이드 이펙트가 발생했다. TS는 상태의 타입을 검사하므로 이런 사고를 미연에 방지할 수 있다.

아래 코드는 TS로 작성되어 에러를 잡아내고 있다.

```ts
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

&nbsp;

### 2. 의존성 배열을 사용하는 훅

### useEffect

렌더링 후 함수 컴포넌트의 동작을 지정해주는 `useEffect` 훅의 타입에 대해 살펴보자.

```ts
function useEffect(effect: EffectCallback, deps?: DependencyList): void;
type DependencyList = ReadonlyArray<any>; // 의존성 배열 리스트
type EffectCallback = () => void | Destructor; // void 반환 함수 혹은 상태 초기화 함수인 Destructor 반환
```

`effect`는 `Promise` 타입은 반환하지 않는데, `useEffect`에서 비동기 함수를 호출할 경우, `Race Condition`을 발생시킬 수 있기 때문이다.

> 💡Race Condition은 멀티스레딩 환경에서 동시에 여러 프로세스나 스레드가 공유된 자원에 접근하려고 할 때 발생할 수 있는 문제다. 이러한 상황에서 실행 순서나 타이밍을 예측할 수 없게 되어 프로그램의 동작이 원하지 않는 방향으로 흐를 수 있다.

&nbsp;

`deps`는 `effect` 수행되기 위한 조건을 나열한 배열이다. `deps`의 배열의 원소가 변경되면 실행한다는 식으로 사용된다.

```ts
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

  // ...
};
```

❗`deps`에 객체를 넣을 땐 얕은 비교로만 판단하기 때문에, 실체 객체 값이 변하지 않았더라도 객체의 참조 값이 바뀌면 콜백 함수가 실행되기 때문에 원치 않는 렌더링을 방지하기 위해서는 사용할 값을 특정해서 `deps`에서 사용해줘야 한다.

> 💡 얕은 비교: 객체나 배열과 같은 복합 데이터 타입의 값을 비교할 때 내부의 각 요소나 속성을 재귀적으로 비교하지 않고, 해당 값들의 참조나 기본 타입 값만을 간단하게 비교하는 것을 말한다.

&nbsp;

```ts
const { id, name } = value;

useEffect(() => {
  // value.name과 value.id 대신 name, id를 직접 사용하게끔 수정
}, [id, name]);
```

`Destructor` 함수는 빈 배열일 경우 컴포넌트가 언마운트될 때 한 번만 실행된다. `deps` 배열이 존재할 경우, 배열의 값이 변경될 때마다 실행된다.

&nbsp;

### useLayoutEffect

`useEffect`는 레이아웃 배치와 화면 렌더링이 모두 완료된 후에 실행된다. `useLayoutEffect`는 **해당 컴포넌트가 그려지기 전에** 콜백 함수를 실행한다.

```ts
type DependencyList = ReadonlyArray<any>;

function useLayoutEffect(effect: EffectCallback, deps?: DependencyList): void;
```

아래 코드를 실행하면 `setName`이 실행되기까지 오래 걸리므로 꽤 오랜 시간 "안녕하세요, 님!"으로 렌더링된 후, 닉네임이 변경되어 렌더링될 것이다. `useLayoutEffect`를 사용하면 이런 현상을 방지할 수 있다.

```ts
const [name, setName] = useState("");

useEffect(() => {
  // 매우 긴 시간이 흐른 뒤 아래의 setName()을 실행한다고 생각하자
  setName("배달이");
}, []);

return <div>{`안녕하세요, ${name}님!`}</div>;
```

&nbsp;

### useMemo와 useCallback

`useMemo`와 `useCallback`은 이전에 생성된 값 또는 함수를 기억하며, 동일한 값과 함수를 반복해서 생성하지 않도록 하는 훅이다. 계산이 오래 걸리거나 렌더링이 자주 발생하는 `form`에서 유용하게 사용한다.

```ts
type DependencyList = ReadonlyArray<any>;

function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
function useCallback<T extends (...args: any[]) => any>(
  callback: T,
  deps: DependencyList
): T;
```

`deps` 배열이 변경되면 값을 다시 계산한다.

❗`useEffect`와 마찬가지로 얕은 비교를 하기 때문에 `deps` 배열이 변경되지 않았는데도 다시 계산되지 않도록 주의한다.

&nbsp;

### 3. useRef

input 태그에 포커스를 주거나, 특정 컴포넌트 위치로 스크롤하는 등의 DOM 요소를 직접 선택해야 할 때 `useRef`를 사용한다.

아래 코드는 버튼을 클릭하면 input 태그 DOM에 포커스를 설정한다.

```ts
import { useRef } from "react";

const MyComponent = () => {
  const ref = useRef<HTMLInputElement>(null);

  const onClick = () => {
    ref.current?.focus();
  };

  return (
    <>
      <button onClick={onClick}>ref에 포커스!</button>
      <input ref={ref} />
    </>
  );
};

export default MyComponent;
```

`ref`를 제네릭 인자로 `HTMLInputElement`를 받고 초깃값을 `null`로 설정해주는 이유가 있다.

이유를 알기 위해서는 `useRef`의 타입 정의를 살펴봐야 한다.

```ts
function useRef<T>(initialValue: T): MutableRefObject<T>;
function useRef<T>(initialValue: T | null): RefObject<T>;
function useRef<T = undefined>(): MutableRefObject<T | undefined>;

interface MutableRefObject<T> {
  current: T;
}

interface RefObject<T> {
  readonly current: T | null;
}
```

`useRef`는 변경 가능한 타입인 `MutableRefObject` 혹은 readonly 타입인 `RefObject`를 반환한다.

만약 제네릭에 `HTMLInputElement | null`을 넣어 `MutableRefObject` 타입이 되면 `ref.current` 값이 바뀔 수 있기 때문에, `RefObject`를 반환하도록 하기 위해 `useRef<HTMLInputElement>(null)`와 같이 선언해준 것이다.

&nbsp;

### 자식 컴포넌트에 ref 전달하기

input 태그와 같은 기본 HTML 요소가 아니라, 리액트 컴포넌트에 `ref`를 전달할 수도 있을 것이다. `ref`는 props로 넘겨주면 브라우저에서 경고 메시지를 띄운다.

```ts
import { useRef } from "react";

const Component = () => {
  const ref = useRef<HTMLInputElement>(null);
  return <MyInput ref={ref} />;
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
  return <input ref={ref} />;
};
```

<details>

<summary>🤔 ref는 왜 props로 넘겨주면 안되는가?</summary>

1. ref는 React에서 특별하게 취급되는 속성(DOM 요소에 직접 접근하기 때문에)입니다. key와 마찬가지로 일반적인 prop과는 다르게 처리됩니다.
2. 함수 컴포넌트는 인스턴스가 없기 때문에 ref 어트리뷰트를 직접 사용할 수 없습니다
3. ref는 클래스 컴포넌트에서 컴포넌트 인스턴스를 참조하는 데 사용됩니다. 그러나 함수 컴포넌트는 인스턴스가 없어서 ref로 참조할 대상이 없습니다.
4. React는 ref를 특별한 용도로 사용하기 때문에, 일반 prop으로 전달하려고 하면 경고를 발생시킵니다.
5. ref는 불변성(immutability)을 위반할 수 있습니다. props는 불변이어야 하지만, ref는 DOM 요소에 할당될 때 변경될 수 있습니다.

이러한 이유로 React는 ref를 일반 prop으로 전달하는 것을 허용하지 않습니다. 대신

1. forwardRef를 사용하여 ref를 전달하거나
2. 다른 이름의 prop(예: innerRef)을 사용하여 ref를 전달할 수 있습니다.

```js
// forwardRef로 ref를 전달
const ChildComponent = forwardRef((props, ref) => {
  return <input ref={ref} />;
});

function ParentComponent() {
  const inputRef = useRef(null);
  return <ChildComponent ref={inputRef} />;
}

// 다른 이름의 prop을 사용하여 ref를 전달
const Child = ({ customRef }) => {
  return <input ref={customRef} />;
};

const Parent = () => {
  const inputRef = useRef(null);
  return <Child customRef={inputRef} />;
};
```

</details>

&nbsp;

> 💡 참고: 리액트 19 버전부터는 함수 컴포넌트에서 `forwardRef`를 사용하지 않아도 `ref`를 전달할 수 있게끔 변경되었고, `forwardRef`는 점점 deprecated될 예정이다.
>
> ```jsx
> // React 19 이전
> const MyInput = forwardRef((props, ref) => {
>   return <input ref={ref} {...props} />;
> });
>
> // React 19
> const MyInput = ({ ref, ...props }) => {
>   return <input ref={ref} {...props} />;
> };
> ```

`forwardRef`의 타입 정의다.

```ts
// ref의 타입과 props의 타입을 제네릭 매개변수로 받는다.
function forwardRef<T, P = {}>(
  render: ForwardRefRenderFunction<T, P>
): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;
```

&nbsp;

### useImperatvieHandle

`useImperativeHandle`은 `ForwardRefRenderFunction`(`forwardRef`의 콜백 함수)과 함께 쓸 수 있는 훅이다. 이 훅은 **부모 컴포넌트에서 `ref`를 통해 자식 컴포넌트에서 정의한 메서드를 호출**할 수 있게 해준다. 이에 따라 자식 컴포넌트는 내부 상태나 로직을 관리하면서 부모 컴포넌트와의 결합도를 낮출 수 있다.

아래 코드의 예시는 `useImperativeHandle`으로 부모 컴포넌트에서 사용할 메서드를 정의하고 있다.

```ts
// 자식 컴포넌트
// <form> 태그의 submit 함수만 따로 뽑아와서 정의한다
type CreateFormHandle = Pick<HTMLFormElement, "submit">;

type CreateFormProps = {
  defaultValues?: CreateFormValue;
};

const JobCreateForm: React.ForwardRefRenderFunction<
  CreateFormHandle,
  CreateFormProps
> = (props, ref) => {
  // useImperativeHandle를 사용해서 submit 함수를 커스터마이징한다
  useImperativeHandle(ref, () => ({
    submit: () => {
      /* submit 작업을 진행 */
    },
  }));

  // ...
};
```

```ts
// 부모 컴포넌트
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

`useImperativeHandle`을 사용하면 모달(자식 컴포넌트)의 여닫는 메서드를 부모 메서드에서 호출하게도 만들 수 있다.

```jsx
const Modal = forwardRef((props, ref) => {
  const [isOpen, setIsOpen] = useState(false);
  // 부모 컴포넌트에서 사용 가능한 메서드를 정의
  useImperativeHandle(ref, () => ({
    open: () => setIsOpen(true),
    close: () => setIsOpen(false),
  }));

  if (!isOpen) return null;

  return (
    <div className="modal">
      {props.children}
      <button onClick={() => setIsOpen(false)}>Close</button>
    </div>
  );
});

function App() {
  const modalRef = useRef();

  return (
    <div>
      {/* 자식 컴포넌트의 메서드를 사용 */}
      <button onClick={() => modalRef.current.open()}>Open Modal</button>
      <Modal ref={modalRef}>
        <h2>Modal Content</h2>
      </Modal>
    </div>
  );
}
```

&nbsp;

### useRef의 여러 가지 특성

`useRef`의 특성에 대해 알아보자.

- `useRef`로 관리되는 변수는 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않는다. 상태가 변경되더라도 렌더링하지 않게끔 해줄 수 있다.

- 리액트 컴포넌트의 상태는 상태 변경 함수를 호출하고 렌더링된 이후에 업데이트된 상태를 조회할 수 있는 반면, `useRef`로 관리되는 변수는 값을 설정한 후 즉시 조회할 수 있다.

```ts
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
          // isAutoPlayPause는 사실 렌더링에는 영향을 미치지 않고 로직에만 영향을 주므로 상태로 사용해서 불필요한 렌더링을 유발할 필요가 없다
          onClick={() => {
            isAutoPlayPause.current = true;
          }}
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

`isAutoPlayPause`를 `ref`로 선언하여 값이 변경되더라도 리렌더링이 일어나지 않도록 만들었다. 또 `ref`의 `current` 속성에 `null`이 아닌 값을 할당해서 마치 변수처럼 활용할 수도 있다.

&nbsp;

## 9.2 커스텀 훅

### 1. 나만의 훅 만들기

커스텀 훅은 자주 사용되는 컴포넌트 로직을 정의해서 사용하기 위해 사용한다.

커스텀 훅을 만들 때 고려해야 할 점은 다음과 같다.

1. 네이밍 규칙: 커스텀 훅의 이름은 반드시 'use'로 시작
2. 순수성: 같은 입력에 대해 항상 같은 결과를 반환하도록 순수하게 작성
3. 단일 관심사 원칙: 하나의 커스텀 훅은 하나의 관심사만을 다룸. 여러 관심사가 존재한다면 훅을 분리
4. 재사용성 고려: 다른 컴포넌트에서도 사용할 수 있도록 일반화된 형태로 설계
5. 의존성 관리: useEffect 등을 사용할 때 의존성 배열을 적절히 설정하여 무한 루프 등의 문제를 방지
6. 상태 독립성 유지: 커스텀 훅은 서로 독립적이어야 하며, 다른 훅의 상태에 영향을 주지 않아야 함
7. 메모이제이션 고려: 성능 최적화를 위해 필요한 경우 useMemo나 useCallback을 사용하여 값이나 함수를 메모이제이션
8. 파라미터 주의: 커스텀 훅의 파라미터가 내부 상태의 초기값으로 사용될 때, 파라미터 변경이 내부 상태 변경으로 이어지지 않음을 주의

&nbsp;

```ts
// useModal.tsx
import { useState, useCallback } from "react";

interface UseModalReturn {
  isOpen: boolean;
  openModal: () => void;
  closeModal: () => void;
  toggleModal: () => void;
}

function useModal(): UseModalReturn {
  const [isOpen, setIsOpen] = useState<boolean>(false);

  const openModal = useCallback((): void => {
    setIsOpen(true);
  }, []);

  const closeModal = useCallback((): void => {
    setIsOpen(false);
  }, []);

  const toggleModal = useCallback((): void => {
    setIsOpen((prev) => !prev);
  }, []);

  return {
    isOpen,
    openModal,
    closeModal,
    toggleModal,
  };
}

export default useModal;
```

```tsx
import React from "react";
import useModal from "./useModal";
import Modal from "./Modal"; // Modal 컴포넌트

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  children: React.ReactNode;
}

const Modal: React.FC<ModalProps> = ({ isOpen, onClose, children }) => {
  if (!isOpen) return null;

  return (
    <div className="modal">
      <div className="modal-content">
        {children}
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
};

const App: React.FC = () => {
  const { isOpen, openModal, closeModal } = useModal();

  return (
    <div>
      <button onClick={openModal}>Open Modal</button>
      <Modal isOpen={isOpen} onClose={closeModal}>
        <h2>Modal Content</h2>
        <p>This is the modal content.</p>
      </Modal>
    </div>
  );
};

export default App;
```

&nbsp;

🤔 전에 프로젝트를 진행할 때, 모달의 상태 관리를 어떻게 하는 것이 좋을지 고민하곤 했다. 훅에 대해 배운 김에 나름의 기준을 선정해서 어떤 상황에 어떤 방법을 사용하는 것이 적절할지 정리해본다.

**useModal 훅을 사용하면**❓

Pros:

- 여러 컴포넌트에서 동일한 모달 로직을 쉽게 재사용할 수 있다.
- 모든 모달에 대해 동일한 인터페이스를 제공한다.

Cons:

- 여러 모달이 서로 상호작용해야 하는 경우 상태 관리가 어렵다.
- 특정 상황에 커스터마이징이 어렵다.

&nbsp;

**부모 컴포넌트에서 모달의 상태를 관리하면**❓

Pros:

- 상태와 상태 변경의 흐름을 파악하기 쉽다.
- 특정 상황에 맞게 쉽게 커스터마이징할 수 있다.
- 여러 모달 간의 상호작용을 쉽게 관리할 수 있다.

Cons:

- 여러 컴포넌트에서 유사한 모달 로직을 반복해서 보일러플레이트 코드가 발생할 수 있다.
- 부모 컴포넌트가 많은 상태를 관리해야 할 경우 코드가 복잡해질 수 있다.

**정리**

- 단순한 모달이 여러 곳에서 사용되는 경우?: useModal 훅을 만들어 사용
- 복잡한 상호작용이 필요하거나 커스터마이징이 요구될 경우?: 부모 컴포넌트에서 상태 관리

&nbsp;

### 2. 타입스크립트로 커스텀 훅 강화하기

```js
// useInput.jsx
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

```ts
// useInput.tsx
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

이벤트 객체 같은 유추하기 힘든 타입은 tsc가 현재 사용하고 있는 이벤트 객체의 타입을 유추해서 알려주기도 하니 IDE의 타입 추론 기능을 이용해보자.
