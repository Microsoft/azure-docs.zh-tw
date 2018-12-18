---
title: 教學課程 - 建立及管理 Azure 成本管理的匯出資料 |Microsoft Docs
description: 本文說明如何建立及管理匯出的 Azure 成本管理資料來將其用於外部系統中。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/21/2018
ms.topic: tutorial
ms.service: cost-management
manager: dougeby
ms.custom: ''
ms.openlocfilehash: 0765e79b90eed49742f5eead33063907eb1db1f4
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47030841"
---
# <a name="tutorial-create-and-manage-exported-data"></a>教學課程：建立及管理匯出的資料

若您已閱讀過成本分析教學課程，您便已對手動下載成本管理資料相當熟悉。 但是，您可以建立每日定期工作，每天自動將您的成本管理資料匯出至 Azure 儲存體。 匯出的資料格式為 CSV，其中包含所有由成本管理收集的資訊。 您接著可以在 Azure 儲存體中，搭配外部系統使用匯出的資料，並與您擁有的自訂資料合併。 您也可以在外部系統 (例如儀表板或其他財務系統) 中使用您的匯出資料。

本教學課程中的範例會引導您匯出成本管理資料，並驗證該資料已成功匯出。

在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 建立每日匯出
> * 驗證已收集資料

## <a name="prerequisites"></a>必要條件

所有 [Enterprise 合約 (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) 客戶皆可使用資料匯出。 使用者及群組資料匯出的每個訂用帳戶皆支援下列 Azure 權限：

- 擁有者 - 可以建立、修改或刪除已排程的訂用帳戶匯出。
- 參與者 - 可以建立、修改或刪除他們自己的排程匯出。 可以修改其他人所建立的排程匯出名稱。
- 讀者 - 可以排程擁有其權限的匯出。

針對 Azure 儲存體帳戶：
- 需要寫入權限，才能變更設定的儲存體帳戶，無論匯出的權限為何。
- 您的 Azure 儲存體帳戶必須針對 Blob 或檔案儲存體進行設定。

## <a name="sign-in-to-azure"></a>登入 Azure
在 [https://portal.azure.com](https://portal.azure.com/) 登入 Azure 入口網站。

## <a name="create-a-daily-export"></a>建立每日匯出

[成本管理 + 帳單] &gt; 選取訂用帳戶或訂用帳戶中的資源群組 &gt; [匯出] &gt; [新增]。

鍵入匯出的名稱並指定訂用帳戶、Azure 儲存體帳戶、容器，以及檔案儲存體目錄或 Blob 容器，然後按一下 [建立]。

![新增匯出](./media/tutorial-export-acm-data/new-export01.png)

您新增的匯出會出現在匯出清單中。 根據預設會啟用新的匯出，並且會每天執行。 若您想要停用或刪除已排程的匯出，請按一下清單中的任何項目，然後按一下 [停用] 或 [刪除]。

剛開始可能會需要一到兩個小時，匯出才會開始執行。 但是，資料可能需要最多四個小時，才會出現在匯出檔案中。

## <a name="verify-that-data-is-collected"></a>驗證已收集資料

您可以輕鬆驗證您的成本管理資料已收集完成，並使用 Azure 儲存體總管檢視匯出的 CSV 檔案。

在匯出清單中，按一下儲存體帳戶名稱。 在儲存體帳戶頁面上，按一下 [Open in Explorer] \(在總管中開啟\)。 若您看見確認方塊，請按一下 [是] 以在 Azure 儲存體總管中開啟檔案。

![儲存體帳戶頁面](./media/tutorial-export-acm-data/storage-account-page.png)

在 [儲存體總管] 中，巡覽至您想要開啟的容器，然後選取對應到目前月份的資料夾。 隨即顯示 CSV 檔案清單。 選取其中一個，然後按一下 [開啟]。

![儲存體總管](./media/tutorial-export-acm-data/storage-explorer.png)

檔案隨即便會使用設為開啟 CSV 檔案副檔名的程式或應用程式開啟。 以下是 Excel 中的範例。

![範例匯出資料](./media/tutorial-export-acm-data/example-export-data.png)

## <a name="access-exported-data-from-other-systems"></a>從其他系統存取匯出資料

匯出成本管理資料的其中一個目的是從外部系統存取資料。 您可能會使用儀表板系統或其他財務系統。 這類系統之間的差異相當大，因此顯示範例並不實用。  但是，您可以從 [Azure 儲存體簡介](../storage/common/storage-introduction.md)中應用程式存取您的資料開始。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 建立每日匯出
> * 驗證已收集資料

前進到下一個教學課程，透過識別閒置及使用量過低的資源來最佳化和改善效率。

> [!div class="nextstepaction"]
> [檢閱並針對建議採取動作](tutorial-acm-opt-recommendations.md)
