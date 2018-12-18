---
title: 在 Azure Stack 中部署安全的 Service Fabric 叢集 | Microsoft Docs
description: 了解如何在 Azure Stack 中部署安全的 Service Fabric 叢集
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 09/12/2018
ms.author: mattbriggs
ms.reviewer: shnatara
ms.openlocfilehash: d402b2bcd5187cbb6ece78d7e981068c279c1f75
ms.sourcegitcommit: 9eaf634d59f7369bec5a2e311806d4a149e9f425
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2018
ms.locfileid: "48804424"
---
# <a name="deploy-a-service-fabric-cluster-in-azure-stack"></a>在 Azure Stack 中部署 Service Fabric 叢集

使用 Azure Marketplace 中的 [Service Fabric 叢集] 項目，在 Azure Stack 中部署安全的 Service Fabric 叢集。 

如需有關使用 Service Fabric 的詳細資訊，請參閱 Azure 文件中的 [Azure Service Fabric 概觀](https://docs.microsoft.com/azure/service-fabric/service-fabric-overview)和 [Service Fabric 叢集安全性案例](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security)。

## <a name="prerequisites"></a>必要條件

部署 Service Fabric 叢集時需要下列各項：
1. **叢集憑證**  
   這是在部署 Service Fabric 時，您新增至 KeyVault 的 X.509 伺服器憑證。 
   - 此憑證上的 **CN** 必須符合您所建立之 Service Fabric 叢集的完整網域名稱 (FQDN)。 
   - 憑證格式必須是 PFX，因為需要公用與私密金鑰。 
   請參閱建立此伺服器端憑證的[需求](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security)。

    > [!NOTE]  
    > 您可以使用自我簽署的憑證代替 x.509 伺服器憑證進行測試。 自我簽署的憑證不需要符合叢集的 FQDN。

1.  **管理用戶端憑證** – 這是用戶端用來驗證 Service Fabric 叢集的憑證，其可自我簽署。 請參閱建立此用戶端憑證的[需求](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security)。

1.  **Azure Stack Marketplace 中必須可取得下列項目：**
     - **Windows Server 2016** – 範本使用 Windows Server 2016 映像來建立叢集。  
     - **客戶指令碼擴充功能** - 來自 Microsoft 的虛擬機器擴充功能。  
     - **PowerShell 所需階段組態**-來自 Microsoft 的虛擬機器擴充功能。


## <a name="add-a-secret-to-key-vault"></a>將祕密新增至 Key Vault
若要部署 Service Fabric 叢集時，您必須為 Service Fabric 叢集指定正確的 KeyVault「祕密識別碼」或 URL。 Azure Resource Manager 範本會採用 KeyVault 作為輸入，接著在 Service Fabric 叢集的安裝期間擷取叢集憑證。 

> [!IMPORTANT]  
> 您必須使用 PowerShell 將祕密新增至 KeyVault，以便搭配 Service Fabric 使用。 請勿使用入口網站。  

使用下列指令碼來建立 KeyVault，並且在其中新增「叢集憑證」。 (請參閱[必要條件](#prerequisites)。)執行指令碼之前，請檢閱範例指令碼及更新指定的參數，以符合您的環境。 此指令碼也會將您需要提供的值輸出至 Azure Resource Manager 範本。 

> [!TIP]  
> 必須先具備公用供應項目 (包括用於計算、網路、儲存體和 KeyVault 的服務)，指令碼才可能成功。 

  ```PowerShell
    function Get-ThumbprintFromPfx($PfxFilePath, $Password) 
        {
            return New-Object System.Security.Cryptography.X509Certificates.X509Certificate2($PfxFilePath, $Password)
        }
    
    function Publish-SecretToKeyVault ($PfxFilePath, $Password, $KeyVaultName)
       {
            $keyVaultSecretName = "ClusterCertificate"
            $certContentInBytes = [io.file]::ReadAllBytes($PfxFilePath)
            $pfxAsBase64EncodedString = [System.Convert]::ToBase64String($certContentInBytes)
    
            $jsonObject = ConvertTo-Json -Depth 10 ([pscustomobject]@{
                data     = $pfxAsBase64EncodedString
                dataType = 'pfx'
                password = $Password
            })
    
            $jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
            $jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)
            $secret = ConvertTo-SecureString -String $jsonEncoded -AsPlainText -Force
            $keyVaultSecret = Set-AzureKeyVaultSecret -VaultName $KeyVaultName -Name $keyVaultSecretName -SecretValue $secret
            
            $pfxCertObject = Get-ThumbprintFromPfx -PfxFilePath $PfxFilePath -Password $Password
    
            Write-Host "KeyVault id: " -ForegroundColor Green
            (Get-AzureRmKeyVault -VaultName $KeyVaultName).ResourceId
            
            Write-Host "Secret Id: " -ForegroundColor Green
            (Get-AzureKeyVaultSecret -VaultName $KeyVaultName -Name $keyVaultSecretName).id
    
            Write-Host "Cluster Certificate Thumbprint: " -ForegroundColor Green
            $pfxCertObject.Thumbprint
       }
    
    #========================== CHANGE THESE VALUES ===============================
    $armEndpoint = "https://management.local.azurestack.external"
    $tenantId = "your_tenant_ID"
    $location = "local"
    $clusterCertPfxPath = "Your_path_to_ClusterCert.pfx"
    $clusterCertPfxPassword = "Your_password_for_ClusterCert.pfx"
    #==============================================================================
    
    Add-AzureRmEnvironment -Name AzureStack -ARMEndpoint $armEndpoint
    Login-AzureRmAccount -Environment AzureStack -TenantId $tenantId
    
    $rgName = "sfvaultrg"
    Write-Host "Creating Resource Group..." -ForegroundColor Yellow
    New-AzureRmResourceGroup -Name $rgName -Location $location
    
    Write-Host "Creating Key Vault..." -ForegroundColor Yellow
    $Vault = New-AzureRmKeyVault -VaultName sfvault -ResourceGroupName $rgName -Location $location -EnabledForTemplateDeployment -EnabledForDeployment -EnabledForDiskEncryption
    
    Write-Host "Publishing certificate to Vault..." -ForegroundColor Yellow
    Publish-SecretToKeyVault -PfxFilePath $clusterCertPfxPath -Password $clusterCertPfxPassword -KeyVaultName $vault.VaultName
   ``` 


如需詳細資訊，請參閱[使用 PowerShell 在 Azure Stack 上管理 KeyVault](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-kv-manage-powershell)。

## <a name="deploy-the-marketplace-item"></a>部署 Marketplace 項目

1. 在使用者入口網站中，移至 [+ 建立資源]  >  [計算]  >  [Service Fabric 叢集]。 

   ![選取 Service Fabric 叢集](./media/azure-stack-solution-template-service-fabric-cluster/image2.png)

1. 針對每個頁面 (如「基本概念」)，填妥部署表單。 如果您不確定值為何，請使用預設值。 選取 [確定] 前進到下一個頁面：

   ![基本概念](media/azure-stack-solution-template-service-fabric-cluster/image3.png)

1. 在 [網路設定] 頁面上，您可以指定要對您的應用程式開啟的特定連接埠：

   ![網路設定](media/azure-stack-solution-template-service-fabric-cluster/image4.png)

1. 在 [安全性] 頁面上，新增您經由[建立 Azure KeyVault](#add-a-secret-to-key-vault) 和上傳祕密所得到的值。

   針對 [管理用戶端憑證指紋]，輸入 [管理用戶端憑證] 的指紋。 (請參閱[必要條件](#prerequisites)。)
   
   - 來源金鑰保存庫：指定指令碼結果中的整個 keyVault id 字串。 
   - 叢集憑證 URL：指定指令碼結果中 Secret Id 中的整個 URL。 
   - 叢集憑證指紋：指定指令碼結果中的 Cluster Certificate Thumbprint。
   - 管理用戶端憑證指紋：指定您在必要條件中建立的 [管理用戶端憑證指紋]。 

   ![指令碼輸出](media/azure-stack-solution-template-service-fabric-cluster/image5.png)

   ![安全性](media/azure-stack-solution-template-service-fabric-cluster/image6.png)

1. 完成精靈，然後選取 [建立] 來部署 Service Fabric 叢集。



## <a name="access-the-service-fabric-cluster"></a>存取 Service Fabric 叢集
您可以使用 Service Fabric Explorer 或 Service Fabric PowerShell 來存取 Service Fabric 叢集。


### <a name="use-service-fabric-explorer"></a>使用 Service Fabric Explorer
1.  驗證網頁瀏覽器是否可以存取您的管理用戶端憑證，而且可以向 Service Fabric 叢集進行驗證。  

    a. 開啟 Internet Explorer 並移至 [網際網路選項] > [內容] > [憑證]。
  
    b. 在 [憑證] 上，選取 [匯入] 啟動 [憑證匯入精靈]，然後按 [下一步]。 在 [要匯入的檔案] 頁面上，按一下 [瀏覽]，然後選取您提供給 Azure Resource Manager 範本的 [管理用戶端憑證]。
        
       > [!NOTE]  
       > 此憑證不是先前新增至 KeyVault 的叢集憑證。  

    c. 確定您已在 [檔案總管] 視窗的擴充下拉式清單中選取 [個人資訊交換]。  

       ![個人資訊交換](media/azure-stack-solution-template-service-fabric-cluster/image8.png)  

    d. 在 [憑證存放區] 頁面上，選取 [個人]，然後完成精靈。  
       ![憑證存放區](media/azure-stack-solution-template-service-fabric-cluster/image9.png)  
1. 若要尋找 Service Fabric 叢集的 FQDN：  

    a. 請移至與 Service Fabric 叢集相關聯的資源群組，並找出 [公用 IP 位址] 資源。 選取與公用 IP 位址相關聯的物件，以開啟 [公用 IP 位址] 刀鋒視窗。  

      ![公用 IP 位址](media/azure-stack-solution-template-service-fabric-cluster/image10.png)   

    b. 在 [公用 IP 位址] 刀鋒視窗上，FQDN 會顯示為 [DNS 名稱]。  

      ![DNS 名稱](media/azure-stack-solution-template-service-fabric-cluster/image11.png)  

1. 若要尋找 Service Fabric Explorer 和用戶端連線端點的 URL，請檢閱範本部署的結果。

1. 在瀏覽器中，前往 https://*FQDN*:19080。 使用步驟 2 中 Service Fabric 叢集的 FQDN 來取代 *FQDN*。   
   如果您已使用自我簽署的憑證，您會收到連線不安全的警告。 若要前往網站，請選取 [更多資訊]，然後 [前往網頁]。 

1. 若要向網站進行驗證，您必須選取要使用的憑證。 選取 [更多選擇]，挑選適當的憑證，然後按一下 [確定] 以連線到 Service Fabric Explorer。 

   ![驗證](media/azure-stack-solution-template-service-fabric-cluster/image14.png)



## <a name="use-service-fabric-powershell"></a>使用 Service Fabric PowerShell

1. 從 Azure Service Fabric 文件中的[在 Windows 上準備開發環境](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started#install-the-sdk-and-tools)，安裝 Microsoft Azure Service Fabric SDK。  

1. 完成安裝之後，設定系統環境變數，以確保可從 PowerShell 存取 Service Fabric Cmdlet。  
    
    a. 移至 [控制台] > [系統及安全性] > [系統]，然後選取 [進階系統設定]。  
    
      ![控制台](media/azure-stack-solution-template-service-fabric-cluster/image15.png) 

    b. 在 [系統屬性] 的 [進階] 索引標籤上，選取 [環境變數]。  

    c. 針對 [系統變數]，編輯 [路徑] 並確定 **C:\\Program Files\\Microsoft Service Fabric\\bin\\Fabric\\Fabric.Code** 位於環境變數的清單頂端。  

      ![環境變數清單](media/azure-stack-solution-template-service-fabric-cluster/image16.png)

1. 變更環境變數的順序之後，重新啟動 PowerShell，然後執行下列 PowerShell 指令碼來存取 Service Fabric 叢集：

   ````PowerShell  
    Connect-ServiceFabricCluster -ConnectionEndpoint "\[Service Fabric
    CLUSTER FQDN\]:19000" \`

    -X509Credential -ServerCertThumbprint
    761A0D17B030723A37AA2E08225CD7EA8BE9F86A \`

    -FindType FindByThumbprint -FindValue
    0272251171BA32CEC7938A65B8A6A553AA2D3283 \`

    -StoreLocation CurrentUser -StoreName My -Verbose
   ````
   
   > [!NOTE]  
   > 指令碼中的叢集名稱前面沒有 https://。 需要連接埠 19000。
 
