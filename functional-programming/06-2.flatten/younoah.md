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



## L.flatMap, flatMap



flatMap은 map한 값에 flatten한 것과 동일한 값을 가진다. 하지만 map과 flatten을 하는것은 모든 값을 참조하기 때문에 비효율적이다.



```javascript
// flatMap
console.log([[1, 2], [3, 4], [5, 6, 7]].flatMap(a => a.map(a => a * a)));
// [1, 4, 9, 16, 25, 36, 49]

// map + flatten
console.log(flatten([[1, 2], [3, 4], [5, 6, 7]].map(a => a.map(a => a * a))));
// [1, 4, 9, 16, 25, 36, 49]
```

먼저 map을 하고 그 결과를 flatten하면 flatMap과 정확히 일치한다. 하지만 두 코드는 배열의 처음부터 끝까지 순회하기 때문에 시간복잡도의 차이는 없다.

하지만 flatMap된 결과에서 take함수를 사용하여 일부분만 뽑아 올때는 지연성으로 작성되지 않았기 때문에 비효율적으로 동작하게된다.



따라서 지연성이 있는 L.flatten을 구현해보는것이 이번 강의 목표다.

### L.flatMap

다형성이 높은 L.flatMap을 만들어 보자.

```javascript
L.flatMap = curry(pipe(L.map, L.flatten));

var it1 = L.flatMap(map(a => a * a), [[1, 2], [3, 4], [5, 6, 7]]);
console.log([...it1]); // [1, 4, 9, 16, 25, 36, 49]

var it2 = L.flatMap(a => a, [[1, 2], [3, 4], [5, 6, 7]]);
console.log([...it2]); // [1, 2, 3, 4, 5, 6, 7]
```



### flatMap

L.flatten이 아닌 flatten을 사용하면 즉시 평가하는 flatMap을 만들수 있다.

```javascript
const flatMap = curry(pipe(L.map, flatten));

console.log(flatMap(a => a, [[1, 2], [3, 4], [5, 6, 7]]));
// [1, 2, 3, 4, 5, 6, 7]
```



## 2차원 배열 다루기

L.flatMap을 활용하면 2차원 배열을 편하게 다룰 수 있다.

```javascript
const arr = [
   [1, 2],
   [3, 4, 5],
   [6, 7, 8],
   [9, 10]
];

go(arr,
   L.flatten,
   L.filter(a => a % 2),
   L.map(a => a * a),
   take(4),
   reduce(add),
   console.log);
//84
```



## 지연성/ 이터러블 프로그래밍, 실무적용

지연성을 활용해서 실무에 어떻게 적용하는지 예시를 보자.

- 유저 정보

```javascript
var users = [
   {
      name: 'a', age: 21, family: [
         {name: 'a1', age: 53}, {name: 'a2', age: 47},
         {name: 'a3', age: 16}, {name: 'a4', age: 15}
      ]
   },
   {
      name: 'b', age: 24, family: [
         {name: 'b1', age: 58}, {name: 'b2', age: 51},
         {name: 'b3', age: 19}, {name: 'b4', age: 22}
      ]
   },
   {
      name: 'c', age: 31, family: [
         {name: 'c1', age: 64}, {name: 'c2', age: 62}
      ]
   },
   {
      name: 'd', age: 20, family: [
         {name: 'd1', age: 42}, {name: 'd2', age: 42},
         {name: 'd3', age: 11}, {name: 'd4', age: 7}
      ]
   }
];
```



- 유저정보 활용

```javascript
go(users,
   L.map(u => u.family),
   L.flatten, // 2차원 배열 펼치기
   L.filter(u => u.age < 20), // 20세 이하 필터
   L.map(u => u.age), // 나이만
   take(3), // 3개만
   console.log); // 16, 15, 19
```

