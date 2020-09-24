---
title: SpringBoot集成Quartz定时任务
reward: false
copyright: false
tags:
  - Quartz
categories:
  - SpringBoot
  - 学习
date: 2020-09-24 10:38:00
---

一、#### Quartz依赖：

```xml
<!-- Quartz定时任务 -->
     <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-quartz</artifactId>
     </dependency>
```

二、#### 创建Job类（需要定时做什么）：

```java
public class MyJob extends QuartzJobBean {

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {

        log.info("任务开始执行了");
		//TODO do something
        log.info("本次任务执行结束了");
    }

}
```

三、#### 任务工厂(通过工厂注入，才可以在Job里使用@AutoWried)：

```java
@Component
public class MyJobFactory extends AdaptableJobFactory {
    @Autowired
    private AutowireCapableBeanFactory capableBeanFactory;

    @Override
    protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
        // 调用父类的方法
        Object jobInstance = super.createJobInstance(bundle);
        // 进行注入
        capableBeanFactory.autowireBean(jobInstance);
        return jobInstance;
    }
}
```

四、#### 定时任务初始化管理：

```java
@Configuration
public class PushServiceQuartzManager {

    //TODO    每天早上8点到9点, 14点到15点 每隔20分钟执行一次
    //    public  static final String DAY_8_9_CROM = "0 0/20 8,14 * * ? ";

    private static final String DAY_8_9_CROM = "0 0/1 8-18 * * ? "; //测试用

    @Autowired
    JobDetailFactoryBean jobDetailFactoryBean;
    @Autowired
    CronTriggerFactoryBean cronTriggerFactoryBean;
    @Autowired
    PushJobFactory pushJobFactory;
    /**
     *  1.创建job对象  做什么
     */
    @Bean
    public JobDetailFactoryBean jobDetailFactoryBean(){
        JobDetailFactoryBean factory = new JobDetailFactoryBean();
        //关联自己的job类
        factory.setJobClass(MyJob.class);
        return factory;
    }
    /**
     *  2.创建触发器 Cron Trigger
     */
    @Bean
    public CronTriggerFactoryBean cronTriggerFactoryBean(){
        CronTriggerFactoryBean factory = new CronTriggerFactoryBean();
        //关联JobDetail
        factory.setJobDetail(jobDetailFactoryBean.getObject());
        //                             秒   分钟 小时 天 月 星期
        factory.setCronExpression(DAY_8_9_CROM);
        //表示每年每月每天的11点34分的10,15,20秒都会执行一次
        return factory;
    }

    /**
     *  3.创建Scheduler对象
     */
    @Bean
    public SchedulerFactoryBean schedulerFactoryBean() {
        SchedulerFactoryBean schedulerFactoryBean = new SchedulerFactoryBean();
        //通过工厂注入，才可以在Job里使用@AutoWried
        schedulerFactoryBean.setJobFactory(MyJobFactory);
        schedulerFactoryBean.setTriggers(cronTriggerFactoryBean.getObject());
//        System.out.println("myJobFactory:"+pushJobFactory);
        return schedulerFactoryBean;
    }
 ｝
```

五、#### 设置SpringBoot启动就开始执行定时任务：

```java
@SpringBootApplication
@EnableAutoConfiguration
@EnableScheduling //加上这个就可以在项目启动后就执行Quartz定时任务了
public class SpringBootApplication {
    public static void main(String[] args) throws UnknownHostException {
		SpringApplication.run(SpringBootApplication.class, args);
	｝

｝
```