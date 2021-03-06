**어제의 궁금증 해결**

생성자 함수 내부에 정의된 메서드는 static 메서드이다.

따라서 인스턴스로 접근할 수 없고 생성자 이름으로만 접근할 수 있다.



반면에 클래스 내부에 일반적으로 선언된 메서드는 인스턴스가 사용할 수 있는 일반적이 메서드이고

static 메서드를 사용하려면 static 키워드를 사용하여야 한다.



## 메소드 vs static 메소드 vs 프로토타입 메소드 차이

❗️흔히 알고 있는 메소드가 프로토타입 메소드이다.

### (in class)

**메소드** : 클래스 내부에 일반적으로 함수를 선언하는 식으로 사용, 인스턴스의 메서드로 사용한다.

**static 메소드** : 클래스로만 접근하여 사용

**프로토타입 메소드** : 메서드를 의미한다. 자바스크립트는 클래스기반이 아니라 프로토타입을 이용해서 메서드를 구현한 것 이다.



### (in 생성자 함수)

**메소드** : 프로토타입 메서드를 말한다.

- 생성자 함수에서는 클래스의 메서드 같은 개념이 없다.

**static 메소드** : 생성자 함수 내부에 일반적인 함수를 선언하면 static 메서드가 된다. (생성자 함수로만 접근가능)

**프로토타입 메소드** : 생성자의 프로토타입 객체에 안에 정의된 메서드이다. 클래스의 메서드 역할을 하기위해 프로토타입 개념을 도입한 것이기 때문에 프로토타입 메서드가 우리가 아는 메서드이다. 

아래와 같이 사용한다.  

```javascript
인스턴스.__proto__.메서드()
```

```javascript
생성자.prototype.메서드()
```



결국 메소드가 프로토타입 메소드였다...



## 이로써 프로토타입이 존재하는 이유를 알게 되었다.

프로토타입은 다른 언어에서 사용하는 클래스를 흉내내기 위해 태어났다.

다른 언어에서 클래스에는 메서드, 프로퍼티, 상속 이라는 대표적인 특징이 있는데 

이들을 구현하기 위해 태어난 아이이다.



생성자함수에 자동으로 생성되는 프로토타입이라는 객체에 변수와 함수를 저장해서 메서드와 프로퍼티처럼 사용하고, 프로토타입(`생성자.prototype = 다른객체`)에 다른 객체를 할당해서 상속을 구현한다.



실제로 생성자 함수와 클래스의 구조는 동일하다.(프로토타입 기반으로 구현)



### 클래스(생성자함수) 상속

![class](https://user-images.githubusercontent.com/41064875/105627925-0acc0d80-5e7d-11eb-82fa-3068c4ea69db.png)

Bridge는 중간다리 역할을 한다. 상속하는 객체의 프로퍼티를 참조하지 않기위해 사용한다.

Bridge를 중간에 두지 않고 한번에 상속하면 상속하는 생성자 내부에 정의된 프로퍼티도 함께 인스턴스로 할당 된다.

(편의상 Bridge이고 자바스크립트에서는 f 라고 정의되어있다. console을 찍어보니 f라 나온다.)



### 순서 

1. 브릿지의 프로토타입에 상속할 생성의 인스턴스로 할당한다.

	```javascript 
	Bridge.prottotype = new 상속할생성자();
	```

2. 상속 받은 생성자의 프로토타입에 브릿지를 인스터로 할당한다.

	```javascript
	상속받는생성자.prototype = new Bridge();
	```

3. 상속 받은 생성자의 프로토타입의 생성자를 자신로 다시 설정한다.

	(prototype이 다른 객체의 인스턴스로 덮어씌워젔기 때문에)

	```javascript
	상속받는생성자.prototype.constructor = 상속받는생성자;
	```



이 귀찮은 과정이 자바스크립트 내부 함수로 구현되어있다.

![class2](https://user-images.githubusercontent.com/41064875/105627932-0ef82b00-5e7d-11eb-9f3e-56f52fd67d3e.png)



간단하게 아래와 같이 사용하면 된다.

```javascript
extendClass(상속하는생성자, 상속받는생성자);
```

