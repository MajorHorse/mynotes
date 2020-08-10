# 前言

这一部分的笔记本来不想写，觉得关于工具的使用没必要当成知识点来记，因为毫无技术含量，写起来也比较繁琐。可是当我真正点开IDE去准备新建项目进行编程时，却发现自己束手无策，不知从哪里入手第一步才好。虽说半个多月前也写过一个demo，但这才隔了几天，自己已经全忘光了。唉，真是老了。痛定思痛，还是决定把idea使用的一些技巧及较为重要的东西给写一下，哪怕是为了增强记忆也好。毕竟，开发平台都玩不溜，这也太不专业了。

# 过程

第一步，新建项目时选择maven，不采用任何模板

![image-20200706171536778](C:\Users\mylov\Desktop\笔记相关\img\maven-sb-step1.png)

第二步，在项目的resource目录下新建若干子目录，供开发需要。SB中有一句话：约定大于配置。所以，我们就用项目默认的名称来命名

![image-20200707145417211](C:\Users\mylov\Desktop\笔记相关\maven-sb-子目录.png)

​	第三步，导入SB所需的各种依赖，在pom文件中写的是两个必备部分

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!--第一部分：继承了springboot框架的父级依赖-->
    <parent>
        <artifactId>spring-boot-dependencies</artifactId>
        <groupId>org.springframework.boot</groupId>
        <version>2.3.0.RELEASE</version>
    </parent>
    <groupId>com.majortom</groupId>
    <artifactId>springboot-demo1</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--第二部分：springboot框架开发web项目的起步依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

第四步，写启动类。在java目录下新建Application类，作为程序的访问入口

```java
/**
 * @author Major Tom
 * @date 2020/7/7 15:16
 */
//springboot的全局自动配置注解
@SpringBootApplication
//以下代码均为固定写法
//springboot项目入口
public class Application {
    public static void main(String[] args) {
        //固定代码，启动springboot程序，初始化spring容器
        SpringApplication.run(Application.class, args);
    }
}
```

然后，在java目录下新建controller包，写一个自己的controller。注意，controller类必须是Application类所在包的类或者是子包的类，否则会报异常

```java
/**
 * @author Major Tom
 * @date 2020/7/7 15:20
 */
@RestController
public class HelloController {
    //访问路径
    @RequestMapping("hello")
    public String hello(){
        return "hello sb";
    }
}
```

第五步，验证。在浏览器地址啦输入http://localhost:8080/hello，即可访问到页面

![image-20200707153001319](C:\Users\mylov\Desktop\笔记相关\maven-sb-test.png)

# 小结

我们可以看到，引入SB框架后项目的新建过程相比原来而讲，简直不要太简单了。当然，这只是第一步，就像要修一座大厦似的，我们刚学会了锤子该怎么轮，后面的事情多着呢。加油!