# decorator pattern
데코레이터 패턴은 객체에 추가적인 요건을 동적으로 추가할 수 있게 해준다. 그래서 이 데코레이터는 sub class를 만듬으로서 기능을 유연하게 확장 가능하다. 

# 상황 예시
***
스타버즈 커피전문점에서 다양한 음료들을 포괄하는 주문 시스템을 갖출려고 한다.  
***
음료를 나타내주는 Beverage 객체를 추상 클래스로 만들어 이 커피점에서 판매되는 모든 음료들은 이것을 상속받아 쓰게 된다.  
이 클래스 안에는 cost() 메소드가 있어 가격을 계산 할 수 있고(서브 클래스가 오버라이드)  
description은 인스턴스 변수로서 음료에 대한 설명이 들어가게 된다.  
이렇게 끝을 내고 싶지만 커피를 주문할때는 우유나, 두유, 모카, 휘핑크림, 아이스 등등 추가조건들이 붙기도 한다  
그러면 가격이 올라가기에 이런 경우를 다 구해볼 것이다.  
일단 상속을 이용하여 모든 case에 맞추어본 결과

![image](https://user-images.githubusercontent.com/34156840/142241225-b88c3871-4d87-4e68-a845-a51251ccbd86.png)  
이런 모양이 나오게 되는데, 만약 이렇게 할시 **눈으로만 봐도 관리와 추가나 수정이 힘들어 질 것이 분명하다**  

**또한 디자인 원칙의 2가지를 위배하고 있는데, 위를 보면 바뀌는 부분이 cost메소드가 자주 바뀌지만 캡슐화가 되어 있지 않다.**  
**마지막 하나는 상속보다는 구성을 활용해야 하는데, 그것을 위반하고 있다.**  
이로서 위의 그림은 디자인 원칙을 지키지 않다 할 수 있다.  

그래서 이 수많은 클래스를 다 관리하기가 힘드니 그냥 인스턴스 변수랑 슈퍼 클래스 상속을 통해 관리한다 해보자.  
![image](https://user-images.githubusercontent.com/34156840/142245608-a7b498f8-5a5d-4567-a709-70af3310e3de.png)    
이런 모양으로 만들고 나니 이제 메뉴에 있는 음료 종류들만 클래스로 만들어 각자 cost를 오버라이드만 하면되게 된다.  
예를들어
```java
  class DarkRoast extends Beverage{
    public int cost(){
      return super.cost() + 100;
    }
  }
```
이런식으로 사용하면 되는듯 하지만  
만약 우유, 휘핑크림 말고도 쿠키나 아이스크림 같은 것이 추가되면 매번 코드의 수정을 해주어야 한다.  
또한 어떤 커피는 휘핑크림을 안넣지만(무조건) 그래도 휘핑크림에 대한 정보를 가지고 있어야 한다.  

***
# OCP

위의 문제들을 통해 해결할 수 있게 OCP를 배워보자
**OCP는 디자인 원칙 중 하나이다**
>클래스는 확장에 대해 열려있지만 코드 변경에 대해서는 닫혀 있어야 한다.  

기존 코드는 **건드리지 않고 확장을 통해** 새로운 동작을 추가할 수 있게 하는 것이 목표이다  
이것을 성공 하면 매우 유연하여 주변환경에 잘 적응하고, 강하고 튼튼한 디자인을 만들 수 있다  
***
### QnA  
Q 확장에 대해 열려있고 코드 변경은 닫혀있는게 모순아닌가?  
A 예를들어 옵저버 패턴같은 경우 옵저버를 새로 추가하면 Subject의 코드는 수정 없이 확장할 수 있다.  
Q 디자인을 할때 모든 부분에서 OCP를 어떻게 준수하는가?  
A 보통은 불가능하다.(!?) 이렇게 할려면 적지 않는 시간과 노력이 필요하다. 그런데 디자인의 모든 부분을 깔끔하게 정돈할만큼 여유있는 경우는 많지 않다.   
(게다가 굳이 그렇게 안해도 되는 경우도 있다.)  
OCP를 지키다 보면 새 추상화가 필요한데, 그러면 코드가 복잡해진다  
**그래서 디자인한 것 중에서 가장 바뀔 확률이 높은 부분을 중점적으로 살피고 원칙을 적용한다**  
(디자인 패턴은 바뀌는 부분에 대해 처리하거나 생각하는 것이 대부분인 것 같다.)  
Q 바뀌는 부분에서 중요한 부분을 어떻게 골라냄?  
A 디자인 경험과 분야에 대한 지식이 크게 작용한다.  
**고로 코드에서 확장해 갈때는 확장 할 부분을 선택에 주의를 들여 무작정 OCP를 적용하는건 시간낭비고, 결과적으로 불필요하고 복잡해 이해하기 힘든 코드만 만들 수 있다.**
***
# 데코레이터 패턴  
위의 케이스들은 클래스가 너무 많아지거나 불필요한 기능이 서브클래스에 들어가는 문제가 발생하였다.  
이를 해결하기 위해 우린 장식(decorate)를 할것이다. 예를들어  
1. DarkRoast객체를 가져온다
2. Mocha 객체로 장식한다(모카추가)  
3. Whip 객체로 장식한다(휘핑크림 추가)
4. cost()메소드를 호출한다. 이때 첨가물의 가격을 계산하는건 해당 객체에게 위임한다. (계산)  

이런식으로 볼 수 있다. 
* 참고로 데코레이터 객체를 래퍼 객체라고 생각해보자.(감싼다라는 의미)  

![image](https://user-images.githubusercontent.com/34156840/142250763-a05b851a-2f49-4efa-81ae-95224a89d8d4.png)  
Mocha와 Whip은 데코레이터이다. 데코레이터들의 형식은 데코레이터가 장식하는 객체를 반영한다. 그래서 위의 경우는 Beverage객체가된다.(반영 = 같은 형식을 갖춘다)  
그럼 구매는 어떻게 하느냐?  
1. 가장 바깥쪽 데코레이터인 Whip의 cost를 부르고
2. Whip에선 Mocha의 cost를 호출
3. Mocha에선 DarkRoast의 cost를 호출한다
4. DarkRoast에선 계산을해 99센트 리턴
5. Mocha가 그걸 받아 자신의 값을 추가해 계산
6. Whip에서 Moca까지 받은 가격에 자신의 가격을 추가해 계산한 최종값을 리턴한다.  

***
### 정리  
* 데코레이터의 수퍼클래스 == 장식하는 객체의 수퍼클래스
* 한 객체를 여러개의 데코레이터로 장식 가능(위의 케이스는 2개의 데코레이터로 감쌈)
* 데코레이터는 장식하는 객체의 수퍼클래스를 가지고 있어서 원래 객체가 들어갈 자리에 데코레이터 객체를 넣어도 문제가 없다
* 데코레이터는 장식하는 객체에게 어떤 행동을 위임 + **원하는 추가적인 작업을 수행 가능**
* 객체는 언제든지 감쌀 수 있어서 실행중에 필요한 데코레이터를 마음대로 적용이 가능  

***
# 데코레이터 정의
데코레이터 패턴은 객체에 추가적인 요건을 **동적으로 첨가한다**  
서브클래스를 만드는 것을 통해 기능을 유연하게 확장할 수 있는 방법을 제공한다  
클래스 다이어그램을 봐보자
![image](https://user-images.githubusercontent.com/34156840/142252535-205c458a-0b94-4025-956d-51f3fb9a4934.png)  
여기서 ConcreteDecorator A와 B의 Component wrappedObj는 그 객체가 장식하는 객체를 위한 인스턴스 변수이다.  
또한 B에서 보는 것처럼 Object newState변수를 추가하여 상태를 확장할 수 있으며, 당연히 메소드도 추가가 가능하다  
### Beverage클래스를 보면
![image](https://user-images.githubusercontent.com/34156840/142253411-8158c427-6420-4e04-965d-7787be2794d0.png)  
Beverage클래스는 당연히 앞에 있는 Component추상 클래스랑 같은거고  
커피 종류들마다 구성요소를 나타내는 구상 클래스를 만들어준다.  
그 다음 contimentDecorator을 시작해 첨가물들이 데코레이터로 표현이 되어 있다.  
>**데코레이터 패턴에서 상속을 받는건 형식을 맞추는 것이지 행동을 물려받는것이 아니다** => 즉 올바른 형식을 맞추는 것이고, 행동은 기본 구성요소들과 다른 데코레이트 등을 인스턴스 변수에 저장하는 식으로 연결  
>게다가 객체 구성을 이용중이여서(인스턴스 변수로 다른 객체 저장) 다양한 음료와 첨가물을 섞어도 된다.
>이 구성을 통해 실행중에도 데코레이터를 조합해서 사용할 수 있다. => 새 행동마다 기존코드 바꿀 필요가 없는 장점  

***
# 직접 코드로 구현
Beverage Class
```java
public abstract class Beverage {
	String description = "제목 없음";
	public String getDescription() {
		return description;
	}
	public abstract double cost();
}

```
첨가물 나타내는 데코레이터를 위한 Beverage랑 구성요소가 같은 추상클래스 CondimentDecorator
```java
//첨가물 나타내는 추상 클래스
public abstract class CondimentDecorator extends Beverage{ //Beverage 객체가 들어갈 자리에 들어갈 수 있어야 해서 확장
	public abstract String getDescription();
}

```
이제 음료들을 각각 구상 클래스로 생성
```java
public class DarkRoast extends Beverage{
	public DarkRoast() {
		// TODO Auto-generated constructor stub
		description = "다크로스트";
	}
	@Override
	public double cost() {
		// TODO Auto-generated method stub
		return 0.99;
	}
	
}
```
```java
public class Espresso extends Beverage{ // Beverage 확장
	public Espresso() {
		description = "에스프레소";
	}

	//여기서 첨가물 계산은 걱정 안하여도 된다.
	public double cost() {
		return 1.99; 
	}
}

```
```java
public class HouseBlend extends Beverage{
	public HouseBlend() {
		description = "하우스 블렌드 커피";
		
	}
	@Override
	public double cost() {
		return 0.89;
	}

}

```
이제 데코레이터들을 구상클래스로 만들자
```java
public class Mocha extends CondimentDecorator{
	Beverage beverage; //감싸고 있는 음료를 저장하기 위한 인스턴스 변수
	
	public Mocha(Beverage beverage) {
		this.beverage = beverage; // 인스턴스 변수를 감싸는 객체로 설정
	}
	
	@Override
	public String getDescription() {
		return beverage.getDescription() + ", 모카" ;
	}

	@Override
	public double cost() {
		return 0.20+beverage.cost();
	}
	
}
```
```java
public class Soy extends CondimentDecorator{
	Beverage beverage; //감싸고 있는 음료를 저장하기 위한 인스턴스 변수
	
	public Soy(Beverage beverage) {
		this.beverage = beverage; // 인스턴스 변수를 감싸는 객체로 설정
	}
	
	@Override
	public String getDescription() {
		return beverage.getDescription() + ", 두유" ;
	}

	@Override
	public double cost() {
		return 0.40+beverage.cost();
	}
}

```
```java
public class SteamMilk extends CondimentDecorator{
	Beverage beverage; //감싸고 있는 음료를 저장하기 위한 인스턴스 변수
	
	public SteamMilk(Beverage beverage) {
		this.beverage = beverage; // 인스턴스 변수를 감싸는 객체로 설정
	}
	
	@Override
	public String getDescription() {
		return beverage.getDescription() + ", 스팀밀크" ;
	}

	@Override
	public double cost() {
		return 0.48+beverage.cost();
	}
}
```
```java
public class Whip extends CondimentDecorator{
	Beverage beverage; //감싸고 있는 음료를 저장하기 위한 인스턴스 변수
	
	public Whip(Beverage beverage) {
		this.beverage = beverage; // 인스턴스 변수를 감싸는 객체로 설정
	}
	
	@Override
	public String getDescription() {
		return beverage.getDescription() + ", 휘핑" ;
	}

	@Override
	public double cost() {
		return 0.56+beverage.cost();
	}
}
```
Main
```java
public class StarBuzzCoffee {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Beverage beverage = new Espresso();
		System.out.println(beverage.getDescription()+" $" + beverage.cost());
		
		Beverage beverage2 = new DarkRoast();
		beverage2 = new Mocha(beverage2);
		beverage2 = new Mocha(beverage2);
		beverage2 = new Whip(beverage2);
		System.out.println(beverage2.getDescription() + " $" + beverage2.cost());
		
		Beverage beverage3 = new HouseBlend();
		beverage3 = new Soy(beverage3);
		beverage3 = new Mocha(beverage3);
		beverage3 = new Whip(beverage3);
		System.out.println(beverage3.getDescription() + " $" + beverage3.cost());
		
	}

}
```
Result  
![image](https://user-images.githubusercontent.com/34156840/142261517-42f9bca2-b7e6-478f-bfc6-89316e8a9aed.png)  

***
### 단점  
> 자잘한 클래스들이 너무 많이 추가되어 남이 이해하기 힘들 수도 있다.(ex 자바 I/O 라이브러리)
> 형식관련 문제도 있다 특정 형식에 의존하는 클라이언트 코드를 가지고와서 생각 없이 데코레이터 패턴을 적용한다
> 데코레이터의 가장 장점이 이것을 추가해도 클라이언트는 사용하는지 안하는지 모르는데, 특정형식에 의존하는 코드에 데코레이터를 추가하면 문제가 발생한다
> 데코레이터를 도입시 구성 요소를 초기화 하는데 필요한 코드가 복잡해진다. 
> 얘를 쓰면 구성요소 인스턴스만 만들고 끝이 아니라 꽤 많은 데코레이터를 감싸야한다. => 이 두개의 단점을 팩토리하고 빌더가 help해준다

***
### 공부를 한 후기  
데코레이터 패턴 자체를 처음 공부한 것은 아니다. 사실 파이썬에서 @를 이용한 데코레이터를 먼저 공부해본적이 있어서 이 내용을 봤을 때는 마음이 조금 편했었다.  
그러나 이 데코레이터 패턴을 어떻게 쓰냐가 아닌 언제 어떻게 어떤 상황에서 쓰는건지에 대해 제대로 배웠음을 느꼈다.  
이 패턴은 추후 팩토리 패턴을 해보면서 연계가 어떻게 이루어 지는지, 이 데코레이터의 단점이 어떻게 극복되는지 공부해 봐야 겠다.














