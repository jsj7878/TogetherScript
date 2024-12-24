# 8장 (JSX에서 TSX로)

## 8.1 리액트 컴포넌트의 타입

- 클래스 컴포넌트 타입
    - 훅 도입 이전에 상태 관리와 생명주기 메서드를 사용할 수 있는 유일한 방법이었음
    - 클래스 기반으로 `React.Component`를 상속받아 정의됨
    - 클래스 컴포넌트에서는 `state`를 사용하여 컴포넌트의 상태를 관리 가능
    - 생명주기 메서드 사용 가능
        - `componentDidMount`: 컴포넌트가 DOM에 마운트된 후 호출
        - `componentDidUpdate`: 컴포넌트가 업데이트된 후 호출
        - `componentWillUnmount`: 컴포넌트가 DOM에서 제거되기 직전에 호출
    - 클래스 컴포넌트가 상속 받는 `React.Component` 와 `React.PureComponent` 는 props와 상태 타입을 제너릭으로 받을 수 있으며 상태에 대한 타입 또한 지정할 수 있다.
- 함수 컴포넌트 타입
    - 함수 표현식을 사용하여 함수 컴포넌트를 선언할 때 `React.FC` 혹은 `React.VFC` 로 타입을 지정할 수 있다.
    - 최신 버전은 VFC가 삭제되고 FC에서 children이 사라졌다. (FC는 기존에 암묵적으로 children을 포함하고 있어서 해당 컴포넌트에서 children 컴포넌트를 사용하지 않아도 children props를 허용했음)\
    - 함수 컴포넌트의 타입을 지정해줄 때는 `React.FC` 혹은 props 타입을 직접 지정하여 타이핑하자
    
    > ❓ children이란?
    컴포넌트의 자식 요소를 나타내는 속성
    컴포넌트 내부에서 props.children을 사용하면 컴포넌트 태그 안의 내용을 동적으로 렌더링 할 수 있음
    예시)
    > 
    > 
    > ```tsx
    > function Wrapper(props) {
    >     return <div className="wrapper">{props.children}</div>;
    > }
    > 
    > function App() {
    >     return (
    >         <Wrapper>
    >             <h1>Hello, World!</h1>
    >             <p>This is a child element.</p>
    >         </Wrapper>
    >     );
    > }
    > ```
    > 
- Children props 타입 지정
    - 가장 보편적인 children 타입은 `ReactNode | undefined` 이다.
    - ReactNode 는 ReactElement 외에도 boolean, number 등 여러 타입을 포함하고 있는 타입으로 구체적으로 타이핑하는 용도에는 적합하지 않다.
    - 특정 요소만 허용하고 싶을 때는 children에 대해 추가적으로 타이핑해줘야 한다.
    

> ❓클래스 컴포넌트에 비해 함수 컴포넌트가 갖는 이점이 뭘까?

1. **간결하고 읽기 쉬운 코드**
> 
> - **클래스 컴포넌트**는 상태와 생명주기 메서드를 사용하려면 장황한 코드가 필요합니다. 특히 `constructor`와 `this` 바인딩 등 추가적인 코드가 포함됩니다.
> - **함수 컴포넌트**는 Hooks 덕분에 상태 관리와 생명주기 로직을 훨씬 간단하게 구현할 수 있습니다.
> 
> 2. **Hooks의 강력한 기능**
> 
> - React의 Hooks는 함수 컴포넌트에서도 상태 관리와 생명주기 로직을 간단히 처리할 수 있도록 합니다.
> - Hooks는 상태와 생명주기 로직을 **필요한 곳에서만 사용**할 수 있어 더 유연한 구조를 제공합니다.
> 
> 3. **로직 재사용성 개선**
> 
> - 클래스 컴포넌트에서 로직을 재사용하려면 고차 컴포넌트(HOC)나 Render Props 패턴을 사용해야 했습니다. 이 방식은 코드가 복잡해지고 중첩이 많아지는 단점이 있었습니다.
> - Hooks는 **커스텀 Hook**을 통해 로직을 간단히 추출하고 재사용할 수 있습니다.

- render 메서드와 함수 컴포넌트의 반환 타입
    - React.ReactElement
        - React.createElement의 반환 타입
        - react는 실제 DOM이 아니라 가상의 DOM을 기반으로 렌더링하는데 이때 가상 DOM의 엘레먼트를 ReactElement 형태로 저장한다.
        - 리액트 컴포넌트를 객체 형태로 저장하기 위한 포맷
    - JSX.Element
        - ReactElement를 확장하고 있는 타입
        - 글로벌 네임 스페이스에 정의되어 있어 외부 라이브러리에서 컴포넌트 타입을 재정의 할 수 있는 유연성을 제공함
        - 컴포넌트 타입을 재정의하거나 변경하는 것이 용이해짐
        - ReactNode
            - React에서 렌더링 할 수 있는 모든 요소를 포함하는 타입
            - React 컴포넌트가 반환하거나 props로 전달 될 수 있는 모든 데이터 유형을 포괄하는 타입
    - 포함관계는 다음과 같다
        - ReactNode > ReactElement > JSX.Element
- 활용하기
    - ReactElement
        - JSX의 createElement 메서드 호출로 생성된 리액트 엘리먼트 타입
    - ReactNode
        - ReactChild 외에 boolean, null, undefined 등 훨씬 넓은 범주의 타입을 포함함
        - 리액트의 render 함수가 반환할 수 있는 모든 형태
    - JSX.Element
        - ReactElement의 특정 타입으로 props와 타입 필드에 대해 any 타입을 가지는 타입
- 사용 예시
    - ReactNode
        - 리액트의 render 함수가 반환할 수 있는 모든 형태를 담고 있기 때문에 리액트 컴포넌트가 가질 수 있는 모든 타입을 의미한다.
        - children props에 어떤 타입이든 지정하고 싶으면 ReactNode 타입으로 children을 지정하자
    - JSX.Element
        - props와 타입 필드가 any인 리액트 엘리먼트를 나타냄
        - 리액트 엘리먼트를 prop으로 받아 render props 패턴으로 컴포넌트를 구현할 때 유용하게 활용할 수 있음
        - prop에 JSX문법만 삽입하거나 props에 접근하여 넘겨받은 컴포넌트의 상세한 데이터를 가져올 수 있음
    - ReactElement
        - 원하는 컴포넌트의 props를 ReactElement의 제네릭으로 사용할 수 있음
- HTML 요소 타입 활용하기
    - HTML 태그의 속성 타입을 활용하는 방법은 2가지가 있다
    - DetailedHTMLProps
        - 쉽게 HTML 태그 속성과 호환되는 타입을 선언할 수 있음