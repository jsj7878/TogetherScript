```jsx
2.1 타입이란
2.2 타입스크립트의 타입 시스템
2.3 원시 타입
2.4 객체 타입
```

# 2.1 타입이란

ECMAScript 표준을 따르는 자바스크립트는 7가지 데이터 타입 = 자료형을 정의한다.

- undefined
- null
- Boolean
- String
- Symbol
- Numeric(Number과 BigInt)
- Object

> 어떤 값이 T 타입이라면 컴파일러 ( 또는 개발자 ) 는 이 값으로 어떤 일을 할 수 있고 , 어떤 일을
할 수 없는지를 사전에 알 수 있다. 타입 시스템은 코드에서 사용되는 유효한 값의 범위를 제
한해서 런타임에서 발생할 수 있는 유효하지 않은 값에 대한 에러를 방지해준다.
> 
&nbsp;

## 🟡 정적 타입과 동적 타입


타입을 결정하는 시점에 따라 정적 타입과 동적 타입으로 분류

> 코드는 `컴파일 - 실행` 단계를 거침
> 
- 정적 타입 : 변수의 타입이 `컴파일 타임`에 결정 = 코드 수준에서 개발자가 타입을 명시해야 함 (C, Java, Typescript)
- 동적 타입 : 변수의 타입이 `런타임`에서 결정 (Python, Javascript)
    - 프로그램을 실행할 때 타입 에러가 발견된다.
- 
&nbsp;
## 🟡 강타입과 약타입


사실 모든 언어에는 타입이 존재한다. 개발자가 타입을 명시하지 않으면 컴파일러 또는 엔진에 의해 런타임에 타입이 자동으로 변경된다. 이를 암묵적 타입변환이라고 한다.

암묵적 타입 변환 여부에 따라 타입 시스템을 강타입과 약타입으로 분류할 수 있다.

- 강타입 언어끼리 : 서로 다른 타입을 갖는 값끼리 연산 시도시 에러 발생
- 약타입 언어끼리 : 서로 다른 타입을 갖는 값끼리 연산 시도시 내부적으로 판단해서 ***암묵적 타입 변환***됨.

![image.png](https://velog.velcdn.com/images%2Fsangmin7648%2Fpost%2F19e2e62e-bc44-4862-a221-747127ba63ff%2Fimage.png)

> 각 언어별로 ‘2’ -1 을 연산했을 때
> 

**약타입 언어** : C++, 자바, 자바스크립트

**강타입 언어** : 파이썬, 타입스크립트

### 🔸타입 시스템

타입 검사기가 프로그램에 타입을 할당하는데 사용하는 규칙 집합을 타입 시스템이라고 한다.

두 가지 타입시스템이 존재한다.

1. 어떤 타입을 사용하는지 컴파일러에 명시적으로 알려줘야 하는 타입시스템
2. 자동으로 타입을 추론하는 타입 시스템

타입 스크립트는 2가지 타입 시스템의 영향을 모두 받아 개발자는 직접 타입을 명시하거나, 자동추론하도록 하는 방법 중에 선택해서 사용할 수 있다.

### 🔸타입스크립트의 컴파일

다른 언어의 컴파일 결과는 고수준 언어에서 바이너리 코드로 변환해 결과적으로 이진 코드를 얻는다.

이와 다르게 `타입스크립트의 컴파일 결과는 자바스크립트 파일`이다.

이는 타입스크립트가 *자바스크립트의 컴파일타임에 런타임 에러를 미리 잡아내기 위한 언어*이기 때문이다. 

---
&nbsp;
&nbsp;
&nbsp;

# 2.2 타입스크립트의 타입 시스템

💡 **명목적 타이핑 vs 구조적 타이핑**

&nbsp;
## 🟡 타입 애너테이션

타입 애너테이션이이란 타입을 명시적으로 선언해서 컴파일러에게 어떤 타입 값이 저장될지 알려주는 문법이다. 

- 언어마다 타입을 명시해주는 방법이 다른데
- 타입스크립트에서는 변수 이름 뒤에 `: type` 형식으로 데이터 타입을 명시한다.

## 🟡 구조적 타이핑

❗ 타입스크립트가 아닌 일반적인 프로그래밍 언어에서

1. 프로그래밍 언어에서 값이나 객체는 하나의 구체적인 타입을 가진다.
2. 타입은 `이름` 으로 구분되며 컴파일 타임 이후에도 남는다. 
3. 서로 다른 클래스에서 일반적으로 타입이 서로 호환되지 않는다.

위를 통틀어 타입 시스템이라고 한다.

> 타입 검사기가 프로그램에 타입을 할당하는 데 사용하는 규칙 집합을 타입 시스템이라고 한다.
> 

❗ 이와 다르게 타입스크립트에서는 `구조적 타이핑`을 한다. 즉 서로 다른 타입을 판단할 때 이름이 아니라 구조로 판단한다.

### 🔸구조적 서브타이핑

---

타입스크립트의 타입은 값의 집합이다. 즉 타입스크립트에서는 특정 값이 여러 타입을 동시에 가질 수 있다.

❓ **어떻게 가능한가?**

> 구조적 서브타이핑 때문에 가능하다. 구조적 서브타이핑이란 객체가 가지고 있는 속성=property로 타입을 구분하는 것이다. 만약 이름이 다르지만 속성이 같다면 이는 같은 타입이다.
> 

더 부모(인자가 적음)인 타입을 자식 클래스/함수/인터페이스에서 사용 가능

타입의 상속도 구조적 타이핑을 기반으로 한다.


💡 **타입이 계층 구조로부터 자유롭다.**

&nbsp;
## 🟡 타입스크립트는 왜 구조적 타이핑을 채택했는가?

타입의 이름으로 타입을 구분하는 것이 명목적 타이핑 타입의 구조로 타입을 구분하는 것이 구조적 타이핑이다.

❕ 타입스크립트가 구조적 타이핑을 채택한 이유는 자바스크립트를 기반으로 했기 때문이다. 

자바스크립트는 `덕 타이핑 duck typing`를 기반으로 하는데 덕 타이핑이란 어떤 함수의 매개변숫값이 올바르게 주어진다면 그 값이 어떻게 만들어졌는지는 신경쓰지 않는다는 개념이다.

### 🔸 덕 타이핑 vs 구조적 타이핑

<aside>
🦢 덕 타이핑

어떤 타입에 부합하는 변수와 메서드를 가질 경우 해당 타입에 속하는 것으로 간주하는 방식이다.

“만약 어떤 새가 오리처럼 걷고, 헤엄치며 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를 것이다.”

</aside>

 자바스크립트의 덕 타이핑과 타입스크립트의 구조적 타이핑은 서로 구분되는 타이핑 방식이지만 실제 사용 코드를 보면 별 차이가 없어보인다. 두 가지 타이핑 방식 모두 이름으로 타입을 구분하지 않고 `객체가 가진 속성을 기반으로 타입을 검사`한다.

### 차이점 ❕

둘의 차이점은 타입을 검사하는 시점이다.

1. 덕 타이핑은 런타임에 타입을 검사한다. 주로 동적 타이핑에서 사용
2. 구조적 타이핑은 컴파일타임에 타입체커가 타입을 검사한다. 주로 정적 타이핑에서 사용

```jsx
+
자바스크립트 - 런타임에 타입 검사(동적 타입) - 덕 타이핑
타입스크립트 - 컴파일 타임에 타입 검사(정적 타입) - 구조적 타이핑
```
&nbsp;
## 🟡 구조적 타이핑에서의 문제와 유니온의 등장

```tsx
interface Cube {
	width: number;
	height: number;
	depth: number;
}
function addLines(c: Cube) {
	let total = 0;
	// ❗
	for (const axis of Object keys(c)) {
		const length = c[axis];
		total + length;
	}
}
```

1. 구조적 타이핑의 유연성으로 인해 
2. 현재 큐브의 속성을 포함한 다른 인터페이스=타입이 
3. addLines 함수에 사용될 경우
4. c[axis]에 number이 반드시 온다는 보장을 할 수 없으므로 에러가 발생할 수 있다.

> 이러한 한계를 극복하고자 `타입스크립트에 명목적 타이핑 언어의 특징`을 가미한 `식별할 수 있는 유니온` 같은 방법이 생겼다.
> 
&nbsp;
## 🟡 타입스크립트는 점진적으로 타입을 확인한다?

점진적 타입 검사란 `컴파일 타임에 타입을 검사`하면서 `필요에 따라 타입 선언 생략을 허용`하는 방식이다. 

- 타입을 지정한 변수와 표현식은 정적으로 타입을 검사
- 타입 선언이 생략된 경우 동적으로 타입을 검사 = 암시적 타입 변환

> 일부 타입 선언을 생략해도 정상 동작하지만 모든 타입을 선언했을 때 가장 좋은 결과를 보여준다.
> 

```tsx
// 타입선언을 생략해 런타임에서 에러가 발생한 경우
const names = ["zig", "colin"];
console. log (names[2].toUpperCase);
// # TypeError: Cannot read property 'toUpperCase' o f undefined
```

위에서는 타입 선언을 생략했기 때문에 컴파일 타임에서 타입 선언 생략을 허용해주면서 암시적 타입 변환이 이루어진다. (타입 추론: string[])

그러면서 런타임까지 넘어가게 되고 여기서 해당하는 값이 없으므로 에러가 발생한다.

```tsx
const names: string[] = ["zig", "colin"];
console.log(names[2].toUpperCase());
```

타입을 선언해주면 컴파일 타임에 에러가 발생한다.

+.타입스크립트 컴파일 옵션에서 noImplicitAny를 true로 하면 any 타입을 허용하지 않는다.

&nbsp;
## 🟡 자바스크립트의 슈퍼셋 타입스크립트

타입스크립트 = 자바스크립트 + `정적 타이핑`

> 모든 자스로 타스로도 동작하지만, 타스는 자스에서 에러가 발생할 수 있다. (: type 추가하면 자스에서 에러 발생 등)
> 

---
&nbsp;
## 🔵 값과 타입

문자열, 숫자, 변수, 매개변수 등은 전부 값이다. 또 객체도 값이다. 자바스크립트에서는 런타임에 함수가 객체로 변환되기 때문에 함수도 값이다.

> 객체, 함수도 값
> 

값은 연산해서 변수에 할당(assign)할 수 있다.

함수는 값이므로 변수에 할당할 수 있다.

### 🔹type과 interface

값 공간과 타입 공간의 이름은 충돌하지 않는다. 즉 같은 이름의 type과 interface가 있을 수 있다. `타입스크립트의 문법인 type으로 선언한 내용은 자바스크립트의 런타임에서 제거`되기 때문이다.

&nbsp;
## 🔵 타입스크립트에서는 왜 값과 타입을 구분해서 작성해야 할까?

> 타입스크립트에서는 값과 타입이 함께 사용된다. 값과 타입은 타입스크립트에서 별도의 네임스페이스에 존재한다. 타입스크립트는 사용되는 위치에 따라 스스로 값 또는 타입을 해석해서 추론한다.
> 

값-타입 공간을 혼동하는 문제를 해결하기 위해 값과 타입을 구분해서 작성해야 하낟.

```tsx
function email({
	person,
	subject,
	body,
	}: {
	person: Person;
	subject: string;
	body: string;
}) 
```

```tsx
type Person = {
  name: string;
  email: string;
};

function email (options: { person: Person; subject: string; body: string }) {}
```

## 🔵 class와 enum

### 🔹 class

자바스크립트 ES6에서 등장한 클래스는 객체 인스턴스를 더욱 쉽게 생성하기 위한 문법 기능이다. 동시에 클래스는 타입으로도 사용된다. 즉 타입스크립트 코드에서 `클래스는 값과 타입 공간 모두에 포함`될 수 있다.

```tsx
class Developer {
	name: string;
	domain: string;
	
	constructor (name: string, domain: string) {
	this name = name;
	this .domain = domain;
	}
}
const me: Developer = new Developer (zig", "frontend");
```

타입스크립트에서 클래스는 `타입 애너테이션` (명시적 타입 선언)으로 사용할 수 있다. 여기서 me 뒤의 Developer은 타입에 해당한다. 

> 클래스는 런타임에서 객체로 변환되어 자바스크립트의 값으로 사용된다.


&nbsp;
### 🔹enum

enum도 런타임에 객체로 변환되는 값이다. enum은 런타임에 실제 객체로 존재하며, 함수로 표현할 수도 있다.

- enum도 클래스처럼 타입 공간에서 타입을 제한하는 역할을 하지만 자바스크립트 런타임에서 실제 값으로도 사용될 수 있다.

❗enum을 타입으로 사용한 예

```tsx
enum WeekDays {
  MON = "Mon",
  TUES = "Tues",
  WEDNES = "Wednes",
  THURS = "Thurs",
  FRI = "Fri",
}
// 'MON' | 'TUES' | 'WEDNES' | 'THURS' | 'FRI'
type WeekDaysKey = keyof typeof WeekDays;

function printDay(key: WeekDaysKey, message: string) {
  const day = WeekDays[key];
  if (day < WeekDays.WEDNES) {
    console.log(`It's still ${day}day, ${message}`);
  }
}
printDay("TUES", "wanna go home");
```

❗enum을 값으로 사용한 예

```tsx
enum MyColors {
  BLUE = "#0000FF",
  YELLOW = "#FFFF00",
  MINT = "#2AC1BC"
}

function whatMintColor(palette: { MINT: string }) {
  return palette.MINT;
}
console.log(whatMintColor(MyColors));

```

palette는 MINT라는 속성을 가지는 객체이다.

> 타입스크립트에서 어떠한 심볼이 값으로 사용된다는 것은 컴파일러를 사용해서 타입스크립트 파일을 자바스크립트로 변환해도 여전히 자바스크립트 파일에 해당 정보가 남아있음을 의미한다. 반면 타입으로만 사용되는 요소는 컴파일 이후에 자바스크립트 파일에서 해당 정보가 사라진다.

<img width="300" alt="image" src="https://github.com/user-attachments/assets/736d5ced-66eb-4bfa-8a48-3b37e2b73c70">



🌴 **트리쉐이킹**
- 자스, 타스에서 사용하지 않는 코드를 삭제하는 방식이다. 나무를 흔들면 죽은 나뭇잎이 떨어지는 모습을 보고 이름을 따왔다. ES6  이후의 개발환경에서는 웹팩, 롤업 같은 모듈 번들러를 사용한다. 이러한 도구로 번들링 작업을 수행할 때 사용하지 않는 코드는 삭제된다.

&nbsp;

## 🟡 타입을 확인하는 방법

typeof, instanceof, 타입 단언 3가지 방법으로 타입을 확인할 수 있다.

- typeof : 연산하기 저에 피연산자의 데이터 타입을 나타내는 문자열을 리턴
    - 리턴값 - 자스의 7가지 데이터 타입(boolean, null, undefined, number, bigint, string, symbol)과 func, 호스트 객체, object 객체

&nbsp;

타입스크립트에서는 값 공간과 타입 공간이 별도로 존재한다. 값에서 사용된 typeof는 자바스크립트 런타임의 typeof 연산자가 되고 타입에서 사용된 typeof는 타입스크립트 타입을 반환한다.

```tsx
interface Person {
  first: string;
  last: string;
}

const person: Person = { first: "zig", last: "song" };

function email(options: { person: Person; subject: string; body: string }): void {}

// 값 공간
const v1 = typeof person; // 값은 'object'
const v2 = typeof email; // 값은 'function'

// 타입 공간
type T1 = typeof person; // 타입은 Person
type T2 = typeof email; // 타입은 (options: { person: Person; subject: string; body: string }) => void
```

&nbsp;

자바스크립트의 클래스는 함수이다. 따라서 typeof를 주의해서 사용해야 한다.

```tsx
class Developer {
  name: string;
  sleepingTime: number;

  constructor(name: string, sleepingTime: number) {
    this.name = name;
    this.sleepingTime = sleepingTime;
  }
}

const d = typeof Developer; // 값은 'function'
type T = typeof Developer; // 타입은 typeof Developer

const zig: Developer = new Developer("zig", 7);
type ZigType = typeof zig; // 타입은 Developer

```

type T의 Developer는 Developer 타입의 인스턴스를 만드는 생성자 함수이다. 이걸로 인스턴스를 만들면 해당 인스턴스는 Developer 타입을 가지지만 Developer 자체의 타입은 생성자 함수이다.

```tsx
new (name: string, sleepingTime: number): Developer
```

&nbsp;

❓ 자바스크립트의 instanceof 연산자를 사용하면 프로토타입 체이닝 어딘가에 생성자의 프로토타입 속성이 존재하는지 판단할 수 있다. typeof 연산자처럼 instanceof 연산자의 필터링으로 타입이 보장된 상태에서 안전하게 값의 타입을 정제하여 사용할 수 있다.

### 🔸타입 단언
---
as 키워드를 사용해 타입을 강제한다. 강제 형변환과 유사하다. 다른 언어의 타입 캐스팅과 다른 부분은 타입스크립트는 결국 자바스클비트로 변환되면서 타입 시스템과 문법은 컴파일 단계에서 제거된다. 따라서 타입 단언은 런타임에서는 효력을 발휘하지 못한다.

```tsx
const loaded_text: unknown; // 어딘가에서 unknown 타입 값을 전달받았다고 가정

const validateInputText = (text: string) => {
  if (text.length < 10) return "최소 10 글자 이상 입력해야 합니다.";
  return "정상 입력된 값입니다.";
};

// `as` 키워드를 사용해서 string으로 강제하지 않으면 타입스크립트 컴파일러 단계에서 에러 발생
validateInputText(loaded_text as string);
```

이외에도 타입 카드 패턴을 사용해 타입을 검사할 수 있다. (4.2)

&nbsp;

&nbsp;

# 2.3 원시 타입

자바스크립트의 7가지 원시 값은 타입스크립트에서 원시 타입으로 존재한다. 자바스크립트의 내장 타입은 파스칼 표기법을 사용하고 같은 것을 타입스크립트에서는 소문자로 표기한다.


```tsx
⌛ 원시 래퍼 객체
```

1. **boolean**
2. **undefined**
3. **null**
4. **number** (NaN, Infinity)
5. **bigInt** - number과 서로 다른 타입이기 때문에 상호작용 불가능
6. **string** - 공백도 string, 템플릿 리터럴 포함
7. **symbol** - Symbol() 함수를 사용하면 어떤 값과도 중복되지 않는 유일한 값을 생성할 수 있다.

❗**undefined와 null의 차이**

```tsx
type Person1 = {
  name: string;
  job?: string; // 직업이 있을 수도 있고 없을 수도 있음

type Person2 = {
  name: string;
  job: string | null; // 직업은 모두가 있으나 무직일 수도 있음
};
```

&nbsp;

&nbsp;

# 2.4 객체 타입

앞에서 언급한 7가지 원시타입이 아니라면 모두 객체 타입이다.

## 🟡 1: object

자바스크립트의 객체의 정의에 대응하는 타입스크립트 타입 시스템은 object 타입이다. 가급적 사용하지 말도록 권장된다. 앞서 말한 7가지 타입에 속하지 않는다.

&nbsp;

## 🟡 2: {}

중괄호는 자바스크립트에서 객체 리터럴 방식으로 객체를 생성할 때 사용한다. 타입스크립트에서 객체를 타이핑할 때도 중괄호를 쓸 수 있는데, 중괄호 안에 객체의 속성 타입을 지정해주는 식으로 사용한다.

> 파라미터처럼 타입을 지정하는데 이것이 객체 내부의 속성 타입과 일치해야 한다.
> 

```tsx
const noticePopup: { title: string; description: string } = {
  title: "IE 지원 종료 안내",
  description: "2022.07.15일부로 배민상회 IE 브라우저 지원을 종료합니다.",
  // startAt: "2022.07.15 10:00:00", 지정한 타입이 존재하지 않으므로 오류
};

```

&nbsp;

빈 객체를 생성하기 위해 중괄호를 사용할 수도 있다. 

```tsx
let notice: {} = {};
// notice.title = "title 속성을 지정할 수 없음";
```

&nbsp;

## 🟡 3: array

> 타입스크립트는 자바스크립트 객체를 세분화해서 타입을 지정할 수 있는 타입 시스템을 가지고 있다.
> 
1. 자바스크립트 객체에는 객체 자료구조, 배열, 함수, 정규식 등이 포함된다.
2. 타입스크립트는 이 각각의 객체에 타입을 지정할 수 있다.
    
    ❗ 자바스크립트의 배열 안의 원소들은 서로 다른 타입을 가질 수 있다. 이와 다르게 타입스크립트는 하나의 타입만 가질 수 있다. 타입스크립트에서 배열이 여러 타입을 가지게 하려면 유니온 타입을 사용해야 한다.
    
    ```tsx
    const mixedArray = [1, "hello", true, { key: "value" }];
    const numbers: number[] = [1, 2, 3];
    // numbers.push("hello"); // 오류: 'string' 타입은 'number' 타입에 추가할 수 없습니다.
    const mixedArray: (number | string | boolean)[] = [1, "hello", true];
    ```
    

&nbsp;

## 🟡 4: type과 interface 키워드

객체를 타이핑하기 위해 type과 interface를 주로 사용한다.  키워드를 사용하면 반복을 줄여서 타입을 사용할 수 있다.

- 타입스크립트에서는 변수 타입을 명시적으로 선언하지 않아도 컴파일러가 자동으로 타입을 추론한다.

+. 언제 type을 쓰고 언제 interface를 써야 하는지?

&nbsp;
## 🟡 5: function

함수도 일종의 객체이지만 typeof로 출력하면 엄연한 function이라는 타입으로 분류한다. 타입스크립트에서 함수를 별도 함수 타입으로 지정할 수 있는데 몇 가지 주의사항이 있다.

1. function 키워드 자체를 타입으로 사용하지 않는다.
2. 함수의 매개변수도 별도 타입으로 지정해야 한다.

```tsx
function adda: number, b: number): number {
	return a+b;
}
```

함수 자체의 타입은 호출 시그니처를 정의하는 방식으로 지정할 수 있다. 이때 화살표 함수 방식으로 호출 시그니처를 정의해야 한다.

> 호출 시그니처는 타입스크립트에서 함수 타입을 정의할 때 사용하는 문법이다. 함수 타입은 해당 함수가 받는 매개변수와 리턴하는 값의 타입으로 결정된다. 호출 시그니처는 이 2가지를 명시하는 역할을 한다.
> 

```tsx
type Multiply = (a: number, b: number) => number;

const multiply: Multiply = (x, y) => x * y;

console.log(multiply(2, 3)); // 출력: 6
```