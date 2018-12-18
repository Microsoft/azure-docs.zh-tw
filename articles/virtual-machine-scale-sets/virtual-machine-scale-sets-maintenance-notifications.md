---
title: Azure 中虛擬機器擴展集的維護通知 | Microsoft Docs
description: 檢視 Azure 中虛擬機器擴展集的維護通知，並啟動自助維護。
services: virtual-machine-scale-sets
documentationcenter: ''
author: shants123
editor: ''
tags: azure-service-management,azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/09/2018
ms.author: shants
ms.openlocfilehash: 82a3ce9f899e94a1cc737f2ca2dc1dc79688a224
ms.sourcegitcommit: 7b845d3b9a5a4487d5df89906cc5d5bbdb0507c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42142093"
---
# <a name="planned-maintenance-notifications-for-virtual-machine-scale-sets"></a>虛擬機器擴展集的計劃性維護通知

為了提升虛擬機器 (VM) 之主機基礎結構的可靠性、效能和安全性，Azure 會定期執行更新。 更新可能包括修補裝載環境，或將硬體升級與解除委任。 大多數更新不會影響所裝載的 VM。 不過，在下列情況下，更新則會影響 VM：

- 如果維護不需要重新開機，Azure 會在主機更新時，使用就地移轉來暫停 VM。 套用不需要重新開機的維護作業時，會依容錯網域逐一套用。 如果收到任何警告健康情況訊號，就會停止進度。

- 如果維護需要重新開機，您將會收到說明已規劃何時進行維護的通知。 在這些情況下，系統會提供您一個時間範圍，供您在最適合的時間自行啟動維護。


預定進行的維護作業若需要重新開機，會排定在不同波段。 每一個波段都有不同的範圍 (區域)：

- 波段開始時會傳送通知給客戶。 根據預設，通知會傳送給訂用帳戶擁有者和共同擁有者。 您可以使用 Azure [活動記錄警示](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)，對通知新增收件者和傳訊選項，例如電子郵件、SMS 和 Webhook。  
- 隨著通知會提供一個「自助時段」。 在此時段，您可以找出該波段包含哪些 VM。 您可以根據自己的排程需求，主動啟動維護。
- 在自助期間之後，「排定維護期間」隨即開始。 在此時段的某個時間點，Azure 會為您的 VM 排定並套用必要的維護。 

之所以要有兩個時段，目的是要讓您在知道 Azure 何時會自動啟動維護的同時，有足夠的時間啟動維護並重新啟動 VM。


您可以使用 Azure 入口網站、PowerShell、REST API 和 Azure CLI 來查詢虛擬機器擴展集 VM 的維護時段，並啟動自助維護。

  
## <a name="should-you-start-maintenance-during-the-self-service-window"></a>您是否應該在自助期間開始維護？  

下列指導方針可協助您決定是否要在您選擇的時間啟動維護。

> [!NOTE] 
> 自助式維護可能不適用於所有的虛擬機器。 若要判斷您的 VM 是否可以使用主動式重新部署，請在維護狀態中尋找 [立即開始]。 目前，Azure 雲端服務 (Web/背景工作角色) 和 Azure Service Fabric 無法使用自助維護。


針對使用「可用性設定組」的部署，不建議使用自助維護。 可用性設定組是在任何時間都只有一個更新網域會受影響的高可用性設定。 針對可用性設定組：

- 讓 Azure 觸發維護工作。 針對需要重新開機的維護，會依更新網域逐一進行維護。 更新網域不一定會依序接收維護。 更新網域之間會有 30 分鐘的暫停。
- 如果擔心會暫時損失某些容量 (1/更新網域計數)，在維護期間配置額外的執行個體，即可輕鬆補足該損失。
- 針對不需要重新開機的維護，系統會在容錯網域層級套用更新。 
    
在下列情況，**請勿**使用自助式維護： 

- 如果您經常關閉 VM (不論是手動、使用 DevTest Labs、使用自動關機，還是依照排程)。 在這些情況下，自助維護可能會將維護狀態還原，而導致產生額外的停機時間。
- 在您知道將會在維護波段結束前被刪除的短期存留虛擬機器上。 
- 工作負載有大量狀態儲存在您想要於更新後進行維護的本機 (暫時) 磁碟中。 
- 如果您經常調整 VM 的大小。 此情況可能會將維護狀態還原。 
- 如果採用的排定事件可讓工作負載在維護關機開始之前 15 分鐘便主動容錯移轉或正常關機。

如果您打算在排程維護階段連續不中斷地執行 VM，而且上述所有反指標都不適用，則請**使用**自助維護。 

在下列情況下，最好使用自助維護：

- 您需要向管理階層或客戶傳達確切的維護時段。 
- 您需要在指定的日期前完成維護。 
- 您需要控制維護序列 (例如在多層式應用程式中) 以確保安全復原。
- 在兩個更新網域之間，您需要 30 分鐘以上的虛擬機器復原時間。 若要控制更新網域之間的時間，您必須在虛擬機器上以一次一個更新網域的方式觸發維護。

 
## <a name="view-virtual-machine-scale-sets-that-are-affected-by-maintenance-in-the-portal"></a>在入口網站中檢視受維護影響的虛擬機器擴展集

在排定一波計劃性維護之後，您就可以使用 Azure 入口網站，檢視受即將到來的一波維護影響的虛擬機器擴展集清單。 

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側功能表中，選取 [所有服務]，然後選取 [虛擬機器擴展集]。
3. 在 [虛擬機器擴展集] 底下，選取 [編輯資料行] 以開啟可用的資料行清單。
4. 在 [可用的資料行] 區段中，選取 [自助維護]，然後將其移至 [選取的資料行] 清單。 選取 [套用] 。  

    若要讓 [自助維護] 項目變得更容易被找到，您可以將 [可用的資料行] 區段中的下拉式清單從 [全部] 變更為 [屬性]。

此時，[自助維護] 資料行就會出現在虛擬機器擴展集的清單中。 每個虛擬機器擴展集可以具備下列自助式維護資料行的其中一個值：

| 值 | 說明 |
|-------|-------------|
| 是 | 虛擬機器擴展集中至少有一個 VM 處於自助服務時段內。 您可以在此自助服務期間隨時開始維護。 | 
| 否 | 受影響之虛擬機器擴展集中沒有任何 VM 處於自助服務時段內。 | 
| - | 您的虛擬機器擴展集不在這一波計劃性維護中。| 

## <a name="notification-and-alerts-in-the-portal"></a>入口網站中的通知和警示

Azure 會將電子郵件傳送至訂用帳戶擁有者和共同擁有者群組，來傳達計劃性維護排程。 您可以藉由建立活動記錄警示，為此通訊新增收件者和管道。 如需詳細資訊，請參閱[使用 Azure 活動記錄監視訂用帳戶活動](../monitoring-and-diagnostics/monitoring-overview-activity-logs.md)。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 在左側功能表中，選取 [監視]。 
3. 在 [監視 - 警示 (傳統)] 窗格中，選取 [+ 新增活動記錄警示]。
4. 在 [新增活動記錄警示] 頁面上，選取或輸入所要求的資訊。 在 [準則] 中，請確定您已設定下列值：
   - **事件類別目錄**：選取 [服務健康狀態]。
   - **服務**：選取 [虛擬機器擴展集與虛擬機器]。
   - **類型**：選取 [計劃性維護]。 
    
若要深入了解如何設定活動記錄警示，請參閱[建立活動記錄警示](../monitoring-and-diagnostics/monitoring-activity-log-alerts.md)
    
    
## <a name="start-maintenance-on-your-virtual-machine-scale-set-from-the-portal"></a>從入口網站開始維護您的虛擬機器擴展集

您可以在虛擬機器擴展集概觀中，看到更多與維護相關的詳細資料。 如果虛擬機器擴展集中至少有一個 VM 包含在這一波計劃性維護中，在頁面頂端附近就會新增一個新的通知功能區。 選取該通知功能區，即可移至 [維護] 頁面。 

在 [維護] 頁面上，您可以查看哪些 VM 執行個體受到計劃性維護影響。 若要啟動維護，請選取與受影響 VM 對應的核取方塊。 然後，選取 [啟動維護]。

啟動維護之後，虛擬機器擴展集中受影響的 VM 就會進行維護而暫時無法使用。 如果您錯過自助服務時段，仍可以看到 Azure 將維護虛擬機器擴展集的時間範圍。
 
## <a name="check-maintenance-status-by-using-powershell"></a>使用 PowerShell 來檢查維護狀態

您可以使用 Azure Powershell 來查看虛擬機器擴展集中的 VM 已排定在何時進行維護。 藉由使用 [Get-AzureRmVmss](https://docs.microsoft.com/powershell/module/azurerm.compute/get-azurermvmss) Cmdlet，只要使用 `-InstanceView` 參數，即可取得計劃性維護資訊。
 
只有在已計劃進行維護的情況下，才會傳回維護資訊。 如果未排定任何會影響 VM 執行個體的維護，此 Cmdlet 就不會傳回任何維護資訊。 

```powershell
Get-AzureRmVmss -ResourceGroupName rgName -VMScaleSetName vmssName -InstanceId id -InstanceView
```

以下是 **MaintenanceRedeployStatus** 下傳回的屬性： 
| 值 | 說明   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | 指出您此時是否可以在 VM 上啟動維護。 ||
| PreMaintenanceWindowStartTime         | 您可以在 VM 上起始維護作業的維護自助時段開始時間。 ||
| PreMaintenanceWindowEndTime           | 您可以在 VM 上起始維護作業的維護自助時段結束時間。 ||
| MaintenanceWindowStartTime            | Azure 會在 VM 上起始維護作業的已排定維護開始時間。 ||
| MaintenanceWindowEndTime              | Azure 會在 VM 上起始維護作業的已排定維護結束時間。 ||
| LastOperationResultCode               | 上次嘗試在 VM 上起始維護作業的結果。 ||



### <a name="start-maintenance-on-your-vm-instance-by-using-powershell"></a>使用 PowerShell 在 VM 執行個體上啟動維護

如果將 **IsCustomerInitiatedMaintenanceAllowed** 設定為 **true**，便可在虛擬機器上啟動維護。 請使用 [Set-AzureRmVmss](/powershell/module/azurerm.compute/set-azurermvmss) Cmdlet 搭配 `-PerformMaintenance` 參數。

```powershell
Set-AzureRmVmss -ResourceGroupName rgName -VMScaleSetName vmssName -InstanceId id -PerformMaintenance 
```

## <a name="check-maintenance-status-by-using-the-cli"></a>使用 CLI 來檢查維護狀態

您可以使用 [az vmss list-instances](/cli/azure/vmss?view=azure-cli-latest#az-vmss-list-instances) 來檢視計劃性維護資訊。
 
只有在已計劃進行維護的情況下，才會傳回維護資訊。 如果未排定任何會影響 VM 執行個體的維護，此命令就不會傳回任何維護資訊。 

```azure-cli
az vmss list-instances -g rgName -n vmssName --expand instanceView
```

以下是針對每個 VM 執行個體在 **MaintenanceRedeployStatus** 下傳回的屬性： 
| 值 | 說明   |
|-------|---------------|
| IsCustomerInitiatedMaintenanceAllowed | 指出您此時是否可以在 VM 上啟動維護。 ||
| PreMaintenanceWindowStartTime         | 您可以在 VM 上起始維護作業的維護自助時段開始時間。 ||
| PreMaintenanceWindowEndTime           | 您可以在 VM 上起始維護作業的維護自助時段結束時間。 ||
| MaintenanceWindowStartTime            | Azure 會在 VM 上起始維護作業的已排定維護開始時間。 ||
| MaintenanceWindowEndTime              | Azure 會在 VM 上起始維護作業的已排定維護結束時間。 ||
| LastOperationResultCode               | 上次嘗試在 VM 上起始維護作業的結果。 ||


### <a name="start-maintenance-on-your-vm-instance-by-using-the-cli"></a>使用 CLI 在 VM 執行個體上啟動維護

如果將 `IsCustomerInitiatedMaintenanceAllowed` 設定為 **true**，下列呼叫就會在 VM 執行個體上起始維護：

```azure-cli
az vmss perform-maintenance -g rgName -n vmssName --instance-ids id
```

## <a name="faq"></a>常見問題集

**問：為什麼需要立即重新啟動我的 VM？**

**答：** 雖然對 Azure 平台進行的大多數更新和升級都不會影響 VM 可用性，但在某些情況下，我們無法避免重新啟動裝載於 Azure 中的 VM。 我們已經累積數個變更，這些變更需要我們重新啟動伺服器，而會導致 VM 重新啟動。

**問：如果我依照建議使用可用性設定組來獲得高可用性，是否安全？**

**答：** 部署在可用性設定組或虛擬機器擴展集內的虛擬機器會使用更新網域。 執行維護時，Azure 會遵守更新網域條件約束，而不會重新啟動來自不同更新網域 (在相同的可用性設定組內) 的 VM。 Azure 也會至少等候 30 分鐘，然後才移至下一個 VM 群組。 

如需有關高可用性的詳細資訊，請參閱 [Azure 中虛擬機器的區域和可用性](../virtual-machines/windows/regions-and-availability.md)。

**問：我如何收到計劃性維護的相關通知？**

**答：** 規劃的維護是從對一或多個 Azure 區域設定排程開始。 之後，電子郵件通知會傳送至訂用帳戶擁有者 (每個訂用帳戶一封電子郵件)。 您可以藉由使用活動記錄警示，為此通知新增管道和收件者。 如果您將 VM 部署至已排定計劃性維護的區域，則不會收到通知。 請改為檢查 VM 的維護狀態。

**問：我在入口網站、PowerShell 或 CLI 中都看不到任何計劃性維護的指示。問題出在哪裡？**

**答：** 在計劃性維護波段期間，只會針對受計劃性維護影響的 VM 提供計劃性維護相關資訊。 如果您沒有看到資料，則該維護波段可能已經結束 (或未開始)，或您的 VM 可能已經裝載在已更新的伺服器上。

**問：是否有方法可以確切知道我的 VM 何時會受到影響？**

**答：** 設定排程時，我們會定義一個數天的時間範圍。 此期間內伺服器 (和虛擬機器) 的確切順序不明。 如果您想要知道 VM 的確切更新時間，可以使用[已排定事件](../virtual-machines/windows/scheduled-events.md)。 使用已排定事件時，您可以從 VM 內進行查詢，並在 VM 重新啟動之前收到 15 分鐘通知。

**問：重新啟動我的 VM 需要多久時間？**

**答：** 視您的 VM 大小而定，在自助維護時段期間，重新啟動最長可能需要數分鐘的時間。 在 Azure 於排程維護時段起始的重新啟動期間，重新啟動通常需要大約 25 分鐘的時間。 如果您使用雲端服務 (Web/背景工作角色)、虛擬機器擴展集或可用性設定組，則在排程維護時段期間，每個 VM 群組 (更新網域) 之間會有 30 分鐘的時間。 

**問：我在我的 VM 上看不到任何維護資訊。哪裡發生錯誤？**

**答：** 有幾個原因會導致您可能在 VM 上看不到任何維護資訊：
   - 您使用標示為「Microsoft 內部」的訂用帳戶。
   - 您的 VM 未排定進行維護。 可能是維護波段已結束、取消或修改，所以您的 VM 不再受到它的影響。
   - 您尚未將 [維護] 資料行新增至您的虛擬機器清單檢視。 雖然我們已將此資料行新增至預設檢視，但如果您將檢視設定為顯示非預設資料行，則必須手動將 [維護] 資料行新增至您的 VM 清單檢視。

**問：我的 VM 第二次排程進行維護。原因為何？**

**答：** 在數個使用案例中，是將您的 VM 排定成在您已經完成維護並重新部署之後進行維護：
   - 我們已取消維護，並且使用不同的裝載重新啟動它。 可能是我們已偵測到發生錯誤的承載，而只是需要部署額外承載。
   - 因為硬體故障，您的 VM 已*服務修復*到另一個節點。
   - 您已經選擇停止 (解除配置)，然後重新啟動 VM。
   - 您已經為 VM 開啟**自動關機**。

## <a name="next-steps"></a>後續步驟

了解如何使用[已排定事件](../virtual-machines/windows/scheduled-events.md)，從 VM 內註冊來取得維護事件。
