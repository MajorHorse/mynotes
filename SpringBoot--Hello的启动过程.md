# 前言

引入SB框架后，我们只需要把Application类的main函数给启动后，就可以成功的在浏览器端访问到项目。回想一下，没有使用SB框架时，我们需要手动配置java环境，然后再安装启动Tomcat，构建完项目后还要对项目进行打包，整成一个war包，然后再把war包拷贝到Tomcat的webapp目录里面部署好，最后启动Tomcat，我们才能成功地访问到项目。可以看出，有了SB框架，我们简直可以说是简单的直接起飞。但是，在方便的同时问题也来了：我啥也没配置，它咋就能访问了呢？

# POM文件详解

先看一下pom文件的完整结构

```xml
<!--第一部分：继承了springboot框架的父级依赖，用来真正管理SB应用里面的所有依赖版本-->
<!--spring-boot-dependencies也就是SB的版本仲裁中心-->
<parent>
    <artifactId>spring-boot-dependencies</artifactId>
    <groupId>org.springframework.boot</groupId>
    <version>2.3.0.RELEASE</version>
</parent>
<dependencies>
        <!--第二部分：springboot框架开发web项目的起步依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
</dependencies>
```

这是引入SB框架的必备依赖，点入spring-boot-dependencies，又是一个依赖库，其中有一个property标签

```xml
<properties>
  <activemq.version>5.15.12</activemq.version>
  <antlr2.version>2.7.7</antlr2.version>
  <appengine-sdk.version>1.9.80</appengine-sdk.version>
   <!--....还有很多-->
  <tomcat.version>9.0.35</tomcat.version>
  <unboundid-ldapsdk.version>4.0.14</unboundid-ldapsdk.version>
  <undertow.version>2.1.0.Final</undertow.version>
  <versions-maven-plugin.version>2.7</versions-maven-plugin.version>
  <webjars-hal-browser.version>3325375</webjars-hal-browser.version>
  <webjars-locator-core.version>0.45</webjars-locator-core.version>
  <wsdl4j.version>1.6.3</wsdl4j.version>
  <xml-maven-plugin.version>1.0.2</xml-maven-plugin.version>
  <xmlunit2.version>2.7.0</xmlunit2.version>
</properties>
 <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
        <version>${spring-boot.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-tomcat</artifactId>
        <version>${spring-boot.version}</version>
      </dependency>
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-undertow</artifactId>
        <version>${spring-boot.version}</version>
 </dependency>
```

property标签下面又是引入的各种外部依赖。不难看出，这个标签里面定义的就是下面引入的各种外部依赖的版本号。仔细看一下里面的内容，发现关于开发中各种常用的东西都包含在里面了。由于里面都已经标注了默认的版本号，所以我们导入SB框架的默认依赖时是不需要写版本号的（当然，没在dependencies管理的仍然需要写版本号）

各种外部依赖通过parents标签定义好了，那么它们又是怎样被导入到我们的项目中去的呢？这里就要看第二部分的spring-boot-starter-web了，这里要说明一点，spring-boot-starter-web了应该分成两部分来看，即spring-boot-starter和web，spring-boot-starter被称为是spring-boot场景启动器，所以这里的意思是引入spring-boot-starter的web模块，即帮我们导入了web模块正常运行所依赖的组件。我们点进去看一下

```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>2.3.0.RELEASE</version>
    <scope>compile</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
    <version>2.3.0.RELEASE</version>
    <scope>compile</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <version>2.3.0.RELEASE</version>
    <scope>compile</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <scope>compile</scope>
  </dependency>
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <scope>compile</scope>
  </dependency>
</dependencies>
```

可以看到，这个文件里面也是引入的各种依赖，其中就包括Tomcat，所以，也就不需要我们自己再配置了。

总结一下就是，parent父标签说明了该项目是一个SB项目，可以使用SB框架的诸多功能，而下面的spring-boot-starter-web则引入了web开发的必备依赖到我们的项目中去。需要认识到，SB框架是一个功能广泛的框架，它的作用不仅仅局限于web开发，它将所有的功能场景都给抽取出来，做成一个个starter（启动器），我们开发时只需要在项目里面导入这些启动器，那么相关的依赖就会导入进来。我们想要用什么功能，就去导入什么starter。

# 关于主程序

主程序的main方法即为程序入口，看一下详细代码

```JAVA
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

@SpringBootApplication:该注解是用来确定程序入口，它标注在SB应用的某个类上，说明该类是SB的主配置类，启动时SB会通过运行该类的main方法来启动SB应用

点入@SpringBootApplication标签内，发现其上面也有许多注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

@SpringBootConfiguration：SB的配置类。它标注在类上表明该类是一个SB的配置类

配置类的作用和配置文件一样，就是做一些基本配置

@EnableAutoConfiguration：开启自动配置功能。运行程序时我们没有做任何配置，既没有配Tomcat，也没有打包，但是应用就可以直接访问了，这就是自动配置帮我们做的。该标签告诉SB开启自动配置的功能，以前我们需要手动做的，让SB以默认的方式自动给我们搞好。

# 自动配置原理

点进@EnableAutoConfiguration标签内

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

@Import({AutoConfigurationImportSelector.class})：导入哪些组件的选择器。点进去可以看到一个方法

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```

这个方法其实就是告诉Spring容器需要导入哪些组件，将所有需要导入的组件以全类名的方式返回，这些组件会被添加到容器中。这个注解会给容器中导入非常多的自动配置类(xxxAutoConfiguration)，这些自动配置类的作用是给容器中导入这个场景需要的所有组件并配置好这些组件。

@AutoConfigurationPackage：自动配置包，作用是将主配置类（使用@SpringBootApplication标注的类）所在的包及下面所有子包里面的组件都扫描进容器中。这里也就解释了为什么controller类必须是Application类所在包的类或者是子包的类，否则会报异常。

```java
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
}
```

@Import({Registrar.class})：@Import为Spring底层注解，作用是给容器导入一个组件，导入的组件由括号里面的Registrar.class来指定。点进Registrar.class

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        AutoConfigurationPackages.register(registry, (String[])(new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0]));
    }

    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new AutoConfigurationPackages.PackageImports(metadata));
    }
}
```

导入的关键语句在这里

```java
new AutoConfigurationPackages.PackageImports(metadata)).getPackageNames().toArray(new String[0])
```

metadata：注解的元信息

getPackageNames()：得到包名

这段语句的作用是得到注解标注的类所在的包及所有子包中的所有组件都扫描进容器中

一句话总结，SB在启动时从类路径下的META-INF/spring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效了，帮我们进行自动配置工作。