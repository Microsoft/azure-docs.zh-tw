---
title: 在 SAP HANA on Azure (大型執行個體) 上進行 HANA 備份和還原 | Microsoft Docs
description: 如何在 SAP HANA on Azure (大型執行個體) 上執行 HANA 備份和還原
services: virtual-machines-linux
documentationcenter: ''
author: saghorpa
manager: jeconnoc
editor: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/28/2018
ms.author: saghorpa
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 2a72fade57b070ac2ac1aea28cbec92700c3797f
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47452542"
---
# <a name="backup-and-restore"></a>備份與還原

>[!IMPORTANT]
>本說明文件不可用 SAP HANA 管理說明文件或 SAP 附註取代。 本文預期讀者對 SAP HANA 管理和操作有深入的了解和專業知識，尤其是關於備份、還原、高可用性和災害復原的主題。 在這份說明文件中，會顯示 SAP HANA Studio 的螢幕擷取畫面。 SAP 管理工具和工具本身的畫面內容、結構和性質，可能會因 SAP HANA 的版本而異。

請務必使用您的 HANA 版本和版次，在您的環境中執行步驟和程序。 為了能讓大眾更全面地了解，這份說明文件中所述的某些程序已簡化，但並非用做最終操作手冊的詳細步驟。 如果您想要為組態建立操作手冊，則需要測試及執行程序，並記錄與特定組態相關的處理程序。 

操作資料庫的其中一個最重要層面，就是保護資料庫免於遭受災難事件。 這些事件可能由任何原因造成，從天然災害到簡單的使用者錯誤都有可能。

備份資料庫時，若提供還原到任何時間點 (例如在某人刪除重要資料之前) 的功能，便可讓還原到儘可能接近中斷之前的狀態。

必須執行兩種類型的備份，才能獲得最佳結果：

- 資料庫備份：完整、增量或差異備份
- 交易記錄備份

除了在應用程式層級執行的完整資料庫備份之外，您可以透過儲存體快照集執行備份。 儲存體快照集並不會取代交易記錄備份。 對於「將資料庫還原到特定時間點」或是「從已經認可的交易清空記錄」來說，交易記錄備份仍然很重要。 不過，儲存體快照集可加快復原速度，因為它們可以快速地提供資料庫的向前復原映像。 

SAP HANA on Azure (大型執行個體) 提供兩個備份和還原選項：

- 自己動手做 (DIY)。 在計算以確保磁碟空間足夠之後，請使用下列其中一個磁碟備份方法來執行完整資料庫和記錄備份。 您可以直接備份到連結至 HANA 大型執行個體單位的磁碟區，或備份到 Azure 虛擬機器 (VM) 中設定的網路檔案共用 (NFS)。 在後面一種情況下，客戶會在 Azure 中設定 Linux VM、將 Azure 儲存體連結至 VM，並透過該 VM 中所設定的 NFS 伺服器來共用儲存體。 如果對直接連結至 HANA 大型執行個體單位的磁碟區執行備份，您必須將備份複製到 Azure 儲存體帳戶 (在設定 Azure VM 來匯出以 Azure 儲存體為基礎的 NFS 共用之後)。 您也可以使用 Azure 備份保存庫或 Azure 冷儲存體。 

   另一個選項是在備份複製到 Azure 儲存體帳戶之後，使用第三方資料保護工具來儲存備份。 針對因合規性和稽核目的而需要較長儲存期的資料，DIY 備份選項可能也有其必要性。 不論是哪種情況，備份都會複製到透過 VM 與 Azure 儲存體所表示的 NFS 共用中。

- 基礎結構備份和還原功能。 您也可以使用 SAP HANA on Azure (大型執行個體) 的底層基礎結構所提供的備份和還原功能。 此選項可滿足備份和快速還原的需要。 本節的其餘部分討論 HANA (大型執行個體) 隨附的備份和還原功能。 本節也涵蓋備份和還原對於 HANA 大型執行個體所提供之災害復原功能的關聯性。

>   [!NOTE]
>   HANA 大型執行個體的底層基礎結構所使用的快照集技術相依於 SAP HANA 快照集。 此時，SAP HANA 快照集還無法搭配 SAP HANA 多租用戶資料庫容器的多個租用戶一起使用。 如果只部署一個租用戶，則可使用 SAP HANA 快照集並利用此方法。

## <a name="using-storage-snapshots-of-sap-hana-on-azure-large-instances"></a>使用 SAP HANA on Azure (大型執行個體) 的儲存體快照

SAP HANA on Azure (大型執行個體) 底下的儲存體基礎結構支援磁碟區的儲存體快照集。 不論是備份還是還原磁碟區都受到支援，但有下列考量事項：

- 系統會經常建立存放磁碟區快照，而不是進行完整的資料庫備份。
- 針對 /hana/data 和 /hana/shared (包括 /usr/sap) 磁碟區來觸發快照集時，快照集技術會先起始 SAP HANA 快照集，再執行儲存體快照集。 在儲存體快照集復原之後，此 SAP HANA 快照集是最後記錄還原的設定點。 您需要作用中的 HANA 執行個體，HANA 快照集才會成功。  在 HSR 案例中，無法執行 HANA 快照集的目前次要節點上不支援儲存體快照集。
- 在順利執行儲存體快照集後，系統會刪除 SAP HANA 快照集。
- 系統會經常建立交易記錄備份並儲存在 /hana/logbackups 磁碟區或 Azure 中。 您可以觸發含有交易記錄備份的 /hana/logbackups 磁碟區，對它個別擷取快照集。 在此情況下，您不需要執行 HANA 快照集。
- 如果您必須將資料庫還原到特定時間點，請要求「Microsoft Azure 支援服務」(適用於生產環境中斷) 或「SAP HANA on Azure 服務管理」將資料庫還原到特定的儲存體快照集。 例如，將沙箱系統還原到其原始狀態的計劃性還原。
- 儲存體快照集所包含的 SAP HANA 快照集是一個位移點，用來套用在儲存體快照集建立後已執行和儲存的交易記錄備份。
- 建立這些交易記錄備份的目的是要將資料庫還原回特定的時間點。

您可以執行以三種磁碟區類別為目標的儲存體快照集：

- 針對 /hana/data 和 /hana/shared (包括 /usr/sap) 的合併快照集。 此快照集需要建立 SAP HANA 快照集，作為儲存體快照集的準備工作。 從儲存體觀點來看，SAP HANA 快照集會確保資料庫處於一致的狀態。 對於還原程序而言，這是一個設定重點。
- 針對 /hana/logbackups 的個別快照集。
- 作業系統分割區。

從 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) 取得最新快照集指令碼和文件。 當您從 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) 下載快照集指令碼套件時，也會取得包含在指令碼套件中的指令碼 PDF 文件。 每個指令碼套件都有它自己的 PDF 文件。

## <a name="storage-snapshot-considerations"></a>儲存體快照考量事項

>[!NOTE]
>儲存體快照集會取用已配置給 HANA 大型執行個體單位的儲存空間。 您必須考慮以下有關排定儲存體快照集和保留多少儲存體快照集等方面的問題。 

SAP HANA on Azure (大型執行個體) 儲存體快照集的獨特技巧包括：

- 特定的儲存體快照集 (在建立快照集的時間點) 只會取用少量儲存體。
- 隨著在存放磁碟區上資料內容發生變更及 SAP HANA 資料檔中的內容發生變更，快照便需要儲存原始區塊內容以及資料變更。
- 因此，儲存體快照的大小會增加。 快照存在的時間越長，儲存體快照就會變得越大。
- 在儲存體快照集的生命週期中對 SAP HANA 資料庫磁碟區進行的變更越多，儲存體快照集所耗用的空間就越大。

SAP HANA on Azure (大型執行個體) 隨附固定的磁碟區大小供 SAP HANA 資料和記錄磁碟區使用。 執行這些磁碟區的快照集會耗用磁碟區空間。 您必須決定何時要排定儲存快照集。 您也必須監視存放磁碟區的空間耗用量，以及管理您儲存的快照集數目。 您可以在匯入大量資料或對 HANA 資料庫執行其他重大變更時，停用儲存體快照集。 


下列各節提供執行這些快照集的資訊，包括一般建議事項：

- 雖然硬體的每一磁碟區都可支援 255 個快照，但建議您讓快照數目保持遠低於這個數字。 建議為 250 或更少。
- 執行儲存體快照集之前，請監視並追蹤可用空間。
- 根據可用空間來降低儲存體快照的數目。 您可以降低保留的快照集數目，也可以擴充磁碟區。 您可以訂購額外的儲存空間，以 1 TB 為單位。
- 在「使用 SAP 平台移轉工具 (R3load) 將資料移到 SAP HANA」或是「從備份還原 SAP HANA 資料庫」等活動進行期間，請在 /hana/data 磁碟區上停用儲存體快照集。 
- 在進行較大規模的 SAP HANA 資料表重組期間，應儘可能避免執行儲存體快照。
- 儲存體快照集也是利用 SAP HANA on Azure (大型執行個體) 之災害復原功能的必要條件。

## <a name="prerequisites-for-using-self-service-storage-snapshots"></a>使用自助服務儲存體快照集的必要條件

為了確保快照集指令碼執行成功，請確定 HANA 大型執行個體伺服器的 Linux 作業系統上已安裝 Perl。 Perl 已預先安裝在您的 HANA 大型執行個體單元上。 若要檢查 Perl 版本，請使用下列命令：

`perl -v`

![執行此命令即可複製公開金鑰](./media/hana-overview-high-availability-disaster-recovery/perl_screen.png)


## <a name="set-up-storage-snapshots"></a>設定儲存體快照集

若要使用 HANA 大型執行個體來設定儲存體快照集，請遵循下列步驟：
1. 確定 HANA (大型執行個體) 伺服器的 Linux 作業系統上已安裝 Perl。
1. 修改 /etc/ssh/ssh\_config 以新增 _MACs hmac-sha1_ 這一行。
1. 針對您目前執行的每個 SAP HANA 執行個體，在主要節點上建立 SAP HANA 備份使用者帳戶 (如果適用)。
1. 在所有 SAP HANA (大型執行個體) 伺服器上安裝 SAP HANA HDB 用戶端。
1. 在每個區域的第一部 SAP HANA (大型執行個體) 伺服器上，建立一個公開金鑰，以便存取其底下控制快照集建立的儲存體基礎結構。
1. 將指令碼和組態檔從 [ GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) 複製到 SAP HANA 安裝中的 **hdbsql** 位置。
1. 視需要修改 HANABackupDetails.txt 檔案以符合適當的客戶規格。

從 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) 取得最新快照集指令碼和文件。 當您從 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) 下載快照集指令碼套件時，也會取得包含在指令碼套件中的指令碼 PDF 文件。 每個指令碼套件都有它自己的 PDF 文件。

### <a name="consideration-for-mcod-scenarios"></a>MCOD 案例的考量
如果您在某個 HANA 大型執行個體單位上使用多個 SAP HANA 執行個體執行 [MCOD 案例](https://launchpad.support.sap.com/#/notes/1681092)，則會有為每個 SAP HANA 執行個體佈建的個別存放磁碟區。 在自助服務快照集自動化的目前版本中，您無法在每個 HANA 執行個體系統識別碼 (SID) 上起始個別快照集。 此功能會檢查組態檔中伺服器的已註冊 SAP HANA 執行個體 (請參閱本文後面的內容)，並對單位上註冊的所有執行個體磁碟區執行同步快照。
 

### <a name="step-1-install-the-sap-hana-hdb-client"></a>步驟 1：安裝 SAP HANA HDB 用戶端

安裝在 SAP HANA on Azure (大型執行個體) 上的 Linux 作業系統包含針對備份和災害復原目的執行 SAP HANA 儲存體快照集時所需的資料夾與指令碼。 檢查 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) 中是否有更近期的版本。 最新的指令碼版本為 3.x。 不同的指令碼在相同的主要版本中可能有不同的次要版本。

>[!IMPORTANT]
>將指令碼 2.1 版移至 3.x 版時，請注意組態檔的結構和某些語法也會一併變更。 請參閱特定章節中的圖說文字。 

安裝 SAP HANA 時，是由您負責在 HANA 大型執行個體單位上安裝 SAP HANA HDB 用戶端。

### <a name="step-2-change-the-etcsshsshconfig"></a>步驟 2：變更 /etc/ssh/ssh\_config

新增 _MACs hmac-sha1_ 這一行來變更 `/etc/ssh/ssh_config`，如下所示：
```
#   RhostsRSAAuthentication no
#   RSAAuthentication yes
#   PasswordAuthentication yes
#   HostbasedAuthentication no
#   GSSAPIAuthentication no
#   GSSAPIDelegateCredentials no
#   GSSAPIKeyExchange no
#   GSSAPITrustDNS no
#   BatchMode no
#   CheckHostIP yes
#   AddressFamily any
#   ConnectTimeout 0
#   StrictHostKeyChecking ask
#   IdentityFile ~/.ssh/identity
#   IdentityFile ~/.ssh/id_rsa
#   IdentityFile ~/.ssh/id_dsa
#   Port 22
Protocol 2
#   Cipher 3des
#   Ciphers aes128-ctr,aes192-ctr,aes256-ctr,arcfour256,arcfour128,aes128-cbc,3des-cbc
#   MACs hmac-md5,hmac-sha1,umac-64@openssh.com,hmac-ripemd160
MACs hmac-sha1
#   EscapeChar ~
#   Tunnel no
#   TunnelDevice any:any
#   PermitLocalCommand no
#   VisualHostKey no
#   ProxyCommand ssh -q -W %h:%p gateway.example.com
```

### <a name="step-3-create-a-public-key"></a>步驟 3︰建立公開金鑰

若要能夠存取 HANA 大型執行個體租用戶的儲存體快照集介面，您必須透過公開金鑰建立登入程序。 在租用戶的第一部 SAP HANA on Azure (大型執行個體) 伺服器上，建立用來存取儲存體基礎結構的公開金鑰。 此公開金鑰可確保不需密碼即可登入儲存體快照集介面。 建立公開金鑰，也表示您不需要維護密碼認證。 在 SAP HANA (大型執行個體) 伺服器上的 Linux 中，請執行下列命令來產生公開金鑰：
```
  ssh-keygen –t dsa –b 1024
```
新的位置是 **_/root/.ssh/id\_dsa.pub**。 請勿輸入實際密碼，否則您每次登入時都需要輸入密碼。 改為按 **Enter** 兩次，在登入時就不需要輸入密碼。

將資料夾切換至 **/root/.ssh/**，然後執行 `ls` 命令，以確定已如預期地更正公開金鑰。 如果金鑰存在，您可以執行下列命令來複製它︰

![執行此命令即可複製公開金鑰](./media/hana-overview-high-availability-disaster-recovery/image2-public-key.png)

此時，請連絡「SAP HANA on Azure 服務管理」並提供公開金鑰給他們。 服務代表會使用公開金鑰，將它註冊在針對 HANA 大型執行個體租用戶所開闢的底層儲存體基礎結構中。

### <a name="step-4-create-an-sap-hana-user-account"></a>步驟 4：建立 SAP HANA 使用者帳戶

若要開始建立 SAP HANA 快照集，您必須在 SAP HANA 中建立使用者帳戶，以供儲存體快照集指令碼使用。 為此目的，在 SAP HANA Studio 中建立 SAP HANA 使用者帳戶。 必須在 SYSTEMDB 之下 (而「非」在 MDC 的 SID 資料庫之下) 建立使用者。 在單一容器環境中，使用者會設定於租用戶資料庫之下。 此帳戶必須具備下列權限：「備份管理」和「目錄讀取」。 在此範例中，使用者名稱是 **SCADMIN**。 HANA Studio 中所建立的使用者帳戶名稱會區分大小寫。 請務必選取 [否]，要求使用者在下次登入時變更密碼。

![在 HANA Studio 中建立使用者](./media/hana-overview-high-availability-disaster-recovery/image3-creating-user.png)

如果您在某單位上使用的 MCOD 部署有多個 SAP HANA 執行個體，則需要針對每個 SAP HANA 執行個體重複執行此步驟。

### <a name="step-5-authorize-the-sap-hana-user-account"></a>步驟 5：授權 SAP HANA 使用者帳戶

在此步驟中，您可針對您所建立的 SAP HANA 使用者帳戶進行授權，讓指令碼不必在執行階段提交密碼。 SAP HANA 命令 `hdbuserstore` 可讓您建立儲存在一或多個 SAP HANA 節點上的 SAP HANA 使用者金鑰。 使用者金鑰可讓使用者存取 SAP HANA，而不需要在指令碼程序內管理密碼。 本文稍後會討論指令碼處理。

>[!IMPORTANT]
>以使用者身分執行下列命令，則會打算執行指令碼。 否則，指令碼無法正常運作。

輸入 `hdbuserstore` 命令，如下所示︰

**非 MDC HANA 安裝程式**
```
hdbuserstore set <key> <host>:<3[instance]15> <user> <password>
```

**MDC HANA 安裝程式**
```
hdbuserstore set <key> <host>:<3[instance]13> <user> <password>
```

在以下範例中，使用者為 **SCADMIN01**、主機名稱為 **lhanad01**，而執行個體編號是 **01**：
```
hdbuserstore set SCADMIN01 lhanad01:30115 <backup username> <password>
```
如果您在某單位上使用具有多個 SAP HANA 執行個體的 HANA MCOD 部署，則需要針對該單位上的每個 SAP HANA 執行個體及關聯備份使用者，重複執行此步驟。

如果您有 SAP HANA 相應放大組態，則需要從單一伺服器管理所有指令碼。 在此範例中，必須針對每個主機改變 SAP HANA 金鑰 **SCADMIN01**，以顯示與金鑰相關的主機。 以 HANA DB 的執行個體編號修訂 SAP HANA 備份帳戶。 此金鑰必須在其被指派過去的主機上具有系統管理權限，而相應放大組態的備份使用者則必須具備所有 SAP HANA 執行個體的存取權限。 假設有三個名稱分別為 **lhanad01**、**lhanad02** 和 **lhanad03** 的相應放大節點，命令序列會像這樣：

```
hdbuserstore set SCADMIN01 lhanad01:30115 SCADMIN <password>
hdbuserstore set SCADMIN01 lhanad02:30115 SCADMIN <password>
hdbuserstore set SCADMIN01 lhanad03:30115 SCADMIN <password>
```

### <a name="step-6-get-the-snapshot-scripts-configure-the-snapshots-and-test-the-configuration-and-connectivity"></a>步驟 6：取得快照集指令碼、設定快照集並測試組態與連線能力

從 [GitHub](https://github.com/Azure/hana-large-instances-self-service-scripts) 下載最新版本的指令碼。 將下載的指令碼和文字檔複製到 **hdbsql** 的工作目錄。 就目前的 HANA 安裝而言，這個目錄會使用下列格式：/hana/shared/D01/exe/linuxx86\_64/hdb。 
``` 
azure_hana_backup.pl 
azure_hana_replication_status.pl 
azure_hana_snapshot_details.pl 
azure_hana_snapshot_delete.pl 
testHANAConnection.pl 
testStorageSnapshotConnection.pl 
removeTestStorageSnapshot.pl
azure_hana_dr_failover.pl
azure_hana_test_dr_failover.pl 
HANABackupCustomerDetails.txt 
``` 

處理 Perl 指令碼時： 

- 請勿修改指令碼，除非 Microsoft Operations 另有指示。
- 當系統要求您修改指令碼或參數檔案時，請一律使用 Linux 文字編輯器，例如 "vi"，而非使用 Windows 編輯器，例如記事本。 使用 Windows 編輯器可能會損毀檔案格式。
- 請一律使用最新的指令碼。 您可以從 GitHub 下載最新版本。
- 全面使用相同版本的指令碼。
- 將指令碼直接用於生產系統之前，請先測試指令碼並熟悉指令碼輸出所需的參數。
- 請勿變更 Microsoft Operations 佈建的伺服器掛接點名稱。 這些指令碼依賴這些標準的掛接點，才可成功執行。


不同指令碼和檔案的用途如下：

- **azure\_hana\_backup.pl**：此指令碼會使用 Linux Cron 排程公用程式來排定，以在 HANA 資料和共用磁碟區、/hana/logbackups 磁碟區或作業系統上執行儲存體快照集。
- **azure\_hana\_replication\_status.pl**：此指令碼提供有關從生產網站到災害復原網站的複寫狀態基本詳細資料。 此指令碼可進行監視以確保正在進行複寫，並顯示正在複寫的項目大小。 它也可提供複寫時間太久或連結中斷時的指引。
- **azure\_hana\_snapshot\_details.pl**：此指令碼提供每個磁碟區在環境中現存之所有快照集的基本詳細資料清單。 此指令碼可在主要伺服器或在災害復原位置的伺服器單位上執行。 此指令碼提供下列資訊，這些資訊會依包含快照集的每個磁碟區細分：
   * 磁碟區中所有快照集的大小
   * 該磁碟區中的每個快照集都包含下列詳細資料： 
      - 快照集名稱 
      - 建立時間 
      - 快照集的大小
      - 快照集的頻率
      - 與該快照集相關聯的 HANA 備份識別碼 (如果相關)
- **azure\_hana\_snapshot\_delete.pl**：此指令碼可刪除一個儲存體快照集或一組快照集。 您可以使用在 HANA Studio 中找到的 SAP HANA 備份識別碼或儲存體快照集名稱。 備份識別碼目前只會與針對 HANA 資料/記錄/共用磁碟區所建立的快照集繫結。 否則，如果您輸入快照集識別碼，則指令碼會搜尋所有與輸入的快照集識別碼相符的快照集。  
- **testHANAConnection.pl**：此指令碼會測試 SAP HANA 執行個體的連線，需要有此連線才能設定儲存體快照集。
- **testStorageSnapshotConnection.pl**：此指令碼有兩種用途。 第一種用途是確保執行指令碼的 HANA 大型執行個體單位可存取指派的儲存體虛擬機器，進而能夠存取 HANA 大型執行個體的儲存體快照集介面。 第二種用途是針對您要測試的 HANA 執行個體建立暫時性的快照集。 您應該對伺服器上的每個 HANA 執行個體執行此指令碼，以確保備份指令碼可如預期般運作。
- **removeTestStorageSnapshot.pl**：此指令碼可刪除以 **testStorageSnapshotConnection.pl** 指令碼所建立的測試快照集。
- **azure\_hana\_dr\_failover.pl**：此指令碼可起始 DR 容錯移轉至另一個區域。 需要在 DR 區域中的 HANA 大型執行個體單位上，或在要對其執行容錯移轉的單位上，執行該指令碼。 此指令碼會停止從主要端到次要端的儲存體複寫，還原 DR 磁碟區上的最新快照集，並提供 DR 磁碟區的掛接點。
- **azure\_hana\_test\_dr\_failover.pl**：此指令碼可執行測試容錯移轉至 DR 網站。 不同於 azure_hana_dr_failover.pl script，此執行不會中斷從主要到次要的儲存體複寫。 而是複製在 DR 網站建立的複寫存放磁碟區，並提供複製磁碟區的掛接點。 
- **HANABackupCustomerDetails.txt**：這個檔案是可修改的組態檔，請加以修改以適應您的 SAP HANA 組態。 HANABackupCustomerDetails.txt 檔案是執行儲存體快照集之指令碼的控制及組態檔。 調整該檔案以因應您的用途和設定。 當您的執行個體部署完成時，您會從「SAP HANA on Azure 服務管理」收到「儲存體備份名稱」和「儲存體 IP 位址」。 您不能修改此檔案中任何變數的順序、排序或間距。 如果這麼做，指令碼將無法正常執行。 此外，您會從「SAP HANA on Azure 服務管理」收到相應放大節點或主要節點 (如果相應放大) 的 IP 位址。 您也知道在 SAP HANA 安裝期間取得的 HANA 執行個體編號。 您現在必須將備份名稱新增至組態檔。

若為相應增加或相應放大部署，在您填入 HANA 大型執行個體單位的伺服器名稱和伺服器 IP 位址之後，組態檔會如下列範例所示。 為每個您想要備份或復原的 SAP HANA SID 填入所有必填欄位。

對於一段時間內不想備份的執行個體列，您也可以在必填欄位前面加上 "#" 加以註解。 如果不需要備份或復原特定執行個體，您也不需要輸入伺服器上包含的所有 SAP HANA 執行個體。 所有欄位必須保持格式，否則所有指令碼會擲回錯誤訊息，而且指令碼會終止。 使用最後一個 SAP HANA 執行個體之後，您可以刪除未使用之任何 SID 資訊詳細資料的其他必要列。 必須填入、加上註解或刪除所有列。

>[!IMPORTANT]
>檔案結構會隨著從 2.1 版移至 3.x 版而變動。 如果您想要使用 3.x 版的指令碼，需要調整組態檔結構。 


```
HANA Server Name: testing01
HANA Server IP Address: 172.18.18.50
```

對於在 HANA 大型執行個體單位上設定的每個執行個體或相應放大設定，您需要定義資料，如下所示：

    
```
######***SID #1 Information***#####
SID1: h01
###Provided by Microsoft Operations###
SID1 Storage Backup Name: clt1h01backup
SID1 Storage IP Address: 172.18.18.11
######     Customer Provided    ######
SID1 HANA instance number: 00
SID1 HANA HDBuserstore Name: SCADMINH01
```
如需相應放大和 HANA 系統複寫設定，請在每一個節點上重複此設定。 此措施可確保在失敗情況下，備份和最終儲存體複寫可繼續運作。   

將所有組態資料放入 HANABackupCustomerDetails.txt 檔案後，請檢查 HANA 執行個體資料的組態是否正確。 使用指令碼 `testHANAConnection.pl`，其與 SAP HANA 相應增加或相應放大組態無關。

```
testHANAConnection.pl
```

如果您有 SAP HANA 相應放大組態，請確定主要的 HANA 執行個體能夠存取所有必要的 HANA 伺服器和執行個體。 測試指令碼並無參數，但您必須將資料新增至 HANABackupCustomerDetails.txt 組態檔，指令碼才能正常執行。 只會傳回殼層命令錯誤碼，所以指令碼無法檢查每個執行個體的錯誤。 即便如此，指令碼會提供一些實用的意見讓您再次檢查。

若要執行指令碼，請輸入下列命令：
```
 ./testHANAConnection.pl
```
如果指令碼成功取得 HANA 執行個體的狀態，則會顯示 HANA 連接成功的訊息。


下一個測試步驟是根據您放入 HANABackupCustomerDetails.txt 組態檔的資料來檢查儲存體的連線，然後執行測試快照集。 在執行 `azure_hana_backup.pl` 指令碼之前，您必須先執行此測試。 如果磁碟區不含快照集，則無法判斷原因是磁碟區空白，還是 SSH 失敗而無法取得快照集詳細資料。 因此，指令碼會執行兩個步驟：

- 它會確認租用戶的儲存體虛擬機器和介面是否可供存取以便讓指令碼執行快照集。
- 依 HANA 執行個體，為每個磁碟區建立測試 (或虛設) 快照集。

基於這個理由，會包含 HANA 執行個體作為引數。 如果執行失敗，則無法提供儲存體連線錯誤檢查。 即使沒有錯誤檢查，指令碼也會提供有用的提示。

1. 執行命令序列以執行此測試：

   ```
   ssh <StorageUserName>@<StorageIP>
   ```

   送交 HANA 大型執行個體單位時，已提供儲存體使用者名稱和儲存體 IP 位址。

1. 執行測試指令碼：
   ```
    ./testStorageSnapshotConnection.pl
   ```

指令碼會嘗試使用在先前設定步驟中提供的公開金鑰和 HANABackupCustomerDetails.txt 檔案中所設定的資料來登入儲存體。 如果登入成功，則會顯示下列內容：

```
**********************Checking access to Storage**********************
Storage Access successful!!!!!!!!!!!!!!
```

如果連線到儲存體主控台時發生問題，其輸出會如下所示：

```
**********************Checking access to Storage**********************
WARNING: Storage check status command 'volume show -type RW -fields volume' failed: 65280
WARNING: Please check the following:
WARNING: Was publickey sent to Microsoft Service Team?
WARNING: If passphrase entered while using tool, publickey must be re-created and passphrase must be left blank for both entries
WARNING: Ensure correct IP address was entered in HANABackupCustomerDetails.txt
WARNING: Ensure correct Storage backup name was entered in HANABackupCustomerDetails.txt
WARNING: Ensure that no modification in format HANABackupCustomerDetails.txt like additional lines, line numbers or spacing
WARNING: ******************Exiting Script*******************************
```

成功登入儲存體虛擬機器介面之後，指令碼會繼續進行階段 2 並建立測試快照集。 SAP HANA 的三個節點相應放大組態輸出如下所示：

```
**********************Creating Storage snapshot**********************
Taking snapshot testStorage.recent for hana_data_hm3_mnt00001_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00001_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00002_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00002_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00003_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_data_hm3_mnt00003_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_backups_hm3_t020_dp ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_backups_hm3_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_hm3_mnt00001_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_hm3_mnt00002_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_log_hm3_mnt00003_t020_vol ...
Snapshot created successfully.
Taking snapshot testStorage.recent for hana_shared_hm3_t020_vol ...
Snapshot created successfully.
```

如果已成功使用此指令碼執行測試快照集，您可以繼續設定實際的儲存體快照集。 如果不成功，請先調查問題，再繼續進行。 測試快照集應先留著，直到完成第一個實際快照集為止。


### <a name="step-7-perform-snapshots"></a>步驟 7：執行快照集

當準備步驟都完成後，您就可以開始設定實際的儲存體快照集組態。 要排程的指令碼可與 SAP HANA 相應增加和相應放大組態搭配運作。 如需定期或正常執行備份指令碼，請透過使用 cron 公用程式排程指令碼。 

您可以建立的快照集備份有三種：
- **HANA**：合併的快照集備份，其中包含 /hana/data 和 /hana/shared (也包含 /usr/sap) 的磁碟區會由協調式快照集所涵蓋。 您可以從這個快照集還原單一檔案。
- **記錄**︰/hana/logbackups 磁碟區的快照集備份。 不會觸發 HANA 快照集來執行此儲存體快照集。 此存放磁碟區用來包含 SAP HANA 交易記錄備份。 這會提高執行頻率，以限制記錄成長並防止資料遺失。 您可以從這個快照集還原單一檔案。 請勿將執行頻率降低到 3 分鐘以下。
- **開機**：包含 HANA 大型執行個體的開機邏輯單元編號 (LUN) 磁碟區快照集。 您只能透過 HANA 大型執行個體的 Type I SKU 來進行此快照集備份。 您無法從包含開機 LUN 之磁碟區的快照集還原單一檔案。


>[!NOTE]
> 這三種快照集類型的呼叫語法，隨著移至 3.x 版指令碼 (其支援 MCOD 部署) 而有所不同。 不需要再指定執行個體的 HANA SID。 您必須確認組態檔 HANABackupCustomerDetails.txt 中已設定單位的 SAP HANA 執行個體。

>[!NOTE]
> 當您第一次執行指令碼時，可能會在多個 SID 環境中顯示一些未預期的錯誤。 重新執行指令碼會修正此問題。



使用指令碼 azure_hana_backup.pl 執行儲存體快照集的全新呼叫語法如下所示：

```
HANA backup covering /hana/data and /hana/shared (includes/usr/sap)
./azure_hana_backup.pl hana <snapshot_prefix> <snapshot_frequency> <number of snapshots retained>

For /hana/logbackups snapshot
./azure_hana_backup.pl logs <snapshot_prefix> <snapshot_frequency> <number of snapshots retained>

For snapshot of the volume storing the boot LUN
./azure_hana_backup.pl boot <HANA Large Instance Type> <snapshot_prefix> <snapshot_frequency> <number of snapshots retained>

```

參數詳細資料如下所示： 

- 第一個參數描繪快照集備份的類型。 允許的值為 **hana**、**logs** 和 **boot**。 
- 只有開機磁碟區備份需要參數 **<HANA Large Instance Type>**。 取決於 HANA 大型執行個體單位而定，會有 "TypeI" 或 "TypeII" 兩個有效值。 若想了解您的單位類型，請參閱 [Azure 上的 SAP HANA (大型執行個體) 概觀和架構](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-overview-architecture)。  
- 參數 **<snapshot_prefix>** 是快照集類型的快照集或備份標籤。 其有兩個目的：一個是為它命名，好讓您知道這些快照集的相關內容。 第二個的目的是要讓指令碼 azure\_hana\_backup.pl 判斷保留在該特定標籤之下的儲存體快照集數目。 如果您使用兩個不同的標籤來排定兩個相同類型的儲存體快照集備份 (例如**hana**)，並定義各自應該保留 30 個快照集，您最後會得到受影響磁碟區的 60 個儲存體快照集。 只允許使用英數 (“A-Z,a-z,0-9”)、底線 (“_”) 和破折線 (“-“) 字元。 
- 參數 **<snapshot_frequency>** 保留供未來開發使用，沒有任何影響。 在執行**記錄**類型的備份時，將其設定為「3 分鐘」，執行其他備份類型時，將其設定為「15 分鐘」。
- 參數 **<number of snapshots retained>** 會藉由定義具有相同快照集首碼 (標籤) 的快照集數目，來間接定義快照集的保留期。 對於透過 cron 來進行的已排程執行作業來說，這個參數很重要。 如果具有相同 snapshot_prefix 的快照集數目超過此參數給定的數目，則會先刪除最舊的快照集，然後再執行新的儲存體快照集。

在相應放大的情況下，指令碼會執行額外檢查，確保您可以存取所有的 HANA 伺服器。 指令碼也會先檢查所有 HANA 執行個體是否都傳回適當的執行個體狀態，然後再建立 SAP HANA 快照集。 SAP HANA 快照集的後面會接著儲存快照集。

執行 `azure_hana_backup.pl` 指令碼會以下列三個階段建立儲存體快照集：

1. 執行 SAP HANA 快照集
1. 執行儲存體快照集
1. 先移除已建立的 SAP HANA 快照集，再執行儲存體快照集

若要執行指令碼，您可從複製指令碼過去的 HDB 可執行檔資料夾中呼叫它。 

保留期取決於您執行指令碼時以參數送出的快照集數目。 儲存體快照集所涵蓋的時間量，是執行期間的函式以及在執行指令碼時提交做為參數的快照集數目函式構成。 如果所保留的快照集數目超出指令碼呼叫中以參數指定的數目，將會先刪除相同標籤的最舊儲存體快照集，然後再執行新的快照集。 您所提供作為最後呼叫參數的數字，就是您可用來控制要保留之快照集數目的數字。 藉此數字，您也可以間接控制快照集所使用的磁碟空間。 

> [!NOTE]
>只要您一變更標籤，就會重新開始計算。 您必須嚴謹處理標籤，您的快照集才不會遭到意外刪除。

## <a name="snapshot-strategies"></a>快照集策略
針對不同類型的快照集，其頻率取決於您是否使用 HANA 大型執行個體災害復原功能。 此功能依賴儲存快照集，其可能需要有關儲存體快照集執行頻率和期間的特殊建議。 

在下面的考量和建議中，假設您「不」使用 HANA 大型執行個體所提供的災害復原功能。 相反地，您會使用儲存體快照集來建立備份，且您能夠提供過去 30 天的時間點復原。 在快照集數量和空間有限的情況下，客戶考量了下列需求：

- 時間點復原的復原時間。
- 使用的空間。
- 從災害潛在復原的復原點和復原時間目標。
- 對磁碟執行的最終 HANA 完整資料庫備份。 每次對磁碟執行完整資料庫備份，或執行 **backint** 介面時，執行儲存體快照集都會失敗。 如果除了儲存體快照集之外，您還打算執行完整資料庫備份，請確定在此期間停用儲存體快照集的執行。
- 每個磁碟區的快照集數目 (只能有 250 個)。


對於不使用 HANA 大型執行個體災害復原功能的客戶，快照集期間較不頻繁。 在這種情況下，客戶會以 12 小時或 24 小時的期間來對 /hana/data 和 /hana/shared (包括 /usr/sap) 執行合併的快照集，並保留這些快照集一個月。 記錄備份磁碟區的快照集也是如此。 然而，記錄備份磁碟區的 SAP HANA 交易記錄備份，則會以 5 到 15 分鐘的期間來執行。

使用 cron 來執行排程儲存體的快照集最為合適。 請針對所有備份和災害復原需求使用相同的指令碼，以及修改指令碼輸入以符合各種要求的備份時間。 這些快照集在 cron 中都會根據其執行時間以不同方式排定：每小時、12 小時、每天或每週。 

下列是 /etc/crontab 中的 cron 排程範例：
```
00 1-23 * * * ./azure_hana_backup.pl hana hourlyhana 15min 46
10 00 * * *  ./azure_hana_backup.pl hana dailyhana 15min 28
00,05,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup.pl logs regularlogback 3min 28
22 12 * * *  ./azure_hana_backup.pl logs dailylogback 3min 28
30 00 * * *  ./azure_hana_backup.pl boot TypeI dailyboot 15min 28
```
在上述範例中，有每小時的合併快照集會涵蓋含有 /hana/data 和 /hana/shared (包括 /usr/sap) 位置的磁碟區。 使用此類快照集以加快過去兩天內的時間點復原速度。 此外，這些磁碟區會有每日快照集。 因此，您會獲得涵蓋兩天的每小時快照集，以及涵蓋四週的每日快照集。 此外，每天會備份一次交易記錄備份磁碟區。 這些備份也會保存四週。 正如您在 crontab 第三行所看到的，系統會排定每 5 分鐘執行一次 HANA 交易記錄備份。 執行儲存體快照集的不同 cron 作業開始時間會錯開，以免系統在特定時間點一次執行所有快照集。 

在以下範例中，您會每小時執行一次合併的快照集，其所涵蓋的磁碟區包含 /hana/data 和 /hana/shared (包括 /usr/sap) 位置。 您會將這些快照集保留兩天。 交易記錄備份磁碟區的快照集會每隔 5 分鐘執行一次，並保留 4 個小時。 和之前一樣，系統會將 HANA 交易記錄檔排定為每 5 分鐘備份一次。 在開始交易記錄備份後，系統會延後 2 分鐘再執行交易記錄備份磁碟區的快照集。 一般情況下，SAP HANA 交易記錄備份應該能在這 2 分鐘的時間內完成。 和之前一樣，儲存體快照集每天會將包含開機 LUN 的磁碟區備份一次，並保留四週之久。

```
10 0-23 * * * ./azure_hana_backup.pl hana hourlyhana 15min 48
0,5,10,15,20,25,30,35,40,45,50,55 * * * * ./azure_hana_backup.pl logs regularlogback 3min 28
2,7,12,17,22,27,32,37,42,47,52,57 * * * *  ./azure_hana_backup.pl logs logback 3min 48
30 00 * * *  ./azure_hana_backup.pl boot TypeII dailyboot 15min 28
```

下圖說明上述範例的順序，但不包括開機 LUN：

![備份與快照集之間的關聯性](./media/hana-overview-high-availability-disaster-recovery/backup_snapshot_updated0921.PNG)

SAP HANA 會定期對 /hana/log 磁碟區執行寫入作業，以將認可的變更記錄至資料庫。 SAP HANA 會定期將儲存點寫入 /hana/data 磁碟區。 SAP HANA 交易記錄備份會依照 crontab 的指定每 5 分鐘執行一次。 您也會看到系統會因為針對 /hana/data 和 /hana/shared 磁碟區觸發合併的儲存體快照集，而每小時執行一次 SAP HANA 快照集。 HANA 快照集成功之後，就會執行合併的儲存體快照集。 在 HANA 交易記錄備份完成大約 2 分鐘後，系統會依照 crontab 的指示，每隔 5 分鐘在 /hana/logbackup 磁碟區上執行一次儲存體快照集。

> 

>[!IMPORTANT]
> 只有在快照集與 SAP HANA 交易記錄備份一起執行時，使用儲存體快照集來進行 SAP HANA 備份才有價值。 這些交易記錄備份必須要涵蓋儲存體快照集之間的時段。 

如果您已對使用者承諾執行 30 天的時間點復原，則需具備下列條件︰

- 在極端情況下，針對已存在 30 天的 /hana/data 和 /hana/shared 存取合併儲存體快照集。
- 擁有連續交易記錄備份，其涵蓋任何合併儲存體快照之間的時間。 因此，最舊的交易記錄備份磁碟區快照集必須達 30 天之久。 如果您將交易記錄備份複製到位於 Azure 儲存體上的另一個 NFS 共用，情況並非如此。 在這種情況下，您可以從該 NFS 共用提取舊的交易記錄備份。

若要受惠於儲存體快照集並最終受益於交易記錄備份的儲存體複寫，您必須變更 SAP HANA 用來寫入交易記錄備份的目的地位置。 您可以在 HANA Studio 中進行這項變更。 雖然 SAP HANA 會自動備份完整的記錄區段，但您應該指定記錄備份間隔來加以確定。 使用災害復原選項時尤其如此，因為您通常會想要以確定的期間來執行記錄備份。 在下列案例中，採用 15 分鐘的記錄備份間隔。

![在 SAP HANA Studio 中排定 SAP HANA 備份記錄](./media/hana-overview-high-availability-disaster-recovery/image5-schedule-backup.png)

您也可以選擇比每隔 15 分鐘更頻繁的備份。 頻率較高的設定經常與 HANA 大型執行個體的災害復原功能搭配使用。 有些客戶會每隔 5 分鐘執行一次交易記錄備份。  

如果資料庫從未進行過備份，則最後一個步驟是執行檔案型備份，以建立必須存在於備份資料目錄內的單一備份項目。 否則，SAP HANA 無法起始您指定的記錄備份。

![建立檔案型備份以建立單一備份項目](./media/hana-overview-high-availability-disaster-recovery/image6-make-backup.png)


在成功執行第一個儲存體快照集之後，您可以刪除步驟 6 所執行的測試快照集。 若要這樣做，請執行 `removeTestStorageSnapshot.pl` 指令碼：
```
./removeTestStorageSnapshot.pl
```

以下是指令碼輸出的範例：
```
Checking Snapshot Status for h80
**********************Checking access to Storage**********************
Storage Snapshot Access successful.
**********************Getting list of volumes that match HANA instance specified**********************
Collecting set of volumes hosting HANA matching pattern *h80* ...
Volume show completed successfully.
Adding volume hana_data_h80_mnt00001_t020_vol to the snapshot list.
Adding volume hana_log_backups_h80_t020_vol to the snapshot list.
Adding volume hana_shared_h80_t020_vol to the snapshot list.
**********************Adding list of snapshots to volume list**********************
Collecting set of snapshots for each volume hosting HANA matching pattern *h80* ...
**********************Displaying Snapshots by Volume**********************
hana_data_h80_mnt00001_t020_vol
Test_HANA_Snapshot.2018-02-06_1753.3
Test_HANA_Snapshot.2018-02-06_1815.2
….
Command completed successfully.
Exiting with return code: 0
Command completed successfully.
```


### <a name="monitoring-the-number-and-size-of-snapshots-on-the-disk-volume"></a>監視磁碟區上的快照集數目和大小

在特定的存放磁碟區上，您可以監視快照集的數目和這些快照集的儲存體耗用量。 `ls` 命令不會顯示快照集目錄或檔案。 不過，Linux 作業系統命令 `du` 會顯示這些儲存體快照集的詳細資料，因為它們儲存在相同的磁碟區上。 此命令可與下列選項搭配使用：

- `du –sh .snapshot`：此選項提供快照集目錄內所有快照集的總計。
- `du –sh --max-depth=1`：此選項會列出所有儲存在 **.snapshot** 資料夾中的快照集及每個快照集的大小。
- `du –hc`：此選項會提供所有快照集使用的總大小。

您可以使用這些命令來確定所建立和儲存的快照集並未耗用磁碟區上的所有儲存體。

>[!NOTE]
>使用上述命令無法看到開機 LUN 的快照集。

### <a name="getting-details-of-snapshots"></a>取得快照集的詳細資料
若要取得更詳細的快照集資料，您也可以使用 `azure_hana_snapshot_details.pl` 指令碼。 如果災害復原位置有作用中的伺服器，則可以在任一位置執行此指令碼。 此指令碼提供下列輸出，這些輸出會依包含快照集的每個磁碟區細分： 
   * 磁碟區中所有快照集的大小
   * 該磁碟區中的每個快照集都包含下列詳細資料： 
      - 快照集名稱 
      - 建立時間 
      - 快照集的大小
      - 快照集的頻率
      - 與該快照集相關聯的 HANA 備份識別碼 (如果相關)

以下是指令碼執行語法的範例：

```
./azure_hana_snapshot_details.pl 
```

因為指令碼會嘗試擷取 HANA 備份識別碼，所以它需要連線到 SAP HANA 執行個體。 此連線需要正確設定 HANABackupCustomerDetails.txt 組態檔。 磁碟區上兩個快照集的輸出如下所示：

```
**********************************************************
****Volume: hana_shared_SAPTSTHDB100_t020_vol       ***********
**********************************************************
Total Snapshot Size:  411.8MB
----------------------------------------------------------
Snapshot:   customer.2016-09-20_1404.0
Create Time:   "Tue Sep 20 18:08:35 2016"
Size:   2.10MB
Frequency:   customer 
HANA Backup ID:   
----------------------------------------------------------
Snapshot:   customer2.2016-09-20_1532.0
Create Time:   "Tue Sep 20 19:36:21 2016"
Size:   2.37MB
Frequency:   customer2
HANA Backup ID:   
```



### <a name="reducing-the-number-of-snapshots-on-a-server"></a>縮減伺服器上的快照數目

如先前所述，您可以針對您儲存的快照集，減少某些標籤的數目。 用來起始快照之命令的最後兩個參數是一個標籤及您想要保留的快照數目。

```
./azure_hana_backup.pl hana dailyhana 15min 28
```

在前一個範例中，快照集標籤是 **dailyhana**，而此標籤內要保留的快照集數目則是 **28**。 在您因應磁碟空間耗用量時，可能會想要縮減所儲存的快照集數目。 若要將快照集數目減少為 15 (舉例來說)，有個簡單的方法是將最後一個參數設定為 **15** 來執行指令碼：

```
./azure_hana_backup.pl hana dailyhana 15min 15
```

如果您以此設定執行指令碼，快照集數目 (包括新的儲存體快照集) 會是 15。 15 個最新的快照集會保留下來，然後會刪除 15 個較舊的快照集。

 >[!NOTE]
 > 有存在時間超過 1 小時的快照集時，此指令碼才會減少快照集數目。 指令碼不會刪除過去 1 小時內的快照集。 這些限制與所提供的選擇性災害復原功能有關。

如果您不想再維護一組含有備份標籤 (即語法範例中的 **hanadaily**) 的快照集，您可以使用 **0** 作為保留數字來執行指令碼。 然後，所有符合該標籤的快照集就會遭到移除。 不過，移除所有快照集可能會影響 HANA 大型執行個體災害復原的功能。

若要刪除特定快照集，第二種選項是使用 `azure_hana_snapshot_delete.pl` 指令碼。 此指令碼的用途在於使用在 HANA Studio 中找到的 HANA 備份識別碼或透過快照集名稱本身，來刪除單一快照集或一組快照集。 備份識別碼目前只會與針對 **hana** 快照集類型所建立的快照集繫結。 **記錄**和**開機**類型的快照集備份不會執行 SAP HANA 快照集，因此找不到這些快照集的備份識別碼。 如果您輸入快照集名稱，則指令碼會尋找不同磁碟區上符合所輸入快照集名稱的所有快照集。 

若要呼叫指令碼，您需要使用指令碼的呼叫語法來指定 HANA 執行個體的 SID：

```
./azure_hana_snapshot_delete.pl <SID>

```

您必須以 **root** 使用者身分執行指令碼。

如果您選取某個快照集，可以個別地刪除每個快照集。 您可先提供包含快照集的磁碟區，然後提供快照集名稱。 如果快照集存在於該磁碟區中，而且存在時間超過 1 小時，系統就會將它刪除。 您可藉由執行 `azure_hana_snapshot_details` 指令碼來尋找磁碟區名稱和快照集名稱。 

>[!IMPORTANT]
>如果有資料只存在於您要刪除的快照集上，刪除快照集後，該資料就會永久遺失。

  
## <a name="file-level-restore-from-a-storage-snapshot"></a>從儲存體快照集進行檔案層級還原
針對快照集類型 **hana** 和 **logs**，您可以直接在 **.snapshot** 目錄中的磁碟區上存取快照集。 每個快照集都有一個子目錄。 您可以將處於快照集製作時間點狀態的每個檔案，從該子目錄複製到實際的目錄結構中。 在指令碼的目前版本中，**沒有**針對快照集還原以自助形式提供的還原指令碼 (雖然可以在容錯移轉期間在 DR 網站上將快照集還原當作自助式 DR 指令碼的一部分執行)。 您必須藉由開啟服務要求來連絡 Microsoft 作業小組，以從現有可用的快照集還原所需的快照集。

>[!NOTE]
>單一檔案還原不適用於與 HANA 大型執行個體單位類型無關的開機 LUN 快照集。 **.snapshot** 目錄不會在開機 LUN 中顯示出來。 
 

## <a name="recover-to-the-most-recent-hana-snapshot"></a>復原到最近的 HANA 快照集

若遇到生產環境停止運作的情況，可以向「Microsoft Azure 支援服務」以客戶事件的形式起始從儲存體快照集復原的程序。 如果生產系統中的資料遭到刪除，而挽救資料的唯一方法是還原生產資料庫，則為高緊急性的情況。

若是其他情況，則時間點復原可能為低緊急性，可提前幾天規劃。 您可以使用「SAP HANA on Azure 服務管理」來規劃此復原，而不是發出高優先性的警示。 例如，您可能打算藉由套用新的增強套件來升級 SAP 軟體。 然後，您必須還原為代表增強套件升級前狀態的快照集。

傳送要求之前，您必須做準備。 接著，「SAP HANA on Azure 服務管理」小組就可以處理該要求，並提供已還原的磁碟區。 之後，您會根據快照集來還原 HANA 資料庫。 

以下將說明您如何為要求做準備：

>[!NOTE]
>視您使用的 SAP HANA 版本而定，您的使用者介面可能與下列螢幕擷取畫面不同。

1. 決定要還原哪一個快照。 除非另有指示，否則只會還原 hana/data 磁碟區。 

1. 關閉 HANA 執行個體。

 ![關閉 HANA 執行個體](./media/hana-overview-high-availability-disaster-recovery/image7-shutdown-hana.png)

1. 將每個 HANA 資料庫節點上的資料磁碟區卸載。 如果資料磁碟區仍掛接至作業系統，快照集還原會失敗。
 ![將每個 HANA 資料庫節點上的資料磁碟區卸載](./media/hana-overview-high-availability-disaster-recovery/image8-unmount-data-volumes.png)

1. 提出 Azure 支援要求，以及包含還原特定快照集的指示。

 - 還原期間︰「SAP HANA on Azure 服務管理」可能會要求您參加電話會議，確保您會協調、驗證和確認所還原的儲存體快照集是正確的。 

 - 還原之後：還原儲存體快照集之後，「SAP HANA on Azure 服務管理」會通知您。

1. 在還原程序完成後，重新掛接所有資料磁碟區。

 ![重新掛接所有資料磁碟區](./media/hana-overview-high-availability-disaster-recovery/image9-remount-data-volumes.png)

1. 當您透過 SAP HANA Studio 重新連接到 HANA DB 時，如果沒有自動出現復原選項，請選取 SAP HANA Studio 內的復原選項。 以下範例顯示還原到最後一個 HANA 快照集。 儲存體快照集會內嵌一個 HANA 快照集。 如果您要復原到最近的儲存體快照集，它應該是最近的 HANA 快照集。 (如果您要復原到較舊的儲存體快照，您就必須根據儲存體快照的建立時間找出 HANA 快照)。

 ![選取 SAP HANA Studio 內的復原選項](./media/hana-overview-high-availability-disaster-recovery/image10-recover-options-a.png)

1. 選取 [將資料庫復原到特定資料備份或儲存體快照集]。

 ![[指定復原類型] 視窗](./media/hana-overview-high-availability-disaster-recovery/image11-recover-options-b.png)

1. 選取 [指定不含目錄的備份]。

 ![[指定備份位置] 視窗](./media/hana-overview-high-availability-disaster-recovery/image12-recover-options-c.png)

1. 在 [目的地類型] 清單中，選取 [快照集]。

 ![[指定要復原的備份] 視窗](./media/hana-overview-high-availability-disaster-recovery/image13-recover-options-d.png)

1. 選取 [完成] 以開始復原程序。

 ![選取 [完成] 以開始復原程序。](./media/hana-overview-high-availability-disaster-recovery/image14-recover-options-e.png)

1. HANA 資料庫會還原並復原到儲存體快照集所包含的 HANA 快照集。

 ![HANA 資料庫會還原並復原到 HANA 快照集](./media/hana-overview-high-availability-disaster-recovery/image15-recover-options-f.png)

### <a name="recover-to-the-most-recent-state"></a>復原到最近狀態

下列程序會還原儲存體快照集所包含的 HANA 快照集。 然後，將交易記錄備份還原至資料庫在還原儲存體快照集之前的最近狀態。

>[!IMPORTANT]
>繼續進行之前，請先確定您擁有一串完整且連續的交易記錄備份。 如果沒有這些備份，您就無法還原資料庫的目前狀態。

1. 完成[復原到最近的 HANA 快照集](#recovering-to-the-most-recent-hana-snapshot)中的步驟 1-6。

1. 選取 [將資料庫復原到最近狀態]。

 ![選取 [將資料庫復原到最近狀態]。](./media/hana-overview-high-availability-disaster-recovery/image16-recover-database-a.png)

1. 指定最近 HANA 記錄備份的位置。 此位置必須包含從 HANA 快照集到最近狀態的所有 HANA 交易記錄備份。

 ![指定最近 HANA 記錄備份的位置](./media/hana-overview-high-availability-disaster-recovery/image17-recover-database-b.png)

1. 選取一個備份作為基礎來從中復原資料庫。 在此範例中，螢幕擷取畫面中的 HANA 快照集是儲存體快照集所包含的 HANA 快照集。 

 ![選取一個備份作為基礎來從中復原資料庫](./media/hana-overview-high-availability-disaster-recovery/image18-recover-database-c.png)

1. 如果 HANA 快照集快時間與最近狀態之間沒有差異，請清除 [使用差異備份] 核取方塊。

 ![如果沒有差異，請清除 [使用差異備份] 核取方塊](./media/hana-overview-high-availability-disaster-recovery/image19-recover-database-d.png)

1. 在摘要畫面上，選取 [完成] 以開始還原程序。

 ![在摘要頁面上按一下 [完成]](./media/hana-overview-high-availability-disaster-recovery/image20-recover-database-e.png)

### <a name="recover-to-another-point-in-time"></a>復原到另一個時間點
若要復原到介於 HANA 快照集 (包含在儲存體快照集中) 與晚於 HANA 快照集時間點復原的快照集之間的時間點，請執行下列步驟：

1. 確定您擁有包含從 HANA 快照到想要復原之時間在內的所有 HANA 交易記錄備份。
1. 開始執行[復原到最近狀態](#recovering-to-the-most-recent-state)下的程序。
1. 在程序的步驟 2 中，於 [指定復原類型] 視窗選取 [將資料庫復原到下列時間點]，然後指定時間點。 
1. 完成步驟 3-6。

## <a name="monitor-the-execution-of-snapshots"></a>監視快照集的執行

當您使用 HANA 大型執行個體的儲存體快照集時，您也需要監視這些快照集的執行情形。 執行儲存體快照集的指令碼會將輸出寫入檔案，然後儲存至與 Perl 指令碼相同的位置。 針對每個儲存體快照集都會寫入一個個別的檔案。 每個檔案的輸出會顯示快照集指令碼所執行的各階段︰

1. 尋找需要建立快照集的磁碟區。
1. 尋找從這些磁碟區建立的快照集。
1. 刪除最後存在的快照集以符合您指定的快照集數目。
1. 建立 SAP HANA 快照集。
1. 建立磁碟區的儲存體快照集。
1. 刪除 SAP HANA 快照集。
1. 將最近的快照集重新命名為 **.0**。

此指令碼 cab 最重要的部分如下：
```
**********************Creating HANA snapshot**********************
Creating the HANA snapshot with command: "./hdbsql -n localhost -i 01 -U SCADMIN01 "backup data create snapshot"" ...
HANA snapshot created successfully.
**********************Creating Storage snapshot**********************
Taking snapshot hourly.recent for hana_data_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for hana_log_backup_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for hana_log_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for hana_shared_lhanad01_t020_vol ...
Snapshot created successfully.
Taking snapshot hourly.recent for sapmnt_lhanad01_t020_vol ...
Snapshot created successfully.
**********************Deleting HANA snapshot**********************
Deleting the HANA snapshot with command: "./hdbsql -n localhost -i 01 -U SCADMIN01 "backup data drop snapshot"" ...
HANA snapshot deletion successfully.
```
您可以從此範例看到指令碼如何記錄 HANA 快照的建立。 在相應放大案例中，是在主要節點上起始此程序。 主要節點會在每個背景工作角色節點上開始同步建立 SAP HANA 快照集。 然後建立儲存體快照集。 順利執行儲存體快照之後，就會刪除 HANA 快照。 系統會從主要節點開始刪除 HANA 快照集。


**後續步驟**
- 請參閱[災害復原原則和準備](hana-concept-preparation.md)。