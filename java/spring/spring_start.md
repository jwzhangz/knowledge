
三种主要的装配机制：
1. 在xml中显式配置
2. 在Java中显式配置
3. 隐式的bean发现机制和自动装配

这几种配置方式可以混合使用。

建议尽可能使用隐式配置，如果使用显式配置，推荐使用类型安全且比xml更强大的JavaConfig。

### 创建项目

在IDEA中创建maven项目，添加spring依赖。然后import这些依赖。

***
### Demo

*CompactDisc.java*
```java
package soundsystem;

public interface CompactDisc {
    void play();
}
```

```java
package soundsystem;

import org.springframework.stereotype.Component;

@Component
public class SgtPeppers implements CompactDisc {
    private String title = "Sgt.Pepper's Longle Hearts Club Band";
    private String artist = "The Beatles";

    public void play() {
        System.out.println("Playing " + title + " by " + artist);
    }
}
```
这里使用了@Component注解，spring会为这个类创建bean，spring会根据类名为这个bean指定一个ID，就是类名第一个字母小写，sgtPeppers。扫描组件默认不开启，需要在config文件中使用@ComponentScan注解。spring会扫描这个包以及其子包，查找带有@Component的类。

```java
package soundsystem;

import org.springframework.context.annotation.ComponentScan;

@ComponentScan
public class CDPlayerConfig {
}
```

*测试代码*
```java
package soundsystem;

import org.junit.Assert;
import org.junit.Assert.*;

import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
 * Created by AA on 2017/10/22.
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = CDPlayerConfig.class)
public class CDPlayerTest {
    @Autowired
    private CompactDisc cd;

    @Test
    public void cdShouldNotBeNull() {
        Assert.assertNotNull(cd);
    }
}
```

使用SpringJUnit4ClassRunner，在测试时自动创建Spring的应用上下文。@ContextConfiguration表示加载CDPlayerConfig.class中的配置。@Autowired注解起自动注入的作用。

###### bean命名
所有bean都会有一个ID，如果不设置ID，spring会默认根据类名设置ID，类名第一个字母小写。也可以显示设置ID。
```java
@Component("beanName")
```

###### 设置扫描基础包
默认@ComponentScan以注解所修饰的配置类所在的包作为基础包扫描组件。

也可以显式指定：
```java
@Configuration
@ComponentScan("soundsystem")
public class CDPlayerConfig{}
```

也可以通过basePackages更清晰的配置基础包，并且可以配置多个基础包。
```java
@Configuration
@ComponentScan(basePackages="soundsystem")
public class CDPlayerConfig{}
```

```java
@Configuration
@ComponentScan(basePackages={"soundsystem", "video"})
public class CDPlayerConfig{}
```

通过 basePackageClasses 属性设置基础包。这些类所在的包将会作为基础包。也可以在包中创建一个空接口，用来配置扫描基础包，避免代码修改后引用失效。
```java
@Configuration
@ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})
public class CDPlayerConfig{}
```

###### 通过注解实现自动装配

Spring可以通过@Autowired注解实现自动装配。@Autowired也可以替换为Java的规范@Inject。

```java
//1. 修饰变量
@Autowired
private CompactDisc cd;

//2. 修饰构造函数
@Autowired
public CDPlayer(CompactDisc cd){
    this.cd = cd;
}

//3. 修饰Setter函数
@Autowired
public void setCompactDisc(CompactDisc cd)
```
@Autowired也可以用在任何函数上。

在Spring上下文中，必须有且只有一个符合要求的bean。如果没有符合的bean，则Spring会抛出一个异常。如果想要避免抛出异常，需要将@Autowired的required属性设置为null。但是变量没有被bean初始化，还是初始值null，被引用时也会出错。
```java
@Autowired(required=false)
```

如果有多个bean满足依赖，Spring也会抛出异常。

###### 通过Java装配bean

通常会将JavaConfig放到单独的包中，使它与其他的应用分离出来。因为它是配置代码，不包含业务逻辑。

```java
//@Configuration表明这是一个配置类
@Configuration
public class CDPlayerConfig{
    //@Bean注解声明这个方法返回一个ID为"sgtPeppers"的bean。
    @Bean
    public CompactDisc sgtPeppers(){
        return new sgtPeppers();
    }
    //也可以设置名字
    @Bean(name="longlyHeartsClubBand")
}
```
@Configuration表明这是一个配置类。

下面代码中，看起来是调用sgtPeppers()生成对象，传入CDPlayer的构造函数，但其实不是。sgtPeppers()方法上添加了@Bean注解，Spring会拦截这个方法所有的调用，并返回该方法已经创建的bean。默认情况下bean都是单例的。
```java
@Bean
public CDPlayer cdPlayer(){
    return new CDPlayer(sgtPeppers());
}
```

下面代码中，Spring在调用cdPlayer()方法时会自动装配compactDisc，只要这个bean已经在Spring上下文中。这个Bean不一定要在该Config文件中声明，可以以任意方式配置。
```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc){
    return new CDPlayer(compactDisc);
}
```

###### 配置profile

在3.1版本中，Spring引入了bean profile功能。可以设置为"dev","prod","qa"。
```java
//修饰配置类
@Configuration
@Profile("dev")
public class DevProfileConfig{
    
}
```

这个类中的bean只有在dev profile激活时才会被创建。

从Spring3.2开始，可以在方法级别上使用@Profile注解，与@Bean注解一起使用。

```java
@Bean
@Profile("prod")
```

###### 激活profile
通过设置spring.profiles.active和spring.profiles.default来激活profile。spring.profiles.active的值被设置为profile，如果没有设置spring.profiles.active，则使用spring.profiles.default的值。如果两个值都没有设置，所有设置profile的bean都不会被创建，只创建那些没有设置profile的bean。

多种方式设置这两个属性：
- 作为DispatcherServlet的初始化参数；
- 作为Web应用的上下文参数
- 作为JNDI条目
- 作为环境变量
- 作为JVM系统属性
- 在集成测试类上，使用@ActiveProfiles注解

标注当前测试用例使用的具体profile。
```java
@ActiveProfiles("test")
public abstract class SpringTransactionalTestCase {
}
```

###### 条件化的bean
