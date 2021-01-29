range, take 함수를 통해 지연 평가에 대해 알아보자.

# range, L.range

## range

range 함수는 0부터 입력된 length까지의 배열을 출력해주는 함수이다.

```javascript
const range = (length) => {
  const res = [];
  let number = -1;
  while (++number < length) {
    res.push(number);
  }

  return res;
};

log(range(5)); //[0, 1, 2, 3, 4]
```

## L.range

L.range 함수는 지연된 range 함수로 제너레이터 함수이다.

```javascript
const L = {};
L.range = function* (length) {
  let number = -1;
  while (++number < length) yield number;
};

const lazyRange = L.range(2);
log(lazyRange.next()); // {value: 0, done: false}
log(lazyRange.next()); // {value: 1, done: false}
log(lazyRange.next()); // {value: undefined, done: true}
```

지연된 range 함수는 기존 range 함수보다 성능에 좋다.

```javascript
log(range(5)); // [0, 1, 2, 3, 4]
log(L.range(5)); // L.range {<suspended>}
```

range 함수는 실행하면 즉시 모든 값이 평가되어 메모리를 차지한다. 하지만 L.range 함수는 실행해도<br>
next() 메소드를 사용하기 전까지 값이 평가되지 않기 때문에 효율적으로 메모리를 사용해 성능을 향상시킬 수 있다.

# take

take 함수는 limit 값과 iterable을 받아서 limit만큼만 iterable을 실행하는 함수이다.

```javascript
const take = (limit, iter) => {
  const res = [];
  for (const value of iter) {
    res.push(value);
    if (res.length === limit) return res;
  }

  return res;
};

log(take(3, [1, 2, 3, 4, 5])); // [1, 2, 3]
```

이처럼 [1, 2, 3, 4, 5]의 배열을 limit 값인 3을 통해 [1, 2, 3]으로 만들어주는 함수이다.

## range, L.range 성능 비교

take 함수를 통해 range 함수와 L.range 함수의 성능 차이를 비교해보자.

```javascript
const test = (name, count, f) => {
  console.time(name);
  while (count--) f();
  console.timeEnd(name);
};

test('range', 10, () => take(5, range(1000000))); // range: 236.69287109375 ms
test('L.range', 10, () => take(5, L.range(1000000))); // L.range: 0.141845703125 ms
```

성능이 엄청나게 차이나는 것을 볼 수 있다. 이유를 살펴보자.

range 함수는 실행과 동시에 값이 평가되기 때문에 `range(1000000);`을 하게 되면 length가 1000000인 배열을 만든다.<br>
하지만 우리가 필요한 것은 length가 5인 배열이기 때문에 필요하지 않은 배열이 만들어진 것이다. 이는 메모리의 낭비이다.

반면에 L.range 함수는 배열을 만들지 않고, 이터레이터 자체를 전달하기 때문에 배열을 만들지도 않고, 메모리의 낭비도 발생하지 않는다.<br>
즉, `필요한 값만 평가해서 사용`하는 것이다.

이처럼 이터러블 중심 프로그래밍에서 지연 평가는 `필요한 값이 있을 때만 평가를 한다`는 점에서 중요한 의미를 가진다.

# L.map, L.filter

지연된 map, filter 함수를 만들어보자.

```javascript
const L = {};

L.map = function* (f, iter) {
  for (const value of iter) yield f(value);
};

L.filter = function* (f, iter) {
  for (const value of iter) {
    if (f(value)) yield value;
  }
};

const lazyMap = L.map((a) => a + 10, [1, 2, 3]);
log(lazyMap); // L.map {<suspended>}
log(lazyMap.next()); // {value: 11, done: false}
log(lazyMap.next()); // {value: 12, done: false}

const lazyFilter = L.filter((a) => a % 2, [1, 2, 3, 4, 5]);
log(lazyFilter); // L.filter {<suspended>}
log(lazyFilter.next()); // {value: 1, done: false}
log(lazyFilter.next()); // {value: 3, done: false}
```

이처럼 값을 평가하지 않고 가지고 있다가 next() 메소드로 필요할 때 값을 평가하여 사용할 수 있게 되었다.
