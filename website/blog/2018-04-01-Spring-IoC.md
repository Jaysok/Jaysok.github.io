---
title: Inversion of Control (IoC)
tags: spring, ioc, di, design pattern, dependency injection, inversion of control, architecture
author: jaysok
created: 2018-04-01
modified: 2018-06-14
---



# IoC (Inversion of Control)

제어의 역전이라고 번역된다. 말도 어렵고 실제로 개념도 처음엔 잘 와닿지 않는 것 같다. 

Refactoring의 저자인 Martin Fowler가 자신의 글에서 남긴 부분에 발췌했다. 

>When these containers talk about how they are so useful because they implement "Inversion of Control" I end up very puzzled. [Inversion of control](https://martinfowler.com/bliki/InversionOfControl.html) is a common characteristic of frameworks, so saying that these lightweight containers are special because they use inversion of control is like saying my car is special because it has wheels.
>
>The question is: "what aspect of control are they inverting?" When I first ran into inversion of control, it was in the main control of a user interface. Early user interfaces were controlled by the application program. ( 중략 ) With graphical (or even screen based) UIs the UI framework would contain this main loop and your program instead provided event handlers for the various fields on the screen. **The main control of the program was inverted, moved away from you to the framework.**
>
>As a result I think we need a more specific name for this pattern. Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various IoC advocates we settled on the name *Dependency Injection*.

대충 이렇다. 

1. 우리 프레임워크에서는 IoC를 지원해줘! 그러니까 좋아! 라고 말하는 것은, 내 자동차는 좋아! 왜냐하면 바퀴가 있거든! 이라고 말하는 것이랑 비슷하다. 
2. 그가 처음 경험해본 IoC라는 것은 초창기의 사용자 인터페이스 라던가, UI 프레임워크에서 메인 루프 프로그램이 돌면서 화면에서 사용자의 입력을 기다리는... 뭐 그런 종류의 것들이다. 이들이 갖는 특성은, 프로그램의 흐름이 나에게서 프레임워크로 넘어가는 순간이 있다는 사실이다. 
3. IoC라는 말은 너무 광범위하다. 따라서 오히려 사람들이 헷갈려한다. 따라서 Dependency Injection이라는 말이 좀 더 전달하고자 하는 바에 가깝다고 본다.



아무튼 결론은 일하다. Spring에서 말하는 IoC를 이해하기 위해서는 DI를 반드시 이해해야한다는 점이다. 

[이 글](https://martinfowler.com/articles/injection.html#InversionOfControl)은 아주 유명한 글로 한 번은 읽어봄직하다.



## Dependency 

코드 간에는 dependency가 존재한다. 당연하지만, 코드가 서로 간에 잘 동작하기 위해서는 맞물리는 부분이 반드시 필요하게 되고 때문에 dependency를 없애는 것은 불가능하다. 일반적으로 의존성이 높아질 수록 응집도가 낮아지고 결합도는 올라가고 반대 역시 성립한다. 응집도를 높이는 이유는 유지보수성과 코드의 확장성 때문이다. 달리말하면 의존성을 낮추는 작업은 코드의 유지보수성과 코드의 확장성을 높이기 위함이다. 

본격적으로 이야기를 하기에 앞서 좀 더 일반론에 가까운 이야기를 해보자. 

> A가 B에 의존한다. 

어떻게 하면 A가 B에 가지는 의존성을 줄일 수 있을까? 이 질문에 대답을 하려면 A가 B에 의존한다는 것에 대한 정의부터 시작을 해야한다. 왜냐면 의존성을 줄일 대상을 특정하지 못했기 때문이다. 의존한다는 것에 대해서 정의해보면서 의존성을 줄일 대상이 무엇인지 생각해보자. 

>  A는 B가 가진 어떤 부분이 필요하다

A는 B가 *필요하다*라는 말은 A가 자신 안에서 B를 *사용한다*는 말이다. 그런데 여기서 잠시 확인하고 가야할 부분이 있다. 여기서 우리가 암묵적으로 전제하고 있는 사실은 A가 B에 대한 의존성이 높다는 것이었다.  이점을 인지하고 다시 하려던 말을 해보겠다. 

### 사용한다

*사용한다*는 말은 B가 어떻게 만들어졌는지와는 직접적으로 관계가 없다. 우리는 컴퓨터를 사용한다. 컴퓨터가 어떻게 만들어졌는지는 관심도 없고 알고 싶지도 않다. 그렇지만 우리는 컴퓨터를 사용할 수 있다. 이유는 컴퓨터가 제공해주는 인터페이스를 통해서 접근하기 때문이다. 

즉, A는 B가 가진 어떤 부분을 사용만 할 수 있으면 된다. 그 이외의 모든 부분에는 관심자체를 두지 않으면 된다. 이쯤 되면 느낌이 조금 올 것이다. A는 B의 인터페이스에만 접근할 수 있으면 된다. 그런데 의존성이 높았다는 말은 A가 B의 인터페이스만 알면되는데, 그 이외의 정보를 너무 많이 알고 있기 때문에 생기는 일이다. 

따라서 의존성을 최대로 낮출 수 있는 지점은 A가 B를 사용하는데 필요한 정보만 알고 있는 지점일 것이다. 그 이상으로 의존성을 낮추면 B를 제대로 사용하지 못하는 문제가 생기고 그 이상으로 알고 있으면 의존성이 높아진다. 

그러면 지금까지 한 이야기를 정리해서 다음의 문장을 만들어봤다.

> A는 인터페이스를 통해서만 B를 사용한다.

A는 인터페이스에는 의존하지만 그건 정말 최소한의 의존일 뿐이다. 

B는 인터페이스에 A가 사용할 부분만 제공해주면 되는 것이다. 

A는 B에 대해서 알 필요가 없다. A는 인터페이스만 알고 있으면 된다.

 

여기까지는 일반론이다. 이제부터는 코드를 가지고 이야기해보자.



```java
class A {
    GoodInterfaceForB instance;
    public A(){
	    instance = new B();         
    }
    public work() {
        instance.use();
    }
}

interface GoodInterfaceForB {
    void use();
}

class B implements GoodInterfaceForB {
    @Override
    public void use() {
        // do something
    }
}
```

위의 코드는 의도적으로 안 좋게 작성한 예시이다. 

A는 B를 인터페이스를 통해서 사용하고 있긴 하지만 여전히 의존성이 높다. 왜그럴까?

A가 B의 *생성*에 관여하고 있기 때문이다. 별 달리 좋은 말이 생각나지 않아서 그냥 생성이라고 표현했는데, 아무튼 핵심은 이것이다. 

>  A가 B의 존재를 확보하는데 관여했다. 

A가 B의 존재를 확보하는데 관여하고 있다보니 B가 어떻게 만들어져야 되는지 여전히 알아야되는 문제가 생겨버린 것이다. 

### 생성한다

앞에서 우리가 전제했던 것은 *이미 우리가 B를 가지고 있다*라는 점이다. 그런데 코드로 직접 작성을 해보니 느껴질 것이다. B를 사용해야하므로 B를 어떤 방식으로든 만들어와야 한다는 점이다.  

그럼 B가 A의 생성에 관여하지 않게 코드를 바꿔보자.

```java
class A {
    GoodInterfaceForB instance;
    public A(B b){
	    instance = b;         
    }
    public work() {
        instance.use();
    }
}

interface GoodInterfaceForB {
    void use();
}

class B implements GoodInterfaceForB {
    @Override
    public void use() {
        // do something
    }
}
```
이제 정말로 A는 B에 거의 알지 못한다. A는 B를 외부로 부터 전달받을 것이고 생성에 관여하지 않게 됐다. 

그러면 이제 생성에 대한 책임 소재 자체가 A에게 있지않다. 그러면 이걸 누군가 생성을 해줘야할 것인데, 그 일을 해주는 것이 소위 말하는 IoC Container가 해주는 일이다. 



아래의 코드를 보자.

```java
public class App {
    public static void main(String[] args) {
        new App().start();
    }
    void start() {
        ApplicationContext factory = new ClassPathXmlApplicationContext("context.xml");
        GoodInterfaceForA a = (GoodInterfaceForA)factory.getBean("A");
        a.work();
    }
}

interface GoodInterfaceForA {
    void work();
}

class A implements GoodInterfaceForA {
    @Override
    void work();
}
```

이제 A를 만드는 책임소재는 factory에 있다. 심지어 위의 코드에서 B의 인스턴스는 보이지 조차 않는다. 

이처럼 IoC Container는 그 안에서 정해진 설정에 따라서 객체를 생성해서 넣어주고, 공통적인 어떤 특정한 작업(AOP)을 우리가 작성한 코드를 통과하게끔 만들 수도 있다. 이러한 기반을 제공해주는 것이 IoC Container인 것이다. 

위에서는 A를 인터페이스를 상속받게 리팩토링 했는데, 이와 같은 작업을 통해서 `a` 자리에 들어올 인스턴스를 셋팅만 바꿔서 갈아끼울 수 있게 된다. 

## DL ( Dependency Lookup )

#### `Main.java`

```java
public static void main(String[] args) {
    	ClassPathXmlApplicationContext factory = new ClassPathXmlApplicationContext("ioc.xml");
		Tool tool = (Tool) factory.getBean("tool");		
		tool.work();		
		factory.close();
}
```

#### `Tool.java`

```java
public interface Tool {
    void work();
}
```

#### `Fork.java`

```java
public class Fork implements Tool {
    pubic void work() {
        System.out.println("Fork works!");
    }
}
```

#### `ioc.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="tool" class="model.Fork"></bean>
</beans>
```

위에서 `Main.java`에서는 컴파일시에 `Tool tool`자리에 어떤 인스턴스가 들어와서 박히는지 알 수 없다. 그렇지만, `Tool`이라는 인터페이스로 받고 있기 때문에, `work`이라는 메서드가 동작할 것이라는 것을 보장받을 수 있다. 

어떤 객체가 들어와서 박히는지는 `ioc.xml`을 통해서 전달이 될 것이다. 이 때 `ClassPathXmlApplicationContext` 타입의 `factory` 객체가 `ioc.xml` 에서 전달받은 쿼리를 가지고 해당하는 키에 따른 값이 있는지 `ioc.xml`을 찾아본다. 때문에  *dependency lookup*이라고 불린다.  

## DI (Dependency Injection)

#### `Main.java`

```java
public static void main(String[] args) {
    	ClassPathXmlApplicationContext factory = new ClassPathXmlApplicationContext("ioc.xml");
		Worker worker = (Worker) factory.getBean("worker");		
		worker.work();		
		factory.close();
}
```

#### `Worker.java`

```java
public class Worker {
	String name;
	Tool tool;
    ...
	public void setTool(Tool tool) {
		this.tool = tool;
	}
	public void work() {
		System.out.println(this.getName() + "일한다!!!");
	}
}
```

#### `ioc.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
    <bean id="tool" class="model.Fork"></bean>
    <bean id="worker" class="model.Worker">
    	<property name="name" value="아이유" />
        <property name="tool" ref="tool" />
    </bean>
</beans>
```

`Main.java`에서는 먼저 `worker`를 DL로 가져온다. 그런데 여기서 중요한 점은, DL로 가져온 `worker`객체에는 이미 `Tool` 인터페이스를 구현하고 있는 `Fork` 객체가 주입되어있다. 

`ClassPathXmlApplicationContext` 클래스가 객체가 많은 마법을 부린다. 때문에 어떤 일이 일어나는지 상세하게 알 수는 없으나 추측할 수 있는 바는 `worker`객체에 `Fork` 클래스의 인스턴스를 하나 넣어준다는 점이다.