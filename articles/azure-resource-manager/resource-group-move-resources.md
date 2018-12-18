---
title: 將 Azure 資源移至新的訂用帳戶或資源群組 | Microsoft Docs
description: 使用 Azure Resource Manager 將資源移到新的資源群組或訂用帳戶。
services: azure-resource-manager
documentationcenter: ''
author: tfitzmac
ms.assetid: ab7d42bd-8434-4026-a892-df4a97b60a9b
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 10/17/2018
ms.author: tomfitz
ms.openlocfilehash: c32c4f97a963485e87e36afc44e9cea2ebcebd90
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394403"
---
# <a name="move-resources-to-new-resource-group-or-subscription"></a>將資源移動到新的資源群組或訂用帳戶

本文說明如何將資源移至新的訂用帳戶或相同訂用帳戶中新的資源群組。 您可以使用入口網站、PowerShell、Azure CLI 或 REST API 來移動資源。 本文中的移動作業可供您使用而不需要任何 Azure 支援的協助。

移動資源時，在此作業期間會同時鎖定來源群組和目標群組。 資源群組上的寫入和刪除作業將會封鎖，直到移動完成。 此鎖定表示您無法新增、更新或刪除資源群組中的資源，但不表示資源已遭到凍結。 例如，如果您將 SQL Server 和其資料庫移至新的資源群組，使用該資料庫的應用程式不會發生停機時間。 它仍可對資料庫讀取和寫入。

您無法變更資源的位置。 移動資源只會將它移動到新的資源群組。 新的資源群組可能會有不同的位置，但那樣不會變更資源的位置。

> [!NOTE]
> 本文說明如何在現有的 Azure 帳戶供應項目內移動資源。 如果您真的想要變更 Azure 帳戶供應項目 (例如，從隨用隨付升級為預付)，同時繼續使用現有的資源，請參閱 [切換至不同的 Azure 訂用帳戶供應項目](../billing/billing-how-to-switch-azure-offer.md)。
>
>

## <a name="checklist-before-moving-resources"></a>移動資源前的檢查清單

在移動資源之前，要執行的重要步驟如下︰ 藉由驗證這些條件，您可以避免錯誤。

1. 來源和目的地的訂用帳戶必須存在於相同的 [Azure Active Directory 租用戶](../active-directory/develop/quickstart-create-new-tenant.md)內。 若要檢查這兩個訂用帳戶都有相同的租用戶識別碼，請使用 Azure PowerShell 或 Azure CLI。

  如果是 Azure PowerShell，請使用：

  ```powershell
  (Get-AzureRmSubscription -SubscriptionName <your-source-subscription>).TenantId
  (Get-AzureRmSubscription -SubscriptionName <your-destination-subscription>).TenantId
  ```

  對於 Azure CLI，請使用：

  ```azurecli-interactive
  az account show --subscription <your-source-subscription> --query tenantId
  az account show --subscription <your-destination-subscription> --query tenantId
  ```

  如果來源和目的地訂用帳戶的租用戶識別碼不相同，請使用下列方法來協調租用戶識別碼：

  * [將 Azure 訂用帳戶的擁有權轉移給另一個帳戶](../billing/billing-subscription-transfer.md)
  * [如何將 Azure 訂用帳戶關聯或新增至 Azure Active Directory](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)

1. 必須針對要移動之資源的資源提供者註冊其目的地訂用帳戶。 否則，您會收到錯誤，指出 **未針對資源類型註冊訂用帳戶**。 將資源移至新的訂用帳戶時，可能會因為該訂用帳戶不曾以指定的資源類型使用過而遇到問題。

  針對 PowerShell，使用下列命令來取得註冊狀態：

  ```powershell
  Set-AzureRmContext -Subscription <destination-subscription-name-or-id>
  Get-AzureRmResourceProvider -ListAvailable | Select-Object ProviderNamespace, RegistrationState
  ```

  若要註冊資源提供者，請使用：

  ```powershell
  Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
  ```

  針對 Azure CLI，使用下列命令來取得註冊狀態：

  ```azurecli-interactive
  az account set -s <destination-subscription-name-or-id>
  az provider list --query "[].{Provider:namespace, Status:registrationState}" --out table
  ```

  若要註冊資源提供者，請使用：

  ```azurecli-interactive
  az provider register --namespace Microsoft.Batch
  ```

1. 移動資源的帳戶至少必須有下列權限：

   * 來源資源群組上的 **Microsoft.Resources/subscriptions/resourceGroups/moveResources/action**。
   * 來源資源群組上的 **Microsoft.Resources/subscriptions/resourceGroups/write**。

1. 移動資源之前，請針對您要將資源移入的訂用帳戶，檢查其訂用帳戶配額。 如果移動資源表示訂用帳戶會超出限制，那麼您必須檢閱您是否可以要求增加配額。 如需限制清單和如何要求增加配額的資訊，請參閱 [Azure 訂用帳戶和服務限制、配額及條件約束](../azure-subscription-service-limits.md)。

1. 可能的話，請將大型移動細分為個別的移動作業。 Resource Manager 在單一作業中嘗試移動超過 800 個資源會立即失敗。 不過，移動少於 800 個資源也可能因為逾時而失敗。

1. 服務必須啟用移動資源的功能。 若要判斷移動是否會成功，請[驗證您的移動要求](#validate-move)。 請參閱本文的以下小節，了解[哪些服務可移動資源](#services-that-can-be-moved)，以及[哪些服務無法移動資源](#services-that-cannot-be-moved)。

## <a name="when-to-call-support"></a>呼叫支援的時機

您可以透過本文顯示的自助式作業，移動大部分資源。 使用自助式作業︰

* 移動 Resource Manager 資源。
* 根據[傳統部署限制](#classic-deployment-limitations)移動傳統資源。

當您需要進行下列作業時，請連絡[支援人員](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview)︰

* 將資源移至新的 Azure 帳戶 (和 Azure Active Directory 租用戶)，而且需要上一節中指示的說明。
* 移動傳統資源，但有限制的問題。

## <a name="validate-move"></a>驗證移動

[驗證移動作業](/rest/api/resources/resources/resources_validatemoveresources)可讓您直接測試移動案例，而不需要實際移動資源。 您可以使用這項作業來判斷移動是否會成功。 若要執行這項作業，您需要：

* 來源資源群組的名稱
* 目標資源群組的資源識別碼
* 要移動的每個資源所具備的資源識別碼
* 帳戶的[存取權杖](/rest/api/azure/#acquire-an-access-token)

傳送下列要求：

```
POST https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<source-group>/validateMoveResources?api-version=2018-02-01
Authorization: Bearer <access-token>
Content-type: application/json
```

使用要求本文：

```json
{
 "resources": ['<resource-id-1>', '<resource-id-2>'],
 "targetResourceGroup": "/subscriptions/<subscription-id>/resourceGroups/<target-group>"
}
```

如果要求的格式正確，此作業將會傳回：

```
Response Code: 202
cache-control: no-cache
pragma: no-cache
expires: -1
location: https://management.azure.com/subscriptions/<subscription-id>/operationresults/<operation-id>?api-version=2018-02-01
retry-after: 15
...
```

202 狀態碼指出已接受驗證要求，但尚未判斷移動作業是否會成功。 `location` 值包含一個 URL，可讓您用來檢查長時間執行作業的狀態。  

若要檢查狀態，請傳送下列要求：

```
GET <location-url>
Authorization: Bearer <access-token>
```

當作業仍在執行時，您會持續收到 202 狀態碼。 請等候 `retry-after` 值中指出的秒數，再重新嘗試。 如果移動作業驗證成功，您會收到 204 狀態碼。 如果移動驗證失敗，則會收到錯誤訊息，例如：

```json
{"error":{"code":"ResourceMoveProviderValidationFailed","message":"<message>"...}}
```

## <a name="services-that-can-be-moved"></a>可以移動的服務

下列清單提供可移至新資源群組和訂用帳戶之 Azure 服務的一般摘要。 如需更多詳細資料，請參閱[資源的移動作業支援](move-support-resources.md)。

* Analysis Services
* API 管理
* App Service 應用程式 (Web 應用程式) - 請參閱 [App Service 限制](#app-service-limitations)
* App Service 憑證
* Application Insights
* 自動化
* Azure Active Directory B2C
* Azure Cosmos DB
* 適用於 MySQL 的 Azure 資料庫
* 適用於 PostgreSQL 的 Azure 資料庫
* Azure DevOps - 購買非 Microsoft 延伸模組的 Azure DevOps 組織必須先[取消其購買](https://go.microsoft.com/fwlink/?linkid=871160)，才能在訂用帳戶之間移動帳戶。
* Azure 地圖服務
* Azure 轉送
* Azure Stack - 註冊
* Batch
* BizTalk 服務
* Bot 服務
* CDN
* 雲端服務 - 請參閱 [傳統部署限制](#classic-deployment-limitations)
* 認知服務
* Container Registry
* 內容仲裁
* 成本管理
* Customer Insights
* 資料目錄
* Data Factory
* 資料湖分析
* Data Lake Store
* DNS
* Event Grid
* 事件中樞
* Front Door
* HDInsight 叢集 - 請參閱 [HDInsight 限制](#hdinsight-limitations)
* IoT 中心
* IoT 中樞
* Key Vault
* Load Balancer - 請參閱 [Load Balancer 限制](#lb-limitations)
* Log Analytics
* Logic Apps
* Machine Learning：Machine Learning Studio Web 服務可以移至相同訂用帳戶 (而非不同的訂用帳戶) 中的資源群組。 其他 Machine Learning 資源可以在訂用帳戶之間移動。
* 受控磁碟 - 請參閱[虛擬機器的條件約束限制](#virtual-machines-limitations)
* 受控識別 - 使用者指派
* 媒體服務
* 通知中樞
* Operational Insights
* Operations Management
* 入口網站儀表板
* Power BI - Power BI Embedded 和 Power BI 工作區集合
* 公用 IP - 請參閱[公用 IP 限制](#pip-limitations)
* Redis 快取 - 如果已為「Redis 快取」執行個體設定虛擬網路，該執行個體便無法移至不同的訂用帳戶。 請參閱[虛擬網路限制](#virtual-networks-limitations)。
* 排程器
* Search
* 服務匯流排
* Service Fabric
* Service Fabric Mesh
* SignalR Service
* 儲存體 - 不同區域中的儲存體帳戶無法在相同的作業中移動。 請改為針對每個區域使用個別的作業。
* 儲存體 (傳統) - 請參閱 [傳統部署限制](#classic-deployment-limitations)
* 串流分析 - 無法移動執行中狀態的串流分析作業。
* SQL Database 伺服器 - 資料庫和伺服器必須位於相同的資源群組。 當您移動 SQL 伺服器時，其所有資料庫也會跟著移動。 此行為會套用至 Azure SQL Database 和 Azure SQL Data Warehouse 資料庫。
* 時間序列深入解析
* 流量管理員
* 虛擬機器 - 針對具有受控磁碟的 VM，請參閱[虛擬機器限制](#virtual-machines-limitations)
* 虛擬機器 (傳統) - 請參閱 [傳統部署限制](#classic-deployment-limitations)
* 虛擬機器擴展集 - 請參閱[虛擬機器限制](#virtual-machines-limitations)
* 虛擬網路 - 請參閱[虛擬網路限制](#virtual-networks-limitations)
* VPN 閘道

## <a name="services-that-cannot-be-moved"></a>無法移動的服務

下列清單提供無法移至新資源群組和訂用帳戶之 Azure 服務的一般摘要。 如需更多詳細資料，請參閱[資源的移動作業支援](move-support-resources.md)。

* AD Domain Services
* AD 混合式健康狀態服務
* 應用程式閘道
* Azure 資料庫移轉
* Azure Databricks
* Azure Migrate
* Batch AI
* 憑證 - App Service 憑證可以移動，但上傳的憑證則有其[限制](#app-service-limitations)。
* Container Instances
* 容器服務
* 資料箱
* Dev Spaces
* Dynamics LCS
* ExpressRoute
* Kubernetes 服務
* 實驗室服務 - 已啟用移動至相同訂用帳戶中新資源群組的功能，但未啟用跨訂用帳戶之間的移動。
* Load Balancer - 請參閱 [Load Balancer 限制](#lb-limitations)
* 受控應用程式
* Microsoft Genomics
* NetApp
* 公用 IP - 請參閱[公用 IP 限制](#pip-limitations)
* 復原服務保存庫 - 也不會移動與「復原服務」保存庫關聯的「計算」、「網路」及「儲存體」資源，請參閱 [復原服務限制](#recovery-services-limitations)。
* SAP HANA on Azure
* 安全性
* Site Recovery
* StorSimple 裝置管理員
* 虛擬網路 (傳統) - 請參閱 [傳統部署限制](#classic-deployment-limitations)

## <a name="virtual-machines-limitations"></a>虛擬機器限制

自 2018 年 9 月 24 日起，支援移動受控磁碟。 

1. 您必須註冊才能啟用此功能。

  ```azurepowershell-interactive
  Register-AzureRmProviderFeature -FeatureName ManagedResourcesMove -ProviderNamespace Microsoft.Compute
  ```

  ```azurecli-interactive
  az feature register --namespace Microsoft.Compute --name ManagedResourcesMove
  ```

1. 註冊要求一開始會傳回 `Registering`狀態。 您可以使用下列方式來檢查目前狀態：

  ```azurepowershell-interactive
  Get-AzureRmProviderFeature -FeatureName ManagedResourcesMove -ProviderNamespace Microsoft.Compute
  ```

  ```azurecli-interactive
  az feature show --namespace Microsoft.Compute --name ManagedResourcesMove
  ```

1. 稍候幾分鐘以讓狀態變更為 `Registered`。

1. 註冊此功能之後，註冊 `Microsoft.Compute` 資源提供者。 即使先前已註冊過此資源提供者，仍請執行此步驟。

  ```azurepowershell-interactive
  Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Compute
  ```

  ```azurecli-interactive
  az provider register --namespace Microsoft.Compute
  ```

此支援意謂著您也可以移動：

* 具有受控磁碟的虛擬機器
* 受控映像
* 受控快照集
* 具有使用受控磁碟之虛擬機器的可用性設定組

以下是尚未支援的條件約束：

* 憑證儲存在 Key Vault 中的虛擬機器可以移動至相同訂用帳戶中的新資源群組，但是無法跨訂用帳戶移動。
* 已設定「Azure 備份」的虛擬機器。 請使用下列因應措施來移動這些虛擬機器
  * 找出您虛擬機器的位置。
  * 找出具有下列命名模式的資源群組：`AzureBackupRG_<location of your VM>_1`，例如 AzureBackupRG_westus2_1
  * 如果是在 Azure 入口網站中，則請選取 [顯示隱藏的類型]
  * 如果是在 PowerShell 中，請使用 `Get-AzureRmResource -ResourceGroupName AzureBackupRG_<location of your VM>_1` Cmdlet
  * 如果是在 CLI 中，請使用 `az resource list -g AzureBackupRG_<location of your VM>_1`
  * 現在，找出類型為 `Microsoft.Compute/restorePointCollections` 且命名模式為 `AzureBackup_<name of your VM that you're trying to move>_###########` 的資源
  * 刪除此資源
  * 完成刪除之後，您便能夠移動您的虛擬機器
* 具有標準 SKU 負載平衡器或標準 SKU 公用 IP 的虛擬機器擴展集無法移動
* 從 Marketplace 資源建立且附加方案的虛擬機器無法在資源群組或訂用帳戶之間移動。 在目前的訂用帳戶中取消佈建虛擬機器，然後於新訂用帳戶中再次部署。


## <a name="virtual-networks-limitations"></a>虛擬網路限制

當您移動虛擬網路時，也必須移動其相依資源。 針對 VPN 閘道，您必須移動 IP 位址、虛擬網路閘道和所有相關聯的連線資源。 區域網路閘道可位於不同的資源群組。

若要移動對等虛擬網路，您必須先停用虛擬網路對等互連。 停用之後，您可以移動虛擬網路。 移動之後，重新啟用虛擬網路對等互連。

如果虛擬網路包含有資源導覽連結的子網路，則無法將虛擬網路移動到其他訂用帳戶。 例如，如果 Redis 快取資源部署到子網路，該子網路具有資源導覽連結。

## <a name="app-service-limitations"></a>App Service 限制

根據您是要移動訂用帳戶內的資源還是將資源移到新的訂用帳戶，移動 App Service 資源的限制會有所不同。

這幾節所述的限制適用於上傳憑證，不適用於 App Service 憑證。 您可以將 App Service 憑證移至新資源群組或訂用帳戶，沒有任何限制。 如果您有多個 Web 應用程式使用相同的 App Service 憑證，請先移動所有的 Web 應用程式，再移動憑證。

### <a name="moving-within-the-same-subscription"></a>在相同的訂用帳戶內移動

在_相同訂用帳戶內_移動 Web 應用程式時，您無法移動已上傳的 SSL 憑證。 不過，您可以只將 Web 應用程式移至新的資源群組，而不移動其上傳的 SSL 憑證，而且您應用程式的 SSL 功能仍可正常運作。

如果您想在移動 Web 應用程式時一併移動 SSL 憑證，請遵循下列步驟：

1.  刪除從 Web 應用程式上傳的憑證。
2.  移動 Web 應用程式。
3.  將憑證上傳至移動的 Web 應用程式。

### <a name="moving-across-subscriptions"></a>跨訂用帳戶移動

在_訂用帳戶之間_移動 Web 應用程式時，適用下列限制：

- 目的地資源群組中必須沒有任何已存在的 App Service。 App Service 資源包括：
    - Web Apps
    - App Service 方案
    - 已上傳或已匯的入 SSL 憑證
    - App Service 環境
- 資源群組中的所有 App Service 資源必須一起移動。
- 只能從其最初建立 App Service 資源的資源群組中移動 App Service 資源。 如果 App Service 資源已不存在於其原始的資源群組中，則必須將其移回原始資源群組，然後才可以在訂用帳戶間移動。

## <a name="classic-deployment-limitations"></a>傳統部署限制

透過傳統模型所部署的資源移動選項，會根據移動訂用帳戶內的資源還是將資源移到新的訂用帳戶而有所不同。

### <a name="same-subscription"></a>相同訂用帳戶

將資源從一個資源群組移到相同訂用帳戶內的另一個資源群組時，適用下列限制︰

* 無法移動虛擬網路 (傳統)。
* 虛擬機器 (傳統) 必須與雲端服務一起移動。
* 只有當移動作業包含雲端服務的所有虛擬機器時，才能移動雲端服務。
* 一次只能移動一個雲端服務。
* 一次只能移動一個儲存體帳戶 (傳統)。
* 透過相同的作業，儲存體帳戶 (傳統) 不能與虛擬機器或雲端服務一起移動。

若要將傳統資源移到相同訂用帳戶內的新資源群組，請透過[入口網站](#use-portal)、[Azure PowerShell](#use-powershell)、[Azure CLI](#use-azure-cli) 或 [REST API](#use-rest-api)，使用標準移動作業。 當您移動 Resource Manager 資源時，您會使用相同的作業。

### <a name="new-subscription"></a>新的訂用帳戶

將資源移到新訂用帳戶時，適用下列限制︰

* 必須透過相同的作業移動訂用帳戶中的所有傳統資源。
* 目標訂用帳戶不得包含任何其他傳統資源。
* 只能透過適用於傳統移動的個別 REST API 來要求移動。 將傳統資源移到新的訂用帳戶時，標準 Resource Manager 移動命令無法運作。

若要將傳統資源移到新的訂用帳戶，請使用傳統資源特定的 REST 作業。 若要使用 REST，請執行下列步驟：

1. 請檢查來源訂用帳戶是否可以參與跨訂用帳戶移動。 請使用下列作業：

  ```HTTP
  POST https://management.azure.com/subscriptions/{sourceSubscriptionId}/providers/Microsoft.ClassicCompute/validateSubscriptionMoveAvailability?api-version=2016-04-01
  ```

     在要求本文中包含：

  ```json
  {
    "role": "source"
  }
  ```

     驗證作業的回應格式如下︰

  ```json
  {
    "status": "{status}",
    "reasons": [
      "reason1",
      "reason2"
    ]
  }
  ```

2. 請檢查目的地訂用帳戶是否可以參與跨訂用帳戶移動。 請使用下列作業：

  ```HTTP
  POST https://management.azure.com/subscriptions/{destinationSubscriptionId}/providers/Microsoft.ClassicCompute/validateSubscriptionMoveAvailability?api-version=2016-04-01
  ```

     在要求本文中包含：

  ```json
  {
    "role": "target"
  }
  ```

     回應的格式與來源訂用帳戶驗證的格式相同。
3. 如果兩個訂用帳戶都通過驗證，使用下列作業將所有傳統資源從某個訂用帳戶移到另一個訂用帳戶︰

  ```HTTP
  POST https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.ClassicCompute/moveSubscriptionResources?api-version=2016-04-01
  ```

    在要求本文中包含：

  ```json
  {
    "target": "/subscriptions/{target-subscription-id}"
  }
  ```

這項作業可能需要幾分鐘的時間執行。

## <a name="recovery-services-limitations"></a>復原服務限制

無法移動用來設定 Azure Site Recovery 相關災害復原的「儲存體」、「網路」或「計算」資源。

舉例來說，假設您已設定將內部部署機器複寫到某個儲存體帳戶 (Storage1)，而想要讓受保護的機器在容錯移轉到 Azure 之後，以連接到虛擬網路 (Network1) 的虛擬機器 (VM1) 身分上線。 您無法跨相同訂用帳戶內的資源群組或跨訂用帳戶來移動任何這些 Azure 資源 - Storage1、VM1 及 Network1。

若要在資源群組之間移動 **Azure 備份**中註冊的虛擬機器：
 1. 暫時停止備份並保留備份資料
 2. 將虛擬機器移到目標資源群組
 3. 在相同/新的保存庫下將其重新保護。使用者可以從移動作業前建立的可用還原點進行還原。
如果使用者是在訂用帳戶之間移動備份虛擬機器，步驟 1 和步驟 2 保持不變。 在步驟 3 中，使用者需要保護在目標訂用帳戶中存在/建立之新保存庫底下的 VM。 復原服務保存庫不支援跨訂用帳戶的備份。

## <a name="hdinsight-limitations"></a>HDInsight 限制

您可以將 HDInsight 叢集移至新的訂用帳戶或資源群組。 不過，您無法跨訂用帳戶來移動連結至 HDInsight 叢集的網路資源 (例如虛擬網路、NIC 或負載平衡器)。 此外，您無法將已連結至叢集虛擬機器的 NIC 移至新的資源群組。

將 HDInsight 叢集移至新的訂用帳戶時，請先移動其他資源 (例如儲存體帳戶)。 然後，移動 HDInsight 叢集本身。

## <a name="search-limitations"></a>搜尋的限制

您不能同時移動放在不同區域的多個搜尋資源。
在這種情況下，您需要分別移動它們。

## <a name="lb-limitations"></a> Load Balancer 限制

可以移動基本 SKU Load Balancer。
不能移動標準 SKU Load Balancer。

## <a name="pip-limitations"></a> 公用 IP 限制

可以移動基本 SKU 公用 IP。
不能移動標準 SKU 公用 IP。

## <a name="use-portal"></a>使用入口網站

若要移動資源，請選取包含這些資源的資源群組，然後選取 [移動] 按鈕。

![移動資源](./media/resource-group-move-resources/select-move.png)

選擇將資源移動到新的資源群組或新的訂用帳戶。

選取要移動的資源和目的地資源群組。 認可您需要更新這些資源的指令碼，然後選取 [確定] 。 如果您在上一個步驟中選取了 [編輯訂用帳戶] 圖示，則也必須選取目的地訂用帳戶。

![選取目的地](./media/resource-group-move-resources/select-destination.png)

在 [通知] 中，您會看到移動作業正在執行。

![顯示移動狀態](./media/resource-group-move-resources/show-status.png)

完成後，您會收到結果的通知。

![顯示移動結果](./media/resource-group-move-resources/show-result.png)

## <a name="use-powershell"></a>使用 PowerShell

若要將現有的資源移動到另一個資源群組或訂用帳戶，請使用 [Move-AzureRmResource](/powershell/module/azurerm.resources/move-azurermresource) 命令。 下列範例顯示如何將多個資源移動到新的資源群組。

```powershell
$webapp = Get-AzureRmResource -ResourceGroupName OldRG -ResourceName ExampleSite
$plan = Get-AzureRmResource -ResourceGroupName OldRG -ResourceName ExamplePlan
Move-AzureRmResource -DestinationResourceGroupName NewRG -ResourceId $webapp.ResourceId, $plan.ResourceId
```

若要移動到新的訂用帳戶，請包含 `DestinationSubscriptionId`參數的值。

## <a name="use-azure-cli"></a>使用 Azure CLI

若要將現有的資源移動到另一個資源群組或訂用帳戶，請使用 [az resource move](/cli/azure/resource?view=azure-cli-latest#az-resource-move) 命令。 提供要移動之資源的資源識別碼。 下列範例顯示如何將多個資源移動到新的資源群組。 請在 `--ids` 參數中，為要移動的資源識別碼提供以空格分隔的清單。

```azurecli
webapp=$(az resource show -g OldRG -n ExampleSite --resource-type "Microsoft.Web/sites" --query id --output tsv)
plan=$(az resource show -g OldRG -n ExamplePlan --resource-type "Microsoft.Web/serverfarms" --query id --output tsv)
az resource move --destination-group newgroup --ids $webapp $plan
```

若要移動到新的訂用帳戶，請提供 `--destination-subscription-id` 參數。

## <a name="use-rest-api"></a>使用 REST API

若要將現有的資源移動到另一個資源群組或訂用帳戶，請執行：

```HTTP
POST https://management.azure.com/subscriptions/{source-subscription-id}/resourcegroups/{source-resource-group-name}/moveResources?api-version={api-version}
```

在要求主體中，您可以指定目標資源群組以及要移動的資源。 如需有關 REST 移動作業的詳細資訊，請參閱 [移動資源](/rest/api/resources/Resources/MoveResources)。

## <a name="next-steps"></a>後續步驟

* 若要了解用於管理訂用帳戶的 PowerShell Cmdlet，請參閱 [搭配使用 Azure PowerShell 與 Azure Resource Manager](powershell-azure-resource-manager.md)。
* 若要了解用於管理訂用帳戶的 Azure CLI 命令，請參閱 [搭配使用 Azure CLI 與 Azure Resource Manager](xplat-cli-azure-resource-manager.md)。
* 若要了解用於管理訂用帳戶的入口網站功能，請參閱 [使用 Azure 入口網站來管理資源](resource-group-portal.md)。
* 若要了解如何將邏輯組織套用到您的資源，請參閱 [使用標記來組織您的資源](resource-group-using-tags.md)。
