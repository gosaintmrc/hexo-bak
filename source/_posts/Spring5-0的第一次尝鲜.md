---
title: Spring5.0的第一次尝鲜
date: 2019-01-31 16:23:00
categories: 
- Spring
tags:  spring
---

对于这次尝鲜，说白了和Spring5.0的新特性基本没有多大的关系，如果说您不小心进来了，却发发现文章的内容和标题似乎不太匹配，那么我将是非常的抱歉，因为这浪费了您宝贵的时间。但是我还是要说：因为这确实是Spring5.0中的一个demo.而我在这里写下这个Demo的原因是这个Demo全部是注解的配置，因为我的习惯还停留在XML的阶段。
好了，让我们引入context包吧，这里使用maven配置：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>5.0.0.RELEASE</version>
    </dependency>
</dependencies>
```

​    这个Demo的实现思路是这样的，首先我们定义一个接口，定义一个类，该类要调用该接口中的方法，但是又没有实现这个接口，此时我们可以考虑下怎么做呢？很简单的，在不涉及其他类的情况下貌似只能在该类中的该方法中使用匿名内部类的方式的完成，但是这和我们之前的说的是一致的，我们要解耦，就要注入。这个Demo说的就是注入。在该Demo中，这个类将该接口私有化并且最终化，以便是有构造器注入。然后在方法中直接调用即可。在配置类中使用匿名内部类的方式创建这个接口的bean。在main方法中首先加载配置类，获取上下文，然后用上下文调用方法所在的类，进而调用方法。其实真正的方法的执行体应该是接口的匿名实现类的方法。

```java
package hello;

public interface MessageService {
    String getMessage();
}
package hello;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class MessagePrinter {
    /**
     * 定义私有的，最终的成员变量，并且使用构造器进行初始化
     */
    final private MessageService service;

    @Autowired
    public MessagePrinter(MessageService service) {
        this.service = service;
    }

    public void printMessage(){
        System.out.println(service.getMessage());
    }
}
package hello;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
public class Application {
    @Bean
    MessageService mockMessageService(){
        return new MessageService() {
            public String getMessage() {
                return "Hello World!";
            }
        };
    }

    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(Application.class);
        MessagePrinter printer = context.getBean(MessagePrinter.class);
        printer.printMessage();//Hello World!
    }
}
```