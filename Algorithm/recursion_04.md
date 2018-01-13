# Recursion의 응용 1 - 미로찾기

## Maze - 미로찾기

- (n-1, n-1)의 좌표를 출구로 가정
- 흰색이 지날 수 있는 길, 파란색이 벽
- 입구에서 출구까지의 경로를 찾는 문제

![](https://github.com/namjunemy/TIL/blob/master/Algorithm/img/maze_01.png?raw=true)

### Recursive Thinking

* 현재 위치에서 출구까지 가는 경로가 있으려면
  * 현재 위치가 출구이거나(이미 내가 출구에 와 있거나). 혹은,
  * **이웃한 셀들 중 하나에서 현재 위치를 지나지 않고 출구까지 가는 경로**가 있거나.
    * 위의 경우를 Recursive하게 생각한다. 전체 문제를 해결하려고 하면 부분 문제의 해결이 이루어 지면서 전체 문제가 해결된다.
  * 위의 둘중에 하나가 성립해야 출구까지 가는 경로가 있다고 할 수 있다.

### 미로찾기(Decision Problem) - 답이 Yes or No인 문제

* 출발점에서 출구까지 가능 경로가 있느냐 없느냐? 의 문제로 생각한다.
* x', y'는 현재 셀과 이웃한 셀들
  * x, y의 x', y'(4개)를 각각 봤을 때, pathway가 없다면 출구까지 가는 경로가 없다.
* recursion을 고민할 때, 가장 먼저 고민할 것은 무한 루프에 빠지지 않는가 이다.
  * Base case로 수렴하는가를 확인해야한다.
  * 현재 아래 코드에서는 pathway인 인접한 두 셀끼리의 무한루프가 발생할 수 있다.(서로가 서로의 x', y'가 될 수 있으므로)

```java
boolean findPath(x, y)
  if (x, y) is the exit
    return true;
  else
    for each neighbouring cell (x', y') of (x, y) do
      if (x', y') is on the pathway
        if findPath(x', y')
          return true;
    return false;
```

* 따라서, x, y의 상황에서 고려할 때 x', y'에서 다시 x, y로 돌아오는 것을 막아줄 필요가 있다.

  * 그것을 방문한 위치와 방문하지 않은 위치로 구분하여 처리한다.
  * 아래와 같이 코드가 변경될 수 있다.
  * 조건문에서 pathway체크와 함께 방문하지 않은 셀의 여부를 체크하기 때문에 무한 루프에 빠지지 않는다.

  ```java
  boolean findPath(x, y)
    if (x, y) is the exit
      return true;
    else
      mark (x, y) as a visited cell;
      for each neighbouring cell (x', y') of (x, y) do
        if (x', y') is on the pathway and not visited
          if findPath(x', y')
            return true;
      return false;
  ```

* 다른 방법으로 Recursion 호출(findPath(x', y'))을 하기전에 pathway체크와 방문여부를 체크하지 않고, 메소드의 시작지점에서 pathway체크와 방문여부를 체크하여 false를 리턴하는 방법도 있다.

  * 이렇게 하면, recursion이 호출되는 횟수는 더 많아 지지만 코드가 간결해지는 면이 있다.

```java
boolean findPath(x, y)
  if (x, y) is either on the wall or a visited cell
    return false;
  else if (x, y) is the exit
    return true;
  else
    mark (x, y) as a visited cell;
    for each neighbouring cell (x', y') of (x, y) do
      if findPath(x', y')
        return true;
    return false;
```

### Maze - Java 구현

* PATH_COLOR : visited이며 아직 출구로 가는 경로가 될 가능성이 있는 cell
* BLOCKED_COLOR : visited이며 출구까지의 경로상에 있지 않음이 밝혀진 cell
* 일단 방문하면 PATH COLOR로 칠하고, 출구까지의 경로상에 있지 않음이 밝혀질 경우 BLOCKED_COLOR로 칠한다.

```java
public class Maze {
  private static final int PATHWAY_COLOR = 0;
  private static final int WALL_COLOR = 1;
  private static final int BLOCKED_COLOR = 2;
  private static final int PATH_COLOR = 3;
  private static int N = 8;
  private static int[][] maze = {
      {0, 0, 0, 0, 0, 0, 0, 1},
      {0, 1, 1, 0, 1, 1, 0, 1},
      {0, 0, 0, 1, 0, 0, 0, 1},
      {0, 1, 0, 0, 1, 1, 0, 0},
      {0, 1, 1, 1, 0, 0, 1, 1},
      {0, 1, 0, 0, 0, 1, 0, 1},
      {0, 0, 0, 1, 0, 0, 0, 1},
      {0, 1, 1, 1, 0, 1, 0, 0}
  };

  public static boolean findMazePath(int x, int y) {
    if (x < 0 || y < 0 || x >= N || y >= N)
      return false;
    else if (maze[x][y] != PATHWAY_COLOR)
      return false;
    else if (x == N - 1 && y == N - 1) {
      maze[x][y] = PATH_COLOR;
      return true;
    } else {
      maze[x][y] = PATH_COLOR;
      if (findMazePath(x - 1, y) || findMazePath(x, y + 1)
          || findMazePath(x + 1, y) || findMazePath(x, y - 1)) {
        return true;
      }
      maze[x][y] = BLOCKED_COLOR;
      return false;
    }
  }

  public static void main(String[] args) {
    printMaze();
    findMazePath(0, 0);
    System.out.println();
    printMaze();
  }

  private static void printMaze() {
    for (int x = 0; x < N; x++) {
      System.out.print("{");
      for (int y = 0; y < N; y++)
        System.out.print(maze[x][y] + ", ");
      System.out.println("}");
    }
  }
}
```

```java
{0, 0, 0, 0, 0, 0, 0, 1, }
{0, 1, 1, 0, 1, 1, 0, 1, }
{0, 0, 0, 1, 0, 0, 0, 1, }
{0, 1, 0, 0, 1, 1, 0, 0, }
{0, 1, 1, 1, 0, 0, 1, 1, }
{0, 1, 0, 0, 0, 1, 0, 1, }
{0, 0, 0, 1, 0, 0, 0, 1, }
{0, 1, 1, 1, 0, 1, 0, 0, }

{3, 2, 2, 2, 2, 2, 2, 1, }
{3, 1, 1, 2, 1, 1, 2, 1, }
{3, 2, 2, 1, 2, 2, 2, 1, }
{3, 1, 2, 2, 1, 1, 2, 2, }
{3, 1, 1, 1, 2, 2, 1, 1, }
{3, 1, 3, 3, 3, 1, 2, 1, }
{3, 3, 3, 1, 3, 3, 3, 1, }
{0, 1, 1, 1, 0, 1, 3, 3, }

Process finished with exit code 0
```
