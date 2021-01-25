# 평가
- 코드가 계산되어 **값**을 만드는 것

# 일급
- 값으로 다룰 수 있다.
- 변수에 담을 수 있다.
- 함수의 인자로 사용될 수 있다.
- 함수의 결과로 사용될 수 있다.

- --> 값이 아니면 그냥 문장이다. **값이 있어야만** 일급이다.<br>
ex) for 문은 문장, map 함수는 일급

# 일급 함수
- 함수를 값으로 다룰 수 있다.
- 조합성과 추상화의 도구

# 고차 함수
- 함수를 값으로 다루는 함수

#### 함수를 인자로 받아서 실행하는 함수 (인자 = 콜백 함수)
```
const apply3 = f => f(3);
const add10 = a => a + 10;
console.log(apply3(add10)); // 13

const times = (f, n) => {
  let i = 0;
  while (i++ < n) {
    f(i);
  }
};
times((a) => console.log(a + 10), 3); // 11 12 13
```
#### 함수를 만들어 리턴하는 함수 (클로저를 만들어 리턴하는 함수)
```
const addMaker = a => b => a+b
const add1 = addMaker(1) // addMaker의 a에 1이 할당된 상태이고 이것이 클로저이다.
console.log(add1(2)) // 3
```

## ES6에서의 리스트 순회
- for of
```
const list = [1,2,3];
for (const value of list) {
  console.log(value);
}
// 1 2 3
```

---
### Array 순회
```
const arr = [1, 2, 3];
for (const value of arr) {
  console.log(value); // 1 2 3
}
```
### Set 순회
```
const set = new Set([1, 2, 3]);
for (const value of set) {
  console.log(value); // 1 2 3
}
```
### Map 순회
```
const map = new Map([
  ['a', 1],
  ['b', 2],
  ['c', 3],
]);
for (const value of map) {
  console.log(value); // ["a", 1] ["b", 2] ["c", 3]
}
```
Array 순회를 보면 arr[0]과 같이 순회를 할 것 같다고 생각이 되지만 Set이나 Map을 보면<br>
그렇지 않다는 것을 알 수 있다. 왜냐하면 set[0]이나 map[0] 해보면 undefined가 출력되기 때문이다.

## for of문은 기존의 방식과 다른 방식으로 동작한다!

> ### Symbol 간단 요약
> Symbol은 유일한 식별자를 만들고 싶을 때 사용한다.<br>
> const mySymbol = Symbol('me')가 있을 때 mySymbol에 'me'라는 심볼 이름을 가진 심볼 값이 할당된다.<br>
> 이때, 심볼 이름이 같다고 같은 심볼인 것은 아니다.<br>
> 같은 심볼을 만들기 위해서는 전역 심볼을 사용해야 하는데, Symbol.for('심볼 이름')로 사용할 수 있다.<br>
> 전역 심볼을 찾기 위해서는 Symbol.keyFor(변수 명)을 사용하자. 이때, 전역 심볼이 아니라면 undefined를 반환한다.<br>

### Symbol.iterator
위의 코드를 Symbol.iterator로 확인해보면 다음과 같은 결과를 볼 수 있다.
```
console.log(arr[Symbol.iterator]); // ƒ values() { [native code] }
console.log(set[Symbol.iterator]); //ƒ values() { [native code] }
console.log(map[Symbol.iterator]); ƒ entries() { [native code] }
```
이를 통해 for of문이 Symbol.iterator을 통해 순회하지 않을까? 라는 생각을 해볼 수 있다.

### 이터러블, 이터레이터 프로토콜
- 이터러블: 이터레이터를 리턴하는 Symbol.iterator()를 가진 값(Symbol.iterator 메소드를 가지고 있는 객체)
- 이터레이터 : { value, done } 객체를 리턴하는 next()를 가진 값(Symbol.iterator 메소드를 실행한 값)
- 이터러블/이터레이터 프로토콜: 이터러블을 for...of, 전개 연산자 등과 함께 동작하도록 한 규약
```
const arrIterator = arr[Symbol.iterator]();
arrIterator.next(); // {value: 1, done: false}
arrIterator.next(); // {value: 2, done: false}
arrIterator.next(); // {value: 3, done: false}
arrIterator.next(); // {value: undefined, done: true}

const setIterator = set[Symbol.iterator]();
setIterator.next(); // {value: 1, done: false}
setIterator.next(); // {value: 2, done: false}
setIterator.next(); // {value: 3, done: false}
setIterator.next(); // {value: undefined, done: true}

const mapIterator = map[Symbol.iterator]();
mapIterator.next(); // {value: ["a", 1], done: false}
mapIterator.next(); // {value: ["b", 2], done: false}
mapIterator.next(); // {value: ["c", 3], done: false}
mapIterator.next(); // {value: undefined, done: true}
```

이를 통해 for of문은 이터레이터의 value를 출력하다가 done: true가 되었을 때 멈춘다는 것을 알 수 있다.<br>
여기에서 map의 경우에 key만 얻어오고 싶다면 keys() 메소드를 사용할 수 있다. 또한, value만 얻어오고 싶다면 <br>
values() 메소드를 사용할 수 있다.
```
const mapKeyIterator = map.keys();
mapKeyIterator.next(); // {value: "a", done: false}
mapKeyIterator.next(); // {value: "b", done: false}
mapKeyIterator.next(); // {value: "c", done: false}
mapKeyIterator.next(); // {value: undefined, done: true}

for (const value of map.values()) {
  console.log(value);
} // 1 2 3
```

### 사용자 정의 이터러블/이터레이터
우선, 이터러블을 만들어보자.
```
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i === 0 ? {done: true} : {value: i--, done: false};
      },
    };
  },
};
```
iterable은 위에서 설명했듯이 Symbol.iterator() 메소드를 가지고 있는 객체이다.<br>
따라서 Symbol.iterator() 메소드를 실행하면 next() 메소드를 사용할 수 있게 된다.<br>

```
const iterator = iterable[Symbol.iterator]();
iterator.next(); // {value: 3, done: false}
iterator.next(); // {value: 2, done: false}
iterator.next(); // {value: 1, done: false}
iterator.next(); // {done: true}
```
또한 for of문으로도 순회 가능하다.
```
for (const value of iterable) {
  console.log(value);
} // 3 2 1
```
하지만 iterator을 순회하면 오류가 발생한다.
```
for (const value of iterator) {
  console.log(value);
  // Uncaught TypeError: iterator is not iterable
       at <anonymous>:1:20
}
```
왜 이런 오류가 발생할까?
for of문은 Symbol.iterator()의 next 메소드를 실행하여 그 결과인 value를 return하는 것인데<br>
작성한 코드대로 보면 iterator은 next 메소드만 가지고 있다.<br>
따라서 이터러블이 아니라고 판단해 오류가 발생하는 것이다.

그래서 필요한 것이 항상 자기 자신을 return해주어 이터레이터로 만들어주는 것이다.
```
const iterable = {
  [Symbol.iterator]() {
    let i = 3;
    return {
      next() {
        return i === 0 ? {done: true} : {value: i--, done: false};
      },
      [Symbol.iterator]() {
        return this;
      },
    };
  },
};

const iterator = iterable[Symbol.iterator]();

for (const value of iterator) {
  console.log(value);
} // 3 2 1
```
이제 정상적으로 작동한다.<br>
이렇게 이터레이터가 자기 자신을 return할 때 **well-formed iterable**이라고 한다.<br>

### 전개 연산자
전개 연산자도 이터러블이 동일하게 적용된다.
```
const arr = [1, 2, 3];
console.log([...arr, ...[4, 5]]) // [1, 2, 3, 4, 5]
```
여기에서도 역시 Symbol.iterator를 없애주면 작동하지 않는다.
```
const arr = [1, 2, 3];
arr[Symbol.iterator] = null;
console.log([...arr, ...[4, 5]]) // arr is not iterable
```
즉, 전개 연산자도 Symbol.iterator() 메소드를 실행해 next() 메소드의 값을 순서대로 return해주는 것을 알 수 있다.
