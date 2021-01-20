---
title: 使用 Azure Migrate 伺服器評量來探索實體伺服器
description: 了解如何使用 Azure Migrate 伺服器評量來探索內部部署實體伺服器。
author: vineetvikram
ms.author: vivikram
ms.manager: abhemraj
ms.topic: tutorial
ms.date: 09/14/2020
ms.custom: mvc
ms.openlocfilehash: 548cee262d874f5bc0f6024a857c2bb8a5466106
ms.sourcegitcommit: 949c0a2b832d55491e03531f4ced15405a7e92e3
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/18/2021
ms.locfileid: "98541337"
---
# <a name="tutorial-discover-physical-servers-with-server-assessment"></a>教學課程：使用伺服器評量來探索實體伺服器

在遷移至 Azure 的過程中，您會探索伺服器以進行評量和遷移。

本教學課程說明如何透過「Azure Migrate：伺服器評量工具」探索內部部署實體伺服器(使用輕量型 Azure Migrate 設備)。 您會將設備部署為實體伺服器，以持續探索機器和效能中繼資料。

在本教學課程中，您會了解如何：

> [!div class="checklist"]
> * 設定 Azure 帳戶。
> * 準備實體伺服器以進行探索。
> * 建立 Azure Migrate 專案。
> * 設定 Azure Migrate 設備。
> * 開始連續探索。

> [!NOTE]
> 教學課程顯示嘗試案例的最快路徑，並使用預設選項。  

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。

## <a name="prerequisites"></a>必要條件

在開始本教學課程之前，請先確認您符合下列必要條件。

**需求** | **詳細資料**
--- | ---
**設備** | 您需要用來執行 Azure Migrate 設備的機器。 機器應具有：<br/><br/> - 已安裝的 Windows Server 2016。<br/> _(目前只有 Windows Server 2016 支援設備的部署。)_<br/><br/> -16 GB RAM，8個 vcpu，大約 80 GB 的磁片儲存體<br/><br/> - 靜態或動態 IP 位址，以及網際網路存取 (直接或透過 Proxy)。
**Windows 伺服器** | 允許 WinRM 連接埠 5985 上的輸入連線 (HTTP)，讓設備可以提取設定和效能中繼資料。
**Linux 伺服器** | 允許連接埠 22 上的輸入連線 (TCP)。

## <a name="prepare-an-azure-user-account"></a>準備 Azure 使用者帳戶

若要建立 Azure Migrate 專案並註冊 Azure Migrate 設備，您需要具有下列權限的帳戶：
- Azure 訂用帳戶的參與者或擁有者權限。
- 註冊 Azure Active Directory (AAD) 應用程式的許可權。

如果您剛建立免費的 Azure 帳戶，您就是訂用帳戶的擁有者。 如果您不是訂用帳戶擁有者，請與擁有者合作以指派權限，如下所示：

1. 在 Azure 入口網站中搜尋「訂用帳戶」，然後在 [服務] 底下選取 [訂用帳戶]。

    ![搜尋 Azure 訂用帳戶的搜尋方塊](./media/tutorial-discover-physical/search-subscription.png)

2. 在 [訂用帳戶] 頁面中，選取您要在其中建立 Azure Migrate 專案的訂用帳戶。 
3. 在訂用帳戶中，選取 [存取控制 (IAM)] > [檢查存取權]。
4. 在 [檢查存取權] 中，搜尋相關的使用者帳戶。
5. 在 [新增角色指派] 中，按一下 [新增]。

    ![搜尋使用者帳戶以檢查存取權並指派角色](./media/tutorial-discover-physical/azure-account-access.png)

6. 在 [新增角色指派] 中，選取 [參與者] 或 [擁有者] 角色，然後選取帳戶 (在我們的範例中為 azmigrateuser)。 然後按一下 [儲存]  。

    ![開啟 [新增角色指派] 頁面，將角色指派給帳戶](./media/tutorial-discover-physical/assign-role.png)

1. 若要註冊設備，您的 Azure 帳戶需要 **註冊 AAD 應用程式的許可權。**
1. 在 Azure 入口網站中，流覽至 **Azure Active Directory**  >  **使用者** 的  >  **使用者設定**。
1. 在 [使用者設定] 中，確認 Azure AD 使用者可以註冊應用程式 (預設為 [是])。

    ![在 [使用者設定] 中確認使用者可以註冊 Active Directory 應用程式](./media/tutorial-discover-physical/register-apps.png)

9. 如果 [應用程式註冊] 設定設為 [否]，請要求租使用者/全域管理員指派必要的許可權。 或者，租使用者/全域管理員可以將 **應用程式開發人員** 角色指派給帳戶，以允許註冊 AAD 應用程式。 [深入了解](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md)。

## <a name="prepare-physical-servers"></a>準備實體伺服器

設定可供設備用來存取實體伺服器的帳戶。

- 若為 **Windows 伺服器**，請針對已加入網域的電腦使用網域帳戶，並針對未加入網域的電腦使用本機帳戶。 使用者帳戶應新增至下列群組：遠端管理使用者、效能監視器使用者，以及效能記錄使用者。
- 針對 **linux 伺服器**，您需要您想要探索的 linux 伺服器上的根帳號。 或者，您可以使用下列命令來設定具有必要功能的非根帳號：

**命令** | **目的**
--- | --- |
setcap CAP_DAC_READ_SEARCH+eip /usr/sbin/fdisk <br></br> setcap CAP_DAC_READ_SEARCH+eip /sbin/fdisk _(if /usr/sbin/fdisk is not present)_ | 若要收集磁碟設定資料
setcap "cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_setuid,<br>cap_setpcap,cap_net_bind_service,cap_net_admin,cap_sys_chroot,cap_sys_admin,<br>cap_sys_resource,cap_audit_control,cap_setfcap=+eip" /sbin/lvm | 若要收集磁碟效能資料
setcap CAP_DAC_READ_SEARCH+eip /usr/sbin/dmidecode | 若要收集 BIOS 序號
chmod a+r /sys/class/dmi/id/product_uuid | 若要收集 BIOS GUID


## <a name="set-up-a-project"></a>設定專案

設定新的 Azure Migrate 專案。

1. 在 Azure 入口網站 > [所有服務] 中，搜尋 **Azure Migrate**。
2. 在 [服務] 下，選取 [Azure Migrate]。
3. 在 [概觀] 中，選取 [建立專案]。
5. 在 [建立專案] 中，選取您的 Azure 訂用帳戶和資源群組。 如果您還沒有資源群組，請加以建立。
6. 在 [專案詳細資料] 中，指定專案名稱以及要在其中建立專案的地理位置。 請檢閱[公用](migrate-support-matrix.md#supported-geographies-public-cloud)和[政府雲端](migrate-support-matrix.md#supported-geographies-azure-government)支援的地理位置。

   ![專案名稱和區域的方塊](./media/tutorial-discover-physical/new-project.png)

7. 選取 [建立]。
8. 等候幾分鐘讓 Azure Migrate 專案完成部署。 **Azure Migrate：伺服器評量** 工具依預設會新增至新專案中。

![顯示依預設新增伺服器評量工具的頁面](./media/tutorial-discover-physical/added-tool.png)

> [!NOTE]
> 如果您已建立專案，您可以使用相同的專案註冊其他設備，以找出並評估更多伺服器。[深入瞭解](create-manage-projects.md#find-a-project)

## <a name="set-up-the-appliance"></a>設定設備

Azure Migrate 設備會執行伺服器探索，並將伺服器設定和效能中繼資料傳送至 Azure Migrate。 您可以藉由執行可從 Azure Migrate 專案下載的 PowerShell 腳本來設定設備。

若要設定設備，請：
1. 提供設備名稱，並在入口網站中產生 Azure Migrate 專案金鑰。
2. 從 Azure 入口網站下載含有 Azure Migrate 安裝程式指令碼的 ZIP 壓縮檔案。
3. 從 ZIP 壓縮檔案解壓縮內容。 以系統管理權限啟動 PowerShell 主控台。
4. 執行 PowerShell 指令碼以啟動設備 Web 應用程式。
5. 進行設備的第一次設定，並使用 Azure Migrate 專案金鑰將其註冊至 Azure Migrate 專案。

### <a name="1-generate-the-azure-migrate-project-key"></a>1. 產生 Azure Migrate 專案金鑰

1. 在 [移轉目標] > [伺服器] >  **[Azure Migrate：伺服器評估]** 中，選取 [探索]。
2. 在 **探索電腦** > **電腦是否已虛擬化？** 中，選取 [實體或其他 (AWS、GCP、Xen 等。)]。
3. 在 **1：產生 Azure Migrate 專案金鑰** 中，為您要為探索之實體或虛擬伺服器設定的 Azure Migrate 設備命名。名稱應使用英數位元，且長度不超過 14 個字元。
1. 按一下 [產生金鑰] ，開始建立必要的 Azure 資源。 在建立資源期間，請勿關閉探索的電腦頁面。
1. 成功建立 Azure 資源之後，系統會產生 **Azure Migrate 專案金鑰**。
1. 複製金鑰，您在設定期間需要此金鑰才能完成設備的註冊。

### <a name="2-download-the-installer-script"></a>2. 下載安裝程式腳本

在 **2：下載 Azure Migrate 設備** 中，按一下 [下載]。

### <a name="verify-security"></a>確認安全性

請先確認 ZIP 檔案安全無虞再進行部署。

1. 在存放下載檔案的目標電腦上，開啟系統管理員命令視窗。
2. 執行下列命令以產生 ZIP 檔案的雜湊：
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - 公用雲端的使用範例：```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public.zip SHA256 ```
    - 政府雲端的使用範例：```  C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip SHA256 ```
3.  確認最新的設備版本與雜湊值：
    - 對於公用雲端：

        **案例** | **下載** _ | _ *雜湊值**
        --- | --- | ---
        實體 (85.8 MB) | [最新版本](https://go.microsoft.com/fwlink/?linkid=2140334) | ce5e6f0507936def8020eb7b3109173dad60fc51dd39c3bd23099bc9baaabe29

    - 對於 Azure Government：

        **案例** | **下載** _ | _ *雜湊值**
        --- | --- | ---
        實體 (85.8 MB) | [最新版本](https://go.microsoft.com/fwlink/?linkid=2140338) | ae132ebc574caf231bf41886891040ffa7abbe150c8b50436818b69e58622276
 

### <a name="3-run-the-azure-migrate-installer-script"></a>3. 執行 Azure Migrate 安裝程式腳本
此安裝程式指令碼會執行下列作業︰

- 安裝用於實體伺服器探索及評定的代理程式與 Web 應用程式。
- 安裝 Windows 角色，包括 Windows 啟用服務、IIS 與 PowerShell ISE。
- 下載並安裝 IIS 可讀寫模組。 [深入了解](https://www.microsoft.com/download/details.aspx?id=7435)。
- 使用 Azure Migrate 的持續設定詳細資料來更新登錄機碼 (HKLM)。
- 在路徑底下建立下列檔案：
    - **組態檔**：%Programdata%\Microsoft Azure\Config
    - **記錄檔**：%Programdata%\Microsoft Azure\Logs

執行指令碼，如下所示：

1. 將 ZIP 壓縮檔案解壓縮至會裝載設備之伺服器上的資料夾。  請確定您未在現有 Azure Migrate 設備的電腦上執行指令碼。
2. 在上述伺服器上，使用系統管理 (提高的) 權限來啟動 PowerShell。
3. 將 PowerShell 目錄變更為已從下載的 ZIP 壓縮檔案解壓縮內容的資料夾。
4. 藉由執行下列命令，以執行名為 **AzureMigrateInstaller.ps1** 的指令碼：

    - 對於公用雲端： 
    
        ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 ```
    - 對於 Azure Government： 
    
        ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>.\AzureMigrateInstaller.ps1 ```

    指令碼會在成功完成時啟動設備 Web 應用程式。

如果發生任何問題，您可以存取位於 C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>時間戳記</em>.log 的指令碼記錄，以進行疑難排解。

### <a name="verify-appliance-access-to-azure"></a>確認設備是否能存取 Azure

確定設備 VM 可以連線至[公用](migrate-appliance.md#public-cloud-urls)和[政府](migrate-appliance.md#government-cloud-urls)雲端的 Azure URL。

### <a name="4-configure-the-appliance"></a>4. 設定設備

第一次設定設備。

1. 在任何可連線至設備的機器上開啟瀏覽器，並開啟設備 Web 應用程式的 URL：**https://設備名稱或 IP 位址:44368**。

   或者，您也可以按一下應用程式捷徑，從桌面開啟應用程式。
2. 接受 **授權條款**，並閱讀第三方資訊。
1. 在 [Web 應用程式] > [設定必要條件] 中，執行下列動作：
    - **連線能力**：應用程式會確認伺服器是否能夠存取網際網路。 如果伺服器使用 Proxy：
        - 按一下 [設定 Proxy] 以指定 Proxy 位址 (格式為 http://ProxyIPAddress 或 http://ProxyFQDN) ) 和接聽連接埠。
        - 如果 Proxy 需要驗證，請指定認證。
        - 僅支援 HTTP Proxy。
        - 如果您已新增 Proxy 詳細資料，或已停用 Proxy 和/或驗證，請按一下 [儲存] 以再次觸發連線檢查。
    - **時間同步**：系統會確認時間。 設備上的時間應該與網際網路時間同步，伺服器探索才能正常運作。
    - **安裝更新**：Azure Migrate Server 評估會檢查設備是否已安裝最新的更新。檢查完成之後，您可以按一下 [檢視設備服務] 以查看設備上執行之元件的狀態和版本。

### <a name="register-the-appliance-with-azure-migrate"></a>向 Azure Migrate 註冊設備

1. 貼上從入口網站複製的 **Azure Migrate 專案金鑰**。 如果沒有金鑰，請移至 **伺服器評估 > 探索 > 管理現有的設備**，選取在金鑰產生時提供的設備名稱，並複製對應的金鑰。
1. 您需要裝置程式碼才能向 Azure 驗證。 按一下 [登入] 會以如下所示的裝置程式碼開啟強制回應。

    ![顯示裝置程式碼的強制回應](./media/tutorial-discover-vmware/device-code.png)

1. 按一下 [複製程式碼和登入] 以複製裝置程式碼，然後在新的瀏覽器索引標籤中開啟 Azure 登入提示。如果未出現，請確定您已在瀏覽器中停用快顯封鎖程式。
1. 在新的索引標籤上，使用您的 Azure 使用者名稱和密碼登入並貼上裝置程式碼。
   
   不支援使用 PIN 登入。
3. 如果不小心在尚未完成登入流程時關閉了登入索引標籤，則需要重新整理設備組態管理員的瀏覽器索引標籤，才能再次啟用登入按鈕。
1. 成功登入之後，請回到設備組態管理員的上一個索引標籤。
4. 如果用於記錄的 Azure 使用者針對在金鑰產生期間建立的 Azure 資源帳戶具有正確的[權限]()，就會起始設備註冊。
1. 成功註冊設備之後，您可以按一下 [檢視詳細資料]查看註冊詳細資料。


## <a name="start-continuous-discovery"></a>開始連續探索

現在，從設備連線至要探索的實體伺服器，然後開始探索。

1. 在 [步驟 1：選取復原點]**提供認證，以探索 Windows 和 Linux 實體或虛擬伺服器**，按一下 [新增認證]。
1. 若為 Windows server，請選取 **Windows Server** 作為來源類型、指定認證的自訂名稱並新增使用者名稱和密碼。按一下 [儲存]。
1. 如果您針對 Linux 伺服器使用密碼型驗證，請選取 **Linux Server (密碼型)** 作為來源類型、指定認證的自訂名稱並新增使用者名稱和密碼。按一下 [儲存]。
1. 如果您針對 Linux 伺服器使用 SSH 金鑰型驗證，可以選取 **Linux Server (密碼型)** 作為來源類型、指定認證的自訂名稱並新增使用者名稱、瀏覽並選取 SSH 私密金鑰檔案。 按一下 [ **儲存**]。

    - Azure Migrate 支援使用 RSA、DSA、ECDSA 和 ed25519 演算法的 ssh-keygen 命令所產生的 SSH 私密金鑰。
    - 目前 Azure Migrate 不支援複雜密碼型的 SSH 金鑰。 若沒有複雜密碼，請使用 SSH 金鑰。
    - 目前 Azure Migrate 不支援 PuTTY 所產生的 SSH 私密金鑰檔案。
    - Azure Migrate 支援 SSH 私密金鑰檔案的 OpenSSH 格式，如下所示：
    
    ![支援 SSH 私密金鑰的格式](./media/tutorial-discover-physical/key-format.png)

1. 如果想要一次新增多個認證，請按一下 [新增更多] 以儲存並新增更多認證。 實體伺服器探索支援使用多個認證。
1. 在 **步驟 2：提供實體或虛擬伺服器詳細資料** 中，按一下 [新增探索來源] 來指定伺服器 **IP 位址/FQDN** 和認證的自訂名稱，以連線到伺服器。
1. 您可以一次 **新增單一項目**，或 **新增多個項目**。 另外也可透過 **匯入 CSV** 提供伺服器詳細資料。


    - 如果您選擇 **新增單一項目**，可以選擇 OS 類型、指定認證的自訂名稱、新增伺服器 **IP 位址/FQDN**，然後按一下 [儲存]。
    - 如果您選擇 **新增多個項目**，可以在文字方塊中指定伺服器 **IP 位址/FQDN** 和認證的自訂名稱，一次新增多筆記錄。**確認** 新增的記錄，然後按一下 [儲存]。
    - 如果您選擇 **匯入 CSV** (預設選項)，您可以下載 CSV 範本檔案，並在檔案中填入伺服器 **IP 位址/FQDN** 和憑證的自訂名稱。 然後將檔案匯入設備，**確認** 檔案中的記錄，然後按一下 [儲存]。

1. 按一下儲存時，設備會嘗試驗證已新增的伺服器連線，並在資料表中顯示每部伺服器的 **驗證狀態**。
    - 如果伺服器驗證失敗，請按一下資料表狀態欄中的 [驗證失敗] 以檢閱錯誤。 修正問題，然後再次驗證。
    - 若要移除伺服器，請選取 [刪除]。
1. 您可以在啟動探索之前，隨時 **重新驗證** 伺服器的連線功能是否正常。
1. 按一下 [開始探索]，以開始探索成功驗證的伺服器。 成功起始探索之後，您可以在資料表中檢查每部伺服器的探索狀態。


這會開始探索。 每部伺服器大約會需要 2 分鐘，才能讓探索到之伺服器的中繼資料出現在 Azure 入口網站中。

## <a name="verify-servers-in-the-portal"></a>在入口網站中驗證伺服器

探索完成後，您可以確認伺服器是否出現在入口網站中。

1. 開啟 Azure Migrate 儀表板。
2. 在 Azure Migrate - 伺服器 >  **Azure Migrate：伺服器評量** 頁面中，按一下圖示以顯示 探索到的伺服器 計數。
## <a name="next-steps"></a>後續步驟

- [評估要移轉至 Azure VM 的實體伺服器](tutorial-assess-physical.md)。
- [檢閱設備在探索期間收集的資料](migrate-appliance.md#collected-data---physical)。