## Mocking
![](http://jmockit.org/tutorial/MockingAPI.png)

In the JMockit library, the Expectations API provides rich support for the use of mocking in automated developer tests. When mocking is used, a test focuses on the behavior of the code under test, as expressed through its interactions with other types it depends upon. Mocking is typically used in the construction of isolated unit tests, where a unit under test is exercised in isolation from the implementation of other units it depends on. Typically, a unit of behavior is embodied in a single class, but it's also fine to consider a whole set of strongly-related classes as a single unit for the purposes of unit testing (as is usually the case when we have a central public class with one or more helper classes, possibly package-private); in general, individual methods should not be regarded as separate units on their own.

Strict unit testing, however, is not a recommended approach; one should not attempt to mock every single dependency. Mocking is best used in moderation; whenever possible, favor integration tests over isolated unit tests. This said, mocking is occasionally also useful in the creation of integration tests, when some particular dependency cannot have its real implementation easily used, or when attempting to create tests for corner cases where a well-placed mocked interaction can greatly facilitate the test.

An interaction between two classes always takes the form of a method or constructor invocation. The set of invocations from a tested class to its dependencies, together with the argument and return values passed between them, define the behavior of interest for the tests of that particular class. In addition, a given test may need to verify the relative order of execution between multiple invocations.

***
### 1. Mocked types and instances
Methods and constructors invoked from the code under test on a dependency are the targets for mocking. Mocking provides the mechanism that we need in order to isolate the tested code from (some of) its dependencies. We specify which particular dependencies are to be mocked for a given test (or tests) by declaring suitable mock fields and/or mock parameters; mock fields are declared as annotated instance fields of the test class, while mock parameters are declared as annotated parameters of a test method. The type of the dependency to be mocked will be the type of the mock field or parameter. Such a type can be any kind of reference type: an interface, a class (including abstract and final ones), an annotation, or an enum.

Mocking 提供了一种机制，使被测试代码从依赖中隔离。这就需要在编写测试类时将这些依赖定义为带@Mocked注解的field或者方法的入参。

By default, all non-private methods (including any that are static, final, or native) of the mocked type will be mocked for the duration of the test. If the declared mocked type is a class, then all of its super-classes up to but not including java.lang.Object will also be mocked, recursively. Therefore, inherited methods will automatically be mocked as well. Also in the case of a class, all of its non-private constructors will get mocked.

默认的，在执行test过程中，被mock的 type(例如class)的所有的non-private方法(包括static，final或native)都将被mock。如果声明的被mock的 type是一个类，则它的上溯至Object的父类（不包括Object）也将被mock。所以通过继承得到的方法会自动的被mock。以class为例，所有的non-private的构造器将会被mock。

When a method or constructor is mocked, its original implementation code won't be executed for invocations occurring during the test. Instead, the call will be redirected to JMockit so it can be dealt with in the manner that was explicitly or implicitly specified for the test.

如果一个方法或构造器被mock，在测试期间，调用这些方法并不会执行方法的代码。这些调用将会被JMockit接管。

The following example test skeleton serves as a basic illustration for the declaration of mock fields and mock parameters, as well as the way in which they are typically used in test code. In this tutorial, we use many code snippets like this, where the parts in bold font are the current focus of explanation.
```java
// "Dependency" is mocked for all tests in this test class.
// The "mockInstance" field holds a mocked instance automatically created for use in each test.
@Mocked Dependency mockInstance;

@Test
public void doBusinessOperationXyz(@Mocked final AnotherDependency anotherMock)
{
   ...
   new Expectations() {{ // an "expectation block"
      ...
      // Record an expectation, with a given value to be returned:
      mockInstance.mockedMethod(...); result = 123;
      ...
   }};
   ...
   // Call the code under test.
   ...
   new Verifications() {{ // a "verification block"
      // Verifies an expected invocation:
      anotherMock.save(any); times = 1;
   }};
   ...
}
```
For a mock parameter declared in a test method, an instance of the declared type will be automatically created by JMockit and passed by the JUnit/TestNG test runner when it executes the test method; therefore, the parameter value will never be null. For a mock field, an instance of the declared type will be automatically created by JMockit and assigned to the field, provided it's not final.

如果为一个测试方法添加mock参数，当JUnit/TestNG test runner执行这些测试方法时，JMockit将自动创建一个实例并作为参数由test runner传入。如果定义一个mock field，JMockit将自动创建一个实例并赋值给field。注意field不要定义为final。

There are a few different annotations available for the declaration of mock fields and parameters, and ways in which the default mocking behavior can be modified to suit the needs of a particular test. Other sections of this chapter go into the details, but the basics are: @Mocked is the central mocking annotation, having one optional attribute which is useful in certain cases; @Injectable is another mocking annotation, which constrains mocking to the instance methods of a single mocked instance; and @Capturing is yet another mocking annotation, which extends mocking to the classes implementing a mocked interface, or the subclasses extending a mocked class. When @Injectable or @Capturing is applied to a mock field or mock parameter, @Mocked is implied so it doesn't need to (but can) be applied as well.

下面的章节会详细介绍一些声明mock fields和parameters，以及修改默认mock行为的一些方法。一些常用的注解：
+ @Mocked 有一个可选的属性，在一些特定的场景很有用。
+ @Injectable 强制被mock的实例方法仅作用于被修饰的实例。
+ @Capturing 将mocking扩展到实现了被mock的interface的类，或者继承一个被mock的class的子类。

当@Injectable or @Capturing应用于mock field或者mock parameter，已经包含了mock的功能，不需要再用@Mocked注解了。

The mocked instances created by JMockit can be used normally in test code (for the recording and verification of expectations), and/or passed to the code under test. Or they may simply go unused. Differently from other mocking APIs, these mocked objects don't have to be the ones used by the code under test when it calls instance methods on its dependencies. By default (ie, when @Injectable is not used), JMockit does not care on which object a mocked instance method is called. This allows the transparent mocking of instances created directly inside code under test, when said code invokes constructors on brand new instances using the new operator; the classes instantiated must be covered by mocked types declared in test code, that's all.

被JMockit创建的mock实例能够被test code正常使用。或者放置不用。这和其他mocking API不同，当被测试代码调用依赖的实例的方法时，这些mock的对象并不是必需的。默认的，JMockit并不关心被mock的实例方法在哪个对象上被调用。因此，当测试代码用new操作符调用构造器创建实例，在被测代码内直接创建透明的mocking，被实例化的class必须被mock 类型覆盖。

***
### 2. Expectations
An expectation represents a set of invocations to a specific mocked method/constructor that is relevant for a given test. An expectation may cover multiple different invocations to the same method or constructor, but it doesn't have to cover all such invocations that occur during the execution of the test. Whether a particular invocation matches a given expectation or not will depend not only on the method/constructor signature but also on runtime aspects such as the instance on which the method is invoked, argument values, and/or the number of invocations already matched. Therefore, several types of matching constraints can (optionally) be specified for a given expectation.

一个expectation代表着一系列的对指定的mock method/constructor的调用。在一个expectation中，可以对同一个方法或构造器定义不同的调用。一个调用能否和一个给定的expectation匹配到不仅要看方法签名，还依赖运行时情况，例如这个方法属于哪个实例，参数值，已经产生的调用次数。所以可以为一个expectation设置一些匹配约束。

When we have one or more invocation parameters involved, an exact argument value may be specified for each parameter. For example, the value "test string" could be specified for a String parameter, causing the expectation to match only those invocations with this exact value in the corresponding parameter. As we will see later, instead of specifying exact argument values, we can specify more relaxed constraints which will match whole sets of different argument values.


The example below shows an expectation for Dependency#someMethod(int, String), which will match an invocation to this method with the exact argument values as specified. Notice that the expectation itself is specified through an isolated invocation to the mocked method. There are no special API methods involved, as is common in other mocking APIs. This invocation, however, does not count as one of the "real" invocations we are interested in testing. It's only there so that the expectation can be specified.

这是一个Dependency#someMethod(int, String)方法的expectation例子。但这个expectation只能精确匹配参数是1和"test"的调用。

```java
@Test
public void doBusinessOperationXyz(@Mocked final Dependency mockInstance)
{
   ...
   new Expectations() {{
      ...
      // An expectation for an instance method:
      mockInstance.someMethod(1, "test"); result = "mocked";
      ...
   }};

   // A call to code under test occurs here, leading to mock invocations
   // that may or may not match specified expectations.
}
```
We will see more about expectations later, after we understand the differences between recording, replaying, and verifying invocations.

***
### 3. The *record-replay-verify* model
Any developer test can be divided in at least three separate execution phases. The phases execute sequentially, one at a time, as demonstrated below.

测试过程一般至少分为三个阶段，每个阶段单独执行。下面逐一介绍。

```java
@Test
public void someTestMethod()
{
   // 1. Preparation: whatever is required before the code under test can be exercised.
   ...
   // 2. The code under test is exercised, usually by calling a public method.
   ...
   // 3. Verification: whatever needs to be checked to make sure the code exercised by
   //    the test did its job.
   ...
}
```
First, we have a preparation phase, where objects and data items needed for the test are created or obtained from somewhere else. Then, code under test is exercised. Finally, the results from exercising the tested code are compared with the expected results.

首先是准备阶段，测试所需的对象以及各种数据。然后执行被测代码，最后将执行结果和预期进行比较。

This model of three phases is also known as the Arrange, Act, Assert syntax, or "AAA" for short. Different words, but the meaning is the same.

In the context of behavior-based testing with mocked types (and their mocked instances), we can identify the following alternative phases, which are directly related to the three previously described conventional testing phases:

三个阶段：

1. The ***record*** phase, during which invocations can be recorded. This happens during test preparation, before the invocations we want to test are executed.
2. The ***replay*** phase, during which the mock invocations of interest have a chance to be executed, as the code under test is exercised. The invocations to mocked methods/constructors previously recorded will now be replayed. Often there isn't a one-to-one mapping between invocations recorded and replayed, though.
3. The ***verify*** phase, during which invocations can be verified to have occurred as expected. This happens during test verification, after the invocations under test had a chance to be executed.



1. record阶段，在执行被测代码之前的准备工作，将被测代码中所依赖的调用进行record。
2. replay阶段，执行被测代码。前一阶段record的调用将会被执行。
3. verify阶段，验证执行结果是否和预期一致。

Behavior-based tests written with JMockit will typically fit the following templates:
```java
import mockit.*;
... other imports ...

public class SomeTest
{
   // Zero or more "mock fields" common to all test methods in the class:
   @Mocked Collaborator mockCollaborator;
   @Mocked AnotherDependency anotherDependency;
   ...

   @Test
   public void testWithRecordAndReplayOnly(mock parameters)
   {
      // Preparation code not specific to JMockit, if any.

      new Expectations() {{ // an "expectation block"
         // One or more invocations to mocked types, causing expectations to be recorded.
         // Invocations to non-mocked types are also allowed anywhere inside this block
         // (though not recommended).
      }};

      // Unit under test is exercised.

      // Verification code (JUnit/TestNG assertions), if any.
   }

   @Test
   public void testWithReplayAndVerifyOnly(mock parameters)
   {
      // Preparation code not specific to JMockit, if any.

      // Unit under test is exercised.

      new Verifications() {{ // a "verification block"
         // One or more invocations to mocked types, causing expectations to be verified.
         // Invocations to non-mocked types are also allowed anywhere inside this block
         // (though not recommended).
      }};

      // Additional verification code, if any, either here or before the verification block.
   }

   @Test
   public void testWithBothRecordAndVerify(mock parameters)
   {
      // Preparation code not specific to JMockit, if any.

      new Expectations() {{
         // One or more invocations to mocked types, causing expectations to be recorded.
      }};

      // Unit under test is exercised.

      new VerificationsInOrder() {{ // an ordered verification block
         // One or more invocations to mocked types, causing expectations to be verified
         // in the specified order.
      }};

      // Additional verification code, if any, either here or before the verification block.
   }
}
```
There are other variations to the above templates, but the essence is that the expectation blocks belong to the record phase and come before the code under test is exercised, while the verification blocks belong to the verify phase. A test method can contain any number of expectation blocks, including none. The same is true for verification blocks.

The fact that anonymous inner classes are used to demarcate blocks of code allows us to take advantage of the "code folding" feature available in modern Java IDEs. The following image shows what it looks like in IntelliJ IDEA.
![](http://jmockit.org/tutorial/code-folding.png)

***
### 4. Regular versus *strict* expectations
Expectations recorded inside a "new Expectations() {...}" block are the regular ones. What this means is that the invocations they specify are expected to occur at least once during the replay phase; they may occur more than once, though, and in a different order relative to other recorded expectations; additionally, invocations that don't match any recorded expectation are allowed to occur in any number and in any order. If no invocation matches a given recorded expectation, a "missing invocation" error gets thrown at the end of the test, causing it to fail (this is only the default behavior, though, as it can be overridden).

在 new Expectations() {...}中可以定义常规的期望，这些被记录的调用将在接下来的replay阶段被至少调用一次。调用顺序可以和定义顺序不一致。如果任意一个expectation没有被调用到，则抛出异常。

The API also supports the concept of strict expectations: those that, when recorded, only allow invocations during replay that exactly match the recordings (within explicitly specified allowances, when needed), both in the number of matching invocations (exactly one, by default) and in the order they occur. Invocations that occur during replay but fail to match a recorded strict expectation are regarded as unexpected, causing an immediate "unexpected invocation" error, and consequently failing the test. This is achieved by using the StrictExpectations subclass.

API也支持严格expectation，即replay阶段的调用和record阶段的记录在调用数量上（默认为一次）和调用顺序保持一致。如replay阶段发生的调用没有匹配到严格expectation，则出现错误"unexpected invocation"，测试失败。StrictExpectations类提供这个功能。

Note that in the case of strict expectations, all invocations occurring during replay that match recorded expectations are implicitly verified. Any remaining invocations that don't match an expectation are considered unexpected, causing the test to fail. The test will also fail if any recorded strict expectation is missed, ie, if no matching invocations occur during replay.

需要注意的是，在strict expectations情况下，replay阶段所有调用都要匹配到recorded expectations。如果有调用没有匹配到expectations，则测试失败。如果有expectation没有被调用，也会导致测试失败。

We can mix expectations of different levels of strictness in the same test by writing multiple expectation blocks, some regular (using Expectations), others strict (using StrictExpectations). Normally, a given mock field or mock parameter will appear in expectation blocks of a single kind, though.

可以在一个test中定义多个不同限制级别的expectation块。例如，同一个test中定义几个Expectations，几个StrictExpectations。一般的一个mock的field或者parameter以单一种类出现在expectation块中。

Most tests will simply make use of "regular" expectations. Usage of strict expectations is probably more a matter of personal preference.

大多数的测试只用常规的expectations就可以了。

#### 4.1 Strict and non-strict *mocks*
Note that we do not specify that a given mocked type/instance should be *strict* or not. Instead, the strictness for a given mock field/parameter is determined by how it is used in the test. Once the first strict expectation is recorded in a "new StrictExpectations() {...}" block, the associated mocked type/instance is considered to be strict for the whole test; otherwise, it will be *not* strict.

我们不会指定一个给定的mock类型是或者不是strict。决定一个mock field或者parameter是否需要strict取决于如何使用它。一旦在"new StrictExpectations() {...}" block中定义strict expectation，在整个test中，相关联的mocked type/instance将会被认为是strict。

***
### 5. Recording results for an expectation
For a given method with non-void return type, a return value can be recorded through an assignment to the result field. When the method gets called in the replay phase, the specified return value will be returned to the caller. The assignment to result should appear right after the invocation that identifies the recorded expectation, inside an expectation block.

在一个expectation中，如果一个方法的返回类型不是void，需要给result赋值来设置返回结果。当这个方法在replay中被调用，这个指定的返回值就会被返回给调用者。对result的赋值应该紧跟着需要record的方法。

If the test instead needs an exception or error to be thrown when the method is invoked, then the result field can still be used: simply assign the desired throwable instance to it. Note that the recording of exceptions/errors to be thrown is applicable to mocked methods (of any return type) as well as to mocked constructors.

在测试中，如果需要被调用的函数抛出异常，result应该被设置为期望的异常。同样也也适用于构造器。

Multiple consecutive results (values to return and/or throwables to throw) can be recorded for the same expectation, by simply assigning the result field multiple times in a row. The recording of multiple return values and/or exceptions/errors to be thrown can be freely mixed for the same expectation. In the case of recording multiple consecutive return values for a given expectation, a single call to the returns(Object...) method can be made. Also, a single assignment to the result field will achieve the same effect, if the value assigned to it is a list or array containing the consecutive values.

同一个expectation可以记录多个连续的结果，返回值或者throw异常，只需要在同一行为result多次的赋值。同一个expectation的多个返回值可以自由的混合使用。也可以使用returns(Object...)方法为一个expectation记录连续多次的返回值。为result的赋值也可以是一个包含结果的list或数组。

The following example test records both types of results for the methods of a mocked DependencyAbc class, to be used when they are invoked from a UnitUnderTest class. Lets say the implementation of the class under test goes like this:

下面的例子，在Expectations中mock一个DependencyAbc，为DependencyAbc类的不同方法记录了返回值。待测代码如下：

```java
public class UnitUnderTest
{
(1)private final DependencyAbc abc = new DependencyAbc();

   public void doSomething()
   {
(2)   int n = abc.intReturningMethod();

      for (int i = 0; i < n; i++) {
         String s;

         try {
(3)         s = abc.stringReturningMethod();
         }
         catch (SomeCheckedException e) {
            // somehow handle the exception
         }

         // do some other stuff
      }
   }
}
```
A possible test for the doSomething() method could exercise the case where SomeCheckedException gets thrown, after an arbitrary number of successful iterations. Assuming that we want (for whatever reasons) to record a complete set of expectations for the interaction between these two classes, we might write the test below. (Often, it's not desirable or important to specify all invocations to mocked methods and - specially - mocked constructors in a given test. We will address this issue later.)

下面是一个测试doSomething()方法的例子，在成功迭代任意数量后，stringReturningMethod可能会抛出异常SomeCheckedException。

```java
@Test
public void doSomethingHandlesSomeCheckedException(@Mocked final DependencyAbc abc) throws Exception
{
   new Expectations() {{
(1)   new DependencyAbc();

(2)   abc.intReturningMethod(); result = 3;

(3)   abc.stringReturningMethod();
      returns("str1", "str2");
      result = new SomeCheckedException();
   }};

   new UnitUnderTest().doSomething();
}
```
This test records three different expectations. The first one, represented by the call to the DependencyAbc() constructor, merely accounts for the fact that this dependency happens to be instantiated in the code under test through the no-args constructor; no result needs to be specified for such an invocation, except for the occasional exception/error to be thrown (constructors have void return type, so it makes no sense to record return values for them). The second expectation specifies that intReturningMethod() will return 3 when called. The third one specifies a sequence of three consecutive results for stringReturningMethod(), where the last result happens to be an instance of the desired exception, allowing the test to achieve its goal (note that it will only pass if the exception is not propagated out).

这个测试记录了3个不同的expectations。第一个，通过调用DependencyAbc()构造器。这是通过无参数构造器实例化，没有返回结果。第二个expectation指定了返回值3，在被调用时，将返回3。第三个expectation指定了3个连续的结果，最后一个是期望的异常。

***
### 6. Matching invocations to specific instances
Previously, we explained that an expectation recorded on a mocked instance, such as "abc.someMethod();" would actually match invocations to DependencyAbc#someMethod() on any instance of the mocked DependencyAbc class. In most cases, tested code uses a single instance of a given dependency, so this won't really matter and can be safely ignored, whether the mocked instance is passed into the code under test or created inside it. But what if we need to verify that invocations occur on a specific instance, between several ones that happen to be used in the code under test? Also, what if only one or a few instances of the mocked class should actually be mocked, with other instances of the same class remaining unmocked? (This second case tends to occur more often when classes from the standard Java libraries, or from other third-party libraries, are mocked.) The API provides a mocking annotation, @Injectable, which will only mock one instance of the mocked type, leaving others unaffected. Additionally, we have a couple ways to constrain the matching of expectations to specific @Mocked instances, while still mocking all instances of the mocked class.

之前，我们讲述了在一个mock的实例上记录了一个expectation，例如"abc.someMethod();"实际上会匹配到被mock的DependencyAbc任意的实例的DependencyAbc#someMethod()方法。大多数情况下，对于一个给定的依赖，测试代码使用一个单一的实例，所以不必在意mock的实例是传入待测代码还是在代码中创建它。如果需要确认调用发生在哪个实例上。或者一个mock类的一部分实例需要被mock，另一些实例不需要mock。当class来源于Java库或者其他第三方库时，第二个例子是更常见的。API提供了 @Injectable注解，只mock一个mock类型的实例。不影响其他的实例。此外我们还有一些方法约束expectations匹配到指定的Mock实例上。

#### 6.1 Injectable mocked instances
Suppose we need to test code which works with multiple instances of a given class, some of which we want to mock. If an instance to be mocked can be passed or injected into the code under test, then we can declare an @Injectable mock field or mock parameter for it. This @Injectable instance will be an "exclusive" mocked instance; any other instance of the same mocked type, unless obtained from a separate mock field/parameter, will remain as a regular, non-mocked instance.

假设测试代码需要给定的class的多个实例，其中一部分需要mock。一个被mock的实例被传入或者注入测试代码，我们可以声明一个 @Injectable mock field或者parameter。其他该类型的实例都是非mock的常规实例，除非还有另一个该类型的mock field/parameter。

When using @Injectable, static methods and constructors are also excluded from being mocked. After all, a static method is not associated with any instance of the class, while a constructor is only associated with a newly created (and therefore different) instance.

当使用 @Injectable 时，static方法和构造器将不会被mock。毕竟static方法不属于class的任何实例，而构造器仅和新建实例相关。

For an example, lets say we have the following class to be tested.
```java
public final class ConcatenatingInputStream extends InputStream
{
   private final Queue<InputStream> sequentialInputs;
   private InputStream currentInput;

   public ConcatenatingInputStream(InputStream... sequentialInputs)
   {
      this.sequentialInputs = new LinkedList<InputStream>(Arrays.asList(sequentialInputs));
      currentInput = this.sequentialInputs.poll();
   }

   @Override
   public int read() throws IOException
   {
      if (currentInput == null) return -1;

      int nextByte = currentInput.read();

      if (nextByte >= 0) {
         return nextByte;
      }

      currentInput = sequentialInputs.poll();
      return read();
   }
}
```
This class could easily be tested without mocking by using ByteArrayInputStream objects for input, but lets say we want to make sure that the InputStream#read() method is properly invoked on each input stream passed in the constructor. The following test will achieve this.

```java
@Test
public void concatenateInputStreams(
   @Injectable final InputStream input1, @Injectable final InputStream input2)
   throws Exception
{
   new Expectations() {{
      input1.read(); returns(1, 2, -1);
      input2.read(); returns(3, -1);
   }};

   InputStream concatenatedInput = new ConcatenatingInputStream(input1, input2);
   byte[] buf = new byte[3];
   concatenatedInput.read(buf);

   assertArrayEquals(new byte[] {1, 2, 3}, buf);
}
```
Note that the use of @Injectable is indeed necessary here, since the class under test extends the mocked class, and the method called to exercise ConcatenatingInputStream is actually defined in the base InputStream class. If InputStream was mocked "normally", the read(byte[]) method would always be mocked, regardless of the instance on which it is called.

注意， @Injectable 是必要的，因为待测试的类继承自mock类，被调用方法也定义于父类InputStream。如果InputStream是常规mock，read(byte[])方法也会被mock，不管属于哪个实例。

#### 6.2 Declaring multiple mocked instances
When using @Mocked or @Capturing (and not @Injectable on the same mock field/parameter), we can still match replay invocations to expectations recorded on specific mocked instances. For that, we simply declare multiple mock fields or parameters of the same mocked type, as the next example shows.

使用 @Mocked or @Capturing ，仍然可以在指定的实例上匹配expectations记录和replay调用。只需要声明多个同一类型的mock field或parameter。

```java
@Test
public void matchOnMockInstance(@Mocked final Collaborator mock, @Mocked Collaborator otherInstance)
{
   new Expectations() {{ mock.getValue(); result = 12; }};

   // Exercise code under test with mocked instance passed from the test:
   int result = mock.getValue();
   assertEquals(12, result);

   // If another instance is created inside code under test...
   Collaborator another = new Collaborator();

   // ...we won't get the recorded result, but the default one:
   assertEquals(0, another.getValue());
}
```
The test above will only pass if the tested code (here embedded in the test method itself, for brevity) invokes getValue() on the exact same instance on which the recording invocation was made. This is typically useful when the code under test makes calls on two or more different instances of the same type, and the test wants to verify that a particular invocation occurred on the expected instance.

测试能够pass仅当在正确的实例上调用的getValue()，也就是在Expectations中记录invocation的实例。这种用法很典型，当测试代码需要调用同一类型的不同实例，并且指定的实例上包含特定的调用。

#### 6.3 Instances created with a given constructor
Specifically for future instances that will later get created by code under test, JMockit provides a couple mechanisms through which we can match invocations on them. Both mechanisms require the recording of an expectation on a specific constructor invocation (a "new" expression) of the mocked class.

当需要在测试代码中创建指定的实例时，JMockit提供几个机制。所有的机制需要在expectation中记录constructor invocation，即new语句创建mock class的实例。

The first mechanism involves simply using the new instance obtained from the recorded constructor expectation, when recording expectations on instance methods. Lets see an example.

第一个机制在expectation中记录了使用特定参数的构造器。并mock了doSomething函数。在测试函数中使用同样参数获得的实例的doSomething方法是被mock的。

```java
@Test
public void newCollaboratorsWithDifferentBehaviors(@Mocked Collaborator anyCollaborator)
{
   // Record different behaviors for each set of instances:
   new Expectations() {{
      // One set, instances created with "a value":
      Collaborator col1 = new Collaborator("a value");
      col1.doSomething(anyInt); result = 123;

      // Another set, instances created with "another value":
      Collaborator col2 = new Collaborator("another value");
      col2.doSomething(anyInt); result = new InvalidStateException();
   }};

   // Code under test:
   new Collaborator("a value").doSomething(5); // will return 123
   ...
   new Collaborator("another value").doSomething(0); // will throw the exception
   ...
}
```
In the above test, we declare a single mock field or mock parameter of the desired class, using @Mocked. This mock field/parameter, however, is not used when recording expectations; instead, we use the instances created on instantiation recordings to record further expectations on instance methods. The future instances created with matching constructor invocations will map to those recorded instances. Also, note that it's not necessarily a one-to-one mapping, but a many-to-one mapping, from potentially many future instances to a single instance used for recorded expectations.

在上例中，我们使用 @Mocked 声明了一个mock field或者parameter，但是并没有使用它。在Expectations中，记录了实例化的对象，mock了doSomething方法。在随后的代码中，使用相同的构造函数将会获得recorded instances。这不仅仅只局限于一对一的匹配，也可以有一对多的匹配。一个记录的instance可以有多个使用。

The second mechanism lets us record a replacement instance for those future instances that match a recorded constructor invocation. With this alternative mechanism, we can rewrite the test as follows.

第二种机制，在Expectations中记录了替代的实例，并mock了实例的方法。

```java
@Test
public void newCollaboratorsWithDifferentBehaviors(
   @Mocked final Collaborator col1, @Mocked final Collaborator col2)
{
   new Expectations() {{
      // Map separate sets of future instances to separate mock parameters:
      new Collaborator("a value"); result = col1;
      new Collaborator("another value"); result = col2;

      // Record different behaviors for each set of instances:
      col1.doSomething(anyInt); result = 123;
      col2.doSomething(anyInt); result = new InvalidStateException();
   }};

   // Code under test:
   new Collaborator("a value").doSomething(5); // will return 123
   ...
   new Collaborator("another value").doSomething(0); // will throw the exception
   ...
}
```
Both versions of the test are equivalent. The second one also allows, when combined with partial mocking, for real (non-mocked) instances to be used as replacements.

***
### 7. Flexible matching of argument values
In both the record and verify phases, an invocation to a mocked method or constructor identifies an expectation. If the method/constructor has one or more parameters, then a recorded/verified expectation like doSomething(1, "s", true); will only match an invocation in the replay phase if it has equal argument values. For arguments that are regular objects (not primitives or arrays), the equals(Object) method is used for equality checking. For parameters of array type, equality checking extends to individual elements; therefore, two different array instances having the same length in each dimension and equal corresponding elements are considered equal.

在record和verify阶段，mock方法或者构造器的调用构成一个expectation。如果method/constructor有一个或多个参数，类似于doSomething(1, "s", true);在reply阶段只有参数相同时才能匹配上。对于一个常规对象（非primitives或数组）参数，相等性检查时会调用equals方法。对于数组参数，相等性测试会扩展到每个元素，因此长度相等并且每个元素都相等的数组是相等的。

In a given test, we often don't know exactly what those argument values will be, or they simply aren't essential for what is being tested. So, to allow a recorded or verified invocation to match a whole set of replayed invocations with different argument values, we can specify flexible argument matching constraints instead of actual argument values. This is done by using anyXyz fields and/or withXyz(...) methods. The "any" fields and "with" methods are all defined in mockit.Invocations, which is the base class for all the expectation/verification classes used in tests; therefore, they can be used in expectation as well as verification blocks.

在一个给定的测试中，我们精诚不能确定参数值具体是什么，或者仅仅是他们是什么并不重要，对测试没影响。因此想要record或者verified invocation 匹配到replay阶段使用所有可能的参数集的invocation，就需要灵活的参数替代具体的参数值。需要使用anyXyz fields and/or withXyz(...) methods。mockit.Invocations定义了 "any" fields and "with" methods。这是一个基础类，在expectation以及verification块中使用。

#### 7.1 Using the "any" fields for argument matching
The most common argument matching constraint tends also to be the least restrictive one: to match invocations with any value for a given parameter (of the proper parameter type, of course). For such cases we have a whole set of special argument matching fields, one for each primitive type (and the corresponding wrapper class), one for strings, and a "universal" one of type Object. The test below demonstrates some uses.

最常用的参数匹配约束一般是限制最小的，即对于一个给定的参数，用任意值匹配调用。下面例子中，anyString表示 new UnitUnderTest().doSomething(item) 中调用 abc.voidMethod 方法使用任意字符串作为参数都可以被匹配到，anyLong表示参数可以是任意的long。

```java
@Test
public void someTestMethod(@Mocked final DependencyAbc abc)
{
   final DataItem item = new DataItem(...);

   new Expectations() {{
      // Will match "voidMethod(String, List)" invocations where the first argument is
      // any string and the second any list.
      abc.voidMethod(anyString, (List<?>) any);
   }};

   new UnitUnderTest().doSomething(item);

   new Verifications() {{
      // Matches invocations to the specified method with any value of type long or Long.
      abc.anotherVoidMethod(anyLong);
   }};
}
```
Uses of "any" fields must appear at the actual argument positions in the invocation statement, never before. You can still have regular argument values for other parameters in the same invocation, though. For more details, see the [API documentation](http://jmockit.org/api1x/mockit/Expectations.html#anyInt).

#### 7.2 Using the "with" methods for argument matching
When recording or verifying an expectation, calls to the withXyz(...) methods can occur for any subset of the arguments passed in the invocation. They can be freely mixed with regular argument-passing (using literal values, local variables, etc.). The only requirement is that such calls appear inside the recorded/verified invocation statement, rather than before it. It's not possible, for example, to first assign the result of a call to withNotEqual(val) to a local variable and then use the variable in the invocation statement. An example test using some of the "with" methods is shown below.

当record或者verify一个expectation时，可以使用withXyz(...)方法来需要匹配到的参数集。唯一需要注意的是这些方法只能用在recorded/verified中的调用语句中。

```java
@Test
public void someTestMethod(@Mocked final DependencyAbc abc)
{
   final DataItem item = new DataItem(...);

   new Expectations() {{
      // Will match "voidMethod(String, List)" invocations with the first argument
      // equal to "str" and the second not null.
      abc.voidMethod("str", (List<?>) withNotNull());

      // Will match invocations to DependencyAbc#stringReturningMethod(DataItem, String)
      // with the first argument pointing to "item" and the second one containing "xyz".
      abc.stringReturningMethod(withSameInstance(item), withSubstring("xyz"));
   }};

   new UnitUnderTest().doSomething(item);

   new Verifications() {{
      // Matches invocations to the specified method with any long-valued argument.
      abc.anotherVoidMethod(withAny(1L));
   }};
}
```
There are more "with" methods than shown above. See the API documentation for more details.

Besides the several predefined argument matching constraints available in the API, JMockit allows the user to provide custom constraints, through the with(Delegate) and withArgThat(Matcher) methods.

#### 7.3 Using the null value to match any object reference
When using at least one argument matching method or field for a given expectation, we can use a "shortcut" to specify that any object reference should be accepted (for a parameter of reference type). Simply pass the null value instead of a withAny(x) or any argument matcher. In particular, this avoids the need to cast the value to the declared parameter type. However, bear in mind that this behavior is only applicable when at least one explicit argument matcher (either a "with" method or an "any" field) is used for the expectation. When passed in an invocation that uses no matchers, the null value will match only the null reference. In the previous test, we could therefore have written:

在一个至少一个参数匹配方法或者field的expectation中，可以使用一个"简写"来指定任意对象引用都能够匹配到。仅需要用null替代withAny(x)或者任意参数匹配。特别是，这避免了将类型强制转换到声明的参数类型。需要注意的是，这只适用于至少有一个显示匹配参数(使用with方法或者any field)的情况。

```java
@Test
public void someTestMethod(@Mocked final DependencyAbc abc)
{
   ...
   new Expectations() {{
      abc.voidMethod(anyString, null);
   }};
   ...
}
```
To specifically verify that a given parameter receives the null reference, the withNull() matcher can be used.
如果要验证入参为null，可以使用withNull()方法。

#### 7.4 Matching values passed through a varargs parameter
Occasionally we may need to deal with expectations for "varargs" methods or constructors. It's valid to pass regular values as a varargs argument, and also valid to use the "with"/"any" matchers for such values. However, it's not valid to combine both kinds of value-passing for the same expectation, when there is a varargs parameter. We need to either use only regular values or only values obtained through argument matchers.

"with"/"any"参数可以用在变参数场景。要么使用常规参数，要么使用匹配参数（"with"/"any"参数），不能混合使用。

In case we want to match invocations where the varargs parameter receives any number of values (including zero), we can specify an expectation with the "(Object[]) any" constraint for the final varargs parameter.

如果要匹配的变参需要接收任意数量的值，可以指定"(Object[]) any"作为参数。

***
### 8. Specifying invocation count constraints
So far, we saw that besides an associated method or constructor, an expectation can have invocation results and argument matchers. Given that code under test can call the same method or constructor multiple times with different or identical arguments, we sometimes need a way to account for all those separate invocations.

一个expectation能够关联一个方法，并设置匹配参数和返回结果。被测代码可以多次调用mock方法，并传入任意或者特定参数。有时候我们需要一个方法对调用进行统计。

The number of invocations expected and/or allowed to match a given expectation can be specified through invocation count constraints. The mocking API provides three special fields just for that: times, minTimes, and maxTimes. These fields can be used either when recording or when verifying expectations. In either case, the method or constructor associated with the expectation will be constrained to receive a number of invocations that falls in the specified range. Any invocations less or more than the expected lower or upper limit, respectively, and the test execution will automatically fail. Lets see some example tests.

在expectation中可以设置期望的调用次数。API提供了三个field：times, minTimes, and maxTimes。这三个field可以用在record和verify阶段。设置这几个field的方法的调用次数必须在期望范围内，否则测试失败。

```java
@Test
public void someTestMethod(@Mocked final DependencyAbc abc)
{
   new Expectations() {{
      // By default, at least one invocation is expected, i.e. "minTimes = 1":
      new DependencyAbc();

      // At least two invocations are expected:
      abc.voidMethod(); minTimes = 2;

      // 1 to 5 invocations are expected:
      abc.stringReturningMethod(); minTimes = 1; maxTimes = 5;
   }};

   new UnitUnderTest().doSomething();
}

@Test
public void someOtherTestMethod(@Mocked final DependencyAbc abc)
{
   new UnitUnderTest().doSomething();

   new Verifications() {{
      // Verifies that zero or one invocations occurred, with the specified argument value:
      abc.anotherVoidMethod(3); maxTimes = 1;

      // Verifies the occurrence of at least one invocation with the specified arguments:
      DependencyAbc.someStaticMethod("test", false); // "minTimes = 1" is implied
   }};
}
```
Unlike the result field, each of these three fields can be specified at most once for a given expectation. Any non-negative integer value is valid for any of the invocation count constraints. If times = 0 or maxTimes = 0 is specified, the first invocation matching the expectation to occur during replay (if any) will cause the test to fail.

和result field不一样，这三个field在一个expectation中最多只能被设置一次。只能设置为非负整数。可以被设置为0.但是一旦被调用，则测试失败。

***
### 9. Explicit verification
Besides specifying invocation count constraints on recorded expectations, we can also verify matching invocations explicitly in a verification block, after the call to the code under test. This is valid for regular expectations, but not for strict expectations, since they are always verified implicitly; there is no point in re-verifying them in a explicit verification block.

除了在recorded expectations中设置调用次数，也可以在待测代码运行后，在verification块中显示验证调用，verification只能和常规expectations配合使用，因为strict expectation总是隐式的验证，所以无需再在verification中验证。

Inside a "new Verifications() {...}" block we can use the same API that's available in a "new Expectations() {...}" block, with the exception of methods and fields used to record return values and thrown exceptions/errors. That is, we can freely use the anyXyz fields, the withXyz(...) argument matching methods, and the times, minTimes, and maxTimes invocation count constraint fields. An example test follows.

能够用在"new Expectations() {...}"块中的API，也同样能够用在在"new Verifications() {...}"块中。

```java
@Test
public void verifyInvocationsExplicitlyAtEndOfTest(@Mocked final Dependency mock)
{
   // Nothing recorded here, though it could be.

   // Inside tested code:
   Dependency dependency = new Dependency();
   dependency.doSomething(123, true, "abc-xyz");

   // Verifies that Dependency#doSomething(int, boolean, String) was called at least once,
   // with arguments that obey the specified constraints:
   new Verifications() {{ mock.doSomething(anyInt, true, withPrefix("abc")); }};
}
```
Note that, by default, a verification checks that at least one matching invocation occurred during replay. When we need to verify an exact number of invocations (including 1), the times = n constraint must be specified.

默认的，一个verification检测匹配到的调用至少在replay中被调用一次。如果需要验证精确的调用次数，需要设置times=n。

#### 9.1 Verifying that an invocation never happened
To do this inside a verification block, add a "times = 0" assignment right after the invocation that is expected to not have happened during the replay phase. If one or more matching invocations did happen, the test will fail.

要验证在replay阶段的方法调用没有发生，需要在verification块中设置times = 0。如果发生一次或多次调用，测试失败。

#### 9.2 Verification in order
Regular verification blocks created with the Verifications class are unordered. The actual relative order in which aMethod() and anotherMethod() were called during the replay phase is not verified, but only that each method was executed at least once. If you want to verify the relative order of invocations, then a "new VerificationsInOrder() {...}" block must be used instead. Inside this block, simply write invocations to one or more mocked types in the order they are expected to have occurred.

通过Verifications class创建常规的verification块是无序的。不检测在replay阶段调用函数的顺序，只检测函数是否被至少调用一次。如果想要验证调用顺序，可以使用"new VerificationsInOrder() {...}" 块。

```java
@Test
public void verifyingExpectationsInOrder(@Mocked final DependencyAbc abc)
{
   // Somewhere inside the tested code:
   abc.aMethod();
   abc.doSomething("blah", 123);
   abc.anotherMethod(5);
   ...

   new VerificationsInOrder() {{
      // The order of these invocations must be the same as the order
      // of occurrence during replay of the matching invocations.
      abc.aMethod();
      abc.anotherMethod(anyInt);
   }};
}
```
Note that the call abc.doSomething(...) was not verified in the test, so it could have occurred at any time (or not at all).

因为并没有验证abc.doSomething(...)，因此在replay阶段调用这个函数对结果没有影响。

#### 9.3 Partially ordered verification
Suppose you want to verify that a particular method (or constructor) was called before/after other invocations, but you don't care about the order in which those other invocations occurred. Inside an ordered verification block, this can be achieved by simply calling the unverifiedInvocations() method at the appropriate place(s). The following test demonstrates it.

假设需要验证这样的情况，一个特定的方法在其他方法执行之前或之后才会调用，但是又不关心其他方法的执行顺序。在VerificationsInOrder块中，只需要在适当的地方调用unverifiedInvocations()方法。

```java
@Mocked DependencyAbc abc;
@Mocked AnotherDependency xyz;

@Test
public void verifyingTheOrderOfSomeExpectationsRelativeToAllOthers()
{
   new UnitUnderTest().doSomething();

   new VerificationsInOrder() {{
      abc.methodThatNeedsToExecuteFirst();
      unverifiedInvocations(); // Invocations not verified must come here...
      xyz.method1();
      abc.method2();
      unverifiedInvocations(); // ... and/or here.
      xyz.methodThatNeedsToExecuteLast();
   }};
}
```
The example above is actually quite sophisticated, as it verifies several things:
+ a) a method that must be called before others;
+ b) a method that must be called after others; and
+ c) that AnotherDependency#method1() must be called just before DependencyAbc#method2().

上面的例子验证了下面几件事情：
+ a) 一个方法必须在其他方法之前调用。
+ b) 一个方法必须在其他方法调用之后。
+ c) AnotherDependency#method1() 必须在 DependencyAbc#method2() 之前调用.

In most tests, we will probably only do one of these different kinds of order-related verifications. But the power is there to make all kinds of complex verifications quite easily.

在大多数测试中，我们可能只需要使用其中的一种顺序验证。

Another situation not covered by the examples above is one where we want to verify that certain invocations occurred in a given relative order, while also verifying the other invocations (in any order). For this, we need to write two separate verification blocks, as illustrated below (where mock is a mock field of the test class).

另一个前面没有提到的场景是，我们需要验证一个特定的调用顺序，同时还需要验证另一个调用顺序，也就是说在同一个执行流程中，存在着多个顺序。只需要两个单独的verification块。

```java
@Test
public void verifyFirstAndLastCallsWithOthersInBetweenInAnyOrder()
{
   // Invocations that occur while exercising the code under test:
   mock.prepare();
   mock.setSomethingElse("anotherValue");
   mock.setSomething(123);
   mock.notifyBeforeSave();
   mock.save();

   new VerificationsInOrder() {{
      mock.prepare(); // first expected call
      unverifiedInvocations(); // others at this point
      mock.notifyBeforeSave(); // just before last
      mock.save(); times = 1; // last expected call
   }};

   // Unordered verification of the invocations previously left unverified.
   // Could be ordered, but then it would be simpler to just include these invocations
   // in the previous block, at the place where "unverifiedInvocations()" is called.
   new Verifications() {{
      mock.setSomething(123);
      mock.setSomethingElse(anyString);
   }};
}
```
Usually, when a test has multiple verification blocks their relative order of execution is important. In the previous test, for example, if the unordered block came before it would have left no "unverified invocations" to match a later call to unverifiedInvocations(); the test would still pass (assuming it originally passed) since it's not required that unverified invocations actually occurred at the called position, but it would not have verified that the unordered group of invocations occurred between the first and last expected calls.

一般的，当测试包含多个verification块时，他们的顺序很重要。在前一个例子中，如果无序块在有序块之前，则后面有序verification块的unverifiedInvocations()将匹配不到"unverified invocations"。测试依然会pass因为实际上unverified invocations调用是0个还是多个都可以，这并不影响测试结果。

#### 9.4 Full verification
Sometimes it may be important to have all invocations to the mocked types involved in a test verified. This is automatically the case when recording strict expectations, since any unexpected invocation causes the test to fail. When regular expectations are explicitly verified, though, a "new FullVerifications() {...}" block can be used to make sure that no invocations are left unverified.

有时，对所有的被mock的方法的调用都需要被验证。如果在record中使用 strict expectation，这些都是自动完成的，因为任何非预期的调用都会导致测试失败。当显式验证常规expectations时，可以使用"new FullVerifications() {...}"块来保证所有的调用都被验证到。

```java
@Test
public void verifyAllInvocations(@Mocked final Dependency mock)
{
   // Code under test included here for easy reference:
   mock.setSomething(123);
   mock.setSomethingElse("anotherValue");
   mock.setSomething(45);
   mock.save();

   new FullVerifications() {{
      // Verifications here are unordered, so the following invocations could be in any order.
      mock.setSomething(anyInt); // verifies two actual invocations
      mock.setSomethingElse(anyString);
      mock.save(); // if this verification (or any other above) is removed the test will fail
   }};
}
```
Note that if a lower limit (a minimum invocation count constraint) is specified for an expectation, then this constraint will always be implicitly verified at the end of the test. Therefore, explicitly verifying such an expectation inside the full verification block is not necessary.

注意，如果一个expectation指定了调用数量约束，那么这个约束将在测试结束时隐式的验证。无需在full verification块中显示的验证。

#### 9.5 Full verification in order
So, we have seen how to do unordered verifications with Verifications, ordered verifications with VerificationsInOrder, and full verifications with FullVerifications. But what about full ordered verifications? Easy enough:

一个验证全部的，有序的verifications例子：

```java
@Test
public void verifyAllInvocationsInOrder(@Mocked final Dependency mock)
{
   // Code under test included here for easy reference:
   mock.setSomething(123);
   mock.setSomethingElse("anotherValue");
   mock.setSomething(45);
   mock.save();

   new FullVerificationsInOrder() {{
      mock.setSomething(anyInt);
      mock.setSomethingElse(anyString);
      mock.setSomething(anyInt);
      mock.save();
   }};
}
```
Notice there is a not so obvious difference in semantics, though. In the verifyAllInvocations test above, we were able to match two separate mock.setSomething(...) invocations with a single invocation in the verification block. In the verifyAllInvocationsInOrder test, however, we had to write two separate invocations to that method inside the block, in the proper order with respect to other invocations.

在verifyAllInvocations例子中，在verification块中，我们用一个单独的invocation来匹配两个mock.setSomething(...)调用。而在verifyAllInvocationsInOrder例子中，我们不得不用两个单独的invocations来表明invocations之间的顺序。

#### 9.6 Restricting the set of mocked types to be fully verified
By default, all invocations to all mocked instances/types in effect for a given test must be verified explicitly when using a "new FullVerifications() {}" or "new FullVerificationsInOrder() {}" block. Now, what if we have a test with two (or more) mocked types but we only want to fully verify invocations to one of them (or to any subset of mocked types when more than two)? The answer is to use the FullVerifications(mockedTypesAndInstancesToVerify) constructor, where only the given mocked instances and mocked types (ie, class objects/literals) are considered. The following test provides an example.

默认的，在FullVerifications块或者FullVerificationsInOrder块中，所有mocked instances/types的调用必须被验证。如果有两个mock type，但只需要验证其中之一，该怎么办？可以使用FullVerifications(mockedTypesAndInstancesToVerify)构造器。下面是例子：

```java
@Test
public void verifyAllInvocationsToOnlyOneOfTwoMockedTypes(
   @Mocked final Dependency mock1, @Mocked AnotherDependency mock2)
{
   // Inside code under test:
   mock1.prepare();
   mock1.setSomething(123);
   mock2.doSomething();
   mock1.editABunchMoreStuff();
   mock1.save();

   new FullVerifications(mock1) {{
      mock1.prepare();
      mock1.setSomething(anyInt);
      mock1.editABunchMoreStuff();
      mock1.save(); times = 1;
   }};
}
```
In the test above, the mock2.doSomething() invocation is never verified.

To restrict verification only to the methods/constructors of a single mocked class, pass the class literal to the FullVerifications(...) or FullVerificationsInOrder(...) constructor. For example, the new FullVerificationsInOrder(AnotherDependency.class) { ... } block would only make sure that all invocations to the mocked AnotherDependency class were verified.

如果限定verification只验证某一个类的mock方法，则可以将这个类作为参数传入FullVerifications(...) 或 FullVerificationsInOrder(...)。例如，new FullVerificationsInOrder(AnotherDependency.class) { ... }。这个块中将会验证所有的AnotherDependency类的调用。

#### 9.7 Verifying that no invocations occurred
To verify that no invocations at all occurred on the mocked types/instances used in a test, add an empty full verification block to it. As always, note that any expectations that were recorded as expected through a specified times/minTimes constraint are verified implicitly and therefore disregarded by the full verification block; in such a case the empty verification block will verify that no other invocations occurred. Additionally, if any expectations were verified in a previous verification block in the same test, they are also disregarded by the full verification block.

想要验证没有调用mock方法，可以添加一个空的full verification block。expectations块中定义的有调用次数限制的invocations，则会被隐式的验证，不需要显示验证。另外如果任意的expectations被前一个verification验证，则会被后面的full verification block忽略。

If the test uses two or more mocked types/instances and you want to verify that no invocations occurred for some of them, specify the desired mocked types and/or instances in the constructor to the empty verification block. An example test follows.

如果测试使用两个或更多的mock类型或实例，并希望其中的一些没有被调用。只需要将想要验证的类型或实例作为参数传入FullVerifications构造器。

```java
@Test
public void verifyNoInvocationsOnOneOfTwoMockedDependenciesBeyondThoseRecordedAsExpected(
   @Mocked final Dependency mock1, @Mocked final AnotherDependency mock2)
{
   new Expectations() {{
      // These two are recorded as expected:
      mock1.setSomething(anyInt);
      mock2.doSomething(); times = 1;
   }};

   // Inside code under test:
   mock1.prepare();
   mock1.setSomething(1);
   mock1.setSomething(2);
   mock1.save();
   mock2.doSomething();

   // Will verify that no invocations other than to "doSomething()" occurred on mock2:
   new FullVerifications(mock2) {};
}
```

#### 9.8 Verifying unspecified invocations that should not happen
A full verification block (ordered or not) also allows us to verify that certain methods and/or constructors never get invoked, without having to record or verify each one of them with a corresponding times = 0 assignment. The following test provides an example.

全验证快也可以用来验证指定方法不被调用。不需要在record或者verify阶段为每个方法设置times = 0。

```java
@Test
public void readOnlyOperation(@Mocked final Dependency mock)
{
   new Expectations() {{
      mock.getData(); result = "test data";
   }};

   // Code under test:
   String data = mock.getData();
   // mock.save() should not be called here
   ...

   new FullVerifications() {{
      mock.getData(); minTimes = 0; // calls to getData() are allowed, others are not
   }};
}
```
If a call to any method (or constructor) of the Dependency class occurs during the replay phase, except for the ones explicitly verified in the verification block (Dependency#getData() in this case), then the test above will fail. On the other hand, it may be easier to use strict expectations in such cases, without any verification block at all.
除了在验证块中显示验证的方法，在replay阶段还有别的依赖类的方法被调用，则上个测试会失败。使用严格期望块，不用验证块可能会简单一些。


***
### 10. Capturing invocation arguments for verification
Invocation arguments can be captured for later verification through a set of special "withCapture(...)" methods. There are three different cases, each with its own specific capturing method: 1) verification of arguments passed to a mocked method, in a single invocation: T withCapture(); 2) verification of arguments passed to a mocked method, in multiple invocations: T withCapture(List<T>); and 3) verification of arguments passed to a mocked constructor: List<T> withCapture(T).

通过一系列的withCapture(...)方法可以在验证块中捕获参数，有3种不同的情况，每种情况都有指定的捕获方法。
+ 1) 参数传递到mock方法，一次调用，T withCapture()。
+ 2) 参数传递到mock方法，多重调用，T withCapture(List<T>)， 捕获多次调用的参数放到一个列表中，它的返回值只会保留最后一次的调用参数，需要获得全部捕获的值必须传递一个 List<T> 给 withCapture 方法。。
+ 3) 参数传递到mock 构造器，List<T> withCapture(T)。

#### 10.1 Capturing arguments from a single invocation
To capture arguments from a single invocation to a mocked method or constructor, we use "withCapture()", as the following example test demonstrates.
```java
@Test
public void capturingArgumentsFromSingleInvocation(@Mocked final Collaborator mock)
{
   // Inside tested code:
   new Collaborator().doSomething(0.5, new int[2], "test");

   new Verifications() {{
      double d;
      String s;
      mock.doSomething(d = withCapture(), null, s = withCapture());

      assertTrue(d > 0.0);
      assertTrue(s.length() > 1);
   }};
}
```
The withCapture() method can only be used in verification blocks. Typically, we use it when a single matching invocation is expected to occur; if more than one such invocation occurs, however, the last one to occur overwrites the values captured by previous ones. It is particularly useful with parameters of a complex type (think a JPA @Entity), which may contain several items whose values need to be checked.

withCapture()只能被用在verification块。但如果调用发生多次，则只保留最后一次捕获的参数，所以一般都是用于期望的调用被执行一次的情况。

#### 10.2 Capturing arguments from multiple invocations
If multiple invocations to a mocked method or constructor are expected, and we want to capture values for all of them, then the withCapture(List) method should be used instead, as in the example below.

如果一个mock方法被多次调用，又希望捕获所有的参数，就可以使用withCapture(List)方法。

```java
@Test
public void capturingArgumentsFromMultipleInvocations(@Mocked final Collaborator mock)
{
   mock.doSomething(dataObject1);
   mock.doSomething(dataObject2);

   new Verifications() {{
      List<DataObject> dataObjects = new ArrayList<>();
      mock.doSomething(withCapture(dataObjects));

      assertEquals(2, dataObjects.size());
      DataObject data1 = dataObjects.get(0);
      DataObject data2 = dataObjects.get(1);
      // Perform arbitrary assertions on data1 and data2.
   }};
}
```
Differently from withCapture(), the withCapture(List) overload can also be used in expectation recording blocks.
withCapture()只能用于验证块，withCapture(List)还可以用在期望块。

#### 10.3 Capturing new instances
Finally, we can capture the new instances of a mocked class that got created during the test.

捕获mock类的新建实例。

```java
@Test
public void capturingNewInstances(@Mocked Person mockedPerson)
{
   // From the code under test:
   dao.create(new Person("Paul", 10));
   dao.create(new Person("Mary", 15));
   dao.create(new Person("Joe", 20));

   new Verifications() {{
      // Captures the new instances created with a specific constructor.
      List<Person> personsInstantiated = withCapture(new Person(anyString, anyInt));

      // Now captures the instances of the same type passed to a method.
      List<Person> personsCreated = new ArrayList<>();
      dao.create(withCapture(personsCreated));

      // Finally, verifies both lists are the same.
      assertEquals(personsInstantiated, personsCreated);
   }};
}
```
***
### 11. Delegates: specifying custom results
We have seen how to record results for invocations through assignments to the result field or calls to the returns(...) method. We have also seen how to match invocation arguments flexibly with the withXyz(...) group of methods and the various anyXyz fields. But what if a test needs to decide the result of a recorded invocation based on the arguments it will receive at replay time? We can do it through a Delegate instance, as exemplified below.

通过向result field赋值可以设置一个期望调用的返回结果。如果一个调用记录的结果取决于replay阶段的入参，该如何设置result？可以创建一个Delegate实例。

```java
@Test
public void delegatingInvocationsToACustomDelegate(@Mocked final DependencyAbc anyAbc)
{
   new Expectations() {{
      anyAbc.intReturningMethod(anyInt, null);
      result = new Delegate() {
         int aDelegateMethod(int i, String s)
         {
            return i == 1 ? i : s.length();
         }
      };
   }};

   // Calls to "intReturningMethod(int, String)" will execute the delegate method above.
   new UnitUnderTest().doSomething();
}
```
The Delegate interface is empty, being used simply to tell JMockit that actual invocations at replay time should be delegated to the "delegate" method in the assigned object. This method can have any name, provided it is the only non-private method in the delegate object. As for the parameters of the delegate method, they should either match the parameters of the recorded method, or there should be none. In any case, the delegate method is allowed to have an additional parameter of type Invocation as its first parameter. (The Invocation object received during replay will provide access to the invoked instance and the actual invocation arguments, along with other abilities.) The return type of a delegate method doesn't have to be the same as the recorded method, although it should be compatible in order to avoid a ClassCastException later.

Delegate接口是空的，仅用来告诉JMockit在replay阶段对mock方法的调用被代理到Delegate实例(赋值给result)的"代理"方法。这个方法可以是任意名称，只要保证它是非private方法。代理方法的参数要么和被记录方法保持一致，要么为空。还有，delegate 方法允许包含一个额外的Invocation类型参数作为第一个参数。(Invocation对象在replay阶段被传入，可以访问调用实例和获取实际调用参数，还有其他功能)代理方法的返回类型可以和记录方法不一致，一般都是一致的，否则会有ClassCastException异常。

Constructors can also be handled through delegate methods. The following example test shows a constructor invocation being delegated to a method which conditionally throws an exception.

```java
@Test
public void delegatingConstructorInvocations(@Mocked Collaborator anyCollaboratorInstance)
{
   new Expectations() {{
      new Collaborator(anyInt);
      result = new Delegate() {
         void delegate(int i) { if (i < 1) throw new IllegalArgumentException(); }
      };
   }};

   // The first instantiation using "Collaborator(int)" will execute the delegate above.
   new Collaborator(4);
}
```

### 12. Cascading mocks
When using complex APIs where functionality is distributed through many different objects, it is not uncommon to see chained invocations of the form obj1.getObj2(...).getYetAnotherObj().doSomething(...). In such cases it may be necessary to mock all objects/classes in the chain, starting with obj1.

有些复杂的API，功能分布在许多不同的对象上，这样的情况并不少见，例如链式调用，obj1.getObj2(...).getYetAnotherObj().doSomething(...)。这种情况下，需要mock从obj1开始链上所有的对象和类。

All three mocking annotations provide this ability. The following test shows a basic example, using the java.net and java.nio APIs.


```java
@Test
public void recordAndVerifyExpectationsOnCascadedMocks(
   @Mocked Socket anySocket, // will match any new Socket object created during the test
   @Mocked final SocketChannel cascadedChannel // will match cascaded instances
) throws Exception
{
   new Expectations() {{
      // Calls to Socket#getChannel() will automatically return a cascaded SocketChannel;
      // such an instance will be the same as the second mock parameter, allowing us to
      // use it for expectations that will match all cascaded channel instances:
      cascadedChannel.isConnected(); result = false;
   }};

   // Inside production code:
   Socket sk = new Socket(); // mocked as "anySocket"
   SocketChannel ch = sk.getChannel(); // mocked as "cascadedChannel"

   if (!ch.isConnected()) {
      SocketAddress sa = new InetSocketAddress("remoteHost", 123);
      ch.connect(sa);
   }

   InetAddress adr1 = sk.getInetAddress();  // returns a newly created InetAddress instance
   InetAddress adr2 = sk.getLocalAddress(); // returns another new instance
   ...

   // Back in test code:
   new Verifications() {{ cascadedChannel.connect((SocketAddress) withNotNull()); }};
}
```
In the test above, calls to eligible methods in the mocked Socket class will return a cascaded mock object whenever they occur during the test. The cascaded mock will allow further cascading, so a null reference will never be obtained from methods which return object references (except for non-eligible return types Object or String which will return null, or collection types which will return a non-mocked empty collection).

不论在测试的什么阶段，调用mock Socket类的相应的方法会返回一个级联的mock对象。级联mock允许更深层次的级联，返回对象的引用绝不会返回null。

Unless there is an available mocked instance from a mock field/parameter (such as cascadedChannel above), a new cascaded instance will get created from the first call to each mocked method. In the example above, the two different methods with the same InetAddress return type will create and return different cascaded instances; the same method will always return the same cascaded instance, though.

New cascaded instances are created with @Injectable semantics, so as to not affect other instances of the same type that may exist during the test.

Finally, it's worth noting that, if necessary, cascaded instances can be replaced with non-mocked ones, with a different mocked instance, or not be returned at all; for that, record an expectation which assigns the result field with the desired instance to be returned, or with null if no such instance is desired.

#### 12.1 Cascading static factory methods
Cascading is extremely useful in scenarios where a mocked class contains static factory methods. In the following example test, lets say we want to mock the javax.faces.context.FacesContext class from JSF (Java EE).

```java
@Test
public void postErrorMessageToUIForInvalidInputFields(@Mocked final FacesContext jsf)
{
   // Set up invalid inputs, somehow.

   // Code under test which validates input fields from a JSF page, adding
   // error messages to the JSF context in case of validation failures.
   FacesContext ctx = FacesContext.getCurrentInstance();

   if (some input is invalid) {
      ctx.addMessage(null, new FacesMessage("Input xyz is invalid: blah blah..."));
   }
   ...

   // Test code: verify appropriate error message was added to context.
   new Verifications() {{
      FacesMessage msg;
      jsf.addMessage(null, msg = withCapture());
      assertTrue(msg.getSummary().contains("blah blah"));
   }};
}
```
What's interesting in the test above is that we never have to worry about FacesContext.getCurrentInstance(), as the "jsf" mocked instance gets automatically returned.

#### 12.2 Cascading self-returning methods
Another scenario where cascading tends to help is when code under test uses a "fluent interface", where a "builder" object returns itself from most of its methods. So, we end up with a method call chain which produces some final object or state. In the example test below we mock the java.lang.ProcessBuilder class.

```java
@Test
public void createOSProcessToCopyTempFiles(@Mocked final ProcessBuilder pb) throws Exception
{
   // Code under test creates a new process to execute an OS-specific command.
   String cmdLine = "copy /Y *.txt D:\\TEMP";
   File wrkDir = new File("C:\\TEMP");
   Process copy = new ProcessBuilder().command(cmdLine).directory(wrkDir).inheritIO().start();
   int exit = copy.waitFor();
   ...

   // Verify the desired process was created with the correct command.
   new Verifications() {{ pb.command(withSubstring("copy")).start(); }};
}
```
Above, methods command(...), directory(...), and inheritIO() configure the process to be created, while start() finally creates it. The mocked process builder object automatically returns itself ("pb") from these calls, while also returning a new mocked Process from the call to start().

***
### 13. Partial mocking
By default, all methods and constructors which can be called on a mocked type and its super-types (except for java.lang.Object) get mocked. This is appropriate for most tests, but in some situations we might need to select only certain methods or constructors to be mocked. Methods/constructors not mocked in an otherwise mocked type will execute normally when called.

一个mock类型默认所有的方法和除了Object的父类方法都被mock。有些场景却需要mock指定的方法或者构造器，没有mock的方法不受影响。

When a class or object is partially mocked, JMockit decides whether to execute the real implementation of a method or constructor as it gets called from the code under test, based on which expectations were recorded and which were not. The following example tests will demonstrate it.

当一个类或者对象部分mock，JMockit依靠期望块中的记录来决定哪些方法要调用真实实现。

```java
public class PartialMockingTest
{
   static class Collaborator
   {
      final int value;

      Collaborator() { value = -1; }
      Collaborator(int value) { this.value = value; }

      int getValue() { return value; }
      final boolean simpleOperation(int a, String b, Date c) { return true; }
      static void doSomething(boolean b, String s) { throw new IllegalStateException(); }
   }

   @Test
   public void partiallyMockingAClassAndItsInstances()
   {
      final Collaborator anyInstance = new Collaborator();

      new Expectations(Collaborator.class) {{
         anyInstance.getValue(); result = 123;
      }};

      // Not mocked, as no constructor expectations were recorded:
      Collaborator c1 = new Collaborator();
      Collaborator c2 = new Collaborator(150);

      // Mocked, as a matching method expectation was recorded:
      assertEquals(123, c1.getValue());
      assertEquals(123, c2.getValue());

      // Not mocked:
      assertTrue(c1.simpleOperation(1, "b", null));
      assertEquals(45, new Collaborator(45).value);
   }

   @Test
   public void partiallyMockingASingleInstance()
   {
      final Collaborator collaborator = new Collaborator(2);

      new Expectations(collaborator) {{
         collaborator.getValue(); result = 123;
         collaborator.simpleOperation(1, "", null); result = false;

         // Static methods can be dynamically mocked too.
         Collaborator.doSomething(anyBoolean, "test");
      }};

      // Mocked:
      assertEquals(123, collaborator.getValue());
      assertFalse(collaborator.simpleOperation(1, "", null));
      Collaborator.doSomething(true, "test");

      // Not mocked:
      assertEquals(2, collaborator.value);
      assertEquals(45, new Collaborator(45).getValue());
      assertEquals(-1, new Collaborator().getValue());
   }
}
```
As shown above, the Expectations(Object...) constructor accepts one or more classes or objects to be partially mocked. If a Class object is given, all methods and constructors defined in that class can be mocked, as well as the methods and constructors of its super-classes; all instances of the specified class will be regarded as mocked instances. If, on the other hand, a regular instance is given, then only methods, not constructors, in the class hierarchy can be mocked; even more, only that particular instance will be mocked.

如上所示，Expectations(Object...)的构造器接受一个或多个类或者对象作为参数，并将其部分mock。如果Expectations的参数是class，则该类以及父类中的所有的方法和构造器都能够被mock。并且这个class的所有实例都将被视为被mock的实例。如果Expectations的参数是一个实例，则只有方法能够被mock，不包括构造器，仅限于class层面(应该是不包括父类的意思)，并且只有这个被传入的实例被mock，其他该类的实例没有被mock。

Notice that in these two example tests there is no mock field or mock parameter. The partial mocking constructor effectively provides yet another way to specify mocked types. It also lets us turn objects stored in local variables into mocked instances. Such objects can be created with any amount of state in internal instance fields; they will keep that state when mocked.

注意，这两个例子中没有出现mock field和parameter(就是前面讲过的带@Mocked注解的入参和field)。

It should be noted that, when we request a class or instance to be partially mocked, it can also have invocations verified on it, even if the verified methods/constructors were not recorded. For example, consider the following test.

还需要注意的是，即使将一个class或者实例部分mock，依然可以在invocations中验证它，即使验证的方法或者构造器没有被记录。见下例。

```java
   @Test
   public void partiallyMockingAnObjectJustForVerifications()
   {
      final Collaborator collaborator = new Collaborator(123);

      new Expectations(collaborator) {};

      // No expectations were recorded, so nothing will be mocked.
      int value = collaborator.getValue(); // value == 123
      collaborator.simpleOperation(45, "testing", new Date());
      ...

      // Unmocked methods can still be verified:
      new Verifications() {{ c1.simpleOperation(anyInt, anyString, (Date) any); }};
   }
```
Finally, a simpler way to apply partial mocking to a tested class is to have a field in the test class annotated as both @Tested (see section below) and @Mocked. In this case, the tested object is not passed to the Expectations constructor, but we still need to record expectations on any methods requiring mocked results.

最后，这里有一个简单的mock被测类的方法，在编写测试类时定义一个带 @Tested 和 @Mocked 注解的field，类型是被测类。这种情况下，虽然测试对象并没有作为参数传入Expectations的构造器，但是我们依然需要在expectations块中mock相关的方法。

***
### 14. Capturing implementation classes and instances
Our discussion of this feature will be based on the (contrived) code below.

基于下面例子进行讨论：

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

businessOperation()是我们想要测试的方法。所使用的类实现了service接口。其中的一个实现定义了一个匿名内部类，这在客户代码中是完全不可见的(除非使用反射)。

#### 14.1 Mocking unspecified implementation classes
Given a base type (be it an interface, an abstract class, or any sort of base class), we can write a test which only knows about the base type but where all implementing/extending implementation classes get mocked. To do so, we declare a "capturing" mocked type which refers only to the known base type. Not only will implementation classes already loaded by the JVM get mocked, but also any additional classes that happen to get loaded by the JVM during later test execution. This ability is activated by the @Capturing annotation, which can be applied to mock fields and mock parameters, as demonstrated below.

对于一个给定的基类(可以是接口，抽象类或者是任何形式的基础类)，我们的测试虽然只声明基类，但是所有的实现或继承类都会被mock。下面例子声明一个 capturing 的基类引用。不仅已经被JVM加载的实现类会被mock，而且在接下来的测试中被JVM加载的该类也会被mock。

```java
public final class UnitTest
{
   @Capturing Service anyService;

   @Test
   public void mockingImplementationClassesFromAGivenBaseType()
   {
      new Expectations() {{ anyService.doSomething(); returns(3, 4); }};

      int result = new TestedUnit().businessOperation();

      assertEquals(7, result);
   }
}
```
In the test above, two return values are specified for the Service#doSomething() method. This expectation will match all invocations to this method, regardless of the actual instance on which the invocation occurs, and regardless of the actual class implementing the method.

在上面的测试中，为Service#doSomething()方法指定了两个返回值。不论调用发生在哪个实例，也不论实现这个方法的是哪个类，在expectation中定义的两次mock会匹配到实际发生的两次调用。

### 14.2 Specifying behavior for future instances
An additional ability related to capturing applies to future instances assignable to the mocked type, and is activated through the "maxInstances" optional attribute. This attribute takes an int value specifying the maximum number of future instances of the mocked type that should be covered by the associated mock field/parameter; when not specified, all assignable instances, both pre-existing and to be created during the test, are covered.

capturing能够设定一个实例的调用次数来mock不同的实例，在声明 @Capturing 时设置可选属性maxInstances。这个属性接受一个int值，设置所声明的当前实例最大捕获次数。如果没有设置这个属性，则所有的实例，包括之前存在的以及在接下来的测试中所创建的实例都将被mock。

The expectations recorded and/or verified on a given capturing mock field or parameter will match invocations to any of the future instances covered by the mock field/parameter. This allows us to record and/or verify different behavior for each set of future instances; for that, we declare two or more capturing mock fields/parameters of the same declared type, each with its own maxInstances value (except perhaps for the last mock field/parameter, which would then cover the remaining future instances).

For the sake of demonstration, the following example test takes control of java.nio.Buffer subclasses and their future instances; in a real test it would be preferable to use real buffers rather than mocked ones.

```java
@Test
public void testWithDifferentBehaviorForFirstNewInstanceAndRemainingNewInstances(
   @Capturing(maxInstances = 1) final Buffer firstNewBuffer,
   @Capturing final Buffer remainingNewBuffers)
{
   new Expectations() {{
      firstNewBuffer.position(); result = 10;
      remainingNewBuffers.position(); result = 20;
   }};

   // Code under test creates several buffers...
   ByteBuffer buffer1 = ByteBuffer.allocate(100);
   IntBuffer  buffer2 = IntBuffer.wrap(new int[] {1, 2, 3});
   CharBuffer buffer3 = CharBuffer.wrap("                ");

   // ... and eventually read their positions, getting 10 for
   // the first buffer created, and 20 for the remaining ones.
   assertEquals(10, buffer1.position());
   assertEquals(20, buffer2.position());
   assertEquals(20, buffer3.position());
}
```
It should be noted that while a capturing mocked type is in scope, all implementation classes will get mocked, regardless of any "maxInstances" limits that may have been specified.

***
### 15. Instantiation and injection of tested classes
A non-final instance field annotated as @Tested in the test class will be considered for automatic instantiation and injection, just before the execution of a test method. If at this time the field still holds the null reference, an instance will be created using a suitable constructor of the tested class, while making sure its internal dependencies get properly injected (when applicable). If the field has already been initialized (not null), then nothing will be done.

一个声明了@Tested的non-final实例field将会在执行测试方法前自动实例化和注入。如果执行测试前发现field是null，则会调用合适的构造器创建实例，如果field不为null，则什么也不做。

In order to inject mocked instances into the tested object, the test class must also contain one or more mock fields or mock parameters declared to be @Injectable. Mock fields/parameters annotated only with @Mocked or @Capturing are not considered for injection. On the other hand, not all injectable fields/parameters need to have mockable types; they can also have primitive or array types. The following example test class will demonstrate.

想要测试对象自动注入，则需要在测试类中用  @Injectable 注解声明mock field 或者 mock parameter。声明 @Mocked 或者 @Capturing 的field 和 parameter不会自动注入。并不是所有的可注入类型都是可mock的类型，还可以是基础类型和数组。

```java
public class SomeTest
{
   @Tested CodeUnderTest tested;
   @Injectable Dependency dep1;
   @Injectable AnotherDependency dep2;
   @Injectable int someIntegralProperty = 123;

   @Test
   public void someTestMethod(@Injectable("true") boolean flag, @Injectable("Mary") String name)
   {
      // Record expectations on mocked types, if needed.

      tested.exerciseCodeUnderTest();

      // Verify expectations on mocked types, if required.
   }
}
```
Note that a non-mockable injectable field/parameter must have a value explicitly specified to it, otherwise the default value would be used. In the case of an injectable field, the value can simply be assigned to the field. Alternatively, it can be provided in the "value" attribute of @Injectable, which is the only way to specify the value in the case of an injectable test method parameter.

Two forms of injection are supported: constructor injection and field injection. In the first case, the tested class must have a constructor which can be satisfied by the injectables made available in the test class. Note that for a given test, the set of available injectables consists of the set of injectable fields declared as instances fields of the test class plus the set of injectable parameters declared in the test method; therefore, different tests in the same test class can provide different sets of injectables for the same tested class.

Once the tested class is initialized with the chosen constructor, its non-final instance fields are considered for injection. For each such field to be injected, an injectable field of the same type is searched in the test class. If only one is found, its current value is read and then stored in the injected field. If there is more than one, the injected field name is used to select between the injectable fields of same type.
