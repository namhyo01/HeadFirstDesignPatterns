# strategy pattern

예) 조는 SumUDuck이라는 오리 연못 시뮬레이션 게임을 만드는 회사에 다니고 있다. 이 게임에서는 헤엄도 치고 꽥꽥거리는 소리도 내는 매우 다양한 오리 종류를 보여줄 수 있다. 이 시스템을 처음으로 디자이한 사람들은 표준적인 객체지향 기법을 사용하여 Duck이라는 super class를 만든 후, 그 클래스를 확장해 다른 모든 종류의 오리를 만들었습니다.

```java
  public abstract class Duck{
    quack();
    swim();
    display(); //각 형식별로 자기 모양을 가지기에 이 함수를 별도 구현해야한다
  }
```
```java
  class MallardDuck extends Duck{
    display(){
    //적당한 모양 표시
    }
  }
```
이런식으로 **Duck 클래스를 상속받아** 사용을 합니다.

***
이젠 오리들을 날 수 있게 만들라는 결정을 위에서 내렸습니다.  
=> 이를 간단하게 생각하여 Duck 클래스에 fly()만 추가하면 모든 오리들이 상속받는 다 생각하였습니다.
```java
  public abstract class Duck{
    quack();
    swim();
    display(); //각 형식별로 자기 모양을 가지기에 이 함수를 별도 구현해야한다
    fly();
  }
```
그러나 **문제가 생겼습니다.**  
오리 인형처럼 날 수 없는 애들마저도 날게 되버린 경우죠.  
즉 Duck의 **모든 서브 클래스가 날 수 있는 것이 아니라는 점입니다.**  
```java
  class RubberDuck extends Duck{
    quack(){
      //삑삑
    }
    display(){
    //적당한 모양 표시
    }
  }
```
그러기에 간단한 생각을 하자면 fly()메소드를 그냥 override하면 되는 것이라 생각을 하였죠.

```java
  class RubberDuck extends Duck{
    quack(){
      //삑삑
    }
    display(){
    //적당한 모양 표시
    }
    fly(){
      //암것도하지마
    }
  }
```
그런데 이젠 나무 오리 인형을 추가하게 되면 어떨까요?  
날수도 없고, 소리도 낼 수 없습니다.  
```java
  class DecoyDuck extends Duck{
    quack(){
      //아무것도 x
    }
    display(){
    //나무오리
    }
    fly(){
      //암것도하지마
    }
  }
```
이렇게 하다보니 조는 상속을 활용하는 것이 옳은 해법이 아닐 수도 있을 것이라 생각을 하였습니다.  
임원진이 앞으로 6개월 마다 제품을 갱신하기로했다고 하였기 때문이죠.
그렇게되면 규격이 계속 바뀌게 될 것이고, 그러면 매번 전에 프로그램에 추가했던 Duck의 서브클래스의 fly()와 quack() 메소드를 일일히 살펴봐야하고,  
상황에 따라 오버라이드 해야 할 수도 있게 됩니다. 이 것이 영원히 반복될 것입니다.  
결국 **일부 형식의 오리만 날거나 꽤꽥 거릴수 있도록 하는더 깔끔한 방법을 찾아야 합니다.**  
그래서 선택한 방법이 fly()를 Duck의 super class 에서 빼고, fly()메소드가 들어있는 **Flyable interface**를 만들 수도 있다.  
이러면 날 수 있는 오리들에 대해서만 그 인터페이스를 구현해서 fly()메소드를 집어 넣을 수 있다.  
모든 오리들도 짖는것이 아니니 Quackable이라는 interface도 만들자

```java
  public abstract class Duck{
    swim();
    display(); //각 형식별로 자기 모양을 가지기에 이 함수를 별도 구현해야한다
  }
```
```java
  interface Flyable{
    fly();
  }
```

```java
  interface Quickable{
    quack();
  }
```
```java
  class MallardDuck extends Duck implements Flyable, Quackable{
    display(){
    //적당한 모양 표시
    }
    fly(){
    
    }
    quack(){
   
    }
  }
```
```java
  class RubberDuck extends Duck implements Quackable{
    display(){
    //적당한 모양 표시
    }
    quack(){
   
    }
  }
```
```java
  class DecoyDuck extends Duck{
    display(){
    //적당한 모양 표시
    }
  }
```
이런식으로 작성 할 수 있습니다.  
그러나 이러한 아이디어는 **코드 중복**을 생각하지 못하였습니다.  
메소드 몇개 오버라이드 해야하는게 안좋다면, 날아가는 동작을 조금 바꾸기 위해 Duck의 서브 클래스 가운데 날아다닐 수 있는 48개의 코드를 전부 고쳐야 하는 상황을 만나게 된다.  
즉 **코드 재사용을 전혀 기대할 수 없고, 코드 관리면에서 또 다른 커다란 문제점이 생기게 된다.**  
물론 날 수 있는 오리 중에서도 날아다니는 방식이 서로 다를 수도 있다.  
=> **소프트웨어를 만들 때 나중에 고쳐야 할 때도 기존 코드에 미치는 영향을 최소한으로 줄이면서, 작업을 할 수 있도록 하는 방법이 없을까?**  
***
### 문제를 명확하게 파악해보자  
  
서브클래스마다 오리의 행동이 바뀔 수 있는데도 모든 서브클래스에 한 행동을 사용하도록 상속 하는 것은 그리 올바르지 않다.  
인터페이스를 사용하는 방법은 괜찮아 보였지만, 자바 인터페이스에는 구현된 코드가 전혀 들어가지 않기에 **코드 재사용이 불가능 하다**  
즉 한 행동을 바꿀 때 마다 그 행동이 정의되어 있는 서로 다른 서브클래스들을 전부 찾아서 코드를 일일이 고쳐야 한다.  
이 과정에서 새 버그가 발생할 수 있다.  
**하지만 이런 상황에서 딱 어울릴 만한 디자인 원칙이 있다.**  
**디자인 원칙**
1. 애플리케이션에서 **달라지는 부분을 찾아**내고, **달라지지않는 부분으로부터 분리**시킨다.

즉 코드에 새로운 요구사항이 있을 때마다 바뀌는 부분이 있다면, 그 행동을 바뀌지 않는 다른 부분으로부터 골라내서 분리한다.  
이 원칙은 다음과 같은 식으로도 이해를 할 수 있는데,  
**바뀌는 부분을 따로 뽑아서 캡슐화 한다.** 그렇게 하면 나중에 **바뀌지 않는 부분에는 영향을 미치지 않은채로 그 부분만 고치거나 확장 가능**  
이 개념은 매우 간단하지만 다른 모든 디자인 패턴의 기반을 이루는 원칙이다.  
모든 패턴은 **시스템의 일부분을 다른 부분과 독립적으로 변화 시키는 방법**을 제공하기 위한 것이다.
***
바뀌는 부분과 그렇지 않은 부분을 분리해보자  
fly()와 quack() 문제를 제외하고 duck클래스는 잘 작동하고 있으며, 나머지는 자주 달라지거나 바뀌지 않는다.  
변화하는 부분과 그대로 있는 부분을 분리하면 두개의 클래스 집합을 만들어야 한다.  
하나는 나는 것과 관련된 집합, 다른 하나는 꽥꽥거리는 것과 관련된 부분이다.  
**각 클래스 집합에는 각각의 행동을 구현한 것을 전부 넣습니다.**  
예를 들어 꽥꽥 거리는 것을 구현하는 클래스를 하나만들고, 삑삑 소리를 내는 것을 구현하는 클래스를 하나 만들고, 아무 소리도 내지 않는 클래스도 하나 만듭니다.  
***
### 오리 행동 디자인
나는 행동과 꽥꽥 거리는 행동을 구현하는 클래스 집합은 어떻게 디자인 해야할까?
**- 우선 최대한 유연하게 만들자**  
**- Duck의 인스턴스에 행동을 할당할 수 있어야 한다**  
예를 들어 MallardDuck 인스턴스를 새로 만들고 특정 형식의 나는 행동으로 초기화 하는 것도 좋은 방법이다.  
**- 오리의 행동을 동적으로 바꿀 수 있으면 더 좋다**  
즉 Duck 클래스에 행동과 관련된 setter메소드를 포함시켜 프로그램 실행중에도 MallardDuck의 나는 행동을 바꿀수 있게 하면 좋다.  
**디자인원칙**
2. 구현이 아닌 **인터페이스에 맞춰서 프로그래밍한다.**  

각 행동은 인터페이스(FlyBehavior, QuackBehavior)로 표현하고 행동을 구현할 때 **인터페이스를 구현하도록 한다**  
즉 행동 인터페이스는 Duck클래스가 아닌, 행동 클래스에서 구현한다.  
이 방법은 지금까지 쓴, 행동을 Duck클래스에서 구체적으로 구현하거나, 서브클래스 자체에서 별도로 구현하는 방법하고는 상반된 방법이다.  
전에 쓴 두 방법은 항상 특정 구현에 의존을 했었기에, 행동을 변경할 여지가 없었다.  
새로운 디자인 경우는 Duck의 서브클래스에서 인터페이스로 표현되는 행동을 사용하기에, 행동을 실제로 구현한 것은 Duck 서브 클래스에 국한되어 있지 않다.  
여기서 **인터페이스에 맞춰서 프로그래밍한다**라는 의미는 **상위 형식에 맞춰서 프로그래밍한다**라는 것을 의미한다.  
즉 자바의 interface를 반드시 사용하라는 것은 아니다.(개념 지칭)  
여기에서 가장 핵심적인 것은 **실제 실행시에 쓰이는 객체가 코드에 의해서 고정되지 않도록**, 어떤 **상위 형식(supertype)에 맞추어 프로그래밍 함으로써 다형성을 활용**해야하는 것이다.  
**상위 형식에 맞추어 프로그래밍**하라는 원칙은 **변수를 선언할 때는 보통 추상 클래스나 인터페이스 같은 상위 형식으로 선언**하라는 의미이다.  
예를 들어 구현에 맞춰서 프로그래밍을 한다면 다음과 같이 할 수 있다.
```java
  pulic abstract class Animal{
    makeSound();
  }
```

```java
  pulic class Dog extends Animal{
    makeSound(){
      bark();
    }
    bark(){//짖음}
  }
```

```java
  pulic class Cat extends Animal{
    makeSound(){
      meow();
    }
    bark(){//야옹}
  }
```
```java
  Dog d = new Dog();
  d.bark();
```
하지만 인터페이스/상위 형식에 맞춰서 프로그래밍 한다면 다음과 같이 할 수 있다.
```java
  Animal animal = new Dog();
  animal.makeSound();
```
더 바람직한 방법은 사우이 형식의 인스턴스를 만드는 과정을(new Dog) 직접 코드로 만드는 대신 구체적으로 구현된 객체를 실행시에 대입하는 것이다.
```java
  a = getAnimal();
  a.makeSound();
```
***
그래서 여기에선 FlyBehavior과 QuackBehavior이라는 두가지 인터페이스를 사용한다.  
그리고 구체적인 행동을 구현하는 클래스들이 있다.  

```java
  interface FlyBehavior{
    fly();
  }
```
```java
  class FlyWithWings implements FlyBehavior{
    fly(){
      //날아다니는거 구현
    }
  }
```
```java
  class FlyNoWay implements FlyBehavior{
    fly(){
      //아무것도 하지 않음 날 수없다.
    }
  }
```
```java
  interface QuackBehavior{
    quack();
  }
```
```java
  class Quack implements QuackBehavior{
    quack(){
      //꽥꽥 소리냄
    }
  }
```
```java
  class Squeak implements QuackBehavior{
    quack(){
      // 삑삑
    }
  }
```
```java
  class MuteQuack implements QuackBehavior{
    quack(){
      // 아무소리도 내지 않는다.
    }
  }
```
이런식으로 디자인하면 다른 형식의 객체에서도 나는 행동과 꽥꽥 거리는 행동을 재사용 할 수 있다.  
그런 행동이 더 이상 Duck클래스에 숨겨져 있는 것이 아니기 때문이다.  
-> 따라서 상속을 쓸 때 떠안게 되는 부담을 떨치고서도 재사용의 장점을 누릴 수 있다.  
또한 기존의 행동 클래스를 수정하거나 날아다니는 행동을 사용하는 Duck클래스를 전혀 건드리지 않고 새로운 행동을 추가할 수 있다.  
***
**가장 중요한 점은 잊 Duck에서 나는 행동과 꽥꽥 소리를 내는 행동을 Duck클래스(또는 서브클래스)에서 정의한 메소드를 써서 구현하지 않고 다른 클래스에 위임한다는 것이다.**  
1. 우선 Duck클래스에 flyBehavior와 quackBehavior라는 두개의 인터페이스 형식의 인스턴스 변수를 추가한다. 나는 행동과 꽥꽥거리는 행동은 이 인터페이스로 옮겨 놨기 때문에 Duck클래스 및 모든 서브클래스에 fly()와 quack()메소드도 제거해야 합니다. 이들 대신 performFly()오 performQuack()이라는 메소드를 넣습니다.
```java
  public class Duck{
    FlyBehavior flyBehavior;
    QuackBehavior quackBehavior;
    
    performQuack();
    swim();
    display();
    performFly();
  }
```
2. performQuack()을 구현해보면
```java
  public class Duck{
    QuackBehavior quackBehavior;//모든 Duck에는 QuackBehavior인터페이스를 구현하는 것에 대한 레퍼런스가 있다.
    //기타코드
    public void performQuack(){
      quackBehavior.quack(); // 꽥꽥 거리는 행동을 직접처리하는 대신 quackBehavior를 참조되는 객체에 그 행동을 위임
    }
  }
```
지금 이 코드에선 객체의 종류에는 전혀 신경 쓸 필요가 없다. quack()를 실행시킬 수 안다는 것이 중요하다.
3. 이제 flyBehavior와 quackBehavior 인스턴스 변수를 설정하는 방법에 대해 생각해보자.
```java
  public class MallardDuck extends Duck{
    public MallardDuck(){
      quackBehavior = new Quack();
      flyBehavior = new FlyWithWings();
    }
    public void display(){
      System.out.println("저는 물오리입니다.");
    }
  
  }
```
그러나 이때 특정 구현에 맞춰서 프로그래밍 하면 안된다고 하였다. 그러나 앞의 생성자에서보면 Quack이라는 구현되어 있는 구상 클래스의 인스턴스를 만들었다.  
하지만 **행동을 구상 클래스로 설정하고 있긴 하지만, 실행시에 쉽게 변경할 수가 있다.**  
그래서 이 코드는 굉장히 유연하다. 인스턴스 변수를 유연한 방법으로 초기화하는 방법을 사용하고 있기 때문이다.  
quackBehavior 인스턴스 변수는 인터페이스 형식이기에, 실행시에 동적으로 QuackBehavior를 구현한 다른 클래스를 할당할 수 있다.  
***
### Duck Code Test
```java
  public abstract class Duck}
      FlyBehavior flyBehavior;
      QuackBehavior quackBehavior;
      public Duck(){}
      public absract void display();
      public void performFly(){
        flyBehavior.fly(); // 행동 클래스에 위임
      }
      public void performQuack(){
        quackBehavior.quack();
      }
      public void swim(){
        System.out.println("모든 오리는 물에 뜹니다. 가짜 오리도요");
      }
  }
```
```java
  public class MallardDuck extends Duck{
    public MallardDuck(){
      quackBehavior = new Quack();
      flyBehavior = new FlyWithWings();
    }
    public void display(){
      System.out.println("저는 물오리입니다.");
    }
  
  }
```

```java
  public interface FlyBehavior{
    public void fly();
  }
  public interface QuackBehavior{
    public void quack();
  }
```
```java
  public class FlyWithWings implements FlyBehavior{
    public void fly(){
      System.out.println("날고 있어요!");
    }
  }
  public class FlyNoWay implements FlyBehavior{
    public void fly(){
      System.out.println("저는 못날아요!");
    }
  }
```
```java
  public class Quack implements QuackBehavior{
    public void quack(){
      System.out.println("꽥");
    }
  }
  public class MuteQuack implements QuackBehavior{
    public void quack(){
      System.out.println("조용~");
    }
  }
    public class Squeak implements QuackBehavior{
    public void quack(){
      System.out.println("삑");
    }
  }
```
```java
  public class MiniDuckSimulator{
    public static void main(String[] args){
      Duck mallard = new MallardDuck();
      mallard.performQuack();
      mallard.performFly();
    }
  }

```
**이젠 동적으로 만들어놓고 활용해야 합니다. setter메소드를 통하여 호출하는 방법으로 설정할 수 있습니다.**
```java
  public abstract class Duck}
      FlyBehavior flyBehavior;
      QuackBehavior quackBehavior;
      public Duck(){}
      public absract void display();
      public void performFly(){
        flyBehavior.fly(); // 행동 클래스에 위임
      }
      public void performQuack(){
        quackBehavior.quack();
      }
      public void swim(){
        System.out.println("모든 오리는 물에 뜹니다. 가짜 오리도요");
      }
      // 오리의 행동을 즉석에서 바꾸고 싶으면 이 메소드를 호출한다.
      public void setFlyBehavior(FlyBehaivor fb){
        flyBehavior = fb;
      }
      public void setQuackBehavior(QuackBehaivor qb){
        quackBehavior = qb;
      }
  }
```
```java
  public class ModelDuck extends Duck{
    public MallardDuck(){
      quackBehavior = new Quack();
      flyBehavior = new FlyNoWay();
    }
    public void display(){
      System.out.println("저는 모형 오리입니다.");
    }
    
  }
```
```java
  public class FlyRocketPowered implements FlyBehavior{
    public void fly(){
      System.out.println("로켓 추진으로 날고 있어요!");
    }
  }

```
```java
  public class MiniDuckSimulator{
    public static void main(String[] args){
      Duck mallard = new MallardDuck();
      mallard.performQuack();
      mallard.performFly();
      
      //추가
      Duck model = new ModelDuck();
      model.performFly();
      model.setFlyBehavior(new FlyRocketPowered()); //이렇게 하면 동적으로 나는 행동을 바꿀 수 있다.
      model.performFly();
    }
  }

```
***
오리의 행동들을 일련의 행동으로 생각하는 대신, 알고리즘군으로 생각합니다.  
즉 Fly나 Quack들의 행동 직합을 알고리즘 군으로 생각합니다.  
또한 클래스 사이의 관계에도 관심을 생각하여 클래스들이 어떤 관계인지를 생각해보자.(A는 B이다 관계인지, A에는 B가 있다인지)  
A는 B이다 보다 A에는 B가 있다가 더 나을수도 있다.  
**A에는B가 있다** 관계를 생각해보면  
각 오리는 FlyBehavior와 QuackBehavior가 있어 각각 행동과 꽥꽥 거리는 행동을 위임받는다.  
이렇게 두 클래스를 합치는 것을 구성을 이용하는 것이라고 한다.   
오리 클래스에서는 행동을 상속받는 대신, 올바른 행동 객체로 구성됨으로서 행동을 부여받는다.
**디자인 원칙**
3. 상속보다는 구성을 활용한다

**이렇게 구현을 활용하면 유연성을 크게 늘릴수 있다.**  
단순히 알고리즘군을 별도의 클래스 집합으로 캡슐화 할 수 있도록 만들어주는것 뿐만 아니라, 구성요소로 사용하는 객체에서 올바른 행동 인터페이스를 구현하기만 하면 실행시에 행동을 바꿀수도 있게해준다.  

## 스트래티지 패턴(Strategy Pattern)
이 패턴을 좀 더 정확하게 정의하자면  
이 패턴에서는 알고리즘군을 정의하고 각각을 캡슐화 하여 교환해서 사용할 수 있도록 만든다.   
이 패턴을 활용하면 알고리즘을 사용하는 클라이언트와는 독립적으로 알고리즘을 변경할 수 있다.




















