```jsx
7.1 API 요청
7.2 API 상태 관리하기
7.3 API 에러 핸들링
7.4 API 모킹
```

<details><summary>요약본</summary>
# 7.1 요약하기

```tsx
7.1.1 fetch로 api 요청하기
7.1.2 서비스 레이어로 분리하기
7.1.3 axios 활용하기
7.1.4 axios 인터셉터 사용하기
7.1.5 api 응답 타입 지정하기
7.1.6 뷰 모델 사용하기
7.1.7 Superstruct를 사용해 런타임에서 응답 타입 검증하기
7.1.8 실제 api 응답 시의 Superstruct 활용사례
```

api 요청은 기본적으로 fetch를 사용할 수 있다. 

🩵**비동기 호출 로직의 유지보수성과 확장성을 높이기 위해 어떻게 구조를 설계할 수 있을까?**

먼저 서비스 레이어로 분리할 수 있다. 이는 비동기 호출 코드를 컴포넌트 영역에서 분리하는 것이다.

fetch보다 다양한 기능을 편리하게 사용하기 위해 axios 라이브러리를 사용한다. (쿼리 매개변수, 커스텀 헤더 추가, 쿠키 읽고 토큰 처리 등)

🩵**서로 다른 baseUrl=api entry일 때 어떻게 하면 효율적으로 관리할 수 있을까?**

이 때 서로 다른 axios 인스턴스를 사용할 수 있다. 

여기에 더해 인터셉터를 사용해 요청과 응답에 공통 로직이나 개별 추가 로직을 정의해 줄 수 있다. 

또 인터셉터에 빌더 패턴을 적용할 수 있는데 이 경우 다양한 옵션을 설정해 선택적으로 인터셉터를 처리해줄 수 있다. 그러나 보일러 플레이트 코드가 많아지는 tradeoff가 존재한다.

🩵**api 응답 타입**

공통 응답 타입을 정의할 수 있다. 이 때 모든 api 응답이 같은 구조를 가지지 않으므로(data가 필수일 수도, 아닐 수도 있다.) **응답 타입은 axios 인스턴스 밖에서 처리해야 한다.**

🩵**뷰모델을 사용해 응답 변경에 유연하게 대응하기**

응답은 변경 가능성이 높다. 뷰 모델에서는 api 응답을 받아 컴포넌트가 사용하기 더 좋은 형태로 변환한다.(필드 이름 재정의 alias 등) 컴포넌트가 뷰모델에 의존하도록 설계한다. 이를 통해 응답이 변경되어도 뷰모델의 로직만 수정하면 된다. 

🩵**뷰모델 방식을 사용할 때 고려할 점**

뷰모델 방식의 단점은 추상화 레이어 코드가 추가되면서 추가적인 타입, 그 파생 타입들이 기하급수적으로 늘어날 수 있다는 것이다. 또 필드명의 재정의할 경우 문제가 생길 여지가 있다. 

그렇다면 어떻게 해야 하는가?

필요한 곳에만 뷰모델을 사용하거나 뷰모델에 게터함수를 추가하는 등의 절충안을 고려해야 한다.

**api 응답 타입 에러가 발생할 때**

api 응답은 실행 시점에만 확인 가능하기 때문에 컴파일 단계에서는 api 응답 타입이 올바른지 알 수 없다. 그러나 타입스크립트는 컴파일 이후 타입 정보가 사라지므로 런타임에서 오류가 발생하지 않는다. 이 때 런타임 응답 타입 오류를 발생시키기 위해 superstruct 라이브러리를 사용할 수 있다. 

Superstruct를 사용하여 인터페이스 정의와 자바스크립트 데이터의 유효성 검사를 쉽게 할 수 있다.

Superstruct는 런타임에서의 데이터 유효성 검사(assert, is, validate)를 통해 개발자와 사용자에게 자세한 런타임 에러를 보여주기 위해 고안되었다.

# 7.2 요약

```tsx
7.2.1 상태 관리 라이브러리에서 호출하기
7.2.2 훅으로 호출하기
```

🩵**비동기 상태 관리**

컴포넌트 내에서 비동기 함수를 직접 호출하지 않는다. 호출하기 위해서는 api의 성공, 실패에 따른 상태가 관리되어야 한다. 이를 상태 관리 라이브러리에서 관리할 수도 있고 (리덕스, mobX), 데이터 패칭 라이브러리를 사용할 수도 있다. (리액트 쿼리, useSwr)

🩵**❓API 상태를 관리할 때 상태 관리 라이브러리를 사용했을 때의 문제점** 

1. 비동기 처리 함수를 호출하기 위해 액션을 추가해야 한다. 이에 따라 관련 상태나 스토어가 계속 늘어나게 된다.
2. 전역 상태 관리자는 모든 비동기 상태에 접근하고 변경할 수 있다. 그 과정에서 의도치 않은 상태 변경 가능성이 높아진다.
3. 여러 컴포넌트가 동일한 상태를 구독할 경우 중복된 API 호출이 발생할 수  있고 한 컴포넌트의 상태 변경이 다른 컴포넌트에 영향을 미칠 수 있다. 

그렇다면 어떤 방법이 또 있을까? 

훅으로 호출하는 방법이 있다. **데이터 패칭 라이브러리**는 훅을 사용해 비동기 상태 관리 문제를 해결할 수 있는데 일단은 상태관리에 최적화된 라이브러리는 아니다. 대표적으로 리액트 쿼리나 useSwr이 있다. 

이 방법을 사용하면 캐시를 사용해 비동기 함수를 호출해서 중복 호출을 줄일 수 있고, 상태 관리 라이브러리에서발생했던 의도치 않은 상태 변경 방지에 도움이 된다.

- 상태 변경이 전역 상태 관리, 스토어에서 이루어지는 대신 데이터는 컴포넌트 수준에서 훅을 통해 관리된다. 각 비동기 상태가 독립적으로 상호영향을 끼치지 않는다.
- 캐싱과 캐싱무력화를 적절히 사용할 수 있고, **컴포넌트가 항상 최신 상태**를 유지하도록 실시간에 가깝게 데이터를 유지하기 위해서 폴링이나 웹소켓을 사용할 수도 있다.

결론: 비동기 호출에 대한 상태 관리 라이브러리에 대한 고민이 필요하다!
</details>

# 7.1 API 요청

## 7.1.1 fetch로 API 요청하기

개요

> 사용자가 장바구니를 조회해서 볼 수 있는 기능
> 

```tsx
sconst CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0);

  useEffect(() => {
    fetch("https://api.baemin.com/cart")
      .then((response) => response.json())
      .then((data) => {
        if (data?.cartItem) {
          setCartCount(data.cartItem.length);
        }
      })
      .catch((error) => {
        console.error("Error fetching cart data:", error);
      });
  }, []);

  return (
    <div>
      <span>Cart Items: {cartCount}</span>
    </div>
  );
};
export default CartBadge;

```

fetch를 사용해서 api 호출한다. 그런데 이후 백엔드 기능 변경 요구가 들어와 api url을 변경해야 한다. 그 밖에도 타임아웃, 커스텀 헤더 수정 등의 기능 변경이 요구된다. 이 경우 비동기 호출 코드를 매번 수정해야 한다.

> ❓ API 호출 로직의 유지보수성과 확장성을 높이기 위해 어떻게 구조를 설계해야 효율적일까요?
> 

## 7.1.2 서비스 레이어로 분리하기

유지보수성과 확장성을 고려하면 비동기 호출 코드는 컴포넌트 영역에서 분리되어 “서비스 레이어”에서 처리되어야 한다.

- 서비스 레이어란?
    
    애플리케이션의 비즈니스 로직을 컴포넌트나 UI와 독립적으로 관리하기 위해 설계된 계층. 주로 비동기 API 호출(외부 API, 백엔드와 통신 처리) 로직을 컴포넌트에서 분리해 서비스 레이어에서 처리한다.
    

## 7.1.3 Axios 활용하기

fetch는 내장 라이브러리이기 때문에 많은 기능을 가지고 있지 않다.(쿼리 매개변수, 커스텀 헤더 추가, 쿠키 읽고 토큰 처리 등) 대신 “Axios” 라이브러리를 사용하면 편리하다.

```tsx
import axios, { AxiosInstance, AxiosPromise } from "axios";

// Axios 인스턴스 생성
const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000, // 요청 타임아웃 5초
});

// API 호출: 장바구니 조회
const fetchCart = (): AxiosPromise<FetchCartResponse> => 
  apiRequester.get<FetchCartResponse>("cart");

// API 호출: 장바구니 추가
const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<PostCartResponse> => 
  apiRequester.post<PostCartResponse>("cart", postCartRequest);

export { fetchCart, postCart };

```

### ❕서로 다른  API Enrty(Base URL)를 사용할 때 효율적으로 관리하기

```tsx
import axios, { AxiosInstance } from "axios";

// 공통 설정을 포함한 기본 설정 객체
const defaultConfig = {
  timeout: 5000, // 모든 요청에 대해 5초 타임아웃
  headers: {
    "Content-Type": "application/json",
  },
};

// 공통 API 서버
const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  ...defaultConfig,
});

// 주문 처리 서버
const orderApiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.or/",
  ...defaultConfig,
});

// 장바구니 처리 서버
const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: "https://cart.baemin.order/",
  ...defaultConfig,
});

```

서로 다른 인스턴스를 구성한다. 

```tsx
// 사용예제
// 공통 API 호출 예제
const fetchCommonData = (): Promise<any> => 
  apiRequester.get("/common-data");

// 주문 관련 API 호출 예제
const fetchOrderDetails = (orderId: string): Promise<any> => 
  orderApiRequester.get(`/orders/${orderId}`);

const createOrder = (orderData: any): Promise<any> => 
  orderApiRequester.post("/orders", orderData);

// 장바구니 관련 API 호출 예제
const fetchCartItems = (): Promise<any> => 
  orderCartApiRequester.get("/cart");

const addToCart = (cartItem: any): Promise<any> => 
  orderCartApiRequester.post("/cart", cartItem);

export { fetchCommonData, fetchOrderDetails, createOrder, fetchCartItems, addToCart };

```

## 7.1.4 Axios 인터셉터 사용하기

### ❔**Axios 인터셉터(Interceptors)란?**

> 요청(리퀘스트)나 응답(리스펀스)가 실제로 서버와 주고받기 전에 가로채서 추가작업을 수행하도록 한다. 공통 로직, 헤더 추가, 에러 처리 등의 작업을 중앙에서 관리할 수 있다.
> 

---

각 리퀘스터는 서로 다른 역할을 담당하는 다른 서버이기 때문에 다른 헤더를 설정해줘야 할 수도 있다.

인터셉터 기능 : 리퀘스터에 따라 비동기 호출 내용을 추가해서 처리한다. 또 API 에러를 처리할 때 하나의 에러 객체로 묶어서 처리할 수 있다.

1. 기본세팅

```tsx
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from "axios";

// 유틸리티 함수
const getUserToken = () => "user-token"; // 유저 토큰 반환
const getAgent = () => "agent-info"; // 에이전트 정보 반환
const getOrderClientToken = () => "order-client-token"; // 주문 클라이언트 토큰 반환

// 기본 설정
const orderApiBaseUrl = "https://api.baemin.com/order";
const orderCartApiBaseUrl = "https://api.baemin.com/cart";
const defaultConfig = { timeout: 5000 };
```

```tsx
// Axios 인스턴스 생성
const apiRequester = axios.create({ baseURL: "https://api.baemin.com", ...defaultConfig });
const orderApiRequester = axios.create({ baseURL: orderApiBaseUrl, ...defaultConfig });
const orderCartApiRequester = axios.create({ baseURL: orderCartApiBaseUrl, ...defaultConfig });
```

1. 헤더 설정 함수

```tsx
// 공통 API 요청 헤더 설정 함수
const setRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
  const config = { ...requestConfig };
  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    user: getUserToken(),
    agent: getAgent(),
  };
  return config;
};

// 주문 API 요청 헤더 설정 함수
const setOrderRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
  const config = { ...requestConfig };
  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    "order-client": getOrderClientToken(),
  };
  return config;
};

```

1. 요청 인터셉터

```tsx
// 요청 인터셉터 적용, 요청 전에 헤더 적용
apiRequester.interceptors.request.use(setRequestDefaultHeader);

orderApiRequester.interceptors.response.use(setOrderRequestDefaultHeader);

orderCartApiRequester.interceptors.response.use(setRequestDefaultHeader);
```

1. 응답 인터셉터

```tsx
// 요청 에러 핸들러
const httpErrorHandler = (error: any) => {
  console.error("HTTP Error:", error.response || error.message);
  return Promise.reject(error);
};

// 응답 인터셉터 적용
apiRequester.interceptors.response.use(
  (response: AxiosResponse) => response,
  httpErrorHandler
);
```

### 🔹Axios 인터셉터에 빌더 패턴 추가 : APIBuilder 클래스

> 빌더 패턴이란?
> 
1. API 클래스

기본 api 클래스로 실제 호출 부분을 구성하고, 위와 같은 api를 호출하기 위한 래퍼(wrapper)를 빌더 패턴으로 만든다.

```tsx
import axios, { AxiosPromise } from "axios";

export type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
export type HTTPHeaders = Record<string, string>;
export type HTTPParams = Record<string, any>;

export class API {
  readonly method: HTTPMethod;
  readonly url: string;
  baseURL?: string;
  headers?: HTTPHeaders;
  params?: HTTPParams;
  data?: unknown;
  timeout?: number;
  withCredentials?: boolean;

  constructor(method: HTTPMethod, url: string) {
    this.method = method;
    this.url = url;
  }

  call<T>(): AxiosPromise<T> {
    const http = axios.create();
```

```tsx
		// 만약 withCrendential이 설정된 api라면 인터셉터를 추가함
    if (this.withCredentials) {
      http.interceptors.response.use(
        (response) => response,
        (error) => {
          if (error.response && error.response.status) {
            console.error("Error Status:", error.response.status);
          }
          return Promise.reject(error);
        }
      );
    }

    return http.request({
      method: this.method,
      url: this.url,
      baseURL: this.baseURL,
      headers: this.headers,
      params: this.params,
      data: this.data,
      timeout: this.timeout,
      withCredentials: this.withCredentials,
    });
  }
}

```

1. APIBuilder 클래스

```tsx
import { API } from "./API";
export class APIBuilder {
  private _instance: API;

  constructor(method: string, url: string, data?: unknown) {
    this._instance = new API(method, url);
    this._instance.baseURL = apiHost;
    this._instance.data = data;
    this._instance.headers = {
      "Content-Type": "application/json; charset=utf-8",
    };
    this._instance.timeout = 5000;
    this._instance.withCredentials = false;
  }
```

1. 생성자
    
    요청 메서드(CRUD)와 url을 입력받아 기본 api 인스턴스를 초기화한다. 기본값에 헤더, 타임아웃 등 설정
    
    ```tsx
    constructor(method: string, url: string, data?: unknown) {
      this._instance = new API(method, url);
      this._instance.baseURL = apiHost; // 기본 URL 설정
      this._instance.data = data; // 요청 데이터 설정
      this._instance.headers = { 
        "Content-Type": "application/json; charset=utf-8" 
      };
      this._instance.timeout = DEFAULT_TIMEOUT; // 타임아웃 설정
      this._instance.withCredentials = false; // 인증 쿠키 포함 여부 기본값
    }
    
    ```
    
2. 정적 메서드
    
    http 메서드별로 간단하게 빌더를 생성할 수 있도록 한다.
    
    ```tsx
    static get = (url: string) => new APIBuilder("GET", url);
    static post = (url: string, data: unknown) => new APIBuilder("POST", url, data);
    static put = (url: string, data: unknown) => new APIBuilder("PUT", url, data);
    static delete = (url: string) => new APIBuilder("DELETE", url);
    ```
    

3. 빌더 메서드

- 각종 설정을 추가하거나 수정하는 메서드들
    
    ```tsx
    // 기본 url 설정
    baseURL(value: string): APIBuilder {
      this._instance.baseURL = value;
      return this;
    }
    // 요청 헤더 설정
    headers(value: Record<string, string>): APIBuilder {
      this._instance.headers = value;
      return this;
    }
    // 요청 타임아웃 설정
    timeout(value: number): APIBuilder {
      this._instance.timeout = value;
      return this;
    }
    // url 쿼리 파라미터 설정
    params(value: Record<string, any>): APIBuilder {
      this._instance.params = value;
      return this;
    }
    // 요청 본문 데이터 설정 : post, put 요청에서 사용
    data(value: unknown): APIBuilder {
      this._instance.data = value;
      return this;
    }
    // 요청에 인증 정보 포함할지 여부 설정
    withCredentials(value: boolean): APIBuilder {
      this._instance.withCredentials = value;
      return this;
    }
    ```
    
- 전체 코드
    
    ```tsx
    import { API } from "./API";
    export class APIBuilder {
      private _instance: API;
    
      constructor(method: string, url: string, data?: unknown) {
        this._instance = new API(method, url);
        this._instance.baseURL = apiHost;
        this._instance.data = data;
        this._instance.headers = {
          "Content-Type": "application/json; charset=utf-8",
        };
        this._instance.timeout = 5000;
        this._instance.withCredentials = false;
      }
    
      static get = (url: string) => new APIBuilder("GET", url);
      static post = (url: string, data: unknown) => new APIBuilder("POST", url, data);
      static put = (url: string, data: unknown) => new APIBuilder("PUT", url, data);
      static delete = (url: string) => new APIBuilder("DELETE", url);
      
      baseURL(value: string): APIBuilder {
        this._instance.baseURL = value;
        return this;
      }
    
      headers(value: Record<string, string>): APIBuilder {
        this._instance.headers = value;
        return this;
      }
    
      timeout(value: number): APIBuilder {
        this._instance.timeout = value;
        return this;
      }
    
      params(value: Record<string, any>): APIBuilder {
        this._instance.params = value;
        return this;
      }
    
      data(value: unknown): APIBuilder {
        this._instance.data = value;
        return this;
      }
    
      withCredentials(value: boolean): APIBuilder {
        this._instance.withCredentials = value;
        return this;
      }
    
      build(): API {
        return this._instance;
      }
    }
    
    ```
    

사용 예제

```tsx
const fetchJobNameList = async (name?: string, size?: number) => {
  const api = APIBuilder.get("/apis/web/jobs")
    .withCredentials(true) // 401 에러 발생 시 인터셉터로 자동 처리
    .params({ name, size }) // 쿼리 파라미터 추가
    .build(); // API 객체 생성

  const { data } = await api.call<ResponseJobNameListResponse>();
  return data;
};
```

### ❕APIBuilder 클래스의 장단점

보일러 플레이트 코드가 많다는 단점. 하지만 옵션이 다양한 경우 인터셉터를 설정값에 따라 적용하고, 필요 없는 인터셉터를 선택적으로 사용할 수 있다는 장점

> 보일러플레이트가 뭔가요?
> 

## 7.1.5 API 응답 타입 지정하기

1. 공통 Response 타입 정의

```tsx
// 공통 Response 타입 정의
interface Response<T> {
  data: T; // data가 필수 속성
  status: string;
  serverDateTime: string;
  errorCode?: string; // FAIL, ERROR
  errorMessage?: string; // FAIL, ERROR
}
```

```tsx
const apiRequester = axios.create({ baseURL: "https://api.baemin.com", ...defaultConfig });
```

```tsx
// 장바구니 조회 API 요청
const fetchCart = (): AxiosPromise<Response<FetchCartResponse>> =>
  apiRequester.get<Response<FetchCartResponse>>("cart");
  
// 장바구니 추가 API 요청
const postCart = (postCartRequest: PostCartRequest): AxiosPromise<Response<PostCartResponse>> =>
  apiRequester.post<Response<PostCartResponse>>("cart", postCartRequest);
```

### ❓Response 타입을 apiRequester 밖에서 처리해야 하는 이유

모든 api 응답이 동일한 구조를 가지지 않을 수 있다. Post(create)나 Put(update) 요청은 응답 데이터가 없을 수 있다.(void) 이런 경우에 타입 충돌이 발생하며 에러가 발생한다. 

```tsx
// 장바구니 업데이트 API 요청
const updateCart = (updateCartRequest: UpdateCartRequest): AxiosPromise<Response<FetchCartResponse>> =>
  apiRequester.put<Response<FetchCartResponse>>("cart", updateCartRequest);
// 공통 Response 타입을 사용했는데 이 경우 data가 필수 속성인데 put은 그렇지 않을 수 있다.
```

- data를 옵셔널 처리하면 안되냐? > 그러면 데이터를 반드시 받아야 하는 요청에서 데이터가 안 올 경우의 에러 핸들링이 제대로 안됨.

1. 다른 api 서버로 전달만 할 때 응답값

**단순 전달용 데이터는 어떻게 표현할까?**

- 해당 값에 어떤 응답이 들어있는지 알 필요가 없다면 “unknown” 타입으로 처리한다.

```tsx
interface response{
	data : {
		cartItems : CartItem[];
		forPass : unknown; // 실제로 사용하지 않는 값.
	};
}
// 만약 실제로 사용해야 하는 값이라면 타입 단언하고 사용하는 것이 좋다
type ForPass = {
  type: "A" | "B" | "C";
};

const isTargetValue = (data: { forPass: unknown }): boolean => {
  return (data.forPass as ForPass).type === "A";
};

```

## 7.1.6 뷰 모델 사용하기

API 응답은 자주 변한다. 뷰 모델을 사용해 API 변경에 따른 범위를 한정해줘야 한다.

> **특정 객체 리스트를 조회해 리스트 각각의 내용과 리스트 전체 길이를 보여주는 화면**
> 

```tsx
interface ListItem { id: number; title: string; description: string; }
interface ListFetchFilter { search?: string; page?: number; limit?: number; }

interface ListResponse {
  items: ListItem[];
}

const fetchList = async (filter?: ListFetchFilter): Promise<ListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get("/apis/get-list-summaries")
    .call<Response<ListResponse>>();

  return { data };
};
```

! 실제 비동기 함수는 컴포넌트 내부에서 직접 호출되지 않는다.

- 왜 이 코드를 “컴포넌트 내부에서 호출”했다고 볼 수 있는가?
    
    비동기 호출이 컴포넌트의 주요 렌더링 로직에 포함된 상태이기 때문이다. 렌더링과 api 호출 간의 역할 분리가 모호하다. fetchList가 외부에서 호출되지만 비동기 호출의 전체 흐름이 컴포넌트 내부에서 관리되고 있다.
    
    api 호출 로직을 서비스 계층으로 분리해야 한다.
    
    > ❓**해당 코드의 서비스 계층을 잘 분리한 코드 예제는 무엇인가? > 7.2에서 더 다룸**
    > 

```tsx
const ListPage: React.FC = () => {
  const [totalItemCount, setTotalItemCount] = useState(0);
  const [items, setItems] = useState<ListItem[]>([]);

  useEffect(() => {
    // 예시를 위한 API 호출과 then 구문
    fetchList(filter).then(({ items }) => {
      setTotalItemCount(items.length);
      setItems(items);
    });
  }, []);

  return (
    <div>
      <Chip label={totalItemCount} />
      <Table items={items} />
    </div>
  );
};
```

API 응답의 items의 이름을 수정하고 싶다는 요구사항을 처리할 때, 기존 컴포넌트마다 이름을 바꿔줘야 하는 문제를 방지하기 위해 “역할을 분리하고 뷰 모델을 도입”하는 방법으로 해결할 수 있다.

### 🔹 뷰 모델 도입

1. 뷰 모델 구현

api 응답(JobListResponse)을 받아 컴포넌트가 사용하기 더 좋은 형태로 변환한다. 

```tsx
class JobList {
  readonly totalItemCount: number;
  readonly items: JobListItemResponse[];

  constructor({ jobItems }: JobListResponse) {
	  // 필드 이름 더 명확한 의미로 재정의
    this.totalItemCount = jobItems.length;
    this.items = jobItems;
  }
}
```

1. 서비스 계층에서 뷰 모델 반환

컴포넌트가 뷰모델인 JobList에 의존하도록 설계

> **API 응답이 변경되어도 뷰모델의 로직만 수정하면 컴포넌트는 영향 받지 않는다.**
> 

```tsx
const fetchJobList = async (
...
  return new JobList(data); // JobList 객체 반환
};
```

전체코드

```tsx
// 기존 ListResponse에 더 자세한 의미를 담기 위한 변화
interface JobListItemResponse {
  name: string;
}

interface JobListResponse {
  jobItems: JobListItemResponse[];
}

class JobList {
  readonly totalItemCount: number;
  readonly items: JobListItemResponse[];
  constructor({ jobItems }: JobListResponse) {
    this.totalItemCount = jobItems.length;
    this.items = jobItems;
  }
}

const fetchJobList = async (
  filter?: ListFetchFilter
): Promise<JobListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get("/apis/get-list-summaries")
    .call<Response<JobListResponse>>();

  return new JobList(data);
};
```

### ❕뷰 모델 방식의 장단점

장점 : API 응답구조가 바뀌어도 뷰모델만 수정함으로써 처리할 수 있다. 유지보수가 쉬워짐

단점 : 추상화 레이어 추가는 코드를 복잡하게 하고 레이어를 관리하고 개발하는데도 비용이 든다. 위의 경우에도 뷰모델을 추가하면서 추가적인 타입들이 선언되었다. API마다 뷰모델을 만들어준다면 API가 20개면 20개의 뷰모델이 추가적으로 생기고 또 그에 맞는 파생 타입들도 더 생겨난다. API가 많아질수록 이러한 타입 선언이 기하급수적으로 늘어날 수 있다.

> API 응답 타입 (**`JobListResponse`**), 뷰 모델 타입 (**`JobList`**), 그리고 각 필드의 하위 타입까지 추가적으로 정의됐다.
> 
> 
> ```tsx
> class JobListItem {
>   constructor(item: JobListItemResponse) {
>     /* JobListItemResponse에서 JobListItem 객체로 변환해주는 코드 */
>   }
> }
> ```
> 

또 다른 문제는 구체적인 의미를 가진 필드명으로 바꿀 경우 API 응답 필드명과 뷰 모델 필드명의 다름으로 인해 혼란이 생길 수 있다.

### 뷰모델을 사용하는 장점을 살릴 수 있는 절충안

> 필요한 곳에만 뷰 모델을 부분적으로 만들어 사용하기, 백엔드와 클라이언트 개발자가 충분히 소통하여 **API 응답 변화를 최소화**하거나, 뷰 모델에 필드를 추가하는 대신 **getter 함수를 추가**하여 어떤 값이 뷰 모델에 추가된 값인지 명확히 알 수 있도록 처리하는 방법을 고려할 수 있습니다.
> 

## 7.1.7  Superstruct를 사용해 런타임에서 응답 타입 검증하기

개요 : 개발 단계에서는 API 응답 형식이 자주 바뀐다. 또한 응답 값의 타입이 string이어야 하는데 number가 들어오는 것처럼 잘못된 타입이 전달될 수 있다. 그러나 타입스크립트는 정적 검사 도구로 런타임에서 발생하는 오류는 찾아낼 수 없다. 런타임에 API 응답의 타입 오류를 방지하려면 Superstruct 같은 라이브러리를 사용한다.

> 런타임 응답 타입 오류 방지 : Superstruct
> 

```tsx
- Superstruct를 사용하여 인터페이스 정의와 자바스크립트 데이터의 유효성 검사를 쉽게 할 수 있다.
- Superstruct는 런타임에서의 데이터 유효성 검사를 통해 개발자와 사용자에게 자세한 런타임 에러를 보여주기 위해 고안되었다.
```

### 🔸 Superstruct 사용방법

```tsx
import {
  assert,
  is,
  validate,
  object,
  number,
  string,
  array,
} from "superstruct";

const Article = object({
  id: number(),
  title: string(),
  tags: array(string()),
  author: object({
    id: number(),
  }),
});

const data = {
  id: 34,
  title: "Hello World",
  tags: ["news", "features"],
  author: {
    id: 1,
  },
};

// 데이터 유효성 검사 돕는 모듈
assert(data, Article); // 유효하지 않으면 에러 리턴
is(data, Article); // 유효성에 따라 boolean 리턴
validate(data, Article); 
// 에러 - [error, data] 형태 튜플 리턴
// 유효 - [undefined, data]
```

- 아티클은 object 객체 형태를 가진 무언가

> 실제 Superstruct 내부로직에서 반환되는 타입은 object()의 반환결과를 한 번 더 감싸서 내려온다.
> 
- 아티클은 id, title, tags, autohr 명세를 가진 스키마이다.
- 세 모듈의 공통점은 같은 파라미터를 받아 데이터가 스키마와 부합하는지 검사한다.
    - 차이점은 모듈마다 데이터의 유효성을 다르게 접근하고 리턴 값의 형태가 다르다.

### 🔸타입스크립트와 Superstruct

1. 기존 타입스크립트 문법 사용

```tsx
import { Infer, number, object, string } from "superstruct";

// 슈퍼스트럭, 위에서의 아티클, 데이터 명세를 가진 스키마
const User = object({
  id: number(),
  email: string(),
  name: string(),
});

// Infer로 타입 생성, 슈퍼스트럭 구조체 User 참조
type User = Infer<typeof User>;
```

Infer를 사용해서 슈퍼스트럭 구조체와 동일한 구조의 타입을 생성할 수 있다. 중복 제거 효과, 유지보수성! 

- Infer를 사용하지 않으면
    
    ```tsx
    // 중복된 정의 (좋지 않은 사례)
    import { object, number, string } from "superstruct";
    
    const UserStruct = object({
      id: number(),
      email: string(),
      name: string(),
    });
    
    // 동일한 구조를 다시 작성
    type User = {
      id: number;
      email: string;
      name: string;
    };
    
    ```
    

1. Superstruct의 “assert” 메서드를 통해 타입 매칭을 확인

```tsx
// 타입스크립트 문법
type User = { id: number; email: string; name: string };

import { assert } from "superstruct";

function isUser(user: User) {
  assert(user, User); // 유효성 검사 통과 못하면 에러발생
  console.log("적절한 유저입니다.");
}
```

1. 적용

```tsx
const user_A = {
  id: 4,
  email: "test@woowahan.email",
  name: "woowa",
};

isUser(user_A); // 출력 : 적절한 유저입니다.
```

```tsx
const user_B = {
  id: 5,
  email: "wrong@woowahan.email",
  name: 4,
};

isUser(user_B); 
// 런타임 에러 :  Argument of type '{ id: number; email: string; name: number; }' 
// is not assignable to parameter of type '{ id: number; email: string; name: string; }'
```

## 7.1.8 실제 API 응답 시의 Superstruct 활용 사례

```tsx
interface ListItem {
  id: string;
  content: string;
}

interface ListResponse {
  items: ListItem[];
}
const fetchList = async (filter?: ListFetchFilter): Promise<ListResponse> => {
  const { data } = await api
    .params({ ...filter })
    .get("/apis/get-list-summaries")
    .call<Response<ListResponse>>();

  return { data };
};
```

1. **타입스크립트는 컴파일타임에 타입을 검증한다. api 응답은 런타임에서 확인할 수 있으므로 api 응답 관련 타입 검사를 타입스크립트에서는 확인할 수 없다.**
2. 이 때 슈퍼스트럭을 사용할 수 있다.

### 🔸Superstruct 적용해보기

```tsx
// Superstruct로 데이터 구조 정의
const ListItem = object({
  id: string(),
  content: string(),
});
// 유효성 검증 함수
function isListItem(listItems: ListItem[]) {
  listItems.forEach((listItem) => assert(listItem, ListItem));
}
```

```tsx
const fetchList = async (filter?: ListFetchFilter): Promise<ListItem[]> => {
	// ... 생략
  data.items.forEach((item) => assert(item, ListItem));
  return data.items; // 검증된 아이템 반환
};

```

# 7.2 API 상태 관리하기

컴포넌트 내에서 비동기 함수를 직접 호출하지 않는다. 비동기 API를 호출하기 위해서는 API의 성공, 실패에 따른 상태가 관리되어야 하므로 상태 관리 라이브러리의 액션이나 훅과 같이 재정의된 형태를 사용해야 한다.

# 7.2.1 상태 관리 라이브러리에서 호출하기

상태 관리 라이브러리의 비동기 함수들은 서비스 코드를 사용해서 비동기 상태를 변화시킬 수 있는 함수를 제공한다. 컴포넌트는 이러한 함수를 사용하여 상태를 구독하며, 상태가 변경될 때 컴포넌트를 다시 렌더링하는 방식으로 동작한다.

## 🔹Redux 예제

dispatch : 가져온 데이터를 리덕스 스토어 상태에 반영 > 상태관리

```tsx
import { useEffect } from "react";
import { useDispatch, useSelector } from "react-redux";

export function useMonitoringHistory() {
  const dispatch = useDispatch();
  // 전역 Store 상태(RootState)에서 필요한 데이터만 가져온다
  const searchState = useSelector(
    (state: RootState) => state.monitoringHistory.searchState
  );
  // history 내역을 검색하는 함수, 검색 조건이 바뀌면 상태를 갱신하고 API를 호출한다
  const getHistoryList = async (
    newState: Partial<MonitoringHistorySearchState>
  ) => {
    const newSearchState = { ...searchState, ...newState };
    dispatch(monitoringHistorySlice.actions.changeSearchState(newSearchState));
    // 비동기 API 호출하기 dispatch(monitoringHistorySlice.actions.fetchData(response));
		const response = await getHistories(newSearchState); 
		dispatch(monitoringHistorySlice.actions.fetchData(response));  
  };

  return { searchState, getHistoryList };
}
```

> 스토어에서 getHistories API만 호출하고, 그 결과를 받아와서 상태를 업데이트하는(상태에 저장하는) 일반적인 방식으로 사용할 수 있다. 그러나 앞의 예시와 같이 getHistoryList 함수에서는 dispatch 코드를 제외하더라도 다음과 같이 API 호출과 상태 관리 코드를 작성해야 한다.
> 

### ▶️  API 호출 상태 관리 : Redux, Axios 인터셉터

리덕스와 axios 인터셉터를 사용해 API 호출 상태를 중앙에서 관리한다. API 호출 전후의 상태를 리덕스 스토어에 저장해 애플리케이션 전반에서 API 로딩 상태를 추적할 수 있다.

```tsx
enum ApiCallStatus {
  Request, // 0, API 요청중, 로딩
  None,    // 1, API 요청없음, 로딩 종료
}
```

**`setAxiosInterceptor` : 리턱스 스토어를 매개변수로 받아 axios 인터셉터 설정. 인터셉터를 통해 API 요청 전, 응답 성공시, 응답 실패시 해당하는 로직을 실행**

1. 요청 인터셉터
- 리덕스의 “setApiCall” 액션을 디스패치해 API 호출 상태를 리퀘스트로 설정
- 요청의 url과 http 메서드 정보를 `urlInfo`로 저장

```tsx
API.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    const { params, url, method } = config;
    store.dispatch(
      setApiCall({
      // API 상태 저장을 위해 redux reducer `setApiCall` 함수를 사용한다 
      // 상태가 `요청됨`인 경우 API가 Loading 중인 상태
        status: ApiCallStatus.Request, // API 호출 상태: 요청됨=로딩 중
        urlInfo: { url, method }, // 요청 URL과 메서드 저장
      })
    );
    return config; // 요청 진행
  },
  (error) => Promise.reject(error) // 요청 에러 처리
);
```

1. 응답 인터셉터 - 성공

setApiCall 액션 디스패치해서 “호출 상태를 none으로 한다.”

```tsx
API.interceptors.response.use(
  (response: AxiosResponse) => {
    const { method, url } = response.config;
    store.dispatch(
      setApiCall({
        status: ApiCallStatus.None, // API 호출 상태: 로딩 종료
        urlInfo: { url, method }, // 요청 정보 저장
      })
    );
    return response?.data?.data || response?.data; // 응답 데이터 반환
  },

```

1. 응답 인터셉터 - 실패

위랑 같은데 오류를 그대로 반환한다.

```tsx
(error: AxiosError) => {
  const {
    config: { url, method },
  } = error;
  store.dispatch(
    setApiCall({
      status: ApiCallStatus.None, // API 호출 상태: 로딩 종료
      urlInfo: { url, method }, // 요청 정보 저장
    })
  );
  return Promise.reject(error); // 오류 전달
}

```

- 전체코드
    
    ```tsx
    enum ApiCallStatus {
      Request,
      None,
    }
    
    const API = axios.create();
    
    const setAxiosInterceptor = (store: EnhancedStore) => {
      API.interceptors.request.use(
        (config: AxiosRequestConfig) => {
          const { params, url, method } = config;
          store.dispatch(
            // API 상태 저장을 위해 redux reducer `setApiCall` 함수를 사용한다 // 상태가 `요청됨`인 경우 API가 Loading 중인 상태
            setApiCall({
              status: ApiCallStatus.Request, // API 호출 상태를 `요청됨`으로 변경
              urlInfo: { url, method },
            })
          );
          return config;
        },
        (error) => Promise.reject(error)
      );
      // onSuccess 시 인터셉터로 처리한다
      API.interceptors.response.use(
        (response: AxiosResponse) => {
          const { method, url } = response.config;
          store.dispatch(
            setApiCall({
              status: ApiCallStatus.None, // API 호출 상태를 `요청되지 않음`으로 변경
              urlInfo: { url, method },
            })
          );
          return response?.data?.data || response?.data;
        },
        (error: AxiosError) => {
          const {
            config: { url, method },
          } = error;
          store.dispatch(
            setApiCall({
              status: ApiCallStatus.None, // API 호출 상태를 `요청되지 않음`으로 변경
              urlInfo: { url, method },
            })
          );
          return Promise.reject(error);
        }
      );
    };
    ```
    

<aside>
📫

핵심

- API를 호출할 때, 호출한 뒤, 호출하고 에러 발생했을 때 3가지 경우 각각에 setApiCall을 호출해서 리덕스 스토어의 api콜 상태를 업데이트한다.
</aside>

리덕스는 비동기 상태가 아닌 전역상태를 위해 만들어진 라이브러리이기 때문에 “미들웨어”라고 불리는 여러 도구를 도입해 비동기 상태를 관리한다. 따라서 보일러플레이트 코드가 많아지는 등 간편하게 비동기 상태를 관리하기 어려운 상황도 발생한다.

반면 MobX 같은 라이브러리에서는 이러한 불편함을 개선하기 위해 비동기 콜백함수를 분리해 액션으로 만들거나 “runInAction”과 같은 메서드를 사용해 상태 변경을 처리한다. 또 어싱크/어웨잇 이나 flow 같은 비동기 상태 관리를 위한 기능도 있어 더욱 간편하게 사용할 수 있다.

## 🔹 mobX 비동기 상태 관리 코드 예시

MobX를 활용하여 **`JobStore`**와 **`Store`** 두 개의 상태 관리 클래스를 정의한 예제입니다. 

각 클래스는 **비동기 데이터 처리** 및 **상태 관리**를 담당하며, MobX의 주요 기능인 **`makeAutoObservable`**과 **`runInAction`**을 사용합니다.

```tsx
type LoadingState = "PENDING" | "DONE" | "ERROR";
```

1. JobStore : 기본 구조

“makeAutoObservable”는 mobX에서 클래스를 관찰 가능한 상태로 만든다. 클래스 내 모든 속성과 메서드를 자동으로 관찰 가능하게 설정한다.

```tsx
class JobStore {
  job: Job[] = []; // Job 목록을 저장하는 배열

  constructor() {
    makeAutoObservable(this); // MobX의 반응성을 자동으로 설정
  }
}
```

1. Store : 복잡한 상태 관리
- “fetchJobList”는 비동기 작업을 수행, api 호출로 job 데이터 가져온다. 호출 결과에 따라 state와 데이터 업데이트
- “makeAutoObservable”로 job, state, errorMsg 속성 관찰 가능해짐
- “runInAction”: mobX에서 비동기 작업 상태 업데이트를 안전하게 처리하기 위해 액션으로 감싸야 한다. 비동기 작업 이후 상태 변경 코드도 포함한다.

```tsx
class Store {
  job: Job[] = []; // Job 목록
  state: LoadingState = "PENDING"; // 로딩 상태 (PENDING, DONE, ERROR)
  errorMsg = ""; // 에러 메시지

  constructor() {
    makeAutoObservable(this); // MobX 반응성 설정
  }

  async fetchJobList() {
    this.job = []; // 기존 Job 목록 초기화
    this.state = "PENDING"; // 로딩 상태 초기화
    this.errorMsg = ""; // 에러 메시지 초기화

    try {
      const projects = await fetchJobList(); // API 호출 (비동기)
      runInAction(() => {
        this.projects = projects; // 성공 시 프로젝트 데이터 저장
        this.state = "DONE"; // 로딩 상태 완료
      });
    } catch (e) {
      runInAction(() => {
        this.state = "ERROR"; // 실패 시 상태를 ERROR로 설정
        this.errorMsg = e.message; // 에러 메시지 저장
      });
    }
  }
}

```

- 전체코드
    
    ```tsx
    import { runInAction, makeAutoObservable } from "mobx";
    import type Job from "models/Job";
    
    class JobStore {
      job: Job[] = [];
      constructor() {
        makeAutoObservable(this);
      }
    }
    
    type LoadingState = "PENDING" | "DONE" | "ERROR";
    
    class Store {
      job: Job[] = [];
      state: LoadingState = "PENDING";
      errorMsg = "";
    
      constructor() {
        makeAutoObservable(this);
      }
    
      async fetchJobList() {
        this.job = [];
        this.state = "PENDING";
        this.errorMsg = "";
        try {
          const projects = await fetchJobList();
          runInAction(() => {
            this.projects = projects;
            this.state = "DONE";
          });
        } catch (e) {
          runInAction(() => {
            this.state = "ERROR";
            this.errorMsg = e.message;
          });
        }
      }
    }
    ```
    

## ❓API 상태를 관리할 때 상태 관리 라이브러리를 사용했을 때의 문제점

1. 비동기 처리 함수를 호출하기 위해 액션을 추가해야 한다. 이에 따라 관련 상태나 스토어가 계속 늘어나게 된다. 
2. 전역 상태 관리자는 모든 비동기 상태에 접근하고 변경할 수 있다. 그 과정에서 의도치 않은 상태 변경 가능성이 높아진다.
3. 여러 컴포넌트가 동일한 상태를 구독할 경우 중복된 API 호출이 발생할 수  있고 한 컴포넌트의 상태 변경이 다른 컴포넌트에 영향을 미칠 수 있다.

# 7.2.2 훅으로 호출하기

데이터 패칭 라이브러리는 훅을 사용해 비동기 상태 관리 문제를 해결하는데 효과적이다. 대표적으로 react-query나 useSwr이 훅을 사용한 방법이다.

이 방법을 사용하면 캐시를 사용해 비동기 함수를 호출해서 중복 호출을 줄일 수 있고, 상태 관리 라이브러리에서발생했던 의도치 않은 상태 변경 방지에 도움이 된다.

- 상태 변경이 전역 상태 관리, 스토어에서 이루어지는 대신 데이턴는 컴포넌트 수준에서 훅을 통해 관리된다. 각 비동기 상태가 독립적으로 상호영향을 끼치지 않는다.

## 🔹 react-query 예시 : 캐싱과 캐싱 무력화

Job 목록을 불러오는 훅과 Job을 업데이트하는 훅을 사용합니다.

- Job이 업데이트되면 다시 API를 호출해야 한다.
- onSuccess 옵션의 invalidateQueries를 사용하면 특정 키의 api를 유효하지 않은 상태로 설정할 수 있다.
1. Job 목록 불러오는 훅 `useFetchJobList`

```tsx
const useFetchJobList = () => {
  return useQuery(
    ["fetchJobList"], // Query Key: fetchJobList
    async () => {
      const response = await JobService.fetchJobList(); // API 호출
      
      // 뷰 모델
      return new JobList(response); // API 응답을 JobList 객체로 변환
    }
  );
};

```

1. Job을 업데이트하는 훅 `useUpdateJob`
- **`useMutation`** 은 데이터 변화를 처리하는데 사용되는 훅이다.
    - 매개변수 : (Mutation Key: 작업 식별자), (API 호출), (성공/실패시 추가 동작)
- 업데이트가 성공되면 기존의 캐싱된 데이터를 무효화해야 한다! **`invalidateQueries` 사용**

```tsx
const useUpdateJob = (
  id: number,
  { onSuccess, ...options }: UseMutationOptions<void, Error, JobUpdateFormValue>
): UseMutationResult<void, Error, JobUpdateFormValue> => {
  const queryClient = useQueryClient();

  return useMutation(
    ["updateJob", id], // Mutation Key: updateJob와 id 조합
    async (jobUpdateForm: JobUpdateFormValue) => {
      await JobService.updateJob(id, jobUpdateForm); // API 호출로 Job 업데이트
    },
    {
      onSuccess: (
        data: void, // updateJob은 성공 시 반환값 없음
        values: JobUpdateFormValue,
        context: unknown
      ) => {
        // 성공 시 Job 목록 캐시 무효화
        queryClient.invalidateQueries(["fetchJobList"]);
        // 사용자 정의 성공 콜백 실행
        onSuccess && onSuccess(data, values, context);
      },
      ...options, // 추가 옵션 (에러 처리 등)
    }
  );
};

```

- 전체 코드
    
    ```tsx
    // Job 목록을 불러오는 훅
    const useFetchJobList = () => {
      return useQuery(["fetchJobList"], async () => {
        const response = await JobService.fetchJobList(); // View Model을 사용해서 결과
        return new JobList(response);
      });
    };
    
    // Job 1개를 업데이트하는 훅
    const useUpdateJob = (
      id: number,
      // Job 1개 update 이후 Query Option
      { onSuccess, ...options }: UseMutationOptions<void, Error, JobUpdateFormValue>
    ): UseMutationResult<void, Error, JobUpdateFormValue> => {
      const queryClient = useQueryClient();
    
      return useMutation(
        ["updateJob", id],
        async (jobUpdateForm: JobUpdateFormValue) => {
          await JobService.updateJob(id, jobUpdateForm);
        },
        {
          onSuccess: (
            data: void, // updateJob의 return 값은 없다 (status 200으로만 성공 판별) values: JobUpdateFormValue,
            context: unknown
          ) => {
            // 성공 시 ‘fetchJobList’를 유효하지 않음으로 설정 queryClient.invalidateQueries(["fetchJobList"]);
            onSuccess && onSuccess(data, values, context);
          },
          ...options,
        }
      );
    };
    ```
    

## 🔹 react-query : 컴포넌트가 최신 상태를 유지하도록 하기

폴링이나 웹소켓 등의 방법을 사용해야 한다.

> 폴링은 클라이언트가 주기적으로 서버에 요청을 보내 데이터를 업데이트하는 것이다. 클라이언트는 일정한 시간 간격으로 서버에 요청을 보내고, 서버는 해당 요청에 대해 최신 상태의 데이터를 응답으로 보내주는 방식을 말한다.
> 

### 간단 폴링 방식 예제

리액트 쿼리를 사용해 Job 목록을 주기적으로, 여기서는 30초마다 갱신=폴링한다.

```tsx
const JobList: React.FC = () => {
  // 비동기 데이터를 필요한 컴포넌트에서 자체 상태로 저장
  const {
    isLoading,
    isError,
    error,
    refetch,
    data: jobList,
  } = useFetchJobList(); // 앞에서 정의

  // 간단한 Polling 로직, 실시간으로 화면이 갱신돼야 하는 요구가 없어서 
  // 30초 간격으로 갱신한다
  useInterval(() => refetch(), 30000);

  // Loading인 경우에도 화면에 표시해준다
  if (isLoading) return <LoadingSpinner />;

  // Error에 관한 내용은 11.3 API 에러 핸들링에서 더 자세하게 다룬다
  if (isError) return <ErrorAlert error={error} />;

  return (
    <>
      {jobList.map((job) => (
        <Job job={job} />
      ))}
    </>
  );
};
```

## ❕상태 관리 라이브러리에 대한 고민이 필요하다.

사내에서 리덕스나 mobX 같은 전역 상태 관리 라이브러리에서 리액트 쿼리로 변경하고자 하는 시도가 이뤄지고 있다.

**왜?** 

비동기 상태 변경을 전역 상태 관리 스토어에서 관리하려면 액션이랑 전역 상태도 복잡해진다. 

에러 발생, 로딩 중과 같은 상태는 전역으로 관리할 필요가 거의 없다. 이를 전역에서 관리하면 오히려 컴포넌트와 결합도와 복잡도가 높아져 유지보수가 어려울 수 있다. 

**그렇다면 리액트 쿼리가 최선인가?**

가장 많이 활용되지만 기본적으로 리액트 쿼리는 상태 관리 라이브러리가 아니다. 어떤 상태 관리 라이브러리를 선택할지는 상황에 따라 판단해야 한다.

ㅠㅠ