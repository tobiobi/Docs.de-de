---
title: "Razor-Syntaxverweis für ASP.NET Core"
author: rick-anderson
description: Informationen zur Razor-Markupsyntax zum Einbetten von serverbasiertem Code in Webseiten
manager: wpickett
ms.author: riande
ms.date: 10/18/2017
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: mvc/views/razor
ms.openlocfilehash: 98021cc76555f0c1402764c845471a4730b01b20
ms.sourcegitcommit: a510f38930abc84c4b302029d019a34dfe76823b
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/30/2018
---
# <a name="razor-syntax-for-aspnet-core"></a>Razor-Syntax für ASP.NET Core

Von [Rick Anderson](https://twitter.com/RickAndMSFT), [Luke Latham](https://github.com/guardrex), [Taylor Mullen](https://twitter.com/ntaylormullen) und [Dan Vicarel](https://github.com/Rabadash8820)

Razor stellt eine Markupsyntax zum Einbetten von serverbasiertem Code in Webseiten dar. Die Razor-Syntax besteht aus dem Razor-Markup, C# und HTML. Dateien, die Razor enthalten, besitzen in der Regel die Dateierweiterung *.cshtml*.

## <a name="rendering-html"></a>Rendern von HTML

Die Standardsprache für Razor ist HTML. Das Rendern von HTML-Code aus einem Razor-Markup funktioniert genauso wie das Rendern von HTML aus einer HTML-Datei. Ein HTML-Markup wird in Razor-Dateien mit der Erweiterung *.cshtml* vom Server unverändert gerendert.

## <a name="razor-syntax"></a>Razor-Syntax

Razor unterstützt C# und verwendet für den Übergang von HTML zu C# das `@`-Symbol. Razor wertet C#-Ausdrücke aus und rendert sie in der HTML-Ausgabe.

Folgt auf ein `@`-Symbol ein für [Razor reserviertes Schlüsselwort](#razor-reserved-keywords), wechselt es in ein Razor-spezifisches Markup. Andernfalls geht es in reines C# über.

Verwenden Sie ein zweites `@`-Symbol, um im Razor-Markup ein `@`-Symbol mit Escapezeichen zu versehen:

```cshtml
<p>@@Username</p>
```

Der Code wird in HTML mit einem einzelnen `@`-Symbol gerendert:

```html
<p>@Username</p>
```

Bei HTML-Attributen und Inhalt mit E-Mail-Adressen wird das `@`-Symbol nicht als ein Übergangszeichen behandelt. Die E-Mail-Adressen im folgenden Beispiel bleiben von der Razor-Analyse unverändert:

```cshtml
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

## <a name="implicit-razor-expressions"></a>Implizite Razor-Ausdrücke

Implizite Razor-Ausdrücke beginnen mit `@` gefolgt von C#-Code:

```cshtml
<p>@DateTime.Now</p>
<p>@DateTime.IsLeapYear(2016)</p>
```

Mit Ausnahme des `await`-C#-Schlüsselworts dürfen implizite Ausdrücke keine Leerzeichen enthalten. Wird die C#-Anweisung eindeutig beendet, können auch Leerzeichen verwendet werden:

```cshtml
<p>@await DoSomething("hello", "world")</p>
```

Implizite Ausdrücke **dürfen keine** C#-Generika enthalten, da die Zeichen innerhalb der Klammern (`<>`) als HTML-Tag interpretiert werden. Der folgende Code ist **ungültig**:

```cshtml
<p>@GenericMethod<int>()</p>
```

Der vorhergehende Code erzeugt einen Compilerfehler, der folgendermaßen aussehen kann:

 * Das Element „int“ wurde nicht geschlossen. Alle Elemente müssen entweder selbstschließend sein oder einen übereinstimmenden Endtag besitzen.
 *  Die Methodengruppe „GenericMethod“ kann nicht in den Nichtdelegattyp „Objekt“ konvertiert werden. Wollten Sie die Methode aufrufen? 
 
Generische Methodenaufrufe müssen von einem [expliziten Razor-Ausdruck](#explicit-razor-expressions) oder einem [Razor-Codeblock](#razor-code-blocks) umschlossen werden.

## <a name="explicit-razor-expressions"></a>Explizite Razor-Ausdrücke

Explizite Razor-Ausdrücke bestehen aus einem `@`-Symbol mit ausgeglichener Klammer. Zum Rendern der Uhrzeit von letzter Woche wird folgendes Razor-Markup verwendet:

```cshtml
<p>Last week this time: @(DateTime.Now - TimeSpan.FromDays(7))</p>
```

Jeglicher Inhalt innerhalb der `@()`-Klammer wird ausgewertet und in der Ausgabe gerendert.

Die im vorherigen Abschnitt beschriebenen impliziten Ausdrücke dürfen in der Regel keine Leerzeichen enthalten. Im folgenden Code wird eine Woche nicht von der aktuellen Uhrzeit abgezogen:

[!code-cshtml[Main](razor/sample/Views/Home/Contact.cshtml?range=17)]

Der Code rendert den folgenden HTML-Code:

```html
<p>Last week: 7/7/2016 4:39:52 PM - TimeSpan.FromDays(7)</p>
```

Explizite Ausdrücke können zum Verketten von Text mit einem Ergebnis des Ausdrucks verwendet werden:

```cshtml
@{
    var joe = new Person("Joe", 33);
}

<p>Age@(joe.Age)</p>
```

Ohne den expliziten Ausdruck wird `<p>Age@joe.Age</p>` als E-Mail-Adresse behandelt und `<p>Age@joe.Age</p>` gerendert. `<p>Age33</p>` wird gerendert, wenn es als expliziter Ausdruck geschrieben wird.


Explizite Ausdrücke können zum Rendern der Ausgabe von generischen Methoden in *.cshtml*-Dateien verwendet werden. Bei impliziten Ausdrücken werden die Zeichen innerhalb der Klammern (`<>`) als HTML-Tag interpretiert. Das folgende Markup ist **keine gültige** Razor-Schreibweise:

```cshtml
<p>@GenericMethod<int>()</p>
```

Der vorhergehende Code erzeugt einen Compilerfehler, der folgendermaßen aussehen kann:

 * Das Element „int“ wurde nicht geschlossen. Alle Elemente müssen entweder selbstschließend sein oder einen übereinstimmenden Endtag besitzen.
 *  Die Methodengruppe „GenericMethod“ kann nicht in den Nichtdelegattyp „Objekt“ konvertiert werden. Wollten Sie die Methode aufrufen? 
 
 Das folgende Markup zeigt die korrekte Schreibweise dieses Codes. Der Code wird als expliziter Ausdruck geschrieben:

```cshtml
<p>@(GenericMethod<int>())</p>
```

## <a name="expression-encoding"></a>Codieren von Ausdrücken

C#-Ausdrücke, die mit einer Zeichenfolge ausgewertet werden, werden HTML-codiert. C#-Ausdrücke, die mit `IHtmlContent` ausgewertet werden, werden direkt durch `IHtmlContent.WriteTo` gerendert. C#-Ausdrücke, die nicht mit `IHtmlContent` ausgewertet werden, werden durch `ToString` in eine Zeichenfolge konvertiert und codiert, bevor sie gerendert werden.

```cshtml
@("<span>Hello World</span>")
```

Der Code rendert den folgenden HTML-Code:

```html
&lt;span&gt;Hello World&lt;/span&gt;
```

Der HTML-Code wird im Browser folgendermaßen angezeigt:

```
<span>Hello World</span>
```

Die Ausgabe `HtmlHelper.Raw` wird nicht codiert, sondern als HTML-Markup gerendert.

> [!WARNING]
> Die Verwendung von `HtmlHelper.Raw` bei einer nicht bereinigten Benutzereingabe stellt ein Sicherheitsrisiko dar. Benutzereingaben können schädlichen JavaScript-Code oder andere Sicherheitsrisiken enthalten. Das Bereinigen von Benutzereingaben ist schwierig. Vermeiden Sie daher die Verwendung von `HtmlHelper.Raw` bei Benutzereingaben.

```cshtml
@Html.Raw("<span>Hello World</span>")
```

Der Code rendert den folgenden HTML-Code:

```html
<span>Hello World</span>
```

## <a name="razor-code-blocks"></a>Razor-Codeblöcke

Razor-Codeblöcke beginnen mit `@` und sind von `{}` eingeschlossen. Im Gegensatz zu Ausdrücken wird C#-Code in Codeblöcken nicht gerendert. Codeblöcke und Ausdrücke in einer Ansicht nutzen den gleichen Bereich und werden der Reihe nach definiert:

```cshtml
@{
    var quote = "The future depends on what you do today. - Mahatma Gandhi";
}

<p>@quote</p>

@{
    quote = "Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.";
}

<p>@quote</p>
```

Der Code rendert den folgenden HTML-Code:

```html
<p>The future depends on what you do today. - Mahatma Gandhi</p>
<p>Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.</p>
```

### <a name="implicit-transitions"></a>Implizite Übergänge

Die Standardsprache in einem Codeblock ist C#. Die Razor-Seite kann jedoch zurück zu HTML wechseln:

```cshtml
@{
    var inCSharp = true;
    <p>Now in HTML, was in C# @inCSharp</p>
}
```

### <a name="explicit-delimited-transition"></a>Durch Trennzeichen getrennte explizite Übergänge

Soll ein Unterabschnitt eines Codeblocks in HTML gerendert werden, umschließen Sie die Zeichen, die gerendert werden sollen, mit dem Razor-Tag **\<text>**:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <text>Name: @person.Name</text>
}
```

Verwenden Sie diese Methode zum Rendern von HTML-Code, der nicht von einem HTML-Tag umschlossen ist. Ohne HTML- oder Razor-Tag tritt ein Razor-Laufzeitfehler auf.

Der **\<text>**-Tag ist nützlich, wenn Sie beim Rendern von Inhalt Leerzeichen steuern möchten:

* Nur der Inhalt zwischen dem **\<text>**-Tag wird gerendert. 
* In der HTML-Ausgabe werden keine Leerzeichen vor oder nach dem **\<text>**-Tag angezeigt.

### <a name="explicit-line-transition-with-"></a>Explizite Zeilenübergänge mit @:

Verwenden Sie die `@:`-Syntax, um den Rest einer kompletten Zeile als HTML-Code in einem Codeblock zu rendern:

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    @:Name: @person.Name
}
```

Ohne das `@:`-Symbol im Code wird ein Razor-Laufzeitfehler erzeugt.

Warnung: Zusätzliche `@`-Zeichen in einer Razor-Datei können zu Compilerfehlern bei späteren Anweisungen im Block führen. Diese Compilerfehler können dann schwer nachvollziehbar sein, da der tatsächliche Fehler vor dem gemeldeten Fehler auftritt. Dieser Fehler tritt häufig auf, wenn mehrere implizite/explizite Ausdrücke in einem einzigen Codeblock kombiniert werden.

## <a name="control-structures"></a>Steuerungsstrukturen

Steuerungsstrukturen sind eine Erweiterung von Codeblöcken. Alle Aspekte von Codeblöcken (Übergang zu Markup, Inline-C#) gelten auch für die folgenden Strukturen:

### <a name="conditionals-if-else-if-else-and-switch"></a>Die Bedingungen @if, else if, else und @switch

`@if` steuert, wenn der Code ausgeführt wird:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
```

`else` und `else if` erfordern kein `@`-Symbol:

```cshtml
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
else if (value >= 1337)
{
    <p>The value is large.</p>
}
else
{
    <p>The value is odd and small.</p>
}
```

Im folgenden Markup wird die Verwendung einer switch-Anweisung veranschaulicht:

```cshtml
@switch (value)
{
    case 1:
        <p>The value is 1!</p>
        break;
    case 1337:
        <p>Your number is 1337!</p>
        break;
    default:
        <p>Your number wasn't 1 or 1337.</p>
        break;
}
```

### <a name="looping-for-foreach-while-and-do-while"></a>Die Schleifen @for, @foreach, @while und @do while

Auf Vorlagen basierender HTML-Code kann mit Schleifen-Steuerungsanweisungen gerendert werden. Zum Rendern einer Liste von Personen:

```cshtml
@{
    var people = new Person[]
    {
          new Person("Weston", 33),
          new Person("Johnathon", 41),
          ...
    };
}
```

Die folgenden Schleifenanweisungen werden unterstützt:

`@for`

```cshtml
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@foreach`

```cshtml
@foreach (var person in people)
{
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>
}
```

`@while`

```cshtml
@{ var i = 0; }
@while (i < people.Length)
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
}
```

`@do while`

```cshtml
@{ var i = 0; }
@do
{
    var person = people[i];
    <p>Name: @person.Name</p>
    <p>Age: @person.Age</p>

    i++;
} while (i < people.Length);
```

### <a name="compound-using"></a>Zusammengesetztes @using

In C# wird eine `using`-Anweisung verwendet, um sicherzustellen, dass ein Objekt freigegeben wurde. In Razor wird derselbe Mechanismus verwendet, um HTML-Hilfsprogramme zu erstellen, die zusätzliche Inhalte enthalten. Im folgenden Code wird ein Formulartag mit der `@using`-Anweisung durch HTML-Hilfsprogramme gerendert:


```cshtml
@using (Html.BeginForm())
{
    <div>
        email:
        <input type="email" id="Email" value="">
        <button>Register</button>
    </div>
}
```

Aktionen auf Bereichsebene können mit [Taghilfsprogrammen](xref:mvc/views/tag-helpers/intro) ausgeführt werden.

### <a name="try-catch-finally"></a>@try, catch, finally

Die Behandlung von Ausnahmen ist vergleichbar mit C#:

[!code-cshtml[Main](razor/sample/Views/Home/Contact7.cshtml)]

### <a name="lock"></a>@lock

Razor kann kritische Abschnitte mit lock-Anweisungen schützen:

```cshtml
@lock (SomeLock)
{
    // Do critical section work
}
```

### <a name="comments"></a>Kommentare

Razor unterstützt C#- und HTML-Kommentare:

```cshtml
@{
    /* C# comment */
    // Another C# comment
}
<!-- HTML comment -->
```

Der Code rendert den folgenden HTML-Code:

```html
<!-- HTML comment -->
```

Razor-Kommentare werden vom Server entfernt, bevor die Webseite gerendert wird. Zur Abgrenzung der Kommentare verwendet Razor `@*  *@`. Der folgende Code ist auskommentiert, damit vom Server kein Markup gerendert wird:

```cshtml
@*
    @{
        /* C# comment */
        // Another C# comment
    }
    <!-- HTML comment -->
*@
```

## <a name="directives"></a>Anweisungen

Razor-Anweisungen werden durch implizite Ausdrücke mit reservierten Schlüsselwörtern nach dem `@`-Symbol dargestellt. Eine Anweisung ändert in der Regel die Analyse einer Ansicht oder aktiviert unterschiedliche Funktionen.

Wenn Sie wissen, wie von Razor Code für eine Ansicht generiert wird, erleichtert dies das Verständnis dafür, wie Anweisungen funktionieren.

[!code-html[Main](razor/sample/Views/Home/Contact8.cshtml)]

Durch den Code wird eine Klasse ähnlich der folgenden Klasse generiert:

```csharp
public class _Views_Something_cshtml : RazorPage<dynamic>
{
    public override async Task ExecuteAsync()
    {
        var output = "Getting old ain't for wimps! - Anonymous";

        WriteLiteral("/r/n<div>Quote of the Day: ");
        Write(output);
        WriteLiteral("</div>");
    }
}
```

Im Abschnitt [Anzeigen der Razor-C#-Klasse, die für eine Ansicht generiert wurde](#viewing-the-razor-c-class-generated-for-a-view) weiter unten wird erklärt, wie die generierte Klasse angezeigt wird.

### <a name="using"></a>@using

Die `@using`-Anweisung fügt die `using`-C#-Anweisung der generierten Ansicht hinzu:

[!code-cshtml[Main](razor/sample/Views/Home/Contact9.cshtml)]

### <a name="model"></a>@model

Die `@model`-Anweisung gibt den Typ des Modells an, das an eine Ansicht übergeben wird:

```cshtml
@model TypeNameOfModel
```

In einer mit einzelnen Benutzerkonten erstellten ASP.NET Core MVC-App enthält die Ansicht *Views/Account/Login.cshtml* die folgende Modelldeklaration:

```cshtml
@model LoginViewModel
```

Die generierte Klasse erbt von `RazorPage<dynamic>`:

```csharp
public class _Views_Account_Login_cshtml : RazorPage<LoginViewModel>
```

Razor macht eine `Model`-Eigenschaft für den Zugriff auf das an die Ansicht übergebene Modell verfügbar:

```cshtml
<div>The Login Email: @Model.Email</div>
```

Die `@model`-Anweisung gibt den Typ dieser Eigenschaft an. Die Anweisung gibt das `T` in `RazorPage<T>` der generierten Klasse an, von der die Ansicht abgeleitet wird. Wird die `@model`-Anweisung nicht angegeben, ist die `Model`-Eigenschaft vom Typ `dynamic`. Der Wert des Modells wird vom Controller an die Ansicht übergeben. Weitere Informationen finden Sie unter „Stark typisierte Modelle und das Schlüsselwort @model“.

### <a name="inherits"></a>@inherits

Die `@inherits`-Anweisung bietet uneingeschränkten Zugriff auf die Klasse, die die Ansicht erbt:

```cshtml
@inherits TypeNameOfClassToInheritFrom
```

Der folgende Code ist ein benutzerdefinierter Typ der Razor-Seite:

[!code-csharp[Main](razor/sample/Classes/CustomRazorPage.cs)]

`CustomText` wird in einer Ansicht angezeigt:

[!code-cshtml[Main](razor/sample/Views/Home/Contact10.cshtml)]

Der Code rendert den folgenden HTML-Code:

```html
<div>Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping a slop bucket on the street below.</div>
```

 `@model` und `@inherits` können in derselben Ansicht verwendet werden. `@inherits` kann in einer *_ViewImports.cshtml*-Datei verwendet werden, die von der Ansicht importiert wird:

[!code-cshtml[Main](razor/sample/Views/_ViewImportsModel.cshtml)]

Der folgende Code ist ein Beispiel für eine stark typisierte Ansicht:

[!code-cshtml[Main](razor/sample/Views/Home/Login1.cshtml)]

Wird „rick@contoso.com“ im Modell übergeben, generiert die Ansicht das folgende HTML-Markup:

```html
<div>The Login Email: rick@contoso.com</div>
<div>Custom text: Gardyloo! - A Scottish warning yelled from a window before dumping a slop bucket on the street below.</div>
```

### <a name="inject"></a>@inject


Mit der `@inject`-Anweisung kann die Razor-Seite einen Dienst vom [Dienstcontainer](xref:fundamentals/dependency-injection) in eine Ansicht einfügen. Weitere Informationen finden Sie unter [Dependency Injection in Ansichten](xref:mvc/views/dependency-injection).

### <a name="functions"></a>@functions

Mit der `@functions`-Anweisung kann eine Razor-Seite einer Ansicht Inhalt auf Funktionsebene hinzuzufügen:

```cshtml
@functions { // C# Code }
```

Zum Beispiel:

[!code-cshtml[Main](razor/sample/Views/Home/Contact6.cshtml)]

Der Code generiert das folgende HTML-Markup:

```html
<div>From method: Hello</div>
```

Der folgende Code wird in der Razor-C#-Klasse generiert:

[!code-csharp[Main](razor/sample/Classes/Views_Home_Test_cshtml.cs?range=1-19)]

### <a name="section"></a>@section

Die `@section`-Anweisung wird in Verbindung mit dem [Layout](xref:mvc/views/layout) verwendet, damit Ansichten Inhalt in verschiedenen Teilen der HTML-Seite rendern können. Weitere Informationen finden Sie unter [Abschnitte](xref:mvc/views/layout#layout-sections-label).

## <a name="tag-helpers"></a>Taghilfsprogramme

Die folgenden drei Anweisungen gehören zu den [Taghilfsprogrammen](xref:mvc/views/tag-helpers/intro).

| Direktive | Funktion |
| --------- | -------- |
| [@addTagHelper](xref:mvc/views/tag-helpers/intro#add-helper-label) | Macht Taghilfsprogramme für eine Ansicht verfügbar. |
| [@removeTagHelper](xref:mvc/views/tag-helpers/intro#remove-razor-directives-label) | Entfernt die zuvor aus einer Ansicht hinzugefügten Taghilfsprogramme. |
| [@tagHelperPrefix](xref:mvc/views/tag-helpers/intro#prefix-razor-directives-label) | Gibt ein Tagpräfix an, um Unterstützung für Taghilfsprogramme zu aktivieren und ihre Verwendung explizit festzulegen. |

## <a name="razor-reserved-keywords"></a>Für Razor reservierte Schlüsselwörter

### <a name="razor-keywords"></a>Razor-Schlüsselwörter

* page (ASP.NET Core 2.0 und höher)
* -Funktionen
* inherits
* Modell
* section
* helper (wird derzeit nicht von ASP.NET Core unterstützt)

Razor-Schlüsselwörter werden mit dem Escapezeichen `@(Razor Keyword)` versehen (z.B. `@(functions)`).

### <a name="c-razor-keywords"></a>Razor-C#-Schlüsselwörter

* Groß- und Kleinschreibung
* do
* default
* for
* foreach
* if
* else
* lock
* switch
* try
* catch
* finally
* Verwenden
* while

Razor-C#-Schlüsselwörter werden mit dem doppeltem Escapezeichen `@(@C# Razor Keyword)` versehen (z.B. `@(@case)`). Das erste `@` dient als Escapezeichen für den Razor-Parser. Das zweite `@` dient als Escapezeichen für den C#-Parser.

### <a name="reserved-keywords-not-used-by-razor"></a>Reservierte Schlüsselwörter, die nicht von Razor verwendet werden

* namespace
* Klasse

## <a name="viewing-the-razor-c-class-generated-for-a-view"></a>Anzeigen der Razor-C#-Klasse, die für eine Ansicht generiert wurde

Fügen Sie dem ASP.NET Core MVC-Projekt die folgende Klasse hinzu:

[!code-csharp[Main](razor/sample/Utilities/CustomTemplateEngine.cs)]

Überschreiben Sie das von MVC hinzugefügte `RazorTemplateEngine` mit der `CustomTemplateEngine`-Klasse:

[!code-csharp[Main](razor/sample/Startup.cs?highlight=4&range=10-14)]

Setzen Sie auf der `return csharpDocument`-Anweisung von `CustomTemplateEngine` einen Haltepunkt. Wenn die Ausführung des Programms am Haltepunkt angehalten wird, können Sie den Wert von `generatedCode` anzeigen.

![Ansicht „Text-Schnellansicht“ von „generatedCode“](razor/_static/tvr.png)

## <a name="view-lookups-and-case-sensitivity"></a>Ansicht der Suchvorgänge und Groß-/Kleinschreibung

Die Razor-Ansichtsengine führt für Ansichten Suchvorgänge aus, die Groß- und Kleinschreibung berücksichtigen. Der tatsächliche Suchvorgang wird jedoch vom zugrunde liegenden Dateisystem bestimmt:

* Dateibasierte Quelle: 
  * Bei Betriebssystemen, die Dateisysteme ohne Berücksichtigung von Groß-/Kleinschreibung verwenden (z.B. Windows), wird bei Suchvorgängen nach physischen Dateianbietern die Groß- und Kleinschreibung nicht berücksichtigt. `return View("Test")` liefert beispielsweise eine Übereinstimmung für */Views/Home/Test.cshtml*, */Views/home/test.cshtml* sowie für jede andere Schreibweise.
  * Bei Dateisystemen, die Groß-/Kleinschreibung berücksichtigen (z.B. Linux, OSX sowie mit `EmbeddedFileProvider`), wird auch bei Suchvorgängen die Groß- und Kleinschreibung berücksichtigt. `return View("Test")` liefert beispielsweise speziell eine Übereinstimmung mit */Views/Home/Test.cshtml*.
* Vorkompilierte Ansichten: Ab ASP.NET Core 2.0 wird bei der Suche nach vorkompilierten Ansichten unter allen Betriebssystemen die Groß- und Kleinschreibung nicht berücksichtigt. Das Verhalten ist mit dem Verhalten des physischen Dateianbieters unter Windows identisch. Unterscheiden sich zwei vorkompilierte Ansichten nur in der Groß-/Kleinschreibung, ist das Ergebnis der Suche nicht deterministisch.

Entwicklern wird empfohlen, sich bei der Groß-/Kleinschreibung von Datei- und Verzeichnisnamen an der Schreibweise folgender Begriffe zu orientieren:

    * Bereichs-, Controller- und Aktionsnamen 
    * Razor-Seiten
    
Mit einer übereinstimmenden Groß- und Kleinschreibung wird sichergestellt, dass die entsprechenden Ansichten für die Bereitstellungen unabhängig von dem zugrunde liegenden Dateisystem gefunden werden.
