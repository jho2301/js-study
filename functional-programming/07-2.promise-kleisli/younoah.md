## Kleisli Composition 관점에서의 Promise 활용

### Kleisli Composition

오류가 있을수 있는 상황에서 안전하게 합성 함수가 동작하도록 하기 위한 규칙

오류라 하믄 **외부변화**에 의해 잘못된 인자를 받는 경우 혹은 결과를 정확히 전달하지 못할 경우를 말한다.



### 외부변화에 따른 오류

 외부에 어떠한 변화가 일어난다면 상황에 따라 합성 함수에서 에러가 일어 날 수 있다.

- 세팅

```javascript
var users = [
   {id: 1, name: 'aa'},
   {id: 2, name: 'bb'},
   {id: 3, name: 'cc'}
];

const getUserById = id =>
   find(u => u.id == id, users);

const f = ({name}) => name;
const g = getUserById;

const fg = id => f(g(id));

console.log(fg(2)); // bb
```



- 외부 변화 이전과 이후에 따른 합성함수의 동작

```javascript
/* 외부 변화 전 */
const res1 = fg(2);
console.log(r); // bb


/* 외부 변화 후 */
// user 객체의 변화 발생
users.pop();
users.pop();

const res2 = fg(2);
console.log(r2); /* error발생 */
```

유저 객체에 어떠한 변화도 없을 때는 합성함수가 잘 동작한다.

하지만 유저 객체에 변화(`user.pop`)가 발생한 이후 없는 값을 조회하므로 합성함수에서 에러가 발생한다.



이런 외부 변화에 따른 합성함수의 에러가 발생할 수 있는 상황을 안전하게 대비하는 것을 **Kleisli Composition** 이라 하고 이를 구현하기 위해, 함수의 합성을 **Promise**를 통해 구현한다.

값을 찾았을 때 그 값이 없으면 Promise.reject()를 반환하면 된다.



### Promise로 구현한 합성함수

```javascript
const getUserById = id =>
   find(u => u.id == id, users) || Promise.reject('존재하지 않는 값');
const f = ({name}) => name;
const g = getUserById;

const fg = id => Promise.resolve(id).then(g).then(f).catch(err => err);
fg(2).then(console.log); // bb

users.pop();
users.pop();

fg(2).then(console.log); // "존재하지 않는 값"
```



## go, pipe, reduce에서 비동기 제어

go, pipe, reduce 같이 체이닝 되는 합성 함수에서 비동기적으로 동작될 수 있게 Promise 사용해보도록 하자.



```javascript
go(1,
   a => a + 10,
   a => Promise.resolve(a + 100),
   a => a + 1000,
   console.log);

/* [object Promise]1000 */
```

기존의 go함수안에서 Promise를 사용하면 Promise의 리턴값 때문에 올바르게 함수가 동작하지 않는다. 



- 기존의 go, pipe, reduce 함수

```javascript
// go 함수
const go = (...args) => reduce((a, f) => f(a), args);

// pipe 함수
const pipe = (f, ...fs) => (...as) => go(f(...as), ...fs);

// reduce 함수
const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  let cur;
  while (!(cur = iter.next()).done) {
    const a = cur.value;
    acc = f(acc, a);
  }
  return acc;
});
```

pipe는 go를, go는 reduce를 사용하고 있기 때문에, reduce만 수정하면 이 문제를 해결 할 수 있다.



- 프로미스의 값을 받을수 있는 reduece

```javascript
const go1 = (a, f) => a instanceof Promise ? a.then(f) : f(a);

const reduce = curry((f, acc, iter) => {
  if (!iter) {
    iter = acc[Symbol.iterator]();
    acc = iter.next().value;
  } else {
    iter = iter[Symbol.iterator]();
  }
  // 재귀, Promise 값을 기다려서 만들어지는 값으로 변환 
  return go1(acc, function recur(acc) {
    let cur;
    while (!(cur = iter.next()).done) {
      const a = cur.value;
      acc = f(acc, a);
      if (acc instanceof Promise) return acc.then(recur);
    }
    return acc;
  });
});
```



## Promise.then의 중요한 규칙

> Promise.then에서 then 메서드를 통해 결과를 꺼냈을때의 값이 반드시 Promise가 아니다.



```javascript
Promise.resolve(Promise.resolve(Promise.resolve(1))).then(console.log); // 1
```

위와같이 Promise가 중첩되어 선언되어 있어도, 한번의 then으로 안에있는 결과값을 얻을 수 있다. 

즉, Promise 체인이 연속적으로 대기가 걸려있어도, 내가 원하는 곳에서 한번의 then으로 해당하는 결과를 얻을 수 있다.



```javascript
 new Promise(resolve => resolve(new Promise(resolve => resolve(1)))).then(console.log); // 1
```

위와 같이 연속적으로 resolve를 한다고 하더라도, 한번의 then으로 값을 얻을 수 있다.

