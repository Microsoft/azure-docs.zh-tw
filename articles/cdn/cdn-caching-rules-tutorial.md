---
title: 教學課程：設定 Azure CDN 快取規則 | Microsoft Docs
description: 在本教學課程中，您可以設定 Azure CDN 全域快取規則及自訂快取規則。
services: cdn
documentationcenter: ''
author: mdgattuso
manager: danielgi
editor: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/20/2018
ms.author: magattus
ms.custom: mvc
ms.openlocfilehash: 10d06d2e792b476a4c973029241d6cb98c0dd444
ms.sourcegitcommit: 4047b262cf2a1441a7ae82f8ac7a80ec148c40c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/11/2018
ms.locfileid: "49094066"
---
# <a name="tutorial-set-azure-cdn-caching-rules"></a>教學課程：設定 Azure CDN 快取規則

> [!NOTE] 
> Azure CDN 快取規則僅適用於**來自 Verizon 的 Azure CDN 標準**和**來自 Akamai 的 Azure CDN 標準**。 針對**來自 Verizon 的 Azure CDN Premium**，您可以使用 [管理] 入口網站中的 [Azure CDN 規則引擎](cdn-rules-engine.md)來執行類似功能。
 

本教學課程說明如何使用 Azure 內容傳遞網路 (CDN) 快取規則，以全域及自訂條件 (例如 URL 路徑和副檔名) 的方式設定或修改預設快取到期行為。 Azure CDN 提供兩種類型的快取規則：
- 全域快取規則：您可以針對設定檔中的每個端點設定一個全域快取規則，這會影響針對端點的所有要求。 全域快取規則會覆寫任何 HTTP 快取指示詞標頭 (如果已設定)。

- 自訂快取規則：您可以針對設定檔中的每個端點設定一或多個自訂快取規則。 自訂快取規則會比對特定路徑和副檔名、會依序處理，並會覆寫全域快取規則 (如果已設定)。 

在本教學課程中，您了解如何：
> [!div class="checklist"]
> - 開啟快取規則頁面。
> - 建立全域快取規則。
> - 建立自訂快取規則。

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>必要條件

您必須先建立 CDN 設定檔和至少一個 CDN 端點，才能完成本教學課程中的步驟。 如需詳細資訊，請參閱[快速入門：建立 Azure CDN 設定檔和端點](cdn-create-new-endpoint.md)。

## <a name="open-the-azure-cdn-caching-rules-page"></a>開啟 Azure CDN 快取規則頁面

1. 在 [Azure 入口網站](https://portal.azure.com)選取 CDN 設定檔，然後選取端點。

2. 在左窗格的 [設定] 下方，選取 [快取規則]。

   ![CDN [快取規則] 按鈕](./media/cdn-caching-rules/cdn-caching-rules-btn.png)

   [快取規則] 頁面隨即出現。

   ![CDN 快取規則頁面](./media/cdn-caching-rules/cdn-caching-rules-page.png)


## <a name="set-global-caching-rules"></a>設定全域快取規則

建立全域快取規則，如下所示：

1. 在 [全域快取規則] 下方，將 [查詢字串快取行為] 設定為 [忽略查詢字串]。

2. 將 [快取行為] 設定為 [缺少時才設定]。
       
3. 針對 [快取到期期間]，在 [天數] 欄位中輸入 10。

    全域快取規則會影響針對端點的所有要求。 如果原始快取指示詞標頭存在 (`Cache-Control` 或 `Expires`)，此規則就會優先採用它們；否則，系統會在未指定的情況下將快取設定為 10 天。 

    ![全域快取規則](./media/cdn-caching-rules/cdn-global-caching-rules.png)

## <a name="set-custom-caching-rules"></a>設定自訂快取規則

建立自訂快取規則，如下所示：

1. 在 [自訂快取規則] 下方，將 [比對條件] 設定為 [路徑]，並將 [比對值] 設定為 `/images/*.jpg`。
    
2. 將 [快取行為] 設定為 [覆寫]，並在 [天數] 欄位中輸入 30。
       
    此自訂快取規則會在您端點的 `/images` 資料夾中，於任何 `.jpg` 影像檔上設定 30 天的快取持續時間。 它會覆寫由原始伺服器所傳送的任何 `Cache-Control` 或 `Expires` HTTP 標頭。

    ![自訂快取規則](./media/cdn-caching-rules/cdn-custom-caching-rules.png)

    
## <a name="clean-up-resources"></a>清除資源

在前述步驟中，您已經建立快取規則。 如果您不想要再使用這些快取規則，可以執行下列步驟予以移除：
 
1. 選取 CDN 設定檔，再針對要移除的快取規則選取端點。

2. 在左窗格的 [設定] 下方，選取 [快取規則]。

3. 在 [全域快取規則] 下方，將 [快取行為] 設為 [未設定]。
 
4. 在 [自訂快取規則] 下方，針對要刪除的規則選取旁邊的核取方塊。

5. 選取 [刪除] 。

6. 在頁面上方選取 [儲存]。


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> - 開啟快取規則頁面。
> - 建立全域快取規則。
> - 建立自訂快取規則。

閱讀下一篇文章，了解如何配置額外的快取規則設定。

> [!div class="nextstepaction"]
> [使用快取規則來控制 Azure CDN 快取行為](cdn-caching-rules.md)



