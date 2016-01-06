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

###　复合匹配
可以使用&& || !分别表示多个匹配条件的且、或和非的逻辑关系。
以下匹配条件将匹配Waiter及其派生类中的serveTo函数。
```java
@Before("execution(* serveTo(..)) && target(aspect.Waiter)")
```
