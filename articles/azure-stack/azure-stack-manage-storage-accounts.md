---
title: 管理 Azure Stack 儲存體帳戶 | Microsoft Docs
description: 了解如何尋找、管理、復原及回收 Azure Stack 儲存體帳戶
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: PowerShell
ms.topic: get-started-article
ms.date: 09/28/2018
ms.author: mabrigg
ms.reviewer: xiaofmao
ms.openlocfilehash: 35c15613192ac12a7d4c64cbe28f62200724d311
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452270"
---
# <a name="manage-storage-accounts-in-azure-stack"></a>在 Azure Stack 中管理儲存體帳戶
了解如何在 Azure Stack 中管理儲存體帳戶，以便根據業務需求來尋找、復原及回收儲存體容量。

## <a name="find"></a>尋找儲存體帳戶
區域中的儲存體帳戶清單可在 Azure Stack 中，透過下列方式檢視：

1. 登入[管理入口網站](https://adminportal.local.azurestack.external)。

2. 在 [系統管理]底下選取 [所有服務]  >  [區域管理]。

3. 從 [資源提供者] 清單中選取 [儲存體]。
   
   ![儲存體資源提供者](media/azure-stack-manage-storage-accounts/image1.png)

5. 在 [儲存體] 中選取 [儲存體帳戶]。
   
   ![](media/azure-stack-manage-storage-accounts/image2.png)
   
   所顯示的刀鋒視窗就是在該區域中的儲存體帳戶清單。
   
   ![](media/azure-stack-manage-storage-accounts/image4.png)

預設會顯示前 10 個帳戶。 您可以按一下清單底部的 [載入更多] 連結，選擇擷取更多項目。

或

如果您只想看到特定儲存體帳戶，可以只**篩選並擷取相關的帳戶**。


**篩選帳戶：**

1. 選取窗格頂端的 [篩選]。
2. 在 [篩選] 窗格上，您可以指定 [帳戶名稱]、\*\*[訂用帳戶識別碼] 或 [狀態]，以微調要顯示的儲存體帳戶清單。 請適當地指定。
3. 選取 [更新]。 清單應該會隨著重新整理。
   
    ![](media/azure-stack-manage-storage-accounts/image5.png)
4. 若要重設篩選：選取 [篩選]、清除選取項目，然後更新。

[搜尋] 文字方塊 (在儲存體帳戶清單窗格的頂端) 可讓您醒目提示帳戶清單中選取的文字。 如果無法輕易地取得完整名稱或識別碼，您可以使用此種方式。

您可以在這裡使用任意文字，以協助找出您想看到的帳戶。

![](media/azure-stack-manage-storage-accounts/image6.png)

## <a name="look-at-account-details"></a>查看帳戶詳細資料
一旦找到您想檢視的帳戶之後，您可以選取特定的帳戶，以檢視特定的詳細資料。 新窗格隨即開啟，其中包含帳戶詳細資料，例如：帳戶類型、建立時間、位置等等。

![](media/azure-stack-manage-storage-accounts/image7.png)

## <a name="recover-a-deleted-account"></a>復原已刪除的帳戶
有時候，您必須復原已刪除的帳戶。

在 Azure Stack 中，有一個很簡單的方式可以執行此作業：

1. 瀏覽至儲存體帳戶清單。 如需詳細資訊，請參閱本文中的[尋找儲存體帳戶](#find)。
2. 在清單中找出該特定帳戶。 您可能需要篩選。
3. 請檢查帳戶的 [狀態]。 它應該指出 [已刪除]。
4. 選取可開啟 [帳戶詳細資料] 窗格的帳戶。
5. 在這個窗格的頂端，找出 [復原] 按鈕，並加以選取。
6. 選取 [是]  加以確認。
   
   ![](media/azure-stack-manage-storage-accounts/image8.png)
7. 復原現在處於 [處理中...請等待] 的狀態，正在等待復原成功的訊息出現。
   您也可以選取入口網站頂端的「鈴鐺」圖示，以檢視進度指示。
   
   ![](media/azure-stack-manage-storage-accounts/image9.png)
   
   一旦成功同步處理已復原的帳戶之後，就可以再次使用該帳戶。

### <a name="some-gotchas"></a>注意事項
* 已刪除的帳戶顯示 [不再保留] 狀態。
  
  超出保留期表示已刪除的帳戶已超出保留期限，可能無法復原。
* 已刪除的帳戶未顯示在帳戶清單中。
  
  如果已刪除的帳戶已經被記憶體回收，您的帳戶就不會顯示在帳戶清單中。 在此情況下，便無法復原該帳戶。 請參閱本文中的[回收容量](#reclaim)。

## <a name="set-the-retention-period"></a>設定保留期限
保留期限設定可讓雲端操作員指定時間間隔天數 (介於 0 到 9999 天)，在此期間，任何已刪除的帳戶都可能復原。 預設保留期限設定為 0 天。 將值設定為 "0" 表示立即不保留任何已刪除的帳戶，並標示供定期記憶體回收。

**變更保留期限：**

1. 登入[管理入口網站](https://adminportal.local.azurestack.external)。
2. 在 [系統管理]底下選取 [所有服務]  >  [區域管理]。
3. 從 [資源提供者] 清單中選取 [儲存體]。
4. 選取頂端的 [設定]，以開啟 [設定] 窗格。
5. 選取 [設定]，然後編輯保留期限值。

   設定天數，然後加以儲存。
   
   此值會立即生效，而且會設定您的整個區域。

   ![](media/azure-stack-manage-storage-accounts/image10.png)

## <a name="reclaim"></a>回收容量
保留期限會有副作用，亦即已刪除的帳戶會繼續耗用容量，直到超出保留期限為止。 身為雲端操作員，即使保留期限尚未到期，您可能還是需要一個回收已刪除帳戶空間的方式。

您可以使用入口網站或 PowerShell 來回收容量。

**使用入口網站回收容量：**
1. 瀏覽至 [儲存體帳戶] 窗格。 請參閱[尋找儲存體帳戶](#find)。
2. 選取窗格頂端的 [回收空間]。
3. 讀取訊息，然後選取 [確定]。

    ![](media/azure-stack-manage-storage-accounts/image11.png)
4. 等候成功通知。請查看入口網站上的鈴鐺圖示。

    ![](media/azure-stack-manage-storage-accounts/image12.png)
5. 重新整理 [儲存體帳戶] 頁面。 已刪除的帳戶已遭到清除，因此不會再顯示在清單中。

您也可以使用 PowerShell 明確地覆寫保留期限，並立即回收容量。

**使用 PowerShell 回收容量：**   

1. 確認您已安裝並設定 Azure PowerShell。 否則，請使用下列指示： 
   * 若要安裝最新的 Azure PowerShell 版本，並將它與您的 Azure 訂用帳戶建立關聯，請參閱[如何安裝和設定 Azure PowerShell](http://azure.microsoft.com/documentation/articles/powershell-install-configure/)。
   如需有關 Azure Resource Manager Cmdlet 的詳細資訊，請參閱[搭配使用 Azure PowerShell 與 Azure Resource Manager](http://go.microsoft.com/fwlink/?LinkId=394767)
2. 執行下列 Cmdlet：

> [!NOTE]  
> 如果您執行這些 Cmdlet，將會永久刪除帳戶及其內容。 無法復原。 使用時請務必小心。

```PowerShell  
    $farm_name = (Get-AzsStorageFarm)[0].name
    Start-AzsReclaimStorageCapacity -FarmName $farm_name
````

如需詳細資訊，請參閱 [Azure Stack PowerShell 文件](https://docs.microsoft.com/powershell/module/azurerm.azurestackstorage)。
 

## <a name="next-steps"></a>後續步驟

 - 如需管理權限的相關資訊，請參閱[管理角色型存取控制](azure-stack-manage-permissions.md)。
 - 如需管理 Azure Stack 儲存體容量的相關資訊，請參閱[管理 Azure Stack 的儲存體容量](azure-stack-manage-storage-shares.md)。