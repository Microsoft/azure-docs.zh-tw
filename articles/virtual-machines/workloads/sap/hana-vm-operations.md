---
title: SAP HANA 在 Azure 上的基礎結構設定和作業 | Microsoft Docs
description: 部署在 Azure 虛擬機器上之 SAP HANA 系統的操作指南。
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: juergent
manager: patfilot
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/27/2018
ms.author: msjuergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: db2d7fbe395a6d7e332d79183a331b45f7767f51
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47434053"
---
# <a name="sap-hana-infrastructure-configurations-and-operations-on-azure"></a>SAP HANA 在 Azure 上的基礎結構設定和作業
此文件提供設定 Azure 基礎結構和已部署在 Azure 原生虛擬機器 (VM) 上之 SAP Hana 系統的作業指導方針。 此文件也包含 M128s VM SKU 的 SAP HANA 相應放大設定資訊。 這份文件並非用以取代標準 SAP 文件，包含下列內容：

- [SAP 管理指南](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/330e5550b09d4f0f8b6cceb14a64cd22.html)
- [SAP 安裝指南](https://service.sap.com/instguides)
- [SAP 附註](https://sservice.sap.com/notes)

## <a name="prerequisites"></a>必要條件
若要使用本指南，您需要下列 Azure 元件的基本知識：

- [Azure 虛擬機器](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-manage-vm)
- [Azure 網路和虛擬網路](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-virtual-network)
- [Azure 儲存體](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-manage-disks)

若要了解有關 SAP NetWeaver 及 Azure 上其他 SAP 元件的詳細資訊，請參閱 [Azure 文件](https://docs.microsoft.com/azure/)的 [Azure 上的 SAP](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/get-started)。

## <a name="basic-setup-considerations"></a>基本設定考量
下列各節說明在 Azure VM 上部署 SAP HANA 系統的基本安裝考量。

### <a name="connect-into-azure-virtual-machines"></a>連線到 Azure 虛擬機器
如 [Azure 虛擬機器規劃指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/planning-guide)中所述，有兩種基本方法可連線到 Azure VM：

- 透過網際網路和跳接 VM 上的公用端點進行連線，或是在執行 SAP Hana 的 VM 上進行連線。
- 透過 [VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal) 或 Azure [ExpressRoute](https://azure.microsoft.com/services/expressroute/) 進行連線。

生產案例需要透過 VPN 或 ExpressRoute 的站對站連線能力。 饋送至使用 SAP 軟體之生產案例中的非生產案例也需要這種類型的連線。 下圖顯示跨站台連線能力的範例：

![跨站台連線能力](media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png)


### <a name="choose-azure-vm-types"></a>選擇 Azure VM 類型
可以用於生產案例的 Azure VM 類型會列在 [IAAS 的 SAP 文件](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html)。 針對非實際執行案例，有更多種類的原生 Azure VM 類型可供使用。

>[!NOTE]
> 針對非生產案例，請使用 [SAP 附註 #1928533](https://launchpad.support.sap.com/#/notes/1928533) 中所列的 VM 類型。 如需了解適用於生產案例的 Azure 虛擬機器使用方式，請在 SAP 發佈的[認證 IaaS 平台清單](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure)中查看經 SAP HANA 認證的虛擬機器。

使用下列方式在 Azure 中部署 VM：

- Azure 入口網站。
- Azure PowerShell Cmdlet。
- Azure CLI。

您也可以透過 [SAP Cloud Platform](https://cal.sap.com/)，將完整安裝的 SAP Hana 平台部署到 Azure VM 服務上。 [在 Azure 上部署 SAP S/4HANA 或 BW/4HANA](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/cal-s4h) 中會敘述安裝程序，或者使用[這裡](https://github.com/AzureCAT-GSI/SAP-HANA-ARM)發行的自動化安裝。

### <a name="choose-azure-storage-type"></a>選擇 Azure 儲存體類型
Azure 針對執行 SAP Hana 的 Azure VM，提供兩種儲存體類型：

- [Azure 標準儲存體](https://docs.microsoft.com/azure/virtual-machines/windows/standard-storage)
- [Azure 進階儲存體](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage)

Azure 針對 Azure 標準儲存體和 Azure 進階儲存體上的 VHD，提供兩種部署方法。 在整體情況允許的情況下，請利用 [Azure 受控磁碟](https://azure.microsoft.com/services/managed-disks/)部署。

如需儲存體類型清單及其在 IOPS 和儲存體輸送量中的 SLA ，請檢閱[受控磁碟的 Azure 文件](https://azure.microsoft.com/pricing/details/managed-disks/)。

### <a name="configuring-the-storage-for-azure-virtual-machines"></a>設定 Azure 虛擬機器的儲存體

目前您購買的 SAP HANA 設備是用於內部部署，所以您不需要在意 I/O 子系統及其功能。 原因是設備廠商必須確認最低儲存體需求符合 SAP HANA。 當您自行建置 Azure 基礎結構時，您應該對這些需求有些了解。 此外，也要了解我們在以下各節中建議的設定需求。 或者，針對您要設定虛擬機器來執行 SAP HANA 的情況。 要求的某些特性會導致以下需求：

- 在 **/hana/log** 上啟用 250MB/秒、最少具有 1 MB I/O 大小的讀取/寫入磁碟區
- 針對 **/hana/data** 啟用至少 400MB/秒、16MB 和 64MB I/O 大小的讀取活動
- 針對 **/hana/data** 啟用至少 250MB/秒、16MB 和 64MB I/O 大小的寫入活動

指定低儲存體延遲對於 DBMS 系統相當重要，即使是像 SAP HANA 那些 DBMS 將資料保存在記憶體內部也一樣。 儲存體中的關鍵路徑通常是在 DBMS 系統之交易記錄寫入的周圍。 但像是寫入儲存點或是在損毀復原之後載入記憶體內部資料之類的作業也很重要。 因此，針對 **/hana/data** 和 **/hana/log** 磁碟區必須使用 Azure 進階磁碟。 為了達到如 SAP 想要的 **/hana/log** 和 **/hana/data** 最小輸送量，您必須使用 MDADM 或 LVM 在多個 Azure 進階儲存體磁碟上建置 RAID 0。 然後使用 RAID 磁碟區作為 **/hana/data** 和 **/hana/log** 磁碟區。 對於 RAID 0 的等量磁碟大小，建議使用：

- 針對 **/hana/log** 使用 64 KB或 128 KB
- 針對 **/hana/log** 使用 32 KB

> [!NOTE]
> 您不需要使用 RAID 磁碟區設定任何備援層級，因為 Azure 進階和標準儲存體會保存三個 VHD 的映像。 RAID 磁碟區的使用方式單純是用來設定會提供足夠 I/O 輸送量的磁碟區。

RAID 下的 Azure VHD 數目累計，是從 IOPS 和儲存體輸送量端累計。 因此，如果您將 RAID 0 放在 3 x P30 Azure 進階儲存體磁碟上，它應該會給您單一 Azure 進階儲存體 P30 磁碟的 3 倍 IOPS 和 3 倍儲存體輸送量。

下列快取建議假設 SAP HANA 的 I/O 特性如下所示：

- 幾乎沒有任何針對 HANA 資料檔案的讀取工作負載。 例外是在資料載入 HANA 時，HANA 執行個體重新啟動之後的大型 I/O。 針對資料檔案讀取較大 I/O 的另一種情況可能是 HANA 資料庫備份。 因此，讀取快取大多不合理，因為在大部分的情況下，必須完整讀取所有資料檔案磁碟區。
- 當 HANA 儲存點和 HANA 損毀復原時，對資料檔案的寫入會發生高載。 寫入儲存點是非同步的，且不會造成任何使用者交易延遲。 因為系統必須再次快速回應，所以在損毀復原期間的資料寫入效能極為重要。 不過，損毀復原應該不是例外狀況
- 幾乎不會從 HANA 重做檔案進行任何讀取。 例外是在執行交易記錄備份、損毀復原，或 HANA 執行個體重新啟動階段的大型 I/O。  
- 對於 SAP HANA 重做記錄檔的主要負載是寫入。 根據工作負載的性質，您的 I/O 可以小到 4 KB，而在其他情況下，I/O 大小也可以是 1 MB 或更大。 對 SAP HANA 重做記錄的寫入延遲效能極為重要。
- 所有寫入都必須以可靠的方式保存在磁碟上

由 SAP HANA 觀察到的這些 I/O 模式結果，使用 Azure 進階儲存體對不同磁碟區的快取設定應該設定如下：

- **/hana/data**：無快取
- **/hana/log**：無快取 - M 系列例外 (請參閱本文件後面部分)
- **/hana/shared**：讀取快取


當調整 VM 大小或決定 VM 時，也請注意整體 VM I/O 輸送量。 [記憶體最佳化的虛擬機器大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory)一文中說明整體虛擬機器儲存體輸送量。

#### <a name="cost-conscious-azure-storage-configuration"></a>節省成本的 Azure 儲存體組態
下表顯示客戶經常用來在 Azure 虛擬機器上裝載 SAP HANA 的虛擬機器類型組態。 可能有部分虛擬機器類型無法符合 SAP HANA 的所有最小準則。 但是目前這些虛擬機器似乎可以針對非生產案例正常執行。 

> [!NOTE]
> 針對生產案例，請在 [IAAS 的 SAP 文件](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html)中檢查 SAP 是否有支援 SAP HANA 的特定虛擬機器類型。


| VM SKU | RAM | 最大 VM I/O<br /> Throughput | /hana/data 和 /hana/log<br /> 與 LVM 或 MDADM 等量 | HANA/shared | /root volume | /usr/sap | hana/backup |
| --- | --- | --- | --- | --- | --- | --- | -- |
| DS14v2 | 128 GB | 768 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S15 |
| E16v3 | 128 GB | 384 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S15 |
| E32v3 | 256 GiB | 768 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S20 |
| E64v3 | 443 GiB | 1200 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S30 |
| GS5 | 448 GiB | 2000 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S30 |
| M32ts | 192 GiB | 500 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S20 |
| M32ls | 256 GiB | 500 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 | 1 x S20 |
| M64ls | 512 GB | 1000 MB/秒 | 3 x P20 | 1 x S20 | 1 x S6 | 1 x S6 |1 x S30 |
| M64s | 1000 GiB | 1000 MB/秒 | 2 x P30 | 1 x S30 | 1 x S6 | 1 x S6 |2 x S30 |
| M64ms | 1750 GiB | 1000 MB/秒 | 3 x P30 | 1 x S30 | 1 x S6 | 1 x S6 | 3 x S30 |
| M128s | 2000 GiB | 2000 MB/秒 |3 x P30 | 1 x S30 | 1 x S6 | 1 x S6 | 2 x S40 |
| M128ms | 3800 GiB | 2000 MB/秒 | 5 x P30 | 1 x S30 | 1 x S6 | 1 x S6 | 2 x S50 |


根據 [SAP TDI 儲存體白皮書](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html)，包含 3 個 P20 的較小 VM 類型所建議使用的磁碟，在空間建議方面超過磁碟區大小。 不過，已如資料表中顯示進行選擇，為 SAP Hana 提供足夠的磁碟輸送量。 如果您需要對 **/hana/backup** 磁碟區 (已調整大小來保持備份為記憶體磁碟區的兩倍) 進行變更，您可以自由調整。   
請針對不同的建議磁碟區，檢查儲存體輸送量是否符合您想要執行的工作負載。 如果工作負載需要更多 **/hana/data** 和 **/hana/log** 磁碟區，您需要增加 Azure 進階儲存體 VHD 的數目。 使用比列出數目更多的 VHD 來調整磁碟區大小，會增加在 Azure 虛擬機器類型限制內的 IOPS 和 I/O 輸送量。 

> [!NOTE]
> 上述組態「不會」受益於 [Azure 虛擬機器的單一虛擬機器 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_6/)，因為它是使用 Azure 進階儲存體和 Azure 標準儲存體的混合。 不過，為了最佳化成本而選擇了這個選項。


#### <a name="azure-storage-configuration-to-benefit-for-meeting-single-vm-sla"></a>符合單一虛擬機器 SLA 而獲益的 Azure 儲存體組態
如果您想要從 [Azure 虛擬機器的單一虛擬機器 SLA](https://azure.microsoft.com/support/legal/sla/virtual-machines/v1_6/) 獲益，您必須單獨使用 Azure 進階儲存體 VHD。

> [!NOTE]
> 針對生產案例，請在 [IAAS 的 SAP 文件](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html)中檢查 SAP 是否有支援 SAP HANA 的特定虛擬機器類型。

| VM SKU | RAM | 最大 VM I/O<br /> Throughput | /hana/data 和 /hana/log<br /> 與 LVM 或 MDADM 等量 | HANA/shared | /root volume | /usr/sap | hana/backup |
| --- | --- | --- | --- | --- | --- | --- | -- |
| DS14v2 | 128 GB | 768 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P15 |
| E16v3 | 128 GB | 384 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P15 |
| E32v3 | 256 GiB | 768 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P20 |
| E64v3 | 443 GiB | 1200 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P30 |
| GS5 | 448 GiB | 2000 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P30 |
| M32ts | 192 GiB | 500 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P20 |
| M32ls | 256 GiB | 500 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P20 |
| M64ls | 512 GB | 1000 MB/秒 | 3 x P20 | 1 x P20 | 1 x P6 | 1 x P6 | 1 x P30 |
| M64s | 1000 GiB | 1000 MB/秒 | 2 x P30 | 1 x P30 | 1 x P6 | 1 x P6 |2 x P30 |
| M64ms | 1750 GiB | 1000 MB/秒 | 3 x P30 | 1 x P30 | 1 x P6 | 1 x P6 | 3 x P30 |
| M128s | 2000 GiB | 2000 MB/秒 |3 x P30 | 1 x P30 | 1 x P6 | 1 x P6 | 2 x P40 |
| M128ms | 3800 GiB | 2000 MB/秒 | 5 x P30 | 1 x P30 | 1 x P6 | 1 x P6 | 2 x P50 |


根據 [SAP TDI 儲存體白皮書](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html)，包含 3 個 P20 的較小 VM 類型所建議使用的磁碟，在空間建議方面超過磁碟區大小。 不過，已如資料表中顯示進行選擇，為 SAP Hana 提供足夠的磁碟輸送量。 如果您需要對 **/hana/backup** 磁碟區 (已調整大小來保持備份為記憶體磁碟區的兩倍) 進行變更，您可以自由調整。  
請針對不同的建議磁碟區，檢查儲存體輸送量是否符合您想要執行的工作負載。 如果工作負載需要更多 **/hana/data** 和 **/hana/log** 磁碟區，您需要增加 Azure 進階儲存體 VHD 的數目。 使用比列出數目更多的 VHD 來調整磁碟區大小，會增加在 Azure 虛擬機器類型限制內的 IOPS 和 I/O 輸送量。 




#### <a name="storage-solution-with-azure-write-accelerator-for-azure-m-series-virtual-machines"></a>儲存體解決方案，使用適用於 Azure M 系列虛擬機器的 Azure Write Accelerator
Azure Write Accelerator 是專門為 M 系列虛擬機器推出的功能。 如同名稱所示，這個功能的目的是改善針對 Azure 進階儲存體之寫入的 I/O 延遲。 針對 SAP HANA，Write Accelerator 只支援用於 **/hana/log** 磁碟區。 因此目前顯示的組態需要變更。 主要變更是 **/hana/data** 與 **/hana/log** 之間的備份，以便只針對 **/hana/log** 磁碟區使用 Azure Write Accelerator。 

> [!IMPORTANT]
> Azure M 系列虛擬機器的 SAP HANA 憑證是專用於 **/hana/log** 磁碟區的 Azure Write Accelerator。 因此，在 Azure M 系列虛擬機器上部署生產案例的 SAP HANA 時，預期會針對 **/hana/log** 磁碟區使用 Azure Write Accelerator 進行設定。  

> [!NOTE]
> 針對生產案例，請在 [IAAS 的 SAP 文件](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html)中檢查 SAP 是否有支援 SAP HANA 的特定虛擬機器類型。

建議的組態看起來如下所示：

| VM SKU | RAM | 最大 VM I/O<br /> Throughput | /hana/data | /hana/log | HANA/shared | /root volume | /usr/sap | hana/backup |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| M32ts | 192 GiB | 500 MB/秒 | 3 x P20 | 2 x P20 | 1 x P20 | 1 x P6 | 1 x P6 |1 x P20 |
| M32ls | 256 GiB | 500 MB/秒 | 3 x P20 | 2 x P20 | 1 x P20 | 1 x P6 | 1 x P6 |1 x P20 |
| M64ls | 512 GB | 1000 MB/秒 | 3 x P20 | 2 x P20 | 1 x P20 | 1 x P6 | 1 x P6 |1 x P30 |
| M64s | 1000 GiB | 1000 MB/秒 | 4 x P20 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 |2 x P30 |
| M64ms | 1750 GiB | 1000 MB/秒 | 3 x P30 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 | 3 x P30 |
| M128s | 2000 GiB | 2000 MB/秒 |3 x P30 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 | 2 x P40 |
| M128ms | 3800 GiB | 2000 MB/秒 | 5 x P30 | 2 x P20 | 1 x P30 | 1 x P6 | 1 x P6 | 2 x P50 |

請針對不同的建議磁碟區，檢查儲存體輸送量是否符合您想要執行的工作負載。 如果工作負載需要更多 **/hana/data** 和 **/hana/log** 磁碟區，您需要增加 Azure 進階儲存體 VHD 的數目。 使用比列出數目更多的 VHD 來調整磁碟區大小，會增加在 Azure 虛擬機器類型限制內的 IOPS 和 I/O 輸送量。

Azure 寫入加速器只能與 [Azure 受控磁碟](https://azure.microsoft.com/services/managed-disks/)搭配運作。 所以至少必須將組成 **/hana/log** 磁碟區的 Azure 進階儲存體磁碟部署為受控磁碟。

Azure 寫入加速器可以支援之每個虛擬機器的 Azure 進階儲存體 VHD 有其上限。 目前的上限是：

- M128xx VM 上限 16 個 VHD
- M64xx VM 上限 8 個 VHD
- M32xx VM 上限 4 個 VHD

如需如何啟用 Azure Write Accelerator 的詳細指示，可以在[寫入加速器](https://docs.microsoft.com/azure/virtual-machines/linux/how-to-enable-write-accelerator)一文中找到。

Azure Write Accelerator 的詳細資料和限制可以在相同文件中找到。

> [!NOTE]
> 所述磁碟組態建議是以 SAP 對其基礎結構提供者所表示的最低需求為目標。 在實際的客戶部署和工作負載案例中，還是會遇到這些建議仍未能提供足夠功能的情形。 這些情況可能是客戶在 HANA 重新啟動之後需要更快的資料重新載入速度，或者備份組態需要儲存體的更高頻寬。 其他案例包括 **/hana/log**，其中 5000 IOPS 對於特定工作負載來說不足。 因此請將這些建議當作起點，並且根據工作負載的需求來調整。
>  

### <a name="set-up-azure-virtual-networks"></a>設定 Azure 虛擬網路
當您有透過 VPN 或 ExpressRoute 對 Azure 的站對站連線時，必須至少有一個 Azure 虛擬網路，可透過虛擬閘道連線到 VPN 或 ExpressRoute 線路。 在簡單部署中，也可以將虛擬閘道部署在裝載 SAP HANA 執行個體的 Azure 虛擬網路 (VNet) 子網路中。 若要安裝 SAP HANA，您可以在 Azure 虛擬網路內建立兩個額外子網路。 一個子網路會裝載執行 SAP HANA 執行個體的 VM。 另一個子網路會執行裝載 SAP HANA Studio、其他管理軟體或應用程式軟體的 Jumpbox 或管理 VM。

當您安裝執行 SAP HANA 的 VM 時，VM 將會需要：

- 已安裝兩個虛擬 NIC：其中一個 NIC 會連線到管理子網路，另一個 NIC 則會從內部部署網路或其他網路，連線到 Azure VM 中的 SAP Hana 執行個體。
- 已針對這兩個虛擬 NIC 部署的靜態私人 IP 位址。

> [!NOTE]
> 您應該透過 Azure 方式將靜態 IP 位址指派給個別 vNIC。 您不應該將客體作業系統內的靜態 IP 位址指派給 vNIC。 某些 Azure 服務 (例如 Azure 備份服務) 依賴至少將主要 vNIC 設為 DHCP 的事實，而不是設為靜態 IP 位址。 另請參閱 [Azure 虛擬機器備份的疑難排解](https://docs.microsoft.com/azure/backup/backup-azure-vms-troubleshoot#networking)文件。 如果您需要將多個靜態 IP 位址指派給 VM，必須將多個 vNIC 指派給 VM。
>
>

不過，對於持續進行的部署，您需要在 Azure 中建立虛擬資料中心網路架構。 此架構建議將會連線到內部部署的 Azure VNet 閘道分隔在個別的 Azure VNet。 此個別 VNet 應主控要送至內部部署或網際網路的所有流量。 這種方法可讓您在此個別的中樞 VNet 部署軟體，稽核和記錄進入 Azure 虛擬資料中心的流量。 因此，您會有一個 VNet，裝載與 Azure 部署的傳入和傳出流量相關的所有軟體和設定。

[Azure 虛擬資料中心：網路觀點](https://docs.microsoft.com/azure/architecture/vdc/networking-virtual-datacenter)與 [Azure 虛擬資料中心和 Enterprise 控制平面](https://docs.microsoft.com/azure/architecture/vdc/)等文章，可為虛擬資料中心方法與相關 Azure VNet 設計提供詳細資訊。


>[!NOTE]
>使用 [Azure VNet 同儕節點](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)在中樞 VNet 與輪輻 VNet 之間流動的流量，則是額外[成本](https://azure.microsoft.com/pricing/details/virtual-network/)。 根據這些成本，您可能需要在執行嚴格中樞與輪輻網路設計，以及執行多個 [Azure ExpressRoute 閘道](https://docs.microsoft.com/azure/expressroute/expressroute-about-virtual-network-gateways)，連線到「輪輻」以略過 VNet 同儕節點，這兩者之間考量折衷辦法。 不過，Azure ExpressRoute 閘道也會產生額外[成本](https://azure.microsoft.com/pricing/details/vpn-gateway/)。 您用來進行網路流量記錄、稽核及監視的協力廠商軟體，也可能會產生額外成本。 取決於透過 VNet 同儕節點和另一端進行資料交換的成本，以及其他 Azure ExpressRoute 閘道與額外軟體授權產生的成本，您可以決定在一個 VNet 進行微型分割時，要使用子網路作為隔離單位，而不是 VNet。


如需以不同方法指派 IP 位址的概觀，請參閱 [Azure 中的 IP 位址類型及配置方法](https://docs.microsoft.com/azure/virtual-network/virtual-network-ip-addresses-overview-arm)。 

針對執行 SAP HANA 的虛擬機器，您應該使用指派的靜態 IP 位址。 原因是 HANA 的某些組態屬性會參考 IP 位址。

[Azure 網路安全性群組 (NSG)](https://docs.microsoft.com/azure/virtual-network/virtual-networks-nsg) 是用來將路由傳送至 SAP HANA 執行個體或 Jumpbox 的流量進行導向。 NSG 和[應用程式安全性群組](https://docs.microsoft.com/azure/virtual-network/security-overview#application-security-groups)最終都會與 SAP HANA 子網路和管理子網路相關聯。

下圖顯示遵循中樞與輪輻 VNet 架構的 SAP HANA 粗略部署結構描述概觀：

![SAP Hana 粗略部署結構描述](media/hana-vm-operations/hana-simple-networking.PNG)

若要在 Azure 中部署不具站台對站台連線能力的 SAP HANA，您仍想要保護 SAP HANA 執行個體不受公用網際網路危害，並隱藏在正向 Proxy 後方。 在這個基本案例中，部署會仰賴 Azure 的內建 DNS 服務來解析主機名稱。 在使用公眾對應 IP 位址的較複雜部署中，Azure 內建的 DNS 服務特別重要。 使用 Azure NSG 與 [Azure NVA](https://azure.microsoft.com/solutions/network-appliances/) 來控制、監視從網際網路到 Azure 中 Azure VNet 架構的路由。 下圖顯示以中樞與輪輻 VNet 架構部署不具站對站連線能力的 SAP HANA 概略結構描述：
  
![不具站台對站台連線能力的 SAP Hana 粗略部署結構描述](media/hana-vm-operations/hana-simple-networking2.PNG)
 

另一個如何使用 Azure NVA 以控制和監視來自網際網路的存取，而不使用中樞與輪輻 VNet 架構的描述，請參閱[部署高可用性的網路虛擬設備](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha)一文。


## <a name="configuring-azure-infrastructure-for-sap-hana-scale-out"></a>設定 SAP HANA 相應放大的 Azure 基礎結構
Microsoft 有一個已通過認證的 M 系列 VM SKU，可用於 SAP HANA 相應放大設定。 VM 類型 M128s 已通過認證，可相應放大至高達 16 個節點。 如需 Azure VM 上 SAP HANA 相應放大認證的變更，請查看[認證 IaaS 平台清單](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure) \(英文\)。

在 Azure VM 中部署相應放大設定的最低 OS 版本如下：

- SUSE Linux 12 SP3
- Red Hat Linux 7.4

16 個節點的相應放大認證

- 一個節點為主要節點
- 最多可有 15 個節點為背景工作角色節點

>[!NOTE]
>在 Azure VM 相應放大部署中，無法使用待命節點
>

無法設定待命節點的原因有兩個層面：

- Azure 目前沒有任何原生的 NFS 服務。 因此，需要協力廠商功能的說協助才能設定 NFS 共用。
- 針對解決方案部署在 Azure 上的解決方案，沒有任何協力廠商 NFS 設定可以滿足 SAP HANA 的儲存體延遲準則。

因此，無法共用 **/hana/data** 與 **hana/log** 磁碟區。 不共用單一節點的這些磁碟區，造成在相應放大設定中無法使用 SAP HANA 待命節點。

因此，使用相應放大設定的單一節點基本設定，如下所示：

![單一節點的相應放大基本概念](media/hana-vm-operations/scale-out-basics.PNG)

SAP HANA 相應放大的 VM 節點基本設定看起來像這樣：

- 針對 **/hana/shared**，您能以 SUSE Linux 12 SP3 為基礎建置高可用性 NFS 叢集。 此叢集會裝載相應放大設定的 **/hana/shared** NFS 共用以及 SAP NetWeaver 或 BW/4HANA 中央服務。 建置這類設定的文件可在[適用於 SUSE Linux Enterprise Server 之 Azure VM 上 NFS 的高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs)一文中找到
- 所有其他磁碟區都**不會**在不同節點間共用，也**不是**以 NFS 為基礎。 使用非共用 **/hana/data** 與 **hana/log** 以相應放大 HANA 安裝的安裝設定與步驟，都會在本文中進一步提供。

>[!NOTE]
>如圖中所示的高可用性 NFS 叢集，目前僅支援搭配 SUSE Linux 使用。 以 Red Hat 為基礎的高可用性 NFS 解決方案，日後才建議使用。

調整節點磁碟區大小的方式和相應放大相同，**/hana/shared** 除外。 針對 M128s VM SKU，建議的大小與類型如下：

| VM SKU | RAM | 最大 VM I/O<br /> Throughput | /hana/data | /hana/log | /root volume | /usr/sap | hana/backup |
| --- | --- | --- | --- | --- | --- | --- | --- |
| M128s | 2000 GiB | 2000 MB/秒 |3 x P30 | 2 x P20 | 1 x P6 | 1 x P6 | 2 x P40 |


請針對不同的建議磁碟區，檢查儲存體輸送量是否符合您想要執行的工作負載。 如果工作負載需要更多 **/hana/data** 和 **/hana/log** 磁碟區，您需要增加 Azure 進階儲存體 VHD 的數目。 使用比列出數目更多的 VHD 來調整磁碟區大小，會增加在 Azure 虛擬機器類型限制內的 IOPS 和 I/O 輸送量。 一併對組成 **hana/log** 磁碟區的磁碟套用 Azure 寫入加速器。
 
在 [SAP HANA TDI 儲存體需求](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html) \(英文\) 文件中，列舉的公式定義用於相應放大的 **/hana/shared** 磁碟區大小，就是每 4 個背景工作角色節點，要有單一背景工作角色節點的記憶體大小。

假設 SAP HANA 相應放大的已認證 M128s Azure VM 有大約 2 TB 記憶體，SAP 建議摘述如下：

- 一個主要節點與最多四個背景工作角色節點，**/hana/shared** 磁碟區所需的大小為 2 TB。 
- 一個主要節點與五到八個背景工作角色節點，**/hana/shared** 磁碟區的大小應為 4 TB。 
- 一個主要節點與 9 到 12 個背景工作角色節點，**/hana/shared** 需有 6 TB 大小。 
- 一個主要節點並使用 12 到 15 個之間的背景工作角色節點，您必須提供大小為 8 TB 的 **/hana/shared** 磁碟區。

相應放大 SAP HANA VM 的單一節點設定圖形中顯示的另一個重要設計，就是 VNet 或更好的子網路設定。 SAP 強烈建議從 HANA 節點間的通訊分隔用戶端/應用程式對應流量。 如圖所示，將兩個不同的 vNIC 附加到 VM，即可達成此目的。 這兩個 vNIC 位在不同的子網路，有兩個不同的 IP 位址。 接著，您可以使用 NSG 或使用者定義的路由，透過路由規則控制流量的流動。

特別是在 Azure 中，無法對特定的 vNIC 強制使用服務品質與配額。 因此，分隔用戶端/應用程式對應流量與節點間通訊，才不會有機會將一個流量串流設定為優先於其他串流。 相反地，仍可以透過分隔來保護相應放大設定的節點間通訊安全。  

>[!IMPORTANT]
>SAP 強烈建議分隔用戶端/應用程式端的網路流量與節點間流量，如本文中所述。 因此，強烈建議您如最後一個圖形所示來建立架構。
>

從網路的觀點來看，最小的必要網路架構如下：

![單一節點的相應放大基本概念](media/hana-vm-operations/scale-out-networking-overview.PNG)

目前支援的限制是一個主要節點加上 15 個背景工作角色節點。

從儲存體的觀點來看，儲存體架構如下：


![單一節點的相應放大基本概念](media/hana-vm-operations/scale-out-storage-overview.PNG)

**/hana/shared** 磁碟區位於高可用性 NFS 共用設定。 而所有其他磁碟機都在「本機」掛接至個別的 VM。 

### <a name="highly-available-nfs-share"></a>高可用性 NFS 共用
高可用性 NFS 叢集目前只能搭配 SUSE Linux 使用。 [適用於 SUSE Linux Enterprise Server 之 Azure VM 上 NFS 的高可用性](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/high-availability-guide-suse-nfs)一文說明如何設定。 如果您並未和任何其他 HANA 設定共用 NFS 叢集 (位在執行 SAP HANA 執行個體的 Azure VNet 外部)，請將該叢集安裝在相同的 VNet 中。 安裝在它自己的子網路中，並確定並非所有的任意流量都能存取子網路。 相反地，您會想要將該子網路的流量限制在對 **/hana/shared** 磁碟區執行流量的 VM IP 位址。

應該路由傳送 **/hana/shared** 流量的 HANA 相應放大 VM 所連接的 vNIC 相關建議如下：

- 由於流向 **/hana/shared** 的流量中等，所以可以透過指派給最小設定的 vNIC 來路由傳送流量
- 最後，在您部署 SAP HANA 相應放大設定的 VNet 中，為 **/hana/shared** 的流量部署第三個子網路，並指派該子網路裝載的第三個 vNIC。 使用第三個 vNIC 與相關聯的 IP 位址，以供 NFS 共用的流量使用。 接著，您可以套用不同的存取權和路由規則。

>[!IMPORTANT]
>在 SAP HANA 以相應放大方式來部署的 VM 與高可用性 NFS 之間的網路流量，在任何情況下都不可透過 [NVA](https://azure.microsoft.com/solutions/network-appliances/) 或類似的虛擬設備來路由傳送。 不過，Azure NSG 並沒有這類裝置。 請查看路由規則，以確保在從執行 SAP HANA 的 VM 存取高可用性 NFS 共用時，都會避開 NVA 或類似虛擬設備。
> 

如果您想在 SAP HANA 設定間共用高可用性 NFS 叢集，請將所有此類 HANA 設定移至相同的 VNet 中。 
 

### <a name="installing-sap-hana-scale-out-n-azure"></a>在 Azure 中安裝 SAP HANA 相應放大
安裝相應放大 SAP 設定，您需要執行的概略步驟如下：

- 部署新的或採用新的 Azure VNet 基礎結構
- 使用 Azure 受控進階儲存體磁碟區，部署新的 VM
- 部署新的或採用現有的高可用性 NFS 叢集
- 採用網路路由以確保像是 VM 間的節點間通訊不會透過 [NVA](https://azure.microsoft.com/solutions/network-appliances/) 路由傳送。 同樣的作法也適用於 VM 與高可用性 NFS 叢集間的流量。
- 安裝 SAP HANA 主要節點。
- 採用 SAP HANA 主要節點的設定參數
- 繼續安裝 SAP HANA 背景工作角色節點

#### <a name="installation-of-sap-hana-in-scale-out-configuration"></a>以相應放大設定安裝 SAP HANA
當您部署 Azure VM 基礎結構且所有其他的準備工作都已完成時，您需要以下列步驟安裝 SAP HANA 相應放大設定：

- 根據 SAP 的文件，安裝 SAP HANA 主要節點
- **安裝之後，您需要變更 global.ini 檔案並將 'basepath_shared = no' 參數新增至 global.ini**。 此參數會讓 SAP HANA 以相應放大方式執行，而不需要在節點之間「共用」**/hana/data** 與 **/hana/log** 磁碟區。 詳細資料記載於 [SAP 附註編號 2080991](https://launchpad.support.sap.com/#/notes/2080991) \(英文\)。
- 變更 global.ini 參數之後，請重新啟動 SAP HANA 執行個體
- 新增其他背景工作角色節點。 另請參閱 <https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.00/en-US/0d9fe701e2214e98ad4f8721f6558c34.html>。 在安裝期間或安裝後，使用本機 hdblcm 之類，指定用於進行 SAP HANA 內部節點間通訊的內部網路。 如需詳細文件，請參閱 [SAP 附註編號 2183363](https://launchpad.support.sap.com/#/notes/2183363) \(英文\)。 

遵循此設定常式，您安裝的相應放大設定將會使用非共用磁碟來執行 **/hana/data** 與 **hana/log**。 不過，**/hana/shared** 磁碟區會放在高可用性 NFS 共用。


## <a name="sap-hana-dynamic-tiering-20-for-azure-virtual-machines"></a>適用於 Azure 虛擬機器的 SAP HANA Dynamic Tiering 2.0

除了 Azure M 系列 VM 上的 SAP HANA 認證，Microsoft Azure 也支援 SAP HANA Dynamic Tiering 2.0 (請參閱更底下的 SAP HANA Dynamic Tiering 文件連結)。 安裝或操作此產品時雖沒有任何差異，(例如，透過 Azure 虛擬機器中的 SAP HANA Cockpit)，但是在 Azure 上正式支援有一些必要的重要事項。 以下說明這些關鍵點。 在本文中，將使用縮寫 "DT 2.0"，而不是使用 Dynamic Tiering 2.0 全名。

SAP BW 或 S4HANA 不支援 SAP HANA Dynamic Tiering 2.0。 主要使用案例現在為原生 HANA 應用程式。


### <a name="overview"></a>概觀

下圖提供有關 Microsoft Azure 上的 DT 2.0 支援概觀。 必須遵循一組必要需求，才能符合正式憑證：

- DT 2.0 必須安裝在專用的 Azure VM 上。 無法在 SAP HANA 執行所在的相同 VM 上執行
- SAP HANA 和 DT 2.0 VM 必須部署在相同的 Azure Vnet 內
- 必須部署已啟用 Azure 加速網路的 SAP HANA 和 DT 2.0 VM
- DT 2.0 VM 的儲存體類型必須是 Azure 進階儲存體
- 必須將多個 Azure 磁碟連結至 DT 2.0 VM
- 必須利用等量分割於多個 Azure 磁碟來建立軟體 raid/等量磁碟區 (透過 lvm 或 mdadm)

下列各節將會說明更多詳細資料。

![SAP HANA DT 2.0 架構概觀](media/hana-vm-operations/hana-dt-20.PNG)



### <a name="dedicated-azure-vm-for-sap-hana-dt-20"></a>SAP HANA DT 2.0 專用的 Azure VM

在 Azure IaaS 上，只有專用 VM 才支援 DT 2.0。 不得在 HANA 執行個體執行所在的相同 Azure VM 上執行 DT 2.0。 一開始有兩個 VM 類型可用於執行 SAP HANA DT 2.0：

M64-32ms、E32sv3 

如需 VM 類型說明，請參閱[這裡](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-memory)

考量到 DT 2.0 的基本概念 (關於卸載「暖」資料來節省成本)，使用對應的 VM 大小合乎情理。 可能的組合沒有相關的嚴格規則。 這取決於特定客戶工作負載。

建議的組態如下：

| SAP HANA VM 類型 | DT 2.0 VM 類型 |
| --- | --- | 
| M128ms | M64-32ms |
| M128s | M64-32ms |
| M64ms | E32sv3 |
| M64s | E32sv3 |


SAP HANA 認證 M 系列 VM 與所支援 DT 2.0 VM (M64-32ms、E32sv3) 的所有組合都可行。


### <a name="azure-networking-and-sap-hana-dt-20"></a>Azure 網路功能與 SAP HANA DT 2.0

在專用 VM 上安裝 DT 2.0 時，DT 2.0 VM 與 SAP HANA VM 之間需要最少 10 Gb 的網路輸送量。 因此，一定要將所有 VM 放在相同的 Azure Vnet 內並啟用 Azure 加速網路。

如需 Azure 加速網路的其他資訊，請參閱[這裡](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli)

### <a name="vm-storage-for-sap-hana-dt-20"></a>適用於 SAP HANA DT 2.0 的 VM 儲存體

根據 DT 2.0 最佳做法指引，每個實體核心的最小磁碟 IO 輸送量應該是 50 MB/秒。 查看真對 DT 2.0 支援的兩個 Azure VM 類型規格，您將會發現 VM 的最大磁碟 IO 輸送量限制：

- E32sv3：768 MB/sec (未快取) 表示每個實體核心的速率為 48 MB/秒
- M64-32ms：1000 MB/sec (未快取) 表示每個實體核心的速率為 62.5 MB/秒

必須將多個 Azure 磁碟連結至 DT 2.0 VM，並在 OS 層級建立軟體 raid (等量分割)，以達到每個 VM 的磁碟輸送量最大限制。 單一 Azure 磁碟無法提供可達到這方面最大 VM 限制的輸送量。 需有 Azure 進階儲存體才能執行 DT 2.0。 

- 如需 Azure 磁碟類型的詳細資訊，請參閱[這裡](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage)
- 有關透過 mdadm 建立軟體 raid 的詳細資訊，請參閱[這裡](https://docs.microsoft.com/azure/virtual-machines/linux/configure-raid)
- 有關設定 LVM 來建立等量磁碟區以達到最大輸送量的詳細資訊，請參閱[這裡](https://docs.microsoft.com/azure/virtual-machines/linux/configure-lvm)

根據大小需求，有不同的選項可達到 VM 的最大輸送量。 以下是可能的資料磁碟區磁碟組態，可讓每個 DT 2.0 VM 類型達到 VM 輸送量上限。 E32sv3 VM 應視為較小工作負載的入門層級。 萬一後來變得不夠快，則可能需要將 VM 大小調整為 M64-32ms。
M64-32ms VM 有很多記憶體，因此 IO 負載可能無法達到特別為讀取密集工作負載所設定的限制。 因此，視客戶特定工作負載而定，等量集合中少量磁碟可能就足夠。 不過，爲了以防萬一而選擇了下列磁碟組態，以保證最大輸送量：


| VM SKU | 磁碟組態 1 | 磁碟組態 2 | 磁碟組態 3 | 磁碟組態 4 | 磁碟組態 5 | 
| ---- | ---- | ---- | ---- | ---- | ---- | 
| M64-32ms | 4 x P50 -> 16 TB | 4 x P40 -> 8 TB | 5 x P30 -> 5 TB | 7 x P20 -> 3.5 TB | 8 x P15 -> 2 TB | 
| E32sv3 | 3 x P50 -> 12 TB | 3 x P40 -> 6 TB | 4 x P30 -> 4 TB | 5 x P20 -> 2.5 TB | 6 x P15 -> 1.5 TB | 


特別是在工作負載為密集讀取的情況下，可以提升 IO 效能，進而為資料庫軟體的資料磁碟區開啟 Azure 主機快取「唯讀」 (建議做法)。 而對於交易記錄，Azure 主機磁碟快取必須是「無」。 

就記錄磁碟區的大小而論，建議的起始點為 15% 資料大小的啟發學習法。 根據成本和輸送量需求，使用不同的 Azure 磁碟類型，即可完成記錄磁碟區建立。 此外，記錄磁碟區適合使用高輸送量，而如果選擇 M64-32ms，則強烈建議開啟寫入加速器 (這是 SAP HANA 的必要功能)。 這可為交易記錄提供最佳的磁碟寫入延遲 (僅適用於 M 系列)。 此外，還有一些需要考量的事項，像是每個 VM 類型的磁碟數目上限。 在 [這裡](https://docs.microsoft.com/azure/virtual-machines/windows/how-to-enable-write-accelerator)可以找到 WA 詳細資訊


以下是一些關於調整記錄磁碟區大小的範例：

| 資料磁碟區大小和磁碟類型 | 記錄磁碟區和磁碟類型組態 1 | 記錄磁碟區和磁碟類型組態 2 |
| --- | --- | --- |
| 4 x P50 -> 16 TB | 5 x P20 -> 2.5 TB | 3 x P30 -> 3 TB |
| 6 x P15 -> 1.5 TB | 4 x P6 -> 256 GB | 1 x P15 -> 256 GB |


如同 SAP HANA 相應放大一樣，/hana/shared 目錄必須在 SAP HANA VM 與 DT 2.0 VM 之間共用。 對於使用專用 VM 作為高可用性 NFS 伺服器的 SAP HANA 相應放大，建議使用相同的架構。 若要提供共用備份磁碟區，可以使用相同的設計。 但是，由客戶決定 HA 是否必要，或者只使用具有足夠儲存體容量的專用 VM 是否足以作為備份伺服器。



### <a name="links-to-dt-20-documentation"></a>DT 2.0 文件的連結 

- [SAP HANA Dynamic Tiering installation and update guide](https://help.sap.com/viewer/88f82e0d010e4da1bc8963f18346f46e/2.0.03/en-US)
- [SAP HANA Dynamic Tiering tutorials and resources](https://www.sap.com/developer/topics/hana-dynamic-tiering.html)
- [SAP HANA Dynamic Tiering PoC](https://blogs.sap.com/2017/12/08/sap-hana-dynamic-tiering-delivering-on-low-tco-with-impressive-performance/)
- [SAP HANA 2.0 SPS 02 dynamic tiering enhancements](https://blogs.sap.com/2017/07/31/sap-hana-2.0-sps-02-dynamic-tiering-enhancements/)




## <a name="operations-for-deploying-sap-hana-on-azure-vms"></a>在 Azure VM 上部署 SAP HANA 的作業
下列各節說明關於在 Azure VM 上部署 SAP HANA 系統的一些作業。

### <a name="back-up-and-restore-operations-on-azure-vms"></a>Azure VM 上的備份和還原作業
下列文件說明如何備份及還原您的 SAP HANA 部署：

- [SAP HANA 備份概觀](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-guide)
- [SAP HANA 檔案層級備份](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-file-level)
- [SAP Hana 儲存體快照集基準](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-backup-storage-snapshots)


### <a name="start-and-restart-vms-that-contain-sap-hana"></a>啟動及重新啟動包含 SAP HANA 的 VM
Azure 公用雲端的重要功能是您僅需支付運算的分鐘數。 例如，當您關閉執行 SAP HANA 的 VM 時，您只需要支付這段時間的儲存體成本。 當您在初始部署時指定 VM 的靜態 IP 位址，就可使用另一項功能。 當您重新啟動具有 SAP HANA 的 VM 時，VM 會使用其先前的 IP 位址來重新啟動。 


### <a name="use-saprouter-for-sap-remote-support"></a>使用 SAPRouter 以取得 SAP 遠端支援
如果您有內部部署位置與 Azure 之間的站對站連線，且您是執行 SAP 元件，則您可能已在執行 SAProuter。 在此情況下，請完成下列項目以取得遠端支援：

- 維護在 SAProuter 組態中裝載 SAP HANA 之 VM 的私用和靜態 IP 位址。
- 設定裝載 HANA VM 的子網路之 NSG，以允許透過 TCP/IP 通訊埠 3299 的流量。

如果您要透過網際網路連線到 Azure，且您沒有 SAP 路由器可供包含 SAP HANA 的 VM 使用，就需要安裝此元件。 在管理子網路中，將 SAProuter 安裝在不同的 VM 中。 下圖顯示不需要透過站對站連線、而需要 SAProuter 部署 SAP HANA 的概略結構描述：

![不具站台對站台連線能力和 SAProuter 的 SAP Hana 粗略部署結構描述](media/hana-vm-operations/hana-simple-networking3.PNG)

請務必在不同的 VM 上安裝 SAPRouter，而非安裝在 Jumpbox VM 上。 不同的 VM 必須有靜態 IP 位址。 若要將您的 SAProuter 連線至 SAP 所裝載的 SAProuter，請連絡 SAP 以索取 IP 位址。 (由 SAP 裝載的 SAProuter 是您在 VM 上安裝之 SAProuter 執行個體的對應項目)。使用 SAP 提供的 IP 位址來設定 SAProuter 執行個體。 在組態集中，唯一必要的連接埠是 TCP 通訊埠 3299。

如需有關如何設定和維護透過 SAProuter 的遠端連線詳細資訊，請參閱 [SAP 文件](https://support.sap.com/en/tools/connectivity-tools/remote-support.html)。

### <a name="high-availability-with-sap-hana-on-azure-native-vms"></a>在 Azure 原生 VM 上具有 SAP Hana 的高可用性
如果您是執行 SUSE Linux Enterprise Server for SAP Applications 12 SP1 或更新版本，可以使用 STONITH 裝置建立 Pacemaker 叢集。 您可以使用裝置設定 SAP HANA 組態，該組態是使用包含 HANA 系統複寫以及自動容錯移轉的同步複寫。 如需設定程序的詳細資訊，請參閱 [Azure 虛擬機器的 SAP HANA 高可用性指南](https://docs.microsoft.com/azure/virtual-machines/workloads/sap/sap-hana-availability-overview)。
