---
title: Azure 資訊安全中心教學課程 - 回應安全性事件 | Microsoft Docs
description: Azure 資訊安全中心教學課程 - 回應安全性事件
services: security-center
documentationcenter: na
author: terrylan
manager: mbaldwin
editor: ''
ms.assetid: 181e3695-cbb8-4b4e-96e9-c4396754862f
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/22/2018
ms.author: yurid
ms.openlocfilehash: a69ecc4bdff8e65c3529afab0c11afccfdd894c7
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32774041"
---
# <a name="tutorial-respond-to-security-incidents"></a>教學課程：回應安全性事件
資訊安全中心會使用進階的分析和威脅情報，持續分析混合式雲端工作負載，提醒您發生惡意活動。 此外，您可以將警示從其他的安全性產品和服務整合到資訊安全中心，並以您自己的指標或情報來源作為基礎建立自訂警示。 一旦有警示產生，您便需要迅速採取行動來進行調查並加以修復。 在本教學課程中，您將了解如何：

> [!div class="checklist"]
> * 將安全性警示分級
> * 進一步調查，以判斷安全性事件的根本原因及範圍
> * 搜尋安全性資料以協助調查

如果您沒有 Azure 訂用帳戶，請在開始前建立[免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="prerequisites"></a>先決條件
若要逐步執行本教學課程中涵蓋的功能，您必須是在資訊安全中心的標準定價層。 前 60 天可以免費試用資訊安全中心標準定價層。 [將 Azure 訂用帳戶上架到資訊安全中心標準定價層](security-center-get-started.md)快速入門會為您逐步解說如何升級至「標準」定價層。

## <a name="triage-security-alerts"></a>將安全性警示分級
資訊安全中心可讓您統一檢視所有安全性警示。 安全性警示會依嚴重性來設定順位，並視情況將相關警示合併到一個安全性事件。 在為警示和事件分級時，您應該：

- 將無需任何其他動作的警示關閉，例如，若警示為誤判
- 採取行動來修復已知的攻擊，例如封鎖惡意 IP 位址的網路流量
- 判斷需要進一步調查的警示


1. 在 [資訊安全中心] 主功能表上，選取 [偵測] 下的 [安全性警訊]：

  ![安全性警示](./media/tutorial-security-incident/tutorial-security-incident-fig1.png)  

2. 在警示清單中對某個安全性事件 (此為警示集合) 按一下，以深入了解此事件。 [偵測到安全性事件] 隨即開啟。

  ![安全性事件](./media/tutorial-security-incident/tutorial-security-incident-fig2.png)

3. 在這個畫面上，您會在上方看到安全性事件描述，以及屬於此事件的警示清單。 按一下您要進一步調查的警示，以取得詳細資訊。

  ![安全性事件](./media/tutorial-security-incident/tutorial-security-incident-fig3.png)

  警示類型可能會有所不同，如需警示類型的詳細資訊及可能的補救步驟，請閱讀[了解 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)。 對於可安全關閉的警示，您可以滑鼠右鍵按一下警示，並選取 [關閉] 選項：

  ![警示](./media/tutorial-security-incident/tutorial-security-incident-fig4.png)

4. 如果惡意活動的根本原因和範圍不明時，請繼續進行下一個步驟以進一步調查。

## <a name="investigate-an-alert-or-incident"></a>調查警示或事件
1. 在 [安全性警示] 頁面上，按一下 [開始調查] 按鈕 (如果您已啟動，名稱會變更為 [繼續調查])。

  ![調查](./media/tutorial-security-incident/tutorial-security-incident-fig5.png)

  調查地圖會以圖形來表示連線到此安全性警示或事件的實體。 按一下地圖中的實體，該實體的相關資訊就會顯示新的實體，並展開地圖。 地圖中選取的實體會在頁面的右側窗格中將其屬性反白顯示。 每個索引標籤上的可用資訊會根據選取的實體而有所不同。 在調查流程中，檢閱所有相關資訊以深入了解攻擊者的移動路線。

2. 如果您需要更多證據，或必須對調查期間找到的實體進一步調查時，請繼續進行下一個步驟。

## <a name="search-data-for-investigation"></a>搜尋資料以進行調查

您可以使用資訊安全中心內的搜尋功能，來尋找更多關於遭入侵系統的證據，以及尋找屬於調查一部分之實體的詳細資料。

若要執行搜尋，請開啟 [資訊安全中心] 儀表板，按一下左側瀏覽窗格中的 [搜尋]，選取您要搜尋之實體所在的工作區，輸入搜尋查詢，然後按一下 [搜尋] 按鈕。

## <a name="clean-up-resources"></a>清除資源
此集合中的其他快速入門和教學課程會以本快速入門為基礎。 如果您打算繼續處理後續的快速入門和教學課程，請繼續執行標準層，並保持將自動佈建維持為啟用狀態。 如果您不打算繼續，或是要返回免費層：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 選取您需要返回免費層的訂用帳戶或原則。 [安全性原則] 隨即開啟。
3. 在 [原則元件] 下，選取 [定價層]。
4. 選取 [免費] 以將訂用帳戶從標準層變更為免費層。
5. 選取 [ **儲存**]。

如果您需要停用自動佈建：

1. 返回 [資訊安全中心] 主功能表，並選取 [安全性原則]。
2. 選取您想要停用自動佈建的訂用帳戶。
3. 在 [安全性原則 - 資料收集] 下，選取 [上架] 底下的 [關閉] 以停用自動佈建。
4. 選取 [ **儲存**]。

>[!NOTE]
> 停用自動佈建不會從已佈建代理程式的 Azure VM 移除 Microsoft Monitoring Agent。 停用自動佈建會限制對資源的安全性監視。
>

## <a name="next-steps"></a>後續步驟
在此教學課程中，您已了解在回應安全性事件時要使用的資訊安全中心功能，例如：

> [!div class="checklist"]
> * 安全性事件，此為資源相關警示的彙總
> * 調查地圖，此地圖會以圖形來表示連線到安全性警示或事件的實體
> * 搜尋功能，用以尋找更多關於遭入侵系統的證據

若要深入了解資訊安全中心的調查功能，請參閱：

> [!div class="nextstepaction"]
> [調查事件及警示](security-center-investigation.md)
