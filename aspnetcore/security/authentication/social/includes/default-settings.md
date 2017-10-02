**Hinweis:** der Aufruf von `AddIdentity` konfiguriert die Standardeinstellungen für das Schema. Die `AddAuthentication(string defaultScheme)` überladen, legt der `DefaultScheme` Eigenschafts- und `AddAuthentication(Action<AuthenticationOptions> configureOptions)` Überladung wird nur die Eigenschaften, die Sie explizit festlegen. Beide dieser Überladungen sollten nur einmal beim Hinzufügen mehrerer Authentifizierungsanbieter aufgerufen werden. Nachfolgende Aufrufe haben das volle Potenzial von überschreiben alle zuvor konfigurierten [authenticationoptions](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.authenticationoptions) Eigenschaften.