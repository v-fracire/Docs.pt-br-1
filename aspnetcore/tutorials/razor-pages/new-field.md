---
title: Adicionar um novo campo em uma página Razor no ASP.NET Core
author: rick-anderson
description: Mostra como adicionar um novo campo a uma página Razor com o Entity Framework Core
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/5/2018
uid: tutorials/razor-pages/new-field
ms.openlocfilehash: e280bc9553113982a1f1a77eabab32575c905237
ms.sourcegitcommit: 9bb58d7c8dad4bbd03419bcc183d027667fefa20
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 12/04/2018
ms.locfileid: "52862285"
---
# <a name="add-a-new-field-to-a-razor-page-in-aspnet-core"></a>Adicionar um novo campo em uma página Razor no ASP.NET Core

Por [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE[](~/includes/rp/download.md)]

Nesta seção, as Migrações do [Entity Framework](/ef/core/get-started/aspnetcore/new-db) Code First são usadas para:

* Adicionar um novo campo ao modelo.
* Migrar a nova alteração de esquema de campo para o banco de dados.

Ao usar o Code First do EF para criar automaticamente um banco de dados, o Code First:

* Adiciona uma tabela ao banco de dados para acompanhar se o esquema do banco de dados está sincronizado com as classes de modelo das quais ele foi gerado.
* Se as classes de modelo não estiverem em sincronia com o banco de dados, o EF gerará uma exceção.

Verificação automática de esquema/modelo em sincronia torna mais fácil encontrar problemas de código/banco de dados inconsistente.

## <a name="adding-a-rating-property-to-the-movie-model"></a>Adicionando uma propriedade de classificação ao modelo de filme

Abra o arquivo *Models/Movie.cs* e adicione uma propriedade `Rating`:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/MovieDateRating.cs?highlight=13&name=snippet)]

Crie o aplicativo.

Edite *Pages/Movies/Index.cshtml* e adicione um campo `Rating`:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexRating.cshtml.?highlight=40-42,61-63)]

Atualize as seguintes páginas:

* Adicione o campo `Rating` às páginas Excluir e Detalhes.
* Atualize [Create.cshtml](https://github.com/aspnet/Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Create.cshtml) com um campo `Rating`.
* Adicione o campo `Rating` à página Editar.

O aplicativo não funcionará até que o BD seja atualizado para incluir o novo campo. Se for executado agora, o aplicativo gerará uma `SqlException`:

`SqlException: Invalid column name 'Rating'.`

Esse erro é causado devido à classe de modelo Movie atualizada ser diferente do esquema da tabela Movie do banco de dados. (Não há nenhuma coluna `Rating` na tabela de banco de dados.)

Existem algumas abordagens para resolver o erro:

1. Faça com que o Entity Framework remova automaticamente e recrie o banco de dados usando o novo esquema de classe de modelo. Essa abordagem é conveniente no início do ciclo de desenvolvimento; ela permite que você desenvolva rapidamente o modelo e o esquema de banco de dados juntos. A desvantagem é que você perde os dados existentes no banco de dados. Não use essa abordagem em um banco de dados de produção. A remoção do BD em alterações de esquema e o uso de um inicializador para propagar automaticamente o banco de dados com os dados de teste é muitas vezes uma maneira produtiva de desenvolver um aplicativo.

2. Modifique explicitamente o esquema do banco de dados existente para que ele corresponda às classes de modelo. A vantagem dessa abordagem é que você mantém os dados. Faça essa alteração manualmente ou criando um script de alteração de banco de dados.

3. Use as Migrações do Code First para atualizar o esquema de banco de dados.

Para este tutorial, use as Migrações do Code First.

Atualize a classe `SeedData` para que ela forneça um valor para a nova coluna. Uma alteração de amostra é mostrada abaixo, mas é recomendável fazer essa alteração em cada bloco `new Movie`.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs?name=snippet1&highlight=8)]

Consulte o [arquivo SeedData.cs concluído](https://github.com/aspnet/Docs/blob/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie22/Models/SeedDataRating.cs).

Compile a solução.

<!-- VS -------------------------->
# <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

<a name="pmc"></a>

### <a name="add-a-migration-for-the-rating-field"></a>Adicionar uma migração para o campo de classificação

No menu **Ferramentas**, selecione **Gerenciador de Pacotes NuGet > Console do Gerenciador de Pacotes**.
No PMC, insira os seguintes comandos:

```powershell
Add-Migration Rating
Update-Database
```

O comando `Add-Migration` informa à estrutura:

* Compare o modelo `Movie` com o esquema de BD `Movie`.
* Crie um código para migrar o esquema de BD para o novo modelo.

O nome “Classificação” é arbitrário e é usado para nomear o arquivo de migração. É útil usar um nome significativo para o arquivo de migração.

<a name="ssox"></a>

Se você excluir todos os registros no BD, o inicializador propagará o BD e incluirá o campo `Rating`. Faça isso com os links Excluir no navegador ou no [SSOX](xref:tutorials/razor-pages/sql#ssox) (Pesquisador de Objetos do SQL Server). Para excluir o banco de dados do SSOX:

* Selecione o banco de dados no SSOX.
* Clique com o botão direito do mouse no banco de dados e selecione *Excluir*.
* Marque **Fechar conexões existentes**.
* Selecione **OK**.
* No [PMC](xref:tutorials/razor-pages/new-field#pmc), atualize o banco de dados:

  ```powershell
  Update-Database
  ```

<!-- Code -------------------------->
# <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<!-- copy/paste this tab to the next. Not worth an include  --> O SQLite não dá suporte a migrações.

* Exclua o banco de dados ou altere o nome do banco de dados no arquivo *appsettings.json*.
* Exclua a pasta *Migrations* (e todos os arquivos dela).

Execute os seguintes comandos da CLI do .NET Core:

```console
dotnet ef migrations add Rating
dotnet ef database update
```

<!-- Mac -------------------------->
# <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio para Mac](#tab/visual-studio-mac)

O SQLite não dá suporte a migrações.

* Exclua o banco de dados ou altere o nome do banco de dados no arquivo *appsettings.json*.
* Exclua a pasta *Migrations* (e todos os arquivos dela).

Execute os seguintes comandos da CLI do .NET Core:

```console
dotnet ef migrations add Rating
dotnet ef database update
```

---  
<!-- End of VS tabs -->

Execute o aplicativo e verifique se você pode criar/editar/exibir filmes com um campo `Rating`. Se o banco de dados não for propagado, defina um ponto de interrupção no método `SeedData.Initialize`.

> [!div class="step-by-step"]
> [Anterior: Adicionando uma pesquisa](xref:tutorials/razor-pages/search)
> [Próximo: Adicionando Validação](xref:tutorials/razor-pages/validation)
