# 7. 비동기 호출

이 장에서는 TS에서 비동기 요청을 어떻게 처리하고 관리하는지를 다룰 것이다.

## 7.1 API 요청

### 1. fetch로 API 요청하기

JS에서 제공하는 내장 라이브러리인 `fetch` 함수를 이용해서 비동기 API 요청을 보낼 수 있다.

```ts
import React, { useEffect, useState } from "react";

const CartBadge: React.FC = () => {
  const [cartCount, setCartCount] = useState(0);

  useEffect(() => {
    fetch("https://api.baemin.com/cart")
      .then((response) => response.json())
      .then(({ cartItem }) => {
        setCartCount(cartItem.length);
      });
  }, []);

  return <>{/*  cartCount 상태를 이용하여 컴포넌트 렌더링 */}</>;
};
```

하지만 코드의 유지보수를 용이하게 하게 위해서는 서비스 레이어로 API 요청하는 부분을 분리하는 것이 좋을 것 같다.

&nbsp;

### 2. 서비스 레이어로 분리하기

```ts
async function fetchCart() {
  const controller = new AbortController();

  const timeoutId = setTimeout(() => controller.abort(), 5000);

  const response = await fetch("https://api.baemin.com/cart", {
    signal: controller.signal,
  });

  clearTimeout(timeoutId);

  return response;
}
```

`fetch`를 호출하는 함수를 따로 분리했다. 하지만 단순히 `fetch` 함수를 분리하는 것만으로는 추후 다양한 API 요청 정책(타임아웃 설정, 쿼리 매개변수, 커스텀 헤더 추가, 쿠키를 읽어 토큰을 집어넣기 등)을 하나하나 모든 `fetch` 함수에 추가하는 것은 상당히 번거로울 것이다.

&nbsp;

### 3. Axios 활용하기

Axios를 사용하면 내장 기능을 통해 타임아웃 같은 API 설정을 쉽게 해줄 수 있다.

```ts
import axios, { AxiosInstance, AxiosPromise } from "axios";

export type FetchCartResponse = unknown;
export type PostCartRequest = unknown;
export type PostCartResponse = unknown;

export const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000,
});

export const fetchCart = (): AxiosPromise<FetchCartResponse> =>
  apiRequester.get<FetchCartResponse>("cart");

export const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<PostCartResponse> =>
  apiRequester.post<PostCartResponse>("cart", postCartRequest);
```

API 엔트리가 2개 이상일 경우, 각 서버의 기본 URL을 호출하도록 인스턴스를 각각 하나씩 만들어 주어야 한다.

```ts
import axios, { AxiosInstance } from "axios";

const defaultConfig = {};

const apiRequester: AxiosInstance = axios.create(defaultConfig);
const orderApiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.order/",
  ...defaultConfig,
});
const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: "https://cart.baemin.order/",
  ...defaultConfig,
});
```

&nbsp;

### 4. Axios 인터셉터 사용하기

각 requester별로 헤더를 다르게 설정해줘야 할 필요가 있을 수도 있다. 그럴 땐 인터셉터를 사용하여 헤더를 커스텀해줄 수 있다.

```ts
import axios, { AxiosInstance, AxiosRequestConfig, AxiosResponse } from "axios";

const getUserToken = () => "";
const getAgent = () => "";
const getOrderClientToken = () => "";
const orderApiBaseUrl = "";
const orderCartApiBaseUrl = "";
const defaultConfig = {};
const httpErrorHandler = () => {};

const apiRequester: AxiosInstance = axios.create({
  baseURL: "https://api.baemin.com",
  timeout: 5000,
});

const setRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
  const config = requestConfig;
  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    user: getUserToken(),
    agent: getAgent(),
  };
  return config;
};

const setOrderRequestDefaultHeader = (requestConfig: AxiosRequestConfig) => {
  const config = requestConfig;
  config.headers = {
    ...config.headers,
    "Content-Type": "application/json;charset=utf-8",
    "order-client": getOrderClientToken(),
  };
  return config;
};

// 인터셉터 기능을 사용해 header를 설정하는 기능을 넣거나 에러를 처리할 수 있다
apiRequester.interceptors.request.use(setRequestDefaultHeader);

const orderApiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
});
// 인터셉터로 기본 apiRequester와는 다른 header를 설정해주고 있다.
orderApiRequester.interceptors.request.use(setOrderRequestDefaultHeader);
// 인터셉터를 사용해 httpError 같은 API 에러를 처리할 수도 있다
orderApiRequester.interceptors.response.use(
  (response: AxiosResponse) => response,
  httpErrorHandler
);

const orderCartApiRequester: AxiosInstance = axios.create({
  baseURL: orderCartApiBaseUrl,
  ...defaultConfig,
});
orderCartApiRequester.interceptors.request.use(setRequestDefaultHeader);
```

&nbsp;

다양한 옵션의 인터셉터 생성을 위해 빌더 패턴을 사용하기도 한다.

```ts
import axios, { AxiosPromise } from "axios";

export type HTTPMethod = "GET" | "POST" | "PUT" | "DELETE";
export type HTTPHeaders = any;
export type HTTPParams = unknown;

class API {
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
    // 만약 `withCredential`이 설정된 API라면 아래 같이 인터셉터를 추가하고, 아니라면 인터셉터를 사용하지 않음
    if (this.withCredentials) {
      http.interceptors.response.use(
        (response) => response,
        (error) => {
          if (error.response && error.response.status === 401) {
            /* 에러 처리 진행 */
          }
          return Promise.reject(error);
        }
      );
    }
    return http.request({ ...this });
  }
}

export default API;
```

&nbsp;

기본 API 클래스로 실제 호출 부분을 작성하고, 이 API를 호출하기 위한 Wrapper를 빌더 패턴으로 만든다.

```ts
import API, { HTTPHeaders, HTTPMethod, HTTPParams } from "./7.1.4-2";

const apiHost = "";

class APIBuilder {
  // 위에 작성한 API 클래스를 활용
  private _instance: API;

  constructor(method: HTTPMethod, url: string, data?: unknown) {
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

  static put = (url: string, data: unknown) => new APIBuilder("PUT", url, data);

  static post = (url: string, data: unknown) =>
    new APIBuilder("POST", url, data);

  static delete = (url: string) => new APIBuilder("DELETE", url);

  baseURL(value: string): APIBuilder {
    this._instance.baseURL = value;
    return this;
  }

  headers(value: HTTPHeaders): APIBuilder {
    this._instance.headers = value;
    return this;
  }

  timeout(value: number): APIBuilder {
    this._instance.timeout = value;
    return this;
  }

  params(value: HTTPParams): APIBuilder {
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

export default APIBuilder;
```

빌더 클래스를 사용하는 예시는 다음과 같다.

```ts
import APIBuilder from "./7.1.4-3";

type Response<T> = { data: T };
type JobNameListResponse = string[];

// axios 객체 생성
const fetchJobNameList = async (name?: string, size?: number) => {
  const api = APIBuilder.get("/apis/web/jobs")
    .withCredentials(true) // 이제 401 에러가 나는 경우, 자동으로 에러를 탐지하는 인터셉터를 사용하게 된다
    .params({ name, size }) // body가 없는 axios 객체도 빌더 패턴으로 쉽게 만들 수 있다
    .build();
  const { data } = await api.call<Response<JobNameListResponse>>();
  return data;
};
```

`APIBuilder` 클래스는 보일러플레이트 코드가 많지만, 옵션이 많은 경우에 다양하게 인터셉터의 옵션을 줄 수 있다는 장점이 있다.

&nbsp;

### 5. API 응답 타입 지정하기

응답 타입은 하나의 Response 타입으로 묶이는 경우가 대부분이다.

```ts
import { AxiosPromise } from "axios";
import {
  FetchCartResponse,
  PostCartRequest,
  PostCartResponse,
  apiRequester,
} from "./7.1.3-1";

export interface Response<T> {
  data: T;
  status: string;
  serverDateTime: string;
  errorCode?: string; // FAIL, ERROR errorMessage?: string; // FAIL, ERROR
}

const fetchCart = (): AxiosPromise<Response<FetchCartResponse>> =>
  apiRequester.get<Response<FetchCartResponse>>("cart");

const postCart = (
  postCartRequest: PostCartRequest
): AxiosPromise<Response<PostCartResponse>> =>
  apiRequester.post<Response<PostCartResponse>>("cart", postCartRequest);

const updateCart = (
  updateCartRequest: unknown
): AxiosPromise<Response<FetchCartResponse>> => apiRequester.get("cart");
```

💡 Response 타입을 `apiRequester` 내에서 처리하면 응답이 없는 API를 처리할 때 까다로워진다. 따라서 Response 타입은 `apiRequester`가 모르게 관리하는게 좋다.

하나의 API 서버에서 다른 API 서버로 넘겨주기만 하는 값도 있을 수 있다. 해당 값에 어떤 응답이 들어있는지 알 수 없거나 값의 형식이 달라지더라도 로직에 영향을 주지 않는 경우, `unknown` 타입을 사용하여 표시해줄 수 있다.

```ts
interface response {
  data: {
    cartItems: CartItem[];
    forPass: unknown;
  };
}
```

만약 `forPass` 안에 로직에서 사용되는 값이 있어도 `unknown`을 유지하는 게 좋다. 로그를 위해 단순히 받아서 넘겨주는 값의 타입은 언제든 변경될 수 있으므로 `forPass` 내의 값을 사용하지 않아야 한다. 다만 프로덕트에서 이미 사용하고 있는 값이라면 로직에서 사용하는 값에 대해서만 타입 단언으로 사용할 수 있다.

```ts
type ForPass = {
  type: "A" | "B" | "C";
};

const isTargetValue = () => (data.forPass as ForPass).type === "A";
```

&nbsp;

### 6. 뷰 모델(View Model) 사용하기

API 응답은 자주 변경될 가능성이 있다. 뷰 모델을 사용하여 API 변경에 따른 범위를 한정해줄 수 있다.

뷰 모델을 사용함으로써 컴포넌트의 변경, 수정을 줄일 수 있다.

```ts
interface JobListResponse {
  jobItems: JobListItemResponse[];
}

class JobListItem {
  constructor(item: JobListItemResponse) {
    /* JobListItemResponse에서 JobListItem 객체로 변환해주는 코드 */
  }
}

// View Model
class JobList {
  // 데이터를 불변으로 만들어 캡슐화
  readonly totalItemCount: number;
  readonly items: JobListItemResponse[];
  // 간단한 비즈니스 로직
  constructor({ jobItems }: JobListResponse) {
    this.totalItemCount = jobItems.length;
    this.items = jobItems.map((item) => new JobListItem(item));
  }
}

// Data Model인 API 응답을 View Model인 JobList로 변환
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

또한 `totalItemCount` 같은 도메인 개념을 넣을 때 백엔드나 UI에서 로직을 추가하여 처리할 필요없이 추가할 필드만 뷰 모델에 추가할 수 있어 편리하다.

다만, 뷰 모델을 사용함으로써 

1. 추상화 레이어를 추가하게 되면 코드가 복잡해지고 레이어를 관리하는데 번거로움이 생긴다.
2. API 응답에 없는 새로운 필드를 만들 때 서버와 클라이언트가 실제 사용하는 도메인이 다르면 서버, 클라이언트 간 의사소통 문제가 생길 수 있다.

결국 API 응답이 바뀌는 일이 최대한 없는 것이 좋고, 어쩔 수 없이 변경될 때는 코드 수정에 들어가는 비용을 최대한 줄이고 도메인의 일관성을 지킬 수 있는 절충안을 찾아야 한다.

&nbsp;

### 7. SuperStruct를 사용해 런타임에서 응답 타입 검증하기

TS는 컴파일 타임 에러를 잡아낼 수는 있지만, 의도하지 않은 API 값이 들어온다든지, 런타임에 예상치 못한 값이 들어와 오류를 발생시키는 건 막을 수 없다. 이를 방지하기 위해 `Superstruct` 같은 라이브러리를 사용할 수 있다. `zod`나 다른 라이브러리를 이용할 수도 있다.

```ts
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

assert(data, Article);
is(data, Article);
validate(data, Article);
```

* `assert(data, Type)`: 유효하지 않을 경우 에러 반환
* `is(data, Type)`: 유효성 검사에 따라 `true` 혹은 `false` 반환
* `validate(data, Type)`: `[error, data]` 형식의 튜플 반환, 유효하지 않을 경우는 에러, 유효한 경우 `[undefined, data value]` 반환
* Infer: `Superstruct`의 스키마를 TS 타입으로 변환

&nbsp;

### 8. 실제 API 응답 시의 Superstruct 활용 사례

API 응답으로 들어온 데이터의 타입이 프론트에서 정의한 타입과 동일한지 유효성 검사를 해볼 수 있다.

아래 함수는 인자로 들어온 `listItems`가 `ListItem`과 동일한 타입인지 검사해서 다른 경우 에러를 던진다.

```ts
import { assert } from "superstruct";

function isListItem(listItems: ListItem[]) {
  listItems.forEach((listItem) => assert(listItem, ListItem));
}
```

&nbsp;

## 7.2 API 상태 관리하기

### 1. 상태 관리 라이브러리에서 호출하기

Redux, MobX의 예시를 상태 관리 예시를 소개한다.

&nbsp;

### 2. 훅으로 호출하기

react-query를 사용해 만든 훅을 통한 상태 관리 예시를 소개한다.

&nbsp;

## 7.3 API 에러 핸들링

### 1. 타입 가드 활용하기

Axios에서 제공되는 `isAxiosError`를 직접 활용할 수도 있고, 명시적인 서버 에러임을 표시하기 위해 해당 함수를 이용해 타입 가드 함수를 만들어 사용하는 예시를 소개하고 있다.

&nbsp;

### 2. 에러 서브클래싱하기

인증 정보 에러, 네트워크 에러, 타임아웃 에러 등 다양한 에러를 핸들링하기 위해 에러를 서브클래싱(기존 클래스를 확장하여 새로운 클래스를 만드는 과정)할 수 있다.

&nbsp;

### 3. 인터셉터를 활용한 에러 처리

Axios 인터셉터를 활용해 HTTP 에러에 일관된 로직을 적용할 수 있다.

```ts
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error
): Promise<Error> => {
  (error) => {
    // 401 에러인 경우 로그인 페이지로 이동
    if (error.response && error.response.status === 401) {
      window.location.href = `${backOfficeAuthHost}/login?targetUrl=${window.location.href}`;
    }
    return Promise.reject(error);
  };
};

// 응답 인터셉터
orderApiRequester.interceptors.response.use(
    // 성공 시 AxiosResponse 반환하는 함수 실행
  (response: AxiosResponse) => response,
  // 실패 시 httpErrorHandler 실행
  httpErrorHandler
);
```

&nbsp;

### 4. 에러 바운더리를 활용한 에러 처리

에러 바운더리는 공통으로 에러를 처리하는 리액트 컴포넌트다. 리액트 컴포넌트 하위에서 발생한 에러를 캐치하고, 헤당 에러를 가까운 부모 에러 바운더리에서 처리할 수 있게 한다.

```ts
import React, { ErrorInfo } from "react";
import ErrorPage from "pages/ErrorPage";

interface ErrorBoundaryProps {}

interface ErrorBoundaryState {
  hasError: boolean;
}

class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(): ErrorBoundaryState {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    this.setState({ hasError: true });

    console.error(error, errorInfo);
  }

  render(): React.ReactNode {
    const { children } = this.props;
    const { hasError } = this.state;

    return hasError ? <ErrorPage /> : children;
  }
}

// 사용 예시
const App = () => {
  return (
    <ErrorBoundary>
      <OrderHistoryPage />
    </ErrorBoundary>
  );
};
```

&nbsp;

### 5. 상태 관리 라이브러리에서의 에러 처리

Redux 및 MobX에서 에러를 처리하는 방법에 대해 소개한다.

&nbsp;

### 6. react-query를 활용한 에러 처리

react-query를 사용하면 `useQuery` 훅의 `isError` 속성을 사용해서 에러를 처리해줄 수 있다.

```ts
const JobComponent: React.FC = () => {
  const { isError, error, isLoading, data } = useFetchJobList();
  if (isError) {
    return (
      <div>{`${error.message}가 발생했습니다. 나중에 다시 시도해주세요.`}</div>
    );
  }
  if (isLoading) {
    return <div>로딩 중입니다.</div>;
  }
  return (
    <>
      {data.map((job) => (
        <JobItem key={job.id} job={job} />
      ))}
    </>
  );
};
```

&nbsp;

### 7. 그 밖의 에러 처리

&nbsp;

## 7.4 API 모킹

개발하다보면 서버 개발이 늦어지거나 프론트엔드 개발과 서버 개발이 동시에 이루어지는 경우가 많다.

이럴 땐 API를 모킹(가짜 모듈을 활용)해서 개발을 진행할 수 있다.

charles 등의 테스팅 툴을 활용하면 이슈 발생 상황을 재현해서 다양한 응답에 따른 테스팅을 진행하는 것도 가능하다. 

> 💡 charles: 웹 디버깅 프록시로, 시스템과 인터넷 간의 모든 HTTP 및 SSL/HTTPS 트래픽을 보고 가로채고 분석할 수 있다.
>
> 우아안기술블로그 참고: [너 혹시 T(Tester)야? : 테스트 효율을 높이는 Charles 툴 활용기](https://techblog.woowahan.com/14550/)
> 
> 💡 axios-mock-adapter: 가짜 axios 요청을 발생시키는 라이브러리
> 
> 💡 NextApiHandler: Next.js에서 API 라우트를 처리하기 위한 함수 타입이다. NextApiHandler는 req(요청)와 res(응답) 객체를 매개변수로 받는 비동기 함수다. 
> 
> NextApiHandler는 미들웨어를 통해 확장될 수 있다. 예를 들면 인증, 로깅, 에러 처리 등의 기능을 추가할 수 있다. 
> 
> req.method를 사용하여 GET, POST 등 다양한 HTTP 메서드를 처리할 수 있고, res 객체를 통해 상태 코드 설정, JSON 응답, 리다이렉트 등 다양한 응답을 처리할 수 있다.
> ```ts
> type NextApiHandler<T = any> = (
>  req: NextApiRequest,
>  res: NextApiResponse<T>
> ) => void | Promise<void>
> ```

&nbsp;

### 1. JSON 파일 불러오기

간단한 조회 같은 경우엔 JSON 파일로 더미 데이터를 만들어 활용할 수 있다. 이전까진 프로젝트를 진행할 땐 더미 데이터를 만들 때 따로 파일을 분리하지 않았었는데, 컴포넌트 코드와 분리하면 가독성이 좋은 코드를 작성할 수 있을 것 같다.

```ts
// mock/service.ts
const SERVICES: Service[] = [
  {
    id: 0,
    name: "배달의민족",
  },
  {
    id: 1,
    name: "만화경",
  },
];

export default SERVICES;

// api.ts
const getServices = ApiRequester.get("/mock/service.ts");
```

&nbsp;

### 2. NextApiHandler 활용하기

Next.js에 내장된 NextApiHandler를 활용할 수도 있다. 하나의 파일에 하나의 핸들러를 `export default`로 구현해야 하고, 파일의 경로가 요청 경로가 된다.

```ts
// api/mock/brand
import { NextApiHandler } from "next";

const BRANDS: Brand[] = [
  {
    id: 1,
    label: "배민스토어",
  },
  {
    id: 2,
    label: "비마트",
  },
];

const handler: NextApiHandler = (req, res) => {
  // request에 대한 유효성 검증
  res.json(BRANDS);
};

export default handler;
```

&nbsp;

### 3. API 요청 핸들러에 분기 추가하기

요청 경로를 수정할 필요없이 평소에 개발할 때 필요한 경우에만 실제 요청을 보내고 그 외에는 목업을 사용하여 개발하는 방법도 있다. API 요청을 훅 또는 별도 함수로 선언해준 후 목업 함수를 내보낼지, 실제 요청 함수를 내보낼지 결정해줄 수 있다. 

```ts
const mockFetchBrands = (): Promise<FetchBrandsResponse> =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        status: "SUCCESS",
        message: null,
        data: [
          {
            id: 1,
            label: "배민스토어",
          },
          {
            id: 2,
            label: "비마트",
          },
        ],
      });
    }, 500);
  });

const fetchBrands = () => {
  if (useMock) {
    return mockFetchBrands();
  }

  return requester.get("/brands");
};
```

개발이 완료된 뒤에도 유지보수 시 목업 함수를 사용할 수도 있고, 평소에 서버에 의존하지 않고 개발할 수 있는 장점이 있다. 하지만 모든 API 요청에 if 분기문으로 목업 데이터인지 실제 요청인지 추가해줘야 하는 번거로움이 존재한다.

&nbsp;

### 4. axios-mock-adapter로 모킹하기

if 분기문으로 목업 여부를 작성하지 않고 라이브러리로 대체할 수도 있다. axios-mock-adapter는 Axios 요청을 가로채서 응답 값을 대신 반환해주는 라이브러리다.

먼저 `MockAdapter` 객체를 생성하고, 해당 객체를 사용하는 방식으로 모킹할 수 있다.

```ts
// mock/index.ts
import axios from "axios";
import MockAdapter from "axios-mock-adapter";
import fetchOrderListSuccessResponse from "fetchOrderListSuccessResponse.json";

interface MockResult {
  status?: number;
  delay?: number;
  use?: boolean;
}

const mock = new MockAdapter(axios, { onNoMatch: "passthrough" });

export const fetchOrderListMock = () =>
// 해당 엔드포인트 접근에 대해 무조건 코드 200과 fetchOrderListSuccessResponse를 반환
  mock.onGet(/\/order\/list/).reply(200, fetchOrderListSuccessResponse);

// fetchOrderListSuccessResponse.json
{
    "data": [
        {
            "orderNo": "ORDER1234", "orderDate": "2022-02-02", "shop": {
            "shopNo": "SHOP1234",
            "name": "가게이름1234" },
            "deliveryStatus": "DELIVERY"
        },
    ]
}
```

GET 뿐만 아니라 다른 HTTP 메서드에 대해서도 목업을 작성해 줄 수 있다. 또한 `networkError`나 `timeoutError` 등의 메서드로 임의로 에러를 발생시킬 수도 있다.

&nbsp;

### 5. 목업 사용 여부 제어하기

로컬 환경에서는 목업을 사용하고, dev나 production 환경에서는 사용하지 않도록 설정해줄 수 있다.

실제 서버 개발이 끝나면 환경 변수를 선언해 `process.env.USE_MOCK === 'true'`일 때만 목업 데이터를 사용하는 등의 처리를 통해 목업 데이터와 실제 서버에서 오는 데이터를 오가며 개발할 수 있다.

이렇게 하면 프로덕션 코드와 목업을 위한 코드를 분리할 필요가 없다. 

```ts
// REACT_APP_MOCK이 true일 때만 목업 데이터 사용
const useMock = Object.is(REACT_APP_MOCK, "true");

const mockFn = ({ status = 200, time = 100, use = true }: MockResult) =>
  use &&
  mock.onGet(/\/order\/list/).reply(
    () =>
      new Promise((resolve) =>
        setTimeout(() => {
          resolve([
            status,
            status === 200 ? fetchOrderListSuccessResponse : undefined,
          ]);
        }, time)
      )
  );

if (useMock) {
  mockFn({ status: 200, time: 100, use: true });
}
```

`useMock` 플래그를 통해 `mockFn`의 실행 여부를 결정한다. 매개변수를 넘겨 특정 mock 함수만 작동하게 하거나 작동하지 않도록 설정해줄 수도 있을 것이다.

만약 스크립트 실행 시 목업 사용 여부를 결정하고 싶을 땐 `package.json` 파일에서 설정해주면 된다.

```json
// package.json
{
  "scripts": {
    ...
    "start:mock": "REACT_APP_MOCK=true npm run start",
    "start": "REACT_APP_MOCK=false npm run start"
    ...
  },
  ...
}
```

axios-mock-adapter는 API 요청을 중간에 가로채기 때문에 실제 API 요청을 주고 받지 않아 브라우저의 개발자 도구의 네트워크 탭에서 API 요청의 흐름을 파악할 수는 없다. 그럴 땐 별도의 도구를 활용해야 한다.

목업을 사용할 때 네트워크 요청을 확인하려면 네트워크에 보낸 요청을 변경해주는 Cypress 같은 도구의 웹훅을 사용할 수 있다.

> 💡 Cypress: 프론트엔드 테스트를 위한 오픈 소스 자바스크립트 엔드 투 엔드 테스트 도구다. 주로 웹 애플리케이션의 동작을 시뮬레이션하고 테스트하는 데 사용된다.

최근에는 서비스 워커를 활용하는 라이브러리인 MSW라는 것도 있다고 하니 추후 공부해보면 좋을 것 같다.

> 💡 서비스 워커(Service Worker): 웹 페이지와 별도로 브라우저가 백그라운드에서 실행되는 스크립트로 응용 프로그램, 브라우저, 그리고 네트워크 사이의 프록시 서버 역할을 합니다.