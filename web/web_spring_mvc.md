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

