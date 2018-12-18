---
title: 使用 Azure 自動化檢視檔案內容變更
description: 使用 [變更追蹤] 的檔案內容變更功能檢視已變更的檔案。
services: automation
ms.service: automation
ms.component: change-inventory-management
author: georgewallace
ms.author: gwallace
ms.date: 07/03/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 0582505d66bbef3064359fa4047676c4ba60b4e9
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37870174"
---
# <a name="view-contents-of-a-file-that-is-being-tracked-with-change-tracking"></a>檢視正在使用 [變更追蹤] 追蹤的檔案內容

檔案內容追蹤可讓您在使用 [變更追蹤] 追蹤變更前後，檢視檔案的內容。 這樣做可在每次變更發生後，將檔案內容儲存到儲存體帳戶。

## <a name="requirements"></a>需求

* 儲存檔案內容需要有使用 Resource Manager 部署模型的標準儲存體帳戶。 不應該使用進階和傳統的部署模型儲存體帳戶。 如需儲存體帳戶的詳細資訊，請參閱[關於 Azure 儲存體帳戶](../storage/common/storage-create-storage-account.md)

* 使用的儲存體帳戶只能連線 1 個自動化帳戶。

* 您的自動化帳戶已啟用 [變更追蹤][](automation-change-tracking.md)。

## <a name="enable-file-content-tracking"></a>啓用檔案內容追蹤

1. 在 Azure 入口網站中，開啟您的自動化帳戶，然後選取 [變更追蹤]。
2. 在最上層功能表中，選取 [編輯設定]。
3. 選取 [檔案內容]，然後按一下 [連結]。 這會開啟 [新增變更追蹤的內容位置] 窗格。

   ![enable](./media/change-tracking-file-contents/enable.png)

4. 選取儲存檔案內容所用的訂用帳戶和儲存體帳戶。 如果您想要啟用所有現有已追蹤檔案的檔案內容追蹤，請針對 [上傳所有設定的檔案內容] 選取 [開啓]。 以後可以變更各檔案路徑的這個選項。

   ![設定儲存體帳戶](./media/change-tracking-file-contents/storage-account.png)

5. 一旦啟用，就會顯示儲存體帳戶和 SAS Uri。 SAS URI 會在 365 天之後過期，按一下 [重新產生] 按鈕即可重新建立。

   ![列出帳戶金鑰](./media/change-tracking-file-contents/account-keys.png)

## <a name="add-a-file"></a>新增檔案

下列步驟會引導您完成開啟檔案的 [變更追蹤]：

1. 在 [變更追蹤] 的 [編輯設定] 頁面中，選取 [Windows 檔案] 或 [Linux 檔案] 索引標籤，然後按一下 [新增]

1. 填寫檔案路徑的資訊，然後選取 [上傳所有設定的檔案內容] 下的 [True]。 此設定只會啓用該檔案路徑的檔案內容追蹤。

   ![新增 Linux 檔案](./media/change-tracking-file-contents/add-linux-file.png)

## <a name="viewing-the-contents-of-a-tracked-file"></a>檢視追蹤的檔案內容

1. 一旦偵測到檔案或路徑中檔案的變更，就會顯示在入口網站中。 從變更清單中選取檔案變更。 [變更詳細資料] 窗格隨即顯示。

   ![列出變更](./media/change-tracking-file-contents/change-list.png)

1. 在 [變更詳細資料] 頁面上，您會看到檔案資訊前後的標準，按一下左上方的 [View File Content Changes] \(檢視檔案內容變更\) 可查看檔案的內容。

  ![變更詳細資料](./media/change-tracking-file-contents/change-details.png)

1. 新的頁面會以並排顯示檢視向您顯示檔案內容。 您也可以選取 [內嵌] 查看變更的內嵌檢視。

  ![檢視檔案變更](./media/change-tracking-file-contents/view-file-changes.png)

## <a name="next-steps"></a>後續步驟

瀏覽變更追蹤的教學課程，以深入了解如何使用此解決方案：

> [!div class="nextstepaction"]
> [對您環境中的變更進行疑難排解](automation-tutorial-troubleshoot-changes.md)

* 使用 [Log Analytics 中的記錄檔搜尋](../log-analytics/log-analytics-log-searches.md) ，檢視詳細的變更追蹤資料。
