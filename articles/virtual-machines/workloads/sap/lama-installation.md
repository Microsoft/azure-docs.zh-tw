---
title: 適用於 Azure 的 SAP LaMa 連接器 | Microsoft Docs
description: 使用 SAP LaMa 在 Azure 上管理 SAP 系統
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: MSSedusch
manager: timlt
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 17/07/2018
ms.author: sedusch
ms.openlocfilehash: 2f3b8371357403071e70dd2e351cd75dbd34f746
ms.sourcegitcommit: d0ea925701e72755d0b62a903d4334a3980f2149
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/09/2018
ms.locfileid: "40007107"
---
# <a name="sap-lama-connector-for-azure"></a>適用於 Azure 的 SAP LaMa 連接器

[1877727]:https://launchpad.support.sap.com/#/notes/1877727
[2343511]:https://launchpad.support.sap.com/#/notes/2343511
[2350235]:https://launchpad.support.sap.com/#/notes/2350235
[2562184]:https://launchpad.support.sap.com/#/notes/2562184
[2628497]:https://launchpad.support.sap.com/#/notes/2628497
[2445033]:https://launchpad.support.sap.com/#/notes/2445033
[Logo_Linux]:media/virtual-machines-shared-sap-shared/Linux.png
[Logo_Windows]:media/virtual-machines-shared-sap-shared/Windows.png
[dbms-guide]:dbms-guide.md
[deployment-guide]:deployment-guide.md
[planning-guide]:planning-guide.md
[hana-ops-guide]:hana-vm-operations.md

> [!NOTE]
> 一般支援陳述式：如果您需要 SAP LaMa 或 Azure 連接器的支援，請一律在元件 BC-VCM-LVM-HYPERV 上開啟採用 SAP 的事件。

許多客戶會使用 SAP LaMa 來操作及監視其 SAP 系統環境。 從 SAP LaMa 3.0 SP05 起，預設會隨附 Azure 連接器。 您可以使用此連接器來解除配置並啟動虛擬機器、複製並重新放置受控磁碟，以及刪除受控磁碟。 透過這些基本作業，您可以使用 SAP LaMa 重新放置、複製 (copy)、複製 (clone) 及重新整理 SAP 系統。

本指南說明如何設定適用於 SAP LaMa 的 Azure 連接器、建立可用來安裝調適性 SAP 系統的虛擬機器，以及如何設定它們。

> [!NOTE]
> 此連接器僅適用於 SAP LaMa 企業版

## <a name="resources"></a>資源

下列 SAP 附註與 Azure 上的 SAP LaMa 主題相關︰

| 附註編號 | 標題 |
| --- | --- |
| [2343511] |適用於 SAP Landscape Management (LaMa) 的 Microsoft Azure 連接器 |
| [2350235] |SAP Landscape Management 3.0 - 企業版 |

另請閱讀 [SAP LaMa 的 SAP 說明入口網站](https://help.sap.com/viewer/p/SAP_LANDSCAPE_MANAGEMENT_ENTERPRISE)。

## <a name="general-remarks"></a>一般備註

* 務必在 [安裝程式]-> [設定]-> [引擎] 中啟用 [自動掛接點建立]  
  如果 SAP LaMa 在虛擬機器上使用 SAP Adaptive Extensions 掛接磁碟區，若未啟用此設定，則掛接點必須存在。

* 在部署新 VM 和取消準備 SAP 執行個體時，使用個別的子網路，請勿使用動態 IP 位址來防止 IP 位址「竊取」  
  如果您在子網路中使用動態 IP 位址配置 (也會由 SAP LaMa 使用)，則使用 SAP LaMa 準備 SAP 系統可能會失敗。 如果 SAP 系統毫無準備，則不會保留 IP 位址，並可能將其配置到其他虛擬機器。

* 如果您登入受控主機，切勿封鎖檔案系統，使其無法取消掛接  
  如果您登入 Linux 虛擬機器，並將工作目錄變更為掛接點中的目錄 (例如 /usr/sap/AH1/ASCS00/exe)，則無法取消掛接磁碟區，且重新放置或取消準備會失敗。

## <a name="set-up-azure-connector-for-sap-lama"></a>設定適用於 SAP LaMa 的 Azure 連接器

從 SAP LaMa 3.0 SP05 起隨附 Azure 連接器。 我們建議一律針安裝 SAP LaMa 3.0 的最新支援套件和修補程式。 Azure 連接器會使用服務主體來對 Microsoft Azure 授權。 請遵循下列步驟來建立適用於 SAP Landscape Management (LaMa) 的服務主體。

1. 移至 https://portal.azure.com。
1. 開啟 [Azure Active Directory] 刀鋒視窗
1. 按一下 [應用程式註冊]
1. 按一下 [新增]
1. 輸入名稱、選取應用程式類型 [Web 應用程式/API]、輸入登入 URL (例如 http://localhost))，然後按一下 [建立]
1. 登入 URL 並未使用，而且可以是任何有效的 URL
1. 選取新的應用程式，然後按一下 [設定] 索引標籤中的金鑰
1. 輸入新金鑰的說明、選取 [永不過期]，然後按一下 [儲存]
1. 記下值。 此值用來作為服務主體的密碼
1. 記下應用程式識別碼。 此值用來作為服務主體的使用者名稱

服務主體預設沒有存取您 Azure 資源的權限。 您必須為服務主體提供其存取權限。

1. 移至 https://portal.azure.com。
1. 開啟資源群組刀鋒視窗
1. 選取您要使用的資源群組
1. 按一下 [存取控制 (IAM)]
1. 按一下 [新增]
1. 選取 [參與者] 角色
1. 輸入您先前建立的應用程式名稱
1. 按一下 [確定]
1. 針對您要使用於 SAP LaMa 的所有資源群組重複步驟 3 到 8

開啟 SAP LaMa 網站並瀏覽至基礎結構。 移至 Cloud Managers 索引標籤，然後按一下 [新增]。 選取 Microsoft Azure 雲端配接器並且按 [下一步]。 輸入以下資訊：

* 標籤：選擇連接器執行個體的名稱
* 使用者名稱：服務主體的應用程式識別碼
* 密碼：服務主體金鑰/密碼
* URL：保留預設值 https://management.azure.com/
* 監視間隔 (秒)：應該至少 300
* 訂用帳戶識別碼：Azure 訂用帳戶識別碼
* Active Directory 租用戶識別碼：Active Directory 租用戶的識別碼
* Proxy 主機：如果 SAP LaMa 需要 Proxy 才能連線到網際網路，則為 Proxy 的主機名稱
* Proxy 連接埠：Proxy 的 TCP 連接埠

按一下 [測試組態] 以驗證您的輸入。 您應該會看到

連線成功：已成功連線到 Microsoft 雲端。 在網站底端找到 7 個資源群組 (只要求 10 個群組)

。

## <a name="provision-a-new-adaptive-sap-system"></a>佈建一個新的調適性 SAP 系統

您可以手動部署新的虛擬機器，或使用[快速入門存放庫](https://github.com/Azure/azure-quickstart-templates)中的其中一個 Azure 範本。 其中包含 [SAP NetWeaver ASCS](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-ascs)[SAP NetWeaver 應用程式伺服器](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-apps)和[資料庫](https://github.com/Azure/azure-quickstart-templates/tree/master/sap-lama-db)的範本。 您也可以使用這些範本，將新主機佈建為系統複製 (copy)/複製 (clone) 等的一部分。

在部署新虛擬機器和取消準備 SAP 執行個體時，我們建議針對您要透過 SAP LaMa 管理的所有虛擬機器使用個別的子網路，請勿使用動態 IP 位址來防止 IP 位址「竊取」。

> [!NOTE]
> 可能的話，請移除所有虛擬機器擴充功能，因為它們可能導致中斷磁碟與虛擬機器連結時的長時間執行階段。

確定使用者 \<hanasid>adm、\<sapsid>adm 和群組 sapsys 存在具有相同識別碼和 gid 或使用 LDAP 的目標電腦上。 在應該用來執行 SAP NetWeaver (A) SCS 的虛擬機器上，啟用並啟動 NFS 伺服器。

### <a name="manual-deployment"></a>手動部署

SAP LaMa 會使用 SAP Host Agent 來與虛擬機器通訊。 如果您手動部署虛擬機器，或者不是使用快速入門存放庫中的 Azure Resource Manager 範本進行部署，請務必安裝最新的 SAP Host Agent 和 SAP Adaptive Extensions。 如需適用於 Azure 的必要修補層級詳細資訊，請參閱 SAP 附註 [2343511]。

#### <a name="manual-deployment-of-a-linux-virtual-machine"></a>手動部署 Linux 虛擬機器

使用 SAP 附註 [2343511] 中所列的其中一個支援作業系統，建立新的虛擬機器。 為 SAP 執行個體新增額外的 IP 組態。 每個執行個體至少需要一個 IP 位址，而且必須使用虛擬主機名稱進行安裝。

SAP NetWeaver ASCS 執行個體需要可供 /sapmnt/\<SAPSID>、/usr/sap/\<SAPSID>、/usr/sap/trans 和 /usr/sap/\<sapsid>adm 使用的磁碟。 SAP NetWeaver 應用程式伺服器不需要額外的磁碟。 與 SAP 執行個體相關的所有項目必須儲存在 ASCS 上並透過 NFS 匯出。 否則，目前不可能使用 SAP LaMa 來新增其他應用程式伺服器。

![Linux 上的 SAP NetWeaver ASCS](media/lama/sap-lama-ascs-app-linux.png)

#### <a name="manual-deployment-for-sap-hana"></a>手動部署 SAP HANA

使用 SAP 附註 [2343511] 中針對 SAP HANA 所列的其中一個支援作業系統，建立新的虛擬機器。 為 SAP HANA 新增一個額外的 IP 組態，且每個 HANA 租用戶一個。

SAP HANA 需要 /hana/shared、/hana/backup、/hana/data 和 /hana/log 的磁碟

![Linux 上的 SAP HANA](media/lama/sap-lama-db-hana.png)

#### <a name="manual-deployment-for-oracle-database-on-linux"></a>在 Linux 上手動部署 Oracle Database

使用 SAP 附註 [2343511] 中針對 Oracle 資料庫所列的其中一個支援作業系統，建立新的虛擬機器。 為 Oracle Database 新增一個額外的 IP 組態。

Oracle Database 需要可供 /oracle、/home/oraod1 和 /home/oracle 使用的磁碟

![Linux 上的 Oracle Database](media/lama/sap-lama-db-ora-lnx.png)

#### <a name="manual-deployment-for-microsoft-sql-server"></a>手動部署 Microsoft SQL Server

使用 SAP 附註 [2343511] 中針對 Microsoft SQL Server 所列的其中一個支援作業系統，建立新的虛擬機器。 為 SQL Server 執行個體新增一個額外的 IP 組態。

SQL Server 資料庫伺服器需要可供資料庫資料和記錄檔使用的磁碟，以及可供 c:\usr\sap 使用的磁碟。

![Linux 上的 Oracle Database](media/lama/sap-lama-db-sql.png)

務必在您想要用於重新放置 SAP NetWeaver 應用程式伺服器或作為系統複製 (copy)/複製 (clone) 目標的虛擬機器上，安裝支援的 Microsoft ODBC Driver for SQL Server。

SAP LaMa 本身無法重新放置 SQL Server，因此您想要用於重新放置資料庫執行個體或作為系統複製 (copy)/複製 (clone) 目標的虛擬機器必須預先安裝 SQL Server。

### <a name="deploy-virtual-machine-using-an-azure-template"></a>使用 Azure 範本來部署虛擬機器

針對虛擬機器的作業系統，從 [SAP Software Marketplace](https://support.sap.com/swdc) 下載下列最新的可用封存檔：

1. SAPCAR 7.21
1. SAP HOST AGENT 7.21
1. SAP ADAPTIVE EXTENSION 1.0 EXT

此外，從 [Microsoft 下載中心](https://www.microsoft.com/download) 下載下列元件

1. Microsoft Visual C++ 2010 可轉散發套件 (x64) (僅限 Windows)
1. Microsoft ODBC Driver for SQL Server (僅限 SQL Server)

部署範本時需要這些元件。 將這些元件提供給範本使用的最簡單方式，就是將它們上傳至 Azure 儲存體帳戶並建立共用存取簽章 (SAS)。

範本具有下列參數：

* sapSystemId：SAP 系統識別碼。 它用來建立磁碟配置 (例如 /usr/sap/\<sapsid>)。

* computerName：新虛擬機器的電腦名稱。 SAP LaMa 也會使用這個參數。 當您使用此範本將新的虛擬機器佈建為系統複製(copy) 的一部分時，SAP LaMa 會等到可以觸達具有此電腦名稱的主機為止。

* osType：您想要部署的作業系統類型。

* dbtype：資料庫的類型。 這個參數用來判斷需要新增幾個額外的 IP 組態，以及磁碟配置的外觀。

* sapSystemSize：您要部署的 SAP 系統大小。 這個參數用來判斷虛擬機器執行個體類型和大小。

* adminUsername：虛擬機器的使用者名稱。

* adminPassword：虛擬機器的密碼。 您也可以提供 SSH 的公開金鑰。

* sshKeyData：虛擬機器的公開 SSH 金鑰。 只對 Linux 作業系統提供支援。

* subnetId：您想使用的子網路識別碼。

* deployEmptyTarget：如果您想要使用虛擬機器作為執行個體重新放置或類似的目標，您可以部署空的目標。 在此情況下，不會連結任何額外的磁碟或 IP 組態。

* sapcarLocation：符合您所部署作業系統的 sapcar 應用程式位置。 sapcar 用來解壓縮您在其他參數中提供的封存。

* sapHostAgentArchiveLocation：SAP Host Agent 封存的位置。 SAP Host Agent 會部署為此範本部署的一部分。

* sapacExtLocation：SAP Adaptive Extensions 的位置。 SAP 附註 [2343511] 會列出 Azure 所需的最低修補程式層級。

* vcRedistLocation:：需要有 VC 執行階段的位置，才能安裝 SAP Adaptive Extensions。 只有 Windows 需要這個參數。

* odbcDriverLocation：您想要安裝的 ODBC 驅動程式位置。 僅支援 Microsoft ODBC Driver for SQL Server。

* sapadmPassword：sapadm 使用者的密碼。

* sapadmId：sapadm 使用者的 Linux 使用者識別碼。 並非 Windows 的必要參數。

* sapsysGid：sapsys 群組的 Linux 群組識別碼。 並非 Windows 的必要參數。

* _artifactsLocation：基底 URI，此範本所需構件的所在位置。 使用隨附指令碼來部署範本時，將使用訂用帳戶中的私人位置，而且會自動產生此值。 只有在您未部署來自 GitHub 的範本時才需要。

* _artifactsLocationSasToken：存取 _artifactsLocation 所需的 sasToken。 使用隨附指令碼來部署範本時，將會自動產生 sasToken。 只有在您未部署來自 GitHub 的範本時才需要。

### <a name="sap-hana"></a>SAP HANA

在下列範例中，我們假設您以系統識別碼 HN1 安裝 SAP HANA，並以系統識別碼 AH1 安裝 SAP NetWeaver 系統。 HANA 執行個體的虛擬主機名稱為 hn1-db、SAP NetWeaver 系統所用 HANA 租用戶的虛擬主機名稱為 ah1-db、SAP NetWeaver ASCS 的虛擬主機名稱為 ah1-ascs，以及第一個 SAP NetWeaver 應用程式伺服器的虛擬主機名稱為 ah1-di-0。

#### <a name="install-sap-netweaver-ascs-for-sap-hana"></a>安裝 SAP NetWeaver ASCS for SAP HANA

啟動 SAP Software Provisioning Manager (SWPM) 之前，您必須掛接 ASCS 的虛擬主機名稱 IP 位址。 建議的方式是使用 sapacext。 如果您使用 sapacext 掛接 IP 位址，請務必在重新開機後重新掛接 IP 位址。

![Linux][Logo_Linux] Linux

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-ascs -n 255.255.255.128
```

![Windows][Logo_Windows] Windows

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h ah1-ascs -n 255.255.255.128
```

執行 SWPM 並使用 *ah1-ascs* 作為 [ASCS 執行個體主機名稱]。

![Linux][Logo_Linux] Linux  
將下列設定檔參數新增至 SAP Host Agent 設定檔 (位於 /usr/sap/hostctrl/exe/host_profile)。 如需詳細資訊，請參閱 SAP 附註 [2628497]。
```
acosprep/nfs_paths=/home/ah1adm,/usr/sap/trans,/sapmnt/AH1,/usr/sap/AH1
```

#### <a name="install-sap-hana"></a>安裝 SAP HANA

如果您使用命令列工具 hdblcm 安裝 SAP HANA，請使用參數 --hostname 來提供虛擬主機名稱。 您需要將資料庫的虛擬主機名稱 IP 位址新增至網路介面。 建議的方式是使用 sapacext。 如果您使用 sapacext 掛接 IP 位址，請務必在重新開機後重新掛接 IP 位址。

針對應用程式伺服器用來連線到 HANA 租用戶的名稱，新增另一個虛擬主機名稱和 IP 位址。

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h hn1-db -n 255.255.255.128
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-db -n 255.255.255.128
```

在應用程式伺服器虛擬機器 (而不是在 HANA 虛擬機器) 上執行 SWPM 的資料庫執行個體安裝。 在 [適用於 SAP 系統的資料庫] 對話方塊中，使用 *ah1-db* 作為 [資料庫主機]。

#### <a name="install-sap-netweaver-application-server-for-sap-hana"></a>安裝適用於 SAP HANA 的 SAP NetWeaver 應用程式伺服器

啟動 SAP Software Provisioning Manager (SWPM) 之前，您必須掛接應用程式伺服器的虛擬主機名稱 IP 位址。 建議的方式是使用 sapacext。 如果您使用 sapacext 掛接 IP 位址，請務必在重新開機後重新掛接 IP 位址。

![Linux][Logo_Linux] Linux

```bash
# /usr/sap/hostctrl/exe/sapacext -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
/usr/sap/hostctrl/exe/sapacext -a ifup -i eth0 -h ah1-di-0 -n 255.255.255.128
```

![Windows][Logo_Windows] Windows

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h ah1-di-0 -n 255.255.255.128
```

建議使用 SAP NetWeaver 設定檔參數 dbs/hdb/hdb_use_ident 來設定用於在 HDB 使用者存放區中尋找金鑰的身分識別。 在使用 SWPM 安裝資料庫執行個體或使用下列程式碼執行 SWPM 之後，您可以手動新增此參數：

```bash
# from https://blogs.sap.com/2015/04/14/sap-hana-client-software-different-ways-to-set-the-connectivity-data/
/sapdb/DVDs/IM_LINUX_X86_64/sapinst HDB_USE_IDENT=SYSTEM_COO
```

如果您手動設定它，則也需要建立新的 HDB 使用者存放區項目。

```bash
# run as <sapsid>adm
/usr/sap/AH1/hdbclient/hdbuserstore LIST
# reuse the port that was listed from the command above, in this example 35041
/usr/sap/AH1/hdbclient/hdbuserstore SET DEFAULT ah1-db:35041@AH1 SAPABAP1 <password>
```

在 [主要應用程式伺服器執行個體] 對話方塊中，使用 *ah1-di-0* 作為 [PAS 執行個體主機名稱]。

#### <a name="post-installation-steps-for-sap-hana"></a>SAP HANA 的後續安裝步驟

務必在您嘗試進行租用戶複製、租用戶移動或建立系統複寫之前，備份 SYSTEMDB 和所有的租用戶資料庫。

### <a name="microsoft-sql-server"></a>連接字串

在下列範例中，我們假設您以系統識別碼 AS1 安裝 SAP NetWeaver 系統。 SAP NetWeaver 系統所用 SQL Server 執行個體的虛擬主機名稱為 as1-db、SAP NetWeaver ASCS 的虛擬主機名稱為 as1-ascs，以及第一個 SAP NetWeaver 應用程式伺服器的虛擬主機名稱為 as1-di-0。

#### <a name="install-sap-netweaver-ascs-for-sql-server"></a>安裝 SAP NetWeaver ASCS for SQL Server

啟動 SAP Software Provisioning Manager (SWPM) 之前，您必須掛接 ASCS 的虛擬主機名稱 IP 位址。 建議的方式是使用 sapacext。 如果您使用 sapacext 掛接 IP 位址，請務必在重新開機後重新掛接 IP 位址。

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-ascs -n 255.255.255.128
```

執行 SWPM 並使用 *as1-ascs* 作為 [ASCS 執行個體主機名稱]。

#### <a name="install-sql-server"></a>安裝 SQL Server

您需要將資料庫的虛擬主機名稱 IP 位址新增至網路介面。 建議的方式是使用 sapacext。 如果您使用 sapacext 掛接 IP 位址，請務必在重新開機後重新掛接 IP 位址。

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-db -n 255.255.255.128
```

在 SQL Server 虛擬機器上執行 SWPM 的資料庫執行個體安裝。 使用 SAPINST_USE_HOSTNAME=*as1-db* 來覆寫用來連線至 SQL Server 的主機名稱。 如果您使用 Azure Resource Manager 範本部署虛擬機器，請務必將用於資料庫資料檔案的目錄設定為 *C:\sql\data* 並將資料庫記錄檔設定為 *C:\sql\log*。

確定使用者 *NT AUTHORITY\SYSTEM* 可存取 SQL Server 且具有 *sysadmin* 伺服器角色。 如需詳細資訊，請參閱 SAP 附註 [1877727] 和 [2562184]。

#### <a name="install-sap-netweaver-application-server"></a>安裝 SAP NetWeaver 應用程式伺服器

啟動 SAP Software Provisioning Manager (SWPM) 之前，您必須掛接應用程式伺服器的虛擬主機名稱 IP 位址。 建議的方式是使用 sapacext。 如果您使用 sapacext 掛接 IP 位址，請務必在重新開機後重新掛接 IP 位址。

```bash
# C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i <network interface> -h <virtual hostname or IP address> -n <subnet mask>
C:\Program Files\SAP\hostctrl\exe\sapacext.exe -a ifup -i "Ethernet 3" -h as1-di-0 -n 255.255.255.128
```

在 [主要應用程式伺服器執行個體] 對話方塊中，使用 *as1-di-0* 作為 [PAS 執行個體主機名稱]。

## <a name="troubleshooting"></a>疑難排解

### <a name="errors-and-warnings-during-discover"></a>探索期間的錯誤和警告

* 已拒絕 SELECT 權限
  * [Microsoft][ODBC SQL Server Driver][SQL Server] SELECT 權限在物件 'log_shipping_primary_databases'、資料庫 'msdb'、結構描述 'dbo' 上遭拒。 [SOAPFaultException]  
  SELECT 權限在物件 'log_shipping_primary_databases'、資料庫 'msdb'、結構描述 'dbo' 上遭拒。
  * 解決方法  
    確定 *NT AUTHORITY\SYSTEM* 可以存取 SQL Server。 請參閱 SAP 附註 [2562184]


### <a name="errors-and-warnings-for-instance-validation"></a>執行個體驗證的錯誤與警告

* HDB 使用者存放區驗證引發例外狀況  
  * 請參閱記錄檢視器  
    com.sap.nw.lm.aci.monitor.api.validation.RuntimeValidationException：識別碼為 'RuntimeHDBConnectionValidator' 的驗證程式發生例外狀況 (驗證：'VALIDATION_HDB_USERSTORE')：無法擷取 hdbuserstore  
    HANA 使用者存放區不在正確的位置
  * 解決方法  
    確定 /usr/sap/AH1/hdbclient/install/installation.ini 正確無誤

### <a name="errors-and-warnings-during-a-system-copy"></a>系統複製 (copy) 期間的錯誤和警告

* 驗證系統佈建步驟時發生錯誤
  * 原因：com.sap.nw.lm.aci.engine.base.api.util.exception.HAOperationException Calling '/usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r' | /usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r
  * 解決方法  
    備份來源 HANA 系統中的所有資料庫

* 資料庫執行個體的系統複製(copy) 步驟「啟動」
  * 主機代理程式作業 '000D3A282BC91EE8A1D76CF1F92E2944' 失敗 (OperationException。 FaultCode：'127'，訊息：「命令執行失敗。 ：[Microsoft][ODBC SQL Server Driver][SQL Server]使用者沒有更改資料庫 'AS2' 的全線，此資料庫不存在，或此資料庫不是處於允許存取檢查的狀態。」)
  * 解決方法  
    確定 *NT AUTHORITY\SYSTEM* 可以存取 SQL Server。 請參閱 SAP 附註 [2562184]

### <a name="errors-and-warnings-during-a-system-clone"></a>系統複製 (clone) 期間的錯誤和警告

* 嘗試在應用程式伺服器或 ASCS 的「強制註冊和啟動執行個體代理程式」步驟中註冊執行個體代理程式時發生錯誤
  * 嘗試註冊執行個體代理程式時發生錯誤。 (RemoteException：「無法從 '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0' 載入執行個體資料：無法存取設定檔 '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0」：沒有這類檔案或目錄。」)
  * 解決方法  
   確定在 ASCS/SCS 上的 sapmnt 共用具有 SAP_AS1_GlobalAdmin 的完整存取權

* 「啟用複製 (clone) 的啟動保護」步驟發生錯誤
  * 無法開啟檔案 '\\as1-ascs\sapmnt\AS1\SYS\profile\AS1_D00_as1-di-0' 原因：沒有這類檔案或目錄
  * 解決方法  
    應用程式伺服器的電腦帳戶需要設定檔的寫入權限

### <a name="errors-and-warnings-during-create-system-replication"></a>建立系統複寫期間的錯誤和警告

* 按一下 [建立系統複寫] 時發生例外狀況
  * 原因：com.sap.nw.lm.aci.engine.base.api.util.exception.HAOperationException   Calling '/usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r' | /usr/sap/hostctrl/exe/sapacext -a ShowHanaBackups -m HN1 -f 50 -h hn1-db -o level=0\;status=5\;port=35013 pf=/usr/sap/hostctrl/exe/host_profile -R -T dev_lvminfo -u SYSTEM -p hook -r
  * 解決方法  
    測試 sapacext 是否可以 `<hanasid`>adm 身分執行

* 未在儲存體步驟中啟用完整複製時發生錯誤
  * 報告路徑 IStorageCopyData.storageVolumeCopyList:1 和欄位 targetStorageSystemId 的內容屬性訊息時發生錯誤
  * 解決方法  
    忽略步驟中的警告並再試一次。 此問題將會在 SAP LaMa 的新支援套件/修補程式中修正。

### <a name="errors-and-warnings-during-relocate"></a>重新配置期間的錯誤和警告

* nfs reexports 不允許使用路徑 '/usr/sap/AH1'。
  * 如需詳細資料，請查看 SAP 附註 [2628497]。
  * 解決方法  
    將 ASCS 匯出新增至 ASCS HostAgent 設定檔。 請參閱 SAP 附註 [2628497]

* 重新配置 ASCS 時未實作函式
  * 命令輸出：exportfs: host:/usr/sap/AX1：未實作函式
  * 解決方法  
    確定 NFS 伺服器服務已在重新放置目標虛擬機器上啟用

### <a name="errors-and-warnings-during-application-server-installation"></a>應用程式伺服器安裝期間的錯誤和警告

* 執行 SAPinst 步驟時發生錯誤：getProfileDir
  * 錯誤：(最後一個步驟所回報的錯誤：在模型呼叫中攔截到 ESAPinstException：步驟 '|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_readProfileDir|ind|ind|ind|ind|readProfile|0|getProfileDir' 的驗證程式回報了一項錯誤：節點 \\\as1-ascs\sapmnt\AS1\SYS\profile 不存在。 以互動模式啟動 SAPinst 來解決此問題)
  * 解決方法  
    確定 SWPM 是以具有設定檔存取權的使用者執行。 在應用程式伺服器安裝精靈中可以設定此使用者

* 執行 SAPinst 步驟時發生錯誤：askUnicode
  * 錯誤：(最後一個步驟所回報的錯誤：在模型呼叫中攔截到 ESAPinstException：步驟 '|NW_DI|ind|ind|ind|ind|0|0|NW_GetSidFromProfiles|ind|ind|ind|ind|getSid|0|NW_getUnicode|ind|ind|ind|ind|unicode|0|askUnicode' 的驗證程式回報了一項錯誤：以互動模式啟動 SAPinst 以解決此問題)
  * 解決方法  
    如果您使用最新的 SAP 核心，SWPM 便無法再使用 ASCS 的郵件伺服器來判斷系統是否為 unicode 系統。 如需詳細資訊，請參閱 SAP 附註 [2445033]。  
    此問題將會在 SAP LaMa 的新支援套件/修補程式中修正。  
    在 SAP 系統的預設設定檔中設定設定檔參數 OS_UNICODE=uc，以解決此問題。

* 執行 SAPinst 步驟時發生錯誤：dCheckGivenServer
  * 執行 SAPinst 步驟時發生錯誤：dCheckGivenServer" version="1.0" 錯誤：(最後一個步驟所回報的錯誤：\<p > 使用者已取消安裝。 \</p>
  * 解決方法  
    確定 SWPM 是以具有設定檔存取權的使用者執行。 在應用程式伺服器安裝精靈中可以設定此使用者

* 執行 SAPinst 步驟時發生錯誤：checkClient
  * 執行 SAPinst 步驟時發生錯誤：checkClient" version="1.0" 錯誤：(此步驟所回報的最後一個錯誤：\<p> 使用者已取消安裝。 \</p>)
  * 解決方法  
    確定您要安裝應用程式伺服器的虛擬機器上已安裝 Microsoft ODBC Driver for SQL Server

* 執行 SAPinst 步驟時發生錯誤：copyScripts
  * 此步驟所回報的最後一個錯誤：系統呼叫失敗。 詳細資料：在使用參數 (\\\as1-ascs/sapmnt/AS1/SYS/exe/uc/NTAMD64/strdbs.cmd, w) 執行系統呼叫 'fopenU' 時，檔案 (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/filesystem/syxxcfstrm2.cpp) 的行 (494) 發生錯誤 13 (0x0000000d) (權限被拒)，堆疊追蹤：  
  CThrThread.cpp: 85: CThrThread::threadFunction()  
  CSiServiceSet.cpp: 63: CSiServiceSet::executeService()  
  CSiStepExecute.cpp: 913: CSiStepExecute::execute()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  iaxxcfile.cpp: 183: iastring CIaOsFileConnect::callMemberFunction(iastring const& name, args_t const& args)  
  iaxxcfile.cpp: 1849: iastring CIaOsFileConnect::newFileStream(args_t const& _args)  
  iaxxbfile.cpp: 773: CIaOsFile::newFileStream_impl(4)  
  syxxcfile.cpp: 233: CSyFileImpl::openStream(ISyFile::eFileOpenMode)  
  syxxcfstrm.cpp: 29: CSyFileStreamImpl::CSyFileStreamImpl(CSyFileStream*,iastring,ISyFile::eFileOpenMode)  
  syxxcfstrm.cpp: 265: CSyFileStreamImpl::open()  
  syxxcfstrm2.cpp: 58: CSyFileStream2Impl::CSyFileStream2Impl(const CSyPath & \\\aw1-ascs/sapmnt/AW1/SYS/exe/uc/NTAMD64/strdbs.cmd, 0x4)  
  syxxcfstrm2.cpp: 456: CSyFileStream2Impl::open()
  * 解決方法  
    確定 SWPM 是以具有設定檔存取權的使用者執行。 在應用程式伺服器安裝精靈中可以設定此使用者

* 執行 SAPinst 步驟時發生錯誤：askPasswords
  * 此步驟所回報的最後一個錯誤：系統呼叫失敗。 詳細資料：在使用參數 (...) 執行系統呼叫 'NetValidatePasswordPolicy' 時，檔案 (\bas/bas/749_REL/bc_749_REL/src/ins/SAPINST/impl/src/syslib/account/synxcaccmg.cpp) 的一行 (359) 發生錯誤 5 (0x00000005) (存取遭拒)，堆疊追蹤：  
  CThrThread.cpp: 85: CThrThread::threadFunction()  
  CSiServiceSet.cpp: 63: CSiServiceSet::executeService()  
  CSiStepExecute.cpp: 913: CSiStepExecute::execute()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  CSiStepExecute.cpp: 764: CSiStepExecute::invokeDialog()  
  DarkModeGuiEngine.cpp: 56: DarkModeGuiEngine::showDialogCalledByJs()  
  DarkModeDialog.cpp: 85: DarkModeDialog::submit()  
  EJSController.cpp: 179: EJSControllerImpl::executeScript()  
  JSExtension.hpp: 1136: CallFunctionBase::call()  
  iaxxcaccount.cpp: 107: iastring CIaOsAccountConnect::callMemberFunction(iastring const& name, args_t const& args)  
  iaxxcaccount.cpp: 1186: iastring CIaOsAccountConnect::validatePasswordPolicy(args_t const& _args)  
  iaxxbaccount.cpp: 430: CIaOsAccount::validatePasswordPolicy_impl()  
  synxcaccmg.cpp: 297: ISyAccountMgt::PasswordValidationMessage CSyAccountMgtImpl::validatePasswordPolicy(saponazure,*****) const )
  * 解決方法  
    務必在步驟 *Isolation* 中新增主機規則，以允許從 VM 至網域控制站的通訊

## <a name="next-steps"></a>後續步驟
* [Azure 上的 SAP Hana 作業指南][hana-ops-guide]
* [適用於 SAP 的 Azure 虛擬機器規劃和實作][planning-guide]
* [適用於 SAP 的 Azure 虛擬機器部署][deployment-guide]
* [適用於 SAP 的 Azure 虛擬機器 DBMS 部署][dbms-guide]