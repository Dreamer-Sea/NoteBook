# JUC中的原子并发类
[toc]
## 原子并发类在JDK 7 和JDK 8中的区别
在JDK 7中，原子并发类的递增和递减都是通过CAS操作实现的。在JDK 8中，是通过Unsafe类中的getAndAddInt方法或getAndAddLong方法实现的。本质上没有区别，知识在JDK 8中将循环逻辑内置了，提高了复用性。
## LongAdder类
LongAdder类是JDK 8中新增的。因为Atomic类在递增和递减的过程中会不断的进行CAS操作，当线程数越来越多(高并发情况)的时候，会浪费很多的CPU资源。
而LongAdder类加入了base和cell的概念，cell的数量为2的幂。在线程数较少的时候，不会初始化cell，所有操作都在base上进行。当线程数多了以后，就会初始化cell，如果线程在一个cell上的CAS操作没成功，它就会去另一个cell上尝试CAS操作，减少了线程的自旋，节省了CPU资源。
## LongAccumulator类
相比LongAdder类，该类能提供自定义的累加规则。