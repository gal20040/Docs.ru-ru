---
title: Публикация приложения ASP.NET Core в Azure с использованием средств командной строки
author: camsoper
description: Сведения о публикации приложения ASP.NET Core в службе приложений Azure с помощью клиента командной строки Git.
manager: wpickett
ms.author: casoper
ms.custom: mvc
ms.date: 11/03/2017
ms.devlang: dotnet
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: get-started-article
services: multiple
uid: tutorials/publish-to-azure-webapp-using-cli
ms.openlocfilehash: 0462a4cf18bba23643ed3b1b4e6b76bdbceb24a8
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
# <a name="publish-an-aspnet-core-app-to-azure-with-command-line-tools"></a>Публикация приложения ASP.NET Core в Azure с использованием средств командной строки

Автор [Кэм Сопер (Cam Soper)](https://twitter.com/camsoper)

[!INCLUDE [Azure App Service Preview Notice](../includes/azure-apps-preview-notice.md)]

В этом руководстве рассказывается, как создавать и развертывать приложение ASP.NET Core в службе приложений Microsoft Azure с помощью программ командной строки.  После завершения у вас будет веб-приложение, созданное в ASP.NET MVC Core, размещенное в качестве веб-приложения службы приложений Azure.  Это руководство написано с использованием программ командной строки Windows, но оно также может применяться в средах MacOS и Linux.  

В этом руководстве вы узнаете, как:

> [!div class="checklist"]
> * создать веб-сайт службы приложений Azure с помощью Azure CLI;
> * развернуть приложение ASP.NET Core в службе приложений Azure, используя программу командной строки Git.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством вам понадобится:

* [Подписка на Microsoft Azure](https://azure.microsoft.com/free/).
* [!INCLUDE [](~/includes/net-core-sdk-download-link.md)]
* Клиент командной строки [Git](https://www.git-scm.com/).

## <a name="create-a-web-application"></a>Создание веб-приложения

Создайте каталог для веб-приложения, создайте приложение ASP.NET Core MVC, а затем запустите веб-сайт локально.

# <a name="windowstabwindows"></a>[Windows](#tab/windows)
```cmd
REM Create a new ASP.NET Core MVC application
dotnet new razor -o MyApplication

REM Change to the new directory that was just created
cd MyApplication

REM Run the application
dotnet run
```

# <a name="othertabother"></a>[Другое](#tab/other)
```bash
# Create a new ASP.NET Core MVC application
dotnet new razor -o MyApplication

# Change to the new directory that was just created
cd MyApplication

# Run the application
dotnet run
```
---

![Данные, выводимые в командной строке](publish-to-azure-webapp-using-cli/_static/new_prj.png)

Проверьте приложение, перейдя по адресу http://localhost:5000.

![Веб-сайт, работающий локально](publish-to-azure-webapp-using-cli/_static/app_test.png)


## <a name="create-the-azure-app-service-instance"></a>Создание экземпляра службы приложений Azure

Используя [Azure Cloud Shell](/azure/cloud-shell/quickstart), создайте группу ресурсов, план и веб-приложение службы приложений.

```azurecli-interactive
# Generate a unique Web App name
let randomNum=$RANDOM*$RANDOM
webappname=tutorialApp$randomNum

# Create the DotNetAzureTutorial resource group
az group create --name DotNetAzureTutorial --location EastUS

# Create an App Service plan.
az appservice plan create --name $webappname --resource-group DotNetAzureTutorial --sku FREE

# Create the Web App
az webapp create --name $webappname --resource-group DotNetAzureTutorial --plan $webappname
```

Перед развертыванием настройте учетные данные для развертывания на уровне учетной записи, используя следующую команду:

```azurecli-interactive
az webapp deployment user set --user-name <desired user name> --password <desired password>
```

URL-адрес развертывания необходим для развертывания приложения с использованием Git.  Получите URL-адрес следующим образом.

```azurecli-interactive
az webapp deployment source config-local-git -n $webappname -g DotNetAzureTutorial --query [url] -o tsv
```
Запишите отображаемый URL-адрес, заканчивающийся на `.git`. Он используется на следующем шаге.

## <a name="deploy-the-application-using-git"></a>Развертывание приложения с помощью Git

Все готово к развертыванию с локального компьютера с помощью Git.

> [!NOTE]
> Любые предупреждения из Git об окончаниях строк можно игнорировать, не опасаясь.

# <a name="windowstabwindows"></a>[Windows](#tab/windows)
```cmd
REM Initialize the local Git repository
git init

REM Add the contents of the working directory to the repo
git add --all

REM Commit the changes to the local repo
git commit -a -m "Initial commit"

REM Add the URL as a Git remote repository
git remote add azure <THE GIT URL YOU NOTED EARLIER>

REM Push the local repository to the remote
git push azure master
```

# <a name="othertabother"></a>[Другое](#tab/other)
```bash
# Initialize the local Git repository
git init

# Add the contents of the working directory to the repo
git add --all

# Commit the changes to the local repo
git commit -a -m "Initial commit"

# Add the URL as a Git remote repository
git remote add azure <THE GIT URL YOU NOTED EARLIER>

# Push the local repository to the remote
git push azure master
```
---

Git предлагает ввести учетные данные развертывания, заданные ранее. После проверки подлинности приложение будет перенесено в удаленное местоположение, создано и развернуто.

![Выходные данные развертывания Git](publish-to-azure-webapp-using-cli/_static/post_deploy.png)

## <a name="test-the-application"></a>Тестирование приложения

Проверьте приложение, перейдя по адресу `https://<web app name>.azurewebsites.net`.  Для отображения адреса в Cloud Shell (или Azure CLI) используйте следующую команду:

```azurecli-interactive
az webapp show -n $webappname -g DotNetAzureTutorial --query defaultHostName -o tsv
```

![Приложение, работающее в Azure](publish-to-azure-webapp-using-cli/_static/app_deployed.png)

## <a name="clean-up"></a>Очистка

После проверки приложения, кода и ресурсов удалите веб-приложение и план, удалив группу ресурсов.

```azurecli-interactive
az group delete -n DotNetAzureTutorial
```

## <a name="next-steps"></a>Следующие шаги

В этом руководстве вы узнали, как:

> [!div class="checklist"]
> * создать веб-сайт службы приложений Azure с помощью Azure CLI;
> * развернуть приложение ASP.NET Core в службе приложений Azure, используя программу командной строки Git.

Теперь научитесь развертывать имеющееся веб-приложение, использующее Cosmos DB, с помощью командной строки.

> [!div class="nextstepaction"]
> [Развертывание приложения в Azure из командной строки с помощью .NET Core](/dotnet/azure/dotnet-quickstart-xplat)
