## 제너레이터와 이터레이터

### 제너레이터

> -  이터레이터이자 이터러블을 생성하는 함수
>
> - 즉, 제너레이터는 이터레이터를 리턴하는 함수이다.
>
> - 제너레이터는 순회할 값을 문장으로 표현한는 것이다.

제너레이터는 함수명앞에 `*` 키워드를 사용해서 선언한다.

제너레이터의 `yield` 를 통해서 이터레이터의 `next()` 메서드 리턴값인 `{ value, done }` 객체를 초기화할 수 있다. (값을 설정할 수 있다.)

또한 리턴값을 지정할 수 있지만 순회에 참조되지는 않는다.

```javascript
function *generator() {
    yield 1;
    if (false) yield 2;
    yield 3;
    return 100; // 이터레이터.next() 마지막 리턴값 {value: 100, done: true}
}

let iterator = generator(); // 제너레이터의 실행결과는 이터레이터이다.
log(iterator[Symbol.iterator] == iterator); 
// 이터레이터는 이터레이터이자 이터러블이다. 
// 이터레이터의 이터레이터는 자기자신이다.

log(iterator.next()); // {value: 1, done: false}
log(iterator.next()); // {value: 3, done: false}
log(iterator.next()); // {value: 100, done: true}, 순회에 참조 되지는 않는다.
log(iterator.next()); // {value: undefined, done: true}

for (const a of gen()) log(a); // 제너레이터의 결과가 이터레이터이기 때문에 순회가 가능하다.
// 1
// 3
```

제너레이터는 문장을 순회할 수 있는 값으로 만들수 있다. 

자바스크립트는 이런 제너레이터를 통해서 어떠한 상태나 어떠한 값이든 순회할 수 있는 형태인 이터러블로 만들수 있다.



## 제너레이터 활용

제너레이터에서 문장과 로직을 통해 값을 발생시키는 것을 제어할 수 있다.



- 홀수값을 발생시키는 제너레이터

```javascript
function *odds(max){
    for (let i = 0; i <= max;, i++) {
        if (i % 2) yield i;
    }
}

let iter = odds(5);
log(iter.next()); // {value: 1, done: false}
log(iter.next()); // {value: 3, done: false}
log(iter.next()); // {value: 5, done: false}
log(iter.next()); // {value: undefined, done: true}

```



- 호출할 때 마다 값을 1씩올려서 무한히 값을 만들어주는 제너레이터

```javascript
function *inifinity() {
    while (true) yield i++;
}

let iter = inifinity();
log(iter.next()); // {value: 1, done: false}
log(iter.next()); // {value: 2, done: false}
log(iter.next()); // {value: 3, done: false}
.
.
.
```

무한히 값을 생성하지만 이터레이터의 `next()`를 평가할 때만 동작하기 때문에 브라우저나 프로그램이 멈추지는 않는다.



- 복합 구현

```javascript
function* inifinity(i = 0) {
  while (true) yield i++;
}

function* limit(l, iter) {
  for (const a of iter) {
    yield a;
    if (a == l) return;
  }
}

function* odds(l) {
  for (const a of limit(l, inifinity(1))) {
    if (a % 2) yield a;
  }
}

for (const a of odds(40)) console.log(a);
```

다양한 로직의 문장으로 순회할 수 있는 값을 만들 수 있다.



## 제너레이터와 전개연산자, 구조 분해, 나머지 연산자

제너레이터는 이터러블 프로토콜을 따른다. 따라서 이터러블 프로토콜을 따르는 문법, 라이브러리, 함수 등과 함께 활용할 수 있다.



- 전개연산자

```javascript
log(...odds(10));
// 1 3 5 7 9
log([...odds(10), ...odds(20)]);
// [1, 3, 5, 7, 9, 1, 3, 5, 7, 9, 11, 13, 15, 17, 19]
```



- 구조분해

```javascript
const [head, ...tail] = odds(5);
log(head); // 1
log(tail); // [3, 5]
```



- 나머지 연산자

```javascript
const [a, b, ...rest] = odds(10);
log(a); // 1
log(b); // 3
log(rest); // [5, 7, 9]
```

