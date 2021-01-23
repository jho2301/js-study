### 사전지식

ES6이전에 자바스크립트에는 클래스라는 개념이 없었다. 그런데 자바스크립트도 객체지향 언어가 가능했던 이유는 프로토타입이 있었기 때문이다. 

자바스크립트는 정확하게 프로토타입 기반 언어이다.

클래스가 없으니 기본적으로 상속기능이 없었다. 그래서 프로토타입 기반으로 상속을 흉내내도록 수형해 사용한다.

(참고로 ES6부터 Class문법이 추가되었지만 문법이 추가된거지 자바스크립트가 클래스 기반으로 바뀌었다는 것은 아니다.)



- 자바스크립트의 모든 것(모든 타입)은 객체(Object)를 상속한다.(프로토타입으로 상속한다?) 따라서 모두 객체이다. 혹은 객체처럼 사용이 가능하다.
- 모든 객체는 prototype 속성 객체를 가지고 있다.

- 생성자의 prototype에 객체를 생성하면 모든 인스턴스가 공동으로 사용 가능하다.

- 프로토타입에는 인스턴스에서 공통으로 사용할 메소드와 변수가 정의 되어있다.

- +자바스크립트의 타입은 생성자 함수로 정의되어있다. ex Array Number 등등 (맞나?)



### 프로토타입

![2021-01-23_19-22-47](/Users/uno/Desktop/2021-01-23_19-22-47.png)

- 생성자 함수(Constructor)에는 프토로타입(Prototype)이라는 객체가 존재한다.

- new 키워드와 생성자함수(Constructor)를 이용해서 인스턴스를 생성하면

- 인스턴스에도 `__proto__` 라는 객체가 생성이 되는데  `__proto__` 는 생성자함수의 프로토타입을 참조한다.

- 이 때 `__proto__` 는 생략이 가능해서 인스턴스로 바로 생성자함수의 프로토타입을 참조할 수 있다.



아래 4가지는 모두 프로토타입을 가리킨다.

```javascript
CONSTRUCTOR.prototype
INSTANCE.__proto__
INSTANCE
Object.getPrototypeOf(INSTANCE)
```



또한 프로토타입으로 생성자에 접근할 수 있다.

```javascript
CONSTRUCTOR
CONSTRUCTOR.prototype.constructor
INSTANCE.__proto__.constructor
INSTANCE.constructor
(Object.getPrototypeOf(INSTANCE)).constructor
```



- `INSTANCE.변수` 라고 사용했을 때 처음에 생성자에 해당 변수가 있는지 찾아본다.

- 만약 없으면 생성자의 프로토타입에 가서 해당 변수를 찾아서 사용한다.

- 만약 생성자와 프로토타입에 각각 같은 변수가 선언되어있다면

- `INSTANCE.변수` 와 `INSTANCE.__proto__.변수` 의 값은 다르다.
- `INSTANCE.변수` 는 생성자에 정의 되어있는 변수를 우선적으로 사용한다.



### 프로토타입 체이닝

![2021-01-23_19-43-18](/Users/uno/Desktop/2021-01-23_19-43-18.png)



어떤 타입의 생성자든 프로토타입이 존재하는데 프로토타입은 객체이다.

따라서 프로토타입은 다시 객체의 인스턴스이고 객체의 프로토타입과 체이닝 되어있다. (`__proto__` 생략 가능하기 때문에)

그래서 어느 타입이든 객체의 프로토타입에 정의되어있는 메서드를 사용할 수 있다.



이때 객체의 프로토타입에 객체에서만 사용하는 메소드까지 정의하면 굳이 다른 타입들까지 사용할 수 있게 되는 꼴이라 객체에서만 사용할 수 있는 함수들은 객체 생성자에 정의가 되어있다. 그래서 객체 생성자 정의를 보면 정의가 복잡하다.





```javascript
var arr = [1, 2, 3];
arr.toString = function() {
    return this.join('_');
}

console.log(arr.toString()); // "1_2_3", 생서자의 정의에 의해
console.log(arr._proto_.toString()); // "1, 2, 3", 프로토타입의 정의에 의해
console.log(arr._proto_._proto_.toString()); //[object Array], 객체의 정의에 의해
```





