## register  

```register```完成注册统计 ```subscriberByEventType```，```typeBySubscriber``` 两个HashMap分别保存注册信息和注册对相关 （EventType，Subscriber）,其中也用到了缓存的方式。
```post```发送消息， stricky 粘性消息处理实际是发生在对象注册接受事件时。  
ThreadMode类型：
- posting  直接处理
- main 在主线程处理，如果当前接受信息线程不是主线程，则使用Handler 完成主线程消息切换
- main_order   无论当前是什么线程，发送给主线程，除非mainposter为null，则在当前线程处理
- background 切换到子线程
- asyn 无论当前是什么线程，切换到子线程。  

通过 ```invokeMethod ``` 反射通用相应方法  

处理粘性事件就是在 EventBus 注册时，遍历stickyEvents，如果当前要注册的事件订阅方法是粘性的，并且该方法接收的事件类型和stickyEvents中某个事件类型相同或者是其父类，则取出stickyEvents中对应事件类型的具体事件，做进一步处理
 
通过subscriber index的方式来配置 降低反射invokeSubscriber 的性能损耗  

Subscriber Index 的核心就是项目编译时使用注解处理器生成保存事件订阅方法信息的索引类，然后项目运行时将索引类实例设置到 EventBus 中。
