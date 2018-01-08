---
uid: identity/overview/extensibility/change-primary-key-for-users-in-aspnet-identity
title: "Ändern der Primärschlüssel für Benutzer in ASP.NET Identity | Microsoft Docs"
author: tfitzmac
description: "In Visual Studio 2013 verwendet die standardwebanwendung einen Zeichenfolgenwert für den Schlüssel für Benutzerkonten an. ASP.NET Identity ermöglicht es Ihnen, den Typ des Ändern der..."
ms.author: aspnetcontent
manager: wpickett
ms.date: 09/30/2014
ms.topic: article
ms.assetid: 44925849-5762-4504-a8cd-8f0cd06f6dc3
ms.technology: 
ms.prod: .net-framework
msc.legacyurl: /identity/overview/extensibility/change-primary-key-for-users-in-aspnet-identity
msc.type: authoredcontent
ms.openlocfilehash: 79812efb4de2461fad3765d6005bbd20393e62b2
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
<a name="change-primary-key-for-users-in-aspnet-identity"></a><span data-ttu-id="f2ecd-104">Primärschlüssel für Benutzer in ASP.NET Identity ändern</span><span class="sxs-lookup"><span data-stu-id="f2ecd-104">Change Primary Key for Users in ASP.NET Identity</span></span>
====================
<span data-ttu-id="f2ecd-105">durch [Tom FitzMacken](https://github.com/tfitzmac)</span><span class="sxs-lookup"><span data-stu-id="f2ecd-105">by [Tom FitzMacken](https://github.com/tfitzmac)</span></span>

> <span data-ttu-id="f2ecd-106">In Visual Studio 2013 verwendet die standardwebanwendung einen Zeichenfolgenwert für den Schlüssel für Benutzerkonten an.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-106">In Visual Studio 2013, the default web application uses a string value for the key for user accounts.</span></span> <span data-ttu-id="f2ecd-107">ASP.NET Identity können Sie den Typ des Schlüssels, der Ihren Anforderungen Daten zu ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-107">ASP.NET Identity enables you to change the type of the key to meet your data requirements.</span></span> <span data-ttu-id="f2ecd-108">Beispielsweise können Sie den Typ des Schlüssels aus einer Zeichenfolge in eine ganze Zahl ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-108">For example, you can change the type of the key from a string to an integer.</span></span>
> 
> <span data-ttu-id="f2ecd-109">In diesem Thema wird gezeigt, wie start mit dem Standardwert-Webanwendung und ändern Sie den kontoschlüssel für den Benutzer in eine ganze Zahl.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-109">This topic shows how to start with the default web application and change the user account key to an integer.</span></span> <span data-ttu-id="f2ecd-110">Die gleichen Änderungen können Sie jeder Typ des Schlüssels in Ihrem Projekt implementieren.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-110">You can use the same modifications to implement any type of key in your project.</span></span> <span data-ttu-id="f2ecd-111">Es zeigt, wie diese Änderungen in der standardwebanwendung übernommen, aber Sie können ähnliche Änderungen an einer benutzerdefinierten Anwendung anwenden.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-111">It shows how to make these changes in the default web application, but you could apply similar modifications to a customized application.</span></span> <span data-ttu-id="f2ecd-112">Es zeigt die Änderungen, die beim Arbeiten mit MVC oder Web Forms erforderlich.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-112">It shows the changes needed when working with MVC or Web Forms.</span></span>
> 
> ## <a name="software-versions-used-in-the-tutorial"></a><span data-ttu-id="f2ecd-113">In diesem Lernprogramm verwendeten Versionen der Software</span><span class="sxs-lookup"><span data-stu-id="f2ecd-113">Software versions used in the tutorial</span></span>
> 
> 
> - <span data-ttu-id="f2ecd-114">Visual Studio 2013 Update 2 (oder höher)</span><span class="sxs-lookup"><span data-stu-id="f2ecd-114">Visual Studio 2013 with Update 2 (or later)</span></span>
> - <span data-ttu-id="f2ecd-115">ASP.NET Identity 2.1 oder höher</span><span class="sxs-lookup"><span data-stu-id="f2ecd-115">ASP.NET Identity 2.1 or later</span></span>


<span data-ttu-id="f2ecd-116">Um die Schritte in diesem Lernprogramm ausführen zu können, benötigen Sie Visual Studio 2013 Update 2 (oder höher) und eine Webanwendung, die mithilfe der ASP.NET Web Application-Vorlage erstellt.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-116">To perform the steps in this tutorial, you must have Visual Studio 2013 Update 2 (or later), and a web application created from the ASP.NET Web Application template.</span></span> <span data-ttu-id="f2ecd-117">Die Vorlage, die in Update 3 geändert wird.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-117">The template changed in Update 3.</span></span> <span data-ttu-id="f2ecd-118">In diesem Thema zeigt, wie die Vorlage in Update 2 und Update 3 zu ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-118">This topic shows how to change the template in Update 2 and Update 3.</span></span>

<span data-ttu-id="f2ecd-119">Dieses Thema enthält folgende Abschnitte:</span><span class="sxs-lookup"><span data-stu-id="f2ecd-119">This topic contains the following sections:</span></span>

- [<span data-ttu-id="f2ecd-120">Ändern Sie den Typ des Schlüssels in die Identity-Klasse</span><span class="sxs-lookup"><span data-stu-id="f2ecd-120">Change the type of the key in the Identity user class</span></span>](#userclass)
- [<span data-ttu-id="f2ecd-121">Hinzufügen von benutzerdefinierten Identität-Klassen, die den Typ des Schlüssels verwenden</span><span class="sxs-lookup"><span data-stu-id="f2ecd-121">Add customized Identity classes that use the key type</span></span>](#customclass)
- [<span data-ttu-id="f2ecd-122">Ändern Sie die Kontext-Klasse und Benutzer-Manager, um den Typ des Schlüssels verwenden</span><span class="sxs-lookup"><span data-stu-id="f2ecd-122">Change the context class and user manager to use the key type</span></span>](#context)
- [<span data-ttu-id="f2ecd-123">Start-Konfiguration ändern, verwenden Sie den Typ des Schlüssels</span><span class="sxs-lookup"><span data-stu-id="f2ecd-123">Change start-up configuration to use the key type</span></span>](#startup)
- [<span data-ttu-id="f2ecd-124">Für MVC mit Update 2 ändern Sie die AccountController übergeben Sie den Typ des Schlüssels</span><span class="sxs-lookup"><span data-stu-id="f2ecd-124">For MVC with Update 2, change the AccountController to pass the key type</span></span>](#mvcupdate2)
- [<span data-ttu-id="f2ecd-125">Für MVC mit Update 3 zu ändern, die AccountController und ManageController, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-125">For MVC with Update 3, change the AccountController and ManageController to pass the key type</span></span>](#mvcupdate3)
- [<span data-ttu-id="f2ecd-126">Ändern Sie für Web Forms Update 2 Konto-Seiten, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-126">For Web Forms with Update 2, change Account pages to pass the key type</span></span>](#webformsupdate2)
- [<span data-ttu-id="f2ecd-127">Ändern Sie für Webformulare mit Update 3 Konto-Seiten, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-127">For Web Forms with Update 3, change Account pages to pass the key type</span></span>](#webformsupdate3)
- [<span data-ttu-id="f2ecd-128">Führen Sie die Anwendung</span><span class="sxs-lookup"><span data-stu-id="f2ecd-128">Run application</span></span>](#run)
- [<span data-ttu-id="f2ecd-129">Weitere Ressourcen</span><span class="sxs-lookup"><span data-stu-id="f2ecd-129">Other resources</span></span>](#other)

<a id="userclass"></a>
## <a name="change-the-type-of-the-key-in-the-identity-user-class"></a><span data-ttu-id="f2ecd-130">Ändern Sie den Typ des Schlüssels in die Identity-Klasse</span><span class="sxs-lookup"><span data-stu-id="f2ecd-130">Change the type of the key in the Identity user class</span></span>

<span data-ttu-id="f2ecd-131">Geben Sie in Ihrem Projekt mithilfe der ASP.NET Web Application-Vorlage erstellt, dass die ApplicationUser-Klasse eine ganze Zahl für den Schlüssel für Benutzerkonten verwendet.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-131">In your project created from the ASP.NET Web Application template, specify that the ApplicationUser class uses an integer for the key for user accounts.</span></span> <span data-ttu-id="f2ecd-132">Ändern Sie die ApplicationUser-Klasse aus IdentityUser erben, die den Typ aufweist, in IdentityModels.cs, **Int** für den generischen TKey-Parameter.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-132">In IdentityModels.cs, change the ApplicationUser class to inherit from IdentityUser that has a type of **int** for the TKey generic parameter.</span></span> <span data-ttu-id="f2ecd-133">Sie übergeben außerdem die Namen der drei benutzerdefinierte Klasse, die noch nicht implementiert.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-133">You also pass the names of three customized class which you have not implemented yet.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample1.cs?highlight=1-2)]

<span data-ttu-id="f2ecd-134">Sie haben den Typ des Schlüssels geändert, aber standardmäßig der Rest der Anwendung weiterhin geht davon aus, dass der Schlüssel eine Zeichenfolge ist.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-134">You have changed the type of the key, but, by default, the rest of the application still assumes the key is a string.</span></span> <span data-ttu-id="f2ecd-135">Sie müssen explizit den Typ des Schlüssels im Code angeben, die eine Zeichenfolge annimmt.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-135">You must explicitly indicate the type of the key in code that assumes a string.</span></span>

<span data-ttu-id="f2ecd-136">In der **ApplicationUser** Klasse, ändern Sie die **GenerateUserIdentityAsync** Methode, um "Int", enthalten, wie in den folgenden hervorgehobenen Code gezeigt.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-136">In the **ApplicationUser** class, change the **GenerateUserIdentityAsync** method to include int, as shown in the highlighted code below.</span></span> <span data-ttu-id="f2ecd-137">Diese Änderung ist nicht für Web Forms-Projekte mit der Vorlage Update 3 erforderlich.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-137">This change is not necessary for Web Forms projects with the Update 3 template.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample2.cs?highlight=2)]

<a id="customclass"></a>
## <a name="add-customized-identity-classes-that-use-the-key-type"></a><span data-ttu-id="f2ecd-138">Hinzufügen von benutzerdefinierten Identität-Klassen, die den Typ des Schlüssels verwenden</span><span class="sxs-lookup"><span data-stu-id="f2ecd-138">Add customized Identity classes that use the key type</span></span>

<span data-ttu-id="f2ecd-139">Die anderen Identity-Klassen, z. B. IdentityUserRole, IdentityUserClaim, IdentityUserLogin, IdentityRole, UserStore, RoleStore, werden weiterhin einen Zeichenfolgenschlüssel verwendet eingerichtet.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-139">The other Identity classes, such as IdentityUserRole, IdentityUserClaim, IdentityUserLogin, IdentityRole, UserStore, RoleStore, are still set up to use a string key.</span></span> <span data-ttu-id="f2ecd-140">Erstellen Sie neue Versionen dieser Klassen, die eine ganze Zahl für den Schlüssel angeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-140">Create new versions of these classes that specify an integer for the key.</span></span> <span data-ttu-id="f2ecd-141">Sie müssen nicht viel Implementierungscode in diesen Klassen bereitstellen, Int in erster Linie nur als Schlüssel festlegen.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-141">You do not need to provide much implementation code in these classes, you are primarily just setting int as the key.</span></span>

<span data-ttu-id="f2ecd-142">Die folgenden Klassen der Datei IdentityModels.cs hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-142">Add the following classes to your IdentityModels.cs file.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample3.cs)]

<a id="context"></a>
## <a name="change-the-context-class-and-user-manager-to-use-the-key-type"></a><span data-ttu-id="f2ecd-143">Ändern Sie die Kontext-Klasse und Benutzer-Manager, um den Typ des Schlüssels verwenden</span><span class="sxs-lookup"><span data-stu-id="f2ecd-143">Change the context class and user manager to use the key type</span></span>

<span data-ttu-id="f2ecd-144">In IdentityModels.cs, ändern Sie die Definition der **ApplicationDbContext** Klasse zur Verwendung der neuen angepasst, Klassen und ein **Int** für den Schlüssel, wie in der hervorgehobene Code gezeigt.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-144">In IdentityModels.cs, change the definition of the **ApplicationDbContext** class to use your new customized classes and an **int** for the key, as shown in the highlighted code.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample4.cs?highlight=1-2)]

<span data-ttu-id="f2ecd-145">Der ThrowIfV1Schema-Parameter ist nicht mehr gültig ist, im Konstruktor.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-145">The ThrowIfV1Schema parameter is no longer valid in the constructor.</span></span> <span data-ttu-id="f2ecd-146">Ändern Sie den Konstruktor aus, damit er nicht auf einen ThrowIfV1Schema-Wert übergeben wird.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-146">Change the constructor so it does not pass a ThrowIfV1Schema value.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample5.cs)]

<span data-ttu-id="f2ecd-147">IdentityConfig.cs öffnen, und Ändern der **ApplicationUserManger** Klasse mit der neue Benutzer speichern Klasse für das Beibehalten von Daten und ein **Int** für den Schlüssel.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-147">Open IdentityConfig.cs, and change the **ApplicationUserManger** class to use your new user store class for persisting data and an **int** for the key.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample6.cs?highlight=1,3,12,14,32,37,48)]

<span data-ttu-id="f2ecd-148">In der Vorlage Update 3 müssen Sie die Klasse ApplicationSignInManager ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-148">In the Update 3 template, you must change the ApplicationSignInManager class.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample7.cs?highlight=1)]

<a id="startup"></a>
## <a name="change-start-up-configuration-to-use-the-key-type"></a><span data-ttu-id="f2ecd-149">Start-Konfiguration ändern, verwenden Sie den Typ des Schlüssels</span><span class="sxs-lookup"><span data-stu-id="f2ecd-149">Change start-up configuration to use the key type</span></span>

<span data-ttu-id="f2ecd-150">Ersetzen Sie im Startup.Auth.cs den OnValidateIdentity-Code, wie unten markiert.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-150">In Startup.Auth.cs, replace the OnValidateIdentity code, as highlighted below.</span></span> <span data-ttu-id="f2ecd-151">Beachten Sie, dass die Definition der GetUserIdCallback analysiert den Zeichenfolgenwert in eine ganze Zahl.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-151">Notice that the getUserIdCallback definition, parses the string value into an integer.</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample8.cs?highlight=7-12)]

<span data-ttu-id="f2ecd-152">Wenn das Projekt nicht die generische Implementierung erkennt die **BenutzerID** -Methode, müssen Sie möglicherweise die ASP.NET Identity NuGet-Paket auf Version 2.1 aktualisieren</span><span class="sxs-lookup"><span data-stu-id="f2ecd-152">If your project does not recognize the generic implementation of the **GetUserId** method, you may need to update the ASP.NET Identity NuGet package to version 2.1</span></span>

<span data-ttu-id="f2ecd-153">Sie haben den Infrastruktur-Klassen, die von ASP.NET Identity verwendet viele Änderungen vorgenommen.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-153">You have made a lot of changes to the infrastructure classes used by ASP.NET Identity.</span></span> <span data-ttu-id="f2ecd-154">Wenn Sie versuchen, das Projekt kompilieren, bemerken Sie viele Fehler.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-154">If you try compiling the project, you will notice a lot of errors.</span></span> <span data-ttu-id="f2ecd-155">Glücklicherweise ähneln die restlichen Fehler.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-155">Fortunately, the remaining errors are all similar.</span></span> <span data-ttu-id="f2ecd-156">Die Identity-Klasse erwartet eine ganze Zahl für den Schlüssel, aber der Controller (oder Web Form) ist einen String-Wert übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-156">The Identity class expects an integer for the key, but the controller (or Web Form) is passing a string value.</span></span> <span data-ttu-id="f2ecd-157">In jedem Fall müssen Sie durch Aufrufen von einer Zeichenfolge und ganze Zahl konvertieren **BenutzerID&lt;Int&gt;**.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-157">In each case, you need to convert from a string to and integer by calling **GetUserId&lt;int&gt;**.</span></span> <span data-ttu-id="f2ecd-158">Sie können die Fehlerliste Kompilierung zu absolvieren, oder führen Sie die folgenden Änderungen.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-158">You can either work through the error list from compilation or follow the changes below.</span></span>

<span data-ttu-id="f2ecd-159">Die verbleibenden Änderungen hängt vom Typ des Projekts, die Sie erstellen und die Updates, die Sie in Visual Studio installiert haben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-159">The remaining changes depend on the type of project you are creating and which update you have installed in Visual Studio.</span></span> <span data-ttu-id="f2ecd-160">Sie können direkt an den entsprechenden Abschnitt über die folgenden Links wechseln.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-160">You can go directly to the relevant section through the following links</span></span>

- [<span data-ttu-id="f2ecd-161">Für MVC mit Update 2 ändern Sie die AccountController übergeben Sie den Typ des Schlüssels</span><span class="sxs-lookup"><span data-stu-id="f2ecd-161">For MVC with Update 2, change the AccountController to pass the key type</span></span>](#mvcupdate2)
- [<span data-ttu-id="f2ecd-162">Für MVC mit Update 3 zu ändern, die AccountController und ManageController, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-162">For MVC with Update 3, change the AccountController and ManageController to pass the key type</span></span>](#mvcupdate3)
- [<span data-ttu-id="f2ecd-163">Ändern Sie für Web Forms Update 2 Konto-Seiten, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-163">For Web Forms with Update 2, change Account pages to pass the key type</span></span>](#webformsupdate2)
- [<span data-ttu-id="f2ecd-164">Ändern Sie für Webformulare mit Update 3 Konto-Seiten, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-164">For Web Forms with Update 3, change Account pages to pass the key type</span></span>](#webformsupdate3)

<a id="mvcupdate2"></a>
## <a name="for-mvc-with-update-2-change-the-accountcontroller-to-pass-the-key-type"></a><span data-ttu-id="f2ecd-165">Für MVC mit Update 2 ändern Sie die AccountController übergeben Sie den Typ des Schlüssels</span><span class="sxs-lookup"><span data-stu-id="f2ecd-165">For MVC with Update 2, change the AccountController to pass the key type</span></span>

<span data-ttu-id="f2ecd-166">Öffnen Sie die AccountController.cs-Datei.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-166">Open the AccountController.cs file.</span></span> <span data-ttu-id="f2ecd-167">Sie müssen die folgenden Methoden ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-167">You need to change the following methods.</span></span>

<span data-ttu-id="f2ecd-168">**ConfirmEmail** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-168">**ConfirmEmail** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample9.cs?highlight=1,3)]

<span data-ttu-id="f2ecd-169">**Heben Sie die Zuordnung** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-169">**Disassociate** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample10.cs?highlight=5,9)]

<span data-ttu-id="f2ecd-170">**Manage(ManageUserViewModel)** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-170">**Manage(ManageUserViewModel)** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample11.cs?highlight=11,17,41)]

<span data-ttu-id="f2ecd-171">**LinkLoginCallback** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-171">**LinkLoginCallback** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample12.cs?highlight=10)]

<span data-ttu-id="f2ecd-172">**RemoveAccountList** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-172">**RemoveAccountList** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample13.cs?highlight=3)]

<span data-ttu-id="f2ecd-173">**HasPassword** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-173">**HasPassword** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample14.cs?highlight=3)]

<span data-ttu-id="f2ecd-174">Sie können jetzt [führen Sie die Anwendung](#run) und einen neuen Benutzer registrieren.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-174">You can now [run the application](#run) and register a new user.</span></span>

<a id="mvcupdate3"></a>
## <a name="for-mvc-with-update-3-change-the-accountcontroller-and-managecontroller-to-pass-the-key-type"></a><span data-ttu-id="f2ecd-175">Für MVC mit Update 3 zu ändern, die AccountController und ManageController, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-175">For MVC with Update 3, change the AccountController and ManageController to pass the key type</span></span>

<span data-ttu-id="f2ecd-176">Öffnen Sie die AccountController.cs-Datei.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-176">Open the AccountController.cs file.</span></span> <span data-ttu-id="f2ecd-177">Sie müssen die folgende Methode zu ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-177">You need to change the following method.</span></span>

<span data-ttu-id="f2ecd-178">**ConfirmEmail** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-178">**ConfirmEmail** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample15.cs?highlight=1,3)]

<span data-ttu-id="f2ecd-179">**SendCode** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-179">**SendCode** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample16.cs?highlight=4)]

<span data-ttu-id="f2ecd-180">Öffnen Sie die ManageController.cs-Datei.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-180">Open the ManageController.cs file.</span></span> <span data-ttu-id="f2ecd-181">Sie müssen die folgenden Methoden ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-181">You need to change the following methods.</span></span>

<span data-ttu-id="f2ecd-182">**Index** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-182">**Index** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample17.cs?highlight=15-17)]

<span data-ttu-id="f2ecd-183">**RemoveLogin** Methoden</span><span class="sxs-lookup"><span data-stu-id="f2ecd-183">**RemoveLogin** methods</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample18.cs?highlight=3,13,17)]

<span data-ttu-id="f2ecd-184">**AddPhoneNumber** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-184">**AddPhoneNumber** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample19.cs?highlight=9)]

<span data-ttu-id="f2ecd-185">**EnableTwoFactorAuthentication** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-185">**EnableTwoFactorAuthentication** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample20.cs?highlight=3-4)]

<span data-ttu-id="f2ecd-186">**DisableTwoFactorAuthentication** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-186">**DisableTwoFactorAuthentication** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample21.cs?highlight=3-4)]

<span data-ttu-id="f2ecd-187">**VerifyPhoneNumber** Methoden</span><span class="sxs-lookup"><span data-stu-id="f2ecd-187">**VerifyPhoneNumber** methods</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample22.cs?highlight=4,18,21)]

<span data-ttu-id="f2ecd-188">**RemovePhoneNumber** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-188">**RemovePhoneNumber** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample23.cs?highlight=3,8)]

<span data-ttu-id="f2ecd-189">**ChangePassword** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-189">**ChangePassword** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample24.cs?highlight=10,13)]

<span data-ttu-id="f2ecd-190">**SetPassword** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-190">**SetPassword** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample25.cs?highlight=5,8)]

<span data-ttu-id="f2ecd-191">**ManageLogins** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-191">**ManageLogins** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample26.cs?highlight=7,12)]

<span data-ttu-id="f2ecd-192">**LinkLoginCallback** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-192">**LinkLoginCallback** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample27.cs?highlight=8)]

<span data-ttu-id="f2ecd-193">**HasPassword** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-193">**HasPassword** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample28.cs?highlight=3)]

<span data-ttu-id="f2ecd-194">**HasPhoneNumber** Methode</span><span class="sxs-lookup"><span data-stu-id="f2ecd-194">**HasPhoneNumber** method</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample29.cs?highlight=3)]

<span data-ttu-id="f2ecd-195">Sie können jetzt [führen Sie die Anwendung](#run) und einen neuen Benutzer registrieren.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-195">You can now [run the application](#run) and register a new user.</span></span>

<a id="webformsupdate2"></a>
## <a name="for-web-forms-with-update-2-change-account-pages-to-pass-the-key-type"></a><span data-ttu-id="f2ecd-196">Ändern Sie für Web Forms Update 2 Konto-Seiten, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-196">For Web Forms with Update 2, change Account pages to pass the key type</span></span>

<span data-ttu-id="f2ecd-197">Für Web Forms Update 2 müssen Sie die folgenden Seiten zu ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-197">For Web Forms with Update 2, you need to change the following pages.</span></span>

<span data-ttu-id="f2ecd-198">**Confirm.aspx.CX**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-198">**Confirm.aspx.cx**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample30.cs?highlight=8)]

<span data-ttu-id="f2ecd-199">**RegisterExternalLogin.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-199">**RegisterExternalLogin.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample31.cs?highlight=36)]

<span data-ttu-id="f2ecd-200">**Manage.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-200">**Manage.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample32.cs?highlight=3,22,47,52,69,85,93,98)]

<span data-ttu-id="f2ecd-201">Sie können jetzt [führen Sie die Anwendung](#run) und einen neuen Benutzer registrieren.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-201">You can now [run the application](#run) and register a new user.</span></span>

<a id="webformsupdate3"></a>
## <a name="for-web-forms-with-update-3-change-account-pages-to-pass-the-key-type"></a><span data-ttu-id="f2ecd-202">Ändern Sie für Webformulare mit Update 3 Konto-Seiten, um den Typ des Schlüssels zu übergeben.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-202">For Web Forms with Update 3, change Account pages to pass the key type</span></span>

<span data-ttu-id="f2ecd-203">Für Web Forms mit Update 3 müssen Sie die folgenden Seiten zu ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-203">For Web Forms with Update 3, you need to change the following pages.</span></span>

<span data-ttu-id="f2ecd-204">**Confirm.aspx.CX**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-204">**Confirm.aspx.cx**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample33.cs?highlight=8)]

<span data-ttu-id="f2ecd-205">**RegisterExternalLogin.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-205">**RegisterExternalLogin.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample34.cs?highlight=36)]

<span data-ttu-id="f2ecd-206">**Manage.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-206">**Manage.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample35.cs?highlight=11,27,32,34,82,87,99,108)]

<span data-ttu-id="f2ecd-207">**VerifyPhoneNumber.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-207">**VerifyPhoneNumber.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample36.cs?highlight=8,23,27)]

<span data-ttu-id="f2ecd-208">**AddPhoneNumber.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-208">**AddPhoneNumber.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample37.cs?highlight=7)]

<span data-ttu-id="f2ecd-209">**ManagePassword.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-209">**ManagePassword.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample38.cs?highlight=11,47,50,68)]

<span data-ttu-id="f2ecd-210">**ManageLogins.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-210">**ManageLogins.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample39.cs?highlight=16,23,32,41,45)]

<span data-ttu-id="f2ecd-211">**TwoFactorAuthenticationSignIn.aspx.cs**</span><span class="sxs-lookup"><span data-stu-id="f2ecd-211">**TwoFactorAuthenticationSignIn.aspx.cs**</span></span>

[!code-csharp[Main](change-primary-key-for-users-in-aspnet-identity/samples/sample40.cs?highlight=14-15,29,53)]

<a id="run"></a>
## <a name="run-application"></a><span data-ttu-id="f2ecd-212">Führen Sie die Anwendung</span><span class="sxs-lookup"><span data-stu-id="f2ecd-212">Run application</span></span>

<span data-ttu-id="f2ecd-213">Sie haben alle erforderlichen Änderungen an Standardeinstellung Web Application-Vorlage.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-213">You have finished all of the required changes to the default Web Application template.</span></span> <span data-ttu-id="f2ecd-214">Führen Sie die Anwendung, und registrieren Sie einen neuen Benutzer.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-214">Run the application and register a new user.</span></span> <span data-ttu-id="f2ecd-215">Nach dem Registrieren des Benutzers, werden Sie feststellen, dass die AspNetUsers-Tabelle eine Id-Spalte enthält, die eine ganze Zahl ist.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-215">After registering the user, you will notice that the AspNetUsers table has an Id column that is an integer.</span></span>

![neuen Primärschlüssel](change-primary-key-for-users-in-aspnet-identity/_static/image1.png)

<span data-ttu-id="f2ecd-217">Wenn Sie zuvor die ASP.NET-Identität Tabellen mit einem anderen primären Schlüssel erstellt haben, müssen Sie einige zusätzliche Änderungen vornehmen.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-217">If you have previously created the ASP.NET Identity tables with a different primary key, you need to make some additional changes.</span></span> <span data-ttu-id="f2ecd-218">Wenn möglich, löschen Sie einfach die vorhandene Datenbank ein.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-218">If possible, just delete the existing database.</span></span> <span data-ttu-id="f2ecd-219">Die Datenbank wird mit dem richtigen Entwurf neu erstellt werden, wenn führen Sie die Webanwendung und fügen Sie einen neuen Benutzer hinzu.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-219">The database will be re-created with the correct design when you run the web application and add a new user.</span></span> <span data-ttu-id="f2ecd-220">Wenn der Löschvorgang nicht möglich ist, führen Sie Code first-Migrationen um Tabellen zu ändern.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-220">If deletion is not possible, run code first migrations to change the tables.</span></span> <span data-ttu-id="f2ecd-221">Allerdings wird der neue Primärschlüssel ganze Zahl nicht als eine SQL-IDENTITY-Eigenschaft in der Datenbank eingerichtet werden.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-221">However, the new integer primary key will not be set up as a SQL IDENTITY property in the database.</span></span> <span data-ttu-id="f2ecd-222">Sie müssen manuell die Id-Spalte als Identität festlegen.</span><span class="sxs-lookup"><span data-stu-id="f2ecd-222">You must manually set the Id column as an IDENTITY.</span></span>

<a id="other"></a>
## <a name="other-resources"></a><span data-ttu-id="f2ecd-223">Weitere Ressourcen</span><span class="sxs-lookup"><span data-stu-id="f2ecd-223">Other resources</span></span>

- [<span data-ttu-id="f2ecd-224">Übersicht über benutzerdefinierte Speicheranbieter für ASP.NET Identity</span><span class="sxs-lookup"><span data-stu-id="f2ecd-224">Overview of Custom Storage Providers for ASP.NET Identity</span></span>](overview-of-custom-storage-providers-for-aspnet-identity.md)
- [<span data-ttu-id="f2ecd-225">Migration von einer vorhandenen Website von SQL-Mitgliedschaft zu ASP.NET Identity</span><span class="sxs-lookup"><span data-stu-id="f2ecd-225">Migrating an Existing Website from SQL Membership to ASP.NET Identity</span></span>](../migrations/migrating-an-existing-website-from-sql-membership-to-aspnet-identity.md)
- [<span data-ttu-id="f2ecd-226">Migrieren von universellen Anbieterdaten Mitgliedschaft und zu ASP.NET Identity Benutzerprofile</span><span class="sxs-lookup"><span data-stu-id="f2ecd-226">Migrating Universal Provider Data for Membership and User Profiles to ASP.NET Identity</span></span>](../migrations/migrating-universal-provider-data-for-membership-and-user-profiles-to-aspnet-identity.md)
- <span data-ttu-id="f2ecd-227">[Beispielanwendung](https://aspnet.codeplex.com/SourceControl/latest#Samples/Identity/ChangePK/readme.txt) mit geänderten Primärschlüssel</span><span class="sxs-lookup"><span data-stu-id="f2ecd-227">[Sample application](https://aspnet.codeplex.com/SourceControl/latest#Samples/Identity/ChangePK/readme.txt) with changed primary key</span></span>