---
title: Dateiuploads in ASP.NET Core
author: ardalis
description: Verwendung von modellbindung und streaming zum Hochladen von Dateien in ASP.NET-MVC der Core.
keywords: ASP.NET Core, Dateiupload, Modell binden, IFormFile, streaming
ms.author: riande
manager: wpickett
ms.date: 07/05/2017
ms.topic: article
ms.assetid: ebc98159-a028-4a94-b06c-43981c79c6be
ms.technology: aspnet
ms.prod: asp.net-core
uid: mvc/models/file-uploads
ms.openlocfilehash: e8608a46d6688df8da6c665a25b6f4db5f480461
ms.sourcegitcommit: 9a9483aceb34591c97451997036a9120c3fe2baf
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 11/10/2017
---
# <a name="file-uploads-in-aspnet-core"></a><span data-ttu-id="d8d6e-104">Dateiuploads in ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="d8d6e-104">File uploads in ASP.NET Core</span></span>

<span data-ttu-id="d8d6e-105">Durch [Steve Smith](https://ardalis.com/)</span><span class="sxs-lookup"><span data-stu-id="d8d6e-105">By [Steve Smith](https://ardalis.com/)</span></span>

<span data-ttu-id="d8d6e-106">ASP.NET MVC-Aktionen unterstützen das Hochladen von einer oder mehreren Dateien, die mit einfachen Modell Bindung für kleinere Dateien oder streaming für größere Dateien.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-106">ASP.NET MVC actions support uploading of one or more files using simple model binding for smaller files or streaming for larger files.</span></span>

[<span data-ttu-id="d8d6e-107">Anzeigen oder Herunterladen des Beispiels von GitHub</span><span class="sxs-lookup"><span data-stu-id="d8d6e-107">View or download sample from GitHub</span></span>](https://github.com/aspnet/Docs/tree/master/aspnetcore/mvc/models/file-uploads/sample/FileUploadSample)

## <a name="uploading-small-files-with-model-binding"></a><span data-ttu-id="d8d6e-108">Hochladen von kleine Dateien mit modellbindung</span><span class="sxs-lookup"><span data-stu-id="d8d6e-108">Uploading small files with model binding</span></span>

<span data-ttu-id="d8d6e-109">Um kleine Dateien hochzuladen, können Sie eine mehrteilige HTML-Formular oder eine POST-Anforderung, die mit JavaScript zu erstellen.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-109">To upload small files, you can use a multi-part HTML form or construct a POST request using JavaScript.</span></span> <span data-ttu-id="d8d6e-110">Ein Beispiel-Formulars mit Razor, die mehrere hochgeladene Dateien unterstützt, wird unten gezeigt:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-110">An example form using Razor, which supports multiple uploaded files, is shown below:</span></span>

```html
<form method="post" enctype="multipart/form-data" asp-controller="UploadFiles" asp-action="Index">
    <div class="form-group">
        <div class="col-md-10">
            <p>Upload one or more files using this form:</p>
            <input type="file" name="files" multiple />
        </div>
    </div>
    <div class="form-group">
        <div class="col-md-10">
            <input type="submit" value="Upload" />
        </div>
    </div>
</form>
```

<span data-ttu-id="d8d6e-111">Damit Dateiuploads unterstützt wird, müssen die HTML-Formularen angeben einer `enctype` von `multipart/form-data`.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-111">In order to support file uploads, HTML forms must specify an `enctype` of `multipart/form-data`.</span></span> <span data-ttu-id="d8d6e-112">Die `files` input-Element oben gezeigten unterstützt mehrere Dateien hochladen.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-112">The `files` input element shown above supports uploading multiple files.</span></span> <span data-ttu-id="d8d6e-113">Lassen Sie die `multiple` Attribut für dieses Element input, nur eine einzelne Datei hochgeladen werden können.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-113">Omit the `multiple` attribute on this input element to allow just a single file to be uploaded.</span></span> <span data-ttu-id="d8d6e-114">Die oben genannten Markup rendert in einem Browser als:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-114">The above markup renders in a browser as:</span></span>

![Dateiupload-Formular](file-uploads/_static/upload-form.png)

<span data-ttu-id="d8d6e-116">Durch die einzelnen Dateien auf den Server hochgeladen zugegriffen werden können [Modell Bindung](xref:mvc/models/model-binding) mithilfe der [IFormFile](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.iformfile) Schnittstelle.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-116">The individual files uploaded to the server can be accessed through [Model Binding](xref:mvc/models/model-binding) using the [IFormFile](https://docs.microsoft.com/aspnet/core/api/microsoft.aspnetcore.http.iformfile) interface.</span></span> <span data-ttu-id="d8d6e-117">`IFormFile`weist die folgende Struktur:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-117">`IFormFile` has the following structure:</span></span>

```csharp
public interface IFormFile
{
    string ContentType { get; }
    string ContentDisposition { get; }
    IHeaderDictionary Headers { get; }
    long Length { get; }
    string Name { get; }
    string FileName { get; }
    Stream OpenReadStream();
    void CopyTo(Stream target);
    Task CopyToAsync(Stream target, CancellationToken cancellationToken = null);
}
```

> [!WARNING]
> <span data-ttu-id="d8d6e-118">Nicht abhängig ist, oder vertrauen der `FileName` Eigenschaft ohne Überprüfung.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-118">Don't rely on or trust the `FileName` property without validation.</span></span> <span data-ttu-id="d8d6e-119">Die `FileName` Eigenschaft sollte nur zu Anzeigezwecken verwendet werden.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-119">The `FileName` property should only be used for display purposes.</span></span>

<span data-ttu-id="d8d6e-120">Beim Hochladen von Dateien mit modellbindung und `IFormFile` -Schnittstelle, die Aktionsmethode kann entweder ein einzelnes akzeptieren `IFormFile` oder ein `IEnumerable<IFormFile>` (oder `List<IFormFile>`), die mehrere Dateien darstellt.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-120">When uploading files using model binding and the `IFormFile` interface, the action method can accept either a single `IFormFile` or an `IEnumerable<IFormFile>` (or `List<IFormFile>`) representing several files.</span></span> <span data-ttu-id="d8d6e-121">Das folgende Beispiel durchläuft einen oder mehrere hochgeladene Dateien, speichert sie in das lokale Dateisystem und gibt die Anzahl und Größe der Dateien hochgeladen.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-121">The following example loops through one or more uploaded files, saves them to the local file system, and returns the total number and size of files uploaded.</span></span>

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/UploadFilesController.cs?name=snippet1)]

<span data-ttu-id="d8d6e-122">Dateien, die hochgeladen wird, mithilfe der `IFormFile` Verfahren werden im Arbeitsspeicher oder auf einem Datenträger, auf dem Webserver vor der Verarbeitung gepuffert.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-122">Files uploaded using the `IFormFile` technique are buffered in memory or on disk on the web server before being processed.</span></span> <span data-ttu-id="d8d6e-123">In der Aktionsmethode die `IFormFile` Inhalt als Stream verfügbar sind.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-123">Inside the action method, the `IFormFile` contents are accessible as a stream.</span></span> <span data-ttu-id="d8d6e-124">Zusätzlich zu den im lokalen Dateisystem Dateien in gestreamt werden [Azure Blob-Speicher](https://azure.microsoft.com/documentation/articles/vs-storage-aspnet5-getting-started-blobs/) oder [Entity Framework](https://docs.microsoft.com/ef/core/index).</span><span class="sxs-lookup"><span data-stu-id="d8d6e-124">In addition to the local file system, files can be streamed to [Azure Blob storage](https://azure.microsoft.com/documentation/articles/vs-storage-aspnet5-getting-started-blobs/) or [Entity Framework](https://docs.microsoft.com/ef/core/index).</span></span>

<span data-ttu-id="d8d6e-125">Definieren Sie eine Eigenschaft vom Typ zum Speichern von binären Daten in einer Datenbank mithilfe von Entity Framework `byte[]` für die Entität:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-125">To store binary file data in a database using Entity Framework, define a property of type `byte[]` on the entity:</span></span>

```csharp
public class ApplicationUser : IdentityUser
{
    public byte[] AvatarImage { get; set; }
}
```

<span data-ttu-id="d8d6e-126">Geben Sie eine Viewmodel-Eigenschaft des Typs `IFormFile`:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-126">Specify a viewmodel property of type `IFormFile`:</span></span>

```csharp
public class RegisterViewModel
{
    // other properties omitted

    public IFormFile AvatarImage { get; set; }
}
```

> [!NOTE]
> <span data-ttu-id="d8d6e-127">`IFormFile`kann direkt als einen Aktionsmethodenparameter oder als Viewmodel-Eigenschaft verwendet werden wie oben gezeigt.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-127">`IFormFile` can be used directly as an action method parameter or as a viewmodel property, as shown above.</span></span>

<span data-ttu-id="d8d6e-128">Kopieren der `IFormFile` in einen Stream und speichern Sie sie auf das Bytearray:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-128">Copy the `IFormFile` to a stream and save it to the byte array:</span></span>

```csharp
// POST: /Account/Register
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Register(RegisterViewModel model)
{
    ViewData["ReturnUrl"] = returnUrl;
    if  (ModelState.IsValid)
    {
        var user = new ApplicationUser {
          UserName = model.Email,
          Email = model.Email
        };
        using (var memoryStream = new MemoryStream())
        {
            await model.AvatarImage.CopyToAsync(memoryStream);
            user.AvatarImage = memoryStream.ToArray();
        }
    // additional logic omitted
    
    // Don't rely on or trust the model.AvatarImage.FileName property 
    // without validation.
}
```

> [!NOTE]
> <span data-ttu-id="d8d6e-129">Seien Sie beim Speichern von Binärdaten in relationalen Datenbanken, wie sie die Leistung beeinträchtigen kann.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-129">Use caution when storing binary data in relational databases, as it can adversely impact performance.</span></span>

## <a name="uploading-large-files-with-streaming"></a><span data-ttu-id="d8d6e-130">Hochladen großer Dateien beim streaming</span><span class="sxs-lookup"><span data-stu-id="d8d6e-130">Uploading large files with streaming</span></span>

<span data-ttu-id="d8d6e-131">Wenn die Größe oder die Häufigkeit des Dateiuploads Ressourcenproblemen für die app verursacht wird, erwägen Sie streaming Hochladen der Datei, anstatt als Ganzes gepuffert wird, wie in der oben gezeigte Modell Bindung-Ansatz.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-131">If the size or frequency of file uploads is causing resource problems for the app, consider streaming the file upload rather than buffering it in its entirety, as the model binding approach shown above does.</span></span> <span data-ttu-id="d8d6e-132">Bei der Verwendung `IFormFile` und modellbindung ist eine viel einfachere Lösung streaming erfordert eine Reihe von Schritten, um ordnungsgemäß zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-132">While using `IFormFile` and model binding is a much simpler solution, streaming requires a number of steps to implement properly.</span></span>

> [!NOTE]
> <span data-ttu-id="d8d6e-133">Jede einzelne gepufferte Datei 64KB überschreiten wird aus dem RAM in eine temporäre Datei auf dem Datenträger auf dem Server verschoben werden.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-133">Any single buffered file exceeding 64KB will be moved from RAM to a temp file on disk on the server.</span></span> <span data-ttu-id="d8d6e-134">Die vom Dateiuploads verwendeten Ressourcen (Datenträger, RAM) richten sich nach der Anzahl und Größe der gleichzeitigen Dateiuploads.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-134">The resources (disk, RAM) used by file uploads depend on the number and size of concurrent file uploads.</span></span> <span data-ttu-id="d8d6e-135">Streaming nicht so viel über bezogen ist, wird über Dezimalstellen.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-135">Streaming is not so much about perf, it's about scale.</span></span> <span data-ttu-id="d8d6e-136">Wenn Sie versuchen, zu viele Uploads zu Puffern, stürzt der Site genügend Arbeitsspeicher oder Speicherplatz.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-136">If you try to buffer too many uploads, your site will crash when it runs out of memory or disk space.</span></span>

<span data-ttu-id="d8d6e-137">Im folgenden Beispiel wird veranschaulicht, wie mit JavaScript/Angular auf eine Controlleraktion gestreamt wird.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-137">The following example demonstrates using JavaScript/Angular to stream to a controller action.</span></span> <span data-ttu-id="d8d6e-138">Die Datei antiforgery-Token wird generiert, verwenden eines benutzerdefinierten Attributs für den Filter und HTTP-Header anstelle der im Anforderungstext übergeben.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-138">The file's antiforgery token is generated using a custom filter attribute and passed in HTTP headers instead of in the request body.</span></span> <span data-ttu-id="d8d6e-139">Da die Aktionsmethode die hochgeladenen Daten direkt verarbeitet wird, ist von einem anderen Filter wurden die modellbindung deaktiviert.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-139">Because the action method processes the uploaded data directly, model binding is disabled by another filter.</span></span> <span data-ttu-id="d8d6e-140">Innerhalb der Aktion werden das Formular Inhalt lesen, verwenden eine `MultipartReader`, die jedes einzelne liest `MultipartSection`, Verarbeiten der Datei oder Speichern des Inhalts nach Bedarf.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-140">Within the action, the form's contents are read using a `MultipartReader`, which reads each individual `MultipartSection`, processing the file or storing the contents as appropriate.</span></span> <span data-ttu-id="d8d6e-141">Sobald alle Abschnitte gelesen wurden, führt die Aktion einen eigenen wurden die modellbindung an.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-141">Once all sections have been read, the action performs its own model binding.</span></span>

<span data-ttu-id="d8d6e-142">Die erste Aktion wird das Formular geladen und ein antiforgery Token in einem Cookie gespeichert (über die `GenerateAntiforgeryTokenCookieForAjax` Attribut):</span><span class="sxs-lookup"><span data-stu-id="d8d6e-142">The initial action loads the form and saves an antiforgery token in a cookie (via the `GenerateAntiforgeryTokenCookieForAjax` attribute):</span></span>

```csharp
[HttpGet]
[GenerateAntiforgeryTokenCookieForAjax]
public IActionResult Index()
{
    return View();
}
```

<span data-ttu-id="d8d6e-143">Das Attribut verwendet, die ASP.NET Core integrierte [Antiforgery](xref:security/anti-request-forgery) Support, um ein Cookie mit einem Anforderungstoken festlegen:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-143">The attribute uses ASP.NET Core's built-in [Antiforgery](xref:security/anti-request-forgery) support to set a cookie with a request token:</span></span>

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/GenerateAntiforgeryTokenCookieForAjaxAttribute.cs?name=snippet1)]

<span data-ttu-id="d8d6e-144">Angular übergibt eine antiforgery Token automatisch in einen Anforderungsheader und mit dem Namen `X-XSRF-TOKEN`.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-144">Angular automatically passes an antiforgery token in a request header named `X-XSRF-TOKEN`.</span></span> <span data-ttu-id="d8d6e-145">Die ASP.NET-MVC-Anwendung Core ist so konfiguriert, dass zu diesem Header in der Konfiguration in finden Sie unter *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-145">The ASP.NET Core MVC app is configured to refer to this header in its configuration in *Startup.cs*:</span></span>

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Startup.cs?name=snippet1)]

<span data-ttu-id="d8d6e-146">Die `DisableFormValueModelBinding` Attribut, das weiter unten gezeigte wird verwendet, um die deaktiviert wurden die modellbindung für die `Upload` Aktionsmethode.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-146">The `DisableFormValueModelBinding` attribute, shown below, is used to disable model binding for the `Upload` action method.</span></span>

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Filters/DisableFormValueModelBindingAttribute.cs?name=snippet1)]

<span data-ttu-id="d8d6e-147">Da wurden die modellbindung deaktiviert ist, die `Upload` Aktionsmethode keine Parameter annehmen.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-147">Since model binding is disabled, the `Upload` action method doesn't accept parameters.</span></span> <span data-ttu-id="d8d6e-148">Es arbeitet direkt mit der `Request` Eigenschaft `ControllerBase`.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-148">It works directly with the `Request` property of `ControllerBase`.</span></span> <span data-ttu-id="d8d6e-149">Ein `MultipartReader` wird verwendet, um jedes Abschnitts lesen.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-149">A `MultipartReader` is used to read each section.</span></span> <span data-ttu-id="d8d6e-150">Die Datei wird mit einem GUID-Dateinamen gespeichert und befindet sich die Schlüssel/Wert-Daten in eine `KeyValueAccumulator`.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-150">The file is saved with a GUID filename and the key/value data is stored in a `KeyValueAccumulator`.</span></span> <span data-ttu-id="d8d6e-151">Sobald alle Abschnitte gelesen wurde, den Inhalt der `KeyValueAccumulator` werden verwendet, um die Daten des Formulars an einen Modelltyp binden.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-151">Once all sections have been read, the contents of the `KeyValueAccumulator` are used to bind the form data to a model type.</span></span>

<span data-ttu-id="d8d6e-152">Die vollständige `Upload` Methode wird unten gezeigt:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-152">The complete `Upload` method is shown below:</span></span>

[!INCLUDE [GetTempFileName](../../includes/GetTempFileName.md)]

[!code-csharp[Main](file-uploads/sample/FileUploadSample/Controllers/StreamingController.cs?name=snippet1)]

## <a name="troubleshooting"></a><span data-ttu-id="d8d6e-153">Problembehandlung</span><span class="sxs-lookup"><span data-stu-id="d8d6e-153">Troubleshooting</span></span>

<span data-ttu-id="d8d6e-154">Im folgenden finden einige allgemeine Probleme, die beim Arbeiten mit Hochladen von Dateien und deren möglichen Lösungen auftreten.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-154">Below are some common problems encountered when working with uploading files and their possible solutions.</span></span>

### <a name="unexpected-not-found-error-with-iis"></a><span data-ttu-id="d8d6e-155">Unerwarteter Fehler bei IIS wurde Nichtgefunden</span><span class="sxs-lookup"><span data-stu-id="d8d6e-155">Unexpected Not Found error with IIS</span></span>

<span data-ttu-id="d8d6e-156">Der folgende Fehler gibt an, das Hochladen der Datei überschreitet den Server konfigurierten `maxAllowedContentLength`:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-156">The following error indicates your file upload exceeds the server's configured `maxAllowedContentLength`:</span></span>

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

<span data-ttu-id="d8d6e-157">Die Standardeinstellung ist `30000000`, also ungefähr 28.6 MB.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-157">The default setting is `30000000`, which is approximately 28.6MB.</span></span> <span data-ttu-id="d8d6e-158">Der Wert kann angepasst werden, indem bearbeiten *"Web.config"*:</span><span class="sxs-lookup"><span data-stu-id="d8d6e-158">The value can be customized by editing *web.config*:</span></span>

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <!-- This will handle requests up to 50MB -->
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

<span data-ttu-id="d8d6e-159">Diese Einstellung gilt nur für IIS.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-159">This setting only applies to IIS.</span></span> <span data-ttu-id="d8d6e-160">Das Verhalten auftreten nicht standardmäßig, wenn auf Kestrel hosten.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-160">The behavior doesn't occur by default when hosting on Kestrel.</span></span> <span data-ttu-id="d8d6e-161">Weitere Informationen finden Sie unter [Anforderungslimits \<RequestLimits\>](https://docs.microsoft.com/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).</span><span class="sxs-lookup"><span data-stu-id="d8d6e-161">For more information, see [Request Limits \<requestLimits\>](https://docs.microsoft.com/iis/configuration/system.webServer/security/requestFiltering/requestLimits/).</span></span>

### <a name="null-reference-exception-with-iformfile"></a><span data-ttu-id="d8d6e-162">NULL-Verweisausnahme auftreten mit IFormFile</span><span class="sxs-lookup"><span data-stu-id="d8d6e-162">Null Reference Exception with IFormFile</span></span>

<span data-ttu-id="d8d6e-163">Wenn Ihr Controller annehmende ist hochgeladenen Dateien mithilfe von `IFormFile` , aber Sie feststellen, dass der Wert immer null ist, vergewissern Sie sich, dass das HTML-Formular angegeben ist ein `enctype` Wert `multipart/form-data`.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-163">If your controller is accepting uploaded files using `IFormFile` but you find that the value is always null, confirm that your HTML form is specifying an `enctype` value of `multipart/form-data`.</span></span> <span data-ttu-id="d8d6e-164">Wenn dieses Attribut nicht festgelegt ist die `<form>` -Element, das Hochladen der Datei wird nicht ausgeführt und die Grenze `IFormFile` Argumente werden null sein.</span><span class="sxs-lookup"><span data-stu-id="d8d6e-164">If this attribute is not set on the `<form>` element, the file upload will not occur and any bound `IFormFile` arguments will be null.</span></span>