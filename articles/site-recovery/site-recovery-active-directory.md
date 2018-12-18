---
title: 以 Azure Site Recovery 保護 Active Directory 和 DNS | Microsoft Docs
description: 本文說明如何使用 Azure Site Recovery 實作 Active directory 的災害復原解決方案。
services: site-recovery
documentationcenter: ''
author: mayanknayar
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 07/19/2018
ms.author: manayar
ms.openlocfilehash: 28ea6c58eed110cfb57f3feaa6b828289bd20e64
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44054541"
---
# <a name="use-azure-site-recovery-to-protect-active-directory-and-dns"></a>使用 Azure Site Recovery 保護 Active Directory 和 DNS

企業應用程式 (例如 SharePoint、Dynamics AX 和 SAP) 須倚賴 Active Directory 和 DNS 基礎結構才能正確運作。 當您設定應用程式的災害復原時，您通常必須先復原 Active Directory 和 DNS，再復原其他應用程式元件，以確保應用程式功能可正常運作。

您可以使用 [Site Recovery](site-recovery-overview.md) 來建立 Active Directory 的災害復原計劃。 發生中斷的狀況時，您可以起始容錯移轉。 您可以讓 Active Directory 在短短幾分鐘內啟動並執行。 如果您已在主要站台 (例如 SharePoint 和 SAP) 中為多個應用程式部署 Active Directory，您可能會想要容錯移轉至完整的網站。 您可以先使用 Site Recovery 進行 Active Directory 的容錯移轉。 接著，您可以使用應用程式專屬復原計劃進行其他應用程式的容錯移轉。

本文說明如何建立 Active Directory 的災害復原解決方案。 其中包括先決條件，以及容錯移轉指示。 開始之前，您應該先熟悉 Active Directory 與 Site Recovery。

## <a name="prerequisites"></a>必要條件

* 如果您要複寫至 Azure，請[準備 Azure 資源](tutorial-prepare-azure.md)，包括訂用帳戶、Azure 虛擬網路、儲存體帳戶和復原服務保存庫。
* 檢閱所有元件的[支援需求](site-recovery-support-matrix-to-azure.md)。

## <a name="replicate-the-domain-controller"></a>複寫網域控制站

- 您必須在至少一個裝載網域控制站和 DNS 的 VM 上設定 [Site Recovery 複寫](#enable-protection-using-site-recovery)。
- 如果您的環境中有[多個網域控制站](#environment-with-multiple-domain-controllers)，則您也必須在目標站台上設定[額外的網域控制站](#protect-active-directory-with-active-directory-replication)。 額外的網域控制站可位於 Azure 中，或位於次要內部部署資料中心。
- 如果您有只有少數幾個應用程式和一個網域控制站，您可以將整個站台一併容錯移轉。 在此情況下，建議您使用 Site Recovery 將網域控制站複寫至目標站台 (位於 Azure 或次要內部部署資料中心)。 相同的複寫網域控制站或 DNS 虛擬機器也可用於[測試容錯移轉](#test-failover-considerations)。
- - 如果您的環境中有許多應用程式和多個網域控制站，或您打算一次容錯移轉數個應用程式，建議您除了使用 Site Recovery 複寫網域控制站虛擬機器之外，也應在目標站台 (位於 Azure 或次要內部部署資料中心) 上設定[額外的網域控制站](#protect-active-directory-with-active-directory-replication)。 對於[測試容錯移轉](#test-failover-considerations)，您可以使用 Site Recovery 所複寫的網域控制站。 對於容錯移轉，您可以在目標站台上使用額外的網域控制站。

## <a name="enable-protection-with-site-recovery"></a>使用 Site Recovery 啟用保護

您可以使用 Site Recovery 來保護裝載網域控制站或 DNS 的虛擬機器。

### <a name="protect-the-vm"></a>保護 VM
使用 Site Recovery 複寫的網域控制站會用於[測試容錯移轉](#test-failover-considerations)。 請確定它符合下列需求︰

1. 網域控制站是通用類別目錄伺服器。
2. 網域控制站應為測試容錯移轉期間所需角色的 FSMO 角色擁有者。 否則在容錯移轉之後，將必須[收回](http://aka.ms/ad_seize_fsmo)這些角色。

### <a name="configure-vm-network-settings"></a>進行 VM 網路設定
對於裝載網域控制站或 DNS 的虛擬機器，使用 Site Recovery，在複寫虛擬機器的 [計算和網路] 設定下進行網路設定。 這可確保虛擬機器在容錯移轉之後會連結至正確的網路。

## <a name="protect-active-directory"></a>保護 Active Directory

### <a name="site-to-site-protection"></a>站對站保護
請在次要網站上建立網域控制站。 當您將伺服器提升至網域控制站角色時，請指定主要網站中所使用之相同網域的名稱。 您可以使用 **Active Directory 網站和服務** 嵌入式管理單元來設定網站要新增至的網站連結物件上的設定。 藉由設定網站連結，您可以控制兩個以上的網站之間執行複寫的時機和頻率。 如需詳細資訊，請參閱[排程網站之間的複寫](https://technet.microsoft.com/library/cc731862.aspx)。

### <a name="site-to-azure-protection"></a>站對 Azure 保護
首先，在 Azure 虛擬網路中建立網域控制站。 當您將伺服器提升至網域控制站角色時，請指定主要網站中所使用的相同網域名稱。

然後，重新設定虛擬網路的 DNS 伺服器以在 Azure 中使用此 DNS 伺服器。

![Azure 網路](./media/site-recovery-active-directory/azure-network.png)

### <a name="azure-to-azure-protection"></a>Azure 對 Azure 保護
首先，在 Azure 虛擬網路中建立網域控制站。 當您將伺服器提升至網域控制站角色時，請指定主要網站中所使用的相同網域名稱。

然後，重新設定虛擬網路的 DNS 伺服器以在 Azure 中使用此 DNS 伺服器。

## <a name="test-failover-considerations"></a>測試容錯移轉考量
為了避免影響到生產工作負載，測試容錯移轉會在與生產網路隔離的網路中執行。

大部分的應用程式都需要網域控制站或 DNS 伺服器才能運作。 因此，在應用程式容錯移轉之前，您必須在要用於測試容錯移轉的隔離網路中建立網域控制站。 為此，最簡單的方法是使用 Site Recovery 來複寫裝載網域控制站或 DNS 的虛擬機器。 然後，在執行應用程式復原計劃的測試容錯移轉前，您應先執行網域控制站虛擬機器的測試容錯移轉。 以下是做法：

1. 使用 Site Recovery [複寫](vmware-azure-tutorial.md)裝載網域控制站或 DNS 的虛擬機器。
2. 建立隔離的網路。 在 Azure 中建立的任何虛擬網路，依預設都會與其他網路隔離。 建議您對此網路使用您的生產網路所使用的相同 IP 範圍。 請勿在此網路上啟用網站對網站連線能力。
3. 提供隔離網路中 DNS IP 位址。 請使用您想要讓 DNS 虛擬機器取得的 IP 位址。 如果您要複寫至 Azure，請提供用於容錯移轉之虛擬機器的 IP 位址。 若要輸入 IP 位址，請在複寫虛擬機器的 [計算和網路] 設定中選取 [目標 IP] 設定。

    ![Azure 測試網路](./media/site-recovery-active-directory/azure-test-network.png)

    > [!TIP]
    > Site Recovery 會嘗試在名稱相同的子網路中建立測試虛擬機器，並使用虛擬機器的 [計算與網路] 設定中提供的 IP 位址。 如果針對測試容錯移轉提供的 Azure 虛擬網路中沒有名稱相同的子網路，則會在依字母順序的第一個子網路中建立測試虛擬機器。
    >
    > 如果目標 IP 位址是所選子網路的一部分，則 Site Recovery 會使用目標 IP 位址嘗試建立測試容錯移轉虛擬機器。 如果目標 IP 不是所選子網路的一部分，則會使用所選子網路中的下一個可用 IP 建立測試容錯移轉虛擬機器。
    >
    >

### <a name="test-failover-to-a-secondary-site"></a>對次要網站執行測試容錯移轉

1. 如果您要複寫至其他內部部署網站，且您是使用 DHCP，請[針對測試容錯移轉設定 DNS 和 DHCP](hyper-v-vmm-test-failover.md#prepare-dhcp)。
2. 對隔離網路中執行的網域控制站虛擬機器進行測試容錯移轉。 請使用網域控制站虛擬機器的最新可用*應用程式一致*復原點來執行測試容錯移轉。
3. 針對包含應用程式執行所在之虛擬機器的復原計劃執行測試容錯移轉。
4. 測試完成之後，請在網域控制站虛擬機器上*清除測試容錯移轉*。 此步驟會刪除針對測試容錯移轉所建立的網域控制站。


### <a name="remove-references-to-other-domain-controllers"></a>移除對其他網域控制站的參考
在起始測試容錯移轉時，請勿在測試網路中納入所有網域控制站。 若要移除對您生產環境中其他網域控制站的參考，您可能必須[收回 FSMO Active Directory 角色](http://aka.ms/ad_seize_fsmo)，並針對遺漏的網域控制站執行[中繼資料清理](https://technet.microsoft.com/library/cc816907.aspx)。


### <a name="issues-caused-by-virtualization-safeguards"></a>因虛擬化保護措施而產生的問題

> [!IMPORTANT]
> 本節所述的部分組態不是標準或預設的網域控制站組態。 如果您不想對生產網域控制站進行這些變更，您可以建立專門供 Site Recovery 用於測試容錯移轉的網域控制站。 您只需對該網域控制站進行這些變更即可。  
>
>

從 Windows Server 2012 開始，[Active Directory Domain Services (AD DS) 已內建額外保護措施](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100)。 如果基礎 hypervisor 平台支援 **VM-GenerationID**，這些保護措施就能協助保護虛擬化網域控制站，避免進行 USN 復原。 Azure 支援 **VM-GenerationID**。 因此，在 Azure 虛擬機器上執行 Windows Server 2012 或更新版本的網域控制站，具有額外的保護措施。


當 **VM-GenerationID** 重設時，AD DS 資料庫的 **InvocationID** 值也會重設。 此外，RID 集區會被捨棄，且 SYSVOL 會標示為非授權。 如需詳細資訊，請參閱 [Active Directory Domain Services 虛擬化的簡介](https://technet.microsoft.com/windows-server-docs/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100)和[安全地虛擬化 DFSR](https://blogs.technet.microsoft.com/filecab/2013/04/05/safely-virtualizing-dfsr/)。

容錯移轉至 Azure 可能會導致 **VM-GenerationID** 重設。 重設 **VM-GenerationID** 後，在網域控制站虛擬機器於 Azure 中啟動時將會執行額外的保護措施。 這可能會導致使用者在登入網域控制站虛擬機器時產生*明顯的延遲*。

此網域控制站只會用於測試容錯移轉，因此不需要虛擬化保護措施。 若要確保網域控制站虛擬機器的 **VM-GenerationID** 值不會變更，您可以將內部部署網域控制站中的下列 DWORD 值變更為 **4**：


`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\gencounter\Start`


#### <a name="symptoms-of-virtualization-safeguards"></a>虛擬化保護措施的徵兆

如果在測試容錯移轉之後觸發了虛擬化保護措施，您可能會看到下列一或多個徵兆︰  

* **GenerationID** 值變更。

    ![世代識別碼變更](./media/site-recovery-active-directory/Event2170.png)

* **InvocationID** 值變更。

    ![叫用識別碼變更](./media/site-recovery-active-directory/Event1109.png)

* 無法使用 SYSVOL 與 NETLOGON 共用。

    ![SYSVOL 共用](./media/site-recovery-active-directory/sysvolshare.png)

    ![NtFrs SYSVOL](./media/site-recovery-active-directory/Event13565.png)

* DFSR 資料庫會被刪除。

    ![DFSR 資料庫會被刪除](./media/site-recovery-active-directory/Event2208.png)


### <a name="troubleshoot-domain-controller-issues-during-test-failover"></a>在測試容錯移轉期間，對網域控制站問題進行疑難排解

> [!IMPORTANT]
> 本節所述的部分組態不是標準或預設的網域控制站組態。 如果您不想對生產網域控制站進行這些變更，您可以建立 Site Recovery 測試容錯移轉專用的網域控制站。 您只需對此專用的網域控制站進行變更即可。  
>
>

1. 在命令提示字元上執行下列命令，以檢查 SYSVOL 與 NETLOGON 資料夾是否共用︰

    `NET SHARE`

2. 在命令提示字元上執行下列命令，以確保網域控制站正常運作：

    `dcdiag /v > dcdiag.txt`

3. 在輸出記錄中尋找下列文字。 若有此文字，即可確認網域控制站運作正常。

    * "passed test Connectivity"
    * "passed test Advertising"
    * "passed test MachineAccount"

如果符合上述條件，表示網域控制站應該是正常運作。 若非如此，請完成下列步驟：

1. 執行網域控制站的權威復原。 記住下列資訊：
    * 雖然我們不建議 [FRS 複寫](https://blogs.technet.microsoft.com/filecab/2014/06/25/the-end-is-nigh-for-frs/)，但如果您使用 FRS 複寫，請依照權威還原的步驟操作。 其程序請見[使用 BurFlags 登錄機碼重新初始化檔案複寫服務](https://support.microsoft.com/kb/290762)。

        如需 BurFlags 的詳細資訊，請參閱部落格文章 [D2 和 D4：有何功用？](https://blogs.technet.microsoft.com/janelewis/2006/09/18/d2-and-d4-what-is-it-for/)。
    * 如果您使用 DFSR 複寫，請完成權威還原的步驟。 其程序請見[對 DFSR 複寫的 SYSVOL (例如 FRS 的 "D4/D2") 強制執行權威和非權威同步](https://support.microsoft.com/kb/2218556)。

        您也可以使用 PowerShell 函式。 如需詳細資訊，請參閱 [DFSR-SYSVOL 授權/非權威還原 PowerShell 函式](https://blogs.technet.microsoft.com/thbouche/2013/08/28/dfsr-sysvol-authoritative-non-authoritative-restore-powershell-functions/)。

2. 將內部部署網域控制站中的下列登錄機碼設為 **0**，以略過初始同步需求。 如果此 DWORD 不存在，您可以在 **Parameters** 節點下加以建立。

    `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Parameters\Repl Perform Initial Synchronizations`

    如需詳細資訊，請參閱[疑難排解 DNS 事件識別碼 4013：DNS 伺服器無法載入 AD 整合的 DNS 區域](https://support.microsoft.com/kb/2001093)。

3. 停用讓通用類別目錄伺服器可用來驗證使用者登入的需求。 若要這麼做，請在內部部署網域控制站中，將下列登錄機碼設為 **1**。 如果此 DWORD 不存在，您可以在 **Lsa** 節點下加以建立。

    `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Lsa\IgnoreGCFailures`

    如需詳細資訊，請參閱[停用讓通用類別目錄伺服器可用來驗證使用者登入的需求](http://support.microsoft.com/kb/241789)。

### <a name="dns-and-domain-controller-on-different-machines"></a>不同電腦上的 DNS 和網域控制站

如果您在相同的 VM 上執行網域控制站和 DNS，您可以略過此程序。


如果 DNS 與網域控制站不在相同的 VM 上，您必須為測試容錯移轉建立 DNS VM。 您可以使用全新的 DNS 伺服器，並建立所有的必要區域。 例如，如果 Active Directory 網域是 contoso.com，您可以使用 contoso.com 的名稱來建立 DNS 區域。 DNS 中對應至 Active Directory 的項目必須更新，如下所示：

1. 確定在復原計劃中的任何其他虛擬機器啟動之前，下列設定均已完成：
   * 區域必須在樹系根名稱之後命名。
   * 區域必須是檔案備份的。
   * 區域必須能夠進行安全和非安全更新。
   * 裝載網域控制站之虛擬機器的解析程式應指向 DNS 虛擬機器的 IP 位址。

2. 在裝載網域控制站的 VM 上執行下列命令：

    `nltest /dsregdns`

3. 執行下列命令，以在 DNS 伺服器上新增區域、允許非安全更新，以及將適用於該區域的項目新增至 DNS：

    `dnscmd /zoneadd contoso.com  /Primary`

    `dnscmd /recordadd contoso.com  contoso.com. SOA %computername%.contoso.com. hostmaster. 1 15 10 1 1`

    `dnscmd /recordadd contoso.com %computername%  A <IP_OF_DNS_VM>`

    `dnscmd /config contoso.com /allowupdate 1`

## <a name="next-steps"></a>後續步驟
深入了解[如何以 Azure Site Recovery 保護企業工作負載](site-recovery-workload.md)。
