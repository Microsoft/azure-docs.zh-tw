---
title: Azure 資料箱閘道預覽版本資訊 | Microsoft Docs
description: 描述執行預覽版本的 Azure 資料箱重大未決問題和解決方式。
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 09/24/2018
ms.author: alkohli
ms.openlocfilehash: f5e19d59dfddc3be849700f3678519179b5b39ba
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49164564"
---
# <a name="azure-data-box-gateway-preview-release-notes"></a>Azure 資料箱閘道預覽版本資訊

## <a name="overview"></a>概觀

下列版本資訊指出 Microsoft Azure 資料箱閘道預覽版本的重大未決問題和已解決問題。

版本資訊會持續更新，並在發現需要提出因應措施的重大問題時有所增補。 部署資料箱閘道之前，請仔細檢閱版本資訊中所含的資訊。

對應於軟體版本**資料箱閘道預覽版本 2.0** 的預覽版本。

## <a name="issues-fixed-in-preview-release"></a>預覽版本修正的問題

下表提供本版已修正問題的摘要。

| 否。 | 問題 |
| --- | --- |
| 1 | 在此版本中，如果重新整理另一個工具 (AzCopy) 所上傳的檔案，然後以可增加/擴充檔案大小的方式更新，則會發現到下列錯誤：錯誤 400: InvalidBlobOrBlock (指定的 blob 或區塊內容無效。)|


## <a name="known-issues-in-preview-release"></a>預覽版本的已知問題

下表提供執行預覽版本的資料箱閘道已知問題的摘要。

| 否。 | 功能 | 問題 | 因應措施/註解 |
| --- | --- | --- | --- |
| **1.** |更新 |無法將在舊的預覽版本中建立的資料箱閘道裝置更新為此版本。 |從新版本下載虛擬硬碟映像，並且設定和部署新的裝置。 如需詳細資訊，請移至[準備部署 Azure 資料箱閘道](data-box-gateway-deploy-prep.md)。 |
| **2.** |佈建的資料磁碟 |佈建特定指定大小的資料磁碟並建立對應的資料箱閘道之後，不得壓縮資料磁碟。 嘗試壓縮磁碟會導致裝置上的所有本機資料遺失。 | |
| **3.** |重新整理 |在此版本中，您一次只可以重新整理一個共用。 | |
| **4.** |重新命名 |不支援將物件重新命名。 |如果這項功能對您的工作流程極為重要，請連絡 Microsoft 支援服務。 |
| **5.** |複製| 如果唯讀檔案已複製到裝置，則不會保留唯讀屬性。 | |
| **6.** |記錄檔| 由於此版本有錯誤，您可能會在 error.xml 看到錯誤碼 110 的執行個體 (具有無法辨識的項目名稱)。 | |
| **7.** |上傳 | 由於此版本有錯誤，您可能會在特定檔案上傳期間看到錯誤碼 2003 的執行個體。 | |
| **8.** |檔案類型 | 不支援下列 Linux 檔案類型：字元檔案、區塊檔案、通訊端、管道、符號連結。  |複製這些檔案會導致在 NFS 共用上建立長度為 0 的檔案。 這些檔案處於錯誤狀態，也會在 error.xml 中回報。 |
| **9.** |刪除 | 由於此版本有錯誤，如果已刪除 NFS 共用，則無法刪除此共用。 共用狀態會顯示「刪除中」。  |當共用使用不支援的檔案名稱時，才會發生這種情況。 |
| **10.** |重新整理 | 權限和存取控制清單 (ACL) 不會保留在重新整理作業中。  | |
| **11.** |線上說明 |Azure 入口網站中的說明連結不會連結至文件。|說明連結將會在公開上市版本中運作。 |



## <a name="next-steps"></a>後續步驟

- [準備部署 Azure 資料箱閘道](data-box-gateway-deploy-prep.md)。


