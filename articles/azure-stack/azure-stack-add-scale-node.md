---
title: Azure Stack 新增縮放節點 | Microsoft Docs
description: 在 Azure Stack 中將節點新增至縮放單位。
services: azure-stack
documentationcenter: ''
author: brenduns
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/20/2018
ms.author: brenduns
ms.reviewer: thoroet
ms.openlocfilehash: 02602243bcb4e426ebf4984e387da8e8c148232e
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "41946442"
---
# <a name="add-additional-scale-unit-nodes-in-azure-stack"></a>在 Azure Stack 中新增更多縮放單位節點

Azure Stack 操作員可藉由新增更多實體電腦，以增加現有縮放單位的整體容量。 實體電腦也稱為縮放單位節點。 您所新增的每個縮放單位節點在 CPU 類型、記憶體、磁碟編號和大小等方面，都必須與縮放單位中既有的節點同類。

> [!NOTE]  
您必須執行 Azure Stack 1807 或更新版本，才能新增更多縮放單位節點。

若要新增縮放單位節點，您必須使用 Azure Stack，並執行您的硬體設備製造商 (OEM) 所提供的工具。 OEM 工具會在硬體生命週期主機 (HLH) 上執行，以確定新的實體電腦符合現有節點的韌體層級。

下列流程圖顯示新增縮放單位節點的一般程序。

![新增縮放單位流程](media/azure-stack-add-scale-node/add-node-flow.png) &#42; *無論 OEM 硬體廠商是否制訂實體伺服器機架設置和更新，韌體都會根據您的支援合約而有所不同。*

新增節點的作業可能需要數小時或數天才能完成。

> [!Note]  
> 當新增縮放單位節點作業已經在進行時，請勿嘗試下列任何作業：
>
>  - 更新 Azure Stack
>  - 輪替憑證
>  - 停止 Azure Stack
>  - 修復縮放單位節點


## <a name="add-scale-unit-nodes"></a>新增縮放單位節點

下列步驟是節點新增方式的高階概觀。 請務必先參考 OEM 提供的容量擴充文件，再依照這些步驟操作。

1. 將新的實體伺服器放置在機架中，並適當地接上纜線。 
2. 啟用實體交換器連接埠，並調整存取控制清單 (ACL) (如果適用)。
3. 在基礎板管理控制器 (BMC) 設定正確的 IP 位址，並根據 OEM 提供的文件套用所有 BIOS 設定。
4. 使用由硬體製造商提供、執行於 HLH 上的工具，將目前的韌體基準套用至所有元件。
5. 在 Azure Stack 管理入口網站中執行「新增節點」作業。
6. 驗證新增節點作業執行成功。 若要這麼做，請查看[縮放單位的**狀態**](#monitor-add-node-operations)。 

## <a name="add-the-node"></a>新增節點

您可以使用管理入口網站或 PowerShell 來新增節點。 新增節點作業會先新增縮放單位節點作為可用的計算容量，然後自動擴充儲存體容量。 容量會自動擴充，因為 Azure Stack 是*計算*和*儲存體*一併調整的超融合系統。

### <a name="use-the-admin-portal"></a>使用管理入口網站

1. 以 Azure Stack 操作員身分登入 Azure Stack 管理入口網站。
2. 瀏覽至 [新增] > [容量] > [縮放單位節點]。
   ![縮放單位節點](media/azure-stack-add-scale-node/select-node1.png)
3. 在 [新增節點] 窗格上，選取「區域」，然後選取您想要將節點新增至其中的「縮放單位」。 此外，也請為您要新增的縮放單位節點指定「BMC IP 位址」。 您一次只能新增一個節點。
   ![新增節點詳細資料](media/azure-stack-add-scale-node/select-node2.png)
 

### <a name="use-powershell"></a>使用 PowerShell

請使用 **New-AzsScaleUnitNodeObject** Cmdlet 來新增節點。  

在使用下列範例 PowerShell 指令碼之前，請先將*節點名稱*和 *IP 位址*值取代為 Azure Stack 環境中的值。

  > [!Note]  
  > 為節點命名時，您必須讓名稱長度不超過 15 個字元。 此外，您也不能使用包含空格或包含下列任何字元的名稱：`\`、`/`、`:`、`*`、`?`、`"`、`<`、`>`、`|`、`\`、`~`、`!`、`@`、`#`、`$`、`%`、`^`、`&`、`(`、`)`、`{`、` }`、`_`。

**新增節點：**
  ```powershell
  ## Add a single Node 
  $NewNode=New-AzsScaleUnitNodeObject -computername "<name_of_new_node>" -BMCIPv4Address "<BMCIP_address_of_new_node>" 
 
  Add-AzsScaleUnitNode -NodeList $NewNode -ScaleUnit "<name_of_scale_unit_cluster>" 
  ```  

## <a name="monitor-add-node-operations"></a>監視新增節點作業 
您可以使用管理入口網站或 PowerShell 來取得新增節點作業的狀態。 新增節點作業可能需要數小時到數天才能完成。

### <a name="use-the-admin-portal"></a>使用管理入口網站 
若要監視新增節點的情形，請在管理入口網站中檢視縮放單位或縮放單位節點物件。 若要這樣做，請移至 [區域管理] > [縮放單位]。 接著，請選取您要檢視的縮放單位或縮放單位節點。 

### <a name="use-powershell"></a>使用 PowerShell
縮放單位和縮放單位節點的狀態可使用 PowerShell 來擷取，如下所示：
  ```powershell
  #Retrieve Status for the Scale Unit
  Get-AzsScaleUnit|select name,state
 
  #Retrieve Status for each Scale Unit Node
  Get-AzsScaleUnitNode |Select Name, ScaleUnitNodeStatus
```

### <a name="status-for-the-add-node-operation"></a>新增節點作業的狀態 
**針對縮放單位：**
|狀態               |說明  |
|---------------------|---------|
|執行中              |所有節點均有效參與縮放單位。|
|已停止              |縮放單位節點已關閉或無法存取。|
|擴充中            |有一或多個縮放單位節點目前正在新增為計算容量。|
|正在設定儲存體  |計算容量已擴充，且正在執行儲存體設定。|
|需要補救 |偵測到需要修復一或多個縮放單位節點的錯誤。|


**針對縮放單位節點：**
|狀態                |說明  |
|----------------------|---------|
|執行中               |節點正有效參與縮放單位。|
|已停止               |節點無法使用。|
|新增中                |節點正在新增至縮放單位。|
|修復中             |正在修復節點。|
|維護            |節點已暫停，且沒有作用中的使用者工作負載正在執行。 |
|需要補救  |偵測到需要修復節點的錯誤。|


## <a name="troubleshooting"></a>疑難排解
以下是新增節點時常見的問題。 

**案例 1：** 新增縮放單位節點作業失敗，但有一或多個節點會列示為「已停止」狀態。  
- 補救措施：使用修復作業來修復一或多個節點。 一次只能執行一個修復作業。

**案例 2：** 已新增一或多個縮放單位節點，但儲存體擴充失敗。 在此案例中，縮放單位節點物件報告了「執行中」狀態，但「設定儲存體」工作並未啟動。  
- 補救措施：執行下列 PowerShell Cmdlet，以使用具特殊權限的端點檢視儲存體健康情況：
  ```powershell
     Get-VirtualDisk -CimSession s-cluster | Get-StorageJob
  ```
 
**案例 3：** 您收到警示，指出儲存體相應放大作業失敗。  
- 補救措施：在此情況下，儲存體設定工作已失敗。 發生此問題時，您必須連絡支援人員。


## <a name="next-steps"></a>後續步驟 
檢視[節點動作](azure-stack-node-actions.md) 
