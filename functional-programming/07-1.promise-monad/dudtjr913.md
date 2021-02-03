# Promise
`Promise`는 비동기 작업을 다룰 때 사용하는 객체이다.
```javascript
const add10 = (v, callback) => {
  setTimeout(() => callback(v + 10), 1000);
};

add10(5, (res) => {
  log(res);
}); // 1초 뒤 15

const add20 = (v) => {
  return new Promise((resolve) => {
    setTimeout(() => resolve(v + 20), 1000);
  });
};

add20(5).then(log); // 1초 뒤 25
```
기존의 callback 함수로 비동기를 처리할 때와 다르게 가독성도 매우 좋다는 것을 확인할 수 있다.
```javascript
add10(5, (res) => {
  add10(res, (res) => {
    add10(res, (res) => {
      log(res);
    });
  });
}); // 3초 뒤 35

add20(5).then(add20).then(log); // 2초 뒤 45
```
콜백 지옥에서 탈출할 수 있다.

Promise에서 가장 중요한 것은 가독성이 아닌 값을 가지는 일급 객체라는 것이다. <br>
Promise는 다음과 같은 상태를 가진다.
- 대기
- 이행
- 거부
즉, Promise는 값으로 활용할 수 있다.

값으로 활용되는 예
```javascript
const delay100 = (a) =>
  new Promise((resolve) => setTimeout(() => resolve(a), 100));

const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));
const add5 = (a) => a + 5;

log(go1(10, add5)); // 15
go1(delay100(10), add5).then(log); // 0.1초 후 15
```

# 모나드와 Promise
`모나드`란 어떠한 값이 들어올지 모르는 상황에서 안전하게 함수를 합성하는 아이디어이다.
```javascript
const add1 = (a) => a + 1;
const square = (a) => a * a;

log(square(add1(1))); // 4 
log(square(add1())); // NaN  
```
값이 할당되지 않았음에도 불구하고 값이 출력되는 위의 함수는 안전하게 합성된 함수가 아니다.
```javascript
[1]
  .map(add1)
  .map(square)
  .forEach((v) => log(v)); // 4

[]
  .map(add1)
  .map(square)
  .forEach((v) => log(v)); // x
```
위처럼 값이 들어있지 않으면 결과를 출력하지 않는 것이 안전하게 합성된 함수이고, 이것이 모나드이다.

그렇다면 Promise는 어떤 성질을 가지고 있을까?
```javascript
Promise.resolve(1)
  .then(add1)
  .then(square)
  .then((v) => log(v)); // 4

Promise.resolve()
  .then(add1)
  .then(square)
  .then((v) => log(v)); // NaN
```
이것을 보면 안전한 합성이 아니라고 생각이 든다.<br>
하지만 Promise는 안전한 합성에 대한 관점이 resolve에 어떠한 값이 들어오는 것에 초점을 맞추지 않고,<br>
비동기적인 상황을 안전하게 합성하기 위한 것에 초점을 맞췄다.

즉, 대기 상태에서의 합성을 안전하게 하려는 것이다.
```javascript
new Promise((resolve) => setTimeout(() => resolve(1), 1000))
  .then(add1)
  .then(square)
  .then((v) => log(v)); // 1초 후 4
```
이처럼 1초 뒤에 값이 들어오는 상황에서 안전하게 함수를 합성하기 위한 것이 Promise이다.<br>
그래서 Promise도 모나드라고 볼 수 있다.
