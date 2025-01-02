# 13. 타입스크립트와 객체 지향

## 13.1 타입스크립트의 객체 지향

TS는 JS에는 없는 기능(`private`과 같은 접근 제어자, 추상 클래스, 추상 메서드 등)을 제공해 좀 더 객체 지향 프로그래밍에 가깝게 구현할 수 있도록 도와줄 수 있다.

웹 개발은 제대로된 객체 지향을 구현하기 어렵다. 웹을 개발할 땐 선언적인 언어 또는 문법을 사용하기 때문이다. 객체 지향 개발에서는 객체 간의 협력과 역할에 집중해야 하지만, 컴포넌트를 개발할 땐 **컴포넌트의 역할과 컴포넌트 간의 협력**에 초점을 맞추기 어려운 것이 현실이다.

HTML 마크업을 선언적으로 작성할 때는 디자인 요구 사항이 제시되면 그에 맞춰 마크업을 진행할 수 밖에 없기 때문이다. 그러한 경우 컴포넌트 간의 협력 관계는 주요 고려 대상이 아니다.

게다가 애플리케이션의 변동 사항에 유연하게 대처하기 위해, 즉 변경에 용이하고 유지보수이 쉬운 설계를 위해 객체 지향을 구현하는 것인데, 사전에 레이아웃의 변화하는 것을 예측하는 것은 불가능하다.

그렇다면 어떻게 프론트엔드에서 객체 지향을 효과적으로 활용할 수 있을까? 회사의 전략, 프로젝트의 방향성에 따라 다양한 접근 방식을 고려해야만 한다.

특히 레이아웃은 변동 사항이 생길 가능성이 높으므로 미확정 영역으로 두고 **공통으로 사용되는 컴포넌트와 비즈니스 영역에서 객체 지향 원칙을 적용해서 설계**하는 것이 도움이 될 것이다.

&nbsp;

## 13.2 우아한형제들의 활용 방식

우아한형제들의 한 팀의 설계 방식은 다음과 같다.

1. 온전히 레이아웃만 담당하는 **컴포넌트 영역**
2. 컴포넌트 영역 위에서 레이아웃과 비즈니스 로직을 연결해주는 **커스텀 훅 영역**
3. 커스텀 훅 영역 위에서 객체로서 상호 협력하는 **모델 영역**
4. 모델 영역 위에서 API를 해석하여 모델로 전달하는 **API 레이어 영역**

---

### 1. 컴포넌트 영역

아래는 장바구니 관련 다이얼로그 컴포넌트 코드다. 온전히 레이아웃만 담당하고 있다.

```tsx
// components/CartCloseoutDialog.tsx
import { useCartStore } from "store/modules/cart";

const CartCloseoutDialog: React.VFC = () => {
  // 비즈니스 로직
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

&nbsp;

### 2. 커스텀 훅 영역

```tsx
// store/cart.ts
class CartStore {
  public async add(target: RecommendProduct): Promise<void> {
    // API를 호출
    const response = await addToCart(
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

최종적으로 `setupContext`라는 컨텍스트 훅에서 사용되는 코드이기 때문에 훅 영역의 로직으로 분류됐다.

아래는 `addToCartRequest` 시리얼라이저 함수다.

> 💡 시리얼라이저 함수란? 컴퓨터 과학에서 데이터 구조나 오브젝트를 일관된 형식으로 변환하거나 표준화하는 것을 직렬화(serialization)라고 한다. 시리얼라이저 함수는 데이터 구조나 객체를 다른 형식으로 변환하는 함수를 말한다.
> 내부 데이터 구조를 API 요청에 맞는 형식으로 변환하거나 객체를 전송 가능한 형태(JSON 등)로 변환하는 것도 직렬화에 해당한다.

```ts
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

&nbsp;

### 3. 모델 영역

아래는 훅 영역에서 사용된 모델 객체인 `RecommendProduct` 코드다.

```ts
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
  public getId(): number {
    return this.id;
  }

  public getName(): string {
    return this.name;
  }

  public getThumbnail(): string {
    return this.thumbnailImageUrl;
  }

  public getPrice(): RecommendProductPrice {
    return this.price;
  }

  public getCalculatedPrice(): number {
    const price = this.getPrice();
    return price.sale?.price ?? price.origin;
  }

  public getItems(): RecommendProductItem[] {
    return this.items;
  }

  public getType(): string {
    return this.type;
  }

  public getRef(): string {
    return this.ref;
  }

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

&nbsp;

### 4. API 레이어 영역

아래는 훅 영역에서 실제로 호출되는 비즈니스 로직인 `addToCart` 함수다.

```ts
// apis/Cart.ts
// APIResponse는 데이터 로드에 성공한 상태와 실패한 상태의 반환 값을 제네릭하게 표현해주는 API 응답 객체이다
interface APIResponse<OK, Error> {
  // API 응답에 성공한 경우의 데이터 형식
  ok: OK;
  // API 응답에 실패한 경우의 에러 형식
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

클래스 컴포넌트와 함수 컴포넌트는 필요한 상황에 적절히 쓰면 된다. 최근 리액트 공식 문서에서는 함수 컴포넌트의 사용을 권장하고 있으나, 어느 한 방법이 절대적인 솔루션이라고 보기 힘들다. 클래스 컴포넌트는 틀에서 찍어내듯 일관적인 템플릿에 맞춘 컴포넌트를 많이 생성해야 할 때 유용하다.

&nbsp;

## 13.3 캡슐화와 추상화

캡슐화와 추상화는 객체 지향 패러다임의 핵심 개념이다.

리액트 + TS 프로젝트의 캡슐화는 객체들(컴포넌트)이 유기적으로 협력하게끔 적절한 도메인 분리로써 이루어 낼 수 있다.

객체를 모델링하는 과정에서, 사람이 인지하기 쉽게끔 적합한(Product, Cart 등) 설계를 하는 것이 곧 추상화다.

Prop drilling이 심해질 수록 결합도는 높아지고 내부 처리 로직이 외부로 드러나게 된다. 즉, Prop drilling은 좋지 못한 관계를 형성하게 하고 캡슐화를 저해한다. 이 문제를 해결하기 위해 옵저버 패턴이 등장했고, 다양한 상태 관리 라이브러리들이 등장했다.

이러한 도구를 적절히 활용하는 안목을 기르는 것이 중요할 것 같다.

> 💡 옵저버 패턴은 객체의 상태 변화를 관찰하는 관찰자들, 즉 옵저버들의 목록을 객체에 등록하여 상태 변화가 있을 때마다 메서드 등을 통해 객체가 직접 목록의 각 옵저버에게 통지하도록 하는 디자인 패턴이다. 주로 분산 이벤트 핸들링 시스템을 구현하는 데 사용된다. 발행/구독 모델로 알려져 있기도 하다.

&nbsp;

### 13.4 정리

객체 지향은 결국 궁극적으로 유지보수성을 높이기 위한 패턴에 불과하다.

객체나 책임을 신경쓰지 말고 개발 속도에 집중하여 화면을 구현한 후 각 부분의 역할과 책임을 나누며 리팩터링하는 방법도 좋은 객체 지향 구조를 위한 발돋움이 될 수 있다.

소프트웨어 공학에 은탄환(Silver bullet)은 없다는 말처럼, 객체 지향도 결국 만능 열쇠는 아니다. 유지보수가 필요없는 결과물에는 객체 지향을 적용하지 않는 편이 최선의 선택이 될 수 있다는 점을 알아두자.
