```jsx
7.1 API 요청
7.2 API 상태 관리하기
7.3 API 에러 핸들링
7.4 API 모킹
```
<details><summary>요약본</summary>

# 7.3 챕터 요약

```tsx
7.3.1 타입 가드 활용하기
7.3.2 에러 서브클래싱하기
7.3.3 인터셉터를 활용한 에러 처리
7.3.4 에러 바운더리를 활용한 에러 처리
7.3.5 상태 관리 라이브러리에서의 에러 처리
7.3.6 리액트 쿼리를 활용한 에러 처리
7.3.7 그 밖의 에러 처리
```

다양한 API 에러가 존재한다. 이를 어떻게 처리할까?

**🩵서버 에러 판별**

- 타입 가드를 활용하는 방법은 axios 라이브러리의 isAxiosError이라는 타입 가드를 사용하고 여기에 추가적으로 제네릭 타입을 한정하는 방식으로 해서 서버 에러를 판별할 수 있다.

**🩵다양한 서버 에러 판별**

서버 에러에도 여러가지가 존재한다. 이를 구체적으로 명시하기 위해 서브클래싱을 활용해서 공통 부모 에러 클래스를 구현하고 이를 상속받는 특정 에러 클래스들을 정의한다. 에러를 판단할 때 instanceof를 활용해 특정 에러클래스인지 판별할 수 있게 되고 다양한 에러 상황에 대응할 수 있다.

**🩵중앙에서 에러를 관리**

인터셉터를 활용해서 요청과 응답을 중앙에서 관리할 수 있다.

**🩵부모 컴포넌트에서 에러 처리**

리액트의 클래스형 컴포넌트를 사용해서 자식 컴포넌트에서 발생한 에러를 부모 컴포넌트인 에러 바운더리 컴포넌트에서 처리해줄 수 있다.

**🩵전역 상태로 에러 처리**

전역 상태로 에러를 처리하는 스토어를 사용해 상태를 관리하고 인터셉터를 설정해 해당 스토어의 상태를 업데이트하는 형식으로 에러를 처리한다.

**🩵데이터 패칭 라이브러리**

리액트 쿼리를 사용하면 로딩, 성공, 에러 등의 요청 상태 관리가 편하다. 

위의 에러 말고도 아주 특정된 에러들이 존재할 수 있다. 이를 조건문 if,else로 분기처리할 수도 있고 인터셉터에서 커스텀 에러를 판별해 따로 처리해줄 수도 있다.

# 7.4 챕터 요약

```tsx
7.4.1 JSON 파일 불러오기
7.4.2 NextApiHandler 활용하기
7.4.3 API 요청 핸들러에 분기 추가하기
7.4.4 axios-mock-adapter로 모킹하기
7.4.5 목업 사용 여부 제어하기
```

모킹은 실제 서버와 네트워크 통신 없이, 프론트에서 가짜 서버를 사용해  api 요청과 응답을 처리하는 방법이다. 이를 어떻게 할 수 있을까?

**🩵초보**

먼저 정적 json 파일을 바로 사용할 수 있다. 

next.js를 사용한다면 NextApiHandler를 활용해 json 파일과 핸들러를 하나의 파일에 구현하고 해당 파일 경로를 api 엔드포인트르로 사용해 mock api를 구성할 수 있다.

**🩵중수**

만약 필요한 경우에만 실제 요청을 보내고 그 외에는 목업을 사용해 개발하려고 하면 플래그를 세워 분기처리해줘야 한다.

만약 서비스 함수에 분기문이 추가되는 것을 바라지 않는다면 axios-mock-adapter과 같은 라이브러리를 사용할 수 있는데 이는 요청을 가로채 요청에 대한 응답값을 대신 반환해준다. 즉 기존의 실제 서버와 통신하는 코드를 수정하지 않으면서 mock 함수로 처리해야 하는 부분만 처리해줄 수 있다.

**🩵고수**

로컬에서는 목업을 사용하고 dev나 운영 환경에서는 사용하지 않게 하려면 플래그 사용, package.json에 스크립트 추가, config 파일 별도 구성, 프록시 사용 등의 방법으로 제어할 수 있다. 

+

추가적으로 결국 모킹은 실제 네트워크 통신을 하는 것이 아니기 때문에 개발자 도구의 네트워크 요청을 확인할 수 없다. 이를 확인하고 싶다면 “Cypress”와 같은 웹훅을 사용하면 된다.

</details>

# 7.3 API 에러 핸들링

```tsx
7.3.1 타입 가드 활용하기
7.3.2 에러 서브클래싱하기
7.3.3 인터셉터를 활용한 에러 처리
7.3.4 에러 바운더리를 활용한 에러 처리
7.3.5 상태 관리 라이브러리에서의 에러 처리
7.3.6 리액트 쿼리를 활용한 에러 처리
7.3.7 그 밖의 에러 처리
```

타입스크립트에서는 다양한 API 에러를 어떻게 처리하고 명시하는가.

## 7.3.1 타입 가드 활용하기

Axios 라이브러리에서는 `isAxiosError` 라는 타입 가드를 제공한다. 개발자가 직접 커스텀 타입 가드를 사용해 구체적이고 명확하게 서버 에러와 에러 객체의 속성을 정의할 수도 있다.

### 🔹 사용자 정의 타입 가드

1. 공통 에러 객체 타입

```tsx
interface ErrorResponse {
  status: string;
  serverDateTime: string;
  errorCode: string;
  errorMessage: string;
}
```

1. 사용자 정의 타입 가드

> axios 타입 가드에, 타입시스템을 활용해 타입을 한정한 사용자 정의 타입 가드
> 

에러 객체가 axios 에러인지 확인한다. 에러 객체의 제네릭 타입을ErrorResponse로 제한했다.

```tsx
function isServerError(error: unknown): error is AxiosError<ErrorResponse> {
  return axios.isAxiosError(error);
}
```

<aside>
📫

사용자 정의 타입 가드를 정의할 때는 타입 가드 함수의 반환 타입으로 **`parameterName is Type`** 형태의 타입 명제(Type Predicate)를 정의해주는 게 좋다. 이때 **`parameterName`**은 타입 가드 함수의 시그니처에 포함된 매개변수여야 한다.

</aside>

1. 사용 예제

특정 주문 내역을 삭제하는 비동기 함수.

```tsx
const onClickDeleteHistoryButton = async (id: string) => {
  try {
    await axios.post("https://....", { id });

    alert("주문 내역이 삭제되었습니다.");
  } catch (error: unknown) {
    if (isServerError(e) && e.response && e.response.data.errorMessage) {
      // 서버 에러일 때의 처리임을 명시적으로 알 수 있다 setErrorMessage(e.response.data.errorMessage);
      return;
    }
    setErrorMessage("일시적인 에러가 발생했습니다. 잠시 후 다시 시도해주세요");
  }
};
```

## 7.3.2 에러 서브클래싱하기

서버 에러에도 여러 종류가 있다. (인증 정보 에러, 네트워크 에러, 타임아웃 에러 등)

이렇게 구체적으로 서버 에러를 명시하기 위해 “서브클래싱”을 활용할 수 있다.

<aside>
📫

서브클래싱

기존(상위 또는 부모) 클래스를 확장해 새로운(하위 또는 자식) 클래스를 만드는 과정을 말한다. 새로운 클래스는 상위 클래스의 모든 속성과 메서드를 상속받아 사용할 수 있고 추가적인 속성과 메서드를 정의할 수도 있다.

</aside>

### ▶️ 특정 페이지의 주문 내역 데이터 가져오기

에러가 발생하면 에러 메시지를 alert로 표시

```tsx
const getOrderHistory = async (page: number): Promise<History> => {
  try {
    const { data } = await axios.get(`https://some.site?page=${page}`);
    const history = await JSON.parse(data);

    return history;
  } catch (error) {
    alert(error.message);
  }
};
```

### ▶️ 서브클래싱을 활용 : 커스텀 에러 클래스

에 클래스를 확장해 http 요청 관련 에러를 처리하는데 3가지 에러 클래스 : `OrderHttpError`, `NetworkError`, `UnauthorizedError`

1. OrderHttpError 클래스

privateResponse : http 응답 데이터 저장용 필드 추가

```tsx
class OrderHttpError extends Error {
  private readonly privateResponse: AxiosResponse<ErrorResponse> | undefined;

  constructor(message?: string, response?: AxiosResponse<ErrorResponse>) {
    super(message); // 부모 클래스 생성자 호출
    this.name = "OrderHttpError";
    this.privateResponse = response;
  }

  get response(): AxiosResponse<ErrorResponse> | undefined {
    return this.privateResponse;
  }
}

```

1. NetworkError 클래스

```tsx
class NetworkError extends Error {
  constructor(message = "") {
    super(message);
    this.name = "NetworkError"; // 이름 설정
  }
}
```

1. UnauthorizedError 클래스

```tsx
class UnauthorizedError extends Error {
  constructor(message: string, response?: AxiosResponse<ErrorResponse>) {
    super(message, response);
    this.name = "UnauthorizedError";
  }
}

```

해당 클래스를 조건에 따라 axios 인터셉터에서 적합한 에러 객체를 전달할 수 있다.

1. httpErrorHandler

`httpErrorHandler` 는 에러를 입력받아 특정 에러 상황에 맞는 “커스텀 에러 객체”를 생성하고 이를 `Promise.reject` 로 반환한다. 

```tsx
const httpErrorHandler = (
  error: AxiosError<ErrorResponse> | Error
): Promise<Error> => {
  let promiseError: Promise<Error>;

  if (axios.isAxiosError(error)) { //  axios에서 발생한 서버 에러라면
    if (Object.is(error.code, "ECONNABORTED")) {
      promiseError = Promise.reject(new TimeoutError());
    } else if (Object.is(error.message, "Network Error")) {
      promiseError = Promise.reject(new NetworkError());
      
    } else {
    // 타임아웃과 네트워크 에러가 아니라면 http 상태별 커스텀 에러 객체 생성
      const { response } = error as AxiosError<ErrorResponse>;
      switch (response?.status) {
        case HttpStatusCode.UNAUTHORIZED:
          promiseError = Promise.reject(
            new UnauthorizedError(response?.data.message, response)
          );
          break;
        default:
          promiseError = Promise.reject(
            new OrderHttpError(response?.data.message, response)
          );
      }
    }
  } else {
    promiseError = Promise.reject(error);
  }

  return promiseError;
};
```

1. 활용

`onActionError` 는 다양한 에러 상황을 처리하고, 알림과 콜백을 적절히 실행한다.

```tsx
const alert = (meesage: string, { onClose }: { onClose?: () => void }) => {};

const onActionError = (
  error: unknown,
  params?: Omit<AlertPopup, "type" | "message">
) => {
  if (error instanceof UnauthorizedError) {
    onUnauthorizedError(
      error.message,
      errorCallback?.onUnauthorizedErrorCallback
    );
  } else if (error instanceof NetworkError) {
    alert("네트워크 연결이 원활하지 않습니다. 잠시 후 다시 시도해주세요.", {
      onClose: errorCallback?.onNetworkErrorCallback,
    });
  } else if (error instanceof OrderHttpError) {
    alert(error.message, params);
  } else if (error instanceof Error) {
    alert(error.message, params);
  } else {
    alert(defaultHttpErrorMessage, params);
  }

  const getOrderHistory = async (page: number): Promise<History> => {
    try {
      const { data } = await fetchOrderHistory({ page }); // api 요청
      const history = await JSON.parse(data);

      return history;
    } catch (error) {
      onActionError(error);
    }
  };
};
```

### ❓instanceof가 에러 처리에 적합한 이유가 무엇인가요?

다양한 에러 타입을 구분하고, 상속 구조를 지원하기 때문에 커스텀 에러 타입 구분에 용이하다. 타입을 확인한 후 접근하므로 타입 검사 면에서도 안정성이 있다.

## 7.3.3 인터셉터를 활용한 에러 처리

axios의 응답 인터셉터를 활용해 http 요청 실패 시 발생하는 에러를 처리하고, 401에러 등의 조건에 따라 추가 동작을 수행시킬 수 있다.

```tsx
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

orderApiRequester.interceptors.response.use(
  (response: AxiosResponse) => response,
  httpErrorHandler
);
```

## 7.3.4 에러 바운더리를 활용한 에러 처리

### ❓에러 바운더리가 뭔가요?

> 리액트 컴포넌트 트리에서 에러가 발생할 때 공통으로 에러를 처리하는 리액트 컴포넌트이다.
> 
- 에러 바운더리를 사용하면 리액트 컴포넌트 트리 하위에 있는 컴포넌트에서 발생한 에러를 캐치하고, 해당 에에러를 “가까운 부모 에러 바운더리에서 처리”할 수 있다.

언제 사용할 수 있을까?

- 에러가 발생한 컴포넌트 대신! 에러 처리를 하거나 예상치 못한 에러 공통 처리할 때 사용

1. ErrorBoundary 클래스

리액트의 에러 바운더리 역할을 수행하는 **클래스형 컴포넌트.**

- 리액트 라이프사이클 메서드인 `getDerivedStateFromError`와 `componentDidCatch`를 활용하여 에러를 처리.
- 제네릭 타입으로 props와 state를 받는다. state에는 hasError 상태를 포함한다.

```tsx
class ErrorBoundary extends React.Component<
  ErrorBoundaryProps,
  ErrorBoundaryState
> {
  ...
}
```

```tsx
static getDerivedStateFromError(): ErrorBoundaryState {
	// 자식 컴포넌트에서 에러가 발생하면 리액트가 호출하는 정적 메서드. 
	// 컴포넌트의 state.hasError를 업데이트
  return { hasError: true };
}
// ...
componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
	// 에러와 관련된 추가 정보를 캡처하고, 로깅이나 에러 보고용으로 사용
  this.setState({ hasError: true });

  console.error(error, errorInfo);
}

```

1. 전체코드

```tsx
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
    // 에러 발생하면 에러페이지 보여주고 아니면 자식 컴포넌트 렌더링
  }
}

const App = () => {
  return (
    <ErrorBoundary>
      <OrderHistoryPage />
    </ErrorBoundary>
  );
};
```

### ❓클래스형 컴포넌트가 무엇인가요?

리액트는 함수형 컴포넌트와 클래스형 컴포넌트를 모두 지원한다. `React.Component` 를 상속받아 구성된다. 또 render() 메서드 안에 jsx를 리턴한다.

가장 큰 특징은 다양한 생명 주기 메서드를(함수형에서 useEffect) 지원한다. 

리액트 16.8부터 훅이 도입되며 함수형 컴포넌트에서 상태 관리 및 생명 주기 메서드 구현이 가능해졌다. 이에 따라 기본적으로 함수형 컴포넌트 사용이 권장된다. 그러나 특정 경우에는 클래스 컴포넌트를 사용해야 한다.

여기서처럼 특정 생명주기 메서드(componeneDidCatch)처럼 고급 생명주기 메서드가 필요한 경우가 그렇다. 그 밖에도 마이그레이션이 어렵거나 기존에 클래스 컴포넌트로 구성이 완료된 상태라면 사용할 것이다.

## 7.3.5 상태 관리 라이브러리에서의 에러 처리

앞의 리덕스 에러 처리 방법

- 전체코드
    
    ```tsx
    // API 호출에 관한 api call reducer
    const apiCallSlice = createSlice({
      name: "apiCall",
      initialState,
      reducers: {
        setApiCall: (state, { payload: { status, urlInfo } }) => {
          /* API State를 채우는 logic */
        },
        setApiCallError: (state, { payload }: PayloadAction<any>) => {
          state.error = payload;
        },
      },
    });
    
    const API = axios.create();
    
    const setAxiosInterceptor = (store: EnhancedStore) => {
      /* 중복 코드 생략 */
      // onSuccess시 처리를 인터셉터로 처리한다
      API.interceptors.response.use(
        (response: AxiosResponse) => {
          const { method, url } = response.config;
    
          store.dispatch( // 성공 처리
            setApiCall({
              status: ApiCallStatus.None, // API 호출 상태를 `요청되지 않음`으로 변경
              urlInfo: { url, method },
            })
          );
    
          return response?.data?.data || response?.data;
        },
        (error: AxiosError) => { // 실패처리
          // 401 unauthorized
          if (error.response?.status === 401) {
            window.location.href = error.response.headers.location;
    
            return;
          }
          // 403 forbidden
          else if (error.response?.status === 403) {
            window.location.href = error.response.headers.location;
            return;
          }
          // 그 외에는 화면에 alert 띄우기
          else {
            message.error(`[서버 요청 에러]: ${error?.response?.data?.message}`);
          }
    
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
    
- apiCallSlice : 리덕스 툴킷의 createSlice를 사용해 api 호출 상태를 관리하는 슬라이스를 정의한다.
    - 주요 리듀서 : `setApiCall`, `setApiCallError`
- 인터셉터를 설정해 성공이나 실패시 리덕스 상태를 업데이트하거나 적절한 처리 수행

```tsx
      // 그 외의 에러는 화면에 alert 띄우기
      else {
        message.error(`[서버 요청 에러]: ${error?.response?.data?.message}`);
      }

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
```

- 401, 403 에러를 바로 처리하고 다른 에러 상태는 reject로 넘겨준다. 이후 액션을 정의하면서 setApiCallError를 사용해 에러를 상태로 처리한다.

리덕스 툴킷의 createAsyncThunk를 사용해 비동기 api 요청과 그 결과를 리덕스 상태로 관리

```tsx
const fetchMenu = createAsyncThunk(
  FETCH_MENU_REQUEST,
  async ({ shopId, menuId }: FetchMenu) => {
    try {
      const data = await api.fetchMenu(shopId, menuId);
      return data;
    } catch (error) {
      setApiCallError({ error });
    }
  }
);
```

## 7.3.6 react-query를 활용한 에러 처리

- 전체코드
    
    ```tsx
    const JobComponent: React.FC = () => {
    // 리액트 쿼리 커스텀 훅, 내부에서 useQuery 사용
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
    

## 7.3.7 그 밖의 에러 처리

커스텀 에러를 어떻게 처리할 것인가

다음과 같은 커스텀 에러가 있다.

```tsx
httpstatus: 200
{
	"status": "C20005", // 성공인 경우 "SUCCESS" 를 응답
	"message": " 장바구니에 품절된 메뉴가 있습니다."
}
```

### ▶️ 간단한 커스텀 에러 처리 : if,else

커스텀 에러 처리하기 위해 요청 함수 내에서 조건문으로 상태를 비교

```tsx
const successHandler = (response: CreateOrderResponse) => {
  if (response.status === "SUCCESS") {
    // 성공 시 진행할 로직을 추가한다
    return;
  }
  throw new CustomError(response.status, response.message);
};

const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post("https://...", data);

    successHandler(response);
  } catch (error) {
    errorHandler(error);
  }
};
```

### ▶️ 커스텀 에러를 사용하는 요청을 일괄적으로 에러 처리하기

인터셉터에서 커스텀 에러를 판단하고 에러를 던진다. 

```tsx
export const apiRequester: AxiosInstance = axios.create({
  baseURL: orderApiBaseUrl,
  ...defaultConfig,
});

export const httpSuccessHandler = (response: AxiosResponse) => {
  if (response.data.status !== "SUCCESS") {
    throw new CustomError(response?.data.message, response);
  }

  return response;
};

apiRequester.interceptors.response.use(httpSuccessHandler, httpErrorHandler);

const createOrder = (data: CreateOrderData) => {
  try {
    const response = apiRequester.post("https://...", data);

    successHandler(response);
  } catch (error) {
    // status가 SUCCESS가 아닌 경우 에러로 전달된다
    errorHandler(error);
  }
};
```

# 7.4 API 모킹

API 완성 전에 개발을 진행해야 할 때, 별도의 가짜 서버=목서버를 제공한다고 해도 모든 예외 사항을 처리하기 쉽지 않다. 

이럴 때 모킹을 사용한다. 모킹은 가짜 모듈을 활용하는 것이다.

장점 : 서버에 문제가 생겨도 서버의 영향을 받지 않고 프론트엔드 개발을 할 수 있다.

> 우아한형제들 프론트엔드에서는 axios-mock-adapter, NextApiHandler 등을 활용해 API를 모킹해서 사용한다.
> 

```tsx
7.4.1 JSON 파일 불러오기
7.4.2 NextApiHandler 활용하기
7.4.3 API 요청 핸들러에 분기 추가하기
7.4.4 axios-mock-adapter로 모킹하기
7.4.5 목업 사용 여부 제어하기
```

# 7.4.1 JSON 파일 불러오기

목업 데이터 파일인 json 파일을 만들어 사용한다.

# 7.4.2 NextApiHandler 활용하기

Next.js를 사용한다면 활용할 수 있다.

```tsx
요약
json 데이터와 그 아래 핸들러를 추가해 유효성을 검증한다.해당 파일ㄹ 경로가 API 엔드포인트가 된다.
```

> “NextApiHandler”는 하나의 파일 안에 하나의 핸들러를 default export로 구현해야 하며 파일의 경로가 요청 경로가 된다.
> 

핸들러 정의

- 응답하고자 하는 값을 정의하고 핸들러 안에서 요청에 대한 응답의 정의
- 핸들러를 사용하면 단순 파일을 불러오는 것에서 발전해, 중간 과정에 응답 처리 로직을 추가할 수 있다.

```tsx
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
  // request 유효성 검증
  res.json(BRANDS);
};

export default handler;

// 사용예제
...
useEffect(() => {
    const fetchBrands = async () => {
      try {
        const response = await fetch("/api/mock/brand"); // Mock API 호출
```

# 7.4.3 API 요청 핸들러에 분기 추가하기

필요한 경우에만 실제 요청을 보내고, 그 외에는 목업을 사용해 개발하고 싶다. API 요청을 훅 또는 별도 함수로 선언하고 조건에 따라 목업 함수를 내보내거나 실제 요청 함수를 내보낸다.

```tsx
const mockFetchBrands = (): Promise<FetchBrandsResponse> =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        status: "SUCCESS",
        message: null,
        data: [
          { id: 1, label: "배민스토어", },
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

### ▶️  +API별로 목업 여부를 설정

```tsx
// mockConfig.ts
export const mockConfig = {
  brands: true, // 브랜드 관련 API는 Mock 사용
  orders: false, // 주문 관련 API는 실제 요청 사용
  products: false, // 상품 관련 API는 실제 요청 사용
};

```

```tsx
import { mockConfig } from "./mockConfig";
import { requester } from "./requester";
import { mockFetchBrands } from "./mockFetchers";

export const fetchBrands = () => {
  if (mockConfig.brands) {
    return mockFetchBrands(); // Mock 데이터 반환
  }
  return requester.get("/brands"); // 실제 API 호출
};
```

조금 더 발전 : API 요청 중앙에서 관리

```tsx
import { mockConfig } from "./mockConfig";
import { requester } from "./requester";
import { mockFetchBrands, mockFetchOrders } from "./mockFetchers";

const apiEndpoints = {
  brands: "/brands",
  orders: "/orders",
  products: "/products",
};

export const apiRequest = (key: keyof typeof apiEndpoints, mockHandler?: () => Promise<any>) => {
  if (mockConfig[key] && mockHandler) {
    return mockHandler(); // Mock 데이터 반환
  }
  return requester.get(apiEndpoints[key]); // 실제 API 호출
};

// API 호출
...
export const fetchBrands = () => apiRequest("brands", mockFetchBrands);
export const fetchOrders = () => apiRequest("orders", mockFetchOrders);

```

그 밖에도 여러 방법이 있다.

# 7.4.4 axios-mock-adapter로 모킹하기

*서비스 함수에 분기문이 추가되는 것을 바라지 않는다면* 라이브러리를 사용하면 된다. 

> axios-mock-adapter는 요청을 가로채서 요청에 대한 `응답 값을 대신 반환`한다.
> 
- MockAdapter 객체를 생성하고, 해당 객체를 사용해 모킹
    - 앞의 방법과는 다르게 mock API의 주소가 필요하지 않다.
    - 단순 응답 바디만 모킹할 수도 있고 상태 코드, 응답 지연 시간 등을 추가로 설정할 수도 있다.
    - 다양한 http 상태 코드에 대한 목업을 정의할 수 있고, api별로 지연 시간을 다르게 설정할 수 있다.
        - 이렇게 응답 처리를하는 부분을 별도 함수로 구현하면 여러 mock 함수에서 사용할 수 있다.

```tsx
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

다양한 http 메서드에 대한 목업 작성

lazyData: 비동기적으로 데이터를 반환하는 mock 데이터 생성기, 일정 시간 후에 성공이나 실패 데이터를 반환.

fetchOrderListMock : 특정 엔드포인트 (/order/list)에 대한 mock 처리를 설정. mock 요청을 활성화=use하면 lazyData를 통해 비동기 응답을 반환.

```tsx
export const lazyData = ( 
  status: number = Math.floor(Math.random() * 10) > 0 ? 200 : 200,
  successData: unknown = defaultSuccessData,
  failData: unknown = defaultFailData,
  time = Math.floor(Math.random() * 1000)
): Promise<any> =>
  new Promise((resolve) => {
    setTimeout(() => {
      resolve([status, status === 200 ? successData : failData]);
    }, time);
  });

export const fetchOrderListMock = ({
  status = 200,
  time = 100,
  use = true,
}: MockResult) =>
  use &&
  mock
    .onGet(/\/order\/list/)
    .reply(() =>
      lazyData(status, fetchOrderListSuccessResponse, undefined, time)
    );

```

에러를 발생시키는 메서드도 있기 대문에 임의로 에러 발생도 가능하다.

```tsx
export const fetch0rderListMock = 0 => mock.onPost(/VorderVlist/).networkError(;
```

# 7.4.5 목업 사용 여부 제어하기

로컬에서는 목업을 사용하고 dev나 운영 환경에서는 사용하지 않으려면 플래그를 사용해 구분하면 된다.

이렇게 구분을 해 두면 프로덕션에서 사용되는 코드와 목업을 위한 코드를 분리할 필요가 없다. 

- 프론트엔드와 서버의 독립. 이후 서버에 문제가 생겨도 프론트엔드 개발을 독립적으로 진행할 수 있다.

```tsx
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

 스크립트 실행시 구분하고자 하면 package.json에 관련 스크립트 추가.

```tsx
// package.json

{
  "scripts": {
    "start:mock": "REACT_APP_MOCK=true npm run start",
    "start": "REACT_APP_MOCK=false npm run start"
  }
}
```

자바스크립트 코드의 실행 여부를 제어하지 않고 config 파일을 별도로 구성하거나 프록시를 사용할 수도 있다.

1. axios-mock-adapter를 사용하는 경우에는 api 요청을 인터셉터하기 때문에 실제로 api 요청을 주고받지 않는다.
2. 브라우저가 제공하는 개발자 도구의 네트워크 탭에서 확인할 수 없다. 
3. 이 때 api 요청의 흐름을 파악하고 싶다면 react-query-devtools 또는 redux test tool 처럼 별도의 도구를 사용해야 한다.
4. 목업을 사용할 때 네트워크 요청을 확인하고 싶다면, 네트워크에 보낸 요청을 변경해주는 “Cypress”같은 도구의 웹훅을 사용하면 된다

<aside>
🔬

**Cypress**

프론트엔드 테스트를 위한 오픈 소스 자바스크립트 엔드 투 엔드 테스트 도구다. 주로 웹 애플리케이션의 동작을 시뮬레이션하고 테스트하는 데 사용된다. Cypress는 사용하기 쉽고 강력한 기능을 제공하여 웹 애플리케이션을 더욱 견고하고 안정적으로 개발할 수 있도록 도와준다.

</aside>

기타

> 앞에서 소개한 모킹 방식 외에도 최근에는 서비스워커를 활용하는 라이브러리인 MSW를 도입한 팀도 있다. MSW를 사용하면 모킹시 개발환경과 운영환경을 분리할 수 있으며, 개발자 도구의 네트워크 탭에서 API 통신을 확인할 수 있다. MSW를 래핑한 개발자 도구를 만들어 사용하기도 한다.
> 

## 우형이야기

### 우아한형제들에서는 데이터 패칭 라이브러리를 어떻게 활용하고 있는가?

Q. 데이터 패칭 라이브러리를 사용하나요? 사용한다면 어떤 기준으로 선택했나요? 또 사용하고 나서 느낀 장단점은 어떤 게 있나요?

1. 사용하지 않습니다. 필요한 경우 Recoil에서 제공하는 useSelector를 활용해 데이터 패칭을 처리합니다.
2. 데이터의 복잡성이 높지 않아서 상태 관리 라이브러리 느낌으로 리액트 쿼리를 사용합니다. 
    1. 팀에서 “상태 관리”를 목적으로 했을 때 상단 부분이 비동기 통신을 위해 쓰인다고 생각했다.
    2. ‘서버에서 가져온 데이터 관리 용도로 클라이언트 상태를 관리하는 MobX나 Redux를 사용하는 게 맞을까?’ 라는 의문.
    3. 결과적으로 리덕스를 걷어냈다.
3. 리액트 쿼리를 사용합니다. 기존에 상태 관리 라이브러리를 사용하면서 서버에서 가져온 값을 처리할 때는 비동기 처리 레이어나 로직 작업을 추가로 했는데 리액트 쿼리를 사용하면서 편리해졌다. 데이터를 다시 가져오거나 refetch, 캐싱해주는 기능은 매우 강력하다. swr보다 리액트 쿼리가 더 다양한 기능을 제공한다 생각해서 선택했다.
4. 데이터 패칭 라이브러리를 사용하는 이유는 서버 데이터를 캐싱, 하나의 api로 여러 컴포넌트에서 활용 가능하다는 점 때문이다. 처음에는 next.js를 염두에 두고 swr을 사용하다가 현재는 제공하는 기능이 다양한 리액트 쿼리를 사용합니다. 
    1. swr과 달리 리액트 쿼리는 서버 상태를 변경하는 useMutation 메서드를 제공해주며 자체적으로 isLoading에 대해 처리도 해줘 편리합니다.
5. 캐싱과 리패칭 면에서 데이터 패칭 라이브러리를 사용하는 게 좋다고 생각합니다. 저희 팀은 리액트 쿼리와 swr 둘 다 사용합니다. 고객 지향 서비스에서는 swr을, 어드민에서는 리액트 쿼리를 사용합니다. 
    1. 고객 지향 서비스는 번들 사이즈가 작은 걸 사용하는게 좋을 것 같다는 생각에 swr을 선택. 리액트 쿼리도 괜찮을 것 같다.
    2. 비교하면 리액트 쿼리가 제공하는 api의 인터페이스가 좀 더 직관적이고 러닝 커브나 낮아 사용하기 편한 것 같다.
