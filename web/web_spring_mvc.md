[使用Spring Tool Suite(STS)和Maven建立的Spring mvc 项目](http://blog.csdn.net/zoubf/article/details/50384756)  

[SpringMVC DispatcherServlet初始化过程](http://blog.csdn.net/tiantiandjava/article/details/47663853)

[Spring 源码解析之HandlerMapping源码解析(一)](http://blog.csdn.net/king_is_everyone/article/details/51446260)

[Spring framework4.2.1源码构建为Intellij项目](http://bsr1983.iteye.com/blog/2234984)

[Spring源码解析——如何阅读源码](http://www.cnblogs.com/xing901022/p/4178963.html)
 
[simple spring web maven 项目无法发布到tomcat下](http://bbs.csdn.net/topics/390900092) 


在Servlet3.0环境中，容器会在类路径中查找实现javax.servlet.ServletContainerInitializer的类，如果发现已有实现类，就会调用该类配置Servlet容器。在Spring中，org.springframework.web.SpringServletContainerInitializer 实现了该接口，这个类会查找实现了 org.springframework.web.WebApplicationInitializer 的类，调用 onStartup() 方法配置servlet容器，将 DispatcherServlet 注册到servlet上下文中。

DispatcherServlet 类的 doDispatch 方法为 request 匹配处理方法。
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception 

mappedHandler = getHandler(processedRequest);
{
  for (HandlerMapping hm : this.handlerMappings)
    HandlerExecutionChain handler = hm.getHandler(request);
}

private List<HandlerMapping> handlerMappings;

initHandlerMappings() 中初始化
private void initHandlerMappings(ApplicationContext context) {
Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
}

//ApplicationContext 定义
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
    

@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {
  //@HandlesTypes 声明的类型会被作为参数webAppInitializerClasses传入
  public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
			throws ServletException {
      //如果不是接口并且不是抽象类，并且是WebApplicationInitializer类或其子类，加入初始化队列
      initializers.add((WebApplicationInitializer)
								ReflectionUtils.accessibleConstructor(waiClass).newInstance());
                
      //向 中记录log
      servletContext.log(initializers.size() + " Spring WebApplicationInitializers detected on classpath");
      
      //
      for (WebApplicationInitializer initializer : initializers) {
			  initializer.onStartup(servletContext);
		  }
  }
}

org.springframework.boot.context.web.SpringBootServletInitializer 实现了接口 WebApplicationInitializer

```

搜索实现了 javax.servlet.ServletContainerInitializer 的类。 org.springframework.web.SpringServletContainerInitializer 实现了该接口。@HandlesTypes 注解声明实现了 WebApplicationInitializer.class 接口的类会被作为参数传入 onStartup() 方法。

```java
@HandlesTypes(WebApplicationInitializer.class)
public class SpringServletContainerInitializer implements ServletContainerInitializer {
    @Override
    public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
        throws ServletException {
}
```

如果参数 webAppInitializerClasses set 中的类不是接口且不是抽象类，并且是 WebApplicationInitializer 类或其子类，创建实例，加入初始化队列。

依次调用初始化队列中的 WebApplicationInitializer 子类的 onStartup() 方法。
```java
for (WebApplicationInitializer initializer : initializers) {
    initializer.onStartup(servletContext);
}
```

WebApplicationInitializer 子类的 onStartup() 方法注册 DispatcherServlet 。

![](http://img.blog.csdn.net/20150814163251144?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##### 初始化 DispatcherServlet
Servlet 容器调用 Servlet 接口的 init(ServletConfig config) 方法。

FrameworkServlet 类的方法 initWebApplicationContext() 初始化 webApplicationContext 。
```java
protected WebApplicationContext initWebApplicationContext() {
```

```java
public void setContextClass(Class<?> contextClass)
```
设置 WebApplicationContext 的类型，也就是实现的子类。这个类会在 createWebApplicationContext() 方法中被创建。

```
contextConfigLocation 变量的说明
 * <p>Passes a "contextConfigLocation" servlet init-param to the context instance,
 * parsing it into potentially multiple file paths which can be separated by any
 * number of commas and spaces, like "test-servlet.xml, myServlet.xml".
 * If not explicitly specified, the context implementation is supposed to build a
 * default location from the namespace of the servlet.
 ```

调用 initStrategies 方法，进行一系列初始化。
```java
protected void initStrategies(ApplicationContext context) {
	initMultipartResolver(context);
	initLocaleResolver(context);
	initThemeResolver(context);
	initHandlerMappings(context);
	initHandlerAdapters(context);
	initHandlerExceptionResolvers(context);
	initRequestToViewNameTranslator(context);
	initViewResolvers(context);
	initFlashMapManager(context);
}
```

```java
initStrategies(ApplicationContext context)
context = AnnotationConfigWebApplicationContext

private void initHandlerMappings(ApplicationContext context) {
  this.handlerMappings = null
  this.detectAllHandlerMappings = true
  //这些 HandlerMapping 是从哪里注册过来的？
  this.handlerMappings = {
    [0]	RequestMappingHandlerMapping  (id=243)	
    [1]	WebMvcConfigurationSupport$EmptyHandlerMapping  (id=264)	
    [2]	BeanNameUrlHandlerMapping  (id=233)	
    [3]	WebMvcConfigurationSupport$EmptyHandlerMapping  (id=266)	
    [4]	WebMvcConfigurationSupport$EmptyHandlerMapping  (id=267)}
  this.handlerAdapters = {
    [0]	RequestMappingHandlerAdapter  (id=187)	
    [1]	HttpRequestHandlerAdapter  (id=190)	
    [2]	SimpleControllerHandlerAdapter  (id=149)	}
```
