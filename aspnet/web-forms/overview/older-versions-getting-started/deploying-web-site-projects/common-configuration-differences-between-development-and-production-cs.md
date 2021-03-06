---
uid: web-forms/overview/older-versions-getting-started/deploying-web-site-projects/common-configuration-differences-between-development-and-production-cs
title: Распространенные конфигурации различия между разработки и эксплуатации (C#) | Документы Microsoft
author: rick-anderson
description: В предыдущих учебниках мы развернули наш веб-сайт, скопировав все необходимые файлы из среды разработки в рабочую среду. Тем не менее я...
ms.author: aspnetcontent
manager: wpickett
ms.date: 04/01/2009
ms.topic: article
ms.assetid: 721a5c37-7e21-48e0-832e-535c6351dcae
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/older-versions-getting-started/deploying-web-site-projects/common-configuration-differences-between-development-and-production-cs
msc.type: authoredcontent
ms.openlocfilehash: 2694e0dba774a5bca13b9acc6b14c3e47226a064
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
<a name="common-configuration-differences-between-development-and-production-c"></a>Распространенные конфигурации различия между разработки и эксплуатации (C#)
====================
по [Скотт Митчелл](https://twitter.com/ScottOnWriting)

[Скачать PDF](http://download.microsoft.com/download/E/8/9/E8920AE6-D441-41A7-8A77-9EF8FF970D8B/aspnet_tutorial05_ConfigDifferences_cs.pdf)

> В предыдущих учебниках мы развернули наш веб-сайт, скопировав все необходимые файлы из среды разработки в рабочую среду. Тем не менее не часто бывает конфигурации различия между средами, которые требуются, каждая среда имеет уникальный файл Web.config. Этот учебник проверяет типичная конфигурация различия и рассматривает стратегии обеспечения сведения о конфигурации отдельных.


## <a name="introduction"></a>Вступление


Последние два учебники рассмотрения развертывание простой веб-приложения. [ *Развертыванию сайта с помощью FTP-клиент* ](deploying-your-site-using-an-ftp-client-cs.md) учебника было показано, как использовать автономный клиент FTP для копирования необходимых файлов в среде разработки до рабочей. Предыдущего учебника [ *развертыванию ваш сайт с помощью Visual Studio*](deploying-your-site-using-visual-studio-cs.md), найденное в развертывании с помощью средства копирования веб-узла и параметр публикации Visual Studio. В обоих учебники каждого файла в рабочей среде был копию файла в среде разработки. Однако не нередко файлы конфигурации в рабочей среде, чтобы отличаться от показанных в среде разработки. Конфигурация веб-приложение хранится в `Web.config` файл и обычно включает сведения о внешних ресурсов, таких как базы данных, web и серверы электронной почты. Он также устанавливает поведение приложения в определенных ситуациях, например действие, предпринимаемое при возникновении необработанного исключения.

При развертывании веб-приложения важно, необходимая информация помещено в рабочей среде. В большинстве случаев `Web.config` не удается скопировать файл в интегрированной среде разработки в рабочую среду как-является. Вместо этого настроенную версию `Web.config` передаются в рабочей среде. Этот учебник кратко рассматриваются некоторые из наиболее распространенных различия в конфигурации; Он также содержатся некоторые методы для обслуживания различные сведения о конфигурации между средами.

## <a name="typical-configuration-differences-between-the-development-and-production-environments"></a>Типичная конфигурация различия между разработкой и рабочих средах

`Web.config` Файл, содержащий набор параметров конфигурации для приложения ASP.NET. Некоторые из этих сведений конфигурации одинаково независимо от среды. Например, параметры проверки подлинности и правила авторизации URL-адреса указано в `Web.config` файла `<authentication>` и `<authorization>` обычно с такими элементами, независимо от среды. Однако другие сведения о конфигурации — например, сведения о внешних ресурсах - обычно отличается в зависимости от среды.

Строки подключения к базам данных — это отличный пример параметров конфигурации, который отличается в зависимости от среды. Когда веб-приложение взаимодействует с сервером базы данных, его необходимо сначала установить соединение, и которая достигается с помощью [строка подключения](http://www.connectionstrings.com/Articles/Show/what-is-a-connection-string). Хотя можно жестко закодировать строку подключения базы данных непосредственно в веб-страницы или код, который подключается к базе данных, рекомендуется разместить его `Web.config` [ `<connectionStrings>` элемент](https://msdn.microsoft.com/library/bf7sd233.aspx) , чтобы строки подключения данные находятся в одном месте централизованного. Зачастую другой базы данных используется во время разработки, чем используемая в производственной среде; Следовательно данные строки подключения должно быть уникальным для каждой среды.

> [!NOTE]
> Будущих учебниках исследовать развертывание приложений, управляемых данными, после чего начнем погружение в особенности как строки подключения базы данных хранятся в файле конфигурации.


Принцип разработки и рабочих средах отличается большей части. Веб-приложения в среде разработки создается, тестирования и отладки небольшой группой разработчиков. В рабочей среде посещении того же приложения с множеством разных одновременно работающих пользователей. ASP.NET содержит ряд функций, помогающих разработчикам при тестировании и отладке приложения, но эти функции должен быть отключен для соображений производительности и безопасности при работе в рабочей среде. Рассмотрим несколько такие параметры конфигурации.

### <a name="configuration-settings-that-impact-performance"></a>Параметры конфигурации, которые влияют на производительность

При посещении страницы ASP.NET в первый раз (или в первый раз после его изменения), необходимо преобразовать его декларативная разметка в класс, и этот класс должен быть скомпилирован. Если веб-приложение использует автоматическую компиляцию кода классов страницы должна быть скомпилирована, слишком. Можно настроить набор параметров компиляции через `Web.config` файла [ `<compilation>` элемент](https://msdn.microsoft.com/library/s10awwz0.aspx).

Атрибут debug является одним из наиболее важных атрибутов в `<compilation>` элемент. Если `debug` атрибута задано значение «true», а затем скомпилированных сборках содержатся символы отладки, которые нужны при отладке приложения в Visual Studio. Но символы отладки увеличивают размер сборки и наложить дополнительные требования к памяти при выполнении кода. Кроме того, когда `debug` атрибута задано значение «true» содержимого, возвращаемого `WebResource.axd` не кэшируются, то есть каждый раз пользователь открывает страницу, их необходимо повторно загрузить статического содержимого, возвращаемого `WebResource.axd`.

> [!NOTE]
> `WebResource.axd` встроенные обработчик HTTP появилась в ASP.NET 2.0, серверные элементы управления используют для извлечения внедренных ресурсов, таких как файлы скриптов, изображений, файлов CSS и другое содержимое. Дополнительные сведения о том, как `WebResource.axd` работает и как ее можно использовать для доступа к внедренных ресурсов из пользовательских серверных элементов управления, в разделе [доступ к внедренные ресурсы через URL-адрес с помощью `WebResource.axd` ](http://aspnet.4guysfromrolla.com/articles/080906-1.aspx).


`<compilation>` Элемента `debug` атрибут обычно имеет значение «true» в среде разработки. На самом деле этого атрибута должно быть присвоено «true» для отладки веб-приложения; При попытке отладить приложение ASP.NET из Visual Studio и `debug` атрибута задано значение «false», Visual Studio будет отображать сообщение о том, что приложение не может быть отлажен до `debug` атрибута задано значение «true» и будет предложение для внесения этого изменения для вас.

Вы должны **никогда не** имеют `debug` атрибута значение «true», в рабочей среде из-за его влияния на производительность. Более подробные сведения по этой теме см. в разделе [Скотт Гатри](https://weblogs.asp.net/scottgu/)в записи блога [не запустить производства ASP.NET приложений с `debug="true"` включено](https://weblogs.asp.net/scottgu/442448).

### <a name="custom-errors-and-tracing"></a>Пользовательских ошибок и трассировки

При возникновении необработанного исключения в приложении ASP.NET он поднимает во время выполнения, после чего происходит одно из трех действий:

- Отображается сообщение об ошибке универсального времени выполнения. Эта страница сообщает пользователю, что ошибка времени выполнения, но не предоставляет все сведения об ошибке.
- Отображается сообщение сведения об исключении, включающее сведения об исключении только что созданное исключение.
- Отображается собственную страницу ошибки, являющийся страницы ASP.NET, создаваемом сообщением желаемые.

Что происходит при возникновении необработанного исключения зависит от `Web.config` файла [ `<customErrors>` раздел](https://msdn.microsoft.com/library/h0hfz6fc.aspx).

Во время разработки и тестирования приложения полезно для просмотра подробных сведений об любое исключение в браузере. Отображение сведений об исключении в приложении в производстве то, потенциальную угрозу безопасности. Кроме того она unflattering и предоставляет веб-сайте выглядеть непрофессионально. В идеальном случае в случае необработанное исключение веб-приложения в среде разработки будут отображаться сведения об исключении, а одного приложения в рабочей среде покажет собственную страницу ошибки.

> [!NOTE]
> Значение по умолчанию `<customErrors>` Настройка раздел показывает сведения об исключении сообщения только в том случае, когда страница посещений через localhost и показана страница ошибки универсального среды выполнения, в противном случае. Это не является идеальным, но он обеспечение знать поведение по умолчанию не поможет найти сведения об исключении для нелокальных посетителей. Проверяет будущих учебника `<customErrors>` разделе более подробно и приводятся сведения по имеют собственную страницу ошибки отображается в случае возникновения ошибки в рабочей среде.


Другой функции ASP.NET, который используется во время разработки трассировки. Трассировки, если параметр включен, записывает сведения о каждого входящего запроса и предоставляет специальные веб-страницы, `Trace.axd`, для просмотра последних сведений о запросе. Можно включить и настроить трассировку через [ `<trace>` элемент](https://msdn.microsoft.com/library/6915t83k.aspx) в `Web.config`.

При включении трассировки убедитесь, что что он отключен в рабочей среде. Поскольку сведения о трассировке включает файлы cookie, данные о сеансе и других конфиденциальных данных, важно отключить трассировку в рабочей среде. Хорошо то, что по умолчанию трассировка отключена и `Trace.axd` файл доступен только через localhost. Если изменить эти параметры по умолчанию в разработке убедитесь, что они отключены обратно в рабочей среде.

## <a name="techniques-for-maintaining-separate-configuration-information"></a>Методы для хранения данных отдельной конфигурации

Наличие параметров конфигурации в средах разработки и эксплуатации усложняет процесс развертывания. В предыдущих учебниках два процесса развертывания включал в себя копирование все необходимые файлы из среды разработки в рабочей среде, но этот подход работает только если сведения о конфигурации совпадают в обеих средах. Существуют различные методы для развертывания приложения с различными данными конфигурации. Давайте каталога некоторые из этих параметров для размещенного веб-приложений.

### <a name="manually-deploying-the-production-environment-configuration-file"></a>Развертывание вручную файл конфигурации для рабочей среды

Самым простым подходом является поддержание две версии `Web.config` файла: один для среды разработки и один для рабочей среды. Развертывание узла в рабочей среде влечет за собой копирование всех файлов на рабочем сервере в среде разработки *за исключением* для `Web.config` файла. Вместо этого производства конкретной среды `Web.config` файл копируется в рабочей среде.

Этот подход не является очень сложным, но можно легко реализовать, так как сведения о конфигурации изменяются редко. Она лучше всего подходит для приложений с небольшой разработчиков, которые размещаются на одном веб-сервере и редко изменяется, сведения о конфигурации. Это наиболее tenable при ручном развертывании файлы приложения с помощью автономного клиента FTP. При использовании средства копирования веб-сайта или параметра публикации Visual Studio необходимо сначала поменять местами некоторые относящиеся к развертыванию `Web.config` файл с тем, производственные перед развертыванием и замены их обратно после завершения развертывания.

### <a name="change-the-configuration-during-the-build-or-deployment-process"></a>Изменение конфигурации во время построения или процесс развертывания

До сих обсуждений предполагается процесса построения и развертывания ad-hoc. Многие крупных проектов программного обеспечения более формализовали процессы, которые используют открытый исходный код, собственной разработки, или сторонние средства. Скорее всего, для таких проектов можно настроить процесс сборки или развертывания правильно изменить сведения о конфигурации, прежде чем он помещается в рабочей среде. При создании вашего веб-приложения с помощью [MSBuild](http://en.wikipedia.org/wiki/MSBuild), [NAnt](http://nant.sourceforge.net/), или другого средства построения, скорее всего добавить шаг построения, чтобы изменить `Web.config` файла следует включить параметры конкретного производства. Или рабочий процесс развертывания может программно подключиться сервера системы управления версиями и получения соответствующего `Web.config` файла.

Фактический подход для получения сведений о соответствующей конфигурации в рабочей среде широко различается в зависимости от средств и рабочего процесса. Таким образом мы не будет углубимся в этом разделе. При использовании сборки популярных средств, как MSBuild или NAnt можно найти статьи развертывания и учебники относящиеся к следующим средствам через Интернет-поиска.

### <a name="managing-configuration-differences-via-the-web-deployment-project-add-in"></a>Различия в управлении конфигурации через веб-развертывания проекта надстройки

В 2006 г. Корпорация Майкрософт выпустила Web разработки проекта надстройки для Visual Studio 2005. Надстройка для Visual Studio 2008 был выпущен в 2008 г. Эта надстройка позволяет разработчикам создавать отдельный проект веб-развертывания вместе с их проект веб-приложения, при построении ASP.NET, явно компилирует веб-приложения и копирует файлы для развертывания на локальном выходной каталог. Проект веб-приложения использует MSBuild незаметно.

По умолчанию в среде разработки `Web.config` файл копируется в выходной каталог, но вы можете настроить проект веб-развертывания для настройки

сведения о конфигурации, который копируется в этот каталог одним из следующих способов:

- Через `Web.config` файла замены раздел, в котором указываются раздел, чтобы заменить и в XML-файл, содержащий текст для замены.
- Указав путь к внешнего источника файла конфигурации. Этот параметр выбран, проект веб-развертывания копирует конкретного `Web.config` файл в выходной каталог (а не `Web.config` файл, используемый в среде разработки).
- Добавление файла MSBuild, используемые веб-приложением развертывания настраиваемые правила.

Для развертывания веб-сайте приложения создать проект веб-развертывания и затем скопируйте файлы из выходной папки проекта в производственной среде.

Чтобы узнать дополнительные сведения об использовании веб-развертывания проекта извлечь [в этой статье веб-развертывания проектов](https://msdn.microsoft.com/magazine/cc163448.aspx) выпуска за апреля 2007 г. [MSDN Magazine](https://msdn.microsoft.com/magazine/default.aspx), или обратитесь к ссылки в разделе "Дополнительные материалы" в Завершение работы с этим учебником.

> [!NOTE]
> Проект веб-развертывания нельзя использовать с Visual Web Developer, так как проект веб-развертывания реализуется как Visual Studio надстройки и выпуски Visual Studio Express (включая Visual Web Developer) не поддерживают надстройки.


## <a name="summary"></a>Сводка

Внешние ресурсы и поведение веб-приложения в разработке обычно отличаются от, когда того же приложения в рабочей среде. Например строки подключения базы данных, параметры компиляции и поведение, когда происходит необработанное исключение, обычно отличаются между средами. Процесс развертывания должны включать эти различия. Как уже говорилось в этом учебнике, самым простым подходом является вручную скопировать файл альтернативную конфигурацию в рабочей среде. Возможны более элегантные решения при использовании надстройки Web развертывания проекта или с более документально оформленными принципами процесса сборки или развертывания, способное таких настроек.

Программирование довольны!

### <a name="further-reading"></a>Дополнительные сведения

Дополнительные сведения по темам, рассматриваемые в этом учебнике см. в следующих ресурсах:

- [Объясняются строки подключения](http://www.connectionstrings.com/Articles/Show/what-is-a-connection-string)
- [@ ConnectionStrings.com строки подключения базы данных](http://www.connectionstrings.com/)
- [Не запускать приложений ASP.NET с `debug="true"` включена](https://weblogs.asp.net/scottgu/Don_1920_t-run-production-ASP.NET-Applications-with-debug_3D001D20_true_1D20_-enabled)
- [Правильно отвечать на необработанные исключения - отображение понятных ошибки страниц](http://aspnet.4guysfromrolla.com/articles/090606-1.aspx)
- [Инструкции: использование Visual Studio 2008 веб-развертывания проекта?](../../../videos/how-do-i/how-do-i-use-a-visual-studio-2008-web-deployment-project.md)
- [Параметры ключа конфигурации при развертывании базы данных](http://aspnet.4guysfromrolla.com/articles/121008-1.aspx)
- [Загрузки проекты развертывания Visual Studio 2008](https://www.microsoft.com/downloads/details.aspx?FamilyId=0AA30AE8-C73B-4BDD-BB1B-FE697256C459&amp;displaylang=en) | [загрузки проекты развертывания Visual Studio 2005](https://download.microsoft.com/download/9/4/9/9496adc4-574e-4043-bb70-bc841e27f13c/WebDeploymentSetup.msi)
- [VS 2008 веб-развертывания проектов](https://weblogs.asp.net/scottgu/archive/2005/11/06/429723.aspx) | [поддержка VS 2008 веб-развертывания проектов с выпуска](https://weblogs.asp.net/scottgu/archive/2008/01/28/vs-2008-web-deployment-project-support-released.aspx)
- [Проекты веб-развертываний](https://msdn.microsoft.com/magazine/cc163448.aspx)

> [!div class="step-by-step"]
> [Назад](deploying-your-site-using-visual-studio-cs.md)
> [Вперед](core-differences-between-iis-and-the-asp-net-development-server-cs.md)
