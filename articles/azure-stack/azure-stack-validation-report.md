---
title: Azure Stack 的驗證報告 | Microsoft Docs
description: 使用 Azure Stack 整備檢查程式報告來檢閱驗證結果。
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 05/08/2018
ms.author: sethm
ms.reviewer: ''
ms.openlocfilehash: 156b84e4941363716721b5cee6c19333ffe7594c
ms.sourcegitcommit: 4b1083fa9c78cd03633f11abb7a69fdbc740afd1
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "49079441"
---
# <a name="azure-stack-validation-report"></a>Azure Stack 驗證報告
使用 Azure Stack 整備檢查程式工具來執行驗證，這些驗證支援 Azure Stack 環境的部署和維護。 此工具會將結果寫入 .json 報告檔案。 報告會針對 Azure Stack 部署的必要條件狀態，顯示相關的詳細和摘要資料。 報告也會顯示現有 Azure Stack 部署祕密輪替的相關資訊。  

 ## <a name="where-to-find-the-report"></a>報告的所在位置
工具在執行時，會將結果記錄至 **AzsReadinessCheckerReport.json**。 此工具也會建立名為 **AzsReadinessChecker.log** 的記錄。 PowerShell 中的驗證結果會一同顯示這些檔案的位置。

![run-validation](./media/azure-stack-validation-report/validation.png)

這兩個檔案都會保存同一部電腦上所執行的後續驗證檢查結果。  例如，可以執行此工具來驗證憑證、再次執行來驗證 Azure 身分識別，然後第三次執行來驗證註冊。 這三個驗證的結果都可在所產生的 .json 報告中看到。  

根據預設，這兩個檔案都會寫入到 C:\Users\<使用者名稱>\AppData\Local\Temp\AzsReadinessChecker\AzsReadinessCheckerReport.json。  
- 使用執行命令列結尾的 **-OutputPath** ***&lt;path&gt;*** 參數來指定不同的報告位置。   
- 使用執行命令結尾的 **-CleanReport** 參數，從 AzsReadinessCheckerReport.json 清除資訊。 關於此工具先前的執行。

## <a name="view-the-report"></a>檢視報告
若要在 PowerShell 中檢視報告，請提供報告的路徑作為 **-ReportPath** 的值。 此命令會顯示報告的內容，並識別還沒有結果的驗證。

例如，若要從 PowerShell 提示字元檢視對報告所在位置開啟的報告，請執行： 
   > `Start-AzsReadinessChecker -ReportPath .\AzsReadinessReport.json` 

輸出結果類似下圖：

![view-report](./media/azure-stack-validation-report/view-report.png)

## <a name="view-the-report-summary"></a>檢視報告摘要
若要檢視報告的摘要，您可以將 **-Summary** 參數新增至 PowerShell 命令列的結尾。 例如︰ 
 > `Start-AzsReadinessChecker -ReportPath .\AzsReadinessReport.json -summary`  

摘要會顯示沒有結果的驗證，並指出已完成的驗證為成功還是失敗。 輸出結果類似下圖：

![report-summary](./media/azure-stack-validation-report/report-summary.png)


## <a name="view-a-filtered-report"></a>檢視篩選的報告
若要檢視依單一類型的驗證而篩選的報告，請使用 **-ReportSections** 參數並搭配下列其中一個值：
- 憑證
- AzureRegistration
- AzureIdentity
- 工作   
- 全部  

例如，若只要檢視憑證的報告摘要，請使用下列 PowerShell 命令列： 
 > `Start-AzsReadinessChecker -ReportPath .\AzsReadinessReport.json -ReportSections Certificate – Summary`


## <a name="see-also"></a>另請參閱
[Start-AzsReadinessChecker Cmdlet 參考](azure-stack-azsreadiness-cmdlet.md)
