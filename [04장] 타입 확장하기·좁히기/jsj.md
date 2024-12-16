# 4장 (타입 확장하기 · 좁히기)

## 4.1 타입 확장하기

- 기존 타입을 확장하여 새로운 타입을 정의하는 것
- interface · type 키워드로 타입을 정의, extends·교차 타입· 유니온 타입으로 타입 확장
- 중복되는 타입을 반복적으로 선언하는 것 보다 기존에 타입을 작성한 것을 바탕으로 타입 확장을 하여 불필요한 코드 중복을 줄일 수 있음

```tsx
//interface키워드 사용시
interface BaseCartItem extends BaseMenuItem {
	quantity: number,
}

//type키워드 사용시
type BaseCartItem = {
	quantity: item
} & BaseMenuItem
```

- 타입 확장의 장점
    - 중복 제거
    - 명시적인 코드 작성
    - 확장성
- 유니온 타입
    - 2개 이상의 타입을 조합하여 사용하는 방식
    - 합집합의 개념으로 가지고 있는 타입들의 모든 값이 유니온 타입의 값이 됨
    - 유니온 타입으로 선언된 값은 유니온 타입에 포함된 모든 타입이 공통으로 갖고 있는 속성에만 접근할 수 있음
- 교차 타입
    - 기존 타입을 합쳐 필요한 모든 기능을 하나의 타입을 만드는 것
    - 유니온과 다르게 타입을 합쳐 모든 속성을 가진 단일 타입이 됨
    - 합집합이 아니라 교집합의 개념을 가짐 (합쳐지는 타입들의 속성을 모두 만족하기 때문)
    - 하지만 타입이 서로 호환되지 않는 경우
        
        ```tsx
        type IdType = string | number;
        type Numeric = number | boolean;
        
        type Universal = IdType & Numeric;
        //이경우 Universal은 교차 타입으로 두 타입을 모두 만족하는 경우에만 유지됨
        //따라서 두 타입을 모두 만족하는 number로 Universal의 타입이 결정됨
        ```
        
- extends와 교차 타입
    - extends를 사용하여 교차 타입을 작성할 수도 있음
    - (코드 써넣기)
    - extends를 쓰면 interface지만 교차 타입과 유니온 타입을 동시에 사용한 새로운 타입은 오로지 type키워드로만 선언할 수 있음
    - extends를 사용한 타입이 교차 타입과 100% 상응하지는 않음
        - extends를 사용하여 이미 이름이 같은 속성을 가지고 있는데 호환되지 않는 다른 타입으로 새로 선언할 경우 해당 속성의 타입이 호환되지 않는 에러가 발생함
        - 교차 타입을 사용했을 경우 에러를 발생시키지 않고 해당 속성의 타입을 never로 설정함
        - type키워드는 교차 타입으로 선언되었을 때 새롭게 추가되는 속성에 대해 미리 알 수 없기에 에러를 발생시키지는 않지만 같은 속성에 대해 서로 호환되지 않는 타입이 선언되었기 때문에 타입을 never로 설정함

## 4.2 타입 좁히기 - 타입 가드

- 타입 스크립트에서 타입 좁히기 → 변수 또는 표현식의 타입 범위를 더 작은 범위로 좁혀나가는 과정
- 타입가드 → 런타임에 조건문을 사용하여 타입을 검사하고 타입 범위를 좁혀주는 기능
- 스코프 → 변수와 함수 등의 식별자가 유효한 범위를 나타냄. 변수와 함수를 선언하거나 사용할 수 있는 영역
- 컴파일 시 타입 정보는 모두 제거되어 런타임에 존재하지 않기 때문에 타입을 사용하여 조건을 만들 수는 없음
- 컴파일해도 타입 정보가 사라지지 않고 런타임에 유효하게 동작하기 위한 방법은 2가지가 있다. 자바 스크립트 연산자를 사용한 타입가드, 사용자 정의 타입 가드
- 자바 스크립트 연산자를 활용한 타입 가드 → typeof, instanceof, in과 같은 연산자를 사용하여 제어문으로 특정 타입 값을 가질 수 밖에 없는 상황을 유도하여 자연스래 타입을 좁히는 방법. 자바 스크립트 연산자를 사용하여 런타임에 유효한 타입 가드를 만듬.
- 사용자 정의 타입 가드 → 사용자가 직접 어떤 타입으로 값을 좁힐지를 직접 지정하는 방식
- typeof 연산자  → 원시 타입을 추론할 때
    - typeof 연산자를 활용하여 원시 타입에 대해 추론할 수 있다. `typeof A === B` 를 조건으로 분기 처리하면 해당 분기 내에서는 A의 타입이 B로 추론되어 사용 가능해짐. 하지만 typeof는 자바 스크립트 연산자이기에 자바 스크립트 타입 시스템만 대응할 수 있음. (null, 배열 타입이 object 타입으로 판별되는 등 타입 시스템이 조금 다름) 따라서 주로 원시 타입을 좁히는 용도로만 사용하는 것이 권장됨
    - typeof 연산자를 사용하여 검사할 수 있는 타입 목록
        - string, number, boolean, undefined, object, function, bigint, symbol
- instanceof 연산자 → 인스턴스화된 객체 타입을 판별할 때
    - `A instance of B` 형태로 사용하여 A에는 타입을 검사할 대상 변수, B에는 특정 객체의 생성자가 들어감
    - A의 프로토타입 체인에 생성자 B가 존재하는 지를 검사해서 존재하면 true, 그렇지 않으면 false를 반환함. 이런 동작 방식으로 인해 A의 프로토타입 속성 변화에 따라 연산자의 결과가 달라질 수 있음
- in 연산자 → 객체의 속성이 있는지 없는지 구분
    - 객체에 속성이 있는지 확인하고 true, false 값을 반환함
    - `A in B` 형태로 사용하여 A라는 속성이 B객체에 존재하는지를 검사함 (프로토타입 체인으로 접근할 수 있으면 true 반환)
    - B 객체 내부에 A 속성이 있는지 없는지를 검사하는 것이므로 B 객체에 존재하는 A속성에 undefined를 할당한다고 해서 false를 반환하는 것이 아니라 delete를 통해 해당 속성을 제거해야만 false를 반환함
    - 여러 객체 타입을 유니온 타입으로 가지고 있을 때 in 연산자를 사용해서 속성의 유무에 따라 조건 분기를 할 수 있다.
        
        ```tsx
        const NoticeDialog: React.FC <NoticeDialogProps> = (props) => {
        	if ("cookieKey" in props) return <NoticeDialogWithCookie {...props} />;
        return <NoticeDialogBase {...props} />;
        };
        ```
        
- is 연산자 → 사용자 정의 타입 가드
    - 사용자 정의 타입 가드 함수를 만들수 있으며 이 경우 반환 타입이 타입 명제가 된다.
        
        > ❓타입 명제
        함수의 반환 타입에 대한 타입 가드를 수행하기 위해 사용 되는 특별한 형태의 함수
        > 
    - `A is B` 형식으로 작성하며 A는 매개변수 이름, B는 타입이다. 반환 값이 참이면 A 매개 변수의 타입을 B타입으로 취급하게 된다.
        
        ```tsx
        const isDCode = (x: string): x is DCode =>
        	dCodeList.includes(x);
        //x가 list에 포함되어 있으면 (참이면) x를 반환하는데 string이 아니라 DCode 타입으로 반환
        ```
        

## 4.3 타입 좁히기 - 식별할 수 있는 유니온

- 식별할 수 있는 유니온
    - 타입 간의 구조 호환을 막기 위해 타입마다 구분할 수 있는 판별자를 달아주어 포함 관계를 제거하는 것
    - 판별자는 유닛 타입으로 선언되어야 정상적으로 동작함
        
        > ❓ 유닛 타입
        다른 타입으로 쪼개지지 않고 오직 하나의 정확한 값을 가지는 타입
        ex) null, undefined, 리터럴, true, 1 등
        void, string, number 와 같은 타입은 유닛 타입으로 적용되지 않음
        > 
    - 공식에선 판별자의 조건을 이렇게 정의한다
        - 리터럴 타입이어야 함
        - 판별자로 선정한 값에 적어도 하나 이상의 유닛 타입이 포함되어야 하며 인스턴스화 할 수 있는 타입은 포함되지 않아야 한다.

## 4.4 Exhaustiveness Checking으로 정확한 타입 분기 유지하기

- Exhaustiveness Checking
    - 모든 케이스에 대해 철저하게 타입을 검사하는 것
    - 타입 좁히기에 사용 되는 패러다임 중 하나
    - 아래처럼 모든 케이스에 대한 타입 분기 처리를 해주지 않았을 때 컴파일 타입 에러가 발생하게 하는 것
        
        ```tsx
        type ProductPrice = "10000" | "20000" | "5000";
        
        const getProductName = (productPrice: ProductPrice): string => {
        	if (productPrice === 10000") return " 배민상품권 1 만 원 ";
        	if (productPrice === 20000") return " 배민상품권 2만 원 ";
        	// if (productPrice == 5000") return " 배민상품권 5 천 원 ";
        	else {
        		exhaustiveCheck(productPrice); 
        		// Error: Argument of type 'string' is not assign able to parameter of type 'never'
        		return "배민상품권";
        	};
        
        const exhaustiveCheck = (param: never) => {
        	throw new Error("type error!");
        };
        ```
        
    
    - exhaustiveCheck 함수처럼 매개 변수를 never 타입으로 선언하여 마지막 else 문에 사용하면 어떤 값도 받을 수 없고 만일 값이 들어온다면 에러를 내뱉게 만들면 분기 처리가 되지 않은 케이스가 들어와서 에러를 발생 시키게 된다.
    - 이를 통해 예상치 못한 런타임 에러를 방지하거나 요구사항이 변경되었을 때 생길 수 있는 위험성을 줄일 수 있다.