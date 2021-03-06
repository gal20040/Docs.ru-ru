---
title: Работа с распределенным кэшем в ASP.NET Core
author: ardalis
description: Сведения об использовании ASP.NET Core распределенного кэширования для повышения производительности приложения и масштабируемость, особенно в среде фермы облако или сервера.
manager: wpickett
ms.author: riande
ms.date: 02/14/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: performance/caching/distributed
ms.openlocfilehash: c40209e3b3f2b5bf28450bb2a88cbe40e9e23230
ms.sourcegitcommit: 9bc34b8269d2a150b844c3b8646dcb30278a95ea
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/12/2018
---
# <a name="work-with-a-distributed-cache-in-aspnet-core"></a>Работа с распределенным кэшем в ASP.NET Core

Автор: [Стив Смит](https://ardalis.com/) (Steve Smith)

Распределенные кэши могут улучшить производительность и масштабируемость приложений ASP.NET Core, особенно в том случае, если размещенные в среде фермы облако или сервера. В этой статье объясняется, как работать с ASP.NET Core встроенных распределенного кэша абстракции и реализации.

[Просмотреть или скачать образец кода](https://github.com/aspnet/Docs/tree/master/aspnetcore/performance/caching/distributed/sample) ([как скачивать](xref:tutorials/index#how-to-download-a-sample))

## <a name="what-is-a-distributed-cache"></a>Что такое распределенного кэша

Распределенный кэш является общим для нескольких серверов приложений (см. [основы кэша](memory.md#caching-basics)). Информация в кэше не хранятся в памяти отдельных веб-серверов и кэшированных данных доступен для всех серверов приложений. Это обеспечивает несколько преимуществ:

1. Кэшированные данные согласованного на всех веб-серверах. Пользователи не видеть разные результаты в зависимости от веб-сервер обрабатывает их запроса

2. Кэшированные данные сохраняется после перезагрузки сервера web и развертываний. Отдельные веб-серверов можно удалить или добавить без ущерба для кэша.

3. Хранилище данных источника содержит меньшее число запросов, выполненных на него (не с нескольких кэшей в памяти или нет кэшировать вообще).

> [!NOTE]
> Если используется SQL Server распределенного кэша, некоторые из этих преимуществ доступны только в значение true, если экземпляр отдельную базу данных для кэша, чем для источника данных приложения.

Как и любой кэш распределенного кэша могут существенно повысить скорость реагирования приложения, поскольку обычно можно получить данные из кэша намного быстрее, чем из реляционной базы данных (или веб-службы).

Конфигурация кэша зависит от конкретной реализации. В этой статье описывается настройка оба Redis и кэширует распределенных SQL Server. Независимо от того, какая реализация установлен, приложение взаимодействует с кэш, используя общий `IDistributedCache` интерфейс.

## <a name="the-idistributedcache-interface"></a>Интерфейс IDistributedCache

`IDistributedCache` Интерфейс включает синхронные и асинхронные методы. Интерфейс позволяет элементы добавлены, получение и удалены из реализации распределенного кэша. `IDistributedCache` Интерфейс содержит следующие методы:

**Get-GetAsync**

Принимает строковый ключ и извлекает кэшированного элемента как `byte[]` Если найден в кэше.

**Набор SetAsync**

Добавляет элемент (как `byte[]`) в кэш с помощью строкового ключа.

**Обновление RefreshAsync**

Обновляет объект в кэше, по его ключу, сброс его скользящий срок хранения (если таковые имеются).

**Удалите асинхронно удаляет**

Удаляет запись по его ключу.

Для использования `IDistributedCache` интерфейс:

   1. Добавьте необходимые пакеты NuGet в файл проекта.

   2. Настройка деталей реализации `IDistributedCache` в ваш `Startup` класса `ConfigureServices` метода и добавьте его в контейнер существует.

   3. В приложении [по промежуточного слоя](xref:fundamentals/middleware/index) или классов контроллеров MVC, запросить экземпляр `IDistributedCache` из конструктора. Экземпляр будут предоставляться [внедрения зависимостей](../../fundamentals/dependency-injection.md) (DI).

> [!NOTE]
> Нет необходимости использовать единственный элемент или Scoped время существования `IDistributedCache` экземпляров (хотя бы для встроенных реализаций). Можно также создать экземпляр везде, где может потребоваться одно (вместо использования [внедрения зависимостей](../../fundamentals/dependency-injection.md)), но это может усложнить код для тестирования и приводит к нарушению [явные зависимости принцип](http://deviq.com/explicit-dependencies-principle/).

В следующем примере показано, как использовать экземпляр `IDistributedCache` в простой программный компонент:

[!code-csharp[](./distributed/sample/src/DistCacheSample/StartTimeHeader.cs?highlight=15,18,21,27,28,29,30,31)]

В приведенном выше коде кэшированное значение является считываются, но никогда не записываются. В этом примере значение устанавливается только при запуске сервера, а также не изменяется. В случае многосерверной последний сервер запуск перезапишет все предыдущие значения, заданные на других серверах. `Get` И `Set` методы используют `byte[]` типа. Таким образом, строковое значение необходимо преобразовывать с помощью `Encoding.UTF8.GetString` (для `Get`) и `Encoding.UTF8.GetBytes` (для `Set`).

В следующем примере кода из *файла Startup.cs* показывает, что будет задаваться значение:

[!code-csharp[](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=2,4,5,6&range=58-66)]

> [!NOTE]
> Поскольку `IDistributedCache` настраивается в `ConfigureServices` метода, она доступна для `Configure` методу в качестве параметра. Добавив его в качестве параметра позволит настроенный экземпляр должен предоставляться через DI.

## <a name="using-a-redis-distributed-cache"></a>С помощью Redis распределенного кэша

[Redis](https://redis.io/) -хранилище данных в памяти открытым исходным кодом, который часто используется в качестве распределенного кэша. Можно использовать локально, и можно настроить [кэш Azure Redis](https://azure.microsoft.com/services/cache/) для приложений ASP.NET Core, размещенной в Azure. Настраивает приложение ASP.NET Core реализации кэша с помощью `RedisDistributedCache` экземпляра.

Настройте реализацию Redis в `ConfigureServices` и откройте его в коде приложения, запросив экземпляр `IDistributedCache` (см. приведенный выше код).

В образце кода `RedisCache` реализация используется в том случае, если сервер настроен для `Staging` среды. Таким образом `ConfigureStagingServices` настраивает метод `RedisCache`:

[!code-csharp[](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=8,9,10,11,12,13&range=27-40)]

> [!NOTE]
> Чтобы установить Redis на локальном компьютере, необходимо установить пакет chocolatey [ https://chocolatey.org/packages/redis-64/ ](https://chocolatey.org/packages/redis-64/) и запустите `redis-server` из командной строки.

## <a name="using-a-sql-server-distributed-cache"></a>Использование SQL Server распределенного кэша

Реализация SqlServerCache допускает распределенного кэша для использования базы данных SQL Server в качестве резервного хранилища. Создание SQL Server таблицы, можно использовать средство кэша sql, средство создает таблицу с именем и схемой, которые вы задаете.

Чтобы использовать средство кэша sql, добавьте `SqlConfig.Tools` для `<ItemGroup>` элемент *.csproj* файл и запустите восстановление dotnet.

[!code-xml[](./distributed/sample/src/DistCacheSample/DistCacheSample.csproj?range=23-25)]

Протестируйте SqlConfig.Tools, выполнив следующую команду

```none
C:\DistCacheSample\src\DistCacheSample>dotnet sql-cache create --help
   ```

Средство кэша SQL для отображения справки об использовании, параметры и команды, теперь можно создавать таблицы в sql server, выполнив команду «Создать кэш sql»:

```none
C:\DistCacheSample\src\DistCacheSample>dotnet sql-cache create "Data Source=(localdb)\v11.0;Initial Catalog=DistCache;Integrated Security=True;" dbo TestCache
   info: Microsoft.Extensions.Caching.SqlConfig.Tools.Program[0]
       Table and index were created successfully.
   ```

Созданный таблица имеет следующую схему:

![Таблица кэша SqlServer](distributed/_static/SqlServerCacheTable.png)

Как и все реализации кэша приложения следует получать и задавать значения кэша, используя экземпляр `IDistributedCache`, а не `SqlServerCache`. В этом образце реализуется `SqlServerCache` в `Production` среды (поэтому оно настроено в `ConfigureProductionServices`).

[!code-csharp[](./distributed/sample/src/DistCacheSample/Startup.cs?highlight=7,8,9,10,11,12&range=42-56)]

> [!NOTE]
> `ConnectionString` (И при необходимости `SchemaName` и `TableName`) обычно должны храниться вне системы управления версиями (например, объектов UserSecrets), так как они могут содержать учетные данные.

## <a name="recommendations"></a>Рекомендации

При выборе реализация `IDistributedCache` для приложения, выбор между Redis и SQL Server, основанное на существующей инфраструктуре и среду, требований к производительности и качества работы команды. Если команда является более удобно работать с Redis, это лучший выбор. Если предпочитает ваша команда SQL Server, можно быть уверенным, что реализация также. Обратите внимание, что традиционные решения кэширования сохраняет данные в памяти, что позволяет для быстрого получения данных. Необходимо хранить в кэше часто используемых данных и хранить все данные в постоянном хранилище серверной части, например SQL Server или хранилища Azure. Кэш redis — это кэширования решение, которое обеспечивает высокую пропускную способность и низкую задержку по сравнению с кэша SQL.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Кэш Azure redis](https://azure.microsoft.com/documentation/services/redis-cache/)
* [База данных SQL в Azure](https://azure.microsoft.com/documentation/services/sql-database/)
* [Кэш в памяти](xref:performance/caching/memory)
* [Обнаружение изменений с помощью маркеров изменений](xref:fundamentals/primitives/change-tokens)
* [Кэширование ответов](xref:performance/caching/response)
* [ПО промежуточного слоя для кэширования ответов](xref:performance/caching/middleware)
* [Вспомогательная функция тега кэша](xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper)
* [Вспомогательная функция тега распределенного кэша](xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper)
