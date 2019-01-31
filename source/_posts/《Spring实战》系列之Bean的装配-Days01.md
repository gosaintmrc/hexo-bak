---
title: 《Spring实战》系列之Bean的装配-Days01
date: 2019-01-31 15:42:57
categories: 
- Spring
tags:  spring
---

### 1 自动化装配bean

Spring通过两个方面实现对bean的自动装配 1 ) 组件扫描（component scaning）：Spring会自动发现Spring上下文中的bean 2 ) 自动装配（autowriting）:Spring自动满足bean之间的依赖

#### 1.1　创建可被发现的bean

现在我们创建一个接口：

```java
public interface CompactDisc {
    void play();
}
```

接着来一个实现类：

```java
import org.springframework.stereotype.Component;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@Component
public class SgtPeppers implements CompactDisc {
    private String title="享受孤独的音乐";
    private String article="奏出和谐的篇章";
    @Override
    public void play() {
        System.out.println(title+article);
    }
}
```

上述的@Component注解的含义是：声明该类是一个bean,此时Spring就有权利去管理这个对象，但是一般的情况下我们需要让Spring容器知道这个类是一个bean,光存在这个注解是不够的，因为Spring容器是发现不了这个注解的，此时可以开启注解的扫描模式，默认的情况下是关闭的。同样的有注解开启的方式和机遇XML的配置方式：比如我们正在CDPlayer这个类中开启这个自动注解的扫描；如下所示：因为我们要在这个类中使用SgtPeppers对象。

```java
package cn.czg.springdemo02;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@Configuration
@ComponentScan
public class CDPlayConfig {
}
```

上述中@Configuration的注解的作用是配置；@ComponentScan的作用是开启注解扫描，默认是该对象同包下的所有的类或者子包下的所有的类
下面是基于XML的配置方式：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xmlns:context="http://www.springframework.org/schema/context"
      xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
  <context:component-scan base-package="cn.czg.springdemo02"/>
</beans>
```

我们可以做相应的测试

```java
package cn.czg.springdemo02;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import static junit.framework.TestCase.assertNotNull;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = CDPlayConfig.class)
public class CDPlayerTest {
    @Autowired
    private CompactDisc cd;

    @Test
    public void test(){
        assertNotNull(cd);
    }
}
```

上述中的测试：
1) 使用的是SpringJunit4测试
2) @ContextConfiguration(classes = CDPlayConfig.class)的作用是去类CDPlayerConfig中去加载配置，因为在CDPlayerConfig中包含了注解@ComponenScan注解。
3) 因此我们最后使用的是断言测试，测试bean中包含CompactDisc这个对象吗。显然是测试通过的，是由Spring自行装配这个bean的。所有添加注解@Component都是bean,而使用ComponentScan注解就可以开启扫描。目前为止我们用到的一些注解及其作用做个总结：
@Component org.springframework.stereotype.Component; 依赖Context包
@Configuration org.springframework.context.annotation.Configuration; 依赖Context包
@ComponentScan org.springframework.context.annotation.ComponentScan; 依赖Context包
@RunWith(SpringJUnit4ClassRunner.class) org.junit.runner.RunWith; 依赖JUnit包以及Spring test包
@ContextConfiguration(classes = CDPlayConfig.class)
org.springframework.test.context.junit4.SpringJUnit4ClassRunner; 依赖test包
@Autowired org.springframework.beans.factory.annotation.Autowired; 依赖bean包
@Test org.junit.Test; 依赖Junit测试

#### 1.2　为组件扫描的bean命名

在使用XML配置bean的时候，我们一般用id来作为bean的唯一标识，但是使用注解也可以来实现：

```java
package cn.czg.springdemo02;

import org.springframework.stereotype.Component;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@Component("loneiest")
public class SgtPeppers implements CompactDisc {
    private String title="享受孤独的音乐";
    private String article="奏出和谐的篇章";
    @Override
    public void play() {
        System.out.println(title+article);
    }
}
```

在@Componenet(“xxx”)来为bean命名，作为唯一的标识，也可以使用Spring的依赖注入来完成.

```java
package cn.czg.springdemo02;

import javax.inject.Named;

/**
* Created by cao zhiguo on 2017/9/1.
*/
/*@Component("loneiest")*/
@Named("loneiest")
public class SgtPeppers implements CompactDisc {
    private String title="享受孤独的音乐";
    private String article="奏出和谐的篇章";
    @Override
    public void play() {
        System.out.println(title+article);
    }
}
```

#### 1.3　设置组件扫描的基础包

我们面临的问题是在上述的代码中，我们所有的类都在相同的包中进行的测试，因此@ComponentScan是没有任何属性的，但是如果我们要扫描不同的包该怎么实现呢？比如说我要装配的bean在不同的包中，此时我们就要为这个注解添加属性了。 基础包：就是以配置类所在的包为基础包，比如上述的代码我们要配置CDPlayerConfig类，因此他所在的包就是base-Package.但是如果我们的bean不在这个基础包中就尴尬了。我们的做法有下面的几种方式：

```java
package cn.czg.springdemo02;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@Configuration
@ComponentScan("cn.czg.springdemo02")
public class CDPlayConfig {
}
```

还有一种方式：设置基础包。两种方式都是一样的

```java
package cn.czg.springdemo02;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@Configuration
@ComponentScan(basePackages = "cn.czg.springdemo02")
public class CDPlayConfig {
}
```

大家可以看到，上述用到的是basePackages是复数的形式，因此可以以数组的形式扫描不同的包

```java
package cn.czg.springdemo02;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@Configuration
@ComponentScan(basePackages = {"cn.czg.springdemo02","cn.czg.gosaint"})
public class CDPlayConfig {
}
```

但是上述的做法还是不安全的，只能的IDE可能会提示你，但是不太智能的IDE应该不会提示你，因为你的基础包使用的是String类型的，因此什么类型都可以写，因此就有如下的做法，直接扫描具体的类。

```java
package cn.czg.springdemo02;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
* Created by cao zhiguo on 2017/9/1.
*/
@Configuration
/*@ComponentScan(basePackages = {"cn.czg.springdemo02","cn.czg.gosaint"})*/
@ComponentScan(basePackageClasses ={ CompactDisc.class,SgtPeppers.class})
public class CDPlayConfig {
}
```

这样的做的更好的有点在于代码因重构，有可能包发生了变化，但是具体的类是没有改变的，我们的扫描依然是正确的。更加安全。

#### 1.4　通过为bean添加注解实现自动装配

@Autowird的注解的使用

```java
package cn.czg.springdemo02;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * Created by cao zhiguo on 2017/9/2.
 */
@Component
public class CDPlayer implements MediaPlayer {
    private CompactDisc sc;

    @Autowired
    public CDPlayer(CompactDisc sc) {
        this.sc = sc;
    }

    public void play(){
        sc.play();
    }
}
```

在这里CDPlay构造器在构建对象的过程中，同时把sc以参数传递进来，进行注入。这是通过该构造器进行实例化并且自动装配这个bean.
@Autowired注解不仅可以用在构造器上，还可以用在setter方法或者任何方法之上。也是表明这种注入的关系。让我们下面看一看这两种的用法。

```java
package cn.czg.springdemo02;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
* Created by cao zhiguo on 2017/9/2.
*/
@Component
public class CDPlayer implements MediaPlayer {
    private CompactDisc sc;

    @Autowired
    public CDPlayer(CompactDisc sc) {
        this.sc = sc;
    }

    /**
    * setter方法
    * @param sc
    */
    @Autowired
    public void setSc(CompactDisc sc) {
        this.sc = sc;
    }

    /**
    * 一般的额普通方法
    * @param sc
    */
    @Autowired
    private void insert(CompactDisc sc){
        this.sc=sc;
    }

    public void play(){
        sc.play();
    }
}
```

关于下一节我们就来验证自动装配是否成立。