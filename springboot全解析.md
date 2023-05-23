第三章：初始化器解析（万事开难头）

一、系统初始化器介绍

1、类名：ApplicationContextInitializer

2、介绍：Spring容器刷新之前执行的一个回调函数

3、作用：向SpringBoot容器中注册属性

4、使用：继承接口自定义实现



二、SpringFactoriesLoader介绍

1、框架内部使用的通用工厂加载机制

2、从classpath下多个jar包特定的位置读取文件并初始化类

3、文件内容必须是kv形式，即properties类型

4、key是全限定名（抽象类|接口库）、value是实现，多个实现用，分隔

SpringFactoriesLoader作用





三、系统初始化器原理解析

作用：

1、上下文刷新即refresh方法前调用

2、用来编码设置一些属性变量通常用在web环境中

3、可以通过order接口进行排序



四、总结



第四章：监听器解析（眼观六路，耳听八方）

一、监听器模式介绍

![image-20221106111612113](C:\Users\hp\AppData\Roaming\Typora\typora-user-images\image-20221106111612113.png)



二、系统监听器介绍



三、监听事件触发机制



四、自定义监听器实战



五、章节回顾



整合apollo监听：

```
@ApolloConfigChangeListener
    public void onChange(ConfigChangeEvent changeEvent) {
        for (String key : changeEvent.changedKeys()) {
            ConfigChange change = changeEvent.getChange(key);
            System.out.println(String.format("Found change - key: %s, oldValue: %s, newValue: %s, changeType: %s", change.getPropertyName(), change.getOldValue(), change.getNewValue(), change.getChangeType()));
        }
    }
```

