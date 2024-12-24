# 8. JSX에서 TSX로

## 8.1 리액트 컴포넌트의 타입

8.1절에선 리액트를 TS로 작성할 때 사용하는 대표적인 리액트 컴포넌트 타입을 살펴보고 어떤 것을 사용하면 좋을지, 그리고 사용할 때 유의할 점에 대해 알아본다.

&nbsp;

### 1. 클래스 컴포넌트 타입

```ts
interface Component<P = {}, S = {}, SS = any>
  extends ComponentLifecycle<P, S, SS> {}

class Component<P, S> {
  /* ... 생략 */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {}
```

클래스 컴포넌트는 

* `React.Component`
* `React.PureComponent`

를 상속받는다. `P`는 props의 타입, `S`는 state의 타입을 의미한다. 

SS는 Snapshot State의 약자로 Snapshot State의 타입을 의미한다. SS는 리액트의 `getSnapshotBeforeUpdate` 생명주기 메서드와 관련이 있다. `getSnapshotBeforeUpdate` 메서드는 DOM이 업데이트되기 전 호출되어 메서드의 반환값이 `componentDidUpdate`(클래스 컴포넌트의 생명주기 메서드로, 컴포넌트의 업데이트가 DOM에 반영된 직후에 호출됨)의 세 번째 매개변수로 전달된다. 이 반환값의 타입이 바로 SS다.

```ts
class ScrollingList extends React.Component<Props, State, number> {
  // DOM 업데이트 전에 스냅샷을 저장
  getSnapshotBeforeUpdate(prevProps: Props, prevState: State): number {
    return this.listRef.current.scrollHeight;
  }

  // DOM 업데이트 후 실행되어 스냅샷 값을 사용해 스크롤 위치 조정
  componentDidUpdate(prevProps: Props, prevState: State, snapshot: number) {
    // snapshot은 getSnapshotBeforeUpdate에서 반환한 값입니다.
    this.listRef.current.scrollTop += this.listRef.current.scrollHeight - snapshot;
  }

  render() { /* ... */ }
}
```

아래와 같이 클래스 컴포넌트는 `React.Component` 타입을 상속받고 있고, `WelcomeProps`를 props의 타입으로 받고 있다.

```ts
interface WelcomeProps {
  name: string;
}

class Welcome extends React.Component<WelcomeProps> {
  /* ... 생략 */
}
```

&nbsp;

### 2. 함수 컴포넌트 타입

```ts
// 함수 선언을 사용한 방식
function Welcome(props: WelcomeProps): JSX.Element {}

// 함수 표현식을 사용한 방식 - React.FC 사용
const Welcome: React.FC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - React.VFC 사용
const Welcome: React.VFC<WelcomeProps> = ({ name }) => {};

// 함수 표현식을 사용한 방식 - JSX.Element를 반환 타입으로 지정
const Welcome = ({ name }: WelcomeProps): JSX.Element => {};

type FC<P = {}> = FunctionComponent<P>;

interface FunctionComponent<P = {}> {
  // props에 children을 추가
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}

type VFC<P = {}> = VoidFunctionComponent<P>;

interface VoidFunctionComponent<P = {}> {
  // children 없음
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P> | undefined;
  contextTypes?: ValidationMap<any> | undefined;
  defaultProps?: Partial<P> | undefined;
  displayName?: string | undefined;
}
```

함수 컴포넌트는 함수 표현식을 사용한 컴포넌트다. 보통은 `React.FC`나 `React.VFC`로 타입을 지정해서 사용한다. VFC는 VoidFunctionComponent로, FC와 달리 `children`이라는 타입을 허용하지 않는다. FC는 암묵적으로 `children`을 포함하고 있어 children을 사용하지 않더라도 `children` props를 허용한다. 함수 컴포넌트를 사용할 때 더 정확한 타입 지정을 위해 VFC를 사용한다. 이런 이유로 FC 타입을 사용하는 것은 권장되는 방식이 아니었다.

그러나 v18부터 VFC가 삭제되고 FC에서 `children`이 사라졌고 이후의 버전에서는 VFC대신 FC 또는 props 타입과 반환 타입을 직접 지정해주는 형태로 선언해야 한다.

```ts
type Props = {
  // FC를 사용하되 명시적으로 children을 선언
  children?: React.ReactNode;
  // 다른 props...
};

// props 타입과 반환 타입을 직접 지정
const Component: React.FC<Props> = ({ children, ...props }) => {
  // 컴포넌트 로직
};
```

&nbsp;

### 3. children props 타입 지정

```ts
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };
```

가장 보편적인 `children` 타입은 `ReactNode | undefined` 타입이다. `ReactNode`는 `ReactElement`를 포함해 `boolean`, `number` 등 여러 타입을 포함하는 상위 타입이다. 때문에 구체적인 타입 선언을 위한 용도로는 쓰기 힘들고, 특정 타입만 허용하고 싶을 때는 `children`에 대해 구체적인 선언이 필요하다. 

```ts
// 들어올 수 있는 문자열을 명시
type WelcomeProps = {
  children: "천생연분" | "더 귀한 분" | "귀한 분" | "고마운 분";
};

// 문자열 타입만 허용
type WelcomeProps = { children: string };

// 리액트 컴포넌트만 허용
type WelcomeProps = { children: ReactElement };
```

&nbsp;

### 4. render 메서드와 함수 컴포넌트의 반환 타입 - React.ReactElement vs JSX.Element vs React.ReactNode

* `React.ReactElement`: 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷, 리액트의 가상 DOM 기반 렌더링을 구현하는 가상 DOM 엘리먼트의 타입이다.
* `JSX.Element`: `ReactElement`를 확장하는 타입으로, 글로벌 네임스페이스에 정의되어 외부 라이브러리에서 컴포넌트 타입을 재정의할 수 있어 유연성을 제공한다. 컴포넌트 타입을 재정하거나 변경하는 것이 용이해진다.
* `React.ReactNode`: `ReactElement`와 원시 타입을 포함한 여러 타입을 포함하고 있다.

포함 관계: `React.ReactNode` > `React.ReactElement` > `JSX.Element`

```ts
// ReactElement 타입의 구조
interface ReactElement<
  P = any,
  T extends string | JSXElementConstructor<any> =
    | string
    | JSXElementConstructor<any>
> {
  type: T;
  props: P;
  key: Key | null;
}
```

```ts
// 글로벌 네임스페이스에 정의된 JSX.Element 타입
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}
```

```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactFragment = {} | Iterable<ReactNode>;
// ReactNode 타입의 구조
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

&nbsp;

### 5. ReactElement, ReactNode, JSX.Element 활용하기

세 타입 모두 리액트의 요소를 나타내는 타입이다. 세 가지 타입의 차이점과 어떤 상황에서 사용하면 좋을지 알아보자.

&nbsp;

### `ReactElement`

`createElement`는 리액트 엘리먼트를 생성하는 메서드다. JSX는 이 메서드를 호출하기 위한 문법이다.

JSX는 리액트 엘리먼트를 생성하기 위한 문법으로, 트랜스파일러에 의해 리액트 엘리먼트가 다음과 같은 코드로 변환되어 생성된다.

```ts
const element = React.createElement(
  "h1",
  { className: "greeting" },
  "Hello, world!"
);

// 주의: 다음 구조는 단순화되었다
const element = {
  type: "h1",
  props: {
    className: "greeting",
    children: "Hello, world!",
  },
};

declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

&nbsp;

### `ReactNode`

`ReactNode`에 포함된 `ReactChild`에 대해 먼저 알아보자.

```ts
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
```

`ReactChild`는 `ReactElement` 타입에 `string`과 `number` 타입을 추가한 타입이다.

```ts
type ReactFragment = {} | Iterable<ReactNode>; // ReactNode의 배열 형태
type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

`ReactNode`는 `ReactChild`를 포함해 `boolean`, `null`, `undefined` 등 리액트의 `render` 함수가 반환할 수 있는 모든 형태를 포함하고 있다.

참고: `ReactPortal`은 `createPortal` 메서드의 반환 타입이다. 구조는 아래와 같다.

> `createPortal` 메서드는 일부 children을 DOM의 다른 부분으로 렌더링할 수 있게 해준다. [리액트 공식 문서의 createPortal 설명 보기](https://ko.react.dev/reference/react-dom/createPortal)

```ts
type ReactPortal = {
  $$typeof: Symbol | number; // Portal을 식별하는 심볼
  key: null | string;
  children: ReactNode; // Portal의 자식 요소
  containerInfo: any; // Portal이 렌더링될 DOM 노드
  implementation: any; // 렌더러 구현체
  ownerDocument?: Document; // Portal이 속한 document
};
```

&nbsp;

### `JSX.Element`

`JSX.Element` 는 `ReactElement`의 제네릭으로 props 와 타입 필드에 대해 `any` 타입을 가지도록 확장하고 있다. 즉, `JSX.Element` 는 `ReactElement`의 특정 타입으로 props와 타입 필드를 `any` 로 가지는 타입이라는 것을 알 수 있다.

&nbsp;

### 6. 사용 예시

그렇다면 세 가지 타입들을 어떤 상황에 어떤 타입을 사용하는 게 좋을까?

### `ReactNode`

리액트의 `render` 함수가 반환할 수 있는 모든 형태를 담고 있기 때문에, 어떤 타입이든 받아오게 하고 싶으면 해당 타입을 사용한다. prop으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 하고 싶을 때 유용하다.

```ts
type PropsWithChildren<P = unknown> = P & {
  children?: ReactNode | undefined;
};

interface MyProps {
  // ...
}

type MyComponentProps = PropsWithChildren<MyProps>;
```

&nbsp;

### `JSX.Element`

`ReactElement`의 props와 타입 필드가 `any` 타입인 타입이다. 때문에 리액트 엘리먼트를 prop으로 전달받아 `render` props 패턴으로 컴포넌트를 구현할 때 유용하게 사용한다.

```ts
interface Props {
  icon: JSX.Element;
}

const Item = ({ icon }: Props) => {
  // prop으로 받은 컴포넌트의 props에 접근할 수 있다
  const iconSize = icon.props.size;

  return <li>{icon}</li>;
};

// icon prop에는 JSX.Element 타입을 가진 요소만 할당할 수 있다
const App = () => {
  return <Item icon={<Icon size={14} />} />;
};
```

`icon`을 `JSX.Element` 타입으로 선언함으로써 이 prop에는 JSX 문법만 삽입할 수 있다. 또, `icon.props`에 접근하여 prop으로 넘겨받은 컴포넌트의 상세 데이터를 가져올 수 있다.

&nbsp;

### `ReactElement`

`JSX.Element`에서 확장하여 타입 추론 관점에서 더 유용하게 활용할 수 있다. 원하는 컴포넌트의 props를 `ReactElement`의 제네릭으로 지정해줄 수 있다. `any`로 표현된 `JSX.Element`의 제네릭 인자에 props 타입을 넣어 정확한 타입으로 명시해줄 수 있다.

```ts
interface IconProps {
  size: number;
}

interface Props {
  // ReactElement의 props 타입으로 IconProps 타입 지정
  icon: React.ReactElement<IconProps>;
}

const Item = ({ icon }: Props) => {
  // icon prop으로 받은 컴포넌트의 props에 접근하면, props의 목록이 추론된다
  const iconSize = icon.props.size;

  return <li>{icon}</li>;
};
```

이렇게 코드를 작성하면 `icon.props`에 접근할 때 어떤 props가 있는지 추론되어 IDE에 표시된다.

&nbsp;

### 7. 리액트에서 기본 HTML 요소 타입 활용하기

리액트에서 기존의 HTML 태그를 확장하여 새로운 컴포넌트를 만들 때, 태그에 내장된 onClick과 같은 이벤트 핸들러를 지원하게 만들어줘야 일관성, 편의성을 모두 챙길 수 있다. 기존의 HTML 태그의 속성 타입을 활용하여 타입을 지정하는 방법을 알아보자.

&nbsp;

### `DetailedHTMLProps`와 `ComponentWithoutRef`

HTML 태그의 속성 타입을 활용하는 대표적인 2가지 방법은 리액트의 두 가지 타입을 활용하는 것이다.

* `React.DetailedHTMLProps`

```ts
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;
// 인덱스드 액세스 타입을 사용해서 타입 추론
type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
};
```

* `React.ComponentWithoutRef`

```ts
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
};
```

`NativeButtonType["onClick"]`은 `MouseEventHandler` 타입의 `onClick` 속성을 가져온다.

&nbsp;

### 언제 `ComponentPropsWithoutRef`를 사용하면 좋을까?

`ComponentPropsWithoutRef`말고도 `HTMLProps`, `ComponentPropsWithRef` 등 HTML 태그의 속성을 지원하기 위한 다양한 타입들이 존재한다.

여기서는 컴포넌트의 props로서 HTML 태그 속성을 확장하고 싶을 때의 상황에 초점을 맞춰서 생각해본다.

button 태그를 확장해서 컴포넌트를 만드려면 기존 button 태그가 가지고 있는 HTML 속성을 props로 받을 수 있어야 한다.

```ts
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

const Button = (props: NativeButtonProps) => {
  return <button {...props}>버튼</button>;
};
```

`HTMLButtonElement`의 속성을 모두 props로 받아 button 태그에 전달해주었다. 그런데 `ref`로 props를 받을 때 주의할 점이 있다. 클래스 컴포넌트와 함수 컴포넌트에서의 `ref` 작동 방식에 차이가 있기 때문이다.

> 💡 ref란? 생성된 DOM 노드나 리액트 엘리먼트에 직접 접근하는 방법이다.
>
> `useRef` 훅 사용 예시:
>  1. DOM 요소에 직접 접근 및 조작
>  2. 불필요한 렌더링 방지
>  3. 렌더링 간 값 저장
>  4. setTimeout이나 setInterval의 ID를 저장하여 타이머 관리

```ts
// 클래스 컴포넌트에서의 ref 사용
class Button extends React.Component<NativeButtonType> {
  buttonRef: React.RefObject<HTMLButtonElement>;
  constructor(props: NativeButtonType) {
    super(props);
    this.buttonRef = React.createRef<HTMLButtonElement>();
  }

  componentDidMount() {
    this.buttonRef.current?.focus();
  }

  render() {
    return <button ref={this.buttonRef}>버튼</button>;
  }
}

// 함수 컴포넌트에서의 ref 사용
function Button(props) {
  const buttonRef = useRef(null);

  // useRef 사용 예시
  useEffect(() => {
      buttonRef.current.focus();
  }, []);

  return <button ref={buttonRef}>버튼</button>;
}
```

&nbsp;

`Button` 컴포넌트 역시 props로 전달된 ref를 통해 button 태그에 접근하여 DOM 노드를 조작할 수 있어야 한다.

```ts
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

// 클래스 컴포넌트
class Button extends React.Component {
  constructor(ref: NativeButtonProps["ref"]) {
    this.buttonRef = ref;
  }

  render() {
    return <button ref={this.buttonRef}>버튼</button>;
  }
}

// 함수 컴포넌트
function Button(ref: NativeButtonProps["ref"]) {
  const buttonRef = useRef(null);

  return <button ref={buttonRef}>버튼</button>;
}
```

클래스 컴포넌트의 `ref` 객체는 마운트된 컴포넌트의 인스턴스를 `current` 속성값으로 가지기 때문에 `ref`가 button 태그를 잘 바라보게 되지만, 함수 컴포넌트는 인스턴스를 생성하지 않기 때문에 `ref`가 button 태그를 바라보지 않기 때문에, `ref`가 제대로 작동하지 않게 된다.

이러한 제약 때문에 함수 컴포넌트에서도 `ref`를 전달 받을 수 있게 하는 `React.forwardRef` 메서드가 존재한다.

```ts
// forwardRef를 사용해 ref를 전달받을 수 있도록 구현
const Button = forwardRef((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});

// buttonRef가 Button 컴포넌트의 button 태그를 바라볼 수 있다
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />
    </div>
  );
};
```

`forwardRef`는 2개의 제네릭 인자를 받는데, 첫 번째는 `ref`의 타입, 두 번째는 props의 타입이다.

```ts
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

// forwardRef에 대한 타입으로 HTMLButtonElement를, props에 대한 타입으로 NativeButtonType을 정의
const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});
```

`ComponentPropsWithoutRef` 타입은 인자로 받은 속성을 모두 포함하지만, `ref` 속성만 제외해서 가져온다.

`ref`를 포함하는 `DetailedHTMLProps`, `HTMLProps`,
`ComponentpropswithRef` 타입들은 실제 동작하지 않는 `ref`를 받도록 지정되어 있기 때문에 예상치 못한 에러가 발생할 수 있다.

HTML 속성을 확장하는 props를 설계할 때는 **`ComponentPropsWithoutRef` 타입을 사용**하여 **`forwarRef`와 함께 사용될 때만 props로 전달되도록 타입을 정의**하는 것이 안전하다.