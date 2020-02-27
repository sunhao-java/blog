---
title: XXL-JOB几个关键线程、队列的分析
tags: [Java,xxl-job]
date: 2020-02-28
categories: 后端
---

# XXL-JOB几个关键线程、队列的分析

> XXL-JOB在初始化时，会执行一系列的方法，这些方法启动了一系列的线程，还创建了一些队列，这些线程、队列对于整个任务调度来说都是至关重要的，现在就来分析一下这些线程、队列。

<!-- more -->

## 调度中心的初始化
1. 调度中心启动时，会先执行 `com.xxl.job.admin.core.conf.XxlJobAdminConfig#afterPropertiesSet` 这个方法（因为这个类是实现 `org.springframework.beans.factory.InitializingBean` 接口的）
2. 这个方法的代码
```java
@Override
public void afterPropertiesSet() throws Exception {
    adminConfig = this;

    xxlJobScheduler = new XxlJobScheduler();
    xxlJobScheduler.init();
}
```

3. 关键在于执行了， `xxlJobScheduler.init()` 。再来看看这个方法的内容（通过添加一些注释）：
```java
public void init() throws Exception {
    // 初始化i18n信息
    initI18n();

    // 调度中心监听执行器的注册
    // 这里主要是监听自动注册的执行器，并且实时更新其状态
    // 线程分析见4
    JobRegistryMonitorHelper.getInstance().start();

    // 调度中心监控失败的调度
    // 首先进行重试，重试的次数为页面配置，每次减1
    // 发送失败告警邮件
    JobFailMonitorHelper.getInstance().start();

    // 调度中心调度监控
    // 源码分析见5
    JobTriggerPoolHelper.toStart();

    // 调度中心的日志整理
    JobLogReportHelper.getInstance().start();

    // 调度线程
    // 用于扫描即将要执行的任务（当前时间+5000毫秒）
    // 源码分析可见https://www.cnblogs.com/jiangyang/p/11576931.html
    JobScheduleHelper.getInstance().start();

    logger.info(">>>>>>>>> init xxl-job admin success.");
}
```

4. 调度中心监听执行器的注册
```java
public void start(){
    // 创建一个线程，该线程用来监听库中新增的自动注册的执行器
    registryThread = new Thread(new Runnable() {
        @Override
        public void run() {
            // 该线程中运行着一个while死循环
            // 当且仅当调用stop方法，将toStop设置为true，死循环结束，也就是监听结束
            // 基本上每个监听线程都是在这么做的
            while (!toStop) {
                // ...........
                // 这里设置线程休眠30s，意味着30s检测一次执行器的状态
                try {
                    TimeUnit.SECONDS.sleep(RegistryConfig.BEAT_TIMEOUT);
                } catch (InterruptedException e) {
                    if (!toStop) {
                        logger.error(">>>>>>>>>>> xxl-job, job registry monitor thread error:{}", e);
                    }
                }
            }
            logger.info(">>>>>>>>>>> xxl-job, job registry monitor thread stop");
        }
    });
    // 设置线程相关参数
    registryThread.setDaemon(true);
    registryThread.setName("xxl-job, admin JobRegistryMonitorHelper");
    registryThread.start();
}
```

5. 调度中心调度监控
这里启动了两个线程池： `fastTriggerPool` 和 `slowTriggerPool` ，当一个job的执行时间超过500毫秒时，将会被放入 `slowTriggerPool` 线程池来运行
