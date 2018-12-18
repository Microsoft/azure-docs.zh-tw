---
title: 教學課程 - 將 Azure Active Directory 記錄封存到 Azure 儲存體帳戶 (預覽) | Microsoft Docs
description: 深入了解如何設定 Azure 診斷以將 Azure Active Directory 記錄推送到儲存體帳戶 (預覽)
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: 045f94b3-6f12-407a-8e9c-ed13ae7b43a3
ms.service: active-directory
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 07/13/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: cf69fdf87f1ce64bd78478f07e575d7ae75c16e0
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49394234"
---
# <a name="tutorial-archive-azure-ad-logs-to-an-azure-storage-account-preview"></a>教學課程：將 Azure AD 記錄封存到 Azure 儲存體帳戶 (預覽)

在本教學課程中，您會了解如何設定 Azure 監視器診斷設定，以將 Azure Active Directory (Azure AD) 記錄路由傳送到 Azure 儲存體帳戶。

## <a name="prerequisites"></a>必要條件 

若要使用此功能，您必須要有：

* 具有 Azure 儲存體帳戶的 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，您可以[註冊免費試用](https://azure.microsoft.com/free/)。
* Azure AD 租用戶。
* 使用者，身分是該 Azure AD 租用戶的「全域管理員」或「安全性管理員」。

## <a name="archive-logs-to-an-azure-storage-account"></a>將記錄封存到 Azure 儲存體帳戶

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

2. 選取 [Azure Active Directory]  >  [活動]  >  [稽核記錄]。 

3. 選取 [匯出設定]。 

4. 在 [診斷設定] 窗格中，執行下列其中一項：
    * 若要變更現有設定，請選取 [編輯設定]。
    * 若要新增設定，請選取 [新增診斷設定]。  
      您最多可以有三個設定。 

    ![匯出設定](./media/quickstart-azure-monitor-route-logs-to-storage-account/ExportSettings.png)

5. 為設定輸入易記名稱以讓您記得其用途 (例如，傳送至 Azure 儲存體帳戶)。 

6. 選取 [封存至儲存體帳戶] 核取方塊，然後選取 [儲存體帳戶]。 

7. 選取您想要將記錄路由傳送到其中的 Azure 訂用帳戶和儲存體帳戶。
 
8. 選取 [確定] 以結束設定。

9. 執行下列其中一個動作或兩個動作都執行：
    * 若要將稽核記錄傳送到儲存體帳戶，請選取 [AuditLogs] 核取方塊。 
    * 若要將登入記錄傳送到儲存體帳戶，請選取 [SignInLogs] 核取方塊。

10. 使用滑桿來設定記錄資料的保留期。 根據預設，這個值是 0，這表示記錄會無限期保留在儲存體帳戶中。 如果您設定其他值，系統會自動清除超過所選天數的事件。

11. 選取 [儲存] 以儲存設定。

    ![診斷設定](./media/quickstart-azure-monitor-route-logs-to-storage-account/DiagnosticSettings.png)

12. 大約 15 分鐘之後，確認記錄是否已推送到儲存體帳戶。 移至 [Azure 入口網站](https://portal.azure.com)，選取 [儲存體帳戶]，選取您稍早使用的儲存體帳戶，然後選取 [Blob]。 

13. 針對 [稽核記錄] 選取 [insights-log-audit]。 針對 [登入記錄] 選取 [insights-logs-signin]。
    ![儲存體帳戶](./media/quickstart-azure-monitor-route-logs-to-storage-account/StorageAccount.png)

## <a name="next-steps"></a>後續步驟

* [解譯 Azure 監視器中的稽核記錄結構描述](reference-azure-monitor-audit-log-schema.md)
* [解譯 Azure 監視器中的登入記錄結構描述](reference-azure-monitor-sign-ins-log-schema.md)
* [常見問題集和已知問題](concept-activity-logs-azure-monitor.md#frequently-asked-questions)
