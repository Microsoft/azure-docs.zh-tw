---
title: 使用 Azure Site Recovery 管理 VMware 災害復原的設定伺服器 | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery 管理將 VMware 災害復原到 Azure 的現有設定伺服器。
author: rayne-wiselman
ms.service: site-recovery
ms.topic: article
ms.date: 07/06/2018
ms.author: raynew
ms.openlocfilehash: 0935867e835fe88568f1cdce1ea8dfcea14a451a
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43669310"
---
# <a name="manage-the-configuration-server-for-vmware-vms"></a>管理 VMware VM 的設定伺服器

當您使用 [Azure Site Recovery](site-recovery-overview.md) 將 VMware VM 和實體伺服器災害復原到 Azure 時，便會設定內部部署設定伺服器。 設定伺服器會協調內部部署 VMware 與 Azure 之間的通訊，以及管理資料複寫。 本文摘要說明部署設定伺服器之後，管理設定伺服器的一般工作。



## <a name="modify-vmware-settings"></a>修改 VMware 設定

您可以依照下列方式存取組態伺服器：
    - 登入該伺服器部署所在的 VM，然後從桌面捷徑啟動 [Azure Site Recovery 組態管理員]。
    - 或者，您可以經由 **https://*ConfigurationServerName*/:44315/** 從遠端存取組態伺服器。 使用系統管理員認證登入。

### <a name="modify-vmware-server-settings"></a>修改 VMware 伺服器設定

1. 若要為不同的 VMware 伺服器與設定伺服器建立關聯，請在登入後選取 [新增 vCenter Server/vSphere ESXi 伺服器]。
2. 輸入詳細資料，然後選取 [確定]。


### <a name="modify-credentials-for-automatic-discovery"></a>修改自動探索所需的認證

1. 若要更新用來連線至 VMware 伺服器以進行 VMware VM 自動探索的認證，請在登入後選取 [編輯]。
2. 輸入新的認證，然後選取 [確定] 。

    ![修改 VMware](./media/vmware-azure-manage-configuration-server/modify-vmware-server.png)


## <a name="modify-credentials-for-mobility-service-installation"></a>修改行動服務安裝的認證

針對您啟用複寫的 VMware VM，修改在其上自動安裝行動服務所用的認證。

1. 在登入後，選取 [管理虛擬機器認證]
2. 輸入新的認證，然後選取 [確定] 。

    ![修改行動服務認證](./media/vmware-azure-manage-configuration-server/modify-mobility-credentials.png)

## <a name="modify-proxy-settings"></a>修改 Proxy 設定

修改設定伺服器電腦用來透過網際網路存取 Azure 的 Proxy 設定。 如果您除了在設定伺服器電腦上執行的預設處理伺服器之外還有處理伺服器電腦，則請修改兩部電腦上的設定。

1. 在登入組態伺服器後，選取 [管理連線]。
2. 更新 Proxy 值。 然後選取 [儲存] 以更新設定。

## <a name="add-a-network-adapter"></a>新增網路介面卡

開放式虛擬化格式 (OVF) 範本會部署包含單一網路介面卡的設定伺服器 VM。

- 您可以[將其他介面卡新增至 VM](vmware-azure-deploy-configuration-server.md#add-an-additional-adapter)，但是必須在保存庫中註冊設定伺服器之前新增。
- 若要在保存庫中註冊設定伺服器之後新增介面卡，請在 VM 屬性中新增介面卡。 然後，您必須在保存庫中註冊伺服器。


## <a name="reregister-a-configuration-server-in-the-same-vault"></a>在同一個保存庫中註冊設定伺服器

如果需要，您可以在同一個保存庫中註冊設定伺服器。 如果您除了在設定伺服器電腦上執行的預設處理伺服器之外，還有其他處理伺服器電腦，則請註冊這兩部電腦。


  1. 在保存庫中，開啟 [管理] > [Site Recovery 基礎結構] > [設定伺服器]。
  2. 在 [伺服器] 中，選取 [下載註冊金鑰] 以下載保存庫認證檔案。
  3. 登入設定伺服器電腦。
  4. 在 **%ProgramData%\ASR\home\svsystems\bin** 中，開啟 **cspsconfigtool.exe**。
  5. 在 [保存庫註冊] 索引標籤上，選取 [瀏覽] 並找出您下載的保存庫認證檔。
  6. 如果需要，請提供 Proxy 伺服器詳細資料。 接著，選取 [註冊]。
  7. 開啟系統管理 PowerShell 命令視窗並執行下列命令：
   ```
      $pwd = ConvertTo-SecureString -String MyProxyUserPassword
      Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber – ProxyUserName domain\username -ProxyPassword $pwd
   ```

      >[!NOTE]
      >若要從設定伺服器**提取最新憑證**以相應放大伺服器，請執行命令 *“<安裝磁碟機\Microsoft Azure Site Recovery\agent\cdpcli.exe>" --registermt*

  8. 最後，執行下列命令以重新啟動 obengine。
  ```
          net stop obengine
          net start obengine
  ```
  
## <a name="upgrade-the-configuration-server"></a>升級設定伺服器

您執行更新彙總來更新設定伺服器。 更新最多可以套用到 N-4 版本。 例如︰

- 如果您執行 9.7、9.8、9.9 或 9.10，就可以直接升級至 9.11。
- 如果您執行 9.6 或更早版本，而且想要升級至 9.11，則在升級到 9.11 之前，必須先升級到 版本 9.7。

用來升級至組態伺服器所有版本的更新彙總連結，可以在 [wiki 更新頁面](https://social.technet.microsoft.com/wiki/contents/articles/38544.azure-site-recovery-service-updates.aspx)取得。

升級伺服器，如下所示：

1. 在保存庫中，移至 [管理] > [Site Recovery 基礎結構] > [組態伺服器]。
2. 如果有可用的更新，[代理程式版本] > [資料行] 中會出現連結。
    更新![](./media/vmware-azure-manage-configuration-server/update2.png)
3. 將更新安裝程式檔案下載到組態伺服器上。

    ![更新](./media/vmware-azure-manage-configuration-server/update1.png)

4. 按兩下以執行安裝程式。
5. 安裝程式會偵測機器上執行的目前版本。 按一下 [是] 開始進行升級。
6. 升級完成時，會驗證伺服器組態。

    ![更新](./media/vmware-azure-manage-configuration-server/update3.png)

7. 按一下 [完成] 以關閉安裝程式。

## <a name="delete-or-unregister-a-configuration-server"></a>將設定伺服器刪除或取消註冊

1. 對組態伺服器下的所有 VM [停用保護](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure)。
2. 將設定伺服器的所有複寫原則[解除關聯](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy)並[刪除](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy)。
3. [刪除](vmware-azure-manage-vcenter.md#delete-a-vcenter-server)與設定伺服器關聯的所有 vCenter 伺服器/vSphere 主機。
4. 在保存庫中，開啟 [Site Recovery 基礎結構] > [設定伺服器]。
5. 選取您想要移除的設定伺服器。 然後在 [詳細資料] 頁面上選取 [刪除]。

    ![刪除設定伺服器](./media/vmware-azure-manage-configuration-server/delete-configuration-server.png)


### <a name="delete-with-powershell"></a>以 PowerShell 刪除

您可以選擇使用 PowerShell 來刪除設定伺服器。

1. [安裝](https://docs.microsoft.com/powershell/azure/install-azurerm-ps?view=azurermps-4.4.0) Azure PowerShell 模組。
2. 使用以下命令登入您的 Azure 帳戶：

    `Connect-AzureRmAccount`
3. 選取保存庫訂用帳戶。

     `Get-AzureRmSubscription –SubscriptionName <your subscription name> | Select-AzureRmSubscription`
3.  設定保存庫內容。

    ```
    $vault = Get-AzureRmRecoveryServicesVault -Name <name of your vault>
    Set-AzureRmSiteRecoveryVaultSettings -ARSVault $vault
    ```
4. 擷取設定伺服器。

    `$fabric = Get-AzureRmSiteRecoveryFabric -FriendlyName <name of your configuration server>`
6. 刪除組態伺服器。

    `Remove-AzureRmSiteRecoveryFabric -Fabric $fabric [-Force] `

> [!NOTE]
> 您可以使用 Remove-AzureRmSiteRecoveryFabric 中的 **-Force** 選項來強制刪除設定伺服器。

## <a name="generate-configuration-server-passphrase"></a>產生設定伺服器複雜密碼

1. 登入組態伺服器，然後以系統管理員身分開啟命令提示字元視窗。
2. 若要將目錄切換至 bin 資料夾，執行命令 **cd %ProgramData%\ASR\home\svsystems\bin**
3. 若要產生複雜密碼檔案，請執行 **genpassphrase.exe -v > MobSvc.passphrase**。
4. 您的複雜密碼將會儲存在位於 **%ProgramData%\ASR\home\svsystems\bin\MobSvc.passphrase** 的這個檔案中。

## <a name="renew-ssl-certificates"></a>更新 SSL 憑證

設定伺服器有內建的 Web 伺服器，可協調它所連線的行動服務、處理伺服器和主要目標伺服器的活動。 Web 伺服器使用 SSL 憑證來驗證用戶端。 憑證會在三年後到期，且可隨時更新。

### <a name="check-expiry"></a>檢查到期日

針對在 2016 年 5 月之前部署的設定伺服器，憑證到期設定為一年。 如果您有即將到期的憑證，會發生以下狀況：

- 當到期日是兩個月或少於兩個月時，服務會開始在入口網站傳送通知，以及透過電子郵件傳送通知 (如果您已訂閱 Site Recovery 通知)。
- 通知橫幅會出現在保存庫資源頁面上。 如需詳細資訊，請選取橫幅。
- 如果您看到 [立即升級] 按鈕，表示您的環境中有些元件尚未升級至 9.4.xxxx.x 或更新版本。 更新憑證之前請先升級元件。 您不能在舊版上更新。

### <a name="renew-the-certificate"></a>更新憑證

1. 在保存庫中，開啟 [Site Recovery 基礎結構] > [設定伺服器]。 選取必要的設定伺服器。
2. 到期日顯示在 [設定伺服器健康情況] 下。
3. 選取 [更新憑證]。

## <a name="update-windows-licence"></a>更新 Windows 授權

使用 OVF 範本所提供的授權是有效期 180 天的評估授權。 如需不中斷的使用權限，您必須使用購買的授權來啟用 Windows。

## <a name="failback-requirements"></a>容錯回復需求

在重新保護和容錯回復期間，內部部署組態伺服器必須會執行並處於連線連線狀態。 若要成功完成容錯回復，要容錯回復的虛擬機器必須存在於組態伺服器資料庫中。

請確定您定期排程備份組態伺服器。 如果發生災害，而且遺失組態伺服器，您必須先從備份副本還原組態伺服器，並確保還原的組態伺服器 IP 位址與註冊到保存庫的 IP 位址相同。 如果還原的組態伺服器使用不同 IP 位址，則容錯回復將無法運作。

## <a name="next-steps"></a>後續步驟

檢閱將 [VMware VM](vmware-azure-tutorial.md) 設定為災害復原至 Azure 的教學課程。
