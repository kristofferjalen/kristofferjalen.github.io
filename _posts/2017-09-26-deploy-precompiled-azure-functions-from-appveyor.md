---
layout: post
title:  "Deploy precompiled Azure Functions from AppVeyor"
published: true
tags: Azure, .NET
---
With [Azure Functions Tools for Visual Studio 2017](https://docs.microsoft.com/en-us/azure/azure-functions/functions-develop-vs) you can easily publish your Azure Functions project to Azure directly from Visual Studio. That is very simple. 

However, if you want to deploy with a Continuous Integration tool, e.g. AppVeyor, you can do that as well. [This article](https://alastairchristian.com/deploying-azure-functions-from-appveyor-75fe03771d0c) took me almost all the way, but that article is aimed at [C# script files](https://docs.microsoft.com/en-us/azure/azure-functions/functions-reference-csharp) (`.csx`). To get it work with [precompiled functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-dotnet-class-library) (`.cs`), there are some minor differences.

For this example, assume there is a folder `src` with an Azure Function project `FunctionApp` that has an Azure Function `MyFunction`. Everything is hosted by an App Service `websitename`.

Step 1
======
In the **Artifacts** section in AppVeyor, add an artifact:
* Path to artifact: `src\FunctionApp\bin\Release\net461`
* Deployment name: `FunctionApp`
* Type: `Auto`

Step 2
======
Login to your Azure Portal, open your Functions App, and click the **Download publish profile** button. 

In the **Deployments** section in AppVeyor, add a deployment: 
* Use Deployment Provider **Web Deploy** and copy and paste values from the **MSDeploy** section in the downloaded publish profile file:
  - Server: https://*{publishUrl}*/msdeploy.axd?site=*{msdeploySite}*
  - Website name: *{msdeploySite}*
  - Username: *{userName}*
  - Password: *{userPWD}*
* In the **Artifact to deploy** textbox, enter the deployment name of the artifact created in step 1.

Step 3
======
In the **Build** section in AppVeyor, add a **Release** configuration to the build matrix.

Start a new build of the latest commit. That resulted in `MSBuild` giving me the following error message:

```C:\path-to-repo\src\FunctionApp\FunctionApp.csproj.metaproj : warning MSB4078: The project file "src\FunctionApp\FunctionApp.csproj" is not supported by MSBuild and cannot be built.```

In the **Environment** section in AppVeyor, change **Build worker image** to Visual Studio 2017 and start a new build again. That gave me this error message:

```C:\Program Files\dotnet\sdk\2.0.0\Sdks\Microsoft.NET.Sdk\build\Microsoft.PackageDependencyResolution.targets(323,5): error : Assets file 'C:\path-to-repo\src\FunctionApp\obj\project.assets.json' not found. Run a NuGet package restore to generate this file. [C:\path-to-repo\src\FunctionApp\FunctionApp.csproj]```

In the **Build** section in AppVeyor, add a **Before build** script of type `CMD`:
```
dotnet restore
```
Start a new build again.

Now, the build was successful and the deployment is completed.


Exported project configuration
==============================
The `appveyor.yml` file looks like this:
```
 version: 1.0.{build}
 image: Visual Studio 2017
 configuration: Release
 before_build:
 - cmd: dotnet restore
 build:
   verbosity: minimal
 artifacts:
 - path: src\FunctionApp\bin\Release\net461
   name: FunctionApp
 deploy:
 - provider: WebDeploy
   server: https://websitename.scm.azurewebsites.net:443/msdeploy.axd?site=websitename
   website: websitename
   username: $websitename
   password:
     secure: [password]
   artifact: FunctionApp
```