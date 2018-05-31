> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# Dynamic Programming 03

Optimal Substructure

## 동적계획법

일반적으로 최적화문제(optimisation problem) 혹은 카운팅(counting)문제에 적용됨

1. 주어진 문제에 대한 순환식(recurrence equation)을 정의한다.
2. 순환식을 memoization 혹은 bottom-up 방식으로 푼다.

> 최적화문제는 어떤 문제의 최대, 최소값에 해당하는 결과를 구하는 문제
>
> 예를 들어, 최단 경로 문제 등

### 특징

* subproblem들을 풀어서 원래 문제를 푸는 방식. 그런 의미에서 분할정복법과 공통성이 있음
* 분할정복법에서는 분할된 문제들이 서로 disjoint(서로 별개다.)하지만 동적계획법에서는 그렇지 않음
* 즉, 서로 overlapping하는 subproblem들을 해결함으로써 원래 문제를 해결

### 분할정복법 vs 동적계획법

* 분할정복법의 경우 두 subproblem은 별개의 문제이다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_07.png?raw=true)

* 동적계획법의 경우 subproblem이 disjoint하지 않다. 서로 별개의 문제가 아니다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_08.png?raw=true)

## Optimal Substructure

* **어떤 문제의 최적해가 그것의 subproblem들의 최적해로부터 효율적으로 구해질 수 있을 때** 그 문제는 optimal substructure를 가진다고 말한다.
* 분할정복법, 탐욕적기법, 동적계획법은 모두 문제가 가진 이런 특성을 이용한다.

### Optimal Substructure를 확인하는 질문

* "최적해의 일부분이 그부분에 대한 최적해인가?"
  * 대답이 yes이면, 순환식이 optimal substructure를 표현하는데 문제가 없다.
  * 대답이 no라면, 순환식은 성립될 수 없다.
* 예를 들면, 최단경로(shortest-path) 문제에서

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_09.png?raw=true)

* 순환식은 optimal substructure를 표현한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_10.png?raw=true)

* 다른 예로, 최장경로(longest-path) 문제
  * 노드를 중복 방문하지 않고 가는 가장 긴 경로
  * Optimal substructure를 가지는가?

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_11.png?raw=true)

* 최장경로는 조금 복잡한 형태의 optimal substructure를 갖는 것이지, 가지지 않는다고 말할 수는 없다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/dp_12.png?raw=true)