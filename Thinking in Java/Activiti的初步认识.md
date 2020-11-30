这两天项目经理给我提了一个新的需求：更改一下公司办公系统内物品申领的步骤。原来的步骤如下图所示：

![image-20200909153123669](img/Activiti的初步认识/image-20200909153123669.png)

产品经理的需求是把流程中部门主管审核这一步给去掉，即提交领用单后直接由管理部门审核，审核通过即可。

拿到这个需求后，我遂即开始思考解决方案。由于之前没有搞过关于流程方面的事情，所以上来就想当然地以为这些是靠纯代码来解决的。于是，我便开始拿到请求头，定位至对应的控制器和方法，一层一层开始搞起。理了一会儿代码逻辑后，问题自然而然的出现了：系统是怎么知道所有的流程的呢？

```java
public enum UseManagementStatus {
    DEPARTMENT_HEAD_REVIEW("部门主管审核", 1),
    MANAGEMENT_REVIEW("管理部门审核", 2),
    EXAMINATION_PASSED("审核通过", 3),
    RETURN("退回", 4),
    COMPLETE_THE_USE("完成领用", 5),
    CANCEL_THE_USE("取消领用", 6),
    ;

    private String status;
    private int index;

    UseManagementStatus(String status, int index) {
        this.status = status;
        this.index = index;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

    public int getIndex() {
        return index;
    }

    public void setIndex(int index) {
        this.index = index;
    }
}
```

系统中的枚举类虽然定义好了各个步骤的名称，可是系统是如何使得对应的人员得到自己所需审核的信息的呢？

带着这个问题继续往下扒，我终于找到了上面这张流程图（PS：刚开始我并不知道系统内还有定义好的流程图）以及一个名为Activiti的神奇工具。

Activiti是一个基于Apache许可的开源BPM平台。BPM（Business Process Management），即业务流程管理。对于一个企业管理系统来说，其内部肯定存在着诸多办事流程，譬如员工的请假、办公用品的领用、费用的报销、财务的报账等等。引入BPM平台后，这些流程就可以被开发为一个个可运行的程序，嵌入企业管理系统内作为系统的一部分。这样一来，所有流程中所需的步骤和待处理的事情就会分配给特定人员，极大地提高了效率。

当然，要想使用Activiti组件，必须引入起步依赖

```java
//activiti起步依赖
compile("org.activiti:activiti-spring-boot-starter-basic:5.22.0")
compile('org.activiti:activiti-modeler:5.22.0') {
    //排除Security依赖，否则需要输入密码
    exclude group: 'org.springframework.security'
}
//引入后便可以在线绘制流程图
 compile ('org.activiti:activiti-explorer:5.22.0') {
        exclude group: 'org.vaadin.addons', module: 'dcharts-widget'
        exclude group: 'org.slf4j', module: 'slf4j-log4j12'
    }
```

引入依赖后，编写配置类，生成流程控制所需的数据表

```java
@Configuration
public class ActivitiConfig {
    @Autowired
    private PlatformTransactionManager transactionManager;
    @Autowired
    private DataSource dataSource;

    /**
     * 初始化配置
     *
     * @return
     */
    @Bean
    public SpringProcessEngineConfiguration processEngineConfiguration() {
        SpringProcessEngineConfiguration configuration = new SpringProcessEngineConfiguration();
        // 执行工作流对应的数据源
        configuration.setDataSource(dataSource);
        // 是否自动创建流程引擎表
        configuration.setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE);
        configuration.setAsyncExecutorActivate(false);
        // 流程历史等级
        configuration.setHistoryLevel(HistoryLevel.FULL);
        // 流程图字体
        configuration.setActivityFontName("宋体");
        configuration.setAnnotationFontName("宋体");
        configuration.setLabelFontName("宋体");
        configuration.setTransactionManager(transactionManager);
        return configuration;

    }
}
```

由于我们在执行工作流程时会产生很多数据（执行该流程的人是谁、所需要的参数是什么、包括想查看之前流程执行的记录等等），所以必须用数据表来保存。而Activiti框架则会自动帮我们生成工作表

![image-20200909162315955](/img/Activiti的初步认识/image-20200909162315955.png)

以上就是自动生成的工作表，其具体含义如下：

```java
/**
 * Activiti的后台是有数据库的支持，所有的表都以ACT_开头。 第二部分是表示表的用途的两个字母标识。 用途也和服务的API对应。
 * ACT_RE_*: 'RE'表示repository。 这个前缀的表包含了流程定义和流程静态资源 （图片，规则，等等）。
 * ACT_RU_*: 'RU'表示runtime。 这些运行时的表，包含流程实例，任务，变量，异步任务，等运行中的数据。 Activiti只在流程实例执行过程中保存这些数据， 在流程结束时就会删除这些记录。 这样运行时表可以一直很小速度很快。
 * ACT_ID_*: 'ID'表示identity。 这些表包含身份信息，比如用户，组等等。
 * ACT_HI_*: 'HI'表示history。 这些表包含历史数据，比如历史流程实例， 变量，任务等等。
 * ACT_GE_*: 通用数据， 用于不同场景下，如存放资源文件。
 */
```

了解了前期知识后，我便开始绘制变更后的流程图

![image-20200909163147415](/img/Activiti的初步认识/image-20200909163147415.png)

绘制完毕，需要把新的流程图保存并发布。保存完毕，数据库对应的ACT_RE_MODEL表便会新增一条数据，储存着当前流程图的ID。流程图的具体内容，会存储到对应的数据表中。流程图这一步搞定后，又解决了如下几个代码层面的问题

Q1：对应用户如何知道需要自己审核的事件？

A：事件监听器。定义好自己的实现TaskListener接口的监听器，用于监听系统发布的新事件。

```java
/***
 *  领用单： 管理部门 监听器
 */
public class UseManagementDeptListener implements TaskListener {

    IdentityService identityService = SpringContextHolder.getBean(IdentityService.class);
    RepositoryService repositoryService = SpringContextHolder.getBean(RepositoryService.class);
    ActivitiTaskService activitiTaskService = SpringContextHolder.getBean(ActivitiTaskService.class);

    @Override
    public void notify(DelegateTask delegateTask) {
        String useNumber = delegateTask.getVariable("useNumber", String.class);
        String deptGp = delegateTask.getVariable("deptGp", String.class);
        activitiTaskService.sendMsg(delegateTask.getVariable("sale"), deptGp, "管理部门审核",
                "您有新的领用单待审核，请及时处理！领用单编号：" + useNumber);
        String category = repositoryService.createProcessDefinitionQuery()
                .processDefinitionId(delegateTask.getProcessDefinitionId()).singleResult().getCategory();
        delegateTask.setCategory(category);
        delegateTask.addCandidateGroup(deptGp);
    }
}
```

由于我所需要做的只是流程的删除，并不是新增，所以监听器不变即可（当然也可以删除一个不需要的监听器，但是不建议，原因下文解释）

Q2：怎么能把审核消息推送给对应用户？

A：整合SpringSecurity开发，定义好人员分组及具体权限。

Q3：少了一个步骤后，之前一提交的和正在审核中的单子怎么办？

A：刚开始，我把多余的流程的相关代码全部删除，只留现在需要的，结果导致之前的单子全崩了，无法审核，就僵在了那里。我问了下产品经理，他给我提的需求是之前的单子不变，还能走原来的流程，从系统更新了新的流程表的那一刻开始，单子才走新的流程表。于是我便对代码进行了若干改造

```java
@Override
public Map<String, List<FinanceReimburseUserDto>> auditAllProcess(Long id) {
    UseManagement useManagement = useManagementRepository.getOne(id);
    Map<String, Object> useManagementId = activitiTaskService.getProcessVariables("useManagementId", id);
    Map<String, List<FinanceReimburseUserDto>> userDtoMap = Maps.newLinkedHashMap();
    String deptGp = (String)useManagementId.get("deptGp");
    //之前没有判断，这里加上是为了兼容老的流程图，防止不能走老流程图
    if(useManagementId.containsKey("gp")){
        String gp = (String)useManagementId.get("gp");
        Map<String, List<User>> usersMap = activitiGroupService.findUser(Lists.newArrayList(deptGp,gp));
        List<User> users = financeReimburseApplyService.departmentFilterAudit(usersMap.get(gp),
                useManagement.getEmployee());
        userDtoMap.put("部门主管审核",FinanceReimburseUserDto.toUserDto(users));
        userDtoMap.put("管理部门审核",FinanceReimburseUserDto.toUserDto(usersMap.get(deptGp)));
    }else{
        Map<String, List<User>> usersMap = activitiGroupService.findUser(Lists.newArrayList(deptGp));
        userDtoMap.put("管理部门审核",FinanceReimburseUserDto.toUserDto(usersMap.get(deptGp)));
    }
    return userDtoMap;
}
```

最关键的，是Activiti框架自身具有我姑且称之为“记忆”的功能，即更改流程图后，新的单子会自动走新上传发布的流程图，而老单子会走之前的图。至于具体实现的原理我结合对应的数据表，初步分析是因为每一个流程图都在数据表里有记录，而每走一步都会在对应的表中创建一条记录，其中记录的具体内容包含了所走流程表的ID，这样，就保证了所有流程的步骤走的是唯一流程表，也就实现了上述需求。

为了验证这个猜想是否正确，我把新流程图中某些步骤的ID给重命名了一下，上传发布为新新流程表（没有删除，只是更新）然后去审核之前基于新流程表创建的单子。果然，单子报错提示找不到具体流程。由此，可以证明上述猜想基本成立。

关于Activiti目前只是搞了两天，大概知道了其具体的内容，其中核心部分的例如如何加载、步骤如何存储、怎么更新我都没怎么具体翻源码，仅处于只知其然不知其所以然的水平。所以，我做的时候都是把这些步骤当成一个黑盒来处理，等以后再慢慢理解吧，毕竟这部分我感觉以后肯定得经常打交道。