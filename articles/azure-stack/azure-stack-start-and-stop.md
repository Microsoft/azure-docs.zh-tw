---
title: 啟動及停止 Azure Stack | Microsoft Docs
description: 了解如何啟動及關閉 Azure Stack。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: 43BF9DCF-F1B7-49B5-ADC5-1DA3AF9668CA
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/09/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: dd1e64d5ad6982c85a8205e3036d30a2ede92f7c
ms.sourcegitcommit: aa988666476c05787afc84db94cfa50bc6852520
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37930285"
---
# <a name="start-and-stop-azure-stack"></a>啟動及停止 Azure Stack
您應該遵循本文中的程序，適當地關閉然後重新啟動 Azure Stack 服務。 關機將實際關閉整個 Azure Stack 環境的電源。 啟動會開啟所有基礎結構角色的電源，並在關機之前讓租用戶資源回復為電源狀態。

## <a name="stop-azure-stack"></a>停止 Azure Stack 

使用以下步驟來關閉 Azure Stack：

1. 針對即將的關機，準備在 Azure Stack 環境的租用戶資源上執行的所有工作負載。 

2. 從可以網路存取 Azure Stack ERCS VM 的機器開啟特殊權限端點工作階段 (PEP)。 如需相關指示，請參閱[使用 Azure Stack 中具有特殊權限的端點](azure-stack-privileged-endpoint.md)。

3. 從 PEP 執行：

    ```powershell
      Stop-AzureStack
    ```

4. 等待所有實體 Azure Stack 節點關閉電源。

> [!Note]  
> 您可以遵循提供 Azure Stack 硬體的原始設備製造商 (OEM) 的指示，以確認實體節點的電源狀態。 

## <a name="start-azure-stack"></a>啟動 Azure Stack 

使用以下步驟來啟動 Azure Stack。 請遵循這些步驟，而不論 Azure Stack 是如何停止的。

1. 將您 Azure Stack 環境中每個實體節點的電源開啟。 遵循提供 Azure Stack 硬體的原始設備製造商 (OEM) 的指示，以確認實體節點的電源開啟指示。

2. 等待直到 Azure Stack 基礎結構服務啟動。 Azure Stack 基礎結構服務完成啟動程序可能需要兩個小時。 您可以使用 [**Get-ActionStatus** Cmdlet](#get-the-startup-status-for-azure-stack) 來確認 Azure Stack 的啟動狀態。

3. 確定所有您的租用戶資源都已回復為它們在關機之前的狀態。 在啟動之後，工作負載管理員可能需要重新設定租用戶資源上執行的工作負載。

## <a name="get-the-startup-status-for-azure-stack"></a>取得 Azure Stack 的啟動狀態

使用以下步驟來啟動 Azure Stack 啟動程序：

1. 從可以網路 Azure Stack ERCS VM 的機器開啟特殊權限端點工作階段。

2. 從 PEP 執行：

    ```powershell
      Get-ActionStatus Start-AzureStack
    ```

## <a name="troubleshoot-startup-and-shutdown-of-azure-stack"></a>針對 Azure Stack 的啟動及關機進行疑難排解

如果開啟 Azure Stack 環境的電源 2 小時後基礎結構和租用戶服務未成功啟動，請執行以下步驟。 

1. 從可以網路 Azure Stack ERCS VM 的機器開啟特殊權限端點工作階段。

2. 請執行： 

    ```powershell
      Test-AzureStack
      ```

3. 檢閱輸出並解決任何健康情況錯誤。 如需詳細資訊，請參閱[執行 Azure Stack 的驗證測試](azure-stack-diagnostic-test.md)。

4. 請執行：

    ```powershell
      Start-AzureStack
    ```

5. 如果執行 **Start-AzureStack** 的結果為失敗，請連絡 Microsoft 客戶服務支援。 

## <a name="next-steps"></a>後續步驟 

如需深入了解 Azure Stack 診斷工具和問題記錄的詳細資訊，請參閱 [Azure Stack 診斷工具](azure-stack-diagnostics.md)。
