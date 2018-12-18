---
title: Azure 匯入/匯出服務的常見問題集 | Microsoft Docs
description: 參閱 Azure 匯入/匯出服務常見問題集的解答。
author: alkohli
services: storage
ms.service: storage
ms.topic: article
ms.date: 05/22/2018
ms.author: alkohli
ms.component: common
ms.openlocfilehash: 6a2d067625e96aefdcb0197e1abb13b0c41fbe7f
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39521433"
---
# <a name="azure-importexport-service-frequently-asked-questions"></a>Azure 匯入/匯出服務：常見問題集 
以下是當您使用 Azure 匯入/匯出服務將資料轉送至 Azure 儲存體時可能會遇到的問題與解答。 問題和解答可分為下列幾個類別：

- 關於 Azure 匯入/匯出服務
- 準備匯入/匯出要用的磁碟
- 匯入/匯出作業
- 寄送磁碟
- 其他資訊 

## <a name="about-importexport-service"></a>關於 Azure 匯入/匯出服務

### <a name="can-i-copy-azure-file-storage-using-the-azure-importexport-service"></a>我可以使用 Azure 匯入/匯出服務複製 Azure 檔案儲存體嗎？

是。 Azure 匯入/匯出服務支援匯入至 Azure 檔案儲存體。 目前不支援 Azure 檔案匯出。

### <a name="is-the-azure-importexport-service-available-for-csp-subscriptions"></a>Azure 匯入/匯出服務可用於 CSP 訂用帳戶嗎？

是。 Azure 匯入/匯出服務支援雲端解決方案提供者 (CSP) 的訂用帳戶。

### <a name="can-i-use-the-azure-importexport-service-to-copy-pst-mailboxes-and-sharepoint-data-to-o365"></a>我可以使用 Azure 匯入/匯出服務將 PST 信箱及 SharePoint 資料複製到 O365 嗎？

是。 如需詳細資訊，請移至[將 PST 檔案或 SharePoint 資料匯入至 Office 365](https://technet.microsoft.com/library/ms.o365.cc.ingestionhelp.aspx)。

### <a name="can-i-use-the-azure-importexport-service-to-copy-my-backups-offline-to-the-azure-backup-service"></a>可以使用 Azure 匯入/匯出服務，將我的離線備份複製到 Azure 備份服務嗎？

是。 如需詳細資訊，請移至 [Azure 備份中的離線備份工作流程](../../backup/backup-azure-backup-import-export.md)。

### <a name="can-i-purchase-drives-for-importexport-jobs-from-microsoft"></a>我可以為了匯入/匯出作業向 Microsoft 購買磁碟機嗎？

否。 您必須寄送自己的磁碟機來進行匯入和匯出作業。


## <a name="preparing-disks-for-importexport"></a>準備匯入/匯出要用的磁碟

### <a name="can-i-skip-the-drive-preparation-step-for-an-import-job-can-i-prepare-a-drive-without-copying"></a>我可以略過匯入作業的磁碟機準備步驟嗎？ 我可以在不進行複製的情況下準備磁碟機嗎？

否。 用來匯入資料的任何磁碟機，都必須使用 Azure WAImportExport 工具備妥。 並使用此工具將資料複製到磁碟機。

### <a name="do-i-need-to-perform-any-disk-preparation-when-creating-an-export-job"></a>我需要在建立匯出作業時執行任何磁碟準備工作嗎？

否。 但建議先執行一些前置檢查。 若要確認所需的磁碟數目，請使用 WAImportExport 工具的 PreviewExport 命令。 如需詳細資訊，請參閱 [Previewing Drive Usage for an Export Job (預覽匯出作業的磁碟機使用量)](https://msdn.microsoft.com/library/azure/dn722414.aspx)。 此命令可根據您要使用的磁碟機大小，協助您預覽所選 Blob 的磁碟機使用情況。 也請確認您可以對為了匯出作業而寄送的硬碟進行讀取和寫入。

## <a name="importexport-jobs"></a>匯入/匯出作業

### <a name="can-i-cancel-my-job"></a>我可以取消作業嗎？
是。 您可以在作業狀態為 [建立中] 或 [運送中] 時取消作業。 如果在這些階段以外，即無法取消作業，而且作業會繼續進行到最後一個階段。

### <a name="how-long-can-i-view-the-status-of-completed-jobs-in-the-azure-portal"></a>我可以在 Azure 入口網站中檢視已完成作業的狀態多久？
您最多可以檢視之前 90 天內已完成作業的狀態。 已完成的作業會在 90 天後刪除。

### <a name="if-i-want-to-import-or-export-more-than-10-drives-what-should-i-do"></a>如果我想匯入或匯出的磁碟機超過 10 個，我該怎麼做？
一項匯入或匯出作業只能參考單一作業中的 10 個磁碟機。 若要寄送的磁碟機超過 10 個，應建立多項作業。 與相同作業相關聯的磁碟機必須在相同的包裹中一起寄送。 如果資料容量需跨越多個磁碟匯入作業，請透過 bulkimport@microsoft.com 連絡 Microsoft，以取得詳細資訊與指引。                                                              

## <a name="shipping-disks"></a>寄送磁碟

### <a name="what-is-the-maximum-number-of-hdd-for-in-one-shipment"></a>一次可寄送最多幾個 HDD？
一次可寄送的 HDD 數量並沒有限制。 如果磁碟屬於多項作業，我們建議您： 
- 在磁碟上貼上對應作業名稱的標籤。
- 以帶有 -1、-2 等尾碼的追蹤號碼來更新作業。

### <a name="should-i-include-anything-other-than-the-hdd-in-my-package"></a>我的包裹中除了 HDD 以外還應該包含任何東西嗎？
寄送包裹中只須包含您的硬碟。 請勿加入電源線或 USB 纜線等項目。

### <a name="do-i-have-to-ship-my-drives-using-fedex-or-dhl"></a>我必須用 FedEx 還是 DHL 寄送我的磁碟機？
您可使用任何已知的貨運公司，例如 FedEx、DHL、UPS 或郵政服務將磁碟機寄送到 Azure 資料中心。 不過，針對將磁碟機從資料中心寄還給您的貨運，您必須提供：

- 美國或歐洲地區的 FedEx 帳戶號碼，或是
- 亞洲和澳洲地區的 DHL 帳戶號碼。

### <a name="are-there-any-restrictions-with-shipping-my-drive-internationally"></a>將我的磁碟機進行國際運送有任何限制嗎？
請注意，您寄送的實體媒體可能需要跨國界。 您必須確定實體媒體和資料的匯入和/或匯出符合相關管轄法律。 在寄出實體媒體之前，請洽詢顧問來確認您的媒體和資料可以合法地寄到所識別的資料中心。 這有助於確保及時送達 Microsoft。

### <a name="when-creating-a-job-the-shipping-address-is-a-location-that-is-different-from-my-storage-account-location-what-should-i-do"></a>建立作業時，寄送地址是與我儲存體帳戶位置不同的位置。 我該怎麼辦？

某些儲存體帳戶位置會對應至替代的運送位置。 先前可用的運送位置也可能暫時對應到替代位置。 工作建立期間，請務必檢查提供的運送地址，然後才運送磁碟機。

### <a name="when-shipping-my-drive-the-carrier-asks-for-the-data-center-contact-address-and-phone-number-what-should-i-provide"></a>寄送我的磁碟機時，貨運公司要求資料中心連絡人地址和電話號碼。 我該提供什麼？

在建立作業時，提供電話號碼和 DC 地址。


## <a name="miscellaneous"></a>其他資訊

### <a name="what-happens-if-i-accidentally-send-an-hdd-that-does-not-conform-to-the-supported-requirements"></a>如果我不小心傳送不符支援需求的 HDD，則會發生什麼情況？

Azure 資料中心會將不符支援需求的磁碟機退回給您。 如果包裹中只有部分磁碟機符合支援需求，將會處理這些磁碟機，而不符需求的磁碟機則會退回給您。

### <a name="does-the-service-format-the-drives-before-returning-them"></a>服務會在退回磁碟機前進行格式化嗎？

否。 所有磁碟機都使用 BitLocker 加密。

### <a name="how-can-i-access-data-that-is-imported-by-this-service"></a>如何存取這個服務所匯入的資料？

使用 Azure 入口網站或[儲存體總管](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer)來存取您 Azure 儲存體帳戶下的資料。  

### <a name="after-the-import-is-complete-what-does-my-data-look-like-in-the-storage-account-is-my-directory-hierarchy-preserved"></a>匯入作業完成後，我的資料在儲存體帳戶中會是什麼樣子？ 我的目錄階層會保留嗎？

準備匯入作業的硬碟時，目的地會由資料集 CSV 的 DstBlobPathOrPrefix 欄位指定。 這是硬碟的資料將複製到其中的儲存體帳戶目的地容器。 在此目的地容器內，會針對硬碟裡的資料夾建立虛擬目錄，並針對檔案建立 Blob。 

### <a name="if-a-drive-has-files-that-already-exist-in-my-storage-account-does-the-service-overwrite-existing-blobs-or-files"></a>如果磁碟機中具有我儲存體帳戶裡已存在的檔案，那麼服務將會覆寫現有的 Blob 或檔案嗎？

視情況而定。 準備磁碟機時，您可以使用資料集 CSV 檔案中名為 /Disposition:<rename|no-overwrite|overwrite> 的欄位，指定是否應該覆寫目的地檔案還是予以忽略。 根據預設，服務將為新檔案重新命名，而不會覆寫現有的 Blob 或檔案。

### <a name="is-the-waimportexport-tool-compatible-with-32-bit-operating-systems"></a>WAImportExport 工具與 32 位元作業系統相容嗎？
否。 WAImportExport 工具只與 64 位元 Windows 作業系統相容。 如需可支援作業系統的完整清單，請移至[支援的作業系統]()。 


### <a name="what-is-the-maximum-block-blob-and-page-blob-size-supported-by-azure-importexport"></a>Azure 匯入/匯出所支援的最大區塊 Blob 和分頁 Blob 大小是多少？

最大區塊 Blob 大小大約是 4.768 TB 或 5,000,000 MB。
最大分頁 Blob 大小為 1 TB。


### <a name="does-azure-importexport-support-aes-256-encryption"></a>Azure 匯入/匯出是否支援 AES-256 加密？
根據預設，Azure 匯入/匯出服務會使用 AES-128 bitlocker 加密。 您可以先透過手動執行 bitlocker 加密，將其變更為 AES-256，然後再複製資料。 

- 如果使用 [WAImportExport V1](http://download.microsoft.com/download/0/C/D/0CD6ABA7-024F-4202-91A0-CE2656DCE413/WaImportExportV1.zip)，以下是命令範例
    ```
    WAImportExport PrepImport /sk:<StorageAccountKey> /csas:<ContainerSas> /t: <TargetDriveLetter> [/format] [/silentmode] [/encrypt] [/bk:<BitLockerKey>] [/logdir:<LogDirectory>] /j:<JournalFile> /id:<SessionId> /srcdir:<SourceDirectory> /dstdir:<DestinationBlobVirtualDirectory> [/Disposition:<Disposition>] [/BlobType:<BlockBlob|PageBlob>] [/PropertyFile:<PropertyFile>] [/MetadataFile:<MetadataFile>] 
    ```
- 如果使用 [WAImportExport V2](https://www.microsoft.com/en-us/download/details.aspx?id=55280)，請指定 "AlreadyEncrypted" 並提供磁碟機集 CSV 中的金鑰。
    ```
    DriveLetter,FormatOption,SilentOrPromptOnFormat,Encryption,ExistingBitLockerKey
    G,AlreadyFormatted,SilentMode,AlreadyEncrypted,060456-014509-132033-080300-252615-584177-672089-411631 |
    ```

## <a name="next-steps"></a>後續步驟

* [什麼是 Azure 匯入/匯出？](storage-import-export-service.md)


