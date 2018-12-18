---
title: 使用自訂視覺服務建置分類器 - Azure 認知服務 | Microsoft Docs
description: 了解如何使用自訂視覺服務來建立分類器，以分辨相片中的物件。
services: cognitive-services
author: anrothMSFT
manager: corncar
ms.service: cognitive-services
ms.component: custom-vision
ms.topic: article
ms.date: 05/02/2018
ms.author: anroth
ms.openlocfilehash: c5183078d2f9d5eb16abef4f5df240f77eea6b8b
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39223364"
---
# <a name="how-to-build-a-classifier-with-custom-vision"></a>如何使用自訂視覺建置分類器

若要使用自訂視覺服務，您必須先建置分類器。 在本文件中，了解如何透過網頁瀏覽器建置分類器。

## <a name="prerequisites"></a>必要條件

若要建置分類器，您必須先具備：

- 有效的 [Microsoft 帳戶](https://account.microsoft.com/account)或 Azure Active Directory OrgID (「公司或學校帳戶」)，讓您可以登入 customvision.ai 並開始使用。

    > [!IMPORTANT] 
    > 目前不支援 Azure Active Directory (Azure AD) 使用者從[國家雲端](https://www.microsoft.com/en-us/trustcenter/cloudservices/nationalcloud) \(英文\) 進行 OrgID 登入。

- 可用來為分類器定型的一連串影像 (每個標記至少 30 個影像)。

- 將分類器定型之後，用來測試分類器的一些影像。

- 選用：與您的 Microsoft 帳戶或 OrgID 相關聯的 Azure 訂用帳戶。 如果您沒有 Azure 訂用帳戶，可在開始之前建立一個[免費帳戶](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

    > [!IMPORTANT]
    > 如果沒有 Azure 訂用帳戶，您將只能建立__有限的試用__專案。 如果您有 Azure 訂用帳戶，系統將在專案建立期間提示您於 [Azure 入口網站](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision)建立自訂視覺服務定型和預測資源。   

## <a name="create-a-new-project"></a>建立新專案

若要建立新專案，請使用下列步驟：

1. 在網頁瀏覽器中，瀏覽到[自訂視覺網頁](https://customvision.ai) \(英文\)。 選取 [登入] 以開始使用服務。

    ![[登入] 頁面的影像](./media/getting-started-build-a-classifier/custom-vision-web-ui.png)

    > [!NOTE]
    > 登入自訂視覺服務之後，您會看見專案清單。 除了用於測試的兩個「有限的試用」專案，專案都會與 Azure 資源相關聯。 如果您是 Azure 使用者，將會看到可存取 [Azure 資源](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#grant-access-to-resources)的相關所有專案。 

2. 若要建立第一個專案，請選取 [新增專案]。 針對您的第一個專案，系統會要求您同意「服務條款」。 選取核取方塊，然後選取 [我同意] 按鈕。 [新增專案] 對話方塊隨即出現。

    ![[新增專案] 對話方塊提供名稱、描述和領域的欄位。](./media/getting-started-build-a-classifier/new-project.png)

3. 輸入專案的名稱和描述。 然後選取其中一個可用領域。 每個領域都會針對特定類型的影像來將分類器最佳化，如下表所述：

    |網域|目的|
    |---|---|
    |__泛型__| 已針對廣泛的影像分類工作進行最佳化。 如果沒有其他適用的領域，或您不確定要選擇哪一個領域，請選取「泛型」領域。 |
    |__食物__|已針對菜餚相片進行最佳化，如同您在餐廳菜單上看見的一樣。 如果您想要將個別水果或蔬菜的相片分類，請使用「食物」領域。|
    |__地標__|已針對可辨識的地標 (包括自然和人工) 進行最佳化。 地標在相片中清楚顯示時，此領域的效果最佳。 即使地標前面的人稍微阻擋到該地標，此領域還是能夠運作。|
    |__零售__|已針對在購物目錄或購物網站上找到的影像進行最佳化。 如果您想在連衣裙、褲子和襯衫之間進行高精確度的分類，請使用此領域。|
    |__成人__|已進行最佳化，能以更好的方式定義成人內容與非成人內容。 例如，如果您想要封鎖身穿泳衣的人的影像，這個領域允許您建置自訂分類器來達到此目的。|
    |__精簡領域__| 已針對行動裝置上的即時分類條件約束進行最佳化。 精簡領域所產生的模型可以匯出到本機執行。|

    您稍後可以視需要變更此領域。

4. 選取資源群組。 [資源群組] 下拉式清單會顯示您的所有 Azure 資源群組，其中包括自訂視覺服務資源。 您也可以建立特定__有限的試用__。 有限的試用項目是非 Azure 使用者將能夠從中選擇的唯一資源群組。

    若要建立專案，請選取 [建立專案]。

## <a name="upload-and-tag-images"></a>上傳和標記影像

1. 若要將影像新增至分類器，請使用 [新增影像] 按鈕，然後選取 [瀏覽本機檔案]。 選取 [開啟] 以移至標記。

    > [!TIP]
    > 選取影像之後，您必須標記它們。 標記會套用到您選取來上傳的影像群組，因此，透過您打算使用的標記來上傳影像可能會更輕鬆。 在標記並上傳所選取的影像之後，您也可以變更它們的標記。

    > [!TIP]
    > 上傳具有不同拍攝角度、光源、背景、類型、樣式、群組、大小等的影像。使用各種相片類型，以確保您的分類器不會產生偏差，而且也能一般化。

    自訂視覺服務接受 .jpg、.png 和 .bmp 格式的定型影像，每個影像最大 6 MB (預測影像在每個影像中最大可以是 4 MB)。我們建議位於最短邊的影像為 256 像素。 自訂視覺服務會相應增加最短邊上任何小於 256 像素的影像。

    ![[新增影像] 控制項會顯示於左上方，並在正下方顯示為按鈕。](./media/getting-started-build-a-classifier/add-images01.png)

    >[!NOTE] 
    > REST API 可以用來從 URL 載入定型影像。

2. 若要設定標記，請在 [我的標記] 欄位中輸入文字，然後使用 [+] 按鈕。 若要上傳影像並標記它們，請使用 [上傳 [number] 個檔案] 按鈕。 您可以在影像中新增多個標記。 

    > [!NOTE]
    > 上傳時間會因您所選取影像的數目和大小而不同。

    ![標記和上傳頁面的影像](./media/getting-started-build-a-classifier/add-images03.png)

3. 一旦上傳影像之後，請選取 [完成]。

    ![進度列會顯示所有工作都已完成。](./media/getting-started-build-a-classifier/add-images04.png)

4. 若要上傳另一組影像，請返回步驟 1。 例如，如果您想要分辨狗與小馬，請上傳並標記小馬的影像。

## <a name="train-and-evaluate-the-classifier"></a>為分類器定型及評估

若要為分類器定型，請選取 [定型] 按鈕。

![[定型] 按鈕位於接近瀏覽器視窗的右上方處。](./media/getting-started-build-a-classifier/train01.png)

只需幾分鐘就能為分類器定型。 在此期間，會顯示定型程序的相關資訊。

![[定型] 按鈕位於接近瀏覽器視窗的右上方處。](./media/getting-started-build-a-classifier/train02.png)

定型之後，即會顯示 [效能]。 精確度和記憶度指標會根據自動測試告訴您分類器的優異程度。 自訂視覺服務會利用名為 [K 次交叉驗證](https://en.wikipedia.org/wiki/Cross-validation_(statistics)) \(英文\) 的程序，使用您送出來定型的影像計算這些數字。

![定型結果會顯示整體的精確度與記憶度，以及分類器中每個標記的精確度與記憶度。](./media/getting-started-build-a-classifier/train03.png)

> [!NOTE] 
> 每次您選取 [定型] 按鈕時，即會為分類器建立新的反覆項目。 您可以在 [效能] 索引標籤上檢視所有舊的反覆項目，而且可以刪除任何可能已過時的反覆項目。 當您刪除反覆項目時，最終會刪除唯一與它相關聯的任何影像。

分類器會使用所有影像來建立可識別每個標記的模型。 為了測試模型品質，分類器會嘗試模型上的每個影像，以查看模型所找到的內容。

隨即顯示分類結果的品質。

|詞彙|定義|
|---|---|
|__精確度__|當您將影像分類時，分類器正確分類影像的可能性有多大？ 在用來為分類器 (狗與小馬) 定型的所有影像中，此模型的正確百分比有多少？ 100 個影像中有 99 個正確標記表示準確度為 99%。|
|__記憶度__|在應已正確分類的所有影像中，您的分類器正確識別出多少個影像？ 100% 的記憶度表示，如果用來為分類器定型的影像中有 38 張狗的影像，分類器找到了 38 隻狗。|

## <a name="next-steps"></a>後續步驟

[測試和模型重新定型](test-your-model.md)

