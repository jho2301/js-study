# 들어가기에 앞서
함수형 프로그래밍 강의를 듣고 있으면서 함수형 프로그래밍이 무엇인지 정확히 알지 못하고 있는 것 같아서 정리해봤다.
### 함수형 프로그래밍
> 객체지향 프로그래밍에서 객체를 기반으로 코드가 돌아가도록 구현했다면, 함수형 프로그래밍은 <br>
**함수를 기반**으로 코드가 돌아가도록 구현하는 것이다.

함수형 프로그래밍에는 다음와 같은 특징이 있다.
- 순수 함수를 통해 부작용을 없애고, 불변성을 유지한다.
- 선언형 프로그래밍이다.
- 고차 함수를 사용해 재사용성이 높다.

특징이 많아서 따로 정리해야 할 것 같고, 간단하게 순수 함수라는 것만 정확히 이해하고 넘어가자.

### 순수 함수
- 하나 이상의 인자를 받아서 그 인자를 가지고 항상 같은 값을 리턴해야 한다.
- 인자를 제외한 다른 변수는 사용할 수 없다.
- 부수 효과가 존재하면 안된다.

```javascript
// 순수 함수
const add = (a, b) => a + b;

log(add(10, 20)); // 30

// 순수 함수 x
let c = 30;
const add = (a, b) => {
  return a + b + c;
};

log(add(10, 20)); // 60
c = 40;
log(add(10, 20)); // 70
```
순수 함수는 같은 입력에 항상 같은 결과를 낳아야 하는데, 순수 함수가 아닌 경우는 보면 c 값을 외부에서 받아오기 때문에<br>
c 값이 변하면 결과 값도 달라지는 것을 확인할 수 있다. 이는 순수 함수가 될 수 없다.

또한, 
```javascript
let c = 30;
const add = (a, b) => {
  c = a;
  return a + b;
};

log(c); // 30
log(add(10, 20)); // 30
log(c); // 10
```
이처럼 외부 변수에 영향을 주는 부수 효과를 가지고 있는 경우에도 순수 함수가 될 수 없다.

즉, 순수하게 전달받은 인자만을 사용하고, 외부의 어떤 변수에도 영향을 미치지 않는 함수가 순수 함수이다.<br>
대표적인 예로 자주 사용하는 map, filter, reduce 메소드가 있고, 이에 대해 알아보도록 하자.

# map
이터러블/이터레이터 프로토콜을 따르는 모든 곳에 사용될 수 있는 다형성이 높은 map 함수를 만들어보자.
```javascript
const products = [
  {name: '사과', price: 5000},
  {name: '배', price: 6000},
  {name: '오렌지', price: 4000},
  {name: '키위', price: 8000},
  {name: '귤', price: 3000},
  {name: '바나나', price: 2000},
];

const map = (f, iter) => {
  const res = [];
  for (const v of iter) {
    res.push(f(v));
  }

  return res;
};

log(map((v) => v.price, products)); // [5000, 6000, 4000, 8000, 3000, 2000]
```
이렇게 보면 기존의 map 메소드와 다를 것이 없어보인다.<br>
아래 코드를 보자.
```javascript
const $elem = document.querySelectorAll('*');
log($elem.map((v) => v.nodeName)) // Uncaught TypeError: $elem.map is not a function
log(map((v) => v.nodeName, $elem)); // ["HTML", "HEAD", "META", "META", "LINK", "TITLE", "BODY", "SCRIPT", "SCRIPT"]
```
$elem은 유사 배열로 Array를 상속받지 않았기 때문에 map 메소드가 존재하지 않는다.<br>
그래서 map 메소드를 사용하기 위해서는 Array.from()을 사용해야 하는데 직접 만든 map 함수에서는 문제없이 출력된다.<br>
즉, 이터러블/이터레이터 프로토콜을 따르는 모든 곳에 사용될 수 있는 것이다.

게다가 앞서 배운 제너레이터 함수 또한 이터러블/이터레이터 프로토콜을 따르기 때문에 map 함수를 사용할 수 있다.
```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}

log(map((v) => v * v, gen())); // [1, 4, 9]
```
정말 다형성이 높다는 것을 알 수 있다.<br>
마지막으로 Map 객체를 변환해보자.
```javascript
const mapObject = new Map();
mapObject.set('a', 10);
mapObject.set('b', 20);

log(new Map(map(([key, value]) => [key, value * 2], mapObject))); // Map(2){"a" => 20, "b" => 40}
```
Map 객체의 원하는 키나 값을 변경할 수 있다.

# filter
filter도 map과 마찬가지로 만들 수 있다.
```javascript
const filter = (f, iter) => {
  const res = [];
  for (const v of iter) {
    f(v) && res.push(v);
  }

  return res;
};

log(...filter((v) => v.price < 5000, products)); // {name: "오렌지", price: 4000} {name: "귤", price: 3000} {name: "바나나", price: 2000}
log(
  filter(
    (v) => v % 2,
    (function* () {
      yield 1;
      yield 2;
      yield 3;
      yield 4;
      yield 5;
    })(),
  ),
); // [1, 3, 5]
```

# reduce
reduce는 값을 계속해서 누적하는 함수이다. <br>
만약 1 ~ 5까지의 배열을 더하는 것을 코드로 표현해보면
```javascript
const add = (a, b) => a + b;

log(add(add(add(add(add(0, 1), 2), 3), 4), 5)); // 15
```
이렇게 누적해서 함수의 인수로 함수가 들어가는 것을 볼 수 있다.

이것을 reduce 함수로 구현해보자.
```javascript
const reduce = (f, acc, iter) => {
  for (const v of iter) {
    acc = f(acc, v);
  }

  return acc;
};

log(reduce((a, b) => a + b, 0, [1, 2, 3, 4, 5])); // 15
```
reduce 함수는 map과 filter와 다르게 acc라는 값을 인자로 받고, 이를 초기값으로 활용해 게속해서 누적한다.<br>
또한 콜백 함수를 받아 콜백 함수의 로직대로 처리하기 위해 f(acc, v)를 누적하는 식으로 구현했다.

하지만 인자로 받는 acc(초기 값)이 없다면 iter가 없기 때문에 문제가 될 수 있다. 수정해보자.
```javascript
const reduce = (f, acc, iter) => {
  // 추가된 부분
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }

  for (const v of iter) {
    acc = f(acc, v);
  }

  return acc;
};

log(reduce((a, b) => a + b, [1, 2, 3, 4, 5])); // 15
```
추가된 부분을 살펴보면 iter이 없으면 초기 값이 없는 상태이기에 acc로 들어온 iter을 이터레이터로 변경해주고,<br>
초기 값을 이터레이터의 첫 번째 value로 설정한다.

즉, `log(reduce((a, b) => a + b, 1, [2, 3, 4, 5]));`으로 동작할 것이다.<br>
이렇게 간단하게 초기 값이 없어도 동작하도록 설정할 수 있다.

```javascript
log(reduce((total_price, product) => total_price + product.price, 0, products)); // 28000
log(
  reduce(
    (a, b) => a * b,
    (function* () {
      yield 1;
      yield 3;
      yield 5;
    })(),
  ),
); // 15
```
reduce 함수를 사용해 위와 같이 여러 가지를 응용해 볼 수 있다.

# map + filter + reduce 
이번에는 map과 filter, reduce를 조합해 사용해보자.
```javascript
const products = [
  {name: '사과', price: 5000},
  {name: '배', price: 6000},
  {name: '오렌지', price: 4000},
  {name: '키위', price: 8000},
  {name: '귤', price: 3000},
  {name: '바나나', price: 2000},
];

// 5000원 이하의 과일의 이름만 받아오고 싶을 때
log(
  map(
    (product) => product.name,
    filter((product) => product.price < 5000, products),
  ),
); // ["오렌지", "귤", "바나나"]

// 5000원 이하의 과일 가격의 합을 구하고 싶을 때
log(
  reduce(
    (acc, v) => acc + v,
    0,
    map(
      (product) => product.price,
      filter((product) => product.price < 5000, products),
    ),
  ),
); // 9000 (filter 먼저)

log(
  reduce(
    (acc, v) => acc + v,
    0,
    filter(
      (price) => price < 5000,
      map((product) => product.price, products),
    ),
  ),
); // 9000 (map 먼저)
```
