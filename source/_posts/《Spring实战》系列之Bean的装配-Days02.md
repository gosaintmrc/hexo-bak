---
title: 《Spring实战》系列之Bean的装配-Days02
date: 2019-01-31 15:45:42
categories: 
- Spring
tags:  spring
---

## 2.1 回顾

对于我第一天在bean的装配中写的，是一些基本的语法或者是Spring本身的一些规定，但是我没有对此进行深究。接下来就让我们仔细的讨论一下细节问题。和传统的类的定义和方法的调用做一些比较。这样就会体现出Ioc的特点。 下面的UML图就是我之前定义的一个接口和自己的一个实现。

​					![](《Spring实战》系列之Bean的装配-Days02\1.png)

```java
public interface CompactDisc {
    void play();
}
```



```java
@Component
public class SgtPeppers implements CompactDisc {
    private String title="享受孤独的音乐";
    private String article="奏出和谐的篇章";
    public void play() {
        System.out.println(title+article);
    }
}
```



## 2.2 下面是我的测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = CDPlayConfig.class)
public class CDPlayerTest {
    @Autowired
    private CompactDisc cd;

    @Test
    public void test(){
        cd.play();
    }
}
```

## 2.3 下面是我的配置文件

```
@Configuration
@ComponentScan
public class CDPlayConfig {
}
```



简单的回顾一下，CompactDisc加了注解@Component之后告诉Spring他自己是一个Bean.而通过@ComponeScan注解扫描到则会个Bean.然后注解@AutoWired自动注入。 基于注解的Bean的装配，我们可以用图的方式明确下：

## 2.4 基于Java注解的bean的装配

此时当我们去掉@ComponentScan注解之后，就会报出如下的错误：

```java
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean found for dependency [cn.czg.test.CompactDisc]: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

没有对应的bean被定义。因为失去了这个注解之后，就再也扫描不到了。此时我们也可以显式的完成这个功能。 现在大家看到了，失去了这个注解后会出现错误，此时我们可以基于Java注解的形式实现bean的自动装配；

很简单，Java提供了相应的注解，@Bean,一般用在方法的上面。 我们之前使用@ComponentScan注解完成扫描，自动的创建bean,但是此时我们将配置文件稍加改造即可。去掉扫描注解后报错，但是添加@Bean注解就可以啦



```java
@Configuration
public class CDPlayConfig {
@Bean
public CompactDisc sgtPeppers(){
	return new SgtPeppers();
	}
}
```

[无论如何](https://www.baidu.com/s?wd=%E6%97%A0%E8%AE%BA%E5%A6%82%E4%BD%95&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，@Bean的作用就是为了获得接口的实力对象，我们在此方法中最终获取了对象实例。

```java
@Bean
public IFoo getFooInstance(){
    int floor =(int) Math.floor(Math.random() * 4);
    if(floor==1){
        return new FooDemo1();
    }else if(floor==2){
        return new FooDemo2();
    }else {
        return new FooDemo3();
    }
}
```

大家可以看看，这个功能还是很厉害的，随着条件的不同，可以创建不同的实例对象。 以上就是基于Java的注解对Bean的装配，现在如果我的CDPalyer需要sgtpapers对象，该怎么办呢？此时我们还是基于注解的配置：且看以下的代码：

```java
@Configuration
public class CDPlayerConfig {
    @Bean
    public CompactDisc compactDisc() {
        return new SgtPeppers();
    }
    @Bean
    public CDPlayer cdPlayer() {
        return new CDPlayer(compactDisc());
    }
}
```

2.3 基于Java注解的注入（构造器注入/属性注入）

上面的代码恰好完成了ComoactDisc的注入。需要说明的是：CDPlayer对象的构造器似乎在调用了方法compactsic();从而返回一个SgtPapers对象的实例。但是在Spring中，由于@Bean注解的存在，却是阻止了方法的调用，直接返回该方法创建的实例对象SgtPapers.假如说上述的代码再修改一下：

```java
@Configuration
public class CDPlayerConfig {
    @Bean
    public CompactDisc compactDisc() {
        return new SgtPeppers();
    }
    @Bean
    public CDPlayer cdPlayer() {
        return new CDPlayer(compactDisc());
    }
    @Bean
    public CDPlayer anotherCdPlayer() {
        return new CDPlayer(compactDisc());
    }
}
```

这样的话，如果说还是按照普通的方法进行调用，那么由于每一个构造CDPlayer的构造器传入了不相同的CompactDisc对象，因此就会产生多个CDPlayer对象，而Spring终的bean默认都是单例的。确保只要哪一个应用需要这个bean,都可以唯一的注入这个相同的bean.

```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc){
	return new CDPlayer(compactDisc);
}
```

也许上述的代码看起来会更加的直观一点。这其实通过cdPlay()方法传入一个CompactDisc对象，通过构造器完成注入。而且和@Bean CompactDisc没有直接的关系。 代码稍加改造，就是setter属性的注入：

```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc){
    CDPlayer cdPlayer=new CDPlayer(compactDisc);
    cdPlayer.setCompactDisc(compactDisc);
    return cdPlayer;
}
```



2.5 基于xml对bean的装配

这个就比较简单了。比如要装配CDPlayer:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans";
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance";
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd";>
    <!--声明bean,构造器注入了compactDisc对象-->
    <bean id="cdPlayer" class="cn.czg.bean.CDPlayer">
        <constructor-arg ref="compactDisc"/>
    </bean>
    <!--声明compactDisc的实例-->
    <bean id="compactDisc" class="cn.czg.bean.SgtPeppers"/>
</beans>
```

## 2.6 导入混合配置

在启动Spring的时候，需要将所有的bean加载到Spring容器当中，我们上面的bean在CDPlayerConfig类中进行了配置，但是并不是所有的bean都需要在这个类中进行配置bean,而且如果配置在一起，整个配置类会显示的及其笨重。比如现在我需要自己定义另外一个配置类，加载不同的bean:

```java
@Configuration
public class CDConfig {
    @Bean
    public CompactDisc compactDisc(){
        return new Sgtpapers();
    }
}
```

但是最终的，还是要在一个最终的配置文件中将其引入。@Import注解此时大显神威。

```java
@Configuration
@ComponentScan
@Import(CDConfig.class)
public class CDPlayerConfig {
    @Bean
    public CDPlayer compactDisc(CompactDisc compactDisc){
        return new CDPlayer(compactDisc);
    }
}
```

还可以将两个配置类组合在一起，组成一个新的配置类

```java
@Configuration
@Import({CDConfig.class,CDPlayerConfig.class})
public class SoundSystemConfig {
}
```

在这里我们定义BlankDisc，同样的也是实现了接口CompactDisc.

```java
public class BlankDisc implements CompactDisc {

    private String title;
    private String artist;
    private List<String> tracks;

    public BlankDisc(String title, String artist, List<String> tracks) {
        this.title = title;
        this.artist = artist;
        this.tracks = tracks;
    }

    public void play() {
        System.out.println("Playing " + title + " by " + artist);
            for (String track : tracks) {
                System.out.println("-Track: " + track);
            }
        }
}
```

此时我们不希望使用@Bean的方式去装配这个bean,而是希望通过XML的形式去装配它。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans";
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"; xmlns:c="http://www.springframework.org/schema/c";
xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans.xsd";>
    <bean id="blankDisc" class="cn.czg.soundsystem.BlankDisc"
            c:_0="Sgt.Paper's lonely Hearts Club Blank"
            c:_1="The Beatles">
    <constructor-arg>
        <list>
            <value>Sgt.Paper's lonely Hearts Club Blank</value>
            <value>Hgsd oijdasfjsa oiasf </value>
            <value>sdfq sgtfq t hyt</value>
            <value>sdfq sgtfq t hytsdarf</value>
        </list>
    </constructor-arg>
    </bean>
</beans>
```

在上述的配置中，c:_0或者c:_1中的0和1就是索引，也就是多参数的情况下使用的一种方式，比如上述的字段title和article这两个字段就可以这样写。value可以更加灵活的说明。下面的是List中的参数的值。此时我们也希望将配置文件和XML共同的加载到Spring文件中，此时就用ImportSource注解

```java
@Configuration
@ComponentScan
@Import(CDConfig.class)
@ImportResource("classpath:applicationContext.xml")
    public class CDPlayerConfig {
    @Bean
    public CDPlayer compactDisc(CompactDisc compactDisc){
        return new CDPlayer(compactDisc);
    }
}
```

## 2.7 在XML 中引入JavaConfig

此时我们的配置类直接以bean的方式配置就可以啦

![](《Spring实战》系列之Bean的装配-Days02\3.png)