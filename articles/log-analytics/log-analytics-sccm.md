---
title: 將 Configuration Manager 連線至 Log Analytics | Microsoft Docs
description: 本文說明將 Configuration Manager 連線至 Log Analytics 並開始分析資料的步驟。
services: log-analytics
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: f2298bd7-18d7-4371-b24a-7f9f15f06d66
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/22/2018
ms.author: magoedte
ms.component: ''
ms.openlocfilehash: 433914bc4501b13ba65015d15b0c513a38bf1273
ms.sourcegitcommit: 3856c66eb17ef96dcf00880c746143213be3806a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/02/2018
ms.locfileid: "48041657"
---
# <a name="connect-configuration-manager-to-log-analytics"></a>將 Configuration Manager 連線至 Log Analytics
您可以將 System Center Configuration Manager 環境連線到 Azure Log Analytics，以同步處理裝置集合資料，並在 Log Analytics 和 Azure 自動化中參考這些集合。  

## <a name="prerequisites"></a>必要條件

Log Analytics 支援 System Center Configuration Manager 目前分支，1606 版和更高版本。  

## <a name="configuration-overview"></a>組態概觀
下列步驟摘要說明用來設定 Configuration Manager 與 Log Analytics 整合的步驟。  

1. 在 Azure 入口網站中，將 Configuration Manager 註冊為 Web 應用程式和/或 Web API 應用程式，並確保您擁有 Azure Active Directory 註冊中的用戶端識別碼和用戶端秘密金鑰。 如需如何完成此步驟的詳細資訊，請參閱[使用入口網站來建立可存取資源的 Active Directory 應用程式和服務主體](../azure-resource-manager/resource-group-create-service-principal-portal.md)。
2. 在 Azure 入口網站中，[對 Configuration Manager (已註冊的 Web 應用程式) 授與 Log Analytics 的存取權限](#grant-configuration-manager-with-permissions-to-log-analytics)。
3. 在 Configuration Manager 中，[使用新增 OMS 連線精靈新增連線](#add-an-oms-connection-to-configuration-manager)。
4. 如果密碼或用戶端祕密金鑰過期或遺失，在 Configuration Manager 中[更新連線屬性](#update-oms-connection-properties)。
5. 在執行 Configuration Manager 服務連線點網站系統角色的電腦上[下載並安裝 Microsoft Monitoring Agent](#download-and-install-the-agent)。 代理程式會將 Configuration Manager 資料傳送至 Log Analytics 工作區。
6. 在 Log Analytics 中，以電腦群組的形式[從 Configuration Manager 匯入集合](#import-collections)。
7. 在 Log Analytics 中，以電腦群組的形式[從 Configuration Manager 檢視資料](log-analytics-computer-groups.md)。

若要深入了解如何將 Configuration Manager 連線至 OMS，請參閱[將資料從 Configuration Manager 同步處理至 Microsoft Operations Management Suite](https://technet.microsoft.com/library/mt757374.aspx)。

## <a name="grant-configuration-manager-with-permissions-to-log-analytics"></a>對 Configuration Manager 授與 Log Analytics 的權限
在下列程序中，您會將 Log Analytics 工作區中的「參與者」角色授與給您稍早為 Configuration Manager 建立的 AD 應用程式和服務主體。  如果您還沒有工作區，請先參閱[在 Azure Log Analytics 中建立工作區](log-analytics-quick-create-workspace.md)再繼續。  這可讓 Configuration Manager 進行驗證並連線到 Log Analytics 工作區。  

> [!NOTE]
> 您必須為 Configuration Manager 指定 Log Analytics 的權限。 否則，當您在 Configuration Manager 中使用組態精靈時，將會收到錯誤訊息。
>

1. 在 Azure 入口網站中，按一下左上角的 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。<br><br> ![Azure 入口網站](media/log-analytics-quick-collect-azurevm/azure-portal-01.png)<br><br>  
2. 在 Log Analytics 工作區清單中，選取要修改的工作區。
3. 從左側窗格中，選取 [存取控制 (IAM)]。
4. 在 [存取控制] 頁面中按一下 [新增]，隨即會開啟 [新增權限] 窗格。
5. 在 [新增權限] 窗格的 [角色] 下拉式清單底下，選取 [參與者] 角色。  
6. 在 [指派存取權的對象] 下拉式清單底下，選取稍早在 AD 中建立的 Configuration Manager 應用程式，然後按一下 [確定]。  

## <a name="download-and-install-the-agent"></a>下載並安裝代理程式
檢閱[將 Windows 電腦連線到 Azure 中的 Log Analytics 服務](log-analytics-agent-windows.md)一文，以了解可用來在裝載了 Configuration Manager 服務連接點網站系統角色的電腦上安裝 Microsoft Monitoring Agent 的方法。  

## <a name="add-an-oms-connection-to-configuration-manager"></a>對 Configuration Manager 新增 OMS 連線
若要新增 OMS 連線，Configuration Manager 環境的[服務連線點](https://technet.microsoft.com/library/mt627781.aspx)必須設定成線上模式。

1. 在 Configuration Manager 的 [管理] 工作區中選取 [OMS 連接器]。 這會開啟**新增 OMS 連線精靈**。 選取 [下一步] 。
2. 在 [一般] 畫面上，確認您已完成下列動作而且您有每個項目的詳細資料，然後選取 [下一步]。

   1. 在 Azure 入口網站中，您已將 Configuration Manager 註冊為 Web 應用程式和/或 Web API 應用程式，而且您擁有[註冊中的用戶端識別碼](../active-directory/develop/quickstart-v1-integrate-apps-with-azure-ad.md)。
   2. 在 Azure 入口網站中，您已經為 Azure Active Directory 中已註冊的應用程式建立應用程式祕密金鑰。  
   3. 在 Azure 入口網站中，您已經為已註冊的 Web 應用程式提供存取 OMS 的權限。  
      ![OMS 連線精靈的一般頁面](./media/log-analytics-sccm/sccm-console-general01.png)
3. 在 [Azure Active Directory] 畫面上對 Log Analytics 設定連線設定 (方法是提供 [租用戶]、[用戶端 ID] 和 [用戶端秘密金鑰])，然後選取 [下一步]。  
   ![OMS 連線精靈的 Azure Active Directory 頁面](./media/log-analytics-sccm/sccm-wizard-tenant-filled03.png)
4. 如果您成功完成其他所有程序，則 [OMS 連線組態] 畫面上的資訊會自動出現在此頁面。 [Azure 訂用帳戶]、[Azure 資源群組] 和 [Operations Management Suite 工作區] 應該會出現連線設定資訊。  
   ![OMS 連線精靈的 OMS 連線頁面](./media/log-analytics-sccm/sccm-wizard-configure04.png)
5. 精靈會使用您輸入的資訊連線至 Log Analytics 服務。 選取想要與 Log Analytics 同步處理的裝置集合，然後按一下 [新增]。  
   ![選取集合](./media/log-analytics-sccm/sccm-wizard-add-collections05.png)
6. 確認 [摘要] 畫面上的連線設定，然後選取 [下一步]。 [進度] 畫面會顯示連線狀態，然後應該 [完成]。

> [!NOTE]
> 您必須將階層中的最上層網站連線至 Log Analytics。 如果您將獨立主要網站連線至 Log Analytics，然後在環境中新增管理中心網站，則必須在新階層內先刪除再重新建立連線。
>
>

在將 Configuration Manager 連結至 Log Analytics 之後，您可以新增或移除集合，並檢視連線的屬性。

## <a name="update-log-analytics-connection-properties"></a>更新 Log Analytics 連線屬性
如果密碼或用戶端秘密金鑰過期或遺失，您必須手動更新 Log Analytics 連線屬性。

1. 在 Configuration Manager 中瀏覽至 [雲端服務]，然後選取 [OMS 連接器] 以開啟 [OMS 連線屬性] 頁面。
2. 在此頁面上按一下 [Azure Active Directory] 索引標籤，檢視 [租用戶]、[用戶端識別碼] 和 [用戶端秘密金鑰到期]。 **確認** **用戶端秘密金鑰**是否已過期。

## <a name="import-collections"></a>匯入集合
在 Configuration Manager 新增了 OMS 連線，並在執行 Configuration Manager 服務連線點網站系統角色的電腦上安裝了代理程式之後，下一個步驟是從 Configuration Manager 將集合匯入 Log Analytics 中作為電腦群組。

在完成從階層匯入裝置集合的初始設定後，每 3 個小時就會擷取一次集合成員資格資訊，以便成員資格會隨時保持最新狀態。 您可以隨時選擇停用此功能。

1. 在 Azure 入口網站中，按一下左上角的 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。
2. 在您的 Log Analytics 工作區清單中，選取 Configuration Manager 用來註冊的工作區。  
3. 選取 [進階設定]。<br><br> ![Log Analytics 進階設定](media/log-analytics-quick-collect-azurevm/log-analytics-advanced-settings-01.png)<br><br>  
4. 依序選取 [電腦群組] 和 [SCCM]。  
5. 選取 [匯入 Configuration Manager 集合成員資格]，然後按一下 [儲存]。  
   ![電腦群組 - SCCM 索引標籤](./media/log-analytics-sccm/sccm-computer-groups01.png)

## <a name="view-data-from-configuration-manager"></a>從 Configuration Manager 檢視資料
在 Configuration Manager 新增了 OMS 連線，並在執行 Configuration Manager 服務連線點網站系統角色的電腦上安裝了代理程式之後，代理程式的資料會傳送至 Log Analytics。 在 Log Analytics 中，Configuration Manager 集合會顯示為[電腦群組](log-analytics-computer-groups.md)。 您可以從 [設定\電腦群組] 下的 [Configuration Manager] 頁面檢視群組。

匯入集合之後，您可以看到已偵測到多少部具有集合成員資格的電腦。 您也可以看到已匯入的集合數目。

![電腦群組 - SCCM 索引標籤](./media/log-analytics-sccm/sccm-computer-groups02.png)

當您按一下其中一項時，[搜尋] 會開啟，顯示所有匯入的群組或是屬於每個群組的所有電腦。 使用 [記錄搜尋](log-analytics-log-searches.md)，您就可以開始深入分析 Configuration Manager 資料。

## <a name="next-steps"></a>後續步驟
* 請使用 [記錄檔搜尋](log-analytics-log-searches.md)，檢視有關 Configuration Manager 資料的詳細資訊。
