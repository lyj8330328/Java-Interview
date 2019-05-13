# 一、Spring IoC容器的设计

Inversion of Control

![](http://mycsdnblog.work/201919091647-C.png)

从接口BeanFactory到HierarchicalBeanFactory，再到ConfigurableBeanFactory，是一条主要的BeanFactory设计路径。

第二条接口设计主线是以ApplicationContext应用上下文接口为核心的接口设计

## 1.1 BeanFactory

### 1.1.1 BeanFactory源码

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.beans.factory;

import org.springframework.beans.BeansException;

public interface BeanFactory {
    String FACTORY_BEAN_PREFIX = "&";
	//根据名字获取Bean
    Object getBean(String var1) throws BeansException;

    <T> T getBean(String var1, Class<T> var2) throws BeansException;

    <T> T getBean(Class<T> var1) throws BeansException;

    Object getBean(String var1, Object... var2) throws BeansException;
	//判断容器中是否含有指定名字的Bean
    boolean containsBean(String var1);
	//检查Bean是否是单例的
    boolean isSingleton(String var1) throws NoSuchBeanDefinitionException;
	//检查Bean是否是原型
    boolean isPrototype(String var1) throws NoSuchBeanDefinitionException;
	//用来查询指定了名字的Bean的Class类型是否是特定的Class类型
    boolean isTypeMatch(String var1, Class var2) throws NoSuchBeanDefinitionException;
	//获取指定名字的Bean的Class类型
    Class<?> getType(String var1) throws NoSuchBeanDefinitionException;
	//查询指定了名字的Bean的所有别名，这个别名是在BeanDefinition中定义的
    String[] getAliases(String var1);
}
```

### 1.1.2 BeanFactory容器设计原理

## 1.2 ApplicationContext

### 1.2.1 ApplicationContext源码

### 1.2.2 ApplicationContext容器设计原理

# 二、IoC容器的初始化过程

通过调用refresh()方法来启动整个初始化过程：BeanDefinition的Resouce定位、载入和注册。

## 2.1 BeanDefinition的Resouce定位

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.context.support;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.core.io.FileSystemResource;
import org.springframework.core.io.Resource;

public class FileSystemXmlApplicationContext extends AbstractXmlApplicationContext {
    public FileSystemXmlApplicationContext() {
    }

    public FileSystemXmlApplicationContext(ApplicationContext parent) {
        super(parent);
    }
	//configLocation就是配置文件(BeanDefinition)的路径
    public FileSystemXmlApplicationContext(String configLocation) throws BeansException {
        this(new String[]{configLocation}, true, (ApplicationContext)null);
    }
	//包含多个配置文件
    public FileSystemXmlApplicationContext(String... configLocations) throws BeansException {
        this(configLocations, true, (ApplicationContext)null);
    }
	//允许指定双亲IOC容器
    public FileSystemXmlApplicationContext(String[] configLocations, ApplicationContext parent) throws BeansException {
        this(configLocations, true, parent);
    }

    public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh) throws BeansException {
        this(configLocations, refresh, (ApplicationContext)null);
    }
	//在对象的初始化过程中，调用refresh方法载入BeanDefinition
    public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent) throws BeansException {
        super(parent);
        this.setConfigLocations(configLocations);
        if (refresh) {
            this.refresh();
        }

    }
	//系统中BeanDefinition的路径，在BeanDefinitionReader的loadBeanDefintion中被调用
    protected Resource getResourceByPath(String path) {
        if (path != null && path.startsWith("/")) {
            path = path.substring(1);
        }

        return new FileSystemResource(path);
    }
}
```

