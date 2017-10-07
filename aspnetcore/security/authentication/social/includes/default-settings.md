**Примечание:** вызов `AddIdentity` настраивает параметры схемы по умолчанию. `AddAuthentication(string defaultScheme)` Перегрузки наборы `DefaultScheme` свойство; и `AddAuthentication(Action<AuthenticationOptions> configureOptions)` перегрузка устанавливает только явно задать свойства. Каждая из перегрузок следует вызывать только один раз при добавлении нескольких поставщиков проверки подлинности. Все ранее настроенные переопределения могут быть последующие вызовы [AuthenticationOptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.authenticationoptions) свойства.