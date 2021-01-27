---
title: 將設定管理員連接到 Azure 監視器 |Microsoft Docs
description: 本文說明將設定管理員連接到 Azure 監視器中的工作區，並開始分析資料的步驟。
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/30/2020
ms.openlocfilehash: ec19396d782bf34e85001892159c0ce785487f09
ms.sourcegitcommit: 436518116963bd7e81e0217e246c80a9808dc88c
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/27/2021
ms.locfileid: "98918884"
---
# <a name="connect-configuration-manager-to-azure-monitor"></a>將設定管理員連接到 Azure 監視器
您可以將 Microsoft Endpoint Configuration Manager 環境連線到 Azure 監視器，以同步處理裝置集合資料，並在 Azure 監視器和 Azure 自動化中參考這些集合。  

> [!IMPORTANT]
> 從設定管理員版本2010開始，此功能已被取代。<!-- 8269855 --> 如需詳細資訊，請參閱[已移除且淘汰的設定管理員功能](/mem/configmgr/core/plan-design/changes/deprecated/removed-and-deprecated-cmfeatures)。

## <a name="prerequisites"></a>先決條件

Azure 監視器支援設定管理員最新分支1606版和更高版本。

>[!NOTE]
>將設定管理員與 Log Analytics 工作區連線的功能是選擇性的，預設不會啟用。 您必須先先啟用這項功能才能使用它。 如需詳細資訊，請參閱[從更新啟用選擇性功能](/configmgr/core/servers/manage/install-in-console-updates#bkmk_options)。

## <a name="configuration-overview"></a>組態概觀

下列步驟摘要說明設定設定管理員與 Azure 監視器整合的步驟。  

1. 在 Azure Active Directory 中，將設定管理員註冊為 Web 應用程式和/或 Web API 應用程式，並確保您擁有來自 Azure Active Directory 註冊的用戶端識別碼和用戶端秘密金鑰。 如需如何完成此步驟的詳細資訊，請參閱[使用入口網站來建立可存取資源的 Active Directory 應用程式和服務主體](../../active-directory/develop/howto-create-service-principal-portal.md)。

2. 在 Azure Active Directory 中， [授與設定管理員 (註冊的 web 應用程式) 具有存取 Azure 監視器的許可權](#grant-configuration-manager-with-permissions-to-log-analytics)。

3. 在設定管理員中，使用 **Azure 服務** 嚮導來新增連接。

4. 在執行設定管理員服務連接點網站系統角色的電腦上[，下載並安裝適用于 Windows 的 Log Analytics 代理程式](#download-and-install-the-agent)。 代理程式會將設定管理員資料傳送至 Azure 監視器中的 Log Analytics 工作區。

5. 在 Azure 監視器中， [從設定管理員將集合匯入](#import-collections) 為電腦群組。

6. 在 Azure 監視器中，以 [電腦群組](computer-groups.md)的形式來查看設定管理員的資料。

## <a name="grant-configuration-manager-with-permissions-to-log-analytics"></a>對 Configuration Manager 授與 Log Analytics 的權限

在下列程序中，您會將 Log Analytics 工作區中的「參與者」角色授與給您稍早為 Configuration Manager 建立的 AD 應用程式和服務主體。 如果您還沒有工作區，請參閱 [在 Azure 監視器中建立工作區](../learn/quick-create-workspace.md) ，然後再繼續進行。 這可讓 Configuration Manager 進行驗證並連線到 Log Analytics 工作區。  

> [!NOTE]
> 您必須在 Log Analytics 工作區中指定設定管理員的許可權。 否則，當您在 Configuration Manager 中使用組態精靈時，將會收到錯誤訊息。
>

1. 在 Azure 入口網站中，按一下左上角的 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics]。

2. 在 Log Analytics 工作區清單中，選取要修改的工作區。

3. 從左窗格中，選取 [存取控制 (IAM)]。

4. 在 [存取控制 (IAM)] 頁面上，按一下 [新增角色指派]，[新增角色指派] 窗格隨即出現。

5. 在 [新增角色指派] 窗格中，於 [角色] 下拉式清單中選取 [參與者] 角色。  

6. 在 [指派存取權的對象] 下拉式清單底下，選取稍早在 AD 中建立的 Configuration Manager 應用程式，然後按一下 [確定]。  

## <a name="download-and-install-the-agent"></a>下載並安裝代理程式

請參閱將 [Windows 電腦連線到 Azure 中的 Azure 監視器](agent-windows.md) 一文，以瞭解在裝載設定管理員服務連接點網站系統角色的電腦上安裝適用于 Windows 的 Log Analytics 代理程式的方法。  

## <a name="connect-configuration-manager-to-log-analytics-workspace"></a>將設定管理員連線到 Log Analytics 工作區

>[!NOTE]
> 為了新增 Log Analytics 連線，您的設定管理員環境必須有設定為線上模式的 [服務連接點](/configmgr/core/servers/deploy/configure/about-the-service-connection-point) 。

> [!NOTE]
> 您必須將階層中的頂層網站連線到 Azure 監視器。 如果您將獨立主要網站連接到 Azure 監視器，然後將管理中心網站新增至您的環境，您必須刪除並重新建立新階層內的連線。

1. 在設定管理員的 [系統 **管理** ] 工作區中，選取 [雲端 **服務** ]，然後選取 [ **Azure 服務**]。 

2. 以滑鼠右鍵按一下 [ **Azure 服務** ]，然後選取 [ **設定 azure 服務**]。 [ **設定 Azure 服務** ] 頁面隨即出現。 
   
3. 在 [一般] 畫面上，確認您已完成下列動作而且您有每個項目的詳細資料，然後選取 [下一步]。

4. 在 Azure 服務精靈的 [Azure 服務]頁面上：

    1. 為 Configuration Manager 中的物件指定 [名稱]。
    2. 指定選用 [描述] 以協助您識別服務。
    3. 選取 Azure 服務 **OMS 連接器**。

    >[!NOTE]
    >OMS 現在稱為 Log Analytics，這是 Azure 監視器的功能。

5. 選取 [下一步]，繼續前往 [Azure 服務精靈] 的 Azure 應用程式內容頁面。

6. 在 Azure 服務嚮導的 [ **應用程式** ] 頁面上，先從清單中選取 Azure 環境，然後按一下 [匯 **入**]。

7. 在 [匯 **入應用程式** ] 頁面上，指定下列資訊：

    1. 指定應用程式的 **Azure AD 租使用者名稱** 。

    2. 為 Azure AD 租使用者指定 **Azure AD 租使用者識別碼** 。 您可以在 [Azure Active Directory **屬性** ] 頁面上找到此資訊。 

    3. 指定應用程式 **名稱的應用程式名稱** 。

    4. 針對 [ **用戶端識別碼**]，指定先前建立的 Azure AD 應用程式的應用程式識別碼。

    5. 指定 **秘密金鑰**，也就是所建立 Azure AD 應用程式的用戶端秘密金鑰。

    6. 指定 **秘密金鑰到期** 日、金鑰的到期日。

    7. 針對 [ **應用程式識別碼 uri**]，指定先前建立的 Azure AD 應用程式的 [應用程式識別碼 uri]。

    8. 選取 [ **驗證** ]，右邊的結果應該會顯示 [ **已成功驗證]！**。

8. **在 [設定] 頁面上**，檢查資訊以確認 **azure** 訂用帳戶、 **Azure 資源群組** 和 **Operations Management Suite 工作區** 欄位已預先填入，指出 Azure AD 應用程式在資源群組中具有足夠的許可權。 如果欄位是空的，則表示您的應用程式沒有必要的許可權。 選取要收集並轉寄至工作區的裝置集合，然後選取 [ **新增**]。

9. 請檢查 [ **確認設定** ] 頁面上的選項，然後選取 **[下一步]** ，開始建立及設定連接。

10. 設定完成時，[ **完成** ] 頁面隨即出現。 選取 [關閉]。 

將設定管理員連結至 Azure 監視器之後，您可以新增或移除集合，並查看連接的屬性。

## <a name="update-log-analytics-workspace-connection-properties"></a>更新 Log Analytics 工作區連接屬性

如果密碼或用戶端秘密金鑰過期或遺失，您必須手動更新 Log Analytics 連接屬性。

1. 在設定管理員的 [系統 **管理** ] 工作區中，選取 [ **雲端服務** ]，然後選取 [ **oms 連接器** ] 以開啟 [ **oms 連線屬性** ] 頁面。
2. 在此頁面上按一下 [Azure Active Directory] 索引標籤，檢視 [租用戶]、[用戶端識別碼] 和 [用戶端秘密金鑰到期]。 **確認****用戶端秘密金鑰** 是否已過期。

## <a name="import-collections"></a>匯入集合

將 Log Analytics 連線新增至設定管理員，並在執行設定管理員服務連接點網站系統角色的電腦上安裝代理程式之後，下一個步驟是從 Azure 監視器中的設定管理員將集合匯入為電腦群組。

在您完成初始設定以從階層匯入裝置集合之後，系統會每3小時取出一次集合資訊，以保持成員資格的最新。 您可以隨時選擇停用此功能。

1. 在 Azure 入口網站中，按一下左上角的 [所有服務]。 在資源清單中輸入 **Log Analytics**。 當您開始輸入時，清單會根據您輸入的文字進行篩選。 選取 [Log Analytics 工作區]。
2. 在您的 Log Analytics 工作區清單中，選取 Configuration Manager 用來註冊的工作區。  
3. 選取 [進階設定]  。
4. 依序選取 [電腦群組] 和 [SCCM]。  
5. 選取 [匯入 Configuration Manager 集合成員資格]，然後按一下 [儲存]。  
   
    ![C # M 電腦群組 advanced 設定的螢幕擷取畫面，其中包含匯入設定管理員集合成員資格的選項。](./media/collect-sccm/sccm-computer-groups01.png)

## <a name="view-data-from-configuration-manager"></a>從 Configuration Manager 檢視資料

將 Log Analytics 連線新增至設定管理員，並在執行設定管理員服務連接點網站系統角色的電腦上安裝代理程式之後，代理程式的資料會傳送至 Azure 監視器中的 Log Analytics 工作區。 在 Azure 監視器中，您的設定管理員集合會顯示為 [ [電腦群組](./computer-groups.md)]。 您可以從 [設定\電腦群組] 下的 [Configuration Manager] 頁面檢視群組。

匯入集合之後，您可以看到已偵測到多少部具有集合成員資格的電腦。 您也可以看到已匯入的集合數目。

![[電腦群組的 advanced settings] 的螢幕擷取畫面，顯示已選取 [匯入設定管理員集合成員資格] 的選項。](./media/collect-sccm/sccm-computer-groups02.png)

當您按一下其中一個時，[記錄查詢編輯器] 就會開啟，其中顯示所有已匯入的群組，或屬於每個群組的所有電腦。 您可以使用 [ [記錄搜尋](../log-query/log-query-overview.md)]，進一步對集合成員資格資料進行深入分析。

## <a name="next-steps"></a>後續步驟

請使用 [記錄檔搜尋](../log-query/log-query-overview.md)，檢視有關 Configuration Manager 資料的詳細資訊。

