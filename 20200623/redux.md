그동안 두차례 컴포넌트 구조화를 경험했습니다.

컴포넌트 수가 많지도 않음에도 불구하고 컴포넌트간에 부모를 거쳐서 props를 전달해주는 것이 마냥 쉽지만은 않았습니다.

만약 컴포넌트가 엄청나게 많아진다면 어떤 컴포넌트에 어떤 state가 있어야하고 서로 어떤방식으로 props를 내려줘야 할지가 더 어려워질 것입니다.

만약 외부에 글로벌한 state 저장소가 있고 컴포넌트는 특정 state가 필요할때만 그 저장소에서 해당 값을 꺼내쓴다면 얼마나 편할까요?

그걸 구현한 것이 바로 redux입니다.

리덕스를 사용하면 react 어플리케이션 외부에 상태관리를 전담해주는 스토어를 하나 만들고,

컴포넌트에서 필요한 state만 redux store에서 빼와서 사용하면 됩니다.

그렇게 하면 어떤 컴포넌트에서 어떤 state를 관리하고 어디에서 어디로 props를 내려줘야할지 등에 대한 고민을 하지않아도 되겠죠?

리덕스 코드를 보기 전에, 간단한 개념만 잡고 넘어가겠습니다.

리액트에서 사용하는 redux에는 다음과 같은 개념들이 있어요.

* 스토어 \(리듀서, 상태값\)

* 액션, 액션생성함수

* 디스패치

* connect 등등..

스토어는 말그대로 state를 관리하는 저장소입니다.

스토어 안에는 state와 reducer가 들어있습니다.

state는 말그대로 컴포넌트 어디서든 가져다 사용할 수 있는 글로벌한 상태값을 말하고,

reducer는 해당 state의 변화와 관련된 로직이 들어있는 함수입니다.

redux store는 특정 컴포넌트에서 어떤 액션이 왔을 때, 리듀서에 적힌 내용대로 state를 변화시킵니다.

따라서 state와 reducer는 세트라고 보시면 됩니다.

액션은 컴포넌트에서 스토어의 리듀서에게 요청하는 행위를 말합니다.

좀 더 풀어 말하면, "나는 액션을 이렇게 보낼테니 이 액션에 알맞게 상태값을 변화시켜 주세요" 라고 스토어에 요청하는 것입니다.

액션은 다음과 같이 생겼습니다.

```jsx
const CLICK = "CLICK"
```

리듀서는 항상 action의 type을 보기때문에 보통 액션 생성함수라는 것을 만들어서 사용함으로써 매번 type에 action을 만드는 것을 대신해줍니다.

액션생성함수는 다음과 같이 생겼습니다.

```jsx
const click = () => ({type: CLICK})
```

따라서 click이라는 함수를 실행하면 다음과 같은 결과가 반환되는 것이죠

```jsx
// 클릭을 호출하면
click()

// 다음과 같이 반환됩니다
{type: "CLICK"}
```

스토어안의 리듀서는 액션을 받아서 타입을 확인하고 해당 타입에 해당되는 로직에 따라 스토어의 상태값을 변화 시켜줍니다.

```js
function reducer(state, action){
    // 액션의 타입을 보고 해당 case로 가서 로직을 실행함.
    switch(action.type){
        case A: 

        case B:

        case C:

        ...

        default:
    }
}
```

그렇다면 컴포넌트는 특정액션을 어떻게 스토어에 보낼까요? 그건 디스패치라는 함수를 통해서 가능합니다.

```jsx
// button을 클릭했을때, click 액션생성함수가 반환하는 값인 {type: "CLICK"}을 store의 리듀서로 보낸다(디스패치한다)
<button onClick={() => store.dispatch(click())}>
```

이렇게하면 스토어안에 있는 리듀서는`{type: "CLICK"}`이라는 값을 받게되는 것이죠.

리액트에서 리덕스를 사용하때는 직접 store.dispatch를 적어주지는 않습니다.

다만 내부적으로 컴포넌트에서 액션을 보낼때 그걸 '디스패치'한다고 하기때문에 반드시 알고계셔야할 용어입니다.

아마 리덕스가 처음이시라 새로 쏟아지는 용어들 때문에 여러모로 헷갈리실텐데요, 아래 그림을 보면서 다시 이해해봅시다.

![](/assets/redux-process.png)

> **"**_리덕스 스토어에는 state와 reducer가 들어가고 state를 바꾸기 위해서는 reducer를 작동시켜야 했지"_
>
> _"reducer안에는 state를 어떤식으로 변화시킬지에 대한 여러가지 로직이 들어있는데, 그러면 reducer안의 특정 로직을 어떻게 작동시켰지?"_
>
> _"아 맞다 action을 dispatch해주면 reducer가 해당 action의 type을 찾아서 그에 알맞게 state를 변화시켜 주었지"_
>
> 등등 위 그림 각각의 구성요소가 무엇인지 먼저 이해해보세요!

이제 대강 개념을 잡으셨다면 제가 미리 준비한 템플릿을 clone해주세요.

```js
git clone https://github.com/react-nodejs-tutor/redux-template.git
```

리덕스를 사용하려면 우선 스토어를 하나 만들어야겠죠?

하지만 처음부터 index.js에 store를 만들지 않고 리듀서파일부터 작성을 해줍니다.

store를 만들때에는 리듀서를 함께 넣어줘야 오류가 나지 않기 때문입니다.

리덕스 공식문서에서는 액션, 액션생성함수와 리듀서를 각각 다른 파일에 작성하곤 하는데,

우리는 ducks패턴이라는 것을 사용할 것이기 때문에 액션, 액션생성함수, 그리고 리듀서를 한 파일\(모듈\)에 묶어서 작성을 하겠습니다.

yarn start를 통해 개발서버를 실행시키면 가장 위에 먼저 보이는 카운터부터 완성을 해보겠습니다.

```js
// src/store/modules/counter.js

// 액션
// 모듈이름을 앞에 붙여줌으로써 액션명 중복방지
const INCREMENT = 'counter/INCREMENT';
const DECREMENT = 'counter/DECREMENT';

// 액션생성함수
export const increment = () => ({ type: INCREMENT });
export const decrement = () => ({ type: DECREMENT });

// 초기 상태값
const initialState = {
    number: 0
};

// 리듀서
export default function counter(state = initialState, action) {
    switch (action.type) {
        case INCREMENT:
            return {
                ...state,
                number: state.number + 1
            };

        case DECREMENT:
            return {
                ...state,
                number: state.number - 1
            };

        default:
            return state;
    }
}
```

리듀서를 작성할때는 ...state로 직접 불변성을 유지해줘야 한다는 것을 잊지마세요!

setState\(\) 함수에서는 불변성 관리를 자동으로 해주었기 때문에 우리가 직접 ... spread를 사용할 필요가 없었어요.

하지만 리듀서에서는 꼭! 해주어야 합니다. 그래야 리덕스 개발자도구를 통해서 되돌아가기 기능을 사용하면서 상태변화를 볼 수가 있어요.

위와 같이 counter모듈을 작성했다면 combineReducers를 통해서 묶어줍니다.

지금은 모듈이 한개지만 나중에는 여러개가 될 수 있으니까요.

따라서 index.js에서 스토어를 만들기전에, 여러 리듀서들을 한번에 합쳐주는 작업을 해줄게요.

```js
// src/store/modules/index.js

import { combineReducers } from 'redux';
import counter from './counter';

export default combineReducers({
    counter
});
```

이렇게 여러 모듈을 하나로 묶으면 스토어에 한번에 집어넣을 수 있게 됩니다.

스토어를 만들고 리듀서를 집어넣어 볼까요?

```js
// src/index.js

import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import { createStore } from 'redux';
import rootReducer from './store/modules';

const store = createStore(rootReducer);
//상태값을 콘솔에 찍어볼 수도 있음
//console.log(store.getState());

ReactDOM.render(<App />, document.getElementById('root'));

serviceWorker.unregister();
```

이렇게 하면 스토어가 만들어집니다. 여기서 store.getState\(\)를 콘솔에 찍어보면 스토어안의 값을 확인할 수가 있어요.

여기까지는 아직 우리 리액트 프로젝트에 스토어를 적용한 것이 아닙니다.

또한 우리에게는 리덕스 개발자도구라는 것이 있기 때문에 더 쉽게 확인할 수가 있습니다.

우리 프로젝트에 리덕스를 적용해보고 개발자도구까지 달아볼까요?

```js
// src/index.js

import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import { createStore } from 'redux';
import rootReducer from './store/modules';
import { Provider } from 'react-redux';

const devTools =
    window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__();
const store = createStore(rootReducer, devTools);

ReactDOM.render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
);

serviceWorker.unregister();
```

다음시간에는 프로젝트에 연결된 리덕스 스토어에서 값을 빼와서 사용해보겠습니다.

