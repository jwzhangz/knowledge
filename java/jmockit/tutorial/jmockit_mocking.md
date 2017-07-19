## Mocking
![](http://jmockit.org/tutorial/MockingAPI.png)

In the JMockit library, the Expectations API provides rich support for the use of mocking in automated developer tests. When mocking is used, a test focuses on the behavior of the code under test, as expressed through its interactions with other types it depends upon. Mocking is typically used in the construction of isolated unit tests, where a unit under test is exercised in isolation from the implementation of other units it depends on. Typically, a unit of behavior is embodied in a single class, but it's also fine to consider a whole set of strongly-related classes as a single unit for the purposes of unit testing (as is usually the case when we have a central public class with one or more helper classes, possibly package-private); in general, individual methods should not be regarded as separate units on their own.

Strict unit testing, however, is not a recommended approach; one should not attempt to mock every single dependency. Mocking is best used in moderation; whenever possible, favor integration tests over isolated unit tests. This said, mocking is occasionally also useful in the creation of integration tests, when some particular dependency cannot have its real implementation easily used, or when attempting to create tests for corner cases where a well-placed mocked interaction can greatly facilitate the test.

An interaction between two classes always takes the form of a method or constructor invocation. The set of invocations from a tested class to its dependencies, together with the argument and return values passed between them, define the behavior of interest for the tests of that particular class. In addition, a given test may need to verify the relative order of execution between multiple invocations.

***
### 1. Mocked types and instances
Methods and constructors invoked from the code under test on a dependency are the targets for mocking. Mocking provides the mechanism that we need in order to isolate the tested code from (some of) its dependencies. We specify which particular dependencies are to be mocked for a given test (or tests) by declaring suitable mock fields and/or mock parameters; mock fields are declared as annotated instance fields of the test class, while mock parameters are declared as annotated parameters of a test method. The type of the dependency to be mocked will be the type of the mock field or parameter. Such a type can be any kind of reference type: an interface, a class (including abstract and final ones), an annotation, or an enum.

By default, all non-private methods (including any that are static, final, or native) of the mocked type will be mocked for the duration of the test. If the declared mocked type is a class, then all of its super-classes up to but not including java.lang.Object will also be mocked, recursively. Therefore, inherited methods will automatically be mocked as well. Also in the case of a class, all of its non-private constructors will get mocked.

When a method or constructor is mocked, its original implementation code won't be executed for invocations occurring during the test. Instead, the call will be redirected to JMockit so it can be dealt with in the manner that was explicitly or implicitly specified for the test.

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

There are a few different annotations available for the declaration of mock fields and parameters, and ways in which the default mocking behavior can be modified to suit the needs of a particular test. Other sections of this chapter go into the details, but the basics are: @Mocked is the central mocking annotation, having one optional attribute which is useful in certain cases; @Injectable is another mocking annotation, which constrains mocking to the instance methods of a single mocked instance; and @Capturing is yet another mocking annotation, which extends mocking to the classes implementing a mocked interface, or the subclasses extending a mocked class. When @Injectable or @Capturing is applied to a mock field or mock parameter, @Mocked is implied so it doesn't need to (but can) be applied as well.

The mocked instances created by JMockit can be used normally in test code (for the recording and verification of expectations), and/or passed to the code under test. Or they may simply go unused. Differently from other mocking APIs, these mocked objects don't have to be the ones used by the code under test when it calls instance methods on its dependencies. By default (ie, when @Injectable is not used), JMockit does not care on which object a mocked instance method is called. This allows the transparent mocking of instances created directly inside code under test, when said code invokes constructors on brand new instances using the new operator; the classes instantiated must be covered by mocked types declared in test code, that's all.

***
### 2. Expectations
An expectation represents a set of invocations to a specific mocked method/constructor that is relevant for a given test. An expectation may cover multiple different invocations to the same method or constructor, but it doesn't have to cover all such invocations that occur during the execution of the test. Whether a particular invocation matches a given expectation or not will depend not only on the method/constructor signature but also on runtime aspects such as the instance on which the method is invoked, argument values, and/or the number of invocations already matched. Therefore, several types of matching constraints can (optionally) be specified for a given expectation.

When we have one or more invocation parameters involved, an exact argument value may be specified for each parameter. For example, the value "test string" could be specified for a String parameter, causing the expectation to match only those invocations with this exact value in the corresponding parameter. As we will see later, instead of specifying exact argument values, we can specify more relaxed constraints which will match whole sets of different argument values.

The example below shows an expectation for Dependency#someMethod(int, String), which will match an invocation to this method with the exact argument values as specified. Notice that the expectation itself is specified through an isolated invocation to the mocked method. There are no special API methods involved, as is common in other mocking APIs. This invocation, however, does not count as one of the "real" invocations we are interested in testing. It's only there so that the expectation can be specified.

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

This model of three phases is also known as the Arrange, Act, Assert syntax, or "AAA" for short. Different words, but the meaning is the same.

In the context of behavior-based testing with mocked types (and their mocked instances), we can identify the following alternative phases, which are directly related to the three previously described conventional testing phases:

1. The ***record*** phase, during which invocations can be recorded. This happens during test preparation, before the invocations we want to test are executed.
2. The ***replay*** phase, during which the mock invocations of interest have a chance to be executed, as the code under test is exercised. The invocations to mocked methods/constructors previously recorded will now be replayed. Often there isn't a one-to-one mapping between invocations recorded and replayed, though.
3. The ***verify*** phase, during which invocations can be verified to have occurred as expected. This happens during test verification, after the invocations under test had a chance to be executed.
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

The API also supports the concept of strict expectations: those that, when recorded, only allow invocations during replay that exactly match the recordings (within explicitly specified allowances, when needed), both in the number of matching invocations (exactly one, by default) and in the order they occur. Invocations that occur during replay but fail to match a recorded strict expectation are regarded as unexpected, causing an immediate "unexpected invocation" error, and consequently failing the test. This is achieved by using the StrictExpectations subclass.

Note that in the case of strict expectations, all invocations occurring during replay that match recorded expectations are implicitly verified. Any remaining invocations that don't match an expectation are considered unexpected, causing the test to fail. The test will also fail if any recorded strict expectation is missed, ie, if no matching invocations occur during replay.

We can mix expectations of different levels of strictness in the same test by writing multiple expectation blocks, some regular (using Expectations), others strict (using StrictExpectations). Normally, a given mock field or mock parameter will appear in expectation blocks of a single kind, though.

Most tests will simply make use of "regular" expectations. Usage of strict expectations is probably more a matter of personal preference.

#### 4.1 Strict and non-strict *mocks*
Note that we do not specify that a given mocked type/instance should be *strict* or not. Instead, the strictness for a given mock field/parameter is determined by how it is used in the test. Once the first strict expectation is recorded in a "new StrictExpectations() {...}" block, the associated mocked type/instance is considered to be strict for the whole test; otherwise, it will be *not* strict.

***
### 5. Recording results for an expectation
For a given method with non-void return type, a return value can be recorded through an assignment to the result field. When the method gets called in the replay phase, the specified return value will be returned to the caller. The assignment to result should appear right after the invocation that identifies the recorded expectation, inside an expectation block.

If the test instead needs an exception or error to be thrown when the method is invoked, then the result field can still be used: simply assign the desired throwable instance to it. Note that the recording of exceptions/errors to be thrown is applicable to mocked methods (of any return type) as well as to mocked constructors.

Multiple consecutive results (values to return and/or throwables to throw) can be recorded for the same expectation, by simply assigning the result field multiple times in a row. The recording of multiple return values and/or exceptions/errors to be thrown can be freely mixed for the same expectation. In the case of recording multiple consecutive return values for a given expectation, a single call to the returns(Object...) method can be made. Also, a single assignment to the result field will achieve the same effect, if the value assigned to it is a list or array containing the consecutive values.

The following example test records both types of results for the methods of a mocked DependencyAbc class, to be used when they are invoked from a UnitUnderTest class. Lets say the implementation of the class under test goes like this:
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
***
### 6. Matching invocations to specific instances
Previously, we explained that an expectation recorded on a mocked instance, such as "abc.someMethod();" would actually match invocations to DependencyAbc#someMethod() on any instance of the mocked DependencyAbc class. In most cases, tested code uses a single instance of a given dependency, so this won't really matter and can be safely ignored, whether the mocked instance is passed into the code under test or created inside it. But what if we need to verify that invocations occur on a specific instance, between several ones that happen to be used in the code under test? Also, what if only one or a few instances of the mocked class should actually be mocked, with other instances of the same class remaining unmocked? (This second case tends to occur more often when classes from the standard Java libraries, or from other third-party libraries, are mocked.) The API provides a mocking annotation, @Injectable, which will only mock one instance of the mocked type, leaving others unaffected. Additionally, we have a couple ways to constrain the matching of expectations to specific @Mocked instances, while still mocking all instances of the mocked class.

#### 6.1 Injectable mocked instances
Suppose we need to test code which works with multiple instances of a given class, some of which we want to mock. If an instance to be mocked can be passed or injected into the code under test, then we can declare an @Injectable mock field or mock parameter for it. This @Injectable instance will be an "exclusive" mocked instance; any other instance of the same mocked type, unless obtained from a separate mock field/parameter, will remain as a regular, non-mocked instance.

When using @Injectable, static methods and constructors are also excluded from being mocked. After all, a static method is not associated with any instance of the class, while a constructor is only associated with a newly created (and therefore different) instance.

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
