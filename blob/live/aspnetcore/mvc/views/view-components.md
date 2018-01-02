---
title: Anzeigen von Komponenten
author: rick-anderson
description: "Anzeigen von Komponenten dienen an einer beliebigen Stelle, dass Sie wiederverwendbare Renderinglogik verfügen."
keywords: ASP.NET Core, Anzeigen von Komponenten, Teilansicht
ms.author: riande
manager: wpickett
ms.date: 02/14/2017
ms.topic: article
ms.assetid: ab4705b7-59d7-4f31-bc97-ea7f292fe926
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/views/view-components
ms.openlocfilehash: 2cf82df78c250cdfdd808d49acfc06dc2ea82f5f
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
# <a name="view-components"></a><span data-ttu-id="47336-104">Anzeigen von Komponenten</span><span class="sxs-lookup"><span data-stu-id="47336-104">View components</span></span>

<span data-ttu-id="47336-105">Von [Rick Anderson](https://twitter.com/RickAndMSFT)</span><span class="sxs-lookup"><span data-stu-id="47336-105">By [Rick Anderson](https://twitter.com/RickAndMSFT)</span></span>

<span data-ttu-id="47336-106">[Anzeigen oder Herunterladen von Beispielcode](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/views/view-components/sample) ([Vorgehensweise zum Herunterladen](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="47336-106">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/views/view-components/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

## <a name="introducing-view-components"></a><span data-ttu-id="47336-107">Einführung zum Anzeigen von Komponenten</span><span class="sxs-lookup"><span data-stu-id="47336-107">Introducing view components</span></span>

<span data-ttu-id="47336-108">Neue zu ASP.NET Core MVC, ähneln sich Komponenten anzeigen, Teilansichten, aber sie sind deutlich leistungsfähiger.</span><span class="sxs-lookup"><span data-stu-id="47336-108">New to ASP.NET Core MVC, view components are similar to partial views, but they are much more powerful.</span></span> <span data-ttu-id="47336-109">Anzeigen von Komponenten nicht wurden die modellbindung verwenden und nur richten sich nach den Daten, die Sie angeben, wenn er aufgerufen.</span><span class="sxs-lookup"><span data-stu-id="47336-109">View components don’t use model binding, and only depend on the data you provide when calling into it.</span></span> <span data-ttu-id="47336-110">Eine Komponente anzeigen:</span><span class="sxs-lookup"><span data-stu-id="47336-110">A view component:</span></span>

* <span data-ttu-id="47336-111">Rendert ein Block, statt eine gesamte Antwort</span><span class="sxs-lookup"><span data-stu-id="47336-111">Renders a chunk rather than a whole response</span></span>
* <span data-ttu-id="47336-112">Enthält die gleichen Trennung von Anliegen und die Prüfbarkeit Vorteile, die zwischen einem Controller und Ansicht gefunden</span><span class="sxs-lookup"><span data-stu-id="47336-112">Includes the same separation-of-concerns and testability benefits found between a controller and view</span></span>
* <span data-ttu-id="47336-113">Parameter und Geschäftslogik können haben</span><span class="sxs-lookup"><span data-stu-id="47336-113">Can have parameters and business logic</span></span>
* <span data-ttu-id="47336-114">Wird normalerweise aus einer Layoutseite aufgerufen.</span><span class="sxs-lookup"><span data-stu-id="47336-114">Is typically invoked from a layout page</span></span>

<span data-ttu-id="47336-115">Anzeigen von Komponenten dienen an einer beliebigen Stelle, dass Sie wiederverwendbare Renderinglogik verfügen, z. B. zu komplex für eine Teilansicht ist:</span><span class="sxs-lookup"><span data-stu-id="47336-115">View components are intended anywhere you have reusable rendering logic that is too complex for a partial view, such as:</span></span>

* <span data-ttu-id="47336-116">Dynamische Navigationsmenüs</span><span class="sxs-lookup"><span data-stu-id="47336-116">Dynamic navigation menus</span></span>
* <span data-ttu-id="47336-117">Tagcloud (, in dem sie die Datenbank (Abfragen)</span><span class="sxs-lookup"><span data-stu-id="47336-117">Tag cloud (where it queries the database)</span></span>
* <span data-ttu-id="47336-118">Login-Bereich</span><span class="sxs-lookup"><span data-stu-id="47336-118">Login panel</span></span>
* <span data-ttu-id="47336-119">Einkaufswagen</span><span class="sxs-lookup"><span data-stu-id="47336-119">Shopping cart</span></span>
* <span data-ttu-id="47336-120">Kürzlich veröffentlichten Artikeln</span><span class="sxs-lookup"><span data-stu-id="47336-120">Recently published articles</span></span>
* <span data-ttu-id="47336-121">Randleiste Inhalte auf einem typischen blog</span><span class="sxs-lookup"><span data-stu-id="47336-121">Sidebar content on a typical blog</span></span>
* <span data-ttu-id="47336-122">Ein Anmeldefenster, die auf jeder Seite gerendert und entweder die Links, melden Sie sich ab, oder melden Sie sich, abhängig von das Protokoll in der Status des Benutzers anzeigen</span><span class="sxs-lookup"><span data-stu-id="47336-122">A login panel that would be rendered on every page and show either the links to log out or log in, depending on the log in state of the user</span></span>

<span data-ttu-id="47336-123">Eine Ansichtskomponente besteht aus zwei Teilen: der-Klasse (normalerweise abgeleitet aus [ViewComponent](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.viewcomponent)) und das Ergebnis gibt (in der Regel eine Ansicht) zurück.</span><span class="sxs-lookup"><span data-stu-id="47336-123">A view component consists of two parts: the class (typically derived from [ViewComponent](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.mvc.viewcomponent)) and the result it returns (typically a view).</span></span> <span data-ttu-id="47336-124">Wie Domänencontroller, eine Ansichtskomponente auf einer POCO werden kann, aber die meisten Entwickler sollten die verfügbaren Methoden und Eigenschaften nutzen durch Ableiten von `ViewComponent`.</span><span class="sxs-lookup"><span data-stu-id="47336-124">Like controllers, a view component can be a POCO, but most developers will want to take advantage of the methods and properties available by deriving from `ViewComponent`.</span></span>

## <a name="creating-a-view-component"></a><span data-ttu-id="47336-125">Erstellen einer Komponente anzeigen</span><span class="sxs-lookup"><span data-stu-id="47336-125">Creating a view component</span></span>

<span data-ttu-id="47336-126">Dieser Abschnitt enthält die allgemeinen Anforderungen zum Erstellen einer Komponente anzeigen.</span><span class="sxs-lookup"><span data-stu-id="47336-126">This section contains the high-level requirements to create a view component.</span></span> <span data-ttu-id="47336-127">Später in diesem Artikel wir untersuchen Sie jeden Schritt im Detail und erstellen Sie eine Komponente anzeigen.</span><span class="sxs-lookup"><span data-stu-id="47336-127">Later in the article, we'll examine each step in detail and create a view component.</span></span>

### <a name="the-view-component-class"></a><span data-ttu-id="47336-128">Die Komponente Ansichtsklasse</span><span class="sxs-lookup"><span data-stu-id="47336-128">The view component class</span></span>

<span data-ttu-id="47336-129">Eine Komponentenklasse anzeigen kann durch eine der folgenden erstellt werden:</span><span class="sxs-lookup"><span data-stu-id="47336-129">A view component class can be created by any of the following:</span></span>

* <span data-ttu-id="47336-130">Ableiten von *ViewComponent*</span><span class="sxs-lookup"><span data-stu-id="47336-130">Deriving from *ViewComponent*</span></span>
* <span data-ttu-id="47336-131">Indem eine Klasse mit der `[ViewComponent]` Attribut oder das Ableiten einer Klasse mit der `[ViewComponent]` Attribut</span><span class="sxs-lookup"><span data-stu-id="47336-131">Decorating a class with the `[ViewComponent]` attribute, or deriving from a class with the `[ViewComponent]` attribute</span></span>
* <span data-ttu-id="47336-132">Erstellen einer Klasse, wenn der Name mit dem Suffix endet *ViewComponent*</span><span class="sxs-lookup"><span data-stu-id="47336-132">Creating a class where the name ends with the suffix *ViewComponent*</span></span>

<span data-ttu-id="47336-133">Controller Schemas müssen wie ansichtskomponenten öffentliche, nicht geschachtelten und nicht abstrakten Klassen sein.</span><span class="sxs-lookup"><span data-stu-id="47336-133">Like controllers, view components must be public, non-nested, and non-abstract classes.</span></span> <span data-ttu-id="47336-134">Der Komponentenname Ansicht wird der Klassenname mit dem "ViewComponent" Suffix entfernt.</span><span class="sxs-lookup"><span data-stu-id="47336-134">The view component name is the class name with the "ViewComponent" suffix removed.</span></span> <span data-ttu-id="47336-135">Sie können auch explizit angegeben werden mithilfe der `ViewComponentAttribute.Name` Eigenschaft.</span><span class="sxs-lookup"><span data-stu-id="47336-135">It can also be explicitly specified using the `ViewComponentAttribute.Name` property.</span></span>

<span data-ttu-id="47336-136">Eine Komponentenklasse für die Sicht:</span><span class="sxs-lookup"><span data-stu-id="47336-136">A view component class:</span></span>

* <span data-ttu-id="47336-137">Unterstützt uneingeschränkt Konstruktor [Abhängigkeitsinjektion](../../fundamentals/dependency-injection.md)</span><span class="sxs-lookup"><span data-stu-id="47336-137">Fully supports constructor [dependency injection](../../fundamentals/dependency-injection.md)</span></span>

* <span data-ttu-id="47336-138">Verwendet keinen Teil des Controller-Lebenszyklus, d. h., Sie können keine [Filter](../controllers/filters.md) in einer Ansichtskomponente</span><span class="sxs-lookup"><span data-stu-id="47336-138">Does not take part in the controller lifecycle, which means you can't use [filters](../controllers/filters.md) in a view component</span></span>

### <a name="view-component-methods"></a><span data-ttu-id="47336-139">Die Komponentenmethoden anzuzeigen</span><span class="sxs-lookup"><span data-stu-id="47336-139">View component methods</span></span>

<span data-ttu-id="47336-140">Eine Ansichtskomponente definiert ihre Logik in einer `InvokeAsync` Methode, die zurückgibt ein `IViewComponentResult`.</span><span class="sxs-lookup"><span data-stu-id="47336-140">A view component defines its logic in an `InvokeAsync` method that returns an `IViewComponentResult`.</span></span> <span data-ttu-id="47336-141">Parameter stammen direkt aus dem Aufruf der Komponente anzeigen, nicht von der modellbindung.</span><span class="sxs-lookup"><span data-stu-id="47336-141">Parameters come directly from invocation of the view component, not from model binding.</span></span> <span data-ttu-id="47336-142">Eine Ansichtskomponente nie direkt eine Anforderung behandelt.</span><span class="sxs-lookup"><span data-stu-id="47336-142">A view component never directly handles a request.</span></span> <span data-ttu-id="47336-143">In der Regel eine Ansichtskomponente Initialisiert ein Modell und übergibt sie an einer Ansicht durch Aufrufen der `View` Methode.</span><span class="sxs-lookup"><span data-stu-id="47336-143">Typically, a view component initializes a model and passes it to a view by calling the `View` method.</span></span> <span data-ttu-id="47336-144">Zeigen Sie zusammengefasst Komponentenmethoden an:</span><span class="sxs-lookup"><span data-stu-id="47336-144">In summary, view component methods:</span></span>

* <span data-ttu-id="47336-145">Definieren einer `InvokeAsync` Methode, die zurückgibt ein`IViewComponentResult`</span><span class="sxs-lookup"><span data-stu-id="47336-145">Define an `InvokeAsync` method that returns an `IViewComponentResult`</span></span>
* <span data-ttu-id="47336-146">In der Regel ein Modell initialisiert und übergibt sie an einer Ansicht durch Aufrufen der `ViewComponent` `View` Methode</span><span class="sxs-lookup"><span data-stu-id="47336-146">Typically initializes a model and passes it to a view by calling the `ViewComponent` `View` method</span></span>
* <span data-ttu-id="47336-147">Parameter aus der aufrufenden Methode nicht HTTP stammen, es erfolgt keine modellbindung</span><span class="sxs-lookup"><span data-stu-id="47336-147">Parameters come from the calling method, not HTTP, there is no model binding</span></span>
* <span data-ttu-id="47336-148">Werden direkt als ein HTTP-Endpunkt nicht erreichbar, sie werden aufgerufen, aus dem Code (in der Regel in einer Ansicht).</span><span class="sxs-lookup"><span data-stu-id="47336-148">Are not reachable directly as an HTTP endpoint, they are invoked from your code (usually in a view).</span></span> <span data-ttu-id="47336-149">Eine Ansichtskomponente behandelt nie eine Anforderung.</span><span class="sxs-lookup"><span data-stu-id="47336-149">A view component never handles a request</span></span>
* <span data-ttu-id="47336-150">Alle Details aus der aktuellen HTTP-Anforderung, anstatt die Signatur sind überladen werden.</span><span class="sxs-lookup"><span data-stu-id="47336-150">Are overloaded on the signature rather than any details from the current HTTP request</span></span>

### <a name="view-search-path"></a><span data-ttu-id="47336-151">Suchpfad für die Ansicht</span><span class="sxs-lookup"><span data-stu-id="47336-151">View search path</span></span>

<span data-ttu-id="47336-152">Die Laufzeit sucht, für die Ansicht in den folgenden Pfaden:</span><span class="sxs-lookup"><span data-stu-id="47336-152">The runtime searches for the view in the following paths:</span></span>

   * <span data-ttu-id="47336-153">Ansichten /\<Controller_name > /Components/\<View_component_name > /\<View_name ></span><span class="sxs-lookup"><span data-stu-id="47336-153">Views/\<controller_name>/Components/\<view_component_name>/\<view_name></span></span>
   * <span data-ttu-id="47336-154">Ansichten/freigegeben/Components/\<View_component_name > /\<View_name ></span><span class="sxs-lookup"><span data-stu-id="47336-154">Views/Shared/Components/\<view_component_name>/\<view_name></span></span>

<span data-ttu-id="47336-155">Der Standardname für die Sicht für eine Ansichtskomponente ist *Standard*, was bedeutet, dass die Datei wird in der Regel namens *Default.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="47336-155">The default view name for a view component is *Default*, which means your view file will typically be named *Default.cshtml*.</span></span> <span data-ttu-id="47336-156">Beim Erstellen der Komponente Ansichtsergebnis oder beim Aufrufen, geben Sie einen Namen für die andere Ansicht kann die `View` Methode.</span><span class="sxs-lookup"><span data-stu-id="47336-156">You can specify a different view name when creating the view component result or when calling the `View` method.</span></span>

<span data-ttu-id="47336-157">Es wird empfohlen, Sie benennen die Ansichtsdatei *Default.cshtml* und Verwenden der *Ansichten/freigegeben/Components/\<View_component_name > /\<View_name >* Pfad.</span><span class="sxs-lookup"><span data-stu-id="47336-157">We recommend you name the view file *Default.cshtml* and use the *Views/Shared/Components/\<view_component_name>/\<view_name>* path.</span></span> <span data-ttu-id="47336-158">Die `PriorityList` verwendet in diesem Beispiel verwendete Ansichtskomponente *Views/Shared/Components/PriorityList/Default.cshtml* für die Komponentensicht anzeigen.</span><span class="sxs-lookup"><span data-stu-id="47336-158">The `PriorityList` view component used in this sample uses *Views/Shared/Components/PriorityList/Default.cshtml* for the view component view.</span></span>

## <a name="invoking-a-view-component"></a><span data-ttu-id="47336-159">Aufrufen einer Komponente anzeigen</span><span class="sxs-lookup"><span data-stu-id="47336-159">Invoking a view component</span></span>

<span data-ttu-id="47336-160">Rufen Sie zum Verwenden der Ansichtskomponente auf Folgendes in einer Ansicht:</span><span class="sxs-lookup"><span data-stu-id="47336-160">To use the view component, call the following inside a view:</span></span>

```cshtml
@Component.InvokeAsync("Name of view component", <anonymous type containing parameters>)
```

<span data-ttu-id="47336-161">Werden die Parameter an übergeben der `InvokeAsync` Methode.</span><span class="sxs-lookup"><span data-stu-id="47336-161">The parameters will be passed to the `InvokeAsync` method.</span></span> <span data-ttu-id="47336-162">Die `PriorityList` Ansichtskomponente entwickelt, in dem Artikel aus aufgerufen wird die *Views/Todo/Index.cshtml* Datei anzeigen.</span><span class="sxs-lookup"><span data-stu-id="47336-162">The `PriorityList` view component developed in the article is invoked from the *Views/Todo/Index.cshtml* view file.</span></span> <span data-ttu-id="47336-163">Im folgenden wird die `InvokeAsync` Methode mit zwei Parametern aufgerufen wird:</span><span class="sxs-lookup"><span data-stu-id="47336-163">In the following, the `InvokeAsync` method is called with two parameters:</span></span>

[!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexFinal.cshtml?range=35)]

## <a name="invoking-a-view-component-as-a-tag-helper"></a><span data-ttu-id="47336-164">Aufrufen einer Ansichtskomponente als ein Tag-Hilfsprogramm</span><span class="sxs-lookup"><span data-stu-id="47336-164">Invoking a view component as a Tag Helper</span></span>

<span data-ttu-id="47336-165">Für ASP.NET Core 1.1 und höher können Sie eine Ansichtskomponente als Aufrufen einer [Tag Helper](xref:mvc/views/tag-helpers/intro):</span><span class="sxs-lookup"><span data-stu-id="47336-165">For ASP.NET Core 1.1 and higher, you can invoke a view component as a [Tag Helper](xref:mvc/views/tag-helpers/intro):</span></span>

[!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexTagHelper.cshtml?range=37-38)]

<span data-ttu-id="47336-166">In Pascal-Schreibweise angegeben Klassen- und Parameter für den Tag-Hilfsprogrammen übersetzt ihre [senken Kebab Groß-/Kleinschreibung](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101).</span><span class="sxs-lookup"><span data-stu-id="47336-166">Pascal-cased class and method parameters for Tag Helpers are translated into their [lower kebab case](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101).</span></span> <span data-ttu-id="47336-167">Der Tag-Hilfskomponente aufzurufenden eine Ansichtskomponente auf die `<vc></vc>` Element.</span><span class="sxs-lookup"><span data-stu-id="47336-167">The Tag Helper to invoke a view component uses the `<vc></vc>` element.</span></span> <span data-ttu-id="47336-168">Die Ansichtskomponente wird wie folgt angegeben:</span><span class="sxs-lookup"><span data-stu-id="47336-168">The view component is specified as follows:</span></span>

```cshtml
<vc:[view-component-name]
  parameter1="parameter1 value"
  parameter2="parameter2 value">
</vc:[view-component-name]>
```

<span data-ttu-id="47336-169">Hinweis: Um eine Ansichtskomponente als ein Tag-Hilfsprogramm verwenden, müssen Sie die Assembly registrieren, enthält die Ansicht mit der `@addTagHelper` Richtlinie.</span><span class="sxs-lookup"><span data-stu-id="47336-169">Note: In order to use a View Component as a Tag Helper, you must register the assembly containing the View Component using the `@addTagHelper` directive.</span></span> <span data-ttu-id="47336-170">Beispielsweise ist die View-Komponente in einer Assembly mit dem Namen "MyWebApp", die folgende Anweisung zum Hinzufügen der `_ViewImports.cshtml` Datei:</span><span class="sxs-lookup"><span data-stu-id="47336-170">For example, if your View Component is in an assembly called "MyWebApp", add the following directive to the `_ViewImports.cshtml` file:</span></span>

```cshtml
@addTagHelper *, MyWebApp
```

<span data-ttu-id="47336-171">Sie können eine Ansichtskomponente auf als ein Tag Hilfsprogramm auf einer beliebigen Datei registrieren, die die Komponente für die Sicht verweist.</span><span class="sxs-lookup"><span data-stu-id="47336-171">You can register a View Component as a Tag Helper to any file that references the View Component.</span></span> <span data-ttu-id="47336-172">Finden Sie unter [verwalten Tag Helper Bereich](xref:mvc/views/tag-helpers/intro#managing-tag-helper-scope) für Weitere Informationen zum Registrieren von Tag-Hilfsprogramme.</span><span class="sxs-lookup"><span data-stu-id="47336-172">See [Managing Tag Helper Scope](xref:mvc/views/tag-helpers/intro#managing-tag-helper-scope) for more information on how to register Tag Helpers.</span></span>

<span data-ttu-id="47336-173">Die `InvokeAsync` in diesem Lernprogramm verwendete Methode:</span><span class="sxs-lookup"><span data-stu-id="47336-173">The `InvokeAsync` method used in this tutorial:</span></span>

[!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexFinal.cshtml?range=35)]

<span data-ttu-id="47336-174">Im Hilfsprogramm-Tag Markup:</span><span class="sxs-lookup"><span data-stu-id="47336-174">In Tag Helper markup:</span></span>

[!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexTagHelper.cshtml?range=37-38)]

<span data-ttu-id="47336-175">Im obigen Beispiel die `PriorityList` Ansichtskomponente wird `priority-list`.</span><span class="sxs-lookup"><span data-stu-id="47336-175">In the sample above, the `PriorityList` view component becomes `priority-list`.</span></span> <span data-ttu-id="47336-176">Die Parameter der Komponente anzeigen, werden als Attribute in Kleinbuchstaben Kebab übergeben.</span><span class="sxs-lookup"><span data-stu-id="47336-176">The parameters to the view component are passed as attributes in lower kebab case.</span></span>

### <a name="invoking-a-view-component-directly-from-a-controller"></a><span data-ttu-id="47336-177">Aufrufen einer Ansichtskomponente direkt von einem controller</span><span class="sxs-lookup"><span data-stu-id="47336-177">Invoking a view component directly from a controller</span></span>

<span data-ttu-id="47336-178">Anzeigen von Komponenten in der Regel aus einer Sicht aufgerufen werden, aber Sie direkt von einem Controllermethode aufrufen.</span><span class="sxs-lookup"><span data-stu-id="47336-178">View components are typically invoked from a view, but you can invoke them directly from a controller method.</span></span> <span data-ttu-id="47336-179">Während ansichtskomponenten keine Endpunkte wie Controller definieren, können Sie problemlos eine Controlleraktion, die den Inhalt des zurückgibt implementieren eine `ViewComponentResult`.</span><span class="sxs-lookup"><span data-stu-id="47336-179">While view components do not define endpoints like controllers, you can easily implement a controller action that returns the content of a `ViewComponentResult`.</span></span>

<span data-ttu-id="47336-180">In diesem Beispiel wird die Ansichtskomponente direkt auf dem Controller aufgerufen:</span><span class="sxs-lookup"><span data-stu-id="47336-180">In this example, the view component is called directly from the controller:</span></span>

[!code-csharp[Main](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

## <a name="walkthrough-creating-a-simple-view-component"></a><span data-ttu-id="47336-181">Exemplarische Vorgehensweise: Erstellen einer einfachen Ansicht</span><span class="sxs-lookup"><span data-stu-id="47336-181">Walkthrough: Creating a simple view component</span></span>

<span data-ttu-id="47336-182">[Herunterladen](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/views/view-components/sample), erstellen und Testen Sie den Startcode.</span><span class="sxs-lookup"><span data-stu-id="47336-182">[Download](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/views/view-components/sample), build and test the starter code.</span></span> <span data-ttu-id="47336-183">Es ist ein einfaches Projekt mit einem `Todo` Controller, der zeigt eine Liste der *Todo* Elemente.</span><span class="sxs-lookup"><span data-stu-id="47336-183">It's a simple project with a `Todo` controller that displays a list of *Todo* items.</span></span>

![Liste der ToDos](view-components/_static/2dos.png)

### <a name="add-a-viewcomponent-class"></a><span data-ttu-id="47336-185">Fügen Sie eine ViewComponent-Klasse</span><span class="sxs-lookup"><span data-stu-id="47336-185">Add a ViewComponent class</span></span>

<span data-ttu-id="47336-186">Erstellen einer *ViewComponents* Ordner, und fügen Sie die folgenden `PriorityListViewComponent` Klasse:</span><span class="sxs-lookup"><span data-stu-id="47336-186">Create a *ViewComponents* folder and add the following `PriorityListViewComponent` class:</span></span>

[!code-csharp[Main](view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponent1.cs?name=snippet1)]

<span data-ttu-id="47336-187">Hinweise zum Code:</span><span class="sxs-lookup"><span data-stu-id="47336-187">Notes on the code:</span></span>

* <span data-ttu-id="47336-188">Komponente Ansichtsklassen in enthalten sein können **alle** Ordner des Projekts.</span><span class="sxs-lookup"><span data-stu-id="47336-188">View component classes can be contained in **any** folder in the project.</span></span>
* <span data-ttu-id="47336-189">Da der Klassenname PriorityList**ViewComponent** endet mit dem Suffix **ViewComponent**, die Common Language Runtime wird die Zeichenfolge "PriorityList" verwenden, wenn Sie die Klasse, Komponente aus einer Sicht verweisen.</span><span class="sxs-lookup"><span data-stu-id="47336-189">Because the class name PriorityList**ViewComponent** ends with the suffix **ViewComponent**, the runtime will use the string "PriorityList" when referencing the class component from a view.</span></span> <span data-ttu-id="47336-190">Ich werde, die später ausführlicher erläutert.</span><span class="sxs-lookup"><span data-stu-id="47336-190">I'll explain that in more detail later.</span></span>
* <span data-ttu-id="47336-191">Die `[ViewComponent]` Attribut den Namen verwendet, um eine Ansicht Komponentenverweis ändern.</span><span class="sxs-lookup"><span data-stu-id="47336-191">The `[ViewComponent]` attribute can change the name used to reference a view component.</span></span> <span data-ttu-id="47336-192">Angenommen, wir konnte haben mit dem Namen der Klasse `XYZ` angewendet, und die `ViewComponent` Attribut:</span><span class="sxs-lookup"><span data-stu-id="47336-192">For example, we could have named the class `XYZ` and applied the `ViewComponent` attribute:</span></span>

  ```csharp
  [ViewComponent(Name = "PriorityList")]
     public class XYZ : ViewComponent
     ```

* <span data-ttu-id="47336-193">Die `[ViewComponent]` Attribut teilt die Ansichtsauswahl für die Komponente den Namen `PriorityList` bei der Suche nach der Sichten verknüpft sind, mit der Komponente und die Zeichenfolge "PriorityList" verwenden, wenn Sie die Klasse, Komponente aus einer Sicht verweisen.</span><span class="sxs-lookup"><span data-stu-id="47336-193">The `[ViewComponent]` attribute above tells the view component selector to use the name `PriorityList` when looking for the views associated with the component, and to use the string "PriorityList" when referencing the class component from a view.</span></span> <span data-ttu-id="47336-194">Ich werde, die später ausführlicher erläutert.</span><span class="sxs-lookup"><span data-stu-id="47336-194">I'll explain that in more detail later.</span></span>
* <span data-ttu-id="47336-195">Die Komponente verwendet [Abhängigkeitsinjektion](../../fundamentals/dependency-injection.md) um den Datenkontext verfügbar zu machen.</span><span class="sxs-lookup"><span data-stu-id="47336-195">The component uses [dependency injection](../../fundamentals/dependency-injection.md) to make the data context available.</span></span>
* <span data-ttu-id="47336-196">`InvokeAsync`macht dauert eine Methode, die aus einer Sicht aufgerufen werden, kann eine beliebige Anzahl von Argumenten.</span><span class="sxs-lookup"><span data-stu-id="47336-196">`InvokeAsync` exposes a method which can be called from a view, and it can take an arbitrary number of arguments.</span></span>
* <span data-ttu-id="47336-197">Die `InvokeAsync` Methode gibt den Satz der `ToDo` Elemente, die erfüllen die `isDone` und `maxPriority` Parameter.</span><span class="sxs-lookup"><span data-stu-id="47336-197">The `InvokeAsync` method returns the set of `ToDo` items that satisfy the `isDone` and `maxPriority` parameters.</span></span>

### <a name="create-the-view-component-razor-view"></a><span data-ttu-id="47336-198">Erstellen Sie die Komponente Razor-Ansicht</span><span class="sxs-lookup"><span data-stu-id="47336-198">Create the view component Razor view</span></span>

* <span data-ttu-id="47336-199">Erstellen der *Ansichten/freigegeben/Komponenten* Ordner.</span><span class="sxs-lookup"><span data-stu-id="47336-199">Create the *Views/Shared/Components* folder.</span></span> <span data-ttu-id="47336-200">In diesem Ordner **müssen** heißen *Komponenten*.</span><span class="sxs-lookup"><span data-stu-id="47336-200">This folder **must** be named *Components*.</span></span>

* <span data-ttu-id="47336-201">Erstellen der *Ansichten/freigegeben/Components/PriorityList* Ordner.</span><span class="sxs-lookup"><span data-stu-id="47336-201">Create the *Views/Shared/Components/PriorityList* folder.</span></span> <span data-ttu-id="47336-202">Diese Ordnername muss übereinstimmen, den Namen der Komponente Ansichtsklasse oder den Namen der Klasse ohne das Suffix (wenn es folgt der Konvention und verwendet die *ViewComponent* Suffix im Klassennamen).</span><span class="sxs-lookup"><span data-stu-id="47336-202">This folder name must match the name of the view component class, or the name of the class minus the suffix (if we followed convention and used the *ViewComponent* suffix in the class name).</span></span> <span data-ttu-id="47336-203">Bei Verwendung der `ViewComponent` -Attribut, der Klassennamen müsste die Bezeichnung Attribut entsprechen.</span><span class="sxs-lookup"><span data-stu-id="47336-203">If you used the `ViewComponent` attribute, the class name would need to match the attribute designation.</span></span>

* <span data-ttu-id="47336-204">Erstellen einer *Views/Shared/Components/PriorityList/Default.cshtml* Razor-Ansicht:[!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/Default1.cshtml)]</span><span class="sxs-lookup"><span data-stu-id="47336-204">Create a *Views/Shared/Components/PriorityList/Default.cshtml* Razor view: [!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/Default1.cshtml)]</span></span>
    
   <span data-ttu-id="47336-205">Der Razor-Ansicht kann eine Liste von `TodoItem` und zeigt diese an.</span><span class="sxs-lookup"><span data-stu-id="47336-205">The Razor view takes a list of `TodoItem` and displays them.</span></span> <span data-ttu-id="47336-206">Wenn die Ansichtskomponente `InvokeAsync` Methode nicht übergeben Sie den Namen der Sicht (wie in diesem Beispiel), *Standard* wird für den Ansichtsnamen gemäß der Konvention verwendet.</span><span class="sxs-lookup"><span data-stu-id="47336-206">If the view component `InvokeAsync` method doesn't pass the name of the view (as in our sample), *Default* is used for the view name by convention.</span></span> <span data-ttu-id="47336-207">Später in diesem Lernprogramm zeige ich Ihnen wie der Name der Ansicht zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="47336-207">Later in the tutorial, I'll show you how to pass the name of the view.</span></span> <span data-ttu-id="47336-208">Um das standardmäßige Format für einen bestimmten Controller zu überschreiben, fügen Sie eine Ansicht in den Ansichtordner Controller-spezifische (z. B. *Views/Todo/Components/PriorityList/Default.cshtml)*.</span><span class="sxs-lookup"><span data-stu-id="47336-208">To override the default styling for a specific controller, add a view to the controller-specific view folder (for example *Views/Todo/Components/PriorityList/Default.cshtml)*.</span></span>
    
    <span data-ttu-id="47336-209">Wenn die Ansichtskomponente Controller spezifisch ist, können Sie es in den Ordner Controller-spezifische hinzufügen (*Views/Todo/Components/PriorityList/Default.cshtml*).</span><span class="sxs-lookup"><span data-stu-id="47336-209">If the view component is controller-specific, you can add it to the controller-specific folder (*Views/Todo/Components/PriorityList/Default.cshtml*).</span></span>

* <span data-ttu-id="47336-210">Hinzufügen einer `div` , die einen Aufruf an das Ende der Priorität List-Komponente enthält die *Views/Todo/index.cshtml* Datei:</span><span class="sxs-lookup"><span data-stu-id="47336-210">Add a `div` containing a call to the priority list component to the bottom of the *Views/Todo/index.cshtml* file:</span></span>

    [!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexFirst.cshtml?range=34-38)]

<span data-ttu-id="47336-211">Das Markup `@await Component.InvokeAsync` zeigt die Syntax zum Aufrufen von Komponenten anzeigen.</span><span class="sxs-lookup"><span data-stu-id="47336-211">The markup `@await Component.InvokeAsync` shows the syntax for calling view components.</span></span> <span data-ttu-id="47336-212">Das erste Argument ist der Name der Komponente, die wir aufrufen oder aufgerufen werden soll.</span><span class="sxs-lookup"><span data-stu-id="47336-212">The first argument is the name of the component we want to invoke or call.</span></span> <span data-ttu-id="47336-213">Nachfolgende Parameter werden an die Komponente übergeben.</span><span class="sxs-lookup"><span data-stu-id="47336-213">Subsequent parameters are passed to the component.</span></span> <span data-ttu-id="47336-214">`InvokeAsync`kann eine beliebige Anzahl von Argumenten dauern.</span><span class="sxs-lookup"><span data-stu-id="47336-214">`InvokeAsync` can take an arbitrary number of arguments.</span></span>

<span data-ttu-id="47336-215">Testen der app an.</span><span class="sxs-lookup"><span data-stu-id="47336-215">Test the app.</span></span> <span data-ttu-id="47336-216">Die folgende Abbildung zeigt die TODO-Liste und Elemente mit der Priorität:</span><span class="sxs-lookup"><span data-stu-id="47336-216">The following image shows the ToDo list and the priority items:</span></span>

![TODO-Liste und die Priorität der Elemente](view-components/_static/pi.png)

<span data-ttu-id="47336-218">Sie können auch die Ansichtskomponente direkt auf dem Controller aufrufen:</span><span class="sxs-lookup"><span data-stu-id="47336-218">You can also call the view component directly from the controller:</span></span>

[!code-csharp[Main](view-components/sample/ViewCompFinal/Controllers/ToDoController.cs?name=snippet_IndexVC)]

![Elemente der Priorität von IndexVC-Aktion](view-components/_static/indexvc.png)

### <a name="specifying-a-view-name"></a><span data-ttu-id="47336-220">Einen Ansichtsnamen angeben</span><span class="sxs-lookup"><span data-stu-id="47336-220">Specifying a view name</span></span>

<span data-ttu-id="47336-221">Eine komplexe Ansichtskomponente müssen möglicherweise eine nicht standardmäßige Ansicht unter bestimmten Umständen angeben.</span><span class="sxs-lookup"><span data-stu-id="47336-221">A complex view component might need to specify a non-default view under some conditions.</span></span> <span data-ttu-id="47336-222">Der folgende Code zeigt, wie die Ansicht "PVC" aus der `InvokeAsync` Methode.</span><span class="sxs-lookup"><span data-stu-id="47336-222">The following code shows how to specify the "PVC" view  from the `InvokeAsync` method.</span></span> <span data-ttu-id="47336-223">Update der `InvokeAsync` Methode in der `PriorityListViewComponent` Klasse.</span><span class="sxs-lookup"><span data-stu-id="47336-223">Update the `InvokeAsync` method in the `PriorityListViewComponent` class.</span></span>

[!code-csharp[Main](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityListViewComponentFinal.cs?highlight=4,5,6,7,8,9&range=28-39)]

<span data-ttu-id="47336-224">Kopieren der *Views/Shared/Components/PriorityList/Default.cshtml* Datei in eine Ansicht mit dem Namen *Views/Shared/Components/PriorityList/PVC.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="47336-224">Copy the *Views/Shared/Components/PriorityList/Default.cshtml* file to a view named *Views/Shared/Components/PriorityList/PVC.cshtml*.</span></span> <span data-ttu-id="47336-225">Fügen Sie eine Überschrift, um anzugeben, dass die Sicht PVC verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="47336-225">Add a heading to indicate the PVC view is being used.</span></span>

[!code-cshtml[Main](../../mvc/views/view-components/sample/ViewCompFinal/Views/Shared/Components/PriorityList/PVC.cshtml?highlight=3)]

<span data-ttu-id="47336-226">Update *Views/TodoList/Index.cshtml*:</span><span class="sxs-lookup"><span data-stu-id="47336-226">Update *Views/TodoList/Index.cshtml*:</span></span>

<!-- Views/TodoList/Index.cshtml is never imported, so change to test tutorial -->

[!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexFinal.cshtml?range=35)]

<span data-ttu-id="47336-227">Führen Sie die app, und überprüfen Sie PVC anzeigen.</span><span class="sxs-lookup"><span data-stu-id="47336-227">Run the app and verify PVC view.</span></span>

![Priorität Ansichtskomponente](view-components/_static/pvc.png)

<span data-ttu-id="47336-229">Wenn die Sicht PVC nicht gerendert wird, stellen Sie sicher, dass Sie die Ansichtskomponente mit einer Priorität von 4 oder höher aufrufen.</span><span class="sxs-lookup"><span data-stu-id="47336-229">If the PVC view is not rendered, verify you are calling the view component with a priority of 4 or higher.</span></span>

### <a name="examine-the-view-path"></a><span data-ttu-id="47336-230">Überprüfen Sie den Pfad anzeigen</span><span class="sxs-lookup"><span data-stu-id="47336-230">Examine the view path</span></span>

* <span data-ttu-id="47336-231">Ändern Sie den Parameter Priorität auf drei oder weniger ein, damit die Ansicht Priorität nicht zurückgegeben wird.</span><span class="sxs-lookup"><span data-stu-id="47336-231">Change the priority parameter to three or less so the priority view is not returned.</span></span>
* <span data-ttu-id="47336-232">Benennen Sie vorübergehend die *Views/Todo/Components/PriorityList/Default.cshtml* auf *1Default.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="47336-232">Temporarily rename the *Views/Todo/Components/PriorityList/Default.cshtml* to *1Default.cshtml*.</span></span>
* <span data-ttu-id="47336-233">Testen der app, die Sie erhalten die folgende Fehlermeldung:</span><span class="sxs-lookup"><span data-stu-id="47336-233">Test the app, you'll get the following error:</span></span>

   ```
   An unhandled exception occurred while processing the request.
   InvalidOperationException: The view 'Components/PriorityList/Default' was not found. The following locations were searched:
   /Views/ToDo/Components/PriorityList/Default.cshtml
   /Views/Shared/Components/PriorityList/Default.cshtml
   EnsureSuccessful
   ```

* <span data-ttu-id="47336-234">Kopie *Views/Todo/Components/PriorityList/1Default.cshtml* auf *Views/Shared/Components/PriorityList/Default.cshtml*.</span><span class="sxs-lookup"><span data-stu-id="47336-234">Copy *Views/Todo/Components/PriorityList/1Default.cshtml* to *Views/Shared/Components/PriorityList/Default.cshtml*.</span></span>
* <span data-ttu-id="47336-235">Einige Markup zum Hinzufügen der *Shared* Komponente Todo-Ansicht an, dass die Ansicht ist von der *Shared* Ordner.</span><span class="sxs-lookup"><span data-stu-id="47336-235">Add some markup to the *Shared* Todo view component view to indicate the view is from the *Shared* folder.</span></span>
* <span data-ttu-id="47336-236">Testen der **Shared** Komponentensicht.</span><span class="sxs-lookup"><span data-stu-id="47336-236">Test the **Shared** component view.</span></span>

![TODO-Ausgabe mit der freigegebenen Komponente anzeigen](view-components/_static/shared.png)

### <a name="avoiding-magic-strings"></a><span data-ttu-id="47336-238">Vermeiden von Magic-Zeichenfolgen</span><span class="sxs-lookup"><span data-stu-id="47336-238">Avoiding magic strings</span></span>

<span data-ttu-id="47336-239">Wenn Sie die Sicherheit kompilieren möchten, können Sie den Namen der Sicht hartcodierte Komponente mit dem Klassennamen ersetzen.</span><span class="sxs-lookup"><span data-stu-id="47336-239">If you want compile time safety, you can replace the hard-coded view component name with the class name.</span></span> <span data-ttu-id="47336-240">Erstellen Sie die Komponente anzeigen, ohne das Suffix "ViewComponent":</span><span class="sxs-lookup"><span data-stu-id="47336-240">Create the view component without the "ViewComponent" suffix:</span></span>

[!code-csharp[Main](../../mvc/views/view-components/sample/ViewCompFinal/ViewComponents/PriorityList.cs?highlight=10&range=5-35)]

<span data-ttu-id="47336-241">Hinzufügen einer `using` Anweisung, um Ihre Razor zeigen Sie an, und verwenden die `nameof` Operator:</span><span class="sxs-lookup"><span data-stu-id="47336-241">Add a `using` statement to your Razor view file, and use the `nameof` operator:</span></span>

[!code-cshtml[Main](view-components/sample/ViewCompFinal/Views/Todo/IndexNameof.cshtml?range=1-6,33-)]

## <a name="additional-resources"></a><span data-ttu-id="47336-242">Zusätzliche Ressourcen</span><span class="sxs-lookup"><span data-stu-id="47336-242">Additional Resources</span></span>

* [<span data-ttu-id="47336-243">Abhängigkeitsinjektion in Ansichten</span><span class="sxs-lookup"><span data-stu-id="47336-243">Dependency injection into views</span></span>](dependency-injection.md)