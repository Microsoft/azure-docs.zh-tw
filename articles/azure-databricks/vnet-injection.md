---
title: 在虛擬網路中部署 Azure Databricks
description: 本文說明如何將 Azure Databricks 部署至您的虛擬網路，也稱為 VNet Injection。
services: azure-databricks
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: azure-databricks
ms.topic: conceptual
ms.date: 10/10/2019
ms.openlocfilehash: 07591517211d5334b9bf055d778f00b171e7056f
ms.sourcegitcommit: b4665f444dcafccd74415fb6cc3d3b65746a1a31
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2019
ms.locfileid: "72263461"
---
# <a name="deploy-azure-databricks-in-your-virtual-network"></a>在虛擬網路中部署 Azure Databricks

Azure Databricks 的預設部署是 Azure 上完全受控的服務：所有資料層資源（包括虛擬網路（VNet））都會部署至已鎖定的資源群組。 不過，如果您需要網路自訂，則可以在您自己的虛擬網路（也稱為 VNet Injection）中部署 Azure Databricks 資源，以讓您：

* 使用服務端點，以更安全的方式將 Azure Databricks 連接到其他 Azure 服務（例如 Azure 儲存體）。
* 連接到內部部署資料來源以與 Azure Databricks 搭配使用，利用使用者定義的路由。
* 將 Azure Databricks 連線到網路虛擬裝置，以檢查所有輸出流量，並根據允許和拒絕規則採取動作。
* 將 Azure Databricks 設定為使用自訂 DNS。
* 設定網路安全性群組（NSG）規則，以指定輸出流量限制。
* 在現有的虛擬網路中部署 Azure Databricks 叢集。

將 Azure Databricks 資源部署到您自己的虛擬網路也可讓您利用彈性的 CIDR 範圍（虛擬網路的/16-/24 之間的任何位置，以及子網路之間的/18-/26）。

  > [!NOTE]
  > 您無法取代現有工作區的虛擬網路。 如果您目前的工作區無法容納所需的作用中叢集節點數目，請在較大的虛擬網路中建立另一個工作區。 請遵循[下列詳細的遷移步驟](howto-regional-disaster-recovery.md#detailed-migration-steps)，將資源（筆記本、叢集設定、作業）從舊的工作區複製到新的工作區。

## <a name="virtual-network-requirements"></a>虛擬網路需求

您可以使用 Azure 入口網站中的 Azure Databricks 工作區部署介面，自動以必要的子網路、網路安全性群組和允許清單設定來設定現有的虛擬網路，也可以使用 Azure Resource Manager用來設定虛擬網路和部署工作區的範本。

您要將 Azure Databricks 工作區部署到其中的虛擬網路必須符合下列需求：

### <a name="location"></a>Location

虛擬網路必須位於與 Azure Databricks 工作區相同的位置。

### <a name="subnets"></a>子網路

虛擬網路必須包含兩個子網路，專門用於 Azure Databricks：

   1. 已設定網路安全性群組以允許叢集內部通訊的私人子網路

   2. 具有已設定網路安全性群組的公用子網路，允許與 Azure Databricks 控制層進行通訊。

### <a name="address-space"></a>位址空間

虛擬網路的/16-/24 與私人和公用子網路之間的 cidr 區塊介於/18-/26 之間。

### <a name="whitelisting"></a>加入允許清單

子網路與 Azure Databricks 控制層之間的所有輸出和輸入流量都必須列入允許清單。

## <a name="create-an-azure-databricks-workspace"></a>建立 Azure Databricks 工作區

本節說明如何在 Azure 入口網站中建立 Azure Databricks 工作區，並將它部署在您自己現有的虛擬網路中。 Azure Databricks 使用您所提供的 CIDR 範圍，以兩個新的子網路和網路安全性群組更新虛擬網路、將輸入和輸出子網路流量列入允許清單，並將工作區部署到更新的虛擬網路。

## <a name="prerequisites"></a>必要條件

您必須要有一個虛擬網路，您將在其中部署 Azure Databricks 工作區。 您可以使用現有的虛擬網路或建立一個新的，但是虛擬網路必須與您打算建立的 Azure Databricks 工作區位於相同的區域。 虛擬網路需要/16-/24 之間的 CIDR 範圍。

  > [!Warning]
  > 如果工作區具有較小的虛擬網路（也就是 CIDR 範圍較低），則可以比具有較大虛擬網路的工作區更快地使用 IP 位址（網路空間）。 例如，具有/24 個虛擬網路和/26 子網路的工作區一次最多可以有64節點，而具有/20 個虛擬網路和/22 子網的工作區最多可以容納1024個節點。

  當您設定工作區時，將會自動建立您的子網路，而且您將有機會在設定期間提供子網路的 CIDR 範圍。

## <a name="configure-the-virtual-network"></a>設定虛擬網路

1. 在 Azure 入口網站中，選取  **+ 建立資源 > 分析 > Azure Databricks**  以開啟 Azure Databricks 服務 對話方塊。

2. 請遵循步驟2：在 [消費者入門指南] 中建立 Azure Databricks 工作區，然後在您的虛擬網路選項中選取 [部署 Azure Databricks 工作區]。

   ![建立 Azure Databricks 服務](./media/vnet-injection/create-databricks-service.png)

3. 選取您想要使用的虛擬網路。

   ![虛擬網路選項](./media/vnet-injection/select-vnet.png)

4. 針對兩個子網路，在/18-/26 之間的區塊中提供 CIDR 範圍，專門用於 Azure Databricks：

   * 系統會使用關聯的網路安全性群組建立公用子網路，以允許與 Azure Databricks 控制層進行通訊。
   * 將會使用與允許叢集內部通訊的相關聯網絡安全性群組來建立私人子網路。

5. 按一下 [**建立**]，將 Azure Databricks 工作區部署到虛擬網路。

## <a name="advanced-resource-manager-configurations"></a>Advanced resource manager 設定

如果您想要更充分掌控虛擬網路的設定–例如，您想要使用現有的子網路、使用現有的網路安全性群組，或建立您自己的安全性規則–您可以使用下列 Azure Resource Manager 範本，而不是入口網站虛擬網路設定和工作區部署。

### <a name="all-in-one"></a>全部在一

若要建立虛擬網路、網路安全性群組和 Azure Databricks 的工作區，請使用[全功能範本來 Databricks VNet Injection的工作區](https://azure.microsoft.com/resources/templates/101-databricks-all-in-one-template-for-vnet-injection/)。

當您使用此範本時，您不需要執行任何手動允許清單的子網路流量。

### <a name="network-security-groups"></a>網路安全性群組

若要使用現有虛擬網路的必要規則來建立網路安全性群組，請使用「[網路安全性群組」範本來進行 Databricks VNet Injection](https://azure.microsoft.com/resources/templates/101-databricks-nsg-for-vnet-injection)。

當您使用此範本時，您不需要執行任何手動允許清單的子網路流量。

### <a name="virtual-network"></a>虛擬網路

若要建立具有適當公用和私人子網路的虛擬網路，請使用[虛擬網路範本來進行 Databricks VNet Injection](https://azure.microsoft.com/resources/templates/101-databricks-vnet-for-vnet-injection)。

如果您不使用 [網路安全性群組] 範本也使用此範本，則必須手動將允許清單規則新增至您用於虛擬網路的網路安全性群組。

### <a name="azure-databricks-workspace"></a>Azure Databricks 工作區

若要將 Azure Databricks 工作區部署到具有公用和私人子網路的現有虛擬網路，以及已設定的正確設定網路安全性群組，請使用[Databricks VNet Injection的工作區範本](https://azure.microsoft.com/resources/templates/101-databricks-workspace-with-vnet-injection)。

如果您不使用 [網路安全性群組] 範本也使用此範本，則必須手動將允許清單規則新增至您用於虛擬網路的網路安全性群組。

## <a name="whitelisting-subnet-traffic"></a>允許清單子網路流量

如果您未使用[Azure 入口網站](https://docs.azuredatabricks.net/administration-guide/cloud-configurations/azure/vnet-inject.html#vnet-inject-portal)或[Azure Resource Manager 範本](https://docs.azuredatabricks.net/administration-guide/cloud-configurations/azure/vnet-inject.html#vnet-inject-advanced)來建立網路安全性群組，您必須手動將子網上的下列流量列入允許清單。

|Direction|Protocol|Source|來源連接埠|目的地|目的地連接埠|
|---------|--------|------|-----------|-----------|----------------|
|輸入|\*|VirtualNetwork|\*|\*|\*|
|輸入|\*|控制層 NAT IP|\*|\*|22|
|輸入|\*|控制層 NAT IP|\*|\*|5557|
|輸出|\*|\*|\*|Webapp IP|\*|
|輸出|\*|\*|\*|SQL （服務標記）|\*|
|輸出|\*|\*|\*|儲存體（服務標記）|\*|
|輸出|\*|\*|\*|VirtualNetwork|\*|

使用下列 IP 位址將子網流量列入允許清單。 若是 SQL （中繼存放區）和儲存體（成品和記錄儲存體），您應該使用 Sql 和儲存體[服務標記](https://docs.microsoft.com/azure/virtual-network/security-overview#service-tags)。

|Azure Databricks 區域|服務|公用 IP|
|-----------------------|-------|---------|
|美國東部|控制層 NAT </br></br>Webapp|23.101.152.95/32 </br></br>40.70.58.221/32|
|美國東部 2|控制層 NAT </br></br>Webapp|23.101.152.95/32 </br></br>40.70.58.221/32|
|美國中北部|控制層 NAT </br></br>Webapp|23.101.152.95/32 </br></br>40.70.58.221/32|
|美國中部|控制層 NAT </br></br>Webapp|23.101.152.95/32 </br></br>40.70.58.221/32|
|美國中南部|控制層 NAT </br></br>Webapp|40.83.178.242/32 </br></br>40.118.174.12/32|
|美國西部|控制層 NAT </br></br>Webapp|40.83.178.242/32 </br></br>40.118.174.12/32|
|美國西部 2|控制層 NAT </br></br>Webapp|40.83.178.242/32 </br></br>40.118.174.12/32|
|加拿大中部|控制層 NAT </br></br>Webapp|40.85.223.25/32 </br></br>13.71.184.74/32|
|加拿大東部|控制層 NAT </br></br>Webapp|40.85.223.25/32 </br></br>13.71.184.74/32|
|英國西部|控制層 NAT </br></br>Webapp|51.140.203.27/32 </br></br>51.140.204.4/32|
|英國南部|控制層 NAT </br></br>Webapp|51.140.203.27/32 </br></br>51.140.204.4/32|
|西歐|控制層 NAT </br></br>Webapp|23.100.0.135/32 </br></br>52.232.19.246/32|
|北歐|控制層 NAT </br></br>Webapp|23.100.0.135/32 </br></br>52.232.19.246/32|
|印度中部|控制層 NAT </br></br>Webapp|104.211.89.81/32 </br></br>104.211.101.14/32|
|印度南部|控制層 NAT </br></br>Webapp|104.211.89.81/32 </br></br>104.211.101.14/32|
|印度西部|控制層 NAT </br></br>Webapp|104.211.89.81/32 </br></br>104.211.101.14/32|
|東南亞|控制層 NAT </br></br>Webapp|52.187.0.85/32 </br></br>52.187.145.107/32|
|東亞|控制層 NAT </br></br>Webapp|52.187.0.85/32 </br></br>52.187.145.107/32|
|澳洲東部|控制層 NAT </br></br>Webapp|13.70.105.50/32 </br></br>13.75.218.172/32|
|澳大利亞東南部|控制層 NAT </br></br>Webapp|13.70.105.50/32 </br></br>13.75.218.172/32|
|澳大利亞中部|控制層 NAT </br></br>Webapp|13.70.105.50/32 </br></br>13.75.218.172/32|
|澳大利亞中部 2|控制層 NAT </br></br>Webapp|13.70.105.50/32 </br></br>13.75.218.172/32|
|日本東部|控制層 NAT </br></br>Webapp|13.78.19.235/32 </br></br>52.246.160.72/32|
|日本西部|控制層 NAT </br></br>Webapp|13.78.19.235/32 </br></br>52.246.160.72/32|

## <a name="troubleshooting"></a>疑難排解

### <a name="workspace-launch-errors"></a>工作區啟動錯誤

在 [Azure Databricks 登入] 畫面上啟動自訂虛擬網路中的工作區失敗，發生下列錯誤： **"我們在建立工作區時發生錯誤。請確定自訂網路設定正確，然後再試一次。」**

此錯誤是因為網路設定不符合需求所造成。 當您建立工作區時，請確認您已遵循本主題中的指示。

### <a name="cluster-creation-errors"></a>叢集建立錯誤

@no__t 0Instances 無法連線：無法透過 SSH 連線到資源。 **

可能的原因：從控制層到背景工作角色的流量遭到封鎖。 藉由確保輸入安全性規則符合需求來進行修正。 如果您要部署到與內部部署網路連線的現有虛擬網路，請使用將您的 Azure Databricks 工作區連接到內部部署網路中提供的資訊來檢查您的設定。

**Unexpected 啟動失敗：設定叢集時，發生未預期的錯誤。如果問題持續發生，請重試並聯絡 Azure Databricks。內部錯誤訊息：放置節點時發生超時。**

可能的原因：從背景工作角色到 Azure 儲存體端點的流量遭到封鎖。 藉由確保輸出安全性規則符合需求來進行修正。 如果您使用的是自訂 DNS 伺服器，請同時檢查虛擬網路中的 DNS 伺服器狀態。

@no__t 0Cloud 提供者啟動失敗：設定叢集時發生雲端提供者錯誤。如需詳細資訊，請參閱 Azure Databricks 指南。Azure 錯誤代碼：AuthorizationFailed/InvalidResourceReference。 **

可能的原因：虛擬網路或子網路不再存在。 請確定虛擬網路和子網路都存在。

@no__t 0Cluster 終止。原因：Spark 啟動失敗：Spark 無法及時啟動。發生此問題的原因可能是故障的 Hive 中繼存放區、不正確 Spark 設定或初始化腳本無法運作。請參閱 Spark 驅動程式記錄檔以對此問題進行疑難排解，如果問題持續發生，請洽詢 Databricks。內部錯誤訊息：Spark 無法啟動：驅動程式無法及時啟動。 **

可能的原因：容器無法與裝載實例或 DBFS 儲存體帳戶交談。 藉由將 DBFS 儲存體帳戶的自訂路由新增至具有網際網路的下一個躍點的子網路來修正。

### <a name="notebook-command-errors"></a>筆記本命令錯誤

**命令沒有回應**

可能的原因：背景工作角色通訊遭到封鎖。 藉由確定輸入安全性規則符合需求來進行修正。

**筆記本工作流程失敗，發生例外狀況： databricks. WorkflowException： org. ConnectTimeoutException**

可能的原因：從背景工作角色到 Azure Databricks Webapp 的流量遭到封鎖。 藉由確定輸出安全性規則符合需求來進行修正。

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [使用 Azure Databrick 擷取、轉換和載入資料](databricks-extract-load-sql-data-warehouse.md)
