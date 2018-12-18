---
title: 管理來自其他監視服務的警示
description: 管理 Azure 監視器中的 Nagios、Zabbix 和 SCOM 警示
author: anantr
services: monitoring
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: anantr
ms.component: alerts
ms.openlocfilehash: d9d0cb326fb063e0a6bbfaab6a85961ab2b35416
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49389367"
---
# <a name="manage-alerts-from-other-monitoring-services"></a>管理來自其他監視服務的警示

您現在可以在[統一的警示體驗](https://aka.ms/azure-alerts-overview)中檢視 Nagios、Zabbix 和 System Center Operations Manager 中的警示。 這些警示來自 Nagios/Zabbix 伺服器或 System Center Operations Manager 整合到 Log Analytics 中。 

## <a name="prerequisites"></a>必要條件
Log Analytics 存放庫中具有警示類型的任何記錄，將匯入到統一的警示體驗，所以您必須執行必要組態以收集這些記錄。
1. 針對 **Nagios** 和**Zabbix** 警示，[設定這些伺服器](https://docs.microsoft.com/azure/log-analytics/log-analytics-linux-agents)以將警示傳送至 Log Analytics。
1. 針對 **System Center Operations Manager** 警示，[將 Operations Manager 管理群組連線到 Log Analytics 工作區](https://docs.microsoft.com/azure/log-analytics/log-analytics-om-agents)。 在 System Center Operations Manager 中建立的任何警示會匯入至記錄分析。

## <a name="view-your-alert-instances"></a>檢視警示執行個體
一旦設定匯入到 Log Analytics 之後，您即可開始在[統一的警示體驗](https://aka.ms/azure-alerts-overview)中檢視這些監視服務的警示執行個體。 一旦這些警示出現在統一的警示體驗中，您即可[管理您的警示執行個體](https://aka.ms/managing-alert-instances)、[管理在這些警示上建立的智慧群組](https://aka.ms/managing-smart-groups)以及[變更警示和智慧群組的狀態](https://aka.ms/managing-alert-smart-group-states)。

> [!NOTE]
>  統一的警示體驗中的 Nagios 警示不具狀態 – 例如，警示的[監視條件](https://aka.ms/azure-alerts-overview)將不會從「已引發」變更為「已解決」。 相反地，「已引發」和「已解決」兩者會顯示為個別的警示執行個體。 
