이제 axios라는 라이브러리를 사용해서 api콜을 해볼 예정입니다.

하지만 axios는 promise기반으로 만들어져있기 때문에 promise에 대한 이해가 필수입니다.

따라서 axios를 배우기전에 promsie를 배우고 넘어갈게요.

### **Promise**

promise는 비동기작업을 동기적으로 실행하고 싶을 때 사용합니다.

**비동기작업을 동기적으로 실행하고 싶다구요? 그게 무슨말인가요?**

```js
console.log('1');

setTimeout(()=>{
    console.log('2');
}, 1000)

console.log('3')
```

출력 값의 순서가 어떻게 될까요?

답은 1 3 2 입니다.

이는 setTimeout이 비동기적으로 작동했기 때문인데요,

이를 동기적으로 바꾸어 1 2 3이 나오도록 바꾸어 볼까요?

```js
console.log('1');

setTimeout(()=>{
    console.log('2');
    console.log('3')
}, 1000)
```

이렇게 바꿀수는 있겠지만 setTimeout이라는 비동기작업에다가 console.log\(3\)을 집어넣은 것 밖에 안되기때문에 실제상황에서의 해결책과는 거리가 좀 멀어보입니다.

우리가 원하는 것은 setTimeout\(\(\) =&gt; {console.log\(2\)}, 1000\)는 그대로 놔둔 상태로 1이 출력되고 1초가 있다가 2가 출력되고, 이어서 바로 3이 출력되도록 하는 것입니다.

이렇게 바꿀 수는 있습니다.

```js
function test(flag) {
    if(!flag) {
        console.log(1)
        setTimeout(function() {
           console.log(2)
           test(true);
        }, 1000);
        return;
    } else console.log(3);
}

test();
```

하지만.. 너무 어렵죠? 프로미스를 배우면 보다 쉽게 구현할 수 있습니다.

아래와 같이 프로미스를 사용해볼게요.

```js
function test() {
  return new Promise((resolve, reject) => {
       setTimeout(() => {
            console.log(2);
            resolve();
        }, 1000)
    })
}

console.log(1);
test().then(() => {
    console.log(3);
})
```

위와 같이 우리가 하고싶은 비동기작업을 new를 사용해서 새로운 프로미스 객체를 만들고, 그안에 넣어주면 됩니다.

new Promise안에는 콜백이 들어가는데 그 함수는 resolve, reject를 인자로 받습니다.

우리가 하고싶은 비동기작업을 한 이후에, 완료가되면 resolve안에 값을 담아서 호출해주면됩니다.

만약 제대로 해당 비동기작업이 완료되지않고 중간에 오류가나면 reject\(e\)처럼 reject안에 에러를 넣어서 호출합니다.

그러면 실행할 때는 resolve시에는 .then으로, reject시에는 .catch로 받을 수가 있어요.  
  
많은 블로그에서 콜백헬의 대안으로 프로미스를 사용한다고 설명하는데요, 그 의미가 무엇인지 아래에서 다시 살펴볼게요.

다음과 같은 함수가 있다고 해볼게요.

```js
function increase(number, callback) {
  setTimeout(() => {
      const result = number + 10;
      callback(result);
  }, 1000)
}
```

위 함수는 첫번째 인자로 숫자를 받고, 1초뒤 그 숫자에 10을 더해서 두번째 인자인 콜백에 넣어 호출하는 함수입니다.

따라서 사용할때는 다음과 같이 사용하면 되겠죠?

```js
increase(0, result => {
    console.log(result);
});
```

만약 10을 찍고 20을 찍고 싶다면?

```js
increase(0, result => {
    console.log(result);
    increase(result, result => {
        console.log(result);
    })
});
```

10, 20을 찍고 30을 찍고 싶다면..?

```js
increase(0, result => {
    console.log(result);
    increase(result, result => {
        console.log(result);
        increase(result, result => {
            console.log(result);
        })
    })
});
```

어떤 느낌이 드시나요? 코드가 정말 지저분하죠?

이런걸 바로 콜백헬이라고 합니다.

비동기작업을 동기적으로는 하고싶은데 이렇게 하자니 너무 지저분하고, 읽기도 어려웠을 것입니다.

이런 콜백헬을 개선하기 위해 promise가 등장했는데요, 위 increase함수를 promise로 바꿔볼까요?

```js
function increase(number) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const result = number + 10;
      resolve(result);
    }, 1000)
  });
}
```

마찬가지로 하고싶은 비동기작업을 새로 생성한 프로미스 객체에 넣고, 성공시에 resolve를 호출해 주었습니다.  
프로미스를 반환하는 함수를 호출할때는 resolve시에 .then으로 값을 받을 수 있다고 했죠?

따라서 콜백헬을 다음과 같이 개선할 수 있습니다.

```js
increase(0)
  .then(number => {
      console.log(number);
      return increase(number);
    })
  .then(number => {
      console.log(number);
      return increase(number);
    })
  .then(number => {
      console.log(number);
      return increase(number);
    })
  .then(number => { 
      console.log(number);
      return increase(number);
    })
  .then(number => { 
      console.log(number);
      return increase(number);
    })
```

.then으로 구획이 나누어져 있기 때문에 콜백헬에 비해 훨씬 가독성이 좋습니다.

혹시 프로미스가 처음이시라면 위로 올라가서 다시 한 번 코드를 살펴보세요!

### Async-await

콜백헬을 해결하기위한 방법으로 promise로 사용해보았는데요, 

async-await를 이용하면 보다 가독성이 높은 코드를 작성하실 수 있습니다.

async-await는 promise와 별개의 것이 아니라 promise기반 위에서 작동하는 것입니다.

async-await는 다음과 같은 형식으로 이루어집니다.

```js
async function asyncFunc(){
    await 첫번째 promise
    await 두번째 promise
    await 세번째 promise
    await 네번째 promise
    ...
}
```

await 키워드를 사용하기 위해서는 async 키워드가 function 앞에 붙어야합니다.

또한 await뒤에는 항상 promise 인스턴스가 와야해요.

그렇게하면 해당 promise가 resolve값을 반환하기 전까지는 await가 그다음 진행을 막아줍니다.

다시 한번 볼까요?

```js
// 비동기작업

setTimeout(() => {
    console.log(1)
}, 1000)
console.log(2)

// 결과값: 2 1
```

```js
// promise를 이용해서 비동기를 동기적으로 바꾸면?

function promiseFunc(){
    return new Promise((resolve) => {
        setTimeout(() => {
            console.log(1);
            resolve();
        }, 1000)
    })
}

promiseFunc().then(() => {
    console.log(2);
})

// 결과값: 1 2
```

```js
// async-await를 이용하면?

async function asyncFunc(){
    await promiseFunc();
    console.log(2);
}

// 결과값: 1 2
```

프로미스에서 .then과 .catch를 사용했다면,

async-await를 쓸때는 try~catch로 감싸면 try에서 비동기작업을 하다 에러가 날경우 catch로 잡아줄 수가 있습니다.

```js
async function asyncFunction(){
    try {
        // 네트워크 작업, 소켓요청 등 여러가지 비동기작업을 하다가 에러가 날경우 catch의 e로 에러인자가 넘어감
    } catch(e) {
        console.error(e)
    }
}
```

promise를 작성했으면 그걸 async-await로 바꾸는 연습을 해보시면 도움이 될 것입니다!





