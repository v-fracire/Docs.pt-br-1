---
title: Registro em log no ASP.NET Core
author: ardalis
description: Saiba mais sobre a estrutura de registros no ASP.NET Core. Descubra os provedores de log internos e saiba mais sobre os provedores de terceiros populares.
keywords: ASP.NET Core, registro em log, provedores de log, Microsoft.Extensions.Logging, ILogger, ILoggerFactory, LogLevel, WithFilter, TraceSource, EventLog, EventSource, escopos
ms.author: tdykstra
manager: wpickett
ms.date: 12/15/2017
ms.topic: article
ms.technology: aspnet
ms.prod: asp.net-core
uid: fundamentals/logging/index
ms.openlocfilehash: 3eb167c961b8d089d508ef5622db6ae1cdd99088
ms.sourcegitcommit: 12e5194936b7e820efc5505a2d5d4f84e88eb5ef
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/11/2018
---
# <a name="introduction-to-logging-in-aspnet-core"></a><span data-ttu-id="5f08b-105">Introdução ao registro em log no ASP.NET Core</span><span class="sxs-lookup"><span data-stu-id="5f08b-105">Introduction to logging in ASP.NET Core</span></span>

<span data-ttu-id="5f08b-106">Por [Steve Smith](https://ardalis.com/) e [Tom Dykstra](https://github.com/tdykstra)</span><span class="sxs-lookup"><span data-stu-id="5f08b-106">By [Steve Smith](https://ardalis.com/) and [Tom Dykstra](https://github.com/tdykstra)</span></span>

<span data-ttu-id="5f08b-107">O ASP.NET Core dá suporte a uma API de registro em log que funciona com uma variedade de provedores de logs.</span><span class="sxs-lookup"><span data-stu-id="5f08b-107">ASP.NET Core supports a logging API that works with a variety of logging providers.</span></span> <span data-ttu-id="5f08b-108">Os provedores internos permitem que você envie logs para um ou mais destinos e você pode se conectar a uma estrutura de registros de terceiros.</span><span class="sxs-lookup"><span data-stu-id="5f08b-108">Built-in providers let you send logs to one or more destinations, and you can plug in a third-party logging framework.</span></span> <span data-ttu-id="5f08b-109">Este artigo mostra como usar a API de registro em log e os provedores internos em seu código.</span><span class="sxs-lookup"><span data-stu-id="5f08b-109">This article shows how to use the built-in logging API and providers in your code.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-110">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-110">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="5f08b-111">[Exibir ou baixar código de exemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample2) ([como baixar](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="5f08b-111">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample2) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-112">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-112">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="5f08b-113">[Exibir ou baixar código de exemplo](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample) ([como baixar](xref:tutorials/index#how-to-download-a-sample))</span><span class="sxs-lookup"><span data-stu-id="5f08b-113">[View or download sample code](https://github.com/aspnet/Docs/tree/master/aspnetcore/fundamentals/logging/index/sample) ([how to download](xref:tutorials/index#how-to-download-a-sample))</span></span>

---

## <a name="how-to-create-logs"></a><span data-ttu-id="5f08b-114">Como criar logs</span><span class="sxs-lookup"><span data-stu-id="5f08b-114">How to create logs</span></span>

<span data-ttu-id="5f08b-115">Para criar logs, obtenha um objeto `ILogger` do contêiner de [injeção de dependência](xref:fundamentals/dependency-injection):</span><span class="sxs-lookup"><span data-stu-id="5f08b-115">To create logs, get an `ILogger` object from the [dependency injection](xref:fundamentals/dependency-injection) container:</span></span>

[!code-csharp[](index/sample/Controllers/TodoController.cs?name=snippet_LoggerDI&highlight=7)]

<span data-ttu-id="5f08b-116">Em seguida, chame métodos de registro em log nesse objeto de agente:</span><span class="sxs-lookup"><span data-stu-id="5f08b-116">Then call logging methods on that logger object:</span></span>

[!code-csharp[](index/sample/Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

<span data-ttu-id="5f08b-117">Este exemplo cria logs com a classe `TodoController` como a *categoria*.</span><span class="sxs-lookup"><span data-stu-id="5f08b-117">This example creates logs with the `TodoController` class as the *category*.</span></span> <span data-ttu-id="5f08b-118">As categorias serão explicadas [posteriormente neste artigo](#log-category).</span><span class="sxs-lookup"><span data-stu-id="5f08b-118">Categories are explained [later in this article](#log-category).</span></span>

<span data-ttu-id="5f08b-119">O ASP.NET Core não fornece métodos de agente assíncronos porque o registro em log deve ser tão rápido que não valeria a pena usar assíncronos.</span><span class="sxs-lookup"><span data-stu-id="5f08b-119">ASP.NET Core does not provide async logger methods because logging should be so fast that it isn't worth the cost of using async.</span></span> <span data-ttu-id="5f08b-120">Se você estiver em uma situação em que isso não é verdadeiro, considere alterar a maneira como você faz logs.</span><span class="sxs-lookup"><span data-stu-id="5f08b-120">If you're in a situation where that's not true, consider changing the way you log.</span></span> <span data-ttu-id="5f08b-121">Se seu armazenamento de dados é lento, grave as mensagens de log em um repositório rápido primeiro e, depois, mova-as para um repositório lento.</span><span class="sxs-lookup"><span data-stu-id="5f08b-121">If your data store is slow, write the log messages to a fast store first, then move them to a slow store later.</span></span> <span data-ttu-id="5f08b-122">Por exemplo, registre em uma fila de mensagens que seja lida e persistida em um armazenamento lento por outro processo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-122">For example, log to a message queue that is read and persisted to slow storage by another process.</span></span>

## <a name="how-to-add-providers"></a><span data-ttu-id="5f08b-123">Como adicionar provedores</span><span class="sxs-lookup"><span data-stu-id="5f08b-123">How to add providers</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-124">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-124">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="5f08b-125">Um provedor de logs obtém as mensagens que você cria com um objeto `ILogger` e as exibe ou armazena.</span><span class="sxs-lookup"><span data-stu-id="5f08b-125">A logging provider takes the messages that you create with an `ILogger` object and displays or stores them.</span></span> <span data-ttu-id="5f08b-126">Por exemplo, o provedor Console exibe as mensagens no console e o provedor do Serviço de Aplicativo do Azure pode armazená-las no armazenamento de blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="5f08b-126">For example, the Console provider displays messages on the console, and the Azure App Service provider can store them in Azure blob storage.</span></span>

<span data-ttu-id="5f08b-127">Para usar um provedor, chame o método de extensão `Add<ProviderName>` do provedor no *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="5f08b-127">To use a provider, call the provider's `Add<ProviderName>` extension method in *Program.cs*:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_ExpandDefault&highlight=16,17)]

<span data-ttu-id="5f08b-128">O modelo de projeto padrão permite o registro em log com o método [CreateDefaultBuilder](https://docs.microsoft.com/ dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder?view=aspnetcore-2.0#Microsoft_AspNetCore_WebHost_CreateDefaultBuilder_System_String___):</span><span class="sxs-lookup"><span data-stu-id="5f08b-128">The default project template enables logging with the [CreateDefaultBuilder](https://docs.microsoft.com/ dotnet/api/microsoft.aspnetcore.webhost.createdefaultbuilder?view=aspnetcore-2.0#Microsoft_AspNetCore_WebHost_CreateDefaultBuilder_System_String___) method:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_TemplateCode&highlight=7)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-129">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-129">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="5f08b-130">Um provedor de logs obtém as mensagens que você cria com um objeto `ILogger` e as exibe ou armazena.</span><span class="sxs-lookup"><span data-stu-id="5f08b-130">A logging provider takes the messages that you create with an `ILogger` object and displays or stores them.</span></span> <span data-ttu-id="5f08b-131">Por exemplo, o provedor Console exibe as mensagens no console e o provedor do Serviço de Aplicativo do Azure pode armazená-las no armazenamento de blobs do Azure.</span><span class="sxs-lookup"><span data-stu-id="5f08b-131">For example, the Console provider displays messages on the console, and the Azure App Service provider can store them in Azure blob storage.</span></span>

<span data-ttu-id="5f08b-132">Para usar um provedor, instale o pacote NuGet e chame o método de extensão do provedor em uma instância de `ILoggerFactory`, conforme mostrado no exemplo a seguir.</span><span class="sxs-lookup"><span data-stu-id="5f08b-132">To use a provider, install its NuGet package and call the provider's extension method on an instance of `ILoggerFactory`, as shown in the following example.</span></span>

[!code-csharp[](index/sample//Startup.cs?name=snippet_AddConsoleAndDebug&highlight=3,5-7)]

<span data-ttu-id="5f08b-133">A [DI](xref:fundamentals/dependency-injection) (injeção de dependência) do ASP.NET Core fornece a instância de `ILoggerFactory`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-133">ASP.NET Core [dependency injection](xref:fundamentals/dependency-injection) (DI) provides the `ILoggerFactory` instance.</span></span> <span data-ttu-id="5f08b-134">Os métodos de extensão `AddConsole` e `AddDebug` são definidos nos pacotes [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/) e [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug/).</span><span class="sxs-lookup"><span data-stu-id="5f08b-134">The `AddConsole` and `AddDebug` extension methods are defined in the [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console/) and [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug/) packages.</span></span> <span data-ttu-id="5f08b-135">Cada método de extensão chama o método `ILoggerFactory.AddProvider`, passando uma instância do provedor.</span><span class="sxs-lookup"><span data-stu-id="5f08b-135">Each extension method calls the `ILoggerFactory.AddProvider` method, passing in an instance of the provider.</span></span> 

> [!NOTE]
> <span data-ttu-id="5f08b-136">O aplicativo de exemplo deste artigo adiciona provedores de log no método `Configure` da classe `Startup`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-136">The sample application for this article adds logging providers in the `Configure` method of the `Startup` class.</span></span> <span data-ttu-id="5f08b-137">Se você quiser obter saída de log de código que é executado anteriormente, adicione provedores de log no construtor de classe `Startup`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-137">If you want to get log output from code that executes earlier, add logging providers in the `Startup` class constructor instead.</span></span> 

---

<span data-ttu-id="5f08b-138">Você encontrará informações sobre cada [provedor de log interno](#built-in-logging-providers) e links para [provedores de log de terceiros](#third-party-logging-providers) mais adiante no artigo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-138">You'll find information about each [built-in logging provider](#built-in-logging-providers) and links to [third-party logging providers](#third-party-logging-providers) later in the article.</span></span>

## <a name="sample-logging-output"></a><span data-ttu-id="5f08b-139">Exemplo de saída de registro em log</span><span class="sxs-lookup"><span data-stu-id="5f08b-139">Sample logging output</span></span>

<span data-ttu-id="5f08b-140">Com o código de exemplo mostrado na seção anterior, você verá logs no console ao executar através da linha de comando.</span><span class="sxs-lookup"><span data-stu-id="5f08b-140">With the sample code shown in the preceding section, you'll see logs in the console when you run from the command line.</span></span> <span data-ttu-id="5f08b-141">Aqui está um exemplo da saída do console:</span><span class="sxs-lookup"><span data-stu-id="5f08b-141">Here's an example of console output:</span></span>

```console
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:5000/api/todo/0
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
info: TodoApi.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 42.9286ms
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 148.889ms 404
```
 
<span data-ttu-id="5f08b-142">Esses logs foram criados acessando `http://localhost:5000/api/todo/0`, que dispara a execução das duas chamadas `ILogger` mostradas na seção anterior.</span><span class="sxs-lookup"><span data-stu-id="5f08b-142">These logs were created by going to `http://localhost:5000/api/todo/0`, which triggers execution of both `ILogger` calls shown in the preceding section.</span></span>

<span data-ttu-id="5f08b-143">Aqui está um exemplo de como os mesmos logs aparecem na janela Depuração quando você executa o aplicativo de exemplo no Visual Studio:</span><span class="sxs-lookup"><span data-stu-id="5f08b-143">Here's an example of the same logs as they appear in the Debug window when you run the sample application in Visual Studio:</span></span>

```console
Microsoft.AspNetCore.Hosting.Internal.WebHost:Information: Request starting HTTP/1.1 GET http://localhost:53104/api/todo/0  
Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker:Information: Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
TodoApi.Controllers.TodoController:Information: Getting item 0
TodoApi.Controllers.TodoController:Warning: GetById(0) NOT FOUND
Microsoft.AspNetCore.Mvc.StatusCodeResult:Information: Executing HttpStatusCodeResult, setting HTTP status code 404
Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker:Information: Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 152.5657ms
Microsoft.AspNetCore.Hosting.Internal.WebHost:Information: Request finished in 316.3195ms 404 
```

<span data-ttu-id="5f08b-144">Os logs criados pelas chamadas `ILogger` mostradas na seção anterior começam com "TodoApi.Controllers.TodoController".</span><span class="sxs-lookup"><span data-stu-id="5f08b-144">The logs that were created by the `ILogger` calls shown in the preceding section begin with "TodoApi.Controllers.TodoController".</span></span> <span data-ttu-id="5f08b-145">Os logs que começam com categorias "Microsoft" são do ASP.NET Core.</span><span class="sxs-lookup"><span data-stu-id="5f08b-145">The logs that begin with "Microsoft" categories are from ASP.NET Core.</span></span> <span data-ttu-id="5f08b-146">O próprio ASP.NET Core e o código do aplicativo estão usando a mesma API de registro em log e os mesmos provedores de log.</span><span class="sxs-lookup"><span data-stu-id="5f08b-146">ASP.NET Core itself and your application code are using the same logging API and the same logging providers.</span></span>

<span data-ttu-id="5f08b-147">O restante deste artigo explica alguns detalhes e opções para registro em log.</span><span class="sxs-lookup"><span data-stu-id="5f08b-147">The remainder of this article explains some details and options for logging.</span></span>

## <a name="nuget-packages"></a><span data-ttu-id="5f08b-148">Pacotes NuGet</span><span class="sxs-lookup"><span data-stu-id="5f08b-148">NuGet packages</span></span>

<span data-ttu-id="5f08b-149">As interfaces `ILogger` e `ILoggerFactory` estão em [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/) e as implementações padrão para elas estão em [Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging/).</span><span class="sxs-lookup"><span data-stu-id="5f08b-149">The `ILogger` and `ILoggerFactory` interfaces are in [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/), and default implementations for them are in [Microsoft.Extensions.Logging](https://www.nuget.org/packages/Microsoft.Extensions.Logging/).</span></span>

## <a name="log-category"></a><span data-ttu-id="5f08b-150">Categoria de log</span><span class="sxs-lookup"><span data-stu-id="5f08b-150">Log category</span></span>

<span data-ttu-id="5f08b-151">Um *categoria* é incluída com cada log que você cria.</span><span class="sxs-lookup"><span data-stu-id="5f08b-151">A *category* is included with each log that you create.</span></span> <span data-ttu-id="5f08b-152">Você especifica a categoria ao criar um objeto `ILogger`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-152">You specify the category when you create an `ILogger` object.</span></span> <span data-ttu-id="5f08b-153">A categoria pode ser qualquer cadeia de caracteres, mas uma convenção é usar o nome totalmente qualificado da classe da qual os logs são gravados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-153">The category may be any string, but a convention is to use the fully qualified name of the class from which the logs are written.</span></span> <span data-ttu-id="5f08b-154">Por exemplo: "TodoApi.Controllers.TodoController".</span><span class="sxs-lookup"><span data-stu-id="5f08b-154">For example: "TodoApi.Controllers.TodoController".</span></span>

<span data-ttu-id="5f08b-155">Você pode especificar a categoria como uma cadeia de caracteres ou usar um método de extensão que deriva a categoria do tipo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-155">You can specify the category as a string or use an extension method that derives the category from the type.</span></span> <span data-ttu-id="5f08b-156">Para especificar a categoria como uma cadeia de caracteres, chame `CreateLogger` em uma instância de `ILoggerFactory`, conforme mostrado abaixo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-156">To specify the category as a string, call `CreateLogger` on an `ILoggerFactory` instance, as shown below.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CreateLogger&highlight=7,10)]

<span data-ttu-id="5f08b-157">Na maioria das vezes será mais fácil usar `ILogger<T>`, conforme mostrado no exemplo a seguir.</span><span class="sxs-lookup"><span data-stu-id="5f08b-157">Most of the time, it will be easier to use `ILogger<T>`, as in the following example.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_LoggerDI&highlight=7)]

<span data-ttu-id="5f08b-158">Isso é equivalente a chamar `CreateLogger` com o nome de tipo totalmente qualificado de `T`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-158">This is equivalent to calling `CreateLogger` with the fully qualified type name of `T`.</span></span>

## <a name="log-level"></a><span data-ttu-id="5f08b-159">Nível de log</span><span class="sxs-lookup"><span data-stu-id="5f08b-159">Log level</span></span>

<span data-ttu-id="5f08b-160">Sempre que você grava um log, você especifica o [LogLevel](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel).</span><span class="sxs-lookup"><span data-stu-id="5f08b-160">Each time you write a log, you specify its [LogLevel](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel).</span></span> <span data-ttu-id="5f08b-161">O nível de log indica o grau de gravidade ou importância.</span><span class="sxs-lookup"><span data-stu-id="5f08b-161">The log level indicates the degree of severity or importance.</span></span> <span data-ttu-id="5f08b-162">Por exemplo, você pode gravar um log `Information` quando um método é finalizado normalmente, um log `Warning` quando um método retorna um código de retorno 404 e um log `Error` ao capturar uma exceção inesperada.</span><span class="sxs-lookup"><span data-stu-id="5f08b-162">For example, you might write an `Information` log when a method ends normally, a `Warning` log when a method returns a 404 return code, and an `Error` log when you catch an unexpected exception.</span></span>

<span data-ttu-id="5f08b-163">No exemplo de código a seguir, os nomes dos métodos (por exemplo, `LogWarning`) especificam o nível de log.</span><span class="sxs-lookup"><span data-stu-id="5f08b-163">In the following code example, the names of the methods (for example, `LogWarning`) specify the log level.</span></span> <span data-ttu-id="5f08b-164">O primeiro parâmetro é a [ID de evento de log](#log-event-id).</span><span class="sxs-lookup"><span data-stu-id="5f08b-164">The first parameter is the [Log event ID](#log-event-id).</span></span> <span data-ttu-id="5f08b-165">O segundo parâmetro é um [modelo de mensagem](#log-message-template) com espaços reservados para valores de argumento fornecidos pelos parâmetros de método restantes.</span><span class="sxs-lookup"><span data-stu-id="5f08b-165">The second parameter is a [message template](#log-message-template) with placeholders for argument values provided by the remaining method parameters.</span></span> <span data-ttu-id="5f08b-166">Os parâmetros de método serão explicados com mais detalhes posteriormente neste artigo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-166">The method parameters are explained in more detail later in this article.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

<span data-ttu-id="5f08b-167">Os métodos de log que incluem o nível no nome do método são [métodos de extensão para ILogger](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loggerextensions).</span><span class="sxs-lookup"><span data-stu-id="5f08b-167">Log methods that include the level in the method name are [extension methods for ILogger](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loggerextensions).</span></span> <span data-ttu-id="5f08b-168">Nos bastidores, esses métodos chamam um método `Log` que recebe um parâmetro `LogLevel`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-168">Behind the scenes, these methods call a `Log` method that takes a `LogLevel` parameter.</span></span> <span data-ttu-id="5f08b-169">Você pode chamar o método `Log` diretamente em vez de um desses métodos de extensão, mas a sintaxe é relativamente complicada.</span><span class="sxs-lookup"><span data-stu-id="5f08b-169">You can call the `Log` method directly rather than one of these extension methods, but the syntax is relatively complicated.</span></span> <span data-ttu-id="5f08b-170">Para obter mais informações, consulte a [interface ILogger](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.ilogger) e o [código-fonte de extensões de agente](https://github.com/aspnet/Logging/blob/master/src/Microsoft.Extensions.Logging.Abstractions/LoggerExtensions.cs).</span><span class="sxs-lookup"><span data-stu-id="5f08b-170">For more information, see the [ILogger interface](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.ilogger) and the [logger extensions source code](https://github.com/aspnet/Logging/blob/master/src/Microsoft.Extensions.Logging.Abstractions/LoggerExtensions.cs).</span></span>

<span data-ttu-id="5f08b-171">O ASP.NET Core define os seguintes [níveis de log](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel), ordenados aqui da menor para a maior gravidade.</span><span class="sxs-lookup"><span data-stu-id="5f08b-171">ASP.NET Core defines the following [log levels](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.loglevel), ordered here from least to highest severity.</span></span>

* <span data-ttu-id="5f08b-172">Trace = 0</span><span class="sxs-lookup"><span data-stu-id="5f08b-172">Trace = 0</span></span>

  <span data-ttu-id="5f08b-173">Para informações valiosas somente para um desenvolvedor que esteja depurando um problema.</span><span class="sxs-lookup"><span data-stu-id="5f08b-173">For information that is valuable only to a developer debugging an issue.</span></span> <span data-ttu-id="5f08b-174">Essas mensagens podem conter dados confidenciais de aplicativos e, portanto, não devem ser habilitadas em um ambiente de produção.</span><span class="sxs-lookup"><span data-stu-id="5f08b-174">These messages may contain sensitive application data and so should not be enabled in a production environment.</span></span> <span data-ttu-id="5f08b-175">*Desabilitado por padrão.*</span><span class="sxs-lookup"><span data-stu-id="5f08b-175">*Disabled by default.*</span></span> <span data-ttu-id="5f08b-176">Exemplo: `Credentials: {"User":"someuser", "Password":"P@ssword"}`</span><span class="sxs-lookup"><span data-stu-id="5f08b-176">Example: `Credentials: {"User":"someuser", "Password":"P@ssword"}`</span></span>

* <span data-ttu-id="5f08b-177">Debug = 1</span><span class="sxs-lookup"><span data-stu-id="5f08b-177">Debug = 1</span></span>

  <span data-ttu-id="5f08b-178">Para informações que tenham utilidade de curto prazo durante o desenvolvimento e a depuração.</span><span class="sxs-lookup"><span data-stu-id="5f08b-178">For information that has short-term usefulness during development and debugging.</span></span> <span data-ttu-id="5f08b-179">Exemplo: `Entering method Configure with flag set to true.` Você normalmente não habilitaria logs de nível `Debug` em produção, a menos que estivesse solucionando problemas, devido ao alto volume de logs.</span><span class="sxs-lookup"><span data-stu-id="5f08b-179">Example: `Entering method Configure with flag set to true.` You typically would not enable `Debug` level logs in production unless you are troubleshooting, due to the high volume of logs.</span></span>

* <span data-ttu-id="5f08b-180">Information = 2</span><span class="sxs-lookup"><span data-stu-id="5f08b-180">Information = 2</span></span>

  <span data-ttu-id="5f08b-181">Para rastrear o fluxo geral do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-181">For tracking the general flow of the application.</span></span> <span data-ttu-id="5f08b-182">Esses logs normalmente têm algum valor a longo prazo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-182">These logs typically have some long-term value.</span></span> <span data-ttu-id="5f08b-183">Exemplo: `Request received for path /api/todo`</span><span class="sxs-lookup"><span data-stu-id="5f08b-183">Example: `Request received for path /api/todo`</span></span>

* <span data-ttu-id="5f08b-184">Warning = 3</span><span class="sxs-lookup"><span data-stu-id="5f08b-184">Warning = 3</span></span>

  <span data-ttu-id="5f08b-185">Para eventos anormais ou inesperados no fluxo de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-185">For abnormal or unexpected events in the application flow.</span></span> <span data-ttu-id="5f08b-186">Eles podem incluir erros ou outras condições que não fazem com que o aplicativo pare, mas que talvez precisem ser investigados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-186">These may include errors or other conditions that do not cause the application to stop, but which may need to be investigated.</span></span> <span data-ttu-id="5f08b-187">Exceções manipuladas são um local comum para usar o nível de log `Warning`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-187">Handled exceptions are a common place to use the `Warning` log level.</span></span> <span data-ttu-id="5f08b-188">Exemplo: `FileNotFoundException for file quotes.txt.`</span><span class="sxs-lookup"><span data-stu-id="5f08b-188">Example: `FileNotFoundException for file quotes.txt.`</span></span>

* <span data-ttu-id="5f08b-189">Error = 4</span><span class="sxs-lookup"><span data-stu-id="5f08b-189">Error = 4</span></span>

  <span data-ttu-id="5f08b-190">Para erros e exceções que não podem ser manipulados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-190">For errors and exceptions that cannot be handled.</span></span> <span data-ttu-id="5f08b-191">Essas mensagens indicam uma falha na atividade ou na operação atual (como a solicitação HTTP atual) e não uma falha em todo o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-191">These messages indicate a failure in the current activity or operation (such as the current HTTP request), not an application-wide failure.</span></span> <span data-ttu-id="5f08b-192">Mensagem de log de exemplo:`Cannot insert record due to duplicate key violation.`</span><span class="sxs-lookup"><span data-stu-id="5f08b-192">Example log message: `Cannot insert record due to duplicate key violation.`</span></span>

* <span data-ttu-id="5f08b-193">Critical = 5</span><span class="sxs-lookup"><span data-stu-id="5f08b-193">Critical = 5</span></span>

  <span data-ttu-id="5f08b-194">Para falhas que exigem atenção imediata.</span><span class="sxs-lookup"><span data-stu-id="5f08b-194">For failures that require immediate attention.</span></span> <span data-ttu-id="5f08b-195">Exemplos: cenários de perda de dados, espaço em disco insuficiente.</span><span class="sxs-lookup"><span data-stu-id="5f08b-195">Examples: data loss scenarios, out of disk space.</span></span>

<span data-ttu-id="5f08b-196">Você pode usar o nível de log para controlar a quantidade de saída de log que é gravada em uma mídia de armazenamento específica ou em uma janela de exibição.</span><span class="sxs-lookup"><span data-stu-id="5f08b-196">You can use the log level to control how much log output is written to a particular storage medium or display window.</span></span> <span data-ttu-id="5f08b-197">Por exemplo, em produção, você pode desejar que todos os logs de nível `Information` e inferior vão para um armazenamento de dados de volume e que todos os logs de nível `Warning` e superior vão para um armazenamento de dados de valor.</span><span class="sxs-lookup"><span data-stu-id="5f08b-197">For example, in production you might want all logs of `Information` level and lower to go to a volume data store, and all logs of `Warning` level and higher to go to a value data store.</span></span> <span data-ttu-id="5f08b-198">Durante o desenvolvimento, você normalmente poderia enviar os logs de gravidade `Warning` ou mais alta para o console.</span><span class="sxs-lookup"><span data-stu-id="5f08b-198">During development, you might normally send logs of `Warning` or higher severity to the console.</span></span> <span data-ttu-id="5f08b-199">Então, quando precisar solucionar problemas, você poderá adicionar o nível `Debug`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-199">Then when you need to troubleshoot, you can add `Debug` level.</span></span> <span data-ttu-id="5f08b-200">A seção [Filtragem de log](#log-filtering) mais adiante neste artigo explicará como controlar os níveis de log que um provedor manipula.</span><span class="sxs-lookup"><span data-stu-id="5f08b-200">The [Log filtering](#log-filtering) section later in this article explains how to control which log levels a provider handles.</span></span>

<span data-ttu-id="5f08b-201">A estrutura do ASP.NET Core grava logs de nível `Debug` para eventos de estrutura.</span><span class="sxs-lookup"><span data-stu-id="5f08b-201">The ASP.NET Core framework writes `Debug` level logs for framework events.</span></span> <span data-ttu-id="5f08b-202">Os exemplos de log anteriores neste artigo excluíram logs abaixo do nível `Information`, portanto, os logs de nível `Debug` não foram mostrados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-202">The log examples earlier in this article excluded logs below `Information` level, so no `Debug` level logs were shown.</span></span> <span data-ttu-id="5f08b-203">Aqui está um exemplo de logs do console, caso você execute o aplicativo de exemplo configurado para mostrar logs de nível `Debug` e superiores para o provedor de console.</span><span class="sxs-lookup"><span data-stu-id="5f08b-203">Here's an example of console logs if you run the sample application configured to show `Debug` and higher logs for the console provider.</span></span>

```console
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:62555/api/todo/0
dbug: Microsoft.AspNetCore.Routing.Tree.TreeRouter[1]
      Request successfully matched the route with name 'GetTodo' and template 'api/Todo/{id}'.
dbug: Microsoft.AspNetCore.Mvc.Internal.ActionSelector[2]
      Action 'TodoApi.Controllers.TodoController.Update (TodoApi)' with id '089d59b6-92ec-472d-b552-cc613dfd625d' did not match the constraint 'Microsoft.AspNetCore.Mvc.Internal.HttpMethodActionConstraint'
dbug: Microsoft.AspNetCore.Mvc.Internal.ActionSelector[2]
      Action 'TodoApi.Controllers.TodoController.Delete (TodoApi)' with id 'f3476abe-4bd9-4ad3-9261-3ead09607366' did not match the constraint 'Microsoft.AspNetCore.Mvc.Internal.HttpMethodActionConstraint'
dbug: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action TodoApi.Controllers.TodoController.GetById (TodoApi)
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[1]
      Executing action method TodoApi.Controllers.TodoController.GetById (TodoApi) with arguments (0) - ModelState is Valid
info: TodoApi.Controllers.TodoController[1002]
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      GetById(0) NOT FOUND
dbug: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action method TodoApi.Controllers.TodoController.GetById (TodoApi), returned result Microsoft.AspNetCore.Mvc.NotFoundResult.
info: Microsoft.AspNetCore.Mvc.StatusCodeResult[1]
      Executing HttpStatusCodeResult, setting HTTP status code 404
info: Microsoft.AspNetCore.Mvc.Internal.ControllerActionInvoker[2]
      Executed action TodoApi.Controllers.TodoController.GetById (TodoApi) in 0.8788ms
dbug: Microsoft.AspNetCore.Server.Kestrel[9]
      Connection id "0HL6L7NEFF2QD" completed keep alive response.
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[2]
      Request finished in 2.7286ms 404
```

## <a name="log-event-id"></a><span data-ttu-id="5f08b-204">ID de evento de log</span><span class="sxs-lookup"><span data-stu-id="5f08b-204">Log event ID</span></span>

<span data-ttu-id="5f08b-205">Sempre que você grava um log, você pode especificar uma *ID de evento*.</span><span class="sxs-lookup"><span data-stu-id="5f08b-205">Each time you write a log, you can specify an *event ID*.</span></span> <span data-ttu-id="5f08b-206">O aplicativo de exemplo faz isso usando uma classe `LoggingEvents` definida localmente:</span><span class="sxs-lookup"><span data-stu-id="5f08b-206">The sample app does this by using a locally-defined `LoggingEvents` class:</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

[!code-csharp[](index/sample//Core/LoggingEvents.cs?name=snippet_LoggingEvents)]

<span data-ttu-id="5f08b-207">Uma ID de evento é um valor inteiro que pode ser usado para associar um conjunto de eventos registrados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-207">An event ID is an integer value that you can use to associate a set of logged events with one another.</span></span> <span data-ttu-id="5f08b-208">Por exemplo, um log para adicionar um item a um carrinho de compras pode ser ter a ID de evento 1000 e um log para concluir uma compra pode ter a ID de evento 1001.</span><span class="sxs-lookup"><span data-stu-id="5f08b-208">For instance, a log for adding an item to a shopping cart could be event ID 1000 and a log for completing a purchase could be event ID 1001.</span></span>

<span data-ttu-id="5f08b-209">Na saída do registro em log, a ID do evento pode ser armazenada em um campo ou incluída na mensagem de texto, dependendo do provedor.</span><span class="sxs-lookup"><span data-stu-id="5f08b-209">In logging output, the event ID may be stored in a field or included in the text message, depending on the provider.</span></span> <span data-ttu-id="5f08b-210">O provedor Depuração não mostra as IDs de evento, mas o provedor do console sim, entre colchetes depois da categoria:</span><span class="sxs-lookup"><span data-stu-id="5f08b-210">The Debug provider doesn't show event IDs, but the console provider shows them in brackets after the category:</span></span>

```console
info: TodoApi.Controllers.TodoController[1002]
      Getting item invalidid
warn: TodoApi.Controllers.TodoController[4000]
      GetById(invalidid) NOT FOUND
```

## <a name="log-message-template"></a><span data-ttu-id="5f08b-211">Modelo de mensagem de log</span><span class="sxs-lookup"><span data-stu-id="5f08b-211">Log message template</span></span>

<span data-ttu-id="5f08b-212">Sempre que você grava uma mensagem de log, você fornece um modelo de mensagem.</span><span class="sxs-lookup"><span data-stu-id="5f08b-212">Each time you write a log message, you provide a message template.</span></span> <span data-ttu-id="5f08b-213">O modelo de mensagem pode ser uma cadeia de caracteres ou pode conter espaços reservados nomeados, nos quais valores de argumento serão colocados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-213">The message template can be a string or it can contain named placeholders into which argument values are placed.</span></span> <span data-ttu-id="5f08b-214">O modelo não é uma cadeia de caracteres de formatação e os espaços reservados devem ser nomeados, não numerados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-214">The template isn't a format string, and placeholders should be named, not numbered.</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_CallLogMethods&highlight=3,7)]

<span data-ttu-id="5f08b-215">A ordem dos espaços reservados e não de seus nomes, determina quais parâmetros serão usados para fornecer seus valores.</span><span class="sxs-lookup"><span data-stu-id="5f08b-215">The order of placeholders, not their names, determines which parameters are used to provide their values.</span></span> <span data-ttu-id="5f08b-216">Se você tiver o seguinte código:</span><span class="sxs-lookup"><span data-stu-id="5f08b-216">If you have the following code:</span></span>

```csharp
string p1 = "parm1";
string p2 = "parm2";
_logger.LogInformation("Parameter values: {p2}, {p1}", p1, p2);
```

<span data-ttu-id="5f08b-217">A mensagem de log resultante terá esta aparência:</span><span class="sxs-lookup"><span data-stu-id="5f08b-217">The resulting log message looks like this:</span></span>

```
Parameter values: parm1, parm2
```

<span data-ttu-id="5f08b-218">A estrutura de registros realiza a formatação de mensagens dessa maneira para possibilitar que os provedores de log implementem o [registro em log semântico, também conhecido como registro em log estruturado](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).</span><span class="sxs-lookup"><span data-stu-id="5f08b-218">The logging framework does message formatting in this way to make it possible for logging providers to implement [semantic logging, also known as structured logging](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).</span></span> <span data-ttu-id="5f08b-219">Como os próprios argumentos são passados para o sistema de registro em log, não apenas o modelo de mensagem formatada, mas os provedores de log, também poderão armazenar os valores de parâmetro como campos, além do modelo de mensagem.</span><span class="sxs-lookup"><span data-stu-id="5f08b-219">Because the arguments themselves are passed to the logging system, not just the formatted message template, logging providers can store the parameter values as fields in addition to the message template.</span></span> <span data-ttu-id="5f08b-220">Se você estiver direcionando a saída de log para o Armazenamento de Tabelas do Azure e sua chamada de método do agente tiver esta aparência:</span><span class="sxs-lookup"><span data-stu-id="5f08b-220">If you're directing your log output to Azure Table Storage and your logger method call looks like this:</span></span>

```csharp
_logger.LogInformation("Getting item {ID} at {RequestTime}", id, DateTime.Now);
```

<span data-ttu-id="5f08b-221">Cada entidade da Tabela do Azure poderá ter propriedades `ID` e `RequestTime`, o que simplificará as consultas nos dados de log.</span><span class="sxs-lookup"><span data-stu-id="5f08b-221">Each Azure Table entity can have `ID` and `RequestTime` properties, which simplifies queries on log data.</span></span> <span data-ttu-id="5f08b-222">Você poderá encontrar todos os logs em um determinado intervalo de `RequestTime` sem a necessidade de analisar o tempo limite da mensagem de texto.</span><span class="sxs-lookup"><span data-stu-id="5f08b-222">You can find all logs within a particular `RequestTime` range without the need to parse the time out of the text message.</span></span>

## <a name="logging-exceptions"></a><span data-ttu-id="5f08b-223">Exceções de registro em log</span><span class="sxs-lookup"><span data-stu-id="5f08b-223">Logging exceptions</span></span>

<span data-ttu-id="5f08b-224">Os métodos de agente têm sobrecargas que permitem que você passe uma exceção, como no exemplo a seguir:</span><span class="sxs-lookup"><span data-stu-id="5f08b-224">The logger methods have overloads that let you pass in an exception, as in the following example:</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_LogException&highlight=3)]

<span data-ttu-id="5f08b-225">Provedores diferentes manipulam as informações de exceção de maneiras diferentes.</span><span class="sxs-lookup"><span data-stu-id="5f08b-225">Different providers handle the exception information in different ways.</span></span> <span data-ttu-id="5f08b-226">Aqui está um exemplo da saída do provedor Depuração do código mostrado acima.</span><span class="sxs-lookup"><span data-stu-id="5f08b-226">Here's an example of Debug provider output from the code shown above.</span></span>

```
TodoApi.Controllers.TodoController:Warning: GetById(036dd898-fb01-47e8-9a65-f92eb73cf924) NOT FOUND

System.Exception: Item not found exception.
 at TodoApi.Controllers.TodoController.GetById(String id) in C:\logging\sample\src\TodoApi\Controllers\TodoController.cs:line 226
```

## <a name="log-filtering"></a><span data-ttu-id="5f08b-227">Filtragem de log</span><span class="sxs-lookup"><span data-stu-id="5f08b-227">Log filtering</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-228">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-228">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="5f08b-229">Você pode especificar um nível de log mínimo para um provedor e uma categoria específicos ou para todos os provedores ou todas as categorias.</span><span class="sxs-lookup"><span data-stu-id="5f08b-229">You can specify a minimum log level for a specific provider and category or for all providers or all categories.</span></span> <span data-ttu-id="5f08b-230">Os logs abaixo do nível mínimo não serão passados para esse provedor, para que não sejam exibidos ou armazenados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-230">Any logs below the minimum level aren't passed to that provider, so they don't get displayed or stored.</span></span> 

<span data-ttu-id="5f08b-231">Se quiser suprimir todos os logs, você poderá especificar `LogLevel.None` como o nível de log mínimo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-231">If you want to suppress all logs, you can specify `LogLevel.None` as the minimum log level.</span></span> <span data-ttu-id="5f08b-232">O valor inteiro de `LogLevel.None` é 6, que é maior do que `LogLevel.Critical` (5).</span><span class="sxs-lookup"><span data-stu-id="5f08b-232">The integer value of `LogLevel.None` is 6, which is higher than `LogLevel.Critical` (5).</span></span>

<span data-ttu-id="5f08b-233">**Criar regras de filtro na configuração**</span><span class="sxs-lookup"><span data-stu-id="5f08b-233">**Create filter rules in configuration**</span></span>

<span data-ttu-id="5f08b-234">Os modelos de projeto criam código que chama `CreateDefaultBuilder` para configurar o registro em log para os provedores Console e Depuração.</span><span class="sxs-lookup"><span data-stu-id="5f08b-234">The project templates create code that calls `CreateDefaultBuilder` to set up logging for the Console and Debug providers.</span></span> <span data-ttu-id="5f08b-235">O método `CreateDefaultBuilder` também configura o registro em log para procurar a configuração em uma seção `Logging`, usando código semelhante ao seguinte:</span><span class="sxs-lookup"><span data-stu-id="5f08b-235">The `CreateDefaultBuilder` method also sets up logging to look for configuration in a `Logging` section, using code like the following:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_ExpandDefault&highlight=15)]

<span data-ttu-id="5f08b-236">Os dados de configuração especificam níveis de log mínimo por provedor e por categoria, como no exemplo a seguir:</span><span class="sxs-lookup"><span data-stu-id="5f08b-236">The configuration data specifies minimum log levels by provider and category, as in the following example:</span></span>

[!code-json[](index/sample2/appsettings.json)]

<span data-ttu-id="5f08b-237">Este JSON cria seis regras de filtro, uma para o provedor Depuração, quatro para o provedor Console e uma que se aplica a todos os provedores.</span><span class="sxs-lookup"><span data-stu-id="5f08b-237">This JSON creates six filter rules, one for the Debug provider, four for the Console provider, and one that applies to all providers.</span></span> <span data-ttu-id="5f08b-238">Você verá posteriormente como apenas uma dessas regras é escolhida para cada provedor quando um objeto `ILogger` é criado.</span><span class="sxs-lookup"><span data-stu-id="5f08b-238">You'll see later how just one of these rules is chosen for each provider when an `ILogger` object is created.</span></span>

<span data-ttu-id="5f08b-239">**Regras de filtro no código**</span><span class="sxs-lookup"><span data-stu-id="5f08b-239">**Filter rules in code**</span></span>

<span data-ttu-id="5f08b-240">Você pode registrar regras de filtro no código, conforme mostrado no exemplo a seguir:</span><span class="sxs-lookup"><span data-stu-id="5f08b-240">You can register filter rules in code, as shown in the following example:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_FilterInCode&highlight=4-5)]

<span data-ttu-id="5f08b-241">O segundo `AddFilter` especifica o provedor Depuração usando seu nome de tipo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-241">The second `AddFilter` specifies the Debug provider by using its type name.</span></span> <span data-ttu-id="5f08b-242">O primeiro `AddFilter` se aplica a todos os provedores porque ele não especifica um tipo de provedor.</span><span class="sxs-lookup"><span data-stu-id="5f08b-242">The first `AddFilter` applies to all providers because it doesn't specify a provider type.</span></span>

<span data-ttu-id="5f08b-243">**Como as regras de filtragem são aplicadas**</span><span class="sxs-lookup"><span data-stu-id="5f08b-243">**How filtering rules are applied**</span></span>

<span data-ttu-id="5f08b-244">Os dados de configuração e o código `AddFilter`, mostrados nos exemplos anteriores, criam as regras mostradas na tabela a seguir.</span><span class="sxs-lookup"><span data-stu-id="5f08b-244">The configuration data and the `AddFilter` code shown in the preceding examples create the rules shown in the following table.</span></span> <span data-ttu-id="5f08b-245">As primeiras seis vêm do exemplo de configuração e as últimas duas vêm do exemplo de código.</span><span class="sxs-lookup"><span data-stu-id="5f08b-245">The first six come from the configuration example and the last two come from the code example.</span></span>

| <span data-ttu-id="5f08b-246">Número</span><span class="sxs-lookup"><span data-stu-id="5f08b-246">Number</span></span> | <span data-ttu-id="5f08b-247">Provider</span><span class="sxs-lookup"><span data-stu-id="5f08b-247">Provider</span></span>      | <span data-ttu-id="5f08b-248">Categorias que começam com...</span><span class="sxs-lookup"><span data-stu-id="5f08b-248">Categories that begin with ...</span></span>          | <span data-ttu-id="5f08b-249">Nível de log mínimo</span><span class="sxs-lookup"><span data-stu-id="5f08b-249">Minimum log level</span></span> |
| :----: | ------------- | --------------------------------------- | ----------------- |
| <span data-ttu-id="5f08b-250">1</span><span class="sxs-lookup"><span data-stu-id="5f08b-250">1</span></span>      | <span data-ttu-id="5f08b-251">Depurar</span><span class="sxs-lookup"><span data-stu-id="5f08b-251">Debug</span></span>         | <span data-ttu-id="5f08b-252">Todas as categorias</span><span class="sxs-lookup"><span data-stu-id="5f08b-252">All categories</span></span>                          | <span data-ttu-id="5f08b-253">Informações</span><span class="sxs-lookup"><span data-stu-id="5f08b-253">Information</span></span>       |
| <span data-ttu-id="5f08b-254">2</span><span class="sxs-lookup"><span data-stu-id="5f08b-254">2</span></span>      | <span data-ttu-id="5f08b-255">Console</span><span class="sxs-lookup"><span data-stu-id="5f08b-255">Console</span></span>       | <span data-ttu-id="5f08b-256">Microsoft.AspNetCore.Mvc.Razor.Internal</span><span class="sxs-lookup"><span data-stu-id="5f08b-256">Microsoft.AspNetCore.Mvc.Razor.Internal</span></span> | <span data-ttu-id="5f08b-257">Aviso</span><span class="sxs-lookup"><span data-stu-id="5f08b-257">Warning</span></span>           |
| <span data-ttu-id="5f08b-258">3</span><span class="sxs-lookup"><span data-stu-id="5f08b-258">3</span></span>      | <span data-ttu-id="5f08b-259">Console</span><span class="sxs-lookup"><span data-stu-id="5f08b-259">Console</span></span>       | <span data-ttu-id="5f08b-260">Microsoft.AspNetCore.Mvc.Razor.Razor</span><span class="sxs-lookup"><span data-stu-id="5f08b-260">Microsoft.AspNetCore.Mvc.Razor.Razor</span></span>    | <span data-ttu-id="5f08b-261">Depurar</span><span class="sxs-lookup"><span data-stu-id="5f08b-261">Debug</span></span>             |
| <span data-ttu-id="5f08b-262">4</span><span class="sxs-lookup"><span data-stu-id="5f08b-262">4</span></span>      | <span data-ttu-id="5f08b-263">Console</span><span class="sxs-lookup"><span data-stu-id="5f08b-263">Console</span></span>       | <span data-ttu-id="5f08b-264">Microsoft.AspNetCore.Mvc.Razor</span><span class="sxs-lookup"><span data-stu-id="5f08b-264">Microsoft.AspNetCore.Mvc.Razor</span></span>          | <span data-ttu-id="5f08b-265">Erro</span><span class="sxs-lookup"><span data-stu-id="5f08b-265">Error</span></span>             |
| <span data-ttu-id="5f08b-266">5</span><span class="sxs-lookup"><span data-stu-id="5f08b-266">5</span></span>      | <span data-ttu-id="5f08b-267">Console</span><span class="sxs-lookup"><span data-stu-id="5f08b-267">Console</span></span>       | <span data-ttu-id="5f08b-268">Todas as categorias</span><span class="sxs-lookup"><span data-stu-id="5f08b-268">All categories</span></span>                          | <span data-ttu-id="5f08b-269">Informações</span><span class="sxs-lookup"><span data-stu-id="5f08b-269">Information</span></span>       |
| <span data-ttu-id="5f08b-270">6</span><span class="sxs-lookup"><span data-stu-id="5f08b-270">6</span></span>      | <span data-ttu-id="5f08b-271">Todos os provedores</span><span class="sxs-lookup"><span data-stu-id="5f08b-271">All providers</span></span> | <span data-ttu-id="5f08b-272">Todas as categorias</span><span class="sxs-lookup"><span data-stu-id="5f08b-272">All categories</span></span>                          | <span data-ttu-id="5f08b-273">Depurar</span><span class="sxs-lookup"><span data-stu-id="5f08b-273">Debug</span></span>             |
| <span data-ttu-id="5f08b-274">7</span><span class="sxs-lookup"><span data-stu-id="5f08b-274">7</span></span>      | <span data-ttu-id="5f08b-275">Todos os provedores</span><span class="sxs-lookup"><span data-stu-id="5f08b-275">All providers</span></span> | <span data-ttu-id="5f08b-276">Sistema</span><span class="sxs-lookup"><span data-stu-id="5f08b-276">System</span></span>                                  | <span data-ttu-id="5f08b-277">Depurar</span><span class="sxs-lookup"><span data-stu-id="5f08b-277">Debug</span></span>             |
| <span data-ttu-id="5f08b-278">8</span><span class="sxs-lookup"><span data-stu-id="5f08b-278">8</span></span>      | <span data-ttu-id="5f08b-279">Depurar</span><span class="sxs-lookup"><span data-stu-id="5f08b-279">Debug</span></span>         | <span data-ttu-id="5f08b-280">Microsoft</span><span class="sxs-lookup"><span data-stu-id="5f08b-280">Microsoft</span></span>                               | <span data-ttu-id="5f08b-281">Rastrear</span><span class="sxs-lookup"><span data-stu-id="5f08b-281">Trace</span></span>             |

<span data-ttu-id="5f08b-282">Quando você cria um objeto `ILogger` para usar para gravar logs, o objeto `ILoggerFactory` seleciona uma única regra por provedor para aplicar a esse agente.</span><span class="sxs-lookup"><span data-stu-id="5f08b-282">When you create an `ILogger` object to write logs with, the `ILoggerFactory` object selects a single rule per provider to apply to that logger.</span></span> <span data-ttu-id="5f08b-283">Todas as mensagens gravadas pelo objeto `ILogger` são filtradas com base nas regras selecionadas.</span><span class="sxs-lookup"><span data-stu-id="5f08b-283">All messages written by that `ILogger` object are filtered based on the selected rules.</span></span> <span data-ttu-id="5f08b-284">A regra mais específica possível para cada par de categoria e provedor é selecionada dentre as regras disponíveis.</span><span class="sxs-lookup"><span data-stu-id="5f08b-284">The most specific rule possible for each provider and category pair is selected from the available rules.</span></span>

<span data-ttu-id="5f08b-285">O algoritmo a seguir é usado para cada provedor quando um `ILogger` é criado para uma determinada categoria:</span><span class="sxs-lookup"><span data-stu-id="5f08b-285">The following algorithm is used for each provider when an `ILogger` is created for a given category:</span></span>

* <span data-ttu-id="5f08b-286">Selecione todas as regras que correspondem ao provedor ou seu alias.</span><span class="sxs-lookup"><span data-stu-id="5f08b-286">Select all rules that match the provider or its alias.</span></span> <span data-ttu-id="5f08b-287">Se nenhuma for encontrada, selecione todas as regras com um provedor vazio.</span><span class="sxs-lookup"><span data-stu-id="5f08b-287">If none are found, select all rules with an empty provider.</span></span>
* <span data-ttu-id="5f08b-288">Do resultado da etapa anterior, selecione as regras com o prefixo de categoria de maior correspondência.</span><span class="sxs-lookup"><span data-stu-id="5f08b-288">From the result of the preceding step, select rules with longest matching category prefix.</span></span> <span data-ttu-id="5f08b-289">Se nenhuma for encontrada, selecione todas as regras que não especificam uma categoria.</span><span class="sxs-lookup"><span data-stu-id="5f08b-289">If none are found, select all rules that don't specify a category.</span></span>
* <span data-ttu-id="5f08b-290">Se várias regras forem selecionadas use a **última**.</span><span class="sxs-lookup"><span data-stu-id="5f08b-290">If multiple rules are selected take the **last** one.</span></span>
* <span data-ttu-id="5f08b-291">Se nenhuma regra for selecionada, use `MinimumLevel`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-291">If no rules are selected, use `MinimumLevel`.</span></span>
 
<span data-ttu-id="5f08b-292">Por exemplo, suponha que você tem a lista anterior de regras e cria um objeto `ILogger` para a categoria "Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine":</span><span class="sxs-lookup"><span data-stu-id="5f08b-292">For example, suppose you have the preceding list of rules and you create an `ILogger` object for category "Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine":</span></span>

* <span data-ttu-id="5f08b-293">Para o provedor Depuração as regras 1, 6 e 8 se aplicam.</span><span class="sxs-lookup"><span data-stu-id="5f08b-293">For the Debug provider, rules 1, 6, and 8 apply.</span></span> <span data-ttu-id="5f08b-294">A regra 8 é mais específica, portanto é a que será selecionada.</span><span class="sxs-lookup"><span data-stu-id="5f08b-294">Rule 8 is most specific, so that's the one selected.</span></span>
* <span data-ttu-id="5f08b-295">Para o provedor Console as regras 3, 4, 5 e 6 se aplicam.</span><span class="sxs-lookup"><span data-stu-id="5f08b-295">For the Console provider, rules 3, 4, 5, and 6 apply.</span></span> <span data-ttu-id="5f08b-296">A regra 3 é a mais específica.</span><span class="sxs-lookup"><span data-stu-id="5f08b-296">Rule 3 is most specific.</span></span>

<span data-ttu-id="5f08b-297">Quando você cria logs com um `ILogger` para a categoria "Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine", os logs de nível `Trace` e superiores vão para o provedor Depuração e os logs de nível `Debug` e superiores vão para o provedor Console.</span><span class="sxs-lookup"><span data-stu-id="5f08b-297">When you create logs with an `ILogger` for category "Microsoft.AspNetCore.Mvc.Razor.RazorViewEngine", logs of `Trace` level and above will go to the Debug provider, and logs of `Debug` level and above will go to the Console provider.</span></span>

<span data-ttu-id="5f08b-298">**Aliases de provedor**</span><span class="sxs-lookup"><span data-stu-id="5f08b-298">**Provider aliases**</span></span>

<span data-ttu-id="5f08b-299">Você pode usar o nome do tipo para especificar um provedor na configuração, mas cada provedor define um *alias* menor que é mais fácil de usar.</span><span class="sxs-lookup"><span data-stu-id="5f08b-299">You can use the type name to specify a provider in configuration, but each provider defines a shorter *alias* that is easier to use.</span></span> <span data-ttu-id="5f08b-300">Para os provedores internos, use os seguintes aliases:</span><span class="sxs-lookup"><span data-stu-id="5f08b-300">For the built-in providers, use the following aliases:</span></span>

- <span data-ttu-id="5f08b-301">Console</span><span class="sxs-lookup"><span data-stu-id="5f08b-301">Console</span></span>
- <span data-ttu-id="5f08b-302">Depurar</span><span class="sxs-lookup"><span data-stu-id="5f08b-302">Debug</span></span>
- <span data-ttu-id="5f08b-303">EventLog</span><span class="sxs-lookup"><span data-stu-id="5f08b-303">EventLog</span></span>
- <span data-ttu-id="5f08b-304">AzureAppServices</span><span class="sxs-lookup"><span data-stu-id="5f08b-304">AzureAppServices</span></span>
- <span data-ttu-id="5f08b-305">TraceSource</span><span class="sxs-lookup"><span data-stu-id="5f08b-305">TraceSource</span></span>
- <span data-ttu-id="5f08b-306">EventSource</span><span class="sxs-lookup"><span data-stu-id="5f08b-306">EventSource</span></span>

<span data-ttu-id="5f08b-307">**Nível mínimo padrão**</span><span class="sxs-lookup"><span data-stu-id="5f08b-307">**Default minimum level**</span></span>

<span data-ttu-id="5f08b-308">Há uma configuração de nível mínimo que entra em vigor somente se nenhuma regra de código ou de configuração se aplicar a um provedor e uma categoria determinados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-308">There is a minimum level setting that takes effect only if no rules from configuration or code apply for a given provider and category.</span></span> <span data-ttu-id="5f08b-309">O exemplo a seguir mostra como definir o nível mínimo:</span><span class="sxs-lookup"><span data-stu-id="5f08b-309">The following example shows how to set the minimum level:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_MinLevel&highlight=3)]

<span data-ttu-id="5f08b-310">Se você não definir explicitamente o nível mínimo, o valor padrão será `Information`, o que significa que logs `Trace` e `Debug` serão ignorados.</span><span class="sxs-lookup"><span data-stu-id="5f08b-310">If you don't explicitly set the minimum level, the default value is `Information`, which means that `Trace` and `Debug` logs are ignored.</span></span>

<span data-ttu-id="5f08b-311">**Funções de filtro**</span><span class="sxs-lookup"><span data-stu-id="5f08b-311">**Filter functions**</span></span>

<span data-ttu-id="5f08b-312">Você pode escrever código em uma função de filtro para aplicar regras de filtragem.</span><span class="sxs-lookup"><span data-stu-id="5f08b-312">You can write code in a filter function to apply filtering rules.</span></span> <span data-ttu-id="5f08b-313">Uma função de filtro é invocada para todos os provedores e categorias que não têm regras atribuídas a eles por configuração ou código.</span><span class="sxs-lookup"><span data-stu-id="5f08b-313">A filter function is invoked for all providers and categories that do not have rules assigned to them by configuration or code.</span></span> <span data-ttu-id="5f08b-314">O código na função tem acesso ao tipo de provedor, à categoria e ao nível de log para decidir se uma mensagem deve ser registrada.</span><span class="sxs-lookup"><span data-stu-id="5f08b-314">Code in the function has access to the provider type, category, and log level to decide whether or not a message should be logged.</span></span> <span data-ttu-id="5f08b-315">Por exemplo:</span><span class="sxs-lookup"><span data-stu-id="5f08b-315">For example:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_FilterFunction&highlight=5-13)]

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-316">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-316">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="5f08b-317">Alguns provedores de log permitem especificar quando os logs devem ser gravados em uma mídia de armazenamento ou ignorados, com base no nível de log e na categoria.</span><span class="sxs-lookup"><span data-stu-id="5f08b-317">Some logging providers let you specify when logs should be written to a storage medium or ignored based on log level and category.</span></span>

<span data-ttu-id="5f08b-318">Os métodos de extensão `AddConsole` e `AddDebug` fornecem sobrecargas que permitem que você passe critérios de filtragem.</span><span class="sxs-lookup"><span data-stu-id="5f08b-318">The `AddConsole` and `AddDebug` extension methods provide overloads that let you pass in filtering criteria.</span></span> <span data-ttu-id="5f08b-319">O código de exemplo a seguir faz com que o provedor de console ignore os logs abaixo do nível `Warning`, enquanto o provedor Depuração ignora os logs criados pela estrutura.</span><span class="sxs-lookup"><span data-stu-id="5f08b-319">The following sample code causes the console provider to ignore logs below `Warning` level, while the Debug provider ignores logs that the framework creates.</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_AddConsoleAndDebugWithFilter&highlight=6-7)]

<span data-ttu-id="5f08b-320">O método `AddEventLog` tem uma sobrecarga que recebe uma instância `EventLogSettings`, que pode conter uma função de filtragem em sua propriedade `Filter`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-320">The `AddEventLog` method has an overload that takes an `EventLogSettings` instance, which may contain a filtering function in its `Filter` property.</span></span> <span data-ttu-id="5f08b-321">O provedor TraceSource não fornece nenhuma dessas sobrecargas, porque seu nível de registro em log e outros parâmetros são baseados no `SourceSwitch` e no `TraceListener` que ele usa.</span><span class="sxs-lookup"><span data-stu-id="5f08b-321">The TraceSource provider does not provide any of those overloads, since its logging level and other parameters are based on the `SourceSwitch` and `TraceListener` it uses.</span></span>

<span data-ttu-id="5f08b-322">Você pode definir regras de filtragem para todos os provedores registrados com uma instância `ILoggerFactory` usando o método de extensão `WithFilter`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-322">You can set filtering rules for all providers that are registered with an `ILoggerFactory` instance by using the `WithFilter` extension method.</span></span> <span data-ttu-id="5f08b-323">O exemplo abaixo limita os logs de estrutura (categoria começa com "Microsoft" ou "Sistema") a avisos, permitindo que aplicativo registre no nível de depuração.</span><span class="sxs-lookup"><span data-stu-id="5f08b-323">The example below limits framework logs (category begins with "Microsoft" or "System") to warnings while letting the app log at debug level.</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_FactoryFilter&highlight=6-11)]

<span data-ttu-id="5f08b-324">Se quiser usar a filtragem para impedir que todos os logs de uma determinada categoria sejam gravados, você poderá especificar `LogLevel.None` como o nível de log mínimo para essa categoria.</span><span class="sxs-lookup"><span data-stu-id="5f08b-324">If you want to use filtering to prevent all logs from being written for a particular category, you can specify `LogLevel.None` as the minimum log level for that category.</span></span> <span data-ttu-id="5f08b-325">O valor inteiro de `LogLevel.None` é 6, que é maior do que `LogLevel.Critical` (5).</span><span class="sxs-lookup"><span data-stu-id="5f08b-325">The integer value of `LogLevel.None` is 6, which is higher than `LogLevel.Critical` (5).</span></span>

<span data-ttu-id="5f08b-326">O método de extensão `WithFilter` é fornecido pelo pacote NuGet [Microsoft.Extensions.Logging.Filter](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Filter).</span><span class="sxs-lookup"><span data-stu-id="5f08b-326">The `WithFilter` extension method is provided by the [Microsoft.Extensions.Logging.Filter](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Filter) NuGet package.</span></span> <span data-ttu-id="5f08b-327">O método retorna um nova instância `ILoggerFactory`, que filtrará as mensagens de log passadas para todos os provedores de agente registrados com ela.</span><span class="sxs-lookup"><span data-stu-id="5f08b-327">The method returns a new `ILoggerFactory` instance that will filter the log messages passed to all logger providers registered with it.</span></span> <span data-ttu-id="5f08b-328">Isso não afeta nenhuma outra instância de `ILoggerFactory`, incluindo a instância de `ILoggerFactory` original.</span><span class="sxs-lookup"><span data-stu-id="5f08b-328">It does not affect any other `ILoggerFactory` instances, including the original `ILoggerFactory` instance.</span></span>

---

## <a name="log-scopes"></a><span data-ttu-id="5f08b-329">Escopos de log</span><span class="sxs-lookup"><span data-stu-id="5f08b-329">Log scopes</span></span>

<span data-ttu-id="5f08b-330">Você pode agrupar um conjunto de operações lógicas dentro de um *escopo* para anexar os mesmos dados a cada log que é criado como parte desse conjunto.</span><span class="sxs-lookup"><span data-stu-id="5f08b-330">You can group a set of logical operations within a *scope* in order to attach the same data to each log that is created as part of that set.</span></span> <span data-ttu-id="5f08b-331">Por exemplo, é conveniente que cada log criado como parte do processamento de uma transação inclua a ID da transação.</span><span class="sxs-lookup"><span data-stu-id="5f08b-331">For example, you might want every log created as part of processing a transaction to include the transaction ID.</span></span>

<span data-ttu-id="5f08b-332">Um escopo é um tipo `IDisposable` retornado pelo método `ILogger.BeginScope<TState>` e que dura até que seja descartado.</span><span class="sxs-lookup"><span data-stu-id="5f08b-332">A scope is an `IDisposable` type that is returned by the `ILogger.BeginScope<TState>` method and lasts until it is disposed.</span></span> <span data-ttu-id="5f08b-333">Um escopo é usado pelo encapsulamento das chama de agente em um bloco `using`, conforme mostrado aqui:</span><span class="sxs-lookup"><span data-stu-id="5f08b-333">You use a scope by wrapping your logger calls in a `using` block, as shown here:</span></span>

[!code-csharp[](index/sample//Controllers/TodoController.cs?name=snippet_Scopes&highlight=4-5,13)]

<span data-ttu-id="5f08b-334">O código a seguir habilita os escopos para o provedor de console:</span><span class="sxs-lookup"><span data-stu-id="5f08b-334">The following code enables scopes for the console provider:</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-335">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-335">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="5f08b-336">Em *Program.cs*:</span><span class="sxs-lookup"><span data-stu-id="5f08b-336">In *Program.cs*:</span></span>

[!code-csharp[](index/sample2/Program.cs?name=snippet_Scopes&highlight=4)]

> [!NOTE]
> <span data-ttu-id="5f08b-337">A configuração da opção de agente de console `IncludeScopes` é necessária para habilitar o registro em log baseado em escopo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-337">Configuring the `IncludeScopes` console logger option is required to enable scope-based logging.</span></span> <span data-ttu-id="5f08b-338">A configuração de `IncludeScopes` usando arquivos de configuração *appsettings* estará disponível com o lançamento do ASP.NET Core 2.1.</span><span class="sxs-lookup"><span data-stu-id="5f08b-338">Configuration of `IncludeScopes` using *appsettings* configuration files will be available with the release of ASP.NET Core 2.1.</span></span>

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-339">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-339">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

<span data-ttu-id="5f08b-340">Em *Startup.cs*:</span><span class="sxs-lookup"><span data-stu-id="5f08b-340">In *Startup.cs*:</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_Scopes&highlight=6)]

---

<span data-ttu-id="5f08b-341">Cada mensagem de log inclui as informações com escopo definido:</span><span class="sxs-lookup"><span data-stu-id="5f08b-341">Each log message includes the scoped information:</span></span>

```
info: TodoApi.Controllers.TodoController[1002]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApi.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      Getting item 0
warn: TodoApi.Controllers.TodoController[4000]
      => RequestId:0HKV9C49II9CK RequestPath:/api/todo/0 => TodoApi.Controllers.TodoController.GetById (TodoApi) => Message attached to logs created in the using block
      GetById(0) NOT FOUND
```

## <a name="built-in-logging-providers"></a><span data-ttu-id="5f08b-342">Provedores de log internos</span><span class="sxs-lookup"><span data-stu-id="5f08b-342">Built-in logging providers</span></span>

<span data-ttu-id="5f08b-343">O ASP.NET Core vem com os seguintes provedores:</span><span class="sxs-lookup"><span data-stu-id="5f08b-343">ASP.NET Core ships the following providers:</span></span>

* [<span data-ttu-id="5f08b-344">Console</span><span class="sxs-lookup"><span data-stu-id="5f08b-344">Console</span></span>](#console)
* [<span data-ttu-id="5f08b-345">Depurar</span><span class="sxs-lookup"><span data-stu-id="5f08b-345">Debug</span></span>](#debug)
* [<span data-ttu-id="5f08b-346">EventSource</span><span class="sxs-lookup"><span data-stu-id="5f08b-346">EventSource</span></span>](#eventsource)
* [<span data-ttu-id="5f08b-347">EventLog</span><span class="sxs-lookup"><span data-stu-id="5f08b-347">EventLog</span></span>](#eventlog)
* [<span data-ttu-id="5f08b-348">TraceSource</span><span class="sxs-lookup"><span data-stu-id="5f08b-348">TraceSource</span></span>](#tracesource)
* [<span data-ttu-id="5f08b-349">Serviço de Aplicativo do Azure</span><span class="sxs-lookup"><span data-stu-id="5f08b-349">Azure App Service</span></span>](#appservice)

<a id="console"></a>
### <a name="the-console-provider"></a><span data-ttu-id="5f08b-350">O provedor de console</span><span class="sxs-lookup"><span data-stu-id="5f08b-350">The console provider</span></span>

<span data-ttu-id="5f08b-351">O pacote de provedor [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console) envia a saída de log para o console.</span><span class="sxs-lookup"><span data-stu-id="5f08b-351">The [Microsoft.Extensions.Logging.Console](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console) provider package sends log output to the console.</span></span> 

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-352">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-352">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddConsole()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-353">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-353">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddConsole()
```

<span data-ttu-id="5f08b-354">As [sobrecargas do AddConsole](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.consoleloggerextensions) permitem que você passe um nível de log mínimo, uma função de filtro e um valor booliano que indica se escopos são compatíveis.</span><span class="sxs-lookup"><span data-stu-id="5f08b-354">[AddConsole overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.consoleloggerextensions) let you pass in an a minimum log level, a filter function, and a boolean that indicates whether scopes are supported.</span></span> <span data-ttu-id="5f08b-355">Outra opção é passar um objeto `IConfiguration`, que pode especificar suporte de escopos e níveis de log.</span><span class="sxs-lookup"><span data-stu-id="5f08b-355">Another option is to pass in an `IConfiguration` object, which can specify scopes support and logging levels.</span></span> 

<span data-ttu-id="5f08b-356">Se você estiver considerando o provedor de console para uso em produção, lembre-se de que ele tem um impacto significativo no desempenho.</span><span class="sxs-lookup"><span data-stu-id="5f08b-356">If you are considering the console provider for use in production, be aware that it has a significant impact on performance.</span></span>

<span data-ttu-id="5f08b-357">Quando você cria um novo projeto no Visual Studio, o método `AddConsole` tem essa aparência:</span><span class="sxs-lookup"><span data-stu-id="5f08b-357">When you create a new project in Visual Studio, the `AddConsole` method looks like this:</span></span>

```csharp
loggerFactory.AddConsole(Configuration.GetSection("Logging"));
```

<span data-ttu-id="5f08b-358">Esse código se refere à seção `Logging` do arquivo *appSettings.json*:</span><span class="sxs-lookup"><span data-stu-id="5f08b-358">This code refers to the `Logging` section of the *appSettings.json* file:</span></span>

[!code-json[](index/sample//appsettings.json)]

<span data-ttu-id="5f08b-359">As configurações mostradas limitam os logs de estrutura a avisos, permitindo que o aplicativo para faça registros no nível de depuração, conforme explicado na [Filtragem de log](#log-filtering) seção.</span><span class="sxs-lookup"><span data-stu-id="5f08b-359">The settings shown limit framework logs to warnings while allowing the app to log at debug level, as explained in the [Log filtering](#log-filtering) section.</span></span> <span data-ttu-id="5f08b-360">Para obter mais informações, consulte [Configuração](xref:fundamentals/configuration/index).</span><span class="sxs-lookup"><span data-stu-id="5f08b-360">For more information, see [Configuration](xref:fundamentals/configuration/index).</span></span>

---

<a id="debug"></a>
### <a name="the-debug-provider"></a><span data-ttu-id="5f08b-361">O provedor Depuração</span><span class="sxs-lookup"><span data-stu-id="5f08b-361">The Debug provider</span></span>

<span data-ttu-id="5f08b-362">O pacote de provedor [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug) grava a saída de log usando a classe [System.Diagnostics.Debug](https://docs.microsoft.com/dotnet/core/api/system.diagnostics.debug#System_Diagnostics_Debug) (chamadas de método `Debug.WriteLine`).</span><span class="sxs-lookup"><span data-stu-id="5f08b-362">The [Microsoft.Extensions.Logging.Debug](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug) provider package writes log output by using the [System.Diagnostics.Debug](https://docs.microsoft.com/dotnet/core/api/system.diagnostics.debug#System_Diagnostics_Debug) class (`Debug.WriteLine` method calls).</span></span>

<span data-ttu-id="5f08b-363">No Linux, esse provedor grava logs em */var/log/message*.</span><span class="sxs-lookup"><span data-stu-id="5f08b-363">On Linux, this provider writes logs to */var/log/message*.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-364">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-364">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddDebug()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-365">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-365">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddDebug()
```

<span data-ttu-id="5f08b-366">As [sobrecargas de AddDebug](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.debugloggerfactoryextensions) permitem que você passe um nível de log mínimo ou uma função de filtro.</span><span class="sxs-lookup"><span data-stu-id="5f08b-366">[AddDebug overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.debugloggerfactoryextensions) let you pass in a minimum log level or a filter function.</span></span>

---

<a id="eventsource"></a>
### <a name="the-eventsource-provider"></a><span data-ttu-id="5f08b-367">O provedor EventSource</span><span class="sxs-lookup"><span data-stu-id="5f08b-367">The EventSource provider</span></span>

<span data-ttu-id="5f08b-368">Para aplicativos que se destinam ao ASP.NET Core 1.1.0 ou superior, o pacote de provedor [Microsoft.Extensions.Logging.EventSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventSource) pode implementar o rastreamento de eventos.</span><span class="sxs-lookup"><span data-stu-id="5f08b-368">For apps that target ASP.NET Core 1.1.0 or higher, the [Microsoft.Extensions.Logging.EventSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventSource) provider package can implement event tracing.</span></span> <span data-ttu-id="5f08b-369">No Windows, ele usa [ETW](https://msdn.microsoft.com/library/windows/desktop/bb968803).</span><span class="sxs-lookup"><span data-stu-id="5f08b-369">On Windows, it uses [ETW](https://msdn.microsoft.com/library/windows/desktop/bb968803).</span></span> <span data-ttu-id="5f08b-370">O provedor é multiplataforma, mas ainda não há ferramentas de coleta e exibição de eventos para Linux ou macOS.</span><span class="sxs-lookup"><span data-stu-id="5f08b-370">The provider is cross-platform, but there are no event collection and display tools yet for Linux or macOS.</span></span> 

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-371">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-371">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddEventSourceLogger()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-372">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-372">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddEventSourceLogger()
```

---

<span data-ttu-id="5f08b-373">Uma boa maneira de coletar e exibir logs é usar o [utilitário PerfView](https://www.microsoft.com/download/details.aspx?id=28567).</span><span class="sxs-lookup"><span data-stu-id="5f08b-373">A good way to collect and view logs is to use the [PerfView utility](https://www.microsoft.com/download/details.aspx?id=28567).</span></span> <span data-ttu-id="5f08b-374">Há outras ferramentas para exibir os logs do ETW, mas o PerfView proporciona a melhor experiência para trabalhar com os eventos de ETW emitidos pelo ASP.NET.</span><span class="sxs-lookup"><span data-stu-id="5f08b-374">There are other tools for viewing ETW logs, but PerfView provides the best experience for working with the ETW events emitted by ASP.NET.</span></span> 

<span data-ttu-id="5f08b-375">Para configurar o PerfView para coletar eventos registrados por esse provedor, adicione a cadeia de caracteres `*Microsoft-Extensions-Logging` à lista **Provedores Adicionais**.</span><span class="sxs-lookup"><span data-stu-id="5f08b-375">To configure PerfView for collecting events logged by this provider, add the string `*Microsoft-Extensions-Logging` to the **Additional Providers** list.</span></span> <span data-ttu-id="5f08b-376">(Não se esqueça do asterisco no início da cadeia de caracteres).</span><span class="sxs-lookup"><span data-stu-id="5f08b-376">(Don't miss the asterisk at the start of the string.)</span></span>

![Outros provedores de Perfview](index/_static/perfview-additional-providers.png)

<span data-ttu-id="5f08b-378">A captura de eventos no Nano Server demanda algumas configurações adicionais:</span><span class="sxs-lookup"><span data-stu-id="5f08b-378">Capturing events on Nano Server requires some additional setup:</span></span>

* <span data-ttu-id="5f08b-379">Conecte a comunicação remota do PowerShell ao Nano Server:</span><span class="sxs-lookup"><span data-stu-id="5f08b-379">Connect PowerShell remoting to the Nano Server:</span></span>

  ```powershell
  Enter-PSSession [name]
  ```

* <span data-ttu-id="5f08b-380">Crie uma sessão de ETW:</span><span class="sxs-lookup"><span data-stu-id="5f08b-380">Create an ETW session:</span></span>

  ```powershell
  New-EtwTraceSession -Name "MyAppTrace" -LocalFilePath C:\trace.etl
  ```

* <span data-ttu-id="5f08b-381">Adicione provedores de ETW ao [CLR](https://docs.microsoft.com/dotnet/framework/performance/clr-etw-providers), ao ASP.NET Core e a outros, conforme necessário.</span><span class="sxs-lookup"><span data-stu-id="5f08b-381">Add ETW providers for [CLR](https://docs.microsoft.com/dotnet/framework/performance/clr-etw-providers), ASP.NET Core, and others as needed.</span></span> <span data-ttu-id="5f08b-382">O GUID de provedor do ASP.NET Core é `3ac73b97-af73-50e9-0822-5da4367920d0`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-382">The ASP.NET Core provider GUID is `3ac73b97-af73-50e9-0822-5da4367920d0`.</span></span> 

  ```powershell
  Add-EtwTraceProvider -Guid "{e13c0d23-ccbc-4e12-931b-d9cc2eee27e4}" -SessionName MyAppTrace
  Add-EtwTraceProvider -Guid "{3ac73b97-af73-50e9-0822-5da4367920d0}" -SessionName MyAppTrace
  ```

* <span data-ttu-id="5f08b-383">Execute o site e realize as ações para as quais você deseja obter informações de rastreamento.</span><span class="sxs-lookup"><span data-stu-id="5f08b-383">Run the site and do whatever actions you want tracing information for.</span></span>

* <span data-ttu-id="5f08b-384">Interrompa a sessão de rastreamento quando tiver terminado:</span><span class="sxs-lookup"><span data-stu-id="5f08b-384">Stop the tracing session when you're finished:</span></span>

  ```powershell
  Stop-EtwTraceSession -Name "MyAppTrace"
  ```

<span data-ttu-id="5f08b-385">O arquivo resultante *C:\trace.etl* pode ser analisado com PerfView, como em outras edições do Windows.</span><span class="sxs-lookup"><span data-stu-id="5f08b-385">The resulting *C:\trace.etl* file can be analyzed with PerfView as on other editions of Windows.</span></span>

<a id="eventlog"></a>
### <a name="the-windows-eventlog-provider"></a><span data-ttu-id="5f08b-386">O provedor EventLog do Windows</span><span class="sxs-lookup"><span data-stu-id="5f08b-386">The Windows EventLog provider</span></span>

<span data-ttu-id="5f08b-387">O pacote de provedor [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) envia a saída de log para o Log de Eventos do Windows.</span><span class="sxs-lookup"><span data-stu-id="5f08b-387">The [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) provider package sends log output to the Windows Event Log.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-388">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-388">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddEventLog()
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-389">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-389">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddEventLog()
```

<span data-ttu-id="5f08b-390">As [sobrecargas de AddEventLog](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.eventloggerfactoryextensions) permitem que você passe `EventLogSettings` ou um nível de log mínimo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-390">[AddEventLog overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.eventloggerfactoryextensions) let you pass in `EventLogSettings` or a minimum log level.</span></span>

---

<a id="tracesource"></a>
### <a name="the-tracesource-provider"></a><span data-ttu-id="5f08b-391">O provedor TraceSource</span><span class="sxs-lookup"><span data-stu-id="5f08b-391">The TraceSource provider</span></span>

<span data-ttu-id="5f08b-392">O pacote de provedor [Microsoft.Extensions.Logging.TraceSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.TraceSource) usa as bibliotecas e provedores de [System.Diagnostics.TraceSource](https://docs.microsoft.com/dotnet/api/system.diagnostics.tracesource).</span><span class="sxs-lookup"><span data-stu-id="5f08b-392">The [Microsoft.Extensions.Logging.TraceSource](https://www.nuget.org/packages/Microsoft.Extensions.Logging.TraceSource) provider package uses the [System.Diagnostics.TraceSource](https://docs.microsoft.com/dotnet/api/system.diagnostics.tracesource) libraries and providers.</span></span>

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-393">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-393">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

```csharp
logging.AddTraceSource(sourceSwitchName);
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-394">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-394">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddTraceSource(sourceSwitchName);
```

---

<span data-ttu-id="5f08b-395">As [sobrecargas de AddTraceSource](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.tracesourcefactoryextensions) permitem que você passe um comutador de fonte e um ouvinte de rastreamento.</span><span class="sxs-lookup"><span data-stu-id="5f08b-395">[AddTraceSource overloads](https://docs.microsoft.com/aspnet/core/api/microsoft.extensions.logging.tracesourcefactoryextensions) let you pass in a source switch and a trace listener.</span></span>

<span data-ttu-id="5f08b-396">Para usar esse provedor, o aplicativo deve ser executado no .NET Framework (em vez do .NET Core).</span><span class="sxs-lookup"><span data-stu-id="5f08b-396">To use this provider, an application has to run on the .NET Framework (rather than .NET Core).</span></span> <span data-ttu-id="5f08b-397">O provedor permite rotear mensagens a uma variedade de [ouvintes](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/trace-listeners), como o [TextWriterTraceListener](https://docs.microsoft.com/dotnet/api/system.diagnostics.textwritertracelistenerr) usado no aplicativo de exemplo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-397">The provider lets you route messages to a variety of [listeners](https://docs.microsoft.com/dotnet/framework/debug-trace-profile/trace-listeners), such as the [TextWriterTraceListener](https://docs.microsoft.com/dotnet/api/system.diagnostics.textwritertracelistenerr) used in the sample application.</span></span>

<span data-ttu-id="5f08b-398">O exemplo a seguir configura um provedor `TraceSource` que registra mensagens `Warning` e superiores na janela de console.</span><span class="sxs-lookup"><span data-stu-id="5f08b-398">The following example configures a `TraceSource` provider that logs `Warning` and higher messages to the console window.</span></span>

[!code-csharp[](index/sample/Startup.cs?name=snippet_TraceSource&highlight=9-12)]

<a id="appservice"></a>
### <a name="the-azure-app-service-provider"></a><span data-ttu-id="5f08b-399">O provedor do Serviço de Aplicativo do Azure</span><span class="sxs-lookup"><span data-stu-id="5f08b-399">The Azure App Service provider</span></span>

<span data-ttu-id="5f08b-400">O pacote de provedor [Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices) grava logs em arquivos de texto no sistema de arquivos de um aplicativo do Serviço de Aplicativo do Azure e no [armazenamento de blobs](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/#what-is-blob-storage) em uma conta de Armazenamento do Azure.</span><span class="sxs-lookup"><span data-stu-id="5f08b-400">The [Microsoft.Extensions.Logging.AzureAppServices](https://www.nuget.org/packages/Microsoft.Extensions.Logging.AzureAppServices) provider package writes logs to text files in an Azure App Service app's file system and to [blob storage](https://azure.microsoft.com/documentation/articles/storage-dotnet-how-to-use-blobs/#what-is-blob-storage) in an Azure Storage account.</span></span> <span data-ttu-id="5f08b-401">O provedor está disponível somente para aplicativos que se destinam ao ASP.NET Core 1.1.0 ou superior.</span><span class="sxs-lookup"><span data-stu-id="5f08b-401">The provider is available only for apps that target ASP.NET Core 1.1.0 or higher.</span></span> 

# <a name="aspnet-core-2xtabaspnetcore2x"></a>[<span data-ttu-id="5f08b-402">ASP.NET Core 2.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-402">ASP.NET Core 2.x</span></span>](#tab/aspnetcore2x)

<span data-ttu-id="5f08b-403">Se o destino for o .NET Core, você não precisará instalar o pacote de provedor ou chamar explicitamente `AddAzureWebAppDiagnostics`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-403">If targeting .NET Core, you don't have to install the provider package or explicitly call `AddAzureWebAppDiagnostics`.</span></span> <span data-ttu-id="5f08b-404">O provedor fica automaticamente disponível para o aplicativo quando você implanta o aplicativo do Serviço de Aplicativo do Azure.</span><span class="sxs-lookup"><span data-stu-id="5f08b-404">The provider is automatically available to your app when you deploy the app to Azure App Service.</span></span>

<span data-ttu-id="5f08b-405">Se o destino for o .NET Framework, adicione o pacote de provedor ao seu projeto e invoque `AddAzureWebAppDiagnostics`:</span><span class="sxs-lookup"><span data-stu-id="5f08b-405">If targeting .NET Framework, add the provider package to your project and invoke `AddAzureWebAppDiagnostics`:</span></span>

```csharp
logging.AddAzureWebAppDiagnostics();
```

# <a name="aspnet-core-1xtabaspnetcore1x"></a>[<span data-ttu-id="5f08b-406">ASP.NET Core 1.x</span><span class="sxs-lookup"><span data-stu-id="5f08b-406">ASP.NET Core 1.x</span></span>](#tab/aspnetcore1x)

```csharp
loggerFactory.AddAzureWebAppDiagnostics();
```

<span data-ttu-id="5f08b-407">Uma sobrecarga `AddAzureWebAppDiagnostics` permite que você passe [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs) com a qual você pode substituir as configurações padrão, como o modelo de saída de registro em log, o nome do blob e o limite do tamanho do arquivo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-407">An `AddAzureWebAppDiagnostics` overload lets you pass in [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs) with which you can override default settings such as the logging output template, blob name, and file size limit.</span></span> <span data-ttu-id="5f08b-408">(O *modelo Output* é um modelo de mensagem que é aplicado a todos os logs, sobre aquele que você fornece ao chamar um método `ILogger`).</span><span class="sxs-lookup"><span data-stu-id="5f08b-408">(*Output template* is a message template that's applied to all logs on top of the one that you provide when you call an `ILogger` method.)</span></span>

---

<span data-ttu-id="5f08b-409">Ao implantar um aplicativo do Serviço de Aplicativo, seu aplicativo respeita as configurações na seção [Logs de Diagnóstico](https://azure.microsoft.com/documentation/articles/web-sites-enable-diagnostic-log/#enablediag) da página **Serviço de Aplicativo** do Portal do Azure.</span><span class="sxs-lookup"><span data-stu-id="5f08b-409">When you deploy to an App Service app, your application honors the settings in the [Diagnostic Logs](https://azure.microsoft.com/documentation/articles/web-sites-enable-diagnostic-log/#enablediag) section of the **App Service** page of the Azure portal.</span></span> <span data-ttu-id="5f08b-410">Ao alterar essas configurações, elas entram em vigor imediatamente, sem exigir que você reinicie o aplicativo ou reimplante o código nele.</span><span class="sxs-lookup"><span data-stu-id="5f08b-410">When you change those settings, the changes take effect immediately without requiring that you restart the app or redeploy code to it.</span></span> 

![Configurações de registro em log do Azure](index/_static/azure-logging-settings.png)

<span data-ttu-id="5f08b-412">O local padrão para arquivos de log é na pasta *D:\\home\\LogFiles\\Application* e o nome de arquivo padrão é *diagnostics-aaaammdd.txt*.</span><span class="sxs-lookup"><span data-stu-id="5f08b-412">The default location for log files is in the *D:\\home\\LogFiles\\Application* folder, and the default file name is *diagnostics-yyyymmdd.txt*.</span></span> <span data-ttu-id="5f08b-413">O limite padrão de tamanho do arquivo é 10 MB e o número padrão máximo de arquivos mantidos é 2.</span><span class="sxs-lookup"><span data-stu-id="5f08b-413">The default file size limit is 10 MB, and the default maximum number of files retained is 2.</span></span> <span data-ttu-id="5f08b-414">O nome de blob padrão é *{app-name}{timestamp}/aaaa/mm/dd/hh/{guid}-applicationLog.txt*.</span><span class="sxs-lookup"><span data-stu-id="5f08b-414">The default blob name is *{app-name}{timestamp}/yyyy/mm/dd/hh/{guid}-applicationLog.txt*.</span></span> <span data-ttu-id="5f08b-415">Para obter mais informações sobre o comportamento padrão, consulte [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs).</span><span class="sxs-lookup"><span data-stu-id="5f08b-415">For more information about default behavior, see [AzureAppServicesDiagnosticsSettings](https://github.com/aspnet/Logging/blob/c7d0b1b88668ff4ef8a86ea7d2ebb5ca7f88d3e0/src/Microsoft.Extensions.Logging.AzureAppServices/AzureAppServicesDiagnosticsSettings.cs).</span></span>

<span data-ttu-id="5f08b-416">O provedor funciona somente quando o projeto é executado no ambiente do Azure.</span><span class="sxs-lookup"><span data-stu-id="5f08b-416">The provider only works when your project runs in the Azure environment.</span></span> <span data-ttu-id="5f08b-417">Ele não tem nenhum efeito quando é executado localmente &mdash; ele não grava em arquivos locais ou no armazenamento de desenvolvimento local para blobs.</span><span class="sxs-lookup"><span data-stu-id="5f08b-417">It has no effect when you run locally &mdash; it does not write to local files or local development storage for blobs.</span></span>

## <a name="third-party-logging-providers"></a><span data-ttu-id="5f08b-418">Provedores de log de terceiros</span><span class="sxs-lookup"><span data-stu-id="5f08b-418">Third-party logging providers</span></span>

<span data-ttu-id="5f08b-419">Aqui estão algumas estruturas de registros de terceiros que funcionam com o ASP.NET Core:</span><span class="sxs-lookup"><span data-stu-id="5f08b-419">Here are some third-party logging frameworks that work with ASP.NET Core:</span></span>

* <span data-ttu-id="5f08b-420">[elmah.io](https://github.com/elmahio/Elmah.Io.Extensions.Logging) – provedor para o serviço Elmah.Io</span><span class="sxs-lookup"><span data-stu-id="5f08b-420">[elmah.io](https://github.com/elmahio/Elmah.Io.Extensions.Logging) - provider for the Elmah.Io service</span></span>

* <span data-ttu-id="5f08b-421">[JSNLog](http://jsnlog.com) – registra as exceções de JavaScript e outros eventos do lado do cliente no registro do lado do servidor.</span><span class="sxs-lookup"><span data-stu-id="5f08b-421">[JSNLog](http://jsnlog.com) - logs JavaScript exceptions and other client-side events in your server-side log.</span></span>

* <span data-ttu-id="5f08b-422">[Loggr](https://github.com/imobile3/Loggr.Extensions.Logging) – provedor para o serviço Loggr</span><span class="sxs-lookup"><span data-stu-id="5f08b-422">[Loggr](https://github.com/imobile3/Loggr.Extensions.Logging) - provider for the Loggr service</span></span>

* <span data-ttu-id="5f08b-423">[NLog](https://github.com/NLog/NLog.Extensions.Logging) – provedor para a biblioteca NLog</span><span class="sxs-lookup"><span data-stu-id="5f08b-423">[NLog](https://github.com/NLog/NLog.Extensions.Logging) - provider for the NLog library</span></span>

* <span data-ttu-id="5f08b-424">[Serilog](https://github.com/serilog/serilog-extensions-logging) – provedor para a biblioteca Serilog</span><span class="sxs-lookup"><span data-stu-id="5f08b-424">[Serilog](https://github.com/serilog/serilog-extensions-logging) - provider for the Serilog library</span></span>

<span data-ttu-id="5f08b-425">Algumas estruturas de terceiros podem fazer o [log semântico, também conhecido como registro em log estruturado](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).</span><span class="sxs-lookup"><span data-stu-id="5f08b-425">Some third-party frameworks can do [semantic logging, also known as structured logging](https://softwareengineering.stackexchange.com/questions/312197/benefits-of-structured-logging-vs-basic-logging).</span></span>

<span data-ttu-id="5f08b-426">O uso de uma estrutura de terceiros é semelhante ao uso de um dos provedores internos: adicione um pacote NuGet ao seu projeto e chame um método de extensão em `ILoggerFactory`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-426">Using a third-party framework is similar to using one of the built-in providers: add a NuGet package to your project and call an extension method on `ILoggerFactory`.</span></span> <span data-ttu-id="5f08b-427">Para obter mais informações, consulte a documentação de cada estrutura.</span><span class="sxs-lookup"><span data-stu-id="5f08b-427">For more information, see each framework's documentation.</span></span>

<span data-ttu-id="5f08b-428">Você também pode criar seus próprios provedores personalizados para dar suporte a outras estruturas de registros ou a seus próprios requisitos de registro em log.</span><span class="sxs-lookup"><span data-stu-id="5f08b-428">You can create your own custom providers as well, to support other logging frameworks or your own logging requirements.</span></span>

## <a name="azure-log-streaming"></a><span data-ttu-id="5f08b-429">Fluxo de log do Azure</span><span class="sxs-lookup"><span data-stu-id="5f08b-429">Azure log streaming</span></span>

<span data-ttu-id="5f08b-430">O fluxo de log do Azure permite que você exiba a atividade de log em tempo real:</span><span class="sxs-lookup"><span data-stu-id="5f08b-430">Azure log streaming enables you to view log activity in real time from:</span></span> 

* <span data-ttu-id="5f08b-431">Do servidor de aplicativos</span><span class="sxs-lookup"><span data-stu-id="5f08b-431">The application server</span></span> 
* <span data-ttu-id="5f08b-432">Do servidor Web</span><span class="sxs-lookup"><span data-stu-id="5f08b-432">The web server</span></span>
* <span data-ttu-id="5f08b-433">De uma solicitação de rastreio com falha</span><span class="sxs-lookup"><span data-stu-id="5f08b-433">Failed request tracing</span></span> 

<span data-ttu-id="5f08b-434">Para configurar o fluxo de log do Azure:</span><span class="sxs-lookup"><span data-stu-id="5f08b-434">To configure Azure log streaming:</span></span> 

* <span data-ttu-id="5f08b-435">Navegue até a página **Logs de Diagnóstico** da página do portal do seu aplicativo</span><span class="sxs-lookup"><span data-stu-id="5f08b-435">Navigate to the **Diagnostics Logs** page from your application's portal page</span></span>
* <span data-ttu-id="5f08b-436">Defina o **Log de aplicativo (Sistema de Arquivos)** como ativado.</span><span class="sxs-lookup"><span data-stu-id="5f08b-436">Set **Application Logging (Filesystem)** to on.</span></span> 

![Página de logs de diagnóstico do Portal do Azure](index/_static/azure-diagnostic-logs.png)

<span data-ttu-id="5f08b-438">Navegue até a página **Fluxo de Log** para exibir as mensagens de aplicativo.</span><span class="sxs-lookup"><span data-stu-id="5f08b-438">Navigate to the **Log Streaming** page to view application messages.</span></span> <span data-ttu-id="5f08b-439">Elas são registradas pelo aplicativo por meio da interface `ILogger`.</span><span class="sxs-lookup"><span data-stu-id="5f08b-439">They are logged by application through the `ILogger` interface.</span></span> 

![Fluxo de log do aplicativo do Portal do Azure](index/_static/azure-log-streaming.png)


## <a name="see-also"></a><span data-ttu-id="5f08b-441">Consulte também</span><span class="sxs-lookup"><span data-stu-id="5f08b-441">See also</span></span>

[<span data-ttu-id="5f08b-442">Registro em log de alto desempenho com LoggerMessage</span><span class="sxs-lookup"><span data-stu-id="5f08b-442">High-performance logging with LoggerMessage</span></span>](xref:fundamentals/logging/loggermessage)