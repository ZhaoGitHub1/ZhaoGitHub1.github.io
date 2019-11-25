---
title: SpringBoot源码分析之---SpringBoot项目启动类SpringApplication浅析
categories: 源码分析
tag: 
  - springboot
  - springboot源码分析
date: 2018-09-06 21:45:44
description: SpringBoot启动过程源码解读
---

** {{ title }}：** <Excerpt in index | 首页摘要>

<!-- more -->
<The rest of contents | 余下全文>

![](springApplication-analyze/words12.jpg)

## 源码版本说明

> 本文源码采用版本为`SpringBoot 2.1.0BUILD`,对应的`SpringFramework 5.1.0.RC1`

> 注意：本文只是从整体上梳理流程，不做具体深入分析

## SpringBoot入口类
``` java
@SpringBootApplication 
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```
　　这是我们日常使用springboot开发见到次数最多的引导类了，完成这个类的编写，就完成了一个springboot项目的框架，springboot就回自动为我们完成一些默认配置，并帮我们初始化上下文容器，但细节我们是不知道的，下面我们就一起探索下`SpringApplication.run(DemoApplication .class, args);`这行代码背后的故事！

## SpringApplication初始化阶段
```java
public SpringApplication(Class<?>... primarySources) {
	this(null, primarySources);
}

// 初始化准备阶段
@SuppressWarnings({ "unchecked", "rawtypes" })
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
	this.resourceLoader = resourceLoader;
	Assert.notNull(primarySources, "PrimarySources must not be null");
	// 参数初始化
	this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
	// 推断应用类型
	this.webApplicationType = deduceWebApplicationType();
	// 加载ApplicationContextInitializer系列初始化器（从spring.factories文件加载，并实例化和排序后存到this.initializers）
	setInitializers((Collection) getSpringFactoriesInstances(
			ApplicationContextInitializer.class));
	// 加载ApplicationListener系列监听器（从spring.factories文件加载，并实例化和排序后存到this.listeners）
	setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
	// 推断应用入口类（main函数所在类）
	this.mainApplicationClass = deduceMainApplicationClass();
}
```
### 推断应用类型
```java
// 推断应用类型
this.webApplicationType = deduceWebApplicationType();
```
```java
private WebApplicationType deduceWebApplicationType() {
	if (ClassUtils.isPresent(REACTIVE_WEB_ENVIRONMENT_CLASS, null)
			&& !ClassUtils.isPresent(MVC_WEB_ENVIRONMENT_CLASS, null)
			&& !ClassUtils.isPresent(JERSEY_WEB_ENVIRONMENT_CLASS, null)) {
		return WebApplicationType.REACTIVE;
	}
	for (String className : WEB_ENVIRONMENT_CLASSES) {
		if (!ClassUtils.isPresent(className, null)) {
			return WebApplicationType.NONE;
		}
	}
	return WebApplicationType.SERVLET;
}
```
```java
private static final String REACTIVE_WEB_ENVIRONMENT_CLASS = "org.springframework."
		+ "web.reactive.DispatcherHandler";

private static final String MVC_WEB_ENVIRONMENT_CLASS = "org.springframework."
		+ "web.servlet.DispatcherServlet";

private static final String JERSEY_WEB_ENVIRONMENT_CLASS = "org.glassfish.jersey.server.ResourceConfig";

private static final String[] WEB_ENVIRONMENT_CLASSES = { "javax.servlet.Servlet",
		"org.springframework.web.context.ConfigurableWebApplicationContext" };
```
根据当前应用ClassPath下是否存在相关类，来确定应用类型。
### 加载`ApplicationContextInitializer`系列初始化器
```java
// 加载ApplicationContextInitializer系列初始化器（从spring.factories文件加载，并实例化和排序后存到this.initializers）
setInitializers((Collection) getSpringFactoriesInstances(
		ApplicationContextInitializer.class));
```
```java
public void setInitializers(
		Collection<? extends ApplicationContextInitializer<?>> initializers) {
	this.initializers = new ArrayList<>();
	this.initializers.addAll(initializers);
}
```
```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
	return getSpringFactoriesInstances(type, new Class<?>[] {});
}
```
```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
		Class<?>[] parameterTypes, Object... args) {
	ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
	// Use names and ensure unique to protect against duplicates
	Set<String> names = new LinkedHashSet<>(
			SpringFactoriesLoader.loadFactoryNames(type, classLoader));
	List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
			classLoader, args, names);
	AnnotationAwareOrderComparator.sort(instances);
	return instances;
}
```
```java
@SuppressWarnings("unchecked")
private <T> List<T> createSpringFactoriesInstances(Class<T> type,
		Class<?>[] parameterTypes, ClassLoader classLoader, Object[] args,
		Set<String> names) {
	List<T> instances = new ArrayList<>(names.size());
	for (String name : names) {
		try {
			Class<?> instanceClass = ClassUtils.forName(name, classLoader);
			Assert.isAssignable(type, instanceClass);
			Constructor<?> constructor = instanceClass
					.getDeclaredConstructor(parameterTypes);
			T instance = (T) BeanUtils.instantiateClass(constructor, args);
			instances.add(instance);
		}
		catch (Throwable ex) {
			throw new IllegalArgumentException(
					"Cannot instantiate " + type + " : " + name, ex);
		}
	}
	return instances;
}
```
　　利用Spring工厂加载机制，实例化ApplicationContextInitializer接口的实现类，被加载的实现类都配置在`MATE-INF/spring.factories`文件中，`getSpringFactoriesInstances(Class<T> type,Class<?>[] parameterTypes, Object... args)`这个方法就负责加载配置类并实例化和排序后返回，后面监听器、异常收集器和Runner等也是通过这个类实现实例化对应实现类的。下面是`spring-boot-autoconfigure\src\main\resources\META-INF\spring.factories`文件的配置内容。
```factories
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.SharedMetadataReaderFactoryContextInitializer,\
org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.autoconfigure.BackgroundPreinitializer

# Auto Configuration Import Listeners
org.springframework.boot.autoconfigure.AutoConfigurationImportListener=\
org.springframework.boot.autoconfigure.condition.ConditionEvaluationReportAutoConfigurationImportListener

# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnClassCondition

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.rest.RestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration

# Failure analyzers
org.springframework.boot.diagnostics.FailureAnalyzer=\
org.springframework.boot.autoconfigure.diagnostics.analyzer.NoSuchBeanDefinitionFailureAnalyzer,\
org.springframework.boot.autoconfigure.jdbc.DataSourceBeanCreationFailureAnalyzer,\
org.springframework.boot.autoconfigure.jdbc.HikariDriverConfigurationFailureAnalyzer,\
org.springframework.boot.autoconfigure.session.NonUniqueSessionRepositoryFailureAnalyzer

# Template availability providers
org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider=\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.mustache.MustacheTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafTemplateAvailabilityProvider,\
org.springframework.boot.autoconfigure.web.servlet.JspTemplateAvailabilityProvider

```
### 加载ApplicationListener系列监听器
```java
// 加载ApplicationListener系列监听器（从spring.factories文件加载，并实例化和排序后存到this.listeners）
setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
```
```java
public void setListeners(Collection<? extends ApplicationListener<?>> listeners) {
	this.listeners = new ArrayList<>();
	this.listeners.addAll(listeners);
}
```

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
	return getSpringFactoriesInstances(type, new Class<?>[] {});
}
```
　　如上面的初始化器，`ApplicationListener`系列监听器也是通过`getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args)`这个方法完成加载、排序及实例化的，而后存入到`this.listeners`中。

### 推断应用入口类
```java
// 推断应用入口类（main函数所在类）
this.mainApplicationClass = deduceMainApplicationClass();
```
```java
private Class<?> deduceMainApplicationClass() {
	try {
		// 通过new一个运行时异常获取堆栈信息
		StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
		for (StackTraceElement stackTraceElement : stackTrace) {
			// 找到main函数所在的入口类
			if ("main".equals(stackTraceElement.getMethodName())) {
				return Class.forName(stackTraceElement.getClassName());
			}
		}
	}
	catch (ClassNotFoundException ex) {
		// Swallow and continue
	}
	return null;
}
```
　　推断应用入口类这部分比较有意思，他是通过new了一个运行时异常来拿到main线程的堆栈信息，遍历所有方法找到main方法所在的类。
## 运行阶段
```java
// 运行阶段
public ConfigurableApplicationContext run(String... args) {
	// 初始化容器启动计时器
	StopWatch stopWatch = new StopWatch();
	// 开始计时
	stopWatch.start();
	// 初始化上下文ConfigurableApplicationContext
	ConfigurableApplicationContext context = null;
	// 初始化异常收集器
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	// 配置系统参数"java.awt.headless"
	configureHeadlessProperty();
	// 获取SpringApplicationRunListener系列监听器（从spring.factories文件加载，并实例化和排序）
	SpringApplicationRunListeners listeners = getRunListeners(args);
	// 遍历所有SpringApplicationRunListener系列监听器，广播ApplicationStartingEvent
	listeners.starting();
	try {
		// 处理args参数
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(
				args);
		// 准备环境（创建、配置、绑定环境、广播ApplicationEnvironmentPreparedEvent）
		ConfigurableEnvironment environment = prepareEnvironment(listeners,
				applicationArguments);
		configureIgnoreBeanInfo(environment);
		// 打印Banner
		Banner printedBanner = printBanner(environment);
		// 根据应用类型创建上下文
		context = createApplicationContext();
		// 获取SpringBootExceptionReporter系列异常收集器（从spring.factories文件加载，并实例化和排序）
		exceptionReporters = getSpringFactoriesInstances(
				SpringBootExceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
		// 上下文前置处理（执行ApplicationContextInitializer系列初始化器、加载资源、广播ApplicationPreparedEvent）
		prepareContext(context, environment, listeners, applicationArguments,
				printedBanner);
		// 刷新上下文（）
		refreshContext(context);
		// 上下文后置处理(目前啥也没干)
		afterRefresh(context, applicationArguments);
		// 启动完成，打印用时
		stopWatch.stop();
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass)
					.logStarted(getApplicationLog(), stopWatch);
		}
		// 遍历前面设置的ConfigurableApplicationContext监听器，发布ApplicationStartedEvent
		listeners.started(context);
		// 按顺序回调实现了ApplicationRunner或CommandLineRunner接口的Runners
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
		// 处理异常（发布ExitCodeEvent和ApplicationFailedEvent事件、异常收集器处理异常）
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
		// 遍历前面设置好的SpringApplicationRunListener监听器，发布ApplicationReadyEvent
		listeners.running(context);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	return context;
}
```
### 获取`SpringApplicationRunListener`系列监听器
```java
// 获取SpringApplicationRunListener系列监听器（从spring.factories文件加载，并实例化和排序）
SpringApplicationRunListeners listeners = getRunListeners(args);
```
```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
	Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
	return new SpringApplicationRunListeners(logger, getSpringFactoriesInstances(
			SpringApplicationRunListener.class, types, this, args));
}
```
　　这里我们再一次见到`getSpringFactoriesInstances(Class<T> type,Class<?>[] parameterTypes, Object... args)`这个方法，功能同上。
### 广播`ApplicationStartingEvent`事件
```java
// 遍历所有SpringApplicationRunListener系列监听器，广播ApplicationStartingEvent
listeners.starting();
```
```java
public void starting() {
	for (SpringApplicationRunListener listener : this.listeners) {
		listener.starting();
	}
}
```
```java
public void starting() {
	this.initialMulticaster.multicastEvent(
			new ApplicationStartingEvent(this.application, this.args));
}
```
　　这里会遍历上面拿到的排序好的所有`SpringApplicationRunListener`系列监听器，广播`ApplicationStartingEvent`事件，这代表Spring应用开始启动，在这之前只进行了注册化初始化器和监听器。
### 准备环境、广播`ApplicationEnvironmentPreparedEvent`事件
```java
// 准备环境（创建、配置、绑定环境、广播ApplicationEnvironmentPreparedEvent）
ConfigurableEnvironment environment = prepareEnvironment(listeners,
		applicationArguments);
```
```java
private ConfigurableEnvironment prepareEnvironment(
		SpringApplicationRunListeners listeners,
		ApplicationArguments applicationArguments) {
	// Create and configure the environment
	ConfigurableEnvironment environment = getOrCreateEnvironment();
	configureEnvironment(environment, applicationArguments.getSourceArgs());
	listeners.environmentPrepared(environment);
	bindToSpringApplication(environment);
	if (this.webApplicationType == WebApplicationType.NONE) {
		environment = new EnvironmentConverter(getClassLoader())
				.convertToStandardEnvironmentIfNecessary(environment);
	}
	ConfigurationPropertySources.attach(environment);
	return environment;
}
```
```java
public void environmentPrepared(ConfigurableEnvironment environment) {
	for (SpringApplicationRunListener listener : this.listeners) {
		listener.environmentPrepared(environment);
	}
}
```

```java
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
	this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(
			this.application, this.args, environment));
}
```
　　这里开始创建、配置和绑定`ConfigurableEnvironment`环境，环境准备好之后开始遍历`SpringApplicationRunListener`系列监听器，广播`ApplicationEnvironmentPreparedEvent`事件，代表环境已准备好。
### 开始打印Banner
```java
// 打印Banner
Banner printedBanner = printBanner(environment);
```
```java
private Banner printBanner(ConfigurableEnvironment environment) {
	// Mode.OFF：不打印banner
	if (this.bannerMode == Banner.Mode.OFF) {
		return null;
	}
	// 加载banner资源，如果自定义了banner样式，在这里加载，否则加载默认banner
	ResourceLoader resourceLoader = (this.resourceLoader != null)
			? this.resourceLoader : new DefaultResourceLoader(getClassLoader());
	// 初始化bannerPrinter
	SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(
			resourceLoader, this.banner);
	// Mode.LOG：通过日志打印banner
	if (this.bannerMode == Mode.LOG) {
		return bannerPrinter.print(environment, this.mainApplicationClass, logger);
	}
	// 默认通过控制台打印banner
	return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}
```
　　关于Banner，SpringBoot支持关闭banner打印、打印到log日志和打印到system日志三种方式；同时支持自定义banner，自定义banner又有图片和txt文本两种（同时存在时先打印图片banner，在打印文本banner），图片banner又支持`gif`, `jpg`, `png`这三种类型的图片格式banner（`git`优先于`jpg`优先于`png`），自定义banner非常简单，只需要将banner文件放到`classpath:`下就好了（`resources`目录下），如果存在多个banner文件，想指定某一个文件，只需要在`application.properties`文件加入如下配置就好了，非常方便。
```propertoties
spring.banner.image.location=banner.png
spring.banner.location=banner.txt
```

### 根据应用类型创建上下文
```java
// 根据应用类型创建上下文
context = createApplicationContext();
```
```java
protected ConfigurableApplicationContext createApplicationContext() {
	Class<?> contextClass = this.applicationContextClass;
	if (contextClass == null) {
		try {
			switch (this.webApplicationType) {
			case SERVLET:
				contextClass = Class.forName(DEFAULT_WEB_CONTEXT_CLASS);
				break;
			case REACTIVE:
				contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
				break;
			default:
				contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
			}
		}
		catch (ClassNotFoundException ex) {
			throw new IllegalStateException(
					"Unable create a default ApplicationContext, "
							+ "please specify an ApplicationContextClass",
					ex);
		}
	}
	return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

```java
public static final String DEFAULT_WEB_CONTEXT_CLASS = "org.springframework.boot."
		+ "web.servlet.context.AnnotationConfigServletWebServerApplicationContext";

public static final String DEFAULT_REACTIVE_WEB_CONTEXT_CLASS = "org.springframework."
		+ "boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext";
		
public static final String DEFAULT_CONTEXT_CLASS = "org.springframework.context."
		+ "annotation.AnnotationConfigApplicationContext";
```
　　开始创建`ConfigurableApplicationContext`上下文，其中`Servlet`类型的web应用会创建`AnnotationConfigServletWebServerApplicationContext`类型的上下文，`Reactive`类型的web应用会创建`AnnotationConfigReactiveWebServerApplicationContext`类型的上下文，非web应用会创建`AnnotationConfigApplicationContext`类型的上下文。
### 获取`SpringBootExceptionReporter`系列异常收集器
```java
// 获取SpringBootExceptionReporter系列异常收集器（从spring.factories文件加载，并实例化和排序）
exceptionReporters = getSpringFactoriesInstances(
		SpringBootExceptionReporter.class,
		new Class[] { ConfigurableApplicationContext.class }, context);
```

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
		Class<?>[] parameterTypes, Object... args) {
	ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
	// Use names and ensure unique to protect against duplicates
	Set<String> names = new LinkedHashSet<>(
			SpringFactoriesLoader.loadFactoryNames(type, classLoader));
	List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
			classLoader, args, names);
	AnnotationAwareOrderComparator.sort(instances);
	return instances;
}
```
　　可以看到，加载异常收集器与上面初始化器和监听器如出一辙，不做过多阐述。
### 上下文前置处理、广播`ApplicationPreparedEvent`事件
```java
// 上下文前置处理（执行ApplicationContextInitializer系列初始化器、加载资源、广播ApplicationPreparedEvent）
prepareContext(context, environment, listeners, applicationArguments,
			printedBanner);
```
```java
private void prepareContext(ConfigurableApplicationContext context,
		ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
		ApplicationArguments applicationArguments, Banner printedBanner) {
	// 关联上下文和环境
	context.setEnvironment(environment);
	//
	postProcessApplicationContext(context);
	// 去重并排序前面获取好的ApplicationContextInitializer初始化器，执行初始化
	applyInitializers(context);
	// 遍历前面设置好的SpringApplicationRunListener，但并没有发布（目前什么都没做，貌似为了以后扩展）
	listeners.contextPrepared(context);
	if (this.logStartupInfo) {
		logStartupInfo(context.getParent() == null);
		logStartupProfileInfo(context);
	}

	// Add boot specific singleton beans
	// 添加启动特定的单例bean
	ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
	beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
	if (printedBanner != null) {
		beanFactory.registerSingleton("springBootBanner", printedBanner);
	}

	if (beanFactory instanceof DefaultListableBeanFactory) {
		((DefaultListableBeanFactory) beanFactory)
				.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
	}
	// Load the sources
	// 加载sources资源
	Set<Object> sources = getAllSources();
	Assert.notEmpty(sources, "Sources must not be empty");

	load(context, sources.toArray(new Object[0]));
	// 广播ApplicationPreparedEvent
	listeners.contextLoaded(context);
}
```
　　在启动上下文之前，会调用前面初始化好的`ApplicationContextInitializer`接口实现类，当然也包含我们自定义的，所以我们可以自定义初始化器，在上下文启动前做一些操作。之后会广播`ApplicationPreparedEvent`事件，通知`SpringApplicationRunListener`监听器`ConfigurableApplicationContext`上下文已准备好（框架中用`listeners.contextLoaded(context);`方法广播了`ApplicationPreparedEvent`事件，而`ApplicationLoadedEvent`事件并没有发布，感觉这里以后还会变动。。。）。
### 刷新上下文
```java
// 刷新上下文（）
refreshContext(context);
```
```java
private void refreshContext(ConfigurableApplicationContext context) {
	refresh(context);
	if (this.registerShutdownHook) {
		try {
			context.registerShutdownHook();
		}
		catch (AccessControlException ex) {
			// Not allowed in some environments.
		}
	}
}
```
### 发布`ApplicationStartedEvent`事件
```java
// 遍历前面设置的ConfigurableApplicationContext监听器，发布ApplicationStartedEvent
listeners.started(context);
```
```java
public void started(ConfigurableApplicationContext context) {
	for (SpringApplicationRunListener listener : this.listeners) {
		listener.started(context);
	}
}
```
```java
@Override
public void started(ConfigurableApplicationContext context) {
	context.publishEvent(
			new ApplicationStartedEvent(this.application, this.args, context));
}
```
　　发布`ApplicationStartedEvent`事件，通知　`SpringApplicationRunListener`系列监听器`ConfigurableApplicationContext`上下文已启动完成，Spring Bean 已初始化完成。
### 回调实现了`ApplicationRunner`或`CommandLineRunner`接口的`Runners`
```java
// 按顺序回调实现了ApplicationRunner或CommandLineRunner接口的Runners
callRunners(context, applicationArguments);
```
```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
	List<Object> runners = new ArrayList<>();
	runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
	runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
	AnnotationAwareOrderComparator.sort(runners);
	for (Object runner : new LinkedHashSet<>(runners)) {
		if (runner instanceof ApplicationRunner) {
			callRunner((ApplicationRunner) runner, args);
		}
		if (runner instanceof CommandLineRunner) {
			callRunner((CommandLineRunner) runner, args);
		}
	}
}
```
　　这里是有顺序的，`ApplicationRunner`的实现类优先于`CommandLineRunner`的实现类被回调
### 发布`ApplicationReadyEvent`事件
```java
try {
	// 遍历前面设置好的SpringApplicationRunListener监听器，发布ApplicationReadyEvent
	listeners.running(context);
}
```
```java
public void running(ConfigurableApplicationContext context) {
	for (SpringApplicationRunListener listener : this.listeners) {
		listener.running(context);
	}
}
```

```java
@Override
public void running(ConfigurableApplicationContext context) {
	context.publishEvent(
			new ApplicationReadyEvent(this.application, this.args, context));
}
```
　　发布`ApplicationReadyEvent`事件,通知`SpringApplicationRunListener`系列监听器`ConfigurableApplicationContext`上下文已经在运行，整个容器已经准备好。

### 发布`ExitCodeEvent`和`ApplicationFailedEvent`事件和异常收集器收集异常信息
```java
catch (Throwable ex) {
	handleRunFailure(context, ex, exceptionReporters, listeners);
	throw new IllegalStateException(ex);
}
```
```java
private void handleRunFailure(ConfigurableApplicationContext context,
		Throwable exception,
		Collection<SpringBootExceptionReporter> exceptionReporters,
		SpringApplicationRunListeners listeners) {
	try {
		try {
			handleExitCode(context, exception);
			if (listeners != null) {
				listeners.failed(context, exception);
			}
		}
		finally {
			reportFailure(exceptionReporters, exception);
			if (context != null) {
				context.close();
			}
		}
	}
	catch (Exception ex) {
		logger.warn("Unable to close ApplicationContext", ex);
	}
	ReflectionUtils.rethrowRuntimeException(exception);
}
```

　　如果启动过程中出现异常，springboot将会发布`ExitCodeEvent`事件通知上下文停止或重启，并发布`ApplicationFailedEvent`事件通知`SpringApplicationRunListener`系列监听器，最后`SpringBootExceptionReporter`异常收集器收集打印异常。
## 总结
### SpringBoot启动过程大致脉络
- 准备阶段
  - 参数初始化
  - 推断应用类型
  - 加载ApplicationContextInitializer系列初始化器
  - 加载ApplicationListener系列监听器
  - 推断应用入口类（main函数所在类）

- 运行阶段
  - 初始化容器启动计时器，开始计时
  - 初始化上下文ConfigurableApplicationContext、异常收集器
  - 配置系统参数"java.awt.headless"
  - 获取SpringApplicationRunListener系列监听器
  - 遍历所有SpringApplicationRunListener系列监听器，广播ApplicationStartingEvent
  - 处理args参数
  - 准备环境（创建、配置、绑定环境、广播ApplicationEnvironmentPreparedEvent）
  - 配置忽略Bean信息
  - 打印Banner
  - 根据应用类型创建上下文
  - 获取SpringBootExceptionReporter系列异常收集器
  - 上下文前置处理（执行ApplicationContextInitializer系列初始化器、加载资源、广播ApplicationPreparedEvent）
  - 刷新上下文
  - 启动完成，打印用时
  - 遍历前面设置的ConfigurableApplicationContext监听器，发布ApplicationStartedEvent
  - 回调实现了ApplicationRunner或CommandLineRunner接口的Runners
  - 遍历前面设置好的SpringApplicationRunListener监听器，发布ApplicationReadyEvent

　　至此，整个SpringBoot项目已经启动完成，我们可以看到，整个过程中Spring的事件驱动机制起着举足轻重的作用，有了这个机制我们可以知晓容器的启动过程，并且可以监听到某些事件，对容器中我们关心的实例做进一步处理，我们深入理解事件驱动机制很有必要，它将帮助我们更好的理解和使用这个Spring框架体系。如果想要文中中文版SpringBoot注释源码，可以在[我的github](https://github.com/ZhaoGitHub1/spring-boot)下载，如果发现哪里写的不对，烦请留言通知我。