---
title: 如何使用 Azure Resource Manager 範本建立 ILB ASE | Microsoft Docs
description: 了解如何使用 Azure Resource Manager 範本建立內部負載平衡器 ASE
services: app-service
documentationcenter: ''
author: stefsch
manager: nirma
editor: ''
ms.assetid: 091decb6-b0de-42a1-9f2f-c18d9b2e67df
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: stefsch
ms.openlocfilehash: 20531cb301cad23fbadb617bdf33e710a4481be4
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44050029"
---
# <a name="how-to-create-an-ilb-ase-using-azure-resource-manager-templates"></a>如何使用 Azure Resource Manager 範本建立 ILB ASE範本建立 ILB ASE

> [!NOTE] 
> 這篇文章是關於 App Service 環境 v1。 有較新版本的 App Service 環境，更易於使用，並且可以在功能更強大的基礎結構上執行。 若要深入了解新版本，請從 [App Service 環境簡介](intro.md)開始。
>

## <a name="overview"></a>概觀
使用虛擬網路內部位址 (而不是公用 VIP) 可以建立 App Service 環境。  此內部位址是由稱為內部負載平衡器 (ILB) 的 Azure 元件提供。  使用 Azure 入口網站可以建立 ILB ASE。  也可以透過 Azure Resource Manager 範本使用自動化建立。  本文逐步解說使用 Azure Resource Manager 範本建立 ILB ASE 所需的步驟和語法。

自動建立 ILB ASE 涉及三個步驟︰

1. 首先會使用內部負載平衡器位址 (而不是公用 VIP)，在虛擬網路中建立基底 ASE。  在此步驟中，根網域名稱會指派給 ILB ASE。
2. 一旦建立 ILB ASE，就會上傳 SSL 憑證。  
3. 上傳的 SSL 憑證會明確指派給 ILB ASE 做為其「預設」SSL 憑證。  如果應用程式是使用指派給 ASE 的一般根網域 (例如 https://someapp.mycustomrootcomain.com) 來定址，此 SSL 憑證將使用於 ILB ASE 上應用程式的 SSL 流量

## <a name="creating-the-base-ilb-ase"></a>建立基底 ILB ASE
在 GitHub 上 ([這裡][quickstartilbasecreate]) 可以取得範例 Azure Resource Manager 範本及其相關聯的參數檔案。

「azuredeploy.parameters.json」  檔案中的大部分參數通用於建立 ILB ASE 以及繫結至公用 VIP 的 ASE。  建立 ILB ASE 時，以下清單會呼叫特殊附註或唯一的參數︰

* interalLoadBalancingMode︰在大多數情況下，此屬性設定為 3，這表示連接埠 80/443 上的 HTTP/HTTPS 流量，以及 ASE 上的 FTP 服務所接聽的控制項/資料通道連接埠將會繫結至 ILB 配置的虛擬網路內部位址。  如果此屬性改為設定為 2，則只有 FTP 服務相關的連接埠 (控制和資料通道) 會繫結至 ILB 位址，而 HTTP/HTTPS 流量將保留在公用 VIP 上。
* dnsSuffix︰這個參數會定義要指派給 ASE 的預設根網域。  在 Azure App Service 的公用種變化中，所有 Web 應用程式的預設根網域皆為 azurewebsites.net 。  不過，由於 ILB ASE 位於客戶虛擬網路的內部，所以不適合使用公用服務的預設根網域。  相反地，ILB ASE 應具有適合在公司的內部虛擬網路內使用的預設根網域。  例如，假定的 Contoso Corporation 可能會將 internal-contoso.com  的預設根網域用於只能在 Contoso 虛擬網路內解析和存取的應用程式。 
* ipSslAddressCount︰在 *azuredeploy.json* 檔案中，這個參數的值會自動預設為 0，因為 ILB ASE 只有單一 ILB 位址。  ILB ASE 沒有明確的 IP-SSL 位址，因此 ILB ASE 的 IP-SSL 位址集區必須設為零，否則會發生佈建錯誤。 

一旦針對 ILB ASE 填入 azuredeploy.parameters.json  檔案，就可以使用下列 Powershell 程式碼片段建立 ILB ASE。  變更檔案 PATH，以符合 Azure Resource Manager 範本檔案位於您電腦上的位置。  也請記得提供您自己的 Azure Resource Manager 部署名稱和資源群組名稱的值。

    $templatePath="PATH\azuredeploy.json"
    $parameterPath="PATH\azuredeploy.parameters.json"

    New-AzureRmResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath

提交 Azure Resource Manager 範本之後，需要數小時的時間才能建立 ILB ASE。  建立完成後，ILB ASE 會顯示在觸發部署之訂用帳戶的 App Service 環境清單的入口網站 UX 中。

## <a name="uploading-and-configuring-the-default-ssl-certificate"></a>上傳和設定「預設」SSL 憑證
建立 ILB ASE 後， SSL 憑證應與 ASE 相關聯，做為用來建立應用程式的 SSL 連線的「預設」SSL 憑證。  繼續討論假定的 Contoso Corporation 範例，如果 ASE 的預設 DNS 尾碼是 *internal-contoso.com*，則連接到 *https://some-random-app.internal-contoso.com* 需要有對 **.internal-contoso.com* 有效的 SSL 憑證。 

有各種不同的方式可取得有效的 SSL 憑證，包括內部 CA、向外部簽發者購買憑證，以及使用自我簽署的憑證。  無論 SSL 憑證的來源，都需要正確設定下列憑證屬性︰

* *主體*︰  此屬性必須設為 **.your-root-domain-here.com*
* *主體替代名稱*︰  此屬性必須同時包含 **.your-root-domain-here.com* 和 **.scm.your-root-domain-here.com*。  第二個項目的原因是將使用 *your-app-name.scm.your-root-domain-here.com*  形式的位址，進行與每個應用程式相關聯的 SCM/Kudu 網站的 SSL 連線。

備妥有效的 SSL 憑證，還需要兩個額外的準備步驟。  SSL 憑證必須能夠轉換/另存為 .pfx 檔案。  請記住，.pfx 檔案必須包含所有中繼和根憑證，而且也必須使用密碼保護。

然後必須將結果產生的.pfx 檔案轉換成 base64 字串，因為會使用 Azure Resource Manager 範本上載 SSL 憑證。  因為 Azure Resource Manager 範本是文字檔案，所以必須將 .pfx 檔案轉換成 base64 字串，才能納入為範本的參數。

下列 Powershell 程式碼片段顯示產生自我簽署憑證、將憑證匯出為 .pfx 檔案、將 .pfx 檔案轉換成 base64 編碼的字串，然後將 base64 編碼字串儲存至個別檔案的範例。  Base64 編碼的 Powershell 程式碼改寫自 [Powershell 指令碼部落格][examplebase64encoding]。

    $certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

    $certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
    $password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

    $fileName = "exportedcert.pfx"
    Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password     

    $fileContentBytes = get-content -encoding byte $fileName
    $fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
    $fileContentEncoded | set-content ($fileName + ".b64")

成功產生 SSL 憑證並轉換成 base64 編碼字串後，GitHub 上的範例 Azure Resource Manager 範本即可用於[設定預設 SSL 憑證][configuringDefaultSSLCertificate]。

「azuredeploy.parameters.json」  檔案中的參數如下所列︰

* appServiceEnvironmentName︰要設定之 ILB ASE 的名稱。
* existingAseLocation︰包含 ILB ASE 部署所在的 Azure 區域的文字字串。  例如 "South Central US"。
* pfxBlobString：.pfx 檔案的 based64 編碼字串表示法。  使用稍早所示的程式碼片段，您會複製 "exportedcert.pfx.b64" 中包含的字串並貼入做為 pfxBlobString 屬性的值。
* password：用來保護 .pfx 檔案的密碼。
* certificateThumbprint︰憑證的指紋。  如果您從 Powershell 擷取此值 (例如先前程式碼片段中的 $certificate.Thumbprint  )，您可以使用現況值。  不過，如果您從 Windows 憑證對話方塊複製此值，請記得去除多餘的空格。  CertificateThumbprint 應如下所示︰AF3143EB61D43F6727842115BB7F17BBCECAECAE
* certificateName︰您自己選擇的好記字串識別碼，可用來識別憑證。  此名稱做為 Microsoft.Web/certificates  實體 (表示 SSL 憑證) 的唯一 Azure Resource Manager 識別碼的一部分。  名稱**必須**以下列尾碼結尾︰\_yourASENameHere_InternalLoadBalancingASE。  入口網站會使用這個尾碼做為憑證要用於保護啟用 ILB 之 ASE 的指示器。

「azuredeploy.parameters.json」  的縮寫範例如下所示︰

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

一旦填入「azuredeploy.parameters.json」  檔案，就可以使用下列 Powershell 程式碼片段設定 SSL 憑證。  變更檔案 PATH，以符合 Azure Resource Manager 範本檔案位於您電腦上的位置。  也請記得提供您自己的 Azure Resource Manager 部署名稱和資源群組名稱的值。

    $templatePath="PATH\azuredeploy.json"
    $parameterPath="PATH\azuredeploy.parameters.json"

    New-AzureRmResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath

提交 Azure Resource Manager 範本之後，每個 ASE 前端需要大約 40 分鐘的時間套用變更。  例如，有一個預設大小的 ASE 使用兩個前端，則範本將需要大約一小時 20 分鐘的時間才能完成。  執行範本時，將無法調整 ASE。  

完成範本後，可透過 HTTPS 存取 ILB ASE 上的應用程式，並使用預設 SSL 憑證保護連線。  如果 ILB ASE 上的應用程式是使用應用程式名稱加上預設主機名稱的組合來定址，則會使用預設 SSL 憑證。  例如，*https://mycustomapp.internal-contoso.com* 會使用 **.internal-contoso.com* 的預設 SSL 憑證。

不過，就如同在公用多租用戶服務上執行的應用程式，開發人員也可以為個別的應用程式設定自訂主機名稱，然後為個別的應用程式設定唯一的 SNI SSL 憑證繫結。  

## <a name="getting-started"></a>開始使用
若要開始使用 App Service 環境，請參閱 [App Service 環境簡介](app-service-app-service-environment-intro.md)

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[quickstartilbasecreate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-create/
[examplebase64encoding]: http://powershellscripts.blogspot.com/2007/02/base64-encode-file.html 
[configuringDefaultSSLCertificate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl/ 

