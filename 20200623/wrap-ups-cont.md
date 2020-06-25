// 여기서부터 24일 수업내용

이어서 삭제작업을 진행해볼까요? 배열에 insert를 할 때는 배열내장함수에서 push가 아닌 concat을 사용했는데, 이는 불변성을 지키기 위함이라고 말씀드렸습니다. 삭제 작업도 마찬가지입니다. 이 때는 filter를 사용하면 편합니다. handleRemove함수를 한번 작성해볼까요?

```js
handleRemove = id => {
    const {list} = this.state;
    this.setState({
        list: list.filter(item => item.id !== id)
    })
};
```

filter도 map함수와 마찬가지로 콜백함수를 인자로 받는데요, 모든 원소를 한번씩 돌리면서 콜백함수의 몸체부분에서 true를 반환하는 원소들만 걸러줍니다. handleRemove의 경우에는 인자로 받은 id를 가지고있는 원소를 제외한 모든 원소들이 걸러지겠지요?

이제 handleRemove 이벤트핸들러를 버튼에 연결해보겠습니다.

```js
<button onClick={() => this.handleRemove(item.id)}>
    삭제하기
</button>
```

여기를 보면 this.handleRemove\(item.id\)가 아닌 \(\) =&gt; this.handleRemove\(item.id\)로 표현했습니다. jsx에서는 이벤트핸들러를 등록할때 함수이름만 넣고 호출하지 않는다고 했죠? 카운터 예제에서 onClick={this.handleIncrease} 이렇게 했지 onClick={this.handleIncrease\(\)}이렇게 하지 않았습니다.

그 이유는 무엇이었나요? this.handleIncrease\(\)이렇게 호출해버리면 컴포넌트가 렌더됨과 동시에 해당 이벤트핸들러가 호출되고 바로 state로 관리하는 number값이 0에서 1로 증가하기 때문에 즉시 해당 컴포넌트가 리렌더되고, 계속해서 그 작업이 반복되기 때문에 무한루프에 빠진다고 했습니다.

handleRemove의 경우도 마찬가지 입니다. this.handleRemove에 item.id라는 인자를 넣긴 넣어서 전달해주어야 하는데, this.handleRemove\(item.id\)이렇게 해버리면 즉시 호출되어 버리겠죠? 따라서 this.handleRemove\(item.id\)를 한번 더 감싸주는 것입니다.

\(\) =&gt; this.handleRemove\(item.id\) 이렇게요. 이렇게하면 버튼이 클릭되었을때 \(\(\) =&gt; this.handleRemove\(item.id\)\)\(\)가 되면서 this.handleRemove함수에 id가 잘 담겨서 호출됩니다.

