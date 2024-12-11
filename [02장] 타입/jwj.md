# 2.1 타입이란?

## 1. 자료형으로서의 타입

컴퓨터의 메모리 공간은 한정적이므로, 특정 메모리에 값을 저장하기 위한 값의 크기를 명시해줌으로써 컴퓨터가 한 번에 읽을 메모리 크기를 효율적으로, 온전히 불러올 수 있다.

현재 ECMAScript 표준에 따라 자바스크립트는 7가지 데이터 타입(자료형)을 정의한다.

- undefined
- null
- Boolean
- String
- Symbol
- Numeric (Number / BigInt)
- Object

개발자는 타입을 이용해 값의 종류를 명시하고, 메모리를 효율적으로 사용할 수 있다.

&nbsp;

## 2. 집합으로서의 타입

프로그래밍에서의 타입, 특히 타입스크립트에서의 타입은 수학의 집합과 유사하다. 해당 관점에서의 타입은 값이 가질 수 있는 유효한 범위의 집합을 말한다. 타입을 통해 컴파일러나 개발자가 이 값을 통해 어떤 일을 할 수 있는지 사전에 쉽게 알 수 있다.

다음은 타입을 명시함으로써 해당 함수가 어떤 값을 인자로 받을 수 있는지 명시하는 예시다.

```ts
function double(n: number) {
  return n * 2;
}

double(2); // 4
double("z"); // 🚨 Error: Argument of type 'string' is not assignable to parameter of type 'number'.(2345)
```

&nbsp;

## 3. 정적 타입과 동적 타입

타입을 결정하는 시점에 따라 타입을 정적 타입(static type)과 동적 타입(dynamic type)으로 구분할 수 있다.

정적 타입 시스템에선 컴파일타임에 모든 변수의 타입이 결정된다. (자바, C, **타입스크립트** 등)

동적 타입 시스템에선 런타임에 변수 타임이 결정된다. (파이썬, 자바스크립트 등)

런타임에서 타입을 예측할 수 없다면 프로그램이 프로그램 작성자의 예상대로 작동하지 않을 위험성이 있다.

&nbsp;

## 4. 강타입과 약타입

암묵적 타입 변환(컴파일러 또는 엔진에 의해 런타임에 타입이 자동으로 변환) 여부에 따라 타입을 강타입(strongly type)과 약타입(weakly type)으로 분류할 수 있다.

강타입 특징의 언어는 서로 다른 타입끼리의 연산을 컴파일러 또는 인터프리터가 허락하지 않는다. (**타입스크립트**)

약타입 특징의 언어는 컴파일러 또는 인터프리터의 내부적인 판단으로 타입을 변환하여 어떻게든 연산을 수행한다. (자바스크립트)

암묵적 타입 변환은 개발자에게 명시적으로 타입을 변환하지 않아도 되는 편리함을 제공하지만, 개발자의 의도와 다르게 동작해 유효하지 않은 연산을 수행할 위험성이 있다.

타입스크립트는 이러한 위험성을 배제할 수 있는 선택권을 줌으로써 프로그램을 **'타입 안전하게(type-safe)'** 만드는데 도움을 준다.

&nbsp;

## 5. 컴파일 방식

일반적으로 컴파일은 사람이 이해하기 쉬운 코드를 기계어로 바꿔주는 과정을 말한다.

바이너리 코드로 변환되는 자바, C# 등의 코드와 다르게 타입스크립트의 컴파일 결과물은 **자바스크립트 파일**이다. 타입스크립트는 자바스크립트의 컴파일타임에 런타임 에러를 사전에 잡아내기 위해 탄생했다.

&nbsp;

# 2.2 타입스크립트의 타입 시스템

## 1. 타입 애너테이션 방식

타입 애너테이션(type annotaion)은 변수나 상수 혹은 함수의 인자와 반환 값에 타입을 명시적으로 선언해서 어떤 타입 값이 저장될 것인지를 컴파일러에 직접 알려주는 문법이다.

타입스크립트에서는 **변수 이름 뒤에 `: type` 구문을 붙여 데이터 타입을 명시**한다. **타입스크립트는 자바스크립트의 슈퍼셋**이기 때문에 `: type` 구문을 붙이지 않아도 코드가 정상 작동한다. 하지만 타입을 제거하면 타입스크립트의 이점을 온전히 살리기 어려울 것이다.

다음은 타입스크립트에서의 변수 선언 코드다.

```ts
let isDone: boolean = false;
let decimal: number = 6;
let color: string = "blue";
let list: number[] = [1, 2, 3];
let x: [string, number]; // tuple
```

&nbsp;

## 2. 구조적 타이핑

타입을 사용하는 여러 언어에서 값이나 객체는 하나의 구체적인 타입을 갖는다. 타입은 이름으로 구분되며 컴파일타임 이후에도 남는다. 이것을 명목적으로 구체화한 타입 시스템(Nominal Reified Type Systems)이라고 부르기도 한다.

또한 서로 다른 클래스끼리 명확한 상속 관계나 공통으로 가지고 있는 인터페이스가 없다면 타입은 서로 호환되지 않는다.

그러나 **타입스크립트에서는 구조로 타입을 구분**한다. 이를 구조적 타이핑(Structural Type System)이라고 한다.

아래의 예제와 같이 구조만 같으면 서로 호환된다.

```ts
interface Developer {
  faceValue: number;
}

interface BankNote {
  faceValue: number;
}

let developer: Developer = { faceValue: 52 };
let bankNote: BankNote = { faceValue: 10000 };

developer = bankNote; // OK
bankNote = developer; // OK
```

&nbsp;

## 3. 구조적 서브타이핑

타입스크립트의 타입은 '값의 집합'으로 이해할 수 있다. 타입스크립트에서는 특정 값이 복수의 타입을 가질 수 있다.

```ts
type stringOrNumber = string | number;
```

집합으로 나타낼 수 있는 타입스크립트의 타입 시스템과 같은 개념을 지탱하는 개념이 구조적 서브타이핑이다. 서브타이핑은 상위 유형의 요소에서 작동하도록 작성된 프로그램 요소(일반적으로 서브루틴 또는 함수)가 서브타입의 요소에서도 작동할 수 있다는 의미이다.

구조적 서브타이핑에서는 객체가 가지고 있는 속성을 바탕으로 타입을 구분한다. **이름이 다른 객체라도 가진 속성만 동일하다면 타입스크립트에서는 서로 호환 가능**하다.

```ts
interface Pet {
  name: string;
}

interface Cat {
  name: string;
  age: number;
}

let pet: Pet;
let cat: Cat = { name: "Zag", age: 2 };

// ✅ OK
pet = cat;
```

Pet의 name 속성을 Cat도 가지고 있으므로 Cat 타입의 cat을 Pet 타입의 pet에도 할당할 수 있다.

구조적 서브타이핑은 함수의 매개변수에도 적용된다.

타입스크립트에서는 오로지 타입 내부 구조에 의해 서로 다른 두 타입 간의 호환성이 결정된다. 타입 A가 타입 B의 서브 타입이라면 A 타입의 인스턴스는 B 타입이 필요한 곳에 언제든지 위치할 수 있다. **즉, 타입이 계층 구조로부터 자유롭다.**

&nbsp;

## 4. 자바스크립트를 닮은 타입스크립트

자바, C++과 같은 언어는 타입의 이름만으로 타입을 구분하는 명목적 타이핑을 사용한다. 명목적 타이핑은 타입의 동일성을 확인하는 과정에서 조금 더 안전하다.

그럼에도 타입스크립트는 구조적 타이핑을 채택했는데, 이는 자바스크립트를 모델링한 언어이기 때문이다. 자바스크립트는 **어떤 함수의 매개변숫값이 올바르게 주어지면 그 값이 어떻게 만들어졌는지 신경 쓰지 않고 사용하는 덕 타이핑을 기반**으로 하고 있다.

> 덕 타이핑: 어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식

타입스크립트는 이런 동작을 그대로 모델링해 더욱 유연한 타이핑이 가능하다. 타입스크립트는 쉬운 사용성, 안전성 두 가지 목표의 균형을 위해 구조적 타입 시스템을 채택했다.

조그마한 차이점이 있다면 자바스크립트의 덕 타이핑은 런타임에 타입을 검사하나, 타입스크립트의 구조적 타이핑은 컴파일타임에 타입체커가 타입을 검사한다.

&nbsp;

## 5. 구조적 타이핑의 결과

구조적 타이핑의 특징 때문에 예기치 못한 결과가 나올 때도 있다.

```ts
interface Cube {
  width: number;
  height: number;
  depth: number;
  // name: "Awesome Cube" 이러한 추가 속성을 가진 객체도 할당 가능하기 때문에 오류가 발생한다.
}

function addLines(c: Cube) {
  let total = 0;

  for (const axis of Object.keys(c)) {
    // 🚨 Element implicitly has an 'any' type
    // because expression of type 'string' can't be used to index type 'Cube'.
    // 🚨 No index signature with a parameter of type 'string'
    // was found on type 'Cube'
    const length = c[axis];

    total += length;
  }
}
```

`Cube`는 모두 `number` 타입이지만 `c`에 들어올 객체는 `width`, `height`, `depth` 외에도 어떤 속성이든 가질 수 있기 때문에 `c[axis]`의 타입을 `number`라고 확정할 수 없어 오류를 발생시킨다.

이러한 한계를 극복하기 위해 명목적 타이핑 언어의 특징을 가미한 식별할 수 있는 유니온(Discriminated Unions)와 같은 방법이 생겨났다.

&nbsp;

## 6. 타입스크립트의 점진적 타입 확인

타입스크립트는 점진적으로 타입을 확인한다. 컴파일 타임에 타입을 검사하면서 **필요에 따라 타입 선언 생략을 허용**한다. 타입이 선언되지 않은 변수와 표현식에는 동적으로 타입을 검사한다.

때문에 타입스크립트는 타입을 선언해주지 않아도 되지만 컴파일타임에 프로그램의 모든 타입을 알고 있을 때 최상의 결과를 보여준다.

```ts
function add(x, y) {
  return x + y;
}

// 위 코드는 아래와 같이 암시적 타입 변환이 일어난다.
function add(x: any, y: any): any;
```

`any` 타입은 타입스크립트에서 모든 타입의 종류를 포함하는 가장 상위 타입으로 어떤 타입 값이든 할당할 수 있다. 하지만 정확한 타이핑을 위해 tsconfig에서 any 타입으로의 추론을 허락하지 않는 `noImplicitAny` 옵션을 `true`로 설정해주는 것을 권장한다.

&nbsp;

## 7. 자바스크립트 슈퍼셋으로서의 타입스크립트

타입스크립트는 모든 자바스크립트의 문법을 포함한다.

```ts
let developer = "Colin";

console.log(developer.toUppercase());

// Property ‘toUppercase’ does not exist on type ‘string’.
// Did you mean ‘toUpperCase’?
```

자바스크립트에서도 타입스크립트 컴파일러를 유용하게 사용할 수 있다.

`developer`의 변수가 문자열이라는 것을 알려주지 않아도 타입스크립트는 초깃값으로 타입을 추론해서 `toUpperCase` 메서드로 대체할 것을 제안한다.

&nbsp;

## 8. 값 vs 타입

타입스크립트에서 값 공간과 타입 공간의 이름은 서로 충돌하지 않는다. 이는 타입스크립트에서 `type`으로 선언한 내용이 자바스크립트 런타임에서 제거되기 때문이다.

따라서 아래와 같은 코드가 유효하다.

```ts
type Developer = { isWorking: true };
const Developer = { isTyping: true }; // OK
type Cat = { name: string; age: number };
const Cat = { slideStuffOffTheTable: true }; // OK
```

보통 타입스크립트에서 값은 할당 연산자(`=`)로 작성하고, 타입은 `:` 또는 `as`로 작성하기 때문에 구분하기 쉽다.

하지만 타입스크립트는 **작성된 코드 문맥을 파악해서 값 또는 타입을 스스로 판단**한다. 때문에 구조 다음과 같은 코드를 구조 분해 할당하면 오류가 발생할 수 있다.

```ts
function email(options: { person: Person; subject: string; body: string }) {
  // ...
}

// 위 함수를 구조 분해 할당했음.
function email({ person, subject, body }) {
  // ...
}
```

```ts
function email({
  person: Person, // 🚨
  subject: string, // 🚨
  body: string, // 🚨
}) {
  // ...
}
```

위 코드에서 `Person`과 `string`이 값으로 해석되었기 때문에 오류가 발생한다. `person`과 `Person`이 값 공간에 있는 것으로 해석되어 각 함수의 매개변수 객체 내부 속성의 키-값 쌍으로 해석되었다. 때문에 아래와 같이 작성해야 한다.

```ts
function email({
  person,
  subject,
  body,
}: {
  person: Person;
  subject: string;
  body: string;
}) {
  // ...
}
```

타입스크립트에서 `class`와 `enum`은 값 공간과 타입 공간에 동시 존재하는 심볼이기 때문에 혼동을 주기도 한다.

```ts
class Developer {
  name: string;

  domain: string;

  constructor(name: string, domain: string) {
    this.name = name;
    this.domain = domain;
  }
}

const me: Developer = new Developer("zig", "frontend");
```

`me` 뒤에 `Developer`는 타입, `new` 뒤에 `Developer`는 클래스 생성자의 함수인 값으로 동작한다. 타입스크립트에서 **클래스는 타입으로도 사용 가능하지만 런타임에서 객체로 변환되어 자바스크립트의 값으로 사용되는 특징**이 있다.

`enum`도 `class`와 유사하게 동작한다.

```ts
enum WeekDays {
  MON = "Mon",
  TUES = "Tues",
  WEDNES = "Wednes",
  THURS = "Thurs",
  FRI = "Fri",
}
// ‘MON’ | ‘TUES’ | ‘WEDNES’ | ‘THURS’ | ‘FRI’
type WeekDaysKey = keyof typeof WeekDays;

function printDay(key: WeekDaysKey, message: string) {
  const day = WeekDays[key];
  if (day <= WeekDays.WEDNES) {
    console.log(`It’s still ${day}day, ${message}`);
  }
}

printDay("TUES", "wanna go home");
```

위의 `enum`은 타입으로 사용됐다.

```ts
enum MyColors {
  BLUE = "#0000FF",
  YELLOW = "#FFFF00",
  MINT = "#2AC1BC",
}

function whatMintColor(palette: { MINT: string }) {
  return palette.MINT;
}

whatMintColor(MyColors); // ✅
```

위의 `enum`은 `palette`에 할당되어 `MINT`라는 속성을 가지는 일반적인 객체처럼 동작하고 있다.

타입스크립트에서 **값으로 사용되는 심볼은 자바스크립트로의 변환 뒤에도 여전히 자바스크립트 파일에 해당 정보가 남지만, 타입으로 사용되는 요소는 컴파일 이후에 자바스크립트에서 해당 정보가 사라진다.**

&nbsp;

## 9. 타입을 확인하는 법

`typeof`나 `instanceof`, 그리고 타입 단언(`as`)을 사용해서 타입을 확인할 수 있다.

`typeof`는 **피연산자의 데이터 타입을 나타내는 문자열을 반환**한다. (Boolean, null, undefined, Number, BigInt, String, Symbol, Function, 호스트 객체, object 객체)

```ts
typeof 2022; // "number"
```

`typeof` 역시 피연산자가 값인지 타입인지에 따라 다른 결과를 반환한다.

```ts
interface Person {
  first: string;
  last: string;
}

const person: Person = { first: "zig", last: "song" };

function email(options: { person: Person; subject: string; body: string }) {}

const v1 = typeof person; // 값은 ‘object’ const
v2 = typeof email; // 값은 ‘function’

type T1 = typeof person; // 타입은 Person
type T2 = typeof email; // 타입은 (options: { person: Person; subject: string; body:string; }) = > void
```

값에서 사용된 `typeof`는 자바스크립트에서의 `typeof`와 동일하게 동작한다.

&nbsp;

자바스크립트의 `class`에선 `typeof` 연산자를 쓸 때 주의할 점이 있다.

```ts
class Developer {
  name: string;

  sleepingTime: number;

  constructor(name: string, sleepingTime: number) {
    this.name = name;
    this.sleepingTime = sleepingTime;
  }
}

const d = typeof Developer; // 값이 ‘function’
type T = typeof Developer; // 타입이 typeof Developer
```

자바스크립트의 클래스는 결국 함수이기 때문에 값 공간에서 `Developer`는 함수지만 타입 공간에서의 `Developer`는 `Developer`의 인스턴스의 타입이 아니라 생성자 함수로 취급된다.

```ts
const zig: Developer = new Developer("zig", 7);
type ZigType = typeof zig; // 타입이 Developer
```

`zig`는 `Developer`가 인스턴스 타입으로 생성되었기 때문에 타입 공간에서의 `typeof zig` 즉, `type ZigType`은 `Developer`를 반환한다.

하지만 `Developer` 자체는 `Developer` 타입을 만드는 생성자 함수이기에 `typeof Developer` 타입도 그 자체로 `typeof Developer`가 된다. 즉, 풀어서 설명하면 아래 코드와 같다.

```ts
// 해당 생성자 함수를 의미
new (name: string, sleepingTime: number): Developer
```

&nbsp;

자바스크립트에서는 `instanceof` 연산자를 통해 프로토타입 체이닝 어딘가에 생성자의 프로토타입 속성이 존재하는지 판단할 수 있다. **`typeof` 연산자처럼 `instanceof` 연산자의 필터링으로 타입이 보장된 상태에서 안전하게 값의 타입을 정제하여 사용**할 수 있다.

```ts
let error: unknown;

if (error instanceof Error) {
  // true or false
  showAlertModal(error.message);
} else {
  throw Error(error);
}
```

> 자바스크립트의 `class`는 '프로토타입'을 통해 "고전적인" 관점의 객체 지향을 흉내내고 있다. 자바스크립트의 `class`는 상속보단 '위임'에 가깝다.
> (프로토타입 체이닝이란? https://velog.io/@sik2/JS-CoreJavaScript-%ED%94%84%EB%A1%9C%ED%86%A0%ED%83%80%EC%9E%85-%EC%B2%B4%EC%9D%B4%EB%8B%9DPrototype-Link-Prototype-Object)

&nbsp;

또한 **타입 단언(`as`)을 통해 타입을 강제**할 수도 있다. 타입 단언은 개발자가 해당 값의 타입을 확신할 때 사용되며 강제 형 변환과 유사한 기능을 제공한다. (컴파일 단계에선 형 변환을 강제할 수 있으나 결국 타입스크립트의 코드는 자바스크립트로 변환되는 코드이기에 다른 언어의 타입 캐스팅과 유사하나 일치하는 개념은 아니다.)

```ts
const loaded_text: unknown; // 어딘가에서 unknown 타입 값을 전달받았다고 가정

const validateInputText = (text: string) => {
  if (text.length < 10) return "최소 10글자 이상 입력해야 합니다.";
  return "정상 입력된 값입니다.";
};

validateInputText(loaded_text as string); // as 키워드를 사용해서 string으로 강제하지 않으면 타입스크립트 컴파일러 단계에서 에러 발생
```

&nbsp;

이외에도 **타입 가드**라는 패턴으로 타입을 검사할 수 있다. 특정 조건을 검사해서 타입을 정제하고 타입 안정성을 높이는 패턴이다. (4.2 절에서 자세히 다룸)

&nbsp;

# 2.3 원시 타입

자바스크립트의 값은 타입을 가지지만, 변수는 타입을 가지지 않는다. 타입스크립트는 변수에 타입을 지정할 수 있는 타입 시스템 체계가 있다. 특정 타입의 변수에는 해당 타입의 값만 할당할 수 있다는 의미다.

자바스크립트의 7가지 원시 값은 타입스크립트에서 원시 타입으로 존재한다. 타입스크립트에서의 내장 원시 타입은 모두 소문자로 표기되어 있어 혼동하지 않도록 주의한다.

타입스크립트의 원시 타입

- boolean
- undefined
- null
- number
- bigInt
- string
- symbol

&nbsp;

## boolean

오직 `true` 혹은 `false` 값만 할당할 수 있는 타입이다.

```ts
const isEmpty: boolean = true;
const isLoading: boolean = false;

// errorAction.type과 ERROR_TEXT가 같은지 비교한 결괏값을 boolean 타입으로 반환하는 함수
function isTextError(errorCode: ErrorCodeType): boolean {
  const errorAction = getErrorAction(errorCode);
  if (errorAction) {
    return errorAction.type === ERROR_TEXT;
  }
  return false;
}
```

자바스크립트에는 `boolean` 원시 값은 아니지만 형 변환을 통해 `true` / `false`로 취급되는 `Truthy` / `Falsy` 값이 존재한다.

&nbsp;

## undefined

정의되지 않았다는 의미의 타입으로, 오직 `undefined` 값만 할당할 수 있다. 초기화되지 않은 값을 의미한다. 옵셔널로 지정되어 있는 값에도 `undefined`를 할당 가능하다. 즉, 초기화되어 있지 않거나 존재하지 않음을 나타내는 타입이다.

```ts
let value: string;
console.log(value); // undefined (값이 아직 할당되지 않음)

type Person = {
  name: string;
  job?: string;
};
```

&nbsp;

## null

오직 `null`만 할당할 수 있다. 명시적, 의도적으로 값이 아직 비어있을 수 있음을 보여주는 타입이다.

```ts
let value: null | undefined;
console.log(value); // undefined (값이 아직 할당되지 않음)

value = null;
console.log(value); // null
```

다음 예시를 통해 `undefined`와 `null`의 차이점을 자세하게 알아보자.

```ts
type Person1 = {
  name: string;
  job?: string;
};

type Person2 = {
  name: string;
  job: string | null;
};
```

`Person1`은 `job`이라는 속성이 있을 수도, 없을 수도 있음을 나타낸다. `job`의 속성의 유무를 통해 무직인지 아닌지를 나타낸다.

`Person2`는 `job`이라는 속성은 사람마다 가지지만 그 값이 비어있을 수도 있다는 것을 나타낸다. `null`로써 명시적으로 무직인 상태를 나타낸다.

&nbsp;

## number

자바스크립트의 숫자에 해당하는 모든 원시 값을 할당할 수 있는 타입이다.

```ts
const maxLength: number = 10;
const maxWidth: number = 120.3;
const maximum: number = +Infinity;
const notANumber: number = NaN;
```

&nbsp;

## bigInt

ES2020에서 새로 도입된 데이터 타입으로, 타입스크립트 3.2 이상 버전부터 사용 가능하다. 자바스크립트는 `Number.MAX_SAFE_INTEGER(2^53-1)`을 넘어가는 값을 처리할 수 없었지만 `bigInt`를 사용하면 이보다 큰 수를 처리할 수 있다. `number` 타입과는 엄연히 다르므로 상호 작용은 불가능하다.

```ts
const bigNumber1: bigint = BigInt(999999999999);
const bigNumber2: bigInt = 999999999999n;
```

&nbsp;

## string

문자열을 할당할 수 있는 타입이다.

```ts
const receiverName: string = “KG”;
const receiverPhoneNumber: string = “010-0000-0000”;
const letterContent: string = `안녕, 내 이름은 ${senderName}이야.`;
```

&nbsp;

## symbol

ES2015에서 도입된 데이터 타입으로 `Symbol()` 함수를 사용하면 어떤 값과도 중복되지 않는 유일한 값을 생성할 수 있다.

```ts
const MOVIE_TITLE = Symbol("title");
const MUSIC_TITLE = Symbol("title");
console.log(MOVIE_TITLE === MUSIC_TITLE); // false

let SYMBOL: unique symbol = Symbol(); // A variable whose type is a 'unique symbol'
// type must be 'const'
```

&nbsp;

타입스크립트의 모든 타입은 기본적으로 `null`과 `undefined`를 포함한다. 하지만 `tsconfig`의 `strictNullChecks` 옵션을 활성화했을 때는 사용자가 명시적으로 해당 타입에 `null`이나 `undefined`를 포함해야만 해당 타입을 사용할 수 있다.

`!`연산자로 해당 타입이 `null`이나 `undefined`가 아님을 단언할 수도 있다. 일반적으로는 타입 가드를 사용하는 편이 안전하다고 여겨진다고 한다.

&nbsp;

# 2.4 객체 타입

## object

자바스크립트 객체의 정의에 맞게 타입스크립트 타입 시스템도 `object` 타입이다.

`any` 타입과 마찬가지로 타입스크립트의 의미가 퇴색되지 않게끔 가급적 사용을 권장하지 않고 있다.

앞서 다룬 원시 타입들은 `object` 타입에 속하지 않는다.

```ts
function isObject(value: object) {
  return (
    Object.prototype.toString.call(value).replace(/\[|\]|\s|object/g, "") ===
    "Object"
  );
}
// 객체, 배열, 정규 표현식, 함수, 클래스 등 모두 object 타입과 호환된다
isObject({});
isObject({ name: "KG" });
isObject([0, 1, 2]);
isObject(new RegExp("object"));
isObject(() => {
  console.log("hello wolrd");
});
isObject(class Class {});

// 그러나 원시 타입은 호환되지 않는다
isObject(20); // false
isObject("KG"); // false
```

&nbsp;

## {}

중괄호는 자바스크립트에서 객체 리터럴 방식으로 객체를 생성할 때 사용한다. 타입스크립트에서도 유사하게 사용하지만 객체의 속성 타입을 지정해줘야 한다.

```ts
// 정상
const noticePopup: { title: string; description: string } = {
  title: "IE 지원 종료 안내",
  description: "2022.07.15일부로 배민상회 IE 브라우저 지원을 종료합니다.",
};

// SyntaxError
const noticePopup: { title: string; description: string } = {
  title: "IE 지원 종료 안내",
  description: "2022.07.15일부로 배민상회 IE 브라우저 지원을 종료합니다.",
  startAt: "2022.07.15 10:00:00", // startAt은 지정한 타입에 존재하지 않으므로 오류
};
```

`{}` 타입으로 지정된 객체에는 어떤 값도 속성으로 할당할 수 없다. 빈 객체를 의미하기 위해 사용하지만 유틸리티 타입으로 `Record<string, never>`처럼 사용하는 것을 권장한다. (유틸리티 타입은 5장에서 자세히 다룸) 사실 완전히 비어있는 순수한 객체도 아니다.

```ts
let noticePopup: {} = {};

noticePopup.title = "IE 지원 종료 안내"; // (X) title 속성을 지정할 수 없음

console.log(noticePopup.toString()); // [object Object]
```

&nbsp;

## array

타입스크립트에서는 배열을 array라는 별도 타입으로 다룬다. 자바스크립트와 다르게 하나의 타입 값만 가질 수 있으나 원소의 개수는 자유롭다.

```ts
const getCartList = async (cartId: number[]) => {
  const res = await CartApi.GET_CART_LIST(cartId);
  return res.getData();
};

getCartList([]); // (O) 빈 배열도 가능하다
getCartList([1001]); // (O)
getCartList([1001, 1002, 1003]); // (O) number 타입 원소 몇 개가 들어와도 상관없다
getCartList([1001, "1002"]); // (X) ‘1002’는 string 타입이므로 불가하다
```

&nbsp;

## type과 interface

타입스크립트에서는 일반적으로 객체를 타이핑(선언)할 때 `type`과 `interface` 둘 중에 하나를 사용한다.

공식 문서에서는 전역적으로 사용할 땐 `interface`를, 작은 범위 내에서 한정적으로 사용할 땐 `type`을 사용하는 것을 권장하는 것 같다.

```ts
type NoticePopupType = {
  title: string;
  description: string;
};

interface INoticePopup {
  title: string;
  description: string;
}
const noticePopup1: NoticePopupType = {
  /* ... */
};
const noticePopup2: INoticePopup = {
  /* ... */
};
```

&nbsp;

## function

`function` 키워드 자체를 타입으로 사용하지는 않는다.

함수 선언 시 매개변수와 반환값의 타입을 명시해주어야 한다.

```ts
function add(a: number, b: number): number {
  return a + b;
}
```

함수 자체의 타입을 정의하고 싶을 땐 호출 시그니처(Call Signature)를 사용한다. 함수의 매개변수와 반환 값의 타입을 명시할 수 있다. 아래는 그 예시다.

```ts
type add = (a: number, b: number) => number;
```

자바스크립트와 다르게, 일반적으로 타입스크립트에서는 함수 자체의 타입을 명시할 때는 화살표 함수 방식으로만 호출 시그니처를 정의한다.
