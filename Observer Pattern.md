# opserver pattern
옵저버 패턴은 객체에게 연락망을 돌리는 거랑 비슷하다 생각할 수 있다  
**뭔가 중요한 일이 일어났을 때 객체들한테 새소식을 알려줄 수 있는 패턴이다.**  
또한 여기선 일대다 관계와 느슨한 결합에 대해서도 나타납니다.  

***
자 당신은 차세대 인터넷 기반 기상 정보 스테이션 구축 프로젝트를 수행하게 되었음을 알았습니다.  
기상정보 스테이션은 WeatherData객체를 바탕으로 만들어질 것이고,  
이 객체는 현재 기상 조건(기온, 습도, 기압)을 추적하는 객체이다.  
기본적으로 세가지 항목을 화면에 표시할 것인데, 세 항목은 현재 조건, 기상 통계, 간단한 기상 예보이다.  
이 3가지 항목은 모두 WeatherData 객체에서 최신 측정치를 수집할때 **실시간**으로 갱신된다.  
또한 이 스테이션은 **확장 가능한 스테이션**이다.  
다른 개발자들이 직접 날씨 표시 장치를 만들고 거기에 개발한 것을 그대로 가져다 쓸 수 있게 **API**를 발표할 예정이다.  
물론 그 API도 개발해야한다.  
또한 고객이 확보되고 나면 정보가 화면에 표시되는 회수를 바탕으로 요금을 부과할 예정이다.  

***
이 시스템은 기상스테이션(실제 기상 정보 수집->센서등으로)과 WeatherData객체(기상 스테이션에서 오는 데이터를 추적하는 객체), 사용자에게 현재 기상조건을 보여주는 디스플레이로 3가지 요소로 이루어진다.  
  
  WeatherData객체에선 기상 스테이션 장비로부터 데이터를 가져올 수있고, 데이터를 가져온 후 디스플레이 장비에 3가지 항목을 표시할 수 있다.  
1. 현재 조건(온도, 습도, 압력)
2. 기상 통계
3. 간단한 기상 예보

  WeaterData클래스
```java
  public class WeatherData{
    getTemperature(); //최근에 측정된 온도
    getHumidity(); //습도
    getPressure(); //압력 
    //----------------- 이들은 WeatherData가 알아서해주기에 걱정하지 않아도 된다.
    measurementsChanged() // 기상 관측값이 갱신될때마다 알려줄 메소드
    //기타메소드
  }
```
**measurementsChanged()를 현재 조건, 기상통계, 기상 예측** 이렇게 세가지 디스플레이를 갱신할 수 있도록 구현해야 한다.  

  일단 분석을 해보면 (앞의 getter들은 무시) 새 측정 데이터가 나올때마다 measurementsChanged 메소드가 호출된다.  
  기상데이터를 사용하는 세개의 디스플레이 항목을 구현.  
  1. 현재 조건 표시
  2. 기상 통계 표시
  3. 기상 예보 표시
  이들을 **새로운 측정값이 들어올 때마다 디스플레이에 갱신** 해야만 한다.
  또 시스템이 확장 가능해야한다.   
  다른 개발자들이 별도의 디스플레이 항목을 만들 수 있게하고, 사용자들이 마음대로 디스플레이 항목을 추가/제거가 가능하게 해야한다.  
  그런데 현재 상황에선 세가지 기본 디스플레이 형식만 알고 있다.  
  ```java
    public class WeatherData{
      float temp = getTemperature();
      float humidity = getHumidity();
      float pressure = getPressure();
      //디스플레이 갱신
      //각 애들에게 최신 측정 값을 전달
      currentConditionDisplay.update(temp, humidity, pressure); 
      statisticsDisplay.update(temp, humidi9ty, pressure);
      forecastDisplay.update(temp, humidity, pressure); 
      
    }
  ```
  일반적으로 이렇게 작성 할 수 있다.
  ***
  이 코드에 무슨 문제가있는가  
currentConditionDisplay.update(temp, humidity, pressure);   
statisticsDisplay.update(temp, humidi9ty, pressure);  
forecastDisplay.update(temp, humidity, pressure);   
요 코드들은 **구체적인 구현에 맞춰서 코딩을 했기에**, 프로그램을 고치지 않고는 **다른 디스플레이 항목을 추가/제거 불가**  
그래서 이들을 생각해보면 바뀔 수 있는 부분이다. 즉 **캡슐화 해야한다**  

***
### 옵저버 패턴
신문이나 잡지를 어떤 식으로 구독하는가?
1. 신문사가 사업을 시작하고 신문을 찍어낸다  
2. 독자가 특정 신문사/잡지사에 구독 신청을 하면 매번 새로운 신문/잡지가 나올 때마다 배달을 받을 수 있다. 계속 구독자로 남아있는 다면 계속 신문/잡지를 받습니다.
3. 신문을 더 이상 보고 싶지 않다면 구독 해지신청을 합니다. 그럼 더 이상 신문이 오지 않는다.
4. 신문사가 계속 영업을 하는 이상 여러 개인 독자, 호텔, 항공사 및 기타 회사 등에서 꾸준히 구독 및 해지를 하게 된다.

즉 **출판사+구독자=옵저버 패턴**  
축판사를 **주제(subject)**, 구독자를 **옵저버(observer)** 라고 부르는 것만 알아두자  
즉 subject객체는 일부 데이터를 관리하는데, subject의 데이터가 달라지면 옵저버한테도 그 소식이 다가간다.  
옵저버들은 주제 객체를 구독하고 있고(등록되어 있다) 주제의 데이터가 바뀌면 갱신 내용을 전달 받는다.  
예를들어 Duck객체는 옵저버가 아니므로 주제객체의 데이터가 바뀌어도 아무 연락을 받지 않는다.  
만약 Duck이 이 주제 객체에 자기도 옵저버가 되고 싶다하면 포함해줍니다. 물론 탈퇴를 하고 싶다고 하면 탈퇴도 가능합니다.  
**또한 누군가의 옵저버이면서 주제일수도 있다.(동시에 해당 가능)**  
### 옵저버 패턴 정의
> **옵저버 패턴에서는 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고, 자동으로 내용이 갱신되는 방식으로 일대다 의존성을 정의한다**  

일대다관계는 주제와 옵저버에 의해 정의가 된다.  
옵저버는 주제에 의존하고, 주제의 상태가 변하면 옵저버들한테 연락이 간다.  
연락방법에 따라 옵저버의 값이 새로운 값으로 갱신될 수도 있다.  
옵저버 패턴을 구현하는 방법은 여러가지가 있지만, 대부분 subject와 observer인터페이스가 들어있는 클래스 디자인을 바탕으로 한다.  

```java
  interface Subject{
    registerObserver() // 옵저버 등록
    removeObserver() // 옵저버 제거
    notifyObserver()
  }
```
.  

```java
  interface Observer{
    update() // 옵저버가 될 가능성이 있는 객체에서는 무조건 이 인터페이스를 구현해야만 한다. 주제의 상태가 바뀌었을때 호출되는 update메소드만 있습니다.
  }
```
**주제 역할을 하는 구상 클래스에서는 항상 Subject인터페이스를 구현해야한다.**  
> **등록 및 해지를 위한 메소드 위에 상태가 바뀔때마다 모든 옵저버들에게 연락을 하기위한 notifyObservers메소드도 구현해야한다**  
```java
  class ConcreteSubject implements Subject{
      registerObserver(){...}
      removeObserver(){...}
      notifyObserver(){...}
      //주제 클래스에서도 상태를 설정하고 알아내기 위한 setter/getter메소드가 들어갈 수 있다.
      getState()
      setState()
  }
```
**옵저버 인터페이스만 구현한다면 무엇이든 옵저버 클래스가 될 수 있다.**  
**각 옵저버는 특정 주제 객체에 등록을 해서 연락을 받을 수 있다**  
```java
  class ConcreteObserver implements Observer{
    update()
    //기타 옵저버용 메소드
  }
```
#### Question
일대다 관계인 이유는?
> 옵저버 패턴에서 상태를 저장하고 지배하는 것은 subject객체이다. 그래서 상태가 들어있는 객체는 **하나만** 있을 수 있다.  
> 그러나 옵저버는 사용하긴 하지만 반드시 상태를 가지고 있어야 하는 것은 아니다. 따라서 옵저버는 **여러개** 존재 가능하고, 주제 객체에서 상태가 바뀜을 기다리는 주제에 의존적인 성질이도니다.  
> 그래서 하나의 주제와 여러개의 옵저버가 연관된 일대다 관계가 성립한다

의존성이 내용이랑 상관이 있는가?
> 데이터의 주인은 **주제**이다. 
> 옵저버는 데이터가 변경되었을 때 주제에서 갱신해 주기를 기다리는 입장이기에 의존성을 가진다 할 수 있다.
> 이로서 여러 객체에서 동일한 데이터를 제어하도록 하는 것보다 더 깔끔한 객체지향 디자인을 할 수 있다.

***
### 느슨한 결합
두 객체가 느슨하게 결합되어 잇다는 것은 그 둘이 **상호작용을 하긴 하지만** **서로에 대해 서로 잘 모른다는 것을 의미**한다.  
옵저버 패턴에서는 **주제와 옵저버가 느슨하게 결합되어** 있는 객체 디자인을 제공한다.  

* 주제가 옵저버에 대해 아는 것은 **옵저버가 Observer인터페이스를 구현한다는 것 뿐이다.**  
* 옵저버의 구상 클래스가 무엇인지, 무엇을 하는지 등에 대해서는 알 필요가 없다.
* 옵저버는 언제든지 새로 추가하고 제거가능 하다
* 새로운 형식의 옵저버를 추가할때도 **subject를 전혀 변경할 필요가 없다.**
* 그냥 새로운 클래스에서 인터페이스를 구현하고 옵저버로 등록만 하면 된다
* 주제와 옵저버는 서로 독립적으로 재사용 할 수 있다.
* 주제나 옵저버가 바뀌더라도 서로에게 영향을 미치지 않는다.

이로서  
**디자인 원칙**
> **서로 상호작용을 하는 객체 사이에서는 가능하면 느슨하게 결합하는 디자인을 사용해야 한다**

**느슨하게 결합하는 디자인을 사용하면 변경사항이 생겨도 무난히 처리할 수 있는 유연한 객체 지향 시스템을 구축할 수 있다. 객체 사이의 상호의존성을 최소화하기 때문이다.**  
***
이제 아까의 상황을 다시 생각해보자
옵저버 패턴의 일대다 의존성을 생각해보면 WeatherData클래스가 일, 기상 측정치를 사용하는 디스플레이 항목은 다가 된다.  
그러나 모호한점이 우선 기상 측정 값을 디스플레이 항목에 전달하게 되면 디스플레이에서 자기가 원하는 정보를 얻기 위해서 WeatherData객체에 등록을 하게 된다.  
그러면 기상 스테이션에서 디스플레이 항목을 알고 있으면 메소드 하나만 호출해서 측정 값을 알려 줄 수 있다.  
모든 디스플레이 항목이 다를 수 있다. 이 부분에서 **공통 인터페이스를 사용** 해야 한다.  
구성요소의 형식이 달라도 똑같은 인터페이스를 구현해야만 WeatherData 객체에서 측정값을 보낼 수 있다.
즉 모든 디스플레이에 WeatherData에서 호출할 수 있는 update()메소드가 있어야한다.  
그리고 update()는 모든 항목들이 구현하는 공통 인터페이스에서 정의할 것이다.  

그래서 정리해보면  

  
```java
  interface Subject{
    registerObserver() // 옵저버 등록
    removeObserver() // 옵저버 제거
    notifyObserver()
  }
```
```java
  interface Observer{
    update() // 옵저버가 될 가능성이 있는 객체에서는 무조건 이 인터페이스를 구현해야만 한다. 주제의 상태가 바뀌었을때 호출되는 update메소드만 있습니다.
  }
```
이렇게 subject클래스를 구현하고,
```java
  class WeatherData implements Subject{
      registerObserver(){...}
      removeObserver(){...}
      notifyObserver(){...}
      //주제 클래스에서도 상태를 설정하고 알아내기 위한 setter/getter메소드가 들어갈 수 있다.
      getTemperature()
      gethumidity()
      getPressure()
      measurementsChanged()
  }
```
모든 디스플레이 항목에서 구현하는 인터페이스를 하나 더 만들어, 디스플레이 항목에서는 display() 메소드만 구현할 수 있다.  
```java
  interface DisplayElement{
    display()
  }
```
```java
  class CurrentConditions implements Observer, DisplayElement{
    update()
    display(){
      //현재 측정값을 화면에 표시
    }
  }
```
```java
  class statisticsDisplay implements Observer, DisplayElement{
    update()
    display(){
      // 평균/최저/최고치 표시
    }
  }
```

```java
  class ForecastDisplay implements Observer, DisplayElement{
    update()
    display(){
      // 기상예보표시
    }
  }
```

```java
  class ThirdPartyDisplay implements Observer, DisplayElement{
    update()
    display(){
      // 측정값을 바탕으로 다른 내용 표시
    }
  }
```

***
### 가상 스테이션 구현

```java
  interface Subject{
    public void registerObserver(Observer o); // 옵저버 등록
    public void removeObserver(Observer o); // 옵저버 제거
    public void notifyObserver(); //주제 객체에 상대가 변경되었을 때 모든 옵저버들에게 알리기 위해 호출한다.
  }
```
```java
  interface Observer{
    public void update(float temp, float humidity, float pressure); // 기상 정보가 변경되었을 때 옵저버한테 전달되는 상태 값들이다.
  }
```
```java
  interface DisplayElement{
    public void display();
  }
```
Subject를 구현해보자
```java
public class WeatherData implements Subject{
	private ArrayList<Observer> observers;
	private float temperature;
	private float humidity;
	private float pressure;
	
	public WeatherData() {
		// TODO Auto-generated constructor stub
		observers = new ArrayList();
	}
	
	@Override
	public void registerObserver(Observer o) {
		observers.add(o);
	}

	@Override
	public void removeObserver(Observer o) {
		int i = observers.indexOf(o);
		if(i>=0)
			observers.remove(i);
	}

	@Override
	public void notifyObserver() {
		for(int i=0;i<observers.size();i++) {
			Observer observer = (Observer)observers.get(i);
			observer.update(temperature, humidity, pressure);
		}
		
	}
	
	public void measurementsChanged() {
		notifyObserver();
	}
	public void setMeasurements(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}
	//기타 메소드
}

```

디스플레이 항목을 만들어 보자
```java
public class CurrentConditionsDisplay implements Observer, DisplayElement{
	private float temperature;
	private float humidity;
	private Subject weatherData;
	
	public CurrentConditionsDisplay(Subject weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this); //옵저버로 등록
	}
	
	@Override
	public void display() {
		// TODO Auto-generated method stub
		System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
	}

	@Override
	public void update(float temperature, float humidity, float pressure) {
		// TODO Auto-generated method stub
		this.temperature = temperature;
		this.humidity = humidity;
		display();
	}
}
```
Test Code
```java
  public class WeatherStation {

	public static void main(String[] args) {
		WeatherData weatherData = new WeatherData();
		
		CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);
		
		weatherData.setMeasurements(80, 65, 30.4f);
		weatherData.setMeasurements(82, 70, 29.2f);
		weatherData.setMeasurements(78, 90, 29.2f);
	}
}

```
***
옵저버와 subject간 데이터를 송수신 할때는 **push와 pull이 있다.**  
push는 데이터를 subject에서 보내는 형식(위의 코드)  
pull은 옵저버가 데이터를 가져가는 형식이다.(getter이용)  
둘다 자바에서 내장된 옵저버 패턴이 있기는 하다.  
만약 subject가 확장이 된다면 상태가 더 추가가 될 것이다.  
pull이라면 갱신된 상태를 전송하기 위해 메소드를 일일이 고칠 필요없이 getter메소드만 하나 추가하고 필요한 옵저버가 가져가게 할 수도 있다.  
***
자바 내장에서 옵저버 패턴을 지원을 하기도 한다(push, pull 다 가능)  
Observer interface(java.util)와 Observable클래스를 예로 들 수 있다.
Observer interface는 앞에서 본거랑 유사하지만  
Observable클래스는 클래스이기에 조금 특별합니다.
```java
  class Observable{
    addObserver()
    deleteObserver()
    notifyObserver()
    setChanged()
  }
```
그래서 WeatherData를 Observable에서 확장해야 한다.  
고로 이젠 Observable을 확장하고, 언제 Observer에 연락할지만 알려주면 된다.

  
Observable에서 연락을 돌리는 방법은  
1. 첫 번째로 setChanged() 메소드를 호출하여 객체의 상태가 바뀌었다는 것을 알려준다.
2. 그 다음 notifyObservers() or notifyObservers(Object arg)둘 중 하나를 호출한다. 여기서 arg는 연락할 때 각 옵저버 객체한테 전달할 데이터 객체를 받아들인다  
  
옵저버가 연락을 받는 방법은
update()를 이용하지만 조금 다른게
update(Observable o, Object arg)로서 연락을 보내는 주제 객체가 인자로 들어가고, notifyObservers()메소드에서 인자로 전달된 데이터 객체를 넣는다(없음 null)  
  
push방식을 사용할때는 데이터를 notifyObservers(arg)형태로 전달해야 한다.  
아니면 옵저버 쪽에서 전달받은 Observable객체로부터 원하는 데이터를 가져가는 pull방식을 써야한다.  
setchanged() 메소드는 상태가 바꼈다는 것을 알리기 위한 용도로 사용된다. 그래서 나중에 notifyObservers()메소드가 호출될 때 그 메소드에서 옵저버들을 갱신한다.  
setChanged()가 호출되지 않은 상태라면 notifyObservers가 아무리 호출해도 옵저버들은 아무 연락을 못 받는다.  
그래서 setChanged는 연락을 최적화 시켜주어 옵저버들을 갱신하는 방법에 있어 더 광범위한 유연성을 제공하기 위해 만들어졌다.  
```java
  setChanged(){
    changed = true
  }
  notifyObservers(Object arg){
    if(changed){
      목록에 있는 모든 옵저버들에 대해{
        update(this, arg)
      }
      changed = false
    }
  }
  notifyObservers(){
    notifyObservers(null)
  }
```
만약 자주 쓰는 경우가 생긴다면 clearChanged라는 메소드를 만들어 changed를 false로 만드는 작업을 할 수도 있고  
hasChanged라는 메소드는 changed플래그의 현재 상태를 알려줍니다.  

```java
  import java.util.ArrayList;
import java.util.Observable;

@SuppressWarnings("deprecation")
public class WeatherData extends Observable{
	private float temperature;
	private float humidity;
	private float pressure;
	
	public WeatherData() {
	}
	
	public void measurementsChanged() {
		setChanged();
		notifyObservers();
	}
	public void setMeasurements(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}
	//for pull
	public float getTemperature() {
		return temperature;
	}

	public float getHumidity() {
		return humidity;
	}

	public float getPressure() {
		return pressure;
	}
	
	//기타 메소드
}
```


```java
import java.util.Observable;
import java.util.Observer;

public class CurrentConditionsDisplay implements Observer, DisplayElement{
	private float temperature;
	private float humidity;
	Observable observable;
	
	public CurrentConditionsDisplay(Observable observables) {
		this.observable = observables;
		observable.addObserver(this);
	}

	@Override
	public void display() {
		// TODO Auto-generated method stub
		System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity");
	}



	@Override
	public void update(Observable o, Object arg) {
		if(o instanceof WeatherData) {
			WeatherData weatherData = (WeatherData)o;
			this.temperature = weatherData.getTemperature();
			this.humidity = weatherData.getHumidity();
			display();
		}
		
	}

}

```
하지만 이런식의 사용은 좋지않다  
Observable가 클래스이기에 서브 클래스가 필요하다. 그럼 이미 superclass를 확장하는 클래스에 Observable 기능을 추가를 하지 못한다 -> 재사용성 문제  
또 Observable인터페이스가 없어서 Observer API하고 잘 맞는 클래스를 직접 구현하는 것이 불가능하다  
게다가 Observable API를 잘보면 setChanged() 메소드가 protected로 선언되어 있어서 Observable 서브클래스에서만 setChanged를 호출할 수 있다. 그래서 외부에선 호출할 수가 없다  
그래서 Observable을 확장한 클래스를 쓰는 상황이라면 Observable API를 사용해도되고, 아니면 직접 구현해도 된다.

















