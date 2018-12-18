---
title: " 使用 Azure Site Recovery 管理實體伺服器災害復原的組態伺服器 | Microsoft Docs"
description: 本文說明如何使用 Azure Site Recovery 服務管理將實體伺服器災害復原到 Azure 的現有組態伺服器。
services: site-recovery
author: AnoopVasudavan
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: anoopkv
ms.openlocfilehash: 068d2774791995fab1c07c73e6d733a6e09379f1
ms.sourcegitcommit: a1e1b5c15cfd7a38192d63ab8ee3c2c55a42f59c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/10/2018
ms.locfileid: "37951171"
---
# <a name="manage-the-configuration-server-for-physical-server-disaster-recovery"></a>管理實體伺服器災害復原的組態伺服器

當您使用 [Azure Site Recovery](site-recovery-overview.md) 服務將實體伺服器災害復原到 Azure 時，便會設定內部部署組態伺服器。 組態伺服器會協調內部部署機器與 Azure 之間的通訊，以及管理資料複寫。 本文摘要說明部署設定伺服器之後，管理設定伺服器的一般工作。

## <a name="prerequisites"></a>先決條件

下表摘要說明部署內部部署組態伺服器機器的必要條件。

| **元件** | **需求** |
| --- |---|
| CPU 核心| 8 |
| RAM | 16 GB|
| 磁碟數量 | 3，包括作業系統磁碟、處理序伺服器快取磁碟和用於容錯回復的保留磁碟機 |
| - 磁碟可用空間 (處理序伺服器快取) | 600 GB
| 磁碟可用空間 (保留磁碟) | 600 GB|
| 作業系統  | Windows Server 2012 R2 <br> Windows Server 2016 |
| 作業系統地區設定 | 英文 (美國)|
| VMware vSphere PowerCLI 版本 | [PowerCLI 6.0](https://my.vmware.com/web/vmware/details?productId=491&downloadGroup=PCLI600R1 "PowerCLI 6.0")|
| Windows Server 角色 | 請勿啟用這些角色： <br> - Active Directory Domain Services <br>- 網際網路資訊服務 <br> - Hyper-V |
| 群組原則| 請勿啟用這些群組原則： <br> - 防止存取命令提示字元 <br> - 防止存取登錄編輯工具 <br> - 檔案附件的信任邏輯 <br> - 開啟指令碼執行 <br> [深入了解](https://technet.microsoft.com/library/gg176671(v=ws.10).aspx)|
| IIS | - 沒有預先存在的預設網站 <br> - 啟用[匿名驗證](https://technet.microsoft.com/library/cc731244(v=ws.10).aspx) \(英文\) <br> - 啟用 [FastCGI](https://technet.microsoft.com/library/cc753077(v=ws.10).aspx) 設定  <br> - 沒有預先存在的網站/應用程式接聽連接埠 443<br>|
| NIC 類型 | VMXNET3 (部署為 VMware VM 時) |
| IP 位址類型 | 靜態 |
| 網際網路存取 | 伺服器需要存取這些 URL： <br> - \*.accesscontrol.windows.net<br> - \*.backup.windowsazure.com <br>- \*.store.core.windows.net<br> - \*.blob.core.windows.net<br> - \*.hypervrecoverymanager.windowsazure.com <br> - https://management.azure.com <br> - *.services.visualstudio.com <br> - https://dev.mysql.com/get/Downloads/MySQLInstaller/mysql-installer-community-5.7.20.0.msi (相應放大處理序伺服器不需要) <br> - time.nist.gov <br> - time.windows.com |
| 連接埠 | 443 (控制通道協調流程)<br>9443 (資料傳輸)|

## <a name="download-the-latest-installation-file"></a>下載最新的安裝檔案

Site Recovery 入口網站中提供最新版本的組態伺服器安裝檔案。 此外，您可以從 [Microsoft 下載中心](http://aka.ms/unifiedsetup)下載。

1. 登入 Azure 入口網站並瀏覽至您的復原服務保存庫。
2. 瀏覽至 **Site Recovery 基礎結構** > **設定伺服器** \(位於 [適用於 VMware 和實體機器] 底下)。
3. 按一下 [+伺服器] 按鈕。
4. 在 [新增伺服器] 頁面上，按一下 [下載] 按鈕，下載註冊金鑰。 在組態伺服器安裝期間，您需要使用此金鑰向 Azure Site Recovery 服務註冊它。
5. 按一下 [下載 Microsoft Azure Site Recovery 整合安裝] 連結，以下載最新版本的組態伺服器。

  ![下載頁面](./media/physical-manage-configuration-server/downloadcs.png)


## <a name="install-and-register-the-server"></a>安裝和註冊伺服器

1. 執行統一安裝的安裝檔案。
2. 在 [開始之前] 選取 [安裝設定伺服器和處理序伺服器]。

    ![開始之前](./media/physical-manage-configuration-server/combined-wiz1.png)

3. 在 [協力廠商軟體授權] 中，按一下 [我接受] 來下載並安裝 MySQL。
4. 在 [網際網路設定] 中，指定在設定伺服器上執行的 Provider 要如何透過網際網路連接到 Azure Site Recovery。 確定您已允許必要的 URL。

    - 如果您想要使用電腦上目前設定的 Proxy 來連線，請選取 [使用 Proxy 伺服器連線至 Azure Site Recovery]。
    - 如果您想要讓提供者直接連接，請選取 [不使用 Proxy 伺服器直接連線到 Azure Site Recovery]。
    - 如果現有的 Proxy 需要驗證，或是您想要讓 Provider 使用自訂 Proxy 來連線，請選取 [以自訂 Proxy 設定連線]，並指定位址、連接埠和認證。
     ![防火牆](./media/physical-manage-configuration-server/combined-wiz4.png)
6. 在 [必要條件檢查] 中，安裝程式會執行檢查來確定可以執行安裝。 如果出現有關「通用時間同步處理檢查」的警告，請確認系統時鐘上的時間 ([日期和時間] 設定) 與時區相同。

    ![先決條件](./media/physical-manage-configuration-server/combined-wiz5.png)
7. 在 [MySQL 組態] 中，建立認證來登入已安裝的 MySQL 伺服器執行個體。

    ![MySQL](./media/physical-manage-configuration-server/combined-wiz6.png)
8. 在 [環境詳細資料] 中，選取您是否要複寫 VMware VM。 如果是的話，安裝程式就會檢查是否已安裝 PowerCLI 6.0。
9. 在 [安裝位置] 中，選取您要安裝二進位檔及儲存快取的位置。 您選取的磁碟機至少必須有 5 GB 的可用磁碟空間，但我們建議快取磁碟機至少有 600 GB 的可用空間。

    ![安裝位置](./media/physical-manage-configuration-server/combined-wiz8.png)
10. 在 [網路選取] 中，指定設定伺服器用來傳送和接收複寫資料的接聽程式 (網路介面卡和 SSL 連接埠)。 連接埠 9443 是用來傳送及接收複寫流量的預設連接埠，但您可以修改此連接埠號碼，以符合您的環境需求。 除了連接埠 9443 之外，我們也會開啟網頁伺服器用來協調複寫作業的連接埠 443。 請勿使用連接埠 443 來傳送或接收複寫流量。

    ![網路選擇](./media/physical-manage-configuration-server/combined-wiz9.png)


11. 在 [摘要] 中檢閱資訊，然後按一下 [安裝]。 安裝完成時，會產生複雜密碼。 在您啟用複寫時會需要它，所以請將它複製並保存在安全的位置。


註冊完成後，伺服器會顯示在保存庫的 [設定]  >  刀鋒視窗上。


## <a name="install-from-the-command-line"></a>從命令列安裝

執行安裝檔案，如下所示：

  ```
  UnifiedSetup.exe [/ServerMode <CS/PS>] [/InstallDrive <DriveLetter>] [/MySQLCredsFilePath <MySQL credentials file path>] [/VaultCredsFilePath <Vault credentials file path>] [/EnvType <VMWare/NonVMWare>] [/PSIP <IP address to be used for data transfer] [/CSIP <IP address of CS to be registered with>] [/PassphraseFilePath <Passphrase file path>]
  ```

### <a name="sample-usage"></a>範例用法
  ```
  MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted
  cd C:\Temp\Extracted
  UNIFIEDSETUP.EXE /AcceptThirdpartyEULA /servermode "CS" /InstallLocation "D:\" /MySQLCredsFilePath "C:\Temp\MySQLCredentialsfile.txt" /VaultCredsFilePath "C:\Temp\MyVault.vaultcredentials" /EnvType "VMWare"
  ```


### <a name="parameters"></a>參數

|參數名稱| 類型 | 說明| 值|
|-|-|-|-|
| /ServerMode|必要|指定應該同時安裝組態和處理序伺服器，還是只安裝處理序伺服器|CS<br>PS|
|/InstallLocation|必要|安裝元件的資料夾| 電腦上的任何資料夾|
|/MySQLCredsFilePath|必要|儲存 MySQL 伺服器認證的檔案路徑|此檔案應該具備如下所指定的格式|
|/VaultCredsFilePath|必要|保存庫認證檔的路徑|有效的檔案路徑|
|/EnvType|必要|您想要保護的環境類型 |VMware<br>NonVMware|
|/PSIP|必要|要用於複寫資料傳輸的 NIC IP 位址| 任何有效的 IP 位址|
|/CSIP|必要|接聽組態伺服器的 NIC IP 位址| 任何有效的 IP 位址|
|/PassphraseFilePath|必要|複雜密碼檔案的位置完整路徑|有效的檔案路徑|
|/BypassProxy|選用|指定組態伺服器不使用 Proxy 連接至 Azure|若要這樣做，請從 Venu 取得此值|
|/ProxySettingsFilePath|選用|Proxy 設定 (預設的 Proxy 需要驗證或自訂的 Proxy)|此檔案應該具備如下所指定的格式|
|DataTransferSecurePort|選用|要用於複寫資料的 PSIP 上的連接埠號碼| 有效的連接埠號碼 (預設值是 9433)|
|/SkipSpaceCheck|選用|略過快取磁碟的空間檢查| |
|/AcceptThirdpartyEULA|必要|旗標表示接受協力廠商使用者授權合約| |
|/ShowThirdpartyEULA|選用|顯示協力廠商使用者授權合約。 如果提供作為輸入，則會忽略所有其他參數| |



### <a name="create-file-input-for-mysqlcredsfilepath"></a>建立 MYSQLCredsFilePath 的檔案輸入

MySQLCredsFilePath 參數使用檔案作為輸入。 使用下列格式建立檔案，並將它以輸入 MySQLCredsFilePath 參數傳遞。
```
[MySQLCredentials]
MySQLRootPassword = "Password>"
MySQLUserPassword = "Password"
```
### <a name="create-file-input-for-proxysettingsfilepath"></a>建立 ProxySettingsFilePath 的檔案輸入
ProxySettingsFilePath 參數使用檔案做為輸入。 使用下列格式建立檔案，並將它以輸入 ProxySettingsFilePath 參數傳遞。

```
[ProxySettings]
ProxyAuthentication = "Yes/No"
Proxy IP = "IP Address"
ProxyPort = "Port"
ProxyUserName="UserName"
ProxyPassword="Password"
```
## <a name="modify-proxy-settings"></a>修改 Proxy 設定

您可以修改組態伺服器機器的 proxy 組態，如下所示：

1. 登入組態伺服器。
2. 使用捷徑啟動 cspsconfigtool.exe。
3. 按一下 [保存庫註冊] 索引標籤。
4. 從入口網站下載新的保存庫註冊檔案，並提供給工具作為輸入。

  ![註冊組態伺服器](./media/physical-manage-configuration-server/register-csconfiguration-server.png)
5. 提供新的 Proxy 詳細資料，然後按一下 [註冊] 按鈕。
6. 開啟系統管理 PowerShell 命令視窗。
7. 執行以下命令：
  ```
  $pwd = ConvertTo-SecureString -String MyProxyUserPassword
  Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber – ProxyUserName domain\username -ProxyPassword $pwd
  net stop obengine
  net start obengine
  ```

  >[!WARNING]
  如果您有連結至此組態伺服器的其他處理序伺服器，必須在部署中[修正所有相應放大處理序伺服器上的 Proxy 設定](vmware-azure-manage-process-server.md#modify-proxy-settings-for-an-on-premises-process-server)。

## <a name="reregister-a-configuration-server-with-the-same-vault"></a>使用相同保存庫註冊組態伺服器
  1. 登入您的組態伺服器。
  2. 使用桌面上的捷徑啟動 cspsconfigtool.exe。
  3. 按一下 [保存庫註冊] 索引標籤。
  4. 從入口網站下載新的註冊檔案，並提供給工具作為輸入。
        ![註冊組態伺服器](./media/physical-manage-configuration-server/register-csconfiguration-server.png)
  5. 提供 Proxy 伺服器詳細資料，然後按一下 [註冊] 按鈕。  
  6. 開啟系統管理 PowerShell 命令視窗。
  7. 執行下列命令

      ```
      $pwd = ConvertTo-SecureString -String MyProxyUserPassword
      Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber – ProxyUserName domain\username -ProxyPassword $pwd
      net stop obengine
      net start obengine
      ```

  >[!WARNING]
  如果您有多個處理序伺服器，必須[將它們重新註冊](vmware-azure-manage-process-server.md#reregister-a-process-server)。

## <a name="register-a-configuration-server-with-a-different-vault"></a>向不同保存庫註冊組態伺服器

> [!WARNING]
> 下列步驟會解除組態伺服器和目前保存庫之間的關聯，且組態伺服器底下所有受保護虛擬機器的複寫都會遭到停止。

1. 登入組態伺服器
2. 從系統管理命令提示字元中，執行命令：

    ```
    reg delete HKLM\Software\Microsoft\Azure Site Recovery\Registration
    net stop dra
    ```
3. 使用桌面上的捷徑啟動 cspsconfigtool.exe。
4. 按一下 [保存庫註冊] 索引標籤。
5. 從入口網站下載新的註冊檔案，並提供給工具作為輸入。
6. 提供 Proxy 伺服器詳細資料，然後按一下 [註冊] 按鈕。  
7. 開啟系統管理 PowerShell 命令視窗。
8. 執行下列命令
    ```
    $pwd = ConvertTo-SecureString -String MyProxyUserPassword
    Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber – ProxyUserName domain\username -ProxyPassword $pwd
    net stop obengine
    net start obengine
    ```

## <a name="upgrade-a-configuration-server"></a>升級組態伺服器

執行更新彙總來更新組態伺服器。 更新最多可以套用到 N-4 版本。 例如︰

- 如果您執行 9.7、9.8、9.9 或 9.10 - 您可以直接升級至 9.11。
- 如果您執行 9.6 或更早版本，而且需要升級至 9.11 版，您必須先升級到 9.7 版， 才能升級到 9.11。

用來升級至組態伺服器所有版本的更新彙總連結，可以在 [wiki 更新頁面](https://social.technet.microsoft.com/wiki/contents/articles/38544.azure-site-recovery-service-updates.aspx)取得。

升級伺服器，如下所示：

1. 將更新安裝程式檔案下載到組態伺服器上。
2. 按兩下以執行安裝程式。
3. 安裝程式會偵測機器上執行的目前版本。
4. 按一下 [確定] 以確認，並執行升級。 


## <a name="delete-or-unregister-a-configuration-server"></a>將設定伺服器刪除或取消註冊

> [!WARNING]
> 在開始解除委任組態伺服器之前，請確認下列事項。
> 1. [停用保護](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure)此組態伺服器下的所有虛擬機器。
> 2. 將組態伺服器的所有複寫原則[解除關聯](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy)並[刪除](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy)。
> 3. [刪除](vmware-azure-manage-vcenter.md#delete-a-vcenter-server)與組態伺服器相關聯的所有 VCenters 伺服器/vSphere 主機。


### <a name="delete-the-configuration-server-from-azure-portal"></a>從 Azure 入口網站刪除組態伺服器
1. 在 Azure 入口網站中，從 [保存庫] 功能表瀏覽至 [Site Recovery 基礎結構] > [組態伺服器]。
2. 按一下您需要解除委任的組態伺服器。
3. 在組態伺服器的詳細資料頁面上，按一下 [刪除] 按鈕。
4. 按一下 [是] 以確認刪除伺服器。

### <a name="uninstall-the-configuration-server-and-its-dependencies"></a>將組態伺服器軟體和其相依性解除安裝
  > [!TIP]
  如果您打算再次搭配 Azure Site Recovery 來重複使用組態伺服器，您可以直接跳至步驟 4

1. 以系統管理員身分登入組態伺服器。
2. 開啟 [控制台] > [程式集] > [解除安裝程式]
3. 依下列順序將程式解除安裝：
  * Microsoft Azure 復原服務代理程式
  * Microsoft Azure Site Recovery Mobility Service/主要目標伺服器
  * Microsoft Azure Site Recovery Provider
  * Microsoft Azure Site Recovery 設定伺服器/處理序伺服器
  * Microsoft Azure Site Recovery 組態伺服器相依性
  * MySQL Server 5.5
4. 從系統管理命令提示字元中，執行下列命令。
  ```
  reg delete HKLM\Software\Microsoft\Azure Site Recovery\Registration
  ```

## <a name="delete-or-unregister-a-configuration-server-powershell"></a>將組態伺服器刪除或取消註冊 (PowerShell)

1. [安裝](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-4.4.0) Azure PowerShell 模組
2. 使用命令登入您的 Azure 帳戶
    
    `Connect-AzureRmAccount`
3. 選取保存庫所在的訂用帳戶

     `Get-AzureRmSubscription –SubscriptionName <your subscription name> | Select-AzureRmSubscription`
3.  現在設定保存庫內容
    
    ```
    $vault = Get-AzureRmRecoveryServicesVault -Name <name of your vault>
    Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault
    ```
4. 選取組態伺服器

    `$fabric = Get-AzureRmSiteRecoveryFabric -FriendlyName <name of your configuration server>`
6. 刪除組態伺服器

    `Remove-AzureRmSiteRecoveryFabric -Fabric $fabric [-Force] `

> [!NOTE]
> Remove-AzureRmSiteRecoveryFabric 中的 **-Force** 選項可用來強制移除/刪除組態伺服器。

## <a name="renew-ssl-certificates"></a>更新 SSL 憑證
設定伺服器有內建的 Web 伺服器，可協調它所連線的行動服務、處理伺服器和主要目標伺服器的活動。 Web 伺服器使用 SSL 憑證來驗證用戶端。 憑證會在三年後到期，且可隨時更新。

### <a name="check-expiry"></a>檢查到期日

針對在 2016 年 5 月之前部署的設定伺服器，憑證到期設定為一年。 如果您有即將到期的憑證，會發生以下狀況：

- 當到期日是兩個月或少於兩個月時，服務會開始在入口網站傳送通知，以及透過電子郵件傳送通知 (如果您已訂閱 Azure Site Recovery 通知)。
- 通知橫幅會出現在保存庫資源頁面上。 按一下橫幅可取得詳細資料。
- 如果您看到 [立即升級] 按鈕，表示您的環境中有些元件尚未升級至 9.4.xxxx.x 或更新版本。 更新憑證之前請先升級元件。 您不能在舊版上更新。

### <a name="renew-the-certificate"></a>更新憑證

1. 在保存庫中，開啟 [Site Recovery 基礎結構] > [設定伺服器]，然後按一下所需的設定伺服器。
2. 到期日顯示在 [設定伺服器健康情況] 下
3. 按一下 [更新憑證]。 




## <a name="common-issues"></a>常見問題
[!INCLUDE [site-recovery-vmware-to-azure-install-register-issues](../../includes/site-recovery-vmware-to-azure-install-register-issues.md)]

## <a name="next-steps"></a>後續步驟

檢閱教學課程，了解如何設定[實體伺服器](tutorial-physical-to-azure.md)至 Azure 的災害復原。

