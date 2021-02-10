## 즉시 병렬적 평가하기 - C.map, C.filter

 특정 함수 라인에서만 병렬적으로 평가하는 C.map과 C.filter를 구현해보자.

기존의 map과 filter는 각각 L.map과 takeAll, L.filter와 takeAll로 구현했기 때문에 간단하게 C.takeAll로 구현하면 된다.



**C.takeAll**

```javascript
C.take = curry((l, iter) => take(l, catchNoop([...iter])));

C.takeAll = C.take(Infinity);
```



**C.map, C.filter 구현**

```javascript
C.map = curry(pipe(L.map, C.takeAll));

C.filter = curry(pipe(L.filter, C.takeAll));
```



**예시**

```javascript
C.map(a => delay1000(a * a), [1, 2, 3, 4]).then(console.log);
// [1, 4, 9, 16]

C.filter(a => delay1000(a % 2), [1, 2, 3, 4]).then(console.log);
// [1, 3]
```

map, C.filter를 실행해보면 동시에 평가되는 것을 확인 할 수 있다.