---
title: Habilitar a geração de código QR TOTP para aplicativos de autenticador no ASP.NET Core
author: rick-anderson
description: Descubra como habilitar a geração de código QR para aplicativos de autenticador TOTP que funcionam com autenticação de dois fatores de núcleo do ASP.NET.
ms.author: riande
ms.date: 08/14/2018
uid: security/authentication/identity-enable-qrcodes
ms.openlocfilehash: 437f354f71128a98bae9abdced291e04efc9f48e
ms.sourcegitcommit: fc7eb4243188950ae1f1b52669edc007e9d0798d
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/07/2018
ms.locfileid: "51225376"
---
# <a name="enable-qr-code-generation-for-totp-authenticator-apps-in-aspnet-core"></a>Habilitar a geração de código QR TOTP para aplicativos de autenticador no ASP.NET Core

::: moniker range="<= aspnetcore-2.0"

Códigos QR exige o ASP.NET Core 2.0 ou posterior.

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

ASP.NET Core é fornecido com suporte para aplicativos de autenticador para autenticação individual. Dois aplicativos de autenticador 2FA (autenticação) fator, usando uma baseada em tempo avulso senha algoritmo TOTP (), são o setor de abordagem 2fa recomendado. 2FA usar TOTP é preferencial para SMS 2FA. Um aplicativo autenticador fornece um código de 6 a 8 dígitos que os usuários devem inserir depois de confirmar seu nome de usuário e senha. Normalmente, um aplicativo autenticador é instalado em um Smartphone.

Os modelos de aplicativo web ASP.NET Core autenticadores de suporte, mas não fornecem suporte para a geração de QRCode. Geradores de QRCode facilitam a configuração do 2FA. Este documento o orientará durante a adição [código QR](https://wikipedia.org/wiki/QR_code) geração para a página de configuração 2FA.

## <a name="adding-qr-codes-to-the-2fa-configuration-page"></a>Adicionar códigos QR para a página de configuração 2FA

Usam estas instruções *qrcode.js* do https://davidshimjs.github.io/qrcodejs/ repositório.

* Baixe o [biblioteca de javascript qrcode.js](https://davidshimjs.github.io/qrcodejs/) para o `wwwroot\lib` pasta em seu projeto.

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

* Siga as instruções em [identidade de Scaffold](xref:security/authentication/scaffold-identity) para gerar */Areas/Identity/Pages/Account/Manage/EnableAuthenticator.cshtml*.
* Na */Areas/Identity/Pages/Account/Manage/EnableAuthenticator.cshtml*, localize o `Scripts` seção no final do arquivo:

::: moniker-end

::: moniker range="= aspnetcore-2.0"

* Na *Pages/Account/Manage/EnableAuthenticator.cshtml* (páginas do Razor) ou *Views/Manage/EnableAuthenticator.cshtml* (MVC), localize o `Scripts` seção no final do arquivo:

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

```cshtml
@section Scripts {
    @await Html.PartialAsync("_ValidationScriptsPartial")
}
```

* Atualizar o `Scripts` seção para adicionar uma referência para o `qrcodejs` biblioteca que você adicionou e uma chamada para gerar o código QR. Ele deve ser da seguinte maneira:

```cshtml
@section Scripts {
    @await Html.PartialAsync("_ValidationScriptsPartial")

    <script type="text/javascript" src="~/lib/qrcode.js"></script>
    <script type="text/javascript">
        new QRCode(document.getElementById("qrCode"),
            {
                text: "@Html.Raw(Model.AuthenticatorUri)",
                width: 150,
                height: 150
            });
    </script>
}
```

* Exclua o parágrafo que links para essas instruções.

Executar seu aplicativo e certifique-se de que você pode digitalizar o código QR e validar o código que comprova o autenticador.

## <a name="change-the-site-name-in-the-qr-code"></a>Alterar o nome do site em que o código QR

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

O nome do site em que o código QR é obtido do nome do projeto que você escolher ao criar inicialmente o seu projeto. Você pode alterá-lo procurando os `GenerateQrCodeUri(string email, string unformattedKey)` método na */Areas/Identity/Pages/Account/Manage/EnableAuthenticator.cshtml*.

::: moniker-end

::: moniker range="= aspnetcore-2.0"

O nome do site em que o código QR é obtido do nome do projeto que você escolher ao criar inicialmente o seu projeto. Você pode alterá-lo procurando a `GenerateQrCodeUri(string email, string unformattedKey)` método na *Pages/Account/Manage/EnableAuthenticator.cshtml.cs* arquivo (páginas do Razor) ou o *Controllers/ManageController.cs* arquivo (MVC).

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

O código padrão do modelo será semelhante ao seguinte:

```csharp
private string GenerateQrCodeUri(string email, string unformattedKey)
{
    return string.Format(
        AuthenicatorUriFormat,
        _urlEncoder.Encode("Razor Pages"),
        _urlEncoder.Encode(email),
        unformattedKey);
}
```

O segundo parâmetro na chamada para `string.Format` é o nome do site, obtido do nome da solução. Ele pode ser alterado para qualquer valor, mas ele sempre deve ser codificado por URL.

## <a name="using-a-different-qr-code-library"></a>Usando uma biblioteca diferente do código QR

Você pode substituir a biblioteca de código QR com sua biblioteca preferencial. O HTML não contém um `qrCode` fornece de sua biblioteca de elemento no qual você pode colocar um código QR por qualquer mecanismo.

A URL formatada corretamente para o código QR está disponível na:

* `AuthenticatorUri` propriedade do modelo.
* `data-url` propriedade no `qrCodeData` elemento.

## <a name="totp-client-and-server-time-skew"></a>TOTP cliente e servidor distorção de tempo

Autenticação de TOTP (senha de uso único baseados em tempo) depende do dispositivo o servidor e o autenticador, tendo a hora correta. Tokens duram por 30 segundos. Se os logons de 2FA TOTP estiverem falhando, verifique se a hora do servidor é precisos e preferencialmente sincronizado a um serviço NTP preciso.

::: moniker-end
