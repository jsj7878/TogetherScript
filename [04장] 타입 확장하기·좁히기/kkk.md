```jsx
4.1 타입 확장하기
4.2 타입 좁히기 - 타입 가드
4.3 타입 좁히기 - 식별할 수 있는 유니온
4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기
```

# 4.1 타입 확장하기

기존 타입으로 새로운 타입 정의하기.

- 타입스크립트에서는 interface, type 키워드로 타입 정의
- extends, 교차 타입, 유니온 타입으로 타입을 확장

# 4.1.1 타입 확장의 장점

타입 확장의 장점은 코드 중복을 줄인다.

```tsx
/**
 * 메뉴 요소 타입
 * 메뉴 이름, 이미지 URL, 할인율, 재고 정보를 담고 있다.
 */
interface BaseMenuItem {
  itemName: string | null;          // 메뉴 이름
  itemImageUrl: string | null;     // 이미지 URL
  itemDiscountAmount: number;      // 할인율
  stock: number | null;            // 재고 정보
}

/**
 * 장바구니 요소 타입
 * 메뉴 타입에 수량 정보가 "추가!!!"되었다.
 */
interface BaseCartItem extends BaseMenuItem {
  quantity: number;                // 수량
}
```

인터페이스  말고 type을 사용하면 아래와 같다.

```tsx
type BaseMenuItem = {
  itemName: string | null;     
  itemImageUrl: string | null;     
  itemDiscountAmount: number;      
  stock: number | null;            
};

type BaseCartItem = {
  quantity: number;
} & BaseMenuItem;
```

> 인터페이스의 타입확장은 extends 키워드를 사용하고 type의 타입확장은 &를 사용한다.
> 

타입확장을 활용하면 장바구니와 관련된 요구사항이 생길 때마다 효율적으로 수정이 가능하다.

# 4.1.2 유니온 타입

유니온 타입은 2개 이상의 타입을 조합해 사용하는 방법이다. 일종의 합집합이다.

```tsx
type MyUnion = A | B;
```

❗유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 “공통으로 갖고 있는 속성”에만 접근할 수 있다.

```tsx
interface CookingStep { // 요리 단계
  orderId: string;
  price: number;  
}

interface DeliveryStep { // 배달 단계
  orderId: string; 
  time: number;   
  distance: string; 
}

function getDeliveryDistance(step: CookingStep | DeliveryStep) {
   return step.distance;
   // Property 'distance' does not exist on type 'CookingStep | DeliveryStep'
   // Property 'distance' does not exist on type 'CookingStep'
}
```

distance는 배달단계 인터페이스에만 존재하기 때문에 에러가 발생한다. 

```tsx
// 해결 : 타입가드 'in' 사용
function getDeliveryDistance(step: CookingStep | DeliveryStep): string | undefined {
  if ('distance' in step) { // step이 DeliveryStep 타입인 경우 distance 반환
    return step.distance;
  } // step이 CookingStep 타입인 경우 undefined 반환
  return undefined;
}
```

# 4.1.3 교차 타입

기존 타입을 합쳐 필요한 기능들을 가진 하나의 타입을 만든다. `&` 키워드를 사용한다. 일종의 교집합이다.

```tsx
type MyIntersection = A & B;
```

```tsx
interface CookingStep { // 요리 단계
  orderId: string;
  price: number;  
}

interface DeliveryStep { // 배달 단계
  orderId: string; 
  time: number;   
  distance: string; 
}
// 교차 타입
type BaedalProgress = CookingStep & DeliveryStep;
```

❗유니온 타입과 다른 점은 두 개의 타입을 합쳐 모든 속성을 가진 “**단일 타입**”이 된다.

```tsx
function logBaedalInfo(progress: BaedalProgress): void {
  console.log(`주문 금액: ${progress.price}`);
  console.log(`배달 거리: ${progress.distance}`);
}
```

> 타입스크립트의 타입을 속성의 집합이 아니라 값의 집합으로 이해해야 한다. BaedalProgress 교차타입은 위의 두 개의 타입의 속성을 모두 만족하는 타입=집합이라고 해석할 수 있다.
> 

## ▶️

```tsx
interface DeliveryTip {
  tip: string; 
}

interface StarRating {
  rate: number; 
}

type Filter = DeliveryTip & StarRating; // 공통된 속성이 없다.

const filter: Filter = {
  tip: "1000원 이하", 
  rate: 4,  
};
 
```

❗공통된 속성이 존재하지 않아도 Filter 타입은 공집합=never 타입이 아닌 두 개의 타입을 모두 포함한 타입이 된다. 이는 타입이 속성이 아닌 값의 집합으로 해석되기 때문이다.

## ▶️ 교차타입을 사용할 때 타입이 서로 호환되지 않는 경우

유니온 타입을 서로 교차타입으로 사용할 때?

```tsx
type IdType = string | number;
type Numeric = number | boolean;
type Universal = IdType & Numeric;

const value1: Universal = 42;       // ✅ number 타입 허용
const value2: Universal = "hello"; // ❌ 오류: string은 Numeric에 포함되지 않음
const value3: Universal = true;    // ❌ 오류: boolean은 IdType에 포함되지 않음
```

경우의 수 4가지가 나오는데

1. string + number
2. string + boolean
3. number + number
4. number + boolean

두 타입을 모두 만족하는 경우에만 교차 타입이 유지된다. 따라서 3번 number 타입이 된다.

- 유니온 타입은 "여러 타입 중 하나"를 허용.
- 교차 타입은 "두 타입을 모두 만족"하는 값을 허용.

# 4.1.4 extends와 교차타입

extends 키워드로 교차타입 작성하기

```tsx
interface BaseMenuItem {
  itemName: string | null;        
  itemImageUrl: string | null;   
  itemDiscountAmount: number;      
  stock: number | null;           
}

interface BaseCartItem extends BaseMenuItem {
  quantity: number;            
}
```

BaseCartItem은 BaseMenuItem의 속성을 모두 포함하는 상위 집합이 된다. (자식)

이를 교차타입의 관점에서 작성하면 아래와 같다.

```tsx

type BaseMenuItem = {
  itemName: string | null;      
  itemImageUrl: string | null;  
  itemDiscountAmount: number;  
  stock: number | null;        
};

type BaseCartItem = {
  quantity: number;  
} & BaseMenuItem;

/* 장바구니 아이템 예제 */
const cartItem: BaseCartItem = {
  itemName: "지은이네 떡볶이",
  itemImageUrl: "https://www.woowahan.com/images/jieun-tteokbokki.png",
  itemDiscountAmount: 20,
  stock: 100,
  quantity: 2,
};
```

❗교차 타입을 사용한 코드는 인터페이스가 아닌 type로 선언해야 한다. 유니온 타입과 교차 타입을 사용한 새로운 타입을 type 키워드로만 선언할 수 있다.

## ❕extends 키워드를 사용한 타입이 교차 타입과 완전히 상응하지 않는다.

```tsx
/* 배달 팁 타입 */
interface DeliveryTip {
  tip: number; 
}

interface Filter extends DeliveryTip {
  tip: string; // [X] 'string' 타입은 'DeliveryTip.tip'의 'number' 타입과 호환되지 않음
}
```

이를 type으로 작성하면 에러가 발생하지 않는다.

```tsx
type DeliveryTip = {
	tip : number;
}

type Filter = DeliveryTip & {
	tip : string; // [O] tip 속성의 타입은 never
};

const invalidFilter: Filter = {
  tip: "5000원", // 오류: `tip`은 `never` 타입이므로 어떤 값도 허용되지 않음
};
```

> type 키워드는 교차 타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없기 때문에 선언시 에러가 발생하지 않는다. 하지만 tip이라는 같은 속성에 대해 서로 호환되지 않는 타입이 서넝ㄴ되어 결국 never 타입이 된 것이다.
> 

```tsx
교차타입
- 다른 속성 : 병합됨
- 공통 속성 : 타입이 호환되지 않을 경우 인터페이스 사용시 선언에서 에러 발생,
						타입 사용시 선언은 에러 발생하지 않으나 실제 타입에 never 할당됨

```

# 4.1.5 배달의 민족 메뉴 시스템에 타입 확장 적용하기

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/3529de1c-a19d-49ac-aa2d-b4bf2e3fafce/9474487f-ca7c-4d45-8c61-e93ef3257722/image.png)

이를 메뉴 인터페이스로 나타내자

```tsx
interface Menu {
  name: string; // 메뉴 이름
  image: string; // 메뉴 이미지 URL
}
```

```tsx
/* 메인 메뉴 컴포넌트 */
function MainMenu() {
  const menuList: Menu[] = [
  // Menu 타입을 원소로 갖는 배열
    { name: "1인분", image: "1인분.png" },
    { name: "2인분", image: "2인분.png" },
    { name: "3인분", image: "3인분.png" },
  ];

  return (
    <ul>
      {menuList.map((menu, index) => (
        <li key={index}>
          <img src={menu.image} alt={menu.name} />
          <span>{menu.name}</span>
        </li>
      ))}
    </ul>
  );
}
```

요구사항이 추가되었다.

1. 특정 메뉴를 길게 누르면 gif 파일이 재생되어야 한다.
2. 특정 메뉴는 이미지 대신 별도의 텍스트만 노출되어야 한다.

**❕ 요구사항을 만족하는 타입의 작성방법 2가지**

```tsx
1. 타입 내에서 속성 추가 : 기존 메뉴 인퍼페이스에 추가된 정보 추가
2. 타입 확장 활용 : 기존 메뉴 인터페이스는 유지한 채 요구사항에 따른 별도 타입 만들기
```

## 🎧 방법1 : 타입 내에 속성 추가

```tsx
interface Menu {
  name: string;              // 메뉴 이름
  image: string;             // 메뉴 이미지
  gif?: string;              // gif 파일 (선택적)
  text?: string;             // 텍스트 (선택적)
}
```

## 🎧 방법2 : 타입 확장 활용

```tsx
interface Menu {
  name: string;      // 메뉴 이름
  image: string;     // 메뉴 이미지
}

/**
 * GIF를 활용한 메뉴 타입 : 반드시 gif 속성을 포함
 */
interface SpecialMenu extends Menu {
  gif: string;       // GIF 파일 경로
}

/**
 * 텍스트를 활용한 메뉴 타입 : 반드시 text 속성을 포함
 */
interface PackageMenu extends Menu {
  text: string;      // 텍스트 정보
}

// 메뉴 데이터
const menuList: (Menu | SpecialMenu | PackageMenu)[] = [
  { name: "떡볶이", image: "tteokbokki.png", gif: "tteokbokki.gif" }, // SpecialMenu
  { name: "순대", image: "soondae.png" },                            // Menu
  { name: "김밥", image: "", text: "김밥만 판매 중입니다." },         // PackageMenu
];

// 데이터 사용
menuList.forEach((menu) => {
  if ("gif" in menu) {
    console.log(`GIF 파일 재생: ${menu.gif}`);
  } else if ("text" in menu) {
    console.log(`텍스트만 노출: ${menu.text}`);
  } else {
    console.log(`이미지 노출: ${menu.image}`);
  }
});

```

결론은 타입을 확장하는 방법이 좋다.

# 4.2 타입 좁히기 - 타입 가드

> 타입 좁히기는 변수 또는 표현식의 “타입 범위”를 “더 작은 범위로 좁혀나가는 과정”을 말한다. 타입 좁히기를 통해 더 정확하고 명시적인 타입추론을 할 수 있게 되고, 복잡한 타입을 작은 범위로 축소해 타입 안정성을 높일 수 있다.
> 

# 4.2.1 타입 가드에 따라 분기 처리하기

조건문과 타입 가드를 활용해 변수나 표현식의 타입 범위를 좁혀 다양한 상황에 따라 다른 동작을 수행하는 것이다. 

> 타입 가드는 런타임에 조건문을 사용해 타입을 검사하고 타입 범위를 좁혀주는 기능이다.
> 

<aside>
🏴󠁧󠁢󠁳󠁣󠁴󠁿

스코프

타입스크립트에서 스코프는 변수와 함수 등의 식별자가 유효한 범위를 나타낸다. 즉 변수와 함수를 선언하거나 사용할 수 있는 영역을 말한다.

</aside>

## ❓함수가 A | B 타입의 매개변수를 받을 때 타입에 따라 로직을 처리하려면 어떻게 해야 하는가?

**if문을 사용하는 방법**

- ***컴파일 시 타입 정보가 제거되어 런타임에 존재하지 않는다.*** 즉 타입을 사용해 조건을 만들 수 없다. 어떻게 해야 컴파일해도 타입 정보가 사라지지 않을까?
    - “런타임”에서도 “유효한 타입 정보”를 가지게 하려면 “타입 가드”를 사용한다.

## ▶️ 두 가지 종류의 타입 가드

```tsx
1. 자바스크립트 연산자를 사용한 타입 가드
2. 사용자 정의 타입 가드
```

1. 자바스크립트 연산자를 사용한 타입 가드
    
    `typeof, instanceof, in`과 같은 연산자를 사용하는 방법이다. 
    
2. 사용자 정의 타입 가드 : 사용자가 직접 어떤 타입으로 값을 좁힐지 직접 지정하는 방식이다.

# 4.2.2 원시 타입을 추론할 때 : typeof 연산자 활용하기

type of A === B 를 조건으로 분기 처리하며, 해당 분기 내에서는 A의 타입이 B로 추론된다. 

❕다만 typeof는 자바스크립트 타입 시스템만 대응할 수 있다. 자바스크립트의 동작 방식으로 인해 null과 배열 타입 등이 object 타입으로 판별되는 등 복잡한 타입을 검증하기에는 한계가 있다. 따라서 typeof 연산자는 주로 원시 타입을 좁히는 용도로만 사용할 것을 권장한다. 

- **typeof 연산자로 검사할 수 있는 타입 목록**

```tsx
string / number / boolean / undefined / object / function / bigint / symbol
```

```tsx
const replaceHyphen: (date: sting | Date) => string | Date = (date) => {
	if (typeof date === "string"){
	// 이 분기에서는 date의 타입이 string로 추론된다.
		return date.replace(/-/g, "/");
	}
	return date;
};
```

# 4.2.3 인스턴스화된 객체 타입을 판별할 때 : instanceof 연산자 활용하기

selected 매개변수가 Date인지 검사한 후 Range 타입의 객체를 리턴하도록 분기 처리한다.

```tsx
interface Range {
  start: Date;
  end: Date;
}

interface DatePickerProps {
  selectedDates?: Date | Range;
}

export function convertToRange(selected?: Date | Range): Range | undefined {
  if (!selected) return undefined;
  return selected instanceof Date
    ? { start: selected, end: selected }
    : selected;
}

const DatePicker = ({ selectedDates }: DatePickerProps) => {
  const [selected, setSelected] = useState<Range | undefined>(
    convertToRange(selectedDates)
  );

  return (
    <div>
      <p>Start Date: {selected?.start.toDateString()}</p>
      <p>End Date: {selected?.end.toDateString()}</p>
    </div>
  );
};

export default DatePicker;

```

instanceof 연산자는 인스턴스화된 객체 타입을 판별하는 타입 가드로 사용할 수 있다. 

- A instanceof B 형태, A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어간다.
- instanceof는 A의 프로토타입 체인에 생성자 B가 존재하는지를 검사해서 존재하면 true를 리턴한다.

❗즉 A의 프로토타입 속성 변화에 따라 instanceof 연산자의 결과가 달라질 수 있다.

```tsx
const onKeyDown = (event : React.KeyboardEvent) => {
	if (event.target instanceof HTMLInputElement && event.key === "Enter") {
		// 이 분기에서는 event.target 타입이 HTMLInputElement 이며
		// event.key가 'Enter'이다.
		event.target.blur();
		onCTAButtonClick(event);
	}
};
```

# 4.2.4 객체의 속성이 있는지 없는지에 따른 구분 : in 연산자 활용하기

> in 연산자는 객체에 속성이 있는지 확인한 다음 true 또는 false를 리턴한다. 속성이 있는지 여부에 따라 분기처리할 수 있다.
> 
- `A in B` : A라는 속성이 B 객체에 존재하는지를 검사
    - 만약 B 객체의 A 속성이 undefined 여도 true를 리턴 (존재 여부만 판단)

```tsx
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
  | BasicNoticeDialogProps // 1이라고 치자
  | NoticeDialogWithCookieProps; // 2라고 치자
```

❔NoticeDialogProps의 객체 타입이 1인지 2인지에 따라 렌터링하는 컴포넌트가 달라지도록 분기처리를 어떻게 할 수 있을까?

- cookieKey 속성 여부를 가지고 판단할 수 있다.

```tsx
const NoticeDialog : React.FC<NoticeDialogProps> = (props) => {
	if ("cookieKey" in props) return <NoticeDialogWithCookie {...props} />;
	return <NoticeDialogBase {...props} />;
};
```

자바스크립트의 in 연산자는 런타임의 값만 검사하지만 타입스크립트에서는 객체 타입에 속성이 존재하는지를 검사한다. 

*얼리 리턴 : 특정 조건에 부합하지 않으면 바로 리턴하는 것.

# 4.2.5 is 연산자로 사용자 정의 타입 가드 만들어 활용하기

직접 타입 가드 함수 만들기. 이러한 타입의 타입 가드는 리턴 타입이 “타입 명제=type predicaters”인 함수를 정의하여 사용할 수 있다. 

- 타입 명제는 A is B 형식으로 작성하면 되는데 A는 매개변수 이름이고 B는 타입이다.
- 리턴 타입을 “타입 명제”로 지정하게 되면 리턴 값이 참일 때 A 매개변수의 타입을 B 타입으로 취급하게 된다.

*타입 명제 : 함수의 리턴 타입에 대한 타입 가드를 수행하기 위해 사용되는 특별한 형태의 함수

```tsx
const isDestinationCode = (x:string): x is DestinationCode => 
	destinationCodeList.includes(x);
```

1. 함수의 리턴 값을 boolean이 아닌 `x is DestinattionCode` 로 해서
2. 타입스크립트에게 이 함수가 사용되는 곳의 타입을 추론할 때 해당 조건을 타입 가드로 사용하도록 알린다.

> 리턴 값이 참일 때 x의 타입을 DestinationCode로 취급한다.
> 

🔹 **리턴 값이 boolean과 is를 활용할 때의 차이**

```tsx
// isDestinationCode 함수 리턴 값 boolean으로 했을 때
function isDestinationCode(code: string): boolean {
  // 코드 검증 로직 추가
  destinationCodeList.includes(x);
}
```

```tsx
const getAvailableDestinationNameList = async (): Promise<DestinationName[]> => {
  const data = await AxiosRequest<string[]>("get", "../destinations");
  const destinationNames: DestinationName[] = [];

  data?.forEach((str) => {
    if (isDestinationCode(str)) {
      destinationNames.push(DestinationNamesSet[str]);
      /*
      isDestinationCode의 반환 값에 is 를 사용하지 않고 boolean 이라고 한다면 다음 에러가 발생
      
			Element implicitly has an 'any' type because expression of type 'string'
			can't be used to index type 
			'Record'MESSAGE _PLATFORM" | "COUPON PLATFORM" | "BRAZE",
			"통합메시지플랫폼" | " 쿠폰대장간 " | "braze">'
			*/
  });

  return destinationNames;
};

```

❕**isDestinationCode 함수의 리턴값이 is, 타입 명제를 사용하지 않고 boolean으로 했을 때 어떤 일이 발생하는가?**

- 타입스크립트는 어떻게 추론하는가?
    1. 타입스크립트는 includes 함수를 해석해 타입 추론을 할 수 없다.
    2. if문 스코프의 str 타입을 좁히지 못하고 string로 추론한다.
    3. destinationNames의 타입은 DestinationName[] 이기 때문에 string 타입의 str 을 push 할 수 없다는 에러가 발생한다

# 4.3 타입 좁히기 - 식별할 수 있는 유니온

태그된 유니온===식별할 수 있는 유니온은 타입 좁히기에 많이 사용되는 방식이다.

# 4.3.1 에러 정의하기

```tsx
type TextError = {
  errorCode: string;
  errorMessage: string;
};

type ToastError = {
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; // 토스트를 띄워줄 시간 (초 단위)
};

type AlertError = {
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void; // 얼럿 창의 확인 버튼을 누른 뒤 액션
};
```

이 에러 타입의 유니온 타입을 원소로 하는 배열

```tsx
type ErrorFeedbackType = TextError | ToastError | AlertError;

const errorArr: ErrorFeedbackType[] = [
  { errorCode: "100", errorMessage: "텍스트 에러" },
  { errorCode: "200", errorMessage: "토스트 에러", toastShowDuration: 30 },
  { errorCode: "300", errorMessage: "얼럿 에러", onConfirm: ()=> {} }
];
```

유니온 타입을 이용해서 여러 에러 객체를 관리한다.

- 토스트에러의 toastShowDuration 필드와 얼럿에러의 onConfirm 필드를 모두 가지는 객체에 대해서는 타입 에러가 발생해야 한다. 그러나?

```tsx
const errorArr: ErrorFeedbackType[] = [
	// ...
	{
		errorCode : "999",
		errorMessage: string;
		toastShowDuration: 3000,
		onConfirm: () => {},
	}, // expected error
];
```

**자바스크립트는 “덕 타이핑” 언어이기 때문에 별도의 타입 에러를 뱉지 않는다.** 

❓ 어떻게 해결할 수 있을까?

# 4.3.2 식별할 수 있는 유니온

에러 타입을 구분하는 방법이 필요하다. 

- 각 타입이 유사한 구조를 가지지만 서로 호환되지 않아야 한다.
- “식별할 수 있는 유니온”를 활용

> 식별할 수 있는 유니온이란 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아 주어 포함관계를 제거하는 것이다.
> 

```tsx
type TextError = {
	errorType: "TEXT"; // 판별자 추가
  errorCode: string;
  errorMessage: string;
};

type ToastError = {
	errorType: "TOAST";
  errorCode: string;
  errorMessage: string;
  toastShowDuration: number; 
};

type AlertError = {
	errorType: "ALERT";
  errorCode: string;
  errorMessage: string;
  onConfirm: () => void; 
};
```

errorArr을 새로 정의

```tsx
type ErrorFeedbackType = TextError | ToastError | AlertError;

const errorArr: ErrorFeedbackType[] = [
  {
    errorType: "TEXT",
    errorCode: "100",
    errorMessage: "텍스트 에러",
  },
  {
    errorType: "TOAST",
    errorCode: "200",
    errorMessage: "토스트 에러",
    toastShowDuration: 3000,
  },
  {
    errorType: "ALERT",
    errorCode: "300",
    errorMessage: "얼럿 에러",
    onConfirm: () => { console.log("Confirmed"); },
  },
  {
    errorType: "TEXT",
    errorCode: "999",
    errorMessage: "잘못된 에러",
    toastShowDuration: 3000, 
    // Object literal may only specify known properties, 
    // and 'toastShowDuration' does not exist in type 'TextError'
  },
  {
    errorType: "TOAST",
    errorCode: "210",
    errorMessage: "토스트 에러",
    onConfirm: () => {},
    // Object literal may only specify known properties, and
    // 'onConfirm' does not exist in type 'ToastError'
  },
  {
    errorType: "ALERT",
    errorCode: "310",
    errorMessage: "얼럿 에러",
    toastShowDuration: 5000, 
    // Object literal may only specify known properties, 
    // and 'toastShowDuration' does not exist in type 'AlertError'
  },
  {
    errorType: "ALERT",
    errorCode: "310",
    errorMessage: "얼럿 에러",
    // onConfirm 필드가 없으므로 에러 발생
  },
];
```

타입 에러가 정상적으로 발생한다.

**+. type 이름이 붙은 속성이 판별자로 작동하나요?**

⏩ 그럴 필요는 없습니다. 어떤 속성이든 고유한 값으로 각 유니온 타입을 구분할 수 있다면 판별자로 작동합니다.

```tsx
type Dog = {
  animalType: "dog"
  breed: string;
  bark: () => void;
};

type Cat = {
  animalType: "cat";
  color: string;
  meow: () => void;
};

type Bird = {
  species: "bird";
  wingspan: number;
  fly: () => void;
};
```

세 개의 타입이 존재하고 있을 때

```tsx
type Pet = Dog | Cat | Bird;
```

유니온 타입이 생겼을 때 각 멤버에 “공통으로 존재하는 속성”이 있고, 그 속성의 값이 “멤버마다 고유”하다면 해당 속성이 판별자로 사용됩니다.

# 4.3.3 식별할 수 있는 유니온의 판별자 선정

> 식별할 수 있는 유니온의 판별자는 유닛 타입으로 선언되어야 정상적으로 동작한다.
> 

***유닛 타입은 다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가지는 타입을 말한다.** 

- null, undefined, 리터럴 타입을 비롯해 true, 1 등 정확한 값을 나타내는 타입이 유닛 타입에 해당한다.
- 반면에 다양한 타입을 할당할 수 있는 void, string, number와 같은 타입은 유닛 타입으로 적용되지 않는다.

```tsx
유니온의 판별자로 사용할 수 있는 타입
1. 리터럴 타입이어야 한다.
2. 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며, 
		인스턴스화할 수 있는 타입은 포함되지 않아야 한다.
```

```tsx
interface A {
	value: "a"; // unit type
	answer: 1;
}
interface B {
	value: string; // not unit type
	answer: 2;
}
interface C {
	value: Error; // instantiable type
	answer: 3;
}
```

3개의 인터페이스가 있을 때

```tsx
type Unions = A | B | C;

function handle(param: Unions) {
  // 판별자가 'value'일 때
  param.answer; // 1 | 2 | 3

  // a가 리터럴 타입이므로 타입이 좁혀진다. 단 이는 string에 포함되므로 param은 A | B
  if (param.value === 'a'){
	  param.answer; // 1 | 2
	}
...
```

.

```tsx
	// 유닛 타입이 아니거나 인스턴스화할 수 있는 타입일 경우 타입이 좁혀지지 않는다.
	if (typeof param.value === "string"){
		param.answer; // 1 | 2 | 3
	}
	if (param.value instanceof Error) {
		param.answer;
	}
```

판별자가 value일 때 판별자로 선정한 값 중 ‘a’만 유일하게 유닛 타입이다. 이 때만 유닛 타입을 포함하고 있기 때문에 타입이 좁혀진다.

```tsx
	// 판별자가 answer일 때
	param.value; // string | Error
	
	// 판별자가 유닛 타입이므로 타입이 좁혀진다. 
	if (param.answer === 1){
		param.value; // 'a'
	}
}
```

# 4.4 Exhaustiveness Checking로 정확한 타입 분기 유지하기

Exhaustiveness는 사전적으로 철저함, 완전함을 의미한다. 따라서 어쩌구 체킹은 “모든 케이스에 대해 철저하게 타입을 검사하는 것”을 말한다.

- 타입 가드를 사용해서 타입에 대한 분기 처리를 수행하면
    - 필요하다고 생각되는 부분만 분기 처리해 요구사항에 맞는 코드를 작성할 수 있다.
- 하지만 때로는 **모든 케이스에 대해 분기 처리**를 해야만 유지보수 측면에서 안전하다.

# 4.4.1 상품권

상품권 가격에 따라 상품권 이름을 리턴해주는 함수를 작성한다.

```tsx
type ProductPrice = "10000" | "20000";

const getProductName = (productPrice: ProductPrice): string => {
	if (productPrice === 10000") return " 배민상품권 1만 원 ";
	if (productPrice === 20000") return " 배민상품권 2만 원 ";
	else {
		return " 배민상품권 ";
	}
};
```

! 새로운 상품권이 생겨 ProductPrice 타입이 업데이트되어야 한다.

```tsx
// 5000원 타입 추가
type ProductPrice = "10000" | "20000" | "5000"; // 타입 추가됨

const getProductName = (productPrice: ProductPrice): string => {
	if (productPrice === 10000") return " 배민상품권 1만 원 ";
	if (productPrice === 20000") return " 배민상품권 2만 원 ";
	if (productPrice === 5000") return " 배민상품권 5천 원 "; // 조건 추가!!!
	else {
		return " 배민상품권 ";
	}
};
```

타입이 추가되면서 getProductName도 수정되었다. 그런데 함수를 수정하지 않아도 에러가 발생하지 않기 때문에 절차적으로 “모든 타입에 대한 타입 검사를 강제”하고 싶을 수 있다.

```tsx
// 타입 검사 추가
type ProductPrice = "10000" | "20000" | "5000";

const getProductName = (productPrice: ProductPrice): string => {
	if (productPrice === 10000") return " 배민상품권 1만 원 ";
	if (productPrice === 20000") return " 배민상품권 2만 원 ";
	// if (productPrice === 5000") return " 배민상품권 5천 원 "; // 주석 처리
	else {
		// 5000 타입에 대한 분기 처리를 지웠으므로 에러가 발생하도록 해야 함
		exhaustiveCheck(productPrice); // 에러 : string is not assign able to 'never' type
		return " 배민상품권 ";
	}
};

const exhaustiveCheck = (param:never) => {
	throw new Error("Type error!");
};
```

> 이렇게 모든 케이스에 대한 타입 분기 처리를 해주지 않았을 때, 컴파일 타임 에러가 발생하게 하는 것을 Exhaustiveness Checking라고 한다.
> 

```tsx
const exhaustiveCheck = (param:never) => {
	throw new Error("Type error!");
};
```

해당 함수는 매개변수로 never 타입을 받는다.

- 즉 매개변수로 그 어떤 값도 받을 수 없고 값이 들어오면 에러를 뱉는다.
- 즉 if else에서 마지막 else 구문에 해당 함수를 넣으면 무조건 에러가 발생한다.

> Exhaustiveness Checking 패턴
> 

## 우형이야기

Q. 런타임 코드에 exhaustiveCheck가 포함되어서 프로적션 코드와 테스트 코드가 같이 섞인 느낌 (프로덕션 코드 중간에 assert 구문을 넣는 듯한 느낌)이 든다. 

A. 프로덕션 코드에 어서션을 추가하는 것도 하나의 패턴이긴 합니다. 단위 테스트의 어서션은 특정 단위의 결과를 확인하는 느낌이라면, 코드 상의 어서션은 코드 중간중간에 무조건 특정값을 가지는 상황을 확인하기 위한 디버깅 또는 주석 같은 느낌으로 사용되는 거 같아요.  

리팩토리 2판에는 “테스트 코드가 있다면 어서션이 디버깅 용도로 사용될 때의 효용을 줄어든다. 단위 테스트를 꾸준히 추가하여 사각을 좁히면 어서션보다 나을 때가 많다. 하지만 소통 측면에서는 어서션이 여전히 매력적이다.”

개인적으로 어서션 자체가 성능에 영향을 주지는 않으니 괜찮은 것 같다.

- 프로덕션 코드에 assert 성격의 코드가 들어가면 번들 사이즈가 커진다는 단점만 떠올랐는데 장점도 있다. 트레이드 오프 지점을 찾아서 잘 적용하는 것이 중요