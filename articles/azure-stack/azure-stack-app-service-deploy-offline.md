---
title: 在 Azure Stack 的離線環境中部署 App Service | Microsoft Docs
description: 如何在中斷連線且受 AD FS 保護的 Azure Stack 環境中部署 App Service 的詳細指導方針。
services: azure-stack
documentationcenter: ''
author: apwestgarth
manager: stefsch
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: app-service
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/15/2018
ms.author: anwestg
ms.openlocfilehash: 9e36e470c3516c55089ce1e44540b6b1eacbb6b2
ms.sourcegitcommit: 744747d828e1ab937b0d6df358127fcf6965f8c8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/16/2018
ms.locfileid: "41954839"
---
# <a name="add-an-app-service-resource-provider-to-a-disconnected-azure-stack-environment-secured-by-ad-fs"></a>將 App Service 資源提供者新增至中斷連線且受 AD FS 保護的 Azure Stack 環境

*適用於：Azure Stack 整合系統和 Azure Stack 開發套件*

> [!IMPORTANT]
> 在部署 Azure App Service 1.3 之前，請先將 1807 更新套用到您的 Azure Stack 整合式系統，或部署最新的 Azure Stack 開發套件。
>
>

您可以遵循本文章中的指示，安裝 [App Service 資源提供者](azure-stack-app-service-overview.md)至以下 Azure Stack 環境：

- 未連線至網際網路
- 受 Active Directory 同盟服務 (AD FS) 保護。

若要將 App Service 資源提供者新增至您的離線 Azure Stack 部署，您必須完成三個最上層工作：

1. 完成[先決條件步驟](azure-stack-app-service-before-you-get-started.md) (例如：購買憑證可能需要幾天才會收到憑證)。
2. [將安裝與協助程式檔案下載並解壓縮](azure-stack-app-service-before-you-get-started.md)至已連線到網際網路的電腦。
3. 建立離線安裝套件。
4. 執行 appservice.exe 安裝程式檔案。

## <a name="create-an-offline-installation-package"></a>建立離線安裝套件

若要在中斷連線的環境中部署 App Service，您必須先在連線到網際網路的電腦建立離線安裝套件。

1. 在連線到網際網路的電腦上執行 AppService.exe 安裝程式。

2. 按一下 [進階] >  [建立離線安裝套件]。

    ![App Service 安裝程式][1]

3. App Service 安裝程式會建立離線安裝套件並顯示其路徑。 您可以按一下 [開啟資料夾]，在檔案總管中開啟資料夾。

    ![App Service 安裝程式](media/azure-stack-app-service-deploy-offline/image02.png)

4. 將安裝程式 (AppService.exe) 和離線安裝套件複製到您的 Azure Stack 主機電腦。

## <a name="complete-the-offline-installation-of-app-service-on-azure-stack"></a>完成 Azure Stack 上的 App Service 離線安裝

1. 從可連線至「Azure Stack 管理」之「Azure 資源管理」端點的電腦，以系統管理員身分執行 appservice.exe。

2. 按一下 [進階] > [完成離線安裝]。

    ![App Service 安裝程式][2]

3. 瀏覽至您先前建立的離線安裝套件位置，然後按 [下一步]。

    ![App Service 安裝程式](media/azure-stack-app-service-deploy-offline/image04.png)

4. 檢閱並接受「Microsoft 軟體授權條款」，然後按 [下一步]。

5. 檢閱並接受協力廠商授權條款，然後按 [下一步]。

6. 請確定 App Service 雲端組態資訊正確。 如果您在 Azure Stack 開發套件部署期間使用了預設設定，在這裡可以接受預設值。 不過，如果您在部署 Azure Stack 時已自訂選項，或是要部署在整合式系統上，則必須編輯此視窗中的值來反映該情況。 例如，如果您使用網域尾碼 mycloud.com，您的 Azure Stack 租用戶 Azure Resource Manager 端點就必須變更為 management.<region>.mycloud.com。 確認您的資訊之後，按 [下一步]。

    ![App Service 安裝程式][3]

7. 在下一個頁面上：
    1. 按一下 [Azure Stack 訂用帳戶] 方塊旁邊的 [連線] 按鈕。
        - 提供您的管理帳戶。 例如： cloudadmin@azurestack.local。 輸入您的密碼，然後按一下 [登入]。
    2. 在 [Azure Stack 訂用帳戶] 方塊中，選取 [預設提供者訂用帳戶]。
    
    > [!NOTE]
    > 目前，App Service 只能部署到**預設提供者訂用帳戶**。  在未來的更新中，App Service 會部署至 Azure Stack 1804 所導入的新計量訂用帳戶，而且現有部署也全都會遷移至這個新的訂用帳戶。
    >
    >
    
    3. 在 [Azure Stack 位置] 方塊中，選取對應到您要部署之區域的位置。 例如，如果要部署至 Azure Stack 開發套件，請選取 [本機]。
    4. 按 [下一步] 。

    ![App Service 安裝程式][4]

8. 您現在可以選擇部署至現有的虛擬網路 (透過[這裡](azure-stack-app-service-before-you-get-started.md#virtual-network)的步驟所設定)，或是允許 App Service 安裝程式建立虛擬網路及關聯的子網路。
    1. 選取 [使用預設設定來建立 VNet]，接受預設值，然後按 [下一步]，或者；
    2. 選取 [使用現有的 VNet 和子網路]。
        1. 選取包含您的虛擬網路的**資源群組**；
        2. 選擇要作為部署目的地的正確**虛擬網路**名稱；
        3. 為每個必要角色子網路選取正確的**子網路**值；
        4. 按一下 [下一步] 

    ![App Service 安裝程式][5]

9. 輸入檔案共用的資訊，然後按 [下一步]。 檔案共用的位址必須使用檔案伺服器的完整網域名稱或 IP 位址。 例如，\\\appservicefileserver.local.cloudapp.azurestack.external\websites 或 \\\10.0.0.1\websites

    > [!NOTE]
    > 在繼續進行之前，安裝程式會先嘗試測試是否能夠與檔案共用連線。  不過，如果您已選擇在現有的虛擬網路中部署，安裝程式可能會無法連線至檔案共用，而會顯示警告來詢問您是否要繼續進行。  請確認檔案共用資訊，如果正確，便繼續進行。
    >
    >

   ![App Service 安裝程式][8]

10. 在下一個頁面上：
    1. 在 [身分識別應用程式識別碼] 方塊中，輸入您要用於身分識別的應用程式 GUID (來自 Azure AD)。
    2. 在 [身分識別應用程式憑證檔案] 方塊中，輸入 (或瀏覽至) 憑證檔案的位置。
    3. 在 [身分識別應用程式憑證密碼] 方塊中，輸入憑證的密碼。 此密碼是當您使用指令碼來建立憑證時所記下的密碼。
    4. 在 [Azure Resource Manager 根憑證檔案] 方塊中，輸入 (或瀏覽至) 憑證檔案的位置。
    5. 按 [下一步] 。

    ![App Service 安裝程式][10]

11. 針對三個憑證檔案方塊中的每一個，按一下 [瀏覽]，然後瀏覽至適當的憑證檔案。 您必須提供每個憑證的密碼。 這些憑證是您在[建立必要的憑證步驟](azure-stack-app-service-before-you-get-started.md#get-certificates)中建立的憑證。 輸入所有資訊之後，按 [下一步]。

    | Box | 憑證檔案名稱範例 |
    | --- | --- |
    | **App Service 預設 SSL 憑證檔案** | \_.appservice.local.AzureStack.external.pfx |
    | **App Service API SSL 憑證檔案** | api.appservice.local.AzureStack.external.pfx |
    | **App Service 發行者 SSL 憑證檔案** | ftp.appservice.local.AzureStack.external.pfx |

    如果您在建立憑證時使用不同的網域尾碼，您的憑證檔案名稱不會使用 *local.AzureStack.external*。 而是會改用您的自訂網域資訊。

    ![App Service 安裝程式][11]

12. 針對用來裝載 App Service 資源提供者資料庫的伺服器執行個體，輸入 SQL Server 詳細資料，然後按 [下一步]。 按 [下一步]，安裝程式即會驗證 SQL 連線屬性。 您**必須**輸入內部 ip 或 SQL Server 名稱的完整網域名稱。

    > [!NOTE]
    > 在繼續進行之前，安裝程式會先嘗試測試是否能夠與 SQL Server 連線。  不過，如果您已選擇在現有的虛擬網路中部署，安裝程式可能會無法連線至 SQL Server，而會顯示警告來詢問您是否要繼續進行。  請確認 SQL Server 資訊，如果正確，便繼續進行。
    >
    > 從 Azure App Service on Azure Stack 1.3 開始，安裝程式將會檢查 SQL Server 是否已在 SQL Server 層級啟用資料庫的內含項目。  如果未啟用，系統將會以下列例外狀況提示您：
    > ```sql
    >    Enable contained database authentication for SQL server by running below command on SQL server (Ctrl+C to copy)
    >    ***********************************************************
    >    sp_configure 'contained database authentication', 1;  
    >    GO  
    >    RECONFIGURE;  
    >    GO
    >    ***********************************************************
    > ```
    > 如需更多詳細資料，請參閱 [Azure App Service on Azure Stack 1.3 的版本資訊](azure-stack-app-service-release-notes-update-three.md)。
   
   ![App Service 安裝程式][12]

13. 檢閱角色執行個體和 SKU 選項。 填入的預設值為「ASDK 部署」中每個角色的執行個體數目下限和最低 SKU。 系統會提供 vCPU 和記憶體的需求摘要，以協助您規劃部署。 進行選擇之後，按一下 [下一步]。

     > [!NOTE]
     > 針對生產環境部署，請依照 [Azure Stack 中的 Azure App Service 伺服器角色容量規劃](azure-stack-app-service-capacity-planning.md)中的指引進行操作。
     >
     >

    | 角色 | 最少執行個體 | 最低 SKU | 注意 |
    | --- | --- | --- | --- |
    | Controller | 1 | Standard_A2 - (2 vCPU, 3584 MB) | 管理及維護 App Service 雲端的健全狀況。 |
    | 管理性 | 1 | Standard_A2 - (2 vCPUs, 3584 MB) | 管理 App Service Azure Resource Manager 和 API 端點、入口網站擴充功能 (管理員、租用戶、Functions 入口網站)，以及資料服務。 為了支援容錯移轉，已將建議的執行個體增加為 2 個。 |
    | 發行者 | 1 | Standard_A1 - (1 vCPU, 1792 MB) | 透過 FTP 和 Web 部署發佈內容。 |
    | FrontEnd | 1 | Standard_A1 - (1 vCPU, 1792 MB) | 將要求傳送至 App Service 應用程式。 |
    | 共用背景工作 | 1 | Standard_A1 - (1 vCPU, 1792 MB) | 裝載 Web 或 API 應用程式和 Azure Functions 應用程式。 建議您新增更多執行個體。 身為操作員，您可以定義您的供應項目，並選擇任何 SKU 層。 各層必須具有至少一個 vCPU。 |

    ![App Service 安裝程式][14]

    > [!NOTE]
    > **Windows Server 2016 Core 不是支援的平台映像，無法與 Azure Stack 上的 Azure App Service 搭配使用。請勿將評估映像用於生產環境部署。Azure App Service on Azure Stack 要求在用於部署的映像上必須啟用 Microsoft.Net 3.5.1 SP1。 市集摘要整合的 Windows Server 2016 映像並未啟用此功能。**

14. 在 [選取平台映像] 方塊中，從可以在適用於 App Service 雲端的運算資源提供者的可用映像中，選擇您的部署 Windows Server 2016 虛擬機器映像。 按 [下一步] 。

15. 在下一個頁面上：
     1. 輸入背景工作角色虛擬機器系統管理員使用者名稱和密碼。
     2. 輸入其他角色的虛擬機器系統管理員使用者名稱和密碼。
     3. 按 [下一步] 。

    ![App Service 安裝程式][16]

16. 在摘要頁面上：
    1. 確認您所做的選擇。 若要進行變更，請使用 [上一步] 按鈕來瀏覽先前的頁面。
    2. 如果設定均正確，請選取核取方塊。
    3. 若要開始部署，請按一下 [下一步]。

    ![App Service 安裝程式][17]

17. 在下一個頁面上：
    1. 追蹤安裝進度。 根據預設的選取項目而定，部署 Azure App Service on Azure Stack 大約需要 60 分鐘。
    2. 當安裝程式順利完成之後，請按一下 [結束]。

    ![App Service 安裝程式][18]

## <a name="validate-the-app-service-on-azure-stack-installation"></a>確認 Azure Stack 上的 App Service 安裝

1. 在 Azure Stack 管理入口網站中，前往 [管理 - App Service]。

2. 在狀態下的概觀，查看 [狀態]是否顯示為 [所有角色都已準備完成]。

    ![App Service 管理](media/azure-stack-app-service-deploy/image12.png)
    
> [!NOTE]
> 如果您已選擇部署到現有的虛擬網路和內部 IP 位址以連線到您的檔案伺服器，便必須新增輸出安全性規則，以在背景工作角色子網路與檔案伺服器之間啟用 SMB 流量。  若要這樣做，請移至管理入口網站中的 WorkersNsg，然後使用下列屬性新增輸出安全性規則：
> * 來源：任何
> * 來源連接埠範圍：*
> * 目的地：IP 位址
> * 目的地 IP 位址範圍：檔案伺服器的 IP 範圍
> * 目的地連接埠範圍：445
> * 通訊協定：TCP
> * 動作：允許
> * 優先順序：700
> * 名稱：Outbound_Allow_SMB445
>

## <a name="test-drive-app-service-on-azure-stack"></a>測試 Azure Stack 上的 App Service

當您部署並註冊 App Service 資源提供者之後，請測試它，以確定使用者可以部署 Web 及 API 應用程式。

> [!NOTE]
> 您需要在方案內建立含有 Microsoft.Web 命名空間的供應項目。 然後，您需要具有訂閱此供應項目的租用戶訂用帳戶。 如需詳細資訊，請參閱[建立供應項目](azure-stack-create-offer.md)和[建立方案](azure-stack-create-plan.md)。
>
您*必須*具有租用戶訂用帳戶，才能建立使用 Azure Stack 上之 App Service 的應用程式。 服務管理員只能在管理入口網站內完成的功能，與 App Service 的資源提供者管理有關。 這些功能包括新增容量、設定部署來源，以及新增背景工作層和 SKU。
>
從第三個技術預覽開始，若要建立 Web、API 及 Azure Functions 應用程式，您必須使用租用戶入口網站，並具有租用戶訂用帳戶。

1. 在 Azure Stack 租用戶入口網站中，按一下 [新增] > [Web + 行動] > [Web 應用程式]。

2. 在 [Web 應用程式] 刀鋒視窗中，於 [Web 應用程式] 方塊中輸入名稱。

3. 按一下 [資源群組] 下方的 [新增]。 在 [資源群組] 方塊中輸入名稱。

4. 按一下 [App Service 方案/位置] > [新建]。

5. 在 [App Service 方案] 刀鋒視窗中，於 [App Service 方案] 方塊中輸入名稱。

6. 按一下 [定價層] > [免費 - 共用] 或 [共用 - 共用] > [選取] > [確定] > [建立]。

7. 新 Web 應用程式的磚隨即出現在儀表板上。 按一下此磚。

8. 在 [Web 應用程式] 刀鋒視窗中，按一下 [瀏覽] 以檢視此應用程式的預設網站。

## <a name="deploy-a-wordpress-dnn-or-django-website-optional"></a>部署 WordPress、DNN 或 Django 網站 (選擇性)

1. 在 Azure Stack 租用戶入口網站中，按一下 [+]、移至 Azure Marketplace、部署 Django 網站，然後等待順利完成。 Django Web 平台會使用以檔案系統為基礎的資料庫。 它不需要任何額外的資源提供者，例如 SQL 或 MySQL。

2. 如果您也會部署 MySQL 資源提供者，就可以從 Marketplace 部署 WordPress 網站。 當系統提示您提供資料庫參數時，請使用您選擇的使用者名稱和伺服器名稱，以 User1@Server1 形式輸入使用者名稱。

3. 如果您也會部署 SQL Server 資源提供者，就可以從 Marketplace 部署 DNN 網站。 當系統提示您提供資料庫參數時，請選擇執行 SQL Server 且已連線到您資源提供者之電腦中的資料庫。

## <a name="next-steps"></a>後續步驟

您也可以嘗試其他[平台即服務 (PaaS) 服務](azure-stack-tools-paas-services.md)。

- [SQL Server 資源提供者](azure-stack-sql-resource-provider-deploy.md)
- [MySQL 資源提供者](azure-stack-mysql-resource-provider-deploy.md)

<!--Links-->
[Azure_Stack_App_Service_preview_installer]: http://go.microsoft.com/fwlink/?LinkID=717531
[App_Service_Deployment]: http://go.microsoft.com/fwlink/?LinkId=723982
[AppServiceHelperScripts]: http://go.microsoft.com/fwlink/?LinkId=733525

<!--Image references-->
[1]: ./media/azure-stack-app-service-deploy-offline/app-service-exe-advanced-create-package.png
[2]: ./media/azure-stack-app-service-deploy-offline/app-service-exe-advanced-complete-offline.png
[3]: ./media/azure-stack-app-service-deploy-offline/app-service-azure-stack-arm-endpoints.png
[4]: ./media/azure-stack-app-service-deploy-offline/app-service-azure-stack-subscription-information.png
[5]: ./media/azure-stack-app-service-deploy-offline/app-service-default-VNET-config.png
[6]: ./media/azure-stack-app-service-deploy-offline/app-service-custom-VNET-config.png
[7]: ./media/azure-stack-app-service-deploy-offline/app-service-custom-VNET-config-with-values.png
[8]: ./media/azure-stack-app-service-deploy-offline/app-service-fileshare-configuration.png
[9]: ./media/azure-stack-app-service-deploy-offline/app-service-fileshare-configuration-error.png
[10]: ./media/azure-stack-app-service-deploy-offline/app-service-identity-app.png
[11]: ./media/azure-stack-app-service-deploy-offline/app-service-certificates.png
[12]: ./media/azure-stack-app-service-deploy-offline/app-service-sql-configuration.png
[13]: ./media/azure-stack-app-service-deploy-offline/app-service-sql-configuration-error.png
[14]: ./media/azure-stack-app-service-deploy-offline/app-service-cloud-quantities.png
[15]: ./media/azure-stack-app-service-deploy-offline/app-service-windows-image-selection.png
[16]: ./media/azure-stack-app-service-deploy-offline/app-service-role-credentials.png
[17]: ./media/azure-stack-app-service-deploy-offline/app-service-azure-stack-deployment-summary.png
[18]: ./media/azure-stack-app-service-deploy-offline/app-service-deployment-progress.png
