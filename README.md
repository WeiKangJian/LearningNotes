 这里是我学习笔记的目录界面。还有很多内容都会在学习的过程中逐步的更新，主要是JAVA后端开发方向的一些知识点，计划后面在项目的开发中着力更新高并发，高可用和分布式这类的知识点。也是对自己学习成果的一个检验。

欢迎各位同学的意见和指导。
***

# :coffee: Java基础
## :moon: 容器
- [深入底层之ArrayList和LinkList](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%AE%B9%E5%99%A8/%E6%B7%B1%E5%85%A5%E5%BA%95%E5%B1%82%E4%B9%8BArrayList%E5%92%8CLinkList.md)
- [再探集合之Iterable（快速失败，安全失败)](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%AE%B9%E5%99%A8/%E5%86%8D%E6%8E%A2%E9%9B%86%E5%90%88%E4%B9%8BIterable%EF%BC%88%E5%BF%AB%E9%80%9F%E5%A4%B1%E8%B4%A5%EF%BC%8C%E5%AE%89%E5%85%A8%E5%A4%B1%E8%B4%A5%29.md)
- [HashMap底层原理初探](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%AE%B9%E5%99%A8/HashMap%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86%E5%88%9D%E6%8E%A2.md)
- [LinkHashmap和TreeMap凭啥是有序（红黑树浅析）](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%AE%B9%E5%99%A8/LinkHashmap%E5%92%8CTreeMap%E5%87%AD%E5%95%A5%E6%98%AF%E6%9C%89%E5%BA%8F%E7%9A%84%EF%BC%88%E7%BA%A2%E9%BB%91%E6%A0%91%E6%B5%85%E6%9E%90%EF%BC%89.md)

## :computer: 多线程
- [多线程基础回顾](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80%E5%9B%9E%E9%A1%BE.md)
- [乐观悲观锁，自旋锁。和Syncrynized锁的三种状态](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80/%E4%B9%90%E8%A7%82%E6%82%B2%E8%A7%82%E9%94%81%EF%BC%8C%E8%87%AA%E6%97%8B%E9%94%81%E3%80%82%E5%92%8CSyncrynized%E9%94%81%E7%9A%84%E4%B8%89%E7%A7%8D%E7%8A%B6%E6%80%81.md)
- [多线程进阶之线程池（线程复用的原理，一篇够了）](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E8%BF%9B%E9%98%B6/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E8%BF%9B%E9%98%B6%E4%B9%8B%E7%BA%BF%E7%A8%8B%E6%B1%A0%EF%BC%88%E7%BA%BF%E7%A8%8B%E5%A4%8D%E7%94%A8%E7%9A%84%E5%8E%9F%E7%90%86%EF%BC%8C%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86%EF%BC%89.md)
- [深入锁和并发的核心（1）——AQS](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E8%BF%9B%E9%98%B6/%E6%B7%B1%E5%85%A5%E9%94%81%E5%92%8C%E5%B9%B6%E5%8F%91%E7%9A%84%E6%A0%B8%E5%BF%83%EF%BC%881%EF%BC%89%E2%80%94%E2%80%94AQS.md)
- [深入锁和并发的核心（2）——并发集合](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E8%BF%9B%E9%98%B6/%E6%B7%B1%E5%85%A5%E9%94%81%E5%92%8C%E5%B9%B6%E5%8F%91%E7%9A%84%E6%A0%B8%E5%BF%83%EF%BC%882%EF%BC%89%E2%80%94%E2%80%94%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88.md)
- [深入锁和并发的核心（3）——并发工具类](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%A4%9A%E7%BA%BF%E7%A8%8B%E8%BF%9B%E9%98%B6/%E6%B7%B1%E5%85%A5%E9%94%81%E5%92%8C%E5%B9%B6%E5%8F%91%E7%9A%84%E6%A0%B8%E5%BF%83%EF%BC%883%EF%BC%89%E2%80%94%E2%80%94%E5%B9%B6%E5%8F%91%E5%B7%A5%E5%85%B7%E7%B1%BB.md)

## :cloud: JVM虚拟机
- [一些高级特性和补充知识点（反射）](https://github.com/WeiKangJian/LearningNotes/blob/master/JVM/%E4%B8%80%E4%BA%9B%E9%AB%98%E7%BA%A7%E7%89%B9%E6%80%A7%E5%92%8C%E8%A1%A5%E5%85%85%E7%9F%A5%E8%AF%86%E7%82%B9.md)
- [再刷JVM（1）——深入内存模型JMM](https://github.com/WeiKangJian/LearningNotes/blob/master/JVM/%E5%86%8D%E5%88%B7JVM%EF%BC%881%EF%BC%89%E2%80%94%E2%80%94%E6%B7%B1%E5%85%A5%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8BJMM.md)
- [再刷JVM（2）——分析监控JVM的工具](https://github.com/WeiKangJian/LearningNotes/blob/master/JVM/%E5%86%8D%E5%88%B7JVM%EF%BC%882%EF%BC%89%E2%80%94%E2%80%94JVM%E5%88%86%E6%9E%90%E5%92%8C%E7%9B%91%E6%8E%A7%E5%B7%A5%E5%85%B7.md)

## :orange: JAVA I/O
- [IO流学习总结](https://github.com/WeiKangJian/LearningNotes/blob/master/IO%E8%BF%9B%E9%98%B6/IO%E6%B5%81%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93%20.md)

***

# :bulb: 后端进阶
- [初探Spring之IOC](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%A1%86%E6%9E%B6/Spring/%E5%88%9D%E6%8E%A2Spring%E4%B9%8BIOC.md)
- [再探SpringIOC（1），深入源码了解构建过程](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%A1%86%E6%9E%B6/Spring/%E5%86%8D%E6%8E%A2SpringIOC%EF%BC%8C%E6%B7%B1%E5%85%A5%E6%BA%90%E7%A0%81%E4%BA%86%E8%A7%A3%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B.md)
- [再探SpringIOC（2），了解作用域和生命周期](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%A1%86%E6%9E%B6/Spring/%E5%86%8D%E6%8E%A2SpringIOC%EF%BC%8C%E4%BA%86%E8%A7%A3bean%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%92%8C%E4%BD%9C%E7%94%A8%E5%9F%9F%20(1).md)
- [SpringMVC工作原理分析](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%A1%86%E6%9E%B6/Spring/SpringMVC%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90.md)
- [Tomcat原理剖析](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%A1%86%E6%9E%B6/Spring/Tomcat%E5%8E%9F%E7%90%86%E5%89%96%E6%9E%90.md)
- [JDK动态代理](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%A1%86%E6%9E%B6/Spring/JDK%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86.md)
- [分析SprigAOP底层实现，别再问我啦](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%A1%86%E6%9E%B6/Spring/%E5%88%AB%E5%86%8D%E9%97%AE%E6%88%91SpringAOP%E5%95%A6%EF%BC%8C%E9%83%BD%E5%9C%A8%E8%BF%99%E9%87%8C%E4%BA%86.md)
- [SpringBoot集成和配置]
- [SpringBoot中拦截器和权限验证]

***

## :floppy_disk: 数据库
- [MySQL](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9B%B8%E5%85%B3/MySQL.md)
- [Redis基础](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9B%B8%E5%85%B3/Redis%E5%9F%BA%E7%A1%80%E6%80%BB%E7%BB%93.md)
***
## :apple: 网络
- [深入了解HTTPS](https://github.com/WeiKangJian/LearningNotes/blob/master/%E7%BD%91%E7%BB%9C%E7%9B%B8%E5%85%B3/%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3HTTPS.md)
- [TCP/IP]
***

## :spider: 算法
- [递归](https://github.com/WeiKangJian/LearningNotes/blob/master/%E7%AE%97%E6%B3%95/%E9%80%92%E5%BD%92.md)
- [回溯](https://github.com/WeiKangJian/LearningNotes/blob/master/%E7%AE%97%E6%B3%95/%E5%9B%9E%E6%BA%AF.md)
- [排序](https://github.com/WeiKangJian/LearningNotes/blob/master/%E7%AE%97%E6%B3%95/%E6%8E%92%E5%BA%8F.md)
- [动态规划]()
- [贪心]()
***

## :eyes:搜索引擎
- [lucene源码阅读(1)](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E/lucene%E5%BA%95%E5%B1%82%E5%AD%A6%E4%B9%A0%EF%BC%88%E4%B8%80%EF%BC%89.md)
- [solr配置和数据库导入](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E/solr%E5%9F%BA%E7%A1%80.md)
- [solrj——搜索引擎整合项目](https://github.com/WeiKangJian/LearningNotes/blob/master/%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E/solr%20%E2%80%94%E2%80%94%E6%90%9C%E7%B4%A2%E5%BC%95%E6%93%8E%E5%AE%9E%E6%88%98.md)
- [ES初步了解，有时间再实现]

***
## 🐾 百度实习日记
- [实习面经](https://github.com/WeiKangJian/LearningNotes/blob/master/%E5%AE%9E%E4%B9%A0/%E7%99%BE%E5%BA%A6%E7%A0%94%E5%8F%91%E5%B2%97%E5%AE%9E%E4%B9%A0%E9%9D%A2%E7%BB%8F.md)

## :wrench: 工具 
- [Git]
- [Docker]
- [Maven]
- [Linux避坑指南](https://github.com/WeiKangJian/LearningNotes/blob/master/Linux/Linux%E9%83%A8%E7%BD%B2%E4%B8%8A%E7%9A%84%E9%82%A3%E4%BA%9B%E5%9D%91.md)
- [秒杀系统设计总结](https://github.com/WeiKangJian/LearningNotes/blob/master/%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1/%E7%A7%92%E6%9D%80%E7%B3%BB%E7%BB%9F%E8%AE%BE%E8%AE%A1%E6%80%BB%E7%BB%93.md)
***



### License

本笔记仓库的内容，在创作时候有部分参考了网络的其他技术博客和引用了相关图片，在具体的文中都有标明出处。大部分都是我的原创。在您引用本仓库内容或者对内容进行修改演绎时，请署名并以相同方式共享，谢谢。
***
### 补充
有很多部分的知识点是以前学习的，现在课程较多没来的及补上，所以将这一部分知识点和未来的一些计划学习的内容列在目录中，之后会逐步更新






