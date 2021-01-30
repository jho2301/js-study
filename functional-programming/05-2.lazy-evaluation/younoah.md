## 엄격한 평가와 지연 평가의 평가 순서



지연 평가는 필요한 정보만 평가하므로, 일반적인 엄격한 평가보다 훨씬 빠른 것을 나타냄을 보여줄 수 있는데 아래 예시를 통해서 각각의 평가의 순서를 알아보자.

```javascript
// 엄격한 평가
go(range(10),
    map(n => n + 10),
    filter(n => n % 2),
    take(2),
	console.log
);

// 지연 평가
go(L.range(10),
    L.map(n => n + 10),
    L.filter(n => n % 2),
    take(2),
    console.log
);
```



#### 엄격한 평가

`range` 에서 0부터 9를 담은 배열을 먼저 생성한 후에, 그 생성한 배열을 `map` 으로 넘기고, 그 결과 배열을 다시 `filter` 로 넘기고, 다시 `take` 로 넘기 면서 순차적으로 평가된다.

순서는 아래와 같다.

![strict-eval](https://user-images.githubusercontent.com/41064875/106350705-4d2c9900-631a-11eb-964c-28150e224451.png)

(이미지 출처: https://opentogether.tistory.com/72)

#### 지연 평가

지연 평가 함수인 `L.range` , `L.map` , `L.filter` 함수들은 이터레이터를 생성한 후 대기를 한다. 제일 먼저 `take` 함수가 실행되면서  `L.filter` 에게 값을 하나 요청한다.  `L.filter` 는 다시 `L.map` 에게 값을 하나 요청하고 `L.map` 은 다시 `L.range` 에게 값을 하나 요청한다.

즉,  `take` ->  `L.filter`  -> `L.map`  ->  `L.range`  순으로 실행이 된다.

`L.range` 에서 0을 생성한 후, `L.map` 에서 이 요소에 10을 더한 후, `L.filter` 가 그 값을 판별해 `false` 되어 더이상 `go`함수를 진행하지 않고, 다시 `L.range` 에서 1을 생성하는 식으로 반복한다.

순서는 아래와 같다.

![lazy-eval](https://user-images.githubusercontent.com/41064875/106350704-4c940280-631a-11eb-98ea-4e03516f1048.png)

(이미지 출처: https://opentogether.tistory.com/72)

 엄격한 평가에서는 `range`에서 인자로 넣은 값만큼 `map` 에서 전부 배열을 생성한 후, 이어서 모든 함수를 거쳐가는 반면에, 지연 평가에서는 당장 조건에 해당하는 값을, 동적으로 평가하여 그때 마다 필요로 하는 값을 평가한다. 



## map, filter 계열 함수들의 결합 법칙

위의 예제를 보았듯이 `map`,  `filter` 와 같은 함수들이 순차적으로 실행되어 나온 결과 값과  `L.map`, `L.filter` 와 같은 함수들이 순차적으로 실행되어 나온 결과 값이 같은 것을 알 수 있다.

이처럼

사용하는 데이터가 무엇이든지, 사용하는 보조 함수(콜백 함수)가 순수 함수(단순 연산 함수)라면 아래 처럼 결합법칙이 성립한다.



![combination](https://user-images.githubusercontent.com/41064875/106350703-4aca3f00-631a-11eb-8a37-22d28db9b7d7.png)

(이미지 출처: https://opentogether.tistory.com/72)

예를 들어 0부터 9까지의 값에 map을 우선 다 적용한 뒤 filter를 적용한 결과나, 0부터 하나씩 map과 filter를 차례대로 9까지 적용한 결과가 같다.
