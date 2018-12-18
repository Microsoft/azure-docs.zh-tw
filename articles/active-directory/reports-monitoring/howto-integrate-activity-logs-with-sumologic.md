---
title: 如何使用 Azure 監視器將 Azure Active Directory 記錄與 SumoLogic 整合 (預覽) | Microsoft Docs
description: 了解如何使用 Azure 監視器將 Azure Active Directory 記錄與 SumoLogic 整合 (預覽)
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: 2c3db9a8-50fa-475a-97d8-f31082af6593
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 07/13/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: d13eb22bd58dc7e680a27738549665bc2b691898
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49392205"
---
# <a name="integrate-azure-ad-logs-with-sumologic-by-using-azure-monitor-preview"></a>使用 Azure 監視器將 Azure AD 記錄與 SumoLogic 整合 (預覽)

在此文章中，您將了解如何使用 Azure 監視器將 Azure Active Directory (Azure AD) 記錄與 SumoLogic 整合。 您首先會將記錄路由傳送至 Azure 事件中樞，然後將事件中樞與 SumoLogic 整合。

## <a name="prerequisites"></a>必要條件

若要使用此功能，您必須要有：
* 包含 Azure AD 活動記錄的 Azure 事件中樞。 了解如何[將活動記錄串流至事件中樞](quickstart-azure-monitor-stream-logs-to-event-hub.md)。 
* 已啟用 SumoLogic 單一登入的訂用帳戶。

## <a name="steps-to-integrate-azure-ad-logs-with-sumologic"></a>用來整合 Azure AD 記錄與 SumoLogic 的步驟 

1. 首先，[將 Azure AD 記錄串流至 Azure 事件中樞](quickstart-azure-monitor-stream-logs-to-event-hub.md)。
2. 設定 SumoLogic 執行個體以[收集 Azure Active Directory 的記錄](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Collect_Logs_for_Azure_Active_Directory)。
3. [安裝 Azure AD SumoLogic 應用程式](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards)，以使用可即時分析環境的預先設定儀表板。

 ![儀表板](./media/howto-integrate-activity-logs-with-sumologic/overview-dashboard.png)

## <a name="next-steps"></a>後續步驟

* [解譯 Azure 監視器中的稽核記錄結構描述](reference-azure-monitor-audit-log-schema.md)
* [解譯 Azure 監視器中的登入記錄結構描述](reference-azure-monitor-sign-ins-log-schema.md)
* [常見問題集和已知問題](concept-activity-logs-azure-monitor.md#frequently-asked-questions)
