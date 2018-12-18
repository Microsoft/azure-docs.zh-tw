---
title: 使用 Resource Manager 範本建立 Azure App Service Environment
description: 說明如何使用 Resource Manager 範本建立外部或 ILB Azure App Service Environment
services: app-service
documentationcenter: na
author: ccompy
manager: stefsch
ms.assetid: 6eb7d43d-e820-4a47-818c-80ff7d3b6f8e
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/13/2017
ms.author: ccompy
ms.openlocfilehash: 92422a254bcfd5b31731dda6d1790cc85f467860
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094975"
---
# <a name="create-an-ase-by-using-an-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本立 ASE

## <a name="overview"></a>概觀
Azure App Service Environment (ASE) 可以使用網際網路可存取端點，或是 Azure 虛擬網路 (VNet) 中內部位址上的端點來建立。 使用內部端點建立時，該端點是由稱為內部負載平衡器 (ILB) 的 Azure 元件提供。 使用內部 IP 位址的 ASE 稱為 ILB ASE。 具有公用端點的 ASE 稱為外部 ASE。 

ASE 可以使用 Azure 入口網站或 Azure Resource Manager 範本來建立。 本文逐步解說使用 Resource Manager 範本建立外部 ASE 或 ILB ASE 所需的步驟和語法。 若要了解如何在 Azure 入口網站中建立 ASE，請參閱 [建立外部 ASE][MakeExternalASE] 或 [建立 ILB ASE][MakeILBASE] 。

在入口網站中建立 ASE 時，可以選擇同時建立 VNet，或選擇要部署至既有 VNet。 從範本建立 ASE 時，必須具備下列項目： 

* Resource Manager VNet。
* 該 VNet 中的子網路。 建議的 ASE 子網路大小是具有 256 個位址的 `/24`，以容納未來成長和規模調整需求。 ASE 建立之後，無法變更大小。
* 您的 VNnet 中的資源識別碼。 您可以從 Azure 入口網站中的虛擬網路屬性底下取得這項資訊。
* 您想要部署的訂用帳戶。
* 您想要部署的位置。

若要自動化 ASE 建立：

1. 從範本建立 ASE。 如果是建立外部 ASE，完成此步驟就建立完成。 如果是建立 ILB ASE，還要再執行一些工作。

2. 建立 ILB ASE 之後，會上傳與您的 ILB ASE 網域相符的 SSL 憑證。

3. 上傳的 SSL 憑證會指派給 ILB ASE 作為其「預設」SSL 憑證。  如果 ILB ASE 上的應用程式是使用指派給 ASE 的一般根網域 (例如 https://someapp.mycustomrootdomain.com))，此憑證將使用於此應用程式的 SSL 流量。


## <a name="create-the-ase"></a>建立 ASE
在 GitHub 上的[範例][quickstartasev2create]中有可建立 ASE 的 Resource Manager 範本及其相關聯的參數檔案。

如果您想要建立 ILB ASE，請使用這些 Resource Manager 範本[範例][quickstartilbasecreate]。 它們符合該使用案例。 azuredeploy.parameters.json 檔案中的大部分參數是建立 ILB ASE 和外部 ASE 的通用參數。 建立 ILB ASE 時，以下清單會呼叫特殊附註的參數或唯一參數︰

* *internalLoadBalancingMode*︰在大多數情況下，此屬性設定為 3，這表示連接埠 80/443 上的 HTTP/HTTPS 流量，以及 ASE 上的 FTP 服務所接聽的控制項/資料通道連接埠將會繫結至 ILB 配置的虛擬網路內部位址。 如果此屬性設定為 2，只有 FTP 服務相關的連接埠 (控制和資料通道) 會繫結至 ILB 位址， HTTP/HTTPS 流量將保留在公用 VIP 上。
* dnsSuffix︰這個參數定義要指派給 ASE 的預設根網域。 在 Azure App Service 的公用種變化中，所有 Web 應用程式的預設根網域皆為 azurewebsites.net 。 由於 ILB ASE 位於客戶虛擬網路的內部，所以不適合使用公用服務的預設根網域。 相反地，ILB ASE 應具有適合在公司的內部虛擬網路內使用的預設根網域。 例如，Contoso Corporation 可能會將 internal-contoso.com 的預設根網域用於只能在 Contoso 虛擬網路內解析和存取的應用程式。 
* ipSslAddressCount︰在 azuredeploy.json 檔案中，這個參數的值會自動預設為 0，因為 ILB ASE 只有單一 ILB 位址。 ILB ASE 沒有明確的 IP-SSL 位址。 因此，ILB ASE 的 IP-SSL 位址集區必須設定為零。 否則，就會發生佈建錯誤。 

填入 azuredeploy.parameters.json 檔案後，就可以使用下列 Powershell 程式碼片段建立 ASE。 將檔案路徑變更為您電腦上 Resource Manager 範本檔案的位置。 記得提供您自己的 Resource Manager 部署名稱和資源群組名稱的值：

```powershell
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzureRmResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

建立 ASE 約需一小時。 然後在入口網站中，ASE 會顯示在觸發部署之訂用帳戶的 ASE 清單中。

## <a name="upload-and-configure-the-default-ssl-certificate"></a>上傳和設定預設SSL 憑證
SSL 憑證必須與 ASE 相關聯，作為用來建立應用程式的 SSL 連線的「預設」SSL 憑證。 如果 ASE 的預設 DNS 尾碼是 *internal-contoso.com*，則連線到 https://some-random-app.internal-contoso.com 需要 **.internal-contoso.com* 的有效 SSL 憑證。 

使用內部憑證授權單位、向外部簽發者購買憑證、或使用自我簽署的憑證，取得有效的 SSL 憑證。 無論 SSL 憑證的來源為何，都需要正確設定下列憑證屬性︰

* **Subject**：此屬性必須設為 *.your-root-domain-here.com。
* **Subject Alternative Name**此屬性必須同時包含 *.your-root-domain-here.com 和 *.scm.your-root-domain-here.com。 系統將使用 your-app-name.scm.your-root-domain-here.com 形式的位址，進行與每個應用程式相關聯的 SCM/Kudu 網站的 SSL 連線。

備妥有效的 SSL 憑證，還需要兩個額外的準備步驟。 將 SSL 憑證轉換/儲存為 .pfx 檔案。 請記住，.pfx 檔案必須包含所有中繼和根憑證。 使用密碼保護其安全。

必須將 .pfx 檔案轉換成 base64 字串，因為上傳 SSL 憑證會使用 Resource Manager 範本。 因為 Resource Manager 範本是文字檔案，所以必須將 .pfx 檔案轉換成 base64 字串， 如此才能將其納入範本參數。

使用以下 PowerShell 程式碼片段來進行：

* 產生自我簽署憑證。
* 匯出 .pfx 檔案形式的憑證。
* 將 .pfx 檔案轉換為 base64 編碼的字串。
* 將 base64 編碼的字串儲存到別的檔案。 

以下 base64 編碼的 Powershell 程式碼是改寫自 [Powershell 指令碼部落格][examplebase64encoding]：

```powershell
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password     

$fileContentBytes = get-content -encoding byte $fileName
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$fileContentEncoded | set-content ($fileName + ".b64")
```

成功產生 SSL 憑證並轉換成 base64 編碼字串後，使用 GitHub 上的[設定預設 SSL 憑證][quickstartconfiguressl]範例 Resource Manager 範本。 

azuredeploy.parameters.json 檔案中有以下參數︰

* ︰設定 ILB ASE 的名稱。
* ︰包含 ILB ASE 部署所在的 Azure 區域的文字字串。  例如："South Central US (美國中南部)"。
* pfxBlobString：.pfx 檔案的 based64 編碼字串表示法。 使用稍早的程式碼片段，複製 "exportedcert.pfx.b64" 中的字串。 貼上字串作為 pfxBlobString 屬性的值。
* : The  used to secure the .pfx file.
* ︰憑證的指紋。 如果您從 Powershell 擷取此值 (例如先前程式碼片段中的 $certificate.Thumbprint )，可以直接使用該值。 如果您從 Windows 憑證對話方塊中複製此值，請記得去除多餘的空格。 CertificateThumbprint 應該像這樣：AF3143EB61D43F6727842115BB7F17BBCECAECAE。
* ︰您自己選擇的好記字串識別碼，可用來識別憑證。 Microsoft.Web/certificates 實體表示 SSL 憑證，而此名稱會作為實體的唯一 Resource Manager 識別碼的一部分。 名稱「必須」以下列尾碼結尾︰\_yourASENameHere_InternalLoadBalancingASE。 Azure 入口網站會以這個尾碼為指標，表示憑證要用於保護啟用 ILB 的 ASE。

以下是 azuredeploy.parameters.json  的縮簡範例︰

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "appServiceEnvironmentName": {
      "value": "yourASENameHere"
    },
    "existingAseLocation": {
      "value": "East US 2"
    },
    "pfxBlobString": {
      "value": "MIIKcAIBAz...snip...snip...pkCAgfQ"
    },
    "password": {
      "value": "PASSWORDGOESHERE"
    },
    "certificateThumbprint": {
      "value": "AF3143EB61D43F6727842115BB7F17BBCECAECAE"
    },
    "certificateName": {
      "value": "DefaultCertificateFor_yourASENameHere_InternalLoadBalancingASE"
    }
  }
}
```

填入 azuredeploy.parameters.json 檔案後，使用下列 Powershell 程式碼片段設定預設 SSL。 將檔案路徑變更為您電腦上 Resource Manager 範本檔案的位置。 記得提供您自己的 Resource Manager 部署名稱和資源群組名稱的值：

```powershell
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzureRmResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

每個 ASE 前端套用變更大約需要 40 分鐘。 例如，有一個預設大小的 ASE 使用兩個前端，則範本將需要大約一小時 20 分鐘的時間才能完成。 執行範本時，無法調整 ASE。  

範本完成之後，可以透過 HTTPS 存取 ILB ASE 上的應用程式。 此連線使用預設 SSL 憑證加以保護。 如果 ILB ASE 上的應用程式是使用應用程式名稱加上預設主機名稱的組合來定址，則會使用預設 SSL 憑證。 例如， https://mycustomapp.internal-contoso.com 會使用 **.internal-contoso.com* 的預設 SSL 憑證。

不過，就如同在公用多租用戶服務上執行的應用程式，開發人員可以為個別的應用程式設定自訂主機名稱。 他們也可以為個別的應用程式設定唯一 SNI SSL 憑證繫結。

## <a name="app-service-environment-v1"></a>App Service 環境 v1 ##
App Service Environment 有兩個版本：ASEv1 和 ASEv2。 前述資訊架構在 ASEv2 上。 本節說明 ASEv1 與 ASEv2 之間的差異。

在 ASEv1 中，要手動管理所有資源， 包括前端、背景工作角色和用於 IP 型 SSL 的 IP 位址。 首先，您必須將想要在其中裝載的背景工作角色集區相應放大，才能相應放大 App Service 方案。

ASEv1 使用與 ASEv2 不同的定價模式。 在 ASEv1 中，您需要支付每個配置的 vCPU。 包括用於前端或未裝載任何工作負載之背景工作角色的 vCPU。 在 ASEv1 中，ASE 的預設最大調整大小總計是 55 個主機， 包括背景工作角色與前端。 ASEv1 的其中一個優點，是可以部署在傳統虛擬網路和 Resource Manager虛擬網路中。 若要深入了解 ASEv1，請參閱 [App Service Environment v1 簡介][ASEv1Intro]。

若要使用 Resource Manager 範本建立 ASEv1，請參閱[使用 Resource Manager 範本建立 ILB ASE v1][ILBASEv1Template]。


<!--Links-->
[quickstartilbasecreate]: http://azure.microsoft.com/documentation/templates/201-web-app-asev2-ilb-create
[quickstartasev2create]: http://azure.microsoft.com/documentation/templates/201-web-app-asev2-create
[quickstartconfiguressl]: http://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl
[quickstartwebapponasev2create]: http://azure.microsoft.com/documentation/templates/201-web-app-asp-app-on-asev2-create
[examplebase64encoding]: http://powershellscripts.blogspot.com/2007/02/base64-encode-file.html 
[configuringDefaultSSLCertificate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl/
[Intro]: ./intro.md
[MakeExternalASE]: ./create-external-ase.md
[MakeASEfromTemplate]: ./create-from-template.md
[MakeILBASE]: ./create-ilb-ase.md
[ASENetwork]: ./network-info.md
[UsingASE]: ./using-an-ase.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/security-overview.md
[ConfigureASEv1]: app-service-web-configure-an-app-service-environment.md
[ASEv1Intro]: app-service-app-service-environment-intro.md
[mobileapps]: ../../app-service-mobile/app-service-mobile-value-prop.md
[Functions]: ../../azure-functions/index.yml
[Pricing]: http://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/resource-group-overview.md
[ConfigureSSL]: ../../app-service/web-sites-purchase-ssl-web-site.md
[Kudu]: http://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../application-gateway/application-gateway-web-application-firewall-overview.md
[ILBASEv1Template]: app-service-app-service-environment-create-ilb-ase-resourcemanager.md
