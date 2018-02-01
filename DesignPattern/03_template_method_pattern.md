# 3. Template Method Pattern

> 출처 : Head First Design Patterns & 인프런 - '자바 디자인 패턴의 이해'(GoF Design Pattern)
>
> [Code Repository](https://github.com/namjunemy/design_pattern)

**"공통적인 프로세스를 묶어주기"**

### 학습목표

* 일정한 프로세스를 가진 요구사항을 템플릿 메소드 패턴을 이용하여 구현할 수 있다.

### Template Method Pattern

* 알고리즘의 구조를 메소드에 정의하고 하위 클래스에서 알고리즘 구조의 변경없이 알고리즘을 재정의 하는 패턴
  * 템플릿 메소드를 건드리지 않고 일부 기능 변경이 가능하다.
* 어떤 경우에 사용할까?
  * 구현하려는 알고리즘이 일정한 프로세스가 있다.
  * 구현하려는 알고리즘이 변경 가능성이 있다.
* 패턴의 구현 절차
  * 알고리즘을 여러 단계로 나눈다.
  * 나눠진 알고리즘의 단계를 메소드로 선언한다.
  * 알고리즘을 수행할 템플릿 메소드를 만들고, 나눠진 메소드들을 호출한다.
  * 하위 클래스에서 나눠진 메소드들을 구현한다.

### 기본 설계

* 알고리즘을 여러 단계로 나눈 뒤, 메소드로 선언한다.(operationX() 메소드)
* 템플릿 메소드를 만들고, 나눠진 메소드들을 호출한다.(templateMethod())
* 하위 클래스에서 나눠진 메소드들을 구현한다(Concrete Class)

![](https://github.com/namjunemy/TIL/blob/master/DesignPattern/img/template_method_01.png?raw=true)

  

## 요구 사항

* 신작 게임의 접속을 구현해주세요.
  * requestConnection(String str):String
* 유저가 게임 접속시 다음을 고려해야 합니다.
  * 보안 과정 : 보안 관련 부분을 처리합니다.
    * doSecurity(String string):String
  * 인증 과정 : user name과 password가 일치하는지 확인합니다.
    * authentication(String id, String password):boolean
  * 권한 과정 : 접속자가 유료회원인지 무료회원인지 게임 마스터인지 확인합니다.
    * authorization(String userName):int
  * 접속 과정 : 접속자에게 커넥션 정보를 넘겨줍니다.
    * connection(String info):String

### 구현

* AbstGameConnectionHelper에서 메소드 분리, 선언, 템플릿 메소드 생성 및 나눈 메소드 호출
* DefaultGameConnectionHelper에서 실제 메소드 구현

```java
package com.nj.library;

public abstract class AbstGameConnectionHelper {
  protected abstract String doSecurity(String string);

  protected abstract boolean authentication(String id, String password);

  protected abstract int authorization(String userName);

  protected abstract String connection(String info);

  public String requestConnection(String encodedInfo) {
    String decodedInfo = doSecurity(encodedInfo);

    String id = "aaa";
    String password = "bbb";
    if (!authentication(id, password)) {
      throw new Error("아이디 암호 불일치");
    }

    String userName = "userName";
    int i = authorization(userName);

    switch (i) {
      case 0:
        System.out.println("게임 매니저");
        break;
      case 1:
        System.out.println("유료 회원");
        break;
      case 2:
        System.out.println("무료 회원");
        break;
      case 3:
        System.out.println("권한 없음");
        break;
      default:
        System.out.println("기타 경우");
        break;

    }
    return connection(decodedInfo);
  }
}
```

```java
package com.nj.library;

public class DefaultGameConnectionHelper extends AbstGameConnectionHelper {
  @Override
  protected String doSecurity(String string) {
    System.out.println("디코드");
    return string;
  }

  @Override
  protected boolean authentication(String id, String password) {
    System.out.println("아이디/암호 확인 과정");
    return true;
  }

  @Override
  protected int authorization(String userName) {
    System.out.println("권한 확인");
    return 0;
  }

  @Override
  protected String connection(String info) {
    System.out.println("마지막 접속 단계");
    return info;
  }

  @Override
  public String requestConnection(String encodedInfo) {
    return super.requestConnection(encodedInfo);
  }
}
```

```java
import com.nj.library.AbstGameConnectionHelper;
import com.nj.library.DefaultGameConnectionHelper;

public class Main {
  public static void main(String[] args) {
    AbstGameConnectionHelper connectionHelper = new DefaultGameConnectionHelper();
    connectionHelper.requestConnection("김남준");
  }
}
```

```java
디코드
아이디/암호 확인 과정
권한 확인
게임 매니저
마지막 접속 단계

Process finished with exit code 0
```

  

## 실습 예제

* 보안 부분이 정부 정책에 의해서 강화 되었습니다. 강화된 방식으로 코드를 변경해야 합니다.
* 밤 10시 이후에 접속에 제한 되도록 합니다.
* 템플릿 메소드의 변경 없이, 

```java
package com.nj.library;

public class DefaultGameConnectionHelper extends AbstGameConnectionHelper {
  @Override
  protected String doSecurity(String string) {
    System.out.println("강화된 알고리즘 적용");
    System.out.println("디코드");
    return string;
  }

  @Override
  protected boolean authentication(String id, String password) {
    System.out.println("아이디/암호 확인 과정");
    return true;
  }

  @Override
  protected int authorization(String userName) {
    System.out.println("유저의 나이, 현재 시간 확인 후 10시 이후면 접속 불가");
    return -1;
  }

  @Override
  protected String connection(String info) {
    System.out.println("마지막 접속 단계");
    return info;
  }

  @Override
  public String requestConnection(String encodedInfo) {
    return super.requestConnection(encodedInfo);
  }
}
```

```java
package com.nj.library;

public abstract class AbstGameConnectionHelper {
  protected abstract String doSecurity(String string);

  protected abstract boolean authentication(String id, String password);

  protected abstract int authorization(String userName);

  protected abstract String connection(String info);

  public String requestConnection(String encodedInfo) {
    String decodedInfo = doSecurity(encodedInfo);

    String id = "aaa";
    String password = "bbb";
    if (!authentication(id, password)) {
      throw new Error("아이디 암호 불일치");
    }

    String userName = "userName";
    int i = authorization(userName);

    switch (i) {
      case -1:
        System.out.println("셧다운 유저");
        break;
      case 0:
        System.out.println("게임 매니저");
        break;
      case 1:
        System.out.println("유료 회원");
        break;
      case 2:
        System.out.println("무료 회원");
        break;
      case 3:
        System.out.println("권한 없음");
        break;
      default:
        System.out.println("기타 경우");
        break;

    }
    return connection(decodedInfo);
  }
}
```

```java
강화된 알고리즘 적용
디코드
아이디/암호 확인 과정
유저의 나이, 현재 시간 확인 후 10시 이후면 접속 불가
셧다운 유저
마지막 접속 단계

Process finished with exit code 0
```



