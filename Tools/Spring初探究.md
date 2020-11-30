# 							Spring初探究之IoC思想

## 前言

​		之前学习Java时一直在有意识的回避框架部分，这也就导致了本科时自己虽然也大大小小的做了不少东西，但本质上都是在重复的造轮子，并没有对所学的知识进行一个细致的梳理以及深层次的探究。说来也惭愧，直到现在我才意识到之前自已一直都是在原地踏步，甚至是略有退步，学习的知识全部都零零散散，浅尝辄止，不成体系。学习要想取得长久的、实质性的进步，总结和反思必不可少。而在编程领域，我觉得领悟和练习变得至关重要。不自己码一遍，理一下思路，永远也不可能真的学会，更别说运用了。

## Spring是什么&为什么要用

​		Spring是一个开源的开发框架，它的诞生是为了解决企业级应用开发的复杂性，现在已经不仅局限于企业应用，而是被全世界众多的web开发者所喜爱并运用。框架的主要优势就在于其分层的架构，分层架构允许使用者自主选择使用哪一个组件。同时，框架也集成了大量已经封装好的功能，方便开发者调用，大大节省了开发时间。Spring使用JavaBean来完成以前只能由EJB来完成的事情。简单来讲，Spring是一个核心思想为控制反转（IoC）和面向切面（AOP）的分层的JavaSE/EE一站式轻量级开源框架。

### 何为IoC（控制反转）

​		IOC：Inversion of Control，即控制反转，它并不是一种技术，而是一种编程思想。在Java开发中IoC意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

### 传统开发模式

​		在传统开发模式中，我们要运用MVC的思想，将开发分为三层，而我们后端程序员的开发工作则大部分集中在Controller层。首先，需要先在dao层写一个接口，用来对数据库进行crud操作

```java
public interface UserDao {
    void getUser();
}
```

​		然后需要再写一个实现类，实现这个接口

```java
public class UserDaoImpl implements UserDao{
    public void getUser() {
        System.out.println("默认方法查询用户信息");
    }
}
```

​		然后，我们还得在service层写一个接口，用来处理用户提交的请求等操作

```java
public interface UserService {
    void getUser();
}
```

​		最后，我们在对这个接口写一个实现，用来调用dao层的方法

```java
public class UserServiceImpl implements UserService{
    private UserDao userDao=new UserDaoImpl();
    public void getUser() {
        userDao.getUser();
    }
```

​		这样一通操作下来，也就完成了一个最基本的业务操作。在不涉及更新迭代添加新需求的情况下，这样看起来很美好：逻辑清晰，层次合理。这时，客户来了个新需求：我想用MySQL的方式来查询数据库。这也好办，我们只需要再写一个dao的实现即可

```java
public class UserDaoMySQLImpl implements UserDao{
    public void getUser() {
        System.out.println("用MySQL方查询用户信息");
    }
}
```

​		然后，在service层里修改下调用的方法

```java
public class UserServiceImpl implements UserService{
    //private UserDao userDao=new UserDaoImpl();
    private UserDao userDao=new UserDaoMySQLImpl();
    public void getUser() {
        userDao.getUser();
    }
}
```

​		恩，看起来也不麻烦嘛。此时，客户又来了个新需求：整啥MySQL，Oracle多洋气啊！于是，我们只能再添加实现类

```java
public class UserDaoOracleImpl implements UserDao{
    public void getUser() {
        System.out.println("用Oracle方法查询用户信息");
    }
}
```

​		service层里面继续修改

```java
public class UserServiceImpl implements UserService{
    //private UserDao userDao=new UserDaoImpl();
    //private UserDao userDao=new UserDaoMySQLImpl();
    private UserDao userDao=new UserDaoOracleImpl();
    public void getUser() {
        userDao.getUser();
    }
}
```

​		此时，用户新需求有来了：我忽然觉得sqlserver好像更洋气一点.....

​		我们：qnmd，屁事儿这么多！能不能一步到位啊？！老子不干了，受这委屈！

​		我们愤怒的准备卷铺盖走人。收拾东西之余，回想起这么些年在编程江湖起起伏伏，顿觉心有不甘，便禁不住开始思考起来，看看这事儿能不能有更优雅的解决方案。

### 引入IoC思想

​		再重新思考一下传统开发模式的开发过程，发现整个过程虽然是客户在提需求，但是实际操作者仍然是我们程序猿自己。换句话说，客户的需求是复杂多变的，而我们的开发模式及项目的整体架构就导致了客户每次的新需求都需要我们来重新编写实现类，改实现方法来完成用户的需求。要知道，开发的最优解是高内聚、低耦合。显然，我们的项目耦合度过高，层层嵌套，牵一发而动全身。用户想加一个方式，我们需要改好几个核心类。这只是一个小项目而已，要是那种超大型项目，我们这样直接修改的操作简直是一场灾难。有什么办法能让我们在不更改核心类的前提下，用户提什么需求自己就能实现呢？

```java
public class UserServiceImpl implements UserService{
    //private UserDao userDao=new UserDaoImpl();
    //private UserDao userDao=new UserDaoMySQLImpl();
    //private UserDao userDao=new UserDaoOracleImpl();
    private UserDao userDao;
    public void setUserDao(UserDao userDao){
        this.userDao=userDao;
    }
    public void getUser() {
        userDao.getUser();
    }
}
```

​		在这里，我们引入了一个set操作，即我们不直接new对象，用户有什么需求，自己可以手动new对象后注入至service层。不要小看了这一个简单的set方法，这可是实现了主动权的一个变更，即主动权由开发者变成了用户。我们开发者只需要把各种实现类给写好，用户想用哪个就直接自己new哪个，无需更改service层的任何一行代码，这是个思想的跨越。

## 总结

​		借助Spring的IoC思想，把创建和查找依赖对象的选择权交给了容器，对象和对象是松散耦合，利于功能的复用，我们可以设计出更松耦合的优良程序，使程序的整个体系结构变得更加灵活。