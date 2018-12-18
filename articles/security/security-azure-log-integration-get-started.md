---
title: 開始使用 Azure 記錄整合 | Microsoft Docs
description: 學習如何安裝 Azure 記錄整合服務，以及整合來自 Azure 儲存體、Azure 稽核記錄和 Azure 資訊安全中心警示的記錄。
services: security
documentationcenter: na
author: Barclayn
manager: MBaldwin
editor: TomShinder
ms.assetid: 53f67a7c-7e17-4c19-ac5c-a43fabff70e1
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ums.workload: na
ms.date: 06/07/2018
ms.author: barclayn
ms.custom: azlog
ms.openlocfilehash: 2f97a2e8ad38bb3c78333cc2c8eedad8f520e68a
ms.sourcegitcommit: 04fc1781fe897ed1c21765865b73f941287e222f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39036797"
---
# <a name="azure-log-integration-with-azure-diagnostics-logging-and-windows-event-forwarding"></a>使用 Azure 診斷記錄和 Windows 事件轉送進行 Azure 記錄整合


>[!IMPORTANT]
> Azure 記錄整合功能將於 2019/06/01 淘汰。 AzLog 下載將於 2018 年 6 月 27 日停用。 如需繼續進行的指導，請檢閱 [Use Azure monitor to integrate with SIEM tools](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/) (使用 Azure 監視器與 SIEM 工具整合) 一文 

只有在無法從安全性資訊與事件管理 (SIEM) 廠商取得 [Azure 監視器](../monitoring-and-diagnostics/monitoring-get-started.md)連接器時，才應該使用 Azure 記錄整合。

Azure 記錄整合使得 Azure 記錄檔可供您的 SIEM 使用，讓您可以為所有資產建立整合的安全性儀表板。
如需 Azure 監視器連接器狀態的詳細資訊，請連絡 SIEM 廠商。

> [!IMPORTANT]
> 如果您的主要興趣在於收集虛擬機器記錄，大多數 SIEM 廠商會將此選項包含在其解決方案中。 使用 SIEM 廠商的連接器永遠是較好的替代做法。

本文幫助您開始使用 Azure 記錄整合。 內容著重在安裝 Azure 記錄整合服務，以及整合本服務與 Azure 診斷。 Azure 記錄整合服務便能自部署於 Azure 基礎結構即服務之虛擬機器的 Windows 安全性事件頻道中，收集 Windows 事件記錄資訊。 這類似於您可能會在內部部署系統中使用的「事件轉送」。

> [!NOTE]
> SIEM 本身會將 Azure 記錄整合的輸出與 SIEM 整合。 如需詳細資訊，請參閱[使用內部部署 SIEM 整合 Azure 記錄整合](https://blogs.msdn.microsoft.com/azuresecurity/2016/08/23/azure-log-siem-configuration-steps/)。

Azure 記錄整合服務會在執行 Windows Server 2008 R2 或更新版本 (Windows Server 2016 或 Windows Server 2012 R2 較佳) 的實體或虛擬電腦上執行。

實體電腦可以在內部或在主機服務提供者的網站上執行。 如果您選擇在虛擬機器上執行 Azure 記錄整合服務，該虛擬機器可位於內部或公用雲端，例如 Microsoft Azure 中。

執行 Azure 記錄整合服務的實體或虛擬機器需要網路連線至 Azure 公用雲端。 本文提供所需設定的詳細資料。

## <a name="prerequisites"></a>必要條件

Azure 記錄整合安裝至少需要下列項目︰

* **Azure 訂用帳戶**。 如果您沒有帳戶，您可以註冊 [免費帳戶](https://azure.microsoft.com/free/)。
* 可用於 Windows Azure 診斷 (WAD) 記錄的**儲存體帳戶**。 您可以使用預先設定的儲存體帳戶或建立新儲存體帳戶。 稍後在本文中會說明如何設定儲存體帳戶。

  > [!NOTE]
  > 視情況而定，您不一定需要儲存體帳戶。 本文所涵蓋的 Azure 診斷案例需要儲存體帳戶。

* **兩個系統**： 
  * 執行 Azure 記錄整合服務的機器。 此機器會收集所有記錄資訊並於稍後匯入 SIEM。 這個系統：
    * 可以是內部部署或裝載在 Microsoft Azure 中。  
    * 必須執行 x64 版本的 Windows Server 2008 R2 SP1 或更高版本，並已安裝 Microsoft .NET 4.5.1。 請參閱[判斷安裝的 .NET Framework 版本](https://msdn.microsoft.com/library/hh925568)，以判斷已安裝的 .NET 版本。  
    * 必須能夠連線到用於 Azure 診斷記錄的 Azure 儲存體帳戶。 稍後在本文中會說明如何確認連線能力。
  * 您想要監視的機器。 這是當作 [Azure 虛擬機器](../virtual-machines/virtual-machines-windows-overview.md)執行的 VM。 這部機器的記錄資訊會傳送到 Azure 記錄整合服務的機器。

如需如何使用 Azure 入口網站建立虛擬機器的快速示範，請觀看下列影片：<br /><br />


> [!VIDEO https://channel9.msdn.com/Blogs/Azure-Security-Videos/Azure-Log-Integration-Videos-Create-a-Virtual-Machine/player]

## <a name="deployment-considerations"></a>部署考量

在測試期間，您可以使用任何符合最低作業系統需求的系統。 在生產環境中，您可能需要規劃相應放大或相應增加以因應負載。

您可以執行 Azure 記錄整合服務的多個執行個體。 不過，在每個實體或虛擬機器上，您只能執行一個此服務的執行個體。 此外，您可以對 WAD 的 Azure 診斷儲存體帳戶進行負載平衡。 要提供給執行個體的訂用帳戶數目取決於您的容量。

> [!NOTE]
> 關於何時要相應放大 Azure 記錄整合機器的執行個體 (亦即正在執行 Azure 記錄整合服務的機器)、儲存體帳戶或訂用帳戶，我們目前並未提供特定建議。 請根據您在這些地方的效能觀察調整決策。

您也可以相應增加 Azure 記錄整合服務，以協助改善效能。 下列效能計量能協助您調整您選擇執行 Azure 記錄整合服務的電腦︰

* 在 8 個處理器 (核心) 的機器上，Azure 記錄整合的單一執行個體每日可以處理大約 2400 萬個事件 (約一小時 100 萬個事件)。
* 在 4 個處理器 (核心) 的機器上，Azure 記錄整合的單一執行個體每日可以處理大約 150 萬個事件 (約一小時 62,500 個事件)。

## <a name="install-azure-log-integration"></a>安裝 Azure 記錄整合

若要安裝 Azure 記錄整合，請下載 [Azure 記錄整合](https://www.microsoft.com/download/details.aspx?id=53324)安裝檔案。 完成安裝程序。 選擇是否要提供遙測資訊給 Microsoft。

Azure 記錄整合服務會從其安裝所在的電腦收集遙測資料。  

收集的遙測資料包括下列這些：

* Azure 記錄整合的執行期間所發生的例外狀況。
* 所處理之查詢和事件數目的相關計量。
* 使用了哪些 Azlog.exe 命令列選項的相關統計資料。 

> [!NOTE]
> 我們建議您允許 Microsoft 收集遙測資料。 您可以清除 [允許 Microsoft 收集遙測資料] 核取方塊，關閉遙測資料的收集。
>

![安裝窗格的螢幕擷取畫面，已選取遙測核取方塊](./media/security-azure-log-integration-get-started/telemetry.png)


下列影片涵蓋安裝程序：<br /><br />

> [!VIDEO https://channel9.msdn.com/Blogs/Azure-Security-Videos/Azure-Log-Integration-Videos-Install-Azure-Log-Integration/player]

## <a name="post-installation-and-validation-steps"></a>安裝後續和驗證步驟

完成基本安裝之後，您便可以執行安裝後續和驗證步驟︰

1. 以系統管理員身分啟動 PowerShell。 然後移至 C:\Program Files\Microsoft Azure Log Integration。
2. 匯入 Azure 記錄整合 Cmdlet。 若要匯入 Cmdlet，請執行 `LoadAzlogModule.ps1` 指令碼。 輸入 `.\LoadAzlogModule.ps1`，然後按 Enter 鍵 (請注意此命令中使用 **.\\**)。 您應該會看到類似下圖的內容：

  ![LoadAzlogModule.ps1 命令之輸出的螢幕擷取畫面](./media/security-azure-log-integration-get-started/loaded-modules.png)
3. 接下來，設定 Azure 記錄整合以使用特定的 Azure 環境。 「Azure 環境」是您想要使用的 Azure 雲端資料中心的類型。 雖然現在有數個 Azure 環境，目前相關的選項是 **AzureCloud** 或 **AzureUSGovernment**。 以系統管理員的身分執行 PowerShell，請確定您是在 C:\Program Files\Microsoft Azure Log Integration\ 中。 然後執行這個命令：

  `Set-AzlogAzureEnvironment -Name AzureCloud` (針對 **AzureCloud**)
  
  如果您想要使用美國政府 Azure 雲端，請在 **-Name** 變數使用 **AzureUSGovernment**。 目前不支援其他 Azure 雲端。  

  > [!NOTE]
  > 命令成功時，您不會收到任何回應。 

4. 您需要擁有用於 Azure 診斷的儲存體帳戶名稱，才能監視系統。 在 Azure 入口網站中，移至 [虛擬機器]。 尋找您要監視的 Windows 虛擬機器。 在 [屬性] 區段中，選取 [診斷設定]。  然後選取 [代理程式]。 記下指定的儲存體帳戶名稱。 您需要此帳戶的名稱以進行後續步驟。

  ![Azure 診斷設定窗格的螢幕擷取畫面](./media/security-azure-log-integration-get-started/storage-account-large.png) 

  ![啟用來賓層級監視按鈕的螢幕擷取畫面](./media/security-azure-log-integration-get-started/azure-monitoring-not-enabled-large.png)

  > [!NOTE]
  > 如果建立虛擬機器時沒有啟用監視，您可以如上圖中所示啟用它。

5. 現在，回到 Azure 記錄整合的機器。 確認您已從 Azure 記錄整合安裝所在的系統連線至儲存體帳戶。 執行 Azure 記錄整合服務的電腦需要存取儲存體帳戶，以擷取 Azure 診斷在每個受監視系統上記錄的資訊。 若要確認連線能力： 
  1. [下載 Azure 儲存體總管](http://storageexplorer.com/)。
  2. 完成安裝。
  3. 安裝完成時選取 [下一步]。 讓 [啟動 Microsoft Azure 儲存體總管] 核取方塊保持選取狀態。  
  4. 登入 Azure。
  5. 確認您可以看到您設定用於 Azure 診斷的儲存體帳戶： 

   ![儲存體總管中儲存體帳戶的螢幕擷取畫面](./media/security-azure-log-integration-get-started/storage-explorer.png)

  6. 儲存體帳戶之下會出現幾個選項。 在 [資料表] 下，您應該會看到 **WADWindowsEventLogsTable** 資料表。

  如果建立虛擬機器時沒有啟用監視，您可以如稍早所述的方式啟用它。


## <a name="integrate-windows-vm-logs"></a>整合 Windows VM 記錄

在此步驟中，您會設定執行 Azure 記錄整合服務的電腦，使其連線至包含記錄的儲存體帳戶。

若要完成此步驟，您需要幾個東西：  
* **FriendlyNameForSource：** 您可以使用這個好記的名稱，為您已設定虛擬機器用來儲存 Azure 診斷資訊的儲存體帳戶命名。
* **StorageAccountName：** 您設定 Azure 診斷時，指定之儲存體帳戶的名稱。  
* **StorageKey：** 為此虛擬機器儲存 Azure 診斷資訊之儲存體帳戶的儲存體金鑰。  

若要取得儲存體金鑰，請完成下列步驟︰
1. 移至 [Azure 入口網站](http://portal.azure.com)。
2. 選取導覽窗格中的 [所有服務]。
3. 在 [篩選] 方塊中輸入**儲存體**。 然後選取 [儲存體帳戶]。

  ![顯示所有服務中儲存體帳戶的螢幕擷取畫面](./media/security-azure-log-integration-get-started/filter.png)

4. 儲存體帳戶清單隨即出現。 按兩下您指派給記錄儲存體的帳戶。

  ![顯示了儲存體帳戶清單的螢幕擷取畫面](./media/security-azure-log-integration-get-started/storage-accounts.png)

5. 在 [設定] 底下，選取 [存取金鑰]。

  ![顯示功能表中存取金鑰選項的螢幕擷取畫面](./media/security-azure-log-integration-get-started/storage-account-access-keys.png)

6. 複製 **key1**，然後將其儲存在安全的位置，以供進行下一個步驟時存取。
7. 在您安裝Azure 記錄整合的伺服器上，以系統管理員身分開啟命令提示字元視窗  (請務必以系統管理員身分開啟命令提示字元視窗，而不是開啟 PowerShell)。
8. 移至 C:\Program Files\Microsoft Azure Log Integration。
9. 執行這個命令：`Azlog source add <FriendlyNameForTheSource> WAD <StorageAccountName> <StorageKey>`。
 
  範例：
  
  `Azlog source add Azlogtest WAD Azlog9414 fxxxFxxxxxxxxywoEJK2xxxxxxxxxixxxJ+xVJx6m/X5SQDYc4Wpjpli9S9Mm+vXS2RVYtp1mes0t9H5cuqXEw==`

  如果您想要讓訂用帳戶識別碼顯示在事件 XML 中，請將訂用帳戶識別碼加在易記名稱之後︰

  `Azlog source add <FriendlyNameForTheSource>.<SubscriptionID> WAD <StorageAccountName> <StorageKey>`
  
  範例：
  
  `Azlog source add Azlogtest.YourSubscriptionID WAD Azlog9414 fxxxFxxxxxxxxywoEJK2xxxxxxxxxixxxJ+xVJx6m/X5SQDYc4Wpjpli9S9Mm+vXS2RVYtp1mes0t9H5cuqXEw==`

> [!NOTE]
> 最多等待 60 分鐘，然後檢視從儲存體帳戶提取的事件。 若要在Azure 記錄整合中檢視事件，請選取 [事件檢視器] > [Windows 記錄] > [轉送的事件]。

下方影片涵蓋前述步驟︰<br /><br />

> [!VIDEO https://channel9.msdn.com/Blogs/Azure-Security-Videos/Azure-Log-Integration-Videos-Enable-Diagnostics-and-Storage/player]


## <a name="if-data-isnt-showing-up-in-the-forwarded-events-folder"></a>如果「轉送的事件」資料夾中沒有顯示資料
如果一個小時之後資料未顯示在「轉送的事件」資料夾中，請完成以下步驟︰

1. 檢查執行 Azure 記錄整合服務的機器。 確認它可以存取 Azure。 若要測試連線，請嘗試從瀏覽器前往 [Azure 入口網站](http://portal.azure.com)。
2. 確定使用者帳戶 Azlog 具有 users\Azlog 資料夾的寫入權限。
  1. 開啟檔案總管。
  2. 移至 C:\users。
  3. 以滑鼠右鍵按一下 C:\users\Azlog。
  4. 選取 [安全性]。
  5. 選取 [NT 服務\Azlog]。 檢查此帳戶的權限。 如果帳戶不在此索引標籤中，或是沒有顯示適當的權限，您可以在此索引標籤中授與帳戶權限。
3. 在執行 `Azlog source list` 命令時，請確定輸出裡有列出您在 `Azlog source add` 命令中新增的儲存體帳戶。
4. 若要查看 Azure 記錄整合服務是否有報告任何錯誤，請移至 [事件檢視器] > [Windows 記錄] > [應用程式]。

如果您在安裝和設定期間遇到任何問題，可以建立[支援要求](../azure-supportability/how-to-create-azure-support-request.md)。 [服務] 請選取 [記錄整合]。

另一個支援選項是 [Azure 記錄整合 MSDN 論壇](https://social.msdn.microsoft.com/Forums/home?forum=AzureLogIntegration)。 在 MSDN 論壇中，社群會藉由回答問題以及分享有關如何充分利用 Azure 記錄整合的秘訣和技巧，來提供支援。 Azure 記錄整合小組也會監看這個論壇。 他們會盡可能提供協助。

## <a name="integrate-azure-activity-logs"></a>整合 Azure 活動記錄

「Azure 活動記錄」是訂用帳戶記錄，可讓您深入探索 Azure 中發生的訂用帳戶層級事件。 所涵蓋的資料範圍從 Azure Resource Manager 作業資料到服務健康情況事件的更新。 Azure 資訊安全中心警示也會包含在此記錄中。
> [!NOTE]
> 在嘗試執行本文中的步驟之前，您必須先檢閱[開始使用](security-azure-log-integration-get-started.md)一文並完成此處的步驟。

### <a name="steps-to-integrate-azure-activity-logs"></a>整合 Azure 活動記錄步驟

1. 開啟命令提示字元並執行此命令：```cd c:\Program Files\Microsoft Azure Log Integration```
2. 執行這個命令：```azlog createazureid```

    此命令會提示您登入 Azure。 此命令接著會建立託管 Azure 訂用帳戶 (其中登入的使用者是系統管理員、共同管理員或擁有者) 的 Azure AD 租用戶中的 Azure Active Directory 服務主體。 若登入的使用者只是 Azure AD 租用戶中的來賓使用者，則此命令將會失敗。 對 Azure 的驗證會透過 Azure AD 來進行。 建立 Azure 記錄整合的服務主體會建立 Azure AD 身分識別，以獲得 Azure 訂用帳戶的讀取權限。
3.  執行下列命令，授權上一個步驟中所建立的 Azure 記錄整合服務主體讀取訂用帳戶之活動記錄的權限。 您必須是訂用帳戶的擁有者才能執行命令。

    ```Azlog.exe authorize subscriptionId``` 範例：

```AZLOG.exe authorize ba2c2367-d24b-4a32-17b5-4443234859```

4.  檢查下列資料夾以確認其中是否建立了 Azure Active Directory 稽核記錄 JSON 檔案：
    - C:\Users\azlog\AzureResourceManagerJson
    - C:\Users\azlog\AzureResourceManagerJsonLD

> [!NOTE]
> 如需有關將 JSON 檔案中的資訊帶入到安全性資訊和事件管理 (SIEM) 系統的特定指示，請連絡您的 SIEM 廠商。

可透過 [Azure 記錄檔整合 MSDN 論壇](https://social.msdn.microsoft.com/Forums/office/home?forum=AzureLogIntegration)取得社群協助。 此論壇可透過分享問題、解答、秘訣和技巧，讓 Azure 記錄整合社群的成員彼此支援。 此外，Azure 記錄整合小組會監看這個論壇，並且會盡全力提供協助。

您也可以建立[支援要求](../azure-supportability/how-to-create-azure-support-request.md)。 選取 [記錄整合] 作為您要求支援的服務。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure 記錄整合，請參閱下列文章：在嘗試執行本文中的步驟之前，您必須先檢閱＜開始使用＞一文並完成此處的步驟。

* [Azure 記錄的 Azure 記錄整合](https://www.microsoft.com/download/details.aspx?id=53324)。 下載中心會納入 Azure 記錄整合的詳細資料、系統需求和安裝指示。
* [Azure 記錄整合簡介](security-azure-log-integration-overview.md)。 本文為您介紹 Azure 記錄整合、其主要功能以及運作方式。
* [合作夥伴設定步驟](https://blogs.msdn.microsoft.com/azuresecurity/2016/08/23/azure-log-siem-configuration-steps/)。 此部落格文章說明如何設定 Azure 記錄整合，以搭配使用合作夥伴解決方案 Splunk、HP ArcSight 和 IBM QRadar。 描述我們目前有關如何設定 SIEM 元件的指引。 如需其他詳細資料，請洽詢您的 SIEM 廠商。
* [Azure 記錄整合常見問題集 (FAQ)](security-azure-log-integration-faq.md)。 此常見問題集會回答 Azure 記錄整合的相關常見問題。
* [整合 Azure 資訊安全中心警示與 Azure 記錄整合](../security-center/security-center-integrating-alerts-with-log-integration.md)。 本文說明如何同步處理 Azure 診斷和 Azure 活動記錄所收集的資訊安全中心警示和虛擬機器安全性事件。 您可以使用 Azure Log Analytics 或 SIEM 解決方案來同步處理記錄。
* [Azure 診斷和 Azure 稽核記錄的新功能](https://azure.microsoft.com/blog/new-features-for-azure-diagnostics-and-azure-audit-logs/)。 此部落格文章為您介紹 Azure 稽核記錄和其他功能，協助您深入了解您的 Azure 資源的作業。
