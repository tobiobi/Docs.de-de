---
title: Hosten von ASP.NET Core unter Windows mit IIS
author: guardrex
description: Erfahren Sie, wie ASP.NET Core-Apps in Windows Server Internet Information Services (IIS) gehostet werden.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 03/13/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: host-and-deploy/iis/index
ms.openlocfilehash: 64eb85f75a6c2e10bf8c39f32eeda5311744f2a2
ms.sourcegitcommit: 7d02ca5f5ddc2ca3eb0258fdd6996fbf538c129a
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2018
---
# <a name="host-aspnet-core-on-windows-with-iis"></a>Hosten von ASP.NET Core unter Windows mit IIS

Von [Luke Latham](https://github.com/guardrex) und [Rick Anderson](https://twitter.com/RickAndMSFT)

## <a name="supported-operating-systems"></a>Unterstützte Betriebssysteme

Die folgenden Betriebssysteme werden unterstützt:

* Windows 7 oder höher
* Windows Server 2008 R2 oder höher&#8224;

&#8224;Vom Konzept her gilt die in diesem Dokument beschriebene IIS-Konfiguration auch für das Hosten von ASP.NET Core-Apps mit IIS unter Nano Server. Die Anweisungen für Nano Server finden Sie im Tutorial [ASP.NET Core mit IIS auf Nano Server](xref:tutorials/nano-server).

Der [HTTP.SYS-Server](xref:fundamentals/servers/httpsys) (zuvor [WebListener](xref:fundamentals/servers/weblistener) genannt) funktioniert nicht in einer Reverseproxykonfiguration mit IIS. Verwenden Sie den [Kestrel-Server](xref:fundamentals/servers/kestrel).

## <a name="application-configuration"></a>Anwendungskonfiguration

### <a name="enable-the-iisintegration-components"></a>Aktivieren der IISIntegration-Komponenten

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

Eine typische *Program.cs*-Datei ruft [CreateDefaultBuilder](/dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder) auf, um die Einrichtung eines Hosts zu starten. `CreateDefaultBuilder` konfiguriert [Kestrel](xref:fundamentals/servers/kestrel) als Webserver und aktiviert die IIS-Integration durch Konfigurierung des Basispfads und Ports für das [ASP.NET Core-Modul](xref:fundamentals/servers/aspnet-core-module):

```csharp
public static IWebHost BuildWebHost(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        ...
```

Das ASP.NET Core-Modul generiert einen dynamischen Port, der dem Back-End-Prozess zugewiesen wird. Die `UseIISIntegration`-Methode übernimmt diesen dynamischen Port und konfiguriert Kestrel zum Lauschen von `http://locahost:{dynamicPort}/`. Dies überschreibt andere URL-Konfigurationen, z.B. Aufrufe von `UseUrls` oder [Kestrels Listen-API](xref:fundamentals/servers/kestrel#endpoint-configuration). Aus diesem Grund sind Aufrufe von `UseUrls` oder der `Listen`-API von Kestrel nicht erforderlich, wenn das Modul verwendet wird. Wenn `UseUrls` oder `Listen` aufgerufen wird, lauscht Kestrel auf den Port, der bei der bei Ausführung der App ohne den IIS angegeben wird.

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

Nehmen Sie eine Abhängigkeit vom Paket [Microsoft.AspNetCore.Server.IISIntegration](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IISIntegration/) in die App-Abhängigkeiten auf. Verwenden Sie Middleware für die Integration von IIS, indem Sie die Erweiterungsmethode [UseIISIntegration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderiisextensions.useiisintegration) zu [WebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilder) hinzufügen:

```csharp
var host = new WebHostBuilder()
    .UseKestrel()
    .UseIISIntegration()
    ...
```

Sowohl [UseKestrel](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderkestrelextensions.usekestrel) als auch [UseIISIntegration](/dotnet/api/microsoft.aspnetcore.hosting.webhostbuilderiisextensions.useiisintegration) sind erforderlich. Ein Code, der `UseIISIntegration` aufruft, hat keinen Einfluss auf die Codeportabilität. Wenn die App nicht hinter IIS ausgeführt wird (die App wird z.B. direkt auf Kestrel ausgeführt), hat `UseIISIntegration` keine Funktion.

Das ASP.NET Core-Modul generiert einen dynamischen Port, der dem Back-End-Prozess zugewiesen wird. Die `UseIISIntegration`-Methode übernimmt diesen dynamischen Port und konfiguriert Kestrel zum Lauschen von `http://locahost:{dynamicPort}/`. Dies überschreibt andere URL-Konfigurationen, z.B. Aufrufe von `UseUrls`. Deshalb ist ein Aufruf von `UseUrls` nicht erforderlich, wenn das Modul verwendet wird. Wenn `UseUrls` aufgerufen wird, lauscht Kestrel auf den Port, der bei der Ausführung der App ohne den IIS angegeben wird.

Wenn `UseUrls` in einer ASP.NET Core 1.0-App aufgerufen wird, rufen Sie es **vor** Aufrufen von `UseIISIntegration` auf, damit der vom Modul konfigurierte Port nicht überschrieben wird. Diese Aufrufreihenfolge ist in ASP.NET Core 1.1 nicht erforderlich, weil die Moduleinstellung `UseUrls` überschreibt.

---

Weitere Informationen zum Hosten finden Sie unter [Hosting in ASP.NET Core (Hosten in ASP.NET Core)](xref:fundamentals/hosting).

### <a name="iis-options"></a>IIS-Optionen

Um IIS-Optionen zu konfigurieren, beziehen Sie eine Dienstkonfiguration für [IISOptions](/dotnet/api/microsoft.aspnetcore.builder.iisoptions) in [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices) ein. Im folgenden Beispiel ist das Weiterleiten von Clientzertifikaten an die App zum Auffüllen von `HttpContext.Connection.ClientCertificate` deaktiviert:

```csharp
services.Configure<IISOptions>(options => 
{
    options.ForwardClientCertificate = false;
});
```

| Option                         | Standard | Einstellung |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | Bei Festlegung auf `true` legt die Middleware für die IIS-Integration den per [Windows-Authentifizierung](xref:security/authentication/windowsauth) authentifizierten `HttpContext.User` fest. Bei Festlegung auf `false` stellt die Middleware nur eine Identität für `HttpContext.User` bereit und antwortet auf explizite Anforderungen durch `AuthenticationScheme`. Die Windows-Authentifizierung muss in IIS aktiviert sein, damit `AutomaticAuthentication` funktioniert. Weitere Informationen finden Sie im Thema [Windows-Authentifizierung](xref:security/authentication/windowsauth). |
| `AuthenticationDisplayName`    | `null`  | Legt den Anzeigename fest, der Benutzern auf Anmeldungsseiten angezeigt wird |
| `ForwardClientCertificate`     | `true`  | Wenn diese Option `true` ist und der Anforderungsheader `MS-ASPNETCORE-CLIENTCERT` vorhanden ist, wird das `HttpContext.Connection.ClientCertificate` aufgefüllt. |

### <a name="proxy-server-and-load-balancer-scenarios"></a>Proxyserver und Lastenausgleichsszenarien

Die Middleware für die Integration von IIS, die ForwardedHeadersMiddleware konfiguriert, und das ASP.NET Core-Modul sind so konfiguriert, dass sie das Schema (HTTP/HTTPS) und die Remote-IP-Adresse an die Stelle weiterleiten, von der die Anforderung stammte. Möglicherweise ist zusätzliche Konfiguration für Apps erforderlich, die hinter weiteren Proxyservern und Lastenausgleichsmodulen (Load Balancer) gehostet werden. Weitere Informationen hierzu feinden Sie unter [Konfigurieren von ASP.NET Core zur Verwendung mit Proxyservern und Lastenausgleich](xref:host-and-deploy/proxy-load-balancer).

### <a name="webconfig-file"></a>Datei „web.config“

Mit der Datei *web.config* wird das [ASP.NET Core-Modul](xref:fundamentals/servers/aspnet-core-module) konfiguriert. Das Erstellen, das Transformieren und die Veröffentlichung von *web.config* erfolgt durch das .NET Core Web SDK (`Microsoft.NET.Sdk.Web`). Das SDK wird am Anfang der Projektdatei festgelegt:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

Wenn in der Projektdatei keine Datei namens *web.config* vorhanden ist, wird sie zur Konfiguration des [ASP.NET Core-Moduls](xref:fundamentals/servers/aspnet-core-module) mit dem richtigen *processPath*- und *arguments*-Attribut erstellt und in die [veröffentlichte Ausgabe](xref:host-and-deploy/directory-structure) verschoben.

Ist eine Datei namens *web.config* im Projekt vorhanden, wird sie zur Konfiguration des ASP.NET Core-Moduls mit dem richtigen *processPath*- und *arguments*-Attribut transformiert und in die veröffentlichte Ausgabe verschoben. Die Transformation ändert die IIS-Konfigurationseinstellungen in der Datei nicht.

Die Datei *web.config* kann zusätzliche IIS-Konfigurationseinstellungen zum Steuern der aktiven IIS-Module bereitstellen. Informationen zu IIS-Modulen, die Anforderungen mit ASP.NET Core-Apps verarbeiten können, finden Sie im Thema [IIS-Module](xref:host-and-deploy/iis/modules).

Um zu verhindern, dass das Web-SDK die Datei *web.config* transformiert, verwenden Sie die Eigenschaft **\<IsTransformWebConfigDisabled>** in der Projektdatei:

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

Wenn die Transformation der Datei durch das Web-SDK deaktiviert wird, müssen die Attribute *processPath* und *arguments* manuell durch den Entwickler festgelegt werden. Weitere Informationen finden Sie unter [Konfigurationsreferenz für das ASP.NET Core-Modul](xref:host-and-deploy/aspnet-core-module).

### <a name="webconfig-file-location"></a>Speicherort der Datei „web.config“

.NET Core-Apps werden in einem Reverseproxy zwischen IIS und dem Kestrel-Server gehostet. Zum Erstellen des Reverseproxys muss die Datei *web.config* im Stammpfad des Inhalts (normalerweise dem App-Basispfad) der bereitgestellten App vorhanden sein. Dies ist der physische Pfad der Website, der in IIS bereitgestellt wurde. Die Datei *Web.config* wird im Stammverzeichnis der App benötigt, um die Veröffentlichung mehrerer Apps mit Web Deploy zu ermöglichen.

Im physischen Pfad der App sind vertrauliche Dateien vorhanden, wie z.B. *\<assembly_name>.runtimeconfig.json*, *\<assembly_name>.xml* (XML-Dokumentationskommentare) und *\<assembly_name>.deps.json*. Wenn die Datei *web.config* vorhanden ist und die Website normal startet, werden diese vertraulichen Dateien von IIS bei Anforderung nicht offengelegt. Wenn die Datei *web.config* fehlt, falsch benannt wurde oder die Website nicht für den normalen Start konfigurieren kann, macht IIS vertrauliche Dateien möglicherweise öffentlich verfügbar.

**Die Datei *web.config* muss immer in der Bereitstellung vorhanden, richtig benannt und in der Lage sein, die Website für einen normalen Start zu konfigurieren. Entfernen Sie die Datei *web.config* niemals aus einer Produktionsbereitstellung.**

## <a name="iis-configuration"></a>IIS-Konfiguration

**Windows Server-Betriebssysteme**

Aktivieren Sie die Serverrolle **Webserver (IIS)**, und richten Sie Rollendienste ein.

1. Verwenden Sie den Assistenten **Rollen und Features hinzufügen** im Menü **Verwalten** oder den Link in **Server-Manager**. Aktivieren Sie im Schritt **Serverrollen** das Kontrollkästchen für **Webserver (IIS)**.

   ![Die Rolle „Webserver (IIS)“ wird im Schritt „Serverrollen auswählen“ ausgewählt.](index/_static/server-roles-ws2016.png)

1. Nach dem Schritt **Features** wird der Schritt **Rollendienste** für Webserver (IIS) geladen. Wählen Sie die gewünschten IIS-Rollendienste aus, oder übernehmen Sie die bereitgestellten Standardrollendienste.

   ![Die Standardrollendienste werden im Schritt „Rollendienste auswählen“ ausgewählt.](index/_static/role-services-ws2016.png)

   **Windows-Authentifizierung (optional)**  
   Um die Windows-Authentifizierung zu aktivieren, erweitern Sie die folgenden Knoten: **Webserver** > **Sicherheit**. Wählen Sie das Feature **Windows-Authentifizierung** aus. Weitere Informationen finden Sie unter [Windows-Authentifizierung \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) und [Konfigurieren der Windows-Authentifizierung](xref:security/authentication/windowsauth).

   **WebSockets (optional)**  
   WebSockets wird mit ASP.NET Core 1.1 oder höher unterstützt. Um WebSockets zu aktivieren, erweitern Sie die folgenden Knoten: **Webserver** > **Anwendungsentwicklung**. Wählen Sie das Feature **WebSocket-Protokoll** aus. Weitere Informationen finden Sie unter [WebSockets](xref:fundamentals/websockets).

1. Fahren Sie mit dem Schritt **Bestätigung** fort, um die Webserverrolle und die Dienste zu installieren. Ein Server-/IIS-Neustart ist nach der Installation der Rolle **Webserver (IIS)** nicht erforderlich.

**Windows-Desktopbetriebssysteme**

Aktivieren Sie die **IIS-Verwaltungskonsole** und die **WWW-Dienste**.

1. Navigieren Sie zu **Systemsteuerung** > **Programme** > **Programme und Features** > **Windows-Features aktivieren oder deaktivieren** (links auf dem Bildschirm).

1. Öffnen Sie den Knoten **Internetinformationsdienste**. Öffnen Sie den Knoten **Webverwaltungstools**.

1. Aktivieren Sie das Kontrollkästchen für **IIS-Verwaltungskonsole**.

1. Aktivieren Sie das Kontrollkästchen für **WWW-Dienste**.

1. Akzeptieren Sie die Standardfeatures für **WWW-Dienste**, oder passen Sie die IIS-Features an.

   **Windows-Authentifizierung (optional)**  
   Um die Windows-Authentifizierung zu aktivieren, erweitern Sie die folgenden Knoten: **WWW-Dienste** > **Sicherheit**. Wählen Sie das Feature **Windows-Authentifizierung** aus. Weitere Informationen finden Sie unter [Windows-Authentifizierung \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) und [Konfigurieren der Windows-Authentifizierung](xref:security/authentication/windowsauth).

   **WebSockets (optional)**  
   WebSockets wird mit ASP.NET Core 1.1 oder höher unterstützt. Um WebSockets zu aktivieren, erweitern Sie die folgenden Knoten: **WWW-Dienste** > **Anwendungsentwicklungsfeatures**. Wählen Sie das Feature **WebSocket-Protokoll** aus. Weitere Informationen finden Sie unter [WebSockets](xref:fundamentals/websockets).

1. Wenn die IIS-Installation einen Neustart erfordert, starten Sie das System neu.

![Die IIS-Verwaltungskonsole und WWW-Dienste werden in Windows-Features ausgewählt.](index/_static/windows-features-win10.png)

---

## <a name="install-the-net-core-windows-server-hosting-bundle"></a>Installieren des Pakets „.NET Core Windows Server Hosting“

1. Installieren Sie das *Paket „.NET Core Windows Server Hosting“* im Hostsystem. Das Paket installiert die .NET Core-Runtime, die .NET Core-Bibliothek und das [ASP.NET Core-Modul](xref:fundamentals/servers/aspnet-core-module). Das Modul erstellt den Reverseproxy zwischen IIS und dem Kestrel-Server. Wenn das System nicht über eine Internetverbindung verfügt, beziehen und installieren Sie [Microsoft Visual C++ 2015 Redistributable](https://www.microsoft.com/download/details.aspx?id=53840), bevor Sie das Paket „.NET Core Windows Server Hosting“ installieren.

   1. Navigieren Sie zu der [.NET-Seite „All Downloads“](https://www.microsoft.com/net/download/all).
   1. Wählen Sie die neueste Nicht-Vorschau-.NET Core-Runtime in der Liste aus (**.NET Core** > **Runtime** > **.NET Core-Runtime x.y.z**). Sofern Sie nicht mit Preview-Software (Vorschausoftware) arbeiten möchten, sollten Sie Runtimes vermeiden, in deren Linktext das Wort „preview“ vorhanden ist.
   1. Wählen Sie auf der Downloadseite von .NET Core-Runtime unter **Windows** den Link **Server Hosting Installer** aus, um das *Paket „.NET Core Windows Server Hosting“* herunterzuladen.

   **Wichtig** Wenn das Hostingpaket vor IIS installiert wird, muss die Paketinstallation repariert werden. Führen Sie nach der Installation von IIS erneut das Hostingpaket aus.
   
   Damit das Installationsprogramm in einem x64-Betriebssystem keine x86-Pakete installiert, führen Sie das Installationsprogramm über eine Administratoreingabeaufforderung mit dem Switch `OPT_NO_X86=1` aus.

1. Starten Sie das System neu, oder führen Sie **net stop was /y** gefolgt von **net start w3svc** über eine Eingabeaufforderung aus. Durch den Neustart von IIS wird eine Änderung an der PATH-Systemeinstellung durch den Installer vorgenommen.

> [!NOTE]
> Informationen zur Verwendung einer IIS-Freigabekonfiguration finden Sie unter [ASP.NET Core-Modul mit IIS-Freigabekonfiguration](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration).

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>Installieren von Web Deploy beim Veröffentlichen mit Visual Studio

Wenn Sie Apps auf Servern mit [Web Deploy](/iis/publish/using-web-deploy/introduction-to-web-deploy) bereitstellen, installieren Sie die neueste Version von Web Deploy auf dem Server. Um Web Deploy zu installieren, verwenden Sie den [Webplattform-Installer (Web PI)](https://www.microsoft.com/web/downloads/platform.aspx) oder rufen einen Installer direkt aus dem [Microsoft Download Center](https://www.microsoft.com/download/details.aspx?id=43717) ab. Die bevorzugte Methode ist die Verwendung von WebPI. WebPI bietet ein eigenständiges Setup und eine Konfiguration für Hostinganbieter.

## <a name="create-the-iis-site"></a>Erstellen der IIS-Website

1. Erstellen Sie auf dem Hostingsystem einen Ordner zum Speichern der veröffentlichten Ordner und Dateien der App. Das Layout der App-Bereitstellung wird im Thema [Verzeichnisstruktur](xref:host-and-deploy/directory-structure) beschrieben.

1. Erstellen Sie innerhalb des neuen Ordners einen Ordner *Protokolle*, um darin stdout-Protokolle von ASP.NET Core zu speichern, wenn die stdout-Protokollierung aktiviert ist. Wenn die App mit einem Ordner *Protokolle* in der Payload bereitgestellt wird, können Sie diesen Schritt überspringen. Anweisungen zum Aktivieren von MSBuild zum automatischen Erstellen des Ordners *Protokolle* bei der Erstellung des lokalen Projekts finden Sie im Thema zur [Verzeichnisstruktur](xref:host-and-deploy/directory-structure).

   **Wichtig** Verwenden Sie das stdout-Protokoll, um Probleme beim App-Start zu behandeln. Verwenden Sie die stdout-Protokollierung nicht für die routinemäßige App-Protokollierung. Für die Protokollgröße oder die Anzahl von erstellten Protokolldateien ist kein Grenzwert festgelegt. Weitere Informationen zum stdout-Protokoll finden Sie im Thema zur [Protokollerstellung und Umleitung](xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection). Informationen zur Protokollierung in einer ASP.NET Core-App finden Sie im Thema [Protokollierung](xref:fundamentals/logging/index).

1. Öffnen Sie im **IIS-Manager** den Serverknoten im Bereich **Verbindungen**. Klicken Sie mit der rechten Maustaste auf den Ordner **Websites**. Klicken Sie im Kontextmenü auf **Website hinzufügen**.

1. Geben Sie einen **Websitenamen** an, und legen Sie den **physischen Pfad** auf den Bereitstellungsordner der App fest. Geben Sie die Konfiguration unter **Bindung** an, und erstellen Sie die Website, indem Sie auf **OK** klicken:

   ![Geben Sie den Websitenamen, den physischen Pfad und den Hostnamen im Schritt „Website hinzufügen“ an.](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > Allgemeine Platzhalterbindungen (`http://*:80/` und `http://+:80`) dürfen **nicht** verwendet werden. Platzhalterbindungen auf oberster Ebene gefährden die Sicherheit Ihrer App. Dies gilt für starke und schwache Platzhalter. Verwenden Sie statt Platzhaltern explizite Hostnamen. Platzhalterbindungen in untergeordneten Domänen (z.B. `*.mysub.com`) verursachen kein Sicherheitsrisiko, wenn Sie die gesamte übergeordnete Domäne steuern (im Gegensatz zu `*.com`, das angreifbar ist). Weitere Informationen finden Sie unter [rfc7230 im Abschnitt 5.4](https://tools.ietf.org/html/rfc7230#section-5.4).

1. Wählen Sie unter dem Serverknoten **Anwendungspools** aus.

1. Klicken Sie mit der rechten Maustaste auf den App-Pool der Website, und wählen Sie im Kontextmenü **Grundeinstellungen** aus.

1. Legen Sie im Fenster **Anwendungspool bearbeiten** die **.NET CLR-Version** auf **Kein verwalteter Code** fest:

   ![Legen Sie „Kein verwalteter Code“ für die .NET CLR-Version fest.](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core wird in einem separaten Prozess ausgeführt und verwaltet die Runtime. Für ASP.NET Core ist das Laden der Desktop-CLR nicht erforderlich. Das Festlegen der **.NET CLR-Version** auf **Kein verwalteter Code** ist optional.

1. Vergewissern Sie sich, dass die Prozessmodellidentität über die richtigen Berechtigungen verfügt.

   Wenn die Standardidentität des App-Pools (**Prozessmodell** > **Identität**) von **ApplicationPoolIdentity** in eine andere Identität geändert wird, stellen Sie sicher, dass die neue Identität über die erforderlichen Berechtigungen zum Zugriff auf den Ordner, die Datenbank und andere erforderliche Ressourcen der App verfügt. Der App-Pool benötigt beispielsweise Lese- und Schreibzugriff für Ordner, in denen die App Lese- und Schreibvorgänge für Dateien ausführt.

**Konfigurieren der Windows-Authentifizierung (optional)**  
Weitere Informationen finden Sie unter [Konfigurieren der Windows-Authentifizierung](xref:security/authentication/windowsauth).

## <a name="deploy-the-app"></a>Bereitstellen der App

Stellen Sie die App in dem Ordner bereit, den Sie im Hostingsystem erstellt haben. [Web Deploy](/iis/publish/using-web-deploy/introduction-to-web-deploy) ist die empfohlene Methode zur Bereitstellung.

### <a name="web-deploy-with-visual-studio"></a>Web Deploy mit Visual Studio

Informationen zum Erstellen eines Veröffentlichungsprofils zur Verwendung mit Web Deploy finden Sie im Thema [Visual Studio publish profiles for ASP.NET Core app deployment (Visual Studio-Veröffentlichungsprofile zum Bereitstellen von ASP.NET Core-Apps)](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles). Wenn der Hostinganbieter ein Veröffentlichungsprofil oder Unterstützung für das Erstellen eines solchen Profils bereitstellt, laden Sie dessen Profil herunter, und importieren Sie es mithilfe des Visual Studio-Dialogfelds **Veröffentlichen**.

![Dialogfeld „Veröffentlichen“](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>Web Deploy außerhalb von Visual Studio

[Web Deploy](/iis/publish/using-web-deploy/introduction-to-web-deploy) kann über die Befehlszeile auch außerhalb von Visual Studio verwendet werden. Weitere Informationen finden Sie unter [Web Deployment Tool](/iis/publish/using-web-deploy/use-the-web-deployment-tool) (Webbereitstellungstool).

### <a name="alternatives-to-web-deploy"></a>Alternativen zu Web Deploy

Verwenden Sie eine der folgenden Methoden, um die App zum Hostingsystem zu migrieren: manuelles Kopieren, XCopy, Robocopy oder PowerShell.

## <a name="browse-the-website"></a>Navigieren auf der Website

![Der Microsoft Edge-Browser hat die IIS-Startseite geladen.](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>Gesperrte Bereitstellungsdateien

Die Dateien im Bereitstellungsordner werden gesperrt, wenn die App ausgeführt wird. Gesperrte Dateien können während der Bereitstellung nicht überschrieben werden. Um gesperrte Dateien in einer Bereitstellung freizugeben, beenden Sie den App-Pool mit **einer** der folgenden Methoden:

* Verwenden Sie Web Deploy, und verweisen Sie auf `Microsoft.NET.Sdk.Web` in der Projektdatei. Eine Datei namens *app_offline.htm* befindet sich im Stammverzeichnis des Web-App-Verzeichnisses. Wenn die Datei vorhanden ist, fährt das ASP.NET Core Module die App ordnungsgemäß herunter und verarbeitet die Datei *app_offline.htm* während der Bereitstellung. Weitere Informationen finden Sie unter [Konfigurationsreferenz für das ASP.NET Core-Modul](xref:host-and-deploy/aspnet-core-module#appofflinehtm).
* Beenden Sie den App-Pool im IIS-Manager auf dem Server manuell.
* Verwenden Sie PowerShell, um den App-Pool zu beenden und neu zu starten (erfordert PowerShell 5 oder höher):

  ```PowerShell
  $webAppPoolName = 'APP_POOL_NAME'

  # Stop the AppPool
  if((Get-WebAppPoolState $webAppPoolName).Value -ne 'Stopped') {
      Stop-WebAppPool -Name $webAppPoolName
      while((Get-WebAppPoolState $webAppPoolName).Value -ne 'Stopped') {
          Start-Sleep -s 1
      }
      Write-Host `-AppPool Stopped
  }

  # Provide script commands here to deploy the app

  # Restart the AppPool
  if((Get-WebAppPoolState $webAppPoolName).Value -ne 'Started') {
      Start-WebAppPool -Name $webAppPoolName
      while((Get-WebAppPoolState $webAppPoolName).Value -ne 'Started') {
          Start-Sleep -s 1
      }
      Write-Host `-AppPool Started
  }
  ```

## <a name="data-protection"></a>Schutz von Daten

Der [ASP.NET Core-Stapel zum Schutz von Daten](xref:security/data-protection/index) wird von mehreren ASP.NET-[Middlewarekomponenten](xref:fundamentals/middleware/index) genutzt, darunter auch von Middleware, die bei der Authentifizierung verwendet wird. Selbst wenn die APIs zum Schutz von Daten nicht aus dem Benutzercode aufgerufen werden, sollte der Schutz von Daten mit einem Bereitstellungsskript oder im Benutzercode konfiguriert werden, um einen persistenten kryptografischen [Schlüsselspeicher](xref:security/data-protection/implementation/key-management) zu erstellen. Wenn der Schutz von Daten nicht konfiguriert ist, werden die Schlüssel beim Neustarten der App im Arbeitsspeicher gespeichert und verworfen.

Falls der Schlüsselbund im Arbeitsspeicher gespeichert wird, wenn die App neu gestartet wird, gilt Folgendes:

* Alle cookiebasierten Authentifizierungstoken für ungültig erklärt. 
* Benutzer müssen sich bei ihrer nächsten Anforderung erneut anmelden. 
* Alle mit dem Schlüsselbund geschützte Daten können nicht mehr entschlüsselt werden. Dies kann [CSRF-Token](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration) und [ASP.NET Core-MVC-TempData-Cookies](xref:fundamentals/app-state#tempdata) einschließen.

Zum Konfigurieren des Schutzes von Daten unter IIS mithilfe des persistenten Schlüsselbunds verwenden Sie **einen** der folgenden Ansätze:

* **Erstellen einer Registrierungsstruktur für den Schutz von Daten**

  Schlüssel für den Schutz von Daten, die von ASP.NET Core-Apps verwendet werden, werden in der Registrierung außerhalb der Apps gespeichert. Um die Schlüssel für eine bestimmte App zu dauerhaft zu speichern, müssen Sie Registrierungsschlüssel für den App-Pool erstellen.

  Bei eigenständigen IIS-Installationen, die ohne Webfarm vorgesehen sind, kann das [PowerShell-Skript „Provision-AutoGenKeys.ps1“ für den Schutz von Daten](https://github.com/aspnet/DataProtection/blob/dev/Provision-AutoGenKeys.ps1) für jeden App-Pool genutzt werden, das mit einer ASP.NET Core-App verwendet wird. Dieses Skript erstellt einen Registrierungsschlüssel in der HKLM-Registrierung, der nur für das Workerprozesskonto des App-Pools der App zugänglich ist. Schlüssel werden in ruhendem Zustand mit DPAPI mit einem computerweiten Schlüssel verschlüsselt.

  In Webfarmszenarios kann eine App so konfiguriert werden, dass sie einen UNC-Pfad verwendet, um den Schlüsselbund für den Schutz von Daten zu speichern. Standardmäßig werden die Schlüssel für den Schutz von Daten nicht verschlüsselt. Stellen Sie sicher, dass die Dateiberechtigungen für die Netzwerkfreigabe auf das Windows-Konto beschränkt sind, mit dem die App ausgeführt wird. Ein X.509-Zertifikat kann zum Schutz von Schlüsseln im ruhenden Zustand verwendet werden. Ziehen Sie einen Mechanismus in Erwägung, um es Benutzern zu ermöglichen, Zertifikate hochzuladen: Platzieren Sie Zertifikate im Zertifikatspeicher des Benutzers für vertrauenswürdige Anbieter, und stellen Sie sicher, dass sie auf allen Computern verfügbar sind, auf denen die App des Benutzers ausgeführt wird. Details finden Sie unter [Konfigurieren des Schutzes von Daten in ASP.NET Core](xref:security/data-protection/configuration/overview).

* **Konfigurieren des IIS-Anwendungspools zum Laden des Benutzerprofils**

  Diese Einstellung befindet sich im Abschnitt **Prozessmodell** unter **Erweiterte Einstellungen** für den App-Pool. Legen Sie „Benutzerprofil laden“ auf `True` fest. Dadurch werden Schlüssel im Benutzerprofilverzeichnis gespeichert und mit DPAPI mit einem Schlüssel geschützt, der nur für das Benutzerkonto gilt, das vom App-Pool verwendet wird.

* **Verwenden des Dateisystems als Schlüsselbundspeicher**

  Passen Sie den App-Code so an, dass er [das Dateisystem als Schlüsselbundspeicher verwendet](xref:security/data-protection/configuration/overview). Verwenden Sie ein X.509-Zertifikat, um den Schlüsselbund zu schützen, und stellen Sie sicher, dass es sich bei dem Zertifikat um ein vertrauenswürdiges Zertifikat handelt. Wenn es sich um ein selbstsigniertes Zertifikat handelt, müssen Sie es im vertrauenswürdigen Stammspeicher platzieren.

  Wenn IIS in einer Webfarm verwendet wird:

  * Verwenden Sie eine Dateifreigabe, auf die alle Computer zugreifen können.
  * Stellen Sie ein X509-Zertifikat auf jedem Computer bereit. Konfigurieren Sie den [Schutz von Daten im Code](xref:security/data-protection/configuration/overview).

* **Festlegen einer computerweiten Richtlinie für den Schutz von Daten**

  Das System zum Schutz von Daten verfügt über eine eingeschränkte Unterstützung zum Festlegen einer [computerweiten Standardrichtlinie](xref:security/data-protection/configuration/machine-wide-policy) für alle Apps, die die Datenschutz-APIs nutzen. Weitere Details finden Sie in der Dokumentation zum [Schutz von Daten](xref:security/data-protection/index).

## <a name="sub-application-configuration"></a>Konfiguration von untergeordneten Anwendungen

Untergeordnete Apps, die unter der Stamm-App hinzugefügt werden, sollten kein ASP.NET Core-Modul als Handler enthalten. Wenn das Modul als Handler in einer *web.config* einer untergeordneten App hinzugefügt wird, erhalten Sie beim Versuch, zur untergeordneten App zu navigieren, den Fehler *500.19: Interner Serverfehler*. Dieser verweist auf die fehlerhafte Konfigurationsdatei.

Das folgende Beispiel zeigt den Inhalt einer veröffentlichten Datei *web.config* für eine untergeordnete ASP.NET Core-App:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <aspNetCore processPath="dotnet" 
      arguments=".\<assembly_name>.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

Wenn Sie eine untergeordnete App ohne ASP.NET Core unterhalb einer ASP.NET Core-App hosten, entfernen Sie den geerbten Handler ausdrücklich aus der Datei *Web.config* der untergeordneten App:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <remove name="aspNetCore" />
    </handlers>
    <aspNetCore processPath="dotnet" 
      arguments=".\<assembly_name>.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

Weitere Informationen zum Konfigurieren des ASP.NET Core-Moduls finden Sie im Thema [Einführung in das ASP.NET Core-Modul](xref:fundamentals/servers/aspnet-core-module) und in der [Konfigurationsreferenz für das ASP.NET Core-Modul](xref:host-and-deploy/aspnet-core-module).

## <a name="configuration-of-iis-with-webconfig"></a>Konfiguration von IIS mit der Datei „web.config“

Die IIS-Konfiguration wird im Hinblick auf IIS-Features, die für eine Reverseproxykonfiguration gelten, vom Abschnitt **\<system.webServer>** der Datei *Web.config* beeinflusst. Wenn IIS auf Systemebene für die Verwendung der dynamischen Komprimierung konfiguriert ist, kann diese Einstellung mit dem Element **\<urlCompression>** in der Datei *web.config* der App deaktiviert werden.

Weitere Informationen finden Sie in der [Konfigurationsreferenz für \<system.webServer>](/iis/configuration/system.webServer/), der [Konfigurationsreferenz für das ASP.NET Core-Modul](xref:host-and-deploy/aspnet-core-module) und unter [IIS-Module mit ASP.NET Core](xref:host-and-deploy/iis/modules). Um Umgebungsvariablen für einzelne Apps festzulegen, die in isolierten App-Pools ausgeführt werden (unterstützt für IIS 10.0 oder höher), lesen Sie den Abschnitt zum *Befehl „AppCmd.exe“* im Thema [Umgebungsvariablen \<environmentVariables>](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) in der IIS-Referenzdokumentation.

## <a name="configuration-sections-of-webconfig"></a>Konfigurationsabschnitte von „web.config“

Konfigurationsabschnitte von ASP.NET 4.x-Apps in der Datei *web.config* werden von ASP.NET Core-Apps nicht zur Konfiguration verwendet:

* **\<system.web>**
* **\<appSettings>**
* **\<connectionStrings>**
* **\<location>**

ASP.NET Core-Apps werden mit anderen Konfigurationsanbietern konfiguriert. Weitere Informationen finden Sie unter [Konfiguration](xref:fundamentals/configuration/index).

## <a name="application-pools"></a>Anwendungspools

Wenn Sie mehrere Websites auf einem Server hosten, isolieren Sie die Apps voneinander, indem Sie jede App in ihrem eigenen App-Pool ausführen. Im IIS-Dialogfeld **Website hinzufügen** wird standardmäßig diese Konfiguration eingesetzt. Wenn ein **Websitename** angegeben ist, wird der Text automatisch in das Textfeld **Anwendungspool** übertragen. Ein neuer App-Pool mit dem Namen der Website wird erstellt, wenn die Website hinzugefügt wird.

## <a name="application-pool-identity"></a>Identität des Anwendungspools

Mit einem Konto für die Identität des App-Pools können Sie eine App unter einem eindeutigen Konto ausführen, ohne Domänen oder lokale Konten erstellen und verwalten zu müssen. Unter IIS 8.0 oder höher erstellt der IIS-Administratorworkerprozess (WAS) ein virtuelles Konto mit dem Namen des neuen App-Pools und führt die Workerprozesse des App-Pools standardmäßig unter diesem Konto aus. Stellen Sie sicher, dass in der IIS-Verwaltungskonsole unter **Erweiterte Einstellungen** für den App-Pool die **Identität** auf **ApplicationPoolIdentity** festgelegt ist:

![Dialogfeld „Erweiterte Einstellungen“ für den Anwendungspool](index/_static/apppool-identity.png)

Der IIS-Verwaltungsprozess erstellt im Windows-Sicherheitssystem einen sicheren Bezeichner mit dem Namen des App-Pools. Ressourcen können mithilfe dieser Identität geschützt werden. Allerdings ist diese Identität kein echtes Benutzerkonto und wird in der Windows-Benutzerverwaltungskonsole nicht angezeigt.

Wenn der IIS-Workerprozess erhöhte Rechte für den Zugriff auf Ihre Anwendung erfordert, ändern Sie die Zugriffssteuerungsliste (ACL) für das Verzeichnis mit der App:

1. Öffnen Sie Windows Explorer, und navigieren Sie zum Verzeichnis.

1. Klicken Sie mit der rechten Maustaste auf das Verzeichnis, und klicken Sie auf **Eigenschaften**.

1. Klicken Sie auf der Registerkarte **Sicherheit** auf die Schaltfläche **Bearbeiten** und dann auf die Schaltfläche **Hinzufügen**.

1. Wählen Sie die Schaltfläche **Speicherorte** aus, und stellen Sie sicher, dass das System ausgewählt ist.

1. Geben Sie im Bereich **Geben Sie die Namen der auszuwählenden Objekte ein** den Wert **IIS AppPool\\<Name_des_AppPools>** ein. Klicken Sie auf die Schaltfläche **Namen überprüfen**. Überprüfen Sie für *DefaultAppPool* die Namen mit **IIS AppPool\DefaultAppPool**. Bei Auswahl der Schaltfläche **Namen überprüfen** wird im Bereich für Objektnamen der Wert **DefaultAppPool** angegeben. Es ist nicht möglich, den Namen des App-Pools direkt in den Bereich für Objektnamen einzugeben. Verwenden Sie das Format **IIS AppPool\\<Name_des_AppPools>**, wenn Sie die Objektnamen überprüfen.

   ![Auswahl des Dialogfelds für Benutzer oder Gruppen für den App-Ordner: Der Name des App-Pools „DefaultAppPool“ wird an „IIS AppPool\"“ im Bereich der Objektnamen angehängt, bevor „Namen überprüfen“ ausgewählt wird.](index/_static/select-users-or-groups-1.png)

1. Klicken Sie auf **OK**.

   ![Auswahl des Dialogfelds für Benutzer oder Gruppen für den App-Ordner: Nach der Auswahl von „Namen überprüfen“ wird der Objektname „DefaultAppPool“ im Bereich der Objektnamen angezeigt.](index/_static/select-users-or-groups-2.png)

1. Standardmäßig sollten Lese- und Schreibberechtigungen gewährt werden. Erteilen Sie weitere Berechtigungen, sofern erforderlich.

Zugriff kann auch über eine Eingabeaufforderung mit dem Tool **ICACLS** gewährt werden. Im folgenden Befehl wird als Beispiel *DefaultAppPool* verwendet:

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

Weitere Informationen finden Sie im Thema [icacls](/windows-server/administration/windows-commands/icacls).

## <a name="additional-resources"></a>Zusätzliche Ressourcen

* [Problembehandlung bei ASP.NET Core in IIS](xref:host-and-deploy/iis/troubleshoot)
* [Referenz zu häufigen Fehlern bei Azure App Service und IIS mit ASP.NET Core](xref:host-and-deploy/azure-iis-errors-reference)
* [Einführung in das ASP.NET Core-Modul](xref:fundamentals/servers/aspnet-core-module)
* [Konfigurationsreferenz für das ASP.NET Core-Modul](xref:host-and-deploy/aspnet-core-module)
* [IIS-Module mit ASP.NET Core](xref:host-and-deploy/iis/modules)
* [Einführung in ASP.NET Core](../index.md)
* [Die offizielle Microsoft IIS-Website](https://www.iis.net/)
* [Microsoft TechNet-Bibliothek: Windows Server](/windows-server/windows-server-versions)
