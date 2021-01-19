## 0. 실행 콘텍스트 (Execution Context)란?



실행할 코드에 제공할 환경 정보들을 모아놓은 객체로서, 동일한 스코프에 있는 코드들을 실행할 때 필요한 환경 정보를 모아 컨텍스트를 구성하고, 이를 콜 스택에 쌓아서 실행 순서를 보장한다.



자바스크립트에서 실행 콘텍스트의 단위(코드 뭉치)가 있는데 그것이 **함수**이다.

(ES6 이후로는 for문, if문 같은 아이들 한테도 스코프 개념이 생겨서 하나의 실행 단위가 된다.)

자바스크립트는 이런 덩어리 단위로 코드를 실행할 때 필요한 환경 정보들을 담는 객체 즉, 실행 컨텍스트를 구성하고 활용한다.



> 전역 공간도 함수이다.
>
> 자바스크립트 파일이 실행되면 전역공간이 실행되고 전역공간이 끝나면 실행이 종료된다.
>
> 이처럼 전역 공간도 함수처럼 실행되고 종료되는 개념이기 때문에 전역공간도 하나의 함수이다.





## 1. 콜 스택

> 콜 스택이란 현재 어떤 함수가 동작하고 있는지, 다음에 어떤 함수가 호출되어야 하는지 등을 제어하는 자료구조를 말한다.



아래 코드가 어떻게 콜스택에 쌓이는지 보자.

### 예제코드

```javascript
var a = 1;
function outer() {
    console.log(a);
    
    function inner() {
        console.log(a);
        var a = 3;
    }
    
    inner();
    
    console.log(a);
}
outer();
console.log()
```

![img](https://media.vlpt.us/images/younoah/post/9556c003-cd14-438f-abf0-b93d67420051/callStack.png)

가장먼저 전역 공간에 대한 전역 컨텍스트가 콜스택에 쌓인다. 이후에 `outer()` 함수에 대한 실행 컨텍스타가 쌓이고 `outer()` 함수안에 있는 `inner` 함수에 대한 실행 컨텍스트가 쌓인다.





## 2. 실행 컨텍스트(Execution Context)의 3가지 객체

하나의 실행 컨텍스트는 아래의 구조로 구성되어있다.

- 실행컨텍스트
	\- VariavleEnvironment
	\- environmentRecord(snapshot)
	\- outerEnvironmentReference(snapshot)
	\- Lexical Environment
	\- environmentRecord : 현재문맥의 식별자(hoisting)
	\- outerEnvironmentReference : 외부 식별자(scope chain)
	\- ThisBinding



위에서 콜스택을 설명할 때 사용한 코드의 실행 컨텍스트 구조를 예를들면 아래 이미지와 같다.

![img](https://media.vlpt.us/images/younoah/post/04ab7135-9c01-40e7-a658-0e7c6534ce72/Execution_context.png)

`inner()` 함수에 대한 실행 컨텍스트 뿐만 아니라 `outer()` 와 `전역 컨텍스트` 도 같은 구조로 되어있다.

단, `전역 컨텍스트`는 **outerEnvironmentReference** 가 없다. 이유는 아래에서 알게 된다.





### 2.1. Variable Environment

현재 컨텍스트 내의 식별자 정보, 외부 환경 정보, 선언 시점의 `LexicalEnvironment`의 스냅샷을 유지한다.



`Variable Environment`와 `Lexical Environment` 은 구조도 같고 초기에 같은 정보를 담고 있다. 하지만 코드가 진행됨에 따라 `Lexical Environment`는 값이 변경이 된다.

때문에 `Lexical Environment` 를 알면 `Variable Environment` 또한 자연스럽게 이해가 될 것이다.



이것만 기억하자! `Variable Environment`는 `Lexical Environment`의 초기 정보(snapshot)만을 담고 있다.





### 2.2. Lexical Environment

`Lexical Environment`는 식별자와 변수가 매핑되는 곳이라고 할 수 있다.

> ES6 에서 `VariableEnvironment`와 `LexicalEnvironment` 둘의 차이점은 전자가 변수 `var`만 저장하는 반면, 후자는 함수 선언과 변수 `let`과 `const`의 바인딩도 저장합니다.
>
> 출처 : https://estaid.dev/relationship-between-execution-context-and-hoisting/



`Lexical Environment` 는 다시 2가지 객체로 나뉘어 일을 하게 된다.![img](https://media.vlpt.us/images/younoah/post/76fcd656-db9a-40c1-9cef-671aeab5f385/javascript1.png)



#### 2.2.1. environmentRecord

현재 문맥의 식별자 정보를 수집해서 environmentRecord에 저장한다. 이 행위를 다른 명칭으로 **Hoisting**이라고 한다.

> 식별자는 쉽게 말해 변수명, 함수명, 매개변수이름 등이다.



**그렇다면 Hoisting은 무엇인가?**

호이스팅은 식별자 정보를 실행 컨텍스트 맨위로 끌어 올린다. 간단한 예제를 통해서 알아보자.

- 예제코드

```javascript
console.log(a());
console.log(b());
console.log(c());

function a() {
    return 'a';
}
var b = function bb() {
    return 'b';
}
var c = function cc() {
    return 'c';
}
```



- 호이스팅 결과

```javascript
function a() {
    return 'a';
}
var b;
var c
console.log(a());
console.log(b());
console.log(c());

b = function bb() {
    return 'b';
}
c = function cc() {
    return 'c';
}
```

실제로 코드가 이렇게 변경되는 것은 아니다. 개념적으로 어떻게 호이스팅이 되는지를 보이기 위해 작성한 것이지만 개념과 코드가 일치하기 때문에 이렇게 이해해도 문제는 없다.



- environmentRecord

```javascript
{
    function a() {...},
    b,
	c,  
}
```

environmentRecord의 동작(호이스팅)결과 environmentRecord객체에 현재 문맥의 식별자 정보가 담겼다.





#### 2.2.2. outerEnvironmentReference

현재 문맥(실행콘텍스트)에 관련있는 외부(현재 실행 콘텍스트 외부) 식별자 정보를 수집한다.

**스코프**란 식별자에 대한 **유효 범위**를 말한다. 자바스크립트에는 크게 함수 스코프와 블록 스코프가 존재하는데, 이 스코프를 안에서부터 바깥으로 차례로 검색 해나가는 것을 **스코프 체인**이라고 한다. 그리고 이를 가능케 하는 것이 `LexicalEnvironment`의 2번째 수집 자료인 `outerEnvironmentReference` 이다.



다시 처음에 봤던 예제로 돌아와서 outerEnvironmentReference를 통해 어떻게 외부의 식별자 정보를 참조하는 지 보자.

![img](https://media.vlpt.us/images/younoah/post/57ad0f75-830c-4328-891c-ebd698e700e6/scopeChain.png)

outer() 컨텍스트의 내부 컨텍스트는 inner()컨텍스트이고 외부컨텍스트는 전역컨텍스트이다. 따라서 outer() 컨텍스트는 전역컨텍스틀 참조할 수 있지만 inner() 컨텍스트는 참조할 수 없다.

이런식으로 안에서 부터 값이 없다면 외부로 값을 찾아나가면서 스코프 체인을 가능하게 한다.