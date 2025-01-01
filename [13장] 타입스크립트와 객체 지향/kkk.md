```jsx
13.1 타입스크립트의 객체 지향

13.2 우아한 형제들의 활용 방식
13.2.1 컴포넌트 영역
13.2.2 커스텀 훅 영역
13.2.3 모델 영역
13.2.4 API 레이어 영역

13.3 캡슐화와 추상화
13.4 정리
```

# 13.1 타입스크립트의 객체 지향

> 프론트엔드에 객체 지향을 어떻게 적용하지?
> 

자바스크립트로도 객체 지향 프로그래밍을 할 수 있긴 한데 자바스크립트는 **프로토타입 기반**의 객체 지향 언어로 분류된다. 전통적인 객체 지향 방식인 클래스 기반 객체지향 방식과는 차이가 있다. 이 차이 때문에 온전히 객체지향을 구현하기에는 일부 기능이 없어서 부족함이 있다.

이러한 제약을 타입스크립트가 private와 같은 접근 제어자나 추상 클래스, 추상 메서드 같은 기능을 지원하면서 해결해준다. 타입스크립트는 객체 지향을 구현할 수 있도록 도와주는 자바스크립트의 **슈퍼셋**으로 볼 수 있다.

타입스크립트는 점진적 타이핑, 구조적 타이핑 그리고 덕 타이핑이 결합한 언어이다. 이 3가지가 모여서 객체 지향의 폭을 넓혀준다.

- 점진적 타이핑으로 개발자가 명시한 일부만 정적 타입 검사를 거치게 하고 나머지는 자동 추론되도록 가능
- 덕 타이핑은 객체의 변수와 메서드 집합이 객체의 타입을 결정!
- 구조적 타이핑으로 명시적 타이핑(노미널 타이핑)과 달리 객체의 속성에 해당하는 특정 타입의 속성을 가지는지 검사해서 타입 호환성을 결정한다.

## ✅ 노미널 타이핑과 구조적 타이핑

노미널 타이핑은 명시적으로 타입을 선언하는 것으로 대표적으로 자바, C# 등이 있다. 인터페이스와 클래스가 일대일로 대응한다.

타입스크립트와 같은 구조적 타이핑 언어는 하나의 클래스에 여러 인터페이스가 연결될 수 있다. 하나의 인터페이스에 여러 클래스가 연결될 수도 있다.

## ✅ 타입스크립트로 객체지향 구현하기

객체는 별다른게 아니고 자주 쓰는 컴포넌트도 객체의 한 형태다. 컴포넌트는 스스로 책임을 져야 하는 역할을 수행하면서 다른 컴포넌트 객체와 협력하는 독립적인 객체다. 컴포넌트를 조합하는 것도 객체 지향을 활용하는 것이라 볼 수 있다.

## ❓ 객체 지향의 관점에서 타입스크립트가 프론트엔드에 주는 이점?

> 타입스크립트는 prop를 인터페이스로 정의할 수 있다. 객체 지향 패러다임에서는 객체 간의 협력 관계에 초점을 둔다. 컴포넌트 간의 협력 관계를 표현하는 것이 prop이다. 
또한 객체 자체가 아니라 프레임워크에 의해 객체의 의존성이 주입되는 DI(의존성 주입) 패턴을 따르는데 이러한 패턴을 더욱 명확하게 표현할 수 있게 해주는 것이 타입스크립트다.
> 

### 의존성 주입 패턴(DI)

의존성 주입은 객체 간의 의존 관계를 설정하는데 사용된다. DI를 구현하면 객체 간의 결합도를 낮출 수 있다. 구현하려면 a 클래스가 b 클래스에 의존한다고 하면 b의 클래스가 아니라 인터페이스에 의존하도록 설계해야 한다.

=== 구체클래스가 아닌 인터페이스에 의존해야 한다.

> 타입스크립트 자체가 객체 지향적으로 표현하는데 큰 장점을 가지고 있다. 앞서 언급한 대로 타입스크립트는 점진적 타이핑, 구조적 타이핑, 덕 타이핑을 결합한 언어로 객체 지향의 폭을 넓혀준다.
> 

### 객체 간의 협력과 역할

객체 지향을 따르려면 객체, 즉 컴포넌트 간의 역할과 협력에 초점을 맞춰야 한다. 그러나 웹개발에서는 선언적인 문법 = jsx를 사용해 마크업을 처리한다. 이 때 컴포넌트 간의 관계를 떠올리기 어렵다. 디자인 요구사항을 먼저 구현해야 하기 때문이다. 선언적인 문법을 사용하면서 객체 지향을 완벽하게 구현하는 것은 너무 어렵다.

컴포넌트만 객체가 아니고 상품을 담는 장바구니가 있다고 할 때, 이들이 유기적으로 상호작용하면 이것도 객체라고 볼 수 있다. 리액트에서 컴포넌트는 props를 받아 결과물을 렌더링한다. 이 때 props도 객체로 볼 수 있다.

그..렇다면 변동사항이 생길 가능성이 높은 레이아웃 부분을 빼고 공통으로 사용되는 컴포넌트와 비즈니스 영역에서 객체 지향 원칙을 적용해 설계한다면 좋은 구조를 개발할 수 있다.

# 13.2 우아한 형제들의 활용 방식

다음과 같은 설계 방식을 사용한다.

- 온전히 레이아웃만 담당하는 컴포넌트 영역
- 컴포넌트 영역 위에서 레이아웃과 비즈니스 로직을 연결해주는 커스텀 훅 영역
- 훅 영역 위에 객체로서 상호 협력하는 모델 영역
- 모델 영역 위에서 api를 해석하여 모델로 전달하는 api 영역

자세히 알아보자

- 디렉토리 구조 예상
    
    ```tsx
    src/
    ├── components/
    │   └── CartCloseoutDialog.tsx      // 다이얼로그 컴포넌트, UI 로직
    ├── models/
    │   ├── externals/
    │   │   └── Cart/
    │   │       └── Request.ts          // 외부 요청 모델 정의 (AddToCartRequest 인터페이스)
    │   │   └── lib/
    │   │       └── IRequestHeader.ts   // 요청 헤더 인터페이스 정의
    │   ├── internals/
    │   │   └── Cart/
    │   │       └── RecommendProduct.ts // RecommendProduct 클래스 정의
    │   │   └── Stuff/
    │   │       └── Product.ts          // Product 관련 클래스 정의 (추정)
    ├── serializers/
    │   └── cart/
    │       └── addToCartRequest.ts     // API 요청 데이터 생성 로직
    ├── store/
    │   └── modules/
    │       └── cart.ts                 // CartStore 및 StoreProvider 정의
    
    ```
    

## 13.2.1 컴포넌트 영역

> 온전히 레이아웃만 담당하는 컴포넌트 영역
> 

**장바구니 관련 다이얼로그 컴포넌트 코드**

레이아웃 영역만 담당한다. 비즈니스 로직은 useCartStore 내부 어딘가에 존재한다. 

```tsx
// components/CartCloseoutDialog.tsx

import { useCartStore } from "store/modules/cart";

const CartCloseoutDialog: React.VFC = () => {
  const cartStore = useCartStore();

  return (
    <Dialog
      opened={cartStore.PresentationTracker.isDialogOpen("closeout")}
      title="마감 세일이란?"
      onRequestClose={cartStore.PresentationTracker.closeDialog}
    >
      <div
        css={css`
          margin-top: 8px;
        `}
      >
        지점별 한정 수량으로 제공되는 할인 상품입니다. 재고 소진 시 가격이
        달라질 수 있습니다. 유통기한이 다소 짧으나 좋은 품질의 상품입니다.
      </div>
    </Dialog>
  );
};

export default CartCloseoutDialog;
```

## 13.2.2 커스텀 훅 영역

> 컴포넌트 영역 위에서 레이아웃과 비즈니스 로직을 연결해주는 커스텀 훅 영역
> 

**전역 상태를 관리하는 스토어 내의 useCartStore**

왜 훅이 아닌 스토어로 들어가는지 의문을 가질 수 있겠지만, 해당 스토어 객체에서 최종적으로 사용되는 setupContext는 컨텍스트와 관련된 훅을 다루는 유틸리티 함수이기 때문에 훅 영역의 로직으로 봐도 될 것이다.

 **즉 장바구니에 상품을 담는 비즈니스 로직을 레이아웃과 연결해주기 위한 커스텀 훅 영역이다.**

> **시리얼라이저 함수는 데이터를 특정 형식으로 변환하거나 구조화하는 함수이다.**
> 

```tsx
// store/cart.ts
class CartStore {
  public async add(target: RecommendProduct): Promise<void> {
    const response = await addToCart( // api 호출 함수
      addToCartRequest({
        auths: this.requestInfo.AuthHeaders,
        cartProducts: this.productsTracker.PurchasableProducts,
        shopID: this.shopID,
        target,
      })
    );

    return response.fork(
      (error, _, statusCode) => {
        switch (statusCode) {
          case ResponseStatus.FAILURE:
            this.presentationTracker.pushToast(error);
            break;
          case ResponseStatus.CLIENT_ERROR:
            this.presentationTracker.pushToast(
              "네트워크가 연결되지 않았습니다."
            );
            break;
          default:
            this.presentationTracker.pushToast(
              "연결 상태가 일시적으로 불안정합니다."
            );
        }
      },
      (message) => this.applyAddedProduct(target, message)
    );
  }
}

const [CartStoreProvider, useCartStore] = setupContext<CartStore>("CartStore");
export { CartStore, CartStoreProvider, useCartStore };
```

**addToCartRequest 시리얼라이저 함수**

AddToCartRequest 타입의 객체를 반환하며 매개변수로 받는 target는 RecommendProduct 타입을 가진다. 해당 타입에 대한 정의는 어디서 확인할 수 있을까? 아나 ㅠ 좀 빨리 알려줘요 ㅠ

```tsx
// serializers/cart/addToCartRequest.ts
import { AddToCartRequest } from "models/externals/Cart/Request";
import { IRequestHeader } from "models/externals/lib";
import {
  RecommendProduct,
  RecommendProductItem,
} from "models/internals/Cart/RecommendProduct";
import { Product } from "models/internals/Stuff/Product";

interface Params {
  auths: IRequestHeader;
  cartProducts: Product[];
  shopID: number;
  target: RecommendProduct;
}

function addToCartRequest({
  auths,
  cartProducts,
  shopID,
  target,
}: Params): AddToCartRequest {
  const productAlreadyInCart = cartProducts.find(
    (product) => product.getId() === target.getId()
  );

  return {
    body: {
      items: target.getItems().map((item) => ({
        itemId: item.id,
        quantity: getItemQuantityFor(productAlreadyInCart, item),
        salePrice: item.price,
      })),
      productId: target.getId(),
      shopId: shopID,
    },
    headers: auths,
  };
}

export { addToCartRequest };
```

## 13.2.3 모델 영역

> 훅 영역 위에 객체로서 상호 협력하는 모델 영역
> 

“RecommendProduct”는 클래스로 표현된 객체로 추천 상품을 나타내며, 이 객체는 다른 컴포넌트 및 모델 객체와 함께 협력하게 된다. 

```tsx
// models/Cart.ts

export interface AddToCartRequest {
  body: {
    shopId: number;
    items: { itemId: number; quantity: number; salePrice: number }[];
    productId: number;
  };
  headers: IRequestHeader;
}

/**
 * 추천 상품 관련 class
 */

export class RecommendProduct {
  public getId(): number {return this.id;}

  public getName(): string {return this.name;}

  public getThumbnail(): string {return this.thumbnailImageUrl;}

  public getPrice(): RecommendProductPrice {return this.price;}

  public getCalculatedPrice(): number {
    const price = this.getPrice();
    return price.sale?.price ?? price.origin;
  }

  public getItems(): RecommendProductItem[] {return this.items; }

  public getType(): string {return this.type;}

  public getRef(): string {return this.ref;}

  constructor(init: any) {
    this.id = init.id;
    this.name = init.displayName;
    this.thumbnailImageUrl = init.thumbnailImageUrl;
    this.price = {
      sale: init.displayDiscounted
        ? {
            price: Math.floor(init.salePrice),
            percent: init.discountPercent,
          }
        : null,
      origin: Math.floor(init.retailPrice),
    };
    this.type = init.saleUnit;
    this.items = init.items.map((item) => {
      return {
        id: item.id,
        minQuantity: item.minCount,
        price: Math.floor(item.salePrice),
      };
    });
    this.ref = init.productRef;
  }

  private id: number;
  private name: string;
  private thumbnailImageUrl: string;
  private price: RecommendProductPrice;
  private items: RecommendProductItem[];
  private type: string;
  private ref: string;
}
```

## 13.2.4 API 레이어 영역

> 모델 영역 위에서 api를 해석하여 모델로 전달하는 api 영역
> 

**훅에서 실제로 실행되는 addToCart 함수**

```tsx
// apis/Cart.ts

// APIResponse는 데이터 로드에 성공한 상태와 실패한 상태의 반환 값을 제네릭하게 표현해주는 API 응답 객체이다
// (APIResponse<OK, Error>)
interface APIResponse<OK, Error>
  ok: OK;   // API 응답에 성공한 경우의 데이터 형식
  error: Error;
}

export const addToCart = async (
  param: AddToCartRequest
): Promise<APIResponse<string, string>> => {
  return (await GatewayAPI.post<IAddCartResponse>("/v3/cart", param)).map(
    (data) => data.message
  );
};
```

**정리**

많이 하는 착각이 객체 지향 구현 자체를 클래스라고 생각하는 것이다. 전혀 그렇지 않다. 클래스는 객체를 표현하는 방법의 도구일 뿐이다! 컴포넌트를 함수형으로 선언하든 클래스형으로 선언하든 모두 객체를 나타낸다.

**어떤 방법을 사용하는게 나을까?**

기본적으로 함수형 컴포넌트를 권장한다. 그러나 일관된 템플릿에 맞춘 컴포넌트를 많이 생성해야 할 때는 페이지 템플릿을 클래스 컴포넌트로 만들어 사용하기도 한다. 공통으로 정의되어야 할 행동 (내비게이션 뒤로 가기 버튼을 눌렀을 때의 동작 등)을 abstract 메서드로 만들어 사용하기도 한다.

또 이를 단순 props로 넘길 수도 있다. 결국 상황에 적절한 방법을 선택해야 한다.

# 13.3 캡슐화와 추상화

캡슐화란 다른 객체 내부의 데이터를 꺼내와서 직접 다루지 않고, 해당 객체에게 처리할 행위를 따로 요청함을써 협력하는 것이다.

컴포넌트는 객체다. 컴포넌트의 내부 데이터인 상태가 캡슐화의 대상이 될 수 있다. 즉 컴포넌트 내의 상태와 prop을 잘 다루는 것이 캡슐화의 개념에 부합하는 것이다.

prop drilling가 심할수록 컴포넌트 간의 결합도는 높아지며 내부 처리 로직이 외부로 드러난다. 즉 드릴링은 캡슐화를 저해한다. 이런 문제를 해결하기 위해 옵저버 패턴이 등장했으며 나아가 다양한 상태 관리 라이브러리가 생겨났다.

우리의 목적으로 객체들이 유기적으로 협력하게끔 해서 도메인을 잘 분리하는 것이다. 어떻게 더 유기적인 협력 관계를 만들 수 있을지, 명확하게 도메인을 분리할 수 있을지에 집중하자.

어?끝?

# 13.4 정리

1. 객체나 책임을 신경쓰지 않고 일단 개발 속도에 집중해서 화면을 만들고 이후 리팩토링하는 것도 객체 지향을 구현하기 위한 좋은 트레이닝이다. 이렇게 경험을 쌓이면 알아서 설계에 대한 고민을 하게 된다.
2. 설계 방식도 트레이드오프기 때문에 설계에 너무 큰 공을 들이지 않는 것이 더 나은 경우도 많다. 
3. 결국 객체 지향은 유지보수를 위한 개념이므로 트레이드 오프 잘 생각하기!