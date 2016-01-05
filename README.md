### 匹配注解
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
1. 匹配方法名为serveTo，返回值任意，参数任意的方法。
```java
@Before("execution(* serveTo(..))")
```

2.匹配方法名含有serve，返回值任意，参数任意的方法。
```java
@Before("execution(* *serve*(..))")
```
3.匹配方法名为serveTo，返回值任意，参数为(String, int)的方法。
```java
@Before("execution(* serveTo(String, int))")
```

4.匹配方法名为serveTo，返回值任意，第一个参数为String，第二个参数任意的方法。
```java
@Before("execution(* serveTo(String, *))")
```

5.匹配方法名为serveTo，返回值任意，第一个参数为String，剩下参数个数和类型任意的方法。
```java
@Before("execution(* serveTo(String, ..))")
```

6.匹配方法名为serveTo，返回值任意，参数为Object或其子类类型的方法。
```java
@Before("execution(* serveTo(Object+))"
```

>> **Tips：** 对于
```java
@Before("execution(* serveTo(Object))"
```
其匹配的方法的参数将**仅仅是Object类型**，不包其子类类型。

