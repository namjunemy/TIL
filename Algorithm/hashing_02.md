> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 6-2. Hashing 02

## 좋은 해시 함수란?

* 현실에서는 키들이 랜덤하지 않음
* 만약 키들의 통계적 분포에 대해 알고 있다면 이를 이용해서 해시 함수를 고안하는 것이 가능하겠지만 현실적으로 어려움
* 키들이 어떤 특정한 (가시적인) 패턴을 가지더라도 해시함수값이 불규칙적이 되도록 하는게 바람직하다.
  * 해시함수값이 키의 특정 부분에 의해서만 결정되지 않아야 함

## 해시 함수

* Division 기법
  * h(k) = k mod m
  * 예: m = 20 and k = 91 ==> h(k) = 11
  * 장점: 한번의 mod연산으로 계산, 따라서 빠름
  * 단점: 어떤 m값에 대해서는 해시 함수값이 키값의 특정 부분에 의해서 결정되는 경우가 있음, 가령 m = 2^p 이면 키의 하위 p비트가 해시 함수값이 됨
* Multiplication 기법
  * 0에서 1사이의 상수 A를 선택: 0 < A < 1
  * kA의 소수부분만을 택한다
  * 소수 부분에 m을 곱한 후 소수점 아래를 버린다.
  * 예 : m=8, word size = w = 5, k = 21
  * A = 13/32를 선택
  * kA = 21*13/32 = 273/32 = 8 + 17/32
  * m (kA mod 1) = 8 * 17/32 = 17/4 = 4.xxx
  * 즉, h(21) = 4
* 꼭 위의 기법을 사용하여 해시 함수를 만들어야 한다는 것은 아니다. 해시 함수를 만드는 방법들 중 하나일 뿐이다.

## Hashing in Java

* Java의 Object 클래스는 hashCode() 메서드를 가짐, 따라서 모든 클래스는 hashCode() 메서드를 상속받는다. 이 메서드는 하나의 32비트 정수를 반환한다. 32비트 정수라는 것은 음수일 수도 있다는 이야기이다.
* 만약 x.equals(y)이면 x.hashCode() == y.hashCode() 이다. 하지만 역은 성립하지 않는다.
* Object 클래스의 hashCode() 메서드는 객체의 메모리 주소를 반환하는것으로 알려져 있음(but it's implementation-dependent.)
* 필요에 따라 각 클래스마다 이 메서드를 override하여 사용한다.
  * 예) Integer 클래스는 정수값을 hashCode로 사용

### 해시함수의 예 : hashCode() for Strings in Java

```java
public final class String {
  private final char[] s;
  ...
  public int hashCode() {
    int hash = 0;
    for (int i = 0; i < length(); i++)
      hash = s[i] + (31 * hash);
    return hash;
  }
}
```

### 사용자 정의 클래스의 예

* 모든 멤버들을 사용하여 hashCode를 생성한다.

```java
public class Record {
  private String name;
  private int id;
  private double value;
  ...
  public int hashCode() {
    int hash = 17;  //nonzero constant
    hash = 31 * hash + name.hachCode();
    hash = 31 * hash + Integer.valueOf(id).hashCode();
    hash = 31 * hash + Double.valueOf(value).hashCode();
    return hash;
  }
}
```

### hashCode와 hash 함수

* Hash code : -2^31에서 2^31사이의 정수
* Hash 함수 : 0에서 M-1까지의 정수 (배열 인덱스)
  * 0x7fffffff을 &연산 하는 이유는 음수일 경우 양수로 바꿔주는 작업이다.
  * 나머지 연산의 피연산자가 양수여야 하기 때문이다.

```java
private int hash(Key key) {
  return (key.hashCode() & 0x7fffffff) % M;
}
```

### HashMap in Java

* TreeMap 클래스와 유사한 인터페이스를 제공(둘 다 java.util.Map 인터페이스를 구현)
* 내부적으로 하나의 배열을 해시 테이블로 사용
* 해시 함수는 24페이지의 것과 유사함
* chaining으로 충돌 해결
* Load factor를 지정할 수 있음(0 ~ 1 사이의 실수)
* 저장된 키의 개수가 load factor를 초과하면 더 큰 배열을 할당하고 저장된 키들을 재배치(re-hashing)

### HashSet in Java

```java
HashSet<MyKey> set = new HashSet<MyKey>();
set.add(MyKey);
if (set.contains(theKey))
  ...
int k = set.size();
set.remove(theKey);
Iterator<MyKey> it = set.iterator();
while (it.hasNext()) {
  MyKey key = it.next();
  if (key.equals(aKey))
    it.remove();
}
```

