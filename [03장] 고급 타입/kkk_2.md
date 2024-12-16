```jsx
3.3 제네릭 사용법
```
# 3.3 제네릭 사용법

# 3.3.1 함수의 제네릭

타입에 따라 ReadOnlyRepository가 적절하게 사용될 수 있다.

```tsx
function ReadOnlyRepository<T>
(target: ObjectType<T> | EntitySchema<T> | string): // 파라미터 타켓 타입
	Repository<T > { // 리턴 타입
		return getConnection ("ro").getRepository(target); 
	}
	
// 사용 예시
const userRepository = ReadOnlyRepository(User); // User는 ObjectType
const users = await userRepository.find();
console.log(users);
```

# 3.3.2 호출 시그니처의 제네릭

호출 시그니처는 타입스크립트의 함수 타입 문법으로 함수의 매개변수와 반환 타입을 미리 선언하는 것이다. 호출 시그니처를 사용해 개발자는 함수 호출 시 필요한 타입을 별도로 지정할 수 있다. 제네릭 타입을 어디에 위치시키냐에 따라 타입의 범위와 제네릭 타입을 언제 구체 타입으로 한정할지 결정할 수 있다.

```tsx
interface useSelectPaginationProps<T> { 
	categoryAtom: RecoilStatenumber>;
	filterAtom: RecoilState<string[>; 
	sortAtom: RecoilStateSortType>;
	fetcherFunc: (props: CommonListRequest) => Promise «DefaultResponse<ContentListRes ponse<T»>;
}
```

- `<T>`를 useSelectPaginationProps의 타입 별칭으로 한정했다. 따라서 **useSelectPaginaionProps를 사용할 때 타입을 명시함으로써 제네릭 타입을 구체 타입으로 한정한다.**
- useSelectPaginationProps가 사용되는 useSelectPagination 훅의 리턴값도 인자에서 쓰는 제네릭 타입 T와 관련있기 때문에 이처럼 작성했다.

```tsx
export type UseRequesterHookType = (
  RequestData = void,
  ResponseData = void
) => (
  baseURL?: string | Headers,
  defaultHeader?: Headers
) => [RequestStatus, Requester<RequestData, ResponseData>];
```

- <RequestData, ResponseData>는 호출 시그니처의 일부, 즉 괄호 앞에 선언했기 때문에 타입스크립트는 UseRequesterHookType 타입의 함수를 실제 호출할 때 제네릭 타입을 구체 타입으로 한정한다.

배민커머스웹프론트개발팀은 프로젝트 구조를 따르기 위해 아래처럼 작성했다.

```tsx
function useSelectPagination
	<T extends CardListContent | CommonProductResponse
>(
  { categoryAtom, filterAtom, sortAtom, fetcherFunc 
}: useSelectPaginationProps<T>): {
  intersectionRef: RefObject<HTMLDivElement>;
  data: T[];
  categoryId: number;
  isLoading: boolean;
  isEmpty: boolean;
} {
  // ...
  return {
    intersectionRef,
    data: swappedData ? [] : [], // 데이터 처리 로직에 따라 값이 들어갈 것.
    isLoading,
    categoryId,
    isEmpty,
  };
}
```

# 3.3.3 제네릭 클래스

제네릭 클래스는 외부에서 입력된 타입을 클래스 내부에 적용할 수 있는 클래스이다.

```tsx
class LocalDB<T> {
  async put(table: string, row: T): Promise<T> {
    return new Promise<T>((resolve, reject) => {// T타입의 데이터를 DB에 저장
    });
  }

  async get(table: string, key: any): Promise<T> {
    return new Promise<T>((resolve, reject) => { // T타입의 데이터를 DB에서 가져옴
    });
  }

  async getTable(table: string): Promise<T[]> {
    return new Promise<T[]>((resolve, reject) => { // T[]타입의 데이터를 DB에서 가져옴
    });
  }
}

export default class IndexedDB implements ICacheStore {
  private _DB?: LocalDB<{
    key: string;
    value: Promise<Record<string, unknown>>;
    cacheTTL: number;
  }>;

  private DB() {
    if (!this._DB) {
      this._DB = new LocalDB("localCache", {
        ver: 6,
        tables: [
          {
            name: "TABLE_NAME",
            keyPath: "key",
          },
        ],
      });
    }
    return this._DB;
  }
...
}

```

클래스 이름 뒤에 타입 매개변수인 <T>를 선언해준다. <T>는 메서드의 매개변수나 리턴 타입으로 사용될 수 있다. 

> LocalDB 클래스는 외부에서 {key:string; value:Promise<Record<string,unknown>>;catheTTL:number } 타입을 받아들여 클래스 내부에서 사용될 제네릭 타입으로 결정된다.
> 

제네릭 클래스를 사용하면 클래스 전체에 걸쳐 타입 매개변수가 적용된다. 

- **특정 메서드만을 대상**으로 제네릭을 적용하려면 해당 메서드를 제네릭 메서드로 선언한다.

# 3.3.4 제한된 제네릭

타입스크립트에서 제한된 제네릭은 타입 매개변수에 대한 제약 조건을 설정하는 기능을 말한다. 

예시로 string 타입으로 제약하려면 타입 매개변수는 특정 타입을 상속=extends해야 한다.

+.

## ❕제약 예시들

제네릭을 string로 제약걸기

```tsx
function getLength<T extends string>(value: T): number {
  return value.length;
}
// 호출 시 사용 예제
console.log(getLength("Hello")); // 정상 작동, 출력: 5
console.log(getLength(123));     // 오류: number 타입은 string 타입을 상속받지 않음
```

객체 타입 내 특정 키를 제한할 수도 있다.

```tsx
function getKeyValue<T extends { key: string }>(obj: T): string {
  return obj.key;
}

// 호출 예제
console.log(getKeyValue({ key: "value" })); // 정상 작동
console.log(getKeyValue({ key: 123 }));     // 오류: key는 string 타입이어야 함

```

## ❕Exclude 사용 예제

exclude<T,U>는 타입스크립트의 유틸리티 타입으로 타입 T에서 타입 U에 해당하는 부분을 제거한 타입을 리턴한다. 제네릭을 활용해 특정 타입을 제외할 때 유용하다.

- `T`의 각 멤버가 **`U`**에 속하면 제거(**`never`**), 그렇지 않으면 그대로 유지됩니다.

```tsx
type Exclude<T,U> = T extends U? never : T; // 타입 U에 해당하는 부분 제거
```

기본 사용예제

```tsx
type UnionType = "a" | "b" | "c";
type ExcludedType = Exclude<UnionType, "b">;
// 결과: "a" | "c"
```

제네릭과 함께 사용

```tsx
function excludeExample<T, U>(value: T, excludeValue: U): Exclude<T, U> {
  // 실제 로직은 생략
  return value as any;
}
// 사용 예제
const result = excludeExample<"a" | "b" | "c", "b">("a", "b");
// 결과 타입: "a" | "c"
```

❗**교재 코드**

```tsx
// 하 뭔소리지 > 오..
type ErrorRecord<Key extends string> = 
	Exclude<Key, ErrorCodeType> extends never 
	? Partial<Record<Key, boolean>>>
	: never;
```

- Key에서 ErrorCodeType에 해당하는 타입을 제거한다.
- `extends never` : 그 결과가 never인지 확인한다. never이면 Key는 모두 에러코드타입에 포함된다.
- 키가 모두 에러코드타입에 포함되면 선택적 객체 타입을 생성하는데 <Key, boolean> 타입이다.

> 이처럼 타입 매개변수가 특정 타입으로 묶였을 때 키를 타입 매개변수라고 부른다. 그리고 string를 키의 상한 한계 = upper bound 라고 한다.
> 

# 3.3.5 확장된 제네릭

제네릭 타입은 여러 타입을 상속받을 수 있으며 타입 매개변수를 여러 개 둘 수도 있다.

**❓제네릭의 유연성은 유지하면서 타입을 제약해야 할 때는 어떻게 해야 하나요?**

```markdown
<Key extends string> : 제네릭의 유연성이 없어진다.
<Key extends string | number>  : 유니온을 사용해 여러 타입을 받지만 타입 매개변수가 여러개라면?
```

## ❓왜 타입 매개변수가 두 개 필요한가?

```tsx
class APIResponse<T> {
  private readonly data: T | null;
}
```

T라는 단일 제네릭 매개변수를 쓰면 성공 데이터와 에러 데이터를 구분하기 어렵다. 따라서 Ok, Err 이라는 두 개의 타입 매개변수를 사용한다.

```tsx
export class APIResponse<Ok, Err = string> {
  private readonly data: Ok | Err | null;
  private readonly status: ResponseStatus;
  private readonly statusCode: number | null;

  constructor(
    data: Ok | Err | null,
    statusCode: number | null,
    status: ResponseStatus
  ) {
    this.data = data;
    this.status = status;
    this.statusCode = statusCode;
  }
  ...
}
```

- 전체 코드
    
    ```tsx
    export class APIResponse<Ok, Err = string> {
      private readonly data: Ok | Err | null;
      private readonly status: ResponseStatus;
      private readonly statusCode: number | null;
    
      constructor(
        data: Ok | Err | null,
        statusCode: number | null,
        status: ResponseStatus
      ) {
        this.data = data;
        this.status = status;
        this.statusCode = statusCode;
      }
    
      // 성공적인 응답 생성 메서드
      public static Success<T, E = string>(data: T): APIResponse<T, E> {
        return new this<T, E>(data, 200, ResponseStatus.SUCCESS);
      }
    
      // 에러 응답 생성 메서드
      public static Error<T, E = unknown>(init: AxiosError): APIResponse<T, E> {
        if (!init.response) {
          return new this<T, E>(null, null, ResponseStatus.CLIENT_ERROR);
        }
    
        if (init.response.data?.result) {
          return new this<T, E>(
            init.response.data.result,
            init.response.status,
            ResponseStatus.SERVER_ERROR
          );
        }
    
        return new this<T, E>(
          null,
          init.response.status,
          ResponseStatus.FAILURE
        );
      }
    }
    // 사용 코드
    const fetchShopStatus = async (): Promise<APIResponse<ShopResponse | null>> => {
      try {
        const response = await API.get<IShopResponse | null>("/v1/main/shop", config);
        return APIResponse.Success(response.data?.result || null);
      } catch (error) {
        return APIResponse.Error<ShopResponse | null, string>(error as AxiosError);
      }
    };
    
    ```
    

# 3.3.6 제네릭 예시

제네릭의 장점을 코드의 효율적 재사용이다. 현업에서 가장 제네릭이 많이 사용될 때는 API 응답 값의 타입을 지정할 때이다.

```tsx
export interface MobileApiResponst<Data> { // 제네릭 타입 Data 사용
	data : Data;
	statusCode : string;
	statusMessage?: string;
}
```

MobileApiResponse는 실제 API 응답 값의 타입을 지정할 때 아래처럼 사용한다.

```tsx
export const fetchPrinceInfo = (): Promise<MobileApiResponse<PriceInfo>> => {
	const priceUrl = "https: ~~~~" // url 주소
	return request ({
		method: "GET",
		url: priceUrl,
	});
};
export const fetchOrderInfo = (): Promise<MobileApiResponse<Order>> => {
	const orderUrl = "https: ~~~" // url 주소
	return request ({
		method: "GET",
		url: orderUrl,
	});
};
```

## ❗ 제네릭을 굳이 사용하지 않아도 되는 타입

굳이 안필요한데도 제네릭 사용하면 쓸데없이 코드만 길어지고 가독성이 떨어진다. 

```tsx
type GType<T> = T;
type RequirementType = "USE" | "UN_USE" | "NON_SELECT";
interface Order {
	getRequirement(): GType<RequirementType>;
}
```

GType이 해당 함수에서만 사용될 경우 직접 타입 매개변수를 사용하는 것과 사실상 같은 기능을 하고 있다. 

```tsx
type RequirementType = "USE" | "UN_USE" | "NON_SELECT";
interface Order {
	getRequirement(): RequirementType;
}
```

## ❕복잡한 제네릭은 의미 단위로 분할해서 사용하자

제네릭이 과하게 사용되면 가독성을 해친다.