# C.map, C.filter

```javascript
C.takeAll = C.take(Infinity);

C.map = curry(pipe(L.map, C.takeAll));
C.filter = curry(pipe(L.filter, C.takeAll));

C.map((a) => delay500(a * a), [1, 2, 3, 4, 5]).then(log); // [1, 4, 9, 16, 25]
C.filter((a) => delay500(a % 2), [1, 2, 3, 4, 5]).then(log); // [1, 3, 5]
```

reduce나 take에서 값을 요구하기 전에 즉시 병렬적으로 평가해 사용할 수 있도록 <br>
C.map과 C.filter도 만들어 주었다.

지금까지 배운 즉시, 지연, 병렬 함수를 조합해 상황에 맞게 사용할 수 있도록 하자.

# C.reduce, C.take 간결하게 정리

```javascript
// 기존
const catchNoop = (arr) => (
  arr.forEach((a) => (a instanceof Promise ? a.catch(noop) : a)), arr
);

C.reduce = curry((f, acc, iter) => {
  const iter2 = catchNoop(iter ? [...iter] : [...acc]);
  return iter ? reduce(f, acc, iter2) : reduce(f, iter2);
});

C.take = curry((l, iter) => take(l, catchNoop([...iter])));

// 변경 후
const catchNoop = ([...arr]) => (
  arr.forEach((a) => (a instanceof Promise ? a.catch(noop) : a)), arr
);

C.reduce = curry((f, acc, iter) =>
  iter ? reduce(f, acc, catchNoop(iter)) : reduce(f, catchNoop(acc)),
);

C.take = curry((l, iter) => take(l, catchNoop(iter)));
```
