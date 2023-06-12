# spring业务组件

## spring业务

### 幂等性

业务重现。

并发环境下，多次请求皆执行第④步

![mideng](C:\Users\it\Pictures\mideng.png)

解决方案

```sql
-- 幂等表
利用数据库唯一索引做防重处理,当第一次插入是没有问题的,第二次在进行插入会因为唯一索引报错。从而达到拦截的目的。



-- token令牌
如何防止重复提交, 为每次请求生成请求唯一键,服务端对每个唯一键进行生命周期管控。规定时间内只允许一次请求,非第一次请求都属于重复提交。但是前后端改造大,后端要给出单独生成token令牌接口,前端要在每次调用时候先获取token令牌。



-- 基于方法粒度指定唯一键【Tomato】
基于Web方法,从参数中寻找可以作为唯一键,进行控制。改造难度低,仅需要服务端改造,前端无感知。具体参考tomato仓库
```

## spring组件

### AOP

aop价值。使用aop实现图中《通过复制,粘帖的部分》

![aop](C:\Users\it\Pictures\aop.png)

例如，使用aop实现日志记录

自定义注解实现接收参数

```java
import java.lang.annotation.*;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface CustomLog {
    String value();
    String description();
}
```

日志类编辑业务

```java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    @Around("@annotation(com.example.annotation.CustomLog)")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        // 获取目标方法的注解参数
        CustomLog customLog = getCustomLogAnnotation(joinPoint);

        String value = customLog.value();
        String description = customLog.description();

        // 执行业务代码
        System.out.println("Custom Log - Value: " + value);
        System.out.println("Custom Log - Description: " + description);

        // 继续执行目标方法
        Object result = joinPoint.proceed();

        return result;
    }

    private CustomLog getCustomLogAnnotation(ProceedingJoinPoint joinPoint) {
        // 获取目标方法上的@CustomLog注解
        CustomLog customLog = joinPoint.getTarget()
                .getClass()
                .getMethod(joinPoint.getSignature().getName(), joinPoint.getArgs().getClass())
                .getAnnotation(CustomLog.class);

        return customLog;
    }
}
```

使用案例

```java
@Service
public class MyService {

    @CustomLog(value = "Custom Value", description = "Custom Description")
    public void myMethod() {
        // Method logic
    }

}
```

### IOC

