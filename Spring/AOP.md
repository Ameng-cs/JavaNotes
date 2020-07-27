# AOP

==Aspect Oriented Programming(面向切面编程)==.思想是，不去动原来的代码，而是基于原来代码产生代理对象，通过代理的方法，去包装原来的方法，就完成了对以前方法的增强。==AOP的底层原理就是动态代理的实现。==

### AOP的使用场景

- 日志相关操作
- 权限验证
- 安全检查
- 事务控制

### AOP使用步骤

- 导包 

- 写配置           

  ```
  <!--开启基于注解的AOP功能 -->
  <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
  ```

- 编写切面类,在切面类前加上@Aspect注释

  - 在切面类中写通知方法,并且加上切入点表达式

- 将切面类和目标类都加入到SpringIOC容器中

### AOP的五中通知类型

- ==@Before==(excution("权限修饰符 返回值类型 方法签名")): 在目标方法之前运行
- ==@After==(excution("方法签名")):在目标方法结束之后        
- ==@AfterRunturning==(excution("方法签名")):在目标方法正常返回之后运行     
- ==@AfterThrowing==(excution("方法签名")):在目标方法抛出异常之后运行       
- ==@Around==(excution("方法签名")): 环绕通知

### 切入点表达式

- 固定格式:excution(访问权限符 返回值类型 全类名.方法(参数表))

- 通配符:

  - ==*==:

    - 匹配一个或者多个字符:             

      ```
      @Before("execution(public int com.spring.impl.Calculator.*(int,int))") 
      ```

    - 匹配任意一个参数:第一个是int类型,第二个参数任意类型:(匹配两个参数)           

      ```
      @Before("execution(public int com.spring.impl.Calculator.*(int,*))")
      ```

    - 只能匹配一层路径        

    - 权限位置*不能:权限位置不写就行:public[可选项]

  - ==..==    :

    - 匹配任意多个参数和任意类型参数              

      ```
      @Before("execution(public int com.spring.impl.Calculator.*(..))") 
      ```

    - 匹配任意多层路径: 

      ```
      @Before("execution(public int com.spring..*(int,*))")
      ```

  - 我们可以记住这两种:

    - 最精确的:@Before("execution(public int com.spring.impl.方法(参数))")         

    - 最模糊的:@Before("execution(*(任意权限,任意返回值) *.*(..)任意包任意类的任意方法)")(千万别写)

  - &&,||,!
    - 这三个逻辑运算符可以在切入点表达式中使用

- 可重用切入点表达式

  - 声明一个没有实现的返回值为void的空方法

  - 把方法标注@Pointcut注解

    ```java
    @Pointcut("execution(public int com.spring.impl.Calculator.add(int,int))")
    public void empty(){}
    
    //在其他的通知方法的注释里就可以通过引用方法名来引用这个抽取出来的切面表达式
    @AfterReturning(value="empty()",return="result")
    public static void logReturn(JoinPoint joinPoint,Object result){
    System.out.println("计算结果是"+result);
    }
    ```

### 通知方法的执行顺序

    - 正常执行:@before->@After->@AfterReturning    
    - 异常执行:@before->@After->@AfterThrowing

### 在通知方法运行时拿到目标方法的具体信息

- **获取目标方法中的相关信息**
  - 在通知方法的参数列表上添加参数:       ==JoinPoint joinPoint==     该参数封装了目标方法的详细信息

```java
@Before(value ="execution(public int com.spring.impl.Calculator.*(int,int))")
public static void logStart(JoinPoint joinPoint){
    //获取目标方法运行时使用的参数
    Object[] args = joinPoint.getArgs();
    //获取到方法的签名
    Signature signature =  joinPoint.getSignature();
    String methodName = signature.getName();
    System.out.println("["+methodName+"]方法开始执行,用的参数列表"+Arrays.asList(args)+"...");
}
```

| 方法名                    | 功能                                                         |
| :------------------------ | :----------------------------------------------------------- |
| Signature getSignature(); | 获取封装了署名信息的对象,在该对象中可以获取到目标方法名,所属类的Class等信息 |
| Object[] getArgs();       | 获取传入目标方法的参数对象                                   |
| Object getTarget();       | 获取被代理的对象                                             |
| Object getThis();         | 获取代理对象                                                 |

- 获取目标方法的返回值或者异常信息
  - 先在参数列表中任意添加一个参数(Object res或者Exception e)
  - 再在注释中加一个属性:result="参数名"或Throwing = "参数名"(告诉Spring用哪个参数来接收异常或是返回值)

```java
@AfterReturning(value = "execution(public int com.spring.impl.Calculator.*(int,int))",returning = "result",Throwing="exc")
public static void logReturn(JoinPoint joinPoint,Object result,Exception exc){
    //获取到方法的签名
    Signature signature =  joinPoint.getSignature();
    String methodName = signature.getName();
    System.out.println("["+methodName+"]方法执行成功,计算结果是"+result);
}
```

### @Around 环绕通知,功能最强大!

他的强大在于他有一个参数:==ProceedingJoinPoint pjp==

==ProceedingJoinPoint实现了JoinPoint接口==

ProceedingJoinPoint有一个方法为==proceed(args)==,该方法就等于==method.invoke(obj,args).==

他的返回值就是被代理对象的方法的返回值,需要被返回出去

```java
@Around("execution(* com.spring.impl.Calculator.*(..))")
public Object myAround(ProceedingJoinPoint pjp){
    Object[] args = pjp.getArgs();//获取调用方法的参数
    String methodName = pjp.getSignature().getName();//获取调用方法的名字
    Object proceed =null;
    try {
        //就是利用反射调用目标方法即可,就是method.invoke(obj,args)
        System.out.println("[环绕前置通知]...方法["+methodName+"]开始");//此处等于@Before
        proceed = pjp.proceed(args);
        System.out.println("[环绕返回通知]...方法["+methodName+"]返回..返回值为"+proceed);//此处等于@AfterReturning
    } catch (Throwable throwable) {
        System.out.println("[环绕异常通知]...方法["+methodName+"]异常结束,异常信息为"+throwable.getCause());//此处等于@AfterThrowing
        throw new RuntimeException(throwable);
    }finally {
        System.out.println("[环绕结束通知]...方法["+methodName+"]正常结束");//此处等于@After
    }
    //反射调用后返回值一定要返回出去
    return proceed;
}
```

环绕通知是优先于普通通知执行的:

执行顺序:==环绕 前置->普通前置->目标方法执行-> 环绕返回/异常->环绕后置->普通后置->普通返回/异常==

### 多切面情况下的运行顺序

- 默认情况下各个切面的执行顺序是按照各个切面类的类名的首字母排序来排序的    

- 可以在切面类前加@Order(num)注释来指定各个切面的顺序(数值越小,优先级越高)    

- 各个切面先进入的就后出去(若顺序在前则前置通知会先行,但是后置通知其他的则是后行的)    

- 环绕通知若是添加在那个切面,在该切面出去之前,该切面的环绕通知定会先与该切面普通的通知(环绕只影响当前切面)

### 基于XML使用AOP

```
<aop:config>
    <!--指定切面类-->
    <aop:aspect ref="切面类id" order="切面优先级">
        <!--抽取切入点表达式-->
        <aop:pointcut id="表达式id" expression="切入点表达式"/>
        <!--指定方法是何种通知-->
        <aop:after-returning method="通知方法名" pointcut="切入点表达式" returning="参数名"></aop:after-returning>
        <aop:around method="通知方法名" pointcut-ref="表达式id"></aop:around>
    </aop:aspect>
</aop:config>
```

