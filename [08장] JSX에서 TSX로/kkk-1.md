```jsx
8.1 리액트 컴포넌트의 타입
8.1.1 클래스 컴포넌트 타입
8.1.2 함수 컴포넌트 타입
8.1.3 Children props 타입 지정
8.1.4 render 메서드와 함수 컴포넌트의 반환 타입
			 - React.ReactElement vs JSX.Element vs Reactt.ReactNode
8.1.5 ReactElement, ReactNode, JSX.Element 활용하기
8.1.6 사용 예시
8.1.7 리액트에서 기본 HTML 요소 타입 활용하기

8.2 타입스크립트로 리액트 컴포넌트 만들기
8.2.1 JSX로 구현된 Select 컴포넌트
8.2.2 JSDocs로 일부 타입 지정하기
8.2.3 props 인터페이스 적용하기
8.2.4 리액트 이벤트
8.2.5 훅에 타입 추가하기
8.2.6 제네릭 컴포넌트 만들기
8.2.7 HTMLAttributes, ReactProps 적용하기
8.2.8 styled-components를 활용한 스타일 정의
8.2.9 공변성과 반공변성
```

# 8.1 리액트 컴포넌트의 타입

@types/react 패키지에 정의된 리액트 내장타입들. 이들을 구분하고 학습.

헷갈릴 수 있는 대표적인 리액트 컴포넌트 타입을 살펴보면서 상황에 따라 어떤 것을 사용하면 좋을지, 사용할 때의 유의점은 무엇일지 알아보자.

## 8.1.1 클래스 컴포넌트 타입

인터페이스는 리액트 클래스 컴포넌트의 생명 주기 메서드 ComponentLifecycle를 상속받는다. 

리액트 라이브러리 내장 클래스인 Component와, PureComponent

### ▶️  React.Component

제네릭 2개(SS 옵셔널)를 받는 기본 클래스 기반 컴포넌트

- `shouldComponentUpdate` 를 제공하지 않아 모든 업데이트에서 렌더링이 발생한다.
- 사용자가 직접 `shouldComponentUpdate` 를 구현해 성능 최적화 할 수 있다.

### ▶️  React.PureComponent

내부적으로 `shouldComponentUpdate`를 구현해  성능 최적화

- 얕은 비교 shallow compare를 통해 props와 state의 변경여부를 확인한다.
- `shouldComponentUpdate` 를 통해 props와 state가 변경되지 않으면 렌더링을 방지한다.

왜 쓰나요? 불필요한 렌더링을 방지하기 위해 사용합니다. 

```tsx
// 가장 기본적인 클래스 기반 컴포넌트
interface Component<P = {}, S = {}, SS = any>
  extends ComponentLifecycle<P, S, SS> {}

class Component<P, S> {
	/* 제네릭 
	P는 부모에서 받는 props의 타입
	S는 컴포넌트가 내부적으로 관리하는 state 타입
  */
}

class PureComponent<P = {}, S = {}, SS = any> extends Component<P, S, SS> {
/* 내부적으로 shouldComponentUpdate 메서드 정의 */}

```

**shouldComponentUpdate 코드**

```tsx
shouldComponentUpdate(nextProps, nextState) {
  return !shallowEqual(this.props, nextProps) || !shallowEqual(this.state, nextState);
}
```

사용예제

```tsx
interface WelcomeProps {
  name: string;
}

class Welcome extends React.Component<WelcomeProps> {
  /* ... 생략 */
}
```

상태가 있는 컴포넌트일 때는 제네릭의 두번째 인자로 타입을 넘겨주면 상태에 대한 타입을 지정할 수 있다.

## 8.1.2 함수 컴포넌트 타입

### ▶️ React.FC와 React.VFC

함수 표현식을 사용해 함수 컴포넌트를 선언할 때 **함수 컴포넌트의 타입 지정을 위해 제공되는 타입**이다. FC는 FunctionComponent의 약자이다. 

FC가 먼저 등장하고 @types/react 16.9.4버전에서 VFC가 추가되었다. 

~~❓**어떤 차이가 있는가?**~~

~~children이라는 타입을 허용하는지 아닌지에 따른 차이가 있다. FC는 암묵적으로 children을 포함하고 있기 때문에 해당 컴포넌트에서 children을 사용하지 않더라도 children props를 허용한다.~~ 

~~따라서 children props가 필요하지 않은 컴포넌트에서는 더 정확한 타입 지정을 위해 VFC를 많이 사용한다.~~

리액트 v18로 넘어오면서 VFC가 삭제되고 FC에서는 children이 사라졌다. 그래서 FC에서 props 타입, 반환 타입을 지정하는 형태만 가능하다. **이제 children을 사용하려면 명시적으로 props에 추가해야 한다.**

```tsx
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

## 8.1.3 Children props 타입 지정

`PropsWithChildren`은 리액트의 유틸리티 타입으로 **props에 children을 포함하는 타입**을 정의할 때 사용된다.

```tsx
type PropsWithChildren<P> = P & { children?: ReactNode | undefined };
```

가장 보편적인 children 타입은 ReactNode | undefined가 된다. 

```tsx
import React, { PropsWithChildren } from 'react';

type MyProps = {
  title: string;
};

const MyComponent: React.FC<PropsWithChildren<MyProps>> = ({ title, children }) => {
/// ...
// 기존 MyProps 타입에 children 추가, title은 필수, children은 선택이 된다.

// children만 사용
const Container: React.FC<PropsWithChildren<{}>> = ({ children }) => {
```

### ❓ReactNode 타입이 뭔가요

ReactElement 외에도 boolean, number 등 여러 타입을 포함하고 있는 타입이다. 

> 리액트노드는 리액트에서 렌더링 가능한 모든 요소를 포괄하는 타입이다. 리액트 컴포넌트가 반환할 수 있는 모든 유형을 포함한다.
> 

그렇다면 특정 타입만 허용하고 싶을 때는 어떻게 해야 하는가?

개발자가 직접 유니온으로 제한해야 한다.

```tsx
// example 1
type WelcomeProps = {
  children: "천생연분" | "더 귀한 분" | "귀한 분" | "고마운 분";
};
const Welcome: React.FC<WelcomeProps> = ({ children }) => <h1>{children}</h1>;

<Welcome children="천생연분" />; // ✅ 허용
<Welcome children="감사한 분" />; // ❌ 오류: 허용되지 않은 문자열

```

```tsx
// example 2
type WelcomeProps = { children: string };

// example 3
type WelcomeProps = { children: ReactElement };
```

## 8.1.4 render 메서드와 함수 컴포넌트의 반환 타입

“React.ReactElement vs JSX.Element vs React.ReactNode”

> 요약 : 3개의 포함관계, 리턴 타입
> 

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/3529de1c-a19d-49ac-aa2d-b4bf2e3fafce/66749e76-2e64-4c2c-bd0d-1ad2f498a783/image.png)

### ▶️ ReactElement 타입

**정의**

```tsx
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

React.createElement를 호출하는 형태의 구문으로 변환하면 반환 타입은 ReactElement이다. 리액트는 실제 DOM이 아니라 가상의 DOM을 기반으로 렌더링하는데 가상 DOM의 엘리먼트는 ReactElement 형태로 저장된다. 즉 “ReactElement” 타입은 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷이다.

???

**ReactElement란 무엇인가?**

> 리액트 컴포넌트를 표현하기 위핸 객체 포맷이다. 이 객체는 리액트의 가상돔을 구성하는 기본 단위이며, 리액트는 이르 이용해 실제 돔을 효율적으로 업데이트한다.
> 

 리액트에서는 JSX를 사용해 UI를 정의하지만 내부적으로는 JSX가 `React.createElement` 함수로 변환된다. **(JSX는 React.createElement 함수로 변환된다.)**

그리고 해당 함수가 반환하는 타입은 `ReactElement`이다. 이 타입=객체는 가상 돔을 구성하는 최소 단위이다. 

```tsx
const element = <h1>Hello</h1>;
// 내부적으로 변환되는 형태
const element = React.createElement('h1', null, 'Hello'); // 리턴 타입이 ReactElement

```

### ▶️  JSX.Element 타입

리액트의 ReactElement를 확장하고 있는 타입이며, 글로벌 네임스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의할 수 있는 유연성을 제공한다. (타입 재정의, 변경에 편리)

```tsx
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {}
  }
}
```

### ▶️ ReactNode 타입

ReactElement 외에도 다양한 타입을 포함하고 있다. 

```tsx
type ReactText = string | number;
type ReactChild = ReactElement | ReactText;
type ReactFragment = {} | Iterable<ReactNode>;

type ReactNode =
  | ReactChild
  | ReactFragment
  | ReactPortal
  | boolean
  | null
  | undefined;
```

## 8.1.5 ReactElement, ReactNode, JSX.Element 활용하기

3가지 타입 모두 리액트의 요소를 나타내는데 무슨 차이가 있을까? 왜 3개나 있찌? 그에 대한 비교와 어떤 상황에 무엇이 적절한지에 대해 설명한다.

### ⏩ ReactElement

```tsx
// **@types/react 패키지 정의 타입 일부**
declare namespace React {
  // ReactElement
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
...
```

JSX가 리액트 엘리먼트를 생성하는 createElement 메서드를 호출하기 위한 문법이다.

<aside>
🔬

**JSX**

JSX는 자바스크립트의 확장 문법으로 리액트에서 UI를 표현하는 데 사용된다. XML과 비슷한 구조로 되어 있으며 리액트 컴포넌트를 선언하고 사용할 때 더욱 간결하고 가독성 있게 코드를 작성할 수 있도록 도와준다. 또한 HTML과 유사한 문법을 제공하여 리액트 사용자에게 렌더링 로직(마크업)을 쉽게 만들 수 있게 해주고, 컴포넌트 구조와 계층 구조를 편리하게 표현할 수 있도록 해준다.

</aside>

JSX는 리액트 엘리먼트를 생성한다. 트랜스 파일러는 jsx 문법을 createElement 함수로 변환, 이 함수는 리액트엘리먼트를 생성한다.

```tsx
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

리액트는 이렇게 만들어진 리액트 엘리먼트 객체를 읽어서 돔을 구성한다. 

리액트에는 여러 개의 createElement 오버라이딩 메서드가 존재한다. 이들의 리턴 타입은 ReactElemnt 타입을 기반으로 한다. 

> ReactElement 타입은 JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트를 나타내는 타입이다.
> 

### ⏩ ReactNode

```tsx
// **@types/react 패키지 정의 타입 일부**
declare namespace React {
...
  // ReactNode
  type ReactText = string | number;
  type ReactChild = ReactElement | ReactText;
  type ReactFragment = {} | Iterable<ReactNode>; // ReactNode의 배열 형태

  type ReactNode =
    | ReactChild
    | ReactFragment
    | ReactPortal
    | boolean
    | null
    | undefined;
  type ComponentType<P = {}> = ComponentClass<P> | FunctionComponent<P>;
}
...
```

ReactChild 타입이 ReactElement보다 넓은 범위를 가진다. 

ReactNode는 ReactChild보다도 더 넓은 타입을 포함한다. 

즉 ReactNode는 render 함수가 반환할 수 있는 모든 형태를 담고 있다.

### ⏩  JSX.Element

```tsx
// **@types/react 패키지 정의 타입 일부**
declare namespace React {
...
// JSX.Element
declare global {
  namespace JSX {
    interface Element extends React.ReactElement<any, any> {
      // ...
    }
    // ...
  }
}
```

JSX.element는 ReactElement의 제네릭으로 props와 타입 필드에 대해 any 타입을 가지도록 확장하고 있다. 즉 JSX.Element는 ReactElement의 특정 타입으로 props와 타입 필드를 any로 가지는 타입이다. 

## 8.1.6 사용 예시

3가지 타입을 알아봤는데 어떤 상황에 어떤 타입을 사용해야 좋을까?

1. **ReactNode**

리액트의 렌더 함수가 반환할 수 있는 모든 형태를 담고 있다. 즉 리액트 컴포넌트가 가질 수 있는 모든 타입을 의미한다.

> **리액트의 컴포지션 모델이란?**
컴포넌트를 구성하는 기본적인 방식 중 하나로, 컴포넌트를 조합하여 재사용성을 높이고, 유연한 UI 구조를만들 수 있게 한다. 이를 통해 컴포넌트 간 상속 대신 합성을 사용해 기능을 확장하거나 조합할 수 있다.
props.children은 합성을 지원하는 기본적인 방법이다.
> 

리액트의 자식을 사용하면 기존 타입이 아래와 같다.

```tsx
interface MyComponentProps { 
children?: React.ReactNode;
  // ...
}
```

맞다. 이 때의 타입이 리액트 노드였다!! 이렇게 할 경우 어떤 타입이든지 자식으로 들어올 수 있다.

> 리액트 내장 타입인 PropsWithChildren 타입도 ReactNode 타입으로 children을 선언하고 있다.
> 

```tsx
type PropsWithChildren<P = unknown> = P & {
  children?: ReactNode | undefined;
};

interface MyProps {
  // ...
}

type MyComponentProps = PropsWithChildren<MyProps>;
```

**리액트노드 타입은 prop으로 리액트 컴포넌트가 다양한 형태를 가질 수 있게 할 때 사용된다.**

1. **JSX.Element**

props와 타입 필드가 any 타입인 리액트 엘리먼트를 나타낸다. 이러한 특정 때문에 리액트 엘리먼트를 props로 전달받아 **render props 패턴**으로 컴포넌트를 구현할 때 유용하게 활용할 수 있다.

```tsx
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

아이콘에는 JSX 문법만 삽입할 수 있게 된다. 또 icon.props에 접근해 props으로 넘겨받은 컴포넌트의 상세한 데이터를 가져올 수 있다.

1. **ReactElement**

위의 아이콘 타입에서 확장해 추론 고나점에서 더 유용하게 쓸 수 있는 방법은 무엇일까?

JSX.Element 대신에 ReactElement를 사용하는 것이다. 이 때 원하는 컴포넌트의 props를 ReactElement의 제네릭으로 지정해 줄 수 있다. 

만약 JSX.Element가 ReactElement의 props 타입으로 any가 지정되었다면, ReactElement 타입을 활용해 제네릭에 직접 해당 컴포넌트의 props 타입을 명시해준다.

이렇게 작성할 경우 icon.props에 접근할 때 어떤 props가 있는지 추론되어 IDE에 표시된다.

```tsx
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

## 8.1.7 리액트에서 기본 HTML 요소 타입 활용하기

```tsx
const SquareButton = () => <button>정사각형 버튼</button>;
```

button 태그를 확장해서 나만의 컴포넌트를 만드는 것처럼 (기존 태그의 메서드도 활용 가능하도록)

기존 HTML 태그의 속성 타입을 활용해 타입을 지정하는 방법.

### ▶️ DeatiledHTMLProps와 ComponentWithoutRef

html 태그의 속성 타입을 활용하는 대표적인 2가지 방법은 리액트의 저 제목 2가지 타입을 활용하는 것이다. 

**React.DeatiledHTMLProps 활용하기**

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

type ButtonProps = {
  onClick?: NativeButtonProps["onClick"];
};
```

> ButtonProps의 onClick 타입은 실제 html, button 태그의 onClick 이벤트 핸들러 타입과 동일하게 할당되는 것을 확인할 수 있다.
> 

**React.ComponentWithoutRef**

```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;
type ButtonProps = {
  onClick?: NativeButtonType["onClick"];
};
```

> 마찬가지로 리액트의 button onClick 이벤트 핸들러에 대한 타입이 할당된 것을 볼 수 있다.
> 

### ❓언제 **ComponentWithoutRef를 사용하면 좋을까?**

이외에도 HTMLProps, ComponentPropsWithRef 등 HTML 태그의 속성을 지원하기 위한 다양한 타입이 있다. 여러 타입이 지원되다 보니 어떤 상황에 무엇을 해야할지 혼란이 온다.

여기서는 컴포넌트의 props로서 html 태그 속성을 확장하고 싶을 때의 상황에 초점을 맞춰보자.

> html 버튼 태그와 동일한 역할을 하지만 커스텀한 ui를 적용해 재사용성을ㄹ 높이기 위한 버튼 컴포넌트를 만든다고 가정
> 

```tsx
const Button = () => {
  return <button>버튼</button>;
};
```

기존 버튼 태그의 속성을 props로 받아야 함

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

const Button = (props: NativeButtonProps) => {
  return <button {...props}>버튼</button>;
};
```

여기까지의 문제는 ref를 props로 받을 경우 고려해야 할 사항이 있다.

- ref가 머야?
    
    ```tsx
    // ref
    // 생성된 DOM 노드나 리액트 엘리먼트에 접근하는 방법으로 아래와 같이 사용된다.
    
    // 클래스 컴포넌트
    class Button extends React.Component {
      constructor(props) {
        super(props);
        // 버튼 요소를 참조하기 위한 ref 생성
        this.buttonRef = React.createRef();
      }
    
      render() {
        return <button ref={this.buttonRef}>Click Me</button>;
      }
    }
    
    // 함수 컴포넌트
    function Button(props) {
      // 버튼 요소를 참조하기 위한 ref 생성
      const buttonRef = useRef(null);
      return <button ref={buttonRef}>Click Me</button>;
    }
    
    export default Button;
    ```
    

컴포넌트 내에서 ref를 활용해 생성된 DOM 노드에 접근하는 것과 마찬가지로, 

재사용할 수 있는 Button 컴포넌트 역시 props로 전달된 ref를 통해 button 태그에 접근하여 DOM 노드를 조작할 수 있을 것을 예상된다.

> 리액트에서 **ref**를 사용해 커스텀 컴포넌트의 내부 DOM 요소에 접근할 수 있는 메커니즘이 뭔가?
> 

여기까지 기존 html 버튼 컴포넌트를 개발자가 커스텀한 새로운 버튼 컴포넌트가 있다. 기존 것의 메서드 onClick 등을 커스텀한 것에서도 사용하기 위해 props를 받아야 하는데 이를 위한 타입 선언을 해준다. 

그런데!

여기서 만약 커스텀한 컴포넌트의 props에 `ref` 가 들어갈 경우???

ref는 원래 돔 요소에 직접 접근하는 건데 그럼 원본 컴포넌트에 접근하는 건가? 어떻게 될까?

### 🔵 클래스 컴포넌트와 함수 컴포넌트는 ref를 다른 방식으로 전달한다?

~~클래스 컴포넌트에서는 ref를 props로 전달받아 저장한다. NativeButtonProps[”ref”]를 사용하여 ref의 타입을 지정한다. 이를 클레스의 멤버 변수 buttonRef에 저장하고~~ 

~~jsx에서 ref={this.buttonRef}로 ref를 버튼 태그에 연결한다.!~~

~~함수형 컴포넌트에서는 ref를 props로 받아 버튼 태그에 연결한다.~~

**전체코드**

```tsx
type NativeButtonProps = React.DetailedHTMLProps<
  React.ButtonHTMLAttributes<HTMLButtonElement>,
  HTMLButtonElement
>;

// 클래스 컴포넌트
class Button extends React.Component {
  constructor(ref: NativeButtonProps["ref"]) {
    // this.buttonRef = ref;
    super(props);
    this.buttonRef = props.ref;
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

🟡 **클래스 컴포넌트? 함수 컴포넌트?**

클래스 컴포넌트에서는 props로 전달된 ref가 버튼 컴포넌트의 버튼 태그를 그대로 바라보게 된다.

```tsx
// 클래스 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
class WrappedButton extends React.Component {
  constructor() {
    this.buttonRef = React.createRef();
  }

  render() {
    return (
      <div>
        <Button ref={this.buttonRef} /> 전달된 ref는 컴포넌트의 인스턴스=current를 가리킨다.
      </div>
    );
  }
}
// 함수 컴포넌트로 만들어진 Button 컴포넌트를 사용할 때
const WrappedButton = () => {
  const buttonRef = useRef();

  return (
    <div>
      <Button ref={buttonRef} />{" "}
    </div>
  );
};
```

함수 컴포넌트에서는 그렇지 않다! 전달받은 ref가 버튼 컴포넌트의 버튼 태그를 바라보지 않는다!

왜? 클래스 컴포넌트에서 ref 객체는 마운트된 컴포넌트의 인스턴스를 current 속성값으로 가지지만 

함수 컴포넌트에서는 생성된 인스턴스가 없기 때문에 ref에 기대한 값이 할당되지 않는 것이다. 

> 함수형 컴포넌트는 인스턴스가 없다. ref를 직접 props로 전달해도 돔 요소에 연결되지 않는다.
> 

이러한 제약을 극복하고 함수 컴포넌트에서도 ref를 전달받을 수 있도록 하는 방법?

React.forwardRef 메서드를 사용한다!

### ❗React.forwardRef 메서드

함수 컴포넌트에서도 ref를 전달받을 수 있게 한다! 

```tsx
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

forwardRef는 2개의 제네릭 인자를 받을 수 있다. 첫 번째는 ref에 대한 타입 정보이며 두 번째는 props에 대한 타입 정보이다. 앞 코드에서 버튼 컴포넌트에 대한 forwardRef의 타입 선언은 어떻게 할까?

```tsx
type NativeButtonType = React.ComponentPropsWithoutRef<"button">;

// forwardRef의 제네릭 인자를 통해 ref에 대한 타입으로 HTMLButtonElement를, props에 대한 타입으로 NativeButtonType을 정의했다
const Button = forwardRef<HTMLButtonElement, NativeButtonType>((props, ref) => {
  return (
    <button ref={ref} {...props}>
      버튼
    </button>
  );
});
```

NativeButtonType : 이전과 다르게 `ComponentPropsWithoutRef`타입을 사용했다. 이렇게 타입을 작성하면 **버튼 태그에 대한 html 속성을 모두 포함하지만, ref 속성은 제외된다.**

이러한 특징 때문에 DetailedHTMLProps, HTMLProps, ComponentPropsWithRef와 같이 ref 속성을 포함하는 타입과는 다르다.

함수 컴포넌트의 프롭으로 위의 3개처럼 ref를 포함하는 타입을 사용하게 되면!

실제로는 동작하지 않는데 ref를 프롭으로 받을 수  있다. 

따라서 html 속성을 확장하는 프롭을 설계할 때는 `ComponentPropsWithoutRef` 타입을 사용해 ref가 실제로 forwardRef와 함께 사용될 때만 props로 전달되도록 타입을 정의하는 것이 안전하다.

<aside>
🔬

**key**

`ComponentPropsWithoutRef` 타입은 ref 속성을 제외한다. html 속성을 확장한 컴포넌트에서 props를 받아야 할 때는 이 타입을 사용해야 ref를  사용할 수 없도록 해줘야 한다! 왜냐면 ref가 프롭에 들어가면 어차피 동작하지 않으니께..

</aside>