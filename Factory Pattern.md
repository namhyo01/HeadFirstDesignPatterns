# Factory Pattern(팩토리패턴)
**느슨한 결합**을  이용하여 객체지향 디자인을 하자(모르면 옵저버 패턴 보고오자)  
객체의 인스턴스를 만드는 작업이 항상 공개되어 있을 필요가 없고, 오히려 모든 것을 공개했다가 결합과 관련된 문제가 생길 수 있다는 것을 본다  
***
### new의 의미는?
> new라는 것은 자바에서 구상 클래스의 instance를 만드는 것이다. 즉 특정 구현을 사용하는 의미이다.

```java
  Duck duck = new MallardDuck();
```
>위의 예시처럼 Duck이라는 인터페이스를 만들어서 유연하게 사용할려 하지만 MallardDuck이라는 구상 클래스의 인스턴스를 만들어서 써야한다. 
>그나마 다른 방법으로 조건문을 걸어 각각에 맞는 구상 클래스를 구현 하는 방법도 있긴하지만 컴파일시 어떤것의 인스턴스를 만들지 모르는 함정이 있다
>즉 이런식으로 짜면 뭔가 변경하거나 확장할때 마다 코드를 확인하고 추가, 제거해야하는 의미이다. -> **유지보수 어렵고 오류 가능성 높음**  

그런데 자바에서 객체를 만드는 방법은 new말고는 없다. 즉 얘는 문제가 아니고 **"변화"가** 문제인 것이다.  
그래서 인터페이스같은 것을 이용해 다형성을 활용하여 변화에 적응하는데 반대로 구상 클래스를 많이 사용하면 매번 추가시 코드를 고쳐야하므로 변화에 닫혀있는 것이다.  
이 부분이 **OCP**에 내용이다. (기억 안나면 Decorator에서 보고오자)
***
### 상황 예시
피자가게를 운영하고 있고, 파자가게 운영을 위한 코드를 만들었다 가정합시다.

```java
  Pizza orderPizza(String type){
    Pizza pizza;
    
    if(type.equals("cheese")){
      pizza = new CheesePizza();
    }else if(type.equals("greek")){
      pizza = new GreekPizza();
    }else if(type.equals("pepperoni"){
      pizza = new PepperoniPizza();
    }
  }
```
이런 식으로 짤 수 있습니다. 그런데 여기서 greek 종류가 더이상 팔 가치가없어 안팔거나 새로운 타입을 추가하게되면 매번 코드를 고쳐야 하니 안좋은 디자인이 된다.  
전부터 말한 자주 바뀌는 부분이 여기선 구상클래스를 인스턴스로 만들때가 문제인 것을 확인가능 하였습니다.  
고로 바뀌는 부분을 이제 캡슐화 해보면
***
### Simple Factory   
객체 생성을 처리하는 클래스를 **Factory**라고 한다  
그럼 내가 저 orderPizza에서 객체를 생성하는 부분을 빼서 새로 만들면 orderPizza는 클라이언트가 되고, 이것을 호출하는 형식으로 만들 것이다.  
그것을 구현해보자  
```java
public class SimplePizzaFactory {
	public Pizza createPizza(String type) {
		Pizza pizza = null;
		
		if(type.equals("cheese"))
			pizza = new CheesePizza();
		else if(type.equals("pepperoni")) 
			pizza = new Pepperonipizza();
		else if(type.equals("clam"))
			pizza = new ClamPizza();
		else if(type.equals("veggie"))
			pizza = new VeggiePizza();
		return pizza;
	}
}

```

```java
public class PizzaStore {
	SimplePizzaFactory factory;
	
	public PizzaStore(SimplePizzaFactory factory) {
		this.factory = factory;
	}
	
	public Pizza orderPizza(String type) {
		Pizza pizza;
		
		pizza = factory.createPizza(type);
		
		pizza.prepare();
		pizza.bake();
		pizza.cut();
		pizza.box();
		return pizza;
	}
	
}
```
이렇게 Simple Factory를 만들었으나 이건 디자인 패턴이라고 할 수는 없다... 그냥 관용구 정도라고 볼수 있다.  
그러나 많이 사용되지 않는 다는 말이 아니고, 많이 사용하기에 이걸 팩토리 패턴이라고 하는 사람도 있으나 정확한 말은 아니다.  
uml을 보면
![image](https://user-images.githubusercontent.com/34156840/142461396-e5505b94-2bc7-4ab2-b72a-6faf7277af7f.png)  
이런식으로 만들어지는 것을 볼 수있다.  
아직 Pizza클래스는 쓰지 않았으나 이런식으로 구현된다 정도만 알아두자  

> 가끔 저 createPizza()같은 메소드를 static을 써 선언하는 경우도 있다. 이러면 굳이 인스턴스를 선언 안하여도 되지만 정적이기에 동적으로 변동이 어렵다.  

***
자 우리의 피자가게가 너무 성장해서 프렌차이즈를 열수가 있게 되었다.  
그래서 시카고 프렌차이즈점, 뉴욕 프렌차이즈, 캘리포니아 등등등 열게 되었다.  
그래서 우리는 현재까지 잘 써온 이 코드를 다른 지점에서도 쓸 수 있게 할려하지만 지역별로 거기에 맞는 입맛이나 특성에 맞춰 다른 스타일로 적용을 해주어야한다.  
아까 했던 식으로 뉴욕하나 시카고하나 이렇게 각각 팩토리를 만들어주면 가능은 하지만 분점에서 독자적인 방법들을 이용해 굽기 시작하면 어떻게 해야하나?  

피자가게와 피자제작 과정 전체를 한개로 묶어주는 프레임워크를 만들면된다. 그러면서 유연성은 잃어서는 안되는 것이 중요하다.  
그럼 전체로 묶을 수도 있으면서 유연성을 어떻게 주어야하는가? => **추상 메소드를 사용하면 된다**

```java
public abstract class PizzaStore {
	
	public Pizza orderPizza(String type) {
		Pizza pizza;
		
		pizza = createPizza(type);
		
		pizza.prepare();
		pizza.bake();
		pizza.cut();
		pizza.box();
		return pizza;
	}
	protected abstract Pizza createPizza(String type); // => 이로서 pizza 인스턴스를 만드는 것은 팩토리 역할을 하는 method에서 담당
	
}
```
이런식으로 추상 메소드를 이용하여 팩토리 객체 대신 이 메소드를 활용하면 된다  
자 이제 서브 클래스들에서 createPizza를 오버라이드해서 구현하면 된다. 어떻게? 자기 스타일로  

한번 뉴욕풍 피자를 만들어보겠습니다.  
```java
public class NYPizzaStore extends PizzaStore{
	@Override
	Pizza createPizza(String type) {
		if(type.equals("cheese"))
			return new NYStyleCheesePizza();
		else if(type.equals("veggie"))
			return new NYStyleVeggiePizza();
		else if(type.equals("clam"))
			return new NYStyleClamPizza();
		else if(type.equals("pepperoni"))
			return new NYStylePepperoniPizza();
		else
			return null;
	}
	
}
```







