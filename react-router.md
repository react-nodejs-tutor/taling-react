우리가 '싱글페이지 어플리케이션'이라고 부르는 이유는 어플리케이션의 페이지가 하나이기 때문이 아니라 유저가 처음 접속했을때 구체적인 data를 제외한 정적파일을 모두 불러오기 때문입니다.

이제 리액트 라우터로 페이지를 여러개로 나누어 유저가 접속하는 url마다 다른 data를 보여줄 수 있게 해볼게요.

먼저 리액트 라우터를 설치해주세요. \(프로젝트를 새로 만들어주세요\)

```js
yarn add react-router
```

설치한 라우터를 적용하기 위해서 index.js에서 BrowserRouter를 이용해 App을 감싸줍니다.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import { BrowserRouter } from 'react-router-dom';

ReactDOM.render(
    <React.StrictMode>
        <BrowserRouter>
            <App />
        </BrowserRouter>
    </React.StrictMode>,
    document.getElementById('root')
);

serviceWorker.unregister();
```

이제는 각 라우터에 구현할 페이지를 구현해볼게요.

Main컴포넌트와 Sub컴포넌트를 만들고 각각 다른 url에 연결해줍시다.

```js
// src/Main.js

import React, { Component } from 'react';

class Main extends Component {
    render() {
        return (
            <div>
                <h1>메인화면 입니다.</h1>
            </div>
        );
    }
}

export default Main;
```

```js
// src/Sub.js

import React, { Component } from 'react';

class Sub extends Component {
    render() {
        return (
            <div>
                <h2>서브화면입니다.</h2>
            </div>
        );
    }
}

export default Sub;
```

**본격적으로 라우트를 사용해볼텐데요, 사용자가 요청하는 주소에 따라 다른 컴포넌트를 보여주겠습니다.**

이 작업을 할때는 Route컴포넌트를 사용하고 알맞은 props를 넣어줍니다.

```js
<Route path="주소" component={보여주고싶은 컴포넌트}>
```

기존 App.js에 있었던 코드를 지워주고 다음과 같이 작성해줄게요.

```js
// src/App.js

import React, { Component } from 'react';
import { Route } from 'react-router-dom';
import Main from './Main';
import Sub from './Sub';

class App extends Component {
    render() {
        return (
            <div>
                <Route path="/" component={Main} />
                <Route path="/sub" component={Sub} />
            </div>
        );
    }
}

export default App;
```

여기서 /sub으로 들어가면 메인 컴포넌트 내용도 보이는 것을 확인할 수 있습니다.

이는 /sub경로가 /경로와 중복되기 때문입니다. 따라서 exact라는 props를 넣어줍니다.

```js
// src/App.js

import React, { Component } from 'react';
import { Route } from 'react-router-dom';
import Main from './Main';
import Sub from './Sub';

class App extends Component {
    render() {
        return (
            <div>
                <Route exact path="/" component={Main} />
                <Route path="/sub" component={Sub} />
            </div>
        );
    }
}

export default App;
```

이렇게 하면 경로가 완벽히 똑같을때만 컴포넌트를 보여주게 됩니다.

**이번에는 링크를 만들어서 클릭하면 다른 주소로 이동할 수 있게 해줄게요.**

이때는 Link컴포넌트를 사용합니다.

Link컴포넌트는 클릭시 다른 주소로 이동시키는 컴포넌트입니다.

리액트 라우터사용시 일반 a태그를 사용하시면 e.preventDefault\(\)를 해주어야하고 자바스크립트로 주소를 변환해주어야합니다.

그렇지않으면 새로고침을 하게되고 그러면 상태초기화, 컴포넌트 언마운트 후 리렌더링을 하게됩니다.

따라서 Link컴포넌트를 사용하며, 이는 HTML5 History API를 사용하기 때문에 브라우저의 주소만 바꿀뿐 페이지를 새로 불러오지는 않습니다.

```js
// src/App.js

import React, { Component } from 'react';
import { Route, Link } from 'react-router-dom';
import Main from './Main';
import Sub from './Sub';

class App extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li>
                        <Link to="/">
                            메인화면으로 가기
                        </Link>
                    </li>
                    <li>
                        <Link to="/sub">
                            서브화면으로 가기
                        </Link>
                    </li>
                </ul>
                <Route exact path="/" component={Main} />
                <Route path="/sub" component={Sub} />
            </div>
        );
    }
}

export default App;
```

**이번엔 파라미터, 그리고 쿼리를 받을 수 있도록 해볼게요.**

파라미터, 쿼리를 언제 사용해야하느냐에 대한 정해진 규칙은 없지만 통상적으로 파라미터는 특정 id나 이름을 가지고 조회를 할 때,

쿼리는 어떤 키워드를 검색하거나 요청에 필요한 옵션을 전달할 때 사용합니다.

```js
// src/User.js

import React, { Component } from 'react';

const userData = {
    user1: {
        name: 'user1',
        job: 'frontend developer',
    },
    user2: {
        name: 'user2',
        job: 'backend developer',
    },
    user3: {
        name: 'user3',
        job: 'PM',
    },
};

class User extends Component {
    render() {
        // 파라미터를 받아올때는 match props를 사용
        const { username } = this.props.match.params;
        const user = userData[username];
        if (!user) return <div>없는 사용자입니다.</div>;

        return (
            <div>
                <h2>
                    {user.name}의 직업은 {user.job}입니다.
                </h2>
            </div>
        );
    }
}

export default User;
```

```js
// src/App.js

import React, { Component } from 'react';
import { Route, Link } from 'react-router-dom';
import Main from './Main';
import Sub from './Sub';
import User from './User';

class App extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li>
                        <Link to="/">
                            메인화면으로 가기
                        </Link>
                    </li>
                    <li>
                        <Link to="/sub">
                            서브화면으로 가기
                        </Link>
                    </li>
                </ul>
                <Route exact path="/" component={Main} />
                <Route path="/sub" component={Sub} />
                <Route path="/users/:username" component={User} />
            </div>
        );
    }
}

export default App;
```

localhost:3000/user/username1

localhost:3000/user/username2

localhost:3000/user/username3

각각 접속을 해보세요. 잘 구현이 되어있나요?

쿼리도 해봅시다.

```js
// src/Sub.js

import React, { Component } from 'react';
import qs from 'qs';

class Sub extends Component {
    render() {
        const {information} = qs.parse(this.props.location.search.substr(1));
        const isInfoTrue = information === 'true';

        return (
            <div>
                <h2>서브화면입니다.</h2>
                {isInfoTrue && <div>more information...</div>}
            </div>
        );
    }
}

export default Sub;
```

아래처럼 할 수도 있습니다.

```js
// src/Sub.js

import React, { Component } from 'react';

class Sub extends Component {
    render() {
        // search => ?information=true
        const search = this.props.location.search;
        const params = new URLSearchParams(search);
        const info = params.get('information') === 'true' ? true : false;

        return (
            <div>
                <h2>서브화면입니다.</h2>
                {info && <div>more information...</div>}
            </div>
        );
    }
}

export default Sub;
```

이렇게 하고 localhost:3000/sub?information=true로 접속해볼까요? 그러면 세부적인 정보가 나오겠죠?

**이번엔 서브라우트입니다.**

App.js를 보면 라우트가 다음과 같이 선언이 되어 있습니다.

```js
<Route path="/" exact component={Main} />
<Route path="/sub" component={Sub} />
<Route path="/users/:username" component={User} />
```

'/', '/sub'처럼 일관된 주소를 만들어주고 싶은데 users만 파라미터가 들어가서 좀 애매하죠?

**이런 경우에는 서브라우트로 구현하면 깔끔합니다.**

```js
// src/Users.js

import React, { Component } from 'react';
import { Route, Link } from 'react-router-dom';
import User from './User';

class Users extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li>
                        <Link to="/users/user1">user1</Link>
                    </li>
                    <li>
                        <Link to="/users/user2">user2</Link>
                    </li>
                    <li>
                        <Link to="/users/user3">user3</Link>
                    </li>
                </ul>
                <Route render={() => <div>유저를 선택하세요...</div>} />
                <Route path="/users/:username" component={User} />
            </div>
        );
    }
}

export default Users;
```

단순하게 위와 같이 Users컴포넌트를 따로 만들고 그안에서 라우트를 또 선언해주는 방식으로 구현하면 됩니다.

당연히 App.js에서는 Users 컴포넌트를 라우트로 연결해주어야 겠죠?

```js
// src/App.js

import React, { Component } from 'react';
import { Route, Link } from 'react-router-dom';
import Main from './Main';
import Sub from './Sub';
import Users from './Users';

class App extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li>
                        <Link to="/">
                            메인화면으로 가기
                        </Link>
                    </li>
                    <li>
                        <Link to="/sub">
                            서브화면으로 가기
                        </Link>
                    </li>
                </ul>
                <Route exact path="/" component={Main} />
                <Route path="/sub" component={Sub} />
                <Route path="/users" component={Users} />
            </div>
        );
    }
}

export default App;
```

---

**잠깐!**

"라우터를 사용할 때 props는 어떻게 전달하나요?"

서브라우트에서 사용했던 것과 마찬가지로 `render` 를 사용하시면 됩니다.

```jsx
<Route path="/somewhere" component={SomeComponent} something={true} />
```

위와 같이하면 이런식으로는 props전달이 되질 않죠? 하지만 `render`는 jsx를 렌더해주므로 가능합니다.

```jsx
<Route path="/somewhere" render={(props) => <Somecomponent {...props} something={true} />}>
```

...props는 Route컴포넌트가 받은 props를 Somecomponent에도 동일하게 내려준다는 의미입니다.

깔끔하죠?

다음과 같은 방식은 옳지 않습니다.

```jsx
<Route path="/somewhere" component={() => <Somecomponent something={true}/>} />
```

물론 가능은 하지만 리액트 개발팀에서는 다음과 같이 말합니다.

"When you use the component props, the router uses React.createElement to create a new React element from the given component. That means if you provide an inline function to the component attribute, you would create a new component every render. This results in the existing component unmounting and the new component mounting instead of just updating the existing component"

요약하자면 component안에 함수를 넣어서 호출하면 매순간 React.createElement를 호출하기 때문에 \(jsx를 바벨이 트랜스파일링 만들어 내는 함수 기억나시죠?\) 계속해서 새로운 컴포넌트가 언마운트, 마운트를 하기 때문에 매우 비효율적이라는 것입니다. 따라서 위에 쓴 것처럼 render를 사용해서 props를 내려주도록 합시다!

---

**이번엔 존재하지 않는 주소로 이동했을때 404 페이지를 띄워줄게요.**

이때는 Switch컴포넌트를 이용합니다.

```js
// src/App.js

import React, { Component } from 'react';
import { Route, Link, Switch } from 'react-router-dom';
import Main from './Main';
import Sub from './Sub';
import Users from './Users';

class App extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li>
                        <Link to="/">
                            메인화면으로 가기
                        </Link>
                    </li>
                    <li>
                        <Link to="/sub">
                            서브화면으로 가기
                        </Link>
                    </li>
                </ul>
                <Switch>
                    <Route exact path="/" component={Main} />
                    <Route path="/sub" component={Sub} />
                    <Route path="/users" component={Users} />
                    <Route render={() => <div>404 NOT FOUND</div>} />
                </Switch>
            </div>
        );
    }
}

export default App;
```

**마지막으로 NavLink를 사용해보겠습니다.**

Link대신 NavLink를 사용하면 유저가 해당 Link를 클릭했을 때 스타일링을 추가해줄 수 있습니다.

```js
import React, { Component } from 'react';
import { Route, NavLink, Switch } from 'react-router-dom';
import Main from './Main';
import Sub from './Sub';
import Users from './Users';

class App extends Component {
    render() {
        return (
            <div>
                <ul>
                    <li>
                        <NavLink activeStyle={{ color: 'blue' }} to="/">
                            메인화면으로 가기
                        </NavLink>
                    </li>
                    <li>
                        <NavLink activeStyle={{ color: 'blue' }} to="/sub">
                            서브화면으로 가기
                        </NavLink>
                    </li>
                </ul>
                <Switch>
                    <Route exact path="/" component={Main} />
                    <Route path="/sub" component={Sub} />
                    <Route path="/users" component={Users} />
                    <Route render={() => <div>404 NOT FOUND</div>} />
                </Switch>
            </div>
        );
    }
}

export default App;
```

이외에도 history, redirect 등 기타 내용들이 있습니다.

history는 말그대로 웹 브라우저의 History객체를 조작할 수 있게 해줍니다.

예를들어 history.push 혹은 history.goBack등을 사용하면 사용자가 어떤 버튼을 클릭했을 때 특정 url 이동하기, 뒤로가기 등을 할 수 있게되죠.

Redirect와의 차이점은 리다이렉션은 중간에 거쳐간 url이므로 히스토리 객체에 방문 url흔적이 남지 않게됩니다.

\(component \(2\)를 라우터를 사용한 버전으로 바꿔보면서 사용해볼게요! \)

