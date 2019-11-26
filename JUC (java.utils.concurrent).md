# Random类和ThreadLocalRandom类
[toc]
在JUC包中，会大量的使用CAS操作。
## Random类的局限性
在Random类中新的种子都是通过老的种子计算获得的，而计算使用的函数是固定不变的，这在单线程的情况下，不会发生什么问题。如果没有相应的处理措施，在多线程的情况下，可能出现线程A获得了新的种子a，而线程B还是根据旧的种子计算得到新的种子b，这时候种子a和种子b就是一样的。
在Random类中将种子(seed变量)声明为AtomicLong类型，使得种子的更新满足原子性，在Atomic类中，使用了CAS来更新数据保证了同一时间只有一个线程能够成功更新种子。
种子更新的代码：
```java
protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            nextseed = (oldseed * multiplier + addend) & mask;
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }
```
其它更新失败的线程不会被阻塞挂起，而是不断的尝试更新种子，在这种情况下，想要执行更新操作的线程越多，该方法的性能就会越差，降低了程序的并发性能。

## ThreadLocalRandom类
思想和ThreadLocal类似，就是每个线程有一个独占的种子。线程每次更新种子都是更新自己独占的种子，这样就可以避免线程的竞争，提高了程序的并发性能。