---
title: 在 Azure 資料目錄中儲存搜尋和釘選資料資產
description: 強調 Azure 資料目錄中用於儲存資料來源和資料資產以供稍後使用之功能的操作說明文章。
services: data-catalog
author: steelanddata
ms.author: maroche
ms.assetid: 6bd00a81-820d-4b7c-91fa-ab09e575474c
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 786d65eaf667ae8ae9dc2c91d3113f5057a98a27
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052959"
---
# <a name="save-searches-and-pin-data-assets-in-azure-data-catalog"></a>在 Azure 資料目錄中儲存搜尋和釘選資料資產
## <a name="introduction"></a>簡介
Azure 資料目錄提供用來探索資料來源的功能。 您可以快速搜尋和篩選目錄來找出資料來源並了解其預定目的，以更輕鬆地尋找手邊工作的正確資料。

但如果您需要定期使用相同的資料呢？ 如果您和其他使用者定期貢獻你們的知識到目錄中的相同資料來源呢？ 在這些情況下，必須重複發出相同的搜尋非常沒有效率。 這是已儲存的搜尋和已釘選的資料資產可發揮作用的地方。

## <a name="saved-searches"></a>已儲存的搜尋
資料目錄中的已儲存搜尋是可重複使用的每個使用者搜尋定義。 您可以定義搜尋，包括搜尋字詞、標籤及其他篩選條件，然後進行儲存。 已儲存的搜尋定義可以在稍後重新執行，以傳回任何符合其搜尋準則的資料資產。

### <a name="create-a-saved-search"></a>建立已儲存的搜尋
若要建立已儲存的搜尋，請執行下列作業：
1. 在 Azure 資料目錄入口網站的 [目前搜尋] 視窗中，按一下 [儲存]。 

    ![目前搜尋設定的儲存連結](./media/data-catalog-how-to-save-pin/01-save-option.png) 

2. 輸入您想要重複使用的搜尋準則，然後按一下 [儲存]。

    ![目前搜尋設定的已儲存搜尋名稱](./media/data-catalog-how-to-save-pin/02-name.png)

3. 當系統提示您時，請輸入已儲存搜尋的名稱。 請為搜尋將傳回的資料資產選擇有意義且能描述其用途的名稱。

### <a name="manage-saved-searches"></a>管理已儲存的搜尋
在您儲存一或多個搜尋後，[已儲存的搜尋] 選項便會顯示在 [目前搜尋] 方塊下方。 展開清單時，隨即會顯示所有已儲存的搜尋。

 ![已儲存的搜尋的清單](./media/data-catalog-how-to-save-pin/03-list.png)

執行下列任一項：

* 若要執行搜尋，請在清單中選取已儲存的搜尋。

* 若要檢視已儲存搜尋的管理選項清單，請按一下搜尋名稱旁邊的向下箭號。

    ![用於管理已儲存的搜尋的選項](./media/data-catalog-how-to-save-pin/04-managing.png)

* 若要為已儲存的搜尋輸入新名稱，請選取 [重新命名]。 但搜尋定義不會變更。

* 若要從清單中移除已儲存的搜尋，請選取 [刪除]，然後確認刪除。

* 若要將已儲存的搜尋標示為您的預設搜尋，請選取 [儲存為預設值]。 如果您在 Azure 資料目錄首頁執行「空白」搜尋，就會執行您的預設搜尋。 此外，標示為預設搜尋的搜尋會顯示在 [已儲存的搜尋] 清單頂端。

### <a name="organizational-saved-searches"></a>組織已儲存的搜尋
您組織中的所有使用者都可以儲存搜尋供自己使用。 資料目錄管理員也可以儲存搜尋，以供組織內的所有使用者使用。 當系統管理員儲存搜尋時，他們會看到 [在公司內部共用] 選項。 選取此選項可針對組織中的所有使用者共用已儲存的搜尋。

 ![組織已儲存的搜尋](./media/data-catalog-how-to-save-pin/08-organizational-saved-search.png)

## <a name="pinned-data-assets"></a>已釘選的資料資產
利用已儲存的搜尋，您可以儲存並重複使用搜尋定義。 隨著目錄內容變更，搜尋所傳回的資料資產可能會隨著時間而變化。 釘選資料資產時，您可以明確地識別特定資料資產，以便更輕鬆地存取資料資產而不必使用搜尋。

釘選資料資產相當簡單。 若要將資料資產新增至釘選清單，只需按一下**釘選**圖示。 此圖示會顯示在磚檢視之資產磚的角落，以及 Azure 資料目錄入口網站的清單檢視最左邊的資料行中。

![資料資產釘選圖示](./media/data-catalog-how-to-save-pin/05-pinning.png)

取消訂選資料資產相當簡單。 只要按一下**取消釘選**圖示來切換所選取資產的設定即可。

![資料資產取消釘選圖示](./media/data-catalog-how-to-save-pin/06-unpinning.png)

## <a name="the-my-assets-section"></a>[我的資產] 區段
資料目錄入口網站首頁包含 [我的資產] 區段，其中顯示目前的使用者感興趣的資產。 此區段同時包含已釘選的資產和已儲存的搜尋。

![首頁上的 [我的資產] 區段](./media/data-catalog-how-to-save-pin/07-my-assets.png)

## <a name="summary"></a>總結
Azure 資料目錄提供各種功能，以便能更容易地探索所需的資料來源，因此您和其他組織成員可以花較少的時間尋找資料，並且有更多時間使用資料。 儲存的搜尋 和釘選的資料資產是以這些核心功能為基礎所建立，因此使用者可以輕鬆地識別所要重複使用的資料來源。
