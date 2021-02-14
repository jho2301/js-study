## async/await와 파이프라인



```javascript
async function f() {
  const r1 = await go(
    list,
    L.map(a => delay(a * a)),
    L.map(a => delay(a % 2)),
    L.map(a => delay(a + 1)),
    C.take(2),
    reduce((a, b) => delay(a + b))
  );

  const r2 = await go(
    list,
    L.map(a => delay(a * a)),
    L.filter(a => delay(a % 2)),
    reduce((a, b) => delay(a + b))
  );
  const r3 = delay(r1 + r2);
  return r3 + 10;
}

go(f([1, 2, 3, 4, 5, 6, 7, 8]), a => console.log(a, 'f함수'));
```



파이프라인과 async/await을 활용하면 2가지의 파이프라인을 가지고 그 결과들을 한번에 처리해야 하는 로직일 때 유용하게 사용할 수 있다.



## 동기 상황에서 에러핸들링



#### 에러케이스

- 들어오는 인자값이 잘못된 경우
- 함수 안에서 값이 잘못 평가된 경우



#### 에러 처리

- 최대한 비슷한 값을 리턴
- 에러 출력



```javascript
// 예시
function f(list) {
  return list
    .map(a => a + 10) 
    .filter(a => a % 2)
    .slice(0, 2);
}
console.log(f(null)) // 인자값이 잘못된 경우

// try-catch로 에러시 최대한 비슷한 값인 빈 배열 리턴
function f(list) {
  try {
    return list
      .map(a => adsfa) //함수내 에러, 하나의 콜스택에서 발생이므로 에러처리가 된다.
      .filter(a => a % 2)
      .slice(0, 2);
  } catch (e) {
    console.log(e)
    return [];
  }
}
```



## 비동기 상황에서 에러핸들링



```javascript
async function f(list) {
  try {
    return await list
      .map(
        a =>
          new Promise(resolve => {
            resolve(JSON.parse(a)); // 문제발생!
          })
      )
      .filter(a => a % 2)
      .slice(0, 2);
  } catch (e) {
    console.log(e, 'error catch');
    return [];
  }
}

f(['0', '1', '2', '{']).then(console.log).catch(e => console.log(e));

```

비동기가 발생하는 부분에서 프로미스를 잘 처리 해주지 않기 때문에 아무리 async/await을 걸어서 동기적으로 만든 다음에, catch를 해도 에러핸들링이 안된다.

따라서  FxJS 파이프라인 함수들을 이용하면 에러핸들링이 간편해진다.



```javascript
function f(list) {
  try {
    return go(
      list,
      map(
        a =>
          new Promise(resolve => {
            resolve(JSON.parse(a)); // 문제발생!
          })
      ),
      filter(a => a % 2),
      take(2)
    );
  } catch (e) {
    console.log(e, 'error catch');
    return [];
  }
}

console
  .log(f(['0', '1', '2', '{']))
  .then(a => log(a, '함수'))
  .catch(e => console.log('프로미스캐치', e));

```



await에 프로미스를 잘 걸어줘야하고, 프로미스를 await에 잘 걸어줄려면 합성된 함수 안에서 에러가 발생해도 Kelisli방식으로 안전하게 합성되서 프로미스로 준비가 되어있어야한다. ()

파이프라인을 이용하면 에러가 발생해도 안전하게 합성되서 프로미스가 준비되기 때문에 에러 핸들링이 간편해진다.
