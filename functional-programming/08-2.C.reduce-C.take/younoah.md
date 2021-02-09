## 지연된 함수열을 병렬적으로 평가하기 - C.reduce



```javascript
const delay1000 = a =>
  new Promise(resolve => setTimeout(() => resolve(a), 1000));

go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => delay1000(a * a)),
  L.filter(a => a % 2),
  reduce(add),
  console.log
);
```

L.map에서 한번 실행할때 마다 1초정도 걸리는데, 총 6개의 인자이므로 6초가 걸린다.

개씩 실행시켜서 코드가 순회하기 때문이다.

 이를 순차적이 아닌 병렬적으로 처리한다면 시간은 단축될 것이다.



병렬적으로 동작하는 C.reduce를 만들어 보자.

```javascript
const C = {};

C.reduce = curry((f, acc, iter) =>
  iter ? reduce(f, acc, [...iter]) : reduce(f, [...acc])
);
```

reduce로 해당하는 코드를 그대로 넘기지만, `[...iter]` 를 통해서 대기된 함수를 모두 다 실행을 시켜버리고 다시한번 reduce에서 값을 꺼낸다. 

즉, go 첫 인자로 나온 값들을 실행을 전부 한 후 개별적으로 비동기 제어를 해서 앞에서부터 누적을 시키는 것이다.



```javascript
// 순차적
console.time('');
go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => delay1000(a * a)),
  L.filter(a => a % 2),
  reduce(add),
  console.log, // 35
  _ => console.timeEnd('')
);
// 약 6000ms

// 병렬적
console.time('');
go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => delay1000(a * a)),
  L.filter(a => a % 2),
  C.reduce(add),
  console.log, // 35
  _ => console.timeEnd('')
);
// 약 1000ms
```





```javascript
const C = {};
C.reduce = curry((f, acc, iter) =>
  iter ? reduce(f, acc, [...iter]) : reduce(f, [...acc])
);

const delay1000 = a =>
  new Promise(resolve => setTimeout(() => resolve(a), 1000));

go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => delay1000(a * a)),
  L.filter(a => delay1000(a % 2)),
  L.map(a => delay1000(a * a)),
  C.reduce(add),
  console.log
);
```

catch되지 않은 부분이 있다고 나오지만, 결과는 정상적으로 나온다. 

자바스크립트 특성상 콜스택에 프로미스 reject이 있으면 출력이 된다.

catch되지 않는 부분이 있지만, 이후에 잘 catch를 해서 정리를 해줄 것이기 때문에, 비동기적으로 해당하는 에러를 캐치해줄 것이란것을 명시해줘야 한다.



```javascript
function noop() {}
const catchNoop = arr => (
  arr.forEach(a => (a instanceof Promise ? a.catch(noop) : a)), arr
);

C.reduce = curry((f, acc, iter) => {
  const iter2 = catchNoop(iter ? [...iter] : [...acc]);
  return iter ? reduce(f, acc, iter2) : reduce(f, iter2);
});
```

위와 같이 해주면, 정상적으로 동작하면서, catch에러를 품지않는다.



```javascript
C.take = curry((l, iter) => take(l, catchNoop([...iter])));
```

take도 똑같은 방식으로 작성할 수 있다.



```javascript
go(
  [1, 2, 3, 4, 5, 6],
  L.map(a => delay1000(a * a)),
  L.filter(a => delay1000(a % 2)),
  L.map(a => delay1000(a * a)),
  C.take(2),
  reduce(add),
  console.log
);
```

C.take도 병렬적으로 실행되기 때문에 기존 take보다 빠르게 실행된다.