# Use Dependency injection and Entity Framework Core in Azure Functions

## Setup

In this tutorial Visual Studio 2019 is used. However there should not be too much difference if you use different version of Visual Studio or Visual Studio code.

For entity framework core, code-first-approach is used in this tutorial.

***.Net Core 3.1 is used in this tutorial. Warning: DO NOT USE .Net 5 since you will encounter [this issue](https://github.com/Azure/azure-functions-core-tools/issues/2304).***

### Project Creation

Now let's setup the project.

As following GIF shows, in Visual Studio Create a new Azure Function Project. In this tutorial I name it AzFunc, you may name it any name you want.

![Project Creation](/images/AzFuncGif/CreateProject.gif)

### Install Nuget packages

There are few Nuget packages we need to install before we get to the code.

***DUE TO THE FACT WE USE .Net 3.1, WE NEED TO PAY A LITTLE BIT ATTENTION WE INSTALL NUGET PACKAGES.***


