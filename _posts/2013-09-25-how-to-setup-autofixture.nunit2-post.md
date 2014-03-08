---
layout: post
title: How to setup AutoFixture.NUnit2
description: 
modified: 2013-09-25
category: articles
tags: [AutoFixture, C#, NUnit, NUnitAddin, Unit Testing]
image:
  feature: so-simple-sample-image-1.jpg
comments: true
share: true
---

With the release of AutoFixture.NUnit2 earlier today, here is some guidance how to get it working

> NUnit Documentation: Addin Identification, Loading and Installation
>
> NUnit examines all assemblies in the bin/addins directory, looking for public classes with the `NUnitAddinAttribute` and implementing the `IAddin` interface. It loads all those it finds as Addins.

After package install you need to copy `packages\AutoFixture.NUnit2.3.9.0\lib\net40\Ploeh.AutoFixture.NUnit2.Addins.dll` to `bin\addins` folder

But where exactly is `bin/addins` located. The short answer is where is the nunit.exe or nunit-console.exe located

Here is some default locations

- NUnit2.6.2 -> C:\Program Files (x86)\NUnit 2.6.2\bin\addins
- R# 8 -> C:\Program Files (x86)\JetBrains\ReSharper\v8.0\Bin\addins
- TestDriven.Net -> C:\Program Files (x86)\TestDriven.NET 3\NUnit\2.6\addins

**Mark Seemann** better explains how to use AutoDataAttribute [here](http://blog.ploeh.dk/2010/10/08/AutoDataTheorieswithAutoFixture/):

NUnit example

```csharp
[Test, AutoData]
public void IntroductoryTest(
    int expectedNumber, MyClass sut)
{
    int result = sut.Echo(expectedNumber);
    Assert.Equal(expectedNumber, result);
}
```