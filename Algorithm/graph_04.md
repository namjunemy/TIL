> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 7-4. Graph 04 - DAG(Directed Acyclic Graph)

* BFS와 DFS는 방향그래프나 무방향그래프나 똑같이 적용할 수 있다.
* DAG는 방향 사이클(directed cycle)이 없는 방향 그래프
  * 사이클은 한 노드를 출발해서 다시 자기 자신으로 돌아오는 것을 말하는데, 방향 사이클이라는 것은 반드시 엣지의 화살표를 따라가야 하고, 자기 자신으로 돌아오지 않는다는 것을 의미한다.
* 예를 들면, 작업들의 우선순위를 나타내는 것을 표현할 때 사용할 수 있다.
  * 어떤 한 작업이 다른 작업에 반드시 선행되어야 할 경우

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_14.png?raw=true)

## 위상정렬(topological ordering)

* DAG에서 노드들의 순서화 v1, v2, ... , vn,
* 단, 모든 엣지 (vi, vj)에 대해서 i < j가 되도록
* 아래에 표현된 위상정렬의 예를 보면
  * 모든 엣지들이 왼쪽에서 오른쪽으로 유효하고,
  * 이는 DAG으로 표현되어있는 작업들을 수행하는 한가지 유효한 순서가 될 수 있다.
  * 왼쪽에서 오른쪽으로 가면서 작업들을 수행하면 문제가 없다는 뜻이기도 하다.
* 위상정렬은 답이 유일하지 않다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_15.png?raw=true)

## 위상정렬 알고리즘 1

* 진입간선(indegree)이 없는 노드들을 없애가면서 위상 정렬한다.

```java
topologicalSort1(G) {
    for <- 1 to n {
        진입간선이 없는 임의의 정점 u를 선택한다;
        A[i] <- u;
        정점 u와 u의 진출간선을 모두 제거한다;
    }
    >> 배열 A[1...n]에는 정점들이 위상정렬 되어 있다.
}
```

* 위상정렬 알고리즘1 동작 예시

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_16.png?raw=true)

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_17.png?raw=true)

  

## 위상정렬 알고리즘 2

* 모든 노드를 방문되지 않은 상태로 만든다.
* 그리고 하나의 비어있는 연결리스트 R을 만들고
* for문을 돌면서 방문하지 않는 노드일 경우 DFS-TS()를 호출한다.

```json
topologicalSort2(G) {
  for each v in V
    visited[v] <- NO;
  make an empty linked list R;
  for each v in V	// 정점의 순서는 상관없음
    if (visited[v] = NO) then
      DFS-TS(v, R);
}
```
* DFS-TS에서는
  * v노드를 방문 상태로 만들고
  * v노드에 인접한 각각의 노드 x에 대해서 x가 unvisited 노드라면 DFS-TS()를 다시 호출한다.
* 알고리즘이 끝나며면 연결리스트 R에는 정점들이 위상정렬된 순서로 매달려 있다.
```java
DFS-TS(v, R) {
  visited[v] <- YES;
  for each x adjacent to v do
    if (visited[x] = NO) then
      DFS-TS(x, R);
  add v at the front of the linked list R;
}
```

* 위상정렬 알고리즘2 동작 예시
  * 정점의 순서는 상관 없으므로 맨처음에 DFS를 호출하는 노드로 '수프넣기'를 골랐다고 가정
  * DFS를 통해서 '계란 풀어넣기'로 이동한다.
  * 이동했는데 해당 노드에서 갈 수 있는 인접한 노드가 없기 때문에
  * 위의 코드에서 DFS-TS()가 recursive하게 다시 호출되는 for문을 빠져나와서
  * 연결리스트 R의 맨 앞에 ''계란 풀어넣기'' 노드를 삽입한다.
  * 삽입하고 나서 recursion을 빠져나오면 다시 '수프넣기' 노드로 돌아온다.
  * '계란 풀어넣기'노드는 이미 visited된 상태이고, 이제 '수프넣기' 노드에서 갈 수 있는 인접노드가 없으므로
  * `add v at the front of the linked list R;` 문장에 도달하게 되므로 연결리스트의 맨 앞에 노드를 추가한다.
  * '수프넣기' 노드까지  R에 넣고나면, resursion이 끝나므로, topologicalSort2()의 for문으로 돌아가서 다음 노드를 방문한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_18.png?raw=true)

* '냄비에 물붓기'노드에서부터 다시 위에서 했던 작업을 반복한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_19.png?raw=true)

* 마지막 노드까지 연결리스트에 추가해주면 위상정렬 알고리즘은 끝난다.
* 연결리스트 R에는 위상정렬된 노드들이 들어가있다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_20.png?raw=true)

## 정리

* 위상정렬 알고리즘 1은 위상정렬에서 제일 먼저 올 노드를 찾는 알고리즘이고,
* 2는 마지막에 올 노드를 찾아서 정렬하는 알고리즘이다.(연결리스트의 맨 앞에 노드를 추가)
* 알고리즘 2에서 DFS-TS()결과 더 이상 갈 노드가 없는 상태라면 해당 노드는 outgoing 엣지 자체가 없다는 의미이고, 그뜻은 위상정렬에서 맨 마지막에 와도 된다는 의미이다.(이미 R에 추가된 노드를 제외하고 맨 마지막)

