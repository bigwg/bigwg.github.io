---
layout: post
title: 基于Spring环境单元测试最佳实践
subtitle: 关于业务开发如何做好单元测试的探索与实践
categories: Spring
tags: [Spring,Unit Test]
---

## 一、背景

随着团队对单元测试建设的重视与推广，越来越多的项目开始按照规范进行了单元测试的升级改造，但是随着单元测试的增加，各种场景下的单元测试的问题也在不断的暴露出来，这就**导致了在代码开发中投入了大量精力的我们还得在单元测试上被迫分摊大量精力，使我们研发同学在业务代码开发和单元测试上疲于奔命，极大的降低了研发效率。**那究竟是什么问题导致我们研发同学如此的疲于奔命呢？经过我们对问题进行归纳总结并分析之后我们找到了问题的根源，那就是 Spring 框架！

大家都清楚，如果我们只是启动一个简单的 Java 单测代码的话，稳定性与运行效率是非常高的，但是由于目前大部分业务工程都是基于 Spring 框架开发的，这就导致了我们写单测代码时很难脱离 Spring 环境进行，而在进行单测时通常的方式是启动整个 Spring 上下文，这其实导致了一个严重的问题，**那就是我们基于 Spring 的单元测试严重违背了单元测试的环境无关、测试单元最小化原则**。那么我们如何做到真正的环境无关、测试单元最小化呢？

下面我就结合自身以及团队经验来分析一下我们面临的问题以及相应的解决方案，让我们来看看在 Spring 环境下单元测试的最佳实践是怎么样的吧！

## 二、问题和方案

### 2.1 当前面临的问题

上面我们说到在 Spring 环境下我们的单元测试违背了环境无关、测试单元最小化原则，下面我们就来看一看违背原则的一些表象。

* 我只想跑一个方法的单元测试，理论上应该非常快就结束了（秒级），但是为什么我现在运行一个简单的单测需要30多秒？
* 我的单测在 DEV 环境是好使的，但是为什么我把环境切换到 TEST 环境就报错？
* 在办公网络下运行单测是好好的，但是回到家后在非办公网络情况下运行单测却怎么也不成功？
* 为什么我就跑一个简单的测试，不依赖任何外部环境的测试，怎么就启动报错呢？之前都是好的，怎么启动什么都报错？？
* 为什么我在本地跑的单元测试都是通过的，但是用集成工具或发版工具发布编译的时候单元测试结果却都是失败的？

### 2.2 当前问题的根本原因

上面是我和团队内的同学遇到的比较典型的单元测试问题，这些问题其实在非 Spring 环境下（一些不需要 Spring 这种复杂的工业级框架的简单工程或中间件）是比较少见的，**但是在 Spring 环境目前的用法中我们启动了整个 Spring 容器，这就导致我们在启动单测的时候 Spring 的 BeanFactory 中会载入很多的与单测无关的 Spring Bean 进来，同时使用 Spring Boot 的项目因为 Spring Boot 的 AutoConfig 机制更是加剧了这种情况。**

我们的应用会去连接注册中心、连接配置中心、连接 Redis、连接Mysql、连接MQ、连接定时任务中间件等等这一系列工程依赖的组件，由于我们以来的 Spring 环境需要将这些组件的相关类管理在 Spring 的上下文中，所以我们在启动单元测试的时候这一系列的中间件我们的应用都要进行连接，引入的中间件越多，在启动的过程中连接的组件就会越多。我们都知道在工程实践中，我们引入的功能、中间件越多，那我们的稳定性将越差，这一点在单元测试中也是同样的。

<div align=center>
<img src="https://raw.githubusercontent.com/bigwg/bigwg.github.io/main/image/%E5%B7%A5%E7%A8%8B%E4%BE%9D%E8%B5%96.jpeg"/>
</div>

### 2.3 探寻解决方案

现在我们在知道了基于 Spring 环境单元测试低效、不稳定的根源是由于 Spring 环境对工程中所有的中间件都进行了自动管理导致的，那我们有没有什么办法解决这些问题呢？答案是肯定的，**我们只让 Spring 加载必要的部分（即要测试的部分），然后 Mock 依赖的部分，其余所有与本单元测试不相关的部分都不进行加载，这样我们就得到了一个符合环境无关的、测试单元最小化的单元测试，同时它肯定也会表现的稳定与高效。**下面我会介绍具体的使用方式，让我们拭目以待吧！

<div align=center>
<img src="https://raw.githubusercontent.com/bigwg/bigwg.github.io/main/image/%E5%B7%A5%E7%A8%8B%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB%E8%BD%AC%E5%8F%98.png"/>
</div>

## 三、收益

那么，我们解决了上面的问题后可以得到什么样的收益呢？

（1）收益最大的就是我们可以提升研发效率，比如我们在编写代码和单测的时候我们进行的测试通常都是一个一个的进行，**如果我们每次进行单个方法的测试都从30秒缩短到3秒**，那么我们的研发效率会有极大的提升。

（2）当我们彻底解决了单元测试的环境无关后，我们就可以从跟各种环境斗智斗勇的关系中脱离出来，**无论我们的单测运行在什么环境下都是成功的，失败的原因只可能是代码逻辑故障**，同时对加入团队的新同学也更加友好（新同学在不清楚各种基础组件的情况下环境问题会非常要命）。

（3）当我们的单元测试符合测试单元最小化的原则的时候，**我们的单元测试会规避很多未知因素的干扰，一个测试单元有问题一定是这个单元相关的依赖或者自身的问题**，再也不用愁眉苦脸的看着红色报错来猜测是不是哪个根本不需要的模块加载报错导致的单测启动失败了。

如上，我们可以发现解决了这些问题后对我们的研发效率、代码纠错等方面的帮助与收益还是非常明显的，无论是从短期还是长期收益来看这些问题都是非常值得去解决的。

## 四、最佳实践

### 4.1 如何最小化启动单元测试？

在第二节我们遇到的问题其实都可以归结为没有最小化启动单元测试，也就是说我们首先要解决的问题就是最小化启动 Spring 容器。我们可以思考一下，我们目前的大多数工程其实都是通过 SQL 与 Mysql 等数据库进行通信的上层应用，由于我们编写了很多 SQL，而且这些 SQL 也是需要进行单元测试的，所以我们确定了我们的单测需要使用数据库（我们不会连接外部的数据库，而是在引入一个内嵌在应用工程中的数据库来替代，比如我们在单测中使用的 H2），但是其他的**大多数中间件我们其实都是简单的通过 API 接口的形式进行交互，这部分中间件在我们的单测阶段其实是完全没必要的对于这部分的测试是属于集成测试的范畴，所以我们在单测中完全可以 Mock 掉它们或者按需引入（按需引入的前提条件是不依赖任何环境即可启动，例如我们用内嵌数据库 H2 来代替 Mysql，如果我们想测试 Redis 我们也可以用类似的思路，引入一个内嵌的 Redis，但是绝不能引入任何的外部 Redis），最终我们确定了我们 Spring 容器启动应该包含的内容：只有数据库相关的部分！**下面我们看一看具体的代码。

首先，我们需要引入 spring-test 和 spring-boot-test 这两个依赖包

```
<!-- spring 环境的单元测试的核心包 -->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-test</artifactId>
  <version>${当前工程 spring 的版本}</version>
  <scope>test</scope>
</dependency>

<!-- 下文我们用到的 @Import 功能和 @MockBean、@SpyBean 功能在该包里引入，虽然这是个 spring boot 的包，但是普通的 spring 项目也是可以使用的，没有任何问题 -->
<!-- 这里的版本我们需要注意下，如果我们用的是 Spring4.x 的版本，这里要引入 1.5.xx.RELEASE 的版本，推荐使用 1.5.22.RELEASE -->
<!-- 如果我们用的是 Spring5.x 的版本，这里要引入 2.3.xx.RELEASE 的版本，推荐使用 2.3.8.RELEASE -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-test</artifactId>
  <version>2.3.8.RELEASE</version>
  <scope>test</scope>
</dependency>
```

然后就是创建单元测试基类，基于 Testng、Junit4 和 Junit5 的实现方案有些许差别，代码如下：

```
/**
 * 单元测试基类，基于Testng
 */
// Spring容器配置文件，我们是最小化启动，所以只加载DB的配置，用H2替换mysql
@ContextConfiguration("classpath:dataSource.xml")
// 使 @MockBean 和 @SpyBean 注解生效的配置，具体使用方式我们在4.4小节讲
@TestExecutionListeners(MockitoTestExecutionListener.class)
// Spring上下文清理配置，表示每个测试都重新清理一下Spring上下文，这样能够保证每个单测都是隔离的环境
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_CLASS)
public abstract class BaseTest extends AbstractTestNGSpringContextTests {
    
}

/**
 * 单元测试基类，基于Junit4
 */
// Junit4 启动器
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "classpath:dataSource.xml")
// 基于 Junit4 需要添加 DirtiesContextBeforeModesTestExecutionListener.class,DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class
@TestExecutionListeners({MockitoTestExecutionListener.class, DirtiesContextBeforeModesTestExecutionListener.class,
        DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class})
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public abstract class BaseTest {

}

/**
 * 单元测试基类，基于Junit5
 */
// Junit5 启动器
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations = "classpath:dataSource.xml")
// 基于 Junit5 需要添加 DirtiesContextBeforeModesTestExecutionListener.class,DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class
@TestExecutionListeners({MockitoTestExecutionListener.class, DirtiesContextBeforeModesTestExecutionListener.class,
        DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class})
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public abstract class BaseTest {

}
```

上面 @ContextConfiguration 注解引入的 dataSource.xml 配置文件如下（**如果大家的项目是多数据源的话只需要把所有的表都放在这一个数据源下即可，分库分表亦是如此**）：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns="http://www.springframework.org/schema/beans" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx.xsd
       http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc.xsd">


    <jdbc:embedded-database id="h2TestDataSource" type="H2" database-name="xxxDataSource;DATABASE_TO_UPPER=false;MODE=MYSQL;">
        <!-- 这里的 h2/init.sql 是我们要初始化表结构的文件 -->
        <jdbc:script location="classpath:h2/init.sql"/>
    </jdbc:embedded-database>

    <!--  sqlSessionFactory for mybatis -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="h2TestDataSource"/>
        <!-- 配置mybatis配置文件的位置 -->
        <property name="configLocation" value="classpath:mybatis-config.xml"/>
        <property name="mapperLocations">
            <list>
              <!-- 这里的 value 要替换成真实的 mybatis 的 mapper 文件地址 -->
                <value>classpath:mapper/**/*DAO.xml</value>
            </list>
        </property>
    </bean>

    <bean id="sqlSessionTemplate" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg ref="sqlSessionFactory"/>
    </bean>

    <!-- 配置扫描Mapper接口的包路径 -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.meituan.xxx.dao"/>
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    </bean>

    <bean id="mybatisTransactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager"
          p:dataSource-ref="h2TestDataSource"/>

    <tx:annotation-driven transaction-manager="mybatisTransactionManager"/>
</beans>
```

dataSource.xml 配置文件需要放在 test/resources 目录下，我们在写单测的时候只需要继承上面经过改造后的 BaseTest 类即可。

最后我们看一下配置好的 test 目录内容如下，其中 **db.datasets、h2/init.sql、dbunit.yml** 是 Database-rider 组件使用的配置文件，在下面会介绍。

<div align=center>
<img src="https://raw.githubusercontent.com/bigwg/bigwg.github.io/main/image/20220331180209.png"/>
</div>

### 4.2 如何使用 H2 和 Database-rider 进行 SQL 测试？

上文我们已经提到最小化的单元测试是需要包含数据库的，但是如果我们引入一个外部的数据库的话，那我们就违背了单元测试环境无关的原则。所以，我们要用一个内嵌的数据库来替代外部的数据库，而 H2 就是我们需要的那个嵌入式数据库。Database-rider 又是什么呢？它其实是一个方便我们进行数据库测试的工具，它提供了数据初始化、数据清理、结果对比等功能。所以 H2 和 Database-rider 的搭配使用，能够使我们对 SQL 更加高效便捷的进行测试。

首先，我们引入 H2 和 Database-rider 的依赖

```
<!-- H2 数据库依赖 -->
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <version>2.1.210</version>
  <scope>test</scope>
</dependency>
<!-- Database-rider 依赖 -->
<dependency>
  <groupId>com.github.database-rider</groupId>
  <artifactId>rider-core</artifactId>
  <version>1.32.3</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>com.github.database-rider</groupId>
  <artifactId>rider-spring</artifactId>
  <version>1.32.3</version>
  <scope>test</scope>
</dependency>
```

然后我们引入配置

```
<!-- 这里是 H2 数据库的配置，我们可以将其当做一个数据源，配置在 dataSource.xml 文件中 -->
<jdbc:embedded-database id="h2TestDataSource" type="H2" database-name="xxxDataSource;DATABASE_TO_UPPER=false;MODE=MYSQL;">
  <!-- 这里的 h2/init.sql 是我们要初始化表结构的文件，在单元测试启动时会自动帮我们将 init.sql 中的 sql 执行来达到创建表的目的 -->
  <jdbc:script location="classpath:h2/init.sql"/>
</jdbc:embedded-database>
```

创建 dbunit.yml 文件放在 test/resources 目录下，该文件是 Database-rider 配置文件

```
cacheConnection: true           # 缓存数据库连接
cacheTableNames: true           # 缓存数据库表名
leakHunter: true                # 数据库连接泄露检测
expectedDbType: H2              # 指定数据库类型
properties:                     # 其他属性
  caseSensitiveTableNames: true # 数据库表名大小写敏感
  allowEmptyFields: true # 数据集允许字段值为空
```

下面我们举一个简单的例子来看一下如何使用

```
// 这个是注解是 Database-rider 的核心，放在测试类上表示本测试类需要使用 Database-rider 的功能，如果所有单测都需要使用，可以将该注解添加到 BaseTest 上
@DBRider(dataSourceBeanName = "h2TestDataSource")
// 引入依赖到 Spring 上下文中的核心注解，使用该注解把所有需要加入到 Spring 上下文的 Bean 引入，下文会详细介绍
@Import({ServiceB.class, ServiceA.class})  
public class ServiceATest extends BaseTest {
  
    @Autowired
    private ServiceA serviceA;

    @Test
  	// 这个是注解是用于在单测执行前进行数据初始化的，会根据 xxx.json 中的数据进行初始化，cleanBefore = true 意思是在单测启动前会将 H2 中的所有数据清空，创造一个干净的数据环境
  	@DataSet(value = "/db/datasets/xxx/xxx.json", cleanBefore = true)
  	// 这个是注解是用于在单测执行后对数据库中的数据做比对的，会将 H2 中的数据与 xxx.json 中的数据进行对比，如果不一样则单测执行失败，ignoreCols 表示忽略对比的字段，比如一些随机值
  	@ExpectedDataSet(value = "db/datasets/xxx/xxx.json", ignoreCols = {"id"})
    public void testServiceA() {
    }
}
```

> **由于篇幅有限，本文只介绍简单的使用，关于 H2 和 Database-rider 的更多功能大家可以自行探索。**

### 4.3 如何注入被依赖的 Spring Bean？

由于我们使用最小化的 Spring 环境来启动我们的单元测试，所以我们的 Spring 容器中只有 DAO 相关的 Bean，而其他的所有 Bean 都需要我们手动引入，这样我们就得到了一个纯净的、最小化的、按需引入的 Spring 上下文。

**我们先来看一种简单的情况，如果我们要测试的服务是 ServiceA，该服务依赖关系如下：**
ServiceA 依赖 ServiceB 和 ADao
ServiceB 依赖 BDao
代码片段如下：

```
// ServiceA 代码片段
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
    @Autowired
    private ADao aDao;
}

// ServiceB 代码片段
@Service
public class ServiceB {
    @Autowired
    private BDao bDao;
}
```

我们要对 ServiceA 进行测试，由于 ServiceA 的所有依赖和间接依赖都是本地依赖，没有任何外部依赖，所以我们可以将其依赖的所有服务都加载到 Spring 上下文中，测试类代码如下：

```
@Import({ServiceB.class, ServiceA.class})  // 引入依赖到 Spring 上下文中的核心注解，使用该注解把所有需要加入到 Spring 上下文的 Bean 引入
public class ServiceATest extends BaseTest {
    @Autowired
    private ServiceA serviceA;

    @Test
    public void testServiceA() {
    }
}
```

**我们再来看一个稍微复杂一点的情况，如果我们要测试的服务还是 ServiceA，该服务依赖关系如下：**
ServiceA 依赖 ServiceB 和 ADao
ServiceB 依赖 ServiceC 和 BDao
ServiceC 依赖 ServiceD
ServiceD 没有依赖

我们可以看到这个例子要比上面的例子的依赖层级要深一些，这里是4层依赖关系（从 ServiceA 为第一层算起），这里虽然比上面的例子要复杂一些，但是解决方案还是一样的，我们只需要使用 @Import 注解将所有本地依赖全部引入到当前的 Spring 上下文中即可。代码如下：

```
@Import({ServiceA.class, ServiceB.class, ServiceC.class, ServiceD.class})  // 引入依赖到 Spring 上下文中的核心注解，使用该注解把所有需要加入到 Spring 上下文的 Bean 引入
public class ServiceATest extends BaseTest {
    @Autowired
    private ServiceA serviceA;

    @Test
    public void testServiceA() {
    }
}
```

这里大家可能会担心如果我们的依赖层级特别深，依赖的 Bean 特别多，那岂不是 @Import 的时候会特别麻烦？其实不然，通常我们的依赖层级都是在3到4级左右，依赖的 Bean 也是在10个以内，如果依赖层级、依赖 Bean 太多的话那就要思考一下服务拆分的是否合理了，是否满足单一职责原则了，所以这也算是提供了一个服务拆分是否合理的自我检查方法。

> **由于 @Import 注解需要我们自己梳理 Spring 依赖关系，导致效率比较低，虽然可以帮助我们检查依赖树是否层级太深，但是违背了提效的初衷，所以我写了一个 @AutoImport 注解来简化流程，具体使用细节看 补充6.1**

### 4.4 如何 Mock 外部依赖并改变其行为？

首先，我们需要引入一个 Spring 官方的 Mock 功能的依赖和 Mockito 的依赖：

```
<!-- 上文已经提到，这个包提供了 @MockBean和@SpyBean 的功能 -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-test</artifactId>
  <version>2.3.8.RELEASE</version>
  <scope>test</scope>
</dependency>

<!-- 这里推荐使用的版本号是 3.8.0，如果出现不兼容的情况可以适当降低版本 -->
<!-- 这里注意下我们可以选择引入 mockito-core 还是 mockito-inline，这是因为 mockito-core 默认是不支持 mock final类/方法和static方法的 -->
<!-- 所以如果我们想要 mock final类/方法和static方法需要引入 mockito-inline，大家可以按需引入 -->
<dependency>
  <groupId>org.mockito</groupId>
  <artifactId>mockito-core</artifactId>
  <!-- <artifactId>mockito-inline</artifactId> -->
  <version>3.8.0</version>
  <scope>test</scope>
</dependency>
```

下面就是介绍单元测试的重中之重，堪称单元测试的灵魂之中的灵魂的 Mock 了，Mock 主要分为两类，我会通过几个例子和大家介绍一下。

首先，如我们想要使 Mock 功能生效的话需要在 BaseTest 上加一个注解 **@TestExecutionListeners(MockitoTestExecutionListener.class)**，在上文中我们也提到过

```
/**
 * 单元测试基类，基于Testng
 */
@ContextConfiguration("classpath:dataSource.xml")
// 使 @MockBean 和 @SpyBean 注解生效的配置
@TestExecutionListeners(MockitoTestExecutionListener.class)
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_CLASS)
public abstract class BaseTest extends AbstractTestNGSpringContextTests {
    
}

/**
 * 单元测试基类，基于Junit4
 */
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "classpath:dataSource.xml")
@TestExecutionListeners({MockitoTestExecutionListener.class, DirtiesContextBeforeModesTestExecutionListener.class,
        DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class})
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public abstract class BaseTest {

}

/**
 * 单元测试基类，基于Junit5
 */
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations = "classpath:dataSource.xml")
@TestExecutionListeners({MockitoTestExecutionListener.class, DirtiesContextBeforeModesTestExecutionListener.class,
        DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class})
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public abstract class BaseTest {

}
```

**第一类 Mock 是使用 @MockBean 注解，该注解会创建一个所有方法都返回 NULL 的对象来 Mock 原来的对象，适合用来 Mock 外部依赖，然后配合方法打桩来改变原有方法的行为。**

举个例子，我们要测试的服务是 ServiceA 依赖了 ThriftServiceB 的 methodB 方法，ThriftServiceB 是一个外部的 RPC 接口，此时我们需要 Mock 这个接口来保证我们的单测是环境隔离的，代码如下

```
// ServiceA 代码片段
@Service
public class ServiceA {
    @Autowired
    private ThriftServiceB thriftServiceB;
  
  	public Integer methodA(){
      Param param = new Param();
      Result result = thriftServiceB.methodB(param);
      return result.getData();
    }
}

// 测试类代码
@Import({ServiceA.class})  // 引入依赖到 Spring 上下文中的核心注解，使用该注解把所有需要加入到 Spring 上下文的 Bean 引入
public class ServiceATest extends BaseTest {
    @Autowired
    private ServiceA serviceA;
    // 当我们加上这段代码之后，当前启动的 Spring 容器中就生成了一个 name = thriftServiceB 的 Mock Bean，
    // 然后该容器中所有的依赖注入 ThriftServiceB 的 Bean 都注入了我们指定的这个 Mock Bean，此时我们就可以
    // 使用 Mockito 的打桩来任意修改该 Mock Bean 的方法的行为了，没有修改的行为默认返回 null
    @MockBean(name = "thriftServiceB")
    private ThriftServiceB thriftServiceB; 
  
    @Test
    public void testServiceA() {
        Result result = new Result();
        result.setData(1);
        // 这就是我们上文所说的打桩（stub），这段代码的意思就是我们改变 thriftServiceB 这个对象的 methodB 方法的行为，变成我们想要
        // 的样子，这样在我们测试 serviceA.methodA() 时，内部调用 thriftServiceB.methodB(param) 的时候就会按照我们期望的样子进行返回
        // 这个打桩函数的内容有很多，基本大家想用的都可以实现，此处就不一一介绍了，大家感兴趣可以自己查看 API
        Mockito.when(thriftServiceB.methodB(Mockito.any())).thenReturn(result);
      	Integer resultData = serviceA.methodA();
      	Assert.assertEquals(resultData, 1);
    }
}
```

**第二类 Mock 是使用 @SpyBean 注解，该注解会创建一个所有方法都调用原方法的 Mock 对象，适合用来 Mock 内部依赖，然后配合方法打桩来改变原有方法的行为。**

举个例子，比如我们想要测试 ServiceA 的 methodA 的事务是否生效，我们就可以使用该注解来实现，因为我们想要某一个方法抛出异常，二其他的所有方法都保持原样正常运行，所以我们可以用 @SpyBean 注解来解决这个问题

被测试类 ServiceA 依赖 DaoA 与 DaoB，DaoA 的 insertA 方法与 DaoB 的 insertB 方法在 ServiceA 的 methodA 方法中，并在一个事务下，代码如下

```
// ServiceA 代码片段
@Service
public class ServiceA {
    @Autowired
    private DaoA daoA;
    @Autowired
    private DaoB daoB;
  
    @Transaction(rollbackFor = Exception.class)
  	public void methodA(){
      A a = new A();
      daoA.insertA(a);
      B b = new B();
      daoB.insertB(b);
    }
}

// 测试类代码
@Import({ServiceA.class})  // 引入依赖到 Spring 上下文中的核心注解，使用该注解把所有需要加入到 Spring 上下文的 Bean 引入
public class ServiceATest extends BaseTest {
    @Autowired
    private ServiceA serviceA;
    // 当我们加上这段代码之后，当前启动的 Spring 容器中就生成了一个 name = daoB 的 Spy Bean，
    // 然后该容器中所有的依赖注入 daoB 的 Bean 都注入了我们指定的这个 Spy Bean，此时我们就可以
    // 使用 Mockito 的打桩来任意修改该 Spy Bean 的方法的行为了，没有修改的行为默认调用原对象的方法
    @SpyBean(name = "daoB")
    private DaoB daoB; 
  
    @Test
    public void testServiceA() {
        Result result = new Result();
        result.setData(1);
        // 这里我们使用 do.when 的方式进行打桩，在上一个例子里用了另一种打桩方式，我们可以称其为 when.then 的打桩方式，
        // 两种方式的 API 基本一样，但是使用效果有两点区别，下面会详细介绍
        Mockito.doThrow(new RuntimeException("transaction rollback")).when(daoB).insert(Mockito.any());
      	try{
          	serviceA.methodA();
        } catch (RuntimeException e){
          	if (!e.getClass().equals(RuntimeException.class) || Objects.equals(e.getMessage(), "transaction rollback")){
                throw e;
            }
        }
      	A a = serviceA.queryAById(1);
      	Assert.assertNull(a);
    }
}
```

在上面的两个例子中我们使用了两种打桩方式，第一种我们称之为 **when.then** 的方式，第二种我们称之为 **do.when** 的方式，这两种方式的核心区别在于两点

①** do.when** 不是类型安全的（**不会进行编译期返回类型校验，比如使用 doReturn 的时候**），这可能带来意想不到的失败

② **when.then** 的方式打桩时会调用一次原方法（**在使用@SpyBean注解时会先执行一次并造成影响，不过此场景使用概率极低，所以一般不会遇到**），**do.when** 的方式打桩的话不会调用原方法

举个例子：上面我们使用 @SpyBean 来 mock 的 daoB，如果我们使用 **when.then **的方式打桩的话，在执行打桩代码时 daoB.insert() 方法还是会被真实的调用一次，所以可能会产生意想不到的错误，大家可以根据实际情况来选择 Mock 的方式。

> **由于篇幅有限，本文不再赘述 Mockito 更多的功能，大家可以参考 [Mockito官网](https://site.mockito.org/) 和 [SpringTest文档](https://docs.spring.io/spring-framework/docs/current/reference/html/testing.html#testing) 学习更多的 Mock 方法。**

### 4.5 如何迁移历史项目的单元测试？

如果我们是一个全新的项目的话那肯定不用多说了，我是强烈推荐这种方式进行单元测试的，相信我，你会爱上这种简单、便捷、不用与代码斗智斗勇的单元测试方式的！

如果是历史项目呢？那其实是有一定的改造成本的，因为我们需要为每个单元测试划清边界，要找到他的依赖树然后将本地依赖直接 @Import，外部依赖 @MockBean，所以我推荐大家慢慢的迭代迁移。比如我们做某个项目的时候要修改或者新增某个 Service 的单测，这时候我们就可以使用我们上文讲到的方案对该测试对象进行改造，我们可以增加一个单测基类，然后逐步改造，这样成本最低，做起来也更容易完成。测试基类名字我都为大家想好了就叫 SmartBaseTest（一个充满了机智的类名😄），代码和上面的 BaseTest 一样：

```
/**
 * 单元测试基类，基于Testng
 */
@ContextConfiguration("classpath:dataSource.xml")
@TestExecutionListeners(MockitoTestExecutionListener.class)
@DirtiesContext(classMode = DirtiesContext.ClassMode.AFTER_CLASS)
public abstract class SmartBaseTest extends AbstractTestNGSpringContextTests {
    
}

/**
 * 单元测试基类，基于Junit4
 */
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = "classpath:dataSource.xml")
@TestExecutionListeners({MockitoTestExecutionListener.class, DirtiesContextBeforeModesTestExecutionListener.class,
        DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class})
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public abstract class SmartBaseTest {

}

/**
 * 单元测试基类，基于Junit5
 */
@ExtendWith(SpringExtension.class)
@ContextConfiguration(locations = "classpath:dataSource.xml")
@TestExecutionListeners({MockitoTestExecutionListener.class, DirtiesContextBeforeModesTestExecutionListener.class,
        DependencyInjectionTestExecutionListener.class, DirtiesContextTestExecutionListener.class})
@DirtiesContext(classMode = DirtiesContext.ClassMode.BEFORE_EACH_TEST_METHOD)
public abstract class SmartBaseTest {

}
```

## 五、总结

单元测试是软件开发生命周期中非常重要的一环，如果能够写出高效且稳定的单元测试，那无疑是对我们软件质量保障的一大利器，但是在我们目前常见的业务开发依赖 Spring 这种工业级框架的体系下写好真正的“单元测试”是非常困难的。下面我们总结一下在 Spring 环境下写好单元测试的几点要素：

**（1）遵循单元测试环境无关、测试单元最小化原则**

**（2）使用 H2 和 Database-rider 替代数据库进行 SQL 测试**

**（3）对本单元测试无关的功能禁止加载到 Spring 上下文中**

**（4）对本单元测试依赖的外部组件进行 Mock**

以上内容是我们团队在基于 Spring 环境单元测试上的探索，希望能够在构建高效稳定的单元测试上为大家提供帮助。

## 六、补充

### 6.1 @Import 引入依赖太繁琐？快使用 @AutoImport 替代吧~

首先，引入依赖包

```
<dependency>
  <groupId>io.github.bigwg</groupId>
  <artifactId>easy-spring-test</artifactId>
  <version>1.0.1</version>
  <scope>test</scope>
</dependency>
```

然后使用 @AutoImport 注解替代 @Import 注解即可，举个例子

```
// 改造前
@Slf4j
// 待测试的 service 类是 XxxService 但是由于该类依赖或间接依赖了很多其他 service，所以需要手动梳理依赖的 Spring Bean 并引入上下文中，流程比较繁琐，导致易用性降低
@Import({XxxService.class, XxxAService.class, XxxBService.class, XxxCService.class, XxxDService.class, XxxEService.class})
public class XxxServiceTest extends BaseTest {
  	// 注入待测试的 service
    @Autowired
    private XxxService xxxService;
}

// 改造后
@Slf4j
// 使用 @AutoImport 替代后，仅需要 XxxService 即可，@AutoImport 注解会自动检索依赖树，然后将依赖的 Spring Bean 引入上下文中
@AutoImport({XxxService.class})
public class XxxServiceTest extends BaseTest {
    // 注入待测试的 service
    @Autowired
    private XxxService xxxService;
}
```

> **为了方便大家使用与理解，源码已经放在 [**Github**](https://github.com/bigwg/easy-spring-test) 上，有问题的话可以提 [Issue](https://github.com/bigwg/easy-spring-test/issues)。**

### 6.2 关于 Mock 工具的选型

这里我们简单说一下 Mock 工具的选型，是选择 Jmockit 还是 Mockito？首先 Spring 官方推荐使用的 Mock 框架就是 Mockito，所以 Spring 官方在只提供了 Mockito 的 Mock 支持，所以我们可以非常简单的只用 @MockBean 和 @SpyBean 注解就可以解决非常复杂的 Spring 上下文自动注入 Mock 对象的问题，如果我们要使用 Jmockit 那我们就得自己写这个实现，本着不重复造轮子的原则，我是推荐使用 Mockito 的。

**1.Mockito 的社区活跃度远超 Jmockit，并且 Jmockit 已经一年多没有更新了**

开源项目一个重要的指标就是社区活跃度，社区活跃说明关注的人多，随之带来的就是有问题能够更快速的响应和修复，这对我们使用方的选型来说是一个非常关键的点

<div align=center>
<img src="https://raw.githubusercontent.com/bigwg/bigwg.github.io/main/image/mockito_github.png"/>
</div>

<div align=center>
<img src="https://raw.githubusercontent.com/bigwg/bigwg.github.io/main/image/jmockit_github.png"/>
</div>

Jmockit 官方开了一个 Jmockit2 的项目，但是同样是很久没有活跃了

<div align=center>
<img src="https://raw.githubusercontent.com/bigwg/bigwg.github.io/main/image/jmockit2_github.png"/>
</div>

**2.我们的项目大部分都是依赖于 Spring 的，但是 Spring 对 Mock 的支持方面仅支持了 Mockito，并没有支持 Jmockit**

<div align=center>
<img src="https://raw.githubusercontent.com/bigwg/bigwg.github.io/main/image/spring_mockito.png"/>
</div>

## 七、参考资料

[Mockito 官网](https://site.mockito.org/)

[H2DB Github](https://github.com/h2database/h2database)

[Database-rider Github](https://github.com/database-rider/database-rider)

[Spring Boot Test官方文档](https://docs.spring.io/spring-boot/docs/1.5.2.RELEASE/reference/html/boot-features-testing.html)

[Maven Surefire 插件官方文档](http://maven.apache.org/surefire/maven-surefire-plugin/test-mojo.html)

[Mockito: doReturn vs thenReturn](http://sangsoonam.github.io/2019/02/04/mockito-doreturn-vs-thenreturn.html)

[Springboot单元测试:SpyBean vs MockBean](https://juejin.cn/post/6881981078735699976)

[Github issue: AttachNotSupportedException: no providers installed](https://github.com/mockito/mockito/issues/978)
