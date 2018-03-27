> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 5-2. Red-Black Tree 02

## INSERT

- 보통의 BST에서처럼 노드를 INSERT한다.
- 새로운 노드 z를 red노드로 한다.
- RB-INSERT-FIXUP을 호출한다.
- 1 : 또다른 포인터 y를 사용해서 y가 x의 한칸 뒤를 쫓아 내려오록 해야 새로운 노드를 insert할 자리를 관리할 수 있다. 레드블랙트리에서 특수하게 NIL 노드를 사용하지만, 실제로 구현할때는 null이 된다.
- 3 - 7 : 새로운 노드 z가 들어갈 자리를 찾고,
- 8 : z의 parent를 y로 설정해준다.
- 9 - 10 : 예외적인 경우 y가 null이면, 새로운 노드 z는 이 트리의 루트 노드가 된다.
- 11 - 13 : y노드의 키값과 새로운 노드의 키값을 비교하여, y의 왼쪽 자식일지 오른쪽 자식일지 결정한다.
- 14 - 15 : 새로운 노드는 BST의 leaf 노드가 되기 때문에, 새로운 노드의 자식들을 null로 설정한다.
- 16 : 새로운 노드는 RED노드로 설정한다.
- 17 : RB-INSERT-FIXUP(T, z)메소드를 호출한다.
  - 새로운 노드 z의 부모노드가 RED일 경우 레드블랙트리의 요건을 만족하지 않기 때문에, 조정이 필요하다.

```java
RB-INSERT(T, z)
1  y <- nil[T]
2  x <- root[T]
3  while x != nil[T]
4    do y <- x
5      if key[z] < key[x]
6        then x <- left[x]
7        else x <- right[x]
8  p[z] <- y
9  if y = nil[T]
10   then root[T] <- z
11   else if key[z] < key[y]
12          then left[y] <- z
13          else right[y] <- z
14 left[z] <- nil[T]
15 right[z] <- nil[T]
16 color[z] <- RED
17 RB-INSERT-FIXUP(T, z)
```

## RB-INSERT-FIXUP

- 레드블랙트리의 규칙이 위반될 가능성이 있는 조건들
  - 각 노드는 red 혹은 black이다.
    - **위반 가능성 없음**
  - 루트노드는 black이다.
    - 새로운 노드 z가 루트 노드라면 규칙 위반, 아니라면 위반 가능성 없음
    - 새로운 노드 z가 루트 노드라면 간단하게 노드를 블랙으로 바꾸는 작업을 해주면 된다.
  - 모든 리프노드(즉, NIL 노드)는 black이다.
    - 새로운 노드의 자식들을 NIL노드로 설정하므로 **위반 가능성 없음**
  - red노드의 자식노드들은 전부 black이다.(즉, red노드는 연속되어 등장하지 않고)
    - **위반 될 가능성이 있다**. 부모 노드가 원래 RED였다면, RED - RED 가 되므로 위반이 발생한다.
    - 가장 큰 문제가 되는 것이 RED - RED 조건을 위반 하는 것이다.
  - 모든 노드에 대해서 그 노드로 부터 자손인 리프노드에 이르는 모든 경로에는 동일한 개수의 black노드가 존재한다.
    - 새로운 노드를 RED로 추가 했으므로, **위반 가능성 없음**

#### Loop Invariant (루프를 돌면서 변하지 않고 유지되는 조건) :

- z는 red 노드
- 오직 하나의 위반만이 존재한다.
  - 조건 2 : z가 루트노드이면서 red이거나, 또는
  - 조건 4 : z와 그 부모 p[z]가 둘 다 red 이거나.
  - 조건 4를 해결하기 위해 부모노드를 타고 올라가다 보면, 또다른 RED - RED 위반이 있을 수 있다. 최악의 경우 루트노드까지 타고올라가게 되면, 조건 2를 위반 한 것이므로 루트노드를 블랙으로 바꿔주고 종료하면 된다.

#### 종료조건 :

- 부모노드 p[z]가 black이되면 종료한다. 조건2가 위반일 경우 z를 블랙으로 바꿔주고 종료한다.

## 문제 정의

총 6가지 case로 이 문제를 정의한다.

p[p[z]]가 반드시 존재하는 이유는 RED - RED 위반이 발생한 경우, p[z]가 RED이므로 레드블랙트리의 조건에 의해 부모인 p[p[z]]는 **블랙** 노드로 존재할 수 밖에 없다.

- 1, 2, 3 Case는 p[z]가 p[p[z]]의 왼쪽 자식인 경우
- 4, 5, 6 Case는 p[z]가 p[p[z]]의 오른쪽 자식인 경우
- Case 1, 2, 3과 4, 5, 6은 대칭적이기 때문에, 코드상에서 left와 right만 바꿔주면 된다.
- Case 1, 2, 3에 대해 집중적으로 알아본다.

### Case 1 : z의 부모의 형제가 RED인 경우

- 새로운 노드 z의 부모가 RED이면서, 부모의 형제 노드 D가 RED인 경우이다.
- (a) - 오른쪽 자식, (b) - 왼쪽 자식
- 이 경우 부모노드와 부모노드의 형제노드를 BLACK으로 바꾸고, 할아버지 노드를 RED로 바꾼다.
- 그리고 나서, 할아버지 노드 C를 새로운 노드 z로 설정하여 위로 타고 올라가면서 레드 블랙 트리의 조건을 위반하는지를 검사한다.
- Case 1의 경우 문제가 완전히 해결되지는 않았다. z가 두칸 위로 올라가서 그 위치에서 부터 다시 해결해야 한다.

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/red_black_04.png?raw=true)

### Case 2, 3 : z의 부모의 형제가 BLACK인 경우

- z의 부모의 형제가 BLACK인 경우 실제로 BLACK 노드일 수도 있고, NIL노드일 수도 있다. 따라서, 그림에서 노드의 형태로 표현하지 않았다.

- Case 2 : z가 오른쪽 자식인 경우

  - p[z]에 대해서 left-rotation한 후 원래 p[z]를 z로 만든다.
  - Case 3으로 이동

- Case 3 : z가 왼쪽 자식인 경우

  - p[p[z]]에 대해서 right-rotation을 한다.
  - p[z]를 BLACK, p[p[z]]를 RED로 바꾼다.

- **Case 1, 2, 3 정리**

  - Case 1의 문제를 해결하고 나면 문제는 종료되지 않는다. Case 2로 갈수도 있고, Case 3로 갈수도 있으며, 다시 Case 1의 문제가 반복해서 발생할 수 있다. 최악의 경우 루트노드까지 올라가서 레드블랙트리의 조건 2 루트 노드가 레드인 경우를 블랙으로 변경하면서 종료하게 된다.
  - Case 2는 발생하면, Case 3을 거쳐서 해결하면 종료된다.
  - Case 3의 경우 z의 할아버지 노드를 기준으로 right-rotate하면 문제가 해결되고, 종료 된다.
  - Case 1, 2, 3의 경우에 해당하는 것이고, Case 1에서 Case 4, 5, 6으로 넘어갈 수도 있다.

  ![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/red_black_05.png?raw=true)

### Case 4, 5, 6

* Case 1, 2, 3은 p[z]가 p[p[z]]의 왼쪽 자식인 경우들
* Case 4, 5, 6은 p[z]가 p[p[z]]의 오른쪽 자식인 경우들
  * Case 1, 2, 3과 대칭적이므로 생략
  * 결론적으로 Case 1, 2, 3의 정리가 각각 Case 4, 5, 6으로 매칭된다.
  * Case 4의 문제를 해결하고 나면 문제가 해결된 것이 아니고, Case 4로 다시, 또는 Case 5, 6으로, Case 1, 2, 3으로 넘어갈 수도 있다. 물론 2칸씩 올라가면서 해결하게 된다.
  * Case 5의 경우 Case 6을 거쳐서 해결하면 종료된다.

## RB-INSERT-FIXUP Code

* pesudo code
  * z는 새로운 노드
  * 1 : z의 부모가 RED인 동안 RED - RED 위반이 존재하므로 while문 수행

```java
RB-INSERT-FIXUP(T, z)
1  while color[p[z]] = RED
2    do if p[z] = left[p[p[z]]]
3        then y <- right[p[p[z]]]
4          if color[y] = RED
5            then color[p[z]] <- BLACK   >> CASE 1
6                 color[y] <- BLACK      >> CASE 1
7                 color[p[p[z]]] <- RED  >> CASE 1
8                 z <- p[p[z]]           >> CASE 1
9          else if z = right[p[z]]
10                then z <- p[z]         >> CASE 2
11                     LEFT-ROTATE(T, z) >> CASE 2
12              color[p[z]] <- BLACK     >> CASE 3
13              color[p[p[z]]] <- RED    >> CASE 3
14              RIGHT-ROTATE(T, p[p[z]]) >> CASE 3
15      else(same as then clause with "right" and "left" exchanged)
16 color[root[T]] <- BLACK
```



