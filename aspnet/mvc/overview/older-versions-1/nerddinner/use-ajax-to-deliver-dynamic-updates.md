---
uid: mvc/overview/older-versions-1/nerddinner/use-ajax-to-deliver-dynamic-updates
title: Использование AJAX для динамического обновления | Документы Microsoft
author: microsoft
description: Шаг 10 реализует поддержку вошедшего в систему пользователям RSVP их интересуют посещение dinner, использование подхода на основе Ajax интегрировано в подробности dinner...
ms.author: aspnetcontent
manager: wpickett
ms.date: 07/27/2010
ms.topic: article
ms.assetid: 18700815-8e6c-4489-91af-7ea9dab6529e
ms.technology: dotnet-mvc
ms.prod: .net-framework
msc.legacyurl: /mvc/overview/older-versions-1/nerddinner/use-ajax-to-deliver-dynamic-updates
msc.type: authoredcontent
ms.openlocfilehash: 7cea3ee2ec52261521941efac484e91a53f6310b
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2018
---
<a name="use-ajax-to-deliver-dynamic-updates"></a>Использование AJAX для динамического обновления.
====================
по [Microsoft](https://github.com/microsoft)

[Скачать PDF](http://aspnetmvcbook.s3.amazonaws.com/aspnetmvc-nerdinner_v1.pdf)

> Это шаг 10 бесплатных [учебника «Обновление NerdDinner» приложения](introducing-the-nerddinner-tutorial.md) , обходов сквозной как построить небольшой, но завершается веб-приложения с помощью ASP.NET MVC 1.
> 
> Шаг 10 реализует поддержку вошедшего в систему пользователям RSVP их интересуют посещение dinner, использование подхода на основе Ajax интегрировано в странице сведений о компании dinner.
> 
> При использовании ASP.NET MVC 3, которому рекомендуется следовать [Приступая к работе с MVC 3](../../older-versions/getting-started-with-aspnet-mvc3/cs/intro-to-aspnet-mvc-3.md) или [MVC Music Store](../../older-versions/mvc-music-store/mvc-music-store-part-1.md) учебники.


## <a name="nerddinner-step-10-ajax-enabling-rsvps-accepts"></a>Обновление NerdDinner шаг 10: Принимает AJAX Включение ответов на приглашение

Теперь рассмотрим процедуру внедрения поддержка RSVP их интересуют посещение dinner вошедших пользователей. Мы будем обеспечить ее использование подхода на основе AJAX интегрировано в странице сведений о компании dinner.

### <a name="indicating-whether-the-user-is-rsvpd"></a>Указывающее, является ли RSVP'd пользователя

Могут посещать пользователи */Dinners/подробности / [идентификатор*] URL-адрес для просмотра сведений о конкретной dinner:

![](use-ajax-to-deliver-dynamic-updates/_static/image1.png)

Details(), метод действия реализуется следующим образом:

[!code-csharp[Main](use-ajax-to-deliver-dynamic-updates/samples/sample1.cs)]

Наш первый шаг для реализации поддержки RSVP будет добавить вспомогательный метод «IsUserRegistered(username)» объект нашей компании Dinner (внутри разделяемого класса Dinner.cs, созданному ранее). Этот вспомогательный метод возвращает значение true или false в зависимости от того, является ли пользователь в настоящее время RSVP'd для компании Dinner:

[!code-csharp[Main](use-ajax-to-deliver-dynamic-updates/samples/sample2.cs)]

Затем можно добавить следующий код в нашей шаблон представление Details.aspx отобразить соответствующее сообщение, указывающее, зарегистрирован ли пользователь или не для события:

[!code-html[Main](use-ajax-to-deliver-dynamic-updates/samples/sample3.html)]

И теперь при посещении Dinner, зарегистрированных для их увидите это сообщение:

![](use-ajax-to-deliver-dynamic-updates/_static/image2.png)

И при посещении Dinner, они не зарегистрированы для появится следующее сообщение:

![](use-ajax-to-deliver-dynamic-updates/_static/image3.png)

### <a name="implementing-the-register-action-method"></a>Реализация метода действия регистра

Теперь добавим функциональные возможности, необходимые, чтобы пользователи RSVP на обед на странице сведений о.

Чтобы реализовать это, мы создадим новый класс «RSVPController», щелкнув правой кнопкой мыши каталог \Controllers и выбрав команду Add -&gt;команда меню контроллера.

Реализуется метод действия «Register» в новый класс RSVPController принимает идентификатор на обед в качестве аргумента, получает соответствующий объект Dinner проверяет, если в данный момент в список пользователей, зарегистрированных для него вошедшего в систему пользователя и если не добавляет объект RSVP для них:

[!code-csharp[Main](use-ajax-to-deliver-dynamic-updates/samples/sample4.cs)]

Обратите внимание, выше способ возвращение простую строку в качестве выходных данных метода действия. Нам удалось внедренных в шаблоне представления — это сообщение, но так как оно слишком мало, поэтому мы просто используем вспомогательный метод Content() на базовый класс контроллера и возвращают строковое сообщение, например выше.

### <a name="calling-the-rsvpforevent-action-method-using-ajax"></a>Вызов метода действия RSVPForEvent с помощью AJAX

Мы будем использовать AJAX для вызова метода действия Register из наших подробного представления. Такая реализация довольно просто. Сначала мы добавим двух ссылок на библиотеки скриптов:

[!code-html[Main](use-ajax-to-deliver-dynamic-updates/samples/sample5.html)]

Первая библиотека ссылается основной библиотеки ASP.NET AJAX клиентского скрипта. Этот файл составляет приблизительно 24 КБ размер (сжатые данные) и содержит основные клиентские функции AJAX. Второй библиотеки содержатся служебные функции, которые интегрируются с ASP.NET MVC встроенных AJAX вспомогательные методы (которые мы будем использовать ближайшее время).

Мы можем затем вызова AJAX, вызывает метод действия нашей RSVPForEvent на нашем RSVP контроллер выполняет обновление просмотреть код шаблона, который мы добавили ранее, чтобы вместо outputing сообщение «Вы не зарегистрированы для данного события», мы вместо визуализации ссылки, при передаче и RSVPs пользователя:

[!code-aspx[Main](use-ajax-to-deliver-dynamic-updates/samples/sample6.aspx)]

Вспомогательный метод Ajax.ActionLink() выше является встроено в ASP.NET MVC и похож на вспомогательный метод Html.ActionLink(), за исключением того, вместо выполнения стандартных навигации он осуществляет вызов AJAX к методу действия при щелчке ссылки. Выше мы вызова метода действие «Зарегистрировать» на контроллере «RSVP» и передачи DinnerID как параметр «id» в. Последний параметр AjaxOptions, мы передаем указывает, что мы хотим использовать содержимого, возвращаемого из метода действия и обновите HTML &lt;div&gt; элемента на странице, идентификатор которого — «rsvpmsg».

А теперь когда пользователь переходит на обед они не зарегистрированы для, но они будут видеть ссылку RSVP для него:

![](use-ajax-to-deliver-dynamic-updates/_static/image4.png)

Если щелкнуть ссылку «RSVP для данного события» они сделать вызов AJAX на метод действия регистрации на контроллере RSVP и после ее завершения, они видят сообщение об обновленных, аналогичные показанным ниже:

![](use-ajax-to-deliver-dynamic-updates/_static/image5.png)

Пропускная способность сети и трафика при выполнении этого вызова AJAX действительно невелико. Когда пользователь щелкает ссылку «RSVP для данного события», небольшой сетевой запрос HTTP POST адресованных */Dinners/Register/1* URL-адрес, который выглядит как ниже по сети:

[!code-console[Main](use-ajax-to-deliver-dynamic-updates/samples/sample7.cmd)]

И ответ от наших метод действия регистрации — это просто:

[!code-console[Main](use-ajax-to-deliver-dynamic-updates/samples/sample8.cmd)]

Этот вызов упрощенных очень быстро и будет работать даже по медленному сетевому подключению.

### <a name="adding-a-jquery-animation"></a>Добавление jQuery анимации

Функциональные возможности AJAX, которые мы реализовали работает хорошо и быстро. Иногда это может случиться так быстро, что пользователь не заметить, ссылка RSVP заменено новым текстом. Чтобы сделать результат более очевидным, мы можем добавить простая анимация для привлечения внимания к сообщения обновления.

Значение по умолчанию шаблон проекта ASP.NET MVC включает jQuery — прекрасный (и популярным) открытой библиотека JavaScript, которая также поддерживается корпорацией Майкрософт. jQuery предоставляет ряд функций, включая выбор и эффектов библиотека HTML DOM работы с низким приоритетом.

Для использования jQuery мы сначала добавить ссылку на скрипт на него. Так как мы будем использовать jQuery в различных мест в сайте, мы добавим ссылку на скрипт в наш файл Site.master главной страницы, чтобы его можно было использовать все страницы.

[!code-html[Main](use-ajax-to-deliver-dynamic-updates/samples/sample9.html)]

*Совет: Убедитесь в том, что вы установили исправление JavaScript intellisense для VS 2008 с пакетом обновления 1, обеспечивает более широкую поддержку intellisense для JavaScript файлов (включая jQuery). Его можно загрузить из: http://tinyurl.com/vs2008javascripthotfix*

Код, написанный с помощью JQuery часто использует глобальный «$ ()» метод JavaScript, который получает один или несколько элементов HTML, с помощью селектора CSS. Например <em>$("#rsvpmsg")</em> выбирает любой HTML-элемент с идентификатором rsvpmsg, пока <em>$(".something")</em> выбрать все элементы с «текст» CSS имя класса. Можно также написать более сложные запросы, как «возвращать все отмеченные переключателей» с помощью селектора запрос наподобие: <em>$(«входных данных [@type= переключателей] [@checked]»)</em>.

После выбора элементов для возможность выполнения действий, таких как скрытие их можно вызывать методы: *$(«#rsvpmsg").hide();*

В нашем сценарии RSVP мы определим простую функцию JavaScript с именем «AnimateRSVPMessage», выбирающий «rsvpmsg» &lt;div&gt; и выполняет анимацию с размером его содержимого текста. Ниже кода запускает небольшой текст, а затем вызывает его со периодичностью 400 миллисекунд:

[!code-html[Main](use-ajax-to-deliver-dynamic-updates/samples/sample10.html)]

Мы можно затем проводной доступ к этой функции JavaScript, вызываемый после успешного завершения наш вызов AJAX, передав ее имя на нашем вспомогательный метод Ajax.ActionLink() (через AjaxOptions «OnSuccess» свойства события):

[!code-aspx[Main](use-ajax-to-deliver-dynamic-updates/samples/sample11.aspx)]

И теперь при нажатии ссылки «RSVP для этого события» и наши вызов AJAX успешного завершения содержимого сообщения, отправленного назад будет анимации и увеличению:

![](use-ajax-to-deliver-dynamic-updates/_static/image6.png)

Помимо события «OnSuccess», объект AjaxOptions предоставляет методы OnBegin, OnFailure и OnComplete события, которые можно обрабатывать (а также ряд других свойств и полезные параметры).

### <a name="cleanup---refactor-out-a-rsvp-partial-view"></a>Очистка - рефакторинга out RSVP частичного представления

Наш шаблон представления сведений начинает получить немного long, какие сверхурочной работы затруднит немного для понимания. Чтобы сделать код более понятным, добавим путем создания частичное представление — RSVPStatus.ascx —, инкапсулирующими все представления кода RSVP нашу страницу сведений.

Это можно сделать, щелкнув правой кнопкой мыши по папке \Views\Dinners, а затем выберите Add -&gt;просмотреть команды меню. У нас будет их в качестве ее ViewModel строго типизированный объект обед. Мы можно затем скопируйте и вставьте содержимое RSVP из наших представление Details.aspx, в него.

Если мы выполнили, также создадим другой частичного представления — EditAndDeleteLinks.ascx -, инкапсулирующий наш код представление связи Edit и Delete. Мы также получите его принимают в качестве ее ViewModel строго типизированный объект Dinner и скопируйте логики Edit и Delete из наших представление Details.aspx, в него.

Подробную информацию шаблона можно просмотреть, а затем просто содержат два вызова метода Html.RenderPartial() внизу:

[!code-aspx[Main](use-ajax-to-deliver-dynamic-updates/samples/sample12.aspx)]

Это делает код очистки для понимания и обслуживания.

### <a name="next-step"></a>Следующий шаг

Давайте теперь взглянем на как можно еще более используется AJAX и добавить поддержку интерактивного сопоставления для нашего приложения.

> [!div class="step-by-step"]
> [Назад](secure-applications-using-authentication-and-authorization.md)
> [Вперед](use-ajax-to-implement-mapping-scenarios.md)
