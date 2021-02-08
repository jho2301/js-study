## L.map, map, take 비동기 동시성으로 만들기

L.map, map, take 비동기적으로 만들려면 각 함수 내부 구현에서 프로미스를 거르는 작업을 추가해주면 된다.

 

```javascript
go([Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
   L.map(a => a + 10),
   take(2),
   console.log); 

// 출력: ["[object Promise]10", "[object Promise]10"]
```

기존의 L.map, map, take 에서는 비동기적으로(Promise) 동작하지 않는다.



- 기존 L.map

```javascript
// 기존 L.map 함수
L.map = curry(function* (f, iter) {
   for (const a of iter) {
     yield f(a);
   }
 });


// 프로미스값을 거르는 go1함수를 활용한 L.map
const go1 = (a, f) => a instanceof Promise ? a.then(f) : f(a);

L.map = curry(function* (f, iter) {
   for (const a of iter) {
      yield go1(a, f); // go1함수에서 프로미스면 풀어서 값을 만든다.
   }
});
```





```javascript
// 변경 전 take
const take = curry((l, iter) => {
  let res = [];
  iter = iter[Symbol.iterator]();
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    res.push(a);
    if (res.length == l) return res;
  }
  return res;
});

// 변경 후 take
const take = curry((l, iter) => {
   let res = [];
   iter = iter[Symbol.iterator]();
   return function recur(){ // 재귀함수를 반환한다.
      let cur;
      while (!(cur = iter.next()).done) {
         const a = cur.value;
         if(a instanceof Promise) return a.then( a => {
             res.push(a);
             if (res.length == l) return res
             return recur();
         });
         res.push(a);
         if (res.length == l) return res;
      }
      return res;
   } ();
});
```

재귀 함수를 리턴하는 이유는 while문에서 프로미스일 때 then을 리턴하기 때문에  반복문이 제대로 돌수 없기 때문이다. 재귀적으로 동작하는 부분을 재귀함수로 감싸서 리턴하는식으로 구현하고 내부는 재귀적으로 호출하는식으로 구현한다.



다시 찍어보면

```javascript
go([Promise.resolve(1), Promise.resolve(2), Promise.resolve(3)],
   L.map(a => a + 10),
   take(2),
   console.log); // [11, 12]
```



- map 함수

```javascript

const map = curry(pipe(L.map, takeAll));
```

map함수는 `L.map`과 `take` 함수로 구현했기 때문에 비동기적으로 잘 작동된다. 



## L.filter, filter 를 비동기 동시성으로 만들기

Kleisli Composition을 활용하여 L.filter, filter 함수에 프로미스를 받을 수 있도록 할 수 있다.



go구문에서 앞에서 오는 값이 Promise가 되어서 오거나, filter가 내뱉는 값이 Promise일때, 어떻게 하면 지연평가도 되면서 비동기 동시성을 모두 지원할 수 있는지 알아보자.

```javascript
go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => Promise.resolve(a * a)),
  L.filter(a => a % 2),
  take(2),
  console.log
);

// 출력: []
```

L.filter함수에서 들어오는 값이 프로미스이기 때문에 제대로 평가 되지 않는다. 



```javascript
// 변견 전 L.filter
L.filter = curry(function* (f, iter) {
    for (const a of iter) {
        if (f(a)) yield a;
    }
})

// 변견 후 L.filter
const go1 = (a, f) => a instanceof Promise ? a.then(f) : f(a);
const nop = Symbol('nop'); //reject에 아무런 작업을 하지 않기위한 값으로써의 nop 구분자.

L.filter = curry(function* (f, iter){
   for(const a of iter){
      const b = go1(a, f);
      if(b instanceof Promise) yield b.then(b => b ? a : Promise.reject(nop)); // b가 false일 때 다음 작업이 진행되지 않도록 하기위함.
      else if(b) yield a;
   }
});


```



`L.filter`에서 `nop` 이라는 `reject` 를 반환 받았을 때 `take` 함수에서 어떻게 처리해야할지를 추가 구현해줘야한다.

```javascript
const take = curry((l, iter) => {
   let res = [];
   iter = iter[Symbol.iterator]();
   return function recur(){
      let cur;
      while(!(cur = iter.next()).done){
         const a = cur.value;
         if(a instanceof Promise){
         return a
            .then(a => (res.push(a), res).length == l ? res : recur())
            .catch(e => e == nop ? recur() : Promise.reject(e)); // 추가!
         }
         res.push(a);
         if(res.length == l) return res;
      }
      return res;
   }();
});
```

`.catch(e => e == nop ? recur() : Promise.reject(e))`  e가 nop이면 다시 재귀를 돌려서 다음을 시도하고 만약 nop이 아닌 다른 에러이면 error를 뱉는다.



## 프로미스를 지원하는 reduce 만들기



```javascript
go([1, 2, 3, 4],
   L.map(a => Promise.resolve(a * a)),
   L.filter(a => Promise.resolve(a % 2)),
   reduce(add)
   console.log);
```



```javascript
// 기존 reduce
const reduce = curry((f, acc, iter) => {
   if (!iter) {
     iter = acc[Symbol.iterator]();
     acc = iter.next().value;
   } else {
     iter = iter[Symbol.iterator]();
   }
   return go1(acc, function recur(acc) {
     let cur;
     while (!(cur = iter.next()).done) {
       const a = cur.value;
       acc = f(acc, a); /* 문제가 되는 구문 */
       if (acc instanceof Promise) return acc.then(recur);
     }
     return acc;
   });
 });
```

reduce의 "acc = f(acc, a)"에 전달되는 값들이 1과 Promise를 전달하는 식으로 동작하기 때문에 정상적으로 동작하지 못한다. 명확하게 잘 동작하도록 바꿔보자.



```javascript
const reduceF = (acc, a, f) =>
  a instanceof Promise ?
    a.then(a => f(acc, a), e => e == nop ? 
    acc : 
    Promise.reject(e)) :
    f(acc, a);

const reduce = curry((f, acc, iter) => {
   if (!iter) {
     iter = acc[Symbol.iterator]();
     acc = iter.next().value;
   } else {
     iter = iter[Symbol.iterator]();
   }
   return go1(acc, function recur(acc) {
     let cur;
     while (!(cur = iter.next()).done) {
       acc = reduceF(acc, cur.value, f); /* reduceF로 받기 */
       if (acc instanceof Promise) return acc.then(recur);
     }
     return acc;
   });
 });
```

