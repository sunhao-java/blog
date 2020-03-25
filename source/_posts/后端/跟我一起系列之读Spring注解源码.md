title: 跟我一起系列之读Spring注解源码
tags: [源码, spring]
date: 2020-03-25
categories: 后端
---

> 读源码系列之Spring注解

<!-- more -->

# 注解的前世今生
1. 注解在JDK1.5才被发明
2. 什么是注解

		用一个词就可以描述注解，那就是元数据，即一种描述数据的数据。所以，可以说注解就是源代码的元数据。比如，下面这段代码：
		// 这里注解就是标注此方法是一个重写方法
		// 如果父类不存在，或者子类方法名写错了、入参写错了，编译器就会报错
		// 否则，就算没有这个注解，编译、运行都是正常的
		@Override
		public String toString() {
		    return "This is String Representation of current object.";
		}
3. 为何会存在`java.io.Serializable`这个空接口？---Java中的语法糖
4. 如何定义一个接口
	- 修饰词：`@interface`
	- @Retention(生命周期)
		- `SOURCE`：只保留在源码中，编译时会被擦除（Override）
		- `CLASS`：注解将会保留到class文件阶段，但是在加载到JVM时会被抛弃
		- `RUNTIME`：注解一直会被保存到运行时
	- @Target(作用域): TYPE/FIELD/METHOD ...
	- @Inherited: 注解是否可以被继承（下面介绍）
	
# Java中的语法糖
- 字符串拼接
- 条件编译
- 断言
- 枚举与Switch语句
- 字符串与Switch语句
- 可变参数
- 自动装箱/拆箱
- 枚举
- 内部类
- 泛型擦除
- 增强for循环
- lambda表达式
- try-with-resources语句
- JDK10的局部变量类型推断

# Spring-Boot中的注解
## EnableXXX的实现原理
首先我们使用`@EnableSwagger2`举例。我们在使用swagger的时候，只需要在启动类或者一些JavaConfig的配置文件上加上这个注解，就可以使用swagger了。思考一下，这是如何实现的。
1. 看`@EnableSwagger2`的源码
```java
@Retention(value = java.lang.annotation.RetentionPolicy.RUNTIME)
@Target(value = { java.lang.annotation.ElementType.TYPE })
@Documented
@Import({Swagger2DocumentationConfiguration.class})
public @interface EnableSwagger2 {
}
```
有没有发现不太一样的地方？
2. 在这个注解上面还有一个注解，叫`@Import`，这个注解时spring在JavaConfig方式配置时，提供的一个注解，可以引入外部配置
	看代码`org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass#Line303`
3. 通过上面的处理，spring容器就可以拿到swagger的配置`Swagger2DocumentationConfiguration`，加载到上下文中
4. 此时就可以正常使用了
5. 综上，`EnableXXX`基本上是一下逻辑：
	- 提前将你需要的组件的配置以JavaConfig的方式定义好
	- 自定义好一个Enable注解
	- 在Enable注解上，加上@Import注解，参数是定义好的JavaConfig类
	- 然后将该注解加在需要使用组件的工程的启动类上

## ConditionalOnXXX的实现原理
有没有听过spring提供的注解`@org.springframework.context.annotation.Conditional`。@Conditional注解可以根据是否满足某一个特定条件来决定要不要创建某个特定的Bean。比如，当某一个jar包在一个类路径下的时自动配置一个或多个Bean；或者只有某个Bean被创建才会创建另外一个Bean。该注解由Spring4开始提供。
1. `@Conditional`注解中有一个参数value，需要提供一个实现接口`org.springframework.context.annotation.Condition`的类
2. 先看`org.springframework.context.annotation.Condition`，只有一个方法`boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);`
3. 该方法是返回布尔值，也就是说只有当该返回值是true的时候，才会将这个类（有Conditional的类）加载到spring上下文中。
4. 在spring的代码中，由以下步骤调用：
	- ConfigurationClassParser#doProcessConfigurationClass
	- ConditionEvaluator#shouldSkip
	- 获取到类、bean、方法等上面添加的注解，然后循环去执行注解中的`Condition#match`方法，获取到返回值，取并集
	- true的话，就会加载。false的话，就不会加载。

## Profile的实现原理
直接看`@Profile`的源码：
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {
	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();
}
```
1. 说白了也是基于`@Conditional`来实现的
2. 【看源码】
3. 可以发现，spring在上下文环境中，检查这个profile参数
4. 上下文中的参数值来源于启动时，设置的启动参数`spring.profiles.active`传入的
```java
protected Set<String> doGetActiveProfiles() {
		synchronized (this.activeProfiles) {
			if (this.activeProfiles.isEmpty()) {
				// 这里的ACTIVE_PROFILES_PROPERTY_NAME=spring.profiles.active
				String profiles = getProperty(ACTIVE_PROFILES_PROPERTY_NAME);
				if (StringUtils.hasText(profiles)) {
					setActiveProfiles(StringUtils.commaDelimitedListToStringArray(
							StringUtils.trimAllWhitespace(profiles)));
				}
			}
			return this.activeProfiles;
		}
	}
```

## AutoConfigureAfter、AutoConfigureBefore的实现原理
简介：
1. AutoConfigureAfter 声明 当前配置应该在 指定配置之后初始化
2. AutoConfigureBefore 声明 当前配置应该在 指定配置之前初始化

源码：
1. `org.springframework.boot.autoconfigure.AutoConfigurationSorter#getInPriorityOrder`这个方法进行排序
2. 由于这两个方法是由spring-boot定义的，而配置的加载是由spring-framework来执行的，所以并不能影响到spring加载配置的顺序
3. 那这两个类的作用域是哪里呢？看源码，可以看到是从`SpringFactoriesLoader.loadFactoryNames`这个方法取到的配置，进行加载的。【后面有分析】
4. 而这些类都是由spring的spi提供的
5. 所以这两个注解只能用于spring-boot定义的配置【定义在spring.factories中的，后面分析】

## Java的SPI和Spring的SpringFactoriesLoader初探
首先抛出一个问题，我们在使用数据源的时候，只需要把对应数据库的驱动依赖进来，在Java的代码中就可以获取到相应的数据源实现，JDK中，怎么提前知道你这个数据源具体的实现类呢？？？
啥也不说，直接看代码【举例MySQL的驱动包】：
1. 打开源码，找到数据源驱动的具体实现：`com.mysql.jdbc.Driver`
2. 果然是实现`java.sql.Driver`的
3. 那JVM是怎么知道这个类的呢？
4. 看MySQL驱动包下`META-INF/services/java.sql.Driver`文件，该文件中有两行代码（第二行忽略）
	```
	com.mysql.jdbc.Driver
	com.mysql.fabric.jdbc.FabricMySQLDriver
	```
5. 这里果然有驱动的实现类，那就有理由怀疑这个文件在影响到JVM的加载

### Java的 `SPI`
1. 首先，SPI的全称是：Service Provider Interface（Java提供的一套用来被第三方实现或者扩展的API，它可以用来启用框架扩展和替换组件。）
2. 可以先看看Java中怎么通过SPI获取到具体数据源的驱动的：
	```
	java.sql.DriverManager#586
	// 这里去classpath*:META-INF/services/下寻找传入的类名的文件，然后取到该文件中的每一行
	ServiceLoader<Driver> loadedDrivers = ServiceLoader.load(Driver.class);
	Iterator<Driver> driversIterator = loadedDrivers.iterator();
	try{
        while(driversIterator.hasNext()) {
			// 这里的迭代器，会在next方法中对文件中的每一行调用Class.forName进行反射加载到JVM中
            driversIterator.next();
        }
    } catch(Throwable t) {
    // Do nothing
    }
    return null;
	```
3. 所以利用Java的SPI可以这样做：
	- 定义一个接口，用来扩展组件中的一些功能
	- 在组建中，可以使用`ServiceLoader#load`来加载使用者扩展的功能
	- 使用方在扩展该组件时，在`classpath*:META-INF/services/`下以组件接口全路径创建文件
	- 然后将自己扩展的实现该接口的类全路径写入上面的文件中，多个的话，一行一个
	- 应用启动后，就可以将使用方扩展的功能通过SPI的方式加载到JVM中了
4. 已经被弃用的`xny-mybatis`中就使用了Java的SPI来扩展新的接口，可参考[链接](https://gitlab-dmd.oppoer.me/seed/xny-mybatis/blob/master/src/main/java/com/xiniaoyun/mybatis/plugin/BaseDaoInterceptor.java#L70 "链接")
	
### Spring的`SpringFactoriesLoader`
这个主要还是用在`spring-boot-starter`的开发中，只需要引入对应的starter，就可以直接开发了，省去配置的过程（特指JavaConfig的配置），步骤如下：
1. 工程启动类上的注解`@SpringBootApplication`
2. 该注解上的注解`@EnableAutoConfiguration`
3. 这个注解上的注解`@Import(AutoConfigurationImportSelector.class)`
4. 直接调用这里的类的`selectImports`方法
5. 上面的方法中会调用`org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#getCandidateConfigurations`
6. 这时候会有如下一段代码：
	```java
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata,
			AnnotationAttributes attributes) {
		// 通过SpringFactoriesLoader去获取外部定义的配置类
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(
				getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
		Assert.notEmpty(configurations,
				"No auto configuration classes found in META-INF/spring.factories. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

	protected Class<?> getSpringFactoriesLoaderFactoryClass() {
		return EnableAutoConfiguration.class;
	}
	```
7. `SpringFactoriesLoader`与Java的`SPI`区别在于
	- spring的外部定义文件名是`spring.factories`，而Java的是接口全路径
	- spring的文件其实是一个properties文件，而Java的是文本文件
	- spring的话，可以在一个文件中定义多个功能的扩展，而Java就需要N个文件了

## 从spring-boot的自动配置衍生出平台SDK的架构
1. 首先有各个中心的SDK【后面以任务调度中心`platform-tsc-sdk`举例】
2. 另外有一个`platform-sdk-starter`，在这个组件中依赖了各个中心的SDK，只不过都是optional的
	```groovy
    // sdk
    // 消息中心sdk
    compileOnly("com.yezi.platform:platform-mc-sdk:${platform_mc_sdk_version}")
    // 流程中心
    compileOnly("com.yezi.platform:platform-pc-sdk:${platform_pc_sdk_version}")
    // 任务调度中心
    compileOnly("com.yezi.platform:platform-tsc-sdk:${platform_tsc_sdk_version}")
	```
3. 然后定义各个中心的自动配置类，如`TaskSchedulingCenterAutoConfiguration`
	```java
	@Configuration
	@Import(ThreadPoolConfiguration.class)
	@ConditionalOnClass(JobExecuteClient.class)
	public class TaskSchedulingCenterAutoConfiguration {
		// ...
	}
	```
4. 注意上面的`@ConditionalOnClass(JobExecuteClient.class)`，通过对上面spring-boot中的@ConditionalOnXXX的了解，可以判断出这句的意思是：只有当JVM中存在JobExecuteClient这个类的情况下，改配置才会被加载到spring上下文中
5. 而这个类是任务调度中心SDK中特有的一个类
6. 又因为这个SDK在`build.gradle`是`compileOnly`的
7. 所以只有当应用方在他的`build.gradle`中显式的依赖了`com.yezi.platform:platform-tsc-sdk`，任务调度中心的功能才会被启用。