# 3. 고급 타입

## 3.1 타입스크립트만의 독자적 타입 시스템

### 1. any 타입

타입을 명시하지 않은 것과 동일한 효과를 나타낸다.

`any` 타입에는 어떠한 값을 할당하더라도 오류가 발생하지 않는다.

정적 타입의 관점에서 `any` 타입을 남발하는 것은 지양해야 할 패턴으로 알려져 있으나, 상황에 따라 사용해야 할 때가 존재할 수 있다.

> `tsconfig.json`에서 `noImplicitlyAny` 옵션을 활성화함으로써 타입이 명시되지 않은 암묵적인 any 타입에 대한 경고를 발생시킬 수 있다.

대표적인 3가지 사례가 있다.

1. 개발 단계에서 임시 값을 지정할 때

    개발 과정에서 값이 나중에 변경되거나 아직 세부 항목에 대한 타입이 확정되지 않았을 때 사용한다. 세부 스펙이 나오는대로 다른 타입으로 대체해주는 것이 좋다.

2. 어떤 값을 받아올지, 넘겨줄지 정할 수 없을 때

    API 요청 및 응답 처리, 콜백 함수 전달, 타입이 잘 정제되지 않아 파악이 힘든 외부 라이브러리를 사용할 때는 어떤 인자를 주고 받을지 특정하기 어렵다.

3. 값을 예측하기 어려울 때

    외부 라이브러리나 웹 API 요청에 따라서는 다양한 값을 반환하는 API가 있을 수 있다. 대표적으로 브라우저의 Fetch API가 있다.

    ```ts
    async function load {
        const response = await fetch("https://api.com");
        const data = await response.json(); // response.json0 의 리턴 타입은 Promisecany 〉로 정의되어 있다
        return data;
    }
    ```

&nbsp;

### 2. unknown 타입

3.0 버전에서 등장한 `unknown` 타입에는 모든 타입의 값이 할당될 수 있으나 `any`를 제외한 다른 타입으로 선언된 변수에는 `unknown` 타입 값을 할당할 수 없다.

```ts
let unknownValue: unknown;
unknownvalue = 100; // any 타입과 유사하게 숫자이든
unknownvalue = "helLo world"; // 문자열이든
unknownvalue = () => console.log("this is any type"); // 함수이든 상관없이 할당이 가능하지만

let somevalue1: any = unknownValue; // (O) any 타입으로 선언된 변수를 제외한 다른 변수는 모두 할당이 불가
let someValue2: number = unknownValue; // (X)
let someValue3: string = unknownValue; // (X)
```

이름처럼 **어떤 것이 할당될지 아직 모르는 상태의 타입**을 말한다.

`any`와는 다르게 `unknown`으로 할당된 객체 내부에 접근하는 모든 시도(객체 속성 접근, 인스턴스 생성 등)에 에러를 발생시킨다. 타입 명시를 회피한 후 잊어버리는 경우를 방지해준다.

&nbsp;

### 3. void 타입

JS의 경우 명시적인 반환문을 작성하지 않으면 `undefined`가 반환되지만 TS의 경우 `void` 타입을 사용할 수 있고, 이는 `undefined`가 아니다.

`void` 타입으로 지정된 변수에는 `undefined` 또는 `null`만 할당할 수 있다.

함수 내부에 별도 반환문이 없으면 컴파일러가 자동으로 함수 타입을 `void`로 추론하기 때문에 생략해도 상관없다.

&nbsp;

### 4. never 타입

함수와 관련하여 많이 사용되며, 값을 반환할 수 없는 타입을 말한다. 여기서 값을 반환하지 않는다는 것은

1. 에러를 던지거나
2. 무한히 함수가 실행되는 경우

를 말한다.

`throw` 키워드를 통한 에러 발생은 값을 반환하는 것으로 간주되지 않는다. 따라서 `never` 타입을 반환한다.

```ts
function generateError(res: Response): never {
  throw new Error(res.getMessage());
}
```

아래와 같이 무한 루프 함수의 경우도 `never`  타입을 반환하는 경우이다.

```ts
function checkStatus(): never {
  while (true) {
    // ...
  }
}
```

&nbsp;

### 5. Array 타입과 튜플 타입

JS에서는 배열을 객체에 속하는 타입으로 분류하여 배열이라는 단독 자료형으로서 존재하지 않는다.

TS에서는 `Array` 타입을 사용하기 위해 특수한 문법을 사용한다.

1. `자료형[]`

```ts
const array: number[] = [1, 2, 3];
```

2. `Array<자료형>` (제네릭)

```ts
const array: Array<number> = [1, 2, 3]
```

형식상의 차이만 있을 뿐 완전히 같은 방식의 선언이다.

만약 복수의 타입을 갖는 배열을 선언하고 싶으면 유니온 타입을 사용하면 된다.

```ts
const array1: Array<number | string> = [1, "string"];
const array2: number[] | string[] = [1, "string"];

// 후자의 방식은 아래와 같이 선언할 수도 있다
const array3: (number | string)[] = [1, "string"];
```

&nbsp;

대괄호 안에 타입을 명시해서 튜플을 선언해줄 수도 있다.

배열은 모든 요소가 같은 타입이며 길이가 고정되지 않는 반면 튜플은 각 인덱스에 타입을 지정해줄 수 있고 고정 길이를 갖는다.

```ts
// 일반 배열: 모든 요소가 같은 타입이며, 길이가 고정되지 않음
let fruits: string[] = ['사과', '바나나', '오렌지'];
fruits.push('망고');  // OK
fruits[0] = '포도';   // OK

// 튜플: 각 위치별 타입이 지정되고 길이가 고정됨
let person: [string, number] = ['John', 30];
// person[0]은 반드시 string
// person[1]은 반드시 number
// 길이는 반드시 2개
```

> React의 상태 값과 상태에 대한 세터(setter) 쌍을 갖는 `useState` 훅은 튜플 타입을 반환하는 대표적인 예시다.

&nbsp;

스프레드 연산자를 사용해서 튜플과 배열의 성질을 혼합해서 사용할 수도 있다.

```ts
const httpStatusFromPaths: [number, string, ...string[]] = [
  400,
  "Bad Request",
  "/users/:id",
  "/users/:userId",
  "/users/:uuid",
];
// 첫 번째 자리는 숫자(400), 두 번째 자리는 문자열(‘Bad Request’)을 받아야 하고, 그 이후로는 문자열 타입의 원소를 개수 제한 없이 받을 수 있음
```

또한 속성을 선택적으로 사용하고 싶으면 `?` 연산자를 사용해서 속성을 선언해주면 된다. 이렇게 해주면 해당 인덱스에 해당 원소가 없어도 된다.

```ts
const optionalTuple1: [number, number, number?] = [1, 2];
const optionalTuple2: [number, number, number?] = [1, 2, 3]; // 3번째 인덱스에 해당하는 숫자형 원소는 있어도 되고 없어도 됨을 의미한다
```

> 💡 옵셔널(optional):
특정 속성 또는 매개변수가 값이 있을 수도 있고 없을 수도 있는 것을 의미한다. 즉, 선택적 매개변수 (옵셔널 파라미터) 또는 선택적 속성 (옵셔널 프로퍼티)은 필수적으로 존재하지 않아도 되며 선택적으로 사용될 수 있음을 나타낸다. 선택적 속성은 해당 속성에 값을 할당하지 않아도 되고 해당
속성이 없어도 오류가 발생하지 않는다. 이는 타입스크립트에서 좀 더 유연한 데이터 모델링과 사용자 정의 타입을 지원하기 위한 개념이다.

&nbsp;

### 6. enum 타입

타 언어와 비슷하게 동작한다. 각 멤버에 값을 할당해주지 않으면 자동으로 0부터 1씩 늘려가며 값을 할당해준다. 명시적으로 값을 할당해줄 수도 있다.

```ts
enum ProgrammingLanguage {
  Typescript, // 0
  Javascript, // 1
  Java, // 2
  Python, // 3
  Kotlin, // 4
  Rust, // 5
  Go, // 6
}

// 각 멤버에게 접근하는 방식은 자바스크립트에서 객체의 속성에 접근하는 방식과 동일하다
ProgrammingLanguage.Typescript; // 0
ProgrammingLanguage.Rust; // 5
ProgrammingLanguage["Go"]; // 6

// 또한 역방향으로도 접근이 가능하다
ProgrammingLanguage[2]; // “Java”
```

보통 문자열 상수를 생성하는데 유용하다.

```ts
enum ItemStatusType {
  DELIVERY_HOLD = "DELIVERY_HOLD", // 배송 보류
  DELIVERY_READY = "DELIVERY_READY", // 배송 준비 중
  DELIVERING = "DELIVERING", // 배송 중
  DELIVERED = "DELIVERED", // 배송 완료
}

const checkItemAvailable = (itemStatus: ItemStatusType) => {
  switch (itemStatus) {
    case ItemStatusType.DELIVERY_HOLD:
    case ItemStatusType.DELIVERY_READY:
    case ItemStatusType.DELIVERING:
      return false;
    case ItemStatusType.DELIVERED:
    default:
      return true;
  }
};
```

`itemStatus`의 타입을 그냥 문자열로 지정된 경우와 비교했을 때,

1. `enum` 타입에 명시되지 않은 문자열은 받을 수 없어 타입 안정성이 높다.
2. 타입이 다루는 값이 명확해 의미 전달과 응집력이 뛰어나다.
3. 열거형 멤버를 통해 어떤 상태를 나타내는지 쉽게 이해할 수 있어 가독성이 좋다. (`ItemStatusType.DELIVERING` = 상품이 배송 중인 상태를 의미하는 것을 쉽게 알 수 있음.)

&nbsp;

자동으로 추론한 열거형은 할당된 값을 넘어서는 범위로 역방향으로 접근하더라도 막지 못하는 문제가 있다.

이러한 동작을 막기 위해 `const enum`으로 열거형을 선언하는 방법이 있다. 이 방법은 역방향(`value` -> `key`)으로의 접근을 막는다.

`const enum`으로 선연하더라도 숫자 상수로 관리되는 열거형에 선언한 값 이외의 값을 할당하거나 접근할 때 이를 방지하지 못하는 문제가 있었으나,해당 오류는 5.0 버전부터 해결되었다. (5.0 릴리즈 노트: <https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-0.html>) 최신 버전에서는 다음과 같은 에러를 발생시킨다.

```ts
const enum NUMBER {
  ONE = 1,
  TWO = 2,
}
const myNumber: NUMBER = 100; // NUMBER enum에서 100을 관리하고 있지 않지만 이는 에러를 발생시키지 않는다

const enum STRING_NUMBER {
  ONE = "ONE",
  TWO = "TWO",
}
const myStringNumber: STRING_NUMBER = "THREE"; // Error
```

```ts
// Type '100' is not assignable to type 'NUMBER'.
```

열거형은 타입 공간과 값 공간에서 모두 사용되는데, 위의 예시에서 열거형은 JS 코드 변환 시 즉시 실행 함수(IIFE) 형식으로 변환된다. 이때 일부 번들러에서 트리쉐이킹 과정 중 즉시 실행 함수로 변환된 값을 사용하지 않는 코드로 인식해 불필요한 코드의 크기가 증가할 수 있다. 이는 `const enum`이나 `as const assertion`을 사용해 유니온 타입으로 열거형과 동일한 효과를 얻는 방법으로 해결할 수 있다. (해당 부분 잘 이해 안됨)

&nbsp;

## 3.2. 타입 조합

해당 절은 앞에서 다룬 내용을 응용하거나 좀 더 심화한 타입 검사를 수행하는 데 필요한 지식을 다룬다.

### 1. 교차 타입(Intersection)

교차 타입은 여러 가지 타입을 결합하여 만드는 **하나의 단일 타입**이다.

A 타입과 B 타입을 결합해 A와 B를 모두 만족하는 C라는 타입을 만든다.

`A & B` 형식으로 생성하며 별칭(alias)를 붙일 수도 있다.

```ts
type ProductItem = {
  id: number;
  name: string;
  ...
};

type ProductItemWithDiscount = ProductItem & { discountAmount: number };
```

&nbsp;

### 2. 유니온 타입(Union)

유니온 타입은 특정 변수가 가질 수 있는 타입을 모두 나열하는 용도로 사용한다.

A 타입과 B 타입 중 하나만 만족하는 C라는 타입을 만든다.

`A | B` 형식으로 생성한다.

아래와 같이 B에만 존재하는 멤버에 접근하려는 경우, 컴파일 에러를 발생시킨다.

```ts
type A = {
    name: string,
    age: number
}

type B = {
    name: string,
    age: number,
    job: string,
}

type C = A | B;

const printC = (content: C) => {
    console.log(content.job)
}
```

&nbsp;

### 3. 인덱스 시그니처(Index Signatures)

인덱스 시그니처는 객체의 속성 타입과 값 타입 쌍의 틀을 만들어 준다고 생각하면 쉽다. `type`이나 `interface` 내부에 `[Key: K]: T`의 형태로 타입을 명시해준다.

이는 해당 타입의 속성 키는 모두 `K`여야 한다는 의미다.

해당 타입의 멤버들의 키가 어떤 타입인지, 속성값은 어떤 타입인지 명시해줄 때 사용한다.

```ts
interface IndexSignatureEx2 {
  [key: string]: number | boolean;
  length: number;
  isValid: boolean;
  name: string; // 에러 발생
}
```

> 💡 속성과 멤버의 차이? 속성은 주로 객체나 클래스의 데이터를 나타내는 요소(키-값 쌍)를 말한다. 멤버는 속성보다 넓은 의미의 개념으로 속성을 포함한 메서드, 접근자 등을 포함한다.

&nbsp;

### 4. 인덱스드 엑세스 타입(Indexed Access Types)

인덱스드 엑세스 타입은 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용된다.

세 가지 방법이 있다.

1. 유니온 타입 사용

```ts
type Person = {
    name: string;
    age: number;
}

type IndexedAccess = Person["name" | "age"] // string | number
```

2. keyof 사용

```ts
type IndexedAccess = Person[keyof Person]
```

3. 타입 별칭(alias)

```ts
type TypeAlias = "name" | "age";
type IndexedAccess = Person[TypeAlias]
```

또한 배열 요소의 타입을 알고 싶을 때도 사용할 때도 있다.

```ts
const PromotionList = [
  { type: "product", name: "chicken" },
  { type: "product", name: "pizza" },
  { type: "card", name: "chee-up" },
];

type ElementOfPromotionList = typeof PromotionList[number];
type PromotionItemType = ElementOfPromotionList;
// type PromotionItemType = { type: string; name: string }
```

> 책의 코드는 `typeof ElementOf<T>` 이렇게 되어 있는데 위와 같이 사용해야 한다.

&nbsp;

### 5. 맵드 타입(Mapped Types)

JS의 map은 배열 A를 기반으로 한 새로운 배열 B를 만들어내는 배열 메서드다. 맵드 타입은 다른 타입을 기반으로 한 새 타입을 선언할 때 사용하는 문법이다.

인덱스 시그니처를 사용해서 반복적인 타입 선언을 줄이는데 유용하게 쓸 수 있다.

```ts
type Example = {
  a: number;
  b: string;
  c: boolean;
};

// T 타입의 모든 속성을 옵셔널로 갖는 또 다른 타입으로 매핑
type Subset<T> = {
  [K in keyof T]?: T[K];
};

const aExample: Subset<Example> = { a: 3 };
const bExample: Subset<Example> = { b: "hello" };
const acExample: Subset<Example> = { a: 4, c: true };
```

기존에 타입에 `readonly`나 `?` 수식어를 붙여서 매핑해줄 수 있다. 뿐만 아니라 `-?` 같은 식으로 해당 수식어를 제거한 타입을 선언해줄 수 있다.

맵드 타입을 실제로 사용한 예시 코드다. 각각의 바텀시트의 스토어를 생성하는 코드다.

```ts
const BottomSheetMap = {
  RECENT_CONTACTS: RecentContactsBottomSheet,
  CARD_SELECT: CardSelectBottomSheet,
  SORT_FILTER: SortFilterBottomSheet,
  PRODUCT_SELECT: ProductSelectBottomSheet,
  REPLY_CARD_SELECT: ReplyCardSelectBottomSheet,
  RESEND: ResendBottomSheet,
  STICKER: StickerBottomSheet,
  BASE: null,
};

export type BOTTOM_SHEET_ID = keyof typeof BottomSheetMap;

// 불필요한 반복이 발생
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
  // ...
};

// Mapped Types를 통해 효율적으로 타입을 선언할 수 있다
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

`as` 키워드를 사용하면 키의 별칭을 붙여줄 수 있다.

```ts
type BottomSheetStore = {
  [index in BOTTOM_SHEET_ID as `${index}_BOTTOM_SHEET`]: {
    resolver?: (payload: any) => void;
    args?: any;
    isOpened: boolean;
  };
};
```

&nbsp;

### 6. 템플릿 리터럴 타입(Template Literal Types)

템플릿 리터럴 타입은 JS의 템플릿 리터럴 문자열을 사용해 문자열 리터럴 타입을 선언할 수 있는 문법이다. 위에서 본 `as` 키워드를 사용한 예시가 이것이다.

객체 내 복수의 문자열 리터럴을 한 번에 변경할 때 유용하다.

```ts
type Stage =
  | "init"
  | "select-image"
  | "edit-image"
  | "decorate-card"
  | "capture-image";
type StageName = `${Stage}-stage`;
// ‘init-stage’ | ‘select-image-stage’ | ‘edit-image-stage’ | ‘decorate-card-stage’ | ‘capture-image-stage’
```

&nbsp;

### 7. 제네릭(Generic)

제네릭은 정적 언어에서 다양한 타입 간에 재사용성을 높이기 위해 사용하는 문법이다. 타입 매개변수를 받아 다양한 타입을 받을 수 있게 해 보일러플레이트 코드를 줄여준다.

```ts
type ExampleArrayType<T> = T[];

const array1: ExampleArrayType<string> = ["치킨", "피자", "우동"];
```

&nbsp;

제네릭은 **`any`와 유사한 부분이 있으나 둘은 엄연히 다르다**. 아래에 있는 인수를 받아 어떤 타입이든 그대로 반환하는 간단한 함수를 보자.

```ts
function identity(arg: any): any {
  return arg;
}
```

함수의 매개변수가 어떤 타입이든 인자로 받을 수 있다는 점에서 제네릭이지만, `any` 타입에 `number` 타입을 받아도 실제로는 함수가 반환활 때 어떤 타입인지에 대한 정보는 사라진다.

제네릭은 이러한 정보를 남길 수 있게끔 해준다.

```ts
function identity<Type>(arg: Type): Type {
  return arg;
}
```

&nbsp;

꺾쇠괄호 안에 타입을 명시해주지 않아도 컴파일러가 인수를 보고 타입을 추론하기도 해서 타입 추론이 가능한 경우 타입 명시를 생략할 수 있다.

```ts
function exampleFunc<T>(arg: T): T[] {
  return new Array(3).fill(arg);
}

exampleFunc("hello"); // T는 string으로 추론된다
```

특정 요소 타입을 알 수 없을 때는 제네릭 타입에 기본값을 추가할 수 있다.

```ts
// 타입의 기본값을 HTMLElement로
interface SubmitEvent<T = HTMLElement> extends SyntheticEvent<T> {
  submitter: T;
}
```

함수나 클래스 등 내부에서 제네릭을 사용할 땐 특정한 타입에만 존재하는 멤버를 참조하려고 하면 안된다.

```ts
function exampleFunc2<T>(arg: T): number {
  return arg.length; // 에러 발생: Property ‘length’ does not exist on type ‘T’
}
```

모든 타입에 length라는 멤버가 있지는 않으므로 논리적으로  불가능하다. 이럴 때는 아래와 같이 length 속성을 갖는 타입만 받는다는 제약을 걸어주어서 사용할 수 있다.

```ts
interface TypeWithLength {
  length: number;
}

function exampleFunc2<T extends TypeWithLength>(arg: T): number {
  return arg.length;
}
```

확장자가 tsx일 때, 제네릭은 주의해서 사용해야 한다. 제네릭의 꺾쇠괄호와 태그를 혼동하여 문제가 발생할 수 있다. 그럴 땐 아래와 같이 특정 하위 타입만 올 수 있음을 명시해주면 된다. 제네릭을 사용할 땐 `function` 키워드로 선언하는 경우가 많다고 한다.

```ts
// 에러 발생: JSX element ‘T’ has no corresponding closing tag
const arrowExampleFunc = <T>(arg: T): T[] => {
  return new Array(3).fill(arg);
};

// 특정 하위 타입만 올 수 있음을 명시해 에러 회피
const arrowExampleFunc2 = <T extends {}>(arg: T): T[] => {
  return new Array(3).fill(arg);
};
```

&nbsp;

## 3.3 제네릭 사용법

### 1. 함수에서

어떤 함수의 매개변수나 반환 값에 다양한 타입을 넣고 싶을 때 유용하다.

```ts
// ReadOnlyRepository
interface ReadOnlyRepository<T> {
    // 읽기 전용 메서드들
    getById(id: string): Promise<T>;
    getAll(): Promise<T[]>;
    find(predicate: (item: T) => boolean): Promise<T[]>;
    // create, update, delete 메서드는 없음
}

/// User 타입의 ReadOnlyRepository
class UserReadOnlyRepository implements ReadOnlyRepository<User> {
    async getById(id: string): Promise<User> {
        // 읽기 전용 데이터 접근
        return await db.users.findById(id);
    }

    async getAll(): Promise<User[]> {
        return await db.users.findAll();
    }
}
```

> ReadOnlyRepository는 데이터베이스나 데이터 저장소에 대한 읽기 전용 접근을 제공하는 디자인 패턴입니다. 주로 CQRS(Command Query Responsibility Segregation) 패턴에서 사용됩니다. 조회를 제외한 나머지 접근을 허용하지 않습니다.

&nbsp;

### 2. 호출 시그니처에서

호출 시그니처(타입 시그니처)는 TS에서 함수의 매개변수와 반환 타입을 미리 선언하는 것을 말한다. 제네릭 타입을 어디에 위치시키냐에 따라 타입의 범위와 제네릭 타입을 언제 구체 타입으로 한정할지 결정할 수 있다.

> 💡 구체 타입(concrete type)은 꺾쇠괄호의 타입을 명시적으로 선언해주는 명확한 타입을 말한다.
>
> ```ts
> type StringBox = Box<string>;  // string이라는 구체 타입으로 지정
> ```

아래는 배민선물하기팀의 호출 시그니처에서 제네릭을 활용한 예시다.

```ts
interface useSelectPaginationProps<T> {
    categoryAtom: RecoilState<number>;
    filterAtom: RecoilState<string[]>;
    sortAtom: RecoilState<SortType>;
    fetcherFunc: (props: CommonListRequest) = > Promise<DefaultResponse<ContentListRes
    ponse<T>>>;
  }
```

해당 예시에서는 제네릭을 호출 시그니처의 괄호 앞에 선언했기 때문에 해당 타입의 함수를 실제 호출할 때 제네릭 타입을 구체 타입으로 한정한다. (호출 시그니처의 괄호 앞애 선언해주면 해당 hook을 사용할 때마다 다른 타입을 지정할 수 있다.)

```ts
export type UseRequesterHookType = <RequestData = void, ResponseData = void>(
  baseURL?: string | Headers,
  defaultHeader?: Headers
) => [RequestStatus, Requester<RequestData, ResponseData>];

const [status, requester] = useRequester<UserData, UserResponse>();
const [status2, requester2] = useRequester<OrderData, OrderResponse>();
```

만약 타입의 매개변수로 제네릭 매개변수를 받으면 훅을 사용할 때마다 다른 타입을 지정해줄 수 없고, 하나의 타입으로 한정된다.

```ts
// 타입명 뒤에 제네릭을 선언한 경우
type UseRequesterHookType<RequestData = void, ResponseData = void> = (
  baseURL?: string | Headers,
  defaultHeader?: Headers
) => [RequestStatus, Requester<RequestData, ResponseData>];

// 타입을 미리 지정해야 함
const useUserRequester: UseRequesterHookType<UserData, UserResponse> = ...
```

&nbsp;

좀 더 쉬운 예시는 아래를 보자.

```ts
// 후자의 예시
interface GenericIdentityFn<Type> {
    (arg: Type): Type;
}

let myIdentity: GenericIdentityFn<number> = identity;
// 한 번 선언되면 함수 타입이 number로 고정됨

// 전자의 예시
interface GenericIdentityFn {
    <Type>(arg: Type): Type;
}

let myIdentity: GenericIdentityFn = identity;
// 함수를 호출할 때마다 다른 타입 사용 가능

// 첫 번째 방식
let myIdentity1: GenericIdentityFn<number> = identity;
myIdentity1(42);     // OK
myIdentity1("text"); // Error: 항상 number만 받을 수 있음

// 두 번째 방식
let myIdentity2: GenericIdentityFn = identity;
myIdentity2(42);     // OK: number 타입으로 사용
myIdentity2("text"); // OK: string 타입으로 사용
```

&nbsp;

### 3. 제네릭 클래스

제네릭 클래스는 외부에서 받아온 타입을 클래스 내부에 적용할 수 있는 클래스다. 클래스 이름 뒤에 타입 매개변수인 `<T>`를 선언해준다. `<T>`는 메서드의 매개변수나 반환 타입으로 사용될 수 있다.

```ts
class LocalDB<T> {
    // ...
    async put(table: string, row: T): Promise<T> {
      return new Promise<T>((resolved, rejected) = > { /* T 타입의 데이터를 DB에 저장 */ });
    }
  
    async get(table:string, key: any): Promise<T> {
      return new Promise<T>((resolved, rejected) = > { /* T 타입의 데이터를 DB에서 가져옴 */ });
    }
  
    async getTable(table: string): Promise<T[]> {
      return new Promise<T[]>((resolved, rejected) = > { /* T[] 타입의 데이터를 DB에서 가져옴*/ });
    }
  }
  
  export default class IndexedDB implements ICacheStore {
    private _DB?: LocalDB<{ key: string; value: Promise<Record<string, unknown>>;
    cacheTTL: number }>;
  
    private DB() {
      if (!this._DB) {
        this._DB = new LocalDB(“localCache”, { ver: 6, tables: [{ name: TABLE_NAME, keyPath: “key” }] });
      }
      return this._DB;
    }
    // ...
  }
```

특정 메서드만을 대상으로 제네릭을 적용하려면 해당 메서드만 제네릭 메서드로 선언하면 된다.

&nbsp;

### 4. 제한된 제네릭

TS에서의 제한된 제네릭은 타입 매개변수에 대한 제약 조건을 설정하는 기능이다.

제네릭을 `string` 타입으로 제약하고 싶으면 `string` 타입을 `extends`하는 방식으로 제약할 수 있다.

```ts
type ErrorRecord<Key extends string> = Exclude<Key, ErrorCodeType> extends never
  ? Partial<Record<Key, boolean>>
  : never;
```

이 때 `string`을 바운드 타입 매개변수(bounded type parameters)라고 하고 `string`을 키의 상한 한계(upper bound)라고 한다.

기본 타입뿐만 아니라 인터페이스나 클래스를 상속 받을 수도 있고, 유니온 타입을 상속할 수 있다.

```ts
function useSelectPagination<
  T extends CardListContent | CommonProductResponse
>({
  filterAtom,
  sortAtom,
  fetcherFunc,
}: useSelectPaginationProps<T>): {
  intersectionRef: RefObject<HTMLDivElement>;
  data: T[];
  categoryId: number;
  isLoading: boolean;
  isEmpty: boolean;
} {
  // ...
}
```

&nbsp;

### 5. 확장된 제네릭

제네릭은 여러 타입을 상속 받을 수 있고, 타입 매개변수를 여러 개 설정할 수도 있다.

```ts
export class APIResponse<Ok, Err = string> {
  // Ok: 성공 시 데이터 타입
  // Err: 실패 시 데이터 타입 (기본값은 string)
  private readonly data: Ok | Err | null;
}

// 사용하는 쪽 코드
const fetchShopStatus = async (): Promise<
  APIResponse<IShopResponse | null>
> => {
  // ...

  return (await API.get<IShopResponse | null>("/v1/main/shop", config)).map(
    (it) => it.result
  );
};
```

&nbsp;

### 6. 제네릭 사용 예시

실제 현업에서 제네릭이 가장 많이 활용되는 예시는 API 응답 값의 타입을 지정할 때다.

```ts
export interface MobileApiResponse<Data> {
  data: Data;
  statusCode: string;
  statusMessage?: string;
}
```

API의 응답 형식을 설정할 때 어떤 타입의 데이터가 들어오든 데이터와 응답 코드, 응답 메시지를 받아올 수 있다.

&nbsp;

제네릭을 사용할 때 주의할 점은, 불필요하게 제네릭을 사용하면 코드의 복잡도가 증가한다는 것이다. 세 가지 예시를 보자.

1. **굳이 사용하지 않아도 되는 타입 사용**

```ts
type GType<T> = T;
type RequirementType = "USE" | "UN_USE" | "NON_SELECT";
interface Order {
  getRequirement(): GType<RequirementType>;
}
```

`GType`이 `getRequirement` 함수에서만 사용된다고 가정하면 `GType` 없이 `RequirementType`으로만으로도 충분하다.

&nbsp;

2. `any` 타입을 사용

```ts
type whyDoYouUseSuchType<T = any> = { ... }
```

사실상 `any` 타입으로 선언한 것과 다름없다.

&nbsp;

3. 가독성을 고려하지 않은 사용

```ts
ReturnType<
  Record<
    OrderType,
    Partial<
      Record<
        CommonOrderStatus | CommonReturnStatus,
        Partial<Record<OrderRoleType, string[]>>
      >
    >
  >
>;

type CommonStatus = CommonOrderStatus | CommonReturnStatus;

type PartialOrderRole = Partial<Record<OrderRoleType, string[]>>;

type RecordCommonOrder = Record<CommonStatus, PartialOrderRole>;

type RecordOrder = Record<OrderType, Partial<RecordCommonOrder>>;

ReturnType<RecordOrder>;
```

이렇게 제네릭을 남용하면 해당 코드가 어떤 동작을 하는지 한 눈에 알아보기 어렵다.
