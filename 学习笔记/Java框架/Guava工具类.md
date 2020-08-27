# Guava

## 1. utils工具类

### 1.1 Joiner - 字符串连接

> 是一个作用于可迭代的集合对象转字符串，进行拼接的工具类对象；方法如下：
>
> - `public static Joiner on(String separator)`：指定字符串拼接的字符串；
> - `public static Joiner on(char separator)`：指定字符串拼接的字符；
> - `public final String join(Iterable<?> parts)` ：将可迭代的对象进行集合拼接转为字符串；
>   - 如果没有skipNull的话，遇见为空的对象，将会报错；
>   - `useForNull(String str)`：将为空的对象，替换为字符串str；
> - `skipNulls()` 重新创建一个Joiner对象，并将集合对象中的null对象进行移除；
> - `appendTo(StringBuilder b, Iterable<?> parts)`：将集合进行拼接，并将拼接的值写入p中；
> - 

```java


    private List<String> list = Lists.newArrayList("kafka", "zookeeper", "redis", "mongodb");

    private List<String> listWithNull = Lists.newArrayList("mongodb", null, "kafka", "docker");

    /**
     * join(Iterable)
     */
    @Test
    public void testJoiner() {
        String join = Joiner.on(",").join(list);
        System.out.println(join);
    }

    /**
     * 针对join过程，跳过为空的数据
     */
    @Test
    public void testJoinerSkipNull() {
        String join1 = Joiner.on(",").skipNulls().join(listWithNull);
        System.out.println(join1);
    }

    /**
     * 将集合转字符串，写入到StringBuilder中
     */
    @Test
    public void testJoinerToBuilder() {
        StringBuilder stringBuilder = new StringBuilder();
        Joiner.on(",").appendTo(stringBuilder, list);
    }

    /**
     * java8的写法
     */
    @Test
    public void testJava8Joiner() {
        String collect = listWithNull.stream().filter(item -> item != null && !item.isEmpty()).collect(Collectors.joining(","));
        System.out.println(collect);
    }
```

### 1.2 Splitter - 字符串分割

> 针对字符串进行分割的操作；
>
> - `List<String> splitToList(CharSequence sequence)`：将字符串分割为集合；
> - `Splitter omitEmptyStrings()`：将为分割出来以后为空的数据进行清理；
> - `Splitter trimResults()`：将分割出来的结果进行trim；
> - `static Splitter fixedLength(final int length)`：固定长度进行分割；
> - `static Splitter on(final String separator)`：根据字符串进行切割；
> - `static Splitter onPattern(String separatorPattern)`：使用正则表达式进行切分；

```java
package guava.utils;

import com.google.common.base.CharMatcher;
import com.google.common.base.Splitter;
import org.junit.Test;

import java.util.List;
import java.util.Map;

/**
 * @author ${张世林}
 * @date 2020/07/31
 * 作用：
 */
public class SplitterTest {

    @Test
    public void testSplitter() {
        List<String> list = Splitter.on(",").splitToList("hello,world");
        list.forEach(temp -> System.out.println(temp + " "));
        Iterable<String> split = Splitter.on(",").split("hello,world");
        split.forEach(temp -> System.out.println(temp + " "));
    }

    /**
     * 如果分割出来存在着空字符串，则需要对其进行忽略
     *  omitEmptyStrings()
     */
    @Test
    public void testSplitterOmitNull() {
        List<String> list = Splitter.on(",").omitEmptyStrings().splitToList("hello,world,,zhang,,,,san");
        list.forEach(temp -> System.out.println(temp + " "));
        System.out.println(list.size());
    }

    /**
     * 如果分割出来的字符串存在空格，则去掉空格
     *  omitEmptyStrings()
     */
    @Test
    public void testSplitterTrim() {
        List<String> list = Splitter.on(",").trimResults().splitToList("hello, world,,zhang ,,,,san");
        list.forEach(temp -> System.out.print(temp));
        System.out.println(list.size());
    }

    /**
     * 如果分割出来的字符串存在空格，则去掉空格
     *  omitEmptyStrings()
     */
    @Test
    public void testSplitterFixedSplit() {
        List<String> list = Splitter.fixedLength(4).splitToList("hello, world,,zhang ,,,,san");
        List<String> list1 = Splitter.on(",").limit(4).splitToList("hello, world,,zhang ,,,,san");
        list.forEach(temp -> System.out.println(temp));
        list1.forEach(temp -> System.out.println(temp));
        System.out.println(list.size());
    }

    @Test
    public void testSplitterWithPattern() {
        List<String> strings = Splitter.onPattern("\\|").trimResults().splitToList("张世林|||长得帅|是的||吧");
        strings.forEach(System.out::println);
    }

    /**
     * 将字符串进行逗号分割，再根据分割出来的数据进行等号转map
     */
    @Test
    public void testSplitterToMap() {
        Map<String, String> split = Splitter.on(",").withKeyValueSeparator("=").split("name=zhangsan,age=24");
        split.forEach((k,v) -> System.out.println(k + " = " + v));
    }
}

```

### 1.3 Preconditions - 断言检查

> 断言主要是作用于数据的检查，能够及早发现数据异常问题；



### 1.4 Strings - 字符串

> `Strings`是对字符串进行操作的工具类；
>
> - `static String emptyToNull(@Nullable String string)`：将空字符串转换为 null；
>
> - `static String nullToEmpty(@Nullable String string)`：将 null 转换为空字符串；
> - `static String commonPrefix(CharSequence a, CharSequence b)`：得到两个字符串公共的前缀；
> - `static String repeat(String string, int count)`：将字符串进行复制；

```java
package guava.utils;

import com.google.common.base.Strings;
import org.junit.Test;

/**
 * @author ${张世林}
 * @date 2020/08/01
 * 作用：
 */
public class StringsTest {

    /**
     * 将 empty 转换为 空字符串
     */
    @Test
    public void testEmptyToNull() {
        String s = Strings.emptyToNull("");
        System.out.println(s);
    }

    /**
     * 将 null转换为empty
     */
    @Test
    public void testNullToEmpty() {
        String s = Strings.nullToEmpty(null);
        System.out.println(s);
    }

    /**
     * 得到公共的前后缀
     */
    @Test
    public void testCommonPrefix() {
        String s = Strings.commonPrefix("hello", "he is a boy");
        System.out.println(s);
    }

    /**
     * 将字符串进行复制
     */
    @Test
    public void testRepeat() {
        //将 zsl 复制5次
        String zsl = Strings.repeat("zsl", 5);
        System.out.println(zsl);
    }

    @Test
    public void testPadding() {
        String s = Strings.padEnd("zhang shilin", 100, 'c');
        System.out.println(s);
    }
}

```



### 1.5 Charsets 字符集工具类

> 字符集工具类主要是对对字符集编码进行操作；提供了字符集对象的常量；
>
> - `public static final Charset US_ASCII = Charset.forName("US-ASCII");`
> - `public static final Charset ISO_8859_1 = Charset.forName("ISO-8859-1");`
> - `public static final Charset UTF_8 = Charset.forName("UTF-8");`
> - `public static final Charset UTF_16BE = Charset.forName("UTF-16BE");`
> - `public static final Charset UTF_16LE = Charset.forName("UTF-16LE");`
> - `public static final Charset UTF_16 = Charset.forName("UTF-16");`

```java
    @Test
    public void testCharset() {
        Charset charset = Charset.forName("UTF-8");
        assertThat(charset, equalTo(Charsets.UTF_8));
    }
```

### 1.6 CharMatcher-字符操作工具类

> `CharMatcher`是一个字符操作的工具类，该工具类主要是作用于字符的统计、匹配、替换等功能；



### 1.7 Files类-文件操作

> - `static void copy(File from, File to) throws IOException` 文件复制
> - `static void move(File from, File to) throws IOException` 文件移动
> - `static CharSource asCharSource(File file, Charset charset)`
>   - `readLines(LineProcessor<T> processor) throws IOException`  一行一行读取数据，并且每读取一行，就会调用LineProcessor处理器进行处理，如果返回false，则停止读取文件；
> - ......



### 1.8 文件资源操作

> - `CharSource`：字符读取资源，相当于是Reader；
> - `CharSink`：字符写出资源，相当于是Writer；
> - `ByteSource`：字节读取资源，相当于是ByteIputStream；
> - `ByteSink`：字节写出资源，相当于是ByteOutputStream；



### 1.9 Closer

> `Closer`主要是在操作文件流的时候，涉及到catch-finally中的双重抛异常时，需要解决的问题；

```java
@Test
    public void testCreateCloser() throws IOException {
        ByteSource byteSource = Files.asByteSource(new File("D:\\city_target2.txt"));
        Closer closer = Closer.create();
        try {
            InputStream inputStream = closer.register(byteSource.openStream());
            int available = inputStream.available();
            System.out.println(available);
        } catch (Throwable e) {
            closer.rethrow(e);
        } finally {
            closer.close();
        }
    }
```



### 1.10 Base64操作

> - `BaseEncoding`：基于Base64的操作
>   - `encode(String str)`：编码
>   - `decode(String baseCode)`：解码



## 2.EventBus

> `EventBus`是一个专注于进程内的消息总线，主要是多线程之间消息进行通信；
>
> `EventBus`是使用EventBus对象来进行消息的发送，发送消息体格式：
>
> - 消息体格式：使用对象进行消息的发送，监听端与发送端发送的对象需一致；
> - 多个监听者：可以有多个监听者监听同一个对象，实现广播模式；也可监听不同的对象，实现多个消息体的监听；
> - 消息监听者继承：如果一个消息监听者的父类也监听该对象，则也会成为监听对象；

使用EventBus实现一个监听者：

```java
package guava.eventbus;

import com.google.common.eventbus.EventBus;

import java.nio.file.*;
import java.util.logging.Logger;

/**
 * @author ${张世林}
 * @date 2020/08/08
 * 作用：使用EventBus来实现目录变化的监听
 */
public class DirectoryTargetMonitor implements TargetMonitor {

    private final EventBus eventBus;
    private final Path path;
    private WatchService watchService;
    private volatile boolean start = false;

    public DirectoryTargetMonitor(final EventBus eventBus, final String targetPath) {
        this.eventBus = eventBus;
        this.path = Paths.get(targetPath);
    }

    public DirectoryTargetMonitor(final EventBus eventBus, final String targetPath, final String... morePaths) {
        this.eventBus = eventBus;
        this.path = Paths.get(targetPath, morePaths);
    }

    @Override
    public void startMonitor() throws Exception {
        this.watchService = FileSystems.getDefault().newWatchService();
        this.path.register(watchService, StandardWatchEventKinds.ENTRY_CREATE,
                StandardWatchEventKinds.ENTRY_MODIFY,
                StandardWatchEventKinds.ENTRY_DELETE);
        this.start = true;
        while (start) {
            WatchKey watchKey = null;
            try {
                watchKey = watchService.take();
                watchKey.pollEvents().forEach(event -> {
                    WatchEvent.Kind<?> kind = event.kind();
                    Path path = (Path) event.context();
                    Path resolve = DirectoryTargetMonitor.this.path.resolve(path);
                    eventBus.post(new FileChangeEvent(kind, resolve));
                });
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    @Override
    public void stopMonitor() throws Exception {
        System.out.println("thread monitor interrupt");
        Thread.currentThread().interrupt();
        start = false;
        this.watchService.close();
        System.out.println("thread monitor interrupt done");

    }
}
```

```java
package guava.eventbus;

import com.google.common.eventbus.Subscribe;

import java.nio.file.Path;
import java.nio.file.WatchEvent;

/**
 * @author ${张世林}
 * @date 2020/08/08
 * 作用：
 */
public class FileChangeEvent {
    final WatchEvent.Kind<?> kind;
    final Path path;

    public FileChangeEvent(WatchEvent.Kind<?> kind, Path path) {
        this.kind = kind;
        this.path = path;
    }

    public WatchEvent.Kind<?> getKind() {
        return kind;
    }

    public Path getPath() {
        return path;
    }
}

```

```java
package guava.eventbus;

import com.google.common.eventbus.Subscribe;

/**
 * @author ${张世林}
 * @date 2020/08/08
 * 作用：
 */
public class FileChangeListener {

    @Subscribe
    public void fileChangeEvent(FileChangeEvent event) {
        System.out.println(event.getPath());
        System.out.println(event.getKind());
    }

}
```

```java
package guava.eventbus;

public interface TargetMonitor {

    void startMonitor() throws Exception;
    void stopMonitor() throws Exception;

}
```

```java
package guava.eventbus;

import com.google.common.eventbus.EventBus;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：
 */
public class TestFileChange {

    public static void main(String[] args) throws Exception {
        EventBus eventBus = new EventBus();
        eventBus.register(new FileChangeListener());
        DirectoryTargetMonitor monitor = new DirectoryTargetMonitor(eventBus, "D:\\test_watch");
        monitor.startMonitor();
    }

}

```



### 2.1 自己实现的EventBus

- 创建Bus接口，需要注册监听器的接口、发送消息的接口、取消注册、关闭资源、获取Bus名的接口；

```java
package guava.eventbus.myself;

/**
 * 实现一个EventBus接口
 */
public interface Bus {

    /**
     * 注册监听器
     * @param subscriber
     */
    void registry(Object subscriber);

    /**
     * 取消注册监听器
     * @param subscriber
     */
    void unRegistry(Object subscriber);

    /**
     * 发送消息
     *
     * @param event
     */
    void post(Object event);

    /**
     * 指定队列发送消息
     * @param topic
     * @param event
     */
    void post(String topic, Object event);

    void close();

    String getBusName();
}

```

- 创建EventBus对象；**EventBus四开放于客户端使用的，所以是一个入口，具体实现写在Dispatcher中；**

```java
package guava.eventbus.myself;

import java.util.concurrent.Executor;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：自定义EventBus
 */
public class EventBus implements Bus {

    private final Registry registry = new Registry();

    /**
     * EventBus消息总线
     */
    private String busName;

    /**
     * 上下文对象
     */
    private EventBusContext context;

    private final Dispatcher dispatcher;

    private final EventBusExceptionHandler eventBusExceptionHandler;

    private static final String DEFAULT_BUS_NAME = "default";

    private static final String DEFAULT_TOPIC_NAME = "default";

    public EventBus() {
        this(DEFAULT_BUS_NAME);
    }
    public EventBus(final String busName, EventBusExceptionHandler eventBusExceptionHandler) {
        this(DEFAULT_BUS_NAME, null, eventBusExceptionHandler, Dispatcher.SEQ_EXECUTOR_SERVICE);
    }

    public EventBus(final String busName) {
        this(busName, null, null, Dispatcher.SEQ_EXECUTOR_SERVICE);
    }

    public EventBus(final String busName, EventBusContext context, EventBusExceptionHandler eventBusExceptionHandler, Executor executor) {
        this.busName = busName;
        this.context = context;
        this.eventBusExceptionHandler = eventBusExceptionHandler;
        this.dispatcher = Dispatcher.newDispatcher(eventBusExceptionHandler, executor);
    }

    @Override
    public void registry(Object subscriber) {
        this.registry.bind(subscriber);
    }

    @Override
    public void unRegistry(Object subscriber) {
        this.registry.unBind(subscriber);
    }

    @Override
    public void post(Object event) {
        this.post(DEFAULT_TOPIC_NAME, event);
    }

    @Override
    public void post(String topic, Object event) {
        this.dispatcher.dispatch(this, registry, event, topic);
    }

    @Override
    public void close() {
        this.dispatcher.close();
    }

    @Override
    public String getBusName() {
        return busName;
    }
}

```

- 创建消息分发器对象；**消息分发器是具体实现消息发送的逻辑，包含了自定义的分发策略；**

```java
package guava.eventbus.myself;

import java.lang.reflect.Method;
import java.util.concurrent.ConcurrentLinkedDeque;
import java.util.concurrent.Executor;
import java.util.concurrent.ExecutorService;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：EventBus的分发器对象
 */
public class Dispatcher {

    private final Executor executorService;

    private final EventBusExceptionHandler eventBusExceptionHandler;

    public static final Executor SEQ_EXECUTOR_SERVICE = SeqExecutorService.INSTANCE;
    public static final Executor THREAD_EXECUTOR_SERVICE = ThreadExecutorService.INSTANCE;

    public Dispatcher(Executor executorService, EventBusExceptionHandler eventBusExceptionHandler) {
        this.executorService = executorService;
        this.eventBusExceptionHandler = eventBusExceptionHandler;
    }

    static Dispatcher newDispatcher(EventBusExceptionHandler eventBusExceptionHandler, Executor executorService) {
        return new Dispatcher(executorService, eventBusExceptionHandler);
    }

    static Dispatcher seqDispatcher(EventBusExceptionHandler eventBusExceptionHandler) {
        return new Dispatcher(SEQ_EXECUTOR_SERVICE, eventBusExceptionHandler);
    }

    static Dispatcher threadDispatcher(EventBusExceptionHandler eventBusExceptionHandler) {
        return new Dispatcher(THREAD_EXECUTOR_SERVICE, eventBusExceptionHandler);
    }

    /**
     * 执行消息分发
     *
     * @param bus
     * @param registry
     * @param event
     * @param topicName
     */
    public void dispatch(Bus bus, Registry registry, Object event, String topicName) {
        ConcurrentLinkedDeque<SubScriber> subscribers = registry.scanSubscribers(topicName);
        if (subscribers == null) {
            if (eventBusExceptionHandler != null) {
                //TODO 将Context进行封住
                eventBusExceptionHandler.handle(new IllegalArgumentException("the topic not bind subScribe"),
                        new BaseEventBusContext(bus.getBusName(), null, event));
            }
        }
        subscribers.stream()
                .filter(subscriber -> !subscriber.isDisable())
                .filter(subscriber -> {
                    Method scribeMethod = subscriber.getSubScribeMethod();
                    Class<?> parameterType = scribeMethod.getParameterTypes()[0];
                    return (parameterType.isAssignableFrom(event.getClass()));
                })
                .forEach(subscriber -> realInvokeSubscriber(subscriber, event, bus));
    }

    /**
     * 真正去执行
     *
     * @param subscriber
     * @param event
     * @param bus
     */
    private void realInvokeSubscriber(SubScriber subscriber, Object event, Bus bus) {
        Method subScribeMethod = subscriber.getSubScribeMethod();
        Object subScribeObject = subscriber.getSubScribeObject();
        executorService.execute(() -> {
            try {
                subScribeMethod.invoke(subScribeObject, event);
            } catch (Exception e) {
                if (eventBusExceptionHandler != null) {
                    eventBusExceptionHandler.handle(e,
                            new BaseEventBusContext(bus.getBusName(), null, event));
                }
            }
        });
    }

    public void close() {
        if (executorService instanceof ExecutorService) {
            ((ExecutorService) executorService).shutdown();
        }
    }

    /**
     * 执行策略一
     */
    private static class SeqExecutorService implements Executor {
        private final static SeqExecutorService INSTANCE = new SeqExecutorService();

        @Override
        public void execute(Runnable command) {
            command.run();
        }
    }

    /**
     * 执行策略二
     */
    private static class ThreadExecutorService implements Executor {
        private final static ThreadExecutorService INSTANCE = new ThreadExecutorService();

        @Override
        public void execute(Runnable command) {
            new Thread(command).start();
        }
    }

    /**
     * 上下文对象
     */
    private static class BaseEventBusContext implements EventBusContext {
        private final String eventBusName;
        private final SubScriber subScriber;
        private final Object event;

        public BaseEventBusContext(String eventBusName, SubScriber subScriber, Object event) {
            this.eventBusName = eventBusName;
            this.subScriber = subScriber;
            this.event = event;
        }

        @Override
        public String getSource() {
            return this.eventBusName;
        }

        @Override
        public Object getSubscriber() {
            return this.subScriber != null ? subScriber.getSubScribeObject() : null;
        }

        @Override
        public Method getSubscribe() {
            return this.subScriber != null ? subScriber.getSubScribeMethod() : null;
        }

        @Override
        public Object getEvent() {
            return this.event;
        }
    }
}

```

- 创建EventBus的上下文对象，主要是需要各个环节扭转中封装bus的所有对象信息；

```java
package guava.eventbus.myself;

import java.lang.reflect.Method;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：EventBus的上下文
 */
public interface EventBusContext {

    String getSource();

    Object getSubscriber();

    Method getSubscribe();

    Object getEvent();

}

```

- 创建异常处理的Handler对象；**使用java8的接口开发，将异常处理的具体实现逻辑留与客户端；并将Context对象封装bus以及event对象等信息也携带过去；**

```java
package guava.eventbus.myself;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：/处理EventBus的异常
 */
public interface EventBusExceptionHandler {
    void handle(Throwable cause, EventBusContext eventBusContext);
}

```

- 创建Registry对象，该对象是使用topic为key，queue队列为value的形式，保存所有注册进入Bus的监听器；

```java
package guava.eventbus.myself;

import org.junit.Test;

import java.lang.reflect.Method;
import java.lang.reflect.Modifier;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentLinkedDeque;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：
 */
public class Registry {

    private ConcurrentHashMap<String, ConcurrentLinkedDeque<SubScriber>> subscriberContainer = new ConcurrentHashMap<>();

    public void bind(Object subscriber) {
        List<Method> subScribeMethods = getSubScribeMethod(subscriber);
        subScribeMethods.forEach(method -> tierSubscriber(subscriber, method));
    }

    private void tierSubscriber(Object subscriber, Method method) {
        Subscribe subscribe = method.getDeclaredAnnotation(Subscribe.class);
        String topic = subscribe.topic();
        subscriberContainer.computeIfAbsent(topic, key -> new ConcurrentLinkedDeque<SubScriber>());
        subscriberContainer.get(topic).add(new SubScriber(subscriber, method));
    }

    /**
     * 获取所有标注了Subscribe注解的代码
     * @param subscriber
     * @return
     */
    private List<Method> getSubScribeMethod(Object subscriber) {
        final List<Method> methods = new ArrayList<>();
        Class<?> temp = subscriber.getClass();
        while (temp != null) {
            Method[] declaredMethods = temp.getDeclaredMethods();
            Arrays.stream(declaredMethods)
                    .filter(method -> method.isAnnotationPresent(Subscribe.class) && method.getParameterCount() == 1 && method.getModifiers() == Modifier.PUBLIC)
                    .forEach(methods::add);
            temp = temp.getSuperclass();
        }
        return methods;
    }

    public void unBind(Object subscriber) {
        subscriberContainer.forEach((key, queue) -> {
            queue.forEach(s -> {
                if (s.getSubScribeObject() == subscriber) {
                    s.setDisable(true);
                }
            });
        });
    }

    public ConcurrentLinkedDeque<SubScriber> scanSubscribers(String topicName) {
        return subscriberContainer.get(topicName);
    }
}

```

- 注解类@Subscribe，用于标识是一个监听方法；

```java
package guava.eventbus.myself;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Subscribe {

    String topic() default "default";

}

```

- 将监听器进行封装，封装方法Method、封装对象Object；

```java
package guava.eventbus.myself;

import java.lang.reflect.Method;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：
 */
public class SubScriber {

    private final Object subScribeObject;
    private final Method subScribeMethod;

    private boolean disable = false;

    public SubScriber(Object subScribeObject, Method subScribeMethod) {
        this.subScribeObject = subScribeObject;
        this.subScribeMethod = subScribeMethod;
    }

    public Object getSubScribeObject() {
        return subScribeObject;
    }

    public Method getSubScribeMethod() {
        return subScribeMethod;
    }

    public boolean isDisable() {
        return disable;
    }

    public void setDisable(boolean disable) {
        this.disable = disable;
    }
}

```

- 创建监听器，在@subscribe注解中指定topic的名称；

```java
package guava.eventbus.myself;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：
 */
public class SampleListener {

    @Subscribe
    public void test1(String x) {
        System.out.println("SampleListener==>test1=="+ x);
    }

    @Subscribe(topic = "dxgk_v1.0")
    public void test2(String x) {
        System.out.println("SampleListener==>test1=="+ x);
    }
}

```

- 测试类；

```java
package guava.eventbus.myself;

import org.junit.Test;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：
 */
public class Tests {

    @Test
    public void test() {
        EventBus eventBus = new EventBus("test-bus", (cause, context) -> {
            System.out.println("--------------------------------------------");
            cause.printStackTrace();
            System.out.println(context.getSource());
            System.out.println(context.getSubscribe());
            System.out.println(context.getSubscriber());
            System.out.println(context.getEvent());
            System.out.println("--------------------------------------------");
        });
        eventBus.registry(new SampleListener());
        eventBus.post("dxgk_v1.01","测试一下子dxgk_v1.0");
    }

}

```



## 3 concurrency并发包

### 3.1 Monitor锁

> `Monitor`锁机制，是对synchronized关键字或者Lock锁的封装，采用述说的方式来对锁进行形容，因为很多时候需要while来进行await等事件的监听，但是使用Monitor则将其进行封装；

```java
package guava.monitor;

import com.google.common.util.concurrent.Monitor;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;

/**
 * @author ${张世林}
 * @date 2020/08/09
 * 作用：
 */
public class GuavaLock {
    private Monitor monitor = new Monitor();
    private LinkedList<Long> queue = new LinkedList<>();
    private int QUEUE_MAX_SIZE = 10;
    private final Monitor.Guard CAN_ADD = monitor.newGuard(() -> queue.size() >= QUEUE_MAX_SIZE);
    private final Monitor.Guard CAN_TAKE = monitor.newGuard(() -> !queue.isEmpty());

    public long take() {
        Long last = null;
        try {
            monitor.enterWhen(CAN_TAKE);
            last = queue.getLast();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            monitor.leave();
        }
        return last;
    }

    public void add(long data) {
        try {
            monitor.enterWhen(CAN_ADD, 10, TimeUnit.SECONDS);
            queue.addLast(data);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            monitor.leave();
        }
    }

}

```

### 3.2 RateLimiter

> `RateLimiter`是基于令牌桶算法的一种具体实现，其详细描述为：一秒钟内允许通过的请求数量，也就是一秒钟内产生的令牌数；

```java
/**
* 两秒钟处理一个请求
*/
private RateLimiter rateLimiter = RateLimiter.create(0.5);

/**
* 一秒钟处理四个请求
*/
private RateLimiter rateLimiter = RateLimiter.create(4);
```

## 4. Guava Cache

> ​	`Guava Cache`是一种`Memory Cache`，也称内存缓存； 

### 4.1 LRU算法

> `LRU`算法是：缓存中的数据是使用一种链表或其他形式进行记录的；
>
> - 如果是新增一个缓存key：在缓存中是最新的状态，其他的key依次后移；
> - 如果是使用一个缓存key：将其key在缓存中更新为最新的状态，其他key依次后移；
> - 如果新增缓存时缓存到达最大时：删除状态最后的那个key，实现缓存释放；

### 4.2 LRU实例

> - 方式一：`LinkedHashMap`是一个基于LRU算法的数据结构，可直接使用；
>
> - 方式二：自己使用链表 + Map的方式，实现一个LRU算法；

```java
package guava.cache;

/**
 * 使用LRU算法，实现自定义缓存
 */
public interface LRUCache<K,V> {

    /**
     * 向缓存中添加数据
     * @param key
     * @param value
     */
    void put(K key, V value);

    /**
     * 从缓存中获取key
     *
     * @param key
     * @return
     */
    V get(K key);

    /**
     * 删除key
     * @param key
     */
    void remove(K key);

    /**
     * 获取缓存大小
     *
     * @return
     */
    int size();

    /**
     * 最大元素个数
     * @return
     */
    int limit();

    /**
     * 清空缓存
     */
    void clear();
}

```

- LinkedHashMap实现方式：

```java
package guava.cache;

import com.google.common.base.Preconditions;

import java.util.LinkedHashMap;
import java.util.Map;

/**
 * @author ${张世林}
 * @date 2020/08/14
 * 作用：
 */
public class LinkedHashLRUCache<K, V> implements LRUCache<K, V> {
    /**
     * 此处使用内部类来继承LinkedHashMap，能够实现只对外暴露LRUCache接口中的方法，而不会将LinkedHashMap中其他方法暴露出来
     * @param <K>
     * @param <V>
     */
    private static class InternalLRUCache<K, V> extends LinkedHashMap<K, V> {
        /**
         * 缓存最大容量
         */
        private final int limit;

        public InternalLRUCache(int limit) {
            super(16, 0.75f);
            this.limit = limit;
        }

        /**
         * 当大小大于最大值的时候，将历史数据进行删除
         * @param eldest
         * @return
         */
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return size() > limit;
        }
    }

    private final int limit;

    private final InternalLRUCache<K,V> internalLRUCache;

    public LinkedHashLRUCache(int limit) {
        Preconditions.checkArgument(limit > 0, "limit must more than 0");
        this.limit = limit;
        this.internalLRUCache = new InternalLRUCache<>(limit);
    }

    @Override
    public void put(K key, V value) {
        this.internalLRUCache.put(key, value);
    }

    @Override
    public V get(K key) {
        return this.internalLRUCache.get(key);
    }

    @Override
    public void remove(K key) {
        this.internalLRUCache.remove(key);
    }

    @Override
    public int size() {
        return this.internalLRUCache.size();
    }

    @Override
    public int limit() {
        return this.limit;
    }

    @Override
    public void clear() {
        this.internalLRUCache.clear();
    }
}
```

- 使用链表 + Map的方式实现LRU缓存算法；

```java
package guava.cache;

import com.google.common.base.Preconditions;

import java.util.HashMap;
import java.util.LinkedList;
import java.util.Map;

/**
 * @author ${张世林}
 * @date 2020/08/14
 * 作用：
 */
public class LinkedListLRUCache<K, V> implements LRUCache<K, V> {
    private final int limit;

    private final LinkedList<K> keys = new LinkedList<>();

    private final Map<K, V> cache = new HashMap<>();


    public LinkedListLRUCache(int limit) {
        this.limit = limit;
    }

    @Override
    public void put(K key, V value) {
        K k = Preconditions.checkNotNull(key);
        V v = Preconditions.checkNotNull(value);
        //如果缓存满了，则需要删除最老的数据
        if (this.keys.size() >= this.limit) {
            //最老的数据
            K oldestKey = this.keys.removeFirst();
            this.cache.remove(oldestKey);
        }
        //添加缓存
        this.keys.addLast(k);
        this.cache.put(k, v);
    }

    @Override
    public V get(K key) {
        //从链表中获取是否存在该key
        boolean remove = this.keys.remove(key);
        //如果不存在该key，则直接返回
        if (!remove) {
            return null;
        }
        //如果存在key，根据LRU算法，将该key放置最新位置，然后获取该key并返回
        this.keys.addLast(key);
        return this.cache.get(key);
    }

    @Override
    public void remove(K key) {
        boolean remove = this.keys.remove(key);
        if (remove) {
            this.cache.remove(key);
        }
    }

    @Override
    public int size() {
        return this.keys.size();
    }

    @Override
    public int limit() {
        return this.limit;
    }

    @Override
    public void clear() {
        this.keys.clear();
        this.cache.clear();
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        keys.forEach(key -> builder.append("{key=").append(key).append(", value=").append(cache.get(key)).append("}; "));
        return builder.deleteCharAt(builder.length() - 1).toString();
    }
}

```

### 4.3 Guava Cache

> 类对象包含：
>
> - `Cache`：缓存对象接口
> - `LoadingCache`：缓存的具体实现类；
> - `CacheBuilder`：缓存的建造者对象；
>   - `newBuilder()`：创建一个建造者实例；
>   - `CacheBuilder<K, V> recordStats()`：统计得到Stats对象；
>     - `CacheStats stats = cache.stats();`：需要开启recordStats；
>   - `maximumSize(long maximumSize)`：缓存最大容量；
>   - `expireAfterWrite(long duration, TimeUnit unit)`：当写入缓存以后，过期的时间；这个相当于redis中的TTL；
>   - `expireAfterAccess(long duration, TimeUnit unit)`：当加入缓存中以后，如果超过时间未调用，则设置为过期；
>   - `build(CacheLoader<? super K1, V1> loader)`：指定加载的数据到缓存的方式；

> 注意事项：
>
> - **在Cache中的对象，是引用模式，如果使用key获取到缓存中的对象，并执行修改以后，后续从缓存中拉取数据会导致数据不是最初始的数据； **

> 创建缓存Cache的方式：
>
> - 方式一：使用CacheBuilder构建
>
> ```java
>         LoadingCache<Long, Zdr> cache = CacheBuilder.newBuilder()
>                 .maximumSize(10)
>                 .expireAfterWrite(5, TimeUnit.SECONDS)
>                 .build(new CacheLoader<Long, Zdr>() {
>                     @Override
>                     public Zdr load(Long key) throws Exception {
>                         return findZdr(key);
>                     }
>                 });
> ```
>
> - 方式二：直接使用new CacheLoader方式加载；
>
> ```java
>         CacheLoader<Long, Zdr> cacheLoader = new CacheLoader<Long, Zdr>() {
>             @Override
>             public Zdr load(Long key) throws Exception {
>                 return findZdr(key);
>             }
>         };
> 		LoadingCache<Long, Zdr> cache = CacheBuilder.newBuilder().build(cacheLoader);
> ```
>
> - 方式三：使用Supplier；
>
> ```java
>         long entityId = 500230L;
>         CacheLoader<Object, Zdr> cacheLoader = CacheLoader.from(() -> {
>             return findZdr(entityId);
>         });
>         LoadingCache<Long, Zdr> cache = CacheBuilder.newBuilder().build(cacheLoader);
> ```
>
> - 方式四：使用Function；

## 5. Guava Collections

> - `FluentIterable`：类似于java8中的Stream对象；
> - `Lists`：操作List集合对象的工具类；
>   - `static <B> List<List<B>> cartesianProduct(List<? extends B>... lists)`：创建多个集合的笛卡尔积；
>   - `static <F, T> List<T> transform(List<F> fromList, Function<? super F, ? extends T> function)`：对指定的集合进行转换操作，相当于是stream中的map操作；
>   - `static ImmutableList<Character> charactersOf(String string)`：将字符串进行转换为字符集合对象；
>   - `static <E> ArrayList<E> newArrayListWithCapacity(int initialArraySize)`：创建指定集合大小的集合；
>   - `static <E> ArrayList<E> newArrayListWithExpectedSize(int estimatedSize)`：创建比指定大小大1的集合对象；
>   - `static <T> List<T> reverse(List<T> list)`：将集合进行反转；
>   - `static <E> CopyOnWriteArrayList<E> newCopyOnWriteArrayList()`：创建集合安全的CopyOnWriteList对象；
>   - `static <T> List<List<T>> partition(List<T> list, int size)`：对集合数据进行分区；

### 5.1 List

> `Lists`：操作List集合对象的工具类；
>
> - `static <B> List<List<B>> cartesianProduct(List<? extends B>... lists)`：创建多个集合的笛卡尔积；
> - `static <F, T> List<T> transform(List<F> fromList, Function<? super F, ? extends T> function)`：对指定的集合进行转换操作，相当于是stream中的map操作；
> - `static ImmutableList<Character> charactersOf(String string)`：将字符串进行转换为字符集合对象；
> - `static <E> ArrayList<E> newArrayListWithCapacity(int initialArraySize)`：创建指定集合大小的集合；
> - `static <E> ArrayList<E> newArrayListWithExpectedSize(int estimatedSize)`：创建比指定大小大1的集合对象；
> - `static <T> List<T> reverse(List<T> list)`：将集合进行反转；
> - `static <E> CopyOnWriteArrayList<E> newCopyOnWriteArrayList()`：创建集合安全的CopyOnWriteList对象；
> - `static <T> List<List<T>> partition(List<T> list, int size)`：对集合数据进行分区；

```java
package guava.collections;

import com.google.common.collect.ImmutableList;
import com.google.common.collect.Lists;
import com.google.common.primitives.Ints;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;
import java.util.stream.IntStream;

/**
 * @author ${张世林}
 * @date 2020/08/17
 * 作用：
 */
public class ListsTest {

    /**
     * cartesianProduct 笛卡尔积
     */
    @Test
    public void test笛卡尔积() {
        List<List<Integer>> lists = Lists.cartesianProduct(
                Lists.newArrayList(1, 2),
                Lists.newArrayList(3, 4),
                Lists.newArrayList(5, 6)
        );
        System.out.println(lists);
    }

    /**
     * 对元素进行加工
     */
    @Test
    public void testTransFrom() {
        List<Integer> transform = Lists.transform(Lists.newArrayList(1, 2), data -> {
            return data + data;
        });
        System.out.println(transform);
    }

    /**
     * 将字符串转换为字符
     */
    @Test
    public void testStrToListChar() {
        String str = "afd阿尔山若，她太为难wewer;kdsfjfkj.";

        ImmutableList<Character> characters = Lists.charactersOf(str);
        System.out.println(characters);
    }

    /**
     * 初始化容器集合大小
     */
    @Test
    public void testCapacity() {
        ArrayList<String> list = Lists.<String>newArrayListWithCapacity(10);
        IntStream.rangeClosed(0, 11).forEach(i -> list.add(String.valueOf(i)));
        System.out.println(list);
    }

    @Test
    public void testExpectSize() {
        ArrayList<Object> list = Lists.newArrayListWithExpectedSize(10);
        IntStream.rangeClosed(0,100).forEach(i -> list.add(i));

    }

    /**
     * 反转集合数据
     */
    @Test
    public void testRevert() {
        List<Integer> reverse = Lists.reverse(Lists.newArrayList(1, 2, 4));
        System.out.println(reverse);
    }

    /**
     * 创建线程安全的集合
     */
    @Test
    public void testCopyOnWriteList() {
        CopyOnWriteArrayList<String> objects = Lists.<String>newCopyOnWriteArrayList();
    }

    @Test
    public void testPartition() {
        Lists.partition(Lists.newArrayList(1, 2, 3, 4), 2);
    }

}

```

### 5.2 Sets

> - `Sets`：操作Set对象的工具类；
>   - `static <E> HashSet<E> newHashSet(E... elements)`：创建hashSet；
>   - `static <E> Set<E> newConcurrentHashSet()`：创建ConcurrentHashSet对象；
>   - `static <E> Set<E> newIdentityHashSet()`：创建java8中的IdentitySet；
>   - `static <E> LinkedHashSet<E> newLinkedHashSet()`
>   - `static <E> CopyOnWriteArraySet<E> newCopyOnWriteArraySet()`
>   - `static <B> Set<List<B>> cartesianProduct(Set<? extends B>... sets)`：创建多个set 对象的笛卡尔积；
>   - `static <E> Set<Set<E>> combinations(Set<E> set, final int size)`：创建Set集合对象的指定大小的子集；
>   - `static <E> SetView<E> difference(final Set<E> set1, final Set<?> set2)`：得到两个Set集合的差集；
>   - `static <E> SetView<E> intersection(final Set<E> set1, final Set<?> set2)`：得到两个Set集合的交集；
>   - `static <E> SetView<E> union(final Set<? extends E> set1, final Set<? extends E> set2)`：得到两个集合的并集；

```java
package guava.collections;

import com.google.common.collect.Sets;
import org.junit.Test;

import java.util.*;
import java.util.concurrent.CopyOnWriteArraySet;

/**
 * @author ${张世林}
 * @date 2020/08/17
 * 作用：
 */
public class SetsTest {

    @Test
    public void testCreate() {
        HashSet<Integer> integers = Sets.newHashSet(1, 2, 3, 3);
        System.out.println(integers);

        TreeSet<Comparable> treeSet = Sets.newTreeSet();
        treeSet.add("a");
        treeSet.add("b");

        Set<Object> concurrentHashSet = Sets.newConcurrentHashSet();


        Set<Object> objects = Sets.newIdentityHashSet();

        LinkedHashSet<Object> objects1 = Sets.newLinkedHashSet();
        CopyOnWriteArraySet<Object> objects2 = Sets.newCopyOnWriteArraySet();
    }

    /**
     * Set对象的笛卡尔积
     */
    @Test
    public void testCartesian() {
        Set<List<Integer>> lists = Sets.cartesianProduct(
                Sets.newHashSet(1, 2, 3, 3),
                Sets.newHashSet(1, 5, 3, 3)
        );
        System.out.println(lists);
    }

    /**
     * 针对Set集合，产生该Set集合的子集，可以指定子集的大小；不允许超过Set大小
     */
    @Test
    public void testCombinations() {
        HashSet<Integer> set = Sets.newHashSet(1, 2, 3);
        Set<Set<Integer>> combinations = Sets.combinations(set, 2);
        combinations.forEach(System.out::println);
    }

    /**
     * 求出两个Set集合的差集
     */
    @Test
    public void testDiff() {
        HashSet<Integer> set1 = Sets.newHashSet(1, 2, 3);
        HashSet<Integer> set2 = Sets.newHashSet(1, 5, 2);
        Sets.SetView<Integer> difference = Sets.difference(set1, set2);
        System.out.println(difference);
    }

    /**
     * 两个Set集合的交集
     */
    @Test
    public void testIntersection() {
        HashSet<Integer> set1 = Sets.newHashSet(1, 2, 3);
        HashSet<Integer> set2 = Sets.newHashSet(1, 5, 2);
        Sets.SetView<Integer> intersection = Sets.intersection(set1, set2);
        System.out.println(intersection);
    }

    /**
     * 两个Set集合的并集
     */
    @Test
    public void testIUnion() {
        HashSet<Integer> set1 = Sets.newHashSet(1, 2, 3);
        HashSet<Integer> set2 = Sets.newHashSet(1, 5, 2);
        Sets.SetView<Integer> union = Sets.union(set1, set2);
        System.out.println(union);
    }
}

```

### 5.3 Maps

> `Maps`是操作java中Map数据结构的工具类；
>
> - `static <K, V> HashMap<K, V> newHashMap()`：创建一个HashMap对象；
> - `static <K, V> ImmutableMap<K, V> uniqueIndex(Iterable<V> values, Function<? super V, K> keyFunction)`：创建一个固定大小key的map对象；
> - `static <K, V> Map<K, V> asMap(Set<K> set, Function<? super K, V> function)`：创建一个长度可变的Map对象；
> - `static <K, V1, V2> Map<K, V2> transformValues(Map<K, V1> fromMap, Function<? super V1, V2> function)`：对Map的values进行转化；
> - `static <K, V> Map<K, V> filterKeys(Map<K, V> unfiltered, final Predicate<? super K> keyPredicate)`：对Map的keys进行转化；

### 5.4 Table

> `Table`是类比与表对象，主要作用于存储表对象数据结果，相似`List<Map>`对象；
>
> - `ArrayTable`：数组的Table；
> - `TreeBaseTable`：树状Table；
> - `HashBaseTable`：hash类型Table；
> - `ImmutableTable`：不可变的Table；

### 5.5 Immutable Collections

> `ImmutableCollections`不可变的集合类；
>
> - `static <E> ImmutableList<E> of(E e1, E e2, E e3, ...) `：创建一个指定元素大小的不可变集合对象；
> - `static <E> ImmutableList<E> copyOf(E[] elements)`：复制数组中的数据创建不可变对象；

### 5.6 OrderNatural

