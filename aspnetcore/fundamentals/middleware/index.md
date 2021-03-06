---
title: ASP.NET Core-Middleware
author: rick-anderson
description: Erfahren Sie mehr über ASP.NET Core-Middleware und die Anforderungspipeline.
manager: wpickett
ms.author: riande
ms.date: 01/22/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: fundamentals/middleware/index
ms.openlocfilehash: 3312b27f936340a73243224c1a716fe421f178bc
ms.sourcegitcommit: 48beecfe749ddac52bc79aa3eb246a2dcdaa1862
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/22/2018
---
# <a name="aspnet-core-middleware"></a>ASP.NET Core-Middleware

Von [Rick Anderson](https://twitter.com/RickAndMSFT) und [Steve Smith](https://ardalis.com/)

[Anzeigen oder Herunterladen von Beispielcode](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/middleware/index/sample) ([Vorgehensweise zum Herunterladen](xref:tutorials/index#how-to-download-a-sample))

## <a name="what-is-middleware"></a>Was ist Middleware?

Middleware ist Software, die zu einer Anwendungspipeline zusammengesetzt wird, um Anforderungen und Antworten zu verarbeiten. Jede Komponente kann Folgendes ausführen:

* Entscheiden, ob die Anforderung an die nächste Komponente in der Pipeline übergeben werden soll.
* Arbeiten, bevor oder nachdem die nächste Komponente in der Pipeline aufgerufen wird. 

Anforderungsdelegaten werden verwendet, um die Anforderungspipeline zu erstellen. Die Anforderungsdelegaten behandeln jede HTTP-Anforderung.

Sie werden mit [Run](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.runextensions)-, [Map](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mapextensions)- und [Use](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.useextensions)-Erweiterungsmethoden konfiguriert. Ein einzelner Anforderungsdelegat kann inline als anonyme Methode angegeben werden (sogenannte Inline-Middleware), oder er kann in einer wiederverwendbaren Klasse definiert werden. Diese wiederverwendbaren Klassen und anonymen Inline-Methoden sind *Middleware* oder *Middlewarekomponenten*. Jede Middlewarekomponente in der Anforderungspipeline ist für das Aufrufen der jeweils nächsten Komponente in der Pipeline oder, wenn nötig, für das Kurzschließen der Kette zuständig.

Unter [Migrieren von HTTP-Modulen zu Middleware](xref:migration/http-modules) wird der Unterschied zwischen Anforderungspipelines in ASP.NET Core und ASP.NET 4.x erklärt. Außerdem werden dort weitere Beispiele für Middleware gegeben.

## <a name="creating-a-middleware-pipeline-with-iapplicationbuilder"></a>Erstellen einer Middlewarepipeline mit IApplicationBuilder

Die ASP.NET Core-Anforderungspipeline besteht, wie in folgendem Diagramm veranschaulicht, aus einer Sequenz von Anforderungsdelegaten, die nacheinander aufgerufen werden (der Ausführungsthread wird durch die schwarzen Pfeile dargestellt):

![Anforderungsverarbeitungsmuster mit eingehender Anforderung, deren Verarbeitung von drei Middlewares und die ausgehende Antwort der Anwendung. Jede Middleware führt ihre Logik aus und übergibt die Anforderung an die nächste Middleware an der next()-Anweisung. Nachdem die Anforderung von der dritten Middleware verarbeitet wurde, wird sie wieder an die vorherigen Middlewares in umgekehrter Reihenfolge übergeben, nachdem die next()-Anweisungen erreicht wurden. Dann verlässt sie die Anwendung als Antwort an den Client.](index/_static/request-delegate-pipeline.png)

Jeder Delegat kann Vorgänge vor und nach dem nächsten Delegaten ausführen. Ein Delegat kann sich auch dagegen entscheiden, eine Anforderung an den nächsten Delegaten zu übergeben. Dies wird als Kurzschluss einer Anforderungspipeline bezeichnet. Das Kurzschließen ist oft sinnvoll, da es unnötige Arbeit verhindert. Die Middleware für statische Dateien kann z.B. eine Anforderung einer statischen Datei zurückgeben und den Rest der Pipeline kurzschließen. Die Ausnahmebehandlungsdelegaten müssen am Anfang der Pipeline aufgerufen werden, sodass sie Ausnahmen abfangen können, die zu einem späteren Zeitpunkt in der Pipeline ausgelöst werden.

Die einfachste mögliche ASP.NET Core-App enthält einen einzigen Anforderungsdelegaten, der alle Anforderungen verarbeitet. In diesem Fall ist keine tatsächliche Anforderungspipeline vorhanden. Stattdessen wird eine einzelne anonyme Funktion als Antwort auf jede HTTP-Anforderung aufgerufen.

[!code-csharp[](index/sample/Middleware/Startup.cs)]

Der erste [app.Run](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.runextensions)-Delegat beendet die Pipeline.

Mit [app.Use](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.useextensions) können Sie mehrere Anforderungedelegate miteinander verknüpfen. Der Parameter `next` steht für den nächsten Delegaten in der Pipeline. (Denken Sie daran, dass Sie die Pipeline kurzschließen können, indem Sie den Parameter *next* *nicht* aufrufen.) Normalerweise können Sie Aktionen sowohl vor als auch nach dem nächsten Delegaten durchführen. Dies wird in folgendem Beispiel veranschaulicht:

[!code-csharp[](index/sample/Chain/Startup.cs?name=snippet1)]

>[!WARNING]
> Rufen Sie `next.Invoke` nicht auf, nachdem die Antwort an den Client gesendet wurde. An `HttpResponse` vorgenommene Änderungen lösen nach dem Start der Antwort eine Ausnahme aus. Änderungen wie das Festlegen von Headern oder Statuscode usw. lösen beispielsweise eine Ausnahme aus. Wenn Sie nach dem Aufruf von `next` in den Antworttext schreiben, kann dies:
> - einen Protokollverstoß verursachen, wenn Sie z.B. mehr als das genannte `content-length`-Objekt schreiben.
> - Fehler im Textformat auslösen, wenn Sie z.B. eine HTML-Fußzeile in eine CSS-Datei schreiben.
>
> [HttpResponse.HasStarted](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.features.httpresponsefeature#Microsoft_AspNetCore_Http_Features_HttpResponseFeature_HasStarted) ist ein nützlicher Hinweis, der angibt, ob Header gesendet wurden und/oder ob in den Text geschrieben wurde.

## <a name="ordering"></a>Sortieren

Die Reihenfolge, in der Middlewarekomponenten in der `Configure`-Methode hinzugefügt werden, legt die Reihenfolge fest, in der sie bei Anforderungen aufgerufen werden. Bei Antworten gilt die umgekehrte Reihenfolge. Diese Reihenfolge trägt wesentlich zur Sicherheit, Leistung und Funktionalität bei.

Die Methode „Configure“ (siehe unten) fügt die folgenden Middlewarekomponenten hinzu:

1. Ausnahme-/Fehlerbehandlung
2. Statischer Dateiserver
3. Authentifizierung
4. MVC

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)


```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseExceptionHandler("/Home/Error"); // Call first to catch exceptions
                                            // thrown in the following middleware.

    app.UseStaticFiles();                   // Return static files and end pipeline.

    app.UseAuthentication();               // Authenticate before you access
                                           // secure resources.

    app.UseMvcWithDefaultRoute();          // Add MVC to the request pipeline.
}
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseExceptionHandler("/Home/Error"); // Call first to catch exceptions
                                            // thrown in the following middleware.

    app.UseStaticFiles();                   // Return static files and end pipeline.

    app.UseIdentity();                     // Authenticate before you access
                                           // secure resources.

    app.UseMvcWithDefaultRoute();          // Add MVC to the request pipeline.
}
```

-----------

Im obenstehenden Code ist `UseExceptionHandler` die erste zur Pipeline hinzugefügte Middlewarekomponente. Deshalb fängt sie alle Ausnahmen ab, die in späteren Aufrufen ausgelöst werden.

Die Middeware für statische Dateien wird am Anfang der Pipeline aufgerufen, damit sie Anforderungen und Kurzschlüsse verarbeiten kann, ohne dass die verbleibenden Komponenten durchlaufen werden müssen. Die Middleware für statische Dateien stellt **keine** Autorisierungsüberprüfungen bereit. Alle Dateien, die von ihr bearbeitet werden, Dateien unter *wwwroot* inbegriffen, sind öffentlich verfügbar. Unter [Arbeiten mit statischen Dateien](xref:fundamentals/static-files) erfahren Sie, wie Sie statische Dateien schützen können.

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)


Wenn die Anforderung nicht von der Middleware für statische Dateien verarbeitet wird, wird sie an die Identity-Middleware (`app.UseAuthentication`) übergeben, welche die Authentifizierung durchführt. Identity schließt keine unautorisierten Anforderungen kurz. Auch wenn Identity Anforderungen authentifiziert, erfolgt die Autorisierung (und Ablehnung) erst dann, wenn MVC eine spezifische Razor Page oder einen Controller und eine Aktion ausgewählt hat.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Wenn die Anforderung nicht von der Middleware für statische Dateien verarbeitet wird, wird sie an die Identity-Middleware (`app.UseIdentity`) übergeben, welche die Authentifizierung durchführt. Identity schließt keine unautorisierten Anforderungen kurz. Auch wenn Identity Anforderungen authentifiziert, erfolgt die Autorisierung (und Ablehnung) erst dann, wenn  MVC einen spezifischen Controller und eine Aktion ausgewählt hat.

-----------

Im folgenden Beispiel wird eine Middlewarereihenfolge veranschaulicht, bei der Anforderungen für statische Dateien von der Middleware für statische Dateien vor der Middleware für die Antwortkomprimierung verarbeitet werden. Statische Dateien werden durch diese Middlewarereihenfolge nicht komprimiert. Die MVC-Antworten von [UseMvcWithDefaultRoute](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mvcapplicationbuilderextensions#Microsoft_AspNetCore_Builder_MvcApplicationBuilderExtensions_UseMvcWithDefaultRoute_Microsoft_AspNetCore_Builder_IApplicationBuilder_) können komprimiert werden.

```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseStaticFiles();         // Static files not compressed
                                  // by middleware.
    app.UseResponseCompression();
    app.UseMvcWithDefaultRoute();
}
```

<a name="middleware-run-map-use"></a>

### <a name="use-run-and-map"></a>Use, Run und Map

Sie können die HTTP-Pipeline mit `Use`, `Run` und `Map` konfigurieren. Die `Use`-Methode kann die Pipeline kurzschließen (wenn sie keinen `next`-Anforderungsdelegaten aufruft). `Run` ist eine Konvention. Einige Middlewarekomponenten machen möglicherweise `Run[Middleware]`-Methoden verfügbar, die am Ende einer Pipeline ausgeführt werden.

`Map*`-Erweiterungen werden als Konvention zum Branchen der Pipeline verwendet. [Map](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mapextensions) brancht die Anforderungspipeline auf Grundlage von Übereinstimmungen des angegebenen Anforderungspfads. Wenn der Anforderungspfad mit dem angegebenen Pfad beginnt, wird der Branch ausgeführt.

[!code-csharp[](index/sample/Chain/StartupMap.cs?name=snippet1)]

In der folgenden Tabelle sind die Anforderungen und Antworten von `http://localhost:1234` mit dem oben stehenden Code aufgelistet:

| Anforderung | Antwort |
| --- | --- |
| localhost:1234 | Hello from non-Map delegate.  |
| localhost:1234/map1 | Map Test 1 |
| localhost:1234/map2 | Map Test 2 |
| localhost:1234/map3 | Hello from non-Map delegate.  |

Wenn `Map` verwendet wird, werden die übereinstimmenden Pfadsegmente bzw. das übereinstimmende Pfadsegment aus `HttpRequest.Path` entfernt und für jede Anforderung an `HttpRequest.PathBase` angehängt.

[MapWhen](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.mapwhenextensions) brancht die Anforderungspipeline auf Grundlage des Ergebnisses des angegebenen Prädikats. Jedes Prädikat vom Typ `Func<HttpContext, bool>` kann verwendet werden, um Anforderungen einem neuen Branch der Pipeline zuzuordnen. Im folgenden Beispiel wird ein Prädikat verwendet, um das Vorhandensein der Abfragezeichenfolgenvariablen `branch` zu ermitteln:

[!code-csharp[](index/sample/Chain/StartupMapWhen.cs?name=snippet1)]

In der folgenden Tabelle sind die Anforderungen und Antworten von `http://localhost:1234` mit dem oben stehenden Code aufgelistet:

| Anforderung | Antwort |
| --- | --- |
| localhost:1234 | Hello from non-Map delegate.  |
| localhost:1234/?branch=master | Branch used = master|

`Map` unterstützt das Schachteln, wie z.B. in folgendem Code:

```csharp
app.Map("/level1", level1App => {
       level1App.Map("/level2a", level2AApp => {
           // "/level1/level2a"
           //...
       });
       level1App.Map("/level2b", level2BApp => {
           // "/level1/level2b"
           //...
       });
   });
   ```

`Map` kann auch mehrere Segmente auf einmal zuordnen, z.B.:

 ```csharp
app.Map("/level1/level2", HandleMultiSeg);
```

## <a name="built-in-middleware"></a>Integrierte Middleware

ASP.NET Core wird mit den folgenden Middlewarekomponenten ausgeliefert. Diese Tabelle enthält auch die Reihenfolge, in der diese hinzugefügt werden sollten:

| Middleware | description | Reihenfolge |
| ---------- | ----------- | ----- |
| [Authentifizierung](xref:security/authentication/identity) | Bietet Unterstützung für Authentifizierungen. | Bevor `HttpContext.User` erforderlich ist. Terminal für OAuth-Rückrufe. |
| [CORS](xref:security/cors) | Konfiguriert die Ressourcenfreigabe zwischen verschiedenen Ursprüngen (Cross-Origin Resource Sharing, CORS). | Vor Komponenten, die CORS verwenden. |
| [Diagnose](xref:fundamentals/error-handling) | Konfiguriert Diagnosen. | Vor Komponenten, die Fehler erzeugen. |
| [Umgeleitete Header/HttpOverrides](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions) | Leitet Proxyheader an die aktuelle Anforderung weiter. | Vor Komponenten, welche die aktualisierten Felder verwenden (z.B. Scheme, Host, ClientIP, Method). |
| [Zwischenspeichern von Antworten](xref:performance/caching/middleware) | Bietet Unterstützung für das Zwischenspeichern von Antworten. | Vor Komponenten, für die das Zwischenspeichern erforderlich ist. |
| [Antwortkomprimierung](xref:performance/response-compression) | Bietet Unterstützung für das Komprimieren von Antworten. | Vor Komponenten, für die das Komprimieren erforderlich ist. |
| [Anforderungslokalisierung](xref:fundamentals/localization) | Bietet Unterstützung für die Lokalisierung. | Vor der Lokalisierung vertraulicher Komponenten. |
| [Routing](xref:fundamentals/routing) | Definiert Anforderungsrouten und schränkt diese ein. | Terminal für entsprechende Routen. |
| [Sitzung](xref:fundamentals/app-state) | Bietet Unterstützung für das Verwalten von Benutzersitzungen. | Vor Komponenten, für die Sitzungen erforderlich sind. |
| [Statische Dateien](xref:fundamentals/static-files) | Bietet Unterstützung für das Verarbeiten statischer Dateien und das Durchsuchen des Verzeichnisses. | Terminal, wenn eine Anforderung mit den Dateien übereinstimmt. |
| [URL-Umschreibung](xref:fundamentals/url-rewriting) | Bietet Unterstützung für das Umschreiben von URLs und das Umleiten von Anforderungen. | Vor Komponenten, die die URL nutzen. |
| [WebSockets](xref:fundamentals/websockets) | Aktiviert das WebSockets-Protokoll. | Vor Komponenten, die WebSocket-Anforderungen annehmen müssen. |

<a name="middleware-writing-middleware"></a>

## <a name="writing-middleware"></a>Schreiben von Middleware

Für gewöhnlich ist Middleware in einer Klasse gekapselt und wird mit einer Erweiterungsmethode verfügbar gemacht. Sehen Sie sich folgende Middleware an, die die Kultur der aktuellen Anforderung über die Abfragezeichenfolge festlegt:

[!code-csharp[](index/sample/Culture/StartupCulture.cs?name=snippet1)]

Hinweis: Der oben stehende Beispielcode veranschaulicht die Erstellung einer Middlewarekomponente. Lesen Sie den Artikel zu [Globalisierung und Lokalisierung in ASP.NET Core](xref:fundamentals/localization), um mehr über die integrierte Lokalisierungsunterstützung zu erfahren.

Sie können die Middleware testen, indem Sie die Kultur übergeben (z.B. `http://localhost:7997/?culture=no`).

Im folgenden Code wird der Middlewaredelegat in eine Klasse verschoben:

[!code-csharp[](index/sample/Culture/RequestCultureMiddleware.cs)]

> [!NOTE]
> Der Name der Middlewaremethode `Task` muss in ASP.NET Core 1.x `Invoke` lauten. In ASP.NET Core 2.0 oder höher kann der Name `Invoke` oder `InvokeAsync` lauten.

Die folgende Erweiterungsmethode macht die Middleware über [IAppllicationBuilder](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.iapplicationbuilder) verfügbar:

[!code-csharp[](index/sample/Culture/RequestCultureMiddlewareExtensions.cs)]

Der folgende Code ruft die Methode von `Configure` auf:

[!code-csharp[](index/sample/Culture/Startup.cs?name=snippet1&highlight=5)]

Middleware sollte das [Prinzip der expliziten Abhängigkeiten](http://deviq.com/explicit-dependencies-principle/) befolgen, indem sie ihre Abhängigkeiten in ihrem Konstruktor verfügbar macht. Middleware wird einmal während der *Anwendungslebensdauer* erstellt. Lesen Sie den folgenden Abschnitt zu *anforderungsbasierten Abhängigkeiten*, wenn Sie Dienste für Middleware innerhalb einer Anforderung gemeinsam verwenden müssen.

Middlewarekomponenten können Ihre Abhängigkeiten über Dependency Injection mit Konstruktorparametern auflösen. [`UseMiddleware<T>`](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.builder.usemiddlewareextensions#methods_summary) kann auch direkt zusätzliche Parameter annehmen.

### <a name="per-request-dependencies"></a>Voranforderungsbasierte Abhängigkeiten

Weil Middleware zum Zeitpunkt des Anwendungsstarts erstellt wird (und nicht voranforderungsbasiert), werden *bereichsbezogene* Lebensdauerdienste von Middlewarekonstruktoren in den einzelnen Anforderungen nicht gemeinsam mit anderen Typen mit Dependency Injection verwendet. Wenn Sie einen *bereichsbezogenen* Dienst sowohl in Ihrer Middleware als auch in anderen Typen verwenden müssen, fügen Sie diese Dienste zur Signatur der `Invoke`-Methode hinzu. Die `Invoke`-Methode kann zusätzliche Parameter akzeptieren, die durch Dependency Injection aufgefüllt werden. Zum Beispiel:

```c#
public class MyMiddleware
{
    private readonly RequestDelegate _next;

    public MyMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext httpContext, IMyScopedService svc)
    {
        svc.MyProperty = 1000;
        await _next(httpContext);
    }
}
```

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [Migrieren von HTTP-Modulen zu Middleware](xref:migration/http-modules)
* [Application Startup (Starten von Anwendungen)](xref:fundamentals/startup)
* [Erforderliche Funktionen](xref:fundamentals/request-features)
* [Factorybezogene Middlewareaktivierung](xref:fundamentals/middleware/extensibility)
* [Middlewareaktivierung mit einem Drittanbietercontainer](xref:fundamentals/middleware/extensibility-third-party-container)
