---
title: 建立 Azure 匯入/匯出的匯入作業 | Microsoft Docs
description: 了解如何建立 Microsoft Azure 匯入/匯出服務的匯入。
author: muralikk
services: storage
ms.service: storage
ms.topic: article
ms.date: 01/23/2017
ms.author: muralikk
ms.component: common
ms.openlocfilehash: e87b791dda03521def32ca7473336d7d37644773
ms.sourcegitcommit: 31241b7ef35c37749b4261644adf1f5a029b2b8e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/04/2018
ms.locfileid: "43666014"
---
# <a name="creating-an-import-job-for-the-azure-importexport-service"></a>建立 Azure 匯入/匯出服務的匯入作業

使用 REST API 建立 Microsoft Azure 匯入/匯出服務的匯入作業包含下列步驟︰

-   利用 Azure 匯入/匯出工具來準備磁碟機。

-   取得要寄送磁碟機的位置。

-   建立匯入作業。

-   透過支援的貨運服務將磁碟機寄送給 Microsoft。

-   使用出貨詳細資料更新匯入工作。

 請參閱[使用 Microsoft Azure 匯入/匯出服務將資料移轉至 Blob 儲存體](storage-import-export-service.md)以取得匯入/匯出服務概觀及示範如何使用 [Azure 入口網站](https://portal.azure.com/)建立和管理匯入與匯出作業的教學課程。

## <a name="preparing-drives-with-the-azure-importexport-tool"></a>準備具有 Azure 匯入/匯出工具的磁碟機

無論您是透過入口網站或透過 REST API 建立作業，準備匯入作業的磁碟機步驟都相同。

以下是磁碟機準備的簡要概觀。 請參閱 [Azure 匯入匯出工具參考](storage-import-export-tool-how-to-v1.md)以取得完整指示。 您可以在[這裡](http://go.microsoft.com/fwlink/?LinkID=301900)下載 Azure 匯入/匯出工具。

準備您的磁碟機包含︰

-   識別要匯入的資料。

-   在 Windows Azure 儲存體中識別目的地 blob。

-   使用 Azure 匯入/匯出工具，將資料複製到一或多個硬碟。

 Azure 匯入/匯出工具也會在備妥時，針對每個磁碟機產生資訊清單檔案。 資訊清單檔案包含︰

-   適用於上傳和從這些檔案對應至 blob 的所有檔案之列舉型別。

-   每個檔案區段的總和檢查碼。

-   與每個 blob 相關聯的中繼資料及內容的相關資訊。

-   如果要上傳的 blob 與容器中現有的 blob 具有相同名稱時，要採取的動作清單。 可能的選項包括︰a) 以 blob 覆寫檔案、b) 保留現有的 blob 並略過上傳檔案、c) 在名稱後面附加尾碼，使它不會與其他檔案衝突。

## <a name="obtaining-your-shipping-location"></a>取得寄送位置

在建立匯入作業之前，您需要藉由呼叫 [List Locations](https://docs.microsoft.com/rest/api/storageimportexport/locations/list) 作業來取得寄送位置名稱和地址。 `List Locations` 會傳回位置及其郵寄地址的清單。 您可以從傳回的清單中選取位置，並將您的硬碟寄送至該地址。 您也可以使用 `Get Location` 作業來直接取得特定位置的寄送地址。

 請遵循下列步驟來取得寄送位置︰

-   識別儲存體帳戶位置的名稱。 您可以在 Azure 入口網站中儲存體帳戶「儀表板」上的 [位置] 欄位下找到此值，或使用服務管理 API 作業[取得儲存體帳戶屬性](/rest/api/storagerp/storageaccounts#StorageAccounts_GetProperties)來查詢此值。

-   藉由呼叫 `Get Location` 作業來擷取可用來處理此儲存體帳戶的位置。

-   如果位置的 `AlternateLocations` 屬性包含位置本身，則可以使用此位置。 否則，使用其中一個替代位置再次呼叫 `Get Location` 作業。 原始位置可能暫時關閉進行維護。

## <a name="creating-the-import-job"></a>建立匯入作業
若要建立匯入作業，請呼叫 [Put Job](/rest/api/storageimportexport/jobs#Jobs_CreateOrUpdate) 作業。 您必須提供下列資訊：

-   作業的名稱。

-   儲存體帳戶名稱。

-   從上一個步驟中取得的寄送位置名稱。

-   作業類型 (匯入)。

-   在匯入作業完成之後要傳送磁碟機的傳回位址。

-   在作業中的磁碟機清單。 針對每個磁碟機，您必須包含下列磁碟機準備步驟期間取得的資訊︰

    -   磁碟機識別碼

    -   BitLocker 金鑰

    -   硬碟上的資訊清單檔案相對路徑

    -   Base16 編碼資訊清單檔案 MD5 雜湊

## <a name="shipping-your-drives"></a>寄送您的磁碟機
您必須將磁碟機寄送到從上一個步驟取得的地址，且您必須提供具有包裹追蹤號碼的匯入/匯出服務。

> [!NOTE]
>  您必須透過支援的貨運服務公司 (會提供您的包裹追蹤號碼) 寄送您的磁碟機。

## <a name="updating-the-import-job-with-your-shipping-information"></a>使用出貨資料更新匯入作業
在您取得追蹤號碼之後，請呼叫 [Update Job Properties](/api/storageimportexport/jobs#Jobs_Update) 作業以更新貨運公司名稱、作業的追蹤號碼和退貨的貨運公司帳戶號碼。 您可以選擇性指定磁碟機數目及寄送日期。

[!INCLUDE [storage-import-export-delete-personal-info.md](../../../includes/storage-import-export-delete-personal-info.md)]

## <a name="next-steps"></a>後續步驟

* [使用匯入/匯出服務 REST API](storage-import-export-using-the-rest-api.md)
