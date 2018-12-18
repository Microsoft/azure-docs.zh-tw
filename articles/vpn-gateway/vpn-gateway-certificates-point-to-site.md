---
title: 產生並匯出點對站的憑證：PowerShell：Azure | Microsoft Docs
description: 建立自我簽署的根憑證、匯出公開金鑰，以及使用 Windows 10 或 Windows Server 2016 中的 PowerShell 產生用戶端憑證。
services: vpn-gateway
documentationcenter: na
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/05/2018
ms.author: cherylmc
ms.openlocfilehash: 18d705f68ff06621e30c051dac9fb9607fd043ac
ms.sourcegitcommit: f3bd5c17a3a189f144008faf1acb9fabc5bc9ab7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/10/2018
ms.locfileid: "44300931"
---
# <a name="generate-and-export-certificates-for-point-to-site-using-powershell"></a>使用 PowerShell 來產生並匯出點對站的憑證

點對站連線使用憑證進行驗證。 本文說明如何使用 Windows 10 或 Windows Server 2016 中的 PowerShell 建立自我簽署的根憑證，以及產生用戶端憑證。 如果您要尋找不同的憑證指示，請參閱[憑證 - Linux](vpn-gateway-certificates-point-to-site-linux.md) 或[憑證 - MakeCert](vpn-gateway-certificates-point-to-site-makecert.md)。

您必須在執行 Windows 10 或 Windows Server 2016 的電腦上執行本文中的步驟。 用於產生憑證的 PowerShell Cmdlet 是作業系統的一部分，在其他 Windows 版本上無法運作。 因此，您需要 Windows 10 或 Windows Server 2016 電腦來產生憑證。 產生憑證之後，您即可上傳憑證或將其安裝在任何支援的用戶端作業系統上。 

如果您無法使用 Windows 10 或 Windows Server 2016 電腦，則可以使用 [MakeCert](vpn-gateway-certificates-point-to-site-makecert.md) 來產生憑證。 使用任一種方法所產生的憑證均可安裝在任何[支援的](vpn-gateway-howto-point-to-site-resource-manager-portal.md#faq)用戶端作業系統上。

## <a name="rootcert"></a>1.建立自我簽署根憑證

您可以使用 New-SelfSignedCertificate Cmdlet 來建立自我簽署的根憑證。 如需其他的參數資訊，請參閱 [New-SelfSignedCertificate](https://technet.microsoft.com/itpro/powershell/windows/pkiclient/new-selfsignedcertificate)。

1. 從執行 Windows 10 或 Windows Server 2016 的電腦，以提高的權限開啟 Windows PowerShell 主控台。
2. 使用下列範例建立自我簽署的根憑證。 下列範例會建立名為 'P2SRootCert' 的自我簽署的根憑證，其自動安裝在 'Certificates-Current User\Personal\Certificates' 中。 您可以開啟 *certmgr.msc* 或 [管理使用者憑證] 來檢視憑證。

  ```powershell
  $cert = New-SelfSignedCertificate -Type Custom -KeySpec Signature `
  -Subject "CN=P2SRootCert" -KeyExportPolicy Exportable `
  -HashAlgorithm sha256 -KeyLength 2048 `
  -CertStoreLocation "Cert:\CurrentUser\My" -KeyUsageProperty Sign -KeyUsage CertSign
  ```

## <a name="clientcert"></a>2.產生用戶端憑證 

每個使用點對站連線至 VNet 的用戶端電腦都必須安裝用戶端憑證。 您可以從自我簽署根憑證產生用戶端憑證，然後匯出及安裝用戶端憑證。 如果未安裝用戶端憑證，則驗證會失敗。 

下列步驟將逐步引導您完成從自我簽署的根憑證產生用戶端憑證。 您可以從相同根憑證產生多個用戶端憑證。 當您使用下列步驟產生用戶端憑證時，用戶端憑證會自動安裝在您用來產生憑證的電腦上。 如果您想要在另一部用戶端電腦上安裝用戶端憑證，您可以匯出憑證。

此範例會使用 New-SelfSignedCertificate Cmdlet 來產生有效期為一年的用戶端憑證。 如需其他的參數資訊 (例如針對用戶端憑證設定不同的到期值)，請參閱 [New-SelfSignedCertificate](https://technet.microsoft.com/itpro/powershell/windows/pkiclient/new-selfsignedcertificate)。

### <a name="example-1"></a>範例 1

此範例會使用從上一節宣告的 '$cert' 變數。 如果您在建立自我簽署根憑證之後關閉 PowerShell 主控台，或是在新的 PowerShell 主控台工作階段中建立其他用戶端憑證，請使用[範例 2](#ex2) 中的步驟。

修改並執行範例以產生用戶端憑證。 如果您執行下列範例，但未加以修改，結果會是名為 'P2SChildCert' 的用戶端憑證。  如果您要將子憑證命名為其他名稱，請修改 CN 值。 執行這個範例時，請勿變更 TextExtension。 您產生的用戶端憑證會自動安裝在您電腦的 'Certificates - Current User\Personal\Certificates' 中。

```powershell
New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
-Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
-HashAlgorithm sha256 -KeyLength 2048 `
-CertStoreLocation "Cert:\CurrentUser\My" `
-Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
```

### <a name="ex2"></a>範例 2

如果您要建立其他用戶端憑證，或者不是使用您用來建立自我簽署根憑證的相同 PowerShell 工作階段，請使用下列步驟︰

1. 識別安裝在電腦上的自我簽署根憑證。 此 Cmdlet 會傳回安裝於電腦上的憑證清單。

  ```powershell
  Get-ChildItem -Path “Cert:\CurrentUser\My”
  ```
2. 從傳回的清單尋找主體名稱，然後將其旁邊的指紋複製到文字檔。 在下列範例中，有兩個憑證。 CN 名稱是您要從中產生子憑證之自我簽署根憑證的名稱。 在此例中為 'P2SRootCert'。

  ```
  Thumbprint                                Subject
  
  AED812AD883826FF76B4D1D5A77B3C08EFA79F3F  CN=P2SChildCert4
  7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655  CN=P2SRootCert
  ```
3. 使用上一個步驟中的指紋，為根憑證宣告一個變數。 將 THUMBPRINT 替換為您要從中產生子憑證之根憑證的指紋。

  ```powershell
  $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\THUMBPRINT"
  ```

  例如，使用上一個步驟中的 P2SRootCert 的指紋，變數會如下所示︰

  ```powershell
  $cert = Get-ChildItem -Path "Cert:\CurrentUser\My\7181AA8C1B4D34EEDB2F3D3BEC5839F3FE52D655"
  ```
4.  修改並執行範例以產生用戶端憑證。 如果您執行下列範例，但未加以修改，結果會是名為 'P2SChildCert' 的用戶端憑證。 如果您要將子憑證命名為其他名稱，請修改 CN 值。 執行這個範例時，請勿變更 TextExtension。 您產生的用戶端憑證會自動安裝在您電腦的 'Certificates - Current User\Personal\Certificates' 中。

  ```powershell
  New-SelfSignedCertificate -Type Custom -DnsName P2SChildCert -KeySpec Signature `
  -Subject "CN=P2SChildCert" -KeyExportPolicy Exportable `
  -HashAlgorithm sha256 -KeyLength 2048 `
  -CertStoreLocation "Cert:\CurrentUser\My" `
  -Signer $cert -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2")
  ```

## <a name="cer"></a>3.匯出根憑證公開金鑰 (.cer)

[!INCLUDE [Export public key](../../includes/vpn-gateway-certificates-export-public-key-include.md)]


### <a name="export-the-self-signed-root-certificate-and-private-key-to-store-it-optional"></a>匯出自我簽署根憑證和私密金鑰來儲存它 (選擇性)

您可能想要匯出自我簽署的根憑證，並將它安全地儲存作為備份。 如有需要，您可以稍後在另一部電腦上安裝這個自我簽署憑證，然後產生更多用戶端憑證。 若要將自我簽署的根憑證匯出為 .pfx，請選取根憑證，然後使用與[匯出用戶端憑證](#clientexport)所述的相同步驟來匯出。

## <a name="clientexport"></a>4.匯出用戶端憑證

[!INCLUDE [Export client certificate](../../includes/vpn-gateway-certificates-export-client-cert-include.md)]


## <a name="install"></a>5.安裝匯出的用戶端憑證

透過 P2S 連線連接至 VNet 的每個用戶端都需要以本機方式安裝用戶端憑證。

若要安裝用戶端憑證，請參閱[安裝點對站連線的用戶端憑證](point-to-site-how-to-vpn-client-install-azure-cert.md)。

## <a name="install"></a>6.繼續進行 P2S 組態步驟

繼續使用您的點對站設定。

* 如需 **Resource Manager** 部署模型的步驟，請參閱[使用原生 Azure 憑證驗證來設定 P2S](vpn-gateway-howto-point-to-site-resource-manager-portal.md)。 
* 如需**傳統**部署模型步驟，請參閱[設定 VNet 的點對站 VPN 連線 (傳統)](vpn-gateway-howto-point-to-site-classic-azure-portal.md)。
* 如需 P2S 疑難排解詳細資訊，請參閱[針對 Azure 點對站連線進行疑難排解](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md)。
