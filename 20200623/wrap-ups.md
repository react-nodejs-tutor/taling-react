지금까지 배운 내용들이 react문법의 상당부분을 차지합니다.

리액트 문법 자체는 쉽지만 문제는 "어떻게 활용하는가" 입니다.

지금까지 배운 내용들로 간단한 코드를 작성해 보겠습니다.

### input 관리

```js
import React, { Component } from 'react';

class App extends Component {
  state = {
    input: ''
  };
  handleChange = e => {
    this.setState({
      input: e.target.value
    });
  };
  render() {
    return (
      <div>
        <input value={this.state.input} onChange={this.handleChange} />
        <p>현재 input 값: {this.state.input}</p>
      </div>
    );
  }
}

export default App;
```

handleChange함수를 주목해 주세요.

```js
handleChange = e => {
    this.setState({
        input: e.target.value
    });
};
```

handleChange함수는 e값을 파라미터로 받아왔는데요,

e.target을 콘솔로 찍어보면 onChange이벤트가 발생한 input 엘리먼트를 받아올 수 있습니다.

따라서 e.target의 value값을 우리가 관리해주고 있는 input값에 넣어줌으로써 새롭게 상태관리를 해줄 수 있습니다.

#### 여러개의 input관리

```js
import React, { Component } from 'react';

class App extends Component {
  state = {
    username: '',
    password: ''
  };
  handleChange = e => {
    const { value, name } = e.target;
    this.setState({
      [name]: value
    });
  };
  render() {
    // username과 password를 비구조화 할당해서 꺼내옴
    const { username, password } = this.state;
    return (
      <div>
        <input
          name="username"
          value={username}
          onChange={this.handleChange}
        />
        <input
          name="password"
          value={password}
          onChange={this.handleChange}
        />
        <p>
          {username} 의 패스워드는 {password}입니다.
        </p>
      </div>
    );
  }
}

export default App;
```

인풋이 여러개일땐 어떻게 해줘야 할까요?

인풋이 여러개면 handleChange함수 입장에서 어떤 인풋에 변화가 일어난 것인지 모르기 때문에 각 input에 name값을 집어넣어 줘야 합니다.

그렇게 하면 handleChange를 다음과 같이 간단하게 작성할 수 있습니다.

```js
handleChange = e => {
    const { value, name } = e.target;
    this.setState({
        [name]: value
    });
};
```

위 코드를 보면

```js
this.setState({
    [name]: value
});
```

이렇게 name값을 대괄호로 감싸주었는데요, 이는 es6의 computed property names라는 문법입니다.

이렇게 name변수 안에 들어있는 값을 유동적으로 받아오기 위해서는 반드시 대괄호로 감싸주어야 한답니다.

대괄호로 감싸지 않고 다음과 같이 작성하면,

```js
this.setState({
    name: value
})
```

말그대로 name을 key로 가지는 객체를 의미하게 됩니다.

#### 배열에값을 추가하고 보여주기

```js
import React, { Component } from 'react';

class App extends Component {
  id = 1;
  state = {
    username: '',
    password: '',
    list: []
  };
  handleChange = e => {
    const { value, name } = e.target;
    this.setState({
      [name]: value
    });
  };
  handleInsert = () => {
    const { username, password, list } = this.state;
    this.setState({
      username: '',
      password: '',
      list: list.concat({ id: this.id, username, password })
    });
    this.id += 1; // 새 데이터 만들때마다 +1
  };
  render() {
    const { username, password, list } = this.state;
    return (
      <div>
        <input
          name="username"
          value={username}
          onChange={this.handleChange}
        />
        <input
          name="password"
          value={password}
          onChange={this.handleChange}
        />
        <button onClick={this.handleInsert}>추가하기</button>
        <p>
          {username} 의 패스워드는 {password}입니다.
        </p>
        <ul>
          {list.map((item) => (
            <li key={item.id}>
              {item.username}({item.password})
            </li>
          ))}
        </ul>
      </div>
    );
  }
}

export default App;
```

이번에는 button을 만들고 추가를 누르면 빈 배열에 값이 추가되고, 추가된 값들을 보여주는 코드를 작성해봤습니다.

배열에 있는 값을 보여줄때는 자바스크립트 내장함수인 map함수를 사용한다고 말씀드렸죠?

여기서 중요한 점은 map을 통해 배열안에 있는 값들을 보여줄때는 key값을 반드시 달아줘야 한다고 했습니다.

단순히 배열 요소값의 index를 key값을 사용하는 것은 warning메세지를 피하기위한 임시방편에 불과하기 때문에,

우리는 id를 만들어서 고유한 값을 달아주도록 처리했습니다.

단순히 버튼을 만들기만 하고 onClick을 연결하면 유저가 '엔터를 눌렀을때'라는 이벤트\(onKeyPress등으로\)를 따로 만들어줘야 해서 불편합니다.

따라서 이번에는 연관된 input과 button을 form으로 묶어주고 form에 onSubmit이라는 이벤트를 달아봅시다.

```js
<form onSubmit={this.handleInsert}>
    <input
        name="username"
        value={username}
        onChange={this.handleChange}
    />
    <input
        name="password"
        value={password}
        onChange={this.handleChange}
    />
    <button type="submit">추가</button>
</form>
```

```js
handleInsert = e => {
    e.preventDefault();
    const { username, password, list } = this.state;
    this.setState({
        username: '',
        password: '',
        list: list.concat({ id: this.id, username, password })
    });
    this.id += 1;
};
```

form에는 원래 내용을 담아서 서버로 POST http request를 \(주로\) 날리기 때문에, 페이지가 새로고침 됩니다. 따라서 form을 사용한다면 handleInsert에서는 e.preventDefault를 해줘야합니다. form안의 onSubmit이벤트가 기본적으로 가지고있는 새로고침 현상을 막아줌으로써 state가 초기화되는 것을 막아줍니다.

이제 폼을 작성하고 엔터를 눌러보세요. 개발자도구를 열어보면 배열에 성공적으로 값이 담긴것을 확인하실 수 있을거에요. 다만 폼을 작성하고 password input에 포커스가 남아있습니다. username input으로 포커스를 옮겨주려면 reference라는 것을 사용해야 합니다. ref를 사용하는 방식은 크게 두가지가 있는데요, 우선 첫번째 방식은 다음과 같습니다.

```js
import React, { Component } from 'react';

class App extends Component {
  state = {
    username: '',
    password: '',
    list: []
  };
  id = 1;
  usernameInputRef = null;

  ...

  handleInsert = e => {
    e.preventDefault();
    const { username, password, list } = this.state;
    this.setState({
      username: '',
      password: '',
      list: list.concat({ id: this.id, username, password })
    });
    this.id += 1;

    this.usernameInputRef.focus();
  };

  render() {
    const { username, password, list } = this.state;
    return (
      <div>
        <form onSubmit={this.handleInsert}>
          <input
            name="username"
            value={username}
            onChange={this.handleChange}
            ref={ref => {
              this.usernameInputRef = ref;
            }}

        ...
      </div>
    );
  }
}

export default App;
```

리액트는 버추얼돔을 사용하고 state를 통해 변화값을 관리하기 때문에, 사실 ref를 사용해서 직접 dom에 접근하는 것을 권장하지는 않습니다.다만 input에 focus를 주거나, 스크롤 위치를 가져오는 것등과 같은 특수한 케이스에는 ref를 사용해서 dom을 조작하면 됩니다.

위 코드처럼 컴포넌트가 렌더된 이후에 username input을 usernameInputRef에 넣어주고, handleInsert를 할 때 해당 dom element에 focus를 주는 방식으로 코드를 작성하면 됩니다.

두번째는 리액트에서 제공하는 createRef를 사용하는 방식입니다.주목할 점은 단순히 focus가 아닌 current.focus를 사용해야 한다는 것입니다.

```js
usernameInputRef = React.createRef();
```

```js
<input
    name="username"
    placeholder="유저명"
    value={username}
    onChange={this.handleChange}
    ref={this.usernameInputRef}
/>
```

```js
handleInsert = e => {
    e.preventDefault();
    const { username, password, list } = this.state;
    this.setState({
        username: '',
        password: '',
        list: list.concat({ id: this.id, username, password})
    });
    this.id += 1;``

    this.usernameInputRef.current.focus();
};
```

// 여기까지 23일 수업내용

