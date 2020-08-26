---
title: ShutdownHook-优雅关闭
date: 2017-12-14 11:58:37
tags:
---

    需求：
    1.在应用启动的时候将应用的信息注册到zk。
    （下游会判断zk中是否存在应用信息，需要等待应用完全加载成功再写入到zk）
    2.应用完全关闭之前将应用信息从zk删除。
    （如果下游判断zk在应用关闭之后还存在，仍然会请求，但不会有响应，所以需要在应用完全关闭之前将zk中应用的信息摘除）
    
    解决：
    在spring的环境下
    1.
    注册的服务类实现ApplicationListener，重写onApplicationEvent，Thead.sleep(waitTime);
    2.两种方法
        a.使用tomcat的listener，但是需要在每个应用的web.xml中都添加配置，耦合性太强
        b.使用shutdownHook
            JDK提供了Java.Runtime.addShutdownHook(Thread hook)方法，可以注册一个JVM关闭的钩子，这个钩子可以在一下几种场景中被调用：
            程序正常退出
            使用System.exit()
            终端使用Ctrl+C触发的中断
            系统关闭
            OutOfMemory宕机
            使用Kill pid命令干掉进程（注：在使用kill -9 pid时，是不会被调用的）
            
            ApplicationShutdownHooks里有一个IdentityHashMap存放钩子线程的map，
            其初始化的时候会往Shutdown的hooks线程组放一个执行IdentityHashMap钩子map的线程。
            当JVM关闭的时候会执行Shutdown的exit，触发Shutdown的hooks，触发钩子线程组。
    
    
    code:
    public void onApplicationEvent(ApplicationEvent event) {
            int delay = 5000;
            Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                unRegister();
            }, "unRegisterThread"));
            new Thread(() -> {
                try {
                    Thread.sleep(delay);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                register();
            }, "registerDelayThread").start();
    }
    
