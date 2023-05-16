技能掌握程度：熟练使用（熟悉典型应用场景）、理解原理和特性

# 一、Java 基础

## 1. 多线程

### 1.1 线程池

#### 1.1.1 ThreadPoolExecutor

1. 原理：重用线程，实现任务的并发执行和系统资源的合理使用

2. 特性：线程池参数配置系统资源

3. 应用场景：

   1. 定时批量任务执行：每日结算账单生成任务、重试任务

      ```java
      int CPU_COUNT = Runtime.getRuntime().availableProcessors();
      
      
      ```

      

#### 1.1.2 Executors



### 1.2 同步工具

#### 1.2.1 数据竞争

<img src="./images/image-20230513025204800.png" alt="image-20230513025204800" style="zoom:33%;" width="450"/>

<img src="./images/image-20230513025530455.png" alt="image-20230513025530455" style="zoom:33%;" width="450"/>



多线程并发读写同一个共享变量，会导致数据错误，本质是【更新】不是原子操作（读，修改，写），因此必须同步共享变量的读写，保证写变量的【原子性】

#### 1.2.2 轻量级同步工具

1. 使用【atomic】原子类

   ```java
   import java.util.concurrent.atomic.AtomicInteger;
   
   public class AtomicExample {
       private static AtomicInteger count = new AtomicInteger(0);
   
       public static void main(String[] args) throws InterruptedException {
           Thread t1 = new Thread(() -> {
               for (int i = 0; i < 1000000; i++) {
                   count.incrementAndGet();
               }
           });
           t1.start();
         
           for (int i = 0; i < 1000000; i++) {
               count.decrementAndGet();
           }
           
           t1.join();
           System.out.println(count.get()); // 0
       }
   }
   ```

   

2. 使用【ReentrantLock】可重入锁

   ```java
   import java.util.concurrent.locks.ReentrantLock;
   
   public class ReentrantLockExample {
       private static int count = 0;
       private static ReentrantLock lock = new ReentrantLock();
   
       public static void main(String[] args) throws InterruptedException {
           Thread t1 = new Thread(() -> {
               for (int i = 0; i < 1000000; i++) {
                   lock.lock();
                   count++;
                   lock.unlock();
               }
           });
   				t1.start();
         
            for (int i = 0; i < 1000000; i++) {
                 lock.lock();
               	count--;
               	lock.unlock();
             }
   
           t1.join();
           System.out.println(count); // 0
       }
   }
   ```

   

3. 使用【Semaphore】信号量

   ```java
   import java.util.concurrent.Semaphore;
   
   public class SemaphoreExample {
       private static int count = 0;
       private static Semaphore semaphore = new Semaphore(1);
   
       public static void main(String[] args) throws InterruptedException {
           Thread t1 = new Thread(() -> {
               for (int i = 0; i < 1000000; i++) {
                   try {
                       semaphore.acquire();
                       count++;
                   } catch (InterruptedException e) {
                       e.printStackTrace();
                   } finally {
                       semaphore.release();
                   }
               }
           });
           t1.start();
         
           for (int i = 0; i < 1000000; i++) {
               try {
                   semaphore.acquire();
                   count--;
               } catch (InterruptedException e) {
                   e.printStackTrace();
               } finally {
                   semaphore.release();
               }
           }
        
           t1.join();
           System.out.println(count); // 0
       }
   }
   ```

   

#### 1.2.2 计数器 CoutDownLantch

<img src="./images/latch.png" alt="image-20230510204224054" style="zoom:50%;" width="450"/>

1. 原理：基于信号通知机制的同步计数器，同步线程执行顺序

2. 特性：阻塞当前线程，直到其他线程执行完某些操作或任务

3. 应用场景：

   1. 准备信号、开始信号、结束信号

      ```java
      static class Worker implements Runnable {
              private List<String> results;
              private CountDownLatch readySignal;
              private CountDownLatch startSignal;
              private CountDownLatch doneSignal;
      
              // 省略构造器
      
              @Override
              public void run() {
                  // 发送已准备信号
                  readySignal.countDown();
                  try {
                      // 等待执行信号
                      startSignal.await();
                      results.add("Counted down");
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  } finally {
                      // 发送已完成信号
                      // 必须放在 finally，防止异常未发送信号导致主线程一直阻塞
                      doneSignal.countDown();
                  }
              }
          }
      
          public static void main(String[] args) throws InterruptedException {
              int threadNums = 500;
              List<String> results = new ArrayList<>();
              CountDownLatch readySignal = new CountDownLatch(threadNums);
              CountDownLatch startSignal = new CountDownLatch(1);
              CountDownLatch doneSignal = new CountDownLatch(threadNums);
              List<Thread> workers = Stream
                      .generate(() -> new Thread(new Worker(results, readySignal, startSignal, doneSignal))).limit(threadNums).toList();
      
              workers.forEach(Thread::start);
              // 等待其他线程全部已准备信号
              readySignal.await();
              results.add("Workers ready");
              // 发送执行信号
              startSignal.countDown();
              // 等待其他现场全部已完成信号
              doneSignal.await();
              results.add("Workers complete");
      
              results.forEach(System.out::println);
               
              // Workers ready
              // Counted down
              // ... 
              // Workers complete
          }
      ```



## 2. JVM

### 2.1 内存模型

<img src="./images/image-20230513010300231.png" alt="image-20230513010300231" style="zoom:50%;" width="450"/>



<img src="./images/image-20230513010943755.png" alt="image-20230513010943755" style="zoom:25%;" width="450" />



现代多核cpu处理器，每个core运行一个thread，都有它自己的缓存。虽然使用缓存提高了处理器性能，但是又引入了缓存一致性问题，即当core1更新一个共享变量时，在写回策略下，不同步更新内存，core2访问该共享变量时，不是最新值。同时，编译器优化可能会对指令重排序。导致两个问题【更新不可见】和【指令重排序】。

https://www.geeksforgeeks.org/write-through-and-write-back-in-cache/

https://www.geeksforgeeks.org/happens-before-relationship-in-java/

https://www.baeldung.com/java-volatile

#### 2.1.1 volatile 关键字

volatile 可以保证【更新可见性】和避免【指令重排序】

```java
public class VolatileExample {
  
  	private static volatile boolean ready = false;
    private static int number;

    public static void main(String[] args) throws InterruptedException {
        Thread writerThread = new Thread(() -> {
            number = 42; // 1. 写操作
            ready = true; // 2. 使用 volatile 保证写操作不会被重排序
        });
 				writerThread.start();

       
        while (!ready) {
            // 等待直到 ready 变为 true
        }
        System.out.println(number); // 3. 读操作，由于使用了 volatile，这里会保证在写操作完成后执行

        writerThread.join();
    }
}
// 如果 ready 不加 volatile 关键字有可能会打印 number 为 0 
// 1. writerThread 写 number 到主存存在延迟
// 2. 指令重排序，1 2 顺序对调
```



【更新可见性】就是线程能立即看到共享变量的最新值，use case1 【状态标志】：

```java
public class VolatileFlagExample {
    private static volatile boolean isRunning = true;

    public static void main(String[] args) throws InterruptedException {
        Thread workerThread = new Thread(() -> {
            while (isRunning) {
                // do some work
            }
        });

        workerThread.start();
        Thread.sleep(1000);
        isRunning = false; // 使 workerThread 能立刻读到 isRunning 最新值
    }
}
```



【避免重排序】use case2【双重检查锁定】，避免【半初始化】问题

```java
// 在单例模式中，我们有时会使用双重检查锁定（Double-Checked Locking）来确保单例对象只被创建一次。在这种情况下，我们需要将单例对象声明为volatile，以防止由于JVM的指令重排导致其他线程在对象实例化完成前就看到了该对象的引用。
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    // JVM 中 instance = new Singleton() 它不是原子的，分为三个步骤
                    // 1. 分配对象的内存空间
                    // 2. 初始化对象
                    // 3. 将内存地址赋值给instance变量
                  	// 如果不加volatile，2 3可能交换顺序，导致其他线程拿到未初始化完的对象
                    instance = new Singleton(); 
                }
            }
        }
        return instance;
    }
}
```

https://www.baeldung.com/java-volatile

https://chat.openai.com/c/4952150e-9804-413c-8224-4fc3c21dd2ce

### 2.2 类加载机制

### 2.3 GC

### 2.4. 优化





# 二、框架

## 2.1 Spring

### 2.1 Spring IOC 容器



#### 2.1.1 DI 注入方式

##### 构造器注入（推荐）

特性：强制性依赖，返回完全初始化的状态，实现不可变对象

```java
@Component
@ToString
@RequiredArgsConstructor // final 字段且不为 null
public class People {
    private final Animal animal; // 不能注入 null
}

@Component
@ToString
public class People {
    private final Animal animal;
  
    public People(Animal animal) { // 不能注入 null，ioc 容器会忽略值为 null 的 bean，导致注入失败
        this.animal = animal;
    }
  	// Parameter 0 of constructor in com.example.beansioc.People required a bean of type 'com.example.beansioc.Animal' that 【could not be found.】
  	// The following candidates were found but 【could not be injected:】
	  // - User-defined bean method 'getAnimal' in 'Config' 【ignored as the bean value is null】
}

@Component
@ToString
public class People {
    private final Animal animal;
  
    public People(@Nullable Animal animal) { // 手动注入 null
        this.animal = animal;
    }
}
```



##### setter 注入

特性：非强制性依赖（可以设置默认值），可以重新注入

```java
@Component
@ToString
public class Person {
    private Animal animal;

    @Autowired
    public void setAnimal(@Nullable Animal animal) { // 注入null时，使用默认值
        this.animal = animal == null ? new Animal("default", 0) : animal;
    }
}

@Data
@AllArgsConstructor
public class Animal {
    private String name;
    private Integer age;
}

@Configuration
public class Config {
    @Bean(name = "animal")
    public Animal getAnimal() {
        return null;
    }
}
```

#### 2.1.1 Bean 生命周期

1. 原理：根据配置扫描创建 Bean 对象，同时提供各种回调接口用于自定义初始化、销毁等逻辑

2. 特性：提供各阶段的回调接口用于自定义逻辑

3. 应用场景：

   1. 自定义初始化逻辑 @PostConstruct 和自定义销毁逻辑 @PreDestroy

      ```java
      // 定义 POJO 类
      public class MySpringBean {
          @PostConstruct
          public void init() {
              System.out.println("custom init...");
          }
        
          public void greeting() {
              System.out.println("hello my name is " + this.getClass().getSimpleName());
          }
      
          @PreDestroy
          public void destroy() {
              System.out.println("custom destroy...");
          }
      }
      
      // 定义 Bean 配置类
      @Configuration
      public class BeanConfig {
          @Bean(name = "bean")
          public MySpringBean mySpringBean() {
              return new MySpringBean();
          }
      }
      
      
      // @Resource 依赖注入
      @SpringBootApplication
      public class BeanLiftCycleApplication implements CommandLineRunner {
      
          @Resource(name = "bean")
          private MySpringBean bean;
      
          public static void main(String[] args) {
              SpringApplication.run(BeanLiftCycleApplication.class, args);
          }
      
          @Override
          public void run(String... args) throws Exception {
              bean.greeting();
          }
      }
      
      // 输出
      custom init...
      hello my name is MySpringBean
      custom destroy...
      ```

      

#### 2.1.2 Spring 事务

1. 原理：
2. 特性：
3. 应用场景：
   1. @Transaction 注解

#### 2.1.3 AOP 切面

1. 原理：
2. 特性：
3. 应用场景：
   1. AOPContext 

#### 2.1.3 依赖注入

1. 原理：
2. 特性：
3. 应用场景
   1. @Autowire 和 @Resource

#### 2.2 SpringMVC

2.2.1 前端请求流程

2.2.2 拦截器

#### 2.3 MyBatis



# 三、中间件

#### 3.1 Redis

#### 3.2 MQ

#### 3.3 ES



# 四、数据库

#### 4.1 MySQL

#### 4.2 SQL 优化



# 五、系统架构

#### 5.1 分布式

#### 5.2 微服务

5.2.1 SOA

5.2.2 RPC

#### 5.3 负载均衡



# 六、设计模式

#### 6.1 工厂

#### 6.2 代理

#### 6.3 职责链

















