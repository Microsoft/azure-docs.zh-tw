---
title: SAP Hana on Azure (大型執行個體) 的支援案例 | Microsoft Docs
description: SAP Hana on Azure (大型執行個體) 的支援案例及其架構詳細資料
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/06/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 0e9d57c224150454677a03462368038ed8c63edf
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45576488"
---
# <a name="supported-scenarios-for-hana-large-instances"></a>HANA 大型執行個體的支援案例
本文件說明 HANA 大型執行個體 (HLI) 的支援案例以及其架構詳細資料。

>[!NOTE]
>如果這裡未提及您所需的案例，請連絡 Microsoft 服務管理小組來評估您的需求。
在繼續 HLI 單位佈建之前，請透過 SAP 或您的服務實作夥伴驗證設計。

## <a name="terms-and-definitions"></a>詞彙和定義
讓我們來了解文件中所用詞彙和定義。

- SID：HANA 系統的系統識別碼。
- HLI：Hana 大型執行個體。
- DR：災害復原網站。
- 標準 DR：具有僅用於 DR 用途專用資源的系統設定。
- 多用途 DR：DR 網站上設定為使用非生產環境的系統，搭配設定為用於 DR 事件的生產執行個體。 
- 單一 SID：已安裝一個執行個體的系統。
- 多重 SID：已設定多個執行個體的系統。 也稱為 MCOS 環境。


## <a name="overview"></a>概觀
HANA 大型執行個體支援各種架構以滿足您的業務需求。 下列清單涵蓋了案例及其設定詳細資料。 

衍生的架構設計完全來自基礎結構觀點，您必須諮詢 SAP 或您的實作夥伴以進行 HANA 部署。 如果未列出您的案例，請連絡 Microsoft 帳戶小組以檢閱架構並為您提供解決方案。

**這些架構完全遵循 TDI (訂製資料整合) 設計並受到 SAP 支援。**

本文件說明在各支援架構中兩個元件的詳細資訊：

- 乙太網路
- 儲存體

### <a name="ethernet"></a>乙太網路

每個佈建的伺服器都已預先設定乙太網路介面集。 以下是每個 HLI 單位上設定的乙太網路介面詳細資訊。

- **A**：此介面用於用戶端存取。
- **B**：此介面用於節點對節點通訊。 此介面已在所有伺服器上設定 (無論請求的拓撲為何)，但僅用於 
- 相應放大案例。
- **C**：此介面用於節點對儲存體連線能力。
- **D**：此介面用於 STONITH 設定的節點對 ISCSI 裝置連線。 只有在要求 HSR 安裝程式時才會設定此介面。  

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 節點對節點 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | STONITH |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 節點對節點 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | STONITH |

您可以根據在 HLI 單位上設定的拓撲使用介面。 例如，介面 "B" 已針對節點對節點通訊進行設定，這在您設定相應擴充拓撲後很實用。 在單一節點向上擴充設定的情況下，則不會使用這個介面。 檢閱您所需的案例 (在本文件稍後)，以取得介面使用方式的詳細資訊。 

如有需要您可以自行定義其他 NIC 卡。 不過，不能變更現有 NIC 上的設定。

>[!NOTE]
>您仍可能會發現實體介面或聯結等其他介面。 您應為使用案例考慮上述介面，其餘則可以忽略/或減少使用。

在獲指派兩個 IP 位址的情況下，單位的分配應該會看起來如下：

- 指派給乙太網路 "A" 的 IP 位址應該來自您向 Microsoft 提交的「伺服器 IP 集區」位址範圍。 此 IP 位址將用來在 OS 的 /etc/hosts 中進行維護。

- 指派給乙太網路 "C" 的 IP 位址應該用於對 NFS 的通訊。 因此，「不」需要在 etc/hosts 中維護這些位址，即可允許租用戶內的執行個體對執行個體流量。

就 HANA 系統複寫或 HANA 向外延展部署案例而言，並不適合使用有兩個指派之 IP 位址的刀鋒伺服器組態。 如果僅指派兩個 IP 位址且想要部署這類組態，請連絡 Azure 服務管理的 SAP HANA 在第三個 VLAN 中指派第三個 IP 位址。 在三個 NIC 連接埠上指派三個 IP 位址的 HANA 大型執行個體單位，適用下列使用規則：

- 指派給乙太網路 "A" 的 IP 位址應該來自您向 Microsoft 提交的「伺服器 IP 集區」位址範圍。 因此，此 IP 位址將不用來在 OS 的 /etc/hosts 中進行維護。

- 乙太網路 "B" 應專門用來在 etc/hosts 中進行維護，以供不同執行個體之間的通訊使用。 這些位址也會是需要在向外延展 HANA 組態中維護的 IP 位址，以作為 HANA 用於節點間組態的 IP 位址。

- 指派給乙太網路 "C" 的 IP 位址應該用於對 NFS 儲存體的通訊。 因此，不應該在 etc/hosts 中維護此類型的位址。

- 乙太網路 "D" 應專門用於存取前導者的 STONITH 裝置。 當您設定 HANA 系統複寫 (HSR)，且想要使用 SBD 型裝置在作業系統實現自動容錯移轉時，需要此介面。


### <a name="storage"></a>儲存體
儲存體已根據要求的拓撲預先設定。 磁碟區大小和裝載點會隨伺服器、SKU 和拓撲設定的數目而有不同。 檢閱您所需的案例 (稍後會在本文件中說明)，以取得詳細資訊。 如果需要更多儲存空間，您可以購買一 TB 增量。

>[!NOTE]
>裝載點 /usr/sap/<SID> 是 /hana/shared 裝載點的符號連結。


## <a name="supported-scenarios"></a>支援的案例

在架構圖表中，下列標記法可用於圖形：

![Legends.PNG](media/hana-supported-scenario/Legends.PNG)

下列清單顯示支援的案例：

1. 具有一個 SID 的單一節點
2. 單一節點 MCOS
3. 具有 DR (標準) 的單一節點
4. 具有 DR (多用途) 的單一節點
5. 具有 STONITH 的 HSR
6. 具有 DR (標準 / 多用途) 的 HSR 
7. 主機自動容錯移轉 (1+1) 
8. 具待命相應放大
9. 不具待命相應放大
10. 具有 DR 的相應放大



## <a name="1-single-node-with-one-sid"></a>1.具有一個 SID 的單一節點

此拓撲支援在相應增加設定中具有一個 SID 的一個節點。

### <a name="architecture-diagram"></a>架構圖表  

![Single-node-with-one-SID.png](media/hana-supported-scenario/Single-node-with-one-SID.png)

### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 已設定但未使用 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 已設定但未使用 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|/hana/shared/SID | HANA 安裝 | 
|/hana/data/SID/mnt00001 | 資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 記錄檔安裝 | 
|/hana/logbackups/SID | 重做記錄 |

### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。

## <a name="2-single-node-mcos"></a>2.單一節點 MCOS

此拓撲支援在相應增加設定中具有多個 SID 的一個節點。

### <a name="architecture-diagram"></a>架構圖表  

![single-node-mcos.png](media/hana-supported-scenario/single-node-mcos.png)

### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 已設定但未使用 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 已設定但未使用 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|/hana/shared/SID1 | 適用於 SID1 的 HANA 安裝 | 
|/hana/data/SID1/mnt00001 | 適用於 SID1 的資料檔案安裝 | 
|/hana/log/SID1/mnt00001 | 適用於 SID1 的記錄檔安裝 | 
|/hana/logbackups/SID1 | 適用於 SID1 的重做記錄 |
|/hana/shared/SID2 | 適用於 SID2 的 HANA 安裝 | 
|/hana/data/SID2/mnt00001 | 適用於 SID2 的資料檔案安裝 | 
|/hana/log/SID2/mnt00001 | 適用於 SID2 的記錄檔安裝 | 
|/hana/logbackups/SID2 | 適用於 SID2 的重做記錄 |

### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。
- 磁碟區大小發佈是根據記憶體中的資料庫大小而定。 請參閱[概觀和架構](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)一節，以了解多 SID 環境支援記憶體中哪種資料庫大小。

## <a name="3-single-node-with-dr-normal"></a>3.具有 DR (標準) 的單一節點
 
此拓撲支援在相應增加設定中具有一或多個 SID 的一個節點，其中主要 SID 會以儲存體為基礎複寫至 DR 網站。 在圖表中，只會在主要網站上描述單一 SID，但多 SID (MCOS) 也受支援。

### <a name="architecture-diagram"></a>架構圖表  

![Single-node-with-dr.png](media/hana-supported-scenario/Single-node-with-dr.png)

### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 已設定但未使用 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 已設定但未使用 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|/hana/shared/SID | 適用於 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於 SID 的重做記錄 |


### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。
- 針對 MCOS：磁碟區大小發佈是根據記憶體中的資料庫大小而定。 請參閱[概觀和架構](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)一節，以了解多 SID 環境支援記憶體中哪種資料庫大小。
- 在 DR 上：磁碟區和裝載點已在 DR HLI 單位上針對生產 HANA 執行個體安裝進行設定 (標示為「針對 HANA 安裝為必要」)。 
- 在 DR 上：從生產網站透過快照集複寫資料、記錄備份和共用磁碟區 (標示為「儲存體複寫」)。 這些磁碟區僅會在容錯移轉期間進行裝載。 如需詳細資訊，請參閱[災害復原容錯移轉程序](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#disaster-recovery-failover-procedure)文件。
- **SKU 類型一類別**的開機磁碟區已複寫到 DR 節點。


## <a name="4-single-node-with-dr-multipurpose"></a>4.具有 DR (多用途) 的單一節點
 
此拓撲支援在相應增加設定中具有一或多個 SID 的一個節點，其中主要 SID 會以儲存體為基礎複寫至 DR 網站。 在圖表中，只會在主要網站上描述單一 SID，但多 SID (MCOS) 也受支援。 在 DR 網站中，HLI 單位會用於 QA 執行個體，同時生產作業會從主要網站執行。 在 DR 容錯移轉 (或容錯移轉測試) 期間，會卸下 DR 網站的 QA 執行個體。

### <a name="architecture-diagram"></a>架構圖表  

![single-node-with-dr-multipurpose.png](media/hana-supported-scenario/single-node-with-dr-multipurpose.png)

### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 已設定但未使用 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 已設定但未使用 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|**在主要網站上**|
|/hana/shared/SID | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於生產 SID 的重做記錄 |
|**在 DR 網站上**|
|/hana/shared/SID | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/shared/QA-SID | 適用於 QA SID 的 HANA 安裝 | 
|/hana/data/QA-SID/mnt00001 | 適用於 QA SID 的資料檔案安裝 | 
|/hana/log/QA-SID/mnt00001 | 適用於 QA SID 的記錄檔安裝 |
|/hana/logbackups/QA-SID | 適用於 QA SID 的重做記錄 |

### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。
- 針對 MCOS：磁碟區大小發佈是根據記憶體中的資料庫大小而定。 請參閱[概觀和架構](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)一節，以了解多 SID 環境支援記憶體中哪種資料庫大小。
- 在 DR 上：磁碟區和裝載點已在 DR HLI 單位上針對生產 HANA 執行個體安裝進行設定 (標示為「針對 HANA 安裝為必要」)。 
- 在 DR 上：從生產網站透過快照集複寫資料、記錄備份和共用磁碟區 (標示為「儲存體複寫」)。 這些磁碟區僅會在容錯移轉期間進行裝載。 如需詳細資訊，請參閱[災害復原容錯移轉程序](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#disaster-recovery-failover-procedure)文件。 
- 在 DR 上：資料、記錄備份、記錄、QA 的共用磁碟區 (標示為「QA 執行個體安裝」) 已針對 QA 執行個體安裝進行設定。
- **SKU 類型一類別**的開機磁碟區已複寫到 DR 節點。

## <a name="5-hsr-with-stonith"></a>5.具有 STONITH 的 HSR
 
此拓撲支援 HANA 系統複寫 (HSR) 設定的兩個節點。 這項組態僅支援節點上的單一 HANA 執行個體。 也就是說，「不」支援 MCOS 案例。

**至目前為止，僅針對 SUSE 作業系統支援此架構。**


### <a name="architecture-diagram"></a>架構圖表  

![HSR-with-STONITH.png](media/hana-supported-scenario/HSR-with-STONITH.png)



### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 已設定但未使用 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 用於 STONITH |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 已設定但未使用 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 用於 STONITH |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|**在主要節點上**|
|/hana/shared/SID | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於生產 SID 的重做記錄 |
|**在次要節點上**|
|/hana/shared/SID | 適用於次要 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於次要 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於次要 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於次要 SID 的重做記錄 |

### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。
- 針對 MCOS：磁碟區大小發佈是根據記憶體中的資料庫大小而定。 請參閱[概觀和架構](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)一節，以了解多 SID 環境支援記憶體中哪種資料庫大小。
- STONITH：已針對 STONITH 安裝程式設定 SBD。 不過，STONITH 的使用是選擇性的。


## <a name="6-hsr-with-dr"></a>6.具有 DR 的 HSR
 
此拓撲支援 HANA 系統複寫 (HSR) 設定的兩個節點。 標準和多用途 DR 皆受支援。 這些組態僅支援節點上的單一 HANA 執行個體。 也就是說，這些組態「不」支援 MCOS 案例。

圖表中描繪了多用途案例，其中在 DR 網站，HLI 單位會用於 QA 執行個體，同時生產作業會從主要網站執行。 在 DR 容錯移轉 (或容錯移轉測試) 期間，會卸下 DR 網站的 QA 執行個體。 



### <a name="architecture-diagram"></a>架構圖表  

![HSR-with-DR.png](media/hana-supported-scenario/HSR-with-DR.png)

### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 已設定但未使用 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 用於 STONITH |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 已設定但未使用 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 用於 STONITH |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|**在主要網站的主要節點上**|
|/hana/shared/SID | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於生產 SID 的重做記錄 |
|**在主要網站的次要節點上**|
|/hana/shared/SID | 適用於次要 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於次要 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於次要 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於次要 SID 的重做記錄 |
|**在 DR 網站上**|
|/hana/shared/SID | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/shared/QA-SID | 適用於 QA SID 的 HANA 安裝 | 
|/hana/data/QA-SID/mnt00001 | 適用於 QA SID 的資料檔案安裝 | 
|/hana/log/QA-SID/mnt00001 | 適用於 QA SID 的記錄檔安裝 |
|/hana/logbackups/QA-SID | 適用於 QA SID 的重做記錄 |

### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。
- 針對 MCOS：磁碟區大小發佈是根據記憶體中的資料庫大小而定。 請參閱[概觀和架構](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)一節，以了解多 SID 環境支援記憶體中哪種資料庫大小。
- STONITH：已針對 STONITH 安裝程式設定 SBD。 不過，STONITH 的使用是選擇性的。
- 在 DR 上：**需要兩組儲存體磁碟區**，以用於主要和次要節點複寫。
- 在 DR 上：磁碟區和裝載點已在 DR HLI 單位上針對生產 HANA 執行個體安裝進行設定 (標示為「針對 HANA 安裝為必要」)。 
- 在 DR 上：從生產網站透過快照集複寫資料、記錄備份和共用磁碟區 (標示為「儲存體複寫」)。 這些磁碟區僅會在容錯移轉期間進行裝載。 如需詳細資訊，請參閱[災害復原容錯移轉程序](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#disaster-recovery-failover-procedure)文件。 
- 在 DR 上：資料、記錄備份、記錄、QA 的共用磁碟區 (標示為「QA 執行個體安裝」) 已針對 QA 執行個體安裝進行設定。
- **SKU 類型一類別**的開機磁碟區已複寫到 DR 節點。


## <a name="7-host-auto-failover-11"></a>7.主機自動容錯移轉 (1+1)
 
此拓撲支援主機自動容錯移轉設定中的兩個節點。 其中一個節點具有主要/背景工作角色，另一個節點為待命。 **SAP 僅針對 S/4 HANA 支援此案例。** 請參閱 OSS 說明 “[2408419 - SAP S/4HANA - 多節點支援](https://launchpad.support.sap.com/#/notes/2408419)” 以取得詳細資料。



### <a name="architecture-diagram"></a>架構圖表  

![sca](media/hana-supported-scenario/scaleup-with-standby.png)

### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 節點對節點通訊 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 節點對節點通訊 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|**在主要和待命節點上**|
|HANA/shared | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於生產 SID 的重做記錄 |



### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。
- 待命：磁碟區和裝載點已在待命單位上針對 HANA 執行個體安裝進行設定 (標示為「針對 HANA 安裝為必要」)。
 

## <a name="8-scale-out-with-standby"></a>8.具待命相應放大
 
此拓撲支援相應放大組態中的多個節點。 其中一個節點具有主要角色，一或多個節點具有背景工作角色，一或多個節點為待命。 不過，在任何指定的時間點都只能有一個主要節點。


### <a name="architecture-diagram"></a>架構圖表  

![scaleout-nm-standby.png](media/hana-supported-scenario/scaleout-nm-standby.png)

### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 節點對節點通訊 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 節點對節點通訊 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|**在主要、背景工作角色和待命節點上**|
|HANA/shared | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於生產 SID 的重做記錄 |


## <a name="9-scale-out-without-standby"></a>9.不具待命相應放大
 
此拓撲支援相應放大組態中的多個節點。 其中一個節點具有主要角色，一或多個節點具有背景工作角色。 不過，在任何指定的時間點都只能有一個主要節點。


### <a name="architecture-diagram"></a>架構圖表  

![scaleout-nm.png](media/hana-supported-scenario/scaleout-nm.png)


### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 節點對節點通訊 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 節點對節點通訊 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|**在主要和背景工作角色節點上**|
|HANA/shared | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於生產 SID 的重做記錄 |


### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。

## <a name="10-scale-out-with-dr"></a>10.具有 DR 的相應放大
 
此拓撲支援具 DR 相應放大中的多個節點。 標準和多用途 DR 皆受支援。 圖表中僅描繪單一用途 DR。 您可以要求此拓撲，不論是否具有待命節點。


### <a name="architecture-diagram"></a>架構圖表  

![scaleout-with-dr.png](media/hana-supported-scenario/scaleout-with-dr.png)


### <a name="ethernet"></a>乙太網路
下列網路介面已預先設定：

| NIC 邏輯介面 | SKU 類型 | 以 SUSE 作業系統命名 | 以 RHEL 作業系統命名 | 使用案例|
| --- | --- | --- | --- | --- |
| 具有使用  | 類型一 | eth0.tenant | eno1.tenant | 用戶端到 HLI |
| b | 類型一 | eth2.tenant | eno3.tenant | 節點對節點通訊 |
| C | 類型一 | eth1.tenant | eno2.tenant | 節點對儲存體 |
| D | 類型一 | eth4.tenant | eno4.tenant | 已設定但未使用 |
| 具有使用  | 類型二 | vlan<tenantNo> | team0.tenant | 用戶端到 HLI |
| b | 類型二 | vlan<tenantNo+2> | team0.tenant+2 | 節點對節點通訊 |
| C | 類型二 | vlan<tenantNo+1> | team0.tenant+1 | 節點對儲存體 |
| D | 類型二 | vlan<tenantNo+3> | team0.tenant+3 | 已設定但未使用 |

### <a name="storage"></a>儲存體
下列裝載點已預先設定：

| 裝載點 | 使用案例 | 
| --- | --- |
|**在主要節點上**|
|HANA/shared | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 
|/hana/logbackups/SID | 適用於生產 SID 的重做記錄 |
|**在 DR 節點上**|
|HANA/shared | 適用於生產 SID 的 HANA 安裝 | 
|/hana/data/SID/mnt00001 | 適用於生產 SID 的資料檔案安裝 | 
|/hana/log/SID/mnt00001 | 適用於生產 SID 的記錄檔安裝 | 


### <a name="key-considerations"></a>主要考量
- /usr/sap/SID 是 /hana/shared/SID 的符號連結。
-  在 DR 上：磁碟區和裝載點已在 DR HLI 單位上針對生產 HANA 執行個體安裝進行設定 (標示為「針對 HANA 安裝為必要」)。 
- 在 DR 上：從生產網站透過快照集複寫資料、記錄備份和共用磁碟區 (標示為「儲存體複寫」)。 這些磁碟區僅會在容錯移轉期間進行裝載。 如需詳細資訊，請參閱[災害復原容錯移轉程序](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery#disaster-recovery-failover-procedure)文件。 
- **SKU 類型一類別**的開機磁碟區已複寫到 DR 節點。


## <a name="next-steps"></a>後續步驟
- 請參閱 HLI 的[基礎結構和連線能力](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-infrastructure-connectivity)
- 請參閱 HLI 的[高可用性和災害復原](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-high-availability-disaster-recovery)