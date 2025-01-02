# 10장 (상태 관리)

## 10.1 상태 관리

- 상태 : 리액트에서 렌더링에 영향을 줄 수 있는 동적인 데이터 값, 영향을 주는 정보를 담은 순수 자바 스크립트 객체
- 지역 상태
    - 컴포넌트 내부에서 사용되는 상태
    - 체크박스의 체크 여부나 폼의 입력값 등이 해당됨
    - 주로 useState훅을 가장 많이 사용하며 때에 따라 useReducer 같은 훅을 사용하기도 함
- 전역 상태
    - 앱 전체에서 공유하는 상태
    - 여러 개의 컴포넌트가 전역 상태를 사용할 수 있으며 상태가 변경되면 컴포넌트들도 업데이트 된다.
    - Prop drilling 문제를 피하고자 지역 상태를 해당 컴포넌트들 사이의 전역 상태로 공유할 수도 있다.
    
    (다들 어느정도 단계까지 prop이 전달되면 전역상태로 빼는지? → 저번에 서진이한테 물어봤을 때는 한 2단계 넘어가면 뺀다고 했는데 다른 사람들은 어떻게 했나 궁금함)
    
    > ❓ **Prop drilling**
    > 
    > props를 통해 데이터를 전달하는 과정에서 중간 컴포넌트는 해당 데이터가 필요하지 않음에도 자식 컴포넌트에 전달하기 위해 props를 전달해야 하는 과정을 말한다. 컴포넌트의 수가 많아지면 이로 인해 코드가 더 복잡해질 수 있다.
    > 
- 서버 상태
    - 사용자 정보, 글 목록 등 외부 서버에 저장해야 하는 상태 들을 의미한다.
    - UI 상태와 결합하여 관리하게 되며 로딩 여부나 에러 상태 등을 포함한다.
    - 서버 상태는 다른 상태와 동일한 방법으로 관리 되며 최근에는 외부 라이브러리를 사용하여 관리하기도 한다.
- 상태가 업데이트 될 때마다 리렌더링이 발생하기 때문에 유지보수 및 성능 관점에서 상태의 개수를 최소화하는 것이 바람직하다. 가능하면 상태가 없는 stateless 컴포넌트를 활용하는 것이 좋다.
- 어떤 값을 상태로 정의할 때 고려해야할 2가지
    - 시간이 지나도 변하지 않는다면 상태가 아니다
        - 시간이 지나도 변하지 않는 값은 객체 참조 동일성을 유지하는 방법을 고려해볼 수 있다.
        - useMemo를 통해 컴포넌트가 마운트 될 때만 객체 인스턴스를 생성하고 이후 렌더링에서는 이전 인스턴스를 재활용 할 수 있도록 구현하여 불필요한 리렌더링을 막자
        - 하지만 useMemo는 객체 참조 동일성을 유지하기 위해 사용되기 보단 성능 향상을 위한 용도로만 사용되는 것이 권장된다. 또한 메모리 확보를 위해 useMemo의 메모이제이션 데이터가 삭제 될 수도 있으니 useMemo 없이도 올바르게 동작하도록 코드를 작성하고 이후에 useMemo를 추가하도록 하자
        - 변화하지 않는 값을 참조하여 계속 같은 객체를 참조하게 하는 방법은 2가지가 있다.
            - useState로 초기값만 지정
                - 이의 경우 `useState(new Store())` 를 쓸 경우 객체 인스턴스가 실제로 사용되지 않아도 렌더링마다 생성되기 때문에 초깃값 설정에 비용이 커질 수 있다.
                - `useState(()⇒ new Store())` 와 같이 초깃값을 계산하는 콜백을 지정하는 방식을 사용하자.
                - 다만 기술적으로는 잘 작동할 수 있지만 useState 는 상태이므로 시간이 지나면서 변화하여 렌더링에 영향을 주는 데이터여야 하지만 모든 렌더링 과정에서 객체의 참조를 동일하게 유지하고자 하는 것이기에 의미론적으로는 맞지 않다.
            - useRef를 사용
                - useRef의 인자로 new Store()를 사용하면 렌더링마다 불필요한 인스턴스가 생성되므로 아래와 같이 작성해준다.
                    
                    ```tsx
                    const store = useRef<Store>(null);
                    
                    if (!store.current){
                    	store.current = new Store();
                    }
                    ```
                    
                - useRef는 기술적으로는 `useState({children : initialValue})[0]`와 동일하지만 useState는 상태인데 초기값만 할당하는건 의미적으로 적절하지 않다. 따라서 이런식의 사용은 useRef가 더 권장된다.
    - 파생된 값은 상태가 아니다
        - 부모에게서 전달받을 수 있는 props거나 기존 상태에서 계산될 수 있는 값은 상태가 아니다.
        - 데이터는 하나의 출처에서 생성되고 수정해야 하기에 다른 값에서 파생된 값을 상태로 관리하게 되면 기존 출처와 다른 곳에서 관리하게 되는 것이라 해당 데이터의 정확성과 일관성을 보장하기 어렵다.
        - useEffect를 사용한 동기화 작업은 리액트 외부 데이터와 동기화할 때만 사용해야 하며, 내부에 존재하는 데이터를 상태와 동기화 하는데 사용하면 안된다. 내부에 존재하는 상태를 useEffect로 동기화 하면 개발자가 추적하기 어려운 오류가 발생할 수 있기 때문이다.
        
        > ❓ **SSOT란?**
        >
        > 어떠한 데이터도 단 하나의 출처에서 생성하고 수정해야 한다는 원칙을 의미하는 방법론
        리액트 앱에서 상태를 정의할 때도 이를 고려해야한다.
        기존 출처와 다른 출처에서 파생된 데이터를 사용한다면 헤당 데이터의 일관성과 정확성을 보장하기 어렵기 때문
        > 
        - 두 컴포넌트에서 동일한 데이터를 상태로 갖고 있을 때는 두 컴포넌트 간의 상태를 가장 가까운 공통 부모 컴포넌트로 상태를 끌어올려서 SSOT를 지킬 수 있도록 하자. 관리하던 상태의 출처를 props 하나로 통일할 수 있기 때문이다.
        - 예시 (위의 내용과 다르게 부모 컴포넌트로 상태를 끌어올리는건 아니지만 상태의 출처가 여러개인 경우를 하나로 축소하는 내용)
            
            ```tsx
            //이 경우 useEffect로 동기화 작업을 하고 있지만 items와 selectedItems가 동기화 되지 않을 수 있다.
            //여러 상태가 얽혀 있으면 흐름을 파악하기 어렵고, 의도치 않게 동기화 과정이 누락될 수도 있기 때문이다.
            //(useEffect로 동기화 작업을 진행할때의 단점을 의미하는듯)
            //또한 setSlectedItems를 통해 items 에서 가져온 데이터가 아닌 임의의 데이터 셋을 설정하는
            //것도 가능하기에 오류가 발생할 수 있다.
            
            const [items, setItems] = useState<Item[]>([]);
            const [selectedItems, setSelectedItems] = useState<Item[]>([]);
            
            //또한 이렇게 items, selectedItems 2가지 상태를 유지하면서 useEffect로 동기화하게
            //되면 selectedItems 값을 얻기 위해 2번의 렌더링이 된다.
            //items가 변경되면서, items가 변경되는걸 감지하고 selectedItems가 변경되면서
            
            useEffect(() => {
              setSelectedItems(items.filter((item) = > item.isSelected));
            }, [items]);
            
            //따라서 아래와 같이 상태로 정의하지 않고 계산된 값을 자바 스크립트 변수로 담는 것이 있다.
            const [items, setItems] = useState<Item[]>([]);
            const selectedItems = items.filter((item) = > item.isSelected);
            
            //이 경우 items가 변경될 때마다 컴포넌트가 새로 렌더링 되고 렌더링 될 때마다 
            //selectedItems를 다시 계산하게 된다. 또한 단일 출처를 가지게 된다.
            //리 렌더링 횟수를 줄일 수 있지만 매번 렌더링 될때마다 계산을 수행하므로 계산 비용이 크다면
            //useMemo를 사용하여 items가 변경될 때마다 계산을 수행하고 결과를 메모이제이션하여
            //성능을 개선할 수 있다.
            const [items, setItems] = useState<Item[]>([]);
            const selectedItems = useMemo(() => veryExpensiveCalculation(items), [items]);
            ```
            

- useState vs useReducer
- useReducer 사용을 권장하는 경우는 2가지가 있다.
    - 다수의 하위 필드를 포함하고 있는 복잡한 상태 로직
    - 다음 상태가 이전 상태에 의존적일 때
- useReducer는 ‘무엇을 변경할지’ 와 ‘어떻게 변경할지’를 분리하여 dispatch를 통해 어떤 작업을 할지를 액션으로 넘기고 reducer 함수 내에서 상태를 업데이트 하는 방식을 정의한다.
    
    ```tsx
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
    
- 이를 통해 상태 로직을 숨기고 안전성을 높일 수 있다.
- 또는 토글하는 액션 처럼 boolean 상태를 관리하는데 사용할 수도 있다.
    
    ```tsx
    //Before
    const [fold, setFold] = useState(true);
    
    const toggleFold = () => {
      setFold((prev) => !prev);
    };
    
    // After
    const [fold, toggleFold] = useReducer((v) => !v, true);
    ```
    

> ❓ **useState보다 useReducer를 사용하는게 더 좋은거 아님? 상위 개념 아님?**
> 
> 
> 둘의 비교는 다음과 같음
> 
> | **특징** | **useState** | **useReducer** |
> | --- | --- | --- |
> | **복잡성** | 단순한 상태 관리에 적합 | 복잡한 상태 로직 및 여러 상태 간 상호작용에 적합 |
> | **업데이트 방식** | 직접 상태 값을 설정 (`setState`) | 액션을 디스패치(`dispatch`)하여 리듀서가 상태를 업데이트 |
> | **코드 구조** | 상태 업데이트 로직이 컴포넌트 안에 포함 | 상태 업데이트 로직이 리듀서 함수로 분리 |
> | **가독성** | 상태가 단순하면 더 읽기 쉽고 유지보수하기 쉬움 | 상태 로직이 복잡할 경우 더 읽기 쉽고 구조화된 코드를 작성 가능 |
> | **사용 사례** | 간단한 숫자, 문자열, boolean 등의 상태 | 상태가 객체로 이루어져 있거나, 복잡한 업데이트 로직이 필요한 경우 |
> 
> 요약하자면 useState가 더 직관적이고 코드가 간결해지기에 간단한 상태 관리의 경우 useState를 사용하는게 더 좋음
> 
- 상태를 전역 상태로 사용하는 법
    - 리액트 컨텍스트 API (context API)
        - 다른 컴포넌트들과 데이터를 쉽게 공유하기 위한 목적으로 제공되는 API
        - 깊은 레벨에 있는 컴포넌트 사이에 데이터를 전달하는 Prop Drilling 같은 문제를 해결하기 위한 도구로 활용됨
        - 전역적으로 공유해야 하는 데이터를 context로 제공하고 해당 context를 구독한 컴포넌트에서만 데이터를 읽을 수 있게 된다.
        - 상위 컴포넌트의 구현 부에 context provider를 넣어주고 하위 컴포넌트들이 해당 context를 구독하는 방식으로 데이터를 읽어오자.
            
            ```tsx
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
            
        - 유틸리티 함수를 정의하여 더 간단한 코드로 context와 훅을 생성할 수 있다.
        - context API를 사용하여 전역 상태를 관리하는 것은 대규모 혹은 성능이 중요한 어플리케이션에서는 권장되지 않는다. context provider의 props로 주입된 값이나 참조가 변경될 때마다 해당 context를 구독하고 있는 모든 컴포넌트가 리렌더링 되기 때문.
        - 어플리케이션이 커지고 전역 상태가 많아질 수록 불필요한 리렌더링과 상태의 복잡도가 증가한다..

> ❓ **Context란**
>
>React에서 컴포넌트 계층 구조를 따라 Prop drilling 없이 전역적으로 데이터를 공유할 수 있도록 도와주는 기능
> 

## 10.2 상태 관리 라이브러리

- MobX
    - 객체 지향 프로그래밍과 반응형 프로그래밍 패러다임의 영향을 받은 라이브러리
    - 상태 변경 로직 단순하게 작성 가능, 복잡한 업데이트 로직 라이브러리에 위임
- Redux
    - 함수형 프로그래밍의 영향을 받은 라이브러리
    - 특정 UI 프레임워크에 종속되지 않아 독립적으로 상태 관리 라이브러리 사용 가능
- Recoil
    - 상태를 지정할 수 있는 atom과 해당 상태를 변형할 수 있는 순수 함수 selector로 관리
    - 아직 실험적인 상태의 라이브러리
- Zustand
    - Flux 패턴을 사용하여 많은 보일러 플레이트를 가지지 않는 훅 기반의 편리한 API 모듈 제공
    - 클로저로 스토어 내부 상태를 관리하여 라이브러리에 종속되지 않음
