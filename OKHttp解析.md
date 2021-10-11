# OKHttp原理解析

- Interface 接口层：接受网络访问请求
- Protocol 协议层：处理协议逻辑，解析拼接
- Connection 连接层：管理网络连接，发送新的请求，接收服务器访问
- Cache 缓存层：管理本地缓存
- I/O 实际数据读写实现
- Interceptor 拦截器层：拦截网络访问，插入拦截逻辑

### 1.拦截器Interceptor
getResponseWithInterceptorChain():
```
 Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(retryAndFollowUpInterceptor);
    //BridgeInterceptor
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(
        interceptors, null, null, null, 0, originalRequest);
    return chain.proceed(originalRequest);
  }
```
拦截器中process 并，将拦截器链指向下一环。  

retryAndFollowUpInterceptor 重试和跟踪拦截器：
- 网络请求失败后重试
- 服务器返回当前请求需要进行重定向时直接发起新的请求，并在条件允许情况下复用当前连接

BridgeInterceptor:解码，GZIP解压缩    

CacheInterceptor 缓存拦截器:
- 请求有符合要求的cache则直接返回cache
- 服务器返回的请求内容改变时更新当前cache
- 如果当前cache失效，删除

ConnectInterceptor  连接拦截器 : 
 为当前请求找到合适的连接，可以复用或者重新创建，返回的连接由连接池决定  
 
CallServerInterceptor:负责向服务器发起真正的请求，并在接收到服务器返回后读取响应返回。



[拦截器链调用request、response返回图](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/e67029972070a7dd84206023b179dbd1.png)

### 2.任务队列 
dispatcher  三个队列
readyAsyncCalls：待执行异步任务队列
runningAsyncCalls：运行中异步任务队列
runningSyncCalls：运行中同步任务队列
executorService：任务队列线程池：
以上就是整个任务队列的实现细节，总结起来有以下几个特点：
OkHttp采用Dispatcher技术，类似于Nginx，与线程池配合实现了高并发，低阻塞的运行
Okhttp采用Deque作为缓存，按照入队的顺序先进先出
OkHttp最出彩的地方就是在try/finally中调用了finished函数，可以主动控制等待队列的移动，而不是采用锁或者wait/notify，极大减少了编码复杂性


### 3.缓存策略 
> https://developer.aliyun.com/article/78102  

总结起来DiskLruCache主要有以下几个特点：
通过LinkedHashMap实现LRU替换
通过本地维护Cache操作日志保证Cache原子性与可用性，同时为防止日志过分膨胀定时执行日志精简
每一个Cache项对应两个状态副本：DIRTY,CLEAN。CLEAN表示当前可用状态Cache，外部访问到的cache快照均为CLEAN状态；DIRTY为更新态Cache。由于更新和创建都只操作DIRTY状态副本，实现了Cache的读写分离
每一个Cache项有四个文件，两个状态（DIRTY,CLEAN）,每个状态对应两个文件：一个文件存储Cache meta数据，一个文件存储Cache内容数据

### 4.多路复用
> https://developer.aliyun.com/article/78101  
> OkHttp的连接池通过计数+标记清理的机制来管理连接池，使得无用连接可以被会回收，并保持多个健康的keep-alive连接。这也是OkHttp的连接池能保持高效的关键原因。





