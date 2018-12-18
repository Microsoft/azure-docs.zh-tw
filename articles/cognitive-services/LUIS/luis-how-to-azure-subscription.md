---
title: LUIS 中的端點訂用帳戶管理
titleSuffix: Azure Cognitive Services
description: 在本文中，您會為 LUIS 帳戶建立計量端點金鑰，以遵循付款方案對您的端點提供不受限的流量。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: c7c6009d9fec14e8633a37e5384dd203f2f5fa60
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040375"
---
# <a name="manage-azure-endpoint-subscription-keys"></a>管理 Azure 端點訂用帳戶金鑰

僅針對測試和原型，使用免費 (F0) 層。 針對生產系統，使用[付費](https://aka.ms/luis-price-tier)層。 

> [!NOTE]
> 請勿在生產環境中對端點查詢使用[撰寫金鑰](luis-concept-keys.md#authoring-key)。

<a name="create-luis-service"></a>
## <a name="create-luis-endpoint-key"></a>建立 LUIS 端點金鑰

1. 登入 **[Microsoft Azure](https://ms.portal.azure.com/)**。 
2. 按一下左上方面板中的綠色 **+** 號並且在市集中搜尋 "LUIS"，然後按一下 [Language Understandin] 並遵循**建立體驗**來建立 LUIS 訂用帳戶。 

    ![Azure 搜尋服務](./media/luis-azure-subscription/azure-search.png) 

3. 使用包括帳戶名稱、定價層等設定來設定訂用帳戶。 

    ![Azure API 選擇](./media/luis-azure-subscription/azure-api-choice.png) 

4. 建立 LUIS 服務後，您就可以檢視在 [資源管理] -> [金鑰] 中產生的存取金鑰。  

    ![Azure 金鑰](./media/luis-azure-subscription/azure-keys.png)

    > [!Note] 
    > 登入您區域的 [LUIS](luis-reference-regions.md) 網站並[指派新的 LUIS 端點金鑰](luis-how-to-manage-keys.md#assign-endpoint-key)。 您需要步驟 3 的 LUIS 訂用帳戶名稱。

## <a name="change-luis-pricing-tier"></a>變更 LUIS 定價層

1.  在 [Azure](https://portal.azure.com) 中，尋找您的 LUIS 訂用帳戶。 按一下 LUIS 訂用帳戶。
    ![尋找 LUIS 訂用帳戶](./media/luis-usage-tiers/find.png)
2.  按一下 [定價層]，才能查看可用的定價層。 
    ![檢視定價層](./media/luis-usage-tiers/subscription.png)
3.  按一下定價層，然後按一下 [選取] 以儲存變更。 
    ![變更 LUIS 付款層](./media/luis-usage-tiers/plans.png)
4.  定價變更完成時，快顯視窗會確認新的定價層。 
    ![驗證 LUIS 付款層](./media/luis-usage-tiers/updated.png)
5. 請記得在 [發佈] 頁面上[指派此端點金鑰](luis-how-to-manage-keys.md#assign-endpoint-key)，然後將它使用於所有端點查詢。 

## <a name="exceed-pricing-tier-usage"></a>超過定價層使用量
每一層都允許端點以特定比率向您的 LUIS 帳戶提出要求。 如果要求的比率高於您的計量帳戶每分鐘或每個月允許的比率，則要求會收到 HTTP 錯誤「429：太多要求」。

每一層都允許每個月累積要求。 如果要求總數高於允許的比率，則要求會收到 HTTP 錯誤「403： 禁止」。  

## <a name="viewing-summary-usage"></a>檢視摘要使用量
您可以在 Azure 中檢視 LUIS 使用量資訊。 [概觀] 頁面會顯示近期摘要資訊，包括呼叫和錯誤。 如果您提出 LUIS 端點要求，然後立即觀看 [概觀] 頁面，允許使用量最多在五分鐘內出現。

![檢視摘要使用量](./media/luis-usage-tiers/overview.png)

## <a name="customizing-usage-charts"></a>自訂使用量圖表
計量會提供資料的更詳細檢視。

![預設度量](./media/luis-usage-tiers/metrics-default.png)

您可以設定時間週期和計量類型的計量圖表。 

![自訂度量](./media/luis-usage-tiers/metrics-custom.png)

## <a name="total-transactions-threshold-alert"></a>交易總數閾值警示
如果您想要知道何時達到特定交易閾值 (例如 10,000 筆交易) 時，可以建立警示。 

![預設警示](./media/luis-usage-tiers/alert-default.png)

為一段時間的**呼叫總數**計量新增計量警示。 新增應收到警示的所有人員電子郵件地址。 為應收到警示的所有系統新增 Webhook。 您也可以在警示觸發時執行邏輯應用程式。 

## <a name="next-steps"></a>後續步驟

了解如何使用[版本](luis-how-to-manage-versions.md)來管理 LUIS 應用程式的變更。