# L.flatten, flatten
flatten 함수는 flat 메소드와 같이 배열을 펼쳐주는 함수이다.
```javascript
const isIterable = (value) => value && value[Symbol.iterator];

L.flatten = function* (iter) {
  for (const value of iter) {
    if (isIterable(value)) {
      for (const flatValue of value) yield flatValue;
    } else {
      yield value;
    }
  }
};

go([1, [2, 3], 4], L.flatten, take(Infinity), log); // [1, 2, 3, 4]
```
지연된 L.flatten 함수를 통해 즉시 값을 구하는 flatten 함수도 구현할 수 있다.
```javascript
const flatten = pipe(L.flatten, take(Infinity));

log(flatten([1, [2, 3], 4])); // [1, 2, 3, 4]
```

# yield*
`yield* 표현식은 다른 generator 또는 이터러블(iterable) 객체에 yield를 위임할 때 사용됩니다. - MDN`
```javascript
L.flatten = function* (iter) {
  for (const value of iter) {
    if (isIterable(value)) yield* value;
    // for (const flatValue of value) yield flatValue
    else {
      yield value;
    }
  }
};
```
위의 L.flatten을 조금 더 간결하게 `yield* value;`로 사용할 수 있다.

# L.deepFlat, deepFlat
위의 flatten 함수는 한 단계의 iterable만을 펼쳐주는데, 모든 iterable을 펼치는 함수가 L.deepFlat이다.
```javascript
L.deepFlat = function* deep(iter) {
  for (const value of iter) {
    if (isIterable(value)) yield* deep(value);
    // for (const flatValue of deep(value)) yield flatValue
    else {
      yield value;
    }
  }
};

const deepFlat = pipe(L.deepFlat, take(Infinity));

go([1, [2, 3, [4, 5]]], L.deepFlat, take(Infinity), log); // [1, 2, 3, 4, 5]
log(deepFlat([1, [2, 3, [4, 5]]])); // [1, 2, 3, 4, 5]
```

# L.flatMap, flatMap
flatMap 함수는 map 후에 flat을 하는 것을 하나의 함수로 만든 것이다.
```javascript
L.flatMap = curry(pipe(L.map, L.flatten));
const flatMap = curry(pipe(L.map, flatten));

const arr = [
  [1, 2],
  [3, 4],
  [5, 6, 7],
];

log(
  flatMap(
    map((v) => v + 1),
    arr,
  ),
); // [2, 3, 4, 5, 6, 7, 8]
```

# 2차원 배열 다루기
```javascript
const arr = [
  [1, 2],
  [3, 4, 5],
  [6, 7, 8],
  [9, 10],
];

go(
  arr,
  L.flatten,
  L.filter((v) => v % 2),
  reduce((a, b) => a + b),
  log,
); // 25
```
