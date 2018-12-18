---
title: 註冊文字分析 API (Azure 上的 Microsoft 認知服務) | Microsoft Docs
description: 如何註冊使用文字分析和在限制內運作的指示。
services: cognitive-services
author: HeidiSteen
manager: cgronlun
ms.service: cognitive-services
ms.component: text-analytics
ms.topic: get-started-article
ms.date: 3/07/2018
ms.author: heidist
ms.openlocfilehash: dfa5ba138a2e0db75dfc097ca2430fe9c82e826f
ms.sourcegitcommit: 35ceadc616f09dd3c88377a7f6f4d068e23cceec
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/08/2018
ms.locfileid: "39623180"
---
# <a name="how-to-sign-up-for-text-analytics-api"></a>如何註冊文字分析 API

文字分析資源會在雲端中全天候供您使用。 在上傳內容進行分析之前，您必須先進行註冊以取得存取金鑰。 此 API 的每個呼叫都需要在要求上備有存取金鑰。

+ 從 Azure 訂用帳戶來開始。 您可以建立[免費帳戶](https://azure.microsoft.com/free/)進行實驗，完全不需付費。

+ 建立[認知服務 API 帳戶](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account)，並選擇**文字分析 API**。 金鑰會在註冊時產生。

文字分析有提供免費層供您探索及評估，並提供可計費層供生產工作負載使用。 您可以在每個訂用帳戶中擁有多個註冊：一個免費、一個付費，依此類推。 如果要求的數量增加，您可以切換到提供更多交易的層級。

預覽或免費層的服務沒有服務等級協定。 如需詳細資訊，請參閱[認知服務的 SLA](https://azure.microsoft.com/support/legal/sla/cognitive-services/v1_1/)

## <a name="how-to-change-tiers"></a>如何變更層級

從免費層開始，然後轉換到可計費層以供生產工作負載使用。 各漸進層級會提供標準計費。 您可以切換層級，卻依然保有相同的端點和存取金鑰。

1. 登入 [Azure 入口網站](https://portal.azure.com)並[尋找服務](text-analytics-how-to-access-key.md)。

2. 按一下 [定價層]。

   ![左側導覽功能表中的定價層命令](../media/portal-pricing-tier.png)

3. 選取某個階層，然後按一下 [選取]。  選取項目一經處理，新的限制便會生效。 

   ![層級選取頁面中的圖格和選取按鈕](../media/portal-choose-tier.png)

## <a name="how-billing-works"></a>計費的運作方式

計費的根據是交易數目。 您可以在特定層級以每月計費週期購買一批交易，超過數量後，則會依每個交易來計算小型超額費用。 如果您經常超過最大限制，請考慮改用更高的層級。

如需詳細資訊，請參閱[定價頁面](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/)。

### <a name="what-constitutes-a-transaction-in-the-text-analytics-api"></a>文字分析 API 的交易組成要素是什麼？
文件的任何註解都會計為一筆交易。 批次評分呼叫也會列入該筆交易中需要評分的文件數考量。 例如，如果以單一 API 呼叫傳送 1,000 份文件進行情感分析，則會計為 1,000 筆交易。

## <a name="see-also"></a>另請參閱 

 [文字分析概觀](../overview.md)  
 [常見問題集 (FAQ)](../text-analytics-resource-faq.md)

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [取得存取金鑰](text-analytics-how-to-access-key.md)
