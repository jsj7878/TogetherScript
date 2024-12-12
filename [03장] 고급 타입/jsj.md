# 3장 (고급 타입)

## 3.1 타입 스크립트 만의 독자적 타입 시스템

- 타입 스크립트의 타입 시스템이 가지고 있는 개념은 모두 자바 스크립트에서 기인했지만, 단지 자바 스크립트에서 표현할 수단이나 필요성이 없었음
- 자바 스크립트의 타입 개념 + 정적 타이핑 을 통해 타입 스크립트의 타입 시스템이 구축됨
- 타입 스크립트의 타입 계층 구조
    
    ![image.png](3%E1%84%8C%E1%85%A1%E1%86%BC%20(%E1%84%80%E1%85%A9%E1%84%80%E1%85%B3%E1%86%B8%20%E1%84%90%E1%85%A1%E1%84%8B%E1%85%B5%E1%86%B8)%2015af78a2bff180329d8ac9908892f09f/image.png)
    
- any 타입
    - 존재하는 모든 값을 오류 없이 받을 수 있음
    - any를 쓰면 자바 스크립트처럼 타입에 대해 동적 타이핑이 진행되는 것과 비슷하기에 타입 스크립트의 정적 타이핑이 가지는 장점이 사라지게 됨
    - any를 쓰는 것은 지양해야 하지만 어쩔 수 없이 사용해야되는 경우가 있음
    - any를 어쩔수 없이 써야하는 경우
        - 개발 단계에서 임시로 값을 지정해야 할때
            
            매우 복잡한 구성 요소로 이루어진 개발과정에서 추후 값이 변경되거나 세부 항목의 타입이 확정되지 않은 경우 any로 값을 지정하고 개발을 계속 할 수 있다. 하지만 이는 임시 방편이고 타입에 대한 세부 내용이 나오면 다른 타입으로 대체해야한다!
            
        - 어떤 값을 받아올지, 넘겨줄지 정할 수 없을 때
            
            API 요청. 응답 처리, 콜백 함수 전달, 외부 라이브러리 사용 등 어떤 타입을 주고 받을지 특정하기 힘들 경우 any타입을 써야 할 수 도 있다.
            (우정이네는 저번에 외부 라이브러리 썼을때 어떻게 했지? 다 뜯어봤나?)
            
        - 값을 예측 할 수 없을 때 암묵적으로 사용
            
            브라우저의 Fetch API처럼 다양한 값을 반환하는 API를 사용하는 경우
            
            Fetch API의 일부 메서드는 반환을 any타입으로 하기도 한다…
            
    - any타입을 쓰면 실수나 악의적으로 다른 값을 넘겨도 컴파일러 단계에서 확인할 수 없게된다. 이는 런타임에 치명적인 오류를 발생시킬 수 있다.
    - any타입은 개발자에게 편의성과 확장성을 제공하지만 해당 값을 컨트롤하기 위해선 파악해야할 정보 또한 많다. any를 쓰려면 해당 값과 값을 다루는 흐름에 대해 이해하고 쓰자.
- unknown 타입
    - 어떤 값이 할당 될지 아직 모르는 상태의 타입
    - any와 유사하게 모든 타입의 값이 할당 될 수 있지만 any 타입을 제외한 다른 타입으로 할당은 안된다.
    - any와 unknown 타입 비교
        
        
        | any | unknown |
        | --- | --- |
        | -어떤 타입이든 any 타입에 할당 가능
        - any 타입은 어떤 타입으로도 할당 가능 (never제외) | -어떤 타입이든 unknown 타입에 할당 가능
        -unknown 타입은 any 타입 외에 다른 타입으로 할당 불가능 |
    - unknown을 사용하면 할당시에는 문제가 없지만 함수 뿐만 아니라 객체 내부에 접근하는 모든 시도에서 에러가 발생한다.
    - 어떤 타입이 할당 되었는지를 알 수 없기 때문에 값을 가져오거나 내부 속성에 접근할 수 없다.
    - 어떤 값이든 올 수 있지만 엄격하게 타입 검사를 강제하는 의도를 가지고 있다고 볼 수 있다.
    - any 타입을 사용하고 후에 특정 타입으로 수정하는 것을 누락하게 되면 런타임에 버그가 발생하게 되는데, 이 상황을 보완하기 위해 unknown을 사용하여 타입 검사를 강제하고 타입이 식별된 후에 사용할 수 있게 하는 것이다.
    - 보통은 any보다 unknown을 통해 타입을 정하기 어려운 상황을 해결한다.
- void 타입
    - 함수가 어떤 값을 반환하지 않는 경우 void로 지정하여 사용한다.
    - 함수 말고 변수에도 할당이 가능한데, 이 경우에는 null과 undefined만 할당이 가능하다. 하지만 옵션 설정들으로 인한 null 할당이 불가능한 경우, 명시적인 의미를 부여하는 관점에서 봤을 때 void 보다 해당 타입 키워드를 통해 타입을 지정하는 것이 더 바람직하다.
- never 타입
    - 값을 반환할 수 없는 타입
    - 값을 반환할 수 없는 예
        - 에러를 던지는 경우
            
            런타임에 의도적으로 에러를 발생시키고 캐치하는 경우
            
        - 함수가 무한 루프를 도는 경우
            
            무한 루프의 경우 함수가 종료되지 않기에 값을 반환할 수 없다.
            
    - never 타입은 모든 타입의 하위 타입이다. (자신을 제외한 어떤 타입도 never에 할당 불가능)
    - 조건부 타입을 결정할 때 특정 조건을 만족하지 않는 경우 엄격한 타입 검사 목적으로 never를 사용하곤 한다.
- Array 타입
    - 배열 타입을 가리키는 키워드
    - 자바 스크립트와 다르게 하나의 배열에는 하나의 타입만 사용해야함
    - 선언 방식
    
    ```tsx
    const array : number[] = [1,2,3];
    const array : Array<number> = [1,2,3];
    ```
    
    - 유니온 타입 또한 사용이 가능하다.
    
    ```tsx
    const array : Array<number| string> = [1,  "string"]
    const array : number[]: string[] = [1, "string"]
    ```
    
    - 타입 스크립트에서 배열의 길이 까지는 제한 할 수 없지만 대괄호를 사용한 튜플을 통해 원소의 개수 또한 지정이 가능하다.
    - 튜플 사용 예시
    
    ```tsx
    let tuple : [number] = [1];
    tuple = [1,2]; //불가능
    tuple = [1, "string"] //불가능
    
    //이렇게 여러 타입 혼합도 가능하다
    let tuple : [number, string, boolean] = [1,"string", true] 
    ```
    
    - 배열은 허용되지 않은 타입이 섞이는 것을 방지하며 타입 안정성 또한 제공한다.
    - 튜플은 이에 더불어 길이까지 제한하여 원소 개수와 타입을 보장한다.
    - 튜플 사용 예시
        - useState
            - 튜플 타입으로 첫 원소는 상태값, 두번째 원소는 해당 상태를 조작할 수 있는 setter를 의미한다.
            - useState는 튜플 타입으로 배열 원소의 자리마다 명확한 타입, 의미를 부여하기 때문에 오류를 방지 할 수 있다.
        - 배열과 혼합
            
            ```tsx
            //첫번째 number자리에 400, 두번째 string 자리에 Bad Request, 
            //이후에는 string 배열로 string 원소를 개수 제한 없이 받을 수 있다.
            const arrayTuple : [number, string, ...string[]] 
            = [400, "Bad Request", "/users/:id", "users/:uid"..];
            ```
            
        - 옵셔널 프로퍼티
            
            ```tsx
            //세번째 자리의 원소는 있어도 되고 없어도 된다.
            const optionalTuple : [number, number ,number?] = [1,2];
            ```
            
- enum 타입
    - 열거형을 정의할 수 있으며 기본적으로 각 멤버의 값을 0부터 추론한다.
    - 각 멤버의 값을 명시적으로 할당할 수도 있으며 일부 멤버에 값을 할당하지 않아도 이전 멤버의 값을 기준으로 1씩 늘려가며 자동으로 할당한다. (이전 멤버가 숫자인 경우)
    - 문자열끼리 비교하면서 하는 것 보다 enum을 써서 문자열 상수 처럼 쓰는 것이 더 유용하다.
        
        ```tsx
        enum ItemStatusType{
        	HOLD = "HOLD"
        	READY = "READY"
        	...
        }
        
        const checkItemStatus = (itemstatus: ItemStatusType) =>{
        	switch(itemstatus){
        	//열거형 비교
        		case ItemStatusType.HOLD:
        		case ItemStatusType.READY:
        		default:
        			...
        	//문자열 상수 비교
        		case "HOLD":
        		case "READY":
        	}
        }
        ```
        
    - 위의 코드 처럼 타입을 문자열로 지정된 경우보다 열거형으로 가진 경우 얻을 수 있는 이점
        - 타입 안정성 : ItemStatusType 에 명시되지 않은 다른 문자열은 인자로 받을 수 없으며 이로 인해 타입 안정성이 우수해진다.
        - 명확한 의미 전달과 높은 응집력 : ItemStatusType 타입이 다루는 값이 무엇인지 명확하고, 아이템 상태에 대한 값을 모아놨기에 응집력도 뛰어나다.
        - 가독성: 응집도가 높기때문에 어떤 의미인지 쉽게 이해할 수 있다.
    - const enum
        - enum은 역방향으로도 접근할 수 있다.
            
            ```tsx
            enum person {
            	JSJ, //0
            	OJH, //1
            	KHJ, //2
            	JWJ, //3
            	YSJ  //4
            }
            
            person.JSJ //0
            person["YSJ"] //4
            person[3] //JWJ
            ```
            
        - 하지만 할당된 값을 넘어서는 범위로 역방향으로 접근하는 것을 타입 스크립트에서 막아주지 않는다.
        - 이런 동작을 막기위해 const enum을 사용하여 역방향으로의 접근을 허용하지 않게 한다.
        - const enum으로 선언해도 숫자 상수의 경우 선언 값 외의 값을 접근할 때 방지하지 못한다. 문자열 상수 방식은 미리 선언하지 않은 멤버로 접근을 방지하므로 문자열 상수 방식을 사용하는 것이 숫자 상수 방식보다 더 안전하다.
        - enum은 타입, 값 공간에서 모두 사용 되기 때문에 타입 스크립트 코드가 자바 스크립트로 변환될 때 즉시 실행 함수 형식으로 변경이 된다. 이는 일부 번들러에서 즉시 실행 함수 형식으로 변경된 값을 사용하지 않는 코드로 인식을 하지 못하여 트리 쉐이킹 과정에서 인지가 안될 수 있다. 따라서 불필요한 코드의 길이가 증가할 수 있기 때문에 const enum이나 as const assertion을 통해 해결한다.
        
        > ❓ 트리 쉐이킹이란?
        > 
        > 
        > 자바 스크립트에서 사용하지 않는 코드를 제거하는 방식.
        > 
        > 코드를 짜다보면 디펜던시 같은 것들이 계속 추가되면서 자바 스크립트 애플리케이션의 크기를 늘리고 전송 속도 등이 느려지게 된다. 오래되거나 안쓰는 디펜던시를 지워도 코드에서는 제거되지 않았을 수도 있기에 트리 쉐이킹 방식 등을 사용하여 사용하지 않는 코드를 지우고 애플리케이션의 크기를 줄이고자 한다.
        > 
        > 참고 자료
        > 
        > - [https://ui.toast.com/weekly-pick/ko_20180716](https://ui.toast.com/weekly-pick/ko_20180716)
        > - [https://developer.mozilla.org/ko/docs/Glossary/Tree_shaking](https://developer.mozilla.org/ko/docs/Glossary/Tree_shaking)
        

## 3.2 타입 조합

- 교차 타입
    - 여러가지 타입을 결합하여 하나의 단일 타입으로 만들 수 있음.
    - 기존에 있는 다른 타입 들을 합쳐서 해당 타입의 모든 멤버를 가지는 새로운 타입을 생성
    - &을 사용하여 표기하며 생성된 단일 타입에는 타입 별칭(type alias)을 붙일 수도 있다.
    
    ```tsx
    type ProductItem = {
    	id : number,
    	name : string,
    	price : number
    	...
    };
    
    //이렇게 타입을 선언하면 newProduct타입의 변수는 ProductItem의 모든 멤버 + value까지 가짐
    type newProduct = ProductItem & {value : number}
    ```
    
- 유니온 타입
    - 타입 중 하나가 될 수 있는 타입
    - |  을 사용하여 표기하며 타입 별칭 또한 붙일 수 있다.
    - 타입 중 하나가 될 수 있는 타입이기 때문에 유니온 타입의 변수는 해당 타입 들의 속성에 접근 할 수 는 없다. (다른 타입일 수 도 있으니까)
- 인덱스 시그니처
    - 특정 타입의 속성 이름은 알 수 없지만 속성 값의 타입을 알고 있을 때 사용하는 문법
    - 인터페이스 내부에 `[Key : K] : T` 꼴로 타입을 명시해주면 됨 
    (해당 타입의 속성 키는 모두 K타입이고 속성값은 모두 T 타입을 가져야 한다.
    - 인덱스 시그니처 선언 이후 다른 속성을 추가로 명시할 때는 추가 속성이 인덱스 시그니처에 포함되는 타입이어야 한다.
    
    ```tsx
    interface IndexSignatureEx{
    	[key: string] : number | boolean;
    	length : number;
    	isValid : boolean;
    	name : string; //에러 발생
    }
    //[key: string]으로 인해 키 값의 타입이 string인 속성들은 number 혹은 boolean값을 가져야함
    //name이라는 string 타입의 키를 가지고 있는데 이 속성이 number나 bool이 아닌 string을 가지려 
    //해서 에러가 발생함
    ```
    

- 인덱스드 액세스 타입
    - 다른 타입의 특정 속성이 가지는 타입을 조회하기 위해 사용된다.
    
    ```tsx
    type Example = {
    	a: number;
    	b: string;
    	c: boolean;
    };
    
    type IndexedAccess = Example["a"];
    type IndexedAccess2 = Example["a"|"b"]; //number|string
    type IndexedAccess3 = Example[keyof Exmaple]; //number|string|boolean
    
    type ExAlias = "b"|"c";
    type IndexedAccess4 = Example[ExAlias]; //string|boolean
    ```
    
    - 배열의 요소 타입을 조회하기 위해 사용하는 경우도 있다.
    
    ```tsx
    const PromotionList = [
    	{type: "product", name:"chicken"},
    	{type: "product", name:"pizza"},
    	{type: "card", name:"cheer-up"},
    ];
    
    type ElementOf <T> = typeof T[number]; 
    //typeof T : 타입을 가져옴
    //[number] : 배열이나 튜플 타입에서 숫자 인덱스로 접근했을 때 얻는 타입
    //따라서 typeof T[number]를 통해 T의 인덱스에 접근하여 해당 인덱스 값에 속하는 타입을 가져옴
    //근데 특정 숫자가 아닌 number로 했기 때문에 사실 모든 인덱스 값이 가지고 있는 타입을 가져옴
    
    type PromotionItemType = ElementOf<PromotionList>;
    //이로 인해 PromotionItemType은 {type: string, name: string}이 됨
    
    //이 코드 돌아가게 고쳐봄
    const PromotionList = [
    	{type: "product", name:"chicken"},
    	{type: "product", name:"pizza"},
    	{type: "card", name:"cheer-up"},
    ];
    
    type ElementOf <T extends readonly any[]> = T[number]; 
    //[number]를 쓰려면 T가 배열이나 튜플 형태를 가져야되기 때문에 extends로 특정 타입을 가진 
    //기본 값을 추가해야함
    
    type PromotionItemType = ElementOf<typeof PromotionList>;
    //PromotionList가 값 형태이기 때문에 <>안에 typeof를 쓰지 않으면 안됨
    //<> 안에는 타입이 들어가야 하니까..
    
    const chk : PromotionItemType = {type: "qwe", name : "chekc"};
    
    console.log(chk)
    
    ```
    

- 맵드 타입
    - 자바스크립트의 map 메서드 처럼 인덱스 시그니처 문법을 사용해서 다른 타입을 기반으로 한 타입을 선언할 때 사용하는 문법
    - 반복적인 타입 선언을 효과적으로 줄일 수 있다.
    - 수식어를 적용하여 매핑할 수도 있다.
        - readonly : 읽기 전용으로 만들고 싶을 때
        - ? : 옵셔널 파라미터로 만들고 싶을 때
        - 또한 기존 타입의 readonly나 ?앞에 -를 붙이면 해당 수식어를 제거한 타입을 만들 수 있다.
    - as 키워드를 사용하여 키를 재 지정할 수 있다.
- 템플릿 리터럴 타입
    - 자바 스크립트의 템플릿 리터럴 문자열을 사용하여 새로운 문자열 리터럴 타입을 선언
    
    ```tsx
    type Stage =
    | "init"
    | "select-image"
    | "edit-image"
    | "capture-image"
    type StageName = `${Stage}-stage`;
    //'init-stage'|'select-image-stage;| ...
    ```
    

> ❓문자열 리터럴 타입은 뭐고 언제 쓰이지?
문자열 리터럴 타입은 문자열의 특정 값을 타입으로 지정하는 것
> 
> 
> ```tsx
> type Fruit = "apple" | "banana" | "cherry";
> 
> let myFruit: Fruit;
> 
> myFruit = "apple"; // 유효
> myFruit = "banana"; // 유효
> myFruit = "cherry"; // 유효
> 
> myFruit = "orange"; // 오류: "orange"는 'Fruit' 타입에 할당할 수 없음
> ```
> 
> 사용자가 선택할 수 있는 문자열 값을 미리 정의해두고 그 외의 값은 오류로 처리할 수 있음
> 함수의 매개변수로 정확한 문자열만 허용하도록 만들 수 있음
> 템플릿 리터럴 타입을 통해 동적인 문자열 타입을 추가할 수 있음
> 
- 제네릭
    - 특정되지 않고 일반화된 데이터 타입
    - 함수, 타입, 클래스 등에서 내부적으로 사용할 타입을 미리 정해두지 않고 타입 변수를 사용해서 해당 위치를 비워두었다가 실제로 그 값을 사용하게 될 때 외부에서 타입 변수 자리에 타입을 지정하여 사용
    - 함수, 타입, 클래스 등 여러 타입에 대해 따로따로 정의하지 않아도 되어서 재사용성이 크게 향상됨
    - `<T>`처럼 꺽쇠괄호 안에 정의되며 사용할 때는 꺽쇠괄호 안에 원하는 타입을 넣어주면 됨
    - any와 다른 점
        - any타입은 어떤 타입이던지 전부 다 받음 (타입 정보가 없음)
        - 제네릭의 경우 어떤 타입이던지 다 될 수 있지만 생성 시점에 원하는 타입으로 특정이 가능함.
        - 배열 생성할 때도 any는 배열 안에 타입 값이 다양해질 수 있지만 제네릭의 경우 전부 동일한 타입임
    - 또한 특정 요소 타입을 알 수 없을 때는 제네릭에 기본 타입을 추가할 수 있음 (extends)
        
        ```tsx
        interface SubmitEvent<T = HTMLElement> extends SyntheticEvent<T> {...
        ```
        
    - 어떤 타입이든 될 수 있기 때문에 특정 타입에서만 존재하는 멤버를 참조하려고 하면 안됨
    - 제네릭을 사용할 때 파일 확장자가 tsx인 경우 화살표 함수에 제네릭을 사용하면 안됨
    tsx 는 타입 스크립트 + JSX이므로 제네릭의 꺽쇠와 태그의 꺽쇠를 혼동하여 문제가 생김