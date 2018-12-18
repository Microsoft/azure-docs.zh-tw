---
title: Azure Log Analytics 中 IT 服務管理連接器支援的連線 | Microsoft Docs
description: 本文提供如何將 ITSM 產品/服務與 OMS Log Analytics 中的 IT Service Management Connector (ITSMC) 連線，以集中地監視及管理 ITSM 工作項目的相關資訊。
documentationcenter: ''
author: jyothirmaisuri
manager: riyazp
editor: ''
ms.assetid: 8231b7ce-d67f-4237-afbf-465e2e397105
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/24/2018
ms.author: v-jysur
ms.component: na
ms.openlocfilehash: 661107779b74b6e21dec01aecf6d545ec2b7a702
ms.sourcegitcommit: 5892c4e1fe65282929230abadf617c0be8953fd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/29/2018
ms.locfileid: "37131747"
---
# <a name="connect-itsm-productsservices-with-it-service-management-connector"></a>將 ITSM 產品/服務與 IT Service Management Connector 連線
本文提供如何設定 ITSM 產品/服務與 Log Analytics 中 IT Service Management Connector (ITSMC) 之間的連線，以集中管理工作項目的相關資訊。 如需 ITSMC 的詳細資訊，請參閱[概觀](log-analytics-itsmc-overview.md)。

支援下列 ITSM 產品/服務。 選取產品來檢視如何將該產品連線到 ITSMC 的詳細資訊。

- [System Center Service Manager](#connect-system-center-service-manager-to-it-service-management-connector-in-azure)
- [ServiceNow](#connect-servicenow-to-it-service-management-connector-in-azure)
- [Provance](#connect-provance-to-it-service-management-connector-in-azure)
- [Cherwell](#connect-cherwell-to-it-service-management-connector-in-azure)

> [!NOTE]

> ITSM Connector 只能連線到雲端式 ServiceNow 執行個體。 目前不支援內部部署 ServiceNow 執行個體。

## <a name="connect-system-center-service-manager-to-it-service-management-connector-in-azure"></a>將 System Center Service Manager 連線到 Azure 中的 IT Service Management Connector

下列各節提供有關如何將 System Center Service Manager 產品連線到 Azure 中的 ITSMC 之詳細資料。

### <a name="prerequisites"></a>先決條件

請確保已符合下列必要條件︰

- 已安裝 ITSMC。 更多資訊：[新增 IT 服務管理連接器解決方案](log-analytics-itsmc-overview.md#adding-the-it-service-management-connector-solution)。
- 已部署及設定 Service Manager Web 應用程式 (Web 應用程式)。 Web 應用程式的相關在[這裡](#create-and-deploy-service-manager-web-app-service)。
- 已建立及設定的混合式連線。 詳細資訊︰[設定混合式連線](#configure-the-hybrid-connection)。
- Service Manager 的支援版本：2012 R2 或 2016。
- 使用者角色︰[進階操作員](https://technet.microsoft.com/library/ff461054.aspx)。

### <a name="connection-procedure"></a>連線程序

您可以使用下列程序，將 System Center Service Manager 執行個體連線到 ITSMC：

1. 在 Azure 入口網站中，移至 [所有資源]，然後尋找 **ServiceDesk(YourWorkspaceName)**

2.  在 [工作區資料來源] 下方，按一下 [ITSM 連線]。

    ![新增連線](./media/log-analytics-itsmc/add-new-itsm-connection.png)

3. 在右窗格頂端按一下 [新增]。

4. 提供下表中所述的資訊，然後按一下 [確定] 來建立連線。

> [!NOTE]

> 這些全部都是必要參數。

| **欄位** | **說明** |
| --- | --- |
| 連線名稱   | 輸入您想要與 ITSMC 連線之 System Center Service Manager 執行個體的名稱。  稍後當您設定這個執行個體的工作項目/檢視詳細的記錄分析時，會使用這個名稱。 |
| **夥伴類型**   | 選取 **System Center Service Manager**。 |
| **伺服器 URL**   | 輸入 Service Manager Web 應用程式的 URL。 Service Manager Web 應用程式的相關詳細資訊在[這裡](#create-and-deploy-service-manager-web-app-service)。
| **用戶端識別碼**   | 將您所產生 (使用自動指令碼) 用來驗證 Web 應用程式的用戶端識別碼輸入。 自動化指令碼的相關詳細資訊在[這裡](log-analytics-itsmc-service-manager-script.md)。|
| **用戶端祕密**   | 輸入針對此識別碼產生的用戶端祕密。   |
| **資料同步範圍**   | 選取您想要透過 ITSMC 同步的 Service Manager 工作項目。  系統會將這些工作項目匯入 Log Analytics。 **選項︰** 事件、變更要求。|
| **同步資料** | 輸入您想要起算資料的過去天數。 **上限**：120 天。 |
| **在 ITSM 解決方案中建立新的設定項目** | 如果您想要在 ITSM 產品中建立設定項目，請選取此選項。 選取時，OMS 會在支援的 ITSM 系統中建立受影響的 CI 作為設定項目 (如果 CI 不存在)。 **預設**︰停用。 |

![Service Manager 連線](./media/log-analytics-itsmc/service-manager-connection.png)

**順利連線並同步處理時**︰

- 從 Service Manager 選取的工作項目會匯入到 Azure **Log Analytics**。 您可以在 [IT 服務管理連接器] 圖格上檢視這些工作項目的摘要。

- 您可以在這個 Service Manager 執行個體中建立來自 Log Analytics 警示、記錄檔記錄或 Azure 警示的事件。


深入了解： [從 Azure 警示建立 ITSM 工作項目](log-analytics-itsmc-overview.md#create-itsm-work-items-from-azure-alerts)。

### <a name="create-and-deploy-service-manager-web-app-service"></a>建立及部署 Service Manager Web 應用程式服務

為了將內部部署 Service Manager 與 Azure 中的 ITSMC 連線，Microsoft 已在 GitHub 上建立 Service Manager Web 應用程式。

若要為您的 Service Manager 設定 ITSM Web 應用程式，請執行下列作業︰

- **部署 Web 應用程式** – 部署 Web 應用程式、設定屬性，以及驗證 Azure AD。 您可以使用 Microsoft 所提供給您的[自動化指令碼](log-analytics-itsmc-service-manager-script.md)來部署 Web 應用程式。
- **設定混合式連線** - [以手動方式設定此連線](#configure-the-hybrid-connection)。

#### <a name="deploy-the-web-app"></a>部署 Web 應用程式
使用自動化[指令碼](log-analytics-itsmc-service-manager-script.md)來部署 Web 應用程式、設定屬性，以及驗證 Azure AD。

提供下列必要的詳細資料來執行指令碼︰

- Azure 訂用帳戶詳細資料
- 資源群組名稱
- 位置
- Service Manager 伺服器詳細資料 (伺服器名稱、網域、使用者名稱和密碼)
- Web 應用程式的網站名稱前置詞
- 服務匯流排命名空間。

指令碼會使用您指定的名稱 (與其他可使它成為唯一的字串) 來建立 Web 應用程式。 它會產生 **Web 應用程式 URL**、**用戶端識別碼**和**用戶端祕密**。

將值儲存，當您使用 ITSMC 建立連線時會用到這些值。

**檢查 Web 應用程式安裝**

1. 移至 [Azure 入口網站] > [資源]。
2. 選取 Web 應用程式，按一下 [設定] > [應用程式設定]。
3. 確認您在透過指令碼部署應用程式時所提供的 Service Manager 執行個體之相關資訊。

### <a name="configure-the-hybrid-connection"></a>設定混合式連線

您可以使用下列程序，設定將 Service Manager 執行個體與 Azure 中的 ITSMC 連線之混合式連線。

1. 在 **Azure 資源**下，尋找 Service Manager Web 應用程式。
2. 按一下 [設定] > [網路]。
3. 在 [混合式連線] 下，按一下 [設定混合式連線端點]。

    ![混合式連線網路](./media/log-analytics-itsmc/itsmc-hybrid-connection-networking-and-end-points.png)
4. 在 [混合式連線] 刀鋒視窗上，按一下 [新增混合式連線]。

    ![混合式連線新增](./media/log-analytics-itsmc/itsmc-new-hybrid-connection-add.png)

5. 在 [新增混合式連線] 刀鋒視窗上，按一下 [建立新的混合式連線]。

    ![新增混合式連線](./media/log-analytics-itsmc/itsmc-create-new-hybrid-connection.png)

6. 輸入下列值：

    - **端點名稱**︰指定新混合式連線的名稱。
    -  **端點主機**︰Service Manager 管理伺服器的 FQDN。
    - **端點連接埠**：輸入 5724
    - **服務匯流排命名空間**︰使用現有的或建立一個新的服務匯流排命名空間。
    - **位置**：選取位置。
    -  **名稱**︰如果您要建立服務匯流排，請為它指定名稱。

    ![混合式連線值](./media/log-analytics-itsmc/itsmc-new-hybrid-connection-values.png)
6. 按一下 [確定] 將 [建立混合式連線] 刀鋒視窗關閉，然後開始建立混合式連線。

    建立混合式連線之後，它會顯示在刀鋒視窗下。

7. 建立混合式連線之後，請選取連線，然後按一下 [新增所選的混合式連線]。

    ![新增混合式連線](./media/log-analytics-itsmc/itsmc-new-hybrid-connection-added.png)

#### <a name="configure-the-listener-setup"></a>設定接聽程式設定

您可以使用下列程序來設定混合式連線的接聽程式設定。

1. 在 [混合式連線] 刀鋒視窗中，按 [下載連線管理員]，並將它安裝在執行 System Center Service Manager 執行個體的電腦上。

    一旦安裝完成後，可在 [啟動] 功能表下使用 [混合式連線管理員 UI]選項。

2. 按一下 [混合式連線管理員 UI]，系統會提示您輸入 Azure 認證。

3. 使用您的 Azure 認證登入，然後選取您在其中建立混合式連線的訂用帳戶。

4. 按一下 [檔案] 。

混合式連線已成功連線。

![混合式連線成功](./media/log-analytics-itsmc/itsmc-hybrid-connection-listener-set-up-successful.png)
> [!NOTE]

> 建立混合式連線之後，可瀏覽已部署的 Service Manager Web 應用程式來驗證及測試連線。 嘗試連線到 Azure 中的 ITSMC 之前，請確定連線成功。

以下範例影像顯示連線成功的詳細資料︰

![混合式連線測試](./media/log-analytics-itsmc/itsmc-hybrid-connection-test.png)

## <a name="connect-servicenow-to-it-service-management-connector-in-azure"></a>將 ServiceNow 連線到 Azure 中的 IT Service Management Connector

下列各節提供有關如何將 ServiceNow 產品連線到 Azure 中的 ITSMC 之詳細資料。

### <a name="prerequisites"></a>先決條件
請確保已符合下列必要條件︰
- 已安裝 ITSMC。 更多資訊：[新增 IT 服務管理連接器解決方案](log-analytics-itsmc-overview.md#adding-the-it-service-management-connector-solution)。
- ServiceNow 支援的版本：Kingston、Jakarta、Istanbul、Helsinki、Geneva。

**ServiceNow 管理員必須在 ServiceNow 執行個體中執行下列動作**：
- 產生 ServiceNow 產品的用戶端識別碼和用戶端密碼。 如需如何產生用戶端識別碼和祕密的相關資訊，請視需要參閱下列資訊：

    - [針對 Kingston 設定 OAuth](https://docs.servicenow.com/bundle/kingston-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [針對 Jakarta 設定 OAuth](https://docs.servicenow.com/bundle/jakarta-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [針對 Istanbul 設定 OAuth](https://docs.servicenow.com/bundle/istanbul-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [針對 Helsinki 設定 OAuth](https://docs.servicenow.com/bundle/helsinki-platform-administration/page/administer/security/task/t_SettingUpOAuth.html)
    - [針對 Geneva 設定 OAuth](https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/administer/security/task/t_SettingUpOAuth.html)


- 安裝適用於 Microsoft OMS 整合的使用者應用程式 (ServiceNow 應用程式)。 [深入了解](https://store.servicenow.com/sn_appstore_store.do#!/store/application/ab0265b2dbd53200d36cdc50cf961980/1.0.1 )。
- 為安裝的使用者應用程式建立整合使用者角色。 關於如何建立整合使用者角色的資訊在[這裡](#create-integration-user-role-in-servicenow-app)。

### <a name="connection-procedure"></a>**連線程序**
請使用下列程序來建立 ServiceNow 連線：


1. 在 Azure 入口網站中，移至 [所有資源]，然後尋找 **ServiceDesk(YourWorkspaceName)**

2.  在 [工作區資料來源] 下方，按一下 [ITSM 連線]。
    ![新增連線](./media/log-analytics-itsmc/add-new-itsm-connection.png)

3. 在右窗格頂端按一下 [新增]。

4. 提供下表中所述的資訊，然後按一下 [確定] 來建立連線。


> [!NOTE]
> 這些全部都是必要參數。

| **欄位** | **說明** |
| --- | --- |
| 連線名稱   | 輸入您想要與 ITSMC 連線之 ServiceNow 執行個體的名稱。  稍後當您在 OMS 中設定這個 ITSM 的工作項目/檢視詳細的記錄分析時，會使用這個名稱。 |
| **夥伴類型**   | 選取 **ServiceNow**。 |
| **使用者名稱**   | 輸入您在 ServiceNow 應用程式中建立的整合使用者名稱，以支援 ITSMC 的連線。 詳細資訊︰[建立 ServiceNow 應用程式使用者角色](#create-integration-user-role-in-servicenow-app)。|
| **密碼**   | 將與此使用者名稱與相關聯的密碼輸入。 **附註**︰使用者名稱和密碼僅用來產生驗證權杖，並不會儲存在 ITSMC 服務內。  |
| **伺服器 URL**   | 輸入您想要連線到 ITSMC 之 ServiceNow 執行個體的 URL。 |
| **用戶端識別碼**   | 將您想要用於先前產生之 OAuth2 驗證的用戶端識別碼輸入。  如需產生用戶端識別碼和祕密的資訊：[OAuth 設定](http://wiki.servicenow.com/index.php?title=OAuth_Setup)。 |
| **用戶端祕密**   | 輸入針對此識別碼產生的用戶端祕密。   |
| **資料同步範圍**   | 選取您想要透過 ITSMC 同步處理到 Azure Log Analytics 的 ServiceNow 工作項目。  系統會將這些值匯入 Log Analytics。   **選項︰** 事件和變更要求。|
| **同步資料** | 輸入您想要起算資料的過去天數。 **上限**：120 天。 |
| **在 ITSM 解決方案中建立新的設定項目** | 如果您想要在 ITSM 產品中建立設定項目，請選取此選項。 選取時，ITSMC 會在支援的 ITSM 系統中建立受影響的 CI 作為設定項目 (如果 CI 不存在)。 **預設**︰停用。 |

![ServiceNow 連線](./media/log-analytics-itsmc/itsm-connection-servicenow-connection-latest.png)

**順利連線並同步處理時**︰

- 從 ServiceNow 執行個體選取的工作項目會匯入到 Azure **Log Analytics**。 您可以在 [IT 服務管理連接器] 圖格上檢視這些工作項目的摘要。

- 您可以在這個 ServiceNow 執行個體中建立來自 Log Analytics 警示、記錄檔記錄或 Azure 警示的事件。

深入了解： [從 Azure 警示建立 ITSM 工作項目](log-analytics-itsmc-overview.md#create-itsm-work-items-from-azure-alerts)。

### <a name="create-integration-user-role-in-servicenow-app"></a>在 ServiceNow 應用程式中建立整合使用者角色

請使用下列程序：

1.  請瀏覽 [ServiceNow 存放區](https://store.servicenow.com/sn_appstore_store.do#!/store/application/ab0265b2dbd53200d36cdc50cf961980/1.0.1)，並將 **ServiceNow 和 Microsoft OMS 整合的使用者應用程式**安裝到 ServiceNow 執行個體。
2.  安裝完成後，請瀏覽左側導覽列中的 [ServiceNow 執行個體]、[搜尋] 並選取 [Microsoft OMS 整合器]。  
3.  按一下 [安裝檢查清單]。

    如果尚未建立使用者角色，則狀態會顯示為 [未完成]。

4.  在 [建立整合使用者] 旁邊的文字方塊中，輸入可連線到 Azure 中 ITSMC 之使用者的使用者名稱。
5.  輸入這個使用者的密碼，然後按一下 [確定]。  

>[!NOTE]

> 您可以使用這些認證，在 Azure 中進行 ServiceNow 連線。

會使用預設的角色指派來顯示新建立的使用者。

**預設角色**︰
- personalize_choices
- import_transformer
-   x_mioms_microsoft.user
-   itil
-   template_editor
-   view_changer

成功建立使用者之後，[檢查安裝檢查清單] 狀態會改變為 [已完成]，並列出針對應用程式所建立的使用者角色詳細資料。

> [!NOTE]

> ITSM 連接器可以將事件傳送給 ServiceNow，完全不需要在您的 ServiceNow 執行個體上安裝其他任何模組。 如果您在 ServiceNow 執行個體中使用 EventManagement 模組，而且想要使用連接器在 ServiceNow 中建立事件或警示，請將下列角色加入至整合使用者：

>    - evt_mgmt_integration
>    - evt_mgmt_operator  


## <a name="connect-provance-to-it-service-management-connector-in-azure"></a>將 Provance 連線到 Azure 中的 IT Service Management Connector

下列各節提供有關如何將 Provance 產品連線到 Azure 中的 ITSMC 之詳細資料。


### <a name="prerequisites"></a>先決條件

請確保已符合下列必要條件︰


- 已安裝 ITSMC。 更多資訊：[新增 IT 服務管理連接器解決方案](log-analytics-itsmc-overview.md#adding-the-it-service-management-connector-solution)。
- 應該向 Azure AD 註冊 Provance 應用程式 - 並將用戶端識別碼設為可用。 如需詳細資訊，請參閱[如何設定 Active Directory 驗證](../app-service-mobile/app-service-mobile-how-to-configure-active-directory-authentication.md)。

- 使用者角色：管理員。

### <a name="connection-procedure"></a>連線程序

請使用下列程序來建立 Provance 連線：

1. 在 Azure 入口網站中，移至 [所有資源]，然後尋找 **ServiceDesk(YourWorkspaceName)**

2.  在 [工作區資料來源] 下方，按一下 [ITSM 連線]。
    ![新增連線](./media/log-analytics-itsmc/add-new-itsm-connection.png)

3. 在右窗格頂端按一下 [新增]。

4. 提供下表中所述的資訊，然後按一下 [確定] 來建立連線。

> [!NOTE]

> 這些全部都是必要參數。

| **欄位** | **說明** |
| --- | --- |
| 連線名稱   | 輸入您想要與 ITSMC 連線之 Provance 執行個體的名稱。  稍後當您在這個 ITSM 中設定工作項目 / 檢視詳細的記錄分析時，會使用這個名稱。 |
| **夥伴類型**   | 選取 [Provance]。 |
| **使用者名稱**   | 輸入可以連線到 ITSMC 的使用者名稱。    |
| **密碼**   | 將與此使用者名稱與相關聯的密碼輸入。 **附註**︰使用者名稱和密碼僅用來產生驗證權杖，並不會儲存在 ITSMC 服務內。|
| **伺服器 URL**   | 輸入您想要連線到 ITSMC 之 Provance 執行個體的 URL。 |
| **用戶端識別碼**   | 將您在 Provance 執行個體中產生的用戶端識別碼輸入以驗證此連線。  如需用戶端識別碼的詳細資訊，請參閱[如何設定 Active Directory 驗證](../app-service/app-service-mobile-how-to-configure-active-directory-authentication.md)。 |
| **資料同步範圍**   | 選取您想要透過 ITSMC 同步處理到 Azure Log Analytics 的 Provance 工作項目。  系統會將這些工作項目匯入 Log Analytics。   **選項︰** 事件、變更要求。|
| **同步資料** | 輸入您想要起算資料的過去天數。 **上限**：120 天。 |
| **在 ITSM 解決方案中建立新的設定項目** | 如果您想要在 ITSM 產品中建立設定項目，請選取此選項。 選取時，ITSMC 會在支援的 ITSM 系統中建立受影響的 CI 作為設定項目 (如果 CI 不存在)。 **預設**︰停用。|

![Provance 連線](./media/log-analytics-itsmc/itsm-connections-provance-latest.png)

**順利連線並同步處理時**︰

- 從這個 Provance 執行個體選取的工作項目會匯入到 Azure **Log Analytics**。 您可以在 [IT 服務管理連接器] 圖格上檢視這些工作項目的摘要。

- 您可以在這個 Provance 執行個體中建立來自 Log Analytics 警示、記錄檔記錄或 Azure 警示的事件。

深入了解： [從 Azure 警示建立 ITSM 工作項目](log-analytics-itsmc-overview.md#create-itsm-work-items-from-azure-alerts)。

## <a name="connect-cherwell-to-it-service-management-connector-in-azure"></a>將 Cherwell 連線到 Azure 中的 IT Service Management Connector

下列各節提供有關如何將 Cherwell 產品連線到 Azure 中的 ITSMC 之詳細資料。

### <a name="prerequisites"></a>先決條件

請確保已符合下列必要條件︰

- 已安裝 ITSMC。 更多資訊：[新增 IT 服務管理連接器解決方案](log-analytics-itsmc-overview.md#adding-the-it-service-management-connector-solution)。
- 所產生的用戶端識別碼。 詳細資訊︰[產生 Cherwell 的用戶端識別碼](#generate-client-id-for-cherwell)。
- 使用者角色：管理員。

### <a name="connection-procedure"></a>連線程序

請使用下列程序來建立 Provance 連線：

1. 在 Azure 入口網站中，移至 [所有資源]，然後尋找 **ServiceDesk(YourWorkspaceName)**

2.  在 [工作區資料來源] 下方，按一下 [ITSM 連線]。
    ![新增連線](./media/log-analytics-itsmc/add-new-itsm-connection.png)

3. 在右窗格頂端按一下 [新增]。

4. 提供下表中所述的資訊，然後按一下 [確定] 來建立連線。

> [!NOTE]

> 這些全部都是必要參數。

| **欄位** | **說明** |
| --- | --- |
| 連線名稱   | 輸入您想要與 ITSMC 連線之 Cherwell 執行個體的名稱。  稍後當您在這個 ITSM 中設定工作項目 / 檢視詳細的記錄分析時，會使用這個名稱。 |
| **夥伴類型**   | 選取 [Cherwell]。 |
| **使用者名稱**   | 輸入可以連線到 ITSMC 的 Cherwell 使用者名稱。 |
| **密碼**   | 將與此使用者名稱與相關聯的密碼輸入。 **附註**︰使用者名稱和密碼僅用來產生驗證權杖，並不會儲存在 ITSMC 服務內。|
| **伺服器 URL**   | 輸入您想要連線到 ITSMC 之 Cherwell 執行個體的 URL。 |
| **用戶端識別碼**   | 將您在 Cherwell 執行個體中產生的用戶端識別碼輸入以驗證此連線。   |
| **資料同步範圍**   | 選取您想要透過 ITSMC 同步的 Cherwell 工作項目。  系統會將這些工作項目匯入 Log Analytics。   **選項︰** 事件、變更要求。 |
| **同步資料** | 輸入您想要起算資料的過去天數。 **上限**：120 天。 |
| **在 ITSM 解決方案中建立新的設定項目** | 如果您想要在 ITSM 產品中建立設定項目，請選取此選項。 選取時，ITSMC 會在支援的 ITSM 系統中建立受影響的 CI 作為設定項目 (如果 CI 不存在)。 **預設**︰停用。 |


![Provance 連線](./media/log-analytics-itsmc/itsm-connections-cherwell-latest.png)

**順利連線並同步處理時**︰

- 從這個 Cherwell 執行個體選取的工作項目會匯入到 Azure **Log Analytics**。 您可以在 [IT 服務管理連接器] 圖格上檢視這些工作項目的摘要。

- 您可以在這個 Cherwell 執行個體中建立來自 Log Analytics 警示、記錄檔記錄或 Azure 警示的事件。

深入了解： [從 Azure 警示建立 ITSM 工作項目](log-analytics-itsmc-overview.md#create-itsm-work-items-from-azure-alerts)。

### <a name="generate-client-id-for-cherwell"></a>產生 Cherwell 的用戶端識別碼

若要產生用戶端識別碼/Cherwell 的金鑰，請使用下列程序︰

1. 以管理員身分登入您的 Cherwell 執行個體。
2. 按一下 [安全性] > [編輯 REST API 用戶端設定]。
3. 選取 [建立新的用戶端] > [用戶端祕密]。

    ![Cherwell 使用者識別碼](./media/log-analytics-itsmc/itsmc-cherwell-client-id.png)


## <a name="next-steps"></a>後續步驟
 - [建立 Azure 警示的 ITSM 工作項目](log-analytics-itsmc-overview.md#create-itsm-work-items-from-azure-alerts)
