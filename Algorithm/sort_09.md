> 인프런 - 부경대IT융합응용공학과 궘오흠 교수님의 '**영리한 프로그래밍을 위한 알고리즘 강좌** '([링크](https://www.inflearn.com/course/%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%EA%B0%95%EC%A2%8C/))와 '**쉽게 배우는 알고리즘** 관계 중심의 사고법 - 문병로' 참조

# 3-9. Sorting in Java

* 일반적으로 정렬은 가장 기본적인 알고리즘이기 때문에, 대부분의 프로그래밍 언어가 표준 라이브러리의 일부로 정렬을 제공한다.
* 따라서, 일반적인 상황에서 개발자가 직접 알고리즘을 구현할 경우는 많지 않다고 볼 수 있다.
* Java에서의 sorting을 알아본다.


## 기본 타입 데이터의 정렬

* Arrays 클래스가 primitive 타입 데이터를 위한 정렬 메소드를 제공한다.

```java
int[] data = new int[capacity];

//data[0]에서 data[capacity-1]까지 데이터가 꽉 차있는 경우에는 다음과 같이 정렬한다.
Arrays.sort(data);

//배열이 꽉 차있지 않고 size개의 데이터만 있다면 다음과 같이 정렬한다.
Arrays.sort(data, 0, size);
```

* int이외의 다른 primitive 타입 데이터(double, char 등)에 대해서도 제공


## 객체의 정렬: 문자열

* java.util.Arrays;

```java
String[] fruits = new String[]{"pineapple", "apple", "orange", "banana"};

Arrays.sort(fruits);
for(String name : fruits)
  System.out.println(name);
```

  

## ArrayList 정렬: 문자열

* java.util.Collections;

```java
List<String> fruits = new ArrayList<>();
fruits.add("pineapple");
fruits.add("apple");
fruits.add("orange");
fruits.add("banana");

Collections.sort(fruits);
fruits.forEach(System.out::println);
```

  

## 객체의 정렬: 사용자 정의 객체

* 정렬의 대상인 데이터들에 대해 객체들간의 크기 관계 비교 대상을 정의해야 한다.

```java
public class Fruit {
  private String name;
  private int quantity;
  public Fruit(String name, int quantity) {
    this.name = name;
    this.quantity = quantity;
  }
}
```

### Comparable Interface

* Comparable 인터페이스를 implements하고, compareTo 메소드를 오버라이드한다.
* 이름순으로 정렬 

```java
public class Fruit implements Comparable<Fruit> {
  public String name;
  public int quantity;

  public Fruit(String name, int quantity) {
    this.name = name;
    this.quantity = quantity;
  }

  @Override
  public int compareTo(Fruit other) {
    return name.compareTo(other.name);
  }

  @Override
  public String toString() {
    return name + " " + quantity;
  }
}
```

```java
public static void main(String[] args) {
    Fruit[] fruits = new Fruit[4];
    fruits[0] = new Fruit("pineapple", 70);
    fruits[1] = new Fruit("apple", 100);
    fruits[2] = new Fruit("orange", 80);
    fruits[3] = new Fruit("banana", 90);

    Arrays.sort(fruits);
    for(Fruit fruit : fruits)
      System.out.println(fruit);
  }
```

```java
apple 100
banana 90
orange 80
pineapple 70

Process finished with exit code 0
```

* 재고 수량으로 정렬

```java
public class Fruit implements Comparable<Fruit> {
  public String name;
  public int quantity;

  public Fruit(String name, int quantity) {
    this.name = name;
    this.quantity = quantity;
  }

  @Override
  public int compareTo(Fruit other) {
    return quantity - other.quantity
  }

  @Override
  public String toString() {
    return name + " " + quantity;
  }
}
```

### 두가지 정렬을 동시에 제공하려면? Comparator

* 이름 순과 재고수량 순으로 정렬하는 compareTo 메소드는 동시에 가지고 있을 수 없다.
* 하나의 객체 타입에 대해서 2가지 이상의 기준으로 정렬을 지원하려면,
* Comparator를 사용해야 한다. 
* Comparator 인터페이스의 익명구현객체를 만들면서 compare 메소드를 Overriding한다.
  * 인터페이스 자체는 new를 통한 인스턴스 생성이 불가하지만,
  * anonymous Class. 즉, 익명의 클래스를 먼저 만들어서 내부에 compare 메소드를 정의 한 뒤, 해당 익명클래스의 인스턴스를 만든 것이다.
  * 익명클래스에 이름을 붙여 생성하지 않은 이유는, 일회성으로 사용될 것이기 때문이다.
* 비교할 때, comparator 인터페이스를 구현상속한 객체의 compare 메소드를 통해서 비교 후 정렬이 이루어진다.
* Collection에 있는 객체를 정렬하려면 Arrays만 Collections로 바꿔주면 된다.

```java
Comparator<Fruit> nameComparator = new Comparator<Fruit>() {
  public int compare(Fruit fruit1, Fruit fruit2) {
    return fruit1.name.compareTo(fruit2.name);
  }
}

Comparator<Fruit> quantityComparator = new Comparator<Fruit>() {
  public int compare(Fruit fruit1, Fruit fruit2) {
    return fruit1.quantity - fruit2.quantity;
  }
}

Arrays.sort(fruits, nameComparator);
또는
Arrays.sort(fruits, quantityComparator);
```

* 일반적으로 Comparator객체는 데이터 객체의 static member로 둔다. 정렬할 때는 Arrays.sort(fruits, Fruit.nameComparator);와 같이 호출한다.

```java
public class Fruit {
  private String name;
  private int quantity;
  public Fruit(String name, int quantity) {
    this.name = name;
    this.quantity = quantity;
  }
  
  public static Comparator<Fruit> nameComparator = new Comparator<Fruit>() {
    public int compare(Fruit fruit1, Fruit fruit2) {
      return fruit1.name.compareTo(fruit2.name);
    }
  }

  public static Comparator<Fruit> quantityComparator = new Comparator<Fruit>() {
    public int compare(Fruit fruit1, Fruit fruit2) {
      return fruit1.quantity - fruit2.quantity;
    }
  }
}
```

