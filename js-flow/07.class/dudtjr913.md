# Class (prototype)
---

## Class를 설명하기에 앞서

Java나 C++과 같은 클래스 기반의 언어와 달리 JavaScript는 프로토타입 기반 언어이다.<br>
그래서 **"상속"** 이라는 개념이 존재하지 않고, 프로토타입 체이닝에 의해 **"상속처럼 보이게"** 동작한다.

만약 new Array()를 호출한다면 Array의 prototype을 __proto__로 가지는 instance가 생성될 것이다.<br>
> instance는 Class의 속성을 지닌 구체적인 객체이다.

이때, instance는 Array의 prototype에 존재하는 모든 요소들을 사용할 수 있게 되는데, 이것을 상속이라고 보면 될 것 같다.

## Class는?

갑자기 상속에 대해 설명한 이유는 Class를 이해하기 위해서는 상속을 알고 있어야 하기 때문이다.

우선 Class는 공통 집단이라고 이해하면 쉽다.

<img src="https://user-images.githubusercontent.com/64782636/105625309-976dd000-5e6b-11eb-9d18-473913c1a4c3.png" width="70%">

위 이미지를 보면 책은 가장 추상적이고 자바스크립트 책과 자기계발서를 모두 포함하는 집단이다.<br>
이를 프로그래밍 언어로 보면 책은 Class, 자바스크립트나 자기계발서는 책의 instance라고 볼 수 있다.<br>

더 들어가보면 자바스크립트는 코어 자바스크립트, 모던 자바스크립트 Deep Dive, 혼자서 공부하는 자바스크립트와 같은 instance를 <br>가지고 있다.

즉, 자바스크립트나 자기계발서는 Class이자 instance이다.

이는 다른 모든 대상에게 적용될 수 있기 때문에 Class는 공통 집단이고, 추상적이라고 볼 수 있다.

```
function Book(kind) {
  this.kind = kind;
}

Book.prototype.getKind = function () {
  return this.kind;
};

const javaScript = new Book('자바스크립트');
console.log(javaScript.getKind()); // 자바스크립트
```

이렇게 Book이라는 Class를 만들어서 javaScript의 변수에 할당하면, instance가 생성되고, Book의 prototype을 사용할 수 있다.<br>
이것이 Class의 상속이다.

## 사용하는 이유?

- 코드 재사용성 ↑
- 그룹화

## 확장

```
function Book(kind, name) {
  this.kind = kind;
  this.name = name;
}

Book.prototype.getKind = function () {
  return this.kind;
};

Book.prototype.getName = function () {
  return this.name;
};

function JavaScript(kind, name, publisher) {
  this.kind = kind;
  this.name = name;
  this.publisher = publisher;
}

JavaScript.prototype.getKind = function () {
  return this.kind;
};

JavaScript.prototype.getName = function () {
  return this.name;
};

JavaScript.prototype.getPublisher = function () {
  return this.publisher;
};
```

위와 같은 코드가 있을 때, getkind, getName이 겹치는 것을 볼 수 있다.<br>
이런 경우에 확장을 통해 Class를 적극적으로 사용할 수 있다.

방법은 superClass인 Book을 상위로 올리고, subClass인 JavaScript의 prototype을 Book의 instance로 함과 동시에<br>
JavaScript의 prototype에 constructor을 주어 instance를 생성 할 수 있게 하면 된다.

```
  //(...생략)
JavaScript.prototype = new Book();
JavaScript.prototype.constructor = JavaScript;
JavaScript.prototype.getPublisher = function () {
  return this.publisher;
};

const core = new JavaScript('자바스크립트', '코어 자바스크립트', '위키북스');
console.log(core.getPublisher()); // 위키북스
```

정상적으로 작동하는 것 같아 보이지만 core을 콘솔에서 확인해보면,
```
JavaScript {kind: "자바스크립트", name: "코어 자바스크립트", publisher: "위키북스"}
kind: "자바스크립트"
name: "코어 자바스크립트"
publisher: "위키북스"
  __proto__: Book
  constructor: ƒ JavaScript(kind, name, publisher)
  getPublisher: ƒ ()
  kind: undefined
  name: undefined
    __proto__: Object
```

__proto__에 Book Class가 가진 kind와 name 프로퍼티가 존재하는 것을 볼 수 있다.<br>
이는 만약 JavaScript의 kind나 name을 삭제했을 경우 __proto__의 값을 가져올 것이기에 해결해야 한다.

해결방법으로는 더글라스 크락포드가 제시한 방법으로 빈 생성자 함수를 만들어 superClass가 이를 바라보게 하는 것이 있다.
```
  // (...생략)
function Bridge() {}
Bridge.prototype = Book.prototype;
JavaScript.prototype = new Bridge();
  //(...생략)
```

이렇게 Bridge라는 빈 생성자 함수를 만들어 prototype을 superClass의 prototype과 동일하게 만들고, Bridge의 instance를<br>
만들게 되면 순전히 superClass의 prototype만 가져올 수 있게 된다.

