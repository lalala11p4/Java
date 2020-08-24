## spring中的@Scope()的含义以及为什么要有Threadlocal



### @Scope()可以有哪些值？

- singleton单例模式 -- 全局有且仅有一个实例
- prototype原型模式 -- 每次获取Bean的时候会有一个新的实例
- request -- request表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效

- session -- session作用域表示该针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效
- globalsession -- global session作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义

### 对于prototype和singleton的理解

​	由上述的定义可以理解，两者的区别就是在于单例还是多例，spring默认时单例的，可以通过这个注解进行修改(ConfigurableBeanFactory.SCOPE_PROTOTYPE)：

```java
@Controller
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class MapTest {
    public static int a = 1;
    public int b = 1;
    public int c = 1;
    public static HashMap<String, String> map22 = new HashMap<>();

    public static BlockingQueue<String> blockingQueue = new ArrayBlockingQueue<>(5);
}
```

​	具体的理解如下：

- A用户点击访问Controller，假如是多例的，所以在内存中会再new一个这个对象，所以就不存在共享的问题，(但是a，map22，blockingQueue还是共享的，因为是static)那要让多个controller里面的数据有交互怎么办？很简单，在定义一个单例类（或者定义static成员变量），在controller将数据放入到这个类中就可以，类似于redis、zk、数据库的性质一样。
- A用户点击访问Controller（M），假如是多例的，此时访问多例的N，会怎么样？因为单例在初始化的时候就已经完成，所以多例不会生效，N还是单例的。
- A用户点击访问Controller，假如是单例的，所以在内存中对象只有一个，那么成员变量也会共享，B用户改变b的值，A用户b的值也会改变，所以不安全。常规做法，把b定义为final（就不应该有普通成员变量的存在）
- 如果不想使用final即向对于数据进行改变又不想有数据安全问题怎么办？此时就需要Threadlocal，将当前的线程对象作为K，value作为value放入的threadlocal中的thread中的threadlocalmap中。