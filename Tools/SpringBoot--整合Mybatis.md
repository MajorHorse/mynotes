# 前言

SpringBoot框架的基础知识我在之前也断断续续的看过，但是当真正步入实战开发后，由于项目太过庞大，里面也杂糅了各种乱七八糟的东西，导致我看的时候晕得不行。尤其是在整合Mybatis这块，因为Mybatis框架我也没学过，所以看起来更加吃力。所以，索性从零开始，仔细盘一盘SpringBoot整合Mybatis整个的开发流程和各模块的详细用处，好好补补课。

### Step1 新建SB项目

直接选择用Spring Initializr新建项目，选择的依赖为SpringWeb、JDBC、Mybatis和MySQL Driver

![image-20200811150140647](/img/SpringBoot--整合Mybatis/image-20200811150140647.png)

### Step2 修改配置文件

将.property的配置文件修改为yml文件。当然，这一步并不是必须的，这样做的意义是为了切换开发环境。

在resource目录下新建一个application.yml为主配置文件，用于指定某个具体的开发环境

```yml
spring:
  profiles:
    active: dev
```

然后，新建application-dev.yml和application-test.yml等配置文件，这些文件内分别配置具体的环境，这样就可以在开发时动态指定运行环境

```yml
server:
  port: 8080

spring:
  datasource:
    username: root
    password: 1234
    url: jdbc:mysql://localhost:3306/mydemo?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC

mybatis:
  mapper-locations: classpath:mapper/*mapper.xml
  type-aliases-package: com.majortom.entity
```

### Step3 创建数据库

用Navicat在本地MySQL数据库新建一个名为mydemo的库，运行SQL建表命令

```sql
CREATE TABLE `user` (
  `id` int(32) NOT NULL AUTO_INCREMENT,
  `userName` varchar(32) NOT NULL,
  `passWord` varchar(50) NOT NULL,
  `realName` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
```

运行完毕后，数据库里面已经有了一个名为user的表

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema  |
| mydemo            |
| mysql              |
| performance_schema |
| test                |
+--------------------+
5 rows in set

mysql> use mydemo;
Database changed
mysql> use user;
1049 - Unknown database 'user'
mysql> show tables;
+------------------+
| Tables_in_mydemo |
+------------------+
| user             |
+------------------+
1 row in set

mysql> select * from user;
+----+-----------+-----------+----------+
| id | userName | passWord | realName |
+----+-----------+-----------+----------+
|  1 | MajorTom  | 123       | 哈哈哈   |
|  2 | meng     | 123       | heiheihei  |
+----+-----------+-----------+----------+
2 rows in set

```

### Step3 创建Entity、Controller、Service、Mapper

Entity:Userjava

```java
import lombok.Data;

/**
 * @author Major Tom
 * @date 2020/8/11 18:27
 * @description User实体
 */
@Data
//lombok的data注解默认已经生成了构造方法和getter、setter
public class User {
    private Integer id;
    private String userName;
    private String password;
    private String realName;
}
```

Controller:UserController.java

```java
import com.majortom.entity.User;
import com.majortom.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author Major Tom
 * @date 2020/8/11 18:35
 * @description
 */
@RestController
@RequestMapping("/test")
public class UserController {
    @Autowired
    private UserService userService;
    @RequestMapping("getUser/{id}")
    public User getUser(@PathVariable int id){
        return userService.get(id);
    }
    @RequestMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```

Service:UserService.java

```java
import com.majortom.entity.User;
import com.majortom.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * @author Major Tom
 * @date 2020/8/11 18:39
 * @description
 */
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;

    public User get(int id){
        return userMapper.get(id);
    }
}
```

Mapper&Mapper.xml

```java
import com.majortom.entity.User;
import org.apache.ibatis.annotations.Mapper;

/**
 * @author Major Tom
 * @date 2020/8/11 18:41
 * @description
 */
@Mapper
public interface UserMapper {
    User get(int id);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.majortom.mapper.UserMapper">
    <resultMap id="BaseUserResultMap" type="com.majortom.entity.User">
        <result column="id" jdbcType="INTEGER" property="id"/>
        <result column="userName" jdbcType="VARCHAR" property="userName"/>
        <result column="password" jdbcType="VARCHAR" property="password"/>
        <result column="realName" jdbcType="VARCHAR" property="realName"/>
    </resultMap>
    <select id="get" resultType="com.majortom.entity.User">
        select * from user where id =#{id}
    </select>
</mapper>
```

### Step4 启动

启动Application，在地址栏输入http://localhost:8080/test/getUser/1，可以看到返回json数据，大功告成！

|          |            |
| -------- | ---------- |
| id       | 1          |
| userName | "MajorTom" |
| password | "123"      |
| realName | "哈哈哈"   |