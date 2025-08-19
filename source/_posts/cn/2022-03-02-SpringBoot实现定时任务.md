---
layout: post
title: 'SpringBoot实现定时任务'
date: 2022-03-02
author: Futari
cover: 'http://on2171g4d.bkt.clouddn.com/jekyll-banner.png'
tags: Spring quarrz task
---

#### 两种方式实现SpringBoot定时任务
##### 整合quartz：

首先使用util包下自带的Timmer类

~~~ java
public class TimerTaskApp {
    public static void main(String[] args) {
        Timer timer = new Timer();
        TimerTask task = new TimerTask() {
            @Override
            public void run() {
                System.out.println("timer task run...");
            }
        };
        timer.schedule(task,0,2000);
    }
}

~~~



1.导入依赖后定义类继承QuartzJobBean，因为QuartzJobBean是抽象类，所以需要实现它里面的抽象方法

~~~ java
public class MyQuartz extends QuartzJobBean {
    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        System.out.println("quartz task run...");
    }
}

~~~



2.定义工作明细和触发器trigger并绑定对应关系

~~~ java
@Configuration
public class QuartzConfig {

    @Bean
    public JobDetail printJobDetail(){
        //绑定具体的工作
        return JobBuilder
                .newJob(MyQuartz.class)
                .storeDurably()
                .build();
    }

    @Bean
    public Trigger printJobTrigger(){
        ScheduleBuilder schedBuilder = CronScheduleBuilder.cronSchedule("0/5 * * * * ?");
        //绑定对应的工作明细
        return TriggerBuilder.newTrigger().forJob(printJobDetail()).withSchedule(schedBuilder).build();
    }

}
~~~



#### 整合Spring Task（推荐）：

1.在引导类上开启@EnableScheduling注解

2.在具体Service方法上@Scheduled并设置注解参数cron表达式来设置定时任务

###### 定时任务的配置设置在yml文件中设置，如下：

~~~ yml
spring:
	task:
		scheduling:
		pool:
			size:1 #任务调度线程池大小
		#定时任务本质上是调用新线程执行任务，所以可以设置thread前缀
		thread-name-prefix: ssm_
#		设置定时任务线程结束时的参数
		shutdown:
			await-termination: flase
			#关闭时是否等待所有任务完成
~~~



cron表达式格式：秒 分 时 日 月 年 星期，一般星期和日容易起冲突，所以一般规定日之后，就将星期设置为？，代表不设置看情况匹配，*通配符则代表所有，例如0 0 8 * * ？代表每天8点整指定一次

~~~ java
@Component
public class MyBean {

    @Scheduled(cron = "0/1 * * * * ?")
    public void print(){
        System.out.println(Thread.currentThread().getName()+" :spring task run...");
    }

}

~~~





###### 扩展：链式编程

链式编程可以使得代码可读性高，链式编程的原理就是返回一个this对象，就是返回本身，达到链式效果，可以直接return this;

~~~java
例如：StudentBean studentBean = StudentBean.builder().name("ly").age(11).build();
~~~

SuudentBean类，实体类没有return this;

~~~ JAVA
public class StudentBean {
	private String name;
	private int age;
	
	public String getName() {
		return name;
	}
 
	public StudentBean setName(String name) {
		this.name = name;
		return this;
	}
 
	public int getAge() {
		return age;
	}
 
	public StudentBean setAge(int age) {
		this.age = age;
		return this;
	}
}
~~~

##### CRON表达式

cron表达式分为七个域，之间使用空格分隔。其中最后一个域（年）可以为空。每个域都有自己允许的值和一些特殊字符构成。使用这些特殊字符可以使我们定义的表达式更加灵活。

下面是对这些特殊字符的介绍：

逗号（,）：指定一个值列表，例如使用在月域上1,4,5,7表示1月、4月、5月和7月

横杠（-）：指定一个范围，例如在时域上3-6表示3点到6点（即3点、4点、5点、6点）

星号（*）：表示这个域上包含所有合法的值。例如，在月份域上使用星号意味着每个月都会触发

斜线（/）：表示递增，例如使用在秒域上0/15表示每15秒

问号（?）：只能用在日和周域上，但是不能在这两个域上同时使用。表示不指定

井号（#）：只能使用在周域上，用于指定月份中的第几周的哪一天，例如6#3，意思是某月的第三个周五 (6=星期五，3意味着月份中的第三周)

L：某域上允许的最后一个值。只能使用在日和周域上。当用在日域上，表示的是在月域上指定的月份的最后一天。用于周域上时，表示周的最后一天，就是星期六

W：W 字符代表着工作日 (星期一到星期五)，只能用在日域上，它用来指定离指定日的最近的一个工作日

