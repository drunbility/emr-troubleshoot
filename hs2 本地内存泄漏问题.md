### 问题背景

某用户的 hs2 的 进程占用非常高( top res 值),但是该进程的 java heap memory 占用并不高,堆内存占用稳定,且没有发生 Full gc. 明显存在 native memory 的泄漏

### 排查过程

起初通过 nmt 工具来追踪本地内存,并没有追踪到原因.因为 nmt 工具依旧追踪不到 native memory . 后来所在 jvm 本地内存泄漏 ,找到贴子: [Java堆外内存排查小结 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903633524359175)

发现现象基本吻合.接着在 hs2 节点上通过使用 perf 工具抓取进程的所有方法栈统计.找到了非常接近的本地方法栈:

![[Pasted image 20221221210122.png]]

![](https://iwiki.woa.com/download/attachments/1963410565/image2022-7-14_20-7-28.png?version=1&modificationDate=1657800449000&api=v2)

接着搜索关键词: jvm native memory leak java_util_zip  找到结果第一条链接:  [https://bugs.openjdk.org/browse/JDK-8257032](https://bugs.openjdk.org/browse/JDK-8257032)  完全符合 issue 的描述,同时咨询了 kona jdk 关于这个 issue,证实 kona jdk8 上会有这个问题,并没有修复.最终定位到这个原因


### issue 分析

关键的错误在这里:

   * Writes remaining compressed data to the output stream and closes the
   * underlying stream.
   * @exception IOException if an I/O error has occurred
   */
  public void close() throws IOException {
      if (!closed) {
          finish();
          if (usesDefaultDeflater)
              def.end();
          out.close();
          closed = true;
      }
  }

java 中有关zip 压缩的4个重要类 DeflaterOutputStream   ,DeflaterInputStream  ,InflaterOutputStream,InflaterInputStream  这个 close() 方法调用.  如果在初始化中,没有使用默认的 Deflater ,那么就不会调用 def.end(). flater 是 Java 通过 jni 调用 libzip.so 包在进程本地内存中生成的对象.如果 java 层面没有调用 def.end. 那么 flater 在 c++ 代码层面就不会释放它的内存.由此造成了本地内存泄漏. 但是 java 还有一个兜底机制,就是在 java 层面发生 full gc 的时候,会去把 c++ 层面这部分没有 java 引用的对象进行回收. 所以这个问题并没有那么严重或者明显,因为只要发生 full gc ,这部分本地泄漏的内存会被释放掉.


---

该 issue 的详细分析参考:
[Native Memory — The Silent JVM Killer | by Conor Griffin | The Startup | Medium | The Startup](https://medium.com/swlh/native-memory-the-silent-jvm-killer-595913cba8e7)  

[Tracking Down Native Memory Leaks in Elasticsearch | Elastic Blog](https://www.elastic.co/cn/blog/tracking-down-native-memory-leaks-in-elasticsearch)

[Taming memory fragmentation in Venice with Jemalloc | LinkedIn Engineering](https://engineering.linkedin.com/blog/2021/taming-memory-fragmentation-in-venice-with-jemalloc)

