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

#### 1.2.1 计数器 CoutDownLantch

<img src="./images/latch.png" alt="image-20230510204224054" style="zoom:50%;" />

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



## 1.2 JVM

### 1.2.1 内存模型

#### 1.2.2 类加载机制

#### 1.2.3 GC

#### 1.2.4. 优化





# 二、框架

#### 2.1 Spring

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

















