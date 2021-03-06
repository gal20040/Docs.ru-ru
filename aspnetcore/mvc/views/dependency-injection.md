---
title: Внедрение зависимостей в представления в ASP.NET Core
author: ardalis
description: Узнайте, как ASP.NET Core поддерживает внедрение зависимостей в представления MVC.
manager: wpickett
ms.author: riande
ms.date: 10/14/2016
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/views/dependency-injection
ms.openlocfilehash: cc34b9069ec062f08644c0026c1ccdcd00f667ac
ms.sourcegitcommit: 48beecfe749ddac52bc79aa3eb246a2dcdaa1862
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2018
---
# <a name="dependency-injection-into-views-in-aspnet-core"></a>Внедрение зависимостей в представления в ASP.NET Core

Автор: [Стив Смит](https://ardalis.com/) (Steve Smith)

ASP.NET Core поддерживает [внедрение зависимостей](xref:fundamentals/dependency-injection) в представления. Это может быть полезно для связанных с представлениями служб (например, локализации) или данных, необходимых только для заполнения элементов представления. Старайтесь поддерживать [разделение ответственности](http://deviq.com/separation-of-concerns/) между контроллерами и представлениями. Большая часть данных, отображаемых представлениями, должна передаваться от контроллера.

[Просмотреть или скачать образец кода](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/views/dependency-injection/sample) ([как скачивать](xref:tutorials/index#how-to-download-a-sample))

## <a name="a-simple-example"></a>Простой пример

Вы можете внедрить в представление службу с помощью директивы `@inject`. По сути, `@inject` добавляет в представление свойство и заполняет это свойство с помощью внедрения зависимостей.

Синтаксис для `@inject`: `@inject <type> <name>`

Пример `@inject` в действии:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/ToDo/Index.cshtml?highlight=4,5,15,16,17)]

Это представление отображает список экземпляров `ToDoItem`, а также сводку по общей статистике. Сводка заполняется из внедренного `StatisticsService`. Эта служба регистрируется для внедрения зависимости в `ConfigureServices` в файле *Startup.cs*:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Startup.cs?highlight=6,7&range=15-22)]

Функция `StatisticsService` выполняет ряд вычислений с набором экземпляров `ToDoItem`, к которым она обращается через репозиторий:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/StatisticsService.cs?highlight=15,20,25)]

Пример репозитория использует коллекцию в памяти. Представленная выше реализация (работающая со всеми данными в памяти) не рекомендуется для больших наборов данных с удаленным доступом.

Пример показывает данные из модели, привязанной к представлению, и из внедренной в представление службы:

![Список дел к выполнению, показывающий общее количество пунктов, число выполненных пунктов, средний показатель приоритета, а также список задач с их уровнями приоритета и логическими значениями, характеризующими выполнение.](dependency-injection/_static/screenshot.png)

## <a name="populating-lookup-data"></a>Заполнение данных подстановки

Внедрение в представление можно использовать для заполнения параметров в элементах пользовательского интерфейса, например в раскрывающихся списках. Рассмотрим форму профиля пользователя, содержащую параметры для указания пола, штата и другие свойства. Для отрисовки такой формы в стандартном подходе MVC контроллеру нужно будет запрашивать каждый из этих наборов параметров в службах доступа к данным, а затем заполнять привязываемыми наборами модель или `ViewBag`.

Альтернативный вариант получения параметров заключается во внедрении служб непосредственно в представление. В этом случае необходимый контроллеру код сводится к минимуму, так как логика создания элемента представления помещается в само представление. Действие контроллера по отображению формы редактирования профиля должно только передать форму экземпляру профиля:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Controllers/ProfileController.cs?highlight=9,19)]

HTML-форма, используемая для изменения этих свойств, содержит для трех свойств раскрывающиеся списки:

![Представление редактирования профиля с формой для ввода имени, пола, штата и любимого цвета.](dependency-injection/_static/updateprofile.png)

Эти списки заполняются службой, которая внедрена в представление:

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Profile/Index.cshtml?highlight=4,16,17,21,22,26,27)]

`ProfileOptionsService` — это работающая на уровне пользовательского интерфейса служба, которая предоставляет именно те данные, которые нужны в этой форме:

[!code-csharp[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Model/Services/ProfileOptionsService.cs?highlight=7,13,24)]

>[!TIP]
> Не забудьте зарегистрировать типы, которые вы будете запрашивать через внедрение зависимостей в методе `ConfigureServices` в файле *Startup.cs*.

## <a name="overriding-services"></a>Переопределение служб

Помимо добавления новых служб, этот метод также позволяет переопределять ранее добавленные службы на странице. На следующем рисунке показаны все доступные поля на странице, используемой в первом примере:

![Контекстное меню IntelliSense к введенному символу @ с полями Html, Component, StatsService и Url](dependency-injection/_static/razor-fields.png)

Как видно, полями по умолчанию являются `Html`, `Component` и `Url` (а также внедренное нами поле `StatsService`). Если, например, нужно заменить вспомогательные методы HTML на собственные, вы можете легко сделать это с помощью `@inject`:

[!code-cshtml[](../../mvc/views/dependency-injection/sample/src/ViewInjectSample/Views/Helper/Index.cshtml?highlight=3,11)]

При необходимости расширить существующие службы вы можете просто применять этот метод при наследовании от существующей реализации или использовании ее внутри вашей собственной.

## <a name="see-also"></a>См. также

* Блог Саймона Тиммса (Simon Timms): [Getting Lookup Data Into Your View](http://blog.simontimms.com/2015/06/09/getting-lookup-data-into-you-view/) (Внедрение данных подстановки в ваше представление)
