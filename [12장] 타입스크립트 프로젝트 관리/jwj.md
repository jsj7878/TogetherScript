# 12. 타입스크립트 프로젝트 관리

## 12.1 앰비언트 타입 활용하기

### 1. 앰비언트 타입 선언

TS의 타입 선언은 `.ts`, `.tsx`, **`.d.ts`** 확장자에서 할 수 있다.
`.d.ts` 확장자의 파일에서는 타입 선언만 할 수 있으며 값을 표현할 수는 없다. `.d.ts` 확장자에서의 타입 선언을 앰비언트 타입 선언이라고 한다.

앰비언트 타입 선언에는 `declare`라는 키워드를 사용한다. `declare`는 TS 컴파일러에 부가적인 타입 정보를 알려줄 때 사용한다.

대표적으로 앰비언트 타입은 어떤 용도로 활용할 수 있을까?

앰비언트 타입은 'JS 코드 안에 이러한 정보들이 있다'라는 것을 명시해줄 때 사용한다.

TS는 기본적으로 `.ts`와 `.js` 파일만 이해하며 다른 확장자는 인식할 수 없다. 따라서 이 2가지의 확장자를 제외한 파일을 모듈로 가져오면 에러가 발생하는데, 이런 상황에서 `declare` 키워드를 사용하여 특정 확장자를 모듈로 선언하면 컴파일러에 이런 확장자가 있다는 사실을 알려주어 에러가 발생하지 않는다.

아래 앰비언트 타입은 `png` 확장자의 정보를 미리 컴파일러에 전달하여 에러가 발생하게 하지 않는다.

```ts
declare module "*.png" {
  const src: string;
  export default src;
}
```

&nbsp;

JS로 작성된 라이브러리에 타입 추론을 가능하게 만들어준다. 만약 `tsconfig.json` 파일에 `any`를 사용하지 못하게 설정되어 있다면, 프로젝트가 빌드되지 않을 것이다.

이때 JS 라이브러리 내부 함수와 변수의 타입을 앰비언트 타입으로 선언해놓으면 TS는 자동으로 `.d.ts` 확장자 파일을 찾아 타입 검사를 진행하므로 문제없이 컴파일된다. 또한 VSCode와 같은 코드 편집기도 `.d.ts` 파일을 해석해 타입 힌트를 제공한다. 

tsc는 별도의 설정없이 `node_modules/@types` 디렉터리의 타입 선언을 타입 검사에 활용하기도 한다.

TS로 작성된 라이브러리조차도 라이브러리 코드를 컴파일하지 않아도 되도록 JS 파일과 `.d.ts` 파일로 배포하는 것이 일반적이다. `tsconfig.json` 파일에서 `declaration`을 `true`로 설정하면 컴파일러가 자동으로 `.d.ts` 파일을 생성해준다.

&nbsp;

JS 어딘가에 전역 변수가 정의되어 있음을 TS에게 알릴 때 앰비언트 타입을 사용한다.

예시로, 웹뷰를 개발할 때 네이티브 앱과 통신을 위한 인터페이스를 네이티브 앱이 `Window` 객체에 추가할 때, 전역 객체인 `Window`에 변수나 함수를 추가해주면 TS에 직접 구현하지 않아도 실제 런타입 환경에서 해당 변수를 사용할 수 있다.

네이티브 앱에서 `Window` 전역 객체에 `deviceId`나 `appVersion`을 할당하려고 할 때, `Window` 객체의 속성은 TS로 직접 정의한 값이 아니기 때문에 TS는 해당 속성이 `Window` 객체에 없다고 판단한다. 이때 `global namespace`에 있는 `Window` 객체에 해당 속성이 정의되어 있다는 것을 앰비언트 타입을 통해 알려줄 수 있다.

```ts
declare global {
  interface Window {
    deviceId: string | undefined;
    appVersion: string;
  }
}
```

&nbsp;

### 2. 앰비언트 타입 선언 시 주의점

1. TS로 라이브러리를 개발할 때에는 `tsconfig.json`의 `declaration`을 `true`로 설정하면 컴파일러가 `.d.ts` 파일을 생성해주기 때문에 앰비언트 타입 선언을 굳이 해줄 필요가 없다.
2. **앰비언트 변수 선언은 어느 곳에서나 영향을 줄 수 있다.** 서로 다른 라이브러리에서 동일한 이름의 앰비언트 타입을 선언하면 충돌이 발생하야 어떤 타입 선언이 적용될지 알기 어렵다. 또한 앰비언트 타입은 명시적인 import나 export가 없기 때문에 코드의 의존성 관계가 명확하지 않아 코드의 흐름을 파악하기 어렵다.

&nbsp;

### 3. 앰비언트 타입 선언을 잘못 사용했을 때의 문제점

`declare` 키워드를 사용한 앰비언트 변수 선언은 `.ts`에서도 가능하다. 하지만 절대 권장되지 않는 방법이다.

```ts
import React from "react";
import ReactDOM from "react-dom";
import App from "App";

declare global {
interface Window {
Example: string;
}
}
const SomeComponent = () = > {
  return <div>앰비언트 타입 선언은 .tsx 파일에서도 가능</div>;
};
```

**`.ts` 파일 내의 앰비언트 변수 선언은 개발자에게 혼란을 야기할 수 있고, 앰비언트 타입의 의존성 관계가 보이지 않아 변경에 의한 영향 범위를 파악하기 어렵다.**

만약 작은 컴포넌트에서 사용되고 있는 앰비언트 변수 선언이 프로그램의 장애를 발생시킨다면 장애 발생 원인을 찾아내는 것이 쉽지 않을 것이다.

`.d.ts` 확장자 파일 내에서 앰비언트 타입 선언은 개발자 간의 약속이다.

&nbsp;

### 4. 앰비언트 타입 활용하기

`.d.ts` 파일에 앰비언트 타입을 선언하여 전역 변수와 같이 사용할 수 있다. 앰비언트 타입 선언은 모든 코드 내에서 따로 import하지 않고도 사용할 수 있다.

유틸리티 타입을 전역적으로 사용하려면 이 방법이 가장 유용하다.

아래는 `Optional` 유틸리티 타입을 선언해서 사용하는 예시를 보여준다.

```ts
// src/index.d.ts
// 선택한 속성을 모두 옵셔널 속성으로 만드는 유틸리티 함수
type Optional<T extends object, K extends keyof T = keyof T> = Omit<T, K> &
  Partial<Pick<T, K>>;

// src/components.ts
type Props = { name: string; age: number; visible: boolean };
type OptionalProps = Optional<Props>; // { name?: string; age?: number; visible?: boolean;
type VisibleRequiredProps = Optional<Props, "name" | "age">; // { name?: string; age?: number; visible: boolean;
```

`declare type`로 타입을 선언하면 전역에서 사용할 수 있다.

```ts
declare type Nullable<T> = T | null;

const name: Nullable<string> = "jeon";
```

&nbsp;

`declare module`로 기존 모듈의 타입을 확장하거나, 외부 라이브러리의 타입을 정의해줄 수 있다.

```ts
const fontSizes = {
  xl: "30px",
  // ...
};

const colors = {
  gray_100: "#222222",
  gray_200: "#444444",
  // ...
};

const depths = {
  origin: 0,
  foreground: 10,
  dialog: 100,
  // ...
};

const theme = {
  fontSizes,
  colors,
  depths,
};

declare module "styled-components" {
  type Theme = typeof theme;
  export type DefaultTheme = Theme;
}
```

```ts
declare module "*.gif" {
  const src: string;
  export default src;
}
```

&nbsp;

`declare namespace`를 사용하면 `.env` 파일을 사용할 때 `process.env`로 설정값을 쉽게 불러오고 환경변수의 자동 완성 기능을 쓸 수 있다.

```ts
declare namespace NodeJS {
  interface ProcessEnv {
    readonly API_URL: string;
    readonly API_INTERNAL_URL: string;
    // ...
  }
}
```

이렇게 타입을 정의해두면 사용할 때 굳이 타입 단언을 해줄 필요가 없다.

```ts
function log(str: string) {
  console.log(str);
}

declare namespace NodeJS {
  interface ProcessEnv {
    readonly API_URL: string;
  }
}

// .env
API_URL = "localhost:8080";

log(process.env.API_URL);
// log(process.env.API_URL as string);
```

&nbsp;

`declare global`은 전역 변수를 선언할 때 사용한다.

전역 변수인 `Window` 객체의 스코프에서 사용되는 모듈이나 변수를 추가할 수 있다.

아래 코드는 iOS 웹뷰에서 JS로 네이티브 함수를 호출하기 위한 함수를 정의한 것이다. `declare global`로 `Window` 객체에 타입을 정의함으로써 함수 호출 시 자동 완성 기능을 활용할 수 있다.

```ts
declare global {
  interface Window {
    webkit?: {
      messageHandlers?: Record<
        string,
        {
          postMessage?: (parameter: string) => void;
        }
      >;
    };
  }
}
```

&nbsp;

### 5. declare와 번들러의 시너지

`declare global`로 전역 변수를 선언하는 과정과 번들러를 통해 데이터를 주입하는 절차를 함께 사용하면 시너지를 낼 수 있다.

```ts
// data.ts
export const color = {
  white: "#ffffff",
  black: "#000000",
} as const;

// type.ts
import { color } from "./data";
  type ColorSet = typeof color;

  declare global {
  const _color: ColorSet;
}

// index.ts
console.log(_color[“white”]);

// rollup.config.js
import inject from "@rollup/plugin-inject";
import typescript from "@rollup/plugin-typescript";

export default [
  {
    input: "index.ts",
    output: [
      {
        dir: "lib",
        format: "esm",
      },
    ],
    plugins: [typescript(), inject({ _color: ["./data", "color"] })],
  },
];
```

`data.ts`에서 색상을 정의하고, `type.ts`에서 색상 타입을 전역적으로 선언하고 있다. 롤업 번들러 설정에서 `inject` 모듈을 사용하여 `_color`에 해당하는 데이터를 삽입하고 있다.

`inject` 모듈은 import 문의 경로를 분석하여 데이터를 가져온다.

```ts
// import
import { color } from "./data"

// inject
inject({ _color: ["./data", "color"] }) // _color에 ./data 디렉터리의 color를 inject
```

&nbsp;

## 12.2 스크립트와 설정 파일 활용하기

프로젝트 관리할 때, 스크립트와 `tsconfig.json` 파일을 이용하는 팁에 대해 알아보자.

### 1. 스크립트 활용하기

1. 실시간 타입 검사

컴퓨팅 성능이 느리거나 프로젝트 규모가 커지면 IDE의 타입 에러를 알려주는 속도가 느려지는 경우가 있다.
그럴 땐 스크립트를 사용하여 실시간으로 에러를 확인할 수 있다.

```shell
npx tsc --noEmit --incremental -w
```

`noEmit`: JS 출력 파일을 생성하지 않도록 설정
`incremental`: 증분 컴파일(변경 사항이 있는 부분만을 컴파일)을 활성화하여 컴파일 시간을 단축
`w`: 파일 변경 사항을 모니터링하도록 설정

&nbsp;

2. 타입 커버리지 확인

아래 스크립트를 사용하면 현재 프로젝트의 타입 커버리지와 `any`를 사용하고 있는 변수의 위치를 알 수 있다.

```shell
npx type-coverage --detail
```

특히 TS로의 마이그레이션 중인 프로젝트나 레거시 코드가 많은 프로젝트를 다룰 때 타입 커버리지를 체크해서 리팩터링하기 위한 기반을 마련하는데 도움이 되는 정량적인 지표를 얻을 수 있다.

&nbsp;

### 2. 설정 파일 활용하기

`incremental` 속성을 `true`로 설정하면 증분 컴파일로 TS의 컴파일 속도를 높일 수 있다. 타입 검사 시간을 줄여준다.

1. `tsconfig.json` 설정

```json
// tsconfig.json
{
  "compilerOptions": {
    //...
    "incremental": true
  }
}
```

2. 스크립트 활용

```shell
npx tsc --noEmit --incremental --diagnostics
```

&nbsp;

### 3. 에디터 활용하기

IDE에서 개발을 진행하다보면, 정의된 타입이 있는 객체인데도 잘 불러오지 못하거나 자동 완성 기능이 동작하지 않는 경우가 있다.

VSCode에서는 `Restart TS Server` 명령어를 실행해서 TS 서버를 재실행하면 된다.

&nbsp;

## 12.3 타입스크립트 마이그레이션

### 1. 타입스크립트 마이그레이션의 필요성

JS로 개발된 기존 프로젝트를 반드시 TS로 마이그레이션할 필요는 없다.

새로운 설계를 바탕으로 타입을 정의하는 것이 더 수월하다면, 기존의 설계를 버리고 TS로 프로젝트를 새로 구축할 수도 있다.

비즈니스 요구 사항의 변화를 잘 수용할 수 있는 새로운 설계를 바탕으로 타입을 작성하는게 효율적일 수 있으며, 프로젝트의 규모나 내외부 여건을 종합적으로 고려해 마이그레이션 여부를 결정하는 것이 좋다.

&nbsp;

### 2. 점진적인 마이그레이션

진입 장벽과 프로젝트의 안정성을 위해 작은 부분부터 시작하여 점진적으로 마이그레이션을 진행하는 것이 좋다.

&nbsp;

### 3. 마이그레이션 진행하기

다음과 같은 단계를 거쳐 TS로의 마이그레이션을 진행하게 된다.

1. TS 개발 환경 설정, 빌드 파이프라인에 TS 컴파일러를 통합한다. `tsconfig` 설정은 다음과 같이 하는 것을 추천한다.

```json
// tsconfig.json
{
  "compilerOptions": {
    "allowJS": true,
    // TS에서 JS 함수 import 허용
    // JS에서 TS 함수 import 허용
    "noImplicityAny": false
    // 암시적 any 타입이 있을 때 오류가 발생하지 않도록 함
    // ...
  }
}
```

2. JS 파일을 TS 파일로 변환하여 필요한 타입과 인터페이스를 하나씩 정의하며 함수 시그니처를 추가한다.

3. 변환하는 작업이 끝나면 타입이 명시되지 않은 부분을 점검한다.

```json
// tsconfig.json
{
  "compilerOptions": {
    "allowJS": false,
    "noImplicityAny": true
  }
}
```

&nbsp;

## 12.4 모노레포

모노레포는 개별 프로젝트를 별도의 레포지토리로 관리하지 않고 하나의 레포지토리에서 여러 프로젝트를 관리하는 것을 말한다.

### 1. 분산된 구조의 문제점

독립적인 프로젝트는 각각 다른 Jest, Babel, ESLint, TS 등의 설정 파일을 별도로 구성하고 빌드 파이프라인 역시 독립적이다.

만약 이 상황에서 프로젝트끼리의 의존 관계가 생긴다면 각각의 프로젝트가 별도로 관리되는 탓에 한 번 수정하려면 수정이 필요한 프로젝트 코드를 모두 수정해야 하므로 생산성이 떨어지게 된다.

한 곳에서 프로젝트를 관리한다면 모든 프로젝트가 하나의 설정을 공유할 수도 있을 것이고, 의존 관계를 가지는 코드의 수정을 줄일 수 있어 편리할 것이다.

&nbsp;

### 2. 통합할 수 있는 요소 찾기

각각의 프로젝트 내에 공통으로 통합할 수 있는 요소를 찾는다. 공통화를 위한 코드 수정을 거친다.

&nbsp;

### 3. 공통 모듈화로 관리하기

npm과 같은 패키지 매니저를 통해 공통 모듈을 생성해서 관리하면 쉽게 모듈을 통해 코드를 재사용할 수 있고, 특정 기능의 변경에도 해당 모듈의 코드만 수정하면 된다.

하지만 여전히 공통 모듈의 변경이 사용하는 각각의 프로젝트에도 변경 사항을 불러올 수 있다.

또 새로운 모듈이 필요하다면 새로운 레포지토리를 하나 더 생성해 개발 환경을 설정하고 패키지 관리자를 통해 모듈을 배포해야 한다. 새로운 프로젝트를 시작할 때도 빌드를 위한 개발 환경 설정을 별도로 해주어야 한다.

&nbsp;

### 4. 모노레포의 탄생

예전에는 다양한 기능을 가진 프로젝트를 하나의 레포지토리로 관리하는 모놀리식 구조를 주로 사용했다. 모놀리식 구조는 코드 간의 직접적인 의존이 발생해 일부 로직 변경에도 전체 프로젝트에 영향을 줄 수 있었다. 모놀리식 구조는 설계 상의 복잡함과 빌드 및 배포 등에서 효율적이지 않았다.

효율적인 구조에 대한 수요로 거대 프로젝트를 작은 프로젝트의 집합으로 나누어 관리하는 폴리레포 방식과 하나의 레포지토리로 모든 것을 관리하는 모노레포 방식이 등장하게 되었다.

모노레포는 개발 환경 설정을 통합적으로 관리할 수 있어 불필요한 개발 환경 설정이나 코드 중복을 줄여줄 수 있다.

하지만 시간이 지나면서 레포지토리가 거대해질 수 있다. 그리고 하나의 레포지토리에 여러 팀의 이해관계가 얽혀있다면 소유권과 권한 관리가 복잡해질 수 있다. 따라서 각 프로젝트나 모듈의 소유권을 명확히 정의하고 규칙을 설정하는 과정이 필요하다.