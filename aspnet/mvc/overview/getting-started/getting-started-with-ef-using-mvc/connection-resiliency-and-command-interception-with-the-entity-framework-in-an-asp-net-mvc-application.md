---
uid: mvc/overview/getting-started/getting-started-with-ef-using-mvc/connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application
title: Устойчивость подключения и команды перехвата с платформой Entity Framework в приложении ASP.NET MVC | Документы Microsoft
author: tdykstra
description: Contoso университета примера веб-приложения показано, как создавать приложения ASP.NET MVC 5 с помощью Entity Framework 6 Code First и Visual Studio...
ms.author: aspnetcontent
manager: wpickett
ms.date: 01/13/2015
ms.topic: article
ms.assetid: c89d809f-6c65-4425-a3fa-c9f6e8ac89f2
ms.technology: dotnet-mvc
ms.prod: .net-framework
msc.legacyurl: /mvc/overview/getting-started/getting-started-with-ef-using-mvc/connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application
msc.type: authoredcontent
ms.openlocfilehash: 4a43a9120bf3fa69b00b234d65d0f59d3ce9975b
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
<a name="connection-resiliency-and-command-interception-with-the-entity-framework-in-an-aspnet-mvc-application"></a>Устойчивость подключения и команды перехвата с платформой Entity Framework в приложении ASP.NET MVC
====================
по [Tom Dykstra](https://github.com/tdykstra)

[Загрузка завершенного проекта](http://code.msdn.microsoft.com/ASPNET-MVC-Application-b01a9fe8) или [скачать PDF](http://download.microsoft.com/download/0/F/B/0FBFAA46-2BFD-478F-8E56-7BF3C672DF9D/Getting%20Started%20with%20Entity%20Framework%206%20Code%20First%20using%20MVC%205.pdf)

> Contoso университета примера веб-приложения показано, как создавать приложения ASP.NET MVC 5 с помощью Entity Framework 6 Code First и Visual Studio 2013. Сведения о серии руководств см. в [первом руководстве серии](creating-an-entity-framework-data-model-for-an-asp-net-mvc-application.md).


Пока приложение работает локально в IIS Express на компьютере разработчика. Чтобы сделать доступными для других пользователей в Интернете реальному приложению, необходимо развернуть его на веб-поставщик услуг размещения и необходимо развернуть базу данных на сервере базы данных.

В этом учебнике вы узнаете, как использовать две функции Entity Framework 6, которые особенно важны при развертывании облачной среде: устойчивость подключений (автоматических повторных попыток для временных ошибок) и перехвата команды (перехватывать все запросы SQL отправляемые в базу данных для входа или изменения их).

Этот учебник подключения устойчивость и команда перехвата является необязательным. Если пропустить этот учебник, несколько отдельных изменений будет выполняться в последующих учебники.

## <a name="enable-connection-resiliency"></a>Включить устойчивость подключений

При развертывании приложения в Windows Azure, используется для развертывания базы данных для базы данных SQL Azure Windows, облачная служба баз данных. При подключении базы данных в облачную службу, чем при веб-сервер и сервер базы данных непосредственно соединены вместе в одном центре обработки данных, временных ошибок соединения обычно более часто. Даже если облако веб-сервера и облачная служба баз данных размещаются в одном центре обработки данных, существуют дополнительные сетевые подключения между ними, которые могут иметь проблемы, такие как подсистемы балансировки нагрузки.

Также облачной службы обычно совместно используется другими пользователями, что означает, что отклик может повлиять на их. И доступ к базе данных может контролироваться регулирования. Регулирование означает службу базы данных создает исключения, при попытке доступа к нему дополнительные часто, нежели в вашей службе соглашения об уровне (SLA).

Многие или большинство проблем с соединением при обращении к облачной службе являются временными, то есть они самоустраниться за короткий период времени. Поэтому при попытке выполнения операции базы данных получить тип ошибки, которая обычно временных, удалось повторите попытку после короткое время ожидания и операция может оказаться успешной. Можно обеспечить гораздо более широкие для пользователей, если обработки временных ошибок автоматически повторите попытку позднее, обеспечение невидимости большинство из них для клиента. Функция устойчивости подключений в Entity Framework 6 автоматизирует процесс повторного аварийное завершение SQL-запросов.

Функция устойчивости подключений должны быть настроены для конкретной базы данных службы соответствующим образом.

- Он должен знать, какие исключения могут быть временной. Необходимо повторить ошибки, вызванные временная потеря сетевого подключения, не ошибки, вызванные ошибками программы, например.
- Она должна ожидать соответствующий период времени между повторными попытками сбоя операции. Можно отложить больше между повторными попытками для пакетной обработки чем для документации веб-страницы, где пользователь ожидает ответа.
- Он должен повторить попытку на достаточное количество раз, прежде чем прекратить попытки. Может потребоваться повторить несколько раз во время пакетной обработки, как в интерактивном приложении.

Можно настроить эти параметры вручную для среды базы данных поддерживается поставщиком Entity Framework, но значения по умолчанию, которые обычно подходит для сети приложения, использующего базу данных SQL Windows Azure уже настроена, и Это и есть параметры, которые необходимо реализовать для приложения Contoso университета.

Необходимо выполнить, чтобы включить устойчивость подключений создать класс внутри сборки, производный от [DbConfiguration](https://msdn.microsoft.com/data/jj680699.aspx) класса и в этом классе задать базу данных SQL *стратегия выполнения*, в EF – еще один термин для *политика повторов*.

1. В папке DAL, добавьте файл класса с именем *SchoolConfiguration.cs*.
2. Замените код шаблона с помощью следующего кода:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample1.cs)]

    Entity Framework автоматически выполняет код в класс, производный от `DbConfiguration`. Можно использовать `DbConfiguration` классе для выполнения задач настройки в коде, в противном случае это делается в *Web.config* файла. Дополнительные сведения см. в разделе [EntityFramework конфигурации на основе кода](https://msdn.microsoft.com/data/jj680699).
3. В *StudentController.cs*, добавьте `using` инструкции для `System.Data.Entity.Infrastructure`.

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample2.cs)]
4. Изменить все `catch` блокирует перехвата `DataException` исключения, чтобы они перехватывают `RetryLimitExceededException` исключения вместо него. Пример:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample3.cs?highlight=1)]

    Вы использовали `DataException` выявления ошибок, которые может быть временной, чтобы обеспечить понятное сообщение «повторить». Но теперь, когда вы включили политику повтора, только ошибки, скорее всего, будут временными будет уже были испытаны и не удалось выполнить несколько раз, и фактического возвращенное исключение будет вставлено в `RetryLimitExceededException` исключение.

Дополнительные сведения см. в разделе [устойчивость подключений Entity Framework и логика повторных попыток](https://msdn.microsoft.com/data/dn456835).

## <a name="enable-command-interception"></a>Включение перехвата команды

Теперь, когда вы включили политику повтора, как вы тестируете для проверки, что он работает как ожидалось? Это не так просто принудительно временная ошибка происходит, особенно когда у вас локально, и он будет особенно трудно интегрировать фактические временные ошибки в автоматических модульных тестов. Чтобы протестировать функция устойчивости подключений, вам необходим способ перехватывать запросы, которые отправляет Entity Framework для SQL Server и замените тип исключения, который обычно временных ответа SQL Server.

Перехват запросов также можно использовать для реализации рекомендации для облачных приложений: [входа задержки и успешность всех вызовов к внешним службам](../../../../aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry.md#log) такие как базы данных службы. Предоставляет EF6 [выделенной API ведения журнала](https://msdn.microsoft.com/data/dn469464) , можно упростить его делать ведения журнала, но в этом разделе учебника вы научитесь использовать Entity Framework [возможность перехвата](https://msdn.microsoft.com/data/dn469464) напрямую, как в Ведение журнала и для имитации временных ошибок.

### <a name="create-a-logging-interface-and-class"></a>Создание класса и интерфейса записи журнала

Объект [рекомендация для ведения журнала](../../../../aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry.md#log) — это с помощью интерфейса вместо жестко запрограммированного вызовы методов System.Diagnostics.Trace или класс ведения журнала. Облегчает изменить позже на механизме регистрации в журнале, в дальнейшем для этого. Поэтому в этом разделе вы создадите интерфейсе ведения журнала и класса для реализации его/p > 

1. Создание папки в проект и назовите его *ведение журнала*.
2. В *ведение журнала* папки, создайте файл класса с именем *ILogger.cs*и замените код шаблона с помощью следующего кода:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample4.cs)]

    Этот интерфейс предоставляет три уровня трассировки, чтобы указать относительную важность журналы и разработанное для предоставления сведений о задержке для вызовов внешней службы, такие как запросы к базе данных. Методы ведения журнала имеют перегрузки, которые позволяют передавать исключение. Это, чтобы сведения об исключении, включая стека трассировки и внутренние исключения надежно регистрируется с помощью класса, который реализует интерфейс, не полагаясь на который выполняется при каждом вызове метода ведения журнала в приложении.

    Методы TraceApi позволяют отслеживать задержку каждого вызова внешней службы, такие как базы данных SQL.
3. В *ведение журнала* папки, создайте файл класса с именем *Logger.cs*и замените код шаблона с помощью следующего кода:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample5.cs)]

    Реализация использует System.Diagnostics для трассировки. Это встроенная функция .NET, что упрощает создание и использование сведений трассировки. Существует много «прослушиватели» можно использовать с трассировкой System.Diagnostics записывать файлы, например, журналы или записывать их в хранилище больших двоичных объектов в Azure. Некоторые параметры и ссылки на другие ресурсы для получения дополнительной информации см. в [Устранение неполадок веб-сайтов Azure в Visual Studio](https://docs.microsoft.com/azure/app-service-web/web-sites-dotnet-troubleshoot-visual-studio). Для этого учебника, только рассмотрим журналов в Visual Studio **вывода** окна.

    В реальном приложении может потребоваться учитывать пакеты трассировки, отличный от System.Diagnostics и ILogger интерфейс позволяет относительно просто перейдите к механизм различных трассировки, если вы решите сделать это.

### <a name="create-interceptor-classes"></a>Создайте классы перехватчик

Затем вы создадите классы, которые платформа Entity Framework будет вызывать каждый раз, он будет отправлять запрос в базу данных, один для имитации временных ошибок и один для ведения журнала выполнения. Эти классы перехватчик должен быть производным от `DbCommandInterceptor` класса. В них запись переопределения методов, которые вызываются автоматически при выполнения запроса. В этих методах проверки или запрос, который отправляется в базу данных журнала, и можно изменить запрос перед отправкой в базу данных или возвращать результат в Entity Framework самостоятельно без даже передача запроса к базе данных.

1. Чтобы создать класс перехватчика, в журнал каждые SQL-запроса, отправляемые в базу данных, создайте файл класса с именем *SchoolInterceptorLogging.cs* в *DAL* папки и замените код шаблона с Следующий код:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample6.cs)]

    Для успешных запросов или команд этот код записывает журнал сведений с сведения о задержке. Для исключения он создает журнал ошибок.
2. Создание класса перехватчик, который создаст пустой временных ошибок при вводе «Throw» в **поиска** создайте файл класса с именем *SchoolInterceptorTransientErrors.cs* в *DAL* папки и замените код шаблона с помощью следующего кода:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample7.cs)]

    Этот код только переопределения `ReaderExecuting` метод, который вызывается для запросов, которые могут возвращать несколько строк данных. Если вы хотите проверить устойчивость подключений для других типов запросов, можно также переопределить `NonQueryExecuting` и `ScalarExecuting` методов, качестве перехватчик ведения журнала не.

    При запуске страницы студентов и введите «Исключение» в качестве строки поиска, этот код создает пустой исключение базы данных SQL для номер ошибки 20, типа, известного как обычно временной. Другие коды ошибок, в настоящее время распознается в качестве временного являются 64, 233, 10053, 10054, 10060, 10928, 10929, 40197, 40501 и 40613, но они могут быть изменены в новые версии базы данных SQL.

    Код возвращает исключение в Entity Framework, вместо выполнения запроса и результатов запроса обратной передачи. Временное исключение возвращается четыре раза, после чего код принимает обычные процедуры передачи запроса к базе данных.

    Так как все, что зарегистрирован, можно будет видеть, что Entity Framework пытается выполнить запрос предшествовало в четыре раза, и единственное различие в приложении заключается в том, что занимает больше времени для визуализации страницы с результатами запроса.

    Сколько раз повторит попытку Entity Framework является настраиваемым; код определяет четыре раза, так как это значение по умолчанию для политики выполнения базы данных SQL. Если изменить политику выполнения, было также изменение здесь код, который указывает, сколько раз создаются временные ошибки. Можно также изменить код для создания исключения, чтобы Entity Framework вызовет `RetryLimitExceededException` исключение.

    Значение, введите в поле поиска будет находиться в `command.Parameters[0]` и `command.Parameters[1]` (он используется имя и фамилия). Если найдено значение «% вызвал исключение %», «Throw» заменяется в эти параметры «», чтобы одни ученики будет найден и возвращен.

    Это просто удобный способ тестирования устойчивость подключений, основываясь на изменяющихся некоторые входные данные для пользовательского интерфейса приложения. Также можно написать код, создаваемый временных ошибок для всех запросов или обновлений, как описано далее в комментарии о *DbInterception.Add* метод.
3. В *Global.asax*, добавьте следующий `using` инструкции:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample8.cs)]
4. Добавьте выделенные строки к `Application_Start` метод:

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample9.cs?highlight=7-8)]

    Следующие строки кода находятся в каком случае перехватчик кода для запуска при Entity Framework отправляет запросы к базе данных. Обратите внимание, что поскольку вы создали перехватчик отдельные классы для имитации временной ошибки и ведения журнала, вы можете независимо включить или отключить их.

    Можно добавить с помощью перехватчики `DbInterception.Add` метод в любом месте в коде; он не должен быть в `Application_Start` метод. Другой вариант — поместите этот код в классе DbConfiguration, которое было создано ранее, чтобы настроить политику выполнения.

    [!code-csharp[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample10.cs?highlight=6-7)]

    Везде, где поместите этот код следует избегать использования для выполнения `DbInterception.Add` для того же перехватчик более чем один раз, или получите дополнительные перехватчик экземпляров. Например если дважды добавить перехватчик ведения журнала, вы увидите два журнала для каждого запроса SQL.

    Перехватчики выполняются в порядке регистрации (порядок, в котором `DbInterception.Add` метод вызывается). Порядок может играть важную роль в зависимости от выполняемых задач в перехватчик. Например, перехватчик может изменить команду SQL, получаемый в `CommandText` свойство. Если это приводит к изменению команду SQL, далее перехватчик получит измененные команды SQL, а не исходной команды SQL.

    В результате которого позволяет вызвать временные ошибки, указав другое значение в пользовательском Интерфейсе написал код имитации временной ошибки. Кроме того можно написать код перехватчик всегда создать последовательность временных исключений без проверки значение конкретного параметра. Затем можно добавить перехватчик только в том случае, если вы хотите создавать временные ошибки. Если это сделать, тем не менее, не добавляйте перехватчик до после завершения инициализации базы данных. Другими словами выполните операцию по крайней мере одна база данных, например запрос на одном из наборов сущностей, перед началом создания временных ошибок. Платформа Entity Framework выполняет несколько запросов во время инициализации базы данных, и они не выполняются в транзакции, в случае ошибки во время инициализации может возникнуть контекста в несогласованном состоянии.

## <a name="test-logging-and-connection-resiliency"></a>Ведение журнала и подключение устойчивости теста

1. Нажмите клавишу F5, чтобы запустить приложение в режиме отладки и нажмите кнопку **учащихся** вкладки.
2. Посмотрите на Visual Studio **вывода** окна, чтобы отобразить выходные данные трассировки. Может потребоваться прокрутка вверх за некоторые ошибки JavaScript, чтобы получить журналы, записываемые в средство.

    Обратите внимание, что отображаются фактические SQL-запросы, отправляемые в базу данных. Вы увидите некоторые начальных запросов и команд, чтобы приступить к работе, выполняет Entity Framework, проверку версии базы данных и таблицы журнала миграции (вы узнаете о миграции в следующем уроке). Появится запрос для разбиения на страницы, чтобы узнать, сколько учащихся, и наконец видеть запрос, который получает данные студента.

    ![Ведение журнала для обычных запросов](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/_static/image1.png)
3. В **учащихся** , введите «Исключение» в качестве строки поиска и нажмите кнопку **поиска**.

    ![Строка поиска throw](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/_static/image2.png)

    Вы заметите, что браузер видимому зависания несколько секунд, пока Entity Framework повторением запроса несколько раз. Первый Повтор происходит очень быстро, затем ожидать увеличения перед каждой следующей попытки. Процесс больше ожидания перед каждой попыткой повтора называется *экспоненциально растущим*.

    При отображении страницы отображение студентов, имеющих «значение» в именах, посмотрите на окно вывода, и вы увидите тот же запрос была предпринята попытка пять раз, сначала четыре раза возвращением временных исключений. Для каждой временной ошибки будет отображен журнала, создаваемого при формировании временная ошибка в `SchoolInterceptorTransientErrors` класса («возвращение временная ошибка команды...») и вы увидите при записанные в журнал `SchoolInterceptorLogging` возвращает исключение.

    ![Выходные данные журнала, показывающая повторных попыток](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/_static/image3.png)

    Поскольку вы ввели строку поиска, параметризован запрос, возвращающий данные студента:

    [!code-sql[Main](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/samples/sample11.sql)]

    Не заносимых в журнал значения параметров, но это можно сделать. Если вы хотите увидеть значения параметра, можно написать код ведения журнала для получения значений параметров из `Parameters` свойство `DbCommand` объекта, который вы получаете в методы перехватчика.

    Обратите внимание, невозможно повторить этот тест, если не остановить приложение и перезапустите его. Если вы хотите иметь возможность проверить устойчивость подключений, несколько раз в рамках одного запуска приложения, можно написать код, чтобы сбросить счетчик ошибок в `SchoolInterceptorTransientErrors`.
4. Чтобы увидеть разницу стратегия выполнения (политику повторов) преобразует, комментарий `SetExecutionStrategy` строку в *SchoolConfiguration.cs*, снова запустить страницу студентов в режиме отладки и поиск «Throw».

    Это время отладчик останавливается на первой порожденное исключение немедленно при попытке выполнения запроса в первый раз.

    ![Фиктивный исключения](connection-resiliency-and-command-interception-with-the-entity-framework-in-an-asp-net-mvc-application/_static/image4.png)
5. Раскомментируйте *SetExecutionStrategy* строку в *SchoolConfiguration.cs*.

## <a name="summary"></a>Сводка

В этом учебнике вы знаете, как включить устойчивость подключений и журнала команды SQL, Entity Framework составляет и отправляет в базу данных. В следующем уроке вы развернете приложение к Интернету, с помощью Code First Migrations развертывания базы данных.

Оставьте отзыв на том, как вам понравилось этого учебника и что можно улучшить. Можно также запросить новые разделы на [показать мне как с код](http://aspnet.uservoice.com/forums/228522-show-me-how-with-code).

Ссылки на другие ресурсы Entity Framework можно найти в [доступа к данным ASP.NET - рекомендуется использовать ресурсы](../../../../whitepapers/aspnet-data-access-content-map.md).

> [!div class="step-by-step"]
> [Назад](sorting-filtering-and-paging-with-the-entity-framework-in-an-asp-net-mvc-application.md)
> [Вперед](migrations-and-deployment-with-the-entity-framework-in-an-asp-net-mvc-application.md)
