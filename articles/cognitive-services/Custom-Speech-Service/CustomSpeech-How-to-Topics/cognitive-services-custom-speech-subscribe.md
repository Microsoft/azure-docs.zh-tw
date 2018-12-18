---
title: 取得 Azure 自訂語音服務的訂用帳戶金鑰 | Microsoft Docs
description: 了解如何取得訂用帳戶金鑰以呼叫認知服務中的自訂語音服務。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: article
ms.date: 02/08/2017
ms.author: panosper
ms.openlocfilehash: 84ef657af2cc3dc4a7168a815b5e51d6f4f33fd7
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49338364"
---
# <a name="obtain-subscription-keys"></a>取得訂用帳戶金鑰

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

若要開始使用 Azure 自訂語音服務，您必須先將使用者帳戶連結至 Azure 訂用帳戶。 您可以取得免費和付費層的訂用帳戶。 如需定價層的相關資訊，請參閱[定價頁面](https://www.microsoft.com/cognitive-services/en-us/pricing)。

## <a name="get-a-subscription-key"></a>取得訂用帳戶金鑰
1. 您可以透過兩種方式之一，從 Azure 入口網站取得訂用帳戶金鑰：

    * 移至 [Azure 入口網站](https://ms.portal.azure.com)，然後藉由搜尋_認知服務_再選取 [認知服務 API] 來新增認知服務 API。

      ![認知服務搜尋](../../../media/cognitive-services/custom-speech-service/custom-speech-azure-subscription.png)

    * 您也可以直接移至[認知服務 API](https://ms.portal.azure.com/#create/Microsoft.CognitiveServices)。

        ![認知服務 API](../../../media/cognitive-services/custom-speech-service/custom-speech-azure-subscription2.png)

    
1. 填寫下列必要欄位︰

      a. **帳戶名稱**。 使用您覺得合適的名稱。 記住此名稱，以便能在資源清單中找到認知服務訂用帳戶。

      b. **訂用帳戶**。 選取其中一個 Azure 訂用帳戶。

      c. **API 類型**。 選取 [自訂語音服務 (預覽)]。

      d. **位置**。 目前為 [美國西部]。

      e. **定價層**。 選取您覺得合適的定價層。 **F0** 是免費層。 [定價頁面](https://www.microsoft.com/cognitive-services/en-us/pricing)會說明允許的配額。

      ![認知服務帳戶建立](../../../media/cognitive-services/custom-speech-service/custom-speech-azure-cris-blade.png)

1. 您應該尋找儀表板上的檢視，或是資源清單中具有所提供帳戶名稱的服務。 當您選取該項目時，您就會看到服務的概觀。 在左邊的清單中，於 [資源管理] 下選取 [金鑰]。 複製 [金鑰 1]。

      後續步驟需要有此訂用帳戶金鑰。

      ![金鑰 1](../../../media/cognitive-services/custom-speech-service/custom-speech-azure-cris-keys2.png)

      > [!NOTE]
      > 請勿從 [概觀] 頁面複製 [訂用帳戶識別碼]。 下一個步驟需要此訂用帳戶金鑰。
      >

      ![概觀訂用帳戶識別碼](../../../media/cognitive-services/custom-speech-service/custom-speech-azure-cris-keys.png)

1. 若要輸入訂用帳戶金鑰，請在右上方的功能區選取使用者帳戶。 在下拉式功能表中，選取 [訂用帳戶]。

      ![[訂用帳戶] 功能表項目](../../../media/cognitive-services/custom-speech-service/custom-speech-subscription-selection.png)

    隨即會出現訂用帳戶資料表，首次開啟時會是空的。

    ![訂用帳戶資料表](../../../media/cognitive-services/custom-speech-service/custom-speech-subscription-list.png)

1. 選取 [新增]。 輸入訂用帳戶的名稱和訂用帳戶金鑰。 此金鑰可以是訂用帳戶中的**金鑰 1** (主要金鑰) 或**金鑰 2** (次要金鑰)。

      ![訂用帳戶金鑰名稱](../../../media/cognitive-services/custom-speech-service/custom-speech-enter-subsciption.png)

若要上傳資料、訓練模型或進行部署，您需要將自訂語音服務活動連結至 Azure 訂用帳戶。 它可以是免費層或付費層的訂用帳戶。 如需詳細資訊，請參閱[價格頁面](https://www.microsoft.com/cognitive-services/en-us/pricing)。

## <a name="get-a-subscription-id"></a>取得訂用帳戶識別碼
若要取得訂用帳戶識別碼，請移至 Azure 入口網站。 搜尋*認知服務*和*自訂語音服務*，然後遵循指示。

> [!NOTE]
> 此程序稍候需要此訂用帳戶金鑰。
>

## <a name="next-steps"></a>後續步驟
* 開始建立[自訂原音模型](cognitive-services-custom-speech-create-acoustic-model.md)。
* 開始建立[自訂語言模型](cognitive-services-custom-speech-create-language-model.md)。
