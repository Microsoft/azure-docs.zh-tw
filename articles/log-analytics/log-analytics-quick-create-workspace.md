---
title: 在 Azure 入口網站中建立 Log Analytics 工作區 | Microsoft Docs
description: 了解如何建立 Log Analytics 工作區，以在 Azure 入口網站中從您的雲端和內部部署環境啟用管理解決方案和資料收集。
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
ms.topic: conceptal
ms.date: 08/23/2018
ms.author: magoedte
ms.component: na
ms.openlocfilehash: a52654ab620908b9c8c07edf3fcdfbf70068528d
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47220783"
---
# <a name="create-a-log-analytics-workspace-in-the-azure-portal"></a>在 Azure 入口網站中建立 Log Analytics 工作區
您可以在 Azure 入口網站中設定 Log Analytics 工作區，這是一個唯一的 Log Analytics 環境，有其自己的資料存放庫、資料來源和解決方案。  如果您想要從下列來源收集資料，則需要本文中所述的步驟：

* 訂用帳戶中的 Azure 資源
* System Center Operations Manager 監視的內部部署電腦
* 來自 System Center Configuration Manager 的裝置集合 
* Azure 儲存體的診斷或記錄資料

針對其他來源，例如環境中的 Azure VM 和 Windows 或 Linux VM，請參閱下列主題：

*  [從 Azure 虛擬機器收集資料](log-analytics-quick-collect-azurevm.md) 
*  [從混合式 Linux 電腦收集資料](log-analytics-quick-collect-linux-computer.md)
*  [從混合式 Windows 電腦收集資料](log-analytics-quick-collect-windows-computer.md)

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

## <a name="sign-in-to-azure-portal"></a>登入 Azure 入口網站
在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。 

## <a name="create-a-workspace"></a>建立工作區
1. 在 Azure 入口網站中，按一下 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。

    ![Azure 入口網站](media/log-analytics-quick-collect-azurevm/azure-portal-01.png)
  
2. 按一下 [建立]，然後選取下列項目的選項：

  * 為新的 [Log Analytics 工作區] 提供名稱，例如，*DefaultLAWorkspace*。 
  * 如果選取的預設值不合適，請從下拉式清單中選取要連結的 [訂用帳戶]。
  * 針對 [資源群組]，選擇使用已設定的現有資源群組，或建立新的群組。  
  * 選取可用的**位置**。  如需詳細資訊，請查看 [Log Analytics 的可用區域](https://azure.microsoft.com/regions/services/)。
  * 如果您要在 2018 年 4 月 2 之後建立的新訂用帳戶中建立工作區，系統會自動使用「每 GB」定價方案和選項來選取將無法使用的定價層。  如果您要為在 4 月 2 日之前建立的現有訂用帳戶，或已繫結至現有 Enterprise 合約 (EA) 註冊的訂用帳戶建立工作區，請選取您偏好的定價層。  如需特定層的詳細資訊，請參閱 [Log Analytics 價格詳細資料](https://azure.microsoft.com/pricing/details/log-analytics/)。

        ![Create Log Analytics resource blade](media/log-analytics-quick-collect-azurevm/create-loganalytics-workspace-02.png)  

3. 在 [Log Analytics 工作區] 頁面上提供必要資訊之後，按一下 [確定]。  

在驗證資訊及建立工作區時，您可以在功能表的 [通知] 底下追蹤其進度。 

## <a name="next-steps"></a>後續步驟
有了可用的工作區之後，您可以設定監視遙測的集合、執行記錄搜尋以分析該資料，並且新增管理解決方案，以提供額外的資料和分析深入解析。 

* 若要從具有 Azure 診斷或 Azure 儲存體的 Azure 資源啟用資料收集，請參閱[收集 Azure 服務的記錄和計量以便使用於 Log Analytics](log-analytics-azure-storage.md)。  
* [新增 System Center Operations Manager 作為資料來源](log-analytics-om-agents.md)，以從會報告 Operations Manager 管理群組的代理程式收集資料，並且將其儲存在 Log Analytics 工作區中。 
* 連線 [Configuration Manager](log-analytics-sccm.md) 以匯入階層中集合成員的電腦。  
* 檢閱可用的[管理解決方案](https://docs.microsoft.com/azure/monitoring/monitoring-solutions-inventory?toc=%2fazure%2flog-analytics%2ftoc.json)，以及如何從您的工作區新增或移除解決方案。
