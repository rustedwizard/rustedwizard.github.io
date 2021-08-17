# Use Dependency injection and Entity Framework Core (Code First Approach) in Azure Functions v3

## Setup

In this tutorial Visual Studio 2019 is used. However there should not be too much of difference if you use different version of Visual Studio.

For entity framework core, code-first-approach is used in this tutorial.

***.Net Core 3.1 is used in this tutorial. Warning: DO NOT USE .Net 5 since you will encounter [this issue](https://github.com/Azure/azure-functions-core-tools/issues/2304).***

### Project Creation

Now let's setup the project.

As following GIF shows, in Visual Studio Create a new Azure Function Project. In this tutorial I name it AzFunc, you may name it any name you want.

![Project Creation](/images/AzFuncGif/CreateProject.gif)

### Install Nuget packages

There are few Nuget packages we need to install before we get to the code.

***DUE TO THE FACT WE USE .Net 3.1, WE NEED TO PAY A LITTLE BIT ATTENTION WHEN WE INSTALL NUGET PACKAGES.***

Here is a list of Nuget packages we need to install:

* [Microsoft.Azure.Functions.Extensions v1.1.0](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/)

* [Microsoft.EntityFrameworkCore.SqlServer v3.1.18](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/3.1.18)

* [Microsoft.EntityFrameworkCore.Tools v3.1.18](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools/3.1.18)

* [Microsoft.Extension.DependencyInjection v3.1.18](https://www.nuget.org/packages/Microsoft.Extensions.DependencyInjection/3.1.18)

***You may notice that most of above metioned packages are not the latest. The reason is we are using .Net core 3.1, you have to keep the version match (basically 3.1.x matches to .Net core 3.1, 5.0.x mathces to .Net core 5)***

Now as following GIF shows, do following to install packages.

* right click Dependencies in Solution Explorer.

* Click Manage Nuget Packages.

* Search above mentioned packages in Browse tab.

* Select the package and select the version on right hand side. 

* Click install to install package (accept all license agreement during the process).

![Install Nuget Packages](/images/AzFuncGif/InstallPackages.gif)

## Coding

At this point all setup work is complete. We can now get to do some coding.

### Enable Dependency injection

The very first task would be enable Dependency injection in this project. This job relies on the [Microsoft.Azure.Functions.Extensions v1.1.0](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/) package we installed earlier. It provides a very simple way to do so.

Here is what you need to do:

* Create a new C# file name it Startup.cs.

* Put following code in the file.

```csharp
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using System;

[assembly: FunctionsStartup(typeof(AzFunc.StartUp))]

namespace AzFunc
{
    public class StartUp : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {

        }
    }
}
```

What happened in above code is that the attribute ``` [assembly: FunctionsStartup(typeof(AzFunc.StartUp))] ``` tells Azure function when starting up the ```StartUp``` class defined in namespace ```AzFunc``` is where it needs to start. And as you may notice the ``` StartUp ``` class inherited from abstract class ``` FunctionsStartup ``` and override the method ``` void Configure(IFunctionsHostBuilder builder) ```. As a result, at startup this ``` Configure``` method gets called, and this is where give us chance to configure dependency injection as we need.
