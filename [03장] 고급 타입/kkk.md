# 

```tsx
3.1 타입스크립트만의 독자적 타입 시스템
3.1.1 any 타입
3.1.2 unknown 타입
3.1.3 void 타입
3.1.4 never 타입
3.1.5 Array 타입
3.1.6 enum 타입

3.2 타입 조합
3.2.1 교차 타입(Interaction)
3.2.2 유니온 타입(Union)
3.3.3 인덱스 시그니처(Index Signatures)
3.3.4 인덱스드 엑세스 타입(Indexed Access Types)
3.3.5 맵드 타입(Mapped Types)
3.3.6 템플릿 리터럴 타입(Template Literal Types)
3.3.7 제네릭(Generic)
```

# 3.1 타입스크립트만의 독자적 타입 시스템

자바스크립트에서도 any 타입과 같은 “개념”은 사용되고 있었으나 이를 표현하지 않았을 뿐이다. 타입은 타입스크립트에만 존재한다.

![image.png](https://github.com/user-attachments/assets/86cde375-6f4f-4f77-9fd1-38f65654b41f)

# 3.1.1 any 타입

자바스크립트에서 타입을 명시하지 않은 것과 동일한 효과를 나타낸다. 

1. 타입스크립트는 동적 타이핑 특징을 가진 자바스크립트에 정적 타이핑을 적용하는 것이 주된 목적이지만
2. any 타입을 사용하면 이러한 목적이 무시된다.
3. 그래서 기본적으로 any 타입은 가급적 지양해야 한다.
4. 그럼에도 any 타입을 사용해야 하는 경우가 존재한다.

&nbsp;

## ▶️ any 타입을 사용하는 3가지 경우

```tsx
1. 개발 단계에서 임시로 값을 지정해야 할 때
2. 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때
3. 값을 예측할 수 없을 때 암묵적으로 사용
```

### ⏩ 개발 단계에서 임시로 값을 지정해야 할 때

아직 타입이 확정되지 않거나 추후 변경 가능성이 있을 때 개발 속도 향상, 즉 타입 명시에 드는 시간을 줄이기 위해 임시로 any를 사용할 수 있다.

### ⏩ 어떤 값을 받아올지 또는 넘겨줄지 정할 수 없을 때

API 요청 및 응답 처리, 콜백 함수 전달, 타입 파악이 힘든 외부 라이브러리 등을 사용할 때는 any 타입을 사용해야 할 수 있다.

```tsx
type FeedbackModalParams = {
	show: boolean;
	content: string;
	canceLButtonText?: string;
	confirmButtonText?: string;
	beforeOnClose?: 0 => void;
	action?: any;
};
```

“action” 속성은 피드백 모달 창을 그릴 때 사용되는 “함수의 타입”이다. 파라미터의 개수가 리턴 타입이 함수마다 다를 수 있으므로 any 타입을 사용해 다양한 action 함수를 전달할 수 있다.

### ⏩ 값을 예측할 수 없을 때 암묵적으로 사용

외부 라이브러리나 웹 API의 요청에 따라 다양한 값을 반환하는 API가 존재할 수 있다. 예시로 Fetch API의 일부 메서드는 요청 이후의 응답을 특정 포맷으로 파싱하는데 이 때 반환 타입이 any로 매핑되어 있다.

```tsx
async function loado {
	const response = await fetch("https://api.com");
	const data = await response.json0; // response.json0 의 리턴 타입은 Promise<any>로
	정의되어 있다
	return data;
}
```

❗위의 경우에서도 가능한 any 타입을 지양해야 한다. 컴파일 타임에서는 에러가 발생하지 않겠지만 런타임에서 오류가 발생될 수 있는 여지가 많아진다.

&nbsp;
# 3.1.2 unknown 타입

> 무엇이 할당될지 아직 모르는 상태의 타입, unknown은 모든 타입이 될 수 있지만 어떤 타입인지 “아는” 타입은 unknown이 될 수 없다.
> 

unknown 타입은 any 타입처럼 “모든 타입의 값”이 할당될 수 있다. 그러나 다른 타입의 변수에 unknown 타입을 할당할 수 없다. (any 제외)

![image.png](https://github.com/user-attachments/assets/38e8c74b-6a56-4889-be67-ed0613b192cf)

```tsx
let unknownValue: unknown;
unknownvalue = 100; // any 타입과 유사하게 숫자이든
unknownvalue = "helLo world";  // 문자열이든
unknownvalue = 0 => console.log("this is any type"); // 함수이든 상관없이 할당이 가능하지만

let somevalue1: any = unknownValue; // (o) any 타입으로 선언된 변수를 제외한 다른 변수는 모두 할당이 불가
let someValue2: number = unknownValue; // (X)
let someValue3: string = unknownValue; // (X)
```

## ❓ 함수를 unknown 타입 변수에 할당하면?

```tsx
// 할당하는 시점에서는 에러가 발생하지 않음
const unknownFunction: unknown = ( =› console.log("this is unknown type");
// 하지만 실행 시에는 에러가 발생 ; Error: Object is of type 'unknown'.ts (2571)
unknownFunction();
```

1. unknown 타입은 어떤 타입이 할당되었는지 “알 수 없음”을 나타내기 때문에 
2. unkonwn 타입으로 선언된 변수는 **값을 가져오거나 내부 속성에 접근할 수 없다.**

&nbsp;

## ❓ 왜 unknown 타입을 사용하나요?

any 타입을 사용하면 어떤 값이든 허용하고 추후 수정을 다짐하며 임시로 any 타입을 지정하기도 한다. 그러나 수정을 미처 하지 못한 경우 버그가 발생할 수 있다. 

unknown 타입은 이러한 상황에서 any 타입 대신 사용할 수 있다. any 타입과 유사하지만 타입 검사를 강제하고 타입이 식별된 후에 사용할 수 있어 any 타입보다 더 안전하다.

&nbsp;

# 3.1.3 void 타입

함수의 리턴타입이 없는 경우 void이다. 자바스크립트에서 함수에 리턴을 생략하면 기본적으로 “undefined”가 리턴된다. 타입스크립트에서는 void 타입을 사용한다. 하지만 void를 잘 명시하지 않는데 타입스크립트 컴파일러가 리턴이 없으면 알아서 void로 추론해주기 때문이다.

void는 함수에만 국한되는 개념은 아니지만 변수에 할당하는 것은 별 의미가 없다.

&nbsp;

# 3.1.4 never 타입

타입 값을 반환할 수 없는 타입이다. 

## ▶️ 자바스크립트에서 값을 반환할 수 “없는” 경우

### ⏩ 에러를 던지는 경우

1. 자바스크립트에서는 런타임에 의도적으로 thorw 키워드를 사용해 에러를 발생시키고 캐치할 수 있다.
2. ❗ 이는 값을 반환하는 것이 아니다.
3. 함수가 실행 중에 에러를 던진다면 해당 함수의 리턴 타입은 “never”이 된다.

### ⏩ 무한히 함수가 실행되는 경우

무한루프는 함수가 종료되지 않음을 의미하기 때문에 never 타입이 된다.

> never 타입은 모든 타입의 하위 타입이다. never 타입을 제외한 모든 타입은 never 타입에 할당될 수 없다. (가장 자식, 구체화된, 하위)
> 

&nbsp;

# 3.1.5 Array 타입

> 배열 타입을 나타낸다.
> 

## ❓자바스크립트의 배열과 타입스크립트의 배열은 무엇이 다른가요?

1. 자바스크립트에서는 배열을 “객체에 속하는 타입”으로 분류한다. 배열이라는 단독 자료형으로 구분하지 않는다. 
2. 타입스크립트에서 Array라는 타입을 사용하기 위해서는 타입스크립트의 특수한 문법을 함께 다뤄야 한다.

&nbsp;

## ❗Array 키워드와 대괄호[]를 사용할 때의 차이점

> 배열은 Array 키워드 외에도 대괄호([])를 사용해 직접 타입을 명시할 수도 있는데, 이 때의 타입은 배열보다 더 좁은 범위인 튜플을 가리킨다.
> 

❕**배열 타입을 선언하는 2가지 방법**

```tsx
const array: number[] = [1, 2, 3]; 
const array: Array<number> = [1, 2, 3];
```

```tsx
// 유니온 타입
const array1: Array<number | string> = [1, "im string"]
const array2: number[] | string[] = [1, "im string"]
const array3: (number | string)[] = [1, "im string"]
```

❔**튜플이 뭔가요?**

배열 타입의 하위 타입으로 기존 타입스크립트의 배열 기능에 “**길이 제한**”을 추가한 타입 시스템

- 튜플은 대괄호 안에 타입시스템을 기술해서 선언한다.

> 튜플은 배열의 특정 인덱스에 정해진 타입을 선언하는 것과 같다.
> 

```tsx
let tuple: [number] = [1]; // 튜플의 정의: 길이가 1이고, 첫 번째 요소가 number 타입인 배열
tuple = [99]; // O
tuple = [1,2]; // X 불가능
tuple = [1,"string"]; // X 불가능

let tuple: [number, string, boolean] = [1,"string",true]; // 여러 타입과 혼합 가능 
```

## ▶️  튜플의 사용 예시 - useState API

useState API는 배열 원소의 자리마다 명확한 의미를 부여한다. 또 구조 분해 할당을 사용해 사용자가 자유롭게 이름을 정의할 수 있다.

```tsx
import {useState} from "react";

const [value, setValue] = useState(false);
const [username, setUsername] = useState("");
```

### ❗ 객체에서도 구조 분해 할당을 사용할 수 있다.

```tsx
const useStateWithObject = (initialValue:any) => {
	...
	return { value, setValue };
};
// 해당 함수에서 정의된 속성 이름으로 가져와야 한다.
const {value, setValue} = useStateWithObject(false); 
// 사용자 정의 이름으로 사용하려면 위처럼 일단 가져오고 앨리어스한다.
const {value:username, setValue:setUsername} = useStateWithObject('')
```

### ❕튜플과 배열의 성질을 혼합해 사용하는 경우

스프레드 연산자(…)을 사용해 “특정 인덱스”는 서로 다른 타입을 명시하고 나머지는 배열처럼 동일한 자료형을 개수 제한 없이 받도록 할 수 있다.

```tsx
const httpStatusFromPaths : [number, string, ...string[]] = [
	400,
	"Bad Request",
	"/users/:id",
	"/users/:userId",
	"/users/:uuid",
];
// 첫번째 자리와 두번째 자리는 반드시 number과 string, 그 이후는 전부 string 배열
```

“옵셔널 프로퍼티”를 사용해 선택적으로 설정할 수도 있다.

```tsx
// 선택적 프로퍼티를 사용한 경우
const httpStatusWithOptionalCode: [number, string, number?, ...string[]] = [
  404, // 필수 숫자
  "Not Found", // 필수 문자열
  500, // 선택적 숫자 (생략 가능)
  "/articles/:articleId", // 나머지 문자열 배열
  "/posts/:postId",
];

// 선택적 프로퍼티 생략된 경우
const httpStatusWithoutOptionalCode: [number, string, number?, ...string[]] = [
  200,
  "OK",
  "/home",
  "/dashboard",
];
```

&nbsp;

# 3.1.6 enum 타입

“열거형”이라고도 한다. 일종의 구조체를 만드는 타입 시스템이다. 

- 명명한 각 멤버의 값을 스스로 추론한다. 기본적인 추론 방식은 숫자 0부터 1씩 늘려가며 값을 할당한다.
- 누락된 멤버도 이전 멤버 값의 숫자를 기준으로 1씩 늘려가며 할당한다.

```tsx
enum Program {
	Typescript,
	Javascript,
	Java,
	Go,
}
Program.Typescript; // 0
Program.Go; // 3
Program["Go"] // 3
Program[2] // Java

enum Program2 {
	Ts = "ts",
	Js = "js",
	Java = 300,
	K, // 301
	R, // 302
	G, // 303
}
```

## ❕**열거형의 주의점**

### ▶️  자동 추론 열거형

- 자동으로 추론한 열거형은 오류가 발생할 수 있다. 또 역방향으로도 접근할 수 있어 범위를 넘어서서 역방향으로 접근해도 타입스크립트는 막지 않는다.

```tsx
enum Program {
    Ts = "ts",  
    Js = "js",       
    Java = 300,     
    K = "k",       
    R,               // 오류 발생: 문자열 뒤에 자동 값 할당 불가
    G = 400,        
    H               
}
```

enum은 인덱스 개념이 아니고 키-값쌍이다.

### ▶️ 숫자 상수로 관리되는 열거형의 문제점

```tsx
enum Program {
    Ts = "ts",
    Js = "js",
    Java = 300,
    K,
}

console.log(Program[2]) // undefined
console.log(Program[300]) // Java
console.log(Program[301]) // K
console.log(Program[302]) // undefined가 출력되고 에러가 발생하지 않는다.
```

❗이처럼 존재하지 않는 value로 접근해도 에러가 발생하지 않는다.❗

> 이를 해결하기 위해 “const enum”으로 열거형을 선언할 수 있다. 해당 방식을 사용하면 역방향(value)으로의 접근을 허용하지 않는다.
> 

그러나 value가 숫자인 열거형은 여전히 접근이 가능하고 에러가 발생하지 않는다. 문자열 상수 방식으로 접근하면 에러가 발생한다.

🔸 **그러므로 문자열 상수 방식으로 열거형을 사용하는 것이 더 안전하다. 애초에 자동할당이 되지 않도록 명시적으로 정해주는 것이 좋을 듯**🔸

```tsx
const enum NUMBER {
	ONE = 1,
	TWO = 2,
}
const myNumbeer: NUMBER = 100; // enum에 100이 없지만 에러가 발생하지 않는다.

const enum STRING_NUMBER {
	ONE = "1",
	TWO = "2",
}
const myStringNumber : STRING_NUMBER = "THREE"; // ERROR
```

&nbsp;

### ▶️ 열거형은 타입 공간과 값 공간에서 모두 사용되기 때문에 발생하는 문제점

열거형의 가장 큰 문제이다.

1. 열거형은 타입공간과 값 공간에서 모두 사용된다.
2. 컴파일 이후 자바스크립트에서 열거형은 “즉시 실행 함수(IIFE)” 형식으로 변환된다.

이 때 일부 번들러에서 트리쉐이킹 과정에서 *즉시 실행 함수로 변환된 값 = 열거형을 사용하지 않는 코드로 인식하는 경우가 있다.* ?따라서 불필요한 코드의 크기 증가하는 결과를 초래할 수 있다?

이를 해결하기 위해 `const enum` 또는 `as const assertion`을 사용해 유니온 타입으로 열거형과 동일한 효과를 얻는 방법이 있다.

+.트리쉐이킹(사용하지 않는 코드 번들에서 제거)

&nbsp;

# 3.2 타입 조합

# 3.2.1 교차 타입(Intersection)

여러 가지 타입을 결합해 하나의 타입으로 만들 수 있다. `&`를 사용하고 새로 생신 타입에 별칭 alias를 붙일 수 있다.

```tsx
type ProductItem = {
	id: number;
	name : string;
	quantity : number;
};
type ProductItemWithA = ProductItem & {discountAmount : number}; 
```

&nbsp;

# 3.2.2 유니온 타입(Union)

`|`로 표시하며 가질 수 있는 타입을 전부 나열한다. 별칭을 사용할 수 있다.

```tsx
type CardItem = {
	id: number;
	name: string;
};
type PromotionEventItem = ProductItem | CardItem;

const printPromotionItem = (item:PromotionEventItem) => {
	console.log(item.name); // 0, name은 공통 속성
	
	console.log(item.quantity); // 컴파일 에러 발생, ProductItem에만 있는 속성
};
```

&nbsp;

# 3.2.3 인덱스 시그니처(Index Signatures)

1. 특정 타입의 속성 이름은 알 수 없지만 
2. “속성값의 타입”을 알고 있을 때 사용하는 문법이다.

🔹 **[Key: K] : T** 🔹

- 해당 타입의 “속성 키”는 모두 K타입이어야 하고
- “속성값”은 모두 T타입을 가져야 한다.

```tsx
// 속성 키는 모두 string이고 속성값은 모두 number이다.
interface IndexSignatureEx {
	[key : string] : number;
}
```

🔹**다른 속성 추가**🔹

추가로 명시된 속성은 인덱스 시그니처에 포함되는 타입이어야 한다. 

```tsx
interface IndexSignatureEx2 {
	[key: string] : number | boolean;
	length: number;
	isValid: boolean;
	name: string; // 에러 발생
	// 인덱스 시그니처의 키가 string이면 number나 boolean을 가져야 하기 때문
}
```

&nbsp;

# 3.2.4 인덱스드 엑세스 타입(Indexed Access Types)

> “다른 타입”의 “특정 속성이 가지는 타입”을 조회하기 위해 사용
> 

인덱스에 사용되는 타입도 타입이기 때문에 유니온 타입, keyof, 타입 별칭 등의 표현을 사용할 수 있다.

```tsx
type Example = {
	a: number;
	b: string;
	c: boolean;
};
type IndexedAccess = Example['a'];
type IndexedAccess2 = Example['a'|'b'];
type IndexedAccess3 = Example[keyof Example];

type ExAlias = "b" | "c";
type IndexedAccess4 = Example[ExAlias];

```

## ▶️  인덱스드 엑세스 타임 이용해서 배열의 요소 타입을 조회하기

1. 배열 타입의 모든 요소는 전부 동일한 타입을 가진다.
2. 배열의 인덱스는 숫자이다. number로 인덱싱이 가능하다.

```tsx
// TODO 오류 > PR#16으로 코드 수정
const PromotionList = [
	{type: "product", name: "chicken"},
	{type: "product", name: "pizza"},
	{type: "card", name: "cheer-up"},
];
 
// type PromotionItemType = { type: string; name: string }
type PromotionItemType = typeof PromotionList[number];
```

&nbsp;

# 3.2.5 맵드 타입(Mapped Types)

1. 자바스크립트의 map는 배열 A를 기반으로 새로운 배열 B를 만들어내는 배열 메서드이다.
2. 타입스크립트의 map는 “**다른 타입을 기반으로 한 타입**”을 선언할 때 사용하는 문법이다. 

```tsx
type Example = {
	a: number;
	b: string;
	c: boolean;
};

// 맵드 타입 문법 : 각 키에 대해 순회하면서 T[K],각 키의 타입을 가져온다.
type Subset<T> = {
	[K in keyof T]?: T[K];
};

// Example을 기반으로 한 객체들
const aExample : Subset<Example> = {a: 3;};
const bExample: Subset<Example> = {b: "hello"};
const acExample: Subset<Example> = {a: 4, c: true};
```

c. `readonly`와 `?(선택적 매개변수)`를 사용할 수 있다. readonly나 ? 앞에 `-`를 붙여주면 제거할 수 있다. 

```tsx
type ReadOnlyEx = {
	readonly a: number;
	readonly b: string;
};

type CreateMutable<Type> = {
	-readonly [Property in keyof Type]: Type[Property]; // readonly를 뺀다
};

type ResultType = CreateMutable<ReadOnlyEx>; // { a: number; b: string }

type OptionalEx = {
	a?: number;
	b?: string;
	c: boolean;
};

type Concrete<Type> = {
	[Property in keyof Type]-?: Type[Property]; // 옵셔널을 뺀다
};

type ResultType = Concrete<OptionalEx>; /// { a: number; b: string, c: boolean }
```

d. 맵드 타입이 실제로 사용된 예시

❔**BottomSheetMap에 존재하는 모든 키에 대해 각각 resolver, args, isOpened 상태를 관리하는 스토어가 필요하다. 어떻게 구현할 수 있을까?**

```tsx
const BottomSheetMap = {
	RECENT_CONTACTS: RecentContactsBottomSheet,
	CARD_SELECT: CardSelectBottomShet,
	SORT_FILTER: SortFilterBottomSheet,
	PRODUCT_SELECT: ProductSelectBottomSheet,
	...
	BASE: null,
};

// 불필요한 반복 발생
type BottomSheetStore = {
	RECENT_CONTACTS: {
		resolver?: (payload: any) => void;
		args?: any;
		isOpened: boolean;
	};
	CARD_SELECT: {
		resolver?: (payload: any) => void;
		args?: any;
		isOpened: boolean;
	};
	SORT_FILTER : {
		resolver?: (payload: any) => void;
		args?: any;
		isOpened: boolean;
	};
	//...
};
export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;

// Mapped Types를 통해 효율적으로 타입을 선언할 수 있다.
type BottomSheetStore = {
	[index in BOTTOM_SHEET_ID]: {
		resolver?: (payload: any) => void;
		args?: any;
		isOpened: boolean;
	};
};
```

> 인덱스 시그니처 문법을 사용해 반복적인 타입 선언을 효과적으로 줄일 수 있다.
> 

맵드 타입에서는 “as” 키워드를 사용해 키를 재지정할 수 있다. 

```tsx

// 모든 키에 _BOTTOM_SHEET를 붙여 키를 재지정
type BottomSheetStore = {
	[index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
		resolver?: (payloda: any) => void;
		arges?: any;
		isOpened: boolean;
	};
};
```

&nbsp;

# 3.2.6 템플릿 리터럴 타입 (Template Literal Types)

자바스크립트의 템플릿 리터럴 문자열을 사용해 문자열 리터럴 타입을 선언하는 문법이다.

🔹 활용 예시

```tsx
type Stage = 
	| "init"
	| "select-image";

type StageName = `${Stage}-stage`;
// 결과
type StageName = 
	| "init-stage"
	| "select-image-stage";
```

Stage 타입의 모든 유니온 멤버 뒤에 -stage를 붙여 새로운 유니온 타입을 만들었다. 

1. 템플릿 리터럴의 변수 자리에 문자열 리터럴인 “유니온 타입”을 넣으면
2. 해당 유니온 타입 멤버들이 차례로 변수에 들어가 리턴된다.

&nbsp;

# 3.2.7 제네릭(Generic)

제네릭은 C나 자바 같은 정적 언어에서 “다양한 타입” 간에 “**재사용성을 높이기”** 위해 사용하는 문법이다. 타입스크립트도 정적 타입을 가지는 언어이기 때문에 제네릭 문법을 지원한다.

1. 타입스크립트의 제네릭은 타입, 클래스 등에서 
2. *내부적으로 사용할 타입을 미리 정해두지 않고* 타입 변수를 사용해 해당 위치를 비워둔 다음
3. 실제로 그 값을 사용할 때 외부에서 타입 변수 자리에 타입을 지정해 사용하는 방식이다.

일반적으로 <T>와 같이 꺾쇠괄호 내부에 정의된다. 보통 한 글자로 사용한다. (T(Type), E(Element), K(Key), V(Value))

```tsx
type ExampleArrayType<T> = T[];
const array1: ExampleArrayType<string> = ["치킨", "피자", "우동"];
```

## ▶️ <>꺾쇠괄호 안에 타입 명시를 생략하면 컴파일러가 타입을 추론한다.

```tsx
function exampleFUnc<T>(arg: T): T[] {
	return new Array(3).fill(arg);
}

exampleFunc("hello"); // 타입 생략, T는 string으로 추론된다.
// exampleFunc<string>("hello"); 타입 명시
```

## ▶️ 제네릭 타입에 디폴트를 추가할 수 있다.

특정 요소 타입을 알 수 없을 때는 제네릭 타입에 기본값을 추가할 수 있다.

```tsx
interface SubmitEvent<T = HTMLElement> extends SyntheticEvent<T> {
	submitter: T;
}
```

## ▶️ 특정 속성을 가진 타입만 받도록 제약을 걸 수 있다.

```tsx
function example<T>(arg: T): number {
	return arg.length; // 에러 발생, length가 없는 타입이 올 수 있기 때문
}
```

제네릭 꺾쇠괄호 내부에 ‘length 속성을 가진 타입만 받는다’는 제약을 걸어주면 된다.

```tsx
interface TypeWithLength {
	length: number;
}
function example2<T extends TypeWithLength>(arg: T): number {
	return arg.length;
}
```

## ▶️ 제네릭을 TSX에서 사용할 때 주의점

파일 확장자가 tsx일 때 화살표 함수에 제네릭을 사용하면 에러가 발생한다.

```tsx
// 에러 발생 : JSX element 'T' has no corresponding closing tag
const arrowExampleFunc = <T>(arg: T): T[] => {
	return new Array(3).fill(arg);
};
```

❓tsx는 타입스크립트 + JSX이므로 꺾쇠괄호와 HTML 태그의 꺾쇠괄호를 혼동하여 생기는 문제이다.  (`<div>`)

❕제네릭 부분에 extends 키워드를 사용해 컴파일러에게 특정 타입의 하위 타입만 올 수 있음을 알려주면 된다. 보통 제네릭을 사용할 때는 function 키워드로 선언하는 경우가 많다.

```tsx
const arrowExampleFunc2 = <T extends {}>(arg: T): T[] => {
	return new Array(3).fill(arg);
};
```

&nbsp;