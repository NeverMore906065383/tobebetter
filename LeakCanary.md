## LeakCanary  
Appliccatiojn中监听Activity生命周期管理  

ondestory中加入虚引用  

定时查询释放情况，  

如果失败，主动发起gc  
   
然后查询，如果仍在，则判定为泄露。  

部分无法判定的情况：  
静态引用  

