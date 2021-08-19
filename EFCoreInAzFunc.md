# Use Dependency injection and Entity Framework Core (Code First Approach) in Azure Functions v3

## Setup

In this tutorial Visual Studio 2019 is used. However there should not be too much of difference if you use different version of Visual Studio.

For entity framework core, code-first-approach is used in this tutorial.

***.Net Core 3.1 LTS is used in this tutorial. Warning: DO NOT USE .Net 5 since you will encounter [this issue](https://github.com/Azure/azure-functions-core-tools/issues/2304).***

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

### Entity Framework

With Dependency Injection enabled, we can start coding the Entity Framework part. In this tutorial, I will stay simple, a simple Todo program is what I am going to create; which contains only one Model called Todo.

#### Create Todo Model

Before we work with Entity Framework, let's create model first. To do it, As following GIF shows, simply:

* Create a new folder named Model (this is only for purpose of keep code organized).
* Create new Todo.cs file

![Create Todo Model](/images/AzFuncGif/CreateTodo.gif)

Modify Todo.cs file, make sure it contains following code:

```csharp
using System;

namespace AzFunc.Models
{
    //Use Fluent API in DbContext class. So this class can be kept clean
    public class Todos
    {
        public Guid Id { get; set; }

        public string Name { get; set; }

        public string Description { get; set; }

        public string Comment { get; set; }

        public DateTime StartTime { get; set; }

        public DateTime EndTime { get; set; }
    }
}
```

As you can see, it's a very simple class, but it is enough for demonstration purpose. Now, let's get to the DbContext part

#### Create TodoDbContext

To create a TodoDbContext, simple do following:

* Create new folder called DbContext (To keep code organized).
* Create new file called TodoDbContext.cs.

As following GIF shows:

![Create TodoDbContext](/images/AzFuncGif/CreateTodoDbContext.gif)

Modify TodoDbContext, put following code in this file:

```csharp
using AzFunc.Models;
using Microsoft.EntityFrameworkCore;

namespace AzFunc.DbContext
{
    public class TodoDbContext : DbContext
    {
        public DbSet<Todos> todos { get; set; }

        public TodoDbContext(DbContextOptions op) : base(op)
        { }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            builder.Entity<Todos>().HasKey(x => x.Id);
            builder.Entity<Todos>().HasIndex(x => x.Id);
            builder.Entity<Todos>().Property(x => x.Name).HasMaxLength(60).IsRequired();
            builder.Entity<Todos>().Property(x => x.Description).HasMaxLength(500).IsRequired();
            builder.Entity<Todos>().Property(x => x.Comment).HasMaxLength(300);
        }
    }
}
```

In this tutorial, I am using [Fluent API](https://docs.microsoft.com/en-us/ef/ef6/modeling/code-first/fluent/types-and-properties) instead of [Data Annotation](https://docs.microsoft.com/en-us/ef/ef6/modeling/code-first/data-annotations), so I can keep the Todo class clean. As you can see the ```GUID``` type ```ID``` of Todo class is the primary key and index. Property Name and Description can not be null and has maximum length of 60 and 500 characters. As for comment there's a maximum length of 300 characters limit. For StartTime and EndTime, since [DateTime type](https://docs.microsoft.com/en-us/dotnet/api/system.datetime?view=netcore-3.1) in C# is a value type, and it's not nullable (unless you make it nullable by using ```DateTime?```), so by default, it will be required as database column.

#### Enable Code First in project.

If you have any experience in ASP.Net and Entity Framework, at this point you would start using ```Add-Migration``` and ```Update-Database```. ***However, if you try this now in Azure Functions Project, you would get error.*** Because, EntityFramework does not know how to create TodoDbContext when we invoke Add-Migration tool. Luckily, it is very easy and only involve a little bit more coding to tell it how to create one and we are ready to go.

To do so we need to implement interface ```IDesignTimeDbContextFactory```, so let's do following:

* Create new folder called DbContextFactory
* Create new file called TodoDbContextFactory.cs

As follwoing GIF shows:

![Create TodoDbContextFactory](/images/AzFuncGif/CreateTodoDbContextFactory.gif)

Now, modify the file, make sure following code is in the file:

```csharp
using AzFunc.DbContexts;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Design;
using System;

namespace AzFunc.DbContextFactory
{
    public class TodoDbContextFactory : IDesignTimeDbContextFactory<TodoDbContext>
    {
        public TodoDbContext CreateDbContext(string[] args)
        {
            var opBuilder = new DbContextOptionsBuilder<TodoDbContext>();
            opBuilder.UseSqlServer(Environment.GetEnvironmentVariable("SqlConnection"));
            return new TodoDbContext(opBuilder.Options);
        }
    }
}
```

#### Create Migration and Update Database

Above code will enable you to use Code-First-Approach in Azure Functions Project. Now, you may go ahead use ```Add-Migration``` and ```Update-Database```. ***Notice that in above code, we get connection string from environment variable, and since we are use add-migration tool, you need to make sure the environment variable is properly set.***
You can easily do that in ```Nuget Package Manager -> Package Manager Console```, since it's powershell windows you can set environment variable for current session with command:

```powershell
$env:SqlConnection="<your connection string here>"
``` 

Assuming you have Sql Server installed on you local machine, the specific command would look like fowlloing:

```powershell
$env:SqlConnection="Server=localhost; Database=Todo; Integrated Security=true"
Add-Migration init
Update-Database
```

Following GIF shows the entire process:

![Create Migration and Update Database](/images/AzFuncGif/CreateMigration.gif)

You can verify the process is successful by checking following places:

*First verify the migration has been created. At ```Solution Explorer``` you should see a folder named Migrations and have some files like following images.

![Solution Explorer after create migration](/images/AzFuncGif/SolutionExplorer.JPG)

*Check for database, you may check if database has been created by using some tools like Azure Data Studio or Database Explorer in Visual Studio. You should see a new Database name Todo got created and looks like following:

![Database](/images/AzFuncGif/Database.JPG)

At this point, you have Entity Framework all setup and you database and Models are ready to go.

### Hook up DbContext to Dependency Injection

Now, we have our database, model and DbContext ready, we now need to get it work with Dependency Injection we previously setup. We can easily get it done by do following:

#### Setup connection string in local.settings.json

First we need get connection string setup for local machine. Azure Functions Project provides us a place to do so in local.settings.json. You can do so in following steps:

* Open local.settings.json
* Within the scope of ```"Values"``` put a new line like following:

```JSON
"SqlConnection":"Server=localhost; Database=Todo; Integrated Security=true"
```

Remember to replace sql connection string with you own!

***Warning: Please remember local.settings.json is for local development purpose only, so you should NEVER check this file into you repository. Also you should NEVER hard code database connection string anywhere in the project. Luckily, by default, Visual Studio created a .gitignore file which include entry to ignore local.settings.json file. When this function works on Azure cloud, you will need to configure database connection string in Application Settings which covered in this tutorial in later section***


