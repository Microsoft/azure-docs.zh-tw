---
title: 在 Azure App Service 中的應用程式程式碼中使用上傳的 SSL 憑證 | Microsoft Docs
description: ''
services: app-service\web
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/01/2017
ms.author: cephalin
ms.openlocfilehash: 87c9cd5955dda1a379733e5ad48d58f8361f0e6b
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44051471"
---
# <a name="use-an-ssl-certificate-in-your-application-code-in-azure-app-service"></a>在 Azure App Service 中的應用程式程式碼中使用 SSL 憑證

此作法指南會示範如何將其中一種您上傳或匯入至 App Service 應用程式的 SSL 憑證，用於應用程式的程式碼中。 使用案例的範例是應用程式要存取需要驗證憑證的外部服務。 

要在程式碼中使用 SSL 憑證的方法會利用 App Service 中的 SSL 功能，而這需要您的應用程式處於**基本**層或更高層級。 另一個替代方案是在應用程式目錄中包含憑證檔案，並將其直接載入 (請參閱[替代方案：載入檔案形式的憑證](#file))。 不過，此替代方案無法讓您隱藏憑證的私密金鑰，不讓應用程式的程式碼或開發人員知道。 此外，如果您的應用程式程式碼位於開放原始碼存放庫中，請勿選擇將帶有私密金鑰的憑證放在該存放庫中。

當您讓 App Service 管理您的 SSL 憑證時，您可以分開維護憑證以及應用程式程式碼，並保護您的敏感性資料。

## <a name="prerequisites"></a>必要條件

若要完成本作法指南：

- [建立 App Service 應用程式](/azure/app-service/)
- [將自訂 DNS 名稱對應至 Web 應用程式](app-service-web-tutorial-custom-domain.md)
- [上傳 SSL 憑證](app-service-web-tutorial-custom-ssl.md)或[匯入 App Service 憑證](web-sites-purchase-ssl-web-site.md)至您的 Web 應用程式


## <a name="load-your-certificates"></a>載入您的憑證

若要使用上傳至或匯入至 App Service 的憑證，請先讓該憑證可供您的應用程式程式碼存取。 您可以使用 `WEBSITE_LOAD_CERTIFICATES` 應用程式設定來完成此動作。

在 <a href="https://portal.azure.com" target="_blank">Azure 入口網站</a>中，開啟 Web 應用程式頁面。

在左側導覽中，按一下 [SSL 憑證]。

![Certificate uploaded](./media/app-service-web-tutorial-custom-ssl/certificate-uploaded.png)

所有針對此 Web 應用程式上傳和匯入的 SSL 憑證，皆會以這些憑證的指紋顯示。 複製您要使用的憑證指紋。

在左側導覽中，按一下 [應用程式設定]。

新增名為 `WEBSITE_LOAD_CERTIFICATES` 的應用程式，並將其值設定為憑證的指紋。 若要讓多個憑證可供存取，請使用逗號來分隔指紋值。 若要讓所有憑證都可供存取，請將值設定為 `*`。 請記住，這會將憑證放入 `CurrentUser\My` 存放區。

![設定應用程式設定](./media/app-service-web-ssl-cert-load/configure-app-setting.png)

完成時，按一下 [儲存]。

設定的憑證現在已可供程式碼使用。

## <a name="use-certificate-in-c-code"></a>在 C# 程式碼中使用憑證

一旦憑證可供存取之後，您可以透過憑證指紋在 C# 程式碼中存取憑證。 下列程式碼會載入指紋為 `E661583E8FABEF4C0BEF694CBC41C28FB81CD870` 的憑證。

```csharp
using System;
using System.Security.Cryptography.X509Certificates;

...
X509Store certStore = new X509Store(StoreName.My, StoreLocation.CurrentUser);
certStore.Open(OpenFlags.ReadOnly);
X509Certificate2Collection certCollection = certStore.Certificates.Find(
                            X509FindType.FindByThumbprint,
                            // Replace below with your certificate's thumbprint
                            "E661583E8FABEF4C0BEF694CBC41C28FB81CD870",
                            false);
// Get the first cert with the thumbprint
if (certCollection.Count > 0)
{
    X509Certificate2 cert = certCollection[0];
    // Use certificate
    Console.WriteLine(cert.FriendlyName);
}
certStore.Close();
...
```

<a name="file"></a>
## <a name="alternative-load-certificate-as-a-file"></a>替代方案：以檔案形式載入憑證

此區段會示範如何載入應用程式目錄中的憑證檔案。 

下列 C# 範例會從應用程式存放庫的 `certs` 目錄載入名為 `mycert.pfx` 的憑證。

```csharp
using System;
using System.Security.Cryptography.X509Certificates;

...
// Replace the parameter with "~/<relative-path-to-cert-file>".
string certPath = Server.MapPath("~/certs/mycert.pfx");

X509Certificate2 cert = GetCertificate(certPath, signatureBlob.Thumbprint);
...
```

