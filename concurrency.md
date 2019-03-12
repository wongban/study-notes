# Java Concurrency in Practice

## 线程安全性

线程安全性的定义中，最核心的概念就是正确性。正确性的含义是，某个类的行为与其规范完全一致。

当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类是线程安全的。

无状态对象一定是线程安全的。

### 原子性

递增操作`++count`是一个复合操作。这是一个“读取-修改-写入”的操作序列，并且其结果状态依赖于之前的状态。

#### 竞态条件

当某个计算的正确性取决于多个线程的交替执行时序时，就会发生竞态条件。（正确的结果取决于运气）

最常见的竞态条件类型就是：

+ 先检查后执行(Check-Then-Act)：通过一个可能失效的观测结果来决定下一步动作。

  常见的情况：

  + 延迟初始化

    ```java
    public class LazyInitRace {
        private ExpensiveObject instance = null;
        public ExpensiveObject getInstance() {
            if (instance == null) instance = new ExpensiveObject();
            return instance;
        }
    }
    ```

  + 读取-修改-写入
        递增操作

当在无状态类中添加一个状态时，如果该状态完全由线程安全的对象来管理，那么这个类仍然是线程安全的。

当状态变量的数量由一个变为多个时，并不会像由零个变为一个那样简单。

### 加锁机制

#### 内置锁

内置锁是可重入的。某个线程试图获得一个已经由它自己持有的锁，那么这个请求就会成功。“重入”意味着获取锁的操作的粒度是“线程”，而不是“调用”。

### 用锁来保护状态

锁能够使其保护的代码路径以串行形式访问，因此可以通过锁来构造一些协议以实现对共享状态的独占访问。

仅仅将复合操作封装到一个同步代码块中是不够的。如果用同步来协调对某个变量的访问，那么在访问这个变量的所有位置上都需要使用同步。而且，当使用锁来协调对某个变量的访问时，在访问变量的所有位置上都要使用同一个锁。

如果在添加新的方法或代码路径时忘记了使用同步，那么这种加锁协议会很容易被破坏。

如果只是将每个方法都作为同步方法，例如`Vector`，那么并不足以确保`Vector`上复合操作都是原子的：`if (!vector.contains(element)) vector.add(element)`;

虽然`contains`和`add`等方法都是原子方法。但在上面这个“如果不存在则添加”的操作中仍然存在竞态条件。虽然`synchronized`方法可以确保单个操作的原子性，但如果要把多个操作合并为一个复合操作，还是需要额外的加锁机制。

### 活跃性和性能

要判断同步代码块的合理大小，需要在各种设计需求之间进行权衡，包括安全性（必须得到满足）、简单性和性能。