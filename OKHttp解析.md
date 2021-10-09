# OKHttp原理解析

- Interface 接口层：接受网络访问请求
- Protocol 协议层：处理协议逻辑，解析拼接
- Connection 连接层：管理网络连接，发送新的请求，接收服务器访问
- Cache 缓存层：管理本地缓存
- I/O 实际数据读写实现
- Interceptor 拦截器层：拦截网络访问，插入拦截逻辑

