# range, take, map, filter, reduce 중첩 사용
이번에는 지금까지 배운 함수들을 중첩해서 사용하면서 기존 함수와 지연된 함수의 동작 방식이 어떤 순서로 이루어지는지 살펴보자.
```javascript
// 변경 전
const map = curry((f, iter) => {
  const res = [];
  
  for (const value of iter) {
    res.push(f(value));
  }

  return res;
});

L.map = curry(function* (f, iter) {
  for (const value of iter) {
    yield f(value);
  }
});


// 변경 후
const map = curry((f, iter) => {
  const res = [];
  iter = iter[Symbol.iterator]();
  let cur;

  while (!(cur = iter.next()).done) {
    const value = cur.value;
    res.push(f(value));
  }

  return res;
});

L.map = curry(function* (f, iter) {
  iter = iter[Symbol.iterator]();
  let cur;
  while (!(cur = iter.next()).done) {
    const value = cur.value;
    yield f(value);
  }
});

// 나머지 filter, reduce도 동일하게 변경
```
동작 순서를 확인하기 전에 for of문이 동작하는 방식을 명령형으로 표현해보았다.
```javascript
// 일반 함수
go(
  range(10),
  map((n) => n + 10),
  filter((n) => n % 2),
  take(2),
  log,
); // [11, 13]
```
range, map, filter, take 함수에 Breakpoints를 찍어 확인해보면 range => map => filter => take의 순서대로<br>
위에서 아래로 실행되는 것을 확인할 수 있다.
```javascript
// 지연된 함수
go(
  L.range(10),
  L.map((n) => n + 10),
  L.filter((n) => n % 2),
  take(2),
  log,
); // [11, 13]
```
지연된 함수에서도 Breakpoints를 찍어 확인해보면 동작 방식이 일반 함수와 다른 것을 확인할 수 있다.<br>
우선, 실행 순서가 예상과 다르게 take => L.filter => L.map => L.range의 순서로 실행되는 것을 확인할 수 있는데,<br>
지연 함수에서는 next() 메소드가 사용되기 전까지 실행되지 않기 때문에 이런 결과가 나타난다.

take(2)를 통해 for of문을 돌면서 next() 메소드가 실행될 것이고 take 함수가 가지고 있는 iter는 L.filter이기 때문에<br>
take에서 next()를 실행하면 L.filter로 가고, L.filter에서 next()를 실행하면 L.map으로 가고, L.map에서 next()를 실행하면<br>
L.range로 가서 yield값을 받아온 후에 다시 L.map => L.filter 순서대로 가는 것이다.

즉, 순서는 take => L.filter => L.map => L.range => yield값을 받아서 => L.map => L.filter => take로 이루어지며,<br>
필요한 값이 있을 때만 평가해 사용하는 것을 볼 수 있다.

이는 효율성에서도 많은 차이가 나는데, test 함수를 통해 알아보자.
```javascript
const test = (name, count, f) => {
  console.time(name);
  while (count--) f();
  console.timeEnd(name);
};

test('function', 10, () =>
  go(
    range(1000000),
    map((n) => n + 10),
    filter((n) => n % 2),
    take(2),
  ),
); // function: 730.76611328125 ms

test('lazy-function', 10, () =>
  go(
    L.range(1000000),
    L.map((n) => n + 10),
    L.filter((n) => n % 2),
    take(2),
  ),
); // lazy-function: 0.38525390625 ms
```
시간 차이를 통해 지연된 함수의 효율성을 확인할 수 있다.


