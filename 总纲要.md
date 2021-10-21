
# 编程
## Java部分
#### JVM 内存结构
> https://blog.csdn.net/rongtaoup/article/details/89142396
> https://www.cnblogs.com/czwbig/p/11127124.html
> https://blog.csdn.net/u011153817/article/details/105854582
#### JVM、DVM(Dalvik VM)和ART虚拟机的区别
> https://www.cnblogs.com/yuanqiangfei/p/12252521.html  



dvm执行的是.dex格式文件，jvm执行的是.class文件。  
dvm是基于寄存器的虚拟机 而jvm执行是基于虚拟栈的虚拟机。  
    dvm速度快  
    指令数小
-  Dalvik的垃圾回收策略默认是标记擦除回收算法
    
#### 多线程及线程池使用理解  
#### 线程安全
- **谈谈对Synchronized关键字，类锁，方法锁，重入锁的理解**  

**java的对象锁和类锁：** 
java的对象锁和类锁在锁的概念上基本上和内置锁是一致的，但是，两个锁实际是有很大的区别的，对象锁是用于对象实例方法，或者一个对象实例上的，类锁是用于类的静态方法或者一个类的class对象上的。我们知道，类的对象实例可以有很多个，但是每个类只有一个class对象，所以不同对象实例的对象锁是互不干扰的，但是每个类只有一个类锁。但是有一点必须注意的是，其实类锁只是一个概念上的东西，**并不是真实存在的**，它只是用来帮助我们理解锁定实例方法和静态方法的区别的  

- **synchronized 和volatile 关键字的区别**
1. volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
1. volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的
1. volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性
1. volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
1. volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化
    
#### 线程池的使用
#### JVM内存结构  
- runtime+引擎+？
#### 同步并发/锁  

### 类加载机制
java和android 使用的虚拟机不同，类加载机制略有差异  
```
DexClassLoader.loadClass ->
    DexClassLoader.findClass ->
        BaseDexClassLoader.findClass ->
            DexPathList.findClass ->
                DexFile.loadClassBinaryName ->
                    DexFile.defineClass
```

 
-------------------------
## Android部分

###  App启动流程
### 《Android开发艺术探索》
#### Activity 生命周期 和启动模式  
  ``` onstart()/onstop() ``` 从可见性上来说是配对的  
  ``` onresume()/onpause() ``` 从Activity是否在前台来说是配对的  
  A界面 onpause 执行完成后才开始下一个界面的启动，因此，尽量避免在onpause中多余耗时操作。      
    
```
    启动流程整理：
    
```  

LaunchMode的使用
- standard 标准模式
- singleTop 栈顶复用模式
- singleTask 栈内复用模式
- singleInstance 单例模式 
  
#### IPC机制
```serializable和parcable 差异  ```  
serializable 序列化方式适用于java，文件存储和网络传输  
parcable 内存序列化方式，适用于android 

#### Binder 
这里只是简单描述Binder 的上层使用原理，不做深入   
IPC 过程  
客户端通过proxy(IBinder remote) 代理调用remote.transact过程调用远程方法  
在服务端Stub的onTranact()被调用进行反馈，通过协议文件中已经固定的方法id进行区分  
Paracel 序列化包装的data，及返回的reply，作为RPC过程数据传输  
Binder 的死亡代理  
RemoteCallbackList  
AIDL中如歌使用权限验证功能

#### 其他IPC的类型  
- 文件共享  Android基于Linux实现支持同步读写文件。sp文件同样支持同步读写，但是其有内存缓存操作，因此也存在多进程数据丢失等问题
- contentProvider   底层实现是Binder
- Messenger,其底层现实是AIDL
- AIDL 配合service的使用方式demo编写及理解
- Socket
    - TCP : 面向连接的协议，三次握手，稳定  
    - UDP : 无连接的，不稳定，效率更好  


#### Binder连接池  
解决多个不同模块需要进行进程间通讯问题  
BinderPool(静态内部类IBinderImpl进行 Binder对象case) 中开启Service获得IBinderPool对象进行AIDL Binder对象查询

#### View的事件体系  
1. View的滑动  
- scrollTo和scrollBy斗志改变View的内容而不改变View本身在布局中的位置
- 动画方式
- LayoutParams方式  
2. View的事件分发机制  
  
传递路线Activity->window(phoneWindow)->DectorView.

- onInterceptTouchEvent
- onTouchEvent
- dispathTouchEvent
- requestDisallowIntercept

触摸系列动作 肯定会触发dispatch，但不一定会触发onIntercept ，mFirstTouchTarget及DISSALLOW标记来，首次处理后被标记为非null，后续系列事件都不在触发onIntercept，
3. View的滑动冲突 
  
三种滑动冲突，
- 外部拦截发法，重写父容器的 ```onIntercaptTouchEvent() ```
- 内部拦截发，子元素重写dispatchTouchEvent及使用 ```requestDisallowInterceptTouchEvent()```.

### View的工作原理
Activity创建后会将DecorView添加到Window中，并将ViewRootImpl对象和DecorView建立关联
  
**理解MeasuereSpec**  

一个32位的int 高2位代表SpecMode，低30位代表SpecSize  
三种SpecMode：
- UNSPECIFIED 
- EXACTLY  match_parent
- AT_MOST  wrap_content  
  
**view 的工作流程**  
measure,layout,draw  

#### ~~理解RemoteView~~
#### Android的动画深入分析  
view动画：xml定义动画，四种变换：平移，旋转，缩放，透明  
帧动画：顺序播放一组预先定义好的图片  
属性动画：  
动画差值器： 计算随时间变化变化百分比
动画估值器：根据当前变化百分比计算变化值  
#### 理解Window和WindowManager 
**window的创建过程：**  
创建phoneWindow 创建DecorView 添加view

Toast的window 创建过程比较复杂  
显示隐藏 为IPC过程
其过程包含binder池数据交换，因此需要Looper调度  


### 四大组件
-----------------
#### Activity 启动流程流程，生命周期描
```
 instrumentation.startactivity ->  
 (AMS)activitymanagerNative.getdefault.startactivity() ->   
 [activityStackSupervisor] stack.startActivityMayWait() ->
 [activityStackSupervisor] stack.startactivityLocked() -> 
 [activityStackSupervisor] stack.startActivityUncheckLocked ->  
 [activityStack ] stack.resumeTopActivityLocked ->  
 [activityStack ] stack.resumeTopActivityInnerLocked ->  
 [activityStackSupervisor] startSpeficActivityLocked ->  
 realstartactivityLocked ->   
 (IApplicationThread) app.thread.scheduleLaunchActivity ->  
 Handler H.performLaunchActivity().
 ```  
 > http://gityuan.com/images/activity/start_activity.jpg  
 
 最后一个流程中主要工作如下：
 1. 通过ActivityClientRecord 获取activity信息
 2. 通过instrumentation的newActivity()使用类加载器方式创建activty对象
 3. 通过loadApk的makeApplication()创建Application对象
 4. 创建contextImpl对象并通过Activity的attach方法来完成初始化
 5. 调用Activity 的onCreate方法
 
 
#### Service 的类型
#### BroadcastReciever
#### ContentProvider 

#### Android 的消息机制
ThreadLocal  多线程各自维护数据
Messagequeue next方法无限循环，如果没有消息则阻塞
Looper  loop方法中的message.next 为阻塞方法，返回null时才能退出循环
Handler  dispatchMessage完成消息调度

#### Android 的线程和线程池  
线程池的优点：  
1. 重用线程，避免线程的频繁创建销毁带来的性能开销
2. 控制最大并发数，避免大量线程之间因互抢占系统资源而导致的阻塞现象
3. 能够对线程执行开始时间，执行间隔简单管理
4. 有效控制线程数量 
线程池的分类：  
FixedThreadPool 定长线程池  
CachedThreadPool 缓存线程池  
SchduledThreadPool 定时线程池  
SingleThreadExcutor 单例线程池  

#### Bitmap的加载和Cache  
采样率  
Lrucache算法  DiskLrucache 算法
#### 综合技术  
动态加载技术  
1. 资源访问
2. activity 生命周期给管理
3. classloader的管理  
参考任玉刚的DL插件化实现自己的插件，使用框架，自己实现  
类加载机制  
ClassLoader BaseDexClassLoader
PathClassLoader(用来加载系统类和应用类)   
DexClassLoader(用来加载jar、apk、dex文件.加载jar、apk也是最终抽取里面的Dex文件进行加载.)

#### JNI和 NDK编程
#### Android 的性能优化  
1. 布局优化，减少层级
2. 绘制优化
3. 内存泄漏优化（静态变量、单实例，属性动画）
4. 响应速度优化和anr日志分析
5. Listview和bitmap优化
6. 线程优化，使用线程池
 
### 权限管理

### 文件系统

### Style和Theme

### 绘图机制
> 

### oom 内存优化 
### ANR 问题 
  > https://mp.weixin.qq.com/s/ApNSEWxQdM19QoCNijagtg  
  
#### MVC MVP MVVM
#### 自定义控件
#### Android的优化
- apk打包优化，及签名方式
    - apk的构成：
    - 优化方式，混淆及压缩,.9文件等使用
- 性能优化
    - 渲染优化
    - 内存优化

### 第三方库源码理解学习
#### Eventbus
#### Retrofit
#### Rxjava
### 测试

####  泛型中extends和super的区别
- <? extends T>限定参数类型的上界，参数类型必须是T或T的子类型，但对于List<? extends T>，不能通过add()来加入元素，因为不知道<? extends T>是T的哪一种子类；

- <? super T>限定参数类型的下界，参数类型必须是T或T的父类型，不能能过get()获取元素，因为不知道哪个超类；
- ? 和 T在使用上的区别

## andorid 系统架构
> https://blog.csdn.net/Gityuan/article/details/50815403  

**binder作为android 的ipc 方式的原因 ： ** 

1. 相比较其他方式只需要进行一次数据拷贝。  
2. 从稳定性上而言 基于C/S 架构
3. 从验证安全性上
4. 从语言上考量,基于C/S 架构  java 面向对象思想  




## 设计模式  
> https://www.jianshu.com/nb/10186551
#### 单例模式  
饿汉模式、饱汉模式    
特征，volite关键字修饰的静态对象引用，双重校验锁。  

#### 策略模式  
策略模式是对算法的包装，是把调用算法的责任（行为）和算法本身（行为实现）分割开来，委派给不同的对象管理  

#### 观察者模式  
又分为拉模型和推模型，相较而言，推模型直接返回需要数据，而拉模型只提醒更新，具体数据按需自己到主题中获取。

#### 模板模式  
模板方法模式广泛应用于框架设计中，以确保通过父类来控制处理流程的逻辑顺序（如框架的初始化，测试流程的设置等）。
#### 工厂模式  
简单工厂（登录管理），工厂方法（客户向商店购买灯泡），抽象工厂（客户分别向不同种类的设备提供商定制不同品牌的电子设备）

#### 适配器模式  
分为对象适配器，类适配器  
类适配：使用继承的方式，实现接口功能。 
对象适配：使用委派的方式，由一个对象实现接口功能，并进行同功能委派。
#### 状态机模式  
状态机管理状态对象，状态对象内部持有状态机引用完成状态切换
#### 代理模式  
静态代理：委托者持有代理对象完成行为，接口隔离

动态代理：
#### 装饰者模式  
顶层接口，子类实现，对实现了对象进行包装（因此需要子类进行对象传递）。  
分为透明，半透明两种，如果和顶级接口定义不完全一致则为半透明模式。  
和适配器模式的差别在于：适配器模式，改变接口，不改变功能。装饰者模式，不改变接口，增强功能。

#### 责任链模式  
分发和处理，持有后续责任者引用

## 第三方源码分析
Eventbus ：> https://www.jianshu.com/p/d9516884dbd4 
okhttp ： > https://developer.aliyun.com/article/78105


## 数据结构和算法
### 数据结构
#### set，list，map，queue，stack
### 算法


## **kotlin**   
#### 协程   

## 其他作者
  
赵梵分享的 
>  https://www.mubucm.com/doc/3VL42VTB9FH

## 相关网站
> https://www.androiddevtools.cn/

## 参考资料
> 老罗的博客：https://blog.csdn.net/luoshengyang/article/details/8923485
> java设计模式：https://www.jianshu.com/nb/10186551

## 面试题  
> 文档：2020.5.21.note
链接：http://note.youdao.com/noteshare?id=226b2a3a1f098e2b0347a2ce2310ea34&sub=WEBcad798f70bd63f1ebcf4db2fa1f0500e
