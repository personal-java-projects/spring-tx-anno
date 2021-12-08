## 编程式事务控制相关对象

#### 1、`PlatformTransactionManager`

`PlatformTransactionManager `接口是 spring 的事务管理器，它里面提供了我们常用的操作事务的方法。

![image-20211208104542205](image-20211208104542205.png)

注意： 

`PlatformTransactionManager` 是接口类型，不同的 `Dao` 层技术则有不同的实现类，例如：`Dao` 层技术是`jdbc  `或 `mybatis` 时：`org.springframework.jdbc.datasource.DataSourceTransactionManager`  `Dao` 层技术是hibernate时：`org.springframework.orm.hibernate5.HibernateTransactionManager`

#### 2、`TransactionDefinition`

`TransactionDefinition` 是事务的定义信息对象，里面有如下方法：

![image-20211208104803011](image-20211208104803011.png)

1. 事务隔离级别

   设置隔离级别，可以解决事务并发产生的问题，如脏读、不可重复读和虚读。

   * `ISOLATION_DEFAULT`
   * `ISOLATION_READ_UNCOMMITTED`
   * `ISOLATION_READ_COMMITTED`
   * `ISOLATION_REPEATABLE_READ`
   * `ISOLATION_SERIALIZABLE`

2. 事务传播行为

   * `REQUIRED`：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。一般的选择（默认值）
   * `SUPPORTS`：支持当前事务，如果当前没有事务，就以非事务方式执行（没有事务）
   * `MANDATORY`：使用当前的事务，如果当前没有事务，就抛出异常
   * `REQUERS_NEW`：新建事务，如果当前在事务中，把当前事务挂起。
   * `NOT_SUPPORTED`：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
   * `NEVER`：以非事务方式运行，如果当前存在事务，抛出异常
   * `NESTED`：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行 REQUIRED 类似的操作
   * 超时时间：默认值是-1，没有超时限制。如果有，以秒为单位进行设置
   * 是否只读：建议查询时设置为只读

#### 3、`TransactionStatus`

`TransactionStatus` 接口提供的是事务具体的运行状态，方法介绍如下。

![image-20211208105301610](image-20211208105301610.png)

#### 4、知识要点

编程式事务控制三大对象

* `PlatformTransactionManager`
* `TransactionDefinition`
* `TransactionStatus`

## 基于 XML 的声明式事务控制

#### 1、什么是声明式事务控制

Spring 的声明式事务顾名思义就是采用声明的方式来处理事务。这里所说的声明，就是指在配置文件中声明 ，用在 Spring 配置文件中声明式的处理事务来代替代码式的处理事务。

**声明式事务处理的作用**

* 事务管理不侵入开发的组件。具体来说，业务逻辑对象就不会意识到正在事务管理之中，事实上也应该如 此，因为事务管理是属于系统层面的服务，而不是业务逻辑的一部分，如果想要改变事务管理策划的话， 也只需要在定义文件中重新配置即可
* 在不需要事务管理的时候，只要在设定文件上修改一下，即可移去事务管理服务，无需改变代码重新编译 ，这样维护起来极其方便

注意：Spring 声明式事务控制底层就是`AOP`。

#### 2、声明式事务控制的实现

声明式事务控制明确事项：

* 谁是切点？
* 谁是通知？
* 配置切面？

① 引入`tx`命名空间

![image-20211208105915923](image-20211208105915923.png)

```xml-dtd
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd
    http://www.springframework.org/schema/aop
    http://www.springframework.org/schema/aop/spring-aop.xsd
    http://www.springframework.org/schema/tx 
    http://www.springframework.org/schema/tx/spring-tx.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">
```

② 配置事务增强

```xml
<!--平台事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"></property>
</bean>

<!--事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

③ 配置事务 `AOP` 织入

```xml
<!--事务的aop增强-->
<aop:config>
    <aop:pointcut id="myPointcut" expression="execution(* com.example.service.impl.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="myPointcut"></aop:advisor>
</aop:config>
```

④ 测试事务控制转账业务代码

```java
@Override
public void transfer(String outMan, String inMan, double money) {
    accountDao.out(outMan,money);
    int i = 1/0;
    accountDao.in(inMan,money);
}
```

#### 3、切点方法的事务参数的配置

```xml
<!--事务增强配置-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
    <tx:method name="*"/>
    </tx:attributes>
</tx:advice>
```

其中， 代表切点方法的事务参数的配置，例如：

```xml
<tx:method name="transfer" isolation="REPEATABLE_READ" propagation="REQUIRED" timeout="-1" 
read-only="false"/>
```

* `name`：切点方法名称
* `isolation`:事务的隔离级别
* `propogation`：事务的传播行为
* `timeout`：超时时间
* `read-only`：是否只读

#### 4、知识要点

声明式事务控制的配置要点

* 平台事务管理器配置
* 事务通知的配置
* 事务`aop`织入的配置

## 基于注解的声明式事务控制

#### 1、使用注解配置声明式事务控制

1. 编写 `AccoutDao`

   ```java
   @Repository("accountDao")
   public class AccountDaoImpl implements AccountDao {
       @Autowired
       private JdbcTemplate jdbcTemplate;
       
       public void out(String outMan, double money) {
           jdbcTemplate.update("update account set money=money-? where 
           name=?",money,outMan);
       }
                           
       public void in(String inMan, double money) {
           jdbcTemplate.update("update account set money=money+? where 
           name=?",money,inMan);
       }
   }
   ```

2. 编写 `AccoutService`

   ```java
   @Service("accountService")
   @Transactional
   public class AccountServiceImpl implements AccountService {
       @Autowired
       private AccountDao accountDao;
       
       @Transactional(isolation = Isolation.READ_COMMITTED,propagation = 
       Propagation.REQUIRED)
       public void transfer(String outMan, String inMan, double money) {
           accountDao.out(outMan,money);
           int i = 1/0;
           accountDao.in(inMan,money);
       }
   }
   ```

3. 编写 `applicationContext.xml` 配置文件

   ```xml
   <!—之前省略datsSource、jdbcTemplate、平台事务管理器的配置-->
   <!--组件扫描-->
   <context:component-scan base-package="com.example"/>
   <!--事务的注解驱动-->
   <tx:annotation-driven/>
   ```

#### 2、注解配置声明式事务控制解析

① 使用 `@Transactional` 在需要进行事务控制的类或是方法上修饰，注解可用的属性同 `xml` 配置方式，例如隔离 级别、传播行为等。

 ② 注解使用在类上，那么该类下的所有方法都使用同一套注解参数配置。 

③ 使用在方法上，不同的方法可以采用不同的事务参数配置。 

④ `xml`配置文件中要开启事务的注解驱动

#### 3、知识要点

注解声明式事务控制的配置要点

* 平台事务管理器配置（xml方式）
* 事务通知的配置（@Transactional注解配置）
* 事务注解驱动的配置 `<tx:annotation-driven/>`