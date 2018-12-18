---
title: 使用 Azure 備份伺服器來備份 VMware 伺服器
description: 使用 Azure 備份伺服器，將 VMware vCenter/ESXi 伺服器備份至 Azure 或磁碟。 本文提供備份 (或保護) 您 VMware 工作負載的逐步指示。
services: backup
author: markgalioto
manager: carmonm
ms.service: backup
ms.topic: conceptual
ms.date: 07/24/2017
ms.author: adigan
ms.openlocfilehash: ce7b255359c076ddae642ed44f056e444b655e25
ms.sourcegitcommit: 248c2a76b0ab8c3b883326422e33c61bd2735c6c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/23/2018
ms.locfileid: "39216405"
---
# <a name="back-up-a-vmware-server-to-azure"></a>將 VMware 伺服器備份到 Azure

本文說明如何設定 Azure 備份伺服器來協助保護 VMware 伺服器工作負載。 本文假設您已安裝 Azure 備份伺服器。 如果您尚未安裝 Azure 備份伺服器，請參閱[使用 Azure 備份伺服器準備將工作負載進行備份](backup-azure-microsoft-azure-backup.md)。

Azure 備份伺服器可以備份或協助保護 VMware vCenter Server 6.5、6.0 和 5.5 版。


## <a name="create-a-secure-connection-to-the-vcenter-server"></a>建立 vCenter Server 的安全連線

Azure 備份伺服器預設會透過 HTTPS 通道來與各個 vCenter Server 進行通訊。 若要開啟安全通訊，建議您在 Azure 備份伺服器上安裝 VMware 憑證授權單位 (CA) 憑證。 如果您不需要安全通訊，而且想要停用 HTTPS 要求，請參閱[停用安全通訊協定](backup-azure-backup-server-vmware.md#disable-secure-communication-protocol)。 若要建立 Azure 備份伺服器與 vCenter Server 之間的安全連線，請將 Azure 備份伺服器上的受信任憑證匯入。

通常您會透過 vSphere Web 用戶端，使用 Azure 備份伺服器電腦上的瀏覽器連線到 vCenter Server。 您第一次使用 Azure 備份伺服器的瀏覽器連線到 vCenter Server 時，連線為不安全。 下圖顯示不安全的連線。

![VMware 伺服器的不安全連線範例](./media/backup-azure-backup-server-vmware/unsecure-url.png)

若要修正此問題，並建立安全連線，請下載受信任的根 CA 憑證。

1. 在 Azure 備份伺服器上的瀏覽器中，輸入 vSphere Web 用戶端的 URL。 vSphere Web 用戶端登入頁面隨即出現。

    ![vSphere Web 用戶端](./media/backup-azure-backup-server-vmware/vsphere-web-client.png)

    在系統管理員和開發人員的資訊底部找到 [下載受信任根 CA 憑證] 連結。

    ![用來下載受信任根 CA 憑證的連結](./media/backup-azure-backup-server-vmware/vmware-download-ca-cert-prompt.png)

  如果您沒有看到 vSphere Web 用戶端登入頁面，請檢查您瀏覽器的 Proxy 設定。

2. 按一下 [下載受信任的根 CA 憑證]。

    vCenter Server 會將檔案下載到本機電腦。 檔案名稱為 **download**。 依您的瀏覽器而定，您會收到詢問您要開啟還是儲存檔案的訊息。

    ![下載憑證後，下載訊息](./media/backup-azure-backup-server-vmware/download-certs.png)

3. 將檔案儲存至 Azure 備份伺服器上的位置。 當您儲存檔案時，請新增 .zip 副檔名。

    檔案是包含憑證相關資訊的 .zip 檔。 使用 .zip 副檔名，您就可以使用解壓縮工具。

4. 以滑鼠右鍵按一下 **download.zip**，然後選取 [解壓縮全部]，將內容解壓縮。

    .zip 檔案會將其內容解壓縮到名為 **certs** 的資料夾。 certs 資料夾中會出現兩種檔案類型。 根憑證檔案，其副檔名的開頭是編號序列，例如 .0 和 .1。
    
    CRL 檔案，其副檔名的開頭是序列，例如 .r0 和 .r1。 與憑證相關聯的 CRL 檔案。

    ![download 檔案會在本機解壓縮 ](./media/backup-azure-backup-server-vmware/extracted-files-in-certs-folder.png)

5. 在 **certs** 資料夾中，以滑鼠右鍵按一下根憑證檔案，然後按一下 [重新命名]。

    ![將根憑證重新命名 ](./media/backup-azure-backup-server-vmware/rename-cert.png)

    將根憑證的副檔名變更為 .crt。 當系統詢問您是否確定要變更副檔名時，請按一下 [是] 或 [確定]。 否則，請變更檔案的預定功能。 檔案的圖示會變更為代表根憑證的圖示。

6. 以滑鼠右鍵按一下根憑證，然後從快顯功能表中選取 [安裝憑證]。

    [憑證匯入精靈] 對話方塊會隨即出現。

7. 在 [憑證匯入精靈] 對話方塊中，選取 [本機電腦] 作為憑證目的地，然後按 [下一步] 繼續執行。

    ![憑證存放區目的地選項 ](./media/backup-azure-backup-server-vmware/certificate-import-wizard1.png)

    如果系統詢問您是否要允許對電腦進行變更，請對所有變更按一下 [是] 或 [確定]。

8. 在 [憑證存放區] 頁面上，選取 [將所有憑證放入以下的存放區]，然後按一下 [瀏覽] 來選擇憑證存放區。

    ![將憑證放在特定的儲存位置](./media/backup-azure-backup-server-vmware/cert-import-wizard-local-store.png)

    [選取憑證存放區] 對話方塊隨即出現。

    ![憑證存放區資料夾階層](./media/backup-azure-backup-server-vmware/cert-store.png)

9. 選取 [受信任的根憑證授權單位] 作為憑證的目的地資料夾，然後按一下 [確定]。

    ![憑證目的資料夾](./media/backup-azure-backup-server-vmware/certificate-store-selected.png)

    系統會確認將 [受信任的根憑證授權單位] 資料夾作為憑證存放區。 按 [下一步] 。

    ![憑證存放區資料夾](./media/backup-azure-backup-server-vmware/certificate-import-wizard2.png)

10. 在 [完成憑證匯入精靈] 頁面上，確認憑證是否位於所要的資料夾中，然後按一下 [完成] 以完成精靈。

    ![確認憑證位於適當的資料夾](./media/backup-azure-backup-server-vmware/cert-wizard-final-screen.png)

    此時會出現一個對話方塊，向您確認已成功匯入憑證。

11. 請登入 vCenter Server 以確認您的連線是否安全。

  如果系統未成功匯入憑證，而且您也無法建立安全的連線，請參閱有關[取得伺服器憑證](http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.wssdk.dsg.doc/sdk_sg_server_certificate_Appendixes.6.4.html)的 VMware vSphere 文件。

  如果您的組織設有安全界限，而且不想開啟 HTTPS 通訊協定，請使用下列程序來停用安全通訊。

### <a name="disable-secure-communication-protocol"></a>停用安全通訊協定

如果您的組織不需要 HTTPS 通訊協定，請使用下列步驟來停用 HTTPS。 若要停用預設行為，請建立會忽略預設行為的登錄機碼。

1. 複製以下文字並貼到 .txt 檔案中。

  ```
  Windows Registry Editor Version 5.00
  [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Data Protection Manager\VMWare]
  "IgnoreCertificateValidation"=dword:00000001
  ```

2. 將檔案儲存到 Azure 備份伺服器電腦。 至於檔案名稱，請使用 DisableSecureAuthentication.reg。

3. 按兩下檔案以啟動登錄項目。


## <a name="create-a-role-and-user-account-on-the-vcenter-server"></a>在 vCenter Server 上建立角色和使用者帳戶

在 vCenter Server 上，角色是一組預先定義的權限。 vCenter Server 系統管理員會建立角色。 為了指派權限，系統管理員會將使用者帳戶與某個角色進行配對。 若要建立所需的使用者認證以便備份 vCenter Server 電腦，請建立具有特定權限的角色，並讓使用者帳戶與該角色產生關聯。

Azure 備份伺服器會使用使用者名稱和密碼來驗證 vCenter Server。 Azure 備份伺服器會使用這些認證作為所有備份作業的驗證。

若要為備份管理員新增 vCenter Server 角色和其權限︰

1. 登入 vCenter Server，然後在 vCenter Server 的 [導覽] 面板中，按一下 [管理]。

    ![vCenter Server 導覽面板中的管理選項](./media/backup-azure-backup-server-vmware/vmware-navigator-panel.png)

2. 在 [管理] 中選取 [角色]，然後在 [角色] 面板中按一下 [新增角色] 圖示(+ 符號)。

    ![新增角色](./media/backup-azure-backup-server-vmware/vmware-define-new-role.png)

    [建立角色] 對話方塊隨即出現。

    ![建立角色](./media/backup-azure-backup-server-vmware/vmware-define-new-role-priv.png)

3. 在 [建立角色] 對話方塊的 [角色名稱] 方塊中，輸入「BackupAdminRole」。 角色名稱可以是您喜歡的名稱，但它應該是可辨識該角色的用途。

4. 選取適當 vCenter 版本的權限，然後按一下 [確定]。 下表指出 vCenter 6.0 和 vCenter 5.5 所需的權限。

  選取權限時，請按一下父標籤旁的圖示，以展開父權限並檢視子權限。 若要選取 VirtualMachine 權限，您必須深入父子式階層中的好幾層。 您不需要選取父代權限內的所有子權限。

  ![父子式權限階層](./media/backup-azure-backup-server-vmware/cert-add-privilege-expand.png)

  在按一下 [確定] 後，[角色] 面板上的清單中就會出現新的角色。

|vCenter 6.0 和 6.5 的權限| Vcenter 5.5 的權限|
|----------------------------------|---------------------------|
|Datastore.AllocateSpace   | Datastore.AllocateSpace|
|Global.ManageCustomFields | Global.ManageCustomerFields|
|Global.SetCustomFields    |   |
|Host.Local.CreateVM       | Network.Assign |
|Network.Assign            |  |
|Resource.AssignVMToPool   |  |
|VirtualMachine.Config.AddNewDisk  | VirtualMachine.Config.AddNewDisk   |
|VirtualMachine.Config.AdvanceConfig| VirtualMachine.Config.AdvancedConfig|
|VirtualMachine.Config.ChangeTracking| VirtualMachine.Config.ChangeTracking |
|VirtualMachine.Config.HostUSBDevice||
|VirtualMachine.Config.QueryUnownedFiles|    |
|VirtualMachine.Config.SwapPlacement| VirtualMachine.Config.SwapPlacement |
|VirtualMachine.Interact.PowerOff| VirtualMachine.Interact.PowerOff |
|VirtualMachine.Inventory.Create| VirtualMachine.Inventory.Create |
|VirtualMachine.Provisioning.DiskRandomAccess| |
|VirtualMachine.Provisioning.DiskRandomRead|VirtualMachine.Provisioning.DiskRandomRead |
|VirtualMachine.State.CreateSnapshot| VirtualMachine.State.CreateSnapshot|
|VirtualMachine.State.RemoveSnapshot|VirtualMachine.State.RemoveSnapshot |
</br>



## <a name="create-a-vcenter-server-user-account-and-permissions"></a>建立 vCenter Server 使用者帳戶和權限

在設定了具有權限的角色後，請建立使用者帳戶。 使用者帳戶具有名稱和密碼，可提供用於驗證的認證。

1. 若要建立使用者帳戶，請在 vCenter Server 的 [導覽] 面板中按一下 [使用者和群組]。

    ![[使用者和群組] 選項](./media/backup-azure-backup-server-vmware/vmware-userandgroup-panel.png)

    [vCenter 使用者和群組] 面板隨即出現。

    ![[vCenter 使用者和群組] 面板](./media/backup-azure-backup-server-vmware/usersandgroups.png)

2. 在 [vCenter 使用者和群組] 面板中，選取 [使用者] 索引標籤，然後按一下 [新增使用者] 圖示 (+ 符號)。

    [新增使用者] 對話方塊隨即出現。

3. 在 [新增使用者] 對話方塊中新增使用者的資訊，然後按一下 [確定]。 在此程序中，使用者名稱是 BackupAdmin。

    ![[新增使用者] 對話方塊](./media/backup-azure-backup-server-vmware/vmware-new-user-account.png)

    新的使用者帳戶會出現在清單中。

4. 若要讓使用者帳戶與角色產生關聯，請在 [導覽] 面板中按一下 [通用權限]。 在 [通用權限] 面板中，選取 [管理] 索引標籤，然後按一下 [新增] 圖示 (+ 符號)。

    ![[通用權限] 面板](./media/backup-azure-backup-server-vmware/vmware-add-new-perms.png)

    [通用權限根目錄 - 新增權限] 對話方塊隨即出現。

5. 在 [通用權限根目錄 - 新增權限] 對話方塊中，按一下 [新增] 來選擇使用者或群組。

    ![選擇使用者或群組](./media/backup-azure-backup-server-vmware/vmware-add-new-global-perm.png)

    [選取使用者/群組] 對話方塊隨即出現。

6. 在 [選取使用者/群組] 對話方塊中，選擇 [BackupAdmin]，然後按一下 [新增]。

    在 [使用者] 中，使用者帳戶會使用「網域\使用者名稱」格式。 如果您想要使用不同的網域，請從 [網域] 清單中選擇。

    ![新增 BackupAdmin 使用者](./media/backup-azure-backup-server-vmware/vmware-assign-account-to-role.png)

    按一下 [確定] 將選取的使用者新增至 [新增權限] 對話方塊。

7. 您現在已找出使用者，請將使用者指派給角色。 在 [指派的角色] 中，從下拉式清單中選取 [BackupAdminRole]，然後按一下 [確定]。

    ![將使用者指派給角色](./media/backup-azure-backup-server-vmware/vmware-choose-role.png)

  在 [通用權限] 面板的 [管理] 索引標籤上，新的使用者帳戶和相關聯的角色便會出現在清單中。


## <a name="establish-vcenter-server-credentials-on-azure-backup-server"></a>在 Azure 備份伺服器上建立 vCenter Server 認證

在將 VMware 伺服器新增至 Azure 備份伺服器之前，請先安裝 [Azure 備份伺服器的更新 1](https://support.microsoft.com/help/3175529/update-1-for-microsoft-azure-backup-server)。

1. 若要開啟 Azure 備份伺服器，請按兩下 Azure 備份伺服器桌面上的圖示。

    ![Azure 備份伺服器圖示](./media/backup-azure-backup-server-vmware/mabs-icon.png)

    如果您在桌面上找不到此圖示，請從已安裝的應用程式清單開啟 Azure 備份伺服器。 Azure 備份伺服器應用程式名稱是「Microsoft Azure 備份」。

2. 在 Azure 備份伺服器主控台中按一下 [管理]，按一下 [生產伺服器]，然後在工具功能區中按一下 [管理 VMware]。

    ![Azure 備份伺服器主控台](./media/backup-azure-backup-server-vmware/add-vmware-credentials.png)

    [管理認證] 對話方塊隨即出現。

    ![Azure 備份伺服器的 [管理認證] 對話方塊](./media/backup-azure-backup-server-vmware/mabs-manage-credentials-dialog.png)

3. 在 [管理認證] 對話方塊中，按一下 [新增] 以開啟 [新增認證] 對話方塊。

4. 在 [新增認證] 對話方塊中，輸入新認證的名稱和描述。 然後，指定使用者名稱和密碼。 我們會使用「Contoso Vcenter 認證」這個名稱來識別下一個程序中的認證。 所使用的使用者名稱和密碼請與 vCenter Server 所使用的相同。 如果 vCenter Server 和 Azure 備份伺服器不在相同網域中，請在 [使用者名稱] 中指定網域。

    ![Azure 備份伺服器的 [新增認證] 對話方塊](./media/backup-azure-backup-server-vmware/mabs-add-credential-dialog2.png)

    按一下 [新增] 將新的認證新增至 Azure 備份伺服器。 [管理認證] 對話方塊的清單中便會出現新的認證。
    
    ![Azure 備份伺服器的 [管理認證] 對話方塊](./media/backup-azure-backup-server-vmware/new-list-of-mabs-creds.png)

5. 若要關閉 [管理認證] 對話方塊，請按一下右上角的 **X**。


## <a name="add-the-vcenter-server-to-azure-backup-server"></a>將 vCenter Server 新增至 Azure 備份伺服器

我們會使用「生產伺服器新增精靈」來對 Azure 備份伺服器新增 vCenter Server。

若要開啟「生產伺服器新增精靈」，請完成下列程序：

1. 在 Azure 備份伺服器主控台中，依序按一下 [管理]、[生產伺服器] 及 [新增]。

    ![開啟「生產伺服器新增精靈」](./media/backup-azure-backup-server-vmware/add-vcenter-to-mabs.png)

    [生產伺服器新增精靈] 對話方塊隨即出現。

    ![生產伺服器新增精靈](./media/backup-azure-backup-server-vmware/production-server-add-wizard.png)

2. 在 [選取生產伺服器類型] 頁面上選取 [VMware 伺服器]，然後按 [下一步]。

3. 在 [伺服器名稱/IP 位址] 中，指定 VMware 伺服器的完整網域名稱 (FQDN) 或 IP 位址。 如果所有 ESXi 伺服器均由相同的 vCenter 管理，您可以使用 vCenter 名稱。

    ![指定 VMware 伺服器的 FQDN 或 IP 位址](./media/backup-azure-backup-server-vmware/add-vmware-server-provide-server-name.png)

4. 在 [SSL 連接埠] 中，輸入用來與 VMware 伺服器通訊的連接埠。 使用連接埠 443，這是預設連接埠，除非您知道需要不同的連接埠。

5. 在 [指定認證] 中，選取您稍早建立的認證。

    ![指定認證](./media/backup-azure-backup-server-vmware/identify-creds.png)

6. 按一下 [新增] 將 VMware 伺服器新增至 [已新增的 VMware 伺服器] 清單，然後按 [下一步] 移至精靈的下一頁。

    ![新增 VMWare 伺服器和認證](./media/backup-azure-backup-server-vmware/add-vmware-server-credentials.png)

7. 在 [摘要] 頁面中，按一下 [新增] 將指定的 VMware 伺服器新增至 Azure 備份伺服器。

    ![將 VMware 伺服器新增至 Azure 備份伺服器](./media/backup-azure-backup-server-vmware/tasks-screen.png)

  VMware 伺服器備份是無代理程式的備份，因此新的伺服器立即就能完成新增。 [完成] 頁面會顯示結果。

  ![[完成] 頁面](./media/backup-azure-backup-server-vmware/summary-screen.png)

  若要將多個 vCenter Server 執行個體新增至 Azure 備份伺服器，請重複本節的先前步驟。

在將 vCenter Server 新增到 Azure 備份伺服器後，下一個步驟是建立保護群組。 保護群組可指定短期或長期保留的各種詳細資料，而且這是您定義及套用備份原則的地方。 備份原則包含執行備份的排程以及備份的內容。


## <a name="configure-a-protection-group"></a>設定保護群組

如果您未曾使用 System Center Data Protection Manager 或 Azure 備份伺服器，請參閱[規劃磁碟備份](https://technet.microsoft.com/library/hh758026.aspx)，以準備您的硬體環境。 在您檢查過您有適當的儲存體後，請使用 [建立新保護群組] 精靈來新增 VMware 虛擬機器。

1. 在 Azure 備份伺服器主控台中，按一下 [保護]，然後按一下工具功能區中的 [新增]，以開啟 [建立新保護群組] 精靈。

    ![開啟「建立新保護群組」精靈](./media/backup-azure-backup-server-vmware/open-protection-wizard.png)

    [建立新保護群組] 精靈對話方塊隨即出現。

    ![[建立新保護群組] 精靈對話方塊](./media/backup-azure-backup-server-vmware/protection-wizard.png)

    按 [下一步] 前往 [選取保護群組類型] 頁面。

2. 在 [選取保護群組類型] 頁面上，選取 [伺服器]，然後按 [下一步]。 [選取群組成員] 頁面隨即出現。

3. [選取群組成員] 頁面上會出現可用的成員和已選取的成員。 選取您想要保護的成員，然後按 [下一步]。

    ![選擇群組成員](./media/backup-azure-backup-server-vmware/server-add-selected-members.png)

    當您選取成員時，如果您選取的資料夾包含其他資料夾或 VM，則您會同時選取這些資料夾和 VM。 在父資料夾中包含資料夾和 VM 稱為資料夾層級保護。 若要移除資料夾或 VM，請清除核取方塊。

    如果 VM 或包含 VM 的資料夾已受到 Azure 保護，您就無法再次選取該 VM。 也就是說，VM 在受到 Azure 保護後就無法再次受到保護，這可防止針對一部 VM 建立重複的復原點。 如果您想要查看哪個 Azure 備份伺服器執行個體已經保護成員，請指向該成員以查看保護伺服器的名稱。

4. 在 [選取資料保護方式] 頁面上，輸入保護群組的名稱。 選取短期保護 (磁碟) 和線上保護。 如果您想要使用線上保護 (Azure)，必須使用磁碟的短期保護。 按 [下一步] 以繼續進行短期保護範圍。

    ![選擇資料保護方式](./media/backup-azure-backup-server-vmware/name-protection-group.png)

5. 在 [指定短期目標] 頁面上，針對 [保留範圍]，指定您想要保留復原點「儲存到磁碟」的天數。 如果您想要變更取得復原點的時間和天數，請按一下 [修改]。 短期復原點是完整備份。 而不是增量備份。 當您滿意短期目標時，請按 [下一步]。

    ![指定短期目標](./media/backup-azure-backup-server-vmware/short-term-goals.png)

6. 在 [檢閱磁碟配置] 頁面上，檢閱並視需要修改 VM 的磁碟空間。 建議的磁碟配置是以 [指定短期目標] 頁面中指定的保留範圍、工作負載類型及受保護的資料大小 (可在步驟 3 中找到) 為基礎。  

  - **資料大小：** 保護群組中的資料大小。
  - **磁碟空間：** 保護群組的建議磁碟空間數量。 如果您想要修改此設定，您配置的總空間應該稍微大於您預估每個資料來源將成長的數量。
  - **共置資料：** 如果您開啟共置功能，受保護的多個資料來源可以對應至單一複本和復原點磁碟區。 不支援所有工作負載的共置。
  - **自動成長：** 如果您開啟此設定，而且受保護群組中的資料成長超過初始配置，則 System Center Data Protection Manager 會嘗試增加 25% 的磁碟大小。
  - **儲存集區詳細資料：** 顯示儲存體集區的狀態，包括總計和剩餘的磁碟大小。

    ![檢閱磁碟配置](./media/backup-azure-backup-server-vmware/review-disk-allocation.png)

    當您滿意空間配置時，請按 [下一步]。

7. 在 [選擇複本的建立方式] 頁面上，指定您要在 Azure 備份伺服器上產生受保護資料之初始副本或複本的方式。

    預設值是 [自動透過網路] 和 [立即]。 如果您使用預設值，建議您指定離峰時間。 選擇 [稍後] 並指定日期與時間。

    對於大量資料或較差的網路狀況，請考慮使用卸除式媒體來離線複寫資料。

    做好選擇後，請按 [下一步]。

    ![選擇複本的建立方式](./media/backup-azure-backup-server-vmware/replica-creation.png)

8. 在 [一致性檢查選項] 頁面上，選取如何及何時自動執行一致性檢查。 當複本資料變得不一致時，或依據設定的排程，您可以執行一致性檢查。

    如果您不想設定自動一致性檢查，可以執行手動檢查。 在 Azure 備份伺服器主控台的 [保護] 區域，以滑鼠右鍵按一下保護群組，然後選取 [執行一致性檢查]。

    按 [下一步] 移至下一頁。

9. 在 [指定線上保護資料] 頁面上，選取您想要保護的一或多個資料來源。 您可以個別地選取成員，或按一下 [全選] 來選擇所有成員。 在選擇成員後，請按 [下一步]。

    ![指定線上保護資料](./media/backup-azure-backup-server-vmware/select-data-to-protect.png)

10. 在 [指定線上備份排程] 頁面上，指定用來從磁碟備份產生復原點的排程。 復原點在產生後會傳輸至 Azure 中的復原服務保存庫。 當您滿意線上備份排程時，請 [下一步]。

    ![指定線上備份排程](./media/backup-azure-backup-server-vmware/online-backup-schedule.png)

11. 在 [指定線上保留原則] 頁面上，指出您要在 Azure 中保留備份資料的時間長度。 在定義原則之後，請按 [下一步]。

    ![指定線上保留期原則](./media/backup-azure-backup-server-vmware/retention-policy.png)

    您可以在 Azure 中保留資料的時間長度沒有限制。 當您在 Azure 中儲存復原點資料時，唯一的限制是每個受保護的執行個體不能有超過 9999 個復原點。 在此範例中，受保護的執行個體是 VMware 伺服器。

12. 在 [摘要] 頁面上檢閱保護群組成員和設定的詳細資料，然後按一下 [建立群組]。

    ![保護群組成員和設定的摘要](./media/backup-azure-backup-server-vmware/protection-group-summary.png)

## <a name="next-steps"></a>後續步驟
如果您使用 Azure 備份伺服器來保護 VMware 工作負載，對於要如何使用 Azure 備份伺服器來協助保護 [Microsoft Exchange Server](./backup-azure-exchange-mabs.md)、[Microsoft SharePoint 伺服器陣列](./backup-azure-backup-sharepoint-mabs.md)或 [SQL Server 資料庫](./backup-azure-sql-mabs.md)，想必您會感到興趣。

如需註冊代理程式、設定保護群組或備份作業等問題的相關資訊，請參閱[針對 Azure 備份伺服器進行疑難排解](./backup-azure-mabs-troubleshoot.md)。
