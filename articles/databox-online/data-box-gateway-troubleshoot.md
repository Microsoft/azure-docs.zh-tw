---
title: Azure 資料箱閘道疑難排解 | Micrsoft Docs
description: 描述如何針對 Azure 資料箱閘道問題進行疑難排解。
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: overview
ms.date: 10/09/2018
ms.author: alkohli
ms.openlocfilehash: 00797cdf50104b41777cc8b537c140045677c1a5
ms.sourcegitcommit: c282021dbc3815aac9f46b6b89c7131659461e49
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/12/2018
ms.locfileid: "49167319"
---
# <a name="troubleshoot-your-azure-data-box-gateway-issues"></a>針對 Azure 資料箱閘道問題進行疑難排解 

本文說明如何針對 Azure 資料箱閘道問題進行疑難排解。 

> [!IMPORTANT]
> - 資料箱閘道處於預覽狀態。 部署訂購並部署此解決方案之前，請檢閱 [Azure 預覽版使用條款](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)。

在本文中，您將了解：

> [!div class="checklist"]
> * 執行診斷
> * 收集支援套件
> * 使用記錄進行疑難排解


## <a name="run-diagnostics"></a>執行診斷

若要診斷任何裝置錯誤並進行疑難排解，您可以執行診斷測試。 在您裝置的本機 Web UI 中執行下列步驟，以執行診斷測試。

1. 在本機 Web UI 中，移至 [疑難排解] > [診斷測試]。 選取您想要執行的測試，然後按一下 [執行測試]。 這會執行測試，以診斷您的網路、裝置、Web Proxy、時間或雲端設定的任何可能問題。 您會收到裝置正在執行測試的通知。

    ![按一下 [新增使用者]](media/data-box-gateway-troubleshoot/run-diag-1.png)
 
2. 測試完成後會顯示結果。 如果測試失敗，則會顯示建議動作的 URL。 您可以按一下此 URL，以檢視建議的動作。 
 
    ![按一下 [新增使用者]](media/data-box-gateway-troubleshoot/run-diag-2.png)


## <a name="collect-support-package"></a>收集支援套件

記錄套件包含有助於 Microsoft 支援小組疑難排解任何裝置問題的所有相關記錄。 您可以透過本機 Web UI 產生記錄套件。

執行下列步驟來收集支援套件。 

1. 在本機 Web UI 中，移至 [疑難排解] > [支援]。 按一下 [建立支援套件]。 系統會開始收集支援套件。 套件收集可能需要幾分鐘的時間。

    ![按一下 [新增使用者]](media/data-box-gateway-troubleshoot/collect-logs-1.png)
 
2. 建立支援套件之後，按一下 [下載支援套件]。 壓縮的套件會下載至您所選的路徑。 您可以將此套件解壓縮並檢視系統記錄檔。

    ![按一下 [新增使用者]](media/data-box-gateway-troubleshoot/collect-logs-2.png)

## <a name="use-logs-to-troubleshoot"></a>使用記錄進行疑難排解

在上傳和重新整理程序期間發生的所有錯誤都會包含在各自的錯誤檔案中。

1. 若要檢視錯誤檔案，請移至您的共用，並按一下此共用以檢視內容。 

      ![按一下 [新增使用者]](media/data-box-gateway-troubleshoot/troubleshoot-logs-1.png)

2. 按一下 [Microsoft 資料箱閘道] 資料夾。 這個資料夾有兩個子資料夾：

    - 具有上傳錯誤記錄檔的 [上傳] 資料夾。
    - 在重新整理期間發生錯誤的 [重新整理] 資料夾。

    以下是可供重新整理的範例記錄檔。

    ```
    <root container="brownbag1" machine="VM15BS020663" timestamp="07/18/2018 00:11:10" />
    <file item="test.txt" local="False" remote="True" error="16001" />
    <summary runtime="00:00:00.0945320" errors="1" creates="2" deletes="0" insync="3" replaces="0" pending="9" />
    ``` 

3. 當您在此檔案中看到錯誤 (已在此範例中醒目提示) 時，請記下錯誤碼，在此例中是 16001。 針對下列錯誤參考，查閱此錯誤碼的說明。

    |     錯誤碼     |     例外狀況名稱                                         |     錯誤說明                                                                                                                                                                                                                     |
    |--------------------|------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    |    100             |    ERROR_CONTAINER_OR_SHARE_NAME_LENGTH                    |    容器或共用名稱必須介於 3 到 63 個字元之間。                                                                                                                                                                     |
    |    101             |    ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH        |    容器或共用名稱只能包含字母、數字或連字號。                                                                                                                                                       |
    |    102             |    ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH             |    容器或共用名稱只能包含字母、數字或連字號。                                                                                                                                                       |
    |    103             |    ERROR_BLOB_OR_FILE_NAME_CHARACTER_CONTROL               |    Blob 或檔案名稱包含不支援的控制字元。                                                                                                                                                                       |
    |    104             |    ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL               |    Blob 或檔案名稱包含不合法的字元。                                                                                                                                                                                   |
    |    105             |    ERROR_BLOB_OR_FILE_NAME_SEGMENT_COUNT                   |    Blob 或檔案名稱包含太多區段 (每個區段是以斜線 -/ 分隔)。                                                                                                                                              |
    |    106             |    ERROR_BLOB_OR_FILE_NAME_AGGREGATE_LENGTH                |    Blob 或檔案名稱太長。                                                                                                                                                                                                     |
    |    107             |    ERROR_BLOB_OR_FILE_NAME_COMPONENT_LENGTH                |    Blob 或檔案名稱的其中一個區段太長。                                                                                                                                                                            |
    |    108             |    ERROR_BLOB_OR_FILE_SIZE_LIMIT                           |    檔案大小超出上傳的檔案大小上限。                                                                                                                                                                              |
    |    109             |    ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT                       |    Blob 或檔案未正確地對齊。                                                                                                                                                                                               |
    |    110             |    ERROR_NAME_NOT_VALID_UNICODE                            |    Unicode 編碼的檔案名稱或 Blob 無效。                                                                                                                                                                                  |
    |    111             |    ERROR_RESERVED_NAME_NOT_ALLOWED                         |    檔案或 Blob 的名稱或前置詞是保留的名稱，不受支援 (例如 COM1)。                                                                                                                             |
    |    2000            |    ERROR_ETAG_MISMATCH                                     |    Etag 不符表示雲端中和裝置上的區塊 Blob 之間有衝突。 若要解決此衝突，請刪除其中一個檔案 – 不是在雲端中的版本，就是裝置上的版本。    |
    |    2001            |    ERROR_UNEXPECTED_FINALIZE_FAILURE                       |    在上傳檔案後處理檔案時，發生未預期的問題。    如果您看到此錯誤，而且此錯誤持續超過 24 小時，請連絡支援服務。                                                      |
    |    2002            |    ERROR_ALREADY_OPEN                                      |    檔案已在另一個程序中開啟，而且直到控制代碼關閉才能上傳。                                                                                                                                       |
    |    2003            |    ERROR_UNABLE_TO_OPEN                                    |    無法開啟要上傳的檔案。 如果您看見此錯誤，請連絡 Microsoft 支援服務。                                                                                                                                                |
    |    2004            |    ERROR_UNABLE_TO_CONNECT                                 |    無法連線到容器以將資料上傳至該容器。                                                                                                                                                                             |
    |    2005            |    ERROR_INVALID_CLOUD_CREDENTIALS                         |    無法連線到容器，因為帳戶權限錯誤或過期。 檢查您的存取權。                                                                                                               |
    |    2006            |    ERROR_CLOUD_ACCOUNT_DISABLED                            |    無法將資料上傳到帳戶，因為帳戶或共用已停用。                                                                                                                                                            |
    |    2007            |    ERROR_CLOUD_ACCOUNT_PERMISSIONS                         |    無法連線到容器，因為帳戶權限錯誤或過期。 檢查您的存取權。                                                                                                               |
    |    2008            |    ERROR_CLOUD_CONTAINER_SIZE_LIMIT_REACHED                |    無法新增資料，因為容器已滿。 根據類型，檢查 Azure 規格中支援的容器大小。 例如，Azure 檔案服務僅支援 5 TB 的最大檔案大小。                                     |
    |    2998            |    ERROR_UNMAPPED_FAILURE                                  |    發生意外錯誤。 此錯誤可能會自行解決，但若持續超過 24 小時，請連絡 Microsoft 支援服務。                                                                                                     |
    |    16000           |    RefreshException                                        |    無法關閉此檔案。                                                                                                                                                                                                        |
    |    16001           |    RefreshAlreadyExistsException                           |    無法關閉此檔案，因為它已經存在於本機系統上。                                                                                                                                                         |
    |    16002           |    RefreshWorkNeededException                              |    無法重新整理此檔案，因為它並未完整上傳。                                                                                                                                                                          | 


## <a name="next-steps"></a>後續步驟

- 深入了解[此版本中的已知問題](data-box-gateway-release-notes.md)。
