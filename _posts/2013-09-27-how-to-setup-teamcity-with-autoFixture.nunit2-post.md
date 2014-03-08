---
layout: post
title: How to setup TeamCity with AutoFixture.NUnit2
description: 
modified: 2013-09-27
category: articles
tags: [AutoFixture, C#, NUnit, NUnitAddin, Unit Testing]
image:
  feature: so-simple-sample-image-1.jpg
comments: true
share: true
---

In a previous post I explained how setup AutoFixture.NUnit2 to work with a few 3rd party applications, to complete the AutoFixture.NUnit2 story we need to be able to configure [TeamCity](http://www.jetbrains.com/teamcity/) for [Continuous integration](http://en.wikipedia.org/wiki/Continuous_integration) according to Teamcity documentations there are a few ways to do this, but only way I have been successful so far was to configure msbuild script to run [nunit-console.exe](http://confluence.jetbrains.com/display/TCD8/TeamCity+Addin+for+NUnit)

Example of msbuild script to copy Teamcity NUnit Addin and `AutoFixture.NUnit2.Addins.dll`

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003" DefaultTargets="CompleteBuild" ToolsVersion="4.0">
    <PropertyGroup>
    	<NUnitHome>packages\nunit.runners.2.6.2\tools</NUnitHome>
    </PropertyGroup>

    <ItemGroup>
    	<NUnitAddinFiles Include="$(teamcity_dotnet_nunitaddin)-2.6.2.*" />
    	<NUnitAddinFiles Include="packages\AutoFixture.NUnit2.3.9.0\lib\net40\Ploeh.AutoFixture.NUnit2.Addins.dll" />
    </ItemGroup>

    <Target Name="RunTests">
    	<MakeDir Directories="$(NUnitHome)\addins" />
	<Copy SourceFiles="@(NUnitAddinFiles)" DestinationFolder="$(NUnitHome)\addins" />
	<Exec Command="$(NUnitHome)\nunit-console.exe $(TestAssembly)" />
    </Target>

    <Target Name="CompleteBuild" DependsOnTargets="RunTests" />
</Project>
```