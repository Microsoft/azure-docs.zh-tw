---
title: Azure 儲存體簡介 - Azure 中的雲端儲存體 | Microsoft Docs
description: Azure 儲存體是 Microsoft 的雲端儲存體解決方案。 Azure 儲存體提供了高可用性、安全、持久、可大幅調整且具有備援的資料物件儲存體。
services: storage
author: tamram
ms.service: storage
ms.topic: get-started-article
ms.date: 07/11/2018
ms.author: tamram
ms.component: common
ms.openlocfilehash: f03872a125e4e4d2f1e0b458a31d67a07802f2c8
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49387442"
---
# <a name="introduction-to-azure-storage"></a>Azure 儲存體簡介

Azure 儲存體是 Microsoft 針對最新資料儲存體環境推出的雲端儲存體解決方案。 Azure 儲存體提供可大幅調整的資料物件存放區、雲端檔案系統服務、能可靠傳訊的訊息存放區，以及 NoSQL 存放區。 Azure 儲存體的特色：

- **持久與高可用性。** 備援功能可在發生暫時性硬體失敗時，確保您的資料安全無虞。 您也可以選擇在資料中心或地理區域間複寫資料，以便在發生地方性災難或天然災害時獲得額外保護。 以此方式複寫資料，可在發生未預期的中斷事件時保持高可用性。 
- **安全。** 所有寫入 Azure 儲存體的資料皆會由服務進行加密。 Azure 儲存體在存取您資料的人員控管上，提供更細微的控制。
- **可調整。** Azure 儲存體設計為可大幅調整，以符合現今應用程式的資料儲存和效能需求。 
- **受控。** Microsoft Azure 會為您處理維護作業和任何重大問題。
- **可存取。** 您可以從世界各地透過 HTTP 或 HTTPS 存取 Azure 儲存體中的資料。 Microsoft 提供多種語言的 Azure 儲存體 SDK (.NET、Java、Node.js、Python、PHP、Ruby、Go 等) 和成熟的 REST API。 Azure 儲存體支援在 Azure PowerShell 或 Azure CLI 中使用指令碼。 而且在使用資料方面，Azure 入口網站和 Azure 儲存體總管提供簡易的視覺式解決方案。  

## <a name="azure-storage-services"></a>Azure 儲存體服務

Azure 儲存體包含以下資料服務： 

- [Azure Blob](../blobs/storage-blobs-introduction.md)：適用於文字和二進位資料且可大幅調整的物件存放區。
- [Azure 檔案服務](../files/storage-files-introduction.md)：雲端或內部部署的受控檔案共用。
- [Azure 佇列](../queues/storage-queues-introduction.md)：可在應用程式元件之間可靠傳訊的訊息存放區。 
- [Azure 資料表](../tables/table-storage-overview.md)：以無結構描述方式儲存結構化資料的 NoSQL 存放區。

每個服務都會透過儲存體帳戶存取。 若要開始使用，請參閱[建立儲存體帳戶](storage-quickstart-create-account.md)。

## <a name="blob-storage"></a>Blob 儲存體

Azure Blob 儲存體是 Microsoft 針對雲端推出的物件儲存體解決方案。 Blob 儲存體已針對儲存大量非結構化物件資料 (例如文字或二進位資料) 最佳化。 

Blob 儲存體是適合用於：

* 直接提供映像或文件給瀏覽器。
* 儲存檔案供分散式存取。
* 串流傳輸視訊和音訊。
* 儲存備份和還原、災害復原和封存資料。
* 儲存資料供內部部署或 Azure 託管服務進行分析。

使用者可從世界各地透過 HTTP 或 HTTPS 存取 Blob 儲存體中的物件。 使用者或用戶端應用程式可以透過 URL、[Azure 儲存體 REST API](https://docs.microsoft.com/rest/api/storageservices/blob-service-rest-api)、[Azure PowerShell](https://docs.microsoft.com/powershell/module/azure.storage)、[Azure CLI](https://docs.microsoft.com/cli/azure/storage) 或 Azure 儲存體用戶端程式庫存取 Blob。 儲存體用戶端程式庫提供多種語言，包括 [.NET](https://docs.microsoft.com/dotnet/api/overview/azure/storage/client)、[Java](https://docs.microsoft.com/java/api/overview/azure/storage/client)、[Node.js](http://azure.github.io/azure-storage-node)、[Python](https://azure-storage.readthedocs.io/)、[PHP](http://azure.github.io/azure-storage-php/) 和 [Ruby](http://azure.github.io/azure-storage-ruby)。

若要深入了解 Blob 儲存體，請參閱 [Azure 上的物件儲存體簡介](../blobs/storage-blobs-introduction.md)。

## <a name="azure-files"></a>Azure 檔案
[Azure 檔案服務](../files/storage-files-introduction.md)可讓您設定高可用性網路檔案共用，其可使用標準伺服器訊息區塊 (SMB) 通訊協定來存取。 這表示多個 VM 可以透過讀取和寫入權限共用相同的檔案。 您也可以使用 REST 介面或儲存體用戶端程式庫來讀取檔案。

區分 Azure 檔案服務與公司檔案共用上的檔案的方法之一，就是您可以使用指向檔案並包含共用存取簽章 (SAS) 權杖的 URL，從世界各地存取檔案。 您可以產生 SAS 權杖；SAS 權杖可允許特定一段時間內私人資產的特定存取。

檔案共用可以用於許多常見案例：

* 許多內部部署應用程式會使用檔案共用。 這項功能可讓您更輕鬆地將共用資料的應用程式移轉至 Azure。 如果您將檔案共用掛接至內部部署應用程式使用的相同磁碟機代號，則存取檔案共用的應用程式部分應會在變動最小 (如果有的話) 的情況下運作。

* 組態檔可以儲存在檔案共用上並從多個 VM 進行存取。 由群組中多個開發人員所用的工具和公用程式可以儲存於檔案共用，確保所有人都可以找到它們並使用相同的版本。

* 可以寫入檔案共用且稍後進行處理或分析的資料範例有三個：診斷記錄、計量和損毀傾印。

這個階段不支援以 Active Directory 為基礎的驗證和存取控制清單 (ACL)，但未來將會提供支援。 儲存體帳戶認證用來提供檔案共用存取權的驗證。 這表示任何掛接共用的人員都具有共用的完整讀取/寫入權限。

如需 Azure 檔案服務的詳細資訊，請參閱 [Azure 檔案服務簡介](../files/storage-files-introduction.md)。

## <a name="queue-storage"></a>佇列儲存體

Azure 佇列服務用來儲存及擷取訊息。 佇列訊息的大小上限為 64 KB，而一個佇列可以包含數百萬則訊息。 佇列通常用來儲存要以非同步方式處理的訊息清單。

例如，假設您希望客戶能夠上傳圖片，而且要建立每張圖片的縮圖。 您可以讓客戶在上傳圖片時等候您建立縮圖。 另外，也可以使用佇列。 當客戶完成上傳時，將訊息寫入佇列。 然後讓 Azure Function 從佇列擷取訊息並建立縮圖。 這項處理的每個部分都可以個別調整，讓您在針對您的使用量進行微調時有更多控制權。

如需 Azure 佇列的詳細資訊，請參閱[佇列簡介](../queues/storage-queues-introduction.md)。

## <a name="table-storage"></a>表格儲存體

Azure 資料表儲存體現在屬於 Azure Cosmos DB。 若要查看 Azure 資料表儲存體文件，請參閱 [Azure 資料表儲存體概觀](../tables/table-storage-overview.md)。 除了現有的 Azure 資料表儲存體服務，有新的 Azure Cosmos DB 資料表 API 供應項目，可提供輸送量最佳化的資料表、全域發佈，以及自動次要索引。 若要深入了解並試用新的進階體驗，請查看 [Azure Cosmos DB 資料表 API](https://aka.ms/premiumtables)。

如需資料表儲存體的詳細資訊，請參閱 [Azure 資料表儲存體概觀](../tables/table-storage-overview.md)。

## <a name="disk-storage"></a>磁碟儲存體

Azure 儲存體也包含虛擬機器所使用的受控和非受控磁碟功能。 如需有關這些功能的詳細資訊，請參閱[計算服務文件](https://docs.microsoft.com/azure/#pivot=products&panel=Compute)。

## <a name="types-of-storage-accounts"></a>儲存體帳戶類型

下表顯示各種儲存體帳戶，以及各自可以使用的物件。

|**儲存體帳戶的類型**|**一般用途：標準**|**一般用途：進階**|**Blob 儲存體、經常性存取和非經常性存取層**|
|-----|-----|-----|-----|
|**支援的服務**| Blob、檔案、佇列和資料表服務 | Blob 服務 | Blob 服務|
|**支援的 Blob 類型**|區塊 Blob、分頁 Blob 及附加 Blob | 分頁 Blob | 區塊 Blob 和附加 Blob|

### <a name="general-purpose-storage-accounts"></a>一般用途的儲存體帳戶

一般用途的儲存體帳戶有兩種。

#### <a name="standard-storage"></a>標準儲存體

標準儲存體帳戶是最廣泛使用的儲存體帳戶，可用於所有類型的資料。 標準儲存體帳戶會使用磁性媒體來儲存資料。

#### <a name="premium-storage"></a>進階儲存體

進階儲存體可為分頁 blob 提供高效能儲存體，主要用於 VHD 檔案。 進階儲存體帳戶使用 SSD 來儲存資料。 Microsoft 建議對所有 VM 使用進階儲存體。

### <a name="blob-storage-accounts"></a>Blob 儲存體帳戶

Blob 儲存體帳戶是專門的儲存體帳戶，用來儲存區塊 blob 和附加 blob。 您無法將分頁 blob 儲存在這些帳戶中，因此無法儲存 VHD 檔案。 這些帳戶允許您將存取層設定為經常性存取或非經常性存取；存取層可以隨時變更。

經常性存取層用於經常存取的檔案 - 您需支付較高的儲存體成本，但存取 blob 的成本低很多。 對於非經常性存取層中儲存的 blob，您需支付較高的 blob 存取成本，但儲存體的成本低很多。

## <a name="accessing-your-blobs-files-and-queues"></a>存取您的 blob、檔案和佇列

每個儲存體帳戶都有兩種驗證金鑰，任一種金鑰均可用於任何作業。 金鑰有兩個，所以您可以偶爾變換金鑰以增強安全性。 務必將金鑰放在安全的地方，因為其擁有權 (連同帳戶名稱) 允許無限制存取儲存體帳戶中的所有資料。

本節說明保護儲存體帳戶及其資料的兩種方式。 如需保護儲存體帳戶和資料的詳細資訊，請參閱 [Azure 儲存體安全性指南](storage-security-guide.md)。

### <a name="securing-access-to-storage-accounts-using-azure-ad"></a>使用 Azure AD 保護儲存體帳戶的存取權

控制儲存體帳戶金鑰的存取權，是保護儲存體資料存取的其中一種方式。 使用 Resource Manager 角色型存取控制 (RBAC)，您可以將角色指派給使用者、群組或應用程式。 這些角色會繫結至特定一組允許或不允許的動作。 使用 RBAC 授與儲存體帳戶存取權時，只會處理該儲存體帳戶的管理作業，例如變更存取層。 您無法使用 RBAC 來授與資料物件 (如特定容器或檔案共用) 的存取權。 不過，您可以使用 RBAC 授與儲存體帳戶金鑰的存取權，而這些金鑰可用來讀取資料物件。

### <a name="securing-access-using-shared-access-signatures"></a>使用共用存取簽章保護存取權

您可以使用共用存取簽章和預存存取原則來保護您的資料物件。 共用存取簽章 (SAS) 是一個字串，包含可附加至資產 URI 的安全性權杖，可讓您委派特定儲存體物件的存取權，以及指定存取的權限和日期/時間範圍之類的限制。 這項功能有廣泛的功能。 如需詳細資訊，請參閱[使用共用存取簽章 (SAS)](storage-dotnet-shared-access-signature-part-1.md)。

### <a name="public-access-to-blobs"></a>公開存取 Blob

Blob 服務可讓您提供容器與其 Blob 或特定 Blob 的公開存取權。 當您將容器或 Blob 指定為公用時，任何人都可以進行匿名讀取；不需要驗證。 例如，當您有網站使用 Blob 儲存體中的影像、影片或文件時，您會想要執行此動作。 如需詳細資訊，請參閱[管理對容器與 Blob 的匿名讀取權限](../blobs/storage-manage-access-to-resources.md)

## <a name="encryption"></a>加密

有兩個基本加密類型可用於儲存體服務。 如需安全性和加密的詳細資訊，請參閱 [Azure 儲存體安全性指南](storage-security-guide.md)。

### <a name="encryption-at-rest"></a>待用加密

待用的 Azure 儲存體服務加密 (SSE) 會協助您保護資料安全，以符合組織安全性和合規性承諾。 利用此功能，Azure 儲存體會自動加密資料，再保存到儲存體，以及在擷取之前解密。 以完全無感的方式處理所有加密、解密和金鑰管理。


SSE 會自動將所有效能層 (標準和進階)、所有部署模型 (Azure Resource Manager 和傳統) 以及所有「Azure 儲存體」服務 (Blob、佇列、資料表和檔案) 中的資料加密。 SSE 不會影響「Azure 儲存體」效能。

如需待用 SSE 加密的詳細資訊，請參閱[待用資料的 Azure 儲存體服務加密](storage-service-encryption.md)。

### <a name="client-side-encryption"></a>用戶端加密

儲存體用戶端程式庫具有您可以呼叫的方法，在透過網路將資料從用戶端傳送到 Azure 之前，可以程式設計方式將資料加密。 它會以加密方式儲存，這表示待用時也會加密。 讀回資料時，您會在接收資訊之後加以解密。

如需用戶端加密的詳細資訊，請參閱 [Microsoft Azure 儲存體的用戶端 .NET 加密](storage-client-side-encryption.md)。

## <a name="replication"></a>複寫

為了確保資料的持久性，Azure 儲存體會為您複寫多個資料複本。 當您設定儲存體帳戶時，您可選取複寫類型。 在大部分情況下，建立儲存體帳戶後即可修改此設定。 

[!INCLUDE [storage-common-redundancy-options](../../../includes/storage-common-redundancy-options.md)]

如需災害復原資訊，請參閱[如果 Azure 儲存體發生中斷怎麼辦](storage-disaster-recovery-guidance.md)。

## <a name="transferring-data-to-and-from-azure-storage"></a>從 Azure 儲存體來回傳輸資料

您可以使用 AzCopy 命令列公用程式，在儲存體帳戶內或跨儲存體帳戶複製 Blob 和檔案資料。 請參閱下列其中一篇文章中的說明：

* [使用適用於 Windows 的 AzCopy 傳輸資料](storage-use-azcopy.md)
* [使用適用於 Linux 的 AzCopy 傳輸資料](storage-use-azcopy-linux.md)

AzCopy 建置於 [Azure 資料移動程式庫](https://www.nuget.org/packages/Microsoft.Azure.Storage.DataMovement/)之上，其目前可供預覽。

Azure 匯入/匯出服務可用於從儲存體帳戶匯入或匯出大量 blob 資料。 您可準備多個硬碟並將其郵寄至 Azure 資料中心，這些硬碟會在其中從硬碟機傳入/傳出資料並將硬碟機傳送給您。 如需匯入/匯出服務的詳細資訊，請參閱 [使用 Microsoft Azure 匯入/匯出服務將資料傳輸至 Blob 儲存體](../storage-import-export-service.md)。

若要以快速、便宜且可靠的方式，將大量 blob 資料匯入您的儲存體帳戶中，您也可以使用 Azure 資料箱磁碟。 Microsoft 會透過區域電訊廠商，將最多 5 個包含 40 TB 容量的加密固態硬碟 (SSD) 寄送至您的資料中心。 您可快速設定磁碟、透過 USB 連線將資料複製到磁碟，並將磁碟寄回 Azure。 在 Azure 資料中心內，您的資料會從磁碟自動上傳至雲端。 如需有關此解決方案的詳細資訊，請移至 [Azure 資料箱磁碟概觀](https://docs.microsoft.com/azure/databox/data-box-disk-overview)。

## <a name="pricing"></a>價格

如需 Azure 儲存體價格的詳細資訊，請參閱[價格頁面](https://azure.microsoft.com/pricing/details/storage/blobs/)。

## <a name="storage-apis-libraries-and-tools"></a>儲存體 API、程式庫和工具
任何可提出 HTTP/HTTPS 要求的語言皆可存取 Azure 儲存體資源。 另外，Azure 儲存體還提供了幾種熱門語言的程式設計程式庫。 這些程式庫可透過處理詳細資料 (例如同步和非同步叫用、進行批次作業、例外狀況管理、自動重試、運作方式等等) 來簡化使用 Azure 儲存體的許多項目。 程式庫目前適用於下列語言和平台，以及正在研發的其他語言和平台：

### <a name="azure-storage-data-api-and-library-references"></a>Azure 儲存體資料 API 和程式庫參考
* [儲存體服務 REST API](https://docs.microsoft.com/rest/api/storageservices/)
* [適用於 .NET 的儲存體用戶端程式庫](https://docs.microsoft.com/dotnet/api/overview/azure/storage)
* [適用於 Java/Android 的儲存體用戶端程式庫](https://docs.microsoft.com/java/api/overview/azure/storage)
* [適用於 Node.js 的儲存體用戶端程式庫](https://docs.microsoft.com/javascript/api/azure-storage)
* [適用於 Python 的儲存體用戶端程式庫](https://github.com/Azure/azure-storage-python)
* [適用於 PHP 的儲存體用戶端程式庫](https://github.com/Azure/azure-storage-php)
* [適用於 Ruby 的儲存體用戶端程式庫](https://github.com/Azure/azure-storage-ruby)
* [Storage Client Library for C++](https://github.com/Azure/azure-storage-cpp)

### <a name="azure-storage-management-api-and-library-references"></a>Azure 儲存體管理 API 和程式庫參考
* [儲存體資源提供者 REST API](https://docs.microsoft.com/rest/api/storagerp/)
* [適用於 .NET 的儲存體資源提供者用戶端程式庫](https://docs.microsoft.com/dotnet/api/overview/azure/storage/management)
* [儲存體服務管理 REST API (傳統)](https://msdn.microsoft.com/library/azure/ee460790.aspx)

### <a name="azure-storage-data-movement-api-and-library-references"></a>Azure 儲存體資料移動 API 和程式庫參考
* [儲存體匯入/匯出服務 REST API](https://docs.microsoft.com/rest/api/storageimportexport/)
* [適用於 .NET 的儲存體資料移動 用戶端程式庫](https://docs.microsoft.com/dotnet/api/microsoft.windowsazure.storage.datamovement)

### <a name="tools-and-utilities"></a>工具和公用程式
* [儲存體的 Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/module/azure.storage)
* [儲存體的 Azure CLI Cmdlet](https://docs.microsoft.com/cli/azure/storage)
* [AzCopy 命令列公用程式](http://aka.ms/downloadazcopy)
* [Azure 儲存體總管](https://azure.microsoft.com/features/storage-explorer/) 是一個免費的獨立應用程式，可讓您在 Windows、MacOS 和 Linux 上以視覺化方式處理 Azure 儲存體資料。
* [Azure 儲存體用戶端工具](../storage-explorers.md)
* [Azure 開發人員工具](https://azure.microsoft.com/tools/)

## <a name="next-steps"></a>後續步驟

若要讓 Azure 儲存體啟動並執行，請參閱[建立儲存體帳戶](storage-quickstart-create-account.md)。
