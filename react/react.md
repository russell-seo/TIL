
 # Virtual DOM
 
 
  시간이 지나면서 jQuery 는 조금씩 한계를 맞이하게 됩니다. 대표적으로 facebook 이나 Instagram 과 같이 규모가 큰 웹 어플리케이션이 나오면서 쉽게 개발할 수 있는 것도 중요하지만,
  
  사용자와의 interaction이 많은 웹 앱 특성상 성능이 가장 중요했다. 사용자와의 interaction이 많은 웹 앱 특성상 성능이 가장 중요했습니다. 다음과 같은 이유들로 jQuery 에 대한 회의적인 시선이 나오게 됩니다.
  
  첫째로, jQuery는 vanila에 비해 로드해와야 할 패키지가 많아 간단한 코드를 작성하더라도 성능적으로 떨어집니다.
  
  둘째로, jQuery는 조칵칼로 DOM을 깍아서 만드는 것에 비유할 수 있어서 대규모 앱에서 유지보수가 쉽지 않습니다. `모듈화나 컴포넌트화` 와는 거리가 있기 때문이죠.
  
  셋째로, interactive Web, 즉 위에서도 언급했듯 웹 어플리케이션으로 불리는 요즘의 웹에선 사용자에 의한 동적인 DOM 조작이 잦은데, 이때 jQuery를 이용하면, 배치와 화면표시에 많은 연산을 발생시키기 때문에 브라우저의 성능이 낮아지는 문제가 있습니다.
  
  그 흐름을 읽고 Virtual DOM 이라는 개념이 나오게 됩니다.



# React and React Hook

(함수형 컴포넌트 는 렌더링 될때 → Component 함수 호출 → 모든 내부 변수 초기화)

# React 란?

→ 

- 자동으로 업데이트 되는 UI, React 는 UI 기능만 제공한다.
- 즉 전역상태 관리나, 라우팅, 빌드시스템 등을 개발자가 구축.
- create-react-app 을 통해 위의 단점들을 보완했다.
- 만약 react를 사용하지않고 상태값을 변하게 할려면 DOM을 직접 수정해야한다.
- state는 불변 변수로 관리
- virtual Dom 은 이전 UI 상태를 메모리에 유지해서 변경된 부분만 실제 Dom에 반영해서 UI 업데이트를 빠르게 한다.
- 여러 개의 상태값 변경 요청을 배치로 처리한다.

## 바벨, 웹팩

→ 

- 리액트에서 JSX 문법을 createElement 함수를 호출하는 코드로 바벨을 사용
- 웹팩을 사용하는 가장 큰 이유 → 모듈 시스템(ESM, commonJS)를 사용하고 싶어서

# React Hook 란?

→ 

- 기존에는 클래스형 컴포넌트로 사용해왔다면, 리액트 16.8에 새로 React Hook 이 추가되면서 함수형 컴포넌트를 사용하게 된다.
- 

## 🔎 useState

→

- State : 컴포넌트의 상태값, 즉 useState는 컴포넌트의 상태값을 간편하게 업데이트 시킴.
- 상태값 변경 함수는 비동기이면서 배치(batch)로 처리된다.
    - 만약 동기로 처리하면 하나의 상태값 변경함수가 호출 될 때 마다 화면을 다시 그린다. 성능 이슈가 발생한다.

```jsx
const [state, setState] = useState(초기값);

-> state, setState를 배열 형태로 return
```

- 초기값은 state 에 할당되고 state를 변경할려면 setState 함수를 호출해서 변경해야한다.

## 🔎 useEffect

→

- 부수효과를 처리하는 훅 이다.(부수효과 → 외부의 상태를 변경)
    - 예를 들어, 서버 API호출, 이벤트 핸들러 등록 등
- 렌더링 결과가 실제 DOM 에 반영되고 비동기로 호출된다.
- 인자로 콜백 함수(다른 인자로부터 전달된 함수)를 받는다.
    - 렌더링 될때 마다 실행(컴포넌트가 맨 처음 실행 될때, 다시 렌더링 될때)

```jsx
1. 형태
useEffect(() => {
	//작업..
});
```

- 두번째 형태는 인자로 배열을 받는다. → Dependency Array 라고 한다.
    - 화면에 첫 렌더링 될때 실행
    - value 값이 바뀔 때 실행

```jsx
 2. 형태
useEffect(() => {
	//작업...
}, [value]);
```

- 만약 Dependency Array에 빈 배열이 들어가면. 첫 화면에 렌더링 될 때 만 실행.

```jsx
useEffect(() => {
	//작업..
}, []);
```

- 추가로 의존성배열에 원시타입이 아니라 Object 라면 React는 다른 참조값(메모리상)을 가지고 있다고 알고있어 다른 state가 변경되면서 reRendering 될때 useEffect가 호출된다.
    - 이럴 때는 useMemo를 사용하여 Memoization 해주면 된다.
    
- Clean Up - 정리
    - 예를 들어 타이머를 작동시키고, 타이머를 껐을 때 타이머가 정지되는 작업을 수행해주어야 한다.
    - 또는 이벤트리스너를 등록했으면 이도 정리해주는 작업을 해야한다.
    - unMount 될때 혹은 다음 렌더링 시 불린 useEffect 가 실행되기 이전에 실행된다.
    
    ```jsx
    useEffect(() => {
    	//작업 ..
    
    	return () => {
    	//정리 작업..
    	}
    
    });
    ```
    

## 🔎 useRef

→

- useRef를 호출하면 ref Object를 반환해준다.
- ref Object의 값은 바꿀 수 있다.

```jsx
const ref = useRef(value)

-> Ref Object 는 아래와 같다.

{ current: value }

ref.current = "hello"

{ current: "hello" } -> 이처럼 ref 값을 바꿀 수 있다.
```

- 반환된 ref 는 component의 unMount 되기 전까지 값이 유지가 된다.

### useRef 는 두 가지 형태로 사용된다.

1. 저장공간(state와 비슷한 용도)
    - 예를 들어 State 의 변화 → 렌더링 → 컴포넌트 내부 변수 초기화
    - But Ref에 값을 저장해 놓으면 Ref가 변화 → No 렌더링 → 변수들의 값이 유지가됨
    - 즉, 변경시 렌더링을 발생시키지 말아야 할 값을 다룰때 편리하다.
2. DOM 요소에 접근
    - 예를 들어 focus() 같은 작업.
    - vanlia js 의 Document.querySelector()와 비슷하다.
    
    ```jsx
    const inputRef = useRef();
    
    <input ref={inputRef} />
    ```
    

## 🔎 useContext

→

- 전역 컴포넌트에 전달 가능하다.
- Context를 사용하면 컴포넌트를 재사용하기 어려워 질 수있기 떄문에 꼭 필요할 때만
- Prop drilling 을 피하기 위한 목적이라면 Component Composition(컴포넌트 합성)을 먼저 고려해보자.

ValueContext.js

```jsx
import { createContext } from "react";

export const Value = createContext(null);
//null -> 초기값
```

```jsx
import { ValueContext } from "./ValueContext.js";

function App(){

		

		return (
			<ValueContext.Provider value = {'사용자'}>
				//여기 안에있는 하위 컴포넌트는 모두 value 값에 접근 가능
			</ValueContext.Provider>	
		);

};
```

```jsx
import { ValueContext } from "./ValueContext.js";

const User = () => {

	const user = useContext(ValueContxt);

	//위와 같이 정의한 Context component를 useContext의 인자로 가져온다.
}
```

## 🔎 useMemo

→

- Memo 는 Memoizaiton(동일한 값을 리턴하는 함수를 반복적으로 호출해야 한다면 맨 처음 값을 계산해 메모리에 저장하고 다시 사용해야한다면 캐시에서 꺼내서 사용한다.)

```jsx
const value = useMemo(() => {
	return 함수(); -> Memoization할 값을 리턴하는 함수
}, [item]); -> 두번째 인자는 useEffect와 동일하게 의존성배열이 변경될 때만 Memoization을 한다.
```

- 만약 useMemo의 return 값이 원시타입 이라면 같은

### 🔎 useCallback

→

- 위의 useMemo와 비슷한 개념으로 Memoization 하는 것이다. 하지만 다른 것이 있다면
- useMemo는 return 하는 함수만 Meomization 했다면, useCallback은 의존성배열 까지도 캐싱한다.

```jsx
const calculate = useCallback((num) => {
	return num + 1;
}, [item])
```

- 의존성배열의 상태가 변하지 않는 이상 초기화되지 않고 메모리상에 위 예시인 calculate 함수를 가지고 있는다.
- 의존성배열이 바뀌면 다시 초기화 된다.

### useMemo vs useCallback

- useMemo 는 특정 결과 값을 재사용 할 때 사용 한다.
- useCallback 은 특정 함수를 재사용 할 때 사용한다.
