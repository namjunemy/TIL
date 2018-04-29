> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 7-3. Graph 03 - DFS(Depth-First Search, 깊이우선순회)

- 이진트리의 순회 방법인 inorder, preorder, postorder 순회방법이 DFS의 이진트리 버전에 해당한다.
- lever order는 BFS의 이진트리 순회 버전이다.

## 깊이우선순회(DFS)

* 출발점 s에서 시작한다.
* 현재 노드를 visited로 mark하고 인접한 노드들 중 unvisited 노드가 존재하면 그 노드로 간다.
* 2번을 계속 반복한다.
* 노드 8에 도달했을 때처럼 인접한 노드들중 invisited 노드가 존재하지 않는다면, unvisited인 이웃 노드가 존재하지 않는 동안 계속해서 직전 노드로 되돌아간다.
* 다시 2번을 계속 반복한다.
* 시작노드 s로 돌아오고 더 이상 갈 곳이 없으면 종료한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_12.png?raw=true)

* 다음과 같은 흐름으로 깊이우선순회가 이루어 진다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/graph_13.png?raw=true)

### DFS, 깊이우선탐색

* 이진트리의 순회를 recursion으로 구현한 것처럼, 깊이우선탐색도 resursion으로 구현하는 것이 간명하다.
* 01 : 먼저 방문한 노드 v에 대해서 visited 체크를 하고
* 02 : v와 인접한 노드 x들에 대해서
* 03 : visited[x]가 No 이면,
* 04 : DFS(G, x)를 recursive하게 호출한다.
* 순회를 위해서 직전의 노드로 돌아가는 행동이 recursion으로 간명하게 구현된다.

```java
00 DFS(G, v)
01   visited[v] <- YES;
02   for each node x adjacent to v do
03     if visited[x] = No then
04       DFS(G, x);
```

* 그래프가 **disconnected 그래프 이거나 혹은 방향 그래프**라면 DFS에 의해서 모든 노드가 방문되지 않을 수도 있음
* DFS를 반복하여 모든 노드 방문
  * 모든 노드의 visited를 NO로 설정하고,
  * 해당 노드들을 출발노드로 하여 DFS를 호출, 연결되지 않은 그래프에 해당하는 노드는 visited가 NO로 유지되어서 DFS를 호출하게됨
* 시간복잡도
  * 첫번째 for문에 의해서 시간복잡도 O(n)은 피할 수 없고,
  * 두번째 for문에 의해서 v노드와 엣지로 이어진 노드가 visited인지를 체크한다. 따라서, 인접리스트로 표현했다면 시간복잡도는 엣지의 갯수에 비례하게 된다. O(m)
  * 최종적으로 O(n + m)의 시간복잡도를 갖는다.
  * 만약, 인접행렬로 표현했다면 인접노드의 여부를 알기위해서 전체 노드의 수 만큼 검색해야 하므로 O(n)이므로, O(n^2)의 시간복잡도를 갖는다.

```java
DFS-ALL(G)
  for each v in V
    visited[v] <- NO;
  for each v in V
    if (visited[v] = no) then
      DFS(G, v)
```