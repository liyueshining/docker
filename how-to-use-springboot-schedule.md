# Springboot scheduled的使用
最近项目中需要做定时任务，发现springboot的定时任务使用起来简直方便之极。
## 开启schedule功能
首先需要在启动类中加上@EnableScheduling注解来开启定时任务。如下：
```java
@SpringBootApplication
@EnableScheduling
public class SpringbootPracticeApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootPracticeApplication.class, args);
	}

}
```

## 创建 定时任务
在方法前添加@Scheduled注解即可，如下：
```java
@Service
public class TaskScheduleService {
    private static final Logger logger = LoggerFactory.getLogger(TaskScheduleService.class);

    @Scheduled(cron = "0 0/1 * * * ?")
    public void scheduleTask() {
        logger.info("timer scheduled per minute");
    }

    @Scheduled(cron = "0 0 0/6 * * ?")
    public void scheduleJob() {
       logger.info("timer scheduler per six hours");
    }
}
```

## @Scheduled 定时任务的执行方式
### fixedDelay
指定两次任务执行的时间间隔(毫秒)，此时间间隔指的是，前一次任务结束与下一个任务开始的间隔。如：@Scheduled(fixedDelay = 5*1000 )，表示第一个任务结束后，过5秒后，开始第二个任务。
### fixedRate
指定两次任务执行的时间间隔(毫秒)，此时间间隔指的是，前一个任务开始与下一个任务开始的间隔。如：@Scheduled(fixedRate= 5*1000 )，表示第一个任务开始后(第一个任务执行时间小于5秒)，第一个任务开始后的第6秒，开始第二个任务。如果第一个任务执行时间大于5秒，第一个任务结束后，直接开始第二个任务。
### cron
使用表达是进行任务的执行，例如：@Scheduled(cron = "0/15 * * * * ? ")每隔15秒执行一次.
cron一般是六个或七个字段，分别是：
```
1. Seconds （秒） 
2. Minutes （分） 
3. Hours （时） 
4. Day (每月的第几天,day-of-month) 
5. Month （月） 
6. Day （每周的第几天,day-of-week） 
7. Year (年 可选字段)
```

每个字段的范围以及特殊字符
```
秒 ：范围:0－59 
分 ：范围:0－59
时 ：范围:0-23
天（月） ：范围:1-31,但要注意一些特别的月份2月份没有只能1-28，有些月份没有31
月 ：用0-11 或用字符串 “JAN, FEB, MAR, APR, MAY, JUN, JUL, AUG, SEP, OCT, NOV and DEC” 表示
天（周）：用1-7表示（1 ＝ 星期日）或用字符口串“SUN, MON, TUE, WED, THU, FRI and SAT”表示
年：范围:1970－2099
 
“/”：表示为“每”，如“0/10”表示每隔10分钟执行一次,“0”表示为从“0”分开始, “3/20”表示表示每隔20分钟执行一次
“?”：只用于月与周，表示不指定值
“L”：只用于月与周，5L用在月表示为每月的最后第五天天；1L用在周表示每周的最后一天；
“W”：:表示有效工作日(周一到周五),只能出现在day-of-month，系统将在离指定日期的最近的有效工作日触发事件。例如：在 DayofMonth使用5W，如果5日是星期六，则将在最近的工作日：星期五，即4日触发。如果5日是星期天，则在6日(周一)触发；如果5日在星期一到星期五中的一天，则就在5日触发。另外一点，W的最近寻找不会跨过月份 
“#”：用于确定每个月第几个星期几，只能出现在DayofMonth域。例如在4#2，表示某月的第二个星期三。
“*” 代表整个时间段。
 
注意：每个元素可以是一个值(如6),一个连续区间(9-12),一个间隔时间(8-18/4)(/表示每隔4小时),一个列表(1,3,5),通配符。由于"月份中的日期"和"星期中的日期"这两个元素互斥的,必须要对其中一个设置‘?’
```

## 为定时任务创建多线程池
spring 的定时任务默认是单线程的，如果有多个定时任务的话，需要开发者自定义TaskScheduer,如下：
在Configuration类中重新定义TaskScheduler即可
```java
@Configuration
public class SpringBootPracticeConfiguration {

    @Bean
    public TaskScheduler scheduledExecutorService() {
        //used for spring scheduler
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(8);
        scheduler.setThreadNamePrefix("scheduled-thread-");
        return scheduler;
    }
}
```

## 犯过的错
项目中需要定期去更新缓存，想要达到的目的是每6个小时执行一次，之前设置的cron如下：
```
@Scheduled(cron = "* * 0/6 * * ?")
```
发现该表达式会在0点的00分到59分一直执行，问题就出现在前面两个*号，*代表整个时间段。
如果只需要执行一次的话，cron表达式应该是下面这样的。
```
@Scheduled(cron = "0 0 0/6 * * ?")
```
---
