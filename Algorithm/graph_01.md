> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 7-1. Graph Algorithm 01 - 개념과 표현

## Graph

* (무방향) 그래프 G = (V, E)
  * V : 노드 혹은 정점(vertex)
  * E : 노드쌍을 연결하는 Edge 혹은 Link
  * Object들 간의 이진관계를 표현
  * n = |V|, m = |E|

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_01.png?raw=true)

* 방향 그래프와 가중치 그래프
  * 방향그래프(Directed Graph) G = (V, E)
  * Edge (u, v)는 u로부터 v로의 방향을 가짐
* 가중치 그래프
  * Edge마다 가중치(weight)가 존재

### 그래프의 표현

* 인접행렬(adjacency matrix)

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_02.png?raw=true)

* 인접리스트(adjacency list)

  * 정점 집합을 표현하는 하나의 배열과
  * 각 정점마다 인접한 정점들의 연결 리스트

  ![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_03.png?raw=true)

  * 저장 공간 : O(n + m)
  * 어떤 노드 v에 인접한 모든 노드 찾기 : O(degree(v)) 시간
  * 어떤 Edge (u, v)가 존재하는지 검사 : O(degree(u)) 시간

* 방향그래프(directed graph)

  * 인접행렬은 비대칭
  * 인접 리스트는 m개의 노드를 가진다

  ![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_04.png?raw=true)

* 가중치 그래프의 인접행렬 표현

  * 엣지의 존재를 나타내는 값으로 1대신 엣지의 가중치를 저장
  * 엣지가 없을 때 혹은 대각선
    * 그때마다 상황에 따라 정의 가능
    * 특별히 정해진 규칙은 없으며, 그래프와 가중치가 의마하는 바에 따라서
    * Ex) 가중치가 거리 혹은 비용을 의미하는 경우라면 엣지가 없으면 oo(무한대), 대각선은 0
    * Ex) 만약 가중치가 용량을 의미한다면 엣지가 없을때 0, 대각선은 oo(무한대)

### 경로와 연결성

* 인접하다라는 것은 해당 경로를 거쳐서 그 노드에 도달할 수 있다라는 것이고, 연결되어 있다라는 것은 노드와 노드를 연결하는 경로가 존재할 때를 말한다.
* 무방향 그래프 G = (V, E)에서 노드 u와 노드 v를 연결하는 경로(path)가 존재할 때 v와 u는 서로 연결되어 있다고 말함
* 모든 노드 쌍들이 서로 연결된 그래프를 연결된(connected) 그래프라고 한다.
* 연결요소(connected component)

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_05.png?raw=true)

