## 요약

### 콜백함수 특징

다른 함수(A)의 인자로 콜백 함수(B)를 전달하면, A가 B의 **제어권**을 갖게 된다.

(특별한 요청(bind)이 없는 한) A에 **미리 정해놓은 방식**에 따라 B를 호출한다.

미리 정해놓은 방식이란, 어떤 **시점**에 콜백을 호출할지, **인자**에는 어떤 값들을 지정할지, **this**에 무엇을 바인딩 할지 등이다.



> **주의!**
>
> 콜백은 함수이다.
>
> 메소드로 넘겨줘도 함수의 정의만 넘어가게 된다. 메소드처럼 사용이되는것이 아니다. this 바인딩이 메소드의 호출자가 아닌 콜백함수를 사용하는 함수에게 전적으로 달렸다.



### 궁금증

- 강의에서 `arr.forEach(obj.logValues)` 를 예시롤 들었을때 this가 window로 바인딩이 되었는데 이것은 forEach의 정의 때문에 그런것일까? 아니면 다른 이유가 있는것일까? 처음에 this가 arr가 될 줄 알았는데....



- 그리고 문득 든 생각인데 강의에서 왜이렇게 var를 쓸까... var는 문제점이 많아서 사용하지말고 let, const를 사용해야된다고 배웠는데..



## 예제

### 제어권 - 실행시점

```javascript
var cb = function() {
    console.log('1초마다 실행 된다.');
};

setInterval(cb, 1000);
```

setInterval함수가 콜백함수를 1초마다 실행하면서 언제 실행할지를 제어한다.



### 제어권 - 인자

```javascript
var arr = [1, 2, 3, 4, 5];
var entries = [];
arr.forEach(function(v, i) {
    entriese.push([i, v, this[i]]);
}, [10, 20, 30, 40, 50]);

console.log(entries)
```

forEach함수에 첫번째 인자로 콜백함수, 두번째 인자로 thisArg를 받는다.

콜백함수는 첫번째 인자로 arr의 값, 두번째 인자로 arr의 인덱스를 받는것이 정해저 있다.



### 제어권 - this

```javascript
document.body.innerHTML = '<div id="a">abc</div>';

function cbFunc(x) {
    console.log(this, x);
}

document.getElementById('a').addEventListner('click', cbFunc);

$('#a').on('click', cbFunc);
```

addEventListner함수는 콜백함수의 this를 addEventListner를 호출한 요소로 정의한다.

또한 콜백함수의 인자를 event요소로 전달하도록 정의되어있다.



### 주의 예제

콜백함수는 함수이다.

```javascript
var arr = [1, 2, 3, 4, 5];
var obj = {
    vals: [1, 2, 3],
    logValues: function(v, i) {
        if(this.vals) {
            console.log(this.vals, v, i);
        } else {
            console.log(this, v, i);
        }
    }
};

obj.logValues(1, 2); // this는 obj가 된다.
arr.forEach(obj.logValues); // this는 window가 된다.
```

