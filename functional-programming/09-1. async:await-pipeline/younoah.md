## async, awit



async, await은 프로미스와 호흡하기 때문에 프로미스를 잘 알고 있어야한다.

```javascript
function delay() {
    return new Promise(resolve => setTimeOut(() => resolve(a), 500));
}

async function f1() {
    const a = await delay(10);
    log(a);
}

async function f2() {
    const a = await delay(10);
    log(a);
}

f1();
// 10

f2();
// Promis {<pending>}

```



async 함수내에서는 프로미스를 동기적으로 표현이 가능하지만 함수를 리턴받을 때는 프로미스이기 때문에 주의해서 사용해야한다.

```javascript
function delay() {
    return new Promise(resolve => setTimeOut(() => resolve(a), 500));
}

async function f1() {
    const a = await delay(10);
    const b = await delay(15);
    return a + b;
}

console.log(f1()); // Promis {<pending>}
f1().then(log) // 15

(async ()=> {
    console.log(await f1());
}) ();
// 15
```



## 왜 FxJS의 map함수가 필요한지?



```javascript
async function f() {
    const list = [1, 2, 3, 4];
    const res = awaut list.map(async a => await delay(a * a))
    // [promise, promise, promise, promise]
}
```

map메서드는 프로미스를 제어하고 있지 않기 때문에 async/await를 사용해도 map메서드 안에서 비동기 상황을 제어할 수 없다.



```javascript
async function f() {
    const list = [1, 2, 3, 4];
    const res = await map(a => delay(a * a), list)
    // promise {<pending>}
}
```

FxJS의 map함수는 즉시 프로미스 값을 리턴하기 때문에 깔끔하게 async/await를 사용할 수 있다.



## async, await이 있는데 왜 파이프라인이 필요한지?



**파이프라인/체이닝 과 async/await 는 서로 해결하고자 하는 문제가 다르다.**

async/await는 Promise, then, then, then,... 이란 표현식으로 작성된 로직이 복잡하다보니 문장으로 다루기 위한 목적이 있다.

파이프라인/체이닝은 비동기 프로그래밍이 아닌 안전하게 함수를 합성 하는것이 목적이다.



비슷한 느껴진 이유는 파이프라인으로 코딩했을 때 동기/비동기 코드를 하는 모습같아 보여서 그런것뿐이다. 하지만 파이프라인은 안전하게 비동기 상황을 합성된 함수 끝까지 잘 이어나가기 위함인것이다.



```javascript
function f() {
    return go(list,
    	L.map(a => delay(a * a)),
    	L.filter(a => delay(a & 2)),
        L.map(a => delay(a + 1)),
        take(3),
        reduce((a, b) => delay(a + b))
    );
}
go(f([1, 2, 3, 4, 5]), log);
// 38
```

비동기적인 상황을 동기적으로 사용된다. 하지만 async/await을 해결하기 위함이 아닌 함수들의 합성을 비동기적인 상황이여도 안전하게 하기 위함이다.

파이프라인은 단순히 복잡한 코들르 간단하게 하기 위함이다 그사이에 안전하게 처리하기 위해 비동기/동기 처리기능이 있을뿐



```javascript
async function f() {
    let tmp = [];
    for (const a of list) {
        const b = await delay(a * a);
        if (await delay(b % 2)) {
            const c = await delay(b + 1);
            tmp.push(c);
            if (tmp.length == 2) break;
        }
    }
    let res = tmp[0], i = 0;
    while (++i < tmp.length) {
        res = await delay(res + tmp[i]);
    }
}
go(f([1, 2, 3, 4, 5]), log);
// 38
```

위 코드를 FxJS 없이 사용할 때 로직을 구현하면 위와 같다. 위로직에서 프로미스값을 동기적으로 처리하기 위해서 await를 사용한 것 뿐이다.