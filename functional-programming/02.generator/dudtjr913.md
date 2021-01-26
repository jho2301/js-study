# 제너레이터/이터레이터
- 제너레이터: `이터레이터이자 이터러블을 생성하는 함수`

```javascript
function* generator() {
  yield 1;
  yield 2;
  yield 3;
  return 100;
}

const iterator = generator();
log(iterator[Symbol.iterator]); // ƒ [Symbol.iterator]() { [native code] }
log(iterator[Symbol.iterator]() === iterator); // true => well-formed iterator
log(iterator.next()); // {value: 1, done: false}
log(iterator.next()); // {value: 2, done: false}
log(iterator.next()); // {value: 3, done: false}
log(iterator.next()); // {value: 100, done: true}

for (const value of generator()) log(value); // 1 2 3
```

코드를 보면 알 수 있듯이, 제너레이터 함수는 `순회할 값들을 문장으로 표현`하는 것이다.<br>
그래서 
```javascript
function* generator() {
  yield 1;
  if (false) yield 2;
  yield 3;
  return 100;
}

for (const value of generator()) log(value); // 1 3
```
위의 결과를 볼 수 있다.<br>
즉, 제너레이터 함수는 `조건을 만족`하는 문장에서만 값이 출력된다.

이는 자바스크립트에서 중요한 의미를 가진다. <br>
자바스크립트는 이터러블이면 모두 순회할 수 있는데, 제너레이터 함수는 문장을 순회할 수 있는 이터러블 값으로 만들어주기 때문에 <br>
**모든 값을 순회할 수 있도록 만들어준다.**

# odds
제너레이터 함수를 사용해 홀수만 출력하도록 구현해 보자.
```javascript
function* odds(startNum, finishNum) {
  for (let num = startNum; num < finishNum; num++) {
    if (num % 2) yield num;
  }
}

const oddsIterator = odds(3, 8);
log(oddsIterator.next()); // {value: 3, done: false}
log(oddsIterator.next()); // {value: 5, done: false}
log(oddsIterator.next()); // {value: 7, done: false}
log(oddsIterator.next()); // {value: undefined, done: true}
```
시작과 끝 숫자를 받아 홀수만 출력하도록 함수를 구현했다. <br>
어려운 내용은 없고, 반복문이 실행되다가 yield를 만나면 실행을 멈춘다는 것만 알아두자.<br>
조금 더 들어가보자.

```javascript
function* infinity(startNum = 0) {
  while (true) yield startNum++;
}

const infinityIterator = infinity(3);
log(infinityIterator.next()); // {value: 3, done: false}
log(infinityIterator.next()); // {value: 4, done: false}
log(infinityIterator.next()); // {value: 5, done: false}
log(infinityIterator.next()); // {value: 6, done: false}
```
제너레이터 함수이기 때문에 무한히 반복되는 while(true)도 순회할 수 있게 되었다.<br>
이를 이용해보자.

```javascript
function* infinity(startNum = 0) {
  while (true) yield startNum++;
}

function* odds(startNum, finishNum) {
  for (const num of infinity(startNum)) {
    if (num % 2) yield num;
    if (num === finishNum) return;
  }
}

const oddsIterator = odds(3, 8);
log(oddsIterator.next()); // {value: 3, done: false}
log(oddsIterator.next()); // {value: 5, done: false}
log(oddsIterator.next()); // {value: 7, done: false}
log(oddsIterator.next()); // {value: undefined, done: true}
```
무한히 반복되는 infinity 제너레이터 함수를 for of문을 통해 반복되도록 하고, 홀수일 때만 yield를 해주어 값을 받아오고 있다.

다음은 마지막 숫자가 정해진 제한된 제너레이터 함수를 만들어보자.
```javascript
function* limit(iterator, finishNum) {
  for (const num of iterator) {
    yield num;
    if (finishNum === num) return;
  }
}

const limitIterator = limit(infinity(3), 5);
log(limitIterator.next()); // {value: 3, done: false}
log(limitIterator.next()); // {value: 4, done: false}
log(limitIterator.next()); // {value: 5, done: false}
log(limitIterator.next()); // {value: undefined, done: true}
```
이번에는 제너레이터 함수 안에서 이터레이터를 받아 처리하도록 만들었다.

지금까지 한 모든 것들을 조합해서 odds 제너레이터 함수를 만들어보면
```javascript
function* infinity(startNum = 0) {
  while (true) yield startNum++;
}

function* limit(iterator, finishNum) {
  for (const num of iterator) {
    yield num;
    if (finishNum === num) return;
  }
}

function* odds(startNum, finishNum) {
  for (const num of limit(infinity(startNum))) {
    if (num % 2) yield num;
    if (num === finishNum) return;
  }
}

const oddsIterator = odds(3, 8);
log(oddsIterator.next()); // {value: 3, done: false}
log(oddsIterator.next()); // {value: 5, done: false}
log(oddsIterator.next()); // {value: 7, done: false}
log(oddsIterator.next()); // {value: undefined, done: true}
```
이처럼 제너레이터 함수의 인수로 제너레이터 함수가 들어갈 수도 있다.<br>
이를 잘 활용해서 사용하는 것이 중요할 것이다.

강의에서 다양하게 연습해보라고 해서 겹치지 않는 count만큼의 숫자를 가지는 함수를 만들어봤다.
```javascript
// 원하는 배열에서 count만큼의 숫자를 가져오는 함수
const getAnswerNumber = (numberArray, count) => {
  for (const answerNumber of createAnswerNumber(numberArray, count)) {
    if (answerNumber.length === count) {
      return answerNumber;
    }
  }
};

// 받은 인수가 정확한지 판단하는 함수
const isRightArg = (numberArray, count) => {
  if (new Set(numberArray).size !== numberArray.length) {
    return console.error('array에 중복되는 수가 있습니다.');
  }
  if (numberArray.length < count) {
    return console.error('count가 array의 길이보다 작아야 합니다.');
  }

  return true;
};

// 겹치지 않는 숫자를 얻을 수 있도록 구현된 제너레이터 함수
function* createAnswerNumber(numberArray, count) {
  if (!isRightArg(numberArray, count)) return;

  const answer = [];

  while (answer.length !== count) {
    const randomIndex = Math.floor(Math.random() * numberArray.length);
    const randomNumber = numberArray[randomIndex];

    if (!answer.includes(randomNumber)) {
      answer.push(randomNumber);
      yield randomNumber;
    }
  }

  yield answer;
}

const answer = getAnswerNumber([1, 2, 3, 4, 5, 6, 7, 8, 9], 5);
log(answer); // [3, 2, 4, 5, 9]와 같이 겹치지 않는 count만큼의 수
```
제너레이터 함수를 사용하지 않았을 땐 numberArray에서 answerArray로 push된 것을 splice로 삭제해주는 과정을 거쳤었는데,<br>
제너레이터 함수를 사용하면 겹치는 수가 있을 땐 그냥 넘어가도록 해서 배열을 변경하지 않고도 값을 얻을 수 있게 되었다.

운이 좋지 않으면 반복 횟수가 굉장히 많아지기 때문에 위와 같은 경우에는 성능에서는<br>
splice로 배열을 지우는 것이 더 나을 것 같다는 생각이 들었다. 정확하게는 모르겠다...

# for of, 전개 연산자, 구조 분해, Rest 파라미터
제너레이터는 이터러블/이터레이터 프로토콜을 따르고 있어서 for of, 전개 연산자, 구조 분해, Rest 파라미터와 같이 <br>
이터러블/이터레이터 프로토콜을 따르는 문법이나 라이브러리, 헬퍼 함수들과 함께 사용될 수 있다.

```javascript
log(...odds(3, 8)); // 3 5 7
log([...odds(3, 8), ...odds(9, 20)]); // [3, 5, 7, 9, 11, 13, 15, 17, 19]

const [head, ...tail] = odds(3, 12);
log(head); // 3
log(tail); // [5, 7, 9, 11]

const [first, second, ...rest] = odds(3, 12);
log(first); // 3
log(second); // 5
log(rest); // [7, 9, 11]

const func = (...rest) => {
  return rest
};

log(func(...odds(3, 8))); // [3, 5, 7]
```
