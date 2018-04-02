> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 5-2. Red-Black Tree 03 - DELETE

## DELETE

* 보통의 BST에서처럼 DELETE한다.
* 실제로 삭제된 노드 y가 red였으면 종료
* y가 black이었을 경우 RB-DELETE-FIXUP을 호출한다.
* pesudo code
  * 01 : 삭제할 노드 z의 왼쪽, 오른쪽 자식이 NULL이 아니라면 즉, 자식이 없다면
  * 02 : y에 z를 저장하고
  * 03 : 자식이 있다면, z의 Successor를 찾아서 y에 저장한다. BST에서 Successor찾는 과정과 동일
  * 04 : 01-03과정을 거치면 y는 자식이 하나이거나 없다. Successor는 자식노드가 두 개일 수 없다. y의 왼쪽 자식이 null이 아니라면
  * 05 : y의 왼쪽 자식을 x로
  * 06 : 그렇지 않다면, y의 오른쪽 자식을 x로 저장한다.
  * 07 : 그다음 y의 부모를 x의 부모로 설정한다.
  * 08 : 만약, y의 부모가 NULL이라면
  * 09 : x를 트리의 root로 설정한다.
  * 10 : y의 부모노드가 NULL이 아니고, y가 y의 부모의 왼쪽 자식노드라면
  * 11 : 저장된 x를 y의 부모의 왼쪽 자식노드로 설정한다.
  * 12 : y가 y의 부모의 오른쪽 자식노드라면, 저장된 x를 부모노드의 오른쪽 자식으로 설정한다. 여기까지가 실제로 노드를 삭제하는 작업이다.
  * 13 : 삭제하려는 노드 대신 Successor를 삭제한 경우 y와 z가 다르다. 따라서 Successor y노드의 데이터를 카피해주는 작업이 필요하다.
  * 14 - 15 : y의 데이터들을 z노드로 copy해주는 작업
  * 15번 라인까지의 작업은 BST의 DELETE작업이다. 아래의 3줄은 RBT의 경우에 해줘야하는 연산이다.
  * **삭제한 노드 y가 red 노드 였다면, RBT의 규칙에 위반 되지 않는다.**
  * 16 : 삭제한 노드 y가 BLACK 노드일 경우
    * 삭제한 노드가 루트노드인데, 그 자식인 레드노드가 올라와서 루트가 된 경우,
    * 중간의 black노드가 삭제되억서 red - red 위반이 생긴 경우,
    * 트리를 따라 내려가면서 black노드의 갯수가 일치하지 않는 경우가 생길 수 있다.
  * 17 : 문제들을 해결하기 위해 RBT-DELETE-FIXUP(T, x)를 통해서 트리의 규칙을 만족하기 위한 작업을 한다. y의 자식인 x를 넘겨서 정렬한다.
  * 18 : 삭제된 노드 y를 반환한다.

```java
RB-DELETE(T, z)
01 if left[z] = nil[T] or right[z] = nil[T]
02   then y <- z
03   else y <- TREE-SUCCESSOR(z)
04 if left[y] != nil[T]
05   then x <- left[y]
06   else x <- right[y]
07 p[x] <- p[y]
08 if p[y] = nil[T]
09   then root[T] <- x
10   else if y = left[p[y]]
11          then left[p[y]] <- x
12          else right[p[y]] <- x
13 if y != z
14   then key[z] <- key[y]
15     copy y's satellite data into z
16 if color[y] = BLACK
17   then RB-DELETE-FIXUP(T, x)
18 return y    
```

