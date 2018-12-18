---
title: 將 SQL Server 資料庫移轉至 VM 上的 SQL Server | Microsoft Docs
description: 深入了解如何將內部部署使用者資料庫移轉到 Azure 虛擬機器中的 SQL Server。
services: virtual-machines-windows
documentationcenter: ''
author: rothja
manager: craigg
editor: ''
tags: azure-service-management
ms.assetid: 00fd08c6-98fa-4d62-a3b8-ca20aa5246b1
ms.service: virtual-machines-sql
ms.workload: iaas-sql-server
ms.tgt_pltfrm: vm-windows-sql-server
ms.devlang: na
ms.topic: article
ms.date: 08/18/2018
ms.author: jroth
ms.openlocfilehash: 0677faa90c73ffe4c0c1c48600c2f1ef2d05eb50
ms.sourcegitcommit: 4de6a8671c445fae31f760385710f17d504228f8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39628778"
---
# <a name="migrate-a-sql-server-database-to-sql-server-in-an-azure-vm"></a>將 SQL Server 資料庫移轉至 Azure VM 中的 SQL Server

有多個方法可將內部部署 SQL Server 使用者資料庫移轉至 Azure VM 中的 SQL Server。 本文將簡短討論各種方法，並針對各種案例建議最佳的方法。

[!INCLUDE [learn-about-deployment-models](../../../../includes/learn-about-deployment-models-both-include.md)]

## <a name="what-are-the-primary-migration-methods"></a>主要的移轉方法有哪些？
主要的移轉方法如下：

* 使用壓縮執行在內部部署備份，並手動將備份檔案複製到 Azure 虛擬機器
* 執行備份至 URL 並從 URL 還原到 Azure 虛擬機器
* 中斷連結並將資料和記錄檔複製到 Azure blob 儲存體，然後從 URL 連結至 Azure VM 中的 SQL Server
* 將內部部署實體機器轉換為 Hyper-V VHD，接著上傳至 Azure Blob 儲存體，然後使用上傳的 VHD 部署為新的 VM
* 使用 Windows 匯入/匯出服務寄送硬碟機
* 如果您有內部部署的 AlwaysOn 部署，請使用 [加入 Azure 複本精靈](../sqlclassic/virtual-machines-windows-classic-sql-onprem-availability.md) 在 Azure 中建立複本，然後進行容錯移轉，將使用者指向 Azure 資料庫執行個體
* 使用 SQL Server [異動複寫](https://msdn.microsoft.com/library/ms151176.aspx) 將 Azure SQL Server 執行個體設定為訂閱者，然後停用複寫，將使用者指向 Azure 資料庫執行個體

> [!TIP]
> 您也可以使用這些相同的技巧，在 Azure 中的 SQL Server VM 之間移動資料庫。 例如，若要將 SQL Server 資源庫映像 VM 從一個版本升級至另一版本，並沒有支援的方法。 在此情況下，您應該使用新的版本建立新的 SQL Server VM，然後使用本文其中一個移轉技術來移動資料庫。 

## <a name="choosing-your-migration-method"></a>選擇您的移轉方法
為了達到最佳的資料傳輸效能，請使用壓縮的備份檔案將資料庫檔案移轉至 Azure VM。

若要將進行資料庫移轉程序時的停機時間縮到最短，請使用 AlwaysOn 選項或異動複寫選項。

如果無法使用上述方法，則請以手動方式移轉資料庫。 使用此方法時，您通常會從進行資料庫備份開始，後面接著將一份資料庫備份複製到 Azure，然後再執行資料庫還原。 您也可以將資料庫檔案本身複製到 Azure，然後再連接它們。 若要將資料庫移轉到 Azure VM，有數種方法可以完成此手動程序。

> [!NOTE]
> 當您從舊版 SQL Server 升級至 SQL Server 2014 或 SQL Server 2016 時，您應該考慮是否需要變更。 我們建議您先解決所有新版本 SQL Server 不支援的功能相依性作為移轉專案的一部分。 如需有關支援版本和案例的詳細資訊，請參閱 [升級至 SQL Server](https://msdn.microsoft.com/library/bb677622.aspx)。

下表會列出每個主要的移轉方法，並討論何時使用每個方法是最合適的。

| 方法 | 來源資料庫版本 | 目的地資料庫版本 | 來源資料庫備份的大小條件約束 | 注意 |
| --- | --- | --- | --- | --- |
| [使用壓縮執行在內部部署備份，並手動將備份檔案複製到 Azure 虛擬機器](#backup-and-restore) |SQL Server 2005 或更新版本 |SQL Server 2005 或更新版本 |[Azure VM 儲存體限制](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/) | 對於在電腦之間移動資料庫，這是非常簡單且通過完善測試的技術。 |
| [執行備份至 URL 並從 URL 還原到 Azure 虛擬機器](#backup-to-url-and-restore) |SQL Server 2012 SP1 CU2 或更新版本 |SQL Server 2012 SP1 CU2 或更新版本 |< 12.8 TB 適用於 SQL Server 2016，否則為 < 1 TB | 此法是使用 Azure 儲存體將備份檔案移至 VM 的另一種方式。 |
| [中斷連結並將資料和記錄檔複製到 Azure blob 儲存體，然後從 URL 連結至 Azure 虛擬機器中的 SQL Server](#detach-and-attach-from-url) |SQL Server 2005 或更新版本 |SQL Server 2014 或更新版本 |[Azure VM 儲存體限制](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/) |當您打算 [使用 Azure Blob 儲存體服務儲存這些檔案](https://msdn.microsoft.com/library/dn385720.aspx) ，並將其連接至在 Azure VM 中執行的 SQL Server，尤其是針對極大資料庫時，請使用這個方法。 |
| [將內部部署機器轉換為 Hyper-V VHD，接著上傳至 Azure Blob 儲存體，然後使用上傳的 VHD 部署新的虛擬機器](#convert-to-vm-and-upload-to-url-and-deploy-as-new-vm) |SQL Server 2005 或更新版本 |SQL Server 2005 或更新版本 |[Azure VM 儲存體限制](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/) |使用時機包含[使用您自己的 SQL Server 授權時](../../../sql-database/sql-database-paas-vs-sql-server-iaas.md)、移轉將在舊版 SQL Server 上執行的資料庫時，或同時移轉系統和使用者資料庫作為其他使用者資料庫和/或系統資料庫的一部分資料庫相依性時。 |
| [使用 Windows 匯入/匯出服務寄送硬碟機](#ship-hard-drive) |SQL Server 2005 或更新版本 |SQL Server 2005 或更新版本 |[Azure VM 儲存體限制](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/) |當手動複製方法太慢，例如包含極大資料庫時，請使用 [Windows 匯入/匯出服務](../../../storage/common/storage-import-export-service.md) |
| [使用加入 Azure 複本精靈](../sqlclassic/virtual-machines-windows-classic-sql-onprem-availability.md) |SQL Server 2012 或更新版本 |SQL Server 2012 或更新版本 |[Azure VM 儲存體限制](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/) |可將停機時間縮到最短，請在您有內部部署的 AlwaysOn 部署時使用 |
| [使用 SQL Server 異動複寫](https://msdn.microsoft.com/library/ms151176.aspx) |SQL Server 2005 或更新版本 |SQL Server 2005 或更新版本 |[Azure VM 儲存體限制](https://azure.microsoft.com/documentation/articles/azure-subscription-service-limits/) |請在您需要將停機時間縮到最短且沒有內部部署的 AlwaysOn 部署時使用 |

## <a name="backup-and-restore"></a>備份與還原
備份壓縮的資料庫，並將備份複製到 VM，接著還原資料庫。 如果您的備份檔案大於 1 TB，您必須等量磁碟區，因為 VM 磁碟的大小上限為 1 TB。 您可以使用下列一般步驟，使用此手動方法來移轉使用者資料庫：

1. 執行完整資料庫備份至內部部署位置。
2. 使用所需的 SQL 伺服器版本建立或上傳虛擬機器。
3. 根據您的需求設定連線。 請參閱 [連線到 Azure 上的 SQL Server 虛擬機器 (Resource Manager)](virtual-machines-windows-sql-connect.md)。
4. 使用遠端桌面、Windows 檔案總管或命令提示字元的 copy 命令，將您的備份檔案複製到您的 VM。

## <a name="backup-to-url-and-restore"></a>備份至 URL 和還原
除了備份至本機檔案，您可以使用[備份至 URL](https://msdn.microsoft.com/library/dn435916.aspx) 並從 URL 還原至 VM。 當您想要提升效能，且需要超過每個 blob 的大小限制時，建議使用 SQL Server 2016 以支援等量備份組。 對於大型資料庫，建議使用 [Windows 匯入/匯出服務](../../../storage/common/storage-import-export-service.md) 。

## <a name="detach-and-attach-from-url"></a>從 URL 中斷連結和從 URL 連結
中斷資料庫和記錄檔案的連線，並將其傳輸至 [Azure Blob 儲存體](https://msdn.microsoft.com/library/dn385720.aspx)。 然後在您的 Azure VM 上從 URL 連結資料庫。 如果您想要實體資料庫檔案位於 Blob 儲存體之中，請使用這個方法。 這對於非常大型的資料庫很實用。 您可以使用下列一般步驟，使用此手動方法來移轉使用者資料庫：

1. 從內部部署資料庫執行個體中斷連結資料庫檔案。
2. 使用 [AZCopy 命令列公用程式](../../../storage/common/storage-use-azcopy.md)，將中斷連結的資料庫檔案複製到 Azure blob 儲存體。
3. 將資料庫檔案從 Azure URL 連結至 Azure VM 中的 SQL Server 執行個體。

## <a name="convert-to-vm-and-upload-to-url-and-deploy-as-new-vm"></a>轉換為 VM 並上傳至 URL 以及部署為新的 VM
您可以使用這個方法，將內部部署 SQL Server 執行個體中的所有系統和使用者資料庫移轉至 Azure 虛擬機器。 您可以使用下列一般步驟，使用此手動方法來移轉整個 SQL Server 執行個體：

1. 將實體或虛擬機器轉換為 Hyper-V VHD。
2. 使用 [Add-AzureVHD Cmdlet](https://msdn.microsoft.com/library/windowsazure/dn495173.aspx)，將 VHD 檔案上傳至 Azure 儲存體。
3. 使用上傳的 VHD 來部署新的虛擬機器。

> [!NOTE]
> 若要移轉整個應用程式，請考慮使用 [Azure Site Recovery](../../../site-recovery/site-recovery-overview.md)。

## <a name="ship-hard-drive"></a>寄送硬碟機
透過網路進行上傳所費不貲或不可行時，請使用 [Windows 匯入/匯出服務方法](../../../storage/common/storage-import-export-service.md) ，將大量檔案資料移轉至 Azure Blob 儲存體。 您可以使用此服務，將包含該資料的一個或多個硬碟送至 Azure 資料中心，而您的資料將會在此上傳至儲存體帳戶。

## <a name="next-steps"></a>後續步驟
如需在 Azure 虛擬機器上執行 SQL Server 的詳細資訊，請參閱 [Azure 虛擬機器上的 SQL Server 概觀](virtual-machines-windows-sql-server-iaas-overview.md)。

> [!TIP]
> 如果您有 SQL Server 虛擬機器的相關問題，請參閱[常見問題集](virtual-machines-windows-sql-server-iaas-faq.md)。

如需從擷取的映像建立 Azure SQL Server 虛擬機器的指示，請參閱 CSS SQL Server 工程師部落格上的[從擷取之映像複製 Azure SQL 虛擬機器的秘訣和竅門](https://blogs.msdn.microsoft.com/psssql/2016/07/06/tips-tricks-on-cloning-azure-sql-virtual-machines-from-captured-images/)。

