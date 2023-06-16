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

### 循环依赖

单例循环依赖

业务重现

```sql
两个对象的创建，互相依赖与对方
```

spring解决方案

```java
/** 一级缓存：用于存放完全初始化好的 bean **/
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);

/** 二级缓存：存放原始的 bean 对象（尚未填充属性），用于解决循环依赖 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** 三级级缓存：存放 bean 工厂对象，用于解决循环依赖 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);

/**
bean 的获取过程：先从一级获取，失败再从二级、三级里面获取

创建中状态：是指对象已经 new 出来了但是所有的属性均为 null 等待被 init
*/
```

业务流程

```sql
-- A 创建过程中需要 B，于是 A 将自己放到三级缓里面 ，去实例化 B

-- B 实例化的时候发现需要 A，于是 B 先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了！
	-- 然后把三级缓存里面的这个 A 放到二级缓存里面，并删除三级缓存里面的 A
	-- B 顺利初始化完毕，将自己放到一级缓存里面（此时B里面的A依然是创建中状态）

-- 然后回来接着创建 A，此时 B 已经创建结束，直接从一级缓存里面拿到 B ，然后完成创建，并将自己放到一级缓存里面

-- 如此一来便解决了循环依赖的问题
```

### 高并发

解决方案

![system-design](C:\Users\it\Pictures\system-design.png)

```sql
-- 系统拆分
将一个系统拆分为多个子系统，用 RPC 来搞。然后每个系统连一个数据库，这样本来就一个库，现在多个数据库，不也可以扛高并发么。
	-- 熔断
	-- 降级
	
-- 缓存
你可以考虑考虑你的项目里，那些承载主要请求的读场景，怎么用缓存来抗高并发。

-- MQ
你得考虑考虑你的项目里，那些承载复杂写业务逻辑的场景里，如何用 MQ 来异步写，提升并发性。大量的写请求灌入 MQ 里，后边系统消费后慢慢写，控制在 MySQL 承载范围之内。

-- 分库分表
一个数据库拆分为多个库，多个库来扛更高的并发；然后将一个表拆分为多个表，每个表的数据量保持少一点，提高 SQL 跑的性能。

-- 读写分离
大部分时候数据库可能也是读多写少，没必要所有请求都集中在一个库上吧，可以搞个主从架构，主库写入，从库读取，搞一个读写分离。读流量太多的时候，还可以加更多的从库。

-- ElasticSearch
ES 是分布式的，可以随便扩容，分布式天然就可以支撑高并发，因为动不动就可以扩容加机器来扛更高的并发。那么一些比较简单的查询、统计类的操作，可以考虑用 ES 来承载，还有一些全文搜索类的操作，也可以考虑用 ES 来承载。
```

#### 流量控制





## spring组件

### AOP

aop价值。

使用aop实现图中《通过复制,粘帖的部分》

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

ioc理解

```sql
-- IoC（Inversion of Control）是一种设计原则，也是Spring框架的核心概念之一。

-- 它是一种反转控制的思想，即将对象的创建、依赖关系的管理和对象的生命周期交给容器来管理，而不是由开发者手动管理。

-- IoC容器负责实例化对象、注入依赖关系以及销毁对象，开发者只需要声明需要的对象和它们的依赖关系，容器会负责将其组装并提供给其他组件使用。
```

声明方式

```sql
-- @Component注解
@Component是Spring的通用组件注解，它可以用于标识任何类型的组件，包括服务组件。

-- @Service注解
@Service是@Component注解的特殊形式，用于标识服务组件。通常用于表示业务逻辑层的组件，方便在代码中做更明确的区分。

-- @Repository注解
@Repository是@Component注解的特殊形式，用于标识数据访问组件。通常用于表示数据访问层的组件，如DAO（数据访问对象）或Repository。

-- @Controller注解
@Controller是@Component注解的特殊形式，用于标识控制器组件。通常用于表示控制器层的组件，处理用户请求和返回视图。

-- 其他衍生注解来声明服务组件
@SpringBootApplicaton、@Configuration等，它们具有特定的语义和功能，并适用于特定的场景。

-- 注意
建议使用更具体的注解（如@Service、@Repository、@Controller）来标识服务组件，以提高代码的可读性和可维护性。
```

### Interceptor

Interceptor理解

```sql
-- Interceptor（拦截器）是一种常见的设计模式，在软件开发中用于拦截并处理请求、调用或操作。
```

例如 spring自定义拦截请求

自定义拦截器

```java
package com.yami.shop.common.i18n;


import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.i18n.LocaleChangeInterceptor;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @author ellen
 */
@Component("localeChangeInterceptorDemo")
@Slf4j
public class YamiLocaleChangeInterceptorDemo implements HandlerInterceptor {
    @Autowired
    LocaleChangeInterceptor localeChangeInterceptor;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)  {
        try {
            //多语言兼容
            localeChangeInterceptor.preHandle(request,response,handler);
        } catch (ServletException e) {
            e.printStackTrace();
        }
        System.out.println("拦截器执行逻辑处");
        return true;
    }
}
```

注册到spring请求过程中

```java
package com.yami.shop.common.config;

import com.yami.shop.common.i18n.YamiLocaleChangeInterceptorDemo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;


/**
 * @author ellen
 */
//@Configuration
public class WebMvcConfigDemo implements WebMvcConfigurer{

    @Autowired
    private YamiLocaleChangeInterceptorDemo yamiLocaleChangeInterceptorDemo;


    //此为测试方法，不发到生产
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(yamiLocaleChangeInterceptorDemo)
                .addPathPatterns("/**")
                .order(1)
                .excludePathPatterns("/open/**");
    }
}

```

### filter