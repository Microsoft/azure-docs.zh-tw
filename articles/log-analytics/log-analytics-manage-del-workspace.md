---
title: 刪除 Azure Log Analytics 工作區 | Microsoft Docs
description: 了解如何刪除您的 Log Analytics 工作區 (如果您已在個人訂用帳戶中建立工作區) 或重組您的工作區模型。
services: log-analytics
documentationcenter: log-analytics
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/07/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: 54f2af60751ed0d9c64e71efad6fa9aa3ef06589
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37129110"
---
# <a name="delete-an-azure-log-analytics-workspace-with-the-azure-portal"></a>使用 Azure 入口網站來刪除 Azure Log Analytics 工作區
本文章說明如何使用 Azure 入口網站來刪除您可能已不再需要的 Log Analytics 工作區。 

## <a name="to-delete-a-workspace"></a>刪除工作區 
當您刪除 Log Analytics 工作區時，系統會在 30 天內從服務中刪除所有與工作區相關的資料。  刪除工作區時，請小心謹慎，因為可能有會對服務作業造成負面影響的重要資料和設定。 請考慮將其資料儲存在 Log Analytics 中的其他 Azure服務和來源，例如：

* Application Insights
* Azure 資訊安全中心
* Azure 自動化
* 在 Windows 與 Linux 虛擬機器上執行的代理程式
* 在您環境中的 Windows 與 Linux 電腦上執行的代理程式
* System Center Operations Manager
* 管理解決方案 

設定為回報給工作區的任何代理程式和 System Center Operations Manager 管理群組，都將繼續處於孤立狀態。  在您繼續之前，清查哪些代理程式、解決方案和其他Azure 服務會與工作區整合。   
 
如果您是管理員，而且有多位使用者與工作區關聯，則這些使用者與工作區之間的關聯將會中斷。 如果這些使用者與其他工作區相關聯，則可以繼續搭配其他工作區使用 Log Analytics。 不過，如果他們未與其他工作區相關聯，則必須建立工作區才能使用 Log Analytics。 

1. 登入 [Azure 入口網站](http://portal.azure.com)。 
2. 在 Azure 入口網站中，按一下左下角的 [更多服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
3. 在 Log Analytics 訂用帳戶窗格中，選取工作區，然後從中間窗格的頂端按一下 [刪除]。<br><br> ![來自工作區屬性窗格的刪除選項](media/log-analytics-manage-del-workspace/log-analytics-delete-workspace.png)<br>  
4. 當要求您確認刪除工作區的確認訊息窗格出現時，按一下 [是]。<br><br> ![確認刪除工作區](media/log-analytics-manage-del-workspace/log-analytics-delete-workspace-confirm.png)

