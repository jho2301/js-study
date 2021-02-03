## 비동기 동시성 프로그래밍

1. 콜백함수
2. Promise (+ async/await)



### 콜백함수

```javascript
function add10(a, callback){
   setTimeout(() => callback(a + 10), 100); // 원하는시점(100ms이후)에 콜백함수 실행
}

// 콜백 지옥
add10(5, res => {
   add10(res, res => {
      add10(res, res => {
         console.log(res); //35
      });
   });
});
```

함수내에서 필요한 시점에서 콜백함수를 호출할 수 있기 때문에 비동기적으로 동시성 프로그래밍이 가능하다. 하지만 콜백함수를 합성할수록 구조적으로 복잡해지기 때문에 콜백지옥에 빠질수 있다.

> 동시성(합성함수)을 콜백함수 내에서 구현한다.
>
> 콜백함수 내에서 비동기적인 상황을 코드로만 다루어야한다.



### Promise

```javascript
function add20(a){
   return new Promise(resolve => setTimeout(() => resolve(a + 20), 100));
}

add20(5)
   .then(add20)
   .then(add20)
   .then(console.log); //65
```

promise를 사용하면 비동기적인 동시성 프로그래밍의 표현을 훨씬 간단하게 표현할 수 있다.

> 동시성(합성함수)을 프로미스의 리턴값에서 체이닝한다. 
>
> 비동기적인 상황을 프로미스에서의 상태값으로 리턴받아 활용한다.



### 중요 Point!

- 프로미스는 값과 상태를 나타내는 값이다.
- 프로미스의 반환값은 프로미스이다.
- 프로미스는 비동기 값을 일급으로 처리한다. 
- 프로미스는 상태(대기,성공,실패)를 값으로 사용할 수 있기 때문에 응용과 표현력이 좋다.



## 값으로서의 프로미스 활용

프로미스는 비동기적이고 상태값을 지녔기 때문에 일반적인 함수에서는 사용할 수 없다. 하지만 프로미스를 일반적인 값으로 활용할 수도 있다.



```javascript
const delay100 = a => new Promise(resolve =>
   setTimeout(() => resolve(a), 100));

const go1 = (a, f) => f(a);
const add5 = a => a + 5;

console.log(go1(10, add5)); // 15
console.log(go1(delay100(10), add5)); // [object Promise]5
```

delay100은 프로미스를 리턴한다. go라는 함수에 delay100을 인자로하여 프로미스값을 전달하면 동작하지 않는다.



```javascript
const delay100 = a => new Promise(resolve =>
   setTimeout(() => resolve(a), 100));

const go1 = (a, f) => a instanceof Promise ? a.then(f) : f(a);
const add5 = a => a + 5;

var r = go1(10, add5);
console.log(r); // 15

var r2 = go1(delay100(10), add5);
r2.then(console.log); // Promise{<pending>}
//-----------------------------------------------
const n1 = 10;
go1(go1(n1, add5), console.log); //15

const n2 = delay100(10);
go1(go1(n2, add5), console.log); //15
```

go 함수 내부에서 프로미스일 때 then을 이용하여 프로미스의 반환 값에 f함수를 처리하는식으로 구현하면  일반값으로 서 활용할 수 있다.



## 프로미스와 모나드

### 모나드

어떤 특정 상황에서 안전하게 함수를 합성하기 위한 기법이다.



### 합성함수에서의 모나드

```javascript
const g = a => a + 1;
const f = a => a * a;

// 일반적인 합성함수
console.log(f(g(1))); // 4
console.log(f(g()));  // NaN

// 모나드를 활용한 합성함수, []를 모나드로 활용
[1].map(g).map(f).forEach(r => console.log(r)); // 4
[].map(g).map(f).forEach(r => console.log(r)); // 동작안함
```

일반적인 합성함수에서는 값을 넣지 않았을 때도 함수가 실해되어 NaN같은 원하지 않는 값을 리턴한다.

이 때 [배열표현]을 모나드처럼 활용하여 합성함수를 구현하면 값을 넣지 않았을 때는 함수가 실행되지 않기 때문에 안전하게 구현할 수 있다.



### 프로미스에서의 모나드

```javascript
Promise.resolve(2).then(g).then(f).then(r => console.log(r)); // 9
Promise.resolve().then(g).then(f).then(r => console.log(r)); // NaN

new Promise(resolve =>
   setTimeout(() => resolve(2), 100)
   ).then(g).then(f).then(r => log(r)); // 9

/* 이부분이 모나드 라고 생각하자.
new Promise(resolve =>
   setTimeout(() => resolve(2), 100)
   )
*/
```

프로미스는 값이 있냐 없냐가 아닌 비동기적인 상황에서 안전하게 처리한다는 시점에서 모나드이다.

즉, 어느 시점 이후에 값을 알 수 있는 박스(모나드)라고 생각하여 비동기적으로 값을 받아와 안전하게 처리한다.

