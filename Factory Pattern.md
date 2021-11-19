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
**팩토리 메소드는 객체 생성을 하며, 팩토리 메소드를 이용해 객체 생성 작업을 서브클래스에 캡슐화가 가능하다**  
이제 피자클래스를 구현해서 완성해보겠다  
```java
public abstract class Pizza {
	String name;
	String dough;
	String sauce;
	ArrayList<String> toppings = new ArrayList<String>();
	
	void prepare() {
		System.out.println("Preparing " + name);
		System.out.println("Tossing dough...");
		System.out.println("Adding sauce...");
		System.out.println("Adding toppings: ");
		for(int i=0;i<toppings.size();i++)
			System.out.println("   " + toppings.get(i));
	}
	
	void bake() {
		System.out.println("Bake for 25 minutes at 350");
	}
	void cut() {
		System.out.println("Cutting the pizza into diagonal slices");
	}
	void box() {
		System.out.println("Place pizza in official PizzaStore box");
	}
	public String getName() {
		return name;
	}
}
```
이제 구상 서브 클래스를 만들어 봅니다
```java
public class NYStyleCheesePizza extends Pizza{
	public NYStyleCheesePizza() {
		name = "NY Style Sauce and Cheese Pizza";
		dough = "Thin Crust Dough";
		sauce = "Marinara Sauce";
		
		toppings.add("Grated Reggiano Cheese");
	}
}

```
```java
public class ChicagoStyleCheesePizza extends Pizza{
	public ChicagoStyleCheesePizza() {
		name = "Chicago Style Deep Dish Cheese Pizza";
		dough = "Extra Thick Crust Dough";
		sauce = "Plum Tomato Sauce";
		
		toppings.add("Shredded Mozzarella Cheese");
	}
	
	void cut() {
		System.out.println("Cutting the pizza into square slices");
	}
}

```
main
```java
public class PizzaTestDrive {

	public static void main(String[] args) {
		PizzaStore nyStore = new NYPizzaStore();
		PizzaStore chicagoStore = new ChicagoPizzaStore();
		
		Pizza pizza = nyStore.orderPizza("cheese");
		System.out.println("Ethan ordered a " + pizza.getName() + "\n");
		
		pizza = chicagoStore.orderPizza("cheese");
		System.out.println("Joel ordered a " + pizza.getName() + "\n");
	}

}
```
### 정리해서
모든 팩토리 패턴에선 객체 생성을 캡슐화를 한다.  
그리고 서브클래스에서 어떤 클래스를 만들지를 결정하게 함으로서 객체 생성을 캡슐화를 한다.  
생산자 클래스
![image](https://user-images.githubusercontent.com/34156840/142477260-79d573ab-17f9-424a-a396-46495ec845a6.png)  

제품 클래스  
![image](https://user-images.githubusercontent.com/34156840/142477371-e10d684f-afa6-49ad-94f0-de25e88bda0a.png)  
이 둘은 계층 구조가 서로 병렬적으로 구성이 되어있다.  
둘다 시작이 추상클래스 + 확장하는 구상 클래스들을 가지고 있다.  
또한 뉴욕이나 시카고 같은 분점에 대한 구체적인 구현들은 구상 클래스들이 책임을 지고 있다.  
***
# 팩토리 메소드 패턴 정의
> 팩토리 매소드란 객체를 생성하기 위한 인터페이스를 정의하는데, 어떤 클래스의 인스턴스를 만들지는 서브 클래스에서 결정을 한다. 즉 **이 패턴을 이용하면 클래스의 인스턴스를 만드는 일을 서브클래스에게 맡기는 것이다.**  

***
# 객체 의존성
객체 인스턴스를 직접 만들게 되면 구상 클래스에 의존해야 한다.  
앞의 PizzaStore초창기를 보면 답이 나온다.  
그래서 우리는 이 구상 클래스에 대한 **의존성을 줄이는** 것이 좋다는 것을 이해하였다.  
그것이 바로 
**디자인 원칙**
> 추상화된 것에 의존하도록 만들어라. 구상 클래스에 의존하도록 만들지 마라. => **의존성뒤집기 원칙**  
> 여기엔 고수준 구성요소가 저수준 구성요소에 의존하면 안된다는 의미를 내포 => 항상 추상화에 의존  

이 원칙을 적용시킨게 우리가 후에 작성한 방법이다
그리 만들면 PizzaStore는 Pizza라는 추상 클래스에만 의존  
다른 각종 구상 피자들도 Pizza라는 추상 클래스를 의존  
그래서 의존성이 위에서 아래로만 가는게 아니라 뒤집혀진다  
***
# 추상화 팩토리 패턴
자 이젠 원재료들에 관해 품질관리를 해봅시다  
모든 분점이 좋은 재료를 사용하게 하는 방법이 무엇일까요? => 원재료 생산하는 공장에서 분점까지 배달을 하면 된다  
그러나 분점마다 만드는 피자들이 다르니 동일한 재료로 배달하는건 안되므로 이 방법이 안맞게 된다  
이렇게 **서로 다른 재료들을 제공하는 원재료군(families of ingredients)를 처리할 방법을 생각해 봐야 한다**  
그래서 각 분점마다 각각의 원재료군을 만들어 생각을 해봅시다  
***
### 구현
우선 모든 원재료를 생산할 팩토리를 위한 interface부터 만들어보면  
```java
public interface PizzaIngredientFactory {
	public Dough createDough();
	public Sauce createSauce();
	public Cheese createCheese();
	public Veggies[] creatVeggies();
	public Pepperoni createPepperoni();
	public Clams createClam();
}
```
이렇게 각 재료별 생성 메소드를 정의 한다  
그 다음 각 지역별 원재료 공장들을 구상화 합니다.  
```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory{

	@Override
	public Dough createDough() {
		// TODO Auto-generated method stub
		return ThinCrustDough();
	}

	@Override
	public Sauce createSauce() {
		// TODO Auto-generated method stub
		return MarinaraSauce;
	}

	@Override
	public Cheese createCheese() {
		// TODO Auto-generated method stub
		return ReggianoCheese();
	}

	@Override
	public Veggies[] creatVeggies() {
		Veggies veggies[] = {new Garlic(), new Onion(), new Mushroom(), new RedPepper()};
		return veggies;
	}

	@Override
	public Pepperoni createPepperoni() {
		// TODO Auto-generated method stub
		return SlicedPepperoni();
	}

	@Override
	public Clams createClam() {
		// TODO Auto-generated method stub
		return new FreshClams();
	}
	
}

```
자 이제 Pizza 클래스에서도 원재료만 사용하도록 코드를 수정만 하면된다.  
```java
public abstract class Pizza {
	String name;
	String dough;
	String sauce;
	Veggies veggies[];
	Cheese cheese;
	Pepperoni pepperoni;
	Clams clam;
	
	abstract void prepare() {}
	
	void bake() {
		System.out.println("Bake for 25 minutes at 350");
	}
	void cut() {
		System.out.println("Cutting the pizza into diagonal slices");
	}
	void box() {
		System.out.println("Place pizza in official PizzaStore box");
	}
	public String getName() {
		return name;
	}
	void setName(String name) {
		this.name = name;
	}
	public String toString() {
		//Pizza name
	}
	
}

```
자세히 보면 prepare메소드만 abstract로 변한건 빼고는 바뀐점이 없다  
즉 앞에서 배웠던 것처럼 저 메소드를 추상 메소드로 활용하여 Pizza의 구상 서브클래스들이 작업을 하게 됩니다.  
```java
public class CheesePizza extends Pizza{
	PizzaIngredientFactory ingredientFactory;
	public public CheesePizza(PizzaIngredientFactory ingredientFactory) {
		this.ingredientFactory = ingredientFactory;
	}
	@Override
	void prepare() {
		// TODO Auto-generated method stub
		System.out.println("Preparing "+ name);
		dough = ingredientFactory.createDough();
		sauce = ingredientFactory.createSauce();
		cheese = ingredientFactory.createCheese();
	}
	
	
}
```
이런식으로 말이다.  
그럼 피자가게를 다시 살펴보면
```java
public class NYPizzaStore extends PizzaStore{
	@Override
	Pizza createPizza(String type) {
		Pizza pizza = null;
		PizzaIngredientFactory ingredientFactory =new NYPizzaIngredientFactory(); 
		if(type.equals("cheese")) {
			pizza = new NYStyleCheesePizza();
			pizza.setName("New York Style Cheese Pizza");
		}
		else if(type.equals("veggie")) {
			pizza = new NYStyleVeggiePizza();
			pizza.setName("New York Style Veggie Pizza");
		}
		else if(type.equals("clam")) {
			pizza = new NYStyleClamPizza();
			pizza.setName("New York Style Clam Pizza");
		}
		else if(type.equals("pepperoni")) {
			pizza = new NYStylePepperoniPizza();
			pizza.setName("New York Style Pepperoni Pizza");
		}
		else
			return null;
		return pizza;
	}
	
}
```
이렇듯 피자를 각 원재료 공장에서 받아 만들어진 것을 공급하게 됩니다.  
다시한번 정리해봅시다  
아까전의 추상 메소드 방식하고의 차이점은 이번엔 인터페이스를 이용하여 작업을 하였습니다.  
그래서 각 지역별 재료를 만드는 공장이 interface PizzaIngredientFactory를 보면서 작성하게 된다  
고로 이렇게 짜면 제품을 생산하는 실제 팩토리와 분리를 시킬수 있다는 장점이 있다  
***
# 추상 팩토리 패턴 정의
> 추상 팩토리 패턴에선 인터페이스를 이용하여 서로 연관된, 또는 의존하는 객체를 구상 클래스를 지정하지 않고도 생성 가능하다

그래서 이를 사용시 클라이언트에서 추상 interface를 통해 제품들을 공급받으며, 어떤 제품이 생산되는지는 알 필요가 없다. => 이렇게 분리를 한다.  
![image](https://user-images.githubusercontent.com/34156840/142558628-5071f21c-3d3e-4dae-8a7c-e0f102d97dca.png)  
이것이 그 클래스 다이어그램이다.
***
### 추상 팩토리를 보면 거기에 쓰이는 메소드 들은 팩토리 메소드 패턴이 아닌가?
=> 당연히 종종 그런경우가 많다. 인터페이스의 각 메소드는 구상 제품을 생산하는 일을 하고, 추상 팩토리의 서브클래스를 만들어서 각 메소드의 구현을 하니  
팩토리 메소드 패턴을 쓰는건당연하다
# 추상 팩토리 패턴 vs 팩토리 메소드 패턴
둘은 진짜 많이 비슷하고, 실제로 지금 공부하는데도 햇갈리는 부분이 많습니다.  
1. 추상 팩토리 패턴은 객체를 써서 제품을 생성 vs 팩토리 메소드 패턴에선 클래스를 써 제품을 생성
2. 공통점으로 둘다 클라이언트와 구상 형식을 분리시켜 준다(둘다 구상 형식은 서브클래스에서)
3. 추상 팩토리 패턴같은 경우 제품의 추가나 변경이 있을경우 interface변경이라는 단점이 존재
4. 추상 팩토리 패턴은 많은 제품을 생산 가능 vs 팩토리 메소드 패턴은 한가지 제품만 생산(수많은 원재료들 vs 피자하나)
5. 추상 팩토리 패턴에서 팩토리 메소드 패턴을 사용하는 경우가 종종 있다
# 정리하자면
>추상 팩토리 패턴 - 클라이언트에서 연관된 일련의 제품군을 만들때 활용하자  
>팩토리 메소드 패턴 - 클라이언트 코드와 인스턴스를 만들어야 할 구상 클래스를 분리 시켜야 할 때 활용하자 + 어떤 구상 클래스를 필요할 지 미리 알지 못하는 경우 사용하자
>팩토리 메소드 패터는 서브클래스를 만들고 팩토리 메소드만 구현만 하면 


# 공부하면서 느낀점
솔직하게 말해서 처음으로 패턴을 공부하는데 두가지 방법이 동시에 비슷하게 나왔다. 처음에 이것을 이해하는것이 많이 힘들었고, 지금도 만약에 분리해봐 라고 하면 잘 할 자신은 없다.  
이 패턴은 공부가 더 필요할 것 같고 좀더 찾아봐야 할것 같다.  
하지만 객체 생성을 분리시키는 것을 봤을 때 아 이런 자주 바뀌는 부분 중 객체 생성까지도 분리가 가능하구나를 깨닫게 되었다.  
이 패턴이 앞에서 배운 데코레이터 패턴을 위해 사용한다라는데 어떻게 활용되는지는 좀 알아봐야 할듯 하다.



