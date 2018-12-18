---
title: 在 Azure 中建立 Windows Service Fabric 叢集 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 PowerShell，將 Windows Service Fabric 叢集部署到 Azure 虛擬網路和網路安全性群組。
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: ''
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: tutorial
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 09/27/2018
ms.author: ryanwi
ms.custom: mvc
ms.openlocfilehash: 2f503a534f79440e6e6c572b7fb29ce3048ee7bc
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48831564"
---
# <a name="tutorial-deploy-a-service-fabric-windows-cluster-into-an-azure-virtual-network"></a>教學課程：將安全的 Service Fabric Windows 叢集部署到 Azure 虛擬網路

本教學課程是一個系列的第一部分。 您將會了解如何使用 PowerShell 和範本將執行 Windows 的 Service Fabric 叢集部署到 [Azure 虛擬網路 (VNET)](../virtual-network/virtual-networks-overview.md) 和[網路安全性群組](../virtual-network/virtual-networks-nsg.md)中。 完成時，您會有在您可以部署應用程式的雲端中執行的叢集。  若要使用 Azure CLI 建立 Linux 叢集，請參閱[在 Azure 上建立安全的 Linux 叢集](service-fabric-tutorial-create-vnet-and-linux-cluster.md)。

此教學課程說明的是生產環境案例。  如果您想要快速建立測試用的小型叢集，請參閱[建立三個節點的測試叢集](./scripts/service-fabric-powershell-create-test-cluster.md)。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用 PowerShell 在 Azure 中建立 VNET
> * 建立金鑰保存庫並上傳憑證
> * 在 Azure PowerShell 中建立安全的 Service Fabric 叢集
> * 使用 X.509 憑證保護叢集
> * 使用 PowerShell 連線到叢集
> * 刪除叢集

在本教學課程系列中，您將了解如何：
> [!div class="checklist"]
> * 在 Azure 上建立安全叢集
> * [將叢集相應縮小或相應放大](service-fabric-tutorial-scale-cluster.md)
> * [升級叢集的執行階段](service-fabric-tutorial-upgrade-cluster.md)
> * [刪除叢集](service-fabric-tutorial-delete-cluster.md)

## <a name="prerequisites"></a>必要條件

開始進行本教學課程之前：

* 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* 安裝 [Service Fabric SDK 和 PowerShell 模組](service-fabric-get-started.md)
* 安裝 [Azure PowerShell 模組 4.1 版或更新版本](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)

下列程序會建立五個節點的 Service Fabric 叢集。 若要計算在 Azure 中執行 Service Fabric 叢集產生的成本，請使用 [Azure 價格計算機](https://azure.microsoft.com/pricing/calculator/)。

## <a name="key-concepts"></a>重要概念

[Service Fabric 叢集](service-fabric-deploy-anywhere.md)是一組由網路連接的虛擬或實體機器，可用來將您的微服務部署到其中並進行管理。 叢集可擴充至數千部機器。 隸屬於叢集的機器或 VM 即稱為節點。 需為每個節點指派節點名稱 (字串)。 節點具有各種特性，如 placement 屬性。

節點類型定義叢集中一組虛擬機器的大小、數目和屬性。 每個已定義的節點類型會設定為[虛擬機器擴展集](/azure/virtual-machine-scale-sets/)，這是一個 Azure 計算資源，可以用來將一組虛擬機器當做一個集合來部署及管理。 然後每個節點類型可以獨立相應增加或相應減少，可以開啟不同組的連接埠，並可以有不同的容量度量。 節點類型是用來定義一組叢集節點的角色，例如「前端」或「後端」。  您的叢集可以有多個節點類型，但主要節點類型必須至少有五個 VM 供生產環境叢集使用 (或至少有三個 VM 供測試叢集使用)。  [Service Fabric 系統服務](service-fabric-technical-overview.md#system-services)是放置在主要節點類型的節點上。

叢集會受到叢集憑證的保護。 叢集憑證是用來保護節點對節點通訊，並向管理用戶端驗證叢集管理端點的 X.509 憑證。  該憑證也會為 HTTPS 管理 API 及透過 HTTPS 使用的 Service Fabric Explorer 提供 SSL。 自我簽署憑證可用於測試叢集。  對於生產叢集，請使用憑證授權單位 (CA) 提供的憑證作為叢集憑證。

叢集憑證必須︰

* 包含私密金鑰。
* 是為了進行金鑰交換而建立，且可匯出成個人資訊交換 (.pfx) 檔案。
* 有與您用來存取 Service Fabric 叢集的網域相符的主體名稱。 必須如此符合，才能為叢集的 HTTPS 管理端點和 Service Fabric Explorer 提供 SSL。 您無法從憑證授權單位 (CA) 取得 .cloudapp.azure.com 網域的 SSL 憑證。 您必須為您的叢集取得自訂網域名稱。 當您向 CA 要求憑證時，憑證的主體名稱必須與用於您叢集的自訂網域名稱相符。

Azure 金鑰保存庫可用來管理 Azure 中 Service Fabric 叢集的憑證。  在 Azure 中部署叢集時，負責建立 Service Fabric 叢集的 Azure 資源提供者會從金鑰保存庫提取憑證，並將它們安裝在叢集 VM 上。

本教學課程會顯示單一節點類型中五個節點組成的叢集。 不過，對於任何生產環境叢集部署，[容量規劃](service-fabric-cluster-capacity.md)都是一個很重要的步驟。 以下是一些您在該程序中必須考量的事情。

* 您的叢集所需的節點數目和節點類型
* 每個節點類型的屬性 (例如大小、主要、網際網路面向、VM 數目等)
* 叢集的可靠性和持久性的特性

## <a name="download-and-explore-the-template"></a>下載並瀏覽範本

下載下列 Resource Manager 範本檔案：

* [azuredeploy.json][template]
* [azuredeploy.parameters.json][parameters]

此範本會將一個由五部部虛擬機器組成並屬於單一節點類型的安全叢集部署到虛擬網路和網路安全性群組中。  您可以在 [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates) 上找到其他範例範本。  [azuredeploy.json][template] 會部署多項資源，包括下列各項。

### <a name="service-fabric-cluster"></a>Service Fabric 叢集

在 **Microsoft.ServiceFabric/clusters** 資源中，Windows 叢集會以下列特性設定：

* 單一節點類型
* 屬於主要節點類型的五個節點 (可在範本參數中設定)
* 作業系統：Windows Server 2016 Datacenter with Containers (可在範本參數中設定)
* 受保護的憑證 (可在範本參數中設定)
* 啟用[反向 Proxy](service-fabric-reverseproxy.md)
* 啟用 [DNS 服務](service-fabric-dnsservice.md)
* Bronze 的[耐久性層級](service-fabric-cluster-capacity.md#the-durability-characteristics-of-the-cluster) (可在範本參數中設定)
* Silver 的[可靠性層級](service-fabric-cluster-capacity.md#the-reliability-characteristics-of-the-cluster) (可在範本參數中設定)
* 用戶端連線端點：19000 (可在範本參數中設定)
* HTTP 閘道端點：19080 (可在範本參數中設定)

### <a name="azure-load-balancer"></a>Azure Load Balancer

在 **Microsoft.Network/loadBalancers** 資源中，會為下列連接埠設定負載平衡器，並進行探查和規則的設定：

* 用戶端連線端點：19000
* HTTP 閘道端點：19080
* 應用程式連接埠：80
* 應用程式連接埠：443
* Service Fabric 反向 Proxy：19081

如果需要其他應用程式連接埠，則您必須調整 **Microsoft.Network/loadBalancers** 資源和 **Microsoft.Network/networkSecurityGroups** 資源，以允許流量進入。

### <a name="virtual-network-subnet-and-network-security-group"></a>虛擬網路、子網路和網路安全性群組

虛擬網路、子網路和網路安全性群組的名稱會在範本參數中宣告。  虛擬網路和子網路的位址空間也會在範本參數中宣告，並設定於 **Microsoft.Network/virtualNetworks** 資源中：

* 虛擬網路位址空間：172.16.0.0/20
* Service Fabric 子網路位址空間：172.16.2.0/23

在 **Microsoft.Network/networkSecurityGroups** 資源中會啟用下列輸入流量規則。 您可以藉由變更範本變數來變更連接埠值。

* ClientConnectionEndpoint (TCP)：19000
* HttpGatewayEndpoint (HTTP/TCP)：19080
* SMB：445
* Internodecommunication - 1025、1026、1027
* 暫時連接埠範圍 – 49152 到 65534 (至少需要 256 個連接埠)
* 應用程式使用的連接埠：80 和 443
* 應用程式連接埠範圍 – 49152 到 65534 (用於服務之間的通訊，但不會在負載平衡器上開啟)
* 封鎖所有其他連接埠

如果需要其他應用程式連接埠，則您必須調整 **Microsoft.Network/loadBalancers** 資源和 **Microsoft.Network/networkSecurityGroups** 資源，以允許流量進入。

## <a name="set-template-parameters"></a>設定範本參數

[azuredeploy.parameters.json][parameters] 參數檔案會宣告多個用來部署叢集和相關聯資源的值。 您可能需要為自己的部署修改某些參數：

|參數|範例值|注意|
|---|---||
|adminUserName|vmadmin| 叢集虛擬機器的管理員使用者名稱。[虛擬機器的使用者名稱需求](https://docs.microsoft.com/azure/virtual-machines/windows/faq#what-are-the-username-requirements-when-creating-a-vm) |
|adminPassword|Password#1234| 叢集 VM 的系統管理員密碼。 [虛擬機器的密碼需求](https://docs.microsoft.com/azure/virtual-machines/windows/faq#what-are-the-password-requirements-when-creating-a-vm)|
|clusterName|mysfcluster123| 叢集的名稱。 只能包含字母和數字。 長度可介於 3 到 23 個字元之間。|
|location|southcentralus| 叢集的位置。 |
|certificateThumbprint|| <p>如果建立自我簽署憑證或提供憑證檔案，則值應該空白。</p><p>若要使用先前上傳至金鑰保存庫的現有憑證，請填入憑證 SHA1 指紋值。 例如 "6190390162C988701DB5676EB81083EA608DCCF3"</p>. |
|certificateUrlValue|| <p>如果建立自我簽署憑證或提供憑證檔案，則值應該空白。 </p><p>若要使用先前上傳至金鑰保存庫的現有憑證，請填入憑證 URL。 例如，"https://mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346"。</p>|
|sourceVaultValue||<p>如果建立自我簽署憑證或提供憑證檔案，則值應該空白。</p><p>若要使用先前上傳至金鑰保存庫的現有憑證，請填入來源保存庫值。 例如 "/subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT"。</p>|

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>部署虛擬網路和叢集

接下來，請設定網路拓撲並部署 Service Fabric 叢集。 [azuredeploy.json][template] Resource Manager 範本會建立虛擬網路 (VNET)，同時也會建立適用於 Service Fabric 的子網路與網路安全性群組 (NSG)。 範本也會部署啟用憑證安全性的叢集。  對於生產叢集，請使用憑證授權單位 (CA) 提供的憑證作為叢集憑證。 自我簽署憑證可用來保護測試叢集。

### <a name="create-a-cluster-using-an-existing-certificate"></a>使用現有的憑證建立叢集

下列指令碼會使用 [New-AzureRmServiceFabricCluster](/powershell/module/azurerm.servicefabric/New-AzureRmServiceFabricCluster) Cmdlet 和範本在 Azure 中部署新的叢集。 此 Cmdlet 也會在 Azure 中建立新的金鑰保存庫，並上傳您的憑證。

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$clustername = "mysfcluster123"  # must match the clusterName parameter in the template
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# sign in to your Azure account and select your subscription
Connect-AzureRmAccount
Get-AzureRmSubscription
Set-AzureRmContext -SubscriptionId <guid>

# Create a new resource group for your deployment and give it a name and a location.
New-AzureRmResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzureRmServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-KeyVaultName $vaultname -KeyVaultResouceGroupName $vaultgroupname -CertificateFile $certpath
```

### <a name="create-a-cluster-using-a-new-self-signed-certificate"></a>使用新的自我簽署憑證建立叢集

下列指令碼會使用 [New-AzureRmServiceFabricCluster](/powershell/module/azurerm.servicefabric/New-AzureRmServiceFabricCluster) Cmdlet 和範本在 Azure 中部署新的叢集。 此 Cmdlet 也會在 Azure 中建立新的金鑰保存庫、將新的自我簽署憑證新增至金鑰保存庫，並將憑證檔案下載至本機。

```powershell
# Variables.
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"  # must match the location parameter in the template
$templatepath="C:\temp\cluster"

$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
$certfolder="c:\mycertificates\"
$clustername = "mysfcluster123"
$vaultname = "clusterkeyvault123"
$vaultgroupname="clusterkeyvaultgroup123"
$subname="$clustername.$clusterloc.cloudapp.azure.com"

# sign in to your Azure account and select your subscription
Connect-AzureRmAccount
Get-AzureRmSubscription
Set-AzureRmContext -SubscriptionId <guid>

# Create a new resource group for your deployment and give it a name and a location.
New-AzureRmResourceGroup -Name $groupname -Location $clusterloc

# Create the Service Fabric cluster.
New-AzureRmServiceFabricCluster  -ResourceGroupName $groupname -TemplateFile "$templatepath\azuredeploy.json" `
-ParameterFile "$templatepath\azuredeploy.parameters.json" -CertificatePassword $certpwd `
-CertificateOutputFolder $certfolder -KeyVaultName $vaultname -KeyVaultResouceGroupName $vaultgroupname -CertificateSubjectName $subname

```

## <a name="connect-to-the-secure-cluster"></a>連線到安全的叢集

使用隨 Service Fabric SDK 一起安裝的 Service Fabric PowerShell 模組連線到叢集。  首先，將憑證安裝到您的電腦上目前使用者的個人 (My) 存放區。  執行下列 PowerShell 命令：

```powershell
$certpwd="q6D7nN%6ck@6" | ConvertTo-SecureString -AsPlainText -Force
Import-PfxCertificate -Exportable -CertStoreLocation Cert:\CurrentUser\My `
        -FilePath C:\mycertificates\mysfcluster20170531104310.pfx `
        -Password $certpwd
```

您現在即可連線到您安全的叢集。

**Service Fabric** PowerShell 模組提供許多 Cmdlet 來管理 Service Fabric 叢集、應用程式和服務。  使用 [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster) Cmdlet 連接到安全的叢集。 在上一個步驟的輸出中找到憑證 SHA1 指紋和連線端點詳細資料。

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint mysfcluster123.southcentralus.cloudapp.azure.com:19000 `
          -KeepAliveIntervalInSec 10 `
          -X509Credential -ServerCertThumbprint C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -FindType FindByThumbprint -FindValue C4C1E541AD512B8065280292A8BA6079C3F26F10 `
          -StoreLocation CurrentUser -StoreName My
```

使用 [Get-ServiceFabricClusterHealth](/powershell/module/servicefabric/get-servicefabricclusterhealth) Cmdlet 檢查已連線，而且叢集狀況良好。

```powershell
Get-ServiceFabricClusterHealth
```

## <a name="clean-up-resources"></a>清除資源

本教學課程系列的其他文章會使用您剛才建立的叢集。 如果您現在不打算繼續閱讀下一篇文章，您可能要[刪除該叢集](service-fabric-cluster-delete.md)以避免產生費用。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用 PowerShell 在 Azure 中建立 VNET
> * 建立金鑰保存庫並上傳憑證
> * 使用 PowerShell 在 Azure 中建立安全的 Service Fabric 叢集
> * 使用 X.509 憑證保護叢集
> * 使用 PowerShell 連線到叢集
> * 刪除叢集

接下來，前進到下列的教學課程，了解如何調整叢集。
> [!div class="nextstepaction"]
> [調整叢集](service-fabric-tutorial-scale-cluster.md)

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/5-VM-Windows-1-NodeTypes-Secure-NSG/azuredeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/5-VM-Windows-1-NodeTypes-Secure-NSG/azuredeploy.parameters.json
