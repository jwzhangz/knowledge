## Faking

In the JMockit toolkit, the Mockups API provides support for the creation of fake implementations, or "mock-ups". Typically, a mock-up targets a few methods and/or constructors in the class to be faked, while leaving most other methods and constructors unmodified. Also, usually the classes to be faked belong to external libraries, not to the codebase under test.

在JMockit工具包中，Mockup API提供伪造实现的创建。一般的一个需要伪装的类只需要伪装一部分方法或构造器，其他的方法保留不变。并且这些需要伪装的类来自于外部库。

Fake implementations can be particularly useful in tests which depend on external components or resources such as e-mail or web services servers, complex libraries, etc. Often, the mock-ups will be applied from reusable testing infrastructure components, not directly from a test class. In particular, the use of both the Faking API and the Mocking API in the same test class should be viewed with suspicion, as it strongly indicates misuse.

伪装实现在依赖外部组件或资源的测试中特别有用，例如邮件或web服务。

The replacement of real implementations with fake ones is completely transparent to the code which uses those dependencies, and can be switched on and off for the scope of a single test, all tests in a single test class, or for the entire test run.

***
### 1. Mock methods and mock-up classes
In the context of the Mockups API, a mock (or fake) method is any method in a mock-up (fake) class that gets annotated with @Mock. A mock-up class is any class extending the mockit.MockUp<T> generic base class, where <T> is the type to be faked. The example below shows several mock methods defined in a mock-up class for our example "real" class, javax.security.auth.login.LoginContext.

在Mockups API上下文中，mock方法是一个mock-up类的带有 @Mock 注解的任意方法。通过继承mockit.MockUp<T>实现mock-up类，T就是需要被伪装的类。

```java
public final class MockLoginContext extends MockUp<LoginContext>
{
   @Mock
   public void $init(String name, CallbackHandler callback)
   {
      assertEquals("test", name);
      assertNotNull(callback);
   }

   @Mock
   public void login() {}

   @Mock
   public Subject getSubject() { return null; }
}
```
When a mock-up class is applied to a real class, the latter gets the implementation of those methods and constructors which have corresponding mock methods temporarily replaced with the implementations of the matching mock methods, as defined in the mock-up class. In other words, the real class becomes "faked" for the duration of the test which applied the mock-up class. Its methods will respond accordingly whenever it receives invocations during test execution. At runtime, what really happens is that the execution of a faked method/constructor is intercepted and redirected to the corresponding mock method, which then executes and returns (unless an exception/error is thrown) to the original caller, without this one noticing that a different method was actually executed. Normally, the "caller" class is one under test, while the faked class is a dependency.

当一个mock-up类作用于一个真实的类，测试中调用的方法或者构造器将会被替换成mock 方法。在整个测试期间，这个类都将被伪装。所有对被mock的方法的调用都将重定向到mock方法。

Mock classes are often defined as nested (static), inner (non-static), or even as anonymous classes inside a JUnit/TestNG test class. There is nothing preventing mock classes from being top-level, though. That would be useful if the mock class is to be reused in multiple test classes.

mock 类经常作为测试类的内部静态类使用。当然也可以定义为公用类，在多个测试类中重用。

Each @Mock method must have a corresponding "real method/constructor" with the same signature in the targeted real class. For a method, the signature consists of the method name and parameters; for a constructor, it's just the parameters, with the mock method having the special name "$init". If a matching real method/constructor cannot be found for a given mock method, either in the specified real class or in its super-classes (excluding java.lang.Object), an IllegalArgumentException is thrown when the test attempts to apply the mock class. Notice this exception can be caused by a refactoring in the real class (such as renaming the real method), so it's important to understand why it happens.

每个带 @Mock 注解的方法都必须对应一个真实存在的方法，拥有相同的签名。也就是说方法的名字，参数必须一致。

Finally, notice there is no need to have mock methods for all methods and constructors in a real class. Any such method or constructor for which no corresponding mock method exists in the mock-up class will simply stay "as is", that is, it won't be faked.

最后，不必将所有方法和构造器都mock。

***
### 2. Applying mock-ups
A given mock-up class must be applied to a corresponding real class to have any effect. This is usually done for a whole test class or test suite, but can also be done for an individual test. Mock-ups can be applied from anywhere inside a test class: a @BeforeClass method, a @Before method (or @BeforeMethod if using TestNG), or from a @Test method. Once a mock-up class is applied, all executions of the faked methods and constructors of the real class get automatically redirected to the corresponding mock methods.

mock-up类只有被应用到真实类才会生效。作用的范围既可以是整个test，也可以是test suite，还可以是单独的test。mockup类可以在test中的任何地方使用， @BeforeClass 方法或 @Before 方法，或者 @Test 方法。一旦mock-up类被使用，所有被伪装的方法的调用会被重定向到伪装方法。

To apply the MockLoginContext mock class above, we simply instantiate it:

```java
@Test
public void applyingAMockClass() throws Exception
{
   new MockLoginContext());

   // Inside an application class which creates a suitable CallbackHandler:
   new LoginContext("test", callbackHandler).login();

   ...
}
```
Since the mock-up class is applied inside a test method, the faking of LoginContext by MockLoginContext will be in effect only for that particular test.

因为mock-up类在test方法中使用，所以伪装只在这个方法中生效。

When the constructor invocation that instantiates LoginContext executes, the corresponding "$init" mock method in MockLoginContext will be executed. Similarly, when the LoginContext#login method is called, the corresponding mock method will be executed, which in this case will do nothing since the method has no parameters and void return type. The mock-up class instance on which these invocations occur is the one created in the first part of the test.

当执行LoginContext的构造器时，相应的"$init" mock 方法会被执行到。当执行init方法时，实际会执行被mock的 init 方法。

#### 2.1 Kinds of methods which can be faked
So far, we have only faked public instance methods with public instance mock methods. In reality, any other kind of method in a real class can be faked: methods with private, protected or "package-private" accessibility, static methods, final methods, and native methods. Even more, a static method in the real class can be faked by an instance mock method, and vice-versa (an instance real method with a static mock).

到现在为止，我们只伪装了public方法。实际上，任意的其他种类的方法都可以伪装，包括private，protected，"package-private"，static，final，以及native 方法。

Methods to be faked need to have an implementation, though not necessarily in bytecode (in the case of native methods). Therefore, an abstract method cannot be faked directly, and the same applies to the methods of a Java interface. (That said, as shown later the Mockups API can automatically create a proxy class that implements an interface.)

伪装的方法必须被实现。所以抽象类和接口不能被直接fake。

### 2.2 In-line mock-up classes
Sometimes, we need to fake a class only for a single test. In such a situation we can create an anonymous mock-up class inside an individual test method, as demonstrated by the next example.

```java
@Test
public void applyingAnAnonymousMockup() throws Exception
{
   new MockUp<LoginContext>() {
      @Mock void $init(String name) { /* do nothing */ }
      @Mock void login() {}
   });

   new LoginContext("test").login();
}
```
Note that mock methods don't need to be public.

***
### 3. Faking an interface
Most of the time a mock class targets a real class directly. But what if we need a mock object that implements a certain interface, to be passed to code under test? The following example test shows how it is done for the interface javax.security.auth.callback.CallbackHandler.

``` java
@Test
public void fakingAnInterface() throws Exception
{
   CallbackHandler callbackHandler = new MockUp<CallbackHandler>() {
      @Mock
      void handle(Callback[] callbacks)
      {
         // fake implementation here
      }
   }.getMockInstance();

   callbackHandler.handle(new Callback[]
   {
     new NameCallback("Enter name:");
   });
}
```

The MockUp#getMockInstance() method returns a proxy object that implements the desired interface.

The MockUp#getMockInstance()方法返回一个实现了指定接口的proxy对象。

***
### 4. Faking unspecified implementation classes

To demonstrate this feature, lets consider the following code under test.

```java
public interface Service { int doSomething(); }
final class ServiceImpl implements Service { public int doSomething() { return 1; } }

public final class TestedUnit
{
   private final Service service1 = new ServiceImpl();
   private final Service service2 = new Service() { public int doSomething() { return 2; } };

   public int businessOperation()
   {
      return service1.doSomething() + service2.doSomething();
   }
}
```
The method we want to test, businessOperation(), uses classes that implement a separate interface, Service. One of these implementations is defined through an anonymous inner class, which is completely inaccessible (except for the use of Reflection) from client code.

Given a base type (be it an interface, an abstract class, or any sort of base class), we can write a test which only knows about the base type but where all implementing/extending implementation classes get faked. To do so, we create a mock-up whose target type refers only to the known base type, and does so through a type variable. Not only will implementation classes already loaded by the JVM get faked, but also any additional classes that happen to get loaded by the JVM during later test execution. This ability is demonstrated below.

对于一个给定的基类，可以使接口，抽象类以及任何类型的基类。我么可以实现一个测试类，在只知道基类的情况下，伪装所有的子类。

```java
@Test
public <T extends Service> void fakingImplementationClassesFromAGivenBaseType()
{
   new MockUp<T>() {
      @Mock int doSomething() { return 7; }
   };

   int result = new TestedUnit().businessOperation();

   assertEquals(14, result);
}
```
In the test above, all invocations to methods implementing Service#doSomething() will be redirected to the mock method implementation, regardless of the actual class implementing the interface method.

***
### 5. Faking class initializers
When a class performs some work in one or more static initialization blocks, we may need to stub it out so it doesn't interfere with test execution. We can define a special mock method for that, as shown below.

```java
@Test
public void fakingStaticInitializers()
{
   new MockUp<ClassWithStaticInitializers>() {
      @Mock
      void $clinit()
      {
         // Do nothing here (usually).
      }
   };

   ClassWithStaticInitializers.doSomething();
}
```
Special care must be taken when the static initialization code of a class is faked. Note that this includes not only any "static" blocks in the class, but also any assignments to static fields (excluding those resolved at compile time, which do not produce executable bytecode). Since the JVM only attempts to initialize a class once, restoring the static initialization code of a faked class will have no effect. So, if you fake away the static initialization of a class that hasn't been initialized by the JVM yet, the original class initialization code will never be executed in the test run. This will cause any static fields that are assigned with expressions computed at runtime to instead remain initialized with the default values for their types.

***
### 6. Accessing the invocation context
A mock method can optionally declare an extra parameter of type mockit.Invocation, provided it is the first parameter. For each actual invocation to the corresponding faked method/constructor, an Invocation object will be automatically passed in when the mock method is executed.

mock方法有一个可选参数，类型是mockit.Invocation，作为第一个参数。当执行mock方法时，这个参数被自动传入。

This invocation context object provides several getters which can be used inside the mock method. One is the getInvokedInstance() method, which returns the faked instance on which the invocation occurred (null if the faked method is static). Other getters provide the number of invocations (including the current one) to the faked method/constructor, the invocation arguments (if any), and the invoked member (a java.lang.reflect.Method or java.lang.reflect.Constructor object, as appropriate). Below we have an example test.

这个上下文参数提供一些getter方法。getInvokedInstance()返回发生调用的被伪装的对象，如果方法是静态方法，则上下文对象为null。

```java
@Test
public void accessingTheFakedInstanceInMockMethods() throws Exception
{
   final Subject testSubject = new Subject();

   new MockUp<LoginContext>() {
      @Mock
      void $init(Invocation invocation, String name, Subject subject)
      {
         assertNotNull(name);
         assertSame(testSubject, subject);

         // Gets the invoked instance.
         LoginContext loginContext = invocation.getInvokedInstance();

         // Verifies that this is the first invocation.
         assertEquals(1, invocation.getInvocationCount());

         // Forces setting of private Subject field, since no setter is available.
         Deencapsulation.setField(loginContext, subject);
      }

      @Mock
      void login(Invocation invocation)
      {
         // Gets the invoked instance.
         LoginContext loginContext = invocation.getInvokedInstance();

         // getSubject() returns null until the subject is authenticated.
         assertNull(loginContext.getSubject());

         // Private field set to true when login succeeds.
         Deencapsulation.setField(loginContext, "loginSucceeded", true);
      }

      @Mock
      void logout(Invocation invocation)
      {
         // Gets the invoked instance.
         LoginContext loginContext = invocation.getInvokedInstance();

         assertSame(testSubject, loginContext.getSubject());
      }
   };

   LoginContext theFakedInstance = new LoginContext("test", testSubject);
   theFakedInstance.login();
   theFakedInstance.logout();
}
```

***
### 7. Proceeding into the real implementation
Once a @Mock method is executing, any additional calls to the corresponding faked method are also redirected to the mock method, causing its implementation to be re-entered. If, however, we want to execute the real implementation of the faked method, we can call the proceed() method on the Invocation object received as the first parameter to the mock method.

想要在伪装的方法中调用真实的实现，需要调用Invocation对象的proceed()方法。

The example test below exercises a LoginContext object created normally (without any mocking in effect at creation time), using an unspecified configuration. (For the complete version of the test, see the mockit.MockAnnotationsTest class.)

```java
@Test
public void proceedIntoRealImplementationsOfFakedMethods() throws Exception
{
   // Create objects used by the code under test:
   LoginContext loginContext = new LoginContext("test", null, null, configuration);

   // Apply mock-ups:
   ProceedingMockLoginContext mockInstance = new ProceedingMockLoginContext();

   // Exercise the code under test:
   assertNull(loginContext.getSubject());
   loginContext.login();
   assertNotNull(loginContext.getSubject());
   assertTrue(mockInstance.loggedIn);

   mockInstance.ignoreLogout = true;
   loginContext.logout(); // first entry: do nothing
   assertTrue(mockInstance.loggedIn);

   mockInstance.ignoreLogout = false;
   loginContext.logout(); // second entry: execute real implementation
   assertFalse(mockInstance.loggedIn);
}

static final class ProceedingMockLoginContext extends MockUp<LoginContext>
{
   boolean ignoreLogout;
   boolean loggedIn;

   @Mock
   void login(Invocation inv) throws LoginException
   {
      try {
         inv.proceed(); // executes the real code of the faked method
         loggedIn = true;
      }
      finally {
         // This is here just to show that arbitrary actions can be taken inside
         // the mock, before and/or after the real method gets executed.
         LoginContext lc = inv.getInvokedInstance();
         System.out.println("Login attempted for " + lc.getSubject());
      }
   }

   @Mock
   void logout(Invocation inv) throws LoginException
   {
      // We can choose to proceed into the real implementation or not.
      if (!ignoreLogout) {
         inv.proceed();
         loggedIn = false;
      }
   }
}
```
In the example above, all the code inside the tested LoginContext class will get executed, even though some methods (login and logout) are faked. This example is contrived; in practice, the ability to proceed into real implementations would not normally be useful for testing per se, not directly at least.

You may have noticed that use of Invocation#proceed(...) in a mock method effectively behaves like advice (from AOP jargon) for the corresponding real method. This is a powerful ability that can be useful for certain things (think of an interceptor or decorator).

For more details on all the methods available in the mockit.Invocation class, see its API documentation.

***
### 8. Reusing mock-ups between tests
Often, a mock-up class needs to be used throughout multiple tests, or even applied for the test run as a whole. One option is to use test setup methods that run before each test method; with JUnit, we use the @Before annotation; with TestNG, it's @BeforeMethod. Another is to apply mock-ups inside of a test class setup method: @BeforeClass. Either way, the mock-up class is applied by simply instantiating it inside the setup method.

Once applied, a mock-up will remain in effect for the execution of all tests in the test class. The scope of a mock-up applied in a "before" method includes the code in any "after" methods the test class may have (annotated with @After for JUnit or @AfterMethod for TestNG). The same goes for any mock-ups applied in a @BeforeClass method: they will still be in effect during the execution of any AfterClass methods. Once the last "after" or "after class" method finish being executed, though, all mock-ups get automatically "torn down".

For example, if we wanted to fake the LoginContext class with a mock-up class for a bunch of related tests, we would have the following methods in a JUnit test class:
```java
public class MyTestClass
{
   @BeforeClass
   public static void applySharedMockups()
   {
      new MockUp<LoginContext>() {
         // shared @Mock's here...
      };
   }

   // test methods that will share the mock-ups applied above...
}
```
It is also possible to extend from a base test class, which may optionally define "before" methods that apply one or more mock-ups.

***
### 9. Global mock-ups
Sometimes, we need to apply mock-ups for the entire scope of a test suite (all of its test classes), ie, a "global" mock-up. Such a mock-up will remain in effect for the entire test run. This can be done in test code or through external configuration.

有时，我们需要mock-up的作用域是整个test suite(包括肯多test类)，即一个全局的mock-up。可以通过test代码或者外部配置来实现。

#### 9.1 Programmatic application of global mock-ups
To apply mock-ups over a test suite, we can use a TestNG @BeforeSuite method, or a JUnit Suite class. The next example shows a JUnit 4 test suite configuration with the application of global mock-ups.

实现TestNG的 @BeforeSuite 方法或者JUnit Suite类。


```java
@RunWith(Suite.class)
@Suite.SuiteClasses({MyFirstTest.class, MySecondTest.class})
public final class TestSuite
{
   @BeforeClass
   public static void applyGlobalMockUps()
   {
      new LoggingMocks();

      new MockUp<SomeClass>() {
         @Mock someMethod() {}
      };
   }
}
```

In this example, we apply the LoggingMocks mock-up class and an inline mock-up class; their mock methods will be in effect until just after the last test in the test suite has been executed.

#### 9.2 External application through a system property
The mockups system property supports a comma-separated list of fully qualified mock-up class names. If specified at JVM startup time, any such class (which must extend MockUp<T>) will be automatically applied for the whole test run. The mock methods defined in startup mock classes will remain in effect until the end of the test run, for all test classes. Each mock-up class will be instantiated through its no-args constructor, unless an additional value was provided after the class name (for example, as in "-Dmockups=my.mockups.MyMockUp=anArbitraryStringWithoutCommas"), in which case the mock-up class should have a constructor with one parameter of type String.

Note that a system property can be passed to the JVM through the standard "-D" command line parameter. Ant/Maven/etc. build scripts have their own ways of specifying system properties, so check their documentation for details.

*** 10. Applying AOP-style advice
There is one more special @Mock method that can appear in a mock-up class: the "$advice" method. If defined, this mock method will handle executions of each and every method in the target class (or classes, when applying the mock-up over unspecified classes from a base type). Differently from regular mock methods, this one needs to have a particular signature and return type: Object $advice(Invocation).

For demonstration, lets say we want to measure the execution times of all methods in a given class during test execution, while still executing the original code of each method.

```java
public final class MethodTiming extends MockUp<Object>
{
   private final Map<Method, Long> methodTimes = new HashMap<>();

   public MethodTiming(Class<?> targetClass) { super(targetClass); }
   MethodTiming(String className) throws ClassNotFoundException { super(Class.forName(className)); }

   @Mock
   public Object $advice(Invocation invocation)
   {
      long timeBefore = System.nanoTime();

      try {
         return invocation.proceed();
      }
      finally {
         long timeAfter = System.nanoTime();
         long dt = timeAfter - timeBefore;

         Method executedMethod = invocation.getInvokedMember();
         Long dtUntilLastExecution = methodTimes.get(executedMethod);
         Long dtUntilNow = dtUntilLastExecution == null ? dt : dtUntilLastExecution + dt;
         methodTimes.put(executedMethod, dtUntilNow);
      }
   }

   @Override
   protected void onTearDown()
   {
      System.out.println("\nTotal timings for methods in " + targetType + " (ms)");

      for (Entry<Method, Long> methodAndTime : methodTimes.entrySet()) {
         Method method = methodAndTime.getKey();
         long dtNanos = methodAndTime.getValue();
         long dtMillis = dtNanos / 1000000L;
         System.out.println("\t" + method + " = " + dtMillis);
      }
   }
}
```
The mock-up above can be applied inside a test, in a "before" method, in a "before class" method, or even for the entire test run by setting "-Dmockups=testUtils.MethodTiming=my.application.AppClass". It will add up the execution times for all executions of all methods in a given class. As shown in the implementation of the $advice method, it can obtain the java.lang.reflect.Method that is being executed. If desired, the current invocation count and/or the invocation arguments could be obtained through similar calls to the Invocation object. When the mock-up is (automatically) torn down, the onTearDown() method gets executed, dumping measured timings to standard output.
