# 前言

对于框架的学习，我觉得还是先从会用开始。最近逐步开始进行开发实战，越发感觉到自己知识的贫瘠：很多开发中遇到的问题，我甚至不知道该怎样开口去问。而阻挡我的，往往却是一些非技术层面的问题，即工具的使用 。古语有云：工欲善其事，必先利其器。所以，还是先从会用开始，一点点的抽丝剥茧吧。加油！

# 常用注解

@Autowired：自动装配，其作用是为了消除Java代码里面的Getter&Setter方法以及Bean的property值。其默认按照类型匹配的方法，在容器中自动寻找相匹配的Bean。当有且只有一个匹配的Bean被搜寻到时，Spring将其注入到@Autowire的标注的变量中。

@Component：调用无参构造函数创建一个Bean对象，并把对象存入Spring的IOC容器中，交由Spring容器进行管理（相当于用XML的方式配置Bean）。value值指定Bean的ID。如果value为空，则默认Bean的ID是当前类的类名。泛指组件。

@Controller：控制层注解，标记在一个类上。用它标记的类就是一个SpringMVC Controller对象，分发处理器会扫描所有使用该注解的类的方法，并检测该方法是否使用了@RequestMapping注解。用于标注控制层组件，分发请求。

注意：@Controller只是定义了一个控制器类，而使用@RequestMapping注解的方法才是处理请求的处理器。 

```java
@Controller
public class Helloworld{
    //value值即为本次请求路径
	//当用户在浏览器地址栏输入hello后即可访问printHello方法
	@RequestMapping(value="/hello")
	public String printHello() {
		return "hello";
	}

    @Autowried
    private IocSerevce service;
    public void add(){
        service.add();
    }
}
```

@Service：业务逻辑层注解。该注解默认按照名称进行装配，名称可以通过name属性指定。如果没有name属性，默认去寻找注解所在的字段名。如果注解写在setter方法上，则默认按照方法属性名称进行装配。当找不到匹配的bean时，就按照类型进行装配。name名称一旦指定就会按照名称进行装配。用于标注业务层组件，处理具体事务。

@Repository：持久层注解，用于标注数据访问组件，即DAO。其作用是将数据访问层 (DAO 层 ) 的类标识为 Spring Bean，具体只需将该注解标注在 DAO类上即可。

@Repository、@Service、@Controller 和 @Component 都是将类标识为Bean。之所以这样做，是因为标注为Bean之后就方便配置，便于Spring容器进行统一管理，大大简化了web的开发。