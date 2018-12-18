---
title: 對 Azure 中的雲端儲存體應用程式進行疑難排解及監視 | Microsoft Docs
description: 使用診斷工具、計量和警示，針對雲端應用程式進行疑難排解及監視。
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 07/20/2018
ms.author: tamram
ms.custom: mvc
ms.openlocfilehash: ff4d39a69df90c31186153c077b1b6183a08dd34
ms.sourcegitcommit: d4c076beea3a8d9e09c9d2f4a63428dc72dd9806
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/01/2018
ms.locfileid: "39397206"
---
# <a name="monitor-and-troubleshoot-a-cloud-storage-application"></a>對雲端儲存體應用程式進行疑難排解及監視

本教學課程是系列課程的第四個部分，也是最後一個部分。 您可了解如何對雲端儲存體應用程式進行疑難排解及監視。

在本系列的第一部分中，您了解如何：

> [!div class="checklist"]
> * 開啟記錄與計量
> * 啟用授權錯誤的警示
> * 使用不正確的 SAS 權杖來執行測試流量
> * 下載及分析記錄檔

[Azure 儲存體分析](../common/storage-analytics.md)可提供儲存體帳戶的記錄和計量資料。 這些資料可讓您深入了解儲存體帳戶的健康狀態。 若要從 Azure 儲存體分析收集資料，您可以設定記錄、計量和警示。 此程序包含開啟記錄、設定計量，以及啟用警示。

您可透過 Azure 入口網站中的 [診斷] 索引標籤，啟用儲存體帳戶的記錄和計量。 儲存體記錄可讓您記錄儲存體帳戶中的成功和失敗要求詳細資料。 這些記錄檔可讓您查看 Azure 資料表、佇列和 Blob 的讀取、寫入和刪除作業詳細資料， 以及要求失敗的原因，例如逾時、節流和授權錯誤。

## <a name="log-in-to-the-azure-portal"></a>登入 Azure 入口網站

登入 [Azure 入口網站](https://portal.azure.com)

## <a name="turn-on-logging-and-metrics"></a>開啟記錄與計量

從左側功能表中，依序選取 [資源群組] 和 [myResourceGroup]，然後選取資源清單中的儲存體帳戶。

將 [診斷設定 (傳統)] 下方的 [狀態] 設定為 [開啟]。 請確定 **Blob 屬性** 下方的所有選項皆已啟用。

完成時，按一下 [儲存]。

![[診斷] 窗格](media/storage-monitor-troubleshoot-storage-application/enable-diagnostics.png)

## <a name="enable-alerts"></a>啟用警示

警示可依據違反臨界值的計量，傳送電子郵件給系統管理員或觸發 Webhook。 在此範例中，您會啟用 `SASClientOtherError` 計量的警示。

### <a name="navigate-to-the-storage-account-in-the-azure-portal"></a>在 Azure 入口網站中巡覽至儲存體帳戶

在 [監視] 區段下方，選取 [警示 (傳統)]。

選取 [新增計量警示 (傳統)]，並填入必要的資訊以完成 [新增規則] 表單。 從 [計量] 下拉式清單中，選取 `SASClientOtherError`。 若要讓警示在第一次出現錯誤時觸發，請從 [條件] 下拉式清單中選取 [大於或等於]。

![[診斷] 窗格](media/storage-monitor-troubleshoot-storage-application/add-alert-rule.png)

## <a name="simulate-an-error"></a>模擬錯誤

若要模擬有效警示，您可以嘗試要求儲存體帳戶中不存在的 Blob。 下列命令需要儲存體容器名稱。 您可以使用現有容器的名稱，或建立新名稱供此範例使用。

請預留位置取代為實際值 (請確定 `<INCORRECT_BLOB_NAME>` 設定為不存在的值)，然後執行命令。

```azurecli-interactive
sasToken=$(az storage blob generate-sas \
    --account-name <STORAGE_ACCOUNT_NAME> \
    --account-key <STORAGE_ACCOUNT_KEY> \
    --container-name <CONTAINER_NAME> \
    --name <INCORRECT_BLOB_NAME> \
    --permissions r \
    --expiry `date --date="next day" +%Y-%m-%d`)

curl https://<STORAGE_ACCOUNT_NAME>.blob.core.windows.net/<CONTAINER_NAME>/<INCORRECT_BLOB_NAME>?$sasToken
```

下圖的範例警示即是以上述範例執行的失敗模擬而得。

 ![範例警示](media/storage-monitor-troubleshoot-storage-application/email-alert.png)

## <a name="download-and-view-logs"></a>下載及檢視記錄檔

儲存體記錄檔會將資料儲存在您儲存體帳戶 Blob 容器內的一組 Blob 中，該容器名為 **$logs**。 當您列出帳戶中的所有 Blob 容器時，不會顯示這個容器；但如果您直接存取，即可查看其中的內容。

在此案例中，您會使用 [Microsoft Message Analyzer](http://technet.microsoft.com/library/jj649776.aspx) 來與 Azure 儲存體帳戶進行互動。

### <a name="download-microsoft-message-analyzer"></a>下載 Microsoft Message Analyzer

下載 [Microsoft Message Analyzer](https://www.microsoft.com/download/details.aspx?id=44226) 並安裝應用程式。

啟動應用程式，然後選擇 [檔案] > [開啟] > [From Other File Sources] \(從其他檔案來源)。

在 [檔案選取器] 對話方塊中，選取 [+ Add Azure Connection] \(+ 新增 Azure 連線)。 輸入**儲存體帳戶的名稱**和**帳戶金鑰**，然後按一下 [確定]。

![Microsoft Message Analyzer - [新增 Azure 儲存體連線] 對話方塊](media/storage-monitor-troubleshoot-storage-application/figure3.png)

一旦連線後，即可展開儲存體樹狀檢視中的容器來檢視記錄檔 Blob。 選取最新的記錄檔，然後按一下 [確定]。

![Microsoft Message Analyzer - [新增 Azure 儲存體連線] 對話方塊](media/storage-monitor-troubleshoot-storage-application/figure4.png)

在 [新增工作階段] 對話方塊中，按一下 [啟動] 以檢視您的記錄檔。

記錄檔開啟之後，您即可檢視儲存體事件。 在下圖中，您可以看到儲存體帳戶觸發了一項 `SASClientOtherError`。 如需儲存體記錄的其他資訊，請瀏覽[儲存體分析](../common/storage-analytics.md)。

![Microsoft Message Analyzer - 檢視事件](media/storage-monitor-troubleshoot-storage-application/figure5.png)

[儲存體總管](https://azure.microsoft.com/features/storage-explorer/)是另一種可用來與儲存體帳戶互動的工具，包括 **$logs** 容器與其中包含的記錄檔。

## <a name="next-steps"></a>後續步驟

在本系列課程的第四部分 (亦即最後一個部分) 中，您學會如何針對儲存體帳戶進行疑難排解及監視，例如如何：

> [!div class="checklist"]
> * 開啟記錄與計量
> * 啟用授權錯誤的警示
> * 使用不正確的 SAS 權杖來執行測試流量
> * 下載及分析記錄檔

遵循以下連結以查看預先建立的儲存體範例。

> [!div class="nextstepaction"]
> [Azure 儲存體指令碼範例](storage-samples-blobs-cli.md)