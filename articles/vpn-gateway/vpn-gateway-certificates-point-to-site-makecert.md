---
title: 產生並匯出點對站的憑證：MakeCert：Azure | Microsoft Docs
description: 建立自我簽署的根憑證、匯出公開金鑰，以及使用 MakeCert 產生用戶端憑證。
services: vpn-gateway
documentationcenter: na
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/05/2018
ms.author: cherylmc
ms.openlocfilehash: 3ff7e754a55e15a8fa8a32f846efbbbe5025e46e
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44297854"
---
# <a name="generate-and-export-certificates-for-point-to-site-connections-using-makecert"></a>使用 MakeCert 來產生並匯出點對站連線的憑證

點對站連線使用憑證進行驗證。 本文說明如何建立自我簽署的根憑證，以及使用 MakeCert 來產生用戶端憑證。 如果您要尋找不同的憑證指示，請參閱[憑證 - PowerShell](vpn-gateway-certificates-point-to-site.md) 或[憑證 - Linux](vpn-gateway-certificates-point-to-site-linux.md)。

雖然建議您使用 [Windows 10 PowerShell 步驟](vpn-gateway-certificates-point-to-site.md)建立您的憑證，但是提供這些 MakeCert 指示作為選擇性方法。 您使用任一種方法所產生的憑證可以安裝於[任何支援的用戶端作業系統](vpn-gateway-howto-point-to-site-resource-manager-portal.md#faq)。 不過，MakeCert 具有下列限制：

* MakeCert 已被取代。 這表示無法在任何時間點移除這項工具。 當無法再使用 MakeCert 時，任何您已經使用 MakeCert 所產生的憑證將不會受到影響。 MakeCert 只用來產生憑證，而不是驗證機制。

## <a name="rootcert"></a>建立自我簽署根憑證

下列步驟說明如何使用 MakeCert 來建立自我簽署憑證。 這些並非部署模型特定的步驟。 它們同樣適用於資源管理員和傳統部署模型。

1. 下載並安裝 [MakeCert](https://msdn.microsoft.com/library/windows/desktop/aa386968(v=vs.85).aspx)。
2. 安裝之後，您通常可以在下列路徑中找到 makecert.exe 公用程式：'C:\Program Files (x86)\Windows Kits\10\bin\<arch>'。 雖然，它有可能已安裝到另一個位置。 以系統管理員身分開啟命令提示字元，然後瀏覽至 MakeCert 公用程式的位置。 您可以使用下列範例，並針對適當的位置進行調整：

  ```cmd
  cd C:\Program Files (x86)\Windows Kits\10\bin\x64
  ```
3. 在您電腦上的 [個人] 憑證存放區中建立並安裝憑證。 下列範例會建立對應的 .cer 檔案，您在設定 P2S 時會將此檔案上傳至 Azure。 將 'P2SRootCert' 和 'P2SRootCert.cer' 取代為您想使用的憑證名稱。 憑證位於您的 '[憑證 - 目前的使用者]\[個人]\[憑證]' 中。

  ```cmd
  makecert -sky exchange -r -n "CN=P2SRootCert" -pe -a sha256 -len 2048 -ss My
  ```

## <a name="cer"></a>匯出公開金鑰 (.cer)

[!INCLUDE [Export public key](../../includes/vpn-gateway-certificates-export-public-key-include.md)]

匯出的 .cer 檔案必須上傳到 Azure。 如需相關指示，請參閱[設定點對站連線](vpn-gateway-howto-point-to-site-resource-manager-portal.md#uploadfile)。 若要新增其他可信任的根憑證，請參閱這篇文章的[本節](vpn-gateway-howto-point-to-site-resource-manager-portal.md#add)。

### <a name="export-the-self-signed-certificate-and-private-key-to-store-it-optional"></a>匯出自我簽署憑證和私密金鑰來儲存它 (選擇性)

您可能想要匯出自我簽署的根憑證，並將它安全地儲存。 如有需要，您可以稍後在另一部電腦上安裝這個自我簽署憑證，然後產生更多用戶端憑證，或匯出另一個 .cer 檔案。 若要將自我簽署的根憑證匯出為 .pfx，請選取根憑證，然後使用與[匯出用戶端憑證](#clientexport)所述的相同步驟來匯出。

## <a name="create-and-install-client-certificates"></a>建立並安裝用戶端憑證

您未直接在用戶端電腦上安裝自我簽署的憑證。 您需要從自我簽署憑證產生用戶端憑證。 您接著會將用戶端憑證匯出並安裝到用戶端電腦。 下列步驟並非針對特定部署模型。 它們同樣適用於資源管理員和傳統部署模型。

### <a name="clientcert"></a>產生用戶端憑證 

每個使用點對站連線至 VNet 的用戶端電腦都必須安裝用戶端憑證。 您可以從自我簽署根憑證產生用戶端憑證，然後匯出及安裝用戶端憑證。 如果未安裝用戶端憑證，則驗證會失敗。 

下列步驟將逐步引導您完成從自我簽署的根憑證產生用戶端憑證。 您可以從相同根憑證產生多個用戶端憑證。 當您使用下列步驟產生用戶端憑證時，用戶端憑證會自動安裝在您用來產生憑證的電腦上。 如果您想要在另一部用戶端電腦上安裝用戶端憑證，您可以匯出憑證。
 
1. 在用來建立自我簽署憑證的相同電腦上，以系統管理員身分開啟命令提示字元。
2. 修改並執行範例以產生用戶端憑證。
  * 將 *"P2SRootCert"* 變更為您從中產生用戶端憑證的自我簽署根憑證名稱。 確定您使用的是根憑證名稱，亦即您建立自我簽署根憑證時所指定的 'CN=' 值。
  * 將 *P2SChildCert* 變更為所產生的用戶端憑證要使用的名稱。

  如果您執行以下範例而未做任何修改，您的個人憑證存放區中就會有一個從根憑證 P2SRootCert 產生的用戶端憑證，名為 P2SChildcert。

  ```cmd
  makecert.exe -n "CN=P2SChildCert" -pe -sky exchange -m 96 -ss My -in "P2SRootCert" -is my -a sha256
  ```

### <a name="clientexport"></a>匯出用戶端憑證

[!INCLUDE [Export client certificate](../../includes/vpn-gateway-certificates-export-client-cert-include.md)]

### <a name="install"></a>安裝匯出的用戶端憑證

若要安裝用戶端憑證，請參閱[安裝用戶端憑證](point-to-site-how-to-vpn-client-install-azure-cert.md)。

## <a name="next-steps"></a>後續步驟

繼續使用您的點對站設定。 

* 如需 **Resource Manager** 部署模型的步驟，請參閱[使用原生 Azure 憑證驗證來設定 P2S](vpn-gateway-howto-point-to-site-resource-manager-portal.md)。
* 如需**傳統**部署模型步驟，請參閱[設定 VNet 的點對站 VPN 連線 (傳統)](vpn-gateway-howto-point-to-site-classic-azure-portal.md)。

如需有關為 P2S 疑難排解的資訊，請參閱[針對 Azure 點對站連線進行疑難排解](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)。