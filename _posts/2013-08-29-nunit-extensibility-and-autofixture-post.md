---
layout: post
title: NUnit Extensibility and AutoFixture
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2013-08-29
category: articles
tags: [AutoFixture, C#, NUnit, NUnitAddin, Unit Testing]
image:
  feature: so-simple-sample-image-1.jpg
comments: true
share: true
---

As a software consultant we can’t always choose the libraries we use on a client site, and so the story begins.

I have fallen in love with a library called AutoFixture it really simplifies the Arrange phase in writing unit tests. There are many patterns on how to use AutoFixture, but one pattern in particular that I like is where your unit test doesn’t take any dependency on AutoFixture.

Mark Seemann the author of AutoFixture also wrote an extension for xUnit.net  AutoDataAttribute which supplies auto generated arguments to a parameterised unit test.

{% highlight csharp %}
[Theory, AutoData]
public void IntroductoryTest(int expectedNumber, MyClass sut)
{
    int result = sut.Echo(expectedNumber);
    Assert.Equal(expectedNumber, result);
}
Extending AutoDataAttribute allows you to customise the internal Fixture that is used during auto generate process.
{% endhighlight %}

{% highlight csharp %}
public class TestConventionsAttribute
  : AutoDataAttribute
{
   public TestConventionsAttribute()
        : base(new Fixture().Customize(new AutoNSubstituteCustomization())
    {
    }
}
{% endhighlight %}

I wanted to be able to do the same using NUnit as said above, we can’t always choose what unit testing library we use. So how do you extend NUnit? After reading a few blog posts and NUnit Documentation on extensibility I was more confused than when I started.

Now where to start… NUnit contained a TheoryAttribute, but the implementation is really different from what I am used to in xUnit, and the NUnit DataAttribute is a property only attribute, so this wasn’t even an option to extend.

Some more investigation, I found the TestCaseAttribute and some examples of parameterised unit tests with NUnit.

```csharp
[TestCase(12,3,4)]
[TestCase(12,2,6)]
[TestCase(12,4,3)]
public void DivideTest(int n, int d, int q)
{
    Assert.AreEqual( q, n / d );
}
```

This looked much closer to what I was after, so I created an AutoTestCaseAttribute that inherited TestCaseAttribute, after many attempts to dynamically provide the arguments, I found that the property that I needed to set was an internal property, after some reading found out this was by design. #sadface

More reading, found TestCaseSourceAttribute which allowed you to define the parameters else where,

```csharp
static int[] EvenNumbers = new int[] { 2, 4, 6, 8 };
 
[Test, TestCaseSource("EvenNumbers")]
public void TestMethod(int num)
{
    Assert.IsTrue( num % 2 == 0 );
}
```

This really looked like the way to go, but wasn’t meant to be. The arguments that you passed had to be statically defined and again extending the attribute had the same issue as TestCaseAttribute.

The conclusion was that I needed to write an NUnitAddin with a implementation of ITestCaseProvider2

```csharp
public interface ITestCaseProvider
{
    bool HasTestCasesFor(MethodInfo method);
    IEnumerable GetTestCasesFor(MethodInfo method);
}
 
public interface ITestCaseProvider2 : ITestCaseProvider
{
    bool HasTestCasesFor( MethodInfo method, Test suite );
    IEnumerable GetTestCasesFor( MethodInfo method, Test suite );
}
```

Below is where the magic happens.

```csharp
public class AutoTestCaseProvider : ITestCaseProvider2
{
    ...
 
    public bool HasTestCasesFor(MethodInfo method)
    {
        return Reflect.HasAttribute(method, AutoFixtureNUnitFramework.AutoTestCaseAttribute, false);
    }
 
    ...
 
    public IEnumerable GetTestCasesFor(MethodInfo method, Test parentSuite)
    {
        Type[] parameterTypes = method.GetParameters().Select(o => o.ParameterType).ToArray();
 
        ArrayList parameterList = new ArrayList();
 
        var attributes = Reflect.GetAttributes(method, AutoFixtureNUnitFramework.AutoTestCaseAttribute, false);
 
        foreach (TestCaseDataAttribute attr in attributes)
        {
            foreach (var arguments in attr.GetArguments(method, parameterTypes))
            {
                ParameterSet parms = new ParameterSet();
                parms.Arguments = arguments;
 
                parameterList.Add(parms);
            }
        }
 
        return parameterList;
    }
 
    ...
}
```

Using reflection the NUnit.Framework identifies any unit tests with the AutoTestCaseAttribute applied, and then calls the GetArguments method which in turn uses the configured Fixture to generate any arguments and passes them to the parameterised unit test.

```csharp
public abstract class TestCaseDataAttribute : Attribute
{
    ...
  
    public abstract IEnumerable<object[]> GetArguments(MethodInfo method);
  
    ...
}
```
  
```csharp
public class AutoTestCaseAttribute : TestCaseDataAttribute
{
    ...
      
    public override IEnumerable<object[]> GetArguments(MethodInfo method)
    {
        if (method == null)
        {
            throw new ArgumentNullException("method");
        }
  
        var specimens = new List<object>();
        foreach (var p in method.GetParameters())
        {
            CustomizeFixture(p);
  
            var specimen = Resolve(p);
            specimens.Add(specimen);
        }
  
        return new[] { specimens.ToArray() };
    }    
  
    ...
}
```


An example of how to use.

```csharp
[TestFixture]
public class Tests
{
    [Test, AutoTestCase]
    public void IntroductoryTest(int expectedNumber, MyClass sut)
    {
        int result = sut.Echo(expectedNumber);
        Assert.Equal(expectedNumber, result);
    }
}
```

As I am writing this, I have submitted a pull request and awaiting a code review hopefully soon my code will be merged and set free for the world to use.