---
title: 使用 Azure 匯入/匯出將資料傳入和傳出 Azure 儲存體 | Microsoft Docs
description: 了解如何在 Azure 入口網站中建立匯入和匯出作業，以將資料傳入和傳出 Azure 儲存體。
author: alkohli
services: storage
ms.service: storage
ms.topic: article
ms.date: 07/11/2018
ms.author: alkohli
ms.component: common
ms.openlocfilehash: 480d67917abf3a8aaca64aa9aae30be5acf55e11
ms.sourcegitcommit: 9819e9782be4a943534829d5b77cf60dea4290a2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/06/2018
ms.locfileid: "39528551"
---
# <a name="what-is-azure-importexport-service"></a>什麼是 Azure 匯入/匯出服務？

Azure 匯入/匯出服務可用來將硬碟運送到 Azure 資料中心，安全地將大量資料匯入 Azure Blob 儲存體和 Azure 檔案服務中。 這項服務也能用來將資料從 Azure Blob 儲存體傳輸到磁碟機，然後運送到您的內部部署網站。 來自一或多個磁碟機的資料可匯入 Azure Blob 儲存體或 Azure 檔案服務中。 

提供您自己的磁碟機，並使用 Azure 匯入/匯出服務來傳輸資料。 您也可以使用由 Microsoft 提供的磁碟機。 

如果您想要使用由 Microsoft 提供的磁碟機來傳輸資料，則可以使用 [Azure 資料箱磁碟](../../databox/data-box-disk-overview.md)將資料匯入到 Azure。 Microsoft 會透過區域貨運廠商，將每筆訂單最多 5 個包含 40 TB 總容量的加密固態硬碟機 (SSD) 寄送至您的資料中心。 您可快速設定磁碟機、透過 USB 3.0 連線將資料複製到磁碟機，並將磁碟機寄回 Azure。 如需詳細資訊，請移至 [Azure 資料箱磁碟概觀](../../databox/data-box-disk-overview.md)。

## <a name="azure-importexport-usecases"></a>Azure 匯入/匯出使用案例

請考慮在透過網路上傳或下載資料速度太慢，或是取得額外的網路頻寬成本高昂時使用 Azure 匯入/匯出服務。 請在下列情節中使用此服務：

* **將資料移轉至雲端**︰快速地將大量資料移至 Azure，並符合成本效益。
* **內容發佈**︰快速將資料傳送至您的客戶網站。
* **備份**︰備份內部部署資料以便儲存在 Azure 儲存體中。
* **資料復原**︰復原儲存在儲存體中的大量資料，並將其傳遞到您的內部部署位置。

## <a name="importexport-components"></a>匯入/匯出元件

匯入/匯出服務會使用下列元件：

- **匯入/匯出服務**：Azure 入口網站中提供的這個服務可協助使用者建立及追蹤資料匯入 (上傳) 和匯出 (下載) 作業。  

- **WAImportExport 工具**：此為執行下列項目的命令列工具： 
    - 將運送的磁碟機準備好進行匯入。
    - 協助將資料複製到磁碟機。
    - 使用 BitLocker 將磁碟機上的資料加密。
    - 產生匯入建立期間所使用的磁碟機日誌檔案。
    - 協助識別匯出作業所需的磁碟機數目。
    
> [!NOTE]
> WAImportExport 工具提供兩個版本，第 1 版與第 2 版。 建議您使用：
> - 第 1 版來匯入/匯出到 Azure Blob 儲存體。 
> - 第 2 版將資料匯入 Azure 檔案中。
>
> WAImportExport 工具只與 64 位元 Windows 作業系統相容。 如需了解支援的特定作業系統版本，請移至 [Azure 匯入/匯出需求](storage-import-export-requirements.md#supported-operating-systems)。

- **磁碟機**：您可以將固態硬碟 (SSD) 或硬碟 (HDD) 運送至 Azure 資料中心。 當您建立匯入作業時，要運送包含您資料的磁碟機。 當您建立匯出作業時，要將空的磁碟機運送至 Azure 資料中心。 如需了解特定的磁碟類型，請移至[支援的磁碟類型](storage-import-export-requirements.md#supported-hardware)。

## <a name="how-does-importexport-work"></a>匯入/匯出的運作方式為何？

Azure 匯入/匯出服務可讓您建立作業，將資料轉送到 Azure Blob 和 Azure 檔案服務。 使用 Azure 入口網站或 Azure Resource Manager REST API 來建立作業。 每個工作都與單一儲存體帳戶相關聯。 

作業可以是匯入或匯出作業。 匯入作業可讓您將資料匯入 Azure Blob 或 Azure 檔案，而匯出作業可讓您從 Azure Blob 中匯出資料。 若為匯入作業，您要運送包含您資料的磁碟機。 當您建立匯出作業時，要將空的磁碟機運送至 Azure 資料中心。 在每個案例中，您可以針對每個作業運送最多 10 個硬碟。

### <a name="inside-an-import-job"></a>匯入作業之內

概括而言，匯入工作包含下列步驟︰

1. 決定要匯入的資料、您所需的磁碟機數目、您 Azure 儲存體中資料的目的地 blob 位置。
2. 請使用 WAImportExport 工具，將資料複製到磁碟機。 使用 BitLocker 將磁碟機加密。
3. 請在 Azure 入口網站的目標儲存體帳戶中建立匯入作業。 上傳磁碟機日誌檔案。
4. 提供送回的寄件地址和貨運公司客戶編號，以便將磁碟機送回給您。
5. 將磁碟機運送到建立作業期間所提供的運送地址。
6. 在匯入工作詳細資料中更新貨件追蹤號碼，並提交匯入工作。
7. Azure 資料中心會收到磁碟機並進行處理。
8. 使用您的貨運業者帳戶，將磁碟機送到匯入作業中提供的寄件地址。

> [!NOTE]
> 對於本地 (在資料中心所在國家/地區內) 的貨物運送，請提供國內貨運業者帳戶 
>
> 對於國外 (在資料中心所在國家/地區之外) 的貨物運送，請提供國際貨運業者帳戶

 ![圖 1: 匯入工作流程](./media/storage-import-export-service/importjob.png)

如需有關資料匯入的逐步指示，請移至：

- [將資料匯入到 Azure Blob](storage-import-export-data-to-blobs.md)
- [將資料匯入到 Azure 檔案服務](storage-import-export-data-to-files.md)


### <a name="inside-an-export-job"></a>匯出作業之內

> [!IMPORTANT]
> 服務僅支援匯出 Azure Blob。 不支援將 Azure 檔案匯出。

概括而言，匯出工作包含下列步驟︰

1. 決定要匯出的資料、您所需的磁碟機數目、您 Blob 儲存體中資料的來源 Blob 或容器路徑。
3. 請在 Azure 入口網站的來源儲存體帳戶中建立匯出作業。
4. 指定所要匯出資料的來源 Blob 或容器路徑。
5. 提供送回的寄件地址和貨運公司客戶編號，以便將磁碟機送回給您。
6. 將磁碟機運送到建立作業期間所提供的運送地址。
7. 在匯出工作詳細資料中更新貨件追蹤號碼，並提交匯出工作。
8. Azure 資料中心會收到磁碟機並進行處理。
9. 磁碟機會使用 BitLocker 加密，且可透過 Azure 入口網站取得金鑰。  
10. 使用您的貨運業者帳戶，將磁碟機送到匯入作業中提供的寄件地址。

> [!NOTE]
> 對於本地 (在資料中心所在國家/地區內) 的貨物運送，請提供國內貨運業者帳戶 
>
> 對於國外 (在資料中心所在國家/地區之外) 的貨物運送，請提供國際貨運業者帳戶
  
 ![圖 2: 匯出工作流程](./media/storage-import-export-service/exportjob.png)

如需關於匯出資料的逐步指示，請移至[從 Azure Blob 匯出資料](storage-import-export-data-from-blobs.md)。

## <a name="region-availability"></a>區域可用性 

Azure 匯入/匯出服務支援與所有 Azure 儲存體帳戶相互複製資料。 您可以將磁碟機運送到列出的其中一個位置。 如果您的儲存體帳戶是位於這裡所未指定的 Azure 位置，則當您建立作業時，會提供替代的運送位置。

### <a name="supported-shipping-locations"></a>支援的運送位置


|國家 (地區)  |國家 (地區)  |國家 (地區)  |國家 (地區)  |
|---------|---------|---------|---------|
|美國東部    | 北歐        | 印度中部        |US Gov 愛荷華州         |
|美國西部     |西歐         | 印度南部        | 美國 DoD 東部        |
|美國東部 2    | 東亞        |  印度西部        | 美國國防部中央        |
|美國西部 2     | 東南亞        | 加拿大中部        | 中國東部         |
|美國中部     | 澳洲東部        | 加拿大東部        | 中國北部        |
|美國中北部     |  澳大利亞東南部       | 巴西南部        | 英國南部        |
|美國中南部     | 日本西部        |南韓中部         | 德國中部        |
|美國中西部     |  日本東部       | 美國政府維吉尼亞州        | 德國東北部        |


## <a name="security-considerations"></a>安全性考量

磁碟機上的資料會使用「BitLocker 磁碟機加密」來進行加密。 此加密會在資料傳輸時保護資料。

針對匯入作業，會使用兩種方式來加密磁碟機。  


- 在磁碟機準備期間執行 WAImportExport 工具時，於使用 dataset.csv 檔案時指定選項。 

- 在磁碟機上手動啟用 BitLocker 加密。 在磁碟機準備期間，於執行 WAImportExport 工具命令列時，在 driveset.csv 中指定加密金鑰。


針對匯出作業，您的資料複製到磁碟機之後，服務會使用 BitLocker 將磁碟機加密，然後才運送回去給您。 加密金鑰會透過 Azure 入口網站提供給您。

[!INCLUDE [storage-import-export-delete-personal-info.md](../../../includes/storage-import-export-delete-personal-info.md)]


### <a name="pricing"></a>價格

**磁碟機處理費用**

在匯入或匯出工作當中，對於所處理的每個磁碟機會有一項磁碟機處理費用。 請查看 [Azure 匯入/匯出價格](https://azure.microsoft.com/pricing/details/storage-import-export/)的詳細資料。

**運送成本**

當您運送磁碟機至 Azure 時，您要支付運送成本給運送的貨運業者。 當 Microsoft 將磁碟機送回給您時，運送成本將計入您在建立作業時提供的貨運業者帳戶。

**交易成本**

將資料匯入 Azure 儲存體時，除了標準儲存體交易成本之外，無需支付其他的交易成本。 從 Blob 儲存體匯出資料時，適用標準輸出費用。 如需交易成本的更多資訊，請參閱[資料轉送價格](https://azure.microsoft.com/pricing/details/data-transfers/)。



## <a name="next-steps"></a>後續步驟

了解如何使用匯入/匯出服務進行：
* [將資料匯入 Azure Blob](storage-import-export-data-to-blobs.md)
* [從 Azure Blob 匯出資料](storage-import-export-data-from-blobs.md)
* [將資料匯入 Azure 檔案服務](storage-import-export-data-to-files.md)

