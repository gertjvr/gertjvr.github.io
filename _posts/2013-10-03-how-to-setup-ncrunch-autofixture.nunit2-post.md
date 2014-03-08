---
layout: post
title: How to setup NCrunch with AutoFixture.NUnit2
description: 
modified: 2013-10-03
category: articles
tags: [AutoFixture, C#, NUnit, NUnitAddin, Unit Testing]
image:
  feature: so-simple-sample-image-1.jpg
comments: true
share: true
---

Now that the AutoFixture.NUnit2 is out there in the wild, a few people have commented that the deployment story has a bit of friction, and I agree with that statement, but that is how the NUnit Extensibility model works.

After a few attempts to make NCrunch work with AutoFixture.NUnit2 resorted to looking on NCrunch [support forum](http://forum.ncrunch.net/yaf_postsm4652_Getting-NCrunch-to-load-NUnit-Addins.aspx#post4652) it mentioned to include the Addin in the test assembly. This got me thinking... what if we created a class inherited from AutoFixture.NUnit2 Addin. To my surprise this worked.

**Here is how:**

1. Install-Package AutoFixture.NUnit2
2. Install-Package NUnit.Runners (will have to manually reference `tools\nunit.core.interfaces`)
3. Remove this line `[assembly: NUnit.Framework.RequiredAddin(Ploeh.AutoFixture.NUnit2.Addins.Constants.AutoDataExtension)]` from AssemblyInfo.cs
4. Create the below class in each test assembly

```csharp
using NUnit.Core.Extensibility;

namespace ...
{
    [NUnitAddin]
    public class LocalAddin : Ploeh.AutoFixture.NUnit2.Addins.Addin
    {   
    }
}
```

To make NCrunch work with AutoFixture.NUnit2 change the "solution configuration" - "framework utilisation type for NUnit" option from "Dynamic Analysis" to "Static Analysis" this is explained in the [support documentation](http://www.ncrunch.net/documentation/reference_solution-configuration_framework-utilisation-types)

**Why does this work?**
When you are developing NUnit Addins, NUnit will load any classes that implements `IAddin` into the test runner's `AddinRegistry` this is intended for **development only**.

**Interesting side effects**
You can use the buildin TeamCity NUnit Runner with AutoFixture.NUnit2 no need to use [msbuild script](http://gertjvr.wordpress.com/2013/09/27/howto-autofixture-nunit2-with-teamcity/) as previously explained.
