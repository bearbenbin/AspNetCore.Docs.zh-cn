---
title: 第 4 部分，将模型添加到 ASP.NET Core MVC 应用
author: rick-anderson
description: ASP.NET Core MVC 教程系列的第 4 部分。
ms.author: riande
ms.date: 11/16/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: tutorials/first-mvc-app/adding-model
ms.openlocfilehash: bfda45afeea67a11ad775996d94a06125df08bc6
ms.sourcegitcommit: 3593c4efa707edeaaceffbfa544f99f41fc62535
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/04/2021
ms.locfileid: "97854582"
---
# <a name="part-4-add-a-model-to-an-aspnet-core-mvc-app"></a>第 4 部分，将模型添加到 ASP.NET Core MVC 应用

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Tom Dykstra](https://github.com/tdykstra)

在本部分中将添加用于管理数据库中的电影的类。 这些类将是 MVC 应用的“Model”部分 。

可以结合 [Entity Framework Core](/ef/core) (EF Core) 使用这些类来处理数据库。 EF Core 是对象关系映射 (ORM) 框架，可以简化需要编写的数据访问代码。

要创建的模型类称为 POCO 类（源自“简单传统 CLR 对象”），因为它们与 EF Core 没有任何依赖关系   。 它们只定义将存储在数据库中的数据的属性。

在本教程中，首先要编写模型类，然后 EF Core 将创建数据库。

::: moniker range=">= aspnetcore-5.0"

## <a name="add-a-data-model-class"></a>添加数据模型类

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

右键单击 Models 文件夹，然后单击“添加” > “类” 。 将文件命名为 Movie.cs。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

将名为 Movie.cs 的文件添加到“Models”文件夹 。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

右键单击 Models 文件夹，然后单击“添加” > “新类” > “空类”。   将文件命名为 Movie.cs。

---

使用以下代码更新 Movie.cs 文件：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Models/Movie.cs)]

`Movie` 类包含一个 `Id` 字段，数据需要该字段作为主键。

`ReleaseDate` 上的 <xref:System.ComponentModel.DataAnnotations.DataType> 特性指定了数据的类型 (`Date`)。 通过此特性：

* 用户无需在数据字段中输入时间信息。
* 仅显示日期，而非时间信息。

[DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) 会在后续教程中介绍。

## <a name="add-nuget-packages"></a>添加 NuGet 包

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”(PMC)。 

![PMC 菜单](~/tutorials/first-mvc-app/adding-model/_static/pmc.png)

在 PMC 中运行以下命令：

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

前面的命令添加 EF Core SQL Server 提供程序。 提供程序包将 EF Core 包作为依赖项进行安装。 在本教程后面的基架步骤中会自动安装其他包。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI-5.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

从“项目”菜单中选择“管理 NuGet 程序包” 。

在右上方的“搜索”字段中，输入 `Microsoft.EntityFrameworkCore.SQLite` 并按回车键进行搜索。  搜索匹配的 NuGet 程序包，并按“添加”按钮。

![添加 Entity Framework Core NuGet 包](~/tutorials/first-mvc-app-mac/adding-model/_static/add-nuget-packages.png)

“选择项目”对话框将显示，并已选中 `MvcMovie` 项目。 按“确认”按钮。

“接受许可证”对话框将显示。 根据需要查看许可证，然后单击“接受”按钮。

重复上面的步骤，以安装以下 NuGet 程序包：

* `Microsoft.VisualStudio.Web.CodeGeneration.Design`
* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.EntityFrameworkCore.Design`

运行以下 .NET CLI 命令：

```dotnetcli
dotnet tool install --global dotnet-aspnet-codegenerator
```

上述命令会添加 [aspnet-codegenerator 基架工具](xref:fundamentals/tools/dotnet-aspnet-codegenerator)。

---

<a name="dc"></a>

## <a name="create-a-database-context-class"></a>创建数据库上下文类

需要一个数据库上下文类来协调 `Movie` 模型的 EF Core 功能（创建、读取、更新和删除）。 数据库上下文派生自 [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) 并指定要包含在数据模型中的实体。

创建一个“Data”文件夹。

使用以下代码添加 Data/MvcMovieContext.cs 文件： 

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/zDocOnly/MvcMovieContext.cs?name=snippet)]

前面的代码为实体集创建 [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) 属性。 在实体框架术语中，实体集通常与数据表相对应。 实体对应表中的行。

<a name="reg"></a>

## <a name="register-the-database-context"></a>注册数据库环境

ASP.NET Core 通过[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 生成。 在应用程序启动过程中，必须向 DI 注册服务（如 EF Core DB 上下文）。 需要这些服务（如 Razor 页面）的组件通过构造函数参数提供相应服务。 本教程的后续部分介绍了用于获取 DB 上下文实例的构造函数代码。 本部分会将数据库上下文注册到 DI 容器。

将以下 `using` 语句添加到 Startup.cs 顶部：

```csharp
using MvcMovie.Data;
using Microsoft.EntityFrameworkCore;
```

将以下突出显示的代码添加到 `Startup.ConfigureServices`：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_UseSqlite&highlight=5-6)]

---

通过调用 [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) 对象中的一个方法将连接字符串名称传递到上下文。 进行本地开发时，[ASP.NET Core 配置系统](xref:fundamentals/configuration/index)在 *appsettings.json* 文件中读取连接字符串。

<a name="cs"></a>

## <a name="add-a-database-connection-string"></a>添加数据库连接字符串

将连接字符串添加到 *appsettings.json* 文件中：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings.json?highlight=10-12)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings_SQLite.json?highlight=10-12)]

---

生成项目以检查编译器错误。

## <a name="scaffold-movie-pages"></a>基架电影页面

使用基架工具为电影模型生成“创建”、“读取”、“更新”和“删除”(CRUD) 页面。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在解决方案资源管理器中，右键单击“Controllers”文件夹 >“添加”>“新搭建基架的项目”。

![上述步骤的视图](adding-model/_static/add_controller21.png)

在“添加基架”对话框中，选择“包含视图的 MVC 控制器(使用 Entity Framework)”>“添加” 。

![“添加基架”对话框](adding-model/_static/add_scaffold21.png)

填写“添加控制器”对话框：

* 模型类：Movie (MvcMovie.Models)
* 数据上下文类：MvcMovieContext (MvcMovie.Data)

![“添加数据”上下文](adding-model/_static/dc5.png)

* 视图：将每个选项保持为默认选中状态
* 控制器名称：保留默认的 MoviesController
* 选择“添加”

Visual Studio 将创建：

* 电影控制器 (Controllers/MoviesController.cs)
* “创建”、“删除”、“详细信息”、“编辑”和“索引”页面的 Razor 视图文件 (Views/Movies/\*.cshtml)

自动创建这些文件称为“基架”。

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code) 

* 打开项目目录（包含 Program.cs、Startup.cs 和 .csproj 文件的目录）中的命令窗口。

* 在 Linux 上，导出基架工具路径：

  ```console
  export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 运行下面的命令：

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

### <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 打开项目目录（包含 Program.cs、Startup.cs 和 .csproj 文件的目录）中的命令窗口。

* 导出基架工具路径：

  ```console
  export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 运行下面的命令：

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

---

<!-- End of tabs                  -->

你还不能使用基架页面，因为该数据库不存在。 如果运行应用并单击“Movie App”链接，则会出现“无法打开数据库”或“无此类表：Movie”错误消息。

<a name="migration"></a>

## <a name="initial-migration"></a>初始迁移

使用 EF Core [迁移](xref:data/ef-mvc/migrations)功能来创建数据库。 迁移是可用于创建和更新数据库以匹配数据模型的一组工具。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”(PMC)。 

在 PMC 中，输入以下命令：

```powershell
Add-Migration InitialCreate
Update-Database
```

* `Add-Migration InitialCreate`：生成 Migrations/{timestamp}_InitialCreate.cs 迁移文件。 `InitialCreate` 参数是迁移名称。 可以使用任何名称，但是按照惯例，会选择可说明迁移的名称。 因为这是首次迁移，所以生成的类包含用于创建数据库架构的代码。 数据库架构基于在 `MvcMovieContext` 类中指定的模型。

* `Update-Database`：将数据库更新到上一个命令创建的最新迁移。 此命令在用于创建数据库的 Migrations/{time-stamp}_InitialCreate.cs 文件中运行 `Up` 方法。

  数据库更新命令生成以下警告： 

  > No type was specified for the decimal column 'Price' on entity type 'Movie'. This will cause values to be silently truncated if they do not fit in the default precision and scale. Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'.

  你可以忽略该警告，它将后面的教程中得到修复。

[!INCLUDE [more information on the PMC tools for EF Core](~/includes/ef-pmc.md)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

运行以下 .NET Core CLI 命令：

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

* `ef migrations add InitialCreate`：生成 Migrations/{timestamp}_InitialCreate.cs 迁移文件。 `InitialCreate` 参数是迁移名称。 可以使用任何名称，但是按照惯例，会选择可说明迁移的名称。 因为这是首次迁移，所以生成的类包含用于创建数据库架构的代码。 数据库架构基于在 `MvcMovieContext` 类中（位于 Data/MvcMovieContext.cs 文件中）中指定的模型。

* `ef database update`：将数据库更新到上一个命令创建的最新迁移。 此命令在用于创建数据库的 Migrations/{time-stamp}_InitialCreate.cs 文件中运行 `Up` 方法。

---

### <a name="the-initialcreate-class"></a>InitialCreate 类

检查 Migrations/{timestamp}_InitialCreate.cs 迁移文件：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Migrations/20190805165915_InitialCreate.cs?name=snippet)]

`Up` 方法创建 Movie 表，并将 `Id` 配置为主键。 `Down` 方法可还原 `Up` 迁移所做的架构更改。

<a name="test"></a>

## <a name="test-the-app"></a>测试应用

* 运行应用并单击“Movie App”链接。

  如果遇到类似于以下情况的异常：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

  ```console
  SqlException: Cannot open database "MvcMovieContext-1" requested by the login. The login failed.
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

  ```console
  SqliteException: SQLite Error 1: 'no such table: Movie'.
  ```

---
  可能缺失[迁移步骤](#migration)。

* 测试“创建”页。 输入并提交数据。

  > [!NOTE]
  > 可能无法在 `Price` 字段中输入十进制逗号。 若要使 [jQuery 验证](https://jqueryvalidation.org/)支持使用逗号（“,”）表示小数点的非英语区域设置，以及支持非美国英语日期格式，应用必须进行全球化。 有关全球化的说明，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)。

* 测试“编辑”、“详细信息”和“删除”页  。

## <a name="dependency-injection-in-the-controller"></a>控制器中的依赖项注入

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

打开 Controllers/MoviesController.cs 文件并检查构造函数：

<!-- l.. Make copy of Movies controller (or use the old one as I did in the 3.0 upgrade) because we comment out the initial index method and update it later  -->

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将数据库上下文 (`MvcMovieContext`) 注入到控制器中。 数据库上下文将在控制器中的每个 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) 方法中使用。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将数据库上下文 (`MvcMovieContext`) 注入到控制器中。 数据库上下文将在控制器中的每个 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) 方法中使用。

### <a name="use-sqlite-for-development-sql-server-for-production"></a>将 SQLite 用于开发，将 SQL Server 用于生产

选择 SQLite 后，模板生成的代码便可用于开发。 下面的代码演示如何将 <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> 注入到 Startup 中。 注入 `IWebHostEnvironment`，以便 `ConfigureServices` 可以在开发中使用 SQLite 并在生产中使用 SQL Server。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/StartupDevProd.cs?name=snippet_StartupClass&highlight=5,10,16-28)]

---
<!-- end of tabs --->

<a name="strongly-typed-models-keyword-label"></a>
<a name="strongly-typed-models-and-the--keyword"></a>

## <a name="strongly-typed-models-and-the-model-keyword"></a>强类型模型和 @model 关键词

在本教程之前的内容中，已经介绍了控制器如何使用 `ViewData` 字典将数据或对象传递给视图。 `ViewData` 字典是一个动态对象，提供了将信息传递给视图的方便的后期绑定方法。

MVC 还提供将强类型模型对象传递给视图的功能。 此强类型方法启用编译时代码检查。 基架机制通过 `MoviesController` 类和视图使用了此方法（即传递强类型模型）。

检查 Controllers/MoviesController.cs 文件中生成的 `Details` 方法：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_details)]

`id` 参数通常作为路由数据传递。 例如 `https://localhost:5001/movies/details/1` 的设置如下：

* 控制器被设置为 `movies` 控制器（第一个 URL 段）。
* 操作被设置为 `details`（第二个 URL 段）。
* ID 被设置为 1（最后一个 URL 段）。

还可以使用查询字符串传入 `id`，如下所示：

`https://localhost:5001/movies/details?id=1`

在未提供 ID 值的情况下，`id` 参数可定义为[可以为 null 的类型](/dotnet/csharp/programming-guide/nullable-types/index) (`int?`)。

[Lambda 表达式](/dotnet/articles/csharp/programming-guide/statements-expressions-operators/lambda-expressions)会被传入 `FirstOrDefaultAsync` 以选择与路由数据或查询字符串值相匹配的电影实体。

```csharp
var movie = await _context.Movie
    .FirstOrDefaultAsync(m => m.Id == id);
```

如果找到了电影，`Movie` 模型的实例则会被传递到 `Details` 视图：

```csharp
return View(movie);
```

检查 Views/Movies/Details.cshtml 文件的内容：

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/DetailsOriginal.cshtml)]

视图文件顶部的 `@model` 语句可指定视图期望的对象类型。 创建影片控制器时，将包含以下 `@model` 语句：

```cshtml
@model MvcMovie.Models.Movie
```

此 `@model` 指令允许访问控制器传递给视图的影片。 `Model` 对象为强类型对象。 例如，在 Details.cshtml 视图中，代码通过强类型的 `Model` 对象将每个电影字段传递给 `DisplayNameFor` 和 `DisplayFor`HTML 帮助程序。 `Create` 和 `Edit` 方法以及视图也传递一个 `Movie` 模型对象。

检查电影控制器中的 Index.cshtml 视图和 `Index` 方法。 请注意代码在调用 `View` 方法时是如何创建 `List` 对象的。 代码将此 `Movies` 列表从 `Index` 操作方法传递给视图：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_index)]

创建影片控制器后，基架将以下 `@model` 语句包含在 Index.cshtml 文件的顶部：

<!-- Copy Index.cshtml to IndexOriginal.cshtml -->

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?range=1)]

`@model` 指令使你能够使用强类型的 `Model` 对象访问控制器传递给视图的电影列表。 例如，在 Index.cshtml 视图中，代码使用 `foreach` 语句通过强类型 `Model` 对象对电影进行循环遍历：

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?highlight=1,31,34,37,40,43,46-48)]

因为 `Model` 对象为强类型（作为 `IEnumerable<Movie>` 对象），因此循环中的每个项都被类型化为 `Movie`。 除其他优点之外，这意味着可对代码进行编译时检查。

## <a name="additional-resources"></a>其他资源

* [标记帮助程序](xref:mvc/views/tag-helpers/intro)
* [全球化和本地化](xref:fundamentals/localization)

> [!div class="step-by-step"]
> [上一篇：添加视图](adding-view.md)
> [下一篇：使用 SQL](working-with-sql.md)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

## <a name="add-a-data-model-class"></a>添加数据模型类

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

右键单击 Models 文件夹，然后单击“添加” > “类” 。 将文件命名为 Movie.cs。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

将名为 Movie.cs 的文件添加到“Models”文件夹 。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

右键单击 Models 文件夹，然后单击“添加” > “新类” > “空类”。   将文件命名为 Movie.cs。

---

使用以下代码更新 Movie.cs 文件：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Models/Movie.cs)]

`Movie` 类包含一个 `Id` 字段，数据需要该字段作为主键。

`ReleaseDate` 上的 <xref:System.ComponentModel.DataAnnotations.DataType> 特性指定了数据的类型 (`Date`)。 通过此特性：

* 用户无需在数据字段中输入时间信息。
* 仅显示日期，而非时间信息。

[DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) 会在后续教程中介绍。

## <a name="add-nuget-packages"></a>添加 NuGet 包

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”(PMC)。 

![PMC 菜单](~/tutorials/first-mvc-app/adding-model/_static/pmc.png)

在 PMC 中运行以下命令：

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

前面的命令添加 EF Core SQL Server 提供程序。 提供程序包将 EF Core 包作为依赖项进行安装。 在本教程后面的基架步骤中会自动安装其他包。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/add-EF-NuGet-SQLite-CLI.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

从“项目”菜单中选择“管理 NuGet 程序包” 。

在右上方的“搜索”字段中，输入 `Microsoft.EntityFrameworkCore.SQLite` 并按回车键进行搜索。  搜索匹配的 NuGet 程序包，并按“添加”按钮。

![添加 Entity Framework Core NuGet 包](~/tutorials/first-mvc-app-mac/adding-model/_static/add-nuget-packages.png)

“选择项目”对话框将显示，并已选中 `MvcMovie` 项目。 按“确认”按钮。

“接受许可证”对话框将显示。 根据需要查看许可证，然后单击“接受”按钮。

重复上面的步骤，以安装以下 NuGet 程序包：

* `Microsoft.VisualStudio.Web.CodeGeneration.Design`
* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.EntityFrameworkCore.Design`

---

<a name="dc"></a>

## <a name="create-a-database-context-class"></a>创建数据库上下文类

需要一个数据库上下文类来协调 `Movie` 模型的 EF Core 功能（创建、读取、更新和删除）。 数据库上下文派生自 [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext) 并指定要包含在数据模型中的实体。

创建一个“Data”文件夹。

使用以下代码添加 Data/MvcMovieContext.cs 文件： 

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/zDocOnly/MvcMovieContext.cs?name=snippet)]

前面的代码为实体集创建 [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) 属性。 在实体框架术语中，实体集通常与数据表相对应。 实体对应表中的行。

<a name="reg"></a>

## <a name="register-the-database-context"></a>注册数据库上下文

ASP.NET Core 通过[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 生成。 在应用程序启动过程中，必须向 DI 注册服务（如 EF Core DB 上下文）。 需要这些服务（如 Razor 页面）的组件通过构造函数参数提供相应服务。 本教程的后续部分介绍了用于获取 DB 上下文实例的构造函数代码。 本部分会将数据库上下文注册到 DI 容器。

将以下 `using` 语句添加到 Startup.cs 顶部：

```csharp
using MvcMovie.Data;
using Microsoft.EntityFrameworkCore;
```

将以下突出显示的代码添加到 `Startup.ConfigureServices`：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_ConfigureServices&highlight=6-7)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Startup.cs?name=snippet_UseSqlite&highlight=6-7)]

---

通过调用 [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) 对象中的一个方法将连接字符串名称传递到上下文。 进行本地开发时，[ASP.NET Core 配置系统](xref:fundamentals/configuration/index)在 *appsettings.json* 文件中读取连接字符串。

<a name="cs"></a>

## <a name="add-a-database-connection-string"></a>添加数据库连接字符串

将连接字符串添加到 *appsettings.json* 文件中：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings.json?highlight=10-13)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-json[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/appsettings_SQLite.json?highlight=10-13)]

---

生成项目以检查编译器错误。

## <a name="scaffold-movie-pages"></a>基架电影页面

使用基架工具为电影模型生成“创建”、“读取”、“更新”和“删除”(CRUD) 页面。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在解决方案资源管理器中，右键单击“Controllers”文件夹 >“添加”>“新搭建基架的项目”。

![上述步骤的视图](adding-model/_static/add_controller21.png)

在“添加基架”对话框中，选择“包含视图的 MVC 控制器(使用 Entity Framework)”>“添加” 。

![“添加基架”对话框](adding-model/_static/add_scaffold21.png)

填写“添加控制器”对话框：

* 模型类：Movie (MvcMovie.Models)
* 数据上下文类：MvcMovieContext (MvcMovie.Data)

![“添加数据”上下文](adding-model/_static/dc3.png)

* 视图：将每个选项保持为默认选中状态
* 控制器名称：保留默认的 MoviesController
* 选择“添加”

Visual Studio 将创建：

* 电影控制器 (Controllers/MoviesController.cs)
* “创建”、“删除”、“详细信息”、“编辑”和“索引”页面的 Razor 视图文件 (Views/Movies/\*.cshtml)

自动创建这些文件称为“基架”。

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code) 

* 打开项目目录（包含 Program.cs、Startup.cs 和 .csproj 文件的目录）中的命令窗口。

* 在 Linux 上，导出基架工具路径：

  ```console
  export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 运行下面的命令：

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

### <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 打开项目目录（包含 Program.cs、Startup.cs 和 .csproj 文件的目录）中的命令窗口。

* 运行下面的命令：

  ```dotnetcli
  dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

  [!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

---

<!-- End of tabs                  -->

你还不能使用基架页面，因为该数据库不存在。 如果运行应用并单击“Movie App”链接，则会出现“无法打开数据库”或“无此类表：Movie”错误消息。

<a name="migration"></a>

## <a name="initial-migration"></a>初始迁移

使用 EF Core [迁移](xref:data/ef-mvc/migrations)功能来创建数据库。 迁移是可用于创建和更新数据库以匹配数据模型的一组工具。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”(PMC)。 

在 PMC 中，输入以下命令：

```powershell
Add-Migration InitialCreate
Update-Database
```

* `Add-Migration InitialCreate`：生成 Migrations/{timestamp}_InitialCreate.cs 迁移文件。 `InitialCreate` 参数是迁移名称。 可以使用任何名称，但是按照惯例，会选择可说明迁移的名称。 因为这是首次迁移，所以生成的类包含用于创建数据库架构的代码。 数据库架构基于在 `MvcMovieContext` 类中指定的模型。

* `Update-Database`：将数据库更新到上一个命令创建的最新迁移。 此命令在用于创建数据库的 Migrations/{time-stamp}_InitialCreate.cs 文件中运行 `Up` 方法。

  数据库更新命令生成以下警告： 

  > No type was specified for the decimal column 'Price' on entity type 'Movie'. This will cause values to be silently truncated if they do not fit in the default precision and scale. Explicitly specify the SQL server column type that can accommodate all the values using 'HasColumnType()'.

  你可以忽略该警告，它将后面的教程中得到修复。

[!INCLUDE [more information on the PMC tools for EF Core](~/includes/ef-pmc.md)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

运行以下 .NET Core CLI 命令：

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

* `ef migrations add InitialCreate`：生成 Migrations/{timestamp}_InitialCreate.cs 迁移文件。 `InitialCreate` 参数是迁移名称。 可以使用任何名称，但是按照惯例，会选择可说明迁移的名称。 因为这是首次迁移，所以生成的类包含用于创建数据库架构的代码。 数据库架构基于在 `MvcMovieContext` 类中（位于 Data/MvcMovieContext.cs 文件中）中指定的模型。

* `ef database update`：将数据库更新到上一个命令创建的最新迁移。 此命令在用于创建数据库的 Migrations/{time-stamp}_InitialCreate.cs 文件中运行 `Up` 方法。

---

### <a name="the-initialcreate-class"></a>InitialCreate 类

检查 Migrations/{timestamp}_InitialCreate.cs 迁移文件：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Migrations/20190805165915_InitialCreate.cs?name=snippet)]

`Up` 方法创建 Movie 表，并将 `Id` 配置为主键。 `Down` 方法可还原 `Up` 迁移所做的架构更改。

<a name="test"></a>

## <a name="test-the-app"></a>测试应用

* 运行应用并单击“Movie App”链接。

  如果遇到类似于以下情况的异常：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

  ```console
  SqlException: Cannot open database "MvcMovieContext-1" requested by the login. The login failed.
  ```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

  ```console
  SqliteException: SQLite Error 1: 'no such table: Movie'.
  ```

---
  可能缺失[迁移步骤](#migration)。

* 测试“创建”页。 输入并提交数据。

  > [!NOTE]
  > 可能无法在 `Price` 字段中输入十进制逗号。 若要使 [jQuery 验证](https://jqueryvalidation.org/)支持使用逗号（“,”）表示小数点的非英语区域设置，以及支持非美国英语日期格式，应用必须进行全球化。 有关全球化的说明，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)。

* 测试“编辑”、“详细信息”和“删除”页  。

## <a name="dependency-injection-in-the-controller"></a>控制器中的依赖项注入

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

打开 Controllers/MoviesController.cs 文件并检查构造函数：

<!-- l.. Make copy of Movies controller (or use the old one as I did in the 3.0 upgrade) because we comment out the initial index method and update it later  -->

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将数据库上下文 (`MvcMovieContext`) 注入到控制器中。 数据库上下文将在控制器中的每个 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) 方法中使用。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将数据库上下文 (`MvcMovieContext`) 注入到控制器中。 数据库上下文将在控制器中的每个 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) 方法中使用。

### <a name="use-sqlite-for-development-sql-server-for-production"></a>将 SQLite 用于开发，将 SQL Server 用于生产

选择 SQLite 后，模板生成的代码便可用于开发。 下面的代码演示如何将 <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> 注入到 Startup 中。 注入 `IWebHostEnvironment`，以便 `ConfigureServices` 可以在开发中使用 SQLite 并在生产中使用 SQL Server。

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/StartupDevProd.cs?name=snippet_StartupClass&highlight=5,10,16-28)]

---
<!-- end of tabs --->

<a name="strongly-typed-models-keyword-label"></a>
<a name="strongly-typed-models-and-the--keyword"></a>

## <a name="strongly-typed-models-and-the-model-keyword"></a>强类型模型和 @model 关键词

在本教程之前的内容中，已经介绍了控制器如何使用 `ViewData` 字典将数据或对象传递给视图。 `ViewData` 字典是一个动态对象，提供了将信息传递给视图的方便的后期绑定方法。

MVC 还提供将强类型模型对象传递给视图的功能。 此强类型方法启用编译时代码检查。 基架机制通过 `MoviesController` 类和视图使用了此方法（即传递强类型模型）。

检查 Controllers/MoviesController.cs 文件中生成的 `Details` 方法：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_details)]

`id` 参数通常作为路由数据传递。 例如 `https://localhost:5001/movies/details/1` 的设置如下：

* 控制器被设置为 `movies` 控制器（第一个 URL 段）。
* 操作被设置为 `details`（第二个 URL 段）。
* ID 被设置为 1（最后一个 URL 段）。

还可以使用查询字符串传入 `id`，如下所示：

`https://localhost:5001/movies/details?id=1`

在未提供 ID 值的情况下，`id` 参数可定义为[可以为 null 的类型](/dotnet/csharp/programming-guide/nullable-types/index) (`int?`)。

[Lambda 表达式](/dotnet/articles/csharp/programming-guide/statements-expressions-operators/lambda-expressions)会被传入 `FirstOrDefaultAsync` 以选择与路由数据或查询字符串值相匹配的电影实体。

```csharp
var movie = await _context.Movie
    .FirstOrDefaultAsync(m => m.Id == id);
```

如果找到了电影，`Movie` 模型的实例则会被传递到 `Details` 视图：

```csharp
return View(movie);
```

检查 Views/Movies/Details.cshtml 文件的内容：

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/DetailsOriginal.cshtml)]

视图文件顶部的 `@model` 语句可指定视图期望的对象类型。 创建影片控制器时，将包含以下 `@model` 语句：

```cshtml
@model MvcMovie.Models.Movie
```

此 `@model` 指令允许访问控制器传递给视图的影片。 `Model` 对象为强类型对象。 例如，在 Details.cshtml 视图中，代码通过强类型的 `Model` 对象将每个电影字段传递给 `DisplayNameFor` 和 `DisplayFor`HTML 帮助程序。 `Create` 和 `Edit` 方法以及视图也传递一个 `Movie` 模型对象。

检查电影控制器中的 Index.cshtml 视图和 `Index` 方法。 请注意代码在调用 `View` 方法时是如何创建 `List` 对象的。 代码将此 `Movies` 列表从 `Index` 操作方法传递给视图：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_index)]

创建影片控制器后，基架将以下 `@model` 语句包含在 Index.cshtml 文件的顶部：

<!-- Copy Index.cshtml to IndexOriginal.cshtml -->

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?range=1)]

`@model` 指令使你能够使用强类型的 `Model` 对象访问控制器传递给视图的电影列表。 例如，在 Index.cshtml 视图中，代码使用 `foreach` 语句通过强类型 `Model` 对象对电影进行循环遍历：

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?highlight=1,31,34,37,40,43,46-48)]

因为 `Model` 对象为强类型（作为 `IEnumerable<Movie>` 对象），因此循环中的每个项都被类型化为 `Movie`。 除其他优点之外，这意味着可对代码进行编译时检查。

## <a name="additional-resources"></a>其他资源

* [标记帮助程序](xref:mvc/views/tag-helpers/intro)
* [全球化和本地化](xref:fundamentals/localization)

> [!div class="step-by-step"]
> [上一篇：添加视图](adding-view.md)
> [下一篇：使用 SQL](working-with-sql.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="add-a-data-model-class"></a>添加数据模型类

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

右键单击 Models 文件夹，然后单击“添加” > “类” 。 将类命名“Movie”。

[!INCLUDE [model 1b](~/includes/mvc-intro/model1b.md)]

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

* 将类添加到名为“Movie.cs”的“Models”文件夹。

[!INCLUDE [model 1b](~/includes/mvc-intro/model1b.md)]
[!INCLUDE [model 2](~/includes/mvc-intro/model2.md)]

---

## <a name="scaffold-the-movie-model"></a>搭建“电影”模型的基架

在此部分，将搭建“电影”模型的基架。 确切地说，基架工具将生成页面，用于对“电影”模型执行创建、读取、更新和删除 (CRUD) 操作。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在解决方案资源管理器中，右键单击“Controllers”文件夹 >“添加”>“新搭建基架的项目”。

![上述步骤的视图](adding-model/_static/add_controller21.png)

在“添加基架”对话框中，选择“包含视图的 MVC 控制器(使用 Entity Framework)”>“添加” 。

![“添加基架”对话框](adding-model/_static/add_scaffold21.png)

填写“添加控制器”对话框：

* 模型类：Movie (MvcMovie.Models)
* 数据上下文类：选择 + 图标并添加默认的 MvcMovie.Models.MvcMovieContext 

![“添加数据”上下文](adding-model/_static/dc.png)

* 视图：将每个选项保持为默认选中状态
* 控制器名称：保留默认的 MoviesController
* 选择“添加”

![“添加控制器”对话框](adding-model/_static/add_controller2.png)

Visual Studio 将创建：

* Entity Framework Core [数据库上下文类](xref:data/ef-mvc/intro#create-the-database-context) (Data/MvcMovieContext.cs)
* 电影控制器 (Controllers/MoviesController.cs)
* “创建”、“删除”、“详细信息”、“编辑”和“索引”页面的 Razor 视图文件 (Views/Movies/\*.cshtml)

自动创建数据库上下文和 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete)（创建、读取、更新和删除）操作方法和视图的过程称为“搭建基架”。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!--  Until https://github.com/aspnet/Scaffolding/issues/582 is fixed windows needs backslash or the namespace is namespace RazorPagesMovie.Pages_Movies rather than namespace RazorPagesMovie.Pages.Movies
-->

* 打开项目目录（包含 Program.cs、Startup.cs 和 .csproj 文件的目录）中的命令窗口。
* 安装基架工具：

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* 在 Linux 上，导出基架工具路径：

  ```console
    export PATH=$HOME/.dotnet/tools:$PATH
  ```

* 运行下面的命令：

  ```dotnetcli
   dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

<!-- Mac -------------------------->

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 打开项目目录（包含 Program.cs、Startup.cs 和 .csproj 文件的目录）中的命令窗口。
* 安装基架工具：

  ```dotnetcli
   dotnet tool install --global dotnet-aspnet-codegenerator
   ```

* 运行下面的命令：

  ```dotnetcli
   dotnet aspnet-codegenerator controller -name MoviesController -m Movie -dc MvcMovieContext --relativeFolderPath Controllers --useDefaultLayout --referenceScriptLibraries
  ```

[!INCLUDE [explains scaffold generated params](~/includes/mvc-intro/model4.md)]

---

<!-- End of VS tabs                  -->

如果运行应用并单击“Mvc 电影”链接，则会出现以下类似的错误：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```
An unhandled exception occurred while processing the request.

SqlException: Cannot open database "MvcMovieContext-<GUID removed>" requested by the login. The login failed.
Login failed for user 'Rick'.

System.Data.SqlClient.SqlInternalConnectionTds..ctor(DbConnectionPoolIdentity identity, SqlConnectionString
```

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

```
An unhandled exception occurred while processing the request.
SqliteException: SQLite Error 1: 'no such table: Movie'.
Microsoft.Data.Sqlite.SqliteException.ThrowExceptionForRC(int rc, sqlite3 db)

```

---

你需要创建数据库，并且使用 EF Core [迁移](xref:data/ef-mvc/migrations)功能来执行此操作。 通过迁移可创建与数据模型匹配的数据库，并在数据模型更改时更新数据库架构。

<a name="pmc"></a>

## <a name="initial-migration"></a>初始迁移

在本节中，将完成以下任务：

* 添加初始迁移。
* 使用初始迁移来更新数据库。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. 从“工具”菜单中，选择“NuGet 包管理器”>“包管理器控制台”(PMC)。 

   ![PMC 菜单](~/tutorials/first-mvc-app/adding-model/_static/pmc.png)

1. 在 PMC 中，输入以下命令：

   ```powershell
   Add-Migration Initial
   Update-Database
   ```

   `Add-Migration` 命令生成用于创建初始数据库架构的代码。

   数据库架构基于在 `MvcMovieContext` 类中指定的模型。 `Initial` 参数是迁移名称。 可以使用任何名称，但是按照惯例，会使用可说明迁移的名称。 有关详细信息，请参阅 <xref:data/ef-mvc/migrations>。

   `Update-Database` 命令在用于创建数据库的 Migrations/{time-stamp}_InitialCreate.cs 文件中运行 `Up` 方法。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

[!INCLUDE [initial migration](~/includes/RP/model3.md)]

`ef migrations add InitialCreate` 命令生成用于创建初始数据库架构的代码。

数据库架构基于在 `MvcMovieContext` 类中（位于 Data/MvcMovieContext.cs 文件中）中指定的模型。 `InitialCreate` 参数是迁移名称。 可以使用任何名称，但是按照惯例，会选择可说明迁移的名称。

---

## <a name="examine-the-context-registered-with-dependency-injection"></a>检查通过依赖关系注入注册的上下文

ASP.NET Core 通过[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 生成。 服务（例如 EF Core 数据库上下文）在应用程序启动期间通过 DI 注册。 需要这些服务（如 Razor 页面）的组件通过构造函数参数提供相应服务。 本教程的后续部分介绍了用于获取 DB 上下文实例的构造函数代码。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

基架工具自动创建数据库上下文并将其注册到 DI 容器。

我们来研究以下 `Startup.ConfigureServices` 方法。 基架添加了突出显示的行：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=14-15)]

`MvcMovieContext` 为 `Movie` 模型协调 EF Core 功能（创建、读取、更新、删除等）。 数据上下文 (`MvcMovieContext`) 派生自 [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)。 数据上下文指定数据模型中包含哪些实体：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie3/Data/MvcMovieContext.cs)]

前面的代码为实体集创建 [DbSet\<Movie>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) 属性。 在实体框架术语中，实体集通常与数据表相对应。 实体对应表中的行。

通过调用 [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) 对象中的一个方法将连接字符串名称传递到上下文。 进行本地开发时，[ASP.NET Core 配置系统](xref:fundamentals/configuration/index)在 *appsettings.json* 文件中读取连接字符串。

# <a name="visual-studio-code--visual-studio-for-mac"></a>[Visual Studio Code / Visual Studio for Mac](#tab/visual-studio-code+visual-studio-mac)

创建了数据库上下文并将其注册到了 DI 容器。

---

<a name="test"></a>

### <a name="test-the-app"></a>测试应用

* 运行应用并将 `/Movies` 追加到浏览器中的 URL (`http://localhost:port/movies`)。

如果收到如下所示数据库异常：

```console
SqlException: Cannot open database "MvcMovieContext-GUID" requested by the login. The login failed.
Login failed for user 'User-name'.
```

缺少[迁移步骤](#pmc)。

* 测试“创建”链接。 输入并提交数据。

  > [!NOTE]
  > 可能无法在 `Price` 字段中输入十进制逗号。 若要使 [jQuery 验证](https://jqueryvalidation.org/)支持使用逗号（“,”）表示小数点的非英语区域设置，以及支持非美国英语日期格式，应用必须进行全球化。 有关全球化的说明，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/4076#issuecomment-326590420)。

* 测试“编辑”、“详细信息”和“删除”链接。

检查 `Startup` 类：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Startup.cs?name=snippet_ConfigureServices&highlight=13-99)]

上面突出显示的代码显示了要添加到[依赖关系注入](xref:fundamentals/dependency-injection)容器的电影数据库上下文：

* `services.AddDbContext<MvcMovieContext>(options =>` 指定要使用的数据库和连接字符串。
* `=>` 是 [lambda 运算符](/dotnet/articles/csharp/language-reference/operators/lambda-operator)

打开 Controllers/MoviesController.cs 文件并检查构造函数：

<!-- l.. Make copy of Movies controller because we comment out the initial index method and update it later  -->

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_1)]

构造函数使用[依赖关系注入](xref:fundamentals/dependency-injection)将数据库上下文 (`MvcMovieContext`) 注入到控制器中。 数据库上下文将在控制器中的每个 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) 方法中使用。

<a name="strongly-typed-models-keyword-label"></a>
<a name="strongly-typed-models-and-the--keyword"></a>

## <a name="strongly-typed-models-and-the-model-keyword"></a>强类型模型和 @model 关键词

在本教程之前的内容中，已经介绍了控制器如何使用 `ViewData` 字典将数据或对象传递给视图。 `ViewData` 字典是一个动态对象，提供了将信息传递给视图的方便的后期绑定方法。

MVC 还提供将强类型模型对象传递给视图的功能。 凭借此强类型方法可更好地对代码进行编译时检查。 基架机制在创建方法和视图时，通过 `MoviesController` 类和视图使用了此方法（即传递强类型模型）。

检查 Controllers/MoviesController.cs 文件中生成的 `Details` 方法：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_details)]

`id` 参数通常作为路由数据传递。 例如 `https://localhost:5001/movies/details/1` 的设置如下：

* 控制器被设置为 `movies` 控制器（第一个 URL 段）。
* 操作被设置为 `details`（第二个 URL 段）。
* ID 被设置为 1（最后一个 URL 段）。

还可以使用查询字符串传入 `id`，如下所示：

`https://localhost:5001/movies/details?id=1`

在未提供 ID 值的情况下，`id` 参数可定义为[可以为 null 的类型](/dotnet/csharp/programming-guide/nullable-types/index) (`int?`)。

[Lambda 表达式](/dotnet/articles/csharp/programming-guide/statements-expressions-operators/lambda-expressions)会被传入 `FirstOrDefaultAsync` 以选择与路由数据或查询字符串值相匹配的电影实体。

```csharp
var movie = await _context.Movie
    .FirstOrDefaultAsync(m => m.Id == id);
```

如果找到了电影，`Movie` 模型的实例则会被传递到 `Details` 视图：

```csharp
return View(movie);
   ```

检查 Views/Movies/Details.cshtml 文件的内容：

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/DetailsOriginal.cshtml)]

通过将 `@model` 语句包括在视图文件的顶端，可以指定视图期望的对象类型。 创建电影控制器时，会自动在 Details.cshtml 文件的顶端包括以下 `@model` 语句：

```cshtml
@model MvcMovie.Models.Movie
```

此 `@model` 指令使你能够使用强类型的 `Model` 对象访问控制器传递给视图的电影。 例如，在 Details.cshtml 视图中，代码通过强类型的 `Model` 对象将每个电影字段传递给 `DisplayNameFor` 和 `DisplayFor`HTML 帮助程序。 `Create` 和 `Edit` 方法以及视图也传递一个 `Movie` 模型对象。

检查电影控制器中的 Index.cshtml 视图和 `Index` 方法。 请注意代码在调用 `View` 方法时是如何创建 `List` 对象的。 代码将此 `Movies` 列表从 `Index` 操作方法传递给视图：

[!code-csharp[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Controllers/MC1.cs?name=snippet_index)]

创建电影控制器时，基架会自动在 Index.cshtml 文件的顶端包含以下 `@model` 语句：

<!-- Copy Index.cshtml to IndexOriginal.cshtml -->

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?range=1)]

`@model` 指令使你能够使用强类型的 `Model` 对象访问控制器传递给视图的电影列表。 例如，在 Index.cshtml 视图中，代码使用 `foreach` 语句通过强类型 `Model` 对象对电影进行循环遍历：

[!code-cshtml[](~/tutorials/first-mvc-app/start-mvc/sample/MvcMovie22/Views/Movies/IndexOriginal.cshtml?highlight=1,31,34,37,40,43,46-48)]

因为 `Model` 对象为强类型（作为 `IEnumerable<Movie>` 对象），因此循环中的每个项都被类型化为 `Movie`。 除其他优点之外，这意味着可对代码进行编译时检查：

## <a name="additional-resources"></a>其他资源

* [标记帮助程序](xref:mvc/views/tag-helpers/intro)
* [全球化和本地化](xref:fundamentals/localization)

> [!div class="step-by-step"]
> [上一篇：添加视图](adding-view.md)
> [下一篇：使用数据库](working-with-sql.md)

::: moniker-end
