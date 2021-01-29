## 서론

함수가 중첩되어있는 경우 가독성이 좋지않다.

```javascript
console.log(
  reduce(
    add,
    map(
      p => p.price,
      filter(p => p.price < 20000, products)
    )
  )
);
```



함수형 프로그래밍에서는 함수를 값으로 활용한다.

함수를 값으로 다룬다면 함수의 표현력을 높일 수 있다. 



함수들을 연속적으로 사용하는 함수, 함수들을 함축하는 함수를 구현하여

중첩된 함수의 가독성을 높여보자!



## go

> 첫번째 인자에는 시작되는 값을 넣고 나머지 인자에는 함수들을 받아 값을 다음 함수로 넘기면서 **차례대로 함수를 실행**한다.



### 구현

```javascript
// reduce 함수
const reduce = (f, acc, iter) => {
    // acc는 옵셔널 acc가 생략됬을 경우 예외처리
    if(!iter) {
        iter = acc[Symbol.iterator]();
        acc = iter.next();
    }
    for (const a of iter) {
        acc = f(acc, a);
    }
    return acc;
};

// go 함수
const go = (...args) => reduce((a, f) => f(a), args);
```

`go`함수는 함수들을 차례로 실행하면서 하나의 결과로 축약하는 과정이기 때문에 `reduce` 함수를 활용하여 구현한다.



### 활용

```javascript
go(
	0,
    a => a + 1,
    a => a + 10,
    a => a + 100,
    console.log
);
// 111

go(
	add(0, 1), // 2개 이상의 인자를 처리해 하나의 값을 반환하는 함수는 첫 번째 인자로!
    a => a + 1,
    a => a + 10,
    a => a + 100,
    console.log
);
// 111
```

값 0을 시작으로 다음 함수로 넘기고 다시 그 결과를 다음 함수로 넘기면서 하나의 값으로 축약된다. 



## pipe

> 여러 함수들을 차례대로 합쳐서 하나의 **함수를 리턴**한다.
>
> 리턴 받은 함수를 활용하여 여러 함수를 차례대로 실행한다.



### 구현

```javascript
// const pipe = (...fs) => (a) => go(f(a), ...fs); // 인자가 1개일 때
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs); // 인자가 2개일 때
```

pipe함수는 go함수와 같은 동작을 하기 때문에 내부적으로 go함수를 활용한다. 



### 활용

```javascript
const f = pipe(
    (a, b) => a + b, // 여러개의 인자 처리를 첫 함수에서 축약
    a => a + 10,
    a => a + 100
);

console.log(f(0, 1));
```

`pipe` 함수의 리턴 함수에서 여러개의 인자를 받기 위해서는  `pipe` 함수의 첫번째 인자의 함수에서 하나의 값으로 축약한다.

`pipe` 함수 구현에서 첫 번째 함수 `f` 와 나머지 함수인 `...fs` 를 따로 받는다. 그렇게 리턴된 합성함수는 여러개의 인자 `...as` 를 받아 `f` 함수에 넣고 결과를 나머지 함수들이 받으면서 차례대로 실행된다.



## go함수를 사용하여 코드 개선하기



- 이전 코드

```javascript
// map
const map = (f, iter) => {
  const res = [];
  for (const el of iter) {
    res.push(f(el));
  }
  return res;
};

// filter
const filter = (f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
};

// reduce
const reduce = (f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
};

// add 함수
const add = (a, b) => a + b;

// go 함수
const go = (...args) => reduce((a, f) => f(a), args);

// 예제
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 },
];

// 2만원  이하의 제품들의 가격의 총합
console.log(
  reduce(
    add,
    map(
      p => p.price,
      filter(p => p.price < 20000, products)
    )
  )
);
// 30000
```



- go함수를 사용하여 개선된 코드

```javascript
go(
	products,
    products => map(p => p.price, products),
    prices => filter(price => price < 20000, prices),
    prices => reduece(add, prices),
    console.log
);
// 30000
```



## curry

> 인자가 원하는 갯수만큼 들어왔을 때 받아온 함수를 실행한다.
>
> 인자가 원하는 갯수 이하일 때는 받아온 함수를 대기 시킨다.
>
> 즉, 어떤 함수가 원하는 갯수가 될때까지 대시키도록 만드는 함수이다.



### 구현

```javascript
const curry = f => (a, ..._) => _.length ? f(a, ..._) : (..._) => f(a, ..._);
```



### 활용

```javascript
// const multi = (a, b) => a * b;
const multi = curry((a, b) => a * b);

console.log(multi);
console.log(multi(3));
console.log(multi(3)(2));

const multi3 = multi(3);
console.log(multi3(1));
console.log(multi3(2));
console.log(multi3(3));
console.log(multi3(4));
```



## go함수 + curry함수



- go함수 + curry함수를 사용하여 개선된 코드

```javascript
// curry
const curry = f => (a, ..._) => (_.length ? f(a, ..._) : (..._) => f(a, ..._));

// map
const map = curry((f, iter) => {
  const res = [];
  for (const el of iter) {
    res.push(f(el));
  }
  return res;
});

// filter
const filter = curry((f, iter) => {
  let res = [];
  for (const a of iter) {
    if (f(a)) res.push(a);
  }
  return res;
});

// reduce
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  }
  for (const a of iter) {
    acc = f(acc, a);
  }
  return acc;
});

// add 함수
const add = (a, b) => a + b;

// go 함수
const go = (...args) => reduce((a, f) => f(a), args);

// 예제
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 },
];
```

> 기존에 구현한 map, filter, reduce 함수에 curry함수를 적용한다.



- 최종 개선 과정과 결과

```javascript
// 함수의 중첩사용
console.log(
  reduce(
    add,
    map(
      p => p.price,
      filter(p => p.price < 20000, products)
    )
  )
);

// go함수를 활용하여 가독성 업! 로직의 순서대로 함수실행!
go(
  products,
  products => filter(p => p.price < 20000, products),
  products => map(p => p.price, products),
  prices => reduce(add, prices),
  console.log
);

// curry를 감싼 함수들 (중간단계)
go(
  products,
  products => filter(p => p.price < 20000)(products),
  products => map(p => p.price)(products),
  prices => reduce(add)(prices),
  console.log
);

// curry를 감싼 함수들 활용 (최종단계)
go(
  products,
  filter(p => p.price < 20000),
  map(p => p.price),
  reduce(add),
  console.log
);
```

> curry를 감싼 함수들은 우선 대기하다가 앞에서 리턴받은 인자를 받아오면 즉시 실행이된다.



## 함수 조합으로 함수 만들기 (pipe)

> **pipe를 활용하면 함수들을 의미적으로 하나의 함수로 합처서 사용할 수 있다.**



- 예제

```javascript
go(
  products,
  filter(p => p.price < 20000),
  map(p => p.price),
  reduce(add),
  console.log
);

go(
  products,
  filter(p => p.price >= 20000),
  map(p => p.price),
  reduce(add),
  console.log
);
```

같은 데이터를 가지고  두개의 함수가 있다.

이 두 함수는 

```javascript
  reduce(add),
  console.log
```

라는 부분이 중복이다.

pipe를 이용하면 중복을 의미적으로 하나의 함수로 합칠 수 있다.



- pipe를 활용하여 가독성 업!

```javascript
const getTotaPrice = pipe(
  map(p => p.price),
  reduce(add)
);

go(
  products,
  filter(p => p.price < 20000),
  getTotaPrice,
  console.log
);

go(
  products,
  filter(p => p.price >= 20000),
  getTotaPrice,
  console.log
);
```





