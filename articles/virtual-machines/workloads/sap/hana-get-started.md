---
title: 快速入門：在 Azure 虛擬機器上手動安裝單一執行個體 SAP HANA | Microsoft Docs
description: 在 Azure 虛擬機器上手動安裝單一執行個體 SAP HANA 的快速入門指南
services: virtual-machines-linux
documentationcenter: ''
author: hermanndms
manager: jeconnoc
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: c51a2a06-6e97-429b-a346-b433a785c9f0
ms.service: virtual-machines-linux
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 09/06/2018
ms.author: hermannd
ms.openlocfilehash: 6355a7ce203f2bf75b5c93d225502f961deeee43
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47032071"
---
# <a name="quickstart-manual-installation-of-single-instance-sap-hana-on-azure-vms"></a>快速入門：在 Azure VM 上手動安裝單一執行個體 SAP HANA
## <a name="introduction"></a>簡介
當您手動安裝 SAP NetWeaver 7.5 和 SAP HANA 1.0 SP12 時，本指南可協助您在 Azure 虛擬機器 (VM) 上設定單一執行個體的 SAP HANA。 本指南的重點是在 Azure 上部署 SAP HANA。 它不會取代 SAP 文件。 

>[!Note]
>本指南說明如何將 SAP HANA 部署到 Azure VM。 如需將 SAP HANA 部署至 HANA 大型執行個體上的資訊，請參閱[在 Azure 虛擬機器 (VM) 上使用 SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)。
 
## <a name="prerequisites"></a>必要條件
本指南假設您已熟悉如下的這類基礎結構即服務 (IaaS) 基本知識：
 * 如何透過 Azure 入口網站或 PowerShell 部署虛擬機器或虛擬網路。
 * Azure 跨平台命令列介面 (CLI)，包括使用 JavaScript 物件通知 (JSON) 範本的選項。

本指南也假設您已熟悉：
* SAP HANA 與 SAP NetWeaver，以及如何加以內部部署安裝。
* 安裝和操作 SAP HANA，以及在 Azure 上的 SAP 應用程式執行個體。
* 下列概念和程序：
   * 在 Azure 上規劃 SAP 部署，包括 Azure 虛擬網路規劃和 Azure 儲存體使用方式。 請參閱 [Azure 虛擬機器 (VM) 上的 SAP NetWeaver - 規劃和實作指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide)。
   * 部署原則及在 Azure 中部署 VM 的方式。 請參閱[適用於 SAP 的 Azure 虛擬機器部署](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/deployment-guide)。
   * Azure 上的 SAP NetWeaver ASCS (ABAP SAP 中央服務)、SCS (SAP 中央服務)，以及 ERS (將複寫伺服器加入佇列) 高可用性。 請參閱 [Azure VM 上的 SAP NetWeaver 高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide)。
   * 如何運用 Azure 上的 ASCS/SCS 多重 SID 安裝改善效率的詳細資料。 請參閱[建立 SAP NetWeaver 多 SID 組態](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-multi-sid)。 
   * Azure 中以 Linux 驅動的 VM 作為基礎執行 SAP NetWeaver 的準則。 請參閱[在 Microsoft Azure SUSE Linux VM 上執行 SAP NetWeaver](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/suse-quickstart)。 本指南提供 Azure VM 中 Linux 的特定設定，以及如何正確地將 Azure 儲存體磁碟連結至 Linux VM 的詳細資訊。

可以用於生產案例的 Azure VM 類型會列在 [IAAS 的 SAP 文件](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html)。 針對非實際執行案例，有更多種類的原生 Azure VM 類型可供使用。
如需 VM 組態和作業的詳細資訊，請參閱 [Azure 上的 SAP HANA 基礎結構組態和作業](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/hana-vm-operations)文件。
對於 SAP HANA 高可用性，請參閱 [Azure 虛擬機器的 SAP Hana 高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-overview)。

如果您想要取得 SAP HANA 執行個體或 S/4HANA，或是在非常快速時間內部署的 BW/4HANA 系統，應該考慮使用 [SAP 雲端應用裝置程式庫](http://cal.sap.com)。 例如，您可以在[本指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/cal-s4h)中找到在 Azure 上透過 SAP CAL 部署 S/4HANA 系統的相關文件。 您只需要具有可以向 SAP 雲端應用裝置程式庫註冊的 Azure 訂用帳戶和 SAP 使用者。

## <a name="additional-resources"></a>其他資源
### <a name="sap-hana-backup"></a>SAP HANA 備份
如需在 Azure VM 上備份 SAP HANA 資料庫的資訊，請參閱：
* [Azure 虛擬機器上的 SAP HANA 備份指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-guide)
* [檔案層級的 SAP HANA Azure 備份](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-file-level)
* [以儲存體快照集為基礎的 SAP HANA 備份](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-storage-snapshots)

### <a name="sap-cloud-appliance-library"></a>SAP 雲端應用裝置程式庫
如需使用 SAP 雲端應用裝置程式庫部署 S/4HANA 或 BW/4HANA 的資訊，請參閱[在 Microsoft Azure 上部署 SAP S/4HANA 或 BW/4HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/cal-s4h)。

### <a name="sap-hana-supported-operating-systems"></a>SAP HANA 支援的作業系統
如需 SAP HANA 支援作業系統的資訊，請參閱 [SAP 支援附註 #2235581 - SAP HANA︰支援的作業系統](https://launchpad.support.sap.com/#/notes/2235581/E)。 Azure VM 僅支援這些作業系統的其中一部份。 支援下列作業系統在 Azure 上部署 SAP HANA︰ 

* SUSE Linux Enterprise Server 12.x
* Red Hat Enterprise Linux 7.2

如需關於 SAP HANA 和不同 Linux 作業系統的其他 SAP 文件，請參閱：

* [SAP 支援附註 #171356 - 在 Linux 上的 SAP 軟體︰一般資訊](https://launchpad.support.sap.com/#/notes/1984787)
* [SAP 支援附註 #1944799 - SLES 作業系統安裝的 SAP HANA 指南](http://go.sap.com/documents/2016/05/e8705aae-717c-0010-82c7-eda71af511fa.html)
* [SAP 支援附註 #2205917 - 針對 SLES 12 for SAP 應用程式的 SAP HANA DB 建議作業系統設定](https://launchpad.support.sap.com/#/notes/2205917/E)
* [SAP 支援附註 #1984787 - SUSE Linux Enterprise Server 12：安裝注意事項](https://launchpad.support.sap.com/#/notes/1984787)
* [SAP 支援附註 #1391070 - Linux UUID 解決方案](https://launchpad.support.sap.com/#/notes/1391070)
* [SAP 支援附註 #2009879 - 適用於 Red Hat Enterprise Linux (RHEL) 作業系統的 SAP HANA 指南](https://launchpad.support.sap.com/#/notes/2009879)
* [2292690 - SAP HANA DB：適用於 RHEL 7 的建議作業系統設定](https://launchpad.support.sap.com/#/notes/2292690/E)

### <a name="sap-monitoring-in-azure"></a>Azure 中的 SAP 監視
如需 Azure 中之 SAP 監視的相關資訊，請參閱：

* [SAP 附註 2191498](https://launchpad.support.sap.com/#/notes/2191498/E)。 本附註討論對 Azure 上的 Linux VM 進行 SAP「增強型監視」。 
* [SAP 附註 1102124](https://launchpad.support.sap.com/#/notes/1102124/E)。 本附註討論 Linux 上之 SAPOSCOL 的相關資訊。 
* [SAP 附註 2178632](https://launchpad.support.sap.com/#/notes/2178632/E)。 本附註討論 Microsoft Azure 上的 SAP 主要監視計量。

### <a name="azure-vm-types"></a>Azure VM 類型
與 SAP HANA 搭配使用的 Azure VM 類型與SAP 支援的工作負載情節，都記載於 [SAP 認證的 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html)。 

經過 SAP 認證適用於 SAP NetWeaver 或 S/4HANA 應用程式層級的 Azure VM 類型，記載在 [SAP 附註 1928533 - Azure 上的 SAP 應用程式︰支援的產品和 Azure VM 類型](https://launchpad.support.sap.com/#/notes/1928533/E)。

>[!Note]
>只有 Azure Resource Manager 支援 SAP-Linux-Azure 整合，傳統部署模型並不支援。 

## <a name="manual-installation-of-sap-hana"></a>手動安裝 SAP HANA

> [!IMPORTANT]
> 請確定您選取的作業系統在您所使用的特定 VM 類型上已獲得 SAP HANA 認證。 在 [SAP HANA 認證 IaaS 平台](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) 中可查閱 SAP HANA 認證的 VM 類型和作業系統版本清單。 請務必按一下所列 VM 類型的詳細資料，以取得 SAP HANA 針對特定 VM 類型支援的作業系統版本完整清單。 請注意，在本文件的範例中，我們使用了 SAP for SAP HANA 在 M 系列 VM 上不支援的 SLES 作業系統版本。
>

本指南說明如何在 Azure VM 上，以下列兩種不同方式手動安裝 SAP HANA：

* 在「安裝資料庫執行個體」步驟的分散式 NetWeaver 安裝過程中使用 SAP 軟體佈建管理員 (SWPM)
* 使用 SAP HANA 資料庫生命週期管理員工具 HDBLCM，然後安裝 NetWeaver

您也可以使用 SWPM 在單一 VM 中安裝所有元件 (SAP HANA、SAP 應用程式伺服器、ASCS 執行個體)，如本 [SAP HANA 部落格通知](https://blogs.saphana.com/2013/12/31/announcement-sap-hana-and-sap-netweaver-as-abap-deployed-on-one-server-is-generally-available/)所述。 這個選項並未於本快速入門指南中說明，但您必須納入考量的問題都一樣。

開始安裝之前，建議您先閱讀本指南稍後的＜準備 Azure VM 進行 SAP HANA 手動安裝＞一節。 當您只使用預設的 Azure VM 設定時，這樣做有助於避免數個可能發生的基本錯誤。

## <a name="key-steps-for-sap-hana-installation-when-you-use-sap-swpm"></a>當您使用 SAP SWPM 時，SAP HANA 安裝的主要步驟
本節會列出當您使用 SAP SWPM 執行分散式 SAP NetWeaver 7.5 安裝時，適用於手動、單一執行個體 SAP HANA 安裝的主要步驟。 本指南稍後會以螢幕擷取畫面形式更詳細說明個別步驟。

1. 建立包含這兩個測試 VM 的 Azure 虛擬網路。
2. 根據 Azure Resource Manager 模型，在作業系統上 (在我們的範例中是 SUSE Linux Enterprise Server (SLES) 和 SLES for SAP Applications 12 SP1) 部署兩個 Azure VM。
3. 將兩個 Azure 標準或進階儲存體磁碟連結至應用程式伺服器 VM (例如 75-GB 或 500-GB 的磁碟)。
4. 將進階儲存體連結至 HANA DB 伺服器 VM。 如需詳細資訊，請參閱本指南稍後的＜磁碟設定＞一節。
5. 根據大小或輸送量需求連接多個磁碟，然後在 VM 內部的 OS 層級，使用邏輯磁碟區管理或多個裝置的系統管理工具 (MDADM) 來建立等量磁碟區。
6. 在連接的磁碟或邏輯磁碟區上建立 XFS 檔案系統。
7. 在 OS 層級上掛接新的 XFS 檔案系統。 將一個檔案系統用於所有 SAP 軟體。 例如，將其他檔案系統用於 /sapmnt 目錄和備份。 在 SAP HANA DB 伺服器的進階儲存體磁碟上，將 XFS 檔案系統掛接為 /hana 和 /usr/sap。 您必須執行此程序，才能防止 Linux Azure VM 上不算大的根目錄檔案系統遭到填滿。
8. 在 /etc/hosts 檔案中輸入測試 VM 的本機 IP 位址。
9. 在 /etc/fstab 檔案中輸入 **nofail** 參數。
10. 根據您使用的 Linux 作業系統版本，設定 Linux 核心參數。 如需詳細資訊，請參閱討論 HANA 的適當 SAP 附註，以及本指南的＜核心參數＞一節。
11. 新增交換空間。
12. (選擇性) 在測試 VM 上安裝圖形化桌面。 否則，請使用遠端 SAPinst 安裝。
13. 從 SAP Service Marketplace 下載 SAP 軟體。
14. 在應用程式伺服器 VM 上安裝 SAP ASCS 執行個體。
15. 使用 NFS，在測試 VM 之間共用 /sapmnt 目錄。 應用程式伺服器 VM 為 NFS 伺服器。
16. 在 DB 伺服器 VM 上，使用 SWPM 來安裝資料庫執行個體 (包括 HANA)。
17. 在應用程式伺服器 VM 上，安裝主要應用程式伺服器 (PAS)。
18. 啟動 SAP 管理主控台 (SAP MC)。 例如，連線到 SAP GUI 或 HANA Studio。

## <a name="key-steps-for-sap-hana-installation-when-you-use-hdblcm"></a>當您使用 HDBLCM 時，SAP HANA 安裝的主要步驟
本節會列出當您使用 SAP HDBLCM 執行分散式 SAP NetWeaver 7.5 安裝時，適用於手動、單一執行個體 SAP HANA 安裝的主要步驟。 整份指南會以螢幕擷取畫面形式更詳細說明個別步驟。

1. 建立包含這兩個測試 VM 的 Azure 虛擬網路。
2. 根據 Azure Resource Manager 模型，在作業系統上 (在我們的範例中是 SLES 和 SLES for SAP Applications 12 SP1) 部署兩個 Azure VM。
3. 將兩個 Azure 標準或進階儲存體磁碟連結至應用程式伺服器 VM (例如 75-GB 或 500-GB 的磁碟)。
4. 將進階儲存體連結至 HANA DB 伺服器 VM。 如需詳細資訊，請參閱本指南稍後的＜磁碟設定＞一節。
5. 根據大小或輸送量需求連接多個磁碟，然後在 VM 內部的 OS 層級，使用邏輯磁碟區管理或多個裝置的系統管理工具 (MDADM) 來建立等量磁碟區。
6. 在連接的磁碟或邏輯磁碟區上建立 XFS 檔案系統。
7. 在 OS 層級上掛接新的 XFS 檔案系統。 將一個檔案系統用於所有 SAP 軟體，並將另一個檔案系統用於 /sapmnt 目錄 (舉例說明) 及備份。 在 SAP HANA DB 伺服器的進階儲存體磁碟上，將 XFS 檔案系統掛接為 /hana 和 /usr/sap。 您必須執行此程序，以協助防止 Linux Azure VM 上不算大的根目錄檔案系統遭到填滿。
8. 在 /etc/hosts 檔案中輸入測試 VM 的本機 IP 位址。
9. 在 /etc/fstab 檔案中輸入 **nofail** 參數。
10. 根據您使用的 Linux 作業系統版本，設定核心參數。 如需詳細資訊，請參閱討論 HANA 的適當 SAP 附註，以及本指南的＜核心參數＞一節。
11. 新增交換空間。
12. (選擇性) 在測試 VM 上安裝圖形化桌面。 否則，請使用遠端 SAPinst 安裝。
13. 從 SAP Service Marketplace 下載 SAP 軟體。
14. 在 HANA DB 伺服器 VM 上，建立群組識別碼為 1001 的群組 "sapsys"。
15. 使用 HANA 資料庫生命週期管理員 (HDBLCM)，在 DB 伺服器 VM 上安裝 SAP HANA。
16. 在應用程式伺服器 VM 上安裝 SAP ASCS 執行個體。
17. 使用 NFS，在測試 VM 之間共用 /sapmnt 目錄。 應用程式伺服器 VM 為 NFS 伺服器。
18. 在 HANA DB 伺服器 VM 上，使用 SWPM 安裝資料庫執行個體 (包括 HANA)。
19. 在應用程式伺服器 VM 上，安裝主要應用程式伺服器 (PAS)。
20. 啟動 SAP MC。 透過 SAP GUI 或 HANA Studio 連線。

## <a name="preparing-azure-vms-for-a-manual-installation-of-sap-hana"></a>準備 Azure VM 以便手動安裝 SAP HANA
本節包含下列主題：

* OS 更新
* 磁碟設定
* 核心參數
* 檔案系統
* /etc/hosts 檔案
* /etc/fstab 檔案

### <a name="os-updates"></a>OS 更新
請先檢查 Linux 作業系統更新和修正程式，然後再安裝其他軟體。 安裝修補程式可能讓您避免呼叫支援人員。

請確定您是使用︰
* 適用於 SAP 應用程式的 SUSE Linux Enterprise Server。
* 適用於 SAP 應用程式的 Red Hat Enterprise Linux 或適用於 SAP HANA 的 Red Hat Enterprise Linux。 

可以透過 Linux 廠商提供的 Linux 訂用帳戶登錄 OS 部署 (如果您還沒這樣做)。 請注意，SUSE 具有適用於 SAP 應用程式的 OS 映像，已經包含服務且會自動註冊。

以下是使用 **zypper** 命令檢查 SUSE Linux 可用修補程式的範例：

 `sudo zypper list-patches`

視問題的種類而定，修補程式會依分類和嚴重性歸類。 常用的分類值為：**security (安全性)**、**recommended (建議)**、**optional (選用)**、**feature (功能)**、**document (文件)** 或 **yast**。
常用的嚴重性值為：**critical (嚴重)**、**important (重要)**、**moderate (中)**、**low (低)** 或 **unspecified (未指定)**。

**Zypper** 命令只會尋找已安裝套件所需要的更新。 例如，您可以使用此命令：

`sudo zypper patch  --category=security,recommended --severity=critical,important`

您可以新增 `--dry-run` 參數來測試更新，而不需實際更新系統。


### <a name="disk-setup"></a>磁碟設定
Azure 上 Linux VM 中的根目錄檔案系統有大小限制。 因此，必須將額外的磁碟空間連結至 Azure VM 才能執行 SAP。 若是 SAP 應用程式伺服器 Azure VM，使用 Azure 標準儲存體磁碟的使用量可能就已經夠用。 不過，若是 SAP HANA DBMS Azure VM，使用 Azure 進階儲存體磁碟用於生產與非生產實作的使用量是必要的。

根據 [SAP HANA TDI 儲存體需求](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html)，建議使用下列的 Azure 進階儲存體組態︰ 

| VM SKU | RAM |  /hana/data 和 /hana/log <br /> 與 LVM 或 MDADM 等量 | HANA/shared | /root volume | /usr/sap |
| --- | --- | --- | --- | --- | --- |
| GS5 | 448 GB | 2 x P30 | 1 x P20 | 1 x P10 | 1 x P10 | 

在建議的磁碟組態中，HANA 資料磁碟區和記錄磁碟區會放在相同的 Azure 進階儲存體磁碟中，而此種磁碟會與 LVM 或 MDADM 等量。 不需要定義任何 RAID 備援層級，因為 Azure 進階儲存體基於備援會保留磁碟的三個映像。 若要確定您設定足夠的儲存體，請參閱 [SAP HANA TDI 儲存體需求](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html)和 [SAP HANA Server 安裝與更新指南](http://help.sap.com/saphelp_hanaplatform/helpdata/en/4c/24d332a37b4a3caad3e634f9900a45/frameset.htm)。 也請考慮不同的 Azure 進階儲存體磁碟的不同虛擬硬碟 (VHD) 輸送量，如 [VM 高效能進階儲存體與受控磁碟](https://docs.microsoft.com/azure/storage/storage-premium-storage)中所述。 

您可以將額外的進階儲存體磁碟新增至 HANA DBMS VM，用於儲存資料庫或交易記錄備份。

如需這兩個用於設定等量的主要工具詳細資訊，請參閱下列文章：

* [在 Linux 上設定軟體 RAID](../../linux/configure-raid.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [設定 Azure 中 Linux VM 的 LVM](../../linux/configure-lvm.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

針對將磁碟連結至執行 Linux 客體 OS 的 Azure VM，如需詳細資訊請參閱[將磁碟新增至 Linux VM](../../linux/add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)。

Azure 進階儲存體可讓您定義磁碟快取模式。 針對存放 /hana/data 和 /hana/log 的等量集，應停用磁碟快取。 針對其他磁碟區 (磁碟)，快取模式應設為**唯讀**。

如需詳細資訊，請參閱[進階儲存體：Azure 虛擬機器工作負載適用的高效能儲存體](../../windows/premium-storage.md)。

若要尋找用於建立 VM 的範例 JSON 範本，請移至 [Azure 快速入門範本 (英文)](https://github.com/Azure/azure-quickstart-templates)。
vm-simple-sles 範本是基本的範本。 它包含儲存體區段與其他 100 GB 的資料磁碟。 此範本可用來當做基底。 您可以針對特定的組態採用範本。

>[!Note]
>請務必使用 UUID 來連結 Azure 儲存體磁碟，如[在 Microsoft Azure SUSE Linux VM 上執行 SAP NetWeaver](suse-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) 中所述。

在測試環境中，已將兩個 Azure 標準儲存體磁碟連接至 SAP 應用程式伺服器 VM，如下列螢幕擷取畫面所示。 其中一個磁碟用來儲存所有可供安裝的 SAP 軟體 (包括 NetWeaver 7.5、SAP GUI 和 SAP HANA)。 第二個磁碟確保有足夠的可用空間可用於其他需求 (例如，備份和測試資料)，以及要在所有屬於同一個 SAP 環境的 VM 之間共用的 /sapmnt 目錄 (也就是 SAP 設定檔)。

![SAP HANA 應用程式伺服器的 [磁碟] 視窗，其中顯示兩個資料磁碟及其大小](./media/hana-get-started/image003.jpg)


### <a name="kernel-parameters"></a>核心參數
SAP HANA 需要不屬於標準 Azure 資源庫映像且必須手動設定的特定 Linux 核心設定。 依據您是使用 SUSE 還是 Red Hat，參數可能不一樣。 稍早列出的 SAP 附註中提供關於這些參數的資訊。 在顯示的螢幕擷取畫面中，使用 SUSE Linux 12 SP1。 

SLES for SAP Applications 12 GA 和 SLES for SAP Applications 12 SP1 具有一個可取代舊 **sapconf** 工具的新工具 **tuned-adm**。 **tuned-adm** 有特別的 SAP HANA 設定檔可供其使用。 若要針對 SAP HANA 調整系統，請輸入下列內容作為根使用者：

   `tuned-adm profile sap-hana`

如需有關 **tuned-adm** 的詳細資訊，請參閱[關於 tuned-adm 的 SUSE 文件](https://www.suse.com/documentation/sles-for-sap-12/pdfdoc/sles-for-sap-12-sp1.zip)。

在下列螢幕擷取畫面中，您可以看到 **tuned-adm** 如何根據必要的 SAP HANA 設定來變更 `transparent_hugepage` 和 `numa_balancing` 值。

![tuned-adm 工具會根據必要的 SAP HANA 設定來變更值](./media/hana-get-started/image005.jpg)

若要永久保留 SAP HANA 核心設定，請在 SLES 12 上使用 **grub2**。 如需 **grub2** 的詳細資訊，請移至 SUSE 文件的[＜組態檔結構＞](https://www.suse.com/documentation/sles-for-sap-12/pdfdoc/sles-for-sap-12-sp1.zip)一節。

下列螢幕擷取畫面顯示如何在組態檔中變更核心設定，然後使用 **grub2-mkconfig** 進行編譯：

![在組態檔中變更核心設定，然後使用 grub2-mkconfig 進行編譯](./media/hana-get-started/image006.jpg)

另一個選項是使用 YaST 和 [開機載入器] > [核心參數] 設定來變更這些設定：

![YaST 開機載入器中的 [核心參數] 設定索引標籤](./media/hana-get-started/image007.jpg)

### <a name="file-systems"></a>檔案系統
下列螢幕擷取畫面顯示兩個檔案系統，這兩者均建立於這兩個連接之 Azure 標準儲存體磁碟上的 SAP 應用程式伺服器 VM 上。 這兩個檔案系統都屬 XFS 類型，且掛接至 /sapdata 和 /sapsoftware。

您不一定要使用這種方式來建構檔案系統。 您有其他選項可用來建構磁碟空間。 最重要的考量是防止根目錄檔案系統的可用空間用盡。

![在 SAP 應用程式伺服器 VM 上建立兩個檔案系統](./media/hana-get-started/image008.jpg)

關於 SAP HANA DB VM，在資料庫安裝期間，當您使用 SAPinst (SWPM) 和**標準**安裝選項時，會在 /hana 和 /usr/sap 之下安裝所有東西。 SAP HANA 記錄備份的預設位置是在 /usr/sap 之下。 同樣地，因為必須防止根檔案系統的可用空間用盡，所以在使用 SWPM 安裝 SAP HANA 之前，請先確定 /hana 和 /usr/sap 之下有足夠的可用空間。

如需 SAP HANA 的標準檔案系統配置說明，請參閱 [SAP HANA 伺服器安裝與更新指南 (英文)](http://help.sap.com/saphelp_hanaplatform/helpdata/en/4c/24d332a37b4a3caad3e634f9900a45/frameset.htm)。

![在 SAP 應用程式伺服器 VM 上建立的其他檔案系統](./media/hana-get-started/image009.jpg)

當您在標準 SLES/SLES for SAP Applications 12 Azure 資源庫映像上安裝 SAP NetWeaver 時，會顯示一則指出沒有交換空間的訊息，如下列螢幕擷取畫面所示。 若要關閉此訊息，您可以使用 **dd**、**mkswap** 和 **swapon** 手動新增分頁檔。 若要了解做法，請在 SUSE 文件的[＜使用 YaST Partitioner＞](https://www.suse.com/documentation/sles-for-sap-12/pdfdoc/sles-for-sap-12-sp1.zip)一節中搜尋「手動新增分頁檔」。

另一個選項是使用 Linux VM 代理程式設定交換空間。 如需詳細資訊，請參閱 [Azure Linux 代理程式使用者指南](../../extensions/agent-linux.md)。

![指出沒有足夠交換空間的快顯訊息](./media/hana-get-started/image010.jpg)


### <a name="the-etchosts-file"></a>/etc/hosts 檔案
開始安裝 SAP 前，請確定在 /etc/hosts 檔案中包含 SAP VM 的主機名稱和 IP 位址。 在一個 Azure 虛擬網路內部署所有的 SAP VM，然後使用內部 IP 位址，如下所示：

![SAP VM 的主機名稱和 IP 位址會列於 /etc/hosts 檔案中](./media/hana-get-started/image011.jpg)

### <a name="the-etcfstab-file"></a>/etc/fstab 檔案

將 **nofail** 參數新增至 fstab 檔案很實用。 如此一來，如果磁碟發生錯誤，VM 不會在開機程序中停止回應。 但請記得，可能無法取得額外的磁碟空間，而且程序可能會填滿根目錄檔案系統。 如果遺失 /hana，SAP HANA 將無法啟動。

![將 nofail 參數新增至 fstab 檔案](./media/hana-get-started/image000c.jpg)

## <a name="graphical-gnome-desktop-on-sles-12sles-for-sap-applications-12"></a>SLES 12/SLES for SAP Applications 12 上的圖形化 GNOME 桌面
本節包含下列主題：

* 在 xrdp on SLES 12/SLES for SAP Applications 12 上安裝 GNOME 桌面和 xrdp
* 在 SLES 12/SLES for SAP Applications 12 上使用 Firefox 來執行以 Java 為基礎的 SAP MC

您也可以使用替代項目，例如 Xterminal 或 VNC (不在本指南中說明)。

### <a name="installing-the-gnome-desktop-and-xrdp-on-sles-12sles-for-sap-applications-12"></a>在 xrdp on SLES 12/SLES for SAP Applications 12 上安裝 GNOME 桌面和 xrdp
如果您具備 Windows 背景，就可以輕鬆地在 SAP Linux VM 內，直接使用圖形化桌面來執行 Firefox、SAPinst、SAP GUI、SAP MC 或 HANA Studio，並從 Windows 電腦透過遠端桌面通訊協定 (RDP) 連線到 VM。 依貴公司關於將圖形化使用者介面新增到以 Linux 為基礎的生產環境與非生產環境系統之原則而定，您將需要在伺服器上安裝 GNOME。 若要在 Azure SLES 12/SLES for SAP Applications 12 VM 上安裝 GNOME 桌面：

1. 輸入下列命令 (例如在 PuTTY 視窗中) 來安裝 GNOME 桌面：

   `zypper in -t pattern gnome-basic`

2. 安裝 xrdp 以允許透過 RDP 連接到 VM：

   `zypper in xrdp`

3. 編輯 /etc/sysconfig/windowmanager，並將預設的視窗管理員設定為 GNOME：

   `DEFAULT_WM="gnome"`

4. 執行 **chkconfig** 以確保 xrdp 會在重新開機後自動啟動：

   `chkconfig -level 3 xrdp on`

5. 如果發生 RDP 連線問題，請嘗試重新啟動 (例如，從 PuTTY 視窗)：

   `/etc/xrdp/xrdp.sh restart`

6. 如果上一個步驟中所述的 xrdp 重新啟動無法運作，請檢查 .pid 檔案：

   `check /var/run` 

   尋找 `xrdp.pid`。 如果找到的話，請將它移除，並嘗試重新啟動。

### <a name="starting-sap-mc"></a>啟動 SAP MC
在安裝 GNOME 桌面之後，在 Azure SLES 12/SLES for SAP Applications 12 VM 中執行時，從 Firefox 啟動圖形化以 Linux 為基礎的 SAP MC 可能會因為遺失 Java 瀏覽器外掛程式而顯示錯誤。

啟動 SAP MC 的 URL 為 `<server>:5<instance_number>13`。

如需詳細資訊，請參閱[啟動 Web 型 SAP 管理主控台 (英文)](https://help.sap.com/saphelp_nwce10/helpdata/en/48/6b7c6178dc4f93e10000000a42189d/frameset.htm)。

下列螢幕擷取畫面顯示遺失 Java 瀏覽器外掛程式時所顯示的錯誤訊息：

![指出遺失 Java 瀏覽器外掛程式的錯誤訊息](./media/hana-get-started/image013.jpg)

解決這個問題的其中一種方式，是使用 YaST 安裝遺失的外掛程式，如下列螢幕擷取畫面所示：

![使用 YaST 安裝遺失的外掛程式](./media/hana-get-started/image014.jpg)

當您重新輸入 SAP 管理主控台 URL 時，會出現訊息要求您啟用外掛程式：

![要求啟用外掛程式的對話方塊](./media/hana-get-started/image015.jpg)

您可能也會收到關於遺失檔案 (javafx.properties) 的錯誤訊息。 這與「適用於 SAP GUI 7.4 的 Oracle Java 1.8」的需求有關。 (請參閱 [SAP 附註 2059429](https://launchpad.support.sap.com/#/notes/2059424)。)IBM Java 版本或 SLES/SLES for SAP Applications 12 隨附的 openjdk 套件皆未包含所需的 javafx.properties 檔案。 解決方案是從 Oracle 下載 Java SE 8。

如需與 OpenSUSE 上 openjdk 類似問題的相關資訊，請參閱討論區對話 [SAPGui 7.4 Java for openSUSE 42.1 Leap](https://scn.sap.com/thread/3908306)。

## <a name="manual-installation-of-sap-hana-swpm"></a>手動安裝 SAP HANA：SWPM
本節中一系列的螢幕擷取畫面顯示當您使用 SWPM (SAPinst) 時，安裝 SAP NetWeaver 7.5 和 SAP HANA SP12 的主要步驟。 在 NetWeaver 7.5 安裝過程中，SWPM 也可將 HANA 資料庫安裝為單一執行個體。

在範例測試環境中，我們只安裝了一部進階商業應用程式程式設計 (ABAP) 應用程式伺服器。 如下列螢幕擷取畫面所示，我們使用了 [分散式系統] 選項，在一個 Azure VM 上安裝 ASCS 和主要應用程式伺服器執行個體，並在另一個 Azure VM 上安裝 SAP HANA 作為資料庫系統。

![使用 [分散式系統] 選項安裝 ASCS 和主要應用程式伺服器執行個體](./media/hana-get-started/image012.jpg)

將 ASCS 執行個體安裝於應用程式伺服器 VM 上，並在 SAP 管理主控台 (如下列螢幕擷取畫面所顯示) 中設定為「綠色」之後，必須將包含 SAP 設定檔目錄的 /sapmnt 目錄與 SAP HANA DB 伺服器 VM 共用。 DB 安裝步驟需要存取此資訊。 提供存取的最佳方式是使用可利用 YaST 設定的 NFS。

![顯示 ASCS 執行個體已安裝於應用程式伺服器 VM 上並設定為「綠色」的 SAP 管理主控台](./media/hana-get-started/image016.jpg)

在應用程式伺服器 VM 上，應透過 NFS 使用 **rw** 和 **no_root_squash** 選項來共用 /sapmnt 目錄。 預設值為 **ro** 和 **root_squash**，這可能導致安裝資料庫執行個體時發生問題。

![透過 NFS 使用 rw 和 no_root_squash 選項共用 /sapmnt 目錄](./media/hana-get-started/image017b.jpg)

如下列螢幕擷取畫面所示，必須在 SAP HANA DB 伺服器 VM 上使用 **NFS 用戶端** (和 YaST)，來設定應用程式伺服器 VM 中的 /sapmnt 共用。

![使用 NFS 用戶端來設定 /sapmnt 共用](./media/hana-get-started/image018b.jpg)

為了如下列螢幕擷取畫面所示執行分散式 NetWeaver 7.5 安裝 (**資料庫執行個體**)，會登入 SAP HANA 資料庫伺服器 VM 並啟動 SWPM。

![登入 SAP HANA DB 伺服器 VM，然後啟動 SWPM，藉以安裝資料庫執行個體](./media/hana-get-started/image019.jpg)

選取**標準**安裝和安裝媒體的路徑之後，請輸入 DB SID、主機名稱、執行個體號碼及 DB 系統管理員密碼。

![SAP HANA 資料庫系統管理員登入頁面](./media/hana-get-started/image035b.jpg)

輸入 DBACOCKPIT 結構描述的密碼：

![DBACOCKPIT 結構描述的密碼輸入方塊](./media/hana-get-started/image036b.jpg)

輸入 SAPABAP1 結構描述密碼的問題：

![輸入 SAPABAP1 結構描述密碼的問題](./media/hana-get-started/image037b.jpg)

完成每個工作之後，就會在 DB 安裝程序的每個階段旁邊顯示綠色核取記號。 訊息「執行...資料庫執行個體已完成。」隨即顯示。

![含有確認訊息的完成工作視窗](./media/hana-get-started/image023.jpg)

安裝成功後，SAP 管理主控台也應該將 DB 執行個體顯示為「綠色」，並顯示完整的 SAP HANA 程序清單 (hdbindexserver、hdbcompileserver 等)。

![含有 SAP HANA 程序清單的 SAP 管理主控台視窗](./media/hana-get-started/image024.jpg)

下列螢幕擷取畫面會顯示 SWPM 在 HANA 安裝期間於 /hana/shared 目錄下所建立的部分檔案結構。 因為沒有任何選項可指定不同的路徑，所以請務必在安裝 SAP HANA 之前，使用 SWPM 在 /hana 目錄之下掛接額外的磁碟空間。 這可以避免根檔案系統的可用空間不足。

![在 HANA 安裝期間建立 /hana/shared 目錄檔案結構](./media/hana-get-started/image025.jpg)

這個螢幕擷取畫面會顯示 /usr/sap 目錄的檔案結構：

![/usr/sap 目錄檔案結構](./media/hana-get-started/image026.jpg)

分散式 ABAP 安裝的最後一個步驟，是安裝主要應用程式伺服器執行個體：

![將主要應用程式伺服器執行個體顯示為最後一個步驟的 ABAP 安裝](./media/hana-get-started/image027b.jpg)

安裝主要應用程式伺服器執行個體和 SAP GUI 之後，使用 **DBA Cockpit** 交易來確認已正確完成 SAP HANA 安裝：

![確認安裝成功的 DBA Cockpit 視窗](./media/hana-get-started/image028b.jpg)

在最後一個步驟中，您將需要先在 SAP 應用程式伺服器 VM 中安裝 HANA Studio，然後連線至 DB 伺服器 VM 上執行的 SAP HANA 執行個體：

![在 SAP 應用程式伺服器 VM 中安裝 SAP HANA Studio](./media/hana-get-started/image038b.jpg)

## <a name="manual-installation-of-sap-hana-hdblcm"></a>手動安裝 SAP HANA：HDBLCM
除了使用 SWPM 在分散式安裝過程中安裝 SAP HANA 之外，您還能使用 HDBLCM，先獨立安裝 HANA。 例如，您接著可以安裝 SAP NetWeaver 7.5。 本節中的螢幕擷取畫面會顯示此程序的運作方式。

如需有關 HANA HDBLCM 工具的詳細資訊，請參閱：

* [為您的工作選擇正確的 SAP HANA HDBLCM](https://help.sap.com/saphelp_hanaplatform/helpdata/en/68/5cff570bb745d48c0ab6d50123ca60/content.htm)
* [SAP HANA 生命週期管理工具](http://saphanatutorial.com/sap-hana-lifecycle-management-tools/)
* [SAP HANA 伺服器安裝與更新指南](http://help.sap.com/hana/SAP_HANA_Server_Installation_Guide_en.pdf)

為了避免 `\<HANA SID\>adm user` (由 HDBLCM 工具建立) 的預設群組識別碼設定發生問題，請在透過 HDBLCM 安裝 SAP HANA 之前，先使用群組識別碼 `1001` 定義名為 `sapsys` 的新群組：

![使用群組識別碼 1001 定義的新群組 "sapsys"](./media/hana-get-started/image030.jpg)

當您第一次啟動 HDBLCM 時，會顯示簡易的開始功能表。 選取項目 1 [安裝新的系統]，如下列螢幕擷取畫面所示：

![HDBLCM 開始視窗中的 [安裝新的系統] 選項](./media/hana-get-started/image031.jpg)

下列螢幕擷取畫面顯示您先前選取的所有重要選項。

> [!IMPORTANT]
> 針對 HANA 記錄和資料磁碟區命名的目錄以及安裝路徑 (此範例中的 /hana/shared) 不應為根目錄檔案系統的一部分。 這些目錄屬於連結至 VM 的 Azure 資料磁碟 (如＜磁碟安裝＞小節中所述)。 這個方法有助於避免根檔案系統的空間不足。 在下列螢幕擷取畫面中，您可以看到 HANA 系統管理員具有使用者識別碼 `1005`，並且屬於安裝前所定義的 `sapsys` 群組 (識別碼 `1001`)。

![先前選取的所有重要 SAP HANA 元件清單](./media/hana-get-started/image032.jpg)

您可以在 /etc/hosts passwd 目錄中檢查`\<HANA SID\>adm user` (下列螢幕擷取畫面中的 `azdadm`) 的詳細資料：

![HANA \<HANA SID\>adm 使用者詳細資料會列於 /etc/passwd 目錄中](./media/hana-get-started/image033.jpg)

使用 HDBLCM 安裝 SAP HANA 之後，您就能在 SAP HANA Studio 中看見檔案結構，如下列螢幕擷取畫面所示。 尚未提供 SAPABAP1 結構描述 (其中包括所有的 SAP NetWeaver 資料表)。

![SAP HANA Studio 中的 SAP HANA 檔案結構](./media/hana-get-started/image034.jpg)

安裝 SAP HANA 之後，就能在其上安裝 SAP NetWeaver。 如下列螢幕擷取畫面所示，已使用 SWPM 來將安裝作為分散式安裝執行 (如上一節所述)。 當您使用 SWPM 安裝資料庫執行個體時，可以使用 HDBLCM 輸入相同的資料 (例如，主機名稱、HANA SID 和執行個體號碼)。 SWPM 接著會使用現有的 HANA 安裝，並加入多個結構描述。

![使用 SWPM 執行分散式安裝](./media/hana-get-started/image035b.jpg)

下列螢幕擷取畫面會顯示 SWPM 安裝步驟，您可以在其中輸入 DBACOCKPIT 結構描述的相關資料：

![輸入 DBACOCKPIT 結構描述資料的 SWPM 安裝步驟](./media/hana-get-started/image036b.jpg)

輸入 SAPABAP1 結構描述的相關資料：

![輸入 SAPABAP1 結構描述的相關資料](./media/hana-get-started/image037b.jpg)

完成 SWPM 資料庫執行個體安裝後，就能在 SAP HANA Studio 中看見 SAPABAP1 結構描述：

![SAP HANA Studio 中的 SAPABAP1 結構描述](./media/hana-get-started/image038b.jpg)

最後，在完成 SAP 應用程式伺服器和 SAP GUI 安裝後，您就能使用 **DBA Cockpit** 交易來驗證 HANA DB 執行個體：

![使用 DBA Cockpit 來交易驗證 HANA DB 執行個體](./media/hana-get-started/image039b.jpg)


## <a name="sap-software-downloads"></a>SAP 軟體下載
您可以從 SAP Service Marketplace 下載軟體，如下列螢幕擷取畫面所示。

下載適用於 Linux/HANA 的 NetWeaver 7.5：

 ![用於下載 NetWeaver 7.5 的 SAP 服務安裝和升級視窗](./media/hana-get-started/image001.jpg)

下載 HANA SP12 平台版本：

 ![用於下載 HANA SP12 平台版本的 SAP 服務安裝和升級視窗](./media/hana-get-started/image002.jpg)

