go, pipe, curry 함수를 만들면서 코드를 값으로 다루어 표현력을 높이는 방법을 알아보자.
# go
```javascript
go(
  0,
  (a) => a + 1,
  (a) => a + 10,
  (a) => a + 100,
  log,
); // 111
```
우선 go는 위와 같은 인수를 할당해 호출할 때, 0 => 1 => 11 => 111 => console.log로 연쇄적으로 출력되는 함수이다.<br>
이를 구현해보자.
```javascript
const go = (param, ...fs) => reduce((acc, f) => f(acc), param, fs);
```
다소 어려울 수 있는데 go 함수는 첫 번째 인자로 뒤에 호출될 함수들의 인자를 받고, 나머지 인자들은 함수를 받아<br>
reduce를 통해 값을 누적하는 함수이다. 쉽게 설명하면
```javascript
const add1 = (a) => a + 1;
const add10 = (a) => a + 10;
go(
  0,
  add1,
  add10,
);
```
이 코드가 있을 때 add1(0)으로 실행한 뒤 나온 결과 값인 1을 다시 add10(1)로 계속 누적한다고 보면 된다.

# pipe
go 함수는 인자들을 통해 `즉시 값을 평가`하는 데 사용되었다면, pipe 함수는 함수들이 나열되어 있는 `합성된 함수`를 만드는 함수이다.
```javascript
pipe(
  (a) => a + 1,
  (a) => a + 10,
  (a) => a + 100,
  log,
)(0); // 111
```
이렇게 함수를 만들어서 (0)과 같이 한 번 더 실행을 해주어야 값이 평가가 되는 함수이다.
구현해보자.
```javascript
const pipe = (...fs) => (param) => go(param, ...fs);
```
pipe 함수는 합성된 함수를 만들어야 하기 때문에 클로저를 사용해 함수들을 모아두고, param을 받으면 go를 통해 실행된다.

살짝 아쉬운 점이 있는데,
```javascript
pipe(
  (a, b) => a + b,
  (a) => a + 10,
  (a) => a + 100,
  log,
)(0, 1); // NaN
```
이렇게 인자가 두 개 이상인 경우에는 사용할 수 없다는 것이다. 이를 보완해보자.
```javascript
const pipe = (f, ...fs) => (...param) => go(f(...param), ...fs);

pipe(
  (a, b) => a + b,
  (a) => a + 10,
  (a) => a + 100,
  log,
)(0, 1); // 111
```
이렇게 인자를 받을 때 첫 번째 인자를 값으로 평가해서 넘겨주면 된다.

## go를 사용해 가독성 챙기기 💯
```javascript
const addPrice = (total_price, price) => total_price + price;

// go 사용하기 전
log(
  reduce(
    addPrice,
    0,
    map(
      (product) => product.price,
      filter((product) => product.price < 5000, products),
    ),
  ),
); // 9000

// go 사용 후
go(
  products,
  (products) => filter((product) => product.price < 5000, products),
  (products) => map((product) => product.price, products),
  (price) => reduce(addPrice, price),
  log,
); // 9000
```
기존에 아래에서 위로 읽는 것과 비교했을 때 가독성이 매우 좋아진 것을 볼 수 있다.<br>
이 가독성을 더 좋게 만들기 위해 curry 함수를 만들어보자.

# curry
curry 함수는 함수를 받아서 원하는 시점에 값을 평가시킬 수 있는 함수이다.
```javascript
const curry = (f) => (a, ...args) =>
  args.length ? f(a, ...args) : (...rest) => f(a, ...rest);
```
코드가 난해할 수 있는데 천천히 살펴보자.

curry 함수는 인수로 받은 함수를 보관한다. 그러고 그 함수 안에서 새로운 함수를 return하는데, 이때 받은 인수가 2개 이상이면<br>
받은 인수를 통해 보관하던 함수를 곧바로 실행한다. 하지만 인수가 1개일 땐 다시 새로운 함수를 return해 다음 인수가 들어오게 되면<br>
값이 출력된다.
결국, 다음과 같이 동작한다

```javascript
log(curry((a, b) => a + b)(1, 2)); // 3
log(curry((a, b) => a + b)(1)(2)); // 3
```
이를 이용해 기존에 만들었던 map, filter, reduce 함수에 적용할 수 있다.
```javascript
const map = curry((f, iter) => {
  const res = [];
  for (const value of iter) {
    res.push(f(value));
  }

  return res;
});
// 나머지 생략
```
이렇게 하면 이제 map 함수는 인자로 가지고 있는 iter을 나중에 받을 수 있게 되었다.
```javascript
log(map((v) => v + 1)([1, 2, 3])); // [2, 3, 4] 이렇게 사용이 가능해졌다.
```

# go + curry
지금까지 한 코드를 curry와 조합해보자.
```javascript
go(
  products,
  filter((product) => product.price < 5000),
  map((product) => product.price),
  reduce(addPrice),
  log,
); // 9000
```
가독성이 더 좋아졌다. 

기존의 `(products) => filter((product) => product.price < 5000, products)` 코드에서 products가 사라진 이유는<br>
curry 함수를 사용해 `filter((product) => product.price < 5000)`까지만 했을 때, 이 자체가 값이 아닌 함수가 되기 때문이다.<br>
그리고 go 함수에서 내부적으로 `filter((product) => product.price < 5000)(products)`를 해준다.

# go + pipe + curry
마지막으로 지금까지 배운 모든 것을 조합해 중복을 제거하고, 재사용성을 높여보자.<br>
바로 위에서 했던 코드에서 변형을 해보면,
```javascript
const total_price = pipe(
  map((product) => product.price),
  reduce(addPrice),
);

go(
  products,
  filter((product) => product.price < 5000),
  total_price,
  log,
); // 9000
```
pipe를 사용해 합성 함수를 만들어 함수를 더 잘게 쪼갤 수 있고,
```
const getTotalPrice = (f) => pipe(filter(f), total_price);

go(
  products,
  getTotalPrice((product) => product.price < 5000),
  log,
); // 9000

go(
  products,
  getTotalPrice((product) => product.price >= 5000),
  log,
); // 19000
```
이렇게 정말 잘게 쪼개서 사용할 수 있다.

함수형 프로그래밍은 이처럼 `함수들의 조합`으로 `고차 함수`를 만들고, 잘게 나뉜 함수들을 계속해서 `더 잘게 나누고`,<br>
결국 중복을 없애 `재사용성`을 높이는 것이 특징이다.

# 응용
주어진 값을 활용해 다양하게 응용을 해보자.
```javascript
const products = [
  {name: '반팔티', price: 15000, quantity: 1},
  {name: '긴팔티', price: 20000, quantity: 2},
  {name: '핸드폰케이스', price: 15000, quantity: 3},
  {name: '후드티', price: 30000, quantity: 4},
  {name: '바지', price: 25000, quantity: 5},
];

const add = (a, b) => a + b;

// products의 총 수량
const total_quantity = (products) =>
  go(
    products,
    map((product) => product.quantity),
    reduce(add),
  );
log(total_quantity(products)); // 15
```
go 함수를 사용해 총 수량을 값으로 얻을 수 있다. 하지만 함수형 프로그래밍이기에 만족하지 말고 더 생각해보자.
```javascript
const total_quantity = pipe(
  map((product) => product.quantity),
  reduce(add),
);

log(total_quantity(products)); // 15
```
앞서 사용했던 pipe 함수를 사용해 합성 함수로 만들어 가독성이 더 좋아졌다.<br>
하지만 현재 `total_quantity`함수는 products에서만 사용할 수 있기 때문에 추상화를 더 해보면,
```javascript
const sum = (f, iter) => go(iter, map(f), reduce(add));
const total_quantity = sum((product) => product.quantity, products);

log(total_quantity); // 15
```
sum 함수를 만들어 어느 곳에서든 사용할 수 있게 만들었다. 
`log(sum((a) => a, [1, 2, 3, 4])); // 10`
이처럼 더하고 싶은 모든 곳에 사용될 수 있다.

여기에서 만족할 수 있지만 마지막으로 한 번 더 가독성을 높여보자.
```javascript
const sum = curry((f, iter) => go(iter, map(f), reduce(add)));
const total_quantity = sum((product) => product.quantity);

log(total_quantity(products)); // 15
```
sum을 curry 함수로 묶어 최종적으로 마무리했다.
