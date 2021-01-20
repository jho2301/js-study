## this와 this binding

this는 현재 실행되는 코드의 실행 컨텍스트를 가리킨다.

this binding은 this에 실행 컨텍스트의 주체를 연결 짓는것, 즉 this가 무엇을 가리킬지 연결하는것



this binding은 실행 컨텍스트가 활성화 될 때 한다.

실행 컨텍스트는 이 컨텍스트를 지닌 함수가 호출될 때 활성화 된다.

즉, this는 함수를 호출할 때 정해진다. 함수를 어떻게 호출했냐에 따라 this는 달라진다.



## this binding 5가지 케이스

- 전역공간에서 : window / global
- 함수 호출시 : windonw / global
- 메소드 호출시 : 메소드 호출 주체 (메소드명 앞)
- callback 호출시 : 기본적으로 함수 호출시와 동일
- 생성자 함수 호출시 : 인스턴스



### 1. 전역공간에서 : window / global

```javascript
console.log(this);
```

전역 공간에서 this는 브라우저 콘솔이면 window node.js면 global이 된다.



### 2. 함수 호출시 : windonw / global

> 함수는 전역객체의 메소드다 라고 생각하면 좀 더 받아 들이기 쉽다.
>
> ` window.함수명();`

```javascript
function a() {
  console.log(this);
}
a(); // window.a()
// 함수 a를 호출한 주체가 전역공간이기 때문에 this는 window가 된다.

function b() {
  function c() {
    console.log(this);
  }
  c();
}
b(); // window.b()
```

```javascript
var d = {
  e: function () {
    function f() {
      console.log(this); // this는 window
    }
    f(); // 이걸보면 this는 window라는것을 알수 있다. window.f()
  },
};
d.e(); 
// 메소드를 호출한것 처럼보여도 내부적으로는 함수호출하는 방식이기때문에 this는 window 
```





### 3. 메소드 호출시 : 메소드 호출 주체 (메소드명 앞)

> 메소드 호출시 메소드를 호출한 주체(메소드명앞)가 this가 된다.

```javascript
var a = {
  b: function () {
    console.log(this);
  },
};
a.b(); // this는 a

var a = {
  b: {
    c: function () {
      console.log(this);
    },
  },
};
a.b.c(); // this는 a.b
```



**내부 함수 우회법**

> 객체에서 선언된 내부 함수 안에서 this를 출력했을때 전역객체가 뜨는데 이를 회피하는 방법
>
> - this를 변수에 저장해서 사용하기
> - ES5에서는 bind 메서드로도 해결할 수 있다.
>
> - ES6 이후로 arrow function으로 해결할 수 있다.

this변수를 저장해서 사용해보자 아래와 같이 코드가 있다.

```javascript
var a = 10;
var obj = {
  a: 20,
  b: function () {
    console.log(this.a); // this는 obj

    function c() {
      console.log(this.a); // c(); 로 호출했기 때문에 this는 window
    }
    c(); // window.c();
  },
};
obj.b();
```



아래처럼 this를 self에 저장해서 self를 사용하면 된다.

```javascript
var a = 10;
var obj = {
  a: 20,
  b: function () {
    var self = this; // this는 obj, this를 변수에 저장
    console.log(this.a);

    function c() {
      console.log(self.a); // self는 obj
    }
    c();
  },
};
obj.b();
```





### 함수 호출 메서드 : call, apply, bind

- 기본적으로는 함수의 this와 같다. (window/ global)

- 제어권을 가진 함수가 callback의 this를 명시한 경우 그에 따른다. (call, apply)

- 개발자가 this를 바인딩한 채로 callback을 넘기면 그에 따른다. (bind)



**함수 호출 메서드 : call, apply, bind**

함수를 호출할때 this를 변경하고 인자를 전달하면서 호출을 돕는다.

- call

```javascript
함수명.call(thisArg, 인자...); -> 함수가 실행된다.
```

- apply

```javascript
함수명.apply(thisArg, [인자...]); -> 함수가 실행된다.
```

call과 apply차이점은 call은 인자를 그대로 넣지만 apply는 인자를 배열(유사배열)형태를 넣는다는 차이점만 존재하고 똑같이 동작한다.



- bind

```javascript
함수명.bind(thisArg, 인자...); -> 함수를 리턴한다.
```

this를 bining해서 함수를 **리턴**한다. 



call, apply, bind 메서드 모두 인자를 원하는 만큼 넣어줘도 되고 안넣어줘도 된다. 넣어준 인자만 받아서 사용된다.



- call, apply, bind 예제

```javascript
function a(x, y, z) {
  console.log(this, x, y, z);
}
var b = {
  c: 'eee',
};

a.call(b, 1, 2, 3);
a.apply(b, [1, 2, 3]);

var c = a.bind(b);
c(1, 2, 3);

var d = a.bind(b, 1, 2);
d(3);
```



## 4.callback 호출시 : 기본적으로 함수 호출시와 동일



#### 예제1

- 일반

```javascript
var callback = function () {
  console.log(this); // 함수로써 호출되었기 때문에 this는 window
};
var obj = {
  a: 1,
  b: function (cb) {
    cb(); // 함수로써 호출 window.cb();
  },
};
obj.b(callback);
```

- call 사용

```javascript
var callback = function () {
  console.log(this); // this는 obj
};
var obj = {
  a: 1,
  b: function (cb) {
    cb.call(this); // this는 obj, cb()함수의 this를 obj로 지정해서 호출
  },
};
obj.b(callback);
```



#### 예제2

```javascript
var callback = function () {
  console.log(this);
};
var obj = {
  a: 1,
};
setTimeout(callback, 1000); // this는 window
setTimeout(callback.call(obj), 1000); // call로 하면 1초기다리지 않고 즉시 호출된다.
setTimeout(callback.bind(obj), 1000); // bind로 해야 함수가 생서되서 전달되고 1초뒤에 호출된다.
```



#### 예제3

```javascript
document.body.innerHTML += '<div id="a">클릭하세요</div>';
document.getElementById('a').addEventListener('click', function () {
  console.log(this); // this는 <div>
});
```

콜백함수여서 this가 window일 것같지만

`addEventListener()`함수가 콜백함수를 처리할 때 이벤트가 발생한 타겟으로 this를 바인딩 한다.



### 5. 생성자 함수 호출시 : 인스턴스

생성자함수를 사용하면 this는 인스턴스가 된다.

```javascript
function Person(n, a) {
  this.name = n;
  this.age = a;
}
var gomugom = new Person('고무곰', 30);
console.log(gomugome);
```







## 궁금증

메타표기법?

```javascript
func.call(this[, arg1[, arg2[, ...]]]);
```



생성자 함수? 객체 생성할때 쓰는거 아니었나..??? ㄷㄷ

```javascript
function Person(n, a) {
    this.name = n;
    this.age = a;
}
var gomugom = new Person('고무곰', 30);
```

