결과를 만드는 함수인 reduce, take를 응용해보자.
# queryStr
```javascript
const queryStr = pipe(
  Object.entries,
  map(([key, value]) => `${key}=${value}`),
  reduce((a, b) => `${a} & ${b}`),
);

log(queryStr({limit: 10, offset: 10, type: 'notice'})); // limit=10 & offset=10 & type=notice
```
지금까지 만든 함수를 사용해 queryString을 출력하는 함수를 위와 같이 만들 수 있는데, 더 쪼개보자.
# join
```javascript
const join = curry((sep = '', iter) =>
  reduce((a, b) => `${a}${sep}${b}`, iter),
);

const queryStr = pipe(
  Object.entries,
  map(([key, value]) => `${key}=${value}`),
  join(' & '),
);
```
Array를 상속받지 않아도 사용할 수 있는 다형성이 높은 join 함수를 만들었다.
성능을 위해 지연된 함수를 사용할 수도 있다.
```javascript
L.entries = function* (obj) {
  for (const key in obj) {
    yield [key, obj[key]];
  }
};

const queryStr = pipe(
  L.entries,
  L.map(([key, value]) => `${key}=${value}`),
  join(' $ '),
);

log(queryStr({limit: 10, offset: 10, type: 'notice'})); // limit=10 & offset=10 & type=notice
```

# find
join과 같이 다형성이 높은 find 함수도 만들어보자.
```javascript
const users = [
  {age: 32},
  {age: 33},
  {age: 34},
  {age: 28},
  {age: 29},
  {age: 15},
  {age: 42},
];

const find = (f, iter) => go(iter, filter(f), take(1), ([v]) => v);

log(find((v) => v.age < 30, users)); // {age: 28}
```
이처럼 find 함수는 조건을 만족하는 첫 번째의 값을 가져온다.<br>
하지만 지연된 함수가 아니기 때문에 모든 iter 값을 filter 함수를 통해 순회하고, take 함수에 넘겨주는 부분이 아쉬운데,<br>
지연된 함수로 해결할 수 있다.
```javascript
const find = (f, iter) =>
  go(
    iter,
    L.filter(f),
    take(1),
    ([v]) => v,
  );
```
이제는 take에서 한 개의 값을 얻을 때까지만 filter가 실행된다.

# L.map, L.filter => map, filter
기존의 map과 filter 함수를 L.map과 L.filter로 구현해보자.
```javascript
const map = curry((f, iter) => go(iter, L.map(f), take(Infinity)));
const filter = curry((f, iter) => go(iter, L.filter(f), take(Infinity)));
```
take 함수를 통해 iter을 모두 순회시켜주면 가능하다.

이를 살짝 변형해보면
```javascript
const map = curry(pipe(L.map, take(Infinity)));
const filter = curry(pipe(L.filter, take(Infinity)));
```
go 함수를 pipe 함수로 변경해 가독성을 증가시킬 수 있다.
