## 0. 자바스크립트의 데이터 타입

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
	\- Array
	\- Function
	\- RegExp(정규표현식)





이렇게 구분하는 이유는??

왜 구분 해야하는지??

어떤차이가 있는지??

에 대한 궁금증을 해결하기 위해서는 각 데이터 타입이 어떻게 메모리에서 동작되는지를 보면 알 수 있다.



## 1. 변수의 메모리 할당 과정

각각의 데이터 타입의 변수가 어떻게 할당되는지 보자.



### 기본형 타입

1. **메모리 처음 상태**

![img](https://media.vlpt.us/images/younoah/post/46ab4a01-3256-4a09-be1b-4233143e5f21/memory1.png)



1. **변수 선언**

```javascript
let a;
```

![img](https://media.vlpt.us/images/younoah/post/cf3bf8d6-5f75-4e52-9e31-a2c93b5f4398/memory2.png)

우선 변수명(혹은 식별자, 이하 변수)과 변수가 가리키는 값을 저장하기 위해 메모리가 할당되어지고 변수명(식별자)가 저장된다.



1. **변수에 값 할당**

```javascript
let a;
a = 'abc';
```

![img](https://media.vlpt.us/images/younoah/post/e253d3a6-cd92-4e57-807e-268109b39819/memory3.png)

값 abc가 메모리에 할당이 된다.

![img](https://media.vlpt.us/images/younoah/post/5b75faac-9295-41c4-9386-f915b36cab70/memory4.png)

변수 a가 저장된 메모리에 값(abc)가 저장된 메모리 주소를 저장한다.

(변수a가 저장된 메모리가 값을 가리킨다.)



1. **변수에 값 재할당**

```javascript
let a;
a = 'abc';
a = 'abcdef';
```

![img](https://media.vlpt.us/images/younoah/post/c63d12fe-dfab-47e4-923e-059bb5d7695f/memory5.png)

값 abcdef가 메모리에 할당된다.

![img](https://media.vlpt.us/images/younoah/post/2c587bad-a29b-4b1a-a0c5-2ac5b6e9fbf2/copy6.png)

변수 a가 가리키는 값의 주소가 abcdef가 저장된 메모리 주소로 바뀐다.

> **여기서 중요!**
>
> `a` 의 값을 `abc` 에서 `abcdef` 바꾸었는데 5004번 메모리에 있는 `abc` 값을 바꾸지 않고 5005번 메모리를 새로 할당해서 `abcdef` 값을 넣은후 변수 `a` 가 5005번 메모리를 가리키게 한다.
>
> 즉,
>
> **기본형 타입 데이터 변수는 재할당 할때 가리키는 값을 바꾸지 않고**
>
> **새로운 메모리에 값을 할당하여 변수가 가리키는 값의 주소를 바꾼다.**





### 참조형 타입

**전체 코드와 메모리 처음 상태**

```javascript
let obj = {
    a: 1;
    b: 'bbb';
};
obj.a = 2;
```

![img](https://media.vlpt.us/images/younoah/post/255776ee-1788-4ddb-8516-c6df3cad015e/object1.png)



**객체 선언과 할당**

```javascript
let obj = {
...
};
```

![img](https://media.vlpt.us/images/younoah/post/ec44cb4c-8f06-4122-b4dd-556d4a1b6360/object2.png)

변수명과 값을 가리키는 메모리주소를 할당하고 변수명을 저장한다



![img](https://media.vlpt.us/images/younoah/post/afe6c7d9-bef5-47f3-aae9-ab21609b6127/object3.png)

![img](https://media.vlpt.us/images/younoah/post/d07c8bf7-5f5d-447a-a6cb-b31abfddb650/object4.png)

프로퍼티를 참조하는 참조용 메모리를 할당한다. 이 메모리는 프로퍼티가 저장된 주소를 가리킨다.



**객체의 첫번째 프로퍼티 할당**

```javascript
let obj = {
    a: 1;
    ...
};
```

![img](https://media.vlpt.us/images/younoah/post/776e9674-4fde-4ce0-90ef-3ebed4087219/object5.png)

프로퍼티`a`가 할당이 되고 값 1이 저장된 주소를 가리킨다.(변수가 할당되는 구조와 같다.)



**객체의 두번째 프로퍼티 할당**

```javascript
let obj = {
    a: 1;
    b: 'bbb';
};
```

![img](https://media.vlpt.us/images/younoah/post/173ea889-8a7e-4c45-9981-0dbba8074c2b/object6.png)

프로퍼티`b`가 할당이 되고 값 `bbb`가 저장된 주소를 가리킨다.



**완료**

![img](https://media.vlpt.us/images/younoah/post/93af01e7-a73a-4457-b8e7-89caf0ad4fc9/object7.png)

최종적으로 `obj` 가 참조용 메모리(5002)를 가리킨다.



**객체의 프로퍼티의 값 수정**

```javascript
obj.a = 2;
```

![img](https://media.vlpt.us/images/younoah/post/949a3ed6-7272-4aaa-9391-9123f522051d/object8.png)

obj.a의 값을 확인한다.



**여기서 중요!**

![img](https://media.vlpt.us/images/younoah/post/887f817e-1cd7-4505-9a2c-21ef85b91853/object9.png)

기본형 타입 변수와 달리 참조하는 변수가 한 단계(obj.a)가 더 있기 때문에 obj가 가리키는 주소값은 바뀌지 않는다.

단 `obj.a`는 기본형 타입 프로퍼티이므로 `obj.a`가 가리키는 주소값은 바뀐다.

즉,

**참조형 타입 변수의 프로퍼티 값을 재할당 할 때 변수가 가리키는 값의 주소는 바뀌지 않는다.**

단, 참조형 변수 자체를 재할당 하면 해당 변수가 가리키는 값의 주소는 바뀐다.

> ```javascript
> obj.a = 1;
> ```
>
> 는 obj가 가리키는 주소는 그대로이고 obj.a가가리키는 값의 주소가 바뀌는것
>
> ```javascript
> obj = 1;
> ```
>
> 는 obj가 가리키는 값의 주소가 바뀐다.



### 참조형 타입의 중첩된 형태

이번에는 객체 안에 배열타입의 변수를 알아보자.



**전체코드와 메모리 초기상태**

```javascript
let obj = {
    x: 3;
    arr: [3, 4];
};
obj.arr = 'str';
```

![img](https://media.vlpt.us/images/younoah/post/6aa082c1-2b13-454f-8036-19eed5fcd933/object_array1.png)



**객체의 기본형 타입 프로퍼티 할당**

```javascript
let obj = {
    x: 3;
    ...
};
```

![img](https://media.vlpt.us/images/younoah/post/9c04525a-a238-4c9f-86aa-57e140054ca4/object_array2.png)

여기 까지는 위에 내용과 동일하다.



**객체의 참조형 타입(배열) 프로퍼티 할당**

```javascript
let obj = {
    x: 3;
    arr: [3, 4]; // 여기
};
```

![img](https://media.vlpt.us/images/younoah/post/40d26bf1-5104-45ed-9cbd-92c88b24ef00/object_array3.png)

프로퍼티 arr를 할당한다.



![img](https://media.vlpt.us/images/younoah/post/cf7e1f0c-10bf-4808-aec3-d413938ba5e9/object_array4.png)

arr가 참조하는 메모리를 할당한다. 해당 메모리는 8104부터 쭉 메모리를 가리킨다.



![img](https://media.vlpt.us/images/younoah/post/4b251833-c5b0-4db8-a584-8e39cb02d155/object_array5.png)

객체와 비슷하게 인덱스를 저장하고 해당 인덱스가 가리키는 값의 주소를 저장한다.

arr가 5004 메모리를 가리킨다.



**수정**

```javascript
obj.arr = 'str';
```

![img](https://media.vlpt.us/images/younoah/post/de5c3ce6-efcc-4292-a4eb-26163e5727dd/object_array6.png)

5006번에 새로운 값(str)을 할당하고 arr(7104)가 5006을 가리키게 한다.



![img](https://media.vlpt.us/images/younoah/post/7c870d2d-8c37-4d75-9cde-3f6c0e84ec0f/object_array7.png)

이때 5004번 메모리와 8104, 8106 메모리는 참조 되지 않기 때문에 Galbage Collecting인된다.





## 2. 변수 복사

이번에 각각의 데이터 타입의 변수가 어떻게 복사가 되는지 알아보자.



### 기본형 타입 변수 복사

```javascript
let a = 10;
let b = a;
```

![img](https://media.vlpt.us/images/younoah/post/7577d6c4-4307-4c95-93e0-75cfb2818706/copy1.png)

b는 a가 가리키는 값의 주소를 똑같이 가리킨다.



### 참조형 타입 변수 복사

```javascript
let obj1 = {c: 10, d: 'ddd'};
let obj2 = obj1;
```

![img](https://media.vlpt.us/images/younoah/post/c301bd0c-1dcf-476c-8815-912728624c36/copy3.png)

obj2는 obj1이 가리키는 레퍼런스의 주소(5003)를 똑같이 가리킨다.



**기본형 타입 변수의 값 수정**

```javascript
b = 15;
```

![img](https://media.vlpt.us/images/younoah/post/0cab91cd-9520-4e24-b040-5c205aa23c5b/copy4.png)

15의 값을 갖는 메모리(5004)가 새로 할당이 되고 b가 이 주소를 가리킨다.



**참조형 타입 변수의 수정**

```javascript
obj2.c = 20;
```

![img](https://media.vlpt.us/images/younoah/post/a80550b1-2bd5-4f85-b294-3a234aee1ad4/copy5.png)

메모리에 20(5005)이 할당되고 obj.c가 이 주소를 가리킨다.



**여기서 다시 중요!**

![img](https://media.vlpt.us/images/younoah/post/69c96a1e-a7b9-405a-b1e1-a781949141f8/copy6.png)

기본형 타입 변수들이 가리키는 주소는 바뀌었는데

참조형 타입의 변수가 기리키는 주소는 바뀌지 않았다.

**기본형 타입 변**수는 재할당 할때 가리키는 값을 바꾸지 않고 새로운 메모리에 값을 할당하여 변수가 가리키는 값의 주소를 바꾼다.

**참조형 타입 변**수는 재할당 할 때 변수가 가리키는 값의 주소는 바뀌지 않는다.





## 3. 기억하기

### 기본형 타입 데이터들 다시 재사용이 된다.

기본형 타입 데이터들을 메모리에 한번 만들어지면 다시 재사용이 된다.

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



### 값 수정

값을 재할당 할때는 기존의 값이 저장된 메모리에 새로운 값으로 수정되는게 아니다.

새로운 값이 저장된 메모리를 새로 할당하고 기존값과 연결을 끊고 새로운 값과 연결을 한다.

이때 기존의 값은 참조가 끊기게 되고 참조되지 않는 값은 Galbage Collecting이된다.



### 기본형 타입과 참조형 타입의 차이점

> **기본형 타입 변**수는 재할당 할때 가리키는 값을 바꾸지 않고 새로운 메모리에 값을 할당하여 변수가 가리키는 값의 주소를 바꾼다.
>
> **참조형 타입 변수**의 프로퍼티를 재할당 할 때 변수가 가리키는 값의 주소는 바뀌지 않는다.
>
> 단, 참조형 변수 자체를 재할당 하면 해당 변수가 가리키는 값의 주소는 바뀐다.





## 4. 참고

**Garbage Collection이란??**

> 참고
>
> https://www.youtube.com/watch?v=j9Vncn04GsE
>
> https://www.youtube.com/watch?v=tTH4WdpRC2k

C/C++에서는 메모리 관리를 프로그래머가 직접 해주어야 한다.

하지만 파이썬, 자바스크립트, 자바, Go와 같은 언어들에서는 개발자가 메모리를 할당하고 해제하는것을 신경 쓸 필요가 없다.

Garbage Colletor라는 도구가 메모리 해제를 알아서 해주기 때문이다.

Garbage Colletor 사용이 끝난 객체를 알아서 메모리 공간에서 지워주는 도구이다.

**Garbage Colletion 메모린를 해제하는 주요 개념**

- Reference Counting
- Mark & Sweep



**추가**

[자바스크립트의 메모리관리](https://engineering.huiseoul.com/자바스크립트는-어떻게-작동하는가-메모리-관리-4가지-흔한-메모리-누수-대처법-5b0d217d788d)