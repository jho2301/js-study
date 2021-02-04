# Kleisli Composition 관점에서의 Promise
Promise는 Kleisli Composition을 지원하는 도구라고 볼 수 있다.
### Kleisli Composition?
함수 합성 방법으로 `오류가 있을 수 있는 상황에서의 함수 합성을 안전하게 할 수 있도록 하는 규칙`
- 받은 인자가 잘못된 값으로 오류가 발생할 때
- 정확한 인자라고 하더라도 의존하고 있는 외부의 상태에 의해서 오류가 발생할 때

> g(x)의 결과에 오류가 있을 때 f(g(x)) = g(x)로 만들어주는 것을 Kleisli Composition 관점이라고 한다.

```javascript
const users = [
  {id: 1, name: 'aa'},
  {id: 2, name: 'bb'},
  {id: 3, name: 'cc'},
];

const getUserById = (id) => find((user) => user.id === id, users);

const f = ({name}) => name;
const g = getUserById;
const fg = (id) => f(g(id));

log(fg(2));
users.pop();
users.pop();
log(fg(2)); // Uncaught TypeError: Cannot destructure property 'name' of 'undefined' as it is undefined.
```
이처럼 합성 함수에 잘못된 값이 인수로 전달되면 에러가 발생하지만 이를 방지하는 것이 Promise이다.

```javascript
const getUserById = (id) =>
  find((user) => user.id === id, users) || Promise.reject('없어요');

const f = ({name}) => name;
const g = getUserById;
const fg = (id) =>
  Promise.resolve(id)
    .then(g)
    .then(f)
    .catch((error) => error);

users.pop();
users.pop();
fg(2).then(log); //  없어요
```
Promise는 잘못된 값이 인수로 전달될 경우 위와 같이 뒤의 함수로 전달하는 것이 아닌 catch로 가서 에러를 전달한다.<br>
이러한 원리로 안전한 함수 합성이 가능하다.

# go, pipe, reduce에서의 비동기 제어
```javascript
go(
  1,
  (a) => a + 10,
  (a) => Promise.resolve(a + 100),
  (a) => a + 1000,
  (a) => a + 10000,
  log,
); // [object Promise]100010000
```
현재는 go 함수와 pipe 함수에서 비동기 제어를 할 수 없는데, go 함수와 pipe 함수의 제어권을 reduce가 가지고 있기 때문에<br>
reduce 함수에서 비동기 처리를 해주면 비동기 제어가 가능해질 것이다.

```javascript
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  let cur;
  while (!(cur = iter.next()).done) {
    const value = cur.value;
    acc =
      acc instanceof Promise ? acc.then((acc) => f(acc, value)) : f(acc, value);
  }
  return acc;
});
  
  go(
  1,
  (a) => a + 10,
  (a) => Promise.resolve(a + 100),
  (a) => a + 1000,
  (a) => a + 10000,
  log,
); // 11111
});
```
간단한 방법으로 해결할 수 있다.

하지만 위와 같은 경우에서는 Promise가 사용된 아랫부분이 모두 비동기로 처리되기 때문에 성능에 좋지 않다.
```javascript
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }

  return (function recur(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const value = cur.value;
      acc = f(acc, value);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  })(acc);
});
```
이제 비동기와 동기 처리를 구분할 수 있게 되었다.

하지만 아직도 첫 번째 인수로 비동기가 들어오게 되면 처리하지 못하는 문제가 있는데, 이는 go1 함수로 처리할 수 있다.
```javascript
const go1 = (a, f) => (a instanceof Promise ? a.then(f) : f(a));

const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }

  return go1(acc, function recur(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const value = cur.value;
      acc = f(acc, value);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});

go(
  Promise.resolve(1),
  (a) => a + 10,
  (a) => Promise.resolve(a + 100),
  (a) => a + 1000,
  (a) => a + 10000,
  log,
); // 11111
```
