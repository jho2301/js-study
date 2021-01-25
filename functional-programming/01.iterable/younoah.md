# Section0

### 평가

코드가 계산되어 값을 만드는 것

```javascript
1 + 2 // 3으로 평가된다.
[1,2 + 3] // [1, 5]로 평가된다.
```



### 일급

- 값으로 다룰 수 있다.
- 변수에 담을 수 있다.
- 함수의 인자로 사용될 수 있다.
- 함수의 결과로 사용될 수 있다.

```javascript
const a = 10; // 변수에 담을 수 있다.
const add = a => a + 10; // 함수의 인자로 사용될 수 있다.
const r = add10(a); // 함수의 결과로 사용될 수 있다.
```



### 일급함수

- 함수를 값으로 다룰 수 있다.
- 함수가 일급이 때문에 **조합성과 추상화의 도구**로 함수를 잘 사용할 수 있다.

```javascript
const add5 = a => a + 5; // 함수
log(add5); // 함수의 인자로 함수로 사용할 수 있다.
log(add5(5)); // 함수의 결과를 다른 함수에 전달할 수 있다.

const f1 = () => () => 1; // 함수의 결과를 함수로 할 수 있다.
log(f()); 

const f2 = f1(); // 다른 변수에 함수를 담을 수 있다.
log(f2);
log(f2());
```



### 고차함수

> 함수를 값으로 다루는 **함수**

#### 1. 함수를 인자로 받아서 실행하는 함수

```javascript
// ex1
const apply1 = f => f(1);
const add2 = a => a + 2;
log(apply(add2));
// apply1은 add2라는 함수를 인자로 받아서 실행한다.

// ex2
const times = (f, n) => {
    let i = -1;
    while (++i < n) f(i);
};
times(log, 3);
```



#### 2. 함수를 만들어 리턴하는 함수 (클로저를 만들어 리턴하는 함수)

```javascript
const addMaker = a => b => a + b;
const add10 = addMaker(10);
log(add10(10)); // 10 
```

`b => a + b` 가 클로저가 된다.



> **클로저란?**
>
> 외부함수와 외부함수 안에 내부함수가 있다고하자.
>
> 그리고 내부함수가 외부함수의 변수를 사용하고 있다고 하자.
>
> 내부함수가 외부함수의 변수를 기억하고 있는 상태 와 내부함수 자체를 함께 통칭해서 말하는 용어이다.
>
> 함수가 함수를 만들어서 리턴하면 결국 클로저를 만들어서 리턴한다.



# Section1



## ES6에서의 리스트 순회



기존방식 (es6이전) : `for i++`

```javascript
// length와 인덱스에 의존
const list = [1, 2, 3];
for (var i = 0; i < list.length; i++) {
    log(list[i]);
}

// 유사배열도 length와 인덱스에 의존
const str = 'abc';
for (var i = 0; i < str.length; i++) {
    log(str[i]);
}
```



ES6 : `for of`

```javascript
for (const a of list) {
    log(a);
}

for (const a of str) {
    log(a);
}
```

`for of` 문을 활용하여 리스트의 요소들을 하나씩 가지고 올 수 있다.

그렇다면 `for of` 순회는 어떻게 동작할까?



## Array, Set, Map을 통해서 순회 동작 알아보기

`for of` 문은 유사배열 타입(문자열, Set, Map)에 모두 사용이 가능하다.

- **Array**

```javascript
const arr = [1, 2, 3];
for (const a of arr) log(a);
```

배열은 인덱싱이 가능하다. 그렇다면 `for of` 문은 `length`와 인덱싱을 통해 동작하는 것일까? 우선 아래 더 확인해보자.

 

- **Set**

```javascript
const set = new Set([1, 2, 3]);
for (const a of set) log(a);
```

Set은 인덱싱이 불가하다. 



- **Map**

```javascript
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
for (const a of map) log(a);
```

Set도 인덱싱이 불가하다.



> 따라서 `for of` 문이 내부적으로 `length`와 인덱싱을 통해 동작하는게 아니라는 것을 알 수 있다.



그렇다면 어떻게 `for of` 문이 동작할까?



#### 그 전에 Symbol.iterator에 대해 알아보자.

심볼은 ES6부터 도입된 새로운 타입으로 객체의 키(인덱스)로 사용할 수 있다.

한번 배열에 `Symbol.iterator` 로 인덱싱 해보자.

```javascript
const arr = [1, 2, 3];
log(arr[Symbol.iterator]); // 어떤 함수가 출력된다.

arr[Symbol.iterator] = null; // 삭제해보기
for (const a of arr) log(a); // 에러 : arr가 iterable이 아니다.
```

arr뿐만 아니라 set, map에서도 똑같이 `Symbol.iterator` 가 존재한다.

이를 보아 `for of` 문과 `Symbol.iterator` 과 서로 연관이 있을것이다 라는것을 알 수 있다.



## 이터러블 / 이터레이터 프로토콜

배열, Set, Map은 자바스크립트의 내장객체로써 이터러블 / 이터레이터 프로토콜을 따른다.



#### 이터러블

> 이터레이터를 리턴한는 `[Symbol.iterator]()` 라는 메서드를 가진 값

배열, Set, Map 3개의 객체는 이터러블인데 그 이유는 `[Symbol.iterator]()` 라는 메서드를 가지고 있기 때문이다. `객체[Symbol.iterator]` 로 확인해보면 메서드를 가지고 있는것을 확인할 수 있다.

그리고 `객체[Symbol.iterator]()` 로 메서드를 실행하면 **이터레이터**를 리턴한다.

```javascript
const arr = [1, 2, 3];
arr[Symbol.iterator]; // ƒ values() { [native code] }, 메서드
arr[Symbol.iterator](); // Array Iterator {} <- 배열형 이터레이터 객체
```



#### 이터레이터

> `{ value, done}` 객체를 리턴하는 `next()` 라는 메서드를 가진 값

```javascript
const arr = [1, 2, 3];
let iterator = arr[Symbol.iterator]();

iterator.next(); // {value: 1, done: false}
iterator.next(); // {value: 2, done: false}
iterator.next(); // {value: 3, done: false}
iterator.next(); // {value: undefined, done: true}
```

이터러블의 메서드를 통해서 이터레이터를 리턴받을수 있고 이터레이터의 `next()` 메서드를 통해서 이터러블 객체들의 순회정보를 확인할 수 있다.



이때 `for of` 문은 이터레이터의 `next()` 를 호출하면서 `{value, done}` 객체를 받아오고 value의 값을 사용한다. 그러다 `done` 이 false이면 순회를 종료한다.

- 이터레이터를 통한 순회 테스트

```javascript
const arr = [1, 2, 3];
let iterator = arr[Symbol.iterator]();
iterator.next();
for (const a of iterator) log(a);
// 2
// 3
```

이터레이터를 통해서 순회가 가능하고 이터레이터를 일부 진행시키고 순회하면 진행된 곳부터 시작된다.



#### 이터러블 / 이터레이터 프로토콜

> 이터러블 객체를 `for of` 와 전개 연산자 등과 함께 동작하도록 규약 

정리하면 배열, Set, Map은 이터러블 객체이고 이터레이터를 리턴할 수 있도록 규약을 따르고 있기 때문에  `for of` 문과 여러가지 이터러블 연산자들을 사용할 수 있다.

이터레이터 프로토콜은 배열, Set, Map 뿐만아니라 순회가 가능한 형태는 이터러블 프로토콜을 따른다.

또한 다양한 오픈소스와 자바스크립트가 사용되는 환경인 브라우저의 웹API들, DOM 등 들에도 이터러블 프로토콜을 따르고 있다.

- DOM의 이터레이터

```javascript
const all = document.querySelectorAll('*');
let iterator = all[Symbol.iterator]();
log(iterator);
```





**이터러블 객체의 이터레이터를 반환하는 메서드**

추가로 map객체.keys()의 리턴값도 이터레이터 이고 이 이터레이터의 `next()` 를 찍어보면 맵의 키를 밸류로 가지고 있는 {value, done} 객체를 리턴한다.

따라서 아래 코드와 같이 key들을 순회할 수 있다.

```javascript
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
for (const a of map.keys()) log(a);
// a
// b
// c
```

`key()` 메서드와 비슷하게 `values()`, `entries()` 메서드들도 같은 방식으로 구현되어 동작된다.



## 사용자 정의 이터러블

> 새로운 이터러블 객체 만들어 보면서 이터러블에 대해 깊게 알아보자.



1. `[Symbol.iterator]()` 메서드 구현하기
	- value와 done 정의 
	- `return next()`
2. 이터레이터의 반환값을 자기 자신(이터레이터)로 설정하기

```javascript
// 사용자 정의 이터러블 //
const iterable = {
    [Symbol.iterator]() {
        let i = 3;
        return {
            next() {
                return i == 0 ? {done: true} : {value: i--, done: false};
            },
            [Symbol.iterator]() {
                return this;
            }
        }
    }
};

// 테스트 //
for (const a of iterable) log(a);
// 3
// 2
// 1

let iterator = iterable[Symbol.iterator]();
iterator.next();
for (const a of iterator) log(a);
// 2
// 1
```



> **이터레이터의 이터레이터**
>
> 이터레이터의 이터레이터를 자기 자신(이터레이터)으로 반환되도록 해야 잘 만들어진 이터러블이라고 할 수 있다.
>
> 이터러블은 이터레이터로도 순회가 가능해야하기 때문이고 이미 진행된 자기자신의 상태를 다시 순회할 수 있어야하기 때문이다.
>
> 즉, 이터러블의 이터레이터도 이터러블해야한다.



## 추가

### 전개 연산자도 이터러블 프로토콜을 따른다.

**전개연산자란?**

이터러블 객체의 요소를 전개해준다. `...이터러블객체` 키워드로 사용한다.

```javascript
const a = [1, 2, 3, 4];

log(...a);
// 1, 2, 3, 4
```



**전개연산자가 이터러블 프로토콜을 따르는지 확인하기**

```
const a = [1, 2, 3, 4];

a[Symbol.iterator] = null;
log(...a);
// 에러 : not iterable
```

이터러블 객체가 아니라면 전개연산자를 사용할 수 없다.