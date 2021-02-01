## 결과를 만드는 함수 `reduce`, `take`



### reduce

객체로부터 url의 queryString을 만들어내는 함수를 만들어 보자.

```javascript
const queryStr = obj =>
  go(
    obj,
    Object.entries, // [[key, value], [key, value],...]를 변환
    map(([k, v]) => `${k}=${v}`), // 구조분해를 통해 key와 value를 받음
    reduce((a, b) => `${a}&${b}`) // '&'로 연결
  );

console.log(queryStr({ limit: 10, offset: 10, type: 'notice' }));
// limit=10&offset=10&type=notice
```



`queryStr`은 obj를 받아서 그대로 obj로 전달하기 때문에 아래처럼 `pipe`로 대체가 가능하다.

```javascript
const queryStr = pipe(
  Object.entries,
  map(([k, v]) => `${k}=${v}`),
  reduce((a, b) => `${a}&${b}`)
);

console.log(queryStr({ limit: 10, offset: 10, type: 'notice' }));
// limit=10&offset=10&type=notice
```





이어서 `reduce` 를 통해 `join` 함수를 만들어 보자. 이 `join` 함수는 이터러블 값을 순회하면서 축약하기 때문에 Array에서만 사용할 수 있는  `join` 메서드 보다 훨씬 다형성이 높다.

```javascript
const join = curry((sep = ',', iter) => 
	reduce((a, b) => `${a}${sep}${b}`, iter)
);

console.log(a().join(' - ')); // 에러!, join메서드는 Array에만 사용가능
console.log(join(' - ', a())); // 10 - 11 - 12 - 13
```

위와 같이 제너레이터함수가 정의되어 있을 때, 일반적인 `join` 메서드로는 결과를 만들 수 없지만, 위에서 선언한 `join` 함수는 결과를 만들 수 있다.



그리고 만든 `join` 함수는 `reduce` 를 통해 축약을 했기 때문에, 이터러블 프로토콜을 따르고 있기 때문에 `join` 함수에게 가기전에 만들어지는 값들을 지연시킬 수 있다. 그래서 `map` 이 `L.map` 이여도 동일한 결과를 나타낸다. 그리고 `entries` 역시도 다음과 같이 제너레이터 함수로 지연평가 방식으로 구현할 수 있다.

```javascript
L.entries = function* (obj) {
   for (const k in obj) yield [k, obj[k]];
};

const join = curry((sep = ',', iter) => 
	reduce((a, b) => `${a}${sep}${b}`, iter)
);

const queryStr = pipe(
   L.entries,
   L.map(([k, v]) => `${k}=${v}`),
   join('&')
);

console.log(queryStr({ limit: 10, offset: 10, type: 'notice' }));
// limit=10&offset=10&type=notice
```



### take → find

`take` 함수를 통해 `find` 함수를 만들어 보자.

- 비효율적

```javascript
const users = [
  { age: 32 },
  { age: 31 },
  { age: 37 },
  { age: 28 },
  { age: 25 },
  { age: 32 },
  { age: 31 },
  { age: 37 },
];

const find = (f, iter) =>
  go(
    iter,
    filter(f), // take 하기 전에 모든 값들을 다 조회한다.
    take(1), // 하나의 값만 리턴한다.
    ([a]) => a // 구조분해, 배열에서 값을 꺼내 준다.
  );

console.log(find(u => u.age < 30, users));
// {age: 28}
```

`filter` 함수에 콘솔을 찍어보면, `take` 함수 이전에 모든 값들을 다 조회하므로 비효율적이다. 



- 효율적 : `L.filter` 지연 평가

```javascript
const find = (f, iter) => go(
   iter,
   L.filter(f),
   take(1),
   ([a]) => a
);

console.log(find(u => u.age < 30, users));
// {age: 28}
```

지연평가 함수인  `L.filter` 를 사용하면 필요할 때만 값을 조회하므로 효율적이다.



- `curry` 추가

```javascript
const find = curry((f, iter) => go(
   iter,
   filter(a=> (console.log(a), f(a))),
   take(1),
   ([a]) => a));

console.log(find(u => u.age < 30)(users));
// {age: 28}
```



## 지연성 / 이터러블 중심의 코드 다루기



### L.map, L.filter로 map과 filter 만들기

앞에서 배웠던 `map` , `filter` 함수를 `L.map`, `L.filter` 와 `take` 조합으로 만들어보자.



### `L.map` + `take` 로 `map` 만들기

```javascript
// L.map
L.map = curry(function* (f, iter) {
    for (const a of iter) {
        yield f(a);
    }
});

/* L.map + take 로 map 만들기 */
const map = curry((f, iter) => go(
   iter,
   L.map(f),
   take(Infinity)
));

//↓↓↓

const map = curry((f, iter) => go(
   L.map(f, iter),
   take(Infinity)
));

//↓↓↓

const map = curry(pipe(L.map, take(Infinity)));
```



### `L.filter` + `take` 로 `filter` 만들기

```javascript
// L.filter
L.filter = curry(function* (f, iter) {
  for (const a of iter) {
    if (f(a)) yield a;
  }
});

/* L.filter + take 로 filter 만들기 */
const filter = curry(pipe(L.filter, take(Infinity)));

console.log(filter(a => a % 2, L.range(4)));
// [1, 3]
```



### L.flatten, flatten

> `flatten` 함수는 하나의 배열 안에 있는 묶인 배열을 펼쳐서 하나의 배열로 만드는 역할을 해준다.

`[[1, 2], 3, 4, [5, 6], [7, 8, 9]]`  →  `[1, 2, 3, 4, 5, 6, 7, 8, 9]`



- `L.flatten`

```javascript
const isIterable = a => a && a[Symbol.iterator];
//이터러블인지 판별

L.flatten = function* (iter){
   for(const a of iter){
      if(isIterable(a)) for (const b of a) yield b; // === yield*
      else yield a;
   }
};

var it = L.flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]]);
console.log([...it]);
//[1, 2, 3, 4, 5, 6, 7, 8, 9]

console.log(take(3, L.flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]])));
// [1, 2, 3]
```



- `flatten`

 take의 조합으로 즉시평가할수 있는 flatten도 만들 수 있다.

```javascript
const flatten = pipe(L.flatten, take(Infinity));

console.log(flatten([[1, 2], 3, 4, [5, 6], [7, 8, 9]]));
// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```



### yield*

`yield*` 를 활용하면 다음과 같이 코드를 변경 할 수 있습니다.

>  `yield *iterable`은 `for (const val of iterable) yield val;` 과 같다.

```javascript
/* yield 사용 X */
L.flatten = function* (iter) {
   for(const a of iter) {
      if(isIterable(a)) for(const b of a) yield b
      else yield a;
   }
};

/* yield* 사용 O */
L.flatten = function* (iter) {
   for(const a of iter) {
      if(isIterable(a)) yield* a;
      else yield a;
   }
};
```



### L.deepFlat

만약 깊은 이터러블을 모두 평치고 싶다면 아래와 같이 `L.deepFlat` 함수를 구형하여 사용할 수 있다.

```javascript
L.deepFlat = function* f(iter) {
   for (const a of iter) {
      if (isIterable(a)) yield* f(a);
      else yield a;
   }
};

console.log([...L.deepFlat([1, [2, [3, 4], [[5]]]])]);
// [1, 2, 3, 4, 5];
```

