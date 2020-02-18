# JUC中的线程同步器
[toc]
## CountDownLatch
代码示例
```java
public class JoinCountDownLatch{
	private static volatile CountDownLatch countDownLatch = new CountDownLatch(2);
	public static void main(String[] args) throws InterruptedException{
		Thread t1 = new Thread(new Runnable(){
			@Override
			public void run(){
				try{
					Thread.sleep(1000);
				}catch (InterruptedException e){
					e.printStackTrace();
				}finally {
					countDownLatch.countDown();
				}
				System.out.println("child t1 over！");
			}
		})；
		Thread t2 = new Thread(new Runnable(){
			@Override
			public void run(){
				try{
					Thread.sleep(1000);
				}catch (InterruptedException e){
					e.printStackTrace();
				}finally {
					countDownLatch.countDown();
				}
				System.out.println("child t2 over！");
			}
		});
		t1.start();
		t2.start();
		System.out.println("wait all child thread over!");
		countDownLatch.await();
		System.out.println("all child thread over!")；
	}
}
```

## CyclicBarrier
线程执行完后，调用CyclicBarrier.await()方法阻塞自己。

## Join，CountDownLatch和CyclicBarrier三者的对比
join方法不能够在线程池中使用。CyclicBarrier类能够重复使用，CountDownLatch类不能。

## Semaphore信号量
一般在初始化的时候，传入参数0表明初始信号量为0，