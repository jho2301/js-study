# async/await

```javascript
const delay = (time) => {
  return new Promise((resolve) => setTimeout(() => resolve(), time));
};

const delayIdentity = async (a) => {
  await delay(500);
  return a;
};

const asyncF = async () => {
  const a = await delayIdentity(10);
  const b = await delayIdentity(5);
  log(a + b);
};

asyncF(); // 15
```

### async/await의 특징

- 비동기 => 동기
- async를 사용한 함수는 Promise를 return하는 함수로 변경
- Promise의 결과를 await하는 것이기 때문에 Promise를 return하는 함수가 반드시 필요

```javascript
// async를 사용한 함수는 Promise를 return하는 함수로 변경
const asyncF = async () => {
  const a = 10;
  const b = 5;
  return a + b;
};

log(asyncF()); // Promise {<fulfilled>: 15}
```

# Array.prototype.map vs FxJS-map

```javascript
// Array.prototype.map
const delay = (a) => {
  return new Promise((resolve) => setTimeout(() => resolve(a), 500));
};

const f1 = () => {
  const list = [1, 2, 3, 4];
  const res = list.map((a) => delay(a * a));
  log(res);
};

f1();
// [Promise, Promise, Promise, Promise]
// 0: Promise {<fulfilled>: 1}
// 1: Promise {<fulfilled>: 4}
// 2: Promise {<fulfilled>: 9}
// 3: Promise {<fulfilled>: 16}
```

```javascript
// Array.prototype.map
const delay = (a) => {
  return new Promise((resolve) => setTimeout(() => resolve(a), 500));
};

const f1 = async () => {
  const list = [1, 2, 3, 4];
  const res = await list.map(async (a) => await delay(a * a));
  log(res);
};

f1();
// [Promise, Promise, Promise, Promise]
// 0: Promise {<fulfilled>: 1}
// 1: Promise {<fulfilled>: 4}
// 2: Promise {<fulfilled>: 9}
// 3: Promise {<fulfilled>: 16}
```

기존 Array.prototype.map 함수에서는 Promise를 어떤 방법을 사용해도 Promise가 return된다.<br>(Array.prototype.map에서는 Promise를 제어해주지 않는다.)

```javascript
// FxJS - map
const f2 = async () => {
  const list = [1, 2, 3, 4];
  const res = await map((a) => delay(a * a), list);
  log(res);
};

f2(); // [1, 4, 9, 16]
```

우리가 만든 map 함수는 Promise를 처리하도록 구현했기 때문에 값이 출력되는 것을 볼 수 있다.

```javascript
// Array.prototype.map
const f1 = async () => {
  const list = [1, 2, 3, 4];
  const temp = list.map(async (a) => await delay(a * a));
  log(temp); // [Promise, Promise, Promise, Promise] // 받을 준비 x
  const res = await temp;
  log(res); // [Promise, Promise, Promise, Promise] // 받을 준비 x
};

f1();
// FxJS - map
const f2 = async () => {
  const list = [1, 2, 3, 4];
  const temp = map((a) => delay(a * a), list);
  log(temp); // Promise {<pending>} - 받을 준비 완료
  const res = await temp;
  log(res); // [1, 4, 9, 16] - Promise의 결과를 출력
};

f2();
```

```javascript
// 사실, map 함수에서 이미 Promise를 처리할 수 있도록 구현했기 때문에 async await이 필요하지 않다.
const f2 = () => {
  return map((a) => delay(a * a), [1, 2, 3, 4]);
};

f2().then(log); // [1, 4, 9, 16]
```

# 파이프라인과 async/await의 비교

### async/await

- Promise.then.then.then과 같은 체이닝 방식이 아닌 문장으로 비동기를 다루기 위한 목적

### 파이프라인

- 명령형 프로그래밍을 피하면서 함수를 안전하게 합성하기 위함

즉, 파이프라인은 비동기와 아무런 관련이 없고, 파이프라인은 함수 합성, async/await은 함수를 풀어내기 위한 방법이다.

```javascript
// 파이프라인
const delay = (a) => {
  return new Promise((resolve) => setTimeout(() => resolve(a), 200));
};

const f3 = (list) => {
  return go(
    list,
    L.map((a) => delay(a * a)),
    L.filter((a) => delay(a % 2)),
    L.map((a) => delay(a + 1)),
    take(3),
    reduce((a, b) => delay(a + b)),
  );
};

go(f3([1, 2, 3, 4, 5, 6, 7, 8]), log); // 38

// async/await
const f4 = async (list) => {
  const temp = [];
  for (const a of list) {
    const b = await delay(a * a);
    if (await delay(b % 2)) {
      const c = await delay(b + 1);
      temp.push(c);
      if (temp.length === 3) break;
    }
  }
  let res = temp[0];
  let i = 0;
  while (++i < temp.length) {
    res = await delay(res + temp[i]);
  }
  return res;
};

go(f4([1, 2, 3, 4, 5, 6, 7, 8]), log); // 38
```

위와 같이, async/await은 비동기적인 상황을 동기적인 문장으로(명령형) 풀어서 설명하기 위한 목적이 있고, 파이프라인은 단지 명령형 프로그래밍 방식을 해결하기 위한 것이다.

delay 함수가 Promise를 return하지 않고 즉시 값이 return되는 함수라고 할 때, 파이프라인에서는 코드를 바꿀 필요가 없지만, async/await에서는 async/await을 모두 없애줘야 한다.
