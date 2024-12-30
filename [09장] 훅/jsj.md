# 9장 (훅)

## 9.1 리액트 훅

- 훅 도입 이전에는 클래스 컴포넌트만 상태를 가질 수 있었고 클래스 컴포넌트에서는 생명주기 함수에서만 상태 업데이트에 관련된 로직을 처리할 수 있었다.
- 이는 프로젝트가 커지면서 상태를 스토어에 연결하거나 비슷한 로직을 가진 상태 업데이트 등 여러 요소들을 불편하게 만들었다.
- 훅이 도입됨에 따라 함수 컴포넌트에서도 클래스 컴포넌트 처럼 생명주기에 맞춰 로직을 실행 할 수 있게 됨. 이에 따라 비즈니스 로직을 재사용하거나 작은 단위로 코드를 분할하여 테스트하는 것이 용이해졌음
- UseState
    - 상태를 관리하기 위한 훅
    - 코드
        
        ```tsx
        function useState<S>(
          initialState: S | (() => S)
          ): [S, Dispatch<SetStateAction<S>>];
        //상태를 업데이트 할 수 있는 dispatch함수  
        type Dispatch<A> = (value: A) => void;
        //새로운 상태 값, 또는 이전 상태값을 받아 새로운 상태 값으로 업데이트 하는 함수
        type SetStateAction<S> = S | ((prevState: S) => S);
        
        //((prevState: S) => S)를 쓰는 이유는 이런 식으로도 setState가 가능하기 때문이다
        const [count, setCount] = useState(0);
        setCount((prevCount) => prevCount + 1); // 이전 상태 값에 1을 더함
        ```
        
    - 타입 스크립트를 통해 useState를 작성하면 잘못된 속성이 포함된 객체가 추가되어 발생할 수 있는 에러들도 컴파일 타임에 타입 에러를 발견하여 막아낼 수 있다.
        
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
        
- useEffect
    - 렌더링 이후 리액트 함수 컴포넌트에 어떤 일을 해야 하는지 알려주기 위해 사용
    - 코드
        
        ```tsx
        //첫번째 인자의 타입인 EffectCallback은 Destructor를 반환하거나 아무것도 반환하지 않는 함수
        //Promise 타입은 반환하지 않기 때문에 useEffect의 콜백 함수에 비동기 함수가 들어갈 수 없다.
        //useEffect에서 비동기 함수를 호출한다면 경쟁 상태를 불러일으킬 수 있기 때문이다.
        function useEffect(effect: EffectCallback, deps?: DependencyList): void;
        //deps는 옵셔널하게 제공되며 effect가 수행되기 위한 조건을 나열한다. 
        //deps 배열의 원소가 변경되면 실행한다는 식
        type DependencyList = ReadonlyArray<any>;
        type EffectCallback = () => void | Destructor;
        ```
        
    
    - deps의 원소로 객체나 배열을 넣을 경우 useEffects는 얕은 비교로만 deps가 변경되었는지를 파악하기 때문에 실체 객체 값이 바뀌지 않았어도 참조 값이 변경되면 콜백 함수가 실행된다.
        
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
          
          // ...
        };
        //value.name과 value.id 대신 name, id를 직접 사용한다
        const { id, name } = value;
        
        useEffect(() => {
          //이렇게
        }, [id, name]);
        ```
        
    - Destructor를 반환하는데 이는 컴포넌트가 마운트 해제될 때 실행하는 함수이다. deps가 빈 배열이라면 useEffect 의 콜백 함수는 컴포넌트가 처음 렌더링 될때만 실행되며 이때 destructor는 컴포넌트가 마운트 해제될 때 실행된다. 허나 deps 배열이 존재한다면, 배열의 값이 변경될 때마다 destructor가 실행된다.
- useLayoutEffect
    - 해당 컴포넌트가 그려지기 전에 콜백 함수를 실행하기 때문에 렌더링 전에 상태 변경된 것을 반영이 가능하다.
- useMemo, useCallback
    - 이전에 생성된 값 또는 함수를 기억하는 함수이며 동일한 값등을 반복해서 생성하지 않도록 해주는 훅
    - 어떤 값을 계산하는 데 오랜 시간이 걸릴 때나 렌더링이 자주 발생하는 form에서 훅을 유용하게 사용할 수 있음
    
    ```tsx
    type DependencyList = ReadonlyArray<any>;
    
    function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
    function useCallback<T extends (...args: any[]) => any>(callback: T, deps: DependencyList): T;
    ```
    
    - 두 훅  모두 제네릭을 지원하기 위해 T 타입을 선언해주며 useCallback의 경우 함수를 저장하기 위해 제네릭의 기본 타입을 지정하고 있다.
    - useEffect처럼 deps배열에 들어가는 변수가 바뀌지 않고 참조 값이 변경되었지만 해당 의존성이 변경되었기에 다시 계산될 수 있음에 주의하자.
    - 모든 값과 함수를 useMemo와 useCallback을 사용하여 과도하게 메모이제이션하면 컴포넌트의 성능 향상이 보장되지 않을 수 있다.
- useRef
    - DOM을 직접 선택해야 하는 경우에 사용( `<input />` 요소에 포커스를 주거나 특정 컴포넌트 이동 등)
    - 코드
    
    ```tsx
    //useRef는 세 종류의 타입 정의를 가지고 있으며 넣어주는 인자 타입에 따라 변환되는 타입이 달라짐
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
    
    - useRef는 MutableRefObject, RefObject를 반환한다.
        - MutalbeRefObject의 경우 current의 값을 변경할 수 있게 된다. null을 허용하기 위해
        `const ref = useRef<HTMLInputElement | null>(null)`  로 선언하게 되면 current 값이 변경할 수 있는 값이 되기에 ref.current의 값이 바뀌는 사이드 이펙트가 발생할 수 있다.
        (위의 코드에서 첫번째 타입 정의)
        - RefObject의 경우 current가 readonly로 값을 변경할 수 없기에 
        `const ref = useRef<HTMLInputElement>(null)` 로 선언하게 되면 위 코드에서 두번째 타입 정의를 따르게 되어 ref.current 값을 변경할 수 없게 된다.
    - ref 이라는 속성의 이름은 리액트에서 ‘DOM 요소 접근’ 이라는 특수 목적으로 사용되기 때문에 props를 넘겨주는 방식으로 전달할 수 없다. 리액트 컴포넌트에서 ref를 prop로 전달하기 위해서는 forwardRef를 사용해야 한다. (하지만 ref가 아니라 inputRef 등 다른 이름을 사용하면 forwardRef를 사용하지 않아도 된다.)
    - forwardRef의 두번째 인자에 ref를 넣어 자식 컴포넌트로 ref를 전달할 수 있다.
    - 코드
        
        ```tsx
        function forwardRef<T, P = {}>(
          render: ForwardRefRenderFunction<T, P>
        ): ForwardRefExoticComponent<PropsWithoutRef<P> & RefAttributes<T>>;
        
        //인자로 넘겨주는 콜백 함수인 ForwardRefRenderFunction의 타입 정의는 다음과 같다
        
        interface ForwardRefRenderFunction<T, P = {}> {
          (props: P, ref: ForwardedRef<T>): ReactElement | null;
          displayName?: string | undefined;
          defaultProps?: never | undefined;
          propTypes?: never | undefined;
        }
        
        //2개의 타입 매개변수 T, P를 받으며 
        //P는 일반적인 리액트 컴포넌트에서 자식 컴포넌트로 넘겨주는 props의 타입
        //T는 ref로 전달하려는 요소의 타입
        
        //ref의 타입이 T를 래핑한 ForwardedRef<T>인데 FowardedRef의 타입 정의는 다음과 같다.
        
        type ForwardedRef<T> =
        | ((instance: T | null) => void)
        | MutableRefObject<T | null>
        | null;
        
        ```
        
    - 위의 코드 처럼 ForwardedRef에는 오직 MutableRefObject만 들어올 수 있다. MutableRefObject가 RefObject 보다 넓은 범위의 타입을 가지기 때문에 부모 컴포넌트에서 ref를 어떻게 선언했는지와 관계 없이 자식 컴포넌트가 해당 ref를 수용할 수 있게 된다.
    - useImperativeHandle
        - ForwardRefRenderFunction과 함께 쓸 수 있는 훅이며 이 훅을 활용하면 부모 컴포넌트에서 ref를 통해 자식 컴포넌트에서 정의한 커스터마이징 된 메서드를 호출할 수 있게 된다.
        - 이에 따라 자식 컴포넌트는 내부 상태나 로직을 관리하면서 부모 컴포넌트와의 결합도도 낮출 수 있다.
        - 부모 컴포넌트에서 submit 함수를 사용하여 자식 컴포넌트의 특정 메서드를 실행할 수 있게 되는 것이다.
    - 여러 특성
        - useRef로 관리되는 변수는 값이 바뀌어도 컴포넌트의 리렌더링이 발생하지 않는다.
        - 리액트 컴포넌트의 상태는 상태 변경 함수를 호출하고 렌더링된 이후에 업데이트된 상태를 조회할 수 있다. 반면 useRef로 관리하는 변수는 값을 설정한 후 즉시 조회할 수 있다.
    
    > ❓훅의 규칙
    1. 훅은 항상 최상위 레벨에서 호출되어야 한다. 조건문, 반복문, 중첩 함수, 클래스 등의 내부에서는 훅을 호출할 수 없음 
    2. 훅은 항상 함수 컴포넌트나 커스텀 훅 등의 리액트 컴포넌트 내에서만 호출되어야 한다.
    이러한 규칙이 필요한 이유는 리액트의 훅은 호출 순서에 의존하기 때문이다. 모든 컴포넌트 렌더링에서 훅의 순서가 항상 동일하게 유지되어야 하며 이를 통해 항상 동일한 컴포넌트 렌더링이 보장된다.
    > 

## 9.2 커스텀 훅

- 타입 스크립트를 활용하여 타입을 지정하고 커스텀 훅을 더 강화해보자!