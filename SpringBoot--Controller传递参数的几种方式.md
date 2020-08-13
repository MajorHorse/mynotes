SB框架Controller常用的接收传递参数的方式大致有两种：请求路径传参（即Header传参）和body传参。

### 一、请求路径传参

请求路径传参基本上都需要用到注解，用于注明具体的参数。

#### 方法1：@PathVariable 注解

此方法用于获取路径上的参数，在具体的controller方法里在参数前面加上此注解，具体的请求路径后面也要注明参数名称，即url/{参数}的形式：

```java
@RequestMapping("/delete/{id}")
public void delUser(@PathVariable int id){
    userService.del(id);
}
//请求路径的示例写法为：http://localhost:8080/test/delete/5
```

#### 方法2：@RequestParam 注解

此方法把需要传递的参数用注解标注后，然后在请求路径上以url?param1=value1&param2=value2...的形式进行参数传递

```java
@RequestMapping("/get")
public void get(@RequestParam int id){
    userService.get(id);
}
//请求路径的示例写法为：http://localhost:8080/test/delete?id=5
```

此方法有一个较大的便利之处就是顺带着设置好参数的值、参数的默认值以及是否是必须要传递的等诸多参数

```java
@RequestMapping("/get")
//这里已经设置好了参数isDog的值是true，默认是false，不是必须要传递的参数
public void get(@RequestParam int id,@RequestParam(value="true",required = false,defaultValue = "false") boolean isDog){
    userService.get(id);
}
```

### 二、body传参

用此种方法传参时，参数名称和具体的值等详细的信息不会在地址栏上展示，更为安全。

#### 方法1：@RequestBody

此方法以json格式传递参数，即键值对

```json
{
    "userName": "admin", 
    "password": "admin"
}
```

处理请求

```java
@PostMapping(path = "/print")
public void out(@RequestBody User user) {
    System.out.println(User.toString());
}
//请求路径的示例写法为：http://localhost:8080/test/print
//输出结果为json格式
```

#### 方法2：无注解传参

此种方式无需用到任何注解，但是需要提前把传递的参数给封装好

```java
@RequestMapping("/add")
public void createUser(UserDto user){
    userService.add(user);
}

import lombok.Data;

/**
 * @author Major Tom
 * @date 2020/8/13 14:26
 * @description 封装请求参数
 */
@Data
public class UserDto {
    private String userName;
    private String password;
    private String realName;
}
//请求路径的示例写法为：http://localhost:8080/test/add?userName=ma&password=1234&realName=jjj
//此种方式仍然需要在header或者body里面写好要传递的参数，传递后参数会被封装成我们已经定义好的具体对象
```

