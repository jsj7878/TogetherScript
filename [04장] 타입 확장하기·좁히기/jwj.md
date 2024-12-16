# 4. 타입 확장하기 · 좁히기

## 4.1 타입 확장하기

타입 확장은 기존 타입으로 새로운 타입을 정의하는 것을 말한다.

TS에서는 `interface`나 `type`을 통해 타입을 정의하고 `extends`나 교차 타입, 유니온 타입을 사용해서 타입을 확장한다.

&nbsp;

### 1. 타입 확장의 장점

가장 큰 장점은 코드의 중북을 줄여준다는 것이다. 기존의 속성의 변경 사항이 있더라도 확장한 타입의 코드는 수정하지 않아도 되기 때문에 유지보수에도 좋다. 기존의 속성에서 속성 하나만 추가해서 타입을 선언하려면 이렇게 하면 된다.

```ts
// interface를 사용한 확장
interface Person {
  name: string;
  age: number;
}

interface Developer extends Person {
  whichProgrammingLanguage: string;
}

// type을 사용한 확장
type Person = {
  name: string;
  age: number;
};

type Developer = {
  whichProgrammingLanguage: string;
} & Person;
```

&nbsp;

### 2. 유니온 타입

유니온 타입은 두 개 이상의 타입을 조합하여 사용하는 방법이다. 집합의 관점에서 합집합으로 해석할 수 있다.

⚠️ 하지만, 앞에서 공부했듯 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 직접 접근할 수 있다.

`A | B` 타입은 A 타입 또는 B 타입이라는 의미지 A 타입이면서 B 타입이라는 의미는 아니다.

> 💡 TS의 타입을 속성의 집합이 아닌 값의 집합이라고 생각해야 유니온 타입이 합집합이라는 개념을 이해할 수 있다.

&nbsp;

### 3. 교차 타입

교차 타입도 기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입이지만, A 타입과 B 타입에 있는 모든 속성을 가진 단일 타입이다. 때문에 A 타입 혹은 B 타입에만 있는 속성을 접근할 수 있다.

교차 타입은 교집합의 개념과 비슷하다. 헷갈리지만 타입을 속성이 아닌 값의 집합으로 해석한다면 새 타입이 두 타입의 속성을 모두 만족하는 값이라고 인식한다면 어느 정도 이해가 된다.

```ts
/* 배달 팁 */
interface DeliveryTip {
  tip: string;
  }
/* 별점 */
interface StarRating {
  rate: number;
}
/* 주문 필터 */
type Filter = DeliveryTip & StarRating;

const filter: Filter = {
  tip: “1000원 이하”,
  rate: 4,
};
```

&nbsp;

⚠️ 교차 타입을 사용할 때 타입이 서로 호환되지 않는 경우도 있다.

```ts
type IdType = string | number;
type Numeric = number | boolean;

type Universal = IdType & Numeric;
```

`Universal` 타입은 두 타입의 교차 타입이므로 두 타입을 모두 만족하는 경우에만 유지된다. 따라서 `Universal`은 `number` 타입이 된다.

&nbsp;

### 4. extends와 교차 타입

교차 타입은 `interface`와 `extends`를 사용해서도 선언할 수 있다.

⚠️ 이 방법은 `type`과 `&` 연산자를 사용해서 만드는 교차 타입과는 100% 일치하지 않는다.

아래 예시를 보자.

```ts
interface DeliveryTip {
  tip: number;
}

interface Filter extends DeliveryTip {
  tip: string;
  // Interface ‘Filter’ incorrectly extends interface ‘DeliveryTip’
  // Types of property ‘tip’ are incompatible
  // Type ‘string’ is not assignable to type ‘number’
}
```

`tip` 속성의 타입이 호환되지 않는다는 에러가 발생한다.

```ts
type DeliveryTip = {
  tip: number;
};

type Filter = DeliveryTip & {
  tip: string;
};
```

에러가 발생하지 않지만 이때 속성의 타입은 `never`가 된다! `type` 키워드는 교차 타입으로 선언되면 새롭게 추가되는 속성에 대해 미리 알 수 없기 때문이다. `tip`이라는 같은 속성에 대해 서로 호환되지 않는 타입이 선언되어 `never` 타입이 되어 버린 것.

&nbsp;

### 5. 타입 확장 적용하기

```ts
interface Menu {
  name: string;
  image: string;
}
```

`Menu`라는 인터페이스가 있다. 이 인터페이스에 다음과 같은 요구 사항이 추가되었다고 가정해보자.

1. 특정 메뉴에 gif를 추가하고 싶다.
2. 특정 메뉴는 이미지 대신 텍스트만 노출되게 하고 싶다.

&nbsp;

방법은 두 가지가 있다.

1. 기존의 인터페이스에 옵셔널 속성을 추가한다.

```ts
interface Menu {
  name: string;
  image: string;
  gif?: string; // 요구 사항 1. 특정 메뉴를 길게 누르면 gif 파일이 재생되어야 한다
  text?: string; // 요구 사항 2. 특정 메뉴는 이미지 대신 별도의 텍스트만 노출되어야 한다
}
```

2. 타입을 확장해 새로운 타입을 만든다.

```ts
interface SpecialMenu extends Menu {
  gif: string; // 요구 사항 1. 특정 메뉴를 길게 누르면 gif 파일이 재생되어야 한다
}

interface PackageMenu extends Menu {
  text: string; // 요구 사항 2. 특정 메뉴는 이미지 대신 별도의 텍스트만 노출되어야 한다
}
```

&nbsp;

다음과 같은 API 응답이 왔다.

```TS
const menuList = [
  { name: “찜”, image: “찜.png” },
  { name: “찌개”, image: “찌개.png” },
  { name: “회”, image: “회.png” },
];

const specialMenuList = [
  { name: “돈까스”, image: “돈까스.png”, gif: “돈까스.gif” },
  { name: “피자”, image: “피자.png”, gif: “피자.gif” },
];

const packageMenuList = [
  { name: “1인분”, image: “1인분.png”, text: “1인 가구 맞춤형” },
  { name: “족발”, image: “족발.png”, text: “오늘은 족발로 결정” },
];
```

**1번**의 경우, `specialMenuList`는 `Menu` 타입의 원소를 가지므로 `text` 속성이 존재하므로 접근할 수는 있으나, `specialMenuList`에는 `text` 속성을 가진 원소가 없다. 때문에 실행 시 오류를 발생시킨다.

```ts
menuList: Menu[] // OK
specialMenuList: Menu[] // OK
packageMenuList: Menu[] // OK

specialMenuList.map((menu) => menu.text); // TypeError: Cannot read properties of undefined
```

&nbsp;

**2번**의 경우, 각 배열의 타입을 확장할 타입에 맞게 명확히 규정할 수 있다.

```ts
menuList: Menu[] // OK

specialMenuList: Menu[] // NOT OK
specialMenuList: SpecialMenu[] // OK

packageMenuList: Menu[] // NOT OK
packageMenuList: PackageMenu[] // OK

specialMenuList.map((menu) => menu.text); // Property ‘text’ does not exist on type ‘SpecialMenu’
```

&nbsp;

❗따라서 주어진 타입에 무분별하게 속성을 추가해 사용하는 것보다 **타입을 확장하는 편이 좋다.** 네이밍을 통해 타입의 의도를 명확히 할 수 있고, 코드 작성 단계에서 예기치 못한 버그도 예방할 수 있기 때문.

&nbsp;

## 4.2 타입 좁히기 - 타입 가드

TS의 타입 좁히기는 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 것을 말한다.

타입 좁히기를 통해 **더 정확하고 명시적인 타입 추론이 가능**하고, 복잡한 타입을 작은 범위로 축소하여 **타입 안정성**을 높일 수 있다.

&nbsp;

### 1. 타입 가드에 따라 분기 처리하기

TS의 분기 처리는 조건문과 타입 가드를 활용하여 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따른 동작을 수행하는 것을 말한다.

타입 가드는 런타임에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 것을 말한다.

특정 타입을 조건으로 분기 처리를 하고 싶다고 가정해보자. TS 코드가 JS 코드로 변환되면 타입에 대한 정보는 모두 제거되어 런타임에 존재하지 않기 때문에 if문과 타입만으로는 불가능하다.

때문에 타입 가드는 **JS에 존재하는 연산자를 이용한 타입 가드**, **사용자 정의 타입 가드를 이용한 타입 가드**로 구분할 수 있다.

&nbsp;

### 2. 원시 타입을 추론할 때: `typeof` 연산자 활용하기

`typeof` 연산자로 원시 타입을 추론할 수 있다.

하지만 `typeof`는 JS 타입 시스템에만 대용할 수 있어 `null`이나 배열 타입 등이 `object`로 판별되는 등 복잡한 타입을 검증하기는 힘들다.

❗때문에 주로 원시 타입을 좁힐 때만 사용한다.

```ts
const replaceHyphen: (date: string | Date) => string | Date = (date) => {
  if (typeof date === “string”) {
  // 이 분기에서는 date의 타입이 string으로 추론된다
  return date.replace(/-/g, “/”);
  }

  return date;
};
```

&nbsp;

### 3. 인스턴스화된 객체 타입을 판별할 때: `instanceof` 연산자 활용하기

`typeof`와 다르게 `instanceof`를 사용함으로써 인스턴스화된 객체의 타입을 판별하는 타입 가드로 사용한다.

`A instanceof B` 형태로 사용하게 되면 A는 타입을 검사할 변수, B에는 특정 객체의 생성자가 들어간다. A의 프로토타입 체인에 B가 존재하는지 검사하는 방식이다.

다음 코드에서는 `selected`가 `Date`의 인스턴스인지 검사한 뒤 `Range` 타입의 객체를 반환하도록 분기 처리해줬다.

```ts
interface Range {
  start: Date;
  end: Date;
}

interface DatePickerProps {
  selectedDates?: Date | Range;
}

const DatePicker = ({ selectedDates }: DatePickerProps) => {
  const [selected, setSelected] = useState(convertToRange(selectedDates));
  // ...
};

export function convertToRange(selected?: Date | Range): Range | undefined {
  return selected instanceof Date
    ? { start: selected, end: selected }
    : selected;
}
```

```ts
const onKeyDown = (event: React.KeyboardEvent) => {
  if (event.target instanceof HTMLInputElement && event.key === “Enter”) {
  // 이 분기에서는 event.target의 타입이 HTMLInputElement이며
  // event.key가 ‘Enter’이다
  event.target.blur();
  onCTAButtonClick(event);
  }
};
```

&nbsp;

### 4. 객체 내 속성 존재 여부에 따른 구분: `in` 연산자 활용하기

`in` 연산자는 통해 객체에 해당 속성이 있는지 확인 후 `true` 혹은 `false`를 반환한다.

`in`은 프로토타입 체인으로 접근할 수 있는 속성이면 `true`를 반환한다. 속성이 `undefined`라고 해서 `false`를 반환하지는 않는다. `delete` 연산자를 사용하여 객체 내부에서 해당 속성을 제거해야만 `false`를 반환한다.

```ts
interface BasicNoticeDialogProps {
  noticeTitle: string;
  noticeBody: string;
}

interface NoticeDialogWithCookieProps extends BasicNoticeDialogProps {
  cookieKey: string;
  noForADay?: boolean;
  neverAgain?: boolean;
}

export type NoticeDialogProps =
  | BasicNoticeDialogProps
  | NoticeDialogWithCookieProps;
```

`NoticeDialogProps`가 어떤 타입인지 알기 위해 객체 내부에 `cookieKey` 속성이 존재 여부를 따져볼 수 있다.

```ts
const NoticeDialog: React.FC<NoticeDialogProps> = (props) => {
  if (“cookieKey” in props) return <NoticeDialogWithCookie {...props} />;
  return <NoticeDialogBase {...props} />;
};
```

JS에서는 `in` 연산자의 결과를 런타임에 검사하지만, TS에서는 객체 타입에 속성이 존재하는지를 검사한다는 차이점이 있다.

&nbsp;

### 5. `is` 연산자로 사용자 정의 타입 가드 만들어 활용하기

타입 가드를 실현하는 함수를 직접 만들 수 있다.

반환 타입이 타입 명제(type predicates)인 함수를 정의하여 사용할 수 있다. `A is B` 형식으로 작성하고, A는 매개변수 이름, B는 타입이다. 반환 타입을 타입 명제로 지정하게 되면 반환 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급한다.

> 💡 타입 명제란? 함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수이다.

```ts
const isDestinationCode = (x: string): x is DestinationCode =>
  destinationCodeList.includes(x);
```

이 함수는 `string` 타입의 `x`가 `destinationCodeList` 배열의 원소 중에 하나인지 검사하여 참/거짓 값을 반환한다. 함수의 반환 값을 `boolean` 타입이 아닌 `x is DestinationCode` 형태로 선언하여 TS에게 이 함수가 사용되는 곳의 타입을 추론할 때 해당 조건을 타입 가드로 사용하도록 알려주는 것이다.

```ts
const getAvailableDestinationNameList = async (): Promise<DestinationName[]> => {
  const data = await AxiosRequest<string[]>(“get”, “.../destinations”);
  const destinationNames: DestinationName[] = [];
  data?.forEach((str) => {
  if (isDestinationCode(str)) {
    destinationNames.push(DestinationNameSet[str]);
    /*
    isDestinationCode의 반환 값에 is를 사용하지 않고 boolean이라고 한다면 다음 에러가
    발생한다
    - Element implicitly has an ‘any’ type because expression of type ‘string’
    can’t be used to index type ‘Record<”MESSAGE_PLATFORM” | “COUPON_PLATFORM” | “BRAZE”,
    “통합메시지플랫폼” | “쿠폰대장간” | “braze”>’
    */
    }
  });
  return destinationNames;
};
```

만약 `boolean` 타입으로 선언했다면 TS는 함수 내부의 `includes` 함수를 해석해 타입 추론을 할 수 없다. (왜?) if문 스코프의 `str` 타입을 좁히지 못하고 `string`으로만 추론한다. `destinationNames`의 타입은 `Destination[]`이기 때문에 `string` 타입의 `str`을 `push`할 수 없다는 에러가 발생하게 된다.

이렇게 TS에게 반환 값에 대한 타입 정보를 알려주고 싶을 때 `is`를 사용할 수 있다. 반환 값의 타입을 `x is DestinationCode`로 알려줌으로써 TS는 if문 스코프의 `str` 타입을 `DestinationCode`로 추론할 수 있다.

&nbsp;

요약하자면, 타입 가드에는 4가지 방법이 있다.
1. 원시 타입을 추론할 때는 `typeof`를 사용한다.
2. 인스턴스화된 객체의 타입을 추론할 때는 `instanceof`를 사용한다.
3. 객체 내부의 구조가 다른(서로 다른 속성을 가지고 있는) 객체 타입을 추론할 때는 `in`을 사용한다.
4. 타입 명제(`A is B`)를 이용한 사용자 정의 타입 가드 함수를 사용한다.
   
&nbsp;

## 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

Exhaustiveness는 철저함, 완전함을 의미한다. Exhaustiveness Checking이란 **모든 분기 케이스에 대해 철저하게 타입을 검사**하는 것을 말하며 타입 좁히기에 사용되는 패러다임이다.

필요한 부분만 분기 처리를 할 수도 있지만, 떄로는 모든 케이스에 대해 분기 처리를 해야 유지보수 측면에서 안전하다고 생각되는 경우가 있을 수 있다.

```ts
type ProductPrice = “10000” | “20000” | “5000”;

const getProductName = (productPrice: ProductPrice): string => {
  if (productPrice === “10000”) return “배민상품권 1만 원”;
  if (productPrice === “20000”) return “배민상품권 2만 원”;
  // if (productPrice === “5000”) return “배민상품권 5천 원”;
  else {
    exhaustiveCheck(productPrice); // Error: Argument of type '"5000"' is not assignable to parameter of type 'never'
    return “배민상품권”;
  }
};

const exhaustiveCheck = (param: never) => {
  throw new Error(“type error!”);
};
```
> 책에는 `Argument of type ‘string’ is not assignable to parameter of type ‘never’` 에러로 표기해놓았음. 5.6.3 버전까지도 같은 에러를 발생시켰으나 5.7.2 버전에서는 변경되었음. 정확히 어떤 버전에서 변경되었는지 살펴보아야 할 듯.

`ProductPrice` 타입이 업데이트되면 해당 타입을 사용하는 `getProdctName`도 함께 업데이트해주어야 한다. 그러나 함수를 수정해주지 않아도 딱히 에러가 발생하는 것이 아니기 때문에 이를 강제하기 위해 `exhaustiveCheck` 함수를 사용하고 있다.

이 함수의 매개변수는 `never` 타입이다. `never` 타입에는 어떤 값도 들어 올 수 없으며 값이 들어오면 에러를 뱉기 때문에 분기 처리를 해주지 않은 케이스가 존재할 경우 에러를 발생시킬 수 있다.