---
title: 在 Azure 中建立 Linux Service Fabric 叢集 | Microsoft Docs
description: 在本教學課程中，您會了解如何使用 Azure CLI 將 Linux Service Fabric 叢集部署到現有的 Azure 虛擬網路。
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
ms.openlocfilehash: 33b95c1b0e3d654ce8bb6eda3e96b7b3e9c9bc13
ms.sourcegitcommit: 26cc9a1feb03a00d92da6f022d34940192ef2c42
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/06/2018
ms.locfileid: "48831478"
---
# <a name="tutorial-deploy-a-linux-service-fabric-cluster-into-an-azure-virtual-network"></a>教學課程：將 Linux Service Fabric 叢集部署到 Azure 虛擬網路

本教學課程是一個系列的第一部分。 您將會了解如何使用 Azure CLI 和範本將 Linux Service Fabric 叢集部署到 [Azure 虛擬網路 (VNET)](../virtual-network/virtual-networks-overview.md)。 完成時，您會有在您可以部署應用程式的雲端中執行的叢集。 若要使用 PowerShell 建立 Windows 叢集，請參閱[在 Azure 上建立安全的 Windows 叢集](service-fabric-tutorial-create-vnet-and-windows-cluster.md)。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 在 Azure 中使用 Azure CLI 建立 VNET
> * 在 Azure 中使用 Azure CLI 建立安全的 Service Fabric 叢集
> * 使用 X.509 憑證保護叢集
> * 使用 Service Fabric CLI 連接到叢集
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
* 安裝 [Service Fabric CLI](service-fabric-cli.md)
* 安裝 [Azure CLI](/cli/azure/install-azure-cli)

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

此教學課程將部署由單一節點類型中五個節點組成的叢集。 不過，對於任何生產環境叢集部署，[容量規劃](service-fabric-cluster-capacity.md)都是一個很重要的步驟。 以下是一些您在該程序中必須考量的事情。

* 您的叢集所需的節點數目和節點類型
* 每個節點類型的屬性 (例如大小、主要、網際網路面向、VM 數目等)
* 叢集的可靠性和持久性的特性

## <a name="download-and-explore-the-template"></a>下載並瀏覽範本

下載下列 Resource Manager 範本檔案：

* [AzureDeploy.json][template]
* [AzureDeploy.Parameters.json][parameters]

此範本會將一個由五個虛擬機器組成並屬於單一節點類型的安全叢集部署到虛擬網路中。  您可以在 [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates) 上找到其他範例範本。 [AzureDeploy.json][template] 會部署多項資源，包括下列各項。

### <a name="service-fabric-cluster"></a>Service Fabric 叢集

在 **Microsoft.ServiceFabric/clusters** Linux 叢集會以下列特性部署：

* 單一節點類型
* 屬於主要節點類型的五個節點 (可在範本參數中設定)
* 作業系統：Ubuntu 16.04 LTS (可在範本參數中設定)
* 受保護的憑證 (可在範本參數中設定)
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

### <a name="virtual-network-and-subnet"></a>虛擬網路和子網路

虛擬網路和子網路的名稱會在範本參數中宣告。  虛擬網路和子網路的位址空間也會在範本參數中宣告，並設定於 **Microsoft.Network/virtualNetworks** 資源中：

* 虛擬網路位址空間：10.0.0.0/16
* Service Fabric 子網路位址空間：10.0.2.0/24

如果需要任何其他應用程式連接埠，則您必須調整 Microsoft.Network/loadBalancers 資源，以允許流量進入。

## <a name="set-template-parameters"></a>設定範本參數

[AzureDeploy.Parameters][parameters] 參數檔案會宣告多個用來部署叢集和相關聯資源的值。 您可能需要為自己的部署修改某些參數：

|參數|範例值|注意|
|---|---||
|adminUserName|vmadmin| 叢集 VM 的系統管理員使用者名稱。 |
|adminPassword|Password#1234| 叢集 VM 的系統管理員密碼。|
|clusterName|mysfcluster123| 叢集的名稱。 |
|location|southcentralus| 叢集的位置。 |
|certificateThumbprint|| <p>如果建立自我簽署憑證或提供憑證檔案，則值應該空白。</p><p>若要使用先前上傳至金鑰保存庫的現有憑證，請填入憑證 SHA1 指紋值。 例如 "6190390162C988701DB5676EB81083EA608DCCF3"。 </p>|
|certificateUrlValue|| <p>如果建立自我簽署憑證或提供憑證檔案，則值應該空白。</p><p>若要使用先前上傳至金鑰保存庫的現有憑證，請填入憑證 URL。 例如，"https://mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346"。</p>|
|sourceVaultValue||<p>如果建立自我簽署憑證或提供憑證檔案，則值應該空白。</p><p>若要使用先前上傳至金鑰保存庫的現有憑證，請填入來源保存庫值。 例如 "/subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT"。</p>|

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>部署虛擬網路和叢集

接下來，請設定網路拓撲並部署 Service Fabric 叢集。 [AzureDeploy.json][template] Resource Manager 範本會建立虛擬網路 (VNET) 及適用於 Service Fabric 的子網路。 範本也會部署啟用憑證安全性的叢集。  對於生產叢集，請使用憑證授權單位 (CA) 提供的憑證作為叢集憑證。 自我簽署憑證可用來保護測試叢集。

### <a name="create-a-cluster-using-an-existing-certificate"></a>使用現有的憑證建立叢集

下列指令碼會使用 [az sf cluster create](/cli/azure/sf/cluster?view=azure-cli-latest#az_sf_cluster_create) 命令和範本，部署以現有憑證保護的新叢集。 此命令也會在 Azure 中建立新的金鑰保存庫，並上傳您的憑證。

```azurecli
ResourceGroupName="sflinuxclustergroup"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates\MyCertificate.pem"

# sign in to your Azure account and select your subscription
az login
az account set --subscription <guid>

# Create a new resource group for your deployment and give it a name and a location.
az group create --name $ResourceGroupName --location $Location

# Create the Service Fabric cluster.
az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --certificate-password $Password --certificate-file $CertPath \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName  \
   --template-file AzureDeploy.json --parameter-file AzureDeploy.Parameters.json
```

### <a name="create-a-cluster-using-a-new-self-signed-certificate"></a>使用新的自我簽署憑證建立叢集

下列指令碼會使用 [az sf cluster create](/cli/azure/sf/cluster?view=azure-cli-latest#az_sf_cluster_create) 命令和範本，在 Azure 中部署新的叢集。 此命令也會在 Azure 中建立新的金鑰保存庫、將新的自我簽署憑證新增至金鑰保存庫，並將憑證檔案下載至本機。

```azurecli
ResourceGroupName="sflinuxclustergroup"
ClusterName="sflinuxcluster"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates"

az sf cluster create --resource-group $ResourceGroupName --location $Location --cluster-name $ClusterName --template-file C:\temp\cluster\AzureDeploy.json --parameter-file C:\temp\cluster\AzureDeploy.Parameters.json --certificate-password $Password --certificate-output-folder $CertPath --certificate-subject-name $ClusterName.$Location.cloudapp.azure.com --vault-name $VaultName --vault-resource-group $ResourceGroupName
```

## <a name="connect-to-the-secure-cluster"></a>連線到安全的叢集

使用 Service Fabric CLI 命令 `sfctl cluster select` 搭配您的金鑰來連線到叢集。  請注意，只能針對自我簽署憑證使用 **--no-verify** 選項。

```azurecli
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

使用 `sfctl cluster health` 命令來檢查您連接的叢集是否狀況良好。

```azurecli
sfctl cluster health
```

## <a name="clean-up-resources"></a>清除資源

本教學課程系列的其他文章會使用您剛才建立的叢集。 如果您現在不打算繼續閱讀下一篇文章，您可能要[刪除該叢集](service-fabric-cluster-delete.md)以避免產生費用。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 在 Azure 中使用 Azure CLI 建立 VNET
> * 在 Azure 中使用 Azure CLI 建立安全的 Service Fabric 叢集
> * 使用 X.509 憑證保護叢集
> * 使用 Service Fabric CLI 連接到叢集
> * 刪除叢集

接下來，前進到下列的教學課程，了解如何調整叢集。
> [!div class="nextstepaction"]
> [調整叢集](service-fabric-tutorial-scale-cluster.md)

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/5-VM-Ubuntu-1-NodeTypes-Secure/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/5-VM-Ubuntu-1-NodeTypes-Secure/AzureDeploy.Parameters.json
