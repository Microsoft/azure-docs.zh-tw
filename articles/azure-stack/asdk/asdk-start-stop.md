---
title: 啟動及停止 Azure Stack 開發套件 (ASDK) | Microsoft Docs
description: 了解如何啟動及關閉 Azure Stack 開發套件 (ASDK)。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/15/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 98ae7fe0fde922aee28ffc05b64d223a40085447
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340914"
---
# <a name="start-and-stop-the-azure-stack-development-kit-asdk"></a>啟動及停止 Azure Stack 開發套件 (ASDK)
不建議直接重新啟動 ASDK 主機電腦。 而是應該遵循本文中的程序，適當地關閉然後重新啟動 ASDK 服務。 

## <a name="stop-azure-stack"></a>停止 Azure Stack 
若要正常關閉 Azure Stack 服務和 ASDK 主機電腦，請使用下列 PowerShell 命令：

1. 以 AzureStack\CloudAdmin 身分登入 ASDK 主機電腦。
2. 以系統管理員身分開啟 PowerShell (非 PowerShell ISE)。
3. 執行下列命令來建立特殊權限的端點 (PEP) 工作階段： 

   ```powershell
   Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
   ```
4. 接著在 PEP 工作階段中，使用 **Stop-AzureStack** Cmdlet 來停止 Azure Stack 服務，並且關閉 ASDK 主機電腦：

   ```powershell
   Stop-AzureStack
   ```
5. 檢閱 PowerShell 輸出，以確保在 ASDK 主機電腦關機之前，所有 Azure Stack 服務都已成功關閉。 關機程序需要幾分鐘的時間。

## <a name="start-azure-stack"></a>啟動 Azure Stack 
ASDK 服務應該會在主機電腦啟動時自動啟動。 不過，ASDK 基礎結構服務啟動時間會因 ASDK 主機電腦硬體組態的效能而異。 在某些情況下，可能需要數小時才能讓所有服務都成功重新啟動。

不論 ASDK 的關機方式是什麼，您都應該使用下列步驟來確認在主機電腦電源開啟之後，所有 Azure Stack 服務已啟動且完全正常運作： 

1. 開啟 ASDK 主機電腦電源。 
2. 以 AzureStack\CloudAdmin 身分登入 ASDK 主機電腦。
3. 以系統管理員身分開啟 PowerShell (非 PowerShell ISE)。
4. 執行下列命令來建立特殊權限的端點 (PEP) 工作階段：

   ```powershell
   Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
   ```
5. 接著在 PEP 工作階段中，執行下列命令來檢查 Azure Stack 服務的啟動狀態：

   ```powershell
   Get-ActionStatus Start-AzureStack
   ```
6. 檢閱輸出，以確定 Azure Stack 服務已成功重新啟動。

若要深入了解正確關閉並重新啟動 Azure Stack 服務的建議程序，請參閱[啟動及停止 Azure Stack](.\.\azure-stack-start-and-stop.md)。 

## <a name="troubleshoot-startup-and-shutdown"></a>針對啟動及關機進行疑難排解 
如果 Azure Stack 服務在您開啟 ASDK 主機電腦電源的兩個小時內未成功啟動，請執行下列步驟：

1. 以 AzureStack\CloudAdmin 身分登入 ASDK 主機電腦。
2. 以系統管理員身分開啟 PowerShell (非 PowerShell ISE)。
3. 執行下列命令來建立特殊權限的端點 (PEP) 工作階段：

   ```powershell
   Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
   ```
4. 接著在 PEP 工作階段中，執行下列命令來檢查 Azure Stack 服務的啟動狀態：

   ```powershell
   Test-AzureStack
   ```
5. 檢閱輸出並解決任何錯誤。 如需詳細資訊，請參閱[執行 Azure Stack 的驗證測試](.\.\azure-stack-diagnostic-test.md)。
6. 執行 **Start-AzureStack** Cmdlet 以在 PEP 工作階段內重新啟動 Azure Stack 服務：

   ```powershell
   Start-AzureStack
   ```

如果執行 **Start-AzureStack** 導致失敗，請造訪 [Azure Stack 支援論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=azurestack)以取得 ASDK 疑難排解支援。 

## <a name="next-steps"></a>後續步驟 
如需深入了解 Azure Stack 診斷工具和問題記錄的詳細資訊，請參閱 [Azure Stack 診斷工具](.\.\azure-stack-diagnostics.md)。
