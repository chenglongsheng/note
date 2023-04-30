音视频开发使用C和C++ ndk

# APP工程结构

设置文本

```xml
<TextView
	android:id="@+id/tv_hello"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:text="hello world"/>
```

```java
TextView tv_hello = findViewById(R.id.tv_hello);
tv_hello.setText("你好，世界！");
```

设置文本大小

```xml
```

```java
TextView tv_hello = findViewById(R.id.tv_hello);
tv_hello.setTextSize(30);
```



模块化架构和组件化架构



# Fragment

具有生命周期

必须委托在activity中才能运行



1. 创建一个待处理的fragment
2. 获取FragmentManager，一般都是通过getSupportFragmentManager()
3. 开启一个事务transactign，一般调用fragmentManager的beginTransaction()
4. 使用transaction进行fragment的替换
5. 提交事务



# Handler

## 介绍

在 Android 中程序运行跟 Java 中 main 函数类似，会有一个主线程，而这个主线程上做的事情就是用户的触摸事件视觉反馈，画面改变等等，总之就是跟 UI 有关的，所以也叫 UI 线程。

在 Android 中使用多线程也很简单，就是用 Java 的三种开启线程的方式即可（继承 Thread、实现Runnable、实现 Callable）。

但是在这些线程上都无法进行 UI 操作，但是有一些耗时操作又需要放在子线程中进行，否则就会出现 ANR（Application Not Response）。就需要进行线程切换，比如在下载一段文本逐行显示出来，下载文本运行在子线程，显示文本运行在主线程。

完成这个操作需要使用 Android 的 Handler 机制，像 view.post(runnable)，AsynTask，runOnUiThread(runnable)这个都是使用的 Handler。

## handler组成

### handler

主要干两件事，发送消息和处理消息

### Message

传递信息的消息

### MessageQueue

消息队列（核心）

### Looper

循环线程处理队列中的消息

每个线程中都有各自的 Looper

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```

在 `prepare()`中保存到 ThreadLocal 独一份 Looper

```java
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

在 Looper 构造器中 new 出独一份的 MessageQueue

另外一个核心`loop()`中的

```java
for (;;) {
    if (!loopOnce(me, ident, thresholdOverride)) {
        return;
    }
}
```

在这个死循环中
