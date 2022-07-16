## ConfigurableApplicationContext解读



#### ConfigurableApplicationContext介绍

ConfigurableApplicationContext是一个可配置的上下文接口，其继承了ApplicationContext, Lifecycle, Closeable这三个接口。Lifecycle, Closeable接口分别提供了生命循环的接口。

- Lifecycle提供了：start()，stop()，isRunning();
- Closeable提供了: close();

