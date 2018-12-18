---
title: Microsoft Azure 資料箱磁碟常見問題集 | 資料中的 Microsoft Docs
description: 包含 Azure 資料箱磁碟的常見問題和解答，這是可讓您將大量資料傳輸至 Azure 的雲端解決方案
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: overview
ms.date: 09/28/2018
ms.author: alkohli
ms.openlocfilehash: 8acb808e4b60926fbe6b3faa3fa6a7174a190884
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49094236"
---
# <a name="what-is-azure-data-box-disk-preview"></a>什麼是 Azure 資料箱磁碟？ (預覽)

Microsoft Azure 資料箱磁碟雲端解決方案可讓您以快速、實惠和可靠的方式，將數 TB 的資料傳送至 Azure。 此常見問題集包含您在 Azure 入口網站中使用資料箱磁碟時可能遇到的問題及其解答。 

問題和解答可分為下列幾個類別：

- 關於此服務
- 設定和連線 
- 追蹤狀態
- 移轉資料 
- 確認和上傳資料 

> [!IMPORTANT]
> 資料箱磁碟處於預覽狀態。 部署此解決方案之前，請先檢閱 [Azure 預覽版服務條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

## <a name="about-the-service"></a>關於此服務

### <a name="q-what-is-azure-data-box-service"></a>問： 什麼是 Azure 資料箱服務？ 
A.  Azure 資料箱服務是針對離線資料擷取而設計的。 此服務可管理依設計供不同儲存體容量的資料傳輸使用的各種產品。 

### <a name="q-what-are-azure-data-box-disks"></a>問： 什麼是 Azure 資料箱磁碟？
A. Azure 資料箱磁碟可讓您快速、實惠且安全地將數 TB 的資料輸入和輸出 Azure。 Microsoft 會為您提供 1 到 5 個磁碟，儲存體容量上限為 35 TB。 您可以透過 Azure 入口網站中的資料箱服務輕鬆設定、連接和解除鎖定這些磁碟。  

磁碟會使用 Microsoft BitLocker 磁碟機加密進行加密，而您的加密金鑰會在 Azure 入口網站上受到管理。 然後，您可以從客戶的伺服器複製資料。 在資料中心，Microsoft 會使用快速的私人網路上傳連結，將您的資料從磁碟機移轉至雲端，並上傳至 Azure。

### <a name="q-when-should-i-use-data-box-disks"></a>問： 何時應使用資料箱磁碟？
A. 如果您有不超過 40 TB 的資料要傳輸至 Azure，則使用資料箱磁碟將有其效益。

### <a name="q-what-is-the-price-of-data-box-disks"></a>問： 資料箱磁碟的價格為何？
A. 在預覽期間，資料箱磁碟可供免費使用。 出貨也是免費的，但 Azure 儲存體則需收費。

### <a name="q-how-do-i-get-data-box-disks"></a>問： 如何取得資料箱磁碟？ 
A.  若要取得 Azure 資料箱磁碟，請先註冊[資料箱磁碟預覽](http://aka.ms/AzureDataBox)。 接下來登入 Azure 入口網站，並建立磁碟的資料箱訂單。 請提供您的連絡資訊和通知詳細資料。 在您下訂單後，根據可用性，磁碟會在 10 天內寄送給您。   

### <a name="q-what-is-the-maximum-amount-of-data-i-can-transfer-with-data-box-disks-in-one-instance"></a>問： 使用資料箱磁碟最多可在一個執行個體中傳輸多少資料量？
A. 就各有 8 TB (可用容量為 7 TB) 的 5 個磁碟而言，可用容量上限為 35 TB。 因此，您可以在一個執行個體中傳輸 35 TB 的資料。  若要傳輸更多資料，您必須訂購更多磁碟。

### <a name="q-how-can-i-check-if-data-box-disks-are-available-in-my-region"></a>問： 如何確認在我的區域中是否適用資料箱磁碟？ 
A.  在預覽階段期間，資料箱磁碟適用於美國、加拿大、澳大利亞和歐盟的所有國家/地區。  

### <a name="q-which-regions-can-i-store-data-in-with-data-box-disks"></a>問： 在哪些區域中可以使用資料箱磁碟來儲存資料？
A. 在預覽期間，美國、加拿大、澳大利亞、西歐和北歐內的所有區域均支援資料箱磁碟。 只有 Azure 公用雲端區域受到支援。 Azure Government 或其他主權雲端不受支援。

### <a name="q-whom-should-i-contact-if-i-encounter-any-issues--with-data-box-disks"></a>問： 如果遇到任何資料箱磁碟方面的問題，該與誰連絡？
A. 如果您遇到任何資料箱磁碟方面的問題，請連絡[資料箱磁碟支援部門](mailto:expresspodsupport@microsoft.com)。

## <a name="configure-and-connect"></a>設定和連線
 
### <a name="q-can-i-specify-the-number-of-data-box-disks-in-the-order"></a>問： 可以在訂單中指定資料箱磁碟的數目嗎？
A.  否。 您會根據磁碟的資料大小和可用性取得 8 TB 磁碟 (最多 5 個磁碟)。  

### <a name="q-how-do-i-unlock-the-data-box-disks"></a>問： 如何將資料箱磁碟解除鎖定？ 
A.  在 Azure 入口網站中，移至您的資料箱磁碟訂單，並瀏覽至 [裝置詳細資料]。 複製通行金鑰。 從 Azure 入口網站下載及擷取您作業系統適用的資料箱磁碟解鎖工具。 在有資料要複製到磁碟的電腦上執行此工具。 請提供用來將磁碟解除鎖定的通行金鑰。 所有磁碟均可用相同的通行金鑰解除鎖定。 

如需逐步指示，請移至[解除鎖定 Windows 用戶端上的磁碟](data-box-disk-deploy-set-up.md#unlock-disks-on-windows-client)或[解除鎖定 Linux 用戶端上的磁碟](data-box-disk-deploy-set-up.md#unlock-disks-on-linux-client)。

### <a name="q-can-i-use-a-linux-host-computer-to-connect-and-copy-the-data-on-to-the-data-box-disks"></a>問： 是否可使用 Linux 主機電腦連線至資料箱磁碟，並將資料複製到該處？
A.  是。 Linux 和 Windows 用戶端皆可用來連線至資料箱磁碟，並將資料複製到其中。 如需詳細資訊，請移至主機電腦[支援的作業系統](data-box-disk-system-requirements.md)清單。

### <a name="q-my-disks-are-dispatched-but-now-i-want-to-cancel-this-order-why-is-the-cancel-button-not-available"></a>問： 我的磁碟已派送，但現在我想要取消此訂單。 為何 [取消] 按鈕無法使用？
A.  在訂購磁碟後，您只能在出貨之前取消訂單。 一旦磁碟已派送，您即無法取消訂單。 在預覽階段中，您可以免費送回磁碟，但在此解決方案正式推出後，此機制將可能有所變更。 

### <a name="q-can-i-connect-multiple-data-box-disks-at-the-same-to-the-host-computer-to-transfer-data"></a>問： 是否可以同時將多個資料箱磁碟連線至主機電腦以傳輸資料？
A. 是。 您可以將多個資料箱磁碟連線至相同的主機電腦以傳輸資料，並且可以平行執行多個複製作業。

## <a name="track-status"></a>追蹤狀態

### <a name="q-how-do-i-track-the-disks-from-when-i-placed-the-order-to-shipping-the-disks-back"></a>問： 從我下訂單到磁碟送到的這段期間，要如何追蹤磁碟的狀態？ 
A.  您可以在 Azure 入口網站中追蹤資料箱磁碟訂單的狀態。 當您建立訂單時，系統也會提示您提供通知電子郵件。 如果您有提供的話，您將會透過電子郵件獲知訂單的所有狀態變更。 請取得關於如何[設定通知電子郵件](data-box-portal-ui-admin.md#edit-notification-details)的資訊。

### <a name="q-how-do-i-return-the-disks"></a>問： 如何送回磁碟？ 
A.  Microsoft 會在寄送包裹中提供隨附於資料箱磁碟的出貨標籤。 請將標籤黏貼於寄送包裹，並將密封的包裹投遞到貨運公司地點。 如果標籤受損或遺失，請移至 [概觀] > [下載出貨標籤]，並下載新的寄返標籤。

## <a name="migrate-data"></a>移轉資料

### <a name="q-what-is-the-maximum-data-size-that-can-be-used-with-data-box-disks"></a>問： 資料箱磁碟可處理的資料大小上限為何？  
A.  資料箱磁碟解決方案最多可以有 5 個磁碟，可用容量上限為 35 TB。 磁碟本身為 8 TB (可用容量為 7 TB)。 

### <a name="q-what-are-the-maximum-block-blob-and-page-blob-sizes-supported-by-data-box-disks"></a>問： 資料箱磁碟所支援的最大區塊 Blob 和分頁 Blob 大小為何？ 
A.  大小上限由 Azure 儲存體限制所控管。 區塊 Blob 上限大約是 4.768 TiB，而分頁 Blob 大小上限為 8 TiB。 如需詳細資訊，請移至 [Azure 儲存體延展性和效能目標](../storage/common/storage-scalability-targets.md)。 

### <a name="q-what-is-the-data-transfer-speed-for-data-box-disks"></a>問： 資料箱磁碟的資料傳輸速度為何？
A. 使用透過 USB 3.0 連線的磁碟進行測試時，磁碟效能可達 430 MB/s。 實際數字會隨使用的檔案大小而不同。 如果檔案較小，您可能會看到較低的效能。

### <a name="q-how-do-i-know-that-my-data-is-secure-during-transit"></a>問： 如何得知我的資料在傳輸期間安全無虞？ 
A.  資料箱磁碟會使用 BitLocker AES-128 位元加密進行加密，且通行金鑰只能在 Azure 入口網站中取得。 請使用您的帳戶認證登入 Azure 入口網站，以取得通行金鑰。 在執行資料箱磁碟解除鎖定工具時，必須提供此通行金鑰。

### <a name="q-how-do-i-copy-the-data-to-the-data-box-disks"></a>問： 如何將資料複製到資料箱磁碟？ 
A.  使用 SMB 複製工具 (例如 Robocopy、Diskboss) 甚或 Windows 檔案總管的拖放功能，將資料複製到磁碟上。 

### <a name="q-are-there-any-tips-to-speed-up-the-data-copy"></a>問： 是否有任何秘訣可加快複製資料的速度？
A.  若要加快複製程序：

- 使用多個資料複製串流。 例如，採用 Robocopy 時，請使用多執行緒選項。 如需與使用的確切命令有關的詳細資訊，請移至[教學課程：將資料複製到 Azure 資料箱磁碟並確認](data-box-disk-deploy-copy-data.md#copy-data-to-disks)。
- 使用多個工作階段。
- 不透過網路共用進行複製 (此方式可能會受限於網路速度)，而是一律將資料放置在磁碟所連線到的本機電腦上。
- 確定您的整個複製程序均使用 USB 3.0 或更新版本。 請下載並使用 [USBView 工具](https://docs.microsoft.com/windows-hardware/drivers/debugger/usbview)，以識別連線至電腦的 USB 控制器和 USB 裝置。
- 對用來複製資料的電腦建立效能的基準。 請下載並使用 [Bluestop FIO 工具](https://bluestop.org/fio/)，以建立伺服器硬體效能的基準。

### <a name="q-how-to-speed-up-the-data-if-the-source-data-has-small-files-kbs-or-few-mbs"></a>問： 如果來源資料具有小型檔案 (KB 或 MB)，應如何加快資料傳輸速度？
A.  若要加快複製程序：

- 在快速的儲存體上建立本機 VHDx，或在 HDD/SSD (速度較慢) 上建立空白的 VHD。
- 將其掛接到 VM。
- 將檔案複製到 VM 的磁碟。


### <a name="q-can-i-use-multiple-storage-accounts-with-data-box-disks"></a>問： 是否可對資料箱磁碟使用多個儲存體帳戶？
A.  否。 目前僅支援對資料箱磁碟使用一個儲存體帳戶 (一般或傳統)。 經常性存取和非經常性存取 Blob 均受支援。 在預覽期間，僅支援美國、西歐及北歐等區域位於 Azure 公用雲端中的儲存體帳戶。

## <a name="verify-and-upload"></a>確認和上傳

### <a name="q-how-soon-can-i-access-my-data-in-azure-once-ive-shipped-the-disks-back"></a>問： 在送回磁碟後，多久可以存取我在 Azure 中的資料？ 
A.  當複製資料的訂單狀態顯示為完成時，您就應該能夠立即存取您的資料。

### <a name="q-where-is-my-data-located-in-azure-after-the-upload"></a>問： 我的資料在上傳後會存放於 Azure 的何處？
A.  當您在磁碟上的 *BlockBlob* 和 *PageBlob* 資料夾下複製資料後，系統會在 Azure 儲存體帳戶中為 *BlockBlob* 和 *PageBlob* 資料夾下的每個子資料夾建立容器。 如果您直接複製 *BlockBlob* 和 *PageBlob* 資料夾下的檔案，這些檔案將會位於 Azure 儲存體帳戶下的預設容器 *$root* 中。 

### <a name="q-i-just-noticed-that-i-did-not-follow-the-azure-naming-requirements-for-my-containers-will-my-data-fail-to-upload-to-azure"></a>問： 我剛剛發現我的容器並未遵循 Azure 命名需求。 我的資料會不會無法上傳至 Azure？
A. 如果容器名稱包含大寫字母，這些字母會自動轉換為小寫。 如果名稱在其他方面不符合規範 (特殊字元、其他語言等等)，則會上傳失敗。 如需詳細資訊，請移至 [Azure 命名慣例](data-box-disk-limits.md#azure-block-blob-and-page-blob-naming-conventions)。

### <a name="q-how-do-i-verify-the-data-i-copied-onto-multiple-data-box-disks"></a>問： 如何驗證我複製到多個資料箱磁碟上的資料？
A.  資料複製完成後，您可以執行 DataBoxDiskImport 資料夾中提供的 `DataBoxDiskValidation.cmd`，以產生驗證的總和檢查碼。 如果您有多個磁碟，您必須為每個磁碟開啟一個命令視窗，並執行此命令。 請記住，此作業可能十分耗時 (數小時)，視資料大小而定。

### <a name="q-what-happens-to-my-data-after-i-have-returned-the-disks"></a>問： 在我送回磁碟之後，我的資料將會如何？
A.  當資料複製到 Azure 之後，磁碟中的資料將會根據 NIST SP 800-88 修訂 1 指導方針安全地清除。  

### <a name="q-how-is-my-data-protected-during-transit"></a>問： 我的資料在傳輸期間會受到怎樣的保護？ 
A.  資料箱磁碟會使用 AES-128 Microsoft BitLocker 加密進行加密。 必須使用單一通行金鑰才能將所有磁碟解除鎖定並存取資料。

### <a name="q-do-i-need-to-rerun-checksum-validation-if-i-add-more-data-to-the-data-box-disks"></a>問： 如果我將更多資料新增至資料箱磁碟，是否需要重新執行總和檢查碼驗證？
A. 是。 如果您決定要驗證資料 (建議您這麼做)，則必須在新增更多資料磁碟後重新執行驗證。

### <a name="q-i-used-all-my-disks-to-transfer-data-and-need-to-order-more-disks-is-there-a-way-to-quickly-place-the-order"></a>問： 我所有的磁碟都已用來傳輸資料，且需要訂購更多磁碟。 有快速下訂單的方法嗎？
A. 您可以複製您的上一筆訂單。 複製會建立與之前相同的訂單，因此您只需編輯訂單詳細資料即可，而不需要輸入地址、連絡人和通知詳細資料。 



## <a name="next-steps"></a>後續步驟

- 檢閱[資料箱系統需求](data-box-disk-system-requirements.md)。
- 了解[資料箱限制](data-box-disk-limits.md)。
- 在 Azure 入口網站中快速部署 [Azure 資料箱磁碟](data-box-disk-quickstart-portal.md)。
