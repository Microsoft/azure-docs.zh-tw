---
title: 管理 Azure Service Fabric 叢集中的憑證 | Microsoft Docs
description: 說明如何在 Service Fabric 叢集新增新的憑證、變換憑證及移除憑證。
services: service-fabric
documentationcenter: .net
author: ChackDan
manager: timlt
editor: ''
ms.assetid: 91adc3d3-a4ca-46cf-ac5f-368fb6458d74
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/23/2018
ms.author: chackdan
ms.openlocfilehash: a1cfd68b526d8ce63fcfbc3b6e0eac84926fabaa
ms.sourcegitcommit: 30c7f9994cf6fcdfb580616ea8d6d251364c0cd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/18/2018
ms.locfileid: "42141505"
---
# <a name="add-or-remove-certificates-for-a-service-fabric-cluster-in-azure"></a>新增或移除 Azure 中 Service Fabric 叢集的憑證
建議您熟悉 Service Fabric 使用 X.509 憑證的方式，以及熟悉[叢集安全性案例](service-fabric-cluster-security.md)。 您必須瞭解什麼是叢集憑證及其用途，方可繼續進行後續作業。

Azure Service Fabric SDK 的預設憑證載入行為，是部署和使用到期日最久的已定義憑證；而不管其主要或次要設定定義為何。 退回使用傳統行為是進階動作 (不建議您這麼做)，而且需要將 Fabric.Code 設定內的 "UseSecondaryIfNever" 設定參數值設為 false。

當您在叢集建立期間設定憑證安全性時，除了用戶端憑證之外，Service Fabric 還可讓您指定兩個叢集憑證：主要與次要。 請參閱[透過入口網站建立 Azure 叢集](service-fabric-cluster-creation-via-portal.md)或[透過 Azure Resource Manager 建立 Azure 叢集](service-fabric-cluster-creation-via-arm.md)，以詳細了解如何在建立這些叢集時進行叢集設定。 如果您在建立時僅指定一個叢集憑證，該憑證就會作為主要憑證。 在叢集建立完成後，您可新增憑證做為次要憑證。

> [!NOTE]
> 針對安全叢集，您一律必須至少部署一個有效 (未撤銷或過期) 的叢集憑證 (主要或次要)，否則叢集將停止運作。 在所有有效憑證到達到期日的 90 天前，系統會針對節點產生警告追蹤與警告健康狀態事件。 目前並無 Service Fabric 針對此文章送出的電子郵件或其他任何通知。 
> 
> 

## <a name="add-a-secondary-cluster-certificate-using-the-portal"></a>使用入口網站來新增次要叢集憑證
您無法透過 Azure 入口網站使用 Azure PowerShell 新增次要叢集憑證。 本文件稍後會簡要說明此程序。

## <a name="remove-a-cluster-certificate-using-the-portal"></a>使用入口網站來移除叢集憑證
針對安全叢集，您一律必須至少有一個有效 (未撤銷或過期) 的憑證。 系統會使用部署了最久到期日的憑證，若將它移除，則叢集會停止運作；請務必只移除過期的憑證，或移除最快到期的未使用憑證。

若要移除未使用的叢集安全性憑證，請瀏覽至 [安全性] 區段，然後在未使用的憑證上，從快顯功能表中選取 [刪除] 選項。

如果您的目的是要移除標記為主要的憑證，則必須部署到期日比主要憑證還久的次要憑證，以啟用自動變換行為；再於自動變換完成後刪除主要憑證。

## <a name="add-a-secondary-certificate-using-resource-manager-powershell"></a>使用 Resource Manager Powershell 來新增次要憑證
> [!TIP]
> 現在有更好而且更容易的方式，可使用 [Add-AzureRmServiceFabricClusterCertificate](/powershell/module/azurerm.servicefabric/add-azurermservicefabricclustercertificate) Cmdlet 來新增次要憑證。 您不需要遵循本節中的其餘步驟。  此外，您也不需要原本在使用 [Add-AzureRmServiceFabricClusterCertificate](/powershell/module/azurerm.servicefabric/add-azurermservicefabricclustercertificate) Cmdlet 時用來建立及部署叢集的範本。

這些步驟是假設您已熟悉 Resource Manager 的運作方式，並已使用 Resource Manager 範本至少部署一個 Service Fabric 叢集，而且已讓您使用的範本將叢集設定妥當。 此外亦假設您可輕鬆自如地使用 JSON。

> [!NOTE]
> 如果您正在尋找可用來依循或作為起點的範例範本和參數，可從這個 [git 存放庫](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample)下載。 
> 
> 

### <a name="edit-your-resource-manager-template"></a>編輯您的 Resource Manager 範本

為了便於跟著操作，範例 5-VM-1-NodeTypes-Secure_Step2.JSON 包含我們將進行的所有編輯。 您可以從 [git 存放庫](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample)取得該範例。

**務必依照所有步驟操作**

1. 開啟您用來部署叢集的 Resource Manager 範本。 (如果您已從上述存放庫下載該範例，則請使用 5-VM-1-NodeTypes-Secure_Step1.JSON 來部署一個安全的叢集，然後開啟該範本)。

2. 將類型為 "string" 的**兩個新參數** "secCertificateThumbprint" 和 "secCertificateUrlValue" 新增到您範本的參數區段。 您可以複製下列程式碼片段並新增到範本中。 視您的範本來源而定，這些有可能已經定義好，如果是這樣，請移至下一個步驟。 
 
    ```json
       "secCertificateThumbprint": {
          "type": "string",
          "metadata": {
            "description": "Certificate Thumbprint"
          }
        },
        "secCertificateUrlValue": {
          "type": "string",
          "metadata": {
            "description": "Refers to the location URL in your key vault where the certificate was uploaded, it is should be in the format of https://<name of the vault>.vault.azure.net:443/secrets/<exact location>"
          }
        },
    
    ```

3. 對 **Microsoft.ServiceFabric/clusters** 資源進行變更 - 找出您範本中的 "Microsoft.ServiceFabric/clusters" 資源定義。 在該定義的屬性底下，您會發現「憑證」JSON 標籤，看起來應該會像以下 JSON 程式碼片段：
   
    ```JSON
          "properties": {
            "certificate": {
              "thumbprint": "[parameters('certificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
         }
    ``` 

    加入新標籤「thumbprintSecondary」，並為它提供值「[parameters('secCertificateThumbprint')]」。  

    資源定義現在看起來應該像下面這樣 (視您的範本來源而定，可能不會與下面的程式碼片段完全相同)。 

    ```JSON
          "properties": {
            "certificate": {
              "thumbprint": "[parameters('certificateThumbprint')]",
              "thumbprintSecondary": "[parameters('secCertificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
         }
    ``` 

    如果您想要「變換憑證」，請將新憑證指定為主要憑證，並將目前的主要憑證移轉為次要憑證。 這可讓您透過單一部署步驟，就將目前的主要憑證變換成新憑證。
    
    ```JSON
          "properties": {
            "certificate": {
              "thumbprint": "[parameters('secCertificateThumbprint')]",
              "thumbprintSecondary": "[parameters('certificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
         }
    ``` 

4. 對**所有** **Microsoft.Compute/virtualMachineScaleSets** 資源定義進行變更 - 找出 Microsoft.Compute/virtualMachineScaleSets 資源定義。 捲動到 "virtualMachineProfile" 底下的 "publisher": "Microsoft.Azure.ServiceFabric"。

    在 Service Fabric 發行者設定中，您應該會看到像這樣的畫面。
    
    ![Json_Pub_Setting1][Json_Pub_Setting1]
    
    請將新憑證項目新增到其中
    
    ```json
                   "certificateSecondary": {
                        "thumbprint": "[parameters('secCertificateThumbprint')]",
                        "x509StoreName": "[parameters('certificateStoreValue')]"
                        }
                      },
    
    ```

    屬性現在應該看起來像這樣
    
    ![Json_Pub_Setting2][Json_Pub_Setting2]
    
    如果您想要「變換憑證」，請將新憑證指定為主要憑證，並將目前的主要憑證移轉為次要憑證。 這可讓您在單一部署步驟中，將目前的憑證變換為新憑證。     

    ```json
                   "certificate": {
                       "thumbprint": "[parameters('secCertificateThumbprint')]",
                       "x509StoreName": "[parameters('certificateStoreValue')]"
                         },
                   "certificateSecondary": {
                        "thumbprint": "[parameters('certificateThumbprint')]",
                        "x509StoreName": "[parameters('certificateStoreValue')]"
                        }
                      },
    ```

    屬性現在應該看起來像這樣    
    ![Json_Pub_Setting3][Json_Pub_Setting3]

5. 對**所有** **Microsoft.Compute/virtualMachineScaleSets** 資源定義進行變更 - 找出 Microsoft.Compute/virtualMachineScaleSets 資源定義。 捲動到 "OSProfile" 底下的 "vaultCertificates":。 您應該會看到類似下面的畫面。

    ![Json_Pub_Setting4][Json_Pub_Setting4]
    
    請將 secCertificateUrlValue 新增到其中。 請使用下列程式碼片段：
    
    ```json
                      {
                        "certificateStore": "[parameters('certificateStoreValue')]",
                        "certificateUrl": "[parameters('secCertificateUrlValue')]"
                      }
    
    ```
    產生的 Json 現在應該看起來像這樣。
    ![Json_Pub_Setting5][Json_Pub_Setting5]


> [!NOTE]
> 請確定您已針對範本中的所有 Nodetypes/Microsoft.Compute/virtualMachineScaleSets 資源定義重複步驟 4 和 5。 如果您遺漏其中一個，憑證就不會安裝在該虛擬機器擴展集上，而您的叢集中將會有無法預測的結果，包括叢集停止運作 (如果您最終沒有任何可供叢集用於安全性的有效憑證)。 因此，在進一步進行之前，請先仔細檢查。
> 
> 

### <a name="edit-your-template-file-to-reflect-the-new-parameters-you-added-above"></a>編輯您的範本檔案，以反映先前加入的新參數
如果您是依循來自 [git 儲存機制](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/Cert%20Rollover%20Sample)的範例進行操作，則可以開始在範例 5-VM-1-NodeTypes-Secure.paramters_Step2.JSON 中進行變更 

請編輯您的 Resource Manager 範本參數檔，新增 secCertificateThumbprint 和 secCertificateUrlValue 的兩個新參數。 

```JSON
    "secCertificateThumbprint": {
      "value": "thumbprint value"
    },
    "secCertificateUrlValue": {
      "value": "Refers to the location URL in your key vault where the certificate was uploaded, it is should be in the format of https://<name of the vault>.vault.azure.net:443/secrets/<exact location>"
     },

```

### <a name="deploy-the-template-to-azure"></a>將範本部署到 Azure

- 您現在已可將範本部署至 Azure。 開啟 Azure PS 版本 1+ 命令提示字元。
- 登入您的 Azure 帳戶，並選取特定的 Azure 訂用帳戶。 對於擁有多個 Azure 訂用帳戶存取權的使用者而言，這是一個重要步驟。

```powershell
Connect-AzureRmAccount
Select-AzureRmSubscription -SubscriptionId <Subcription ID> 

```

部署範本前先進行測試。 使用您目前在其中部署叢集的同一個資源群組。

```powershell
Test-AzureRmResourceGroupDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>

```

將範本部署至您的資源群組。 使用您目前在其中部署叢集的同一個資源群組。 執行 New-AzureRmResourceGroupDeployment 命令。 您無須指定模式，因為預設值為 **增量**。

> [!NOTE]
> 若您將 [模式] 設為 [完整]，您可能會無意間刪除不在您範本中的資源。 因此請勿在此案例中使用該模式。
> 
> 

```powershell
New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName <Resource Group that your cluster is currently deployed to> -TemplateFile <PathToTemplate>
```

以下是已填入資料的相同 Powershell 範例。

```powershell
$ResouceGroup2 = "chackosecure5"
$TemplateFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure_Step2.json"
$TemplateParmFile = "C:\GitHub\Service-Fabric\ARM Templates\Cert Rollover Sample\5-VM-1-NodeTypes-Secure.parameters_Step2.json"

New-AzureRmResourceGroupDeployment -ResourceGroupName $ResouceGroup2 -TemplateParameterFile $TemplateParmFile -TemplateUri $TemplateFile -clusterName $ResouceGroup2

```

部署完成之後，使用新的憑證連線至叢集，並執行一些查詢。 若您可執行動作。 接著，您可以刪除舊憑證。 

若您是使用自我簽署憑證，請務必將它們匯入至本機 TrustedPeople 憑證存放區。

```powershell
######## Set up the certs on your local box
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\TrustedPeople -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My -FilePath c:\Mycertificates\chackdanTestCertificate9.pfx -Password (ConvertTo-SecureString -String abcd123 -AsPlainText -Force)

```
以下提供連線至安全叢集的命令的快速參考 

```powershell
$ClusterName= "chackosecure5.westus.cloudapp.azure.com:19000"
$CertThumbprint= "70EF5E22ADB649799DA3C8B6A6BF7SD1D630F8F3" 

Connect-serviceFabricCluster -ConnectionEndpoint $ClusterName -KeepAliveIntervalInSec 10 `
    -X509Credential `
    -ServerCertThumbprint $CertThumbprint  `
    -FindType FindByThumbprint `
    -FindValue $CertThumbprint `
    -StoreLocation CurrentUser `
    -StoreName My
```
以下提供取得叢集健康情況的命令的快速參考

```powershell
Get-ServiceFabricClusterHealth 
```

## <a name="deploying-application-certificates-to-the-cluster"></a>將應用程式憑證部署到叢集。

您可以使用與上述步驟 5 中所述的相同步驟，將憑證從金鑰保存庫部署到節點。 您只需定義和使用不同的參數即可。


## <a name="adding-or-removing-client-certificates"></a>新增或移除用戶端憑證

除了叢集憑證之外，您還可以新增用戶端憑證以在 Service Fabric 叢集上執行管理作業。

您可以新增兩種用戶端憑證 - 系統管理員或唯讀。 接著這些憑證便可用來控制對叢集上系統管理員作業和查詢作業的存取。 叢集憑證預設會新增到允許的系統管理員憑證清單中。

您可以指定任何數目的用戶端憑證。 每次進行新增/刪除時，都會導致更新 Service Fabric 叢集的組態


### <a name="adding-client-certificates---admin-or-read-only-via-portal"></a>透過入口網站新增用戶端憑證 - 系統管理員或唯讀

1. 瀏覽至 [安全性] 區段，然後選取 [安全性] 區段頂端的 [+ 驗證] 按鈕。
2. 在 [新增驗證] 區段上，選擇 [驗證類型] - [唯讀用戶端] 或 [系統管理員用戶端]
3. 現在選擇 [驗證方法]。 這會對 Service Fabric 指出其是否應該使用主體名稱或指紋來查詢此憑證。 一般而言，使用主體名稱授權方法不是很好的安全性做法。 

![新增用戶端憑證][Add_Client_Cert]

### <a name="deletion-of-client-certificates---admin-or-read-only-using-the-portal"></a>使用入口網站來刪除用戶端憑證 - 系統管理員或唯讀

若要移除次要憑證，使其不用於叢集安全性，請瀏覽至 [安全性] 區段，然後從特定憑證上的操作功能表中選取 [刪除] 選項。

## <a name="next-steps"></a>後續步驟
如需有關叢集管理的詳細資訊，請參閱下列文件︰

* [Service Fabric 叢集升級程序與您的期望](service-fabric-cluster-upgrade.md)
* [設定用戶端的角色型存取](service-fabric-cluster-security-roles.md)

<!--Image references-->
[Add_Client_Cert]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_13.PNG
[Json_Pub_Setting1]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_14.PNG
[Json_Pub_Setting2]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_15.PNG
[Json_Pub_Setting3]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_16.PNG
[Json_Pub_Setting4]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_17.PNG
[Json_Pub_Setting5]: ./media/service-fabric-cluster-security-update-certs-azure/SecurityConfigurations_18.PNG


