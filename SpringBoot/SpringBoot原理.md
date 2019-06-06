[TOC]

# 一、启动类

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * @Author: 98050
 * @Time: 2019-05-02 20:06
 * @Feature:
 */
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

发现特别的地方有两个：

- 注解：@SpringBootApplication
- run方法：SpringApplication.run()

那么分别来研究这两个部分。

# 二、@SpringBootApplication背后的秘密

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.context.TypeExcludeFilter;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.FilterType;
import org.springframework.context.annotation.ComponentScan.Filter;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    Class<?>[] exclude() default {};

    @AliasFor(
        annotation = EnableAutoConfiguration.class
    )
    String[] excludeName() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackages"
    )
    String[] scanBasePackages() default {};

    @AliasFor(
        annotation = ComponentScan.class,
        attribute = "basePackageClasses"
    )
    Class<?>[] scanBasePackageClasses() default {};
}
```

先不去关注接口中定义的属性，先研究以下三个注解：

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
```

这三个注解就是Spring Boot的核心，所以启动器类也可以这样写：

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;

/**
 * @Author: 98050
 * @Time: 2019-05-02 20:06
 * @Feature:
 */
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

每次写这3个比较累，所以写一个@SpringBootApplication方便点。接下来分别介绍这3个Annotation。

## 2.1 @ComponentScan

@ComponentScan这个注解在Spring中很重要，它对应XML配置中的元素，@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。

我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。

## 2.2 @SpringBootConfiguration

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Configuration;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```

其实 `@SpringBootConfiguration`的核心还是`@Configuration`

这里的@Configuration对我们来说不陌生，它就是JavaConfig形式的Spring Ioc容器的配置类使用的那个@Configuration，SpringBoot社区推荐使用基于JavaConfig的配置形式，所以，这里的启动类标注了@Configuration之后，本身其实也是一个IoC容器的配置类。

## 2.3 @EnableAutoConfiguration

@EnableAutoConfiguration这个Annotation最为重要，所以放在最后来解读，大家是否还记得Spring框架提供的各种名字为@Enable开头的Annotation定义？比如@EnableScheduling、@EnableCaching、@EnableMBeanExport等，@EnableAutoConfiguration的理念和做事方式其实一脉相承，简单概括一下就是：**借助@Import的支持，收集和注册特定场景相关的bean定义**。

- @EnableScheduling是通过@Import将Spring调度框架相关的bean定义都加载到IoC容器。
- @EnableMBeanExport是通过@Import将JMX相关的bean定义加载到IoC容器。

**而@EnableAutoConfiguration也是借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IoC容器，仅此而已！**

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Import;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```

两个比较重要的注解：

- @AutoConfigurationPackage：自动配置包
- @Import({AutoConfigurationImportSelector.class}): 导入自动配置的组件

### 2.3.1 @AutoConfigurationPackage

自动配置的包范围

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.boot.autoconfigure;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.boot.autoconfigure.AutoConfigurationPackages.Registrar;
import org.springframework.context.annotation.Import;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
```

通过@Import将`Registrar`加载到Spring IOC容器中

```java
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
    Registrar() {
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        AutoConfigurationPackages.register(registry, (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName());
    }

    public Set<Object> determineImports(AnnotationMetadata metadata) {
        return Collections.singleton(new AutoConfigurationPackages.PackageImport(metadata));
    }
}
```

`(new AutoConfigurationPackages.PackageImport(metadata)).getPackageName()`，返回了当前主程序类的同级以及子级的包组件。

![](http://mycsdnblog.work/201919022131-a.png)

Application与controller、dao、service同级

demo与example同级

**所以当启动Application后，是不会加载demo的。所以在搭建springboot项目时，一定要把启动器放在项目的最高级目录中**。

### 2.3.2 @Import({AutoConfigurationImportSelector.class})

![1556804669001](http://mycsdnblog.work/201919062223-5.png)

**AutoConfigurationImportSelector 实现了** **DeferredImportSelector ，DeferredImportSelector继承了 ImportSelector**

ImportSelector有一个方法为：**selectImports**。在`AutoConfigurationImportSelector`中给出了实现：

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        try {
            AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
            AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
            List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
            configurations = this.removeDuplicates(configurations);
            configurations = this.sort(configurations, autoConfigurationMetadata);
            Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
            this.checkExcludedClasses(configurations, exclusions);
            configurations.removeAll(exclusions);
            configurations = this.filter(configurations, autoConfigurationMetadata);
            this.fireAutoConfigurationImportEvents(configurations, exclusions);
            return StringUtils.toStringArray(configurations);
        } catch (IOException var6) {
            throw new IllegalStateException(var6);
        }
    }
}
```

关注一下这句代码：

```java
List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
```

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

```java
public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
    String factoryClassName = factoryClass.getName();
    return (List)loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
}
```

```java
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    MultiValueMap<String, String> result = (MultiValueMap)cache.get(classLoader);
    if (result != null) {
        return result;
    } else {
        try {
            Enumeration<URL> urls = classLoader != null ? classLoader.getResources("META-INF/spring.factories") : ClassLoader.getSystemResources("META-INF/spring.factories");
            LinkedMultiValueMap result = new LinkedMultiValueMap();

            while(urls.hasMoreElements()) {
                URL url = (URL)urls.nextElement();
                UrlResource resource = new UrlResource(url);
                Properties properties = PropertiesLoaderUtils.loadProperties(resource);
                Iterator var6 = properties.entrySet().iterator();

                while(var6.hasNext()) {
                    Entry<?, ?> entry = (Entry)var6.next();
                    List<String> factoryClassNames = Arrays.asList(StringUtils.commaDelimitedListToStringArray((String)entry.getValue()));
                    result.addAll((String)entry.getKey(), factoryClassNames);
                }
            }

            cache.put(classLoader, result);
            return result;
        } catch (IOException var9) {
            throw new IllegalArgumentException("Unable to load factories from location [META-INF/spring.factories]", var9);
        }
    }
}
```

这个方法的功能其实就是去加载  **"META-INF/spring.factories"**外部文件。

这个外部文件，有很多自动配置的类。如下：

![1556806242329](http://mycsdnblog.work/201919062223-0.png)

**总结：**

通过EnableAutoConfigurationImportSelector，@EnableAutoConfiguration可以帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。就像一只“八爪鱼”一样。

![](http://mycsdnblog.work/201919022213-P.png)

所以SpringFactoriesLoader是非常重要的一个类，下面对它进行分析

### 2.3.3 SpringFactoriesLoader详解

借助于Spring框架原有的一个工具类：SpringFactoriesLoader的支持，@EnableAutoConfiguration可以智能的自动配置功效才得以大功告成！

**SpringFactoriesLoader**属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置。

![1556806882017](http://mycsdnblog.work/201919062223-a.png)

配合@EnableAutoConfiguration使用的话，它更多是提供一种配置查找的功能支持，即根据@EnableAutoConfiguration的完整类名`org.springframework.boot.autoconfigure.EnableAutoConfiguration`作为查找的Key,获取对应的一组@Configuration类

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
    List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
    Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
    return configurations;
}
```

`this.getSpringFactoriesLoaderFactoryClass()`

```java
protected Class<?> getSpringFactoriesLoaderFactoryClass() {
    return EnableAutoConfiguration.class;
}
```

![1556807205726](http://mycsdnblog.work/201919062224-x.png)

上图就是从SpringBoot的autoconfigure依赖包中的META-INF/spring.factories配置文件中摘录的一段内容，可以很好地说明问题。

所以，@EnableAutoConfiguration自动配置的魔法骑士就变成了：**从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。**

## 2.4 默认配置的原理

### 2.4.1 默认配置类

@EnableAutoConfiguration会开启SpringBoot的自动配置，并且根据你引入的依赖来生效对应的默认配置。那么问题来了：

- 这些默认配置是在哪里定义的呢？
- 为何依赖引入就会触发配置呢？

需要注意项目中引入的一个依赖：`spring-boot-autoconfigure`，其中定义了大量自动配置类：

![1556809179729](http://mycsdnblog.work/201919062224-c.png)

![1556809213048](http://mycsdnblog.work/201919062224-E.png)

非常多，几乎涵盖了现在主流的开源框架。来看一个我们熟悉的，例如SpringMVC，查看mvc 的自动配置类：

![1556809282699](http://mycsdnblog.work/201919062225-V.png)

打开WebMvcAutoConfiguration：

![1556809408300](http://mycsdnblog.work/201919062225-y.png)

我们看到这个类上的4个注解：

- `@Configuration`：声明这个类是一个配置类

- `@ConditionalOnWebApplication(type = Type.SERVLET)`

  满足项目的类是是Type.SERVLET类型，也就是一个普通web工程，

- `@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })`

  **这里的条件是OnClass，也就是满足以下类存在：Servlet、DispatcherServlet、WebMvcConfigurer，其中Servlet只要引入了tomcat依赖自然会有，后两个需要引入SpringMVC才会有。这里就是判断你是否引入了相关依赖，引入依赖后该条件成立，当前类的配置才会生效！**

- `@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)`

  这个条件与上面不同，OnMissingBean，是说环境中没有指定的Bean这个才生效。其实这就是自定义配置的入口，也就是说，如果我们自己配置了一个WebMVCConfigurationSupport的类，那么这个默认配置就会失效！

接着，我们查看该类中定义了什么：

> **视图解析器**

![1556809827948](http://mycsdnblog.work/201919062225-J.png)

> **处理器适配器**

![1556809947047](http://mycsdnblog.work/201919062225-Q.png)

### 2.4.2 默认配置属性

另外，这些默认配置的属性来自哪里呢？

![1556810239953](http://mycsdnblog.work/201919062226-0.png)

这里通过@EnableAutoConfiguration注解引入了两个属性：WebMvcProperties和ResourceProperties。这不正是SpringBoot的属性注入玩法嘛。

查看这两个属性类：

**在WebMvcProperties中找到了内部资源视图解析器的prefix和suffix属性。**

![1556810363718](http://mycsdnblog.work/201919062226-O.png)

**ResourceProperties中主要定义了静态资源（.js,.html,.css等)的路径：**

![1556810437699](http://mycsdnblog.work/201919062226-q.png)

如果我们要覆盖这些默认属性，只需要在application.properties中定义与其前缀prefix和字段名一致的属性即可。

## 2.5 总结

SpringBoot为我们提供了默认配置，而默认配置生效的条件一般有两个：

- 你引入了相关依赖
- 你自己没有配置

1）启动器

所以，我们如果不想配置，只需要引入依赖即可，而依赖版本我们也不用操心，因为只要引入了SpringBoot提供的stater（启动器），就会自动管理依赖及版本了。

因此，玩SpringBoot的第一件事情，就是找启动器，SpringBoot提供了大量的默认启动器。

2）全局配置

另外，SpringBoot的默认配置，都会读取默认属性，而这些属性可以通过自定义`application.properties`文件来进行覆盖。这样虽然使用的还是默认配置，但是配置中的值改成了我们自定义的。

因此，玩SpringBoot的第二件事情，就是通过`application.properties`来覆盖默认属性值，形成自定义配置。我们需要知道SpringBoot的默认属性key，非常多。

最后通过一个简单的流程图总结一下@SpringBootApplication的功能。

![](http://mycsdnblog.work/201919022255-u.png)

# 三、SpringApplication.run执行流程

从SpringApplication的run方法的开始探究执行流程：

## 3.1 构造SpringApplication实例

如果我们使用的是SpringApplication的静态run方法，那么，这个方法里面首先要创建一个SpringApplication对象实例，然后调用这个创建好的SpringApplication的实例方法。

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    return (new SpringApplication(primarySources)).run(args);
}
```

在SpringApplication实例初始化的时候，它会提前做几件事情：

```java
public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) {
    this.sources = new LinkedHashSet();
    this.bannerMode = Mode.CONSOLE;
    this.logStartupInfo = true;
    this.addCommandLineProperties = true;
    this.addConversionService = true;
    this.headless = true;
    this.registerShutdownHook = true;
    this.additionalProfiles = new HashSet();
    this.isCustomEnvironment = false;
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class));
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = this.deduceMainApplicationClass();
}
```

　从源码上来看，主要是deduceFromClasspath()；getSpringFactoriesInstances(xxx.class)；deduceMainApplicationClass()；这三个方法，我们一个一个来看。

1、deduceFromClasspath

```java
static WebApplicationType deduceFromClasspath() {
    if (ClassUtils.isPresent("org.springframework.web.reactive.DispatcherHandler", (ClassLoader)null) && !ClassUtils.isPresent("org.springframework.web.servlet.DispatcherServlet", (ClassLoader)null) && !ClassUtils.isPresent("org.glassfish.jersey.servlet.ServletContainer", (ClassLoader)null)) {
        return REACTIVE;
    } else {
        String[] var0 = SERVLET_INDICATOR_CLASSES;
        int var1 = var0.length;

        for(int var2 = 0; var2 < var1; ++var2) {
            String className = var0[var2];
            if (!ClassUtils.isPresent(className, (ClassLoader)null)) {
                return NONE;
            }
        }

        return SERVLET;
    }
}
// 判断给定的类是否能够加载，就是说类路径下是否存在给定的类
public static boolean isPresent(String className, @Nullable ClassLoader classLoader) {
    try {
        forName(className, classLoader);
        return true;
    } catch (IllegalAccessError var3) {
        throw new IllegalStateException("Readability mismatch in inheritance hierarchy of class [" + className + "]: " + var3.getMessage(), var3);
    } catch (Throwable var4) {
        return false;
    }
}
```

如果org.springframework.web.reactive.DispatcherHandler能够被加载且org.springframework.web.servlet.DispatcherServlet不能够被加载，那么断定web应用类型是REACTIVE；如果javax.servlet.Servlet和org.springframework.web.context.ConfigurableWebApplicationContext任意一个不能被加载，那么断定web应用类型是NONE；如果不能断定是REACTIVE和NONE，那么就是SERVLET类型；三种类型的含义：

NONE：不需要web容器的环境下运行，也就是普通工程

SERVLET：基于servlet的web项目

REACTIVE：响应式web应用，reactive web是Spring5版本的新特性

## 3.2 调用SpringApplication.run方法

```java
public ConfigurableApplicationContext run(String... args) {
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList();
    this.configureHeadlessProperty();
    SpringApplicationRunListeners listeners = this.getRunListeners(args);
    listeners.starting();

    Collection exceptionReporters;
    try {
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments);
        this.configureIgnoreBeanInfo(environment);
        Banner printedBanner = this.printBanner(environment);
        context = this.createApplicationContext();
        exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context);
        this.prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        this.refreshContext(context);
        this.afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch);
        }

        listeners.started(context);
        this.callRunners(context, applicationArguments);
    } catch (Throwable var10) {
        this.handleRunFailure(context, var10, exceptionReporters, listeners);
        throw new IllegalStateException(var10);
    }

    try {
        listeners.running(context);
        return context;
    } catch (Throwable var9) {
        this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null);
        throw new IllegalStateException(var9);
    }
}
```

接下来进行具体方法分析

- **StopWatch类**：计时类，计算SpringBoot应用的启动时间。

- **SpringBootExceptionReporter类**：是一个回调接口，用于支持SpringApplication启动错误的自定义报告。

- **configureHeadlessProperty()方法**：配置Headless模式配置，该模式下系统缺少显示设备、鼠标或键盘，而服务器端往往需要在该模式下工作。

### 3.2.1 getRunListeners(args)方法

获取SpringApplicationRunListeners类，是SpringApplicationRunListener类的集合。

```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
    Class<?>[] types = new Class[]{SpringApplication.class, String[].class};
    return new SpringApplicationRunListeners(logger, this.getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
}
```

通过ClassLoader.getResources加载META-INF/spring.factories路径下的文件信息，从中找key为SpringApplicationRunListener对应类，并实例化。

### 3.2.2 starting()方法

```java
public void starting() {
    Iterator var1 = this.listeners.iterator();

    while(var1.hasNext()) {
        SpringApplicationRunListener listener = (SpringApplicationRunListener)var1.next();
        listener.starting();
    }

}
```

发布ApplicationStartedEvent事件。

- **DefaultApplicationArguments(**args**)类**：提供访问运行一个SpringApplication的arguments的访问入口。

### 3.2.3 prepareEnvironment(listeners,applicationArguments)方法

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments) {
    //1.得到环境对象ConfigurableEnvironment，没有则创建一个
    ConfigurableEnvironment environment = this.getOrCreateEnvironment();
    //2.配置环境信息（激活环境，通过从系统环境变量里取）
    this.configureEnvironment((ConfigurableEnvironment)environment, applicationArguments.getSourceArgs());
    //3.发布ApplicationEnvironmentPreparedEvent事件，加载配置文件
    listeners.environmentPrepared((ConfigurableEnvironment)environment);
    this.bindToSpringApplication((ConfigurableEnvironment)environment);
    if (!this.isCustomEnvironment) {
        environment = (new EnvironmentConverter(this.getClassLoader())).convertEnvironmentIfNecessary((ConfigurableEnvironment)environment, this.deduceEnvironmentClass());
    }

    ConfigurationPropertySources.attach((Environment)environment);
    return (ConfigurableEnvironment)environment;
}
```

创建和配置Environment，同时在调用AbstractEnvironment构造函数时生成PropertySourcesPropertyResolver类。

1、根据`webApplicationType`来创建相应的环境，默认创建`StandardEnvironment()`。

```java
private ConfigurableEnvironment getOrCreateEnvironment() {
    if (this.environment != null) {
        return this.environment;
    } else {
        switch(this.webApplicationType) {
        case SERVLET:
            return new StandardServletEnvironment();
        case REACTIVE:
            return new StandardReactiveWebEnvironment();
        default:
            return new StandardEnvironment();
        }
    }
}
```

2、配置环境信息

```java
protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
    if (this.addConversionService) {
        ConversionService conversionService = ApplicationConversionService.getSharedInstance();
        environment.setConversionService((ConfigurableConversionService)conversionService);
    }

    this.configurePropertySources(environment, args);
    // 配置ConfigurableEnvironment中的激活属性
    this.configureProfiles(environment, args);
}
```

先设置类型转换的服务接口，然后再配置环境信息

```java
protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
    MutablePropertySources sources = environment.getPropertySources();
    if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
        sources.addLast(new MapPropertySource("defaultProperties", this.defaultProperties));
    }

    if (this.addCommandLineProperties && args.length > 0) {
        String name = "commandLineArgs";
        if (sources.contains(name)) {
            PropertySource<?> source = sources.get(name);
            CompositePropertySource composite = new CompositePropertySource(name);
            composite.addPropertySource(new SimpleCommandLinePropertySource("springApplicationCommandLineArgs", args));
            composite.addPropertySource(source);
            sources.replace(name, composite);
        } else {
            sources.addFirst(new SimpleCommandLinePropertySource(args));
        }
    }

}

protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
    environment.getActiveProfiles();
    // additionalProfiles是项目启动时在main中SpringApplication.setAdditionalProfiles("")配置的
    Set<String> profiles = new LinkedHashSet(this.additionalProfiles);
    // 获取环境变量中设置的spring.profiles.active属性
    profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
    // 赋值 activeProfiles
    environment.setActiveProfiles(StringUtils.toStringArray(profiles));
}
```

3、发布ApplicationEnvironmentPreparedEvent事件，加载Spring配置文件信息，例如application.properties

```java
listeners.environmentPrepared((ConfigurableEnvironment)environment);
```

该事件会在ConfigFileApplicationListener监听器中处理，具体请看ConfigFileApplicationListener类，最终会调用ConfigFileApplicationListener中的内部类Loader加载配置信息。

![1557136223672](http://mycsdnblog.work/201919062227-8.png)

```java
public void load() {
	this.profiles = new LinkedList<>();
	this.processedProfiles = new LinkedList<>();
	this.activatedProfiles = false;
	this.loaded = new LinkedHashMap<>();
	// 初始化默认激活环境
	initializeProfiles();
	while (!this.profiles.isEmpty()) {
		Profile profile = this.profiles.poll();
		if (profile != null && !profile.isDefaultProfile()) {
			addProfileToEnvironment(profile.getName());
		}
		// 载入配置文件
		load(profile, this::getPositiveProfileFilter,
				addToLoaded(MutablePropertySources::addLast, false));
		this.processedProfiles.add(profile);
	}
	resetEnvironmentProfiles(this.processedProfiles);
	load(null, this::getNegativeProfileFilter,
	addToLoaded(MutablePropertySources::addFirst, true));
	addLoadedPropertySources();
}


private void initializeProfiles() {
	// The default profile for these purposes is represented as null. We add it
	// first so that it is processed first and has lowest priority.
	// 增加一个null的环境变量
	this.profiles.add(null);
	// 从当前系统环境变量里获取激活的环境
	Set<Profile> activatedViaProperty = getProfilesActivatedViaProperty();
   // 获取排除系统环境变量里的激活环境信息设置SpringApplication.setAdditionalProfiles("");
	this.profiles.addAll(getOtherActiveProfiles(activatedViaProperty));
	// 合并以上得到的激活环境信息，可以看出来从系统环境变量得到的会在之后加载，所以会覆盖以前的
	addActiveProfiles(activatedViaProperty);
	// 如果profiles长度为1，则设置一个default激活环境
	if (this.profiles.size() == 1) { // only has null profile
		for (String defaultProfileName : this.environment.getDefaultProfiles()) {
			Profile defaultProfile = new Profile(defaultProfileName, true);
			this.profiles.add(defaultProfile);
		}
	}
}


private void load(Profile profile, DocumentFilterFactory filterFactory,DocumentConsumer consumer) {
	getSearchLocations().forEach((location) -> {
		boolean isFolder = location.endsWith("/");
		// 加载名字
		Set<String> names = isFolder ? getSearchNames() : NO_SEARCH_NAMES;
		names.forEach(
				(name) -> load(location, name, profile, filterFactory, consumer));
	});
}
private Set<String> getSearchLocations() {
    // 如果是环境变量中配置的有spring.config.location，则直接返回该地址
    if (this.environment.containsProperty("spring.config.location")) {
                return this.getSearchLocations("spring.config.location");
            } else {
        // 如果是环境变量中没有配置spring.config.location，则先从spring.config.additional-location中加载，再从默认中加载
                Set<String> locations = this.getSearchLocations("spring.config.additional-location");
                locations.addAll(this.asResolvedSet(ConfigFileApplicationListener.this.searchLocations, "classpath:/,classpath:/config/,file:./,file:./config/"));
                return locations;
            }
}
private Set<String> getSearchNames() {
  // 如果配置了spring.config.name直接返回，不在启用默认的
	if (this.environment.containsProperty("spring.config.name")) {
                String property = this.environment.getProperty("spring.config.name");
                return this.asResolvedSet(property, (String)null);
            } else {
                return this.asResolvedSet(ConfigFileApplicationListener.this.names, "application");
            }
}
```

SpringApplication从application.properties以下位置的文件加载属性并将它们添加到Spring Environment：

1. 一个/config当前目录的子目录
2. 当前目录当前目录
3. 一个类路径/config包
4. 类路径根类路径根

- **configureIgnoreBeanInfo(**environment**)**方法：配置系统IgnoreBeanInfo属性。

- **printBanner(environment)方法**：打印启动的图形，返回Banner接口实现类。

### 3.2.4 createApplicationContext()方法

根据SpringApplication构造方法生成的webApplicationType变量创建一个ApplicationContext，默认生成**AnnotationConfigApplicationContext**。

```java
protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
        try {
            switch(this.webApplicationType) {
            case SERVLET:
                contextClass = Class.forName("org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext");
                break;
            case REACTIVE:
                contextClass = Class.forName("org.springframework.boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext");
                break;
            default:
                contextClass = Class.forName("org.springframework.context.annotation.AnnotationConfigApplicationContext");
            }
        } catch (ClassNotFoundException var3) {
            throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", var3);
        }
    }

    return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
}
```

（1）时序图

![1557145511355](http://mycsdnblog.work/201919062227-7.png)

（2）分析

a. 方法根据applicationContextClass变量是否为空，决定是否执行下面的switch语句，而applicationContextClass变量是由SpringApplicationBuilder类中的contextClass()方法调用ApplicationContext的setApplicationContextClass()赋值的，默认为空；

b. webApplicationType变量是SpringApplication.run()方法调用构造方法时赋值的;

c. switch语句通过反射根据webApplicationType生成对应的容器，分别是AnnotationConfigServletWebServerApplicationContext、AnnotationConfigReactiveWebServerApplicationContext以及AnnotationConfigApplicationContext的，默认生成的是AnnotationConfigApplicationContext，同时，其构造函数生成AnnotedBeanDefinitonReader和ClassPathBeanDefinitionScanner类;

d. 最后，通过BeanUtils工具类将获取到的容器类转换成ConfigurableApplicationContext类，返回给应用使用。

- **getSpringFactoriesInstances(**SpringBootExceptionRepoter.class, new Class[] {ConfigurableApplicationContext.class }, context**)**方法：获取SpringBootExceptionReporter类的集合。

### 3.2.5 prepareContext()方法

```java
private void prepareContext(ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
	// ⑴.对ApplicationContext设置环境变量；
	context.setEnvironment(environment);
	// ⑵.配置属性ResourceLoader和ClassLoader属性；
	postProcessApplicationContext(context);
	// ⑶.循环初始化继承ApplicationContextInitializer接口的类
	applyInitializers(context);
	listeners.contextPrepared(context);
	if (this.logStartupInfo) {
		logStartupInfo(context.getParent() == null);
		logStartupProfileInfo(context);
	}

	// Add boot specific singleton beans
	context.getBeanFactory().registerSingleton("springApplicationArguments",
			applicationArguments);
	if (printedBanner != null) {
		context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
	}

	// Load the sources
	Set<Object> sources = getSources();
	Assert.notEmpty(sources, "Sources must not be empty");
	load(context, sources.toArray(new Object[sources.size()]));
	listeners.contextLoaded(context);
}

@Override
public void setEnvironment(ConfigurableEnvironment environment) {
	super.setEnvironment(environment);
	this.reader.setEnvironment(environment);
	this.scanner.setEnvironment(environment);
}

protected void applyInitializers(ConfigurableApplicationContext context) {
    Iterator var2 = this.getInitializers().iterator();

    while(var2.hasNext()) {
        ApplicationContextInitializer initializer = (ApplicationContextInitializer)var2.next();
        Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(), ApplicationContextInitializer.class);
        Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
        initializer.initialize(context);
    }

}
```

设置Environment，在ApplicationContext中应用所有相关的后处理，在刷新之前将所有的ApplicationContextInitializers应用于上下文，设置SpringApplicationRunLIstener接口实现类实现多路广播Spring事件，添加引导特定的单例（SpringApplicationArguments, Banner），创建DefaultListableBeanFactory工厂类，从主类中定位资源并将资源中的bean加载进入ApplicationContext中，向ApplicationContext中添加ApplicationListener接口实现类。

### 3.2.6 refreshContext(context)方法

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized(this.startupShutdownMonitor) {
        //1.准刷新上下文环境
        this.prepareRefresh();
        //2.初始化BeanFactory
        ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
        //3.对BeanFactory进行各种填充
        this.prepareBeanFactory(beanFactory);

        try {
            //4.子类覆盖方法做额外的处理，这里会调用子类AnnotationConfigServletWebServerApplicationContext注入
            this.postProcessBeanFactory(beanFactory);
            //5.激活各种BeanFactory处理器
            this.invokeBeanFactoryPostProcessors(beanFactory);
            //6.注册拦截Bean创建的Bean处理，这里只是注册、真正调用的时候是在拿Bean的时候
            this.registerBeanPostProcessors(beanFactory);
            //7.为上下文初始化Message源，即不同语言的消息体，国际化处理
            this.initMessageSource();
            //8.初始化应用消息广播器，并放到applicationEventMulticaster bean中
            this.initApplicationEventMulticaster();
            //9.留给子类来初始化其他bean
            this.onRefresh();
            //10.在所有注册的bean中查找Listener bean，注册到消息广播中
            this.registerListeners();
            //11.初始化剩下的单实例（非惰性）
            this.finishBeanFactoryInitialization(beanFactory);
            //12.完成刷新过程，通知生命周期处理器lifecycleProcessor刷新过程，同时发出ContextRefreshEvent通知别人
            this.finishRefresh();
        } catch (BeansException var9) {
            if (this.logger.isWarnEnabled()) {
                this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
            }

            this.destroyBeans();
            this.cancelRefresh(var9);
            throw var9;
        } finally {
            this.resetCommonCaches();
        }

    }
}
```

调用AbstractApplicationContext的refresh()方法初始化DefaultListableBeanFactory工厂类。

重要方法分析：

**1、this.obtainFreshBeanFactory()**

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    this.refreshBeanFactory();
    return this.getBeanFactory();
}
//实现
protected final void refreshBeanFactory() throws BeansException {
        if (this.hasBeanFactory()) {
            this.destroyBeans();
            this.closeBeanFactory();
        }

        try {
            DefaultListableBeanFactory beanFactory = this.createBeanFactory();
            beanFactory.setSerializationId(this.getId());
            this.customizeBeanFactory(beanFactory);
            this.loadBeanDefinitions(beanFactory);
            synchronized(this.beanFactoryMonitor) {
                this.beanFactory = beanFactory;
            }
        } catch (IOException var5) {
            throw new ApplicationContextException("I/O error parsing bean definition source for " + this.getDisplayName(), var5);
        }
    }
//创建BeanFactory
protected DefaultListableBeanFactory createBeanFactory() {
        return new DefaultListableBeanFactory(this.getInternalParentBeanFactory());
    }
//获取顶级BeanFactory
@Nullable
    protected BeanFactory getInternalParentBeanFactory() {
        //将创建好的上下文对象ConfigurableApplicationContext转换成BeanFactory类型返回
        return (BeanFactory)(this.getParent() instanceof ConfigurableApplicationContext ? ((ConfigurableApplicationContext)this.getParent()).getBeanFactory() : this.getParent());
    }
//获取BeanFactory
public final ConfigurableListableBeanFactory getBeanFactory() {
    synchronized(this.beanFactoryMonitor) {
        if (this.beanFactory == null) {
            throw new IllegalStateException("BeanFactory not initialized or already closed - call 'refresh' before accessing beans via the ApplicationContext");
        } else {
            return this.beanFactory;
        }
    }
}
```

- 持有一个工厂类单例（默认为DefaultListableBeanFactory），并依靠调用者通过工厂类单例或构造方法注册bean；
- 将在AbstarctApplicationContext类生成的id和在DefaultListableBeanFatory生成指向自身的弱引用，存储进入一个由DefaultListableBeanFactory持有的Map中。

**2、this.postProcessBeanFactory(beanFactory);**

```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
}
```

这里会调用子类**AnnotationConfigServletWebServerApplicationContext**注入

```java
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    super.postProcessBeanFactory(beanFactory);
    if (this.basePackages != null && this.basePackages.length > 0) {
        this.scanner.scan(this.basePackages);
    }

    if (!this.annotatedClasses.isEmpty()) {
        this.reader.register(ClassUtils.toClassArray(this.annotatedClasses));
    }
}
//父类中的方法
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 添加后置处理器，在创建Tomcat时会利用这个后置处理器来初始化Tomcat Server类
    beanFactory.addBeanPostProcessor(new WebApplicationContextServletContextAwareProcessor(this));
    beanFactory.ignoreDependencyInterface(ServletContextAware.class);
    this.registerWebApplicationScopes();
}
```

添加后置处理器，在创建Tomcat时会利用这个后置处理器来初始化Tomcat Server类

**3、onRefresh()**

根据类的关系ServletWebServerApplicationContext->AbstractApplicationContext，在ServletWebServerApplicationContext实现了onRefresh方法

```java
protected void onRefresh() {
    super.onRefresh();

    try {
        //创建WebServer容器
        this.createWebServer();
    } catch (Throwable var2) {
        throw new ApplicationContextException("Unable to start web server", var2);
    }
}
```

**主要讲解内置tomcat是什么时候被初始化**

```java
private void createWebServer() {
    WebServer webServer = this.webServer;
    ServletContext servletContext = this.getServletContext();
    if (webServer == null && servletContext == null) {
        ServletWebServerFactory factory = this.getWebServerFactory();
        this.webServer = factory.getWebServer(new ServletContextInitializer[]{this.getSelfInitializer()});
    } else if (servletContext != null) {
        try {
            this.getSelfInitializer().onStartup(servletContext);
        } catch (ServletException var4) {
            throw new ApplicationContextException("Cannot initialize servlet context", var4);
        }
    }

    this.initPropertySources();
}
```

如果webServer和servletContext都为空的时候，那么就对webServer进行初始化，先拿到容器：

```java
protected ServletWebServerFactory getWebServerFactory() {
    String[] beanNames = this.getBeanFactory().getBeanNamesForType(ServletWebServerFactory.class);
    if (beanNames.length == 0) {
        throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to missing ServletWebServerFactory bean.");
    } else if (beanNames.length > 1) {
        throw new ApplicationContextException("Unable to start ServletWebServerApplicationContext due to multiple ServletWebServerFactory beans : " + StringUtils.arrayToCommaDelimitedString(beanNames));
    } else {
        return (ServletWebServerFactory)this.getBeanFactory().getBean(beanNames[0], ServletWebServerFactory.class);
    }
}
```

然后再调用getWebServer方法，它有三个不同的实现：

![](http://mycsdnblog.work/201919062131-Y.png)

进入TomcatServletWebServerFactory中：

```java
public WebServer getWebServer(ServletContextInitializer... initializers) {
    Tomcat tomcat = new Tomcat();
    File baseDir = this.baseDirectory != null ? this.baseDirectory : this.createTempDir("tomcat");
    tomcat.setBaseDir(baseDir.getAbsolutePath());
    Connector connector = new Connector(this.protocol);
    tomcat.getService().addConnector(connector);
    this.customizeConnector(connector);
    tomcat.setConnector(connector);
    tomcat.getHost().setAutoDeploy(false);
    this.configureEngine(tomcat.getEngine());
    Iterator var5 = this.additionalTomcatConnectors.iterator();

    while(var5.hasNext()) {
        Connector additionalConnector = (Connector)var5.next();
        tomcat.getService().addConnector(additionalConnector);
    }

    this.prepareContext(tomcat.getHost(), initializers);
    return this.getTomcatWebServer(tomcat);
}
```

重要方法分析：

**1、this.customizeConnector(connector)**

```java
protected void customizeConnector(Connector connector) {
    int port = this.getPort() >= 0 ? this.getPort() : 0;
    connector.setPort(port);
    if (StringUtils.hasText(this.getServerHeader())) {
        connector.setAttribute("server", this.getServerHeader());
    }

    if (connector.getProtocolHandler() instanceof AbstractProtocol) {
        this.customizeProtocol((AbstractProtocol)connector.getProtocolHandler());
    }

    if (this.getUriEncoding() != null) {
        connector.setURIEncoding(this.getUriEncoding().name());
    }

    connector.setProperty("bindOnInit", "false");
    if (this.getSsl() != null && this.getSsl().isEnabled()) {
        this.customizeSsl(connector);
    }

    TomcatConnectorCustomizer compression = new CompressionConnectorCustomizer(this.getCompression());
    compression.customize(connector);
    Iterator var4 = this.tomcatConnectorCustomizers.iterator();

    while(var4.hasNext()) {
        TomcatConnectorCustomizer customizer = (TomcatConnectorCustomizer)var4.next();
        customizer.customize(connector);
    }

}
```


- **afterRefresh(**CongigurableApplicationContext context, ApplicationArguments args**)**方法：在刷新ApplicationContext之后调用，在SpringAppliation中是方法体为空的函数，故不做任何操作。

- **listeners.started(**context**)**方法：在ApplicationContext已经刷新及启动后，但CommandLineRunners和ApplicationRunner还没有启动时，调用该方法向容器中发布SpringApplicationEvent类或者子类。

- **callRunners(**context, applicationArguments**)方法**：调用应用中ApplicationRunner和CommanLineRunner的实现类，执行其run方法，调用时机是容器启动完成之后，可以用@Order注解来配置Runner的执行顺序，可以用来读取配置文件或连接数据库等操作。

- **listeners.running(**context**)**方法：在容器刷新以及所有的Runner被调用之后，run方法完成执行之前调用该方法。调用之前得到的SpringApplicationRunListeners类running(context)方法。

- 最后，向应用中返回一个之前获得到的ApplicationContext。



