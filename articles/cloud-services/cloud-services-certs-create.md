---
title: 雲端服務和管理憑證 | Microsoft Docs
description: 了解如何建立憑證並搭配 Microsoft Azure 使用
services: cloud-services
documentationcenter: .net
author: jpconnock
manager: timlt
editor: ''
ms.assetid: fc70d00d-899b-4771-855f-44574dc4bfc6
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/19/2017
ms.author: jeconnoc
ms.openlocfilehash: 5c16f06d0cc031cd9b51a3c6cf0beb149a19aeb4
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39001434"
---
# <a name="certificates-overview-for-azure-cloud-services"></a>Azure 雲端服務的憑證概觀
在 Azure 中，憑證用於雲端服務 ([服務憑證](#what-are-service-certificates)) 與驗證管理 API ([管理憑證](#what-are-management-certificates))。 本主題提供兩種憑證類型的一般概觀、如何[建立](#create)這些憑證類型，以及如何將其[部署](#deploy)到 Azure。

在 Azure 中使用的憑證是 x.509 v3 憑證，而且可以由其他受信任的憑證簽署，或者可以自我簽署。 自我簽署的憑證是由自己的建立者簽署，因此預設不受到信任。 大部分的瀏覽器都可以忽略這個問題。 您應該僅在開發和測試雲端服務時使用自我簽署的憑證。 

Azure 所使用的憑證可以包含私密或公開金鑰。 憑證具有指紋，可以明確的方式提供識別它們的方法。 這個指紋用於 Azure [組態檔](cloud-services-configure-ssl-certificate-portal.md) 中，以識別雲端服務應該使用哪個憑證。 

## <a name="what-are-service-certificates"></a>什麼是服務憑證？
服務憑證會附加至雲端服務，並啟用進出服務的安全通訊。 例如，如果您部署了一個 Web 角色，您會想要提供可以驗證公開的 HTTPS 端點的憑證。 在服務定義中定義的服務憑證會自動部署至執行您角色的執行個體的虛擬機器。 

您可以使用 Azure 入口網站或者傳統部署模型，將服務憑證上傳至 Azure。 服務憑證是與特定的雲端服務相關聯。 它們指派給服務定義檔中的部署。

服務憑證可以與您的服務分開管理，而且可以由不同的人員管理。 例如，開發人員所上傳的服務封裝，指的可能是 IT 管理員先前上傳至 Azure 的憑證。 IT 管理員可以管理和更新該憑證 (變更服務的組態)，而不需要上傳新的服務套件。 不使用新的服務套件進行更新是可行的，因為憑證的邏輯名稱、存放區名稱及位置是在服務定義檔中指定，而憑證指紋是在服務組態檔中指定。 若要更新憑證，只需要上傳新憑證，並變更服務組態檔中的憑證指紋值即可。

>[!Note]
>[雲端服務常見問題集](cloud-services-faq.md)文章有一些很有幫助的憑證相關資訊。

## <a name="what-are-management-certificates"></a>什麼是管理憑證？
管理憑證可讓您使用傳統部署模型進行驗證。 許多程式和工具 (例如 Visual Studio 或 Azure SDK) 會使用這些憑證，將各種 Azure 服務的設定與部署自動化。 這些並不是真的與雲端服務相關。 

> [!WARNING]
> 請務必小心！ 這些憑證類型允許使用它們進行驗證的任何人管理與他們相關聯的訂用帳戶。 
> 
> 

### <a name="limitations"></a>限制
每個訂用帳戶有 100 個管理憑證的限制。 此外，在特定服務管理員的使用者識別碼底下的所有訂用帳戶也有 100 個管理憑證的限制。 如果帳戶系統管理員的使用者識別碼已經用來加入 100 個管理憑證，而且還需要有更多憑證，您可以新增共同管理員來加入其他憑證。 

<a name="create"></a>
## <a name="create-a-new-self-signed-certificate"></a>建立新的自我簽署憑證
您可以使用任何可用的工具建立自我簽署的憑證，前提是，這些憑證遵守以下設定：

* X.509 憑證。
* 包含一個私密金鑰。
* 針對金鑰交換 (.pfx 檔案) 而建立。
* 主體名稱必須符合用來存取雲端服務的網域。

    > 您無法取得 cloudapp.net 網域 (或針對任何 Azure 相關網域) 的 SSL 憑證；憑證的主體名稱必須符合用來存取應用程式的自訂網域名稱。 例如，**contoso.net**，而非 **contoso.cloudapp.net**。

* 至少為 2048 位元加密。
* **僅限服務憑證**：用戶端憑證必須位於 *個人* 憑證存放區。

在 Windows 上建立憑證有兩個簡單的方法：使用 `makecert.exe` 公用程式或 IIS。

### <a name="makecertexe"></a>Makecert.exe
此公用程式已被取代，此處不再說明。 如需詳細資訊，請參閱[此 MSDN 文章](https://msdn.microsoft.com/library/windows/desktop/aa386968)。

### <a name="powershell"></a>PowerShell
```powershell
$cert = New-SelfSignedCertificate -DnsName yourdomain.cloudapp.net -CertStoreLocation "cert:\LocalMachine\My" -KeyLength 2048 -KeySpec "KeyExchange"
$password = ConvertTo-SecureString -String "your-password" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath ".\my-cert-file.pfx" -Password $password
```

> [!NOTE]
> 如果您想要搭配 IP 位址 (而不是網域) 來使用憑證，請在 -DnsName 參數中使用 IP 位址。


如果您想要使用這個 [憑證搭配管理入口網站](../azure-api-management-certs.md)，請將它匯出至 **.cer** 檔案：

```powershell
Export-Certificate -Type CERT -Cert $cert -FilePath .\my-cert-file.cer
```

### <a name="internet-information-services-iis"></a>網際網路資訊服務 (IIS)
在網際網路上有許多涵蓋如何使用 IIS 執行這項操作的網頁。 [這裡](https://www.sslshopper.com/article-how-to-create-a-self-signed-certificate-in-iis-7.html) 是我找到的其中一個我認為說明得很好的網頁。 

### <a name="linux"></a>Linux
[本文](../virtual-machines/linux/mac-create-ssh-keys.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)說明如何使用 SSH 建立憑證。

## <a name="next-steps"></a>後續步驟
[將服務憑證上傳到 Azure 入口網站](cloud-services-configure-ssl-certificate-portal.md)。

將[管理 API 憑證](../azure-api-management-certs.md)上傳至 Azure 入口網站。

