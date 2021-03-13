---
title: mockito单元测试mock对象方式
tags: junit
categories: junit
cover: true
comments: true
pin: false
---

## 介绍

单元测试使用Mockito获取不易构造或者对外部环境有依赖的对象，也就是mock一个虚拟对象解决对外部接口的依赖，保证自己的流程逻辑是通的。

## springboot方式

1. maven引入

   ```java
   <dependency>
     <groupId>org.mockito</groupId>
     <artifactId>mockito-all</artifactId>
   	<version>1.9.5</version>
   </dependency>
   ```

2. test类引入：import static org.mockito.Mockito.*;

3. 加入注解，见图

   @Mock

   @InjectMocks

   {% image https://cdn.jsdelivr.net/gh/lixiangyong9131/static/picture/blog/mockito1.png, width=500px,height=500px %}

4. 相关说明

   @InjectMocks 标签会自动填充带@Spy和@Mock标签的bean. 

   @Spy 被它代理的bean,默认执行原生方法。但可以在后期mock想要的方法。 

   @Mock 相当于mockito帮助简单的实例化bean，因此无法执行原生方法。适用于整个bean都mock,如DAO。同时可以结合spring一起管理bean，对bean的管理应该是spring先进行一系列的如初始化bean操作，然后mockito会引用spring生成的bean,并对bean里的指定field进行重新注入。以达到实现部分mock功能 。 

## spring方式

1. 后来在Spring项目（非Springboot）中，上面的方法无法使用，研究出如下写法：

2. maven引入

   ```java
   <dependency>
     <groupId>org.mockito</groupId>
     <artifactId>mockito-all</artifactId>
     <version>2.0.2-beta</version>
   </dependency>
   ```

3. 示例如下

   {% image https://cdn.jsdelivr.net/gh/lixiangyong9131/static/picture/blog/mockito2.png, width=500px,height=500px %}

   {% image https://cdn.jsdelivr.net/gh/lixiangyong9131/static/picture/blog/mockito3.png, width=500px,height=500px %}