---
title: 包含檔案
description: 包含檔案
services: virtual-machines
author: jpconnock
ms.service: virtual-machines
ms.topic: include
ms.date: 05/18/2018
ms.author: jeconnoc
ms.custom: include file
ms.openlocfilehash: d1a6ff8dbd17d2792709a1ce065bcf793154e585
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37780667"
---
# <a name="platform-supported-migration-of-iaas-resources-from-classic-to-azure-resource-manager"></a>平台支援的 IaaS 資源移轉 (從傳統移轉至 Azure Resource Manager)
本文會說明如何將 IaaS 資源從傳統部署模型遷移至 Resource Manager 部署模型，並詳述如何使用虛擬網路站對站閘道，從並存於訂用帳戶中的兩個部署模型連線到資源。 您可以進一步了解 [Azure Resource Manager 功能和優點](../articles/azure-resource-manager/resource-group-overview.md)。 

## <a name="goal-for-migration"></a>移轉目標
Resource Manager 除了可讓您透過範本部署複雜的應用程式之外，還可使用 VM 擴充功能來設定虛擬機器，並且納入了存取管理和標記功能。 Azure Resource Manager 還將虛擬機器的可調整、平行部署納入可用性設定組中。 新部署模型也針對計算、網路及儲存體個別提供生命週期管理功能。 最後，將焦點放在藉由在虛擬網路中強制使用虛擬機器的方式，預設啟用安全性。

在 Azure Resource Manager 之下，針對來自傳統部署模型的幾乎所有功能，都有提供計算、網路及儲存體支援。 若要享有 Azure Resource Manager 新功能的好處，您可以從「傳統」部署模型移轉現有的部署。

## <a name="supported-resources-for-migration"></a>支援移轉的資源
移轉期間支援這些傳統 IaaS 資源

* 虛擬機器
* 可用性設定組 (Availability Sets)
* 雲端服務及虛擬機器
* 儲存體帳戶
* 虛擬網路
* VPN 閘道
* Express Route 閘道 _(僅與虛擬網路位於相同的訂用帳戶中)_
* 網路安全性群組
* 路由表
* 保留的 IP

## <a name="supported-scopes-of-migration"></a>支援的移轉範圍
有 4 種不同方式可完成計算、網路和儲存體資源移轉：

* [移轉 (不在虛擬網路中的) 虛擬機器](#migration-of-virtual-machines-not-in-a-virtual-network)
* [移轉 (虛擬網路中的) 虛擬機器](#migration-of-virtual-machines-in-a-virtual-network)
* [移轉儲存體帳戶](#migration-of-storage-accounts)
* [移轉未連結的資源](#migration-of-unattached-resources)

### <a name="migration-of-virtual-machines-not-in-a-virtual-network"></a>移轉 (不在虛擬網路中的) 虛擬機器
在 Resource Manager 部署模型中，預設會針對應用程式強制執行安全性。 在 Resource Manager 模型中，所有 VM 都必須在虛擬網路內。 Azure 平台會在移轉過程中將 VM 重新啟動 (`Stop`、`Deallocate` 及 `Start`)。 您有兩個選項可將虛擬機器將移轉至虛擬網路︰

* 您可以要求平台建立新的虛擬網路，然後將虛擬機器移轉到新的虛擬網路。
* 您也可以將虛擬機器移轉到 Resource Manager 中的現有虛擬網路。

> [!NOTE]
> 在此移轉範圍內，移轉期間可能會有一段時間不允許進行管理平面和資料平面作業。
>

### <a name="migration-of-virtual-machines-in-a-virtual-network"></a>移轉 (虛擬網路中的) 虛擬機器
就大多數 VM 組態而言，只有中繼資料會在「傳統」部署模型與 Resource Manager 部署模型之間移轉。 基礎 VM 會在相同硬體、相同網路上，使用相同的儲存體來執行。 進行移轉時，可能會有某一段時間不允許進行管理平面作業。 不過，資料平面會繼續運作。 也就是說，在 VM (傳統) 上執行的應用程式不會在移轉期間造成停機時間。

目前不支援下列組態。 如果未來新增支援，則此組態中的某些 VM 可能會造成停機時間 (將會經歷停止、解除配置及重新啟動 VM 的作業)。

* 您在單一雲端服務中有多個可用性設定組。
* 您在單一雲端服務中有一或多個可用性設定組，以及不在可用性設定組中的 VM。

> [!NOTE]
> 在此移轉範圍內，移轉期間可能會有一段時間不允許進行管理平面作業。 針對先前所述的某些組態，將會發生資料平面停機時間。
>

### <a name="migration-of-storage-accounts"></a>移轉儲存體帳戶
為了讓移轉順暢進行，您可以在傳統儲存體帳戶中部署 Resource Manager VM。 透過這項功能，您就可以移轉計算和網路資源，且應該不受儲存體帳戶限制。 將「虛擬機器」和「虛擬網路」移轉過去之後，您必須將儲存體帳戶移轉過去，才能完成移轉程序。

如果您的儲存體帳戶沒有任何相關聯的磁碟或虛擬機器資料，而且只有 blob、檔案、資料表和佇列，則移轉至 Azure Resource Manager 可以透過獨立移轉來完成，無須相依項目。

> [!NOTE]
> Resource Manager 部署模型並沒有「傳統」映像和磁碟的概念。 移轉儲存體帳戶時，「傳統」映像和磁碟不會顯示在 Resource Manager 堆疊中，但是作為基礎的 VHD 會繼續留在儲存體帳戶中。
>

### <a name="migration-of-unattached-resources"></a>移轉未連結的資源
沒有相關聯磁碟或虛擬機器資料的儲存體帳戶可獨立進行遷移。

也可以獨立遷移未連結至任何虛擬機器和虛擬網路的網路安全性群組、路由表和保留的 IP。

<br>

## <a name="unsupported-features-and-configurations"></a>不支援的功能和組態
目前尚有一些功能和組態未受到支援；下面幾節將說明面對此狀況的建議做法。

### <a name="unsupported-features"></a>不支援的功能
目前不支援下列功能。 您可以視需要移除這些設定、移轉 VM，然後再於 Resource Manager 部署模型中重新啟用這些設定。

| 資源提供者 | 功能 | 建議 |
| --- | --- | --- |
| 計算 | 未關聯的虛擬機器磁碟。 | 移轉儲存體帳戶時，將會移轉這些磁碟背後的 VHD blob |
| 計算 | 虛擬機器映像。 | 移轉儲存體帳戶時，將會移轉這些磁碟背後的 VHD blob |
| 網路 | 端點 ACL。 | 移除端點 ACL，然後重試移轉。 |
| 網路 | 應用程式閘道 | 在開始移轉前移除應用程式閘道，然後在移轉完成後重新建立應用程式閘道。 |
| 網路 | 使用 VNet 對等互連的虛擬網路。 | 將虛擬網路移轉至 Resource Manager，然後對等互連。 深入了解 [VNet 對等互連](../articles/virtual-network/virtual-network-peering-overview.md)。 |

### <a name="unsupported-configurations"></a>不支援的組態
目前不支援下列組態。

| 服務 | 組態 | 建議 |
| --- | --- | --- |
| Resource Manager |傳統資源的「角色型存取控制」(RBAC) |由於資源的 URI 在移轉後會經過修改，因此建議您規劃需要在移轉後進行的 RBAC 原則更新。 |
| 計算 |與 VM 關聯的多個子網路 |將子網路組態更新為只參考一個子網路。 這可能要求您從 VM 移除次要 NIC (其參考另一個子網路)，並在移轉完成後重新連結它。 |
| 計算 |隸屬於虛擬網路但未獲指派明確子網路的虛擬機器。 |您可以選擇刪除此 VM。 |
| 計算 |具有警示、自動調整原則的虛擬機器 |移轉會進行到完成，但會捨棄這些設定。 強烈建議您在執行移轉前先評估您的環境。 或者，您也可以在移轉完成之後重新設定警示設定。 |
| 計算 |XML VM 擴充功能 (BGInfo 1.*、Visual Studio Debugger、Web Deploy 及遠端偵錯) |不支援此做法。 建議您從虛擬機器中移除這些擴充功能以繼續進行移轉，否則系統會在移轉過程中自動卸除它們。 |
| 計算 |使用進階儲存體進行開機診斷 |先停用 VM 的「開機診斷」功能，再繼續進行移轉。 您可以在移轉完成之後，於 Resource Manager 堆疊中重新啟用開機診斷。 此外，應該將用於快照和序列記錄檔的 Blob 刪除，這樣您就不再需要支付這些 Blob 的費用。 |
| 計算 | 包含 Web 角色/背景工作角色的雲端服務 | 目前不支援。 |
| 計算 | 包含一個以上可用性設定組或多個可用性設定組的雲端服務。 |目前不支援。 請先將虛擬機器移至相同可用性設定組，然後再移轉。 |
| 計算 | 具 Azure 資訊安全中心擴充功能的 VM | 「Azure 資訊安全中心」會自動在「虛擬機器」上安裝擴充功能，以監視其安全性並引發警示。 如果已在訂用帳戶上啟用「Azure 資訊安全中心」原則，通常就會自動安裝這些擴充功能。 若要移轉虛擬機器，請停用訂用帳戶上的資訊安全中心原則，這將會從虛擬機器移除資訊安全中心監視擴充功能。 |
| 計算 | 具備份或快照集擴充功能的 VM | 這些擴充功能都安裝在使用 Azure 備份功能設定的虛擬機器上。 雖然不支援這些 VM 的移轉，但是可依照[這裡](https://docs.microsoft.com/azure/virtual-machines/windows/migration-classic-resource-manager-faq#vault)的指引，保留在移轉之前所執行的備份。  |
| 網路 |包含虛擬機器和 Web 角色/背景工作角色的虛擬網路 |目前不支援。 請先將 Web/背景工作角色移至他們自己的虛擬網路，然後再移轉。 在移轉傳統虛擬網路之後，移轉的 Azure Resource Manager 虛擬網路可以和傳統虛擬網路對等互連，達到類似之前的組態。|
| 網路 | 傳統 ExpressRoute 線路 |目前不支援。 在開始 IaaS 移轉之前，需要將這些線路移轉至 Azure Resource Manager。 若要深入了解，請參閱[將 ExpressRoute 線路從傳統部署模型移至 Resource Manager 部署模型](../articles/expressroute/expressroute-move.md)。|
| Azure App Service |包含 App Service 環境的虛擬網路 |目前不支援。 |
| Azure HDInsight |包含 HDInsight 服務的虛擬網路 |目前不支援。 |
| Microsoft Dynamics 週期服務 |包含「Dynamics 週期服務」所管理之虛擬機器的虛擬網路 |目前不支援。 |
| Azure AD 網域服務 |包含 Azure AD 網域服務的虛擬網路 |目前不支援。 |
| Azure RemoteApp |包含 Azure RemoteApp 部署的虛擬網路 |目前不支援。 |
| Azure API 管理 |包含 Azure API 管理部署的虛擬網路 |目前不支援。 若要移轉 IaaS VNET，請變更屬於無停機作業的 API 管理部署的 VNET。 |
