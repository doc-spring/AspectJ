### 匹配注解
可以使用`@annotation()`切点函数来匹配注解。所有含有指定注解的方法将被匹配，进而被植入横切逻辑；所有不含有指定方法的方法将被忽略。
```java
@AfterReturning("@annotation(aspect.FunAnnotation)")
```
所有被标注了FunAnnotation注解的方法将被匹配。
```java
public class NaughtyWaiter implements Waiter {

	@FunAnnotation
	@Override
	public void greetTo(String name) {
		System.out.println("greet to " + name + "\n");
	}
	
	@FunAnnotation
	@Override
	public void serveTo(String name) {
		System.out.println("serve to " + name + "\n");
	}

}
```

### 匹配方法
`execution()`切点函数用来匹配方法最为全面，既可以匹配方法名，也可以匹配函数值类型、参数类型，还可以匹配包名、类名。

#####1. 匹配方法名为serveTo，返回值任意，参数任意的方法。
```java
@Before("execution(* serveTo(..))")
```

#####2. 匹配方法名含有serve，返回值任意，参数任意的方法。
```java
@Before("execution(* *serve*(..))")
```
#####3. 匹配方法名为serveTo，返回值任意，参数为(String, int)的方法。
```java
@Before("execution(* serveTo(String, int))")
```

#####4. 匹配方法名为serveTo，返回值任意，第一个参数为String，第二个参数任意的方法。
```java
@Before("execution(* serveTo(String, *))")
```
>> **Tips：** 对于不在java.lang包下的类，需要使用全限定类名，如**java.util.List**

#####5. 匹配方法名为serveTo，返回值任意，第一个参数为String，剩下参数个数和类型任意的方法。
```java
@Before("execution(* serveTo(String, ..))")
```

#####6. 匹配方法名为serveTo，返回值任意，参数为Object或其子类类型的方法。
```java
@Before("execution(* serveTo(Object+))")
```
>> **Tips：** 对于
```java
@Before("execution(* serveTo(Object))")
```
其匹配的方法的参数将**仅仅是Object类型**，不包其子类类型。

#####7. 匹配类名为aspect.NaiveWaiter，返回值任意，参数任意的类的所有方法。
```java
@Before("execution(* aspect.NaiveWaiter.*(..))")
```

#####8. 匹配aspect包及其子孙包下，方法名含有serve，返回值任意，参数任意的类的方法。
```java
@Before("execution(* aspect..*.*serve*(..))")
```


### 仅匹配参数
如果我们仅仅需要匹配方法的参数，那么使用`args()`将更加简单。
```java
@Before("args(String)")
```
将匹配所有参数类型为String的方法。

### 仅匹配类名
如果我们仅仅需要匹配类名，意图将横切逻辑注入到匹配的类的所有方法中，那么使用`within()`或者`target()`将更加简单。

#####1. 匹配类名为NaughtyWaiter的所有方法
```java
@Before("within(aspect.NaughtyWaiter)")
```

#####2. 匹配类名为Waiter及Waiter所有子类的所有方法
```java
@Before("within(aspect.Waiter+)")
```
>> **Tips：** 上面的写法与`@Before("target(aspect.Waiter)")`完全等价。也就是说`within()`是精确类名匹配，而`target()`是匹配类名及其派生类。

#####3. 匹配aspect包下的所有类的所有方法
```java
@Before("within(aspect.*)")
```

#####4. 匹配aspect包及其子孙包下的所有类的所有方法
```java
@Before("within(aspect..*)")
```

### 复合匹配
可以使用&&、||、和!分别表示多个匹配条件的且、或和非的逻辑关系。
以下匹配条件将匹配Waiter及其派生类中的serveTo函数。
```java
@Before("execution(* serveTo(..)) && target(aspect.Waiter)")
```

### 访问连接点信息
#### 利用JointPoint和ProceedingJointPoint访问连接点信息
`JointPoint`可以用于横切逻辑函数访问连接点信息之用，其包括以下常用方法
```java
Object[] getArgs()	//获取参数列表
Signature getSignature()	//获取方法签名
Object getTarget()	//获取Target对象
Object getThis()	//获取代理对象本身
```
`ProceedingJointPoint`继承自`JointPoint`，通常用于环绕增强(`@Arround`)。其主要新增以下方法
```java
Object proceed() throws Throwable	//通过反射执行Target对象在连接点处的方法
```
以下是使用`ProceedingJointPoint`访问连接点信息的例子，总可以通过将横切逻辑函数的第一个参数声明为`JointPoint`来获取连接点信息。
```java
	@Around("execution(* serveTo(..)) && target(aspect.NaiveWaiter)")
	public void greet(ProceedingJoinPoint joinPoint) throws Throwable {
		System.out.println("---joinPoint---");
		
		System.out.println("arg[0] = " + joinPoint.getArgs()[0]);
		System.out.println("target = " + joinPoint.getTarget().getClass());
		
		joinPoint.proceed();
		
		System.out.println("---joinPoint---");
	}
```
运行结果如下：
```
---joinPoint---
arg[0] = Jack
target = class aspect.NaiveWaiter
serve to Jack with price 1000
---joinPoint---
```

#### 绑定连接点方法传入参数
你可能会疑惑既然`execution`那么强大，为什么还要用`args`呢？仅仅是为了一种简化？事实上`args`可以用来绑定连接点方法传入参数，从而使得横切逻辑函数可以很容易访问连接点方法的传入参数。以下给出一个例子。
```java
	@Before("execution(* serveTo(..)) && args(name, price)")
	public void greet(String name, int price) throws Throwable {
		
		System.out.println("name = " + name);
		System.out.println("price = " + price);
		
	}
```
`@Before("execution(* serveTo(..)) && args(name, price)")`通过`void greet(String name, int price)`获知`name`和`price`的类型，从而将匹配方法名为`serveTo`，参数为(String, int)，返回值任意的方法。并试图植入横切逻辑。`args`的强大之处在于它绑定到了连接点方法的传入参数，这使得我们直接可以在`greet(String name, int price)`横切逻辑函数中方便地使用连接点方法的传入参数。
因此以上程序的运行结果如下
```
name = Jack
price = 1000
serve to Jack with price 1000
```

#### 绑定连接点方法返回值
通常用在`AfterReturning`横切类型中，通过`AfterReturning`的`returning`指定返回值对象。
```java
	@AfterReturning(value="target(aspect.Seller)", returning="retValue")
	public void bindReturnValue(int retValue) {
		System.out.println("return value = " + retValue);
	}
```

#### 绑定抛出的异常
必须使用在`AfterThrowing`横切类型中，通过`AfterThrowing`的`throwing`指定抛出异常对象。
```java
	@AfterThrowing(value="target(aspect.Seller)", throwing="exception")
	public void bindException(IllegalArgumentException exception) {
		System.out.println("exception:" + exception.getMessage());
	}
```
以上切面将匹配aspect.Seller类运行时抛出异常类型为`IllegalArgumentException`的方法。以下为运行结果
```
sellBeer()
checkBill
exception:illegal argument
```

#### 绑定proxy对象
可以使用`this()`或者`target()`来绑定proxy对象，此时切点函数仍然具有匹配功能。以下为一个例子
```java
	@Before("target(waiter)")
	public void bindProxyObj(Waiter waiter) {
		System.out.println(waiter.getClass().getName());
	}
```
以上切面将匹配所有waiter类或其子类的方法。并且这样的proxy对象将通过`waiter`对象很容易的在横切逻辑函数中被访问。
```
aspect.NaiveWaiter
greet to Jack

aspect.NaiveWaiter
serve to Jack

aspect.NaiveWaiter
serve to Jack with price 1000

aspect.NaughtyWaiter
greet to Tom

aspect.NaughtyWaiter
serve to Tom

sellBeer()
checkBill
```
