---
title: Начало работы с Razor Pages ASP.NET Core в Visual Studio Code
author: rick-anderson
description: Основные сведения о создании веб-приложении Razor Pages в ASP.NET Core с помощью Visual Studio Code.
manager: wpickett
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.date: 08/27/2017
ms.prod: aspnet-core
ms.technology: aspnet
ms.topic: get-started-article
uid: tutorials/razor-pages-vsc/razor-pages-start
ms.openlocfilehash: 0ad008b4f2b2e74dcf7f3d6c83798d5f03d1d315
ms.sourcegitcommit: c79fd3592f444d58e17518914f8873d0a11219c0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/18/2018
---
# <a name="get-started-with-aspnet-core-razor-pages-in-visual-studio-code"></a>Начало работы с Razor Pages ASP.NET Core в Visual Studio Code

Автор: [Рик Андерсон](https://twitter.com/RickAndMSFT) (Rick Anderson)

В этом учебнике приводятся основные сведения о веб-приложении Razor Pages в ASP.NET Core. Перед работой с этим учебником рекомендуем изучить статью [Введение в Razor Pages](xref:mvc/razor-pages/index). Razor Pages — это рекомендуемый способ создания пользовательского интерфейса для веб-приложений в ASP.NET Core.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [](~/includes/net-core-prereqs-vscode.md)]

## <a name="create-a-razor-web-app"></a>Создание веб-приложения Razor

Из терминала выполните следующие команды.

```console
dotnet new razor -o RazorPagesMovie
cd RazorPagesMovie
dotnet run
```

Указанные выше команды используют [интерфейс командной строки .NET Core](https://docs.microsoft.com/dotnet/core/tools/dotnet) для создания и запуска проекта Razor Pages. Откройте в браузере страницу http://localhost:5000 для просмотра приложения.

![Домашняя или индексная страница](../razor-pages/razor-pages-start/_static/home.png)

[!INCLUDE [razor-pages-start](../../includes/RP/razor-pages-start.md)]

## <a name="open-the-project"></a>Открытие проекта

Нажмите клавиши CTRL+C, чтобы завершить работу приложения.

В Visual Studio Code (VS Code) откройте меню **Файл > Открыть папку** и выберите папку *RazorPagesMovie*.

- Нажмите кнопку **Да** в **предупреждении** "Required assets to build and debug are missing from 'TodoApi'.Add them?" (В TodoApi отсутствуют необходимые ресурсы для сборки и отладки. Добавить их?).
- Щелкните **Восстановить** в **информационном** сообщении "There are unresolved dependencies" (Имеются неразрешенные зависимости).

### <a name="launch-the-app"></a>Запуск приложения

Нажмите клавиши CTRL+F5, чтобы запустить приложение без отладки. Либо в меню **Отладка** выберите пункт **Запуск без отладки**.

В следующем учебнике мы добавим в проект модель. 

> [!div class="step-by-step"]
> [Далее: добавление модели](xref:tutorials/razor-pages-vsc/model)  
