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


### 3.缓存策略 


### 4.多路复用  





