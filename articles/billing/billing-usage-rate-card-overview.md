---
title: Azure 計費 API | Microsoft Docs
description: 了解 Azure 計費使用情況和 RateCard API，其可用來提供 Azure 資源耗用量和趨勢的見解。
services: ''
documentationcenter: ''
author: tonguyen
manager: tonguyen
editor: ''
tags: billing
ms.assetid: 3e817b43-0696-400c-a02e-47b7817f9b77
ms.service: billing
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: billing
ms.date: 5/10/2018
ms.author: mobandyo
ms.openlocfilehash: 650fac6208adf8f904384454b2e66e26e45893f1
ms.sourcegitcommit: ebb460ed4f1331feb56052ea84509c2d5e9bd65c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/24/2018
ms.locfileid: "42918847"
---
# <a name="use-azure-billing-apis-to-programmatically-get-insight-into-your-azure-usage"></a>使用 Azure 計費 API 來以程式設計方式深入了解 Azure 使用情況
使用 Azure 計費 API 將使用情況和資源資料提取到您慣用的資料分析工具。 Azure 資源使用情況和 RateCard API 可協助您準確地預測並管理成本。 這些 API 會實作為資源提供者，並成為 Azure Resource Manager 所公開之 API 系列的一部分。  

> [!div class="nextstepaction"]
> [協助改善 Azure 計費文件](https://go.microsoft.com/fwlink/p/?linkid=2010091)

## <a name="azure-invoice-download-api-preview"></a>Azure 發票下載 API (預覽)
[完成加入](billing-manage-access.md#opt-in)之後，請使用預覽版[發票 API](/rest/api/billing) 來下載發票。 這些功能包括：

* **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或透過 [Azure PowerShell Cmdlet](/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用情況資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。
* **日期篩選** - 使用 `$filter` 參數可依發票期間結束日期的相反時間順序取得所有發票。 

> [!NOTE]
> 此功能目前為第一個預覽版本，可能會進行與舊版不相容的變更。 目前，某些訂用帳戶供應項目 (不支援 EA、CSP、AIO) 和 Azure 德國無法使用此功能。

## <a name="azure-resource-usage-api-preview"></a>Azure 資源使用情況 API (預覽)
使用 Azure [資源使用情況 API](https://msdn.microsoft.com/library/azure/mt219003) 來取得預估的 Azure 耗用量資料。 此 API 包含︰

* **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或透過 [Azure PowerShell Cmdlet](/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用情況資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。
* **每小時或每日彙總** - 呼叫端可以指定要 Azure 使用情況資料的每小時值區或每日值區。 預設值為每日值區。
* **執行個體中繼資料 (包括資源標記)** – 取得執行個體層級詳細資料，例如完整的資源 uri (/subscriptions/{subscription-id}/..)、資源群組資訊與資源標記。 此中繼資料可協助您以決定性及程式設計方式，根據標記為跨領域收費之類的使用案例配置使用量。
* **資源中繼資料** - 資源詳細資料 (例如計量名稱、計量類別、計量子類別、單位和區域) 可讓呼叫端深入了解耗用的內容。 我們也致力於跨 Azure 入口網站、Azure 使用情況 CSV、EA 計費 CSV 和其他向外公開的體驗統一資源中繼資料術語，以讓您跨體驗將資料相互關聯。
* **不同供應項目類型的使用情況** – [CSP](https://docs.microsoft.com/azure/cloud-solution-provider/billing/azure-csp-invoice#retrieve-usage-data-for-a-specific-subscription) 以外的供應項目類型 (例如隨收隨付、MSDN、貨幣承諾、貨幣信用額度和 EA) 皆有提供使用情況資料。

## <a name="azure-resource-ratecard-api-preview"></a>Azure 資源 RateCard API (預覽)
使用 [Azure 資源 RateCard API](https://msdn.microsoft.com/library/azure/mt219005) 來取得可用 Azure 資源的清單，以及每個資源的預估價格資訊。 此 API 包含︰

* **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或透過 [Azure PowerShell Cmdlet](/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取 RateCard 資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增至讀取者、擁有者或參與者角色，以存取特定 Azure 訂用帳戶的使用情況資料。
* **支援隨收隨付、MSDN、貨幣承諾、貨幣信用額度供應項目 (不支援 EA 和 [CSP](https://docs.microsoft.com/azure/cloud-solution-provider/billing/azure-csp-pricelist#get-prices-by-using-the-azure-rate-card))** - 此 API 提供 Azure 供應項目層級費率資訊。  此 API 的呼叫端必須輸入供應項目資訊以取得資源詳細資料和費率。 由於 EA 供應項目已自訂每個註冊的費率，所以我們目前無法提供 EA 費率。 

## <a name="scenarios"></a>案例
以下是一些案例，可產生使用情況和 RateCard API 的組合：

* **Azure 月份花費** - 使用使用情況和 RateCard API 的組合，來取得當月份雲端花費的更佳見解。 您可以分析每小時和每天的使用情況及收費評估值區。
* **設定警示** – 使用使用情況和 RateCard API 來取得預估的雲端耗用量和收費，並設定以資源為基礎或以貨幣為基礎的警示。
* **預測計費** – 取得預估的耗用量及雲端花費，並應用機器學習演算法來預測在計費週期結束時的計費情形。
* **耗用前成本分析** – 使用 RateCard API 來預測當您將工作負載移至 Azure 時，所預期的使用量需要多少費用。 如果您有其他雲端或私用雲端中的現有工作負載，您也可以對應您的使用情況和 Azure 費率，以取得 Azure 花費的較佳評估。 此預估可讓您以供應項目為依據來進行選擇，並比較和對比隨收隨付以外的不同供應項目類型，包含貨幣承諾和貨幣信用額度。 此 API 也可讓您依區域查看成本差異，並可讓您執行假設成本分析，以幫助您做出部署決定。
* **假設分析** -
  
  * 您可以判斷是在另一個區域執行工作負載，還是在 Azure 資源的另一個組態上執行工作負載會比較符合成本效益。 Azure 資源成本可能會因為您所使用的 Azure 區域而有所不同。
  * 您也可以判斷另一個 Azure 供應項目類型是否會在 Azure 資源上提供更好的費率。
  
## <a name="partner-solutions"></a>合作夥伴解決方案
[Cloud Cruiser 和 Microsoft Azure 計費 API 整合](billing-usage-rate-card-partner-solution-cloudcruiser.md)描述 [Azure Pack 適用的 Cloud Cruiser Express](http://www.cloudcruiser.com/partners/microsoft/)如何直接從 Windows Azure Pack (WAP) 入口網站運作。 您可以順暢地從單一使用者介面管理 Microsoft Azure 私人或託管公用雲端的作業和財務等方面。   

## <a name="next-steps"></a>後續步驟
* 查看 GitHub 上的程式碼範例：
  * [發票 API 程式碼範例 (英文)](https://go.microsoft.com/fwlink/?linkid=845124)

  * [使用情況 API 程式碼範例 (英文)](https://github.com/Azure-Samples/billing-dotnet-usage-api)

  * [RateCard API 程式碼範例 (英文)](https://github.com/Azure-Samples/billing-dotnet-ratecard-api)

* 若要深入了解 Azure Resource Manager，請參閱 [Azure Resource Manager 概觀](../azure-resource-manager/resource-group-overview.md)。 



