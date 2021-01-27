## 서론

map, filter, reduce 함수들을 매우 유용한 함수들이다. 하지만 **배열**에서만 사용할 수 있다.

자바스크립트의 환경에서는 배열은 아니더라도 많은 api의 헬퍼 함수들과 값들이 이터러블 프로토콜은 따른다.

따라서 map, filter, reduce 함수들을 이터러블 프로토콜을 따르게 구현하여 사용하면 개발할 때 유용하게 사용할 수 있다.



## map

> 이터러블의 모든 요소를 돌면서 매핑되는 요소들의 값으로 새로운 **배열**로 리턴한다.



### 구현

```javascript
const map = (f, iter) => {
    let res = [];
    for (const a of iter) {
        res.push(f(a));
    }
    return res;
};
```



### 사용예시

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 },
];

console.log(map(p => p.name, products)); 
// ["반팔티", "긴팔티", "핸드폰케이스", "후드티", "바지"]

console.log(map(p => p.price, products)); 
// [15000, 20000, 15000, 30000, 25000]]
```





## filter

> 이터러블의 모든 요소를 돌면선 조건에 맞는 요소만 걸러서 새로운 **배열**로 리턴한다.



### 구현

```javascript
const fileter(f, iter) {
    let res = [];
    for (const a of iter) {
        if (f(a)) res.push(a);
    }
    return res;
};
```



### 사용예시

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 },
];

console.log(...filter(p => p.price < 20000, products));
// {name: "반팔티", price: 15000}
// {name: "핸드폰케이스", price: 15000}

console.log(...filter(p => p.price >= 20000, products));
// {name: "긴팔티", price: 20000}
// {name: "후드티", price: 30000}
// {name: "바지", price: 25000}
```





## reduce

> 이터러블의 모든 요소의 값을 하나의 값으로 (누적하여) 축약한 값을 리턴한다.



### 구현

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 },
];

const reduce(f, acc, iter) {
    // acc는 옵셔널이기 때문에 acc가 생략됬을 경우 예외처리
    if(!iter) {
        iter = acc[Symbol.iterator]();
        acc = iter.next();
    }
    for (const a of iter) {
        acc = f(acc, a);
    }
    return acc;
};

console.log(reduce(add, 0, [1, 2, 3, 4, 5]));
// 15

console.log(reduce(add, [1, 2, 3, 4, 5]));
// 15

console.log(
  reduce((total_price, product) => total_price + product.price, 0, products)
);
// 105000
```



## 복합 사용예시

- 대상 이터러블 객체

```javascript
const products = [
  { name: '반팔티', price: 15000 },
  { name: '긴팔티', price: 20000 },
  { name: '핸드폰케이스', price: 15000 },
  { name: '후드티', price: 30000 },
  { name: '바지', price: 25000 },
];
```



- 제품의 가격만 뽑기

```javascript
console.log(map(p => p.price, products));
// [15000, 20000, 15000, 30000, 25000]
```



- 20000만원 이하의 제품들의 가격만 뽑기

```javascript
console.log(filter(price => price < 20000, map(p => p.price, products)));
//  [15000, 15000]
```



- 20000만원 이하의 제품들의 가격의 총합

```javascript
const add = (a, b) => a + b;

console.log(
  reduce(
    add,
    filter(
      n => n < 20000,
      map(p => p.price, products)
    )
  )
);
```

