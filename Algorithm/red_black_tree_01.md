> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 5-1. Red-Black Tree 01 - 개요

* 앞서 배운 BST의 search, insert, delete 세 가지 연산모두 O(h)의 시간복잡도를 가진다. (h는 트리의 높이)
* 여기서 문제는, 트리의 높이 h가 최악의 경우 n이 될 수 있다. n은 노드의 총 수.
* 그러나, 이것은 실제 최악의 경우에 해당한다. BST에 데이터들이 random하게 구성된다고 가정했을때, 평균 트리의 높이는 O(logn)이 된다.
* 그럼에도 불구하고, 최악의 경우 O(n)이 되는 경우는 좋은 알고리즘은 아니다. 그리고, 현실에서 이미 정렬된 데이터가 BST에 insert된다면, 최악의 경우에 해당할 가능성이 커진다.
* 따라서, BST의 기본적인 아이디어를 유지하면서 Tree의 밸런스가 무너지지 않게 해서, 최악의 경우에도 O(logn)의 시간복잡도를 갖게 하기 위한 대표적인 알고리즘의 예시가 레드블랙트리이다.


## Red-Black Tree

* Binary Search Tree의 일종
* 균형잡힌 트리 : 높이가 O(logn)
* Search, Insert, Delete 연산을 최악의 경우에도 O(logn)시간에 지원
  * insert, delete 알고리즘을 수정하여 트리의 균형이 잡히도록 한다.
* 각 노드는 하나의 키(Key), 왼쪽자식(Left), 오른쪽 자식(Right), 그리고 부모노드(p)의 주소를 저장
* 자식노드가 존재하지 않을 경우 NIL 노드라고 부르는 특수한 노드가 있다고 가정한다.
* 따라서 모든 리프노드는 NIL 노드이다.
* 노드들은 내부노드와 NIL 노드로 분류한다.
* 실제로 구현을 할 때, NIL 노드를 포함하지 않고, 개념적인 설명을 좀 더 편하게 하기 위해서 가상적으로 NIL 노드를 사용한다.
* 개념적으로는 (b)의 그림처럼 생각하지만, 실제 구현은 (c)가 될 것이다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/red_black_01.png?raw=true)

  

### 정의

* 다음의 조건을 만족하는 이진탐색트리

  1. 각 노드는 red 혹은 black이고,
  2. 루트노드는 black이고,
  3. 모든 리프노드(즉, NIL 노드)는 black이고,
  4. red노드의 자식노드들은 전부 black이고(즉, red노드는 연속되어 등장하지 않고),
  5. 모든 노드에 대해서 그 노드로 부터 자손인 리프노드에 이르는 모든 경로에는 동일한 개수의 black노드가 존재한다.

  ![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/red_black_02.png?raw=true)

### Red-Black Tree의 높이

* 노드 x의 높이 h(x)는 자신으로부터 리프노드(NIL 노드)까지의 가장 긴 경로에 포함된 엣지의 개수이다.
* 노드 x의 블랙-높이 bh(x)는 x로부터 리프노드(NIL 노드)까지의 경로상의 블랙노드의 개수이다. (노드 x 자신은 불포함)
* 높이가 h인 노드의 블랙-높이는 bh >= (h/2) 이다.
  * 조건 4에 의해 레드노드는 연속될 수 없으므로 당연히 블랙노드의 수가 높이의 절반 이상이다.
* 노드 x를 루트로하는 임의의 부트리는 적어도 2<sup>bh(x)</sup>-1개의 내부노드를 포함한다.(수학적귀납법)
* 위의 두가지 증명으로 설명 가능한 것은, n개의 내부노드를 가지는 레드블랙트리의 높이는 2log(n+1) 이하이다.
  * n >= 2<sup>bh</sup>-1 >= 2<sup>h/2</sup>-1 이므로, 여기서 bh와 h는 각각 루트 노드의 블랙-높이와 높이
* **정의에서 설명하는 5가지 조건을 만족하는 레드블랙트리라면 자동으로 높이는 O(logn)이 된다.**


레드블랙트리에서의 Search 알고리즘은 BST에서와 같기 때문에, insert와 delete연산에 집중한다.

### Left and Right Rotation

* Insert와 Delete가 모두 필요로하는 Left, Right Rotation을 알아본다.
* 한 노드를 중심으로 부분적으로 트리의 모양을 수정하는 연산이다.
* 시간복잡도는 O(1)이다.
* 이진탐색트리의 특성을 유지한다.
  * Rotate를 해서 서브트리(B)가 옮겨지더라도, BST의 특성은 유지된다. 
* x와 y의 자식들은 모두 서브트리이다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/red_black_03.png?raw=true)

### Left Rotation

* y = right[x] =! NIL이라고 가정
* 루트노드의 부모도 NIL이라고 가정
* pseudo code
  * 01 - x의 오른쪽 자식노드를 y에 저장한다.
  * 02 - x의 오른쪽 자식노드를 B로 만든다.
  * 03 - B의 부모노드를 x로 만드는 link를 연결한다.
  * 04 - y의 부모노드를 현재 x부모노드로 할당한다.
  * 05 - x의 부모노드가 NIL 이라면 즉, x가 루트라면
  * 06 - y를 루트로 설정한다.
  * 07 - 그렇지 않고 x의 부모노드가 존재한다면, 부모노드의 왼쪽 자식이었다면,
  * 08 - y가 x부모노드의 왼쪽 자식노드가 되고
  * 09 - 그렇지 않다면, y는 x의 부모노드의 오른쪽 자식이 된다.
  * 10 - x가 y의 왼쪽 자식이 되게하고,
  * 11 - y가 x의 부모노드가 되도록 한다.
* 차례대로 수행하면 되므로, 시간복잡도는 O(1)이다.

```java
LEFT-ROTATE(T, x)
01  y <- right[x]        //Set y
02  right[x] <- left[y]  //Turn y's left subtree into x's right 
03  p[left[y]] <- x
04  p[y] <- p[x]         //Link x's parent to y
05  if p[x] = NIL[T]
06    then root[T] <- y
07    else if x = left[p[x]]
08      then left[p[x]] <- y
09      else right[p[x]] <- y
10  left[y] <- x         //Put x on y's left
11  p[x] <- y
```


