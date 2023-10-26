---
title: Activiti7学习笔记
date: 2023-09-24 17:10:15
tags: Activiti7
toc: true
---

# 一、什么是工作流

- 多个参与者之间按照某种预定义的规则自动进行传递文档、信息或任务的过程，进而实现某种预期的业务目标
- 如：订单、客户订单处理、供应链管理、合同审核、请假申请、出差申请、加班申请、财务审批等等

#  二、Activiti7概述

## 2.1 概述

- 使用建模语言BNPMN2.0 进行定义
- 画出业务流程图会转成xml文件，然后可以读取文件

## 2.2 使用

1. 部署：activiti7是一个工作流引擎（jar包API），业务系统访问（操作）activiti的接口操作相关数据
2. 定义：使用activit7建模工具定义业务流程，生成.bpmn文件即通过xml定义业务流程。
3. 流程部署：存储.bpmn文件到数据库
4. 启动实例：启动一个流程实例表示开始一次业务流程的运行
5. 用户查询待办任务（task）：通过activiti查询 不需要sql语句
6. 用户办理任务：用户查询待办任务，进行办理。如果流程还未结束需要往下走，如请假审批还需要领导审批，acitivi会自动完成。
7. 流程结束：没有下一个任务点了就代表此次流程实例结束

## 2.3 Activiti7开发环境

1. jdk1.8+

2. mysql5.1.21+

3. Tomcat8.5+

4. IDEA（acticiti的流程定义工具插件可以安装再IDEA下，也可以再eclipse下）

5. 支持spring5

6. BPMN开源设计器（https://gitee.com/MiyueSC/bpmn-process-designer）

   1. 因为该设计器是用Node.js做的，所以需要先把Node.js环境安装好。

   2. 然后把项目解压缩，并且在命令行进入该项目文件夹，执行初始化命令`npm install`。

   3. 然后执行下面的命令，才能让这个BPMN项目兼容高版本的Node.js

      ```
      # Windows系统执行这个命令
      set NODE_OPTIONS=--openssl-legacy-provider
      
      # Linux和MacOS系统执行这个命令
      export NODE_OPTIONS=--openssl-legacy-provider
      ```

   		4. 启动设计器：在命令行中执行`npm run demo`命令启动BPMN项目，然后打开浏览器访问本地的8100端口就能看到该设计器了。
   		4. 

### 2.3.1 下载BPMN设计器

- git下载地址：https://github.com/miyuesc/bpmn-process-designer
- git在线地址：https://miyuesc.github.io/process-designer-v2/
- 本地使用方法：
  1. 因为该BPMN设计器是用Node.js做的，所以需要先把Node.js环境安装好
  2. 把项目解压缩，并且在命令行进入该项目文件夹，执行初始化命令:`cnpm install`
  3. 在命令行中执行`npm run demo`命令启动BPMN项目

### 2.3.2 数据库支持

1. activit的运行需要数据库的支持，使用25张表，把流程定义节点内容读取到数据库表中，供后续使用
2. 支持多种数据库及版本
   1. h2 ：1.3.168+
   2. mysql：5.1.21+
   3. oracle：11.2.0.1.0+
   4. postgres：8.1+
   5. db2：DB2 10.1 using db2jdbc4+
   6. mssql：2008 using sqljdbc4+

# 三、SpringBoot+MySQL搭建activiti7入门案例

## 3.1 创建mysql中activiti数据库的数据库

如：创建activiti`create database activiti default character set utf8;`

### 3.1.1 引入activiti依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!--activiti依赖-->
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring-boot-starter</artifactId>
    <version>7.0.0.GA</version>
</dependency>
<!--mysql依赖-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.33</version>
</dependency>
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.10</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 3.1.2 test下添加测试类`Activiti7Test01`

### 3.1.3 配置数据库信息

0. 注意配置`databaseSchemaUpdate`属性

1. 方法一：配置文件形式（默认形式）

   1. 通过`getDefaultProcessEngine()`方法获取流程引擎对象 会加载`resources` 目录下的 `activiti.cfg.xml`

   2. activiti.cfg.xml内容如下：

      ```java
      <?xml version="1.0" encoding="UTF-8"?>
      <beans xmlns="http://www.springframework.org/schema/beans"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns:context="http://www.springframework.org/schema/context"
             xmlns:tx="http://www.springframework.org/schema/tx"
             xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
      						http://www.springframework.org/schema/contex http://www.springframework.org/schema/context/spring-context.xsd
      						http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
          <!--activiti单独运行的ProcessEngine配置对象(processEngineConfiguration),使用单独启动方式
                  默认情况下：bean的id=processEngineConfiguration
              -->
          <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
          <property name="jdbcDriver" value="com.mysql.cj.jdbc.Driver"></property>
          <property name="jdbcUrl" value="jdbc:mysql://127.0.0.1:3308/activiti?nullCatalogMeansCurrent=true&amp;useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false&amp;serverTimezone=GMT%2B8"></property>
          <property name="jdbcUsername" value="root"></property>
          <property name="jdbcPassword" value="mysql1225"></property>
          <!--数据库生成策略-->
          <!--activiti在生成表的时候,true 表示数据库中如果已经存在,那么直接使用,如果不存在,则直接创建-->
          <property name="databaseSchemaUpdate" value="true"></property>
          </bean>
      
      <!--    &lt;!&ndash;数据库连接池的构造注入&ndash;&gt;-->
      <!--    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">-->
      <!--        <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>-->
      <!--        <property name="url" value="jdbc:mysql://127.0.0.1:3308/activiti?useUnicode=true&amp;characterEncoding=utf8&amp;useSSL=false&amp;serverTimezone=GMT%2B8"></property>-->
      <!--        <property name="username" value="root"></property>-->
      <!--        <property name="password" value="mysql1225"></property>-->
      <!--        <property name="maxActive" value="30"></property>-->
      <!--        <property name="minIdle" value="5"></property>-->
      
      <!--    </bean>-->
      <!--    <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">-->
      <!--        &lt;!&ndash;代表数据源&ndash;&gt;-->
      <!--        <property name="dataSource" ref="dataSource"></property>-->
      <!--        &lt;!&ndash;代表是否生成表结构&ndash;&gt;-->
      <!--        <property name="databaseSchemaUpdate" value="true"/>-->
      <!--    </bean>-->
      
      </beans>
      
      ```

   3. 编写测试方法

      ```java
       /**
           * 获取processsEngine对象的第一种方式
           * 默认的获取方式
           */
          @Test
          public void test01(){
              //通过getDefaultProcessEngine()方法获取流程引擎对象 会加载resources 目录下的 activiti.cfg.xml
              ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
              System.out.println(processEngine);
          }
      ```

2. 方法二: java代码形式

   1. 基于java代码获取processEngine对象

   2. 直接编写测试方法：

      ```java
        /**
           * 获取processsEngine对象的第二种方式
           * 基于java代码获取processEngine对象
           */
          @Test
          public void test02(){
              ProcessEngine processEngine = ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration()
                      .setJdbcDriver("com.mysql.cj.jdbc.Driver")
                    //  .setJdbcUrl("jdbc:mysql://localhost:3308/activiti?serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=false")
                      .setJdbcUrl("jdbc:mysql://localhost:3308/activiti?nullCatalogMeansCurrent=true&serverTimezone=UTC&characterEncoding=utf8&useUnicode=true&useSSL=false")
                      .setJdbcUsername("root")
                      .setJdbcPassword("mysql1225")
                      .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_TRUE)//activiti在生成表的时候,true 表示数据库中如果已经存在,那么直接使用,如果不存在,则直接创建
                      .buildProcessEngine();
              System.out.println("processEngine = "+processEngine);
          }
      ```

### 3.1.4 数据库自动生成表

运行test方法后，即第一次获取流程引擎时，数据库自动生成act开头的表则创建成功。第二部分是说明用途的两字节标识符

ACT_RE_*：`RE`代表`repository`。带有“静态”的意思。例如：流程定义和流程资源（图片、规则等）

ACT_RU_*：`RU`代表`runtime`。保存“运行时信息”。例如：流程实例、用户任务、变量、作业等。Activiti只在流程实例运行中保存运行时数据，并在流程实例结束时删除记录，进而可以保证`小而快`

ACT_ID_*：`ID`代表`identity`。保存”身份信息“。例如：用户、组等

ACT_HI_*：`HI`代表`history`。保存“历史数据”。例如：已完成的流程实例、变量、任务等

ACT_GE_*：通用数据，用于不同场景下

## 3.2 创建BPMN流程图

1. 设计器根据需求画出流程图，导出xml文件放在`rescoures\bpmn`目录下，并重命名`*.bpmn20.xml`形式

2. xml文件右键`View BPMN(Activiti) Diagram`后，点击相关元素设置`Assignee`（任务处理人）等属性

   ![IDEA中查看BPMN](E:\Git_repositories\MyBlogSource\image\IDEA中查看BPMN.png)

## 3.3 流程操作

0. 数据库比较关键的两张表：`act_re_deployment`:流程部署表`act_re_procdef`：流程定义表

   注：一个流程部署可以包含多个流程定义

### 3.3.1 流程部署

1. 需要通过`repositoryService`来完成

```
/**
 * 流程部署操作
 */
@Test
public void test03(){
    //1. 获取processEngine对象
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    //2. 完成流程的部署操作，需要通过repositoryService来完成
    RepositoryService repositoryService = processEngine.getRepositoryService();
    //3. 完成部署
    Deployment deploy = repositoryService.createDeployment()
            .addClasspathResource("bpmn/LeaveApply.bpmn20.xml")
            .name("第一个流程")
            .deploy();
    System.out.println(deploy.getId());
    System.out.println(deploy.getName());
}
```

2. 流程定义信息查询：需要通过`repositoryService`来完成查询

```java
/**
 * 查询当前部署的流程有哪些
 */
@Test
public void test04(){
    //1. 获取processEngine对象
    ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
    //2. 需要通过repositoryService来完成查询
    RepositoryService repositoryService = processEngine.getRepositoryService();
    //repositoryService.createProcessDefinitionQuery()//查询流程定义信息
    List<ProcessDefinition> list = repositoryService.createProcessDefinitionQuery().list();
    System.out.println("流程定义信息如下：");
    for (ProcessDefinition processDefinition:list
         ) {
        System.out.println(processDefinition.getId());
        System.out.println(processDefinition.getName());
    }

    //repositoryService.createDeploymentQuery()//查询流程部署信息==》相关流程定义的信息
    System.out.println("流程部署信息如下：");
    List<Deployment> list1 = repositoryService.createDeploymentQuery().list();
    for (Deployment deployment:list1
         ) {
        System.out.println(deployment.getId());
        System.out.println(deployment.getName());
    }
}
```

3. 即 我们可以通过Activiti相关的API获取相关的流程部署和流程定义的信息

### 3.3.2 发起流程

1.  获取processEngine对象后，通过runtimeService获取流程定义的一个流程实例对象去发起流程 （流程定义：查act_re_procdef表）

2. 编写test

   ```java
       /**
        * 发起一个流程
        * 通过runtimeService获取流程定义的实例
        */
       @Test
       public  void test05() {
           //获取processEngine对象
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
           RuntimeService runtimeService = processEngine.getRuntimeService();
   
           //通过runtimeService获取流程定义的一个流程实例对象    流程定义：查act_re_procdef表
           ProcessInstance processInstance = runtimeService.startProcessInstanceById("MyLeave:1:3");
   
           System.out.println("processInstance.getId() = " + processInstance.getId());
           System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
           System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
   
       }
   ```

2. 发起流程成功后，可在数据库`act_ru_task`表中看到如下信息，`Assignee`是指当前流程处理人

![流程发起act_ru_task表](E:\Git_repositories\MyBlogSource\image\流程发起act_ru_task表.png)

### 3.3.3 查询流程

1.  查询待办执行中的任务处理是通过 TaskService来实现

2. Task 对象对应的其实就是 act_ru_task 这张表的记录

3. 编写test方法代码

   ```java
       /**
        *待办查询
        */
       @Test
       public void test06(){
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
   
           //查询待办 执行中的任务处理是通过 TaskService来实现
           TaskService taskService = processEngine.getTaskService();
           //Task 对象对应的其实就是 act_ru_task 这张表的记录
           List<Task> employeeList = taskService.createTaskQuery().taskAssignee("zhangsan").list();
           if(employeeList!=null && employeeList.size()>0){
               for (Task task : employeeList) {
                   System.out.println("task.getId() = " + task.getId());
                   System.out.println("task.getName() = " + task.getName());
                   System.out.println("task.getAssignee() = " + task.getAssignee());
               }
           }else{
               System.out.println("当前没有待办任务");
           }
       }
   ```

### 3.3.4 审批流程

1. 查询用户的所有待办，当前登录用户查询到相关审批流程信息后可以进行审批

2. 通过TaskService来实现

3. 编写test方法代码

   ```java
      /**
        * 完成待办审批
        */
       @Test
       public void test07(){
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
   
           TaskService taskService = processEngine.getTaskService();
           //taskService.complete("2505");//模拟1  模拟对当前登录用户（employee）完成当前审批节点 使得 Assignee 从 employee --> zhangsan
   
           //模拟2 对模拟流程稍作完善
           //List<Task> list = taskService.createTaskQuery().taskAssignee("zhangsan").list();//模拟对当前登录用户（zhangsan）完成当前审批节点使得 Assignee 从 zhangsan --> lisi
           List<Task> list = taskService.createTaskQuery().taskAssignee("lisi").list();//模拟对当前登录用户（lisi）完成当前审批节点  lisi是最后一个 Assignee  所以lisi完成后数据库中该task会被删除，
           if(list!= null && list.size()>0){
               for (Task task : list) {
                   //模拟对当前登录用户下的某个task做相关审批处理 完成该流程的本审批节点
                   //具体应该根据前端用户的操作数据获取值 进而进行下一步的相关审批
                   taskService.complete(task.getId());
               }
           }else {
               System.out.println("当前没有待办任务");
           }
       }
   ```

### 3.3.5 涉及的表

上面一个审批流程下来所涉及的表结构

| 表名              | 说明                                                   |
| ----------------- | ------------------------------------------------------ |
| act_re_deployment | 部署流程的记录表：一次部署行为会产生一张表             |
| act_re_procdef    | 流程定义表：一张流程图对应的表                         |
| act_hi_procinst   | 流程实例表：发起一个流程，就会创建对应的一张表         |
| act_ru_task       | 流程待办表：当前需要审批的记录表，节点审批后就会被删除 |
| act_hi_actinst    | 历史记录表：流程审批节点的记录信息                     |

# 四、任务分配

## 4.1 固定分配

如Part3案例代码中test方法所写，即把任务流程静态指派给固定的用户，正常业务中都要动态处理，基本不会用到该方式。

## 4.2 表达式

### 4.2.1 值表达式

1. xml配置文件中对应属性设置`${}`占位符
2. 编写test方法，`runtimeService.startProcessInstanceById`方法有重载，在传入流程定义id的同时可以通过map传第二个参数值

### 4.2.2 方法表达式

1. xml配置文件中对应位置设置`${}`，传入`已注册的bean名字.方法`实现调用
2. 测试方法和前边Part3测试方法相同，无需额外传入参数，参数由容器自动注入

## 4.3 监听器配置

1. 监听器分为三种：JavaDelegate TaskListener ExecutionListener

    * 用户任务（UserTask）的监听器为TaskListener
    * Java服务任务（Service Task）的监听器为JavaDelegate
    * 其他的服务的监听器为ExecutionListener
    * TaskListener中参数（DelegateTask）是有关于userTask的
    * JavaDelegate和ExecutionListener参数（DelegateExecution）是有关于流程的

2. 编写监听器类(根据需求实现指定监听器接口，如TaskListener )

    ```java
    //此类定义的是用户进程监听器
    public class MyFirstListener implements TaskListener {
        /**
         * 监听器触发回调方法
         * @param delegateTask
         */
        @Override
        public void notify(DelegateTask delegateTask) {
    
            System.out.println("----->自定义的监听器执行了");
            if(EVENTNAME_CREATE.equals(delegateTask.getEventName())){
                //表示是Task的创建事件被触发了
    
                //1.指定当前Task节点的处理人
                delegateTask.setAssignee("wangwu");
            }
        }
    }
    ```

3. 编写xml配置文件，**指定流程节点**添加扩展属性（以：taskListener为例）  

   ```java
   <bpmn:userTask id="employeeLeaveApply" name="员工提交请假申请">
         <bpmn:extensionElements>
           <activiti:taskListener event="create" class="com.lm.activiti7study.listener.MyFirstListener" />
         </bpmn:extensionElements>
         <bpmn:incoming>Flow_07i37nu</bpmn:incoming>
         <bpmn:outgoing>Flow_0xbl72x</bpmn:outgoing>
    </bpmn:userTask>
   ```

   注：event属性要先写，否则可能会的读不到值而导致监听失败

   ```java
   <bpmn:extensionElements>
     <activiti:taskListener event="create" class="com.lm.activiti7study.listener.MyFirstListener" />
   </bpmn:extensionElements>
   ```

4. 编写test类。无特殊参数，和前边Part3测试方法相同

# 五、流程变量

监听器中更改流程变量不能通过map.set方法。而要通过如：`delegateTask.setVariable(key,obj+"-lm123");//有效操作√`方式进行设置

## 5.1 运行时流程变量

### 5.1.1 全局流程变量

1. 流程变量的默认作用域是流程实例，**随着流程实例的完成而消失**。

2. 当一个流程变量作用域是流程实例时，成为`Global`变量。

   1. 变量名不许重复
   2. 设置相同变量名的值，会进行覆盖
   3. 使用过程中会记录在`act_ru_variable`中

3. 编写监听器代码

   ```java
   /**
    * 更改流程变量
    */
   
   //此类定义的是用户进程监听器
   public class MySecondListener implements TaskListener {
   
       /**
        * 监听器触发回调方法
        * @param delegateTask
        */
   
       @Override
       public void notify(DelegateTask delegateTask) {
           System.out.println("----->MySecondListener监听器执行了");
           if(EVENTNAME_CREATE.equals(delegateTask.getEventName())){
               //表示是Task的创建事件被触发了
               //获取流程变量
               Map<String, Object> variables = delegateTask.getVariables();
               Set<String> keySet = variables.keySet();
               for (String key : keySet) {
                   Object obj = variables.get(key);
                   System.out.println(key+"="+obj);
                   //获取之后修改流程变量
                   // variables.put(key,"lm123");//此方法是无效的
                   delegateTask.setVariable(key,obj+"-lm123");//有效操作√
               }
           }
       }
   }
   ```

4. 设计流程(编写xml文件) ，给指定的流程节点进行`值表达式配置（${}）`。如下以某流程节点为例

   ```java
   <bpmn:userTask id="employeeLeaveApply" name="员工提交请假申请" activiti:assignee="${assignee1}">
         <bpmn:extensionElements>
           <activiti:taskListener event="create" class="com.lm.activiti7study.listener.MyThirdListener"/>
         </bpmn:extensionElements>
         <bpmn:incoming>Flow_07i37nu</bpmn:incoming>
         <bpmn:outgoing>Flow_0xbl72x</bpmn:outgoing>
       </bpmn:userTask>
   ```

5. 编写test方法代码进行部署等测试（启动流程实例时需要通过map传入参数）

   - 测试方法通过`值表达式`进行传值一定是全局变量，不能传局部变量

     ```java
      /**
          * 设置全局流程变量
          */
         @Test
         public  void test2() {
             //获取processEngine对象
             ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
             RuntimeService runtimeService = processEngine.getRuntimeService();
             Map<String,Object> map = new HashMap<>();
             map.put("assignee1","assignee1张");
             map.put("assignee2","assignee2李");
             map.put("assignee3","assignee3王");
             map.put("as1","赵");
             map.put("as2","周");
             //传参 且此时会触发监听器中的修改办法
             //可以在act_ru_variable表中看到
             ProcessInstance processInstance = runtimeService.startProcessInstanceById("LeaveApply:3:7503",map);//模拟测试发起一个流程
     
             System.out.println("processInstance.getId() = " + processInstance.getId());
             System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
             System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
     
         }
     ```

   - 获取全局变量：`runtimeService.getVariableInstances("某实例的Execution_ID")`

     ```java
     /**
          *
          * 查询当前流程变量信息
          */
         @Test
         public void test4(){
             //获取processEngine对象
             ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
             RuntimeService runtimeService = processEngine.getRuntimeService();
     
             //查 某个流程实例 的变量信息 而不是某个流程定义下所有实例的全部流程变量
             //传参 执行时ID   ==》 查act_ru_task中的 Execution_ID
             Map<String, VariableInstance> variableInstances = runtimeService.getVariableInstances("10007");
             System.out.println("----->获取后输出当前流程实例变量");
             Set<String> keySet = variableInstances.keySet();
             for (String key : keySet) {
                 System.out.println(key+"="+variableInstances.get(key));
             }
         }
     ```

### 5.1.2 局部流程变量

1. 编写监听器代码

   ```java
   /**
    * 测试局部和历史流程变量
    *      更改流程变量 
    */
   //此类定义的是用户进程监听器
   public class MyThirdListener implements TaskListener {
   
       /**
        * 监听器触发回调方法
        * @param delegateTask
        */
   
       @Override
       public void notify(DelegateTask delegateTask) {
   
           System.out.println("----->MyThirdListener监听器执行了");
           if(EVENTNAME_CREATE.equals(delegateTask.getEventName())){
               //表示是Task的创建事件被触发了
               //获取流程变量
               Map<String, Object> variables = delegateTask.getVariables();
               Set<String> keySet = variables.keySet();
               for (String key : keySet) {
                   Object obj = variables.get(key);
                   System.out.println(key+"="+obj);
                   //获取之后修改流程变量
                //   delegateTask.setVariable(key,obj+"[]");//有效操作√
               }
           }
       }
   }
   ```

2. 设计流程(编写xml文件)：和全局变量配置文件相同，无变化。（即使用值表达式配置）

3. 编写test方法代码

   1. 设置必要的全局变量

      ```java
       /**
           * runtimeService 设置全局流程变量
           */
          @Test
          public  void test2() {
              //获取processEngine对象
              ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
              RuntimeService runtimeService = processEngine.getRuntimeService();
              //此处为设置全局变量
              Map<String,Object> map = new HashMap<>();
              map.put("assignee1","assignee1张");
              map.put("assignee2","assignee2李");
              map.put("assignee3","assignee3王");
      
              //通过runtimeService获取流程定义的一个流程实例对象    流程定义：查act_re_procdef表
              //使用值表达式需要传入全局变量
              ProcessInstance processInstance = runtimeService.startProcessInstanceById("LeaveApply:4:20003",map);//模拟测试发起一个流程
      
              System.out.println("processInstance.getId() = " + processInstance.getId());
              System.out.println("processInstance.getDeploymentId() = " + processInstance.getDeploymentId());
              System.out.println("processInstance.getDescription() = " + processInstance.getDescription());
      
          }
      ```

   2. 设置局部变量的两种方式如下。**局部变量，通过某网关分支后，局部变量会变得不同**

      ```java
          /**
           * runtimeService 设置局部变量
           */
          @Test
          public void test3(){
              ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
              RuntimeService runtimeService = processEngine.getRuntimeService();
      
              //传参  运行时ID及局部变量键值对形式
              runtimeService.setVariableLocal("22505","leaveId","123");
              runtimeService.setVariableLocal("22505","name","666");
          }
      
          /**
           * TaskService 设置局部变量
           */
          @Test
          public void test5(){
              ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
              TaskService taskService = processEngine.getTaskService();
              taskService.setVariableLocal("22508","taskServiceTest","6666666");
          }
      ```

      ​	区别：传入参数不同:`runtimeService`需要传入运行时ID（查`act_ru_task`表的`Execution_ID`）;`TaskService`需要传入任务ID(查`act_ru_task`表的`ID`）

   3. 读取局部变量（两种方法设置要用两种对应的方法读取，互相读取不到对方设置的局部变量。）

      ```java
      		/**
           * 读取局部变量
           */
          @Test
          public void test4(){
              ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
              RuntimeService runtimeService = processEngine.getRuntimeService();
              TaskService taskService = processEngine.getTaskService();
      
              //传参  运行时ID  查act_ru_task的Execution_ID
              Map<String, Object> variablesLocal = runtimeService.getVariablesLocal("22505");
              System.out.println("------->runtimeService获取当前流程实例的局部变量");
              Set<String> keySet = variablesLocal.keySet();
              for (String key : keySet) {
                  System.out.println(key+"="+variablesLocal.get(key));
              }
      
              //传参  任务ID  查act_ru_task的ID
              Map<String, Object> variablesLocal1 = taskService.getVariablesLocal("22508");
              System.out.println("------->taskService获取当前流程实例的局部变量");
              Set<String> keySet1 = variablesLocal1.keySet();
              for (String key : keySet1) {
                  System.out.println(key+"="+variablesLocal1.get(key));
              }
          }

## 5.2 历史流程变量

1. 流程文件不需要改变。使用值表达式分配方式

2. 编写test方法查询

   ```java
   		/**
        * 读取历史变量
        */
       @Test
       public void test7(){
           ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
           HistoryService historyService = processEngine.getHistoryService();
   
           //historyService.createHistoricVariableInstanceQuery()
           // 所有工作流的历史数据都要通过historyService实现查询
           //存在act_hi_varinst表中 不会被删除 会”永久“保存
           List<HistoricVariableInstance> list = historyService.createHistoricVariableInstanceQuery().list();
           System.out.println("------->historyService获取当前流程实例的历史流程变量");
           for (HistoricVariableInstance historicVariableInstance : list) {
               System.out.println("historicVariableInstance = " + historicVariableInstance);
           }
       }
   ```

# 六、身份服务

## 6.1 审批人



## 6.2 候选人



## 6.3 候选人组

# 七、网关

## 7.1 排他网关

1. 用于流程中进行条件判断，根据不同的条件选择不同的分支路径。只有满足条件的分支会被执行，其他分支会被忽略
2. 会选择第一个条件判断为true的分支进行
3. 如果会有多个分支满足条件，也只会选择第一个满足的，且不会再接着判断其他分支条件
4. 如果分支条件为空，则认为该顺序流为true
5. 流程设计代码（xml文件）
6. 测试代码中需要通过传参形式传入条件判断所需的值

## 7.2 并行网关

1. 用于将流程分成多个的分支，这些分支可以同时执行
2. 当所有分支都执行完毕后，流程会继续向下执行
3. 有并行网关就需要有join网关点
4. 数据库会保存一条主流程实例信息和一条或多条子流程信息
5. 并行网关后会有一个子流程实例生成多个子流程实例

## 7.3 包容网关

用于根据多个条件的组合情况选择分支路径，可以选择满足任意一个条件的分支执行，或者选择满足所有条件的分支执行。

## 7.4 事件网关

用于根据事件的触发选择分支，当指定给的事件触发时，流程会选择对应的分支执行。

# 八、 事件

捕获：当流程执行达到这个事件时，会等待直到触发器动作。

抛出：当流程执行达到这个事件时，会触发一个触发器。

## 8.1 定时器

- 可以在数据库表`act_ru_timer_job`中查询到定时器的实时信息（流程结束时定时器消亡，该表中的信息会随着定时器的消亡而删除）

- `timeDate`：指定一个具体的日期和时间。如：`2024-01-01T00:00:00`

- `timeCycle`：指定一个重复周期。如：`R3/PT1H`表示每间隔1小时触发一次共触发3次

- `timeDate`：指定一个持续时间。如：`PT2H30M`表示持续2小时30分钟

- 需要开启异步执行时才会使用定时器，而异步执行默认为`false`。需要手动在配置文件中修改。即xml文件默认配置或代码形式部署配置时候`asyncExecutorActivate`属性为`true`。保证异步执行任务

  ```ja
      <!--开启异步执行  测试定时器时部署操作和执行操作需要一起 否则部署完成之后定时器会消失-->
      <property name="asyncExecutorActivate" value="true"></property>

### 8.1.1 启动事件

- 使用时需要注意
  1. 子流程不能有定时器启动事件
  2. 定时器启动事件，在流程部署的同时就开始计时。不需要调用`startProcessInstanceByXXX`去启动一个实例，而是会在指定时间自动启动一个实例。调用`startProcessInstanceByXXX`时会在定时启动之外额外启动一个流程
  3. 当部署带有定时器的启动事件的流程的更新版本时，上一版本的定时器作业会被移除。（这是因为通常并不希望旧版本的流程仍然自动启动新的流程实例）

### 8.1.2 中间事件

1. 当我们完成前边的审批处理后，会启用定时器中间事件，完成后会接着执行后续的流程任务事件。
2. 定时器中间事件需要手动启动一个实例后才会触发
3. 如：“开始--->入库审批--->定时器中间事件（PT1M）--->出库审批--->结束”。则在`启动实例`后会进入`入库审批`，`入库审批`完成后，会进入`定时器`，`定时器`持续一分钟后，进入下一步`出库审批`任务，`出库审批`完成后会结束该工作流实例
4. 类似于`进程的sleep`操作

### 8.1.3 边界事件

1. 中断型边界事件
   1. 定时器的`cancelActivity`属性设置为false
   2. 定时器触发边界事件后会转入子流程事件，同时主流程会直接结束不再往后执行
2. 非中断型边界事件
   1. 定时器的`cancelActivity`属性设置为true
   2. 定时器触发边界事件后会转入子流程事件，勇士主流程不会结束，会等待（如：长时间未审批会触发发送通知信息的任务子流程，同时等待审批）
   3. 只有前边的审批完成后，才会转接下一个流程任务
3. 主要是使用`attachedToRef`属性为定时器设置依赖的事件（传入的值为：`依赖事件的ID`）（**必须要配置该属性，否则会编译报错**）

## 8.2 消息事件

需要先定义消息事件

### 8.2.1 启动事件

### 8.2.2 中间事件

### 8.2.3 边界事件

























































































































