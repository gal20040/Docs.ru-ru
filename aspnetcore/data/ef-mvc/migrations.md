---
title: "Основные ASP.NET MVC с основными EF - Migrations - 4 из 10"
author: tdykstra
description: "В этом учебнике сначала с помощью EF Core миграции для управления изменений модели данных в приложении ASP.NET Core MVC."
keywords: "ASP.NET Core, Entity Framework Core, миграция"
ms.author: tdykstra
manager: wpickett
ms.date: 03/15/2017
ms.topic: get-started-article
ms.assetid: 81f6c9c2-a819-4f3a-97a4-4b0503b56c26
ms.technology: aspnet
ms.prod: asp.net-core
uid: data/ef-mvc/migrations
ms.openlocfilehash: 4d81099d1ab97a8a49d96657153a54aa96dd6bf8
ms.sourcegitcommit: 74e22e08e3b08cb576e5184d16f4af5656c13c0c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/25/2017
---
# <a name="migrations---ef-core-with-aspnet-core-mvc-tutorial-4-of-10"></a><span data-ttu-id="f2211-104">Миграция - Core EF учебнику ASP.NET Core MVC (4 из 10)</span><span class="sxs-lookup"><span data-stu-id="f2211-104">Migrations - EF Core with ASP.NET Core MVC tutorial (4 of 10)</span></span>

<span data-ttu-id="f2211-105">По [Tom Dykstra](https://github.com/tdykstra) и [Рик Андерсон](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="f2211-105">By [Tom Dykstra](https://github.com/tdykstra) and [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="f2211-106">Contoso университета примера веб-приложения показано, как создавать веб-приложения ASP.NET MVC ядра с помощью основного Entity Framework и Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="f2211-106">The Contoso University sample web application demonstrates how to create ASP.NET Core MVC web applications using Entity Framework Core and Visual Studio.</span></span> <span data-ttu-id="f2211-107">Сведения о учебника серии см [в первом учебнике ряда](intro.md).</span><span class="sxs-lookup"><span data-stu-id="f2211-107">For information about the tutorial series, see [the first tutorial in the series](intro.md).</span></span>

<span data-ttu-id="f2211-108">В этом учебнике сначала с помощью EF Core миграции для управления изменения в модели данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-108">In this tutorial, you start using the EF Core migrations feature for managing data model changes.</span></span> <span data-ttu-id="f2211-109">На следующих занятиях рассматривается вы добавите дополнительные миграции при изменении модели данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-109">In later tutorials, you'll add more migrations as you change the data model.</span></span>

## <a name="introduction-to-migrations"></a><span data-ttu-id="f2211-110">Общие сведения о миграции</span><span class="sxs-lookup"><span data-stu-id="f2211-110">Introduction to migrations</span></span>

<span data-ttu-id="f2211-111">При разработке нового приложения модель данных меняется часто и каждый раз изменения модели, она возвращает синхронизации с базой данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-111">When you develop a new application, your data model changes frequently, and each time the model changes, it gets out of sync with the database.</span></span> <span data-ttu-id="f2211-112">Эти учебники запущенную настройка платформы Entity Framework для создания базы данных, если он не существует.</span><span class="sxs-lookup"><span data-stu-id="f2211-112">You started these tutorials by configuring the Entity Framework to create the database if it doesn't exist.</span></span> <span data-ttu-id="f2211-113">Затем при каждом изменении модели данных — добавить, удалить, или классы сущностей или изменить класс DbContext — можно удалить базу данных и EF создает новый, который соответствует модели и заполняет его с тестовыми данными.</span><span class="sxs-lookup"><span data-stu-id="f2211-113">Then each time you change the data model -- add, remove, or change entity classes or change your DbContext class -- you can delete the database and EF creates a new one that matches the model, and seeds it with test data.</span></span>

<span data-ttu-id="f2211-114">Синхронизация базы данных с моделью данных, этот метод работает хорошо, пока развертывание приложения в рабочей среде.</span><span class="sxs-lookup"><span data-stu-id="f2211-114">This method of keeping the database in sync with the data model works well until you deploy the application to production.</span></span> <span data-ttu-id="f2211-115">Когда приложение выполняется в производственной среде, он обычно хранит данные, которые вы хотите сохранить, и вы не хотите потерять все, что каждый раз вносятся изменения, такие как добавление нового столбца.</span><span class="sxs-lookup"><span data-stu-id="f2211-115">When the application is running in production it is usually storing data that you want to keep, and you don't want to lose everything each time you make a change such as adding a new column.</span></span> <span data-ttu-id="f2211-116">Средство миграции Core EF решает эту проблему, позволяя EF обновить схему базы данных вместо создания новой базы данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-116">The EF Core Migrations feature solves this problem by enabling EF to update the database schema instead of creating  a new database.</span></span>

## <a name="entity-framework-core-nuget-packages-for-migrations"></a><span data-ttu-id="f2211-117">Entity Framework Core NuGet пакетов для миграции</span><span class="sxs-lookup"><span data-stu-id="f2211-117">Entity Framework Core NuGet packages for migrations</span></span>

<span data-ttu-id="f2211-118">Для работы с миграции, можно использовать **консоль диспетчера пакетов** (PMC) или интерфейса командной строки (CLI).</span><span class="sxs-lookup"><span data-stu-id="f2211-118">To work with migrations, you can use the **Package Manager Console** (PMC) or the command-line interface (CLI).</span></span>  <span data-ttu-id="f2211-119">Эти учебники демонстрируют использование команды CLI.</span><span class="sxs-lookup"><span data-stu-id="f2211-119">These tutorials show how to use CLI commands.</span></span> <span data-ttu-id="f2211-120">Сведения о PMC находится в [конце этого учебника](#pmc).</span><span class="sxs-lookup"><span data-stu-id="f2211-120">Information about the PMC is at [the end of this tutorial](#pmc).</span></span>

<span data-ttu-id="f2211-121">Средства EF для интерфейса командной строки (CLI) доступны на ресурсе [Microsoft.EntityFrameworkCore.Tools.DotNet](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools.DotNet).</span><span class="sxs-lookup"><span data-stu-id="f2211-121">The EF tools for the command-line interface (CLI) are provided in [Microsoft.EntityFrameworkCore.Tools.DotNet](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools.DotNet).</span></span> <span data-ttu-id="f2211-122">Чтобы установить этот пакет, добавьте его в `DotNetCliToolReference` коллекции в *.csproj* файла, как показано.</span><span class="sxs-lookup"><span data-stu-id="f2211-122">To install this package, add it to the `DotNetCliToolReference` collection in the *.csproj* file, as shown.</span></span> <span data-ttu-id="f2211-123">**Примечание:** необходимо установить этот пакет путем редактирования *.csproj* файла; нельзя использовать `install-package` команды или графического пользовательского интерфейса диспетчера пакетов.</span><span class="sxs-lookup"><span data-stu-id="f2211-123">**Note:** You have to install this package by editing the *.csproj* file; you can't use the `install-package` command or the package manager GUI.</span></span> <span data-ttu-id="f2211-124">Можно изменить *.csproj* файла, щелкнув правой кнопкой мыши имя проекта в **обозревателе решений** и выбрав **изменить ContosoUniversity.csproj**.</span><span class="sxs-lookup"><span data-stu-id="f2211-124">You can edit the *.csproj* file by right-clicking the project name in **Solution Explorer** and selecting **Edit ContosoUniversity.csproj**.</span></span>

[!code-xml[](intro/samples/cu/ContosoUniversity.csproj?range=12-15&highlight=2)]
  
<span data-ttu-id="f2211-125">(Номера версий, в этом примере были действительны, во время написания учебника.)</span><span class="sxs-lookup"><span data-stu-id="f2211-125">(The version numbers in this example were current when the tutorial was written.)</span></span> 

## <a name="change-the-connection-string"></a><span data-ttu-id="f2211-126">Измените строку подключения</span><span class="sxs-lookup"><span data-stu-id="f2211-126">Change the connection string</span></span>

<span data-ttu-id="f2211-127">В *appsettings.json* файл, измените имя базы данных в строке соединения на ContosoUniversity2 или другое имя, вы не использовали на компьютере, который вы используете.</span><span class="sxs-lookup"><span data-stu-id="f2211-127">In the *appsettings.json* file, change the name of the database in the connection string to ContosoUniversity2 or some other name that you haven't used on the computer you're using.</span></span>

<span data-ttu-id="f2211-128">[!code-json[Main](intro/samples/cu/appsettings2.json?range=1-4)]</span><span class="sxs-lookup"><span data-stu-id="f2211-128">[!code-json[Main](intro/samples/cu/appsettings2.json?range=1-4)]</span></span>

<span data-ttu-id="f2211-129">Это изменение настраивает проект, чтобы первый миграции создает новую базу данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-129">This change sets up the project so that the first migration will create a new database.</span></span> <span data-ttu-id="f2211-130">Это не является необходимым для начала работы с миграции, но вы увидите Далее почему он имеет смысл.</span><span class="sxs-lookup"><span data-stu-id="f2211-130">This isn't required for getting started with migrations, but you'll see later why it's a good idea.</span></span>

> [!NOTE]
> <span data-ttu-id="f2211-131">В качестве альтернативы для изменения имени базы данных можно удалить базы данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-131">As an alternative to changing the database name, you can delete the database.</span></span> <span data-ttu-id="f2211-132">Используйте **обозреватель объектов SQL Server** (SSOX) или `database drop` команду CLI:</span><span class="sxs-lookup"><span data-stu-id="f2211-132">Use **SQL Server Object Explorer** (SSOX) or the `database drop` CLI command:</span></span>
> ```console
> dotnet ef database drop
> ```
> <span data-ttu-id="f2211-133">Следующий раздел объясняет, как для выполнения команд CLI.</span><span class="sxs-lookup"><span data-stu-id="f2211-133">The following section explains how to run CLI commands.</span></span>

## <a name="create-an-initial-migration"></a><span data-ttu-id="f2211-134">Создание первоначальной миграции</span><span class="sxs-lookup"><span data-stu-id="f2211-134">Create an initial migration</span></span>

<span data-ttu-id="f2211-135">Сохраните изменения и выполните построение проекта.</span><span class="sxs-lookup"><span data-stu-id="f2211-135">Save your changes and build the project.</span></span> <span data-ttu-id="f2211-136">Затем откройте окно командной строки и перейдите в папку проекта.</span><span class="sxs-lookup"><span data-stu-id="f2211-136">Then open a command window and navigate to the project folder.</span></span> <span data-ttu-id="f2211-137">Вот быстрый способ сделать это:</span><span class="sxs-lookup"><span data-stu-id="f2211-137">Here's a quick way to do that:</span></span>

* <span data-ttu-id="f2211-138">В **обозревателе решений**, щелкните правой кнопкой мыши проект и выберите **открыть в проводнике** в контекстном меню.</span><span class="sxs-lookup"><span data-stu-id="f2211-138">In **Solution Explorer**, right-click the project and choose **Open in File Explorer** from the context menu.</span></span>

  ![Открыть в проводнике пункта меню](migrations/_static/open-in-file-explorer.png)

* <span data-ttu-id="f2211-140">В адресной строке введите «cmd» и нажмите клавишу ВВОД.</span><span class="sxs-lookup"><span data-stu-id="f2211-140">Enter "cmd" in the address bar and press Enter.</span></span>

  ![Откройте командное окно](migrations/_static/open-command-window.png)

<span data-ttu-id="f2211-142">В командной строке введите следующую команду:</span><span class="sxs-lookup"><span data-stu-id="f2211-142">Enter the following command in the command window:</span></span>

```console
dotnet ef migrations add InitialCreate
```

<span data-ttu-id="f2211-143">Вывод в командной строке следующим образом:</span><span class="sxs-lookup"><span data-stu-id="f2211-143">You see output like the following in the command window:</span></span>

```console
info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[0]
      User profile is available. Using 'C:\Users\username\AppData\Local\ASP.NET\DataProtection-Keys' as key repository and Windows DPAPI to encrypt keys at rest.
info: Microsoft.EntityFrameworkCore.Infrastructure[100403]
      Entity Framework Core 2.0.0-rtm-26452 initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
Done. To undo this action, use 'ef migrations remove'
```

> [!NOTE]
> <span data-ttu-id="f2211-144">Если вы видите сообщение об ошибке *не исполняемый объект найден соответствующий команда «dotnet-ef»*, в разделе [этой записи блога](http://thedatafarm.com/data-access/no-executable-found-matching-command-dotnet-ef/) справку по устранению неполадок.</span><span class="sxs-lookup"><span data-stu-id="f2211-144">If you see an error message *No executable found matching command "dotnet-ef"*, see [this blog post](http://thedatafarm.com/data-access/no-executable-found-matching-command-dotnet-ef/) for help troubleshooting.</span></span>

<span data-ttu-id="f2211-145">Если вы видите сообщение об ошибке «*не может получить доступ к файла... ContosoUniversity.dll, так как он используется другим процессом.* » значок IIS Express поиска на панели задач Windows, щелкните его правой кнопкой мыши, затем щелкните **ContosoUniversity > остановка узла**.</span><span class="sxs-lookup"><span data-stu-id="f2211-145">If you see an error message "*cannot access the file ... ContosoUniversity.dll because it is being used by another process.*", find the IIS Express icon in the Windows System Tray, and right-click it, then click **ContosoUniversity > Stop Site**.</span></span>

## <a name="examine-the-up-and-down-methods"></a><span data-ttu-id="f2211-146">Изучите вверх и вниз методы</span><span class="sxs-lookup"><span data-stu-id="f2211-146">Examine the Up and Down methods</span></span>

<span data-ttu-id="f2211-147">При выполнении `migrations add` команды EF созданный код, который создаст базу данных с нуля.</span><span class="sxs-lookup"><span data-stu-id="f2211-147">When you executed the `migrations add` command, EF generated the code that will create the database from scratch.</span></span> <span data-ttu-id="f2211-148">Этот код находится в *миграций* папки в файл с именем  *\<timestamp > _InitialCreate.cs*.</span><span class="sxs-lookup"><span data-stu-id="f2211-148">This code is in the *Migrations* folder, in the file named *\<timestamp>_InitialCreate.cs*.</span></span> <span data-ttu-id="f2211-149">`Up` Метод `InitialCreate` класс создает таблицы базы данных, которые соответствуют наборов сущностей модели данных, и `Down` метод удаляет их, как показано в следующем примере.</span><span class="sxs-lookup"><span data-stu-id="f2211-149">The `Up` method of the `InitialCreate` class creates the database tables that correspond to the data model entity sets, and the `Down` method deletes them, as shown in the following example.</span></span>

<span data-ttu-id="f2211-150">[!code-csharp[Main](intro/samples/cu/Migrations/20170215220724_InitialCreate.cs?range=92-118)]</span><span class="sxs-lookup"><span data-stu-id="f2211-150">[!code-csharp[Main](intro/samples/cu/Migrations/20170215220724_InitialCreate.cs?range=92-118)]</span></span>

<span data-ttu-id="f2211-151">Миграция вызовы `Up` метод для реализации изменений модели данных для миграции.</span><span class="sxs-lookup"><span data-stu-id="f2211-151">Migrations calls the `Up` method to implement the data model changes for a migration.</span></span> <span data-ttu-id="f2211-152">При вводе команды для отката обновления, вызывает миграции `Down` метод.</span><span class="sxs-lookup"><span data-stu-id="f2211-152">When you enter a command to roll back the update, Migrations calls the `Down` method.</span></span>

<span data-ttu-id="f2211-153">Данный пример кода является для первоначальной миграции, который был создан при вводе `migrations add InitialCreate` команды.</span><span class="sxs-lookup"><span data-stu-id="f2211-153">This code is for the initial migration that was created when you entered the `migrations add InitialCreate` command.</span></span> <span data-ttu-id="f2211-154">Параметр name миграции («InitialCreate» в примере) используется для имени файла, а также можно указать любое.</span><span class="sxs-lookup"><span data-stu-id="f2211-154">The migration name parameter ("InitialCreate" in the example) is used for the file name and can be whatever you want.</span></span> <span data-ttu-id="f2211-155">Рекомендуется выбрать слово или фразу, в которой говорится, что выполняется в процессе переноса.</span><span class="sxs-lookup"><span data-stu-id="f2211-155">It's best to choose a word or phrase that summarizes what is being done in the migration.</span></span> <span data-ttu-id="f2211-156">Например может имя последующей миграции «AddDepartmentTable».</span><span class="sxs-lookup"><span data-stu-id="f2211-156">For example, you might name a later migration "AddDepartmentTable".</span></span>

<span data-ttu-id="f2211-157">При создании первоначальной миграции базы данных уже существует, создается код создания базы данных, но он не должен выполняться, так как база данных, уже соответствует модели данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-157">If you created the initial migration when the database already exists, the database creation code is generated but it doesn't have to run because the database already matches the data model.</span></span> <span data-ttu-id="f2211-158">При развертывании приложения в другую среду, где базы данных не существует, однако, этот код будет выполняться Создание базы данных, поэтому рекомендуется сначала протестировать.</span><span class="sxs-lookup"><span data-stu-id="f2211-158">When you deploy the app to another environment where the database doesn't exist yet, this code will run to create your database, so it's a good idea to test it first.</span></span> <span data-ttu-id="f2211-159">Вот почему было изменено имя базы данных в более ранней версии — строку подключения, чтобы миграции можно создать новую с нуля.</span><span class="sxs-lookup"><span data-stu-id="f2211-159">That's why you changed the name of the database in the connection string earlier -- so that migrations can create a new one from scratch.</span></span>

## <a name="examine-the-data-model-snapshot"></a><span data-ttu-id="f2211-160">Изучите снимок модели данных</span><span class="sxs-lookup"><span data-stu-id="f2211-160">Examine the data model snapshot</span></span>

<span data-ttu-id="f2211-161">Миграция также создает *моментального снимка* текущей схемы базы данных в *Migrations/SchoolContextModelSnapshot.cs*.</span><span class="sxs-lookup"><span data-stu-id="f2211-161">Migrations also creates a *snapshot* of the current database schema in *Migrations/SchoolContextModelSnapshot.cs*.</span></span> <span data-ttu-id="f2211-162">Вот, как выглядит этот код:</span><span class="sxs-lookup"><span data-stu-id="f2211-162">Here's what that code looks like:</span></span>

<span data-ttu-id="f2211-163">[!code-csharp[Main](intro/samples/cu/Migrations/SchoolContextModelSnapshot1.cs?name=snippet_Truncate)]</span><span class="sxs-lookup"><span data-stu-id="f2211-163">[!code-csharp[Main](intro/samples/cu/Migrations/SchoolContextModelSnapshot1.cs?name=snippet_Truncate)]</span></span>

<span data-ttu-id="f2211-164">Поскольку текущей схемы базы данных представлен в коде, EF Core не должен взаимодействовать с базой данных для создания миграции.</span><span class="sxs-lookup"><span data-stu-id="f2211-164">Because the current database schema is represented in code, EF Core doesn't have to interact with the database to create migrations.</span></span> <span data-ttu-id="f2211-165">При добавлении миграции EF определяет, что изменилось, сравнивая модели данных в файле моментального снимка.</span><span class="sxs-lookup"><span data-stu-id="f2211-165">When you add a migration, EF determines what changed by comparing the data model to the snapshot file.</span></span> <span data-ttu-id="f2211-166">EF взаимодействует с базой данных, только когда требуется обновить базу данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-166">EF interacts with the database only when it has to update the database.</span></span> 

<span data-ttu-id="f2211-167">Должен обеспечивать синхронизацию с миграций, которые ее создать, поэтому невозможно удалить миграции, просто удалив файл с именем файла моментального снимка  *\<timestamp > _\<migrationname > .cs*.</span><span class="sxs-lookup"><span data-stu-id="f2211-167">The snapshot file has to be kept in sync with the migrations that create it, so you can't remove a migration just by deleting the file named  *\<timestamp>_\<migrationname>.cs*.</span></span> <span data-ttu-id="f2211-168">Если вы удалите этот файл, оставшиеся операции миграции будут синхронизированы с файлом моментального снимка базы данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-168">If you delete that file, the remaining migrations will be out of sync with the database snapshot file.</span></span> <span data-ttu-id="f2211-169">Чтобы удалить добавленный последней миграции, используйте [dotnet ef миграции удалить](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) команды.</span><span class="sxs-lookup"><span data-stu-id="f2211-169">To delete the last migration that you added, use the [dotnet ef migrations remove](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove) command.</span></span>

## <a name="apply-the-migration-to-the-database"></a><span data-ttu-id="f2211-170">Применить к базе данных миграции</span><span class="sxs-lookup"><span data-stu-id="f2211-170">Apply the migration to the database</span></span>

<span data-ttu-id="f2211-171">В окне командной строки введите следующую команду, чтобы создать базу данных и таблиц в ней.</span><span class="sxs-lookup"><span data-stu-id="f2211-171">In the command window, enter the following command to create the database and tables in it.</span></span>

```console
dotnet ef database update
```

<span data-ttu-id="f2211-172">Выходные данные команды аналогичен `migrations add` команды, за исключением того, что вы видите журналы для команд, Настройка базы данных SQL.</span><span class="sxs-lookup"><span data-stu-id="f2211-172">The output from the command is similar to the `migrations add` command, except that you see logs for the SQL commands that set up the database.</span></span> <span data-ttu-id="f2211-173">Большинство журналов опущены в следующий результат.</span><span class="sxs-lookup"><span data-stu-id="f2211-173">Most of the logs are omitted in the following sample output.</span></span> <span data-ttu-id="f2211-174">Если вы предпочитаете не см. такой уровень детализации сообщений журнала, можно изменить уровни журнала в *appsettings. Development.JSON* файл.</span><span class="sxs-lookup"><span data-stu-id="f2211-174">If you prefer not to see this level of detail in log messages, you can change the log levels in the *appsettings.Development.json* file.</span></span> <span data-ttu-id="f2211-175">Дополнительные сведения см. в разделе [введение к ведению журнала](xref:fundamentals/logging).</span><span class="sxs-lookup"><span data-stu-id="f2211-175">For more information, see [Introduction to logging](xref:fundamentals/logging).</span></span>

```text
info: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[0]
      User profile is available. Using 'C:\Users\username\AppData\Local\ASP.NET\DataProtection-Keys' as key repository and Windows DPAPI to encrypt keys at rest.
info: Microsoft.EntityFrameworkCore.Infrastructure[100403]
      Entity Framework Core 2.0.0-rtm-26452 initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
info: Microsoft.EntityFrameworkCore.Database.Command[200101]
      Executed DbCommand (467ms) [Parameters=[], CommandType='Text', CommandTimeout='60']
      CREATE DATABASE [ContosoUniversity2];
info: Microsoft.EntityFrameworkCore.Database.Command[200101]
      Executed DbCommand (20ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      CREATE TABLE [__EFMigrationsHistory] (
          [MigrationId] nvarchar(150) NOT NULL,
          [ProductVersion] nvarchar(32) NOT NULL,
          CONSTRAINT [PK___EFMigrationsHistory] PRIMARY KEY ([MigrationId])
      );

<logs omitted for brevity>

info: Microsoft.EntityFrameworkCore.Database.Command[200101]
      Executed DbCommand (3ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      INSERT INTO [__EFMigrationsHistory] ([MigrationId], [ProductVersion])
      VALUES (N'20170816151242_InitialCreate', N'2.0.0-rtm-26452');
Done.
```

<span data-ttu-id="f2211-176">Используйте **обозреватель объектов SQL Server** проверить базу данных, как описано в первом руководстве.</span><span class="sxs-lookup"><span data-stu-id="f2211-176">Use **SQL Server Object Explorer** to inspect the database as you did in the first tutorial.</span></span>  <span data-ttu-id="f2211-177">Вы заметите Добавление таблицы __EFMigrationsHistory, отслеживающей миграций, которые были применены к базе данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-177">You'll notice the addition of an __EFMigrationsHistory table that keeps track of which migrations have been applied to the database.</span></span> <span data-ttu-id="f2211-178">Просмотр данных в этой таблице, и вы увидите одну строку для первой миграции.</span><span class="sxs-lookup"><span data-stu-id="f2211-178">View the data in that table and you'll see one row for the first migration.</span></span> <span data-ttu-id="f2211-179">(Последний журнал в предыдущем примере вывода CLI показывает инструкции INSERT, которая создает эту строку).</span><span class="sxs-lookup"><span data-stu-id="f2211-179">(The last log in the preceding CLI output example shows the INSERT statement that creates this row.)</span></span>

<span data-ttu-id="f2211-180">Запустите приложение, чтобы убедиться, что все работает по-прежнему прежней.</span><span class="sxs-lookup"><span data-stu-id="f2211-180">Run the application to verify that everything still works the same as before.</span></span>

![Страница индекса студентов](migrations/_static/students-index.png)

<a id="pmc"></a>
## <a name="command-line-interface-cli-vs-package-manager-console-pmc"></a><span data-ttu-id="f2211-182">Интерфейс командной строки (CLI) vs. Консоль диспетчера пакетов (PMC)</span><span class="sxs-lookup"><span data-stu-id="f2211-182">Command-line interface (CLI) vs. Package Manager Console (PMC)</span></span>

<span data-ttu-id="f2211-183">EF средства управления миграции доступен из команды .NET Core CLI или командлеты PowerShell в Visual Studio **консоль диспетчера пакетов** (PMC) окна.</span><span class="sxs-lookup"><span data-stu-id="f2211-183">The EF tooling for managing migrations is available from .NET Core CLI commands or from PowerShell cmdlets in the Visual Studio **Package Manager Console** (PMC) window.</span></span> <span data-ttu-id="f2211-184">Этот учебник показывает, как использовать CLI, но при желании можно указать PMC.</span><span class="sxs-lookup"><span data-stu-id="f2211-184">This tutorial shows how to use the CLI, but you can use the PMC if you prefer.</span></span>

<span data-ttu-id="f2211-185">EF команды для команд PMC, в [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools) пакета.</span><span class="sxs-lookup"><span data-stu-id="f2211-185">The EF commands for the PMC commands are in the [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools) package.</span></span> <span data-ttu-id="f2211-186">Этот пакет уже включены в [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage, поэтому устанавливать его не требуется.</span><span class="sxs-lookup"><span data-stu-id="f2211-186">This package is already included in the [Microsoft.AspNetCore.All](xref:fundamentals/metapackage) metapackage, so you don't have to install it.</span></span>

<span data-ttu-id="f2211-187">**Важно:** это не тот же пакет, как установить для CLI, изменив *.csproj* файла.</span><span class="sxs-lookup"><span data-stu-id="f2211-187">**Important:** This is not the same package as the one you install for the CLI by editing the *.csproj* file.</span></span> <span data-ttu-id="f2211-188">Имя данного объекта заканчивается на `Tools`, в отличие от имени пакета CLI, который заканчивается на `Tools.DotNet`.</span><span class="sxs-lookup"><span data-stu-id="f2211-188">The name of this one ends in `Tools`, unlike the CLI package name which ends in `Tools.DotNet`.</span></span>

<span data-ttu-id="f2211-189">Дополнительные сведения о командах CLI см. в разделе [.NET Core CLI](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet).</span><span class="sxs-lookup"><span data-stu-id="f2211-189">For more information about the CLI commands, see [.NET Core CLI](https://docs.microsoft.com/ef/core/miscellaneous/cli/dotnet).</span></span> 

<span data-ttu-id="f2211-190">Дополнительные сведения о командах PMC см. в разделе [Package Manager Console (Visual Studio)](https://docs.microsoft.com/ef/core/miscellaneous/cli/powershell).</span><span class="sxs-lookup"><span data-stu-id="f2211-190">For more information about the PMC commands, see [Package Manager Console (Visual Studio)](https://docs.microsoft.com/ef/core/miscellaneous/cli/powershell).</span></span>

## <a name="summary"></a><span data-ttu-id="f2211-191">Сводка</span><span class="sxs-lookup"><span data-stu-id="f2211-191">Summary</span></span>

<span data-ttu-id="f2211-192">В этом учебнике вы знаете, как создать и применить переноса первого.</span><span class="sxs-lookup"><span data-stu-id="f2211-192">In this tutorial, you've seen how to create and apply your first migration.</span></span> <span data-ttu-id="f2211-193">В следующем уроке вы начнете, просмотрев более сложные вопросы, развернув модели данных.</span><span class="sxs-lookup"><span data-stu-id="f2211-193">In the next tutorial, you'll begin looking at more advanced topics by expanding the data model.</span></span> <span data-ttu-id="f2211-194">Попутно вы создадите и применения дополнительных миграции.</span><span class="sxs-lookup"><span data-stu-id="f2211-194">Along the way you'll create and apply additional migrations.</span></span>

>[!div class="step-by-step"]
<span data-ttu-id="f2211-195">[Назад](sort-filter-page.md)
[Вперед](complex-data-model.md)</span><span class="sxs-lookup"><span data-stu-id="f2211-195">[Previous](sort-filter-page.md)
[Next](complex-data-model.md)</span></span>  