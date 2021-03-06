---
uid: web-forms/overview/ajax-control-toolkit/confirmbutton/using-a-confirmbutton-in-a-repeater-cs
title: Verwenden eine ConfirmButton In ein Repeater (c#) | Microsoft Docs
author: wenz
description: Der Extender ConfirmButton im AJAX Control Toolkit erstellt eine Ja/keine Popupfenster, wenn der Benutzer auf eine Schaltfläche klickt (einschließlich LinkButton-Steuerelement). Es ist Ja nur, wenn...
ms.author: aspnetcontent
manager: wpickett
ms.date: 06/02/2008
ms.topic: article
ms.assetid: a973ed3e-400c-4925-ace2-0b086b479301
ms.technology: dotnet-webforms
ms.prod: .net-framework
msc.legacyurl: /web-forms/overview/ajax-control-toolkit/confirmbutton/using-a-confirmbutton-in-a-repeater-cs
msc.type: authoredcontent
ms.openlocfilehash: 7a0717083a3c1e285f0c8c4cb8503d2bbe42153b
ms.sourcegitcommit: f8852267f463b62d7f975e56bea9aa3f68fbbdeb
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 04/06/2018
---
<a name="using-a-confirmbutton-in-a-repeater-c"></a>Verwenden eine ConfirmButton In ein Repeater (c#)
====================
durch [Christian Wenz](https://github.com/wenz)

[Herunterladen von Code](http://download.microsoft.com/download/8/6/d/86dea6c6-bb92-4fa6-aa14-f8c0f82100f5/ConfirmButton1.cs.zip) oder [PDF herunterladen](http://download.microsoft.com/download/b/6/a/b6ae89ee-df69-4c87-9bfb-ad1eb2b23373/confirmbutton1CS.pdf)

> Der Extender ConfirmButton im AJAX Control Toolkit erstellt eine Ja/keine Popupfenster, wenn der Benutzer auf eine Schaltfläche klickt (einschließlich LinkButton-Steuerelement). Nur wird Wenn Ja geklickt wird, die Schaltfläche Aktion ausgeführt, andernfalls abgebrochen. Dies ist auch möglich, in ein Repeater.


## <a name="overview"></a>Übersicht

Der Extender ConfirmButton im AJAX Control Toolkit erstellt eine Ja/keine Popupfenster, wenn der Benutzer auf eine Schaltfläche klickt (einschließlich LinkButton-Steuerelement). Nur wird Wenn Ja geklickt wird, die Schaltfläche Aktion ausgeführt, andernfalls abgebrochen. Dies ist auch möglich, in ein Repeater.

## <a name="steps"></a>Schritte

Erstens ist eine Datenquelle erforderlich. Dieses Beispiel verwendet die AdventureWorks-Datenbank und die Microsoft SQL Server 2005 Express Edition. Die Datenbank ist ein optionaler Teil einer Visual Studio-Installation (einschließlich express Edition) und steht auch als separater Download unter [ https://go.microsoft.com/fwlink/?LinkId=64064 ](https://go.microsoft.com/fwlink/?LinkId=64064). Die AdventureWorks-Datenbank ist Teil der SQL Server 2005 Samples and Sample Databases (download [ https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e &amp;DisplayLang = En](https://www.microsoft.com/downloads/details.aspx?FamilyID=e719ecf7-9f46-4312-af89-6ad8702e4e6e&amp;DisplayLang=en)). Die einfachste Möglichkeit zum Einrichten der Datenbank ist die Verwendung der Microsoft SQL Server Management Studio Express ([https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang = En](https://www.microsoft.com/downloads/details.aspx?FamilyID=c243a5ae-4bd1-4e3d-94b8-5a0f62bf7796&amp;DisplayLang=en)), und fügen die `AdventureWorks.mdf` Datenbankdatei.

In diesem Beispiel gehen wir davon aus, dass die Instanz von SQL Server 2005 Express Edition aufgerufen wird `SQLEXPRESS` und befindet sich auf dem gleichen Computer wie der Webserver; Dies ist auch die Standardeinstellungen. Wenn Ihr Setup abweicht, müssen Sie passen Sie die Verbindungsinformationen für die Datenbank.

Um die Funktionalität von ASP.NET AJAX und das Steuerelement-Toolkit aktivieren die `ScriptManager` Steuerelement an einer beliebigen Stelle auf der Seite versetzt werden muss (jedoch innerhalb der `<form>` Element):

[!code-aspx[Main](using-a-confirmbutton-in-a-repeater-cs/samples/sample1.aspx)]

Klicken Sie dann, ist eine Datenquelle erforderlich. Der Einfachheit halber werden nur die ersten fünf Einträge in 'AdventureWorks Lieferanten Tabelle abgerufen. Beachten Sie, dass bei Verwendung des Visual Studio-Assistenten zum Erstellen der Datenquelle, den Tabellennamen (`Vendors`) wird derzeit nicht ordnungsgemäß mit dem Präfix `Purchasing`. Das folgende Markup ist die richtige PIN:

[!code-aspx[Main](using-a-confirmbutton-in-a-repeater-cs/samples/sample2.aspx)]

Diese Datenquelle kann dann in ein Repeater verwendet werden. Wie üblich die `DataBinder.Eval()` Methode ruft Daten aus der Datenquelle ab. Die `ConfirmButtonExtender` Steuerelement muss dann platziert werden, innerhalb der `<ItemTemplate>` Abschnitt des wiederholungsmoduls, damit er für jeden Eintrag in der Datenquelle angezeigt wird.

[!code-aspx[Main](using-a-confirmbutton-in-a-repeater-cs/samples/sample3.aspx)]


[![Die Schaltfläche "bestätigen" wird neben jeder Eintrag aus der Datenquelle angezeigt.](using-a-confirmbutton-in-a-repeater-cs/_static/image2.png)](using-a-confirmbutton-in-a-repeater-cs/_static/image1.png)

Die Schaltfläche "bestätigen" wird neben jeder Eintrag aus der Datenquelle angezeigt ([klicken Sie hier, um das Bild in voller Größe angezeigt](using-a-confirmbutton-in-a-repeater-cs/_static/image3.png))

> [!div class="step-by-step"]
> [Nächste](using-a-confirmbutton-in-a-repeater-vb.md)
