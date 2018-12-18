---
title: 教學課程 - 將 Azure Active Directory 記錄串流到 Azure 事件中樞 (預覽) | Microsoft Docs
description: 深入了解如何設定 Azure 診斷以將 Azure Active Directory 記錄推送到事件中樞 (預覽)
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
ms.openlocfilehash: c8157820f7137eb06a0fe10c42645ec91f33edbc
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364047"
---
# <a name="tutorial-stream-azure-ad-logs-to-an-azure-event-hub-preview"></a>教學課程：將 Azure AD 記錄串流到 Azure 事件中樞 (預覽)

在本教學課程中，深入了解如何設定 Azure 監視器診斷設定，以將 Azure Active Directory (Azure AD) 記錄串流到 Azure 事件中樞。 使用這個機制將記錄與 Splunk 和 QRadar 等第三方安全性資訊與事件管理 (SIEM) 工具整合在一起。

## <a name="prerequisites"></a>必要條件 

若要使用此功能，您必須要有：

* Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，您可以[註冊免費試用](https://azure.microsoft.com/free/)。
* Azure AD 租用戶。
* 使用者，身分是該 Azure AD 租用戶的「全域管理員」或「安全性管理員」。
* Azure 訂用帳戶中的事件中樞命名空間和事件中樞。 了解如何[建立事件中樞](https://docs.microsoft.com/azure/event-hubs/event-hubs-create)。

## <a name="archive-logs-to-an-event-hub"></a>將記錄封存到事件中樞

1. 登入 [Azure 入口網站](https://portal.azure.com)。 

2. 選取 [Azure Active Directory]  >  [活動]  >  [稽核記錄]。 

3. 選取 [匯出設定]。  
    
4. 在 [診斷設定] 窗格中，執行下列其中一項：
    * 若要變更現有設定，請選取 [編輯設定]。
    * 若要新增設定，請選取 [新增診斷設定]。  
      您最多可以有三個設定。

      ![匯出設定](./media/quickstart-azure-monitor-stream-logs-to-event-hub/ExportSettings.png)

5. 請選取 [串流到事件中樞] 核取方塊，然後選取 [事件中樞/設定]。

6. 選取您想要將記錄路由傳送到其中的 Azure 訂用帳戶和事件中樞命名空間。  
    訂用帳戶和事件中樞命名空間都必須與記錄串流來源的 Azure AD 租用戶產生關聯。 您也可以在要將記錄傳送到其中的事件中樞命名空間內，指定事件中樞。 如果未指定事件中樞，則會在命名空間中建立一個事件中樞，具有預設名稱 **insights-logs-audit**。

7. 選取 [確定] 以結束事件中樞設定。

8. 執行下列其中一個動作或兩個動作都執行：
    * 若要將稽核記錄傳送到儲存體帳戶，請選取 [AuditLogs] 核取方塊。 
    * 若要將登入記錄傳送到儲存體帳戶，請選取 [SignInLogs] 核取方塊。

9. 選取 [儲存] 以儲存設定。

    ![診斷設定](./media/quickstart-azure-monitor-stream-logs-to-event-hub/DiagnosticSettings.png)

10. 大約 15 分鐘之後，確認事件顯示在事件中樞中。 若要完成這項操作，請從入口網站移至事件中樞，並確認**內送郵件**計數大於零。 

    ![稽核記錄](./media/quickstart-azure-monitor-stream-logs-to-event-hub/InsightsLogsAudit.png)

## <a name="access-data-from-your-event-hub"></a>存取事件中樞內的資料

資料顯示在事件中樞後，您便可透過兩種方式存取和讀取資料：

* **設定支援的 SIEM 工具**。 為了讀取事件中樞內的資料，大部分工具皆需要事件中樞連接字串和 Azure 訂用帳戶的特定權限。 第三方工具與 Azure 監視器的整合包括但不限於：
    * **Splunk**：如需如何將 Azure AD 記錄與 Splunk 整合在一起的詳細資訊，請參閱[如何使用 Azure 監視器將 Azure AD 記錄與 Splunk 整合在一起](tutorial-integrate-activity-logs-with-splunk.md)。
    
    * **IBM QRadar**：DSM 與 Azure 事件中樞通訊協定均可在 [IBM 支援](http://www.ibm.com/support)中下載取得。 如需有關與 Azure 整合的詳細資訊，請移至 [IBM QRadar Security Intelligence Platform 7.3.0](https://www.ibm.com/support/knowledgecenter/SS42VS_DSM/c_dsm_guide_microsoft_azure_overview.html?cp=SS42VS_7.3.0) 網站。
    
    * **Sumo Logic**：若要將 Sumo Logic 設定為從事件中樞取用資料，請參閱[安裝 Azure AD 應用程式及檢視儀表板](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards)。 

* **設定自訂工具**。 如果 Azure 監視器診斷尚未支援您目前的 SIEM，可以使用事件中樞 API 來設定自訂工具。 若要深入了解，請參閱[開始從事件中樞接收訊息](https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-standard-getstarted-receive-eph)。


## <a name="next-steps"></a>後續步驟

* [使用 Azure 監視器將 Azure AD 記錄與 Splunk 整合](tutorial-integrate-activity-logs-with-splunk.md)
* [使用 Azure 監視器將 Azure AD 記錄與 SumoLogic 整合](howto-integrate-activity-logs-with-sumologic.md)
* [解譯 Azure 監視器中的稽核記錄結構描述](reference-azure-monitor-audit-log-schema.md)
* [解譯 Azure 監視器中的登入記錄結構描述](reference-azure-monitor-sign-ins-log-schema.md)
