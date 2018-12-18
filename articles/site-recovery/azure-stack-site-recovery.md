---
title: 使用 Azure Site Recovery 將 Azure Stack VM 複寫至 Azure | Microsoft Docs
description: 了解如何使用 Azure Site Recovery 服務設定將 Azure Stack VM 進行災害復原至 Azure。
services: site-recovery
author: rayne-wiselman
manager: carmonm
ms.topic: conceptual
ms.service: site-recovery
ms.date: 08/30/2018
ms.author: raynew
ms.openlocfilehash: c71f683355a09c8ba2381db406eeb1ccabdb7afa
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43697691"
---
# <a name="replicate-azure-stack-vms-to-azure-preview"></a>將 Azure Stack VM 複寫至 Azure (預覽)

本文說明如何使用 [Azure Site Recovery 服務](https://docs.microsoft.com/azure/site-recovery/site-recovery-overview)設定將 Azure Stack VM 進行災害復原至 Azure。

Site Recovery 有助於商務持續性和災害復原 (BCDR) 策略的進行。 此服務可確保當停電時，無論是計劃性或突發性，您的 VM 工作負載皆維持可供使用。

- Site Recovery 可協調管理將 VM 複寫到 Azure 儲存體。
- 當主要網站停電時，您可以使用 Site Recovery 容錯移轉至 Azure。
- 容錯移轉時，系統會從儲存的 VM 資料中建立 Azure VM，使用者便可繼續存取在這些 Azure VM 上執行的工作負載。
- 若所有一切都恢復啟動並運作，您可以將 Azure VM 容錯回復到主要網站，並再次開始複寫至 Azure 儲存體。


> [!NOTE]
> 適用於 Azure Stack 的 Site Recovery 目前為預覽狀態。


在本文中，您將了解：

> [!div class="checklist"]
> * **步驟 1：準備 Azure stack VM 進行複寫**。 確認 VM 符合 Site Recovery 需求，並準備好安裝 Site Recovery Mobility 服務。 此服務已安裝於您要複寫的每部 VM 上。
> * **步驟 2：設定復原服務保存庫**。 為 Site Recovery 設定保存庫，並指定您要複寫的項目。 系統會在保存庫中設定與管理 Site Recovery 元件和動作。
> * **步驟 3：設定來源複寫環境**。 設定 Site Recovery 組態伺服器。 組態伺服器是一部 Azure Stack VM，可執行 Site Recovery 所需的所有元件。 組態伺服器設定完成後，您可在保存庫中註冊它。
> * **步驟 4：設定目標複寫環境**。 選取您的 Azure 帳戶、Azure 儲存體帳戶，以及您要使用的網路。 複寫期間，VM 資料會複製到 Azure 儲存體。 容錯移轉之後，Azure VM 會加入指定的網路。
> * **步驟 5：啟用複寫**。 設定複寫設定，並啟用 VM 複寫。 複寫啟用時，行動服務將會安裝在 VM 上。 Site Recovery 會執行 VM 的初始複寫，接著開始持續不間斷的複寫。
> * **步驟 6：執行災難復原演練**：複寫啟動並運作時，請透過執行演練確認容錯移轉能如預期般運作。 若要展開演練，請在 Site Recovery 中執行測試容錯移轉。 測試容錯移轉不會影響生產環境。

這些步驟完成時，您可以視需要執行完整容錯移轉至 Azure。

## <a name="architecture"></a>架構

![架構](./media/azure-stack-site-recovery/architecture.png)

**位置** | **元件** |**詳細資料**
--- | --- | ---
**組態伺服器** | 在單一 Azure Stack VM 上執行。 | 在每個訂用帳戶中，設定組態伺服器 VM。 此 VM 會執行下列 Site Recovery 元件：<br/><br/> - 組態伺服器：負責協調內部部署與 Azure 之間的通訊，以及管理資料複寫。 - 處理序伺服器：作為複寫閘道。 負責接收複寫資料，以快取、壓縮和加密進行最佳化，然後將複寫資料傳送至 Azure 儲存體。<br/><br/> 如果您要複寫的 VM 超過以下所述的限制，您可以設定個別的獨立處理序伺服器。 [深入了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-set-up-process-server-scale)。
**行動服務** | 安裝在您要複寫的每部 VM 上。 | 在本文的步驟中，我們準備了一個帳戶，讓行動服務會在複寫啟用時自動安裝在 VM 上。 如果您不想要自動安裝服務，還有一些其他方法可以使用。 [深入了解](https://docs.microsoft.com/azure/site-recovery/vmware-azure-install-mobility-service)。
**Azure** | 在 Azure 中，您需要復原服務保存庫、儲存體帳戶，以及虛擬網路。 |  複寫的資料會儲存在儲存體帳戶中。 當容錯移轉發生時，Azure VM 會新增至 Azure 網路。 


複寫會如下所示運作︰

1. 在保存庫中，您需指定複寫來源和目標、設定組態伺服器、建立複寫原則，以及啟用複寫。
2. 行動服務安裝在機器上 (如果您曾使用推送安裝)，機器遵循複寫原則開始複寫。
3. 伺服器資料初始複本複寫至 Azure 儲存體。
4. 初始複寫完成之後，就會開始將差異變更複寫到 Azure。 機器的追蹤變更會保存在 .hrl 檔案中。
5. 組態伺服器使用 Azure 協調複寫管理 (透過連接埠 HTTPS 443 輸出)。
6. 處理序伺服器接收來源機器所傳來的資料、將其最佳化並加密，再將它傳送至 Azure 儲存體 (透過連接埠 443 輸出)。
7. 複寫機器與組態伺服器通訊 (透過連接埠 HTTPS 443 輸入，以管理複寫)。 機器將複寫資料傳送到處理序伺服器 (透過連接埠 HTTPS 9443 輸入 - 可修改)。
8. 流量透過網際網路複寫到 Azure 儲存體的公用端點。 或者，您可以使用 Azure ExpressRoute 公用對等互連。 不支援從內部部署網站透過站台對站台 VPN 將流量複寫至 Azure。

## <a name="prerequisites"></a>必要條件

以下是設定此情節時所需的項目。

**需求** | **詳細資料**
--- | ---
**Azure 訂用帳戶** | 如果您沒有 Azure 訂用帳戶，請建立[免費帳戶](https://azure.microsoft.com/pricing/free-trial/)。
**Azure 帳戶權限** | 您使用的 Azure 帳戶需要以下權限：<br/><br/> - 建立復原服務保存庫<br/><br/> - 在您用於此情節的資源群組與虛擬網路中建立虛擬機器<br/><br/> - 寫入您指定的儲存體帳戶<br/><br/> 請注意：<br/><br/> - 如果您建立帳戶，您就是訂用帳戶的管理員，並可執行所有動作。<br/><br/> - 如果您使用現有訂用帳戶，而且您不是系統管理員，則需要與系統管理員合作，讓其指派擁有者或參與者權限給您。<br/><br/> - 如果您需要更細微的權限，請檢閱[此文章](https://docs.microsoft.com/azure/site-recovery/site-recovery-role-based-linked-access-control)。 
**Azure Stack VM** | 您的租用戶月租方案中需要有 Azure Stack VM，它將部署成 Site Recovery 組態伺服器。 


### <a name="prerequisites-for-the-configuration-server"></a>組態伺服器必要條件

[!INCLUDE [site-recovery-config-server-reqs-physical](../../includes/site-recovery-config-server-reqs-physical.md)]


 
## <a name="step-1-prepare-azure-stack-vms"></a>步驟 1：準備 Azure Stack VM

### <a name="verify-the-operating-system"></a>驗證作業系統

確定 VM 正在執行表格中整理出的作業系統。


**作業系統** | **詳細資料**
--- | ---
**64 位元 Windows** | Windows Server 2016、Windows Server 2012 R2、Windows Server 2012、Windows Server 2008 R2 (來自 SP1)
**CentOS** | 5.2 到 5.11、6.1 到 6.9、7.0 到 7.3
**Ubuntu** | 14.04 LTS 伺服器、16.04 LTS 伺服器。 檢閱[支援的核心](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#ubuntu-kernel-versions)

### <a name="prepare-for-mobility-service-installation"></a>準備行動服務安裝

每個您想要複寫的 VM 都必須已安裝行動服務。 為了讓處理序伺服器服務會在複寫啟用時自動安裝在 VM 上，請驗證 VM 設定。

#### <a name="windows-machines"></a>Windows 機器

- 您想要啟用複寫的 VM 與執行處理序伺服器的機器 (依預設，這是組態伺服器 VM) 之間必須有網路連線。
- 您在您啟用複寫的機器上，必須有具有管理員權利 (網域或本機) 的帳戶。
    - 您在設定 Site Recovery 時指定此帳戶。 之後當複寫啟用時，處理序伺服器會使用此帳戶安裝行動服務。
    - 此帳戶僅供 Site Recovery 用於推送通知，及更新行動服務。
    - 如果您未使用網域帳戶，您必須停用 VM 上的遠端使用者存取控制：
        - 在登錄中，在 HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System 下方建立 DWORD 值 **LocalAccountTokenFilterPolicy**。
        - 將值設定為 1。
        - 若要在命令提示字元進行，請輸入以下內容：**REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d 1**。
- 在所要複寫 VM 上的 Windows 防火牆中，允許「檔案及印表機共用」和 WMI。
    - 做法是，執行 [wf.msc] 開啟 [Windows 防火牆] 主控台。 以滑鼠右鍵按一下 [輸入規則]   >  [新增規則]。 選取 [預先定義]，然後選取清單中的 [檔案及印表機共用]。 完成精靈，選取允許連線 > [完成]。
    - 對於網域電腦，您可以使用 GPO 這樣做。

    
#### <a name="linux-machines"></a>Linux 機器

- 確定 Linux 電腦與處理序伺服器之間有網路連線。
- 在您要啟用複寫的機器上，您需要一個在來源 Linux 伺服器上為根使用者的帳戶：
    - 您在設定 Site Recovery 時指定此帳戶。 之後當複寫啟用時，處理序伺服器會使用此帳戶安裝行動服務。
    - 此帳戶僅供 Site Recovery 用於推送通知，及更新行動服務。
- 檢查來源 Linux 伺服器上的 /etc/hosts 檔案，該檔案具有將本機主機名稱對應到所有網路介面卡相關聯之 IP 位址的項目。
- 在您要複寫的電腦上安裝最新的 openssh、openssh-server 和 openssl 套件。
- 請確定安全殼層 (SSH) 已啟用且正在連接埠 22 上執行。
- 在 sshd_config 檔案中啟用 SFTP 子系統與密碼驗證：
    1. 若要這麼做，請以根使用者的身分登入。
    2. 在 /etc/ssh/sshd_config 檔案中，找到以 **PasswordAuthentication** 開頭這一行。 取消該行的註解並將值變更為 **yes**。
    3. 尋找以 **Subsystem** 為開頭的行並取消其註解。

        ![Linux 行動服務](./media/azure-stack-site-recovery/linux-mobility.png)

    4. 重新啟動 sshd 服務。


### <a name="note-the-vm-private-ip-address"></a>記下 VM 私人 IP 位址

找到每部所要複製機器的 IP 位址：

1. 在 Azure Stack 入口網站中，按一下 VM。
2. 在 [資源] 功能表中，按一下 [網路介面]。
3. 記下私人 IP 位址。

    ![私人 IP 位址](./media/azure-stack-site-recovery/private-ip.png)


## <a name="step-2-create-a-vault-and-select-a-replication-goal"></a>步驟 2：建立保存庫，然後選取複寫目標

1. 在 Azure 入口網站中，選取 [建立群組] > [監視 + 管理] > [備份和 Site Recovery]。
2. 在 [ **名稱**] 中，輸入保存庫的易記識別名稱。 
3. 在 [資源群組] 中，建立或選取資源群組。 我們使用的是 [contosoRG]。
4. 在 [位置] 中，輸入 Azure 區域。 我們使用**西歐**。
5. 若要從儀表板快速存取保存庫，請按一下 [釘選到儀表板] > [建立]。

   ![建立新保存庫](./media/azure-stack-site-recovery/new-vault-settings.png)

   新的保存庫會出現在 [儀表板] > [所有資源] 上，以及 [復原服務保存庫] 主頁面上。

### <a name="select-a-replication-goal"></a>選取複寫目標

1. 在 [復原服務保存庫] 中 > 指定保存庫名稱。 我們使用的是 [ContosoVMVault]。
2. 在 [使用者入門] 中，選取 Site Recovery。 然後選取 [準備基礎結構]。
3. 在 [保護目標] > [您的電腦位於何處] 中，選取 [內部部署]。
4. 在 [您要將電腦複寫到何處] 中，選取 [複製到 Azure]。
5. 在 [您的電腦虛擬化了嗎] 中，選取 [未虛擬化/其他]。 然後選取 [確定]。

    ![保護目標](./media/azure-stack-site-recovery/protection-goal.png)

## <a name="step-3-set-up-the-source-environment"></a>步驟 3︰設定來源環境

設定組態伺服器機器、在保存庫中註冊它，然後探索您要複寫的機器。

1. 按一下 [準備基礎結構]  >  [來源]。
2. 在 [準備來源] 中，按一下 [+組態伺服器]。

    ![設定來源](./media/azure-stack-site-recovery/plus-config-srv.png)

3. 在 [新增伺服器] 中，檢查 [設定伺服器] 是否出現在 [伺服器類型] 中。
5. 下載 Site Recovery 統一安裝的安裝檔案。
6. 下載保存庫註冊金鑰。 執行「整合安裝」時，您需要該註冊金鑰。 該金鑰在產生後會維持 5 天有效。

    ![設定來源](./media/azure-stack-site-recovery/set-source2.png)


### <a name="run-azure-site-recovery-unified-setup"></a>執行 Azure Site Recovery 統一安裝

若要安裝與登錄組態伺服器，對您用來作為組態伺服器的 VM 建立 RDP 連線，然後執行統一安裝。

開始之前，請先確定時鐘與 VM 上的[時間伺服器同步](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/get-started/windows-time-service/windows-time-service)。 如果時間比當地時間快 5 分鐘以上，安裝就會失敗。

現在安裝組態伺服器：

[!INCLUDE [site-recovery-add-configuration-server](../../includes/site-recovery-add-configuration-server.md)]

> [!NOTE]
> 組態伺服器也可從命令列安裝。 [深入了解](http://aka.ms/installconfigsrv)。

> 可能需要 15 分鐘以上，帳戶名稱才會出現在入口網站。 若要立即更新，請選取 [設定伺服器] > 伺服器名稱 > [重新整理伺服器]。

## <a name="step-4-set-up-the-target-environment"></a>步驟 4︰設定目標環境

選取並確認目標資源。

1. 在 [準備基礎結構] > [目標] 中，選取您要使用的 Azure 訂用帳戶。
2. 指定目標部署模型。
3. Site Recovery 會檢查您是否有一或多個相容的 Azure 儲存體帳戶和網路。 如果找不到它們，您必須建立至少一個儲存體帳戶和虛擬網路，才能完成精靈。


## <a name="step-5-enable-replication"></a>步驟 5：啟用複寫

### <a name="create-a-replication-policy"></a>建立複寫原則

1. 按一下 [準備基礎結構]  >  [複寫設定]。
2. 在 [建立複寫原則]中，指定原則名稱。
3. 在 [RPO 閾值] 中，指定復原點目標 (RPO) 限制。
    - 系統會根據設定的時間建立複寫資料的復原點。
    - 此設定不會影響複寫 (這是連續的)。 如果到達閾值限制，則只會發出警示，不會建立復原點。
4. 在 [復原點保留] 中，指定每個復原點會保留多久的時間。 複寫的 VM 可以還原至所指定時間範圍內的任何時間點。
5. 在 [應用程式一致快照集頻率] 中，指定建立應用程式一致快照集的頻率。

    - 應用程式一致快照集是 VM 內應用程式資料的時間點快照集。
    - 磁碟區陰影複製服務 (VSS) 可確保在建立快照集時，VM 上的應用程式處於一致狀態。
6. 選取 [確定] 以建立原則。


### <a name="confirm-deployment-planning"></a>確認部署規劃

您可以立即略過此步驟。 在 [部署規劃] 下拉式清單中，按一下 [是，已完成]。



### <a name="enable-replication"></a>啟用複寫

請確定您已完成[步驟 1：準備機器](#step-1-prepare-azure-stack-vms)中的所有工作。 接著啟用複寫，如下所述：

1. 選取 [複寫應用程式] > [來源]。
2. 在 [來源] 中，選取設定伺服器。
3. 在 [機器類型] 中，選取 [實體機器]。
4. 選取處理序伺服器 (組態伺服器)。 然後按一下 [確定] 。
5. 在 [目標] 中，選取您想要在容錯移轉後，在其中建立 VM 的訂用帳戶和資源群組。 選擇您要用於容錯移轉 VM 的部署模型。
6. 選取您要儲存複寫資料的 Azure 儲存體帳戶。
7. 選取 Azure VM 在容錯移轉後所要連線的 Azure 網路和子網路。
8. 選取 [立即設定選取的機器]，將網路設定套用至您選取要進行保護的所有機器。 如果您要分別為每部機器選取 Azure 網路，請選取 [稍後設定]。
9. 在 [實體機器] 中，按一下 [+實體機器]。 指定每部機器的 IP 位址名稱，以及您要複寫的作業系統。

    - 使用機器的內部 IP 位址。
    - 如果您指定公用 IP 位址，則複寫可能無法如預期般運作。

10. 在 [屬性]  >  [設定屬性] 中，選取處理序伺服器將用來在電腦上自動安裝行動服務的帳戶。
11. 在 [複寫設定]  >  [設定複寫設定] 中，確認選取正確的複寫原則。
12. 按一下 [啟用複寫] 。
13. 在 [設定]  > [作業] > [Site Recovery 作業] 中，追蹤 [啟用保護] 作業的進度。 執行 [完成保護] 作業之後，機器即準備好進行容錯移轉。

> [!NOTE]
> Site Recovery 會在 VM 已啟用複寫時安裝行動服務。

> 可能需要 15 分鐘或更久的時間，變更才會生效並顯示在入口網站中。

> 若要監視您新增的 VM，請在 [設定伺服器] >  [上次連絡時間] 中查看上次探索 VM 的時間。 若要新增 VM 而不等候已排定的探索，請醒目提示設定伺服器 (但不要選取)，然後選取 [重新整理]。


## <a name="step-6-run-a-disaster-recovery-drill"></a>步驟 6：執行災害復原演練

您對 Azure 執行測試容錯移轉，確定所有一切均如預期運作。 此容錯移轉不會影響生產環境。

### <a name="verify-machine-properties"></a>驗證機器屬性

執行測試容錯移轉之前，請先驗證機器屬性，並確定機器符合 [Azure 需求](https://docs.microsoft.com/azure/site-recovery/vmware-physical-azure-support-matrix#azure-vm-requirements)。 您可以檢視與修改屬性，如下所示：

1. 在 [受保護的項目] 中，按一下 [複寫的項目] > VM。
2. 在 [複寫的項目] 窗格中，將會呈現 VM 資訊、健康情況狀態，以及最新可用復原點的摘要。 如需檢視詳細資訊，請按一下 [屬性]。
3. 在 [計算與網路] 中，視需要修改設定。

    - 您可以修改 Azure VM 名稱、資源群組、目標大小、[可用性設定組](../virtual-machines/windows/tutorial-availability-sets.md)及受控磁碟設定。
    - 您也可以檢視和修改網路設定。 這些包括容錯移轉後 Azure VM 加入的網路/子網路，以及要指派給 VM 的 IP 位址。
1. 在 [磁碟] 中，檢視 VM 上作業系統與資料磁碟的相關資訊。
   

### <a name="run-a-test-failover"></a>執行測試容錯移轉

當您執行測試容錯移轉時，會發生下列情況：

1. 程式會進行必要條件檢查，以確保所有容錯移轉所需之條件都已準備就緒。
2. 容錯移轉使用指定的復原點處理資料：
    - **最近處理**：機器容錯移轉到 Site Recovery 所處理的最新復原點。 隨即顯示時間戳記。 使用此選項時，無須花費時間處理資料，因此它會提供低 RTO (復原時間目標)。
    - **最近的應用程式一致**：機器容錯移轉到最新的應用程式一致復原點。
    - **自訂**。 選取用於容錯移轉的復原點。

3. 系統使用處理後的資料建立 Azure VM。
4. 測試容錯移轉會自動清除演練期間建立的 Azure VM。

對 VM 執行測試容錯移轉，如下所示：

1. 在 [設定] > [複寫的項目] 中，按一下 [VM] > [+測試容錯移轉]。
2. 針對此逐步解說，我們將選擇使用 [最近處理] 復原點。 
3. 在 [測試容錯移轉] 中，選取目標 Azure 網路。
4. 按一下 [確定]  即可開始容錯移轉。
5. 按一下 VM 開啟其屬性以追蹤進度。 或者在 [保存庫名稱]  >  [設定]  >  [工作]  > [Site Recovery 工作] 中，按一下 [測試容錯移轉] 工作。
6. 容錯移轉完成之後，複本 Azure VM 會出現在 Azure 入口網站> [虛擬機器] 中。 檢查 VM 的大小是否合適、是否連線到正確的網路，而且正常執行中。
7. 您現在應該能夠連線到 Azure 中複寫的 VM。 [深入了解](https://docs.microsoft.com/azure/site-recovery/site-recovery-test-failover-to-azure#prepare-to-connect-to-azure-vms-after-failover)。
8. 若要刪除測試容錯移轉期間建立的 Azure VM，請按一下 VM 上的 [清除測試容錯移轉]。 在 [記事] 中，儲存關於測試容錯移轉的任何觀察。

## <a name="fail-over-and-fail-back"></a>容錯移轉和容錯回復

在您已設定複寫，並執行演練確定一切正常運作後，您可以視需要將機器容錯移轉至 Azure。

在您執行容錯移轉之前，如果您要在容錯移轉後與 Azure 中的機器連線，開始連線前請先[準備連線](https://docs.microsoft.com/azure/site-recovery/site-recovery-test-failover-to-azure#prepare-to-connect-to-azure-vms-after-failover)。

接著執行容錯移轉，如下所示：


1. 在 [設定] > [複寫的項目] 中，按一下機器 > [容錯移轉]。
2. 選取您要使用的復原點。
3. 在 [測試容錯移轉] 中，選取目標 Azure 網路。
4. 選取 [Shut down machine before beginning failover] \(先將機器關機再開始容錯移轉)。 透過此設定，Site Recovery 在開始容錯移轉前會先嘗試將來源機器關機。 然而即使關機失敗，仍會繼續容錯移轉。 
5. 按一下 [確定]  即可開始容錯移轉。 您可以 [作業] 頁面上追蹤容錯移轉進度。
6. 容錯移轉完成之後，複本 Azure VM 會出現在 Azure 入口網站> [虛擬機器] 中。 如果您準備在容錯移轉後連線，請檢查 VM 的大小是否合適、是否連線到正確的網路，而且正常執行中。
7. 驗證 VM 後，按一下 [認可] 完成容錯移轉。 這會刪除所有可用的復原點。

> [!WARNING]
> 不要取消正在進行的容錯移轉：在啟動容錯移轉之前，已停止 VM 複寫。 如果您取消正在進行的容錯移轉，容錯移轉會停止，但 VM 不會再次複寫。


### <a name="fail-back-to-azure-stack"></a>容錯回復至 Azure Stack

當主要網站再次啟動並執行時，您可以從 Azure 容錯回復至 Azure Stack。 做法是，您必須下載 Azure VM VHD，然後上傳到 Azure Stack。

1. 將 Azure VM 關機，讓您可以下載 VHD。 
2. 若要開始下載 VHD，請安裝 [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/)。
3. 瀏覽至 Azure 入口網站中的 VM (使用 VM 名稱)。
4. 在 [磁碟] 中，按一下磁碟名稱，然後收集設定。

    - 例如，用於我們測試： https://502055westcentralus.blob.core.windows.net/wahv9b8d2ceb284fb59287/copied-3676553984.vhd 中的 VHD URI 可加以細分，以取得用於下載 VHD 的輸入參數。
        - 儲存體帳戶：502055westcentralus
        - 容器：wahv9b8d2ceb284fb59287
        - VHD 名稱： copied-3676553984.vhd

5. 現在，使用 Azure 儲存體總管下載 VHD。
6. 使用[這些步驟](https://docs.microsoft.com/azure/azure-stack/user/azure-stack-manage-vm-disks#use-powershell-to-add-multiple-unmanaged-disks-to-a-vm)將 VHD 上傳到 Azure Stack。
7. 在現有的 VM 或新 VM 中，附加已上傳的 VHD。
8. 檢查確認作業系統磁碟正確，然後啟動 VM。


容錯回復在這個階段完成。


## <a name="conclusion"></a>結論

在本文中，我們將 Azure Stack VM 複寫到 Azure。 複寫到位後，我們執行災害復原演練，確定能如預期般地容錯移轉至 Azure。 本文也包含執行完整容錯移轉至 Azure 及容錯回復至 Azure Stack 的步驟。

## <a name="next-steps"></a>後續步驟

容錯回復後，您可以重新保護 VM，並開始將它再次複寫到 Azure，做法是重複本文中的步驟。

