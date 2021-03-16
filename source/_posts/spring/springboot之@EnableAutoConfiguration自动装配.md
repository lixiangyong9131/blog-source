---
title: springboot之@EnableAutoConfiguration自动装配
tags: springboot
categories: springboot
cover: true
comments: true
pin: false
---


## springboot是如何自动装配的？

从springboot启动类只有一个注解@SpringBootApplication。直接查看@SpringBootApplication代码：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};
  
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};
}
```

可以看出@SpringBootApplication：包含了@SpringBootConfiguration（打开是@Configuration），@EnableAutoConfiguration，@ComponentScan注解。

## @Configuration

即JavaConfig形式的spring ioc容器的配置类。

任何一个标注了@Configuration的Java类定义都是一个JavaConfig配置类。

任何一个标注了@Bean的方法，其返回值将作为一个bean定义注册到Spring的IoC容器，方法名将默认成该bean定义的id。

## @ComponentScan

@ComponentScan对应XML配置中的元素，@ComponentScan的功能其实就是自动扫描并加载符合条件的组件（比如@Component和@Repository等）或者bean定义，最终将这些bean定义加载到IoC容器中。
我们可以通过basePackages等属性来细粒度的定制@ComponentScan自动扫描的范围，如果不指定，则默认Spring框架实现会从声明@ComponentScan所在类的package进行扫描。
注：所以SpringBoot的启动类最好是放在root package下，因为默认不指定basePackages。

## @EnableAutoConfiguration（自动装配核心）

英文意思就是自动配置，概括一下就是，借助@Import的帮助，将所有符合自动配置条件的bean定义加载到IOC容器

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
	Class<?>[] exclude() default {};
	String[] excludeName() default {};

}
```

里面最关键的是@Import(EnableAutoConfigurationImportSelector.class)，此Selector，帮助SpringBoot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IOC容器。
该配置模块主要使用到了SpringFactoriesLoader。

## SpringFactoriesLoader

SpringFactoriesLoader属于Spring框架私有的一种扩展方案(类似于Java的SPI方案java.util.ServiceLoader)，是Spring工厂加载器，加载所有META-INF/spring.factories文件中的工厂类。
包含三个静态方法：

- loadFactories：加载指定的factoryClass并进行实例化。
- loadFactoryNames：加载指定的factoryClass的名称集合。
- instantiateFactory：对指定的factoryClass进行实例化。

loadFactories方法首先获取类加载器，然后调用loadFactoryNames方法获取所有的指定资源的名称集合、接着调用instantiateFactory方法实例化这些资源类并将其添加到result集合中。最后调用AnnotationAwareOrderComparator.sort方法进行集合的排序。

## AutoConfigurationImportSelector

```java
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
        AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
        List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
        configurations = this.removeDuplicates(configurations);
        Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
        this.checkExcludedClasses(configurations, exclusions);
        configurations.removeAll(exclusions);
        configurations = this.filter(configurations, autoConfigurationMetadata);
        this.fireAutoConfigurationImportEvents(configurations, exclusions);
        return StringUtils.toStringArray(configurations);
    }
```

该方法在springboot启动流程—bean实例化前被执行，返回要实例化的类信息列表。并调用了SpringFactoriesLoader.loadFactoryNames()方法实例化spring.factories文件中的工厂，最终通过各自的工厂创建组件所需bean。

下图有助于我们形象理解自动配置流程

{% image https://cdn.jsdelivr.net/gh/lixiangyong9131/static/picture/blog/spring-autoconfig.jpg, width=400px,height=200px %}

