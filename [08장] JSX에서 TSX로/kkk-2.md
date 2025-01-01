```jsx
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

# 8.2 타입스크립트로 리액트 컴포넌트 만들기

8.1에서 리액트 컴포넌트의 타입에 대해 알아보았다.

타입스크립트의 타입 시스템으로 어떤 타입의 속성이 제대로 전달되지 않았을 경우 에러를 표시해 안정성을 높일 수 있다. 

이 절에선느 타입스크립트로 Select 컴포넌트를  구현하면서 타입스크립트의 장ㅇ점을 정리한다.

## 8.2.1 JSX로 구현된 Select 컴포넌트

```java
const Select = ({ onChange, options, selectedOption }) => {
  const handleChange = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <select
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </select>
  );
};
```

options는 객체로 해당 객체를 순회하면서 key, value 쌍을 map, 순회한다. 

이 때 options는 어떤 객체인지 알 수 없다. 어떻게 알려줄 수 있을까?

## 8.2.2 JSDocs로 일부 타입 지정하기

컴포넌트의 속성 타입을 명시하기 위해 JSDocs를 사용할 수 있다.

```java
/**
* Select 컴포넌트
* @param {Object} props - Select 컴포넌트로 넘겨주는 속성
* @param {Object} props.options - { [key: string]: string } 형식으로 이루어진 option 객체
* @param {string | undefined} props.selectedOption - 현재 선택된 option의 key값 (optional)
* @param {function} props.onChange - select 값이 변경되었을 때 불리는 callBack 함수 (optional)
* @returns {JSX.Element} 
*/
const Select = //...

```

## 8.2.3 props 인터페이스 적용하기

JSDocs의 한계

- 각 속성의 대략적인 타입과 어떤 역할을 하는지 파악할 수 있다.
- 그러나 위에서는 options가 구체적으로 어떤 역할을 하는지, 안의 키와 값이 어떤 의미인지 등 구체적인 정보를 다른 개발자는 알기 어렵다.
- 또 onChange의 매개변수 및 리턴 값에 대한 구체적인 정보를 알기 어렵다. 잘못된 타입이 전달될 수 있다.

어떻게 해결?

타입스크립트를 사용해 더 정교하고 구체적인 타입을 지정해서 해결한다.

코드에서 Option의 구체적인 타입을 지정해주고 onChange도 파라미터와 리턴 타입을 명시해주었다. 

```tsx
// 270p
type Option = Record<string, string>; // {[key: string]: string}

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void; // 파라미턴, 리턴값 타입 명시
}

const Select = ({
  options,
  selectedOption,
  onChange,
}: SelectProps): JSX.Element => {
  //...
};
```

예시

```tsx
// 270p
interface Fruit {
  count: number;
}

interface Param {
  [key: string]: Fruit; // type Param = Record<string, Fruit>과 동일
}

const func: (fruits: Param) => void = ({ apple }: Param) => {
// 매개변수는 apple만 구조분해할당해서 가져온다., 애플이 있다는 가정 하에 작성된 코드이다.
  console.log(apple.count);
};

// OK.인
func({ apple: { count: 0 } });

// Runtime Error (Cannot read properties of undefined (reading 'count'))
func({ mango: { count: 0 } });

func({ apple: { count: 1 }, mango: { count: 2 } }); // 출력: 1
// 망고는 함수에서 사용안됨.
```

## 8.2.4 리액트 이벤트

이벤트 리스너에는 DOM element에 등록되어 처리하는 이벤트 리스너가 있고 (브라우저 네이티브 DOM 이벤트)

리액트 컴포넌트에 등록되는 이벤트 리스너가 있다. 두 개의 동작과정이 완전히 동일하지 않다. 일부 차이가 있다.

### *️⃣ 리액트의 이벤트 시스템 - SyntheticEvent

리액트는 자체적인 합성이벤트 시스템을 사용해 이벤트를 관리하는데, 이 시스템은 브라우저의 네이티브 DOM 이벤트를 감싸는 Wrapper이다.

리액트 이벤트 핸들러 등록은 카멜케이스를 사용한다. (onClick)

**장점** ✅

이러한 래퍼를 통해 감싸는 시스템을 통해 브라우저마다 다르게 동작하는 이벤트를 표준화해 일관되게 동작시킨다. 또 성능 최적화를 위해 이벤트 객체를 재활용한다. (이벤트 풀링)

```tsx
//+ 네이티브 DOM
const button = document.getElementById('myButton');
button.onclick = () => alert('Clicked!');
// 소문자로 표기

// React
function App() {
  return <button onClick={() => alert('Clicked!')}>Click Me</button>;
  // 카멜케이스로 표기
}
```

**예시 : 이벤트 캡처링**

리액트의 이벤트 핸들러는 기본적으로 이벤트가 자식 요소에서 부모 요소로 전파되는 과정인 이벤트 버블링 단계에서 실행된다.

이벤트 전파는 

1. 캡처링 단계 - 이벤트가 dom의 최상위 요소 (window or document)에서 시작하여 대상에 도달하기까지 트리를 타고 내려간다.
2. 타겟 단계 - 이벤트가 실제로 대상 요소에 도달
3. 버블링 단계 - 이벤트가 대상에서 시작해 다시 dom의 상위 요소로 전파

의 3가지 단계를 거치는데 리액트 이벤트 핸들러는 기본적으로 여기서 버블링 단계에서 실행된다. 그런데 캡처링 단계에서 이벤트 핸들러를 등록시키고 싶다면 이벤트 리스터 이름 뒤에 capture를 붙여야 한다.

```tsx
//+
function App() {
  const handleBubble = () => console.log("Bubbling");
  const handleCapture = () => console.log("Capturing");

  return (
    <div onClickCapture={handleCapture} onClick={handleBubble}>
      <button>Click Me</button>
    </div>
  );
}
// 출력순서 Capturing > Bubbling
```

<ChangeEvent<HTMLSelectElement>> : html의 select 엘리먼트에서 발생하는 이벤트

```tsx
type EventHandler<Event extends React.SyntheticEvent> = (
  e: Event
) => void | null;
type ChangeEventHandler = EventHandler<ChangeEvent<HTMLSelectElement>>;

// 일반 이벤트 핸들러
const eventHandler1: GlobalEventHandlers["onchange"] = (e) => {
  e.target; // 일반 Event는 target이 없음
};

// 리액트 합성 이벤트 핸들러
const eventHandler2: ChangeEventHandler = (e) => {
  e.target; // 리액트 이벤트(합성 이벤트)는 target이 있음
};
```

```tsx
const Select = ({ onChange, options, selectedOption }: SelectProps) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return <select onChange={handleChange}>{/* ... */}</select>;
};
```

## 8.2.5 훅에 타입 추가하기

useState 함수에도 타입 매개변수를 지정해줄 수 있다. 제네릭 타입을 명시하지 않으면 타입스크립트 컴파일러가 초기값을 기준으로 타입을 자동 추론한다.

Select 컴포넌트 : 과일 선택

```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  const [fruit, changeFruit] = useState<string | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption={fruit} />
  );
};
```

초기값도 없는데 타입도 명시하지 않았다. 이 경우 undefined가 추론된다. 이 때문에 오류가 발생한다.

```tsx
// useState 타입 명시 생략
// fruit: undefined;
// changeFruit: (v: React.SetStateAction<undefined>) => void;
const [fruit, changeFruit] = useState();

return (
  <Select
    // Error - SetStateAction<undefined>와 맞지 않음
    // (changeFruit에는 undefined만 매개변수로 넘길 수 있음)
    onChange={changeFruit}
    options={fruits}
    selectedOption={fruit}
  />
);
```

> **사이드 이펙트**
프로그램의 실행 결과가 예상치 못한 상태로 변경되거나 예상치 못한 동작을 하게 되는 상황을 가리킨다. 즉, 코드의 실행이 예상과 다르게 동작하여 예상치 못한 결과를 초래하는 것을 의미한다.
> 

코드의 문제는 fruit를 string으로 주면 안된다는 것이다. apple, banana, blueberry 중에서만 선택되어야 한다.

```tsx
//const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const [fruit, changeFruit] = useState("apple");
// 이러면 타입이 string로 추론된다.

// error가 아님 > 그래서 에러 아님
const func = () => {
  changeFruit("orange");
};
```

이것도 안되지. 어떻게 해야 enum처럼 타입을 한정할 수 있을까?

```tsx
// const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

type Fruit = keyof typeof fruits; // 'apple' | 'banana' | 'blueberry';
const [fruit, changeFruit] = useState<Fruit | undefined>("apple");

// 에러 발생
const func = () => {
  changeFruit("orange");
};
```

### useState의 타입을 어떻게 줘야하나 결론

1. 초기값을 세팅해준다
2. 제네릭 타입에 keyof typeof로 키 타입 뽑아서 이걸 넣어준다.

## 8.2.6 제네릭 컴포넌트 만들기

```tsx
// 이전 코드
type Option = Record<string, string>; // {[key: string]: string}

interface SelectProps {
  options: Option;
  selectedOption?: string;
  onChange?: (selected?: string) => void; // 파라미턴, 리턴값 타입 명시
}
//
const Select = ({ onChange, options, selectedOption }: SelectProps) => {
...
```

⚠️ **문제는 selectedOption은 이름 그대로 options 중에서 현재 선택된 것을 나타내야  한다.** 즉 options에 없는 것은 타입이 되면 안된다. 현재는 string이면 다 되는 상태다.

또 현재 JSX에서 onChange의 파라미터인 changeFruit는 유니온으로 타입이 제한된 반면, 

인터페이스 타입의 onChange의 파라미터는 string이다. 여기서 타입 에러가 발생한다.

```tsx
const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select onChange={changeFruit} options={fruits} selectedOption="orange" />
  );
};
```

❓**어떻게 수정하면 될까?**

먼저 선택된 옵션들의 타입을 제한해준다. 또 onChange의 파라미터도 같은 타입으로 제한한다. 이를 제네릭으로 받아서 조금 더 안정성 있고 유연하게 동작하도록 한다.

```tsx
interface SelectProps<OptionType extends Record<string, string>> {
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  options,
  selectedOption,
  onChange,
}: SelectProps<OptionType>) => {
  // Select component implementation
};
```

이제 잘못된 타입을 전달하면 에러 발생

```tsx
const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

const FruitSelect: VFC = () => {
  // ...
  // <Select<Fruit> ... />으로 작성해도 되지만, 넘겨주는 props의 타입으로 타입 추론을 해줍니다
  // Type Error - Type "orange" is not assignable to type "apple" | "banana" | "blueberry" | undefined

  return (
    <Select options={fruits} onChange={changeFruit} selectedOption="orange" />
  );
};
```

## 8.2.7 HTMLAttributes, ReactProps 적용하기

☑️**상황 :** 리액트 컴포넌트에 기본 props를 추가하고 싶다. 직접 넣지 않고 리액트가 제공하는 타입`ComponentPropsWithoutRef`을 활용한다.

> `ComponentPropsWithoutRef` 타입은 리액트 컴포넌트의 prop 타입을 반환해주는 타입
> 

+. ComponentPropsWithoutRef는 기본 엘리먼트 타입에 사용한다. ComponentProps는 커스텀 컴포넌트에서 props 타입을 추출한다. 

1. Type[’key’]를 활용해 객체 형식의 타입 내부 속성값을 가져올 수 있다. 

```tsx
type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;
// 기본 html 엘리먼트 select

interface SelectProps<OptionType extends Record<string, string>> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  // ...
}
```

1. 여러 타입을 가져와야 한다면 Pick 키워드를 사용할 수 있다. 
    1. Pick<Type, ‘key1’ | ‘key2’ …>는 객체 형식의 타입에서 key1, key2…의 속성만 추출해 새로운 타입을 반환한다.
        
        ```tsx
        interface SelectProps<OptionType extends Record<string, string>>
          extends Pick<ReactSelectProps, "id" | "key" | /* ... */> {
          // ...
        }
        ```
        

### 8.2.8 styled-components를 활용한 스타일 정의

☑️**상황** : 컴포넌트에 글꼴 크기와 현재 선택된 글꼴 색상을 설정하도록

```tsx
// 세팅
const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
  color: {
    white: "#FFFFFF",
    black: "#000000",
  },
};

type Theme = typeof theme;
type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];
```

부모 컴포넌트에서 원하는 스타일 적용, 프롭으로 타입을 상속한다. 

```tsx
interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;
```

옵셔널 처리

```tsx
interface SelectProps extends Partial<SelectStyleProps> {
  // ...
}

const Select = <OptionType extends Record<string, string>>({
  fontSize = "default",
  color = "black",
}: // ...
SelectProps<OptionType>) => {
  // ...

  return (
    <StyledSelect
      // ...
      fontSize={fontSize}
      color={color}
      // ...
    />
  );
};
```

## 8.2.9 공변성과 반공변성

onChange처럼 객체의 메서드 타입을 정의하는 것처럼, 객체의 메서드 타입을 정의하는 방법은 2가지가 있다. 

ㅅㅂ 모르겠다 나중ㅇ ㅔ다시 280p

```tsx
interface Props<T extends string> {
  onChangeA?: (selected: T) => void;
  onChangeB?(selected: T): void;
}

const Component = () => {
  const changeToPineApple = (selectedApple: "apple") => {
    console.log("this is pine" + selectedApple);
  };

  return (
    <Select
      // Error
      // onChangeA={changeToPineApple}
      // OK
      onChangeB={changeToPineApple}
    />
  );
};
```

# 8.3 정리

최종 Select 컴포넌트

```tsx
import React, { useState } from "react";
import styled from "styled-components";

const theme = {
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
  color: {
    white: "#FFFFFF",
    black: "#000000",
  },
};

type Theme = typeof theme;

type FontSize = keyof Theme["fontSize"];
type Color = keyof Theme["color"];

interface SelectStyleProps {
  color: Color;
  fontSize: FontSize;
}

const StyledSelect = styled.select<SelectStyleProps>`
  color: ${({ color }) => theme.color[color]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize]};
`;

type ReactSelectProps = React.ComponentPropsWithoutRef<"select">;

interface SelectProps<OptionType extends Record<string, string>>
  extends Partial<SelectStyleProps> {
  id?: ReactSelectProps["id"];
  className?: ReactSelectProps["className"];
  options: OptionType;
  selectedOption?: keyof OptionType;
  onChange?: (selected?: keyof OptionType) => void;
}

const Select = <OptionType extends Record<string, string>>({
  className,
  id,
  options,
  onChange,
  selectedOption,
  fontSize = "default",
  color = "black",
}: SelectProps<OptionType>) => {
  const handleChange: React.ChangeEventHandler<HTMLSelectElement> = (e) => {
    const selected = Object.entries(options).find(
      ([_, value]) => value === e.target.value
    )?.[0];
    onChange?.(selected);
  };

  return (
    <StyledSelect
      id={id}
      className={className}
      fontSize={fontSize}
      color={color}
      onChange={handleChange}
      value={selectedOption && options[selectedOption]}
    >
      {Object.entries(options).map(([key, value]) => (
        <option key={key} value={value}>
          {value}
        </option>
      ))}
    </StyledSelect>
  );
};

const fruits = { apple: "사과", banana: "바나나", blueberry: "블루베리" };

type Fruit = keyof typeof fruits;

const FruitSelect = () => {
  const [fruit, changeFruit] = useState<Fruit | undefined>();

  return (
    <Select
      className="fruitSelectBox"
      options={fruits}
      onChange={changeFruit}
      selectedOption={fruit}
      fontSize="large"
    />
  );
};

export default FruitSelect;
```