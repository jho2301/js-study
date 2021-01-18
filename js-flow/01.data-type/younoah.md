## Data types

자바스크립트는 크게 2가지 타입으로 나뉜다.

1. Primitive Type(기본형 타입)
2. Reference Type(참조형 타입)



### Primitive Type(기본형 타입)

- Number
- String
- Boolean
- null
- undefined
- Symbol



### Reference Type(참조형 타입)

참조형 타입으로는 대표적으로 객체가 있다. 그리고 객체의 하위의 배열, 함수, 정규표현식이 있다.

- Object
	- Array
	- Function
	- RegExp(정규표현식)





이렇게 구분하는 이유는?? 왜 구분 해야하는지? 어떤차이가 있는지??

메모리를 우선 살펴보자.



## 변수의 메모리 할당과정

### 기본형 타입

0. **메모리 처음 상태**

![memory1](/Users/uno/Desktop/web/js/JSFlow/images/memory1.png)

<br>

1. **변수 선언**

```javascript
let a;
```

![memory2](/Users/uno/Desktop/web/js/JSFlow/images/memory2.png)

<br>

2. **변수에 값 할당**

```javascript
let a;
a = 'abc';
```

![memory3](/Users/uno/Desktop/web/js/JSFlow/images/memory3.png)

![memory4](/Users/uno/Desktop/web/js/JSFlow/images/memory4.png)

<br>

3. **변수에 값 재할당**

```javascript
let a;
a = 'abc';
a = 'abcdef';
```

![memory5](/Users/uno/Desktop/web/js/JSFlow/images/memory5.png)

![memory6](/Users/uno/Desktop/web/js/JSFlow/images/memory6.png)

> **여기서 중요!**
>
> `a` 의 값을 `abc` 에서 `abcdef` 바꾸었는데 5004번 메모리에 있는 `abc` 값을 바꾸지 않고 5005번 메모리를 새로 할당해서 `abcdef` 값을 넣은후 변수 `a` 가 5005번 메모리를 가리키게 한다. 
>
> 즉,
>
> **기본형 타입 데이터 변수는 재할당 할때 가리키는 값을 바꾸지 않고 **
>
> **새로운 메모리에 값을 할당하여 변수가 가리키는 값의 주소를 바꾼다.**



<br>

### 참조형 타입

**전체 코드와 메모리 처음 상태**

```javascript
let obj = {
    a: 1;
    b: 'bbb';
};
obj.a = 2;
```

![object1](/Users/uno/Desktop/web/js/JSFlow/images/object1.png)

<br>

**객체 선언과 할당**

```javascript
let obj = {
...
};
```

![object2](/Users/uno/Desktop/web/js/JSFlow/images/object2.png)

![object3](/Users/uno/Desktop/web/js/JSFlow/images/object3.png)

![object4](/Users/uno/Desktop/web/js/JSFlow/images/object4.png)

<br>

**객체의 첫번째 프로퍼티 할당**

```javascript
let obj = {
    a: 1;
    ...
};
```

![object5](/Users/uno/Desktop/web/js/JSFlow/images/object5.png)

<br>

**객체의 두번째 프로퍼티 할당**

```javascript
let obj = {
    a: 1;
    b: 'bbb';
};
```

![object6](/Users/uno/Desktop/web/js/JSFlow/images/object6.png)

<br>

**완료**

![object7](/Users/uno/Desktop/web/js/JSFlow/images/object7.png)

<br>

**객체의 프로퍼티의 값 수정**

```javascript
obj.a = 2;
```

![object8](/Users/uno/Desktop/web/js/JSFlow/images/object8.png)

<br>

**여기서 중요!**

![object9](/Users/uno/Desktop/web/js/JSFlow/images/object9.png)

기본형 타입 변수와 달리 참조하는 변수가 한 단계(obj.a)가 더 있기 때문에 obj가 가리키는 주소값은 바뀌지 않는다.

단 `obj.a`는 기본형 타입 프로퍼티이므로 `obj.a`가 가리키는 주소값은 바뀐다. 

즉, 

**참조형 타입 변수는 재할당 할 때 변수가 가리키는 값의 주소는 바뀌지 않는다.**

<br>

### 참조형 타입의 중첩된 형태

이번에는 객체 안에 배열타입의 변수를 알아보자.

<br>

**전체코드와 메모리 초기상태**

```javascript
let obj = {
    x: 3;
    arr: [3, 4];
};
obj.arr = 'str';
```

![object_array1](/Users/uno/Desktop/web/js/JSFlow/images/object_array1.png)

<br>

**객체의 기본형 타입 프로퍼티 할당**

```javascript
let obj = {
    x: 3;
    ...
};
```

![object_array2](/Users/uno/Desktop/web/js/JSFlow/images/object_array2.png)

여기 까지는 위에 내용과 동일하다.

<br>

**객체의 참조형 타입(배열) 프로퍼티 할당**

```javascript
let obj = {
    x: 3;
    arr: [3, 4]; // 여기
};
```

![object_array3](/Users/uno/Desktop/web/js/JSFlow/images/object_array3.png)



<br>

![object_array4](/Users/uno/Desktop/web/js/JSFlow/images/object_array4.png)



<br>

![object_array5](/Users/uno/Desktop/web/js/JSFlow/images/object_array5.png)

7104(arr) 메모리가 가리키는 값을 5004로 한다.

<br>

**수정**

```javascript
obj.arr = 'str';
```

![object_array6](/Users/uno/Desktop/web/js/JSFlow/images/object_array6.png)

5006번에 새로운 값을 할당하고 arr(7104)가 5006을 가리키게 한다.

<br>

![object_array7](/Users/uno/Desktop/web/js/JSFlow/images/object_array7.png)

이때 5004번 메모리와 8104, 8106 메모리는 참조 되지 않기 때문에 Galbage Collecting을 당하게 된다.

<br>

## 변수 복사

### 기본형 타입 변수 복사

```javascript
let a = 10;
let b = a;
```

![copy1](/Users/uno/Desktop/web/js/JSFlow/images/copy1.png)

b는 a가 가리키는 값의 주소를 똑같이 가리킨다.

<br>

### 참조형 타입 변수 복사

```javascript
let obj1 = {c: 10, d: 'ddd'};
let obj2 = obj1;
```

![copy2](/Users/uno/Desktop/web/js/JSFlow/images/copy2.png)

우선 obj1이 메모리 할당된 상태이고 5003이라는 레퍼런스의 주소를 가리킨다.

<br>

![copy3](/Users/uno/Desktop/web/js/JSFlow/images/copy3.png)

obj2는 obj1이 가리키는 레퍼런스의 주소(5003)를 똑같이 가리킨다.

<br>

**기본형 타입 변수의 값 수정**

```javascript
b = 15;
```

![copy4](/Users/uno/Desktop/web/js/JSFlow/images/copy4.png)

15의 값을 갖늦 메모리(5004)가 새로 할당이 되고 b가 이 주소를 가리킨다.

<br>

**참조형 타입 변수의 수정**

```javascript
obj2.c = 20;
```

![copy5](/Users/uno/Desktop/web/js/JSFlow/images/copy5.png)

메모리에 20(5005)이 할당되고 obj.c가 이 주소를 가리킨다.

<br>

**여기서 다시 중요!**

![copy6](/Users/uno/Desktop/web/js/JSFlow/images/copy6.png)

기본형 타입 변수들이 가리키는 주소는 바뀌었는데 

참조형 타입의 변수가 기리키는 주소는 바뀌지 않았다.

**기본형 타입 변**수는 재할당 할때 가리키는 값을 바꾸지 않고 새로운 메모리에 값을 할당하여 변수가 가리키는 값의 주소를 바꾼다.

**참조형 타입 변**수는 재할당 할 때 변수가 가리키는 값의 주소는 바뀌지 않는다.





## 기억하기

### 기본형 타입 데이터들을 메모리에 한번 만들어지면 다시 재사용이 된다.

(값을 변경할때 메모리의 값이 수정되지 않고 새로운 값이 메모리에 할당되어 그 주소를 가리킨다. )

```javascript
const a = 3;
const b = [3, 4];
```

> 3이 메모리에 할당이 되었는데
>
> a와 b 모두 메모리에 1개 할당 되어있는 3을 가리킨다.
>
> 이렇게 가능하게 한것이 런타임(엔진)에서 그렇게 가능하도록 하게 했기 때문이다.
>
> 기억하자! 1개의 기본형 타입 데이터는 딱 1개만 생성된다.
>
> (정리)
>
> 재할당 할때 기존에 메모리에 생성된 데이터에 새로운 값을 넣는게 아니라
>
> 메모리에 새로 값을 할당하고 기존값과 연결을 끊고 새로운 값과 연결을 한다.



이때 참조되지 않는 값은 Galbage Collecting이된다.





### 기본형 타입과 참조형 타입의 차이점

> **기본형 타입 변**수는 재할당 할때 가리키는 값을 바꾸지 않고 새로운 메모리에 값을 할당하여 변수가 가리키는 값의 주소를 바꾼다.
>
> **참조형 타입 변**수는 재할당 할 때 변수가 가리키는 값의 주소는 바뀌지 않는다.





## 궁금증

강의 설명할 때 변수(식별자)를 저장하는 메모리, 값을 저장하는 메모리, 참조를 위한 메모리, 구분을 지었는데 나름 규칙이 보였다. 실제로 메모리 영역별로 이 규칙이 있을까?

**공부하다가 메모리 영역이 구분되어 있는거 같던데 찾아보자!**



재할당 되었을때 기존에 있는 데이터와 링크(참조관계)를 끈는데

이 데이터는 그럼 쓰레기 데이터가 되었을 거다 그럼 언제 메모리에서 해제가 될까???

프로그램이 종료 될 때까지 유지가 될까?



Galbage Collecting이란??