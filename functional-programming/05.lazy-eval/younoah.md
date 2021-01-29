## 지연 평가 (Lazy Evaluation)

느긋한 연산(지연 평가)은 계산읜 결과 값이 필요할 때 까지 계산을 늦추는 기법이다. 필요할 때까지 계산을 늦추어 불필요한 계산을 줄일수 있다.

#### 지연평가의 이점

1. 불필요한 계산을 하지 않으므로 빠른 계산이 가능하다.
2. 무한 자료 혹은 큰 범위의 자료 구조를 부담없이 사용할 수 있다.
3. 복작합 수식에서 오류를 피할 수 있다.



> 제너레이터와 이터러블 프로토콜을 따르는 함수들과 함께 사용하면 지연평가를 구현할 수 있다.



### range

> 지정한 범위만큼 배열을 생성하는 함수

```javascript
const range = length => {
  let i = -1;
  let res = [];
  while (++i < length) {
    res.push(i);
  }
  return res;
};

var list = range(4);
console.log(list); // [0, 1, 2, 3], 값이 생성되었다.
console.log(reduce(add, list)); // 6
```





### L.range

> 제너레이터를 활용하여 range함수 구현

```javascript
const L = {};

L.range = function* (length) {
  let i = -1;
  while (++i < length) {
    yield i;
  }
};

var list = L.range(4);
console.log(list); // L.range {<suspended>}, 값이 생성되지 않았다.
console.log(reduce(add, list)); // 6 <- 순회 평가가 발생할 때 동적으로 값이 생성된다.
```

제너레이터를 활용하면 지연 평가되는 range를 구현할 수 있다.

지정한 범위만큼 배열을 미리 생성하지 않고, 평가할 때 동적으로 생성하는 함수입니다.

실제로 순회가 발생하는 평가가 실행될 때 동적으로 값을 생성한다.



### take

> 이터러블에서 limit개수만큼 뽑아내어 반환하는 함수

```javascript
const take = (limit, iter) => {
  let res = [];
  for (const a of iter) {
    res.push(a);
    if (res.length == limit) return res;
  }
  return res;
};

// range
console.time('');
console.log(take(5, range(100000))); // [0, 1, 2, 3, 4]
console.timeEnd('');
// 4.06982421875 ms

//L.range
console.time('');
console.log(take(5, L.range(100000))); // [0, 1, 2, 3, 4]
console.timeEnd('');
// 0.299072265625 ms
```

`range` 같은 경우 100000의 배열을 만든 후 5개를 출력하지만, `L.range` 는 미리 배열을 만들지 않고, 필요한 5개의 값만 만든다.

또한 `range` 같은 겨우 `infinity` 값을 인자로 사용할 수 없지만,  `L.range` 는 인자로 `infinity` 값을 집어 넣어 사용할 수 있다.



## 지연성을 가진 함수로 구현



### L.map

```javascript
L.map = function* (f, iter) {
  for (const a of iter) yield f(a);
};

var it = L.map(a => a + 10, [1, 2, 3]); // 이 때 까지는 값이 생성이 안되어 있다.

// 순회평가가 발생할 때 값이 생성된다.
console.log(it.next()); // {value: 11, done: false}
console.log(it.next()); // {value: 12, done: false}
console.log(it.next()); // {value: 13, done: false}

```

L.map은 새로운 Array를 만들지 않고, 이터러블을 순회하면서, 각 요소에 대해 함수를 적용한 값을 yield를 통해 게으른 평가를 수행한다.



### L.filter

```javascript
L.filter = function* (f, iter) {
  for (const a of iter) if (f(a)) yield a;
};

var it = L.filter(a => a % 2, [1, 2, 3, 4]);
console.log(it.next()); // {value: 1, done: false}
console.log(it.next()); // {value: 3, done: false}
```

L.map과 거의 유사한데, 이터러블을 순회하면서, 조건결과 값이 참인 경우에만 값을 yield를 통해 발생한다.

