---
title: 在 Azure 型遠端監視解決方案中偵測裝置問題的教學課程 | Microsoft Docs
description: 本教學課程會示範如何使用規則和動作，來自動偵測遠端監視解決方案中的閾值型裝置問題。
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 07/19/2018
ms.topic: tutorial
ms.custom: mvc
ms.openlocfilehash: 6759568a678394f7cec4ac9f0bdd99d8ed1db9de
ms.sourcegitcommit: f1e6e61807634bce56a64c00447bf819438db1b8
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42886785"
---
# <a name="tutorial-detect-issues-with-devices-connected-to-your-monitoring-solution"></a>教學課程：偵測與監視解決方案連線的裝置問題

在本教學課程中，您可設定遠端監視解決方案加速器，以偵測已連線 IoT 裝置的問題。 若要偵測您的裝置問題，請新增可在解決方案儀表板上產生警示的規則。

為了簡介規則和警示，本教學課程使用模擬的冷卻器裝置。 冷卻器裝置是由名為 Contoso 的組織所管理，且已連線到遠端監視解決方案加速器。 Contoso 已經有一個會在冷卻器壓力超出 298 PSI 時產生重大警示的規則。 身為 Contoso 的操作員，您會想要尋找初始壓力尖峰，以識別可能有感應器發生故障的冷卻器裝置。 若要識別這類裝置，您可新增一個可在冷卻器壓力超出 150 PSI 時產生警告警示的規則。

系統也會要求您在冷卻器裝置過去 5 分鐘內的平均濕度大於 80% 且裝置的溫度大於華氏 75 度時，針對此裝置建立重大警示。

在本教學課程中，您：

>[!div class="checklist"]
> * 檢視您解決方案中的規則
> * 建立規則
> * 建立具有多個條件的規則
> * 編輯現有的規則
> * 開啟和關閉規則

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) 。

[!INCLUDE [iot-accelerators-tutorial-prereqs](../../includes/iot-accelerators-tutorial-prereqs.md)]

## <a name="review-the-existing-rules"></a>檢閱現有的規則

解決方案加速器中的 [規則] 頁面會顯示所有目前規則的清單：

[![規則頁面](./media/iot-accelerators-remote-monitoring-automate/rulesactions_v2-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactions_v2-expanded.png#lightbox)

若只要檢視適用於冷卻器裝置的規則，請套用篩選條件。 當您從清單中選取某個規則時，即可檢視該規則的詳細資訊並進行編輯：

[![檢視規則詳細資料](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdetail_v2-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdetail_v2-expanded.png#lightbox)

## <a name="create-a-rule"></a>建立規則

若要建立一個可在冷卻器裝置壓力超出 150 PSI 時產生警告的規則，請按一下 [新增規則]。 使用下列值來建立規則：

| 設定          | 值                                 |
| ---------------- | ------------------------------------- |
| 規則名稱        | Chiller 警告                       |
| 說明      | Chiller 壓力超出 150 PSI |
| 裝置群組     | **Chillers** 裝置群組             |
| 計算      | 立即                               |
| 條件 1 欄位| pressure                              |
| 條件 1 運算子 | 大於                      |
| 條件 1 值    | 150                               |
| 嚴重性層級  | 警告                               |

[![建立警告規則](./media/iot-accelerators-remote-monitoring-automate/rulesactionsnewrule_v2-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsnewrule_v2-expanded.png#lightbox)

若要儲存新規則，請按一下 [套用]。

您可以在 [規則] 頁面或 [儀表板] 頁面上，查看規則的觸發時間：

[![觸發的警告規則](./media/iot-accelerators-remote-monitoring-automate/warningruletriggered-inline.png)](./media/iot-accelerators-remote-monitoring-automate/warningruletriggered-expanded.png#lightbox)

## <a name="create-an-advanced-rule"></a>建立進階規則

若要建立具有多個條件的規則，以便在冷卻器裝置過去 5 分鐘內的平均濕度大於 80% 且平均溫度大於華氏 75 度時產生重大警示，請按一下 [新增規則]。 使用下列值來建立規則：

| 設定          | 值                                 |
| ---------------- | ------------------------------------- |
| 規則名稱        | 冷卻器的溫度和濕度非常重要    |
| 說明      | 溼度和溫度非常重要 |
| 裝置群組     | **Chillers** 裝置群組             |
| 計算      | 平均值                               |
| 時間週期      | 5                                     |
| 條件 1 欄位| 溼度                              |
| 條件 1 運算子 | 大於                      |
| 條件 1 值    | 80                                |
| 嚴重性層級  | 重要                              |

[![建立多個條件規則第一部分](./media/iot-accelerators-remote-monitoring-automate/rulesactionsnewrule_mult_v2-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsnewrule_mult_v2-expanded.png#lightbox)

若要新增第二個條件，按一下 [+ 新增條件]。 使用新條件的下列值：

| 設定          | 值                                 |
| ---------------- | ------------------------------------- |
| 條件 2 欄位| 溫度                           |
| 條件 2 運算子 | 大於                      |
| 條件 2 值    | 75                                |

[![建立多個條件規則第二部分](./media/iot-accelerators-remote-monitoring-automate/rulesactionsnewrule_mult_cond2_v2-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsnewrule_mult_cond2_v2-expanded.png#lightbox)

若要儲存新規則，請按一下 [套用]。

您可以在 [規則] 頁面或 [儀表板] 頁面上，查看規則的觸發時間：

[![觸發的多個條件規則](./media/iot-accelerators-remote-monitoring-automate/criticalruletriggered-inline.png)](./media/iot-accelerators-remote-monitoring-automate/criticalruletriggered-expanded.png#lightbox)

## <a name="edit-an-existing-rule"></a>編輯現有的規則

若要對現有的規則進行變更，請從規則清單中選取該規則，然後按一下 [編輯]：

[![編輯規則](./media/iot-accelerators-remote-monitoring-automate/rulesactionsedit_v2-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsedit_v2-expanded.png#lightbox)

## <a name="disable-a-rule"></a>停用規則

若要暫時關閉某個規則，您可以在規則清單中將它停用。 選取要停用的規則，然後選擇 [停用]。 清單中該規則的 [狀態] 會變更以指出規則現在已停用。 您可以使用相同的程序來重新啟用先前停用的規則。

[![停用規則](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdisable-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdisable-expanded.png#lightbox)

如果您從清單中選取多個規則，便可以同時啟用和停用多個規則。

## <a name="delete-a-rule"></a>刪除規則

若要永久刪除某個規則，您可以在規則清單中加以刪除。 選取要刪除的規則，然後選擇 [刪除]。

[![刪除規則](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdelete-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdelete-expanded.png#lightbox)

在您確認要刪除規則之後，您將有權從 [維護] 頁面中刪除任何與規則相關聯的警示。

[![刪除規則](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdeletetidy-inline.png)](./media/iot-accelerators-remote-monitoring-automate/rulesactionsdeletetidy-expanded.png#lightbox)

您一次只能刪除一個規則。

[!INCLUDE [iot-accelerators-tutorial-cleanup](../../includes/iot-accelerators-tutorial-cleanup.md)]

## <a name="next-steps"></a>後續步驟

本教學課程示範了如何使用遠端監視解決方案加速器中的 [規則] 頁面，建立及管理可在解決方案中觸發警示的規則。 若要了解如何使用解決方案加速器來管理及設定已連線的裝置，請繼續進行下一個教學課程。

> [!div class="nextstepaction"]
> [設定和管理與監視解決方案連線的裝置問題](iot-accelerators-remote-monitoring-manage.md)