## LeakCanary  
```LeakCanary.install() 的处理：```  
通过intall 中来确认是否监听Act或Fragement    

通过Application监听Activity生命周期  

创建 AndroidRefWatcherBuilder 构建 LeakCanary 所需参数，如提供 DisplayLeakService 内存泄漏通知服务，排除Android系统源码出现的内存泄漏

创建 RefWatcher，关联app包名生成 LeakCanary 应用，通过 registerXxxLifecycleCallbacks() 监听生命周期转发给 RefWatcher 处理

```RefWatcher```中
```
 Retryable.Result ensureGone(final KeyedWeakReference reference, final long watchStartNanoTime) {
    long gcStartNanoTime = System.nanoTime();
    long watchDurationMs = NANOSECONDS.toMillis(gcStartNanoTime - watchStartNanoTime);

    removeWeaklyReachableReferences();

    if (debuggerControl.isDebuggerAttached()) {
      // The debugger can create false leaks.
      return RETRY;
    }
    //第一次判断引用是否释放
    if (gone(reference)) {
      return DONE;
    }
    //如果没有释放主动调用一次gc，避免此前还没出发gc
    gcTrigger.runGc();
    removeWeaklyReachableReferences();
    //如果引用还是没有释放则确认其泄漏
    if (!gone(reference)) {
      long startDumpHeap = System.nanoTime();
      long gcDurationMs = NANOSECONDS.toMillis(startDumpHeap - gcStartNanoTime);

      File heapDumpFile = heapDumper.dumpHeap();
      if (heapDumpFile == RETRY_LATER) {
        // Could not dump the heap.
        return RETRY;
      }
      long heapDumpDurationMs = NANOSECONDS.toMillis(System.nanoTime() - startDumpHeap);

      HeapDump heapDump = heapDumpBuilder.heapDumpFile(heapDumpFile).referenceKey(reference.key)
          .referenceName(reference.name)
          .watchDurationMs(watchDurationMs)
          .gcDurationMs(gcDurationMs)
          .heapDumpDurationMs(heapDumpDurationMs)
          .build();

      heapdumpListener.analyze(heapDump);
    }
    return DONE;
  }
```
 HeapDumpListener.analyze() 其实就是启动了一个前台服务在其他进程分析内存泄漏的引用路径。     
### 总结
简单来说：Appliccatiojn中监听Activity生命周期管理  
ondestory中加入弱引用，查询引用释放情况，  
如果失败，主动触发一次gc并清理缓存的引用。  
再次查询，如果仍在，则判定为泄露之后通过启动一个前台服务，在其他进程中对hprof进行分析。

 
 
 
 

