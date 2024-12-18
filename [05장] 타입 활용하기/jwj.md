# 5. 타입 활용하기

## 5.1 조건부 타입

조건에 따라 다른 타입을 반환하고 싶을 때가 있을 것이다. `extends`, `infer`, `never` 등의 키워드를 활용해 내가 원하는 타입만을 반환하게끔 만들 수 있다.

&nbsp;

### 1. extends와 제네릭을 활용한 조건부 타입

`extends` 키워드는 타입을 조건부로 설정해줄 때도 사용한다. 삼항 연산자를 사용해서 다음과 같이 선언해준다.

```ts
T extends U ? X : Y
```

**타입 T를 U에 할당할 수 있으면 X 타입, 아니면 Y 타입이 된다는 의미**다. T에 들어오는 값에 따라 타입을 바꿔줄 수 있다.

```ts
interface Bank {
  financialCode: string;
  companyName: string;
  name: string;
  fullName: string;
}

interface Card {
  financialCode: string;
  companyName: string;
  name: string;
  appCardType?: string;
}

type PayMethod<T> = T extends 'card' ? Card : Bank;
type CardPayMethodType = PayMethod<'card'>;
type BankPayMethodType = PayMethod<'bank'>;
```
위 코드는 T에 `card`가 들어오면 T의 타입을 `Card`로 설정하고 아닐 경우 `Bank`로 설정해준다.

&nbsp;

### 2. 조건부 타입을 사용하지 않았을 때의 문제점

조건부 타입을 사용하지 않고 유니온 타입으로만 처리할 경우 문제가 발생할 수 있다.  

```ts
interface PayMethodBaseFromRes {
  financialCode: string;
  name: string;
}
interface Bank extends PayMethodBaseFromRes {
  fullName: string;
}
interface Card extends PayMethodBaseFromRes {
  appCardType?: string;
}
type PayMethodInfo<T extends Bank | Card> = T & PayMethodInterface;
type PayMethodInterface = {
  companyName: string;
  //...
}
```
```ts
type PayMethodType = PayMethodInfo<Card> | PayMethodInfo<Bank>;

// 타입으로 "card", "appcard", "bank"를 받아서 해당 결제 수단의 결제 수단 정보 리스트를 반환하는 함수
export const useGetRegisteredList = (
  type: 'card' | 'appcard' | 'bank'
): UseQueryResult<PayMethodType[]> => {
  const url = `baeminpay/codes/${type === 'appcard' ? 'card' : type}`;
  const fetcher = fetcherFactory<PayMethodType[]>({
    onSuccess: (res) => {
      const usablePocketList =
        res?.filter(
          (pocket: PocketInfo<Card> | PocketInfo<Bank>) =>
            pocket?.useType === 'USE'
        ) ?? [];
      return usablePocketList;
    },
  });

  const result = useCommonQuery<PayMethodType[]>(url, undefined, fetcher);
  
  return result;
};
```
책의 설명이 애매한데, 아마 글쓴이의 의도는 `"card"`를 넣을 경우 `PocketInfo<Card>`를 반환할 것을 기대하지만, `PayMethodType`은 `PayMethodInfo<Card > | PayMethodInfo<Bank>`의 유니온 타입이 이기 때문에 의도치 않게 사용하는 쪽에서는 `PocketInfo<Bank>`를 반환할 가능성도 있어 정확한 타입 반환이 어렵다는 것을 설명하고 있는 것 같다.

이러한 문제를 해결하기 위해 `extends`를 사용한다.

&nbsp;

### 3. extends 조건부 타입을 활용하여 개선하기

위에서 본 `PayMethodType`을 아래와 같이 개선할 수 있다.

```ts
type PayMethodType<T extends 'card' | 'appcard' | 'bank'> = T extends
  | 'card'
  | 'appcard'
  ? Card
  : Bank;

export const useGetRegisteredList = <T extends 'card' | 'appcard' | 'bank'>(
  type: T
): UseQueryResult<PayMethodType<T>[]> => {
  const url = `baeminpay/codes/${type === 'appcard' ? 'card' : type}`;

  const fetcher = fetcherFactory<PayMethodType<T>[]>({
    onSuccess: (res) => {
      const usablePocketList =
        res?.filter(
          (pocket: PocketInfo<Card> | PocketInfo<Bank>) =>
            pocket?.useType === 'USE'
        ) ?? [];
      return usablePocketList;
    },
  });
  
  const result = useCommonQuery<PayMethodType<T>[]>(url, undefined, fetcher);

  return result;
};
```

해당 코드는 타입으로 들어올 수 있는 값을 의도적으로 한정하고 있어 다른 값이 들어올 경우 타입 에러를 반환하게 되며, 인자에 맞는 타입만 반환할 수 있어 타입 가드가 따로 필요없다.

&nbsp;

❗️타입 확장을 제외한 `extends` 활용 예시

1. 제네릭과 함께 사용하면 의도한 값을 제외한 다른 값을 허용하지 않아 프로그램을 견고하게 만들 수 있다.
2. 조건부 타입을 통해 반환 값을 사용자가 원하는 값으로 구체화할 수 있어 불필요한 타입 가드, 타입 단언 등을 방지할 수 있다.

&nbsp;

### 4. infer를 활용해서 타입 추론하기

`infer` 키워드는 TS 2.8 버전에서 도입되었으며 조건부 타입 내에서 사용되는 키워드로, 특정 타입을 추론할 수 있게 해준다. `infer`를 사용하면 타입 변수의 값을 동적으로 결정하고, 그 값을 조건부 타입의 참/거짓 분기에서 사용할 수 있다.

`infer`는 반드시 `extends`와 함께 사용되어야 하며, 주로 조건부 타입의 "true" 분기에서 타입을 추론하는 데 사용된다.

```ts
type UnpackPromise<T> = T extends Promise<infer K>[] ? K : any;

const promises = [Promise.resolve('Mark'), Promise.resolve(38)];
type Expected = UnpackPromise<typeof promises>; // string | number
```

`UnpackPromise` 타입은 T를 받아 T가 Promise로 래핑된 경우라면 K를 반환하고, 그렇지 않은 경우 `any`를 반환한다. `Promise<infer K>`는 Promise의 반환 값을 추론해 해당 값의 타입을 K로 정한다는 의미다.

&nbsp;

타입을 조건에 따라 더욱 세밀하게 뽑아내는 배민 라이더 어드민 서비스 예시를 보자.

```ts
interface RouteBase {
  name: string;
  path: string;
  component: ComponentType;
}

export interface RouteItem {
  name: string;
  path: string;
  component?: ComponentType;
  pages?: RouteBase[];
}

export const routes: RouteItem[] = [
  {
    name: '기기 내역 관리',
    path: '/device-history',
    component: DeviceHistoryPage,
  },
  {
    name: '헬멧 인증 관리',
    path: '/helmet-certification',
    component: HelmetCertificationPage,
  },
  // ...
];
```
`RouteBase`와 `RouteItem`은 라이더 어드민에서 라우팅을 위해 사용된다. 권한 API에서 온 사용자 권한과 `name`을 비교하여 인가되지 않은 접근을 막는다. `RouteItem`의 `name`은 `pages`가 있을 때는 단순히 이름이지만, 그렇지 않을 때는 사용자 권한 비교하는데 사용된다.

이러한 배경을 바탕으로 아래 코드를 보자.

```ts
export interface SubMenu {
  name: string;
  path: string;
}

export interface MainMenu {
  name: string;
  path?: string;
  subMenus?: SubMenu[];
}

export type MenuItem = MainMenu | SubMenu;
export const menuList: MenuItem[] = [
    // MainMenu 타입
  {
    name: '계정 관리',
    subMenus: [
        // SubMenu 타입
      {
        name: '기기 내역 관리', // route name
        path: '/device-history',
      },
      { 
        name: '헬멧 인증 관리',
        path: '/helmet-certification',
      },
    ],
  },
  // SubMenu 타입?
  {
    name: '운행 관리',
    path: '/operation',
  },
  // ...
];
```

위의 코드와 유사하게, `MainMenu`와 `SubMenu`는 메뉴 리스트에서 사용되고 권한 API를 통해 반환된 사용자 권한과 `name`을 비교하여 사용자가 접근할 수 있는 메뉴만 렌더링한다. `MainMenu`의 `name`은 `subMenus`를 가지고 있을 때는 단순히 이름이고, 그렇지 않으면 권한으로 간주된다.

모든 `name`에는 정확히 일치하는 문자열(책에 따르면, 동일한 문자열)만 들어와야 한다.

하지만 `name` 속성은 `string` 타입으로만 정의되어 있기 때문에 **어떤 문자열 값이 들어와도 컴파일 에러를 발생시키지는 않는다**. 또한 런타임에도 인가되지 않았다는 페이지만 보여주거나, 메뉴 리스트를 렌더링하지 않기만 하기 때문에, **권한이 없어서 렌더링 되지 않는 거구나라고 잘못 생각**할 수 있다.

그럼 이렇게 권한 검사가 필요한 이름에 대해 별도 타입을 만들어 주는 방법도 생각해볼 수 있다.

```ts
type PermissionNames = '기기 정보 관리' | '안전모 인증 관리' | '운행 여부 조회';
```

하지만 이렇게 처리하면 권한 검사가 필요없는 `subMenus`나 `pages`가 존재하는 `name`에 대해서 또 따로 처리를 해줘야 한다. (어떻게 처리?)

결국 지금 필요한 것은 권한 검사가 필요한 타입을 따로 뽑아내는 것이다.

이떄 `infer`와 불변 객체(`as const`)를 활용해서 `menuList` 배열(혹은 방금 보았던 `routes` 배열)에서 권한 검사가 필요한 값만을 추출하여 타입을 정의하는 식으로 개선할 수 있다.

```ts
export interface MainMenu {
  // ...
  subMenus?: ReadonlyArray<SubMenu>;
}
 
export const menuList = [
  // ...
] as const;
  
interface RouteBase {
  name: PermissionNames;
  path: string;
  component: ComponentType;
}

export type RouteItem =
  | {
    name: string;
    path: string;
    component?: ComponentType;
    pages: RouteBase[];
    }
  | {
    name: PermissionNames;
    path: string;
    component?: ComponentType;
  };
```

먼저 `subMenus`의 값이 변경되지 않도록 readonly 배열로 변환해준다. 그리고 `pages`가 없는 `RouteItem`에 대해 `name`을 `PermissionNames` 타입으로 한정해준다.

다음은 `PermissionNames`를 뽑아내기 위한 타입을 하나 정의해준다. 

```ts
type UnpackMenuNames<T extends ReadonlyArray<MenuItem>> = T extends
  ReadonlyArray<infer U>
    ? U extends MainMenu // U가 MainMenu라면
      ? U['subMenus'] extends infer V // subMenus를 infer V로 추출
        ? V extends ReadonlyArray<SubMenu> // V가 존재한다면(SubMenu 타입에 할당할 수 있으면)
          ? UnpackMenuNames<V> // V를 UnpackMenuNames에 다시 전달
          : U['name'] // V가 존재하지 않으면 U는 권한 타입
        : never
      : U extends SubMenu // U가 SubMenu라면
      ? U['name'] // U는 권한
      : never
    : never;
```

해당 타입의 타입 추론은 재귀적으로 이루어진다.

1. U가 `MainMenu`라면 `subMenus`를 `infer V`로 추출한다.
2. `subMenus`는 옵셔널 타입이기 때문에 V가 존재한다면 `UnpackMenuNames`에 다시 전달한다.
3. V가 존재하지 않으면 `MainMenu`의 `name`은 권한에 해당하므로 `U["name"]`이다.
4. U가 `MainMenu`가 아니라 `SubMenu`라면 `U["name"]`은 권한에 해당한다.

&nbsp;

결과적으로 권한으로 쓰여지는 `name`들만 추출된다.
```ts
export type PermissionNames = UnpackMenuNames<typeof menuList>; // [기기 내역 관리, 헬멧 인증 관리, 운행 관리]
```

&nbsp;

## 5.2 템플릿 리터럴 타입 활용하기

앞서 유니온 타입을 사용해서 변수 타입을 특정 문자열로 지정해줄 수 있었다.

4.1 버전부터 문자열을 확장할 수 있는 템플릿 리터럴 타입을 지원한다. JS의 템플릿 리터럴 문법을 사용해 특정 문자열에 대한 타입을 선언할 수 있다.

```ts
type HeadingNumber = 1 | 2 | 3 | 4 | 5;
type HeaderTag = `h${HeadingNumber}`;

type Vertical = 'top' | 'bottom';
type Horizon = 'left' | 'right';
type Direction = Vertical | `${Vertical}${Capitalize<Horizon>}`;
```

코드의 가독성을 높일 수 있고, 코드의 재사용과 수정에 용이한 장점이 있다.

⚠️ TS의 컴파일러가 유니온을 추론하는데 시간이 오래 걸리면 타입을 추론하지 않고 에러를 뱉을 때가 있다. 타입의 조합의 수가 너무 많지 않게 적절하게 조절해야 한다.
```ts
type Digit = 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9;
// 10^4개의 경우의 수
type Chunk = `${Digit}${Digit}${Digit}${Digit}`;
// (10^4)^2개의 경우의 수
type PhoneNumberType = `010-${Chunk}-${Chunk}`;
```

&nbsp;

## 5.3 커스텀 유틸리티 타입 활용하기

원하는 타입을 정확히 표현하기 위해 TS의 유틸리티 타입을 적절히 이용해 유틸리티 함수나 커스텀 유틸리티 타입을 작성해 사용할 수 있다.

TS의 유틸리티 타입들은 다음과 같다. 

```ts
1. Partial<Type>
// 모든 속성을 선택적으로 만듭니다.
2. Required<Type> 
// 모든 속성을 필수로 만듭니다.
3. Readonly<Type> 
// 모든 속성을 읽기 전용으로 만듭니다.
4. Record<Keys, Type> 
// 특정 키 타입과 값 타입을 가진 객체 타입을 생성합니다.
5. Pick<Type, Keys> 
// 특정 속성만 선택하여 새로운 타입을 만듭니다.
6. Omit<Type, Keys> 
// 특정 속성을 제외한 새로운 타입을 만듭니다.
7. Exclude<UnionType, ExcludedMembers>
// 유니온 타입에서 특정 타입을 제외합니다.
8. Extract<Type, Union>
// 유니온 타입에서 특정 타입만 추출합니다.
9. NonNullable<Type>
// null과 undefined를 제외한 타입을 생성합니다.
10. ReturnType<Type>
// 함수의 반환 타입을 추출합니다.
11. Parameters<Type>
// 함수의 매개변수 타입을 튜플 타입으로 추출합니다.
12. ConstructorParameters<Type>
// 클래스 생성자의 매개변수 타입을 튜플 타입으로 추출합니다.
13. Awaited<Type>
// Promise 타입을 재귀적으로 언래핑합니다.
14. ThisParameterType<Type>
// 함수 타입의 'this' 매개변수의 타입을 추출합니다.
15. OmitThisParameter<Type>
// 함수 타입에서 'this' 매개변수를 제거합니다.
16. ThisType<Type>
// 메서드의 'this' 타입을 지정하는 데 사용되는 마커 인터페이스입니다.
17. Uppercase<StringType>
// 문자열 리터럴 타입의 모든 문자를 대문자로 변환합니다.
18. Lowercase<StringType>
// 문자열 리터럴 타입의 모든 문자를 소문자로 변환합니다.
19. Capitalize<StringType>
// 문자열 리터럴 타입의 첫 글자를 대문자로 변환합니다.
20. Uncapitalize<StringType>
// 문자열 리터럴 타입의 첫 글자를 소문자로 변환합니다.
```

&nbsp;

### 1. 유틸리티 함수를 활용해 styled-components의 중복 타입 선언 피하기

리액트 컴포넌트에서 여러 옵션을 `props`로 받아 컴포넌트를 제작할 때가 있다.
이때 `Pick`이나 `Omit`과 같은 유틸리티 타입을 적절하게 사용하여 원하는 타입만 가져오거나 뺄 수 있다.

```ts
// HrComponent.tsx
export type Props = {
  height?: string;
  color?: keyof typeof colors;
  isFull?: boolean;
  className?: string;
  ...
}

export const Hr: VFC<Props> = ({ height, color, isFull, className }) => {
  ...

  return <HrComponent height= {height} color= {color} isFull= {isFull} className= {className} />;
};

// style.ts
import { Props } from '...';

type StyledProps = Pick<Props, "height" | "color" | "isFull">;

const HrComponent = styled.hr<StyledProps>`
  height: ${({ height }) = > height || "10px"};
  margin: 0;
  background-color: ${({ color }) = > colors[color || "gray7"]};
  border: none;
  ${({ isFull }) => isFull && css`
    margin: 0 -15px;
  `}
`;
```

`StyledProps` 타입은 `Props` 타입에서 `Pick` 유틸리티 타입을 이용해 필요한 속성만 재정의해준 타입이다. 만약 `StyledProps`를 따로 정의한다면 중복 코드가 발생하고 `Props`의 타입이 변경되면 `StyledProps`도 같이 변경해주어야 하기 때문에 코드 관리가 번거로워진다. 

&nbsp;

### 2. PickOne 유틸리티 함수

서로 다른 2개 이상의 객체를 유니온 타입으로 받을 때 타입 검사가 정확하게 진행되지 않을 때가 있다. 

아래의 코드 예제를 보자.

```ts
type Card = {
  card: string
};

type Account = {
  account: string
};

function withdraw(type: Card | Account) {
  ...
}

withdraw({ card: "hyundai", account: "hana" });
```

이렇게 유니온 타입으로 작성하면 `card` 속성과 `account` 속성을 모두 가진 객체도 타입 에러가 발생하지 않는다. 유니온 타입이므로 `card` 속성이 들어올 수도 있고, `account` 속성이 들어올 수도 있기 때문이다. 개발자의 의도대로라면 유효하지 않은 타입이 들어온 상황이므로 에러를 발생시켜야 이상적일 것이다.

&nbsp;

이런 문제를 해결하기 위해 식별할 수 있는 유니온을 사용할 수도 있으나, 모든 타입에 판별자를 달아줘야 하는 번거로움이 존재한다. 만약 이미 모든 구현이 끝난 상태라면 일일히 판별자를 추가해주다가 일부 작성해주지 않는 실수를 저지를 수도 있다.

&nbsp;

궁극적으로 개발자가 구현하고자 하는 타입은 **`card` 또는 `account` 속성 하나만 존재하는 객체를 받는 타입**이다. `account`일 때는 `card`를 받지 못하고, `card`일 때는 `account`를 받지 못하게 하려면 어떻게 해야 할까?

**상호배타적이어야 하는 속성을 옵셔널한 `undefined` 값으로 설정**해주면 된다. 이렇게 하면 의도적으로 `undefined` 값을 할당하지 않는 이상 타입 에러가 발생하게 된다. (`undefined` 타입에는 `undefined` 값 밖에 할당할 수 없기 때문이다.)

```ts
{ account: string; card?: undefined } | { account?: undefined; card: string }
```

결국 선택하고자 하는 하나의 속성을 제외한 나머지 값을 옵셔널 + undefined 타입으로 설정해주면 원하는 속성을 제외하면 타입 에러를 발생시킬 것이다

&nbsp;

이를 구현하는 `PickOne`이라는 커스텀 유틸리티 타입은 다음과 같이 구현할 수 있다.

```ts
type PickOne<T>= {
    [P in keyof T]: Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>>;
}[keyof T];
```

&nbsp;

이제 `PickOne<T>`을 자세히 살펴보자. 두 가지 타입을 합쳐서 만든 것으로 생각할 수 있다. 여기서 T는 항상 객체가 들어온다고 생각하자.

`Example` 객체에 PickOne을 적용하려고 한다.

```ts
interface Example {
    a: string;
    b: number;
}
```

&nbsp;

첫 번째 구성 요소인 `One<T>` 타입을 살펴보자.
   
```ts
type One<T> = { [P in keyof T]: Record<P, T[P]> }[keyof T];
```

이 타입은 **특정 객체를 단일 속성을 갖는 객체들을 유니온으로 나열**한다.

1. `[P in keyof T]`: P는 T 객체의 key 값의 모음이다.
2. `Record<P, T[P]>`: P 타입을 key로 가지고, value로 P를 key로 갖는 T 객체의 값의 레코드 객체다.
3. `{ [P in keyof T]: Record<P, T[P]> }`: key를 T 객체의 키 모음으로 가지고, value로 P 타입 키의 원본 객체 T를 가진다.
4. `[keyof T]`: 3번의 객체를 T의 키 값으로 접근한다. (객체에서 값만 추출한다.)
   


```ts
// [P in keyof T]의 결과
P = 'a' | 'b'

// { [P in keyof T]: Record<P, T[P]> }의 결과
{
    a: Record<'a', string>,  // a : { a: string }
    b: Record<'b', number>   // b : { b: number }
}

// [keyof T]를 적용한 결과
// 위 객체에서 모든 값들만 추출해서 유니온으로 만듦
{ a: string } | { b: number }

// 최종 결과
type Result = One<Example>;
// Record<'a', string> | Record<'b', number>
// == { a: string } | { b: number }
```

&nbsp;

두 번째 구성 요소인 `ExcludeOne<T>`을 살펴보자.

```ts
type ExcludeOne<T> = { [P in keyof T]: Partial<Record<Exclude<keyof T, P>, undefined>> }[keyof T];
```

이 타입은 **특정 객체의 속성 하나를 제외한 모든 속성을 옵셔널 + undefined 속성으로 갖는 단일 객체를 유니온으로 나열**한다.

1. `[P in keyof T]`: P는 T 객체의 key 값의 모음이다.
2. `<Exclude<keyof T, P>`: T 객체의 key 모음에서 P 타입의 key를 제외한다. 이것을 A 타입이라고 하자.
3. `Record<A, undefined>`: A 타입을 key로, undefined를 value로 갖는 레코드 타입이다. (`[key]: undefined`) 이것을 B 타입이라고 하자.
4. `Partial<B>`: B 타입을 모두 옵셔널로 변환한다. (`[key]?: undefined`)
5. 4번에서 생성된 객체를 key 값으로 접근한다. (객체에서 값만 추출한다.)

```ts
// [P in keyof T]의 결과
P = 'a' | 'b'

// keyof T와 P를 대입한 결과
{
  a: Partial<Record<Exclude<'a' | 'b', 'a'>, undefined>>,
  b: Partial<Record<Exclude<'a' | 'b', 'b'>, undefined>>
}

// Exclude한 결과
{
  a: Partial<Record<'b', undefined>>, // a: Partial<{ b: undefined }>,
  b: Partial<Record<'a', undefined>> // b: Partial<{ a: undefined }>
}

// Partial을 적용한 결과
{
  a: { b?: undefined },
  b: { a?: undefined }
}

// 최종 결과
type Result = ExcludeOne<Example>;
// { b?: undefined } | { a?: undefined }
```

&nbsp;

이제 `PickOne<T>`은 다음과 같이 표현할 수 있다.

```ts
type PickOne<T> = One<T> & ExcludeOne<T>;

// 공통으로 갖는 요소와 함께 교차한 결과
type PickOne<T> = { [P in keyof T] : Record<P, T[P]> & Partial<Record<Exclude<keyof T, P>, undefined>> }[keyof T];

type Result = PickOne<Example>;
// { a: string; b?: undefined } | { b: number; a?: undefined }
```

최종적으로 원하는 속성 하나(`One<T>`)를 제외한 모든 속성을 옵셔널 + undefined로 만드는(`ExcludeOne<T>`) 유틸리티 함수가 완성됐다.

마지막으로 처음에 봤던 예제 코드에 `PickOne`을 적용해 보자.

```ts
type CardOrAccount = PickOne<Card & Account>;

function withdraw (type: CardOrAccount) {
  ...
}

withdraw({ card: "hyundai", account: "hana" }); // 에러 발생
```

&nbsp;

### 3. NonNullable 타입 검사 함수를 사용하여 간편하게 타입 가드하기

`null`을 가질 수 있는 값의 `null` 처리를 해줄 때, 일반적으로는 if문을 이용해서 `null` 처리 타입 가드를 적용할 수 있다.

```ts
interface User {
    email: string | null;
    // ...
}

function processUserEmail(user: User) {
    // null 체크 타입 가드
    if (user.email !== null) {
        // user.email을 string 타입으로 처리
        console.log(user.email.toUpperCase());  // OK
    } else {
        console.log('Email not provided');
    }
}
```

&nbsp;

유틸리티 함수를 사용해서 간편하게 타입 가드를 해주는 방법도 있다. `is`와 `NonNullable<T>` 유틸리티 타입으로 타입 검사를 위한 `NonNullable` 유틸리티 함수를 작성해서 사용하면 된다.

```ts
type NonNullable<T> = T extends null | undefined ? never : T;
```

`NonNullable<T>`은 타입 T가 `null`이나 `undefined`일 때 `never`를 반환하고, 아닐 경우 그대로 T 타입을 반환한다.

`NonNullable` 함수는 다음과 같이 작성할 수 있다.

```ts
function NonNullable<T>(value: T): value is NonNullable<T> {
  return value !== null && value !== undefined;
}
```

해당 함수는 `value`가 `null` 또는 `undefined`일 경우 `false`를 반환한다. `is` 키워드로 인해 `true`가 반환되면 `null`이나 `undefined`가 제외된 타입으로 좁혀진다.

&nbsp;

이제 `NonNullable` 함수가 유용하게 쓰이는 예시를 보자.

```ts
class AdCampaignAPI {
  static async operating(shopNo: number): Promise<AdCampaign[]> {
    try {
      return await fetch(`/ad/shopNumber=${shopNo}`);
    } catch (error) {
      return null;
    }
  }
}
```

해당 API는 여러 상품의 광고를 받아오는 로직이다. API 하나에서 에러가 발생했다고 해서 전체 광고가 안보이면 안되니 `try-catch`문으로 에러가 발생하면 `null`을 반환하게 되게끔 처리했다. 

```ts
const shopList = [
  { shopNo: 100, category: "chicken" },
  { shopNo: 101, category: "pizza" },
  { shopNo: 102, category: "noodle" },
];

const shopAdCampaignList = await Promise.all(shopList.map((shop)=> AdCampaignAPI.operating(shop.shopNo)));
```

`shopAdCampaignList`는 `null`을 반환할 수도 있기 때문에 `Array<AdCampaign[] | null>`로 추론된다.

만약 이를 if문으로 타입 가드하게 되면 고차 함수 내 콜백 함수에서 리스트를 순회할 때마다 매번 if문을 거쳐야 한다. 또는 단순하게 필터링한다면 (`shopAdCampaignList.filter ((Shop)=>!!shop)]`, `!!` 연산자로 falsy 값들 (null, undefined, 0, "", false, NaN)을 false로 변환, 나머지는 true로 변환)

이때 `NonNullable` 함수를 사용하면 `null`이 제외된, 좁혀진 타입으로 추론해줄 수 있다.

```ts
const shopAds = shopAdCampaignList.filter(NonNullable); // Array<AdCampaign[]>
```

&nbsp;

## 5.4 불변 객체 타입으로 활용하기

상숫값을 관리할 때 객체를 많이 사용한다. 컴포넌트나 함수에서 이런 객체들을 사용할 때, 열린 타입으로 설정할 수 있다. 하지만 열린 타입으로 설정하면 어떤 값이 들어올지 한정할 수 없어 타입 안정적이지 않다.

이때 객체를 `as const`로 만들어서 불변 객체로 선언하고, `keyof` 연산자로 객체에 접근하면 실제 객체 안에 존재하는 키값만 받도록 해줄 수 있다.

이렇게 하면 객체에 있는 값을 제외한 다른 값은 타입 에러가 발생하고, 자동 완성 기능을 통해 객체에 있는 값을 쉽게 파악할 수 있다.

```ts
const colors = {
  red: "#45452",
  green: "#0C952A",
  blue: "#1A7CFF",
} as const;

type ColorKey = keyof typeof colors;

// const getColorHex = (key: string) => colors[key] // implicitly has an 'any' type
const getColorHex = (key: ColorKey) => colors[key];
```

TS에만 존재하는 `keyof`는 객체의 키 값을 유니온으로 반환하게 하고, TS에서의 `typeof`는 JS에서와는 쓰임이 조금 다르다. JS에서는 특정 변수의 타입을 추출하기 위한 용도로 주로 쓰이지만,TS에서는 변수 혹은 속성의 타입을 추론하는 용도로 쓰인다.

```ts
function createUser(name: string, age: number) {
   return {
       name,
       age,
       createdAt: new Date()
   };
}

type User = ReturnType<typeof createUser>;  // { name: string; age: number; createdAt: Date }
```

&nbsp;

### 1. Atom 컴포넌트에서 theme style 객체 활용하기

Atom 컴포넌트란 Button, Header, Input 등 작은 단위의 컴포넌트를 의미한다. 

상숫값 객체를 관리 사용할 때 아래와 같이 열린 타입으로 받아올 수 도 있다.

```ts
interface Props {
  fontSize?: string;
  backgroundColor?: string;
  color?: string;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
}

const Button: FC<Props> = ({ fontSize, backgroundColor, color, children }) => {
  return (
    <ButtonWrap
      fontSize= {fontSize}
      backgroundColor= {backgroundColor}
      color= {color}
    >
      {children}
    </ButtonWrap>
  );
};


const ButtonWrap = styled.button<Omit<Props, "onClick">>`
  color: ${({ color }) => theme.color[color ?? "default"]};
  background-color: ${({ backgroundColor }) =>
    theme.bgColor[backgroundColor ?? "default"]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize ?? "default"]};
`;
```

`Props`에서 `string` 타입으로 선언해주었던 속성을, 상숫값 객체를 `keyof`와 `typeof` 키워드를 사용해 추출한 타입으로 바꾸어 `Button` 컴포넌트에 활용해보자.

```ts
import React, { FC } from "react";
import styled from "styled-components";

const colors = {
  black: "#000000",
  gray: "#222222",
  white: "#FFFFFF",
  mint: "#2AC1BC",
};

const theme = {
  colors: {
    default: colors.gray,
    ...colors
  },
  backgroundColors: {
    default: colors.white,
    gray: colors.gray,
    mint: colors.mint,
    black: colors.black,
  },
  fontSize: {
    default: "16px",
    small: "14px",
    large: "18px",
  },
};

// 상숫값에서 타입 추출
type ColorType = keyof typeof theme.colors;
type BackgroundColorType = keyof typeof theme.backgroundColors;
type FontSizeType = keyof typeof theme.fontSize;

// 추출한 타입을 Props에 적용
interface Props {
  color?: ColorType;
  backgroundColor?: BackgroundColorType;
  fontSize?: FontSizeType;
  children?: React.ReactNode;
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void | Promise<void>;
}

const Button: FC<Props> = ({ fontSize, backgroundColor, color, children }) => {
  return (
    <ButtonWrap
      fontSize={fontSize}
      backgroundColor={backgroundColor}
      color={color}
    >
      {children}
    </ButtonWrap>
  );
};

const ButtonWrap = styled.button<Omit<Props, "onClick">>`
  color: ${({ color }) => theme.colors[color ?? "default"]};
  background-color: ${({ backgroundColor }) =>
    theme.backgroundColors[backgroundColor ?? "default"]};
  font-size: ${({ fontSize }) => theme.fontSize[fontSize ?? "default"]};
`;
```

이렇게 사용하면 `Button` 컴포넌트를 사용할 때 IDE가 `backgroundColor` 속성에 존재하는 값만을 제시해줄 수 있어 개발 시 편리하다는 장점도 있다.

&nbsp;

## 5.5 Record 원시 타입 키 개선하기

`Record`의 키를 `string`과 같은 원시 타입으로 명시하곤 하는데, 이때 키가 유효하지 않더라도(없는 값이 들어와도) 타입 상으로는 문제가 없기 때문에 오류를 뱉지 않는다. 이는 런타임 에러를 발생시킬 수 있기 때문에 타입을 명시적으로 사용하는 방법을 알아보자.

&nbsp;

### 1. 무한한 키를 집합으로 가지는 Record

```ts
type Category = string;
interface Food {
  name: string;
  // ...
}
const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};
```

`Category`는 `string` 타입이므로 `Record`의 키로 `Category`를 사용하는 `foodByCategory` 객체는 무한한 키 집합을 가지게 된다. 

```ts
foodByCategory["양식"]; // Food[]로 추론
foodByCategory["양식"].map((food) => console.log(food.name)); // 컴파일 에러가 발생하지 않는다

// 런타임 에러 발생!!
foodByCategory["양식"].map((food) => console.log(food.name)); // Uncaught TypeError: Cannot read properties of undefined (reading 'map')
```

이때 JS의 옵셔널 체이닝을 사용해 런타임 에러를 방지할 수는 있다. 

> 💡 옵셔널 체이닝이란? `.?`와 같이 사용한다. 객체의 속성을 찾는 중간에 없는 값에 대해 (`null` 또는 `undefined`) `undefined`를 반환한다.

하지만 어떤 값이 `undefined`인지 매번 판단해야 하는 번거로움이 있다. 또한 의도적으로 `undefined`일 수 있는 값이 존재하면 예상치 못한 런타임 에러가 발생할 수도 있다.

TS의 타입 추론 기능으로 유효하지 않은 키가 사용되었는지, `undefined`일 수 있는 값이 있는지 사전에 파악할 수 있다.

&nbsp;

### 2. 유닛 타입으로 변경하기

키가 유한하다면 유닛 타입을 사용하면 된다.

```ts
type Category = "한식" | "일식";
interface Food {
name: string;
// ...
}
const foodByCategory: Record<Category, Food[]> = {
  한식: [{ name: "제육덮밥" }, { name: "뚝배기 불고기" }],
  일식: [{ name: "초밥" }, { name: "텐동" }],
};

// Property ‘양식’ does not exist on type ‘Record<Category, Food[]>’.
foodByCategory["양식"];
```

이제 `Category`에 없는 양식에는 타입 에러를 발생시킨다.

그러나, 키가 무한해야 하는 상황이라면 적합하지 않다. (양식, 중식 등 여러 키가 들어올 가능성이 존재)

&nbsp;

### 3. Partial을 활용하여 정확한 타입 표현하기

키가 무한해야 하는 상황이라면 `Partial`을 사용해서 해당 값이 `undefined`일 수 있는 상태임을 표시해줄 수 있다.

```ts
type PartialRecrod<K extends string, T> = Partial<Record<K, T>>;
type Category = string;

interface Food {
    name: string;
    // ..
}

const foodByCategory : PartialRecord<Category, Food[]> = {
    한식 : [{ name: "제육덮밥" }, { name: "뚝배기 불고기"}],
    일식 : [{ name: "초밥" }, { name: "텐동" }],
}

foodByCategory["양식"]; // Food[] 또는 undefined
foodByCategory["양식"].map((food) => console.log(food.name)); // object is possibly 'undefined'
foodByCategory["양식"]?.map((food) => console.log(food.name)); // OK
```

&nbsp;