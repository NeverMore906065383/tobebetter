涉及到两个脉络：  
1. 整个应用的启动  
2. 一个新界面被加载

整个应用的启动
----------------
init()
zygote()   serviceManger()
system_server---- AMS/PMS/...

Launcher--- startActivity
Instrumentation()   ActivityThread

https://blog.csdn.net/zhaokaiqiang1992/article/details/50265221?spm=1001.2014.3001.5501
**Activity 启动引导:**  

> https://www.kancloud.cn/digest/androidframeworks/127782
**详细代码梳理:** 
> https://blog.csdn.net/kai_zone/article/details/81530126?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-2.no_search_link


## 总结
从源码细节上上非常多，主要的流程是梳理
AMS 开始发起startactvity请求，startActivityAsUser
AtivityStackSupervisor中开始处理，
-> StartAvtivityMayWait() 完成启动Avtivity信息的获取
—> startActvityLocked(); 


activity 启动流程图（宏观）
> https://img-blog.csdn.net/20180809100656565?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2thaV96b25l/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70

activity 启动  AMS内部流程图（微观）
> http://gityuan.com/images/activity/start_activity.jpg
