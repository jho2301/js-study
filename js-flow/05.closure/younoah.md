## 클로저

> 클로저 정의 :  함수 X 그 함수가 선언될 당시의 LE
>
> - 실행 컨텍스트 A의 내부에 함수B를 선언한 상황에서 
>
> - 실행 컨텍스트 A와 내부 함수 B가 콤비가 되어 무언가를 한다.

<br>

### 무엇을 하는데?

**컨텍스트 A**에서 선언한 **변수**를 **내부함수 B**에서 **접근**할 경우에만 발생하는 특수한 현상

(B의 outerEnvironmentReference는 A의 environmentRecord를 참조)

<br>

### 특수한 현상이 뭔데?

**컨텍스트 A**에서 선언한 **변수 a**를 참조하는 **내부함수 B**를 **A의 외부로 전달**할 경우,

A가 종료된 이후에도 A의 **변수 a가 사라지지 않는 현상**

- 쉽게 말해, **내부함수 B** 를 외부에서 할당하면  **컨텍스트 A 의 지역변수**가 사리지지 않는 현상!

<br>

❗️이런 특수한 현상을 이용해서 

**함수가 종료된 이후에도 사라지지 않는 지역변수를 만들 수 있다.**

<br>

- 일반적인 상황

```javascript
var outer = function() {
    var a = 1;
    var inner = function() {
        console.log(++a);
    };
    inner();
}

outer();
```

<br>

- `outer2` 클로저로 사용

```javascript
var outer = function() {
    var a = 1;
    var inner = function() {
        return ++a;
    };
    inner inner;
}

var outer2 = outer();
```



<br>

### 클로저 활용 예시

> 하나의 함수의 지역변수로 서로다른 클로저를 사용

```javascript
function showMike(age) {
  var name = 'mike';
  return {
    get name() {
      return name;
    },
    set name(_name) {
      throw Error('read only');
    },
    get age() {
      return age;
    },
    set age(_age) {
      age += _age;
    },
  };
}
var dctorMike = showMike(42);
var policeMike = showMike(31);

// dctorMike
console.log(dctorMike.name);
console.log(dctorMike.age);

// policeMike
console.log(policeMike.name);
console.log(policeMike.age);
```



