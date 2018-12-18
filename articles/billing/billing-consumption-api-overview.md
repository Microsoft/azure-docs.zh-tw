---
title: Azure 使用量 API 概觀 | Microsoft Docs
description: 了解 Azure 使用量 API 如何讓您以程式設計方式存取 Azure 資源的成本和使用量資料。
services: billing
documentationcenter: ''
author: Erikre
manager: dougeby
editor: ''
tags: billing
ms.assetid: 68825e85-de64-466d-b11a-8baffde836b5
ms.service: billing
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: billing
ms.date: 6/07/2018
ms.author: erikre
ms.openlocfilehash: 88b7909e78f8bd36ce456eee60587acbbb94b6cd
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286743"
---
# <a name="azure-consumption-api-overview"></a>Azure 使用量 API 概觀 

Azure 使用量 API 可讓您以程式設計方式存取 Azure 資源的成本和使用量資料。 這些 API 目前僅支援 Enterprise 註冊和 Web Direct 訂用帳戶 (但有一些例外)。 API 會持續更新，以支援其他 Azure 訂用帳戶類型。

Azure 使用量 API 可存取：
- 企業和 Web Direct 客戶 
    - 使用量詳細資料 
    - Marketplace 費用 
    - 保留建議 
    - 保留詳細資料 
    - 保留摘要 
- 僅限企業客戶 
    - 價位表 
    - 預算 
    - 餘額 

## <a name="usage-details-api"></a>使用量詳細資料 API

使用使用量詳細資料 API 來取得所有 Azure 第一方資源的費用和使用量資料。 此資訊採用使用量詳細資料記錄格式，目前每天會針對每個資源的每個計量發出一次。 此資訊可用來加總所有資源的成本，或調查特定資源的成本/使用量。 

此 API 包含︰

-   **計量層級使用量資料** - 查看資料，包括使用量成本、發出費用的計量及費用所屬的 Azure 資源。 所有使用量詳細資料記錄都會對應至每日貯體。
-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **篩選** - 使用下列篩選，將您的 API 結果集向下修剪為較小的使用量詳細資料記錄集：
    - 使用結束/使用開始
    - 資源群組
    - 資源名稱
-   **資料彙總** - 使用 OData 依標記或篩選屬性將表達式套用至彙總使用量詳細資料
-   **不同供應項目類型的使用量** - 使用量詳細資訊目前適用於企業和 Web Direct 客戶。

如需詳細資訊，請參閱[使用量詳細資料 API](https://docs.microsoft.com/rest/api/consumption/usagedetails) 的技術規格。

## <a name="marketplace-charges-api"></a>Marketplace 費用 API

使用 Marketplace 費用 API 來取得所有 Marketplace 資源 (Azure 協力廠商供應項目) 的費用和使用量資料。 此資料可用來加總所有 Marketplace 資源的成本，或調查特定資源的成本/使用量。 

此 API 包含︰

-   **計量層級使用量資料** - 查看資料，包括 Marketplace 使用量成本、發出費用的計量及費用所屬的資源。 所有使用量詳細資料記錄都會對應至每日貯體。
-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **篩選** - 使用下列篩選，將您的 API 結果集向下修剪為較小的 Marketplace 記錄集：
    - 結束使用/開始使用
    - 資源群組
    - 資源名稱
-   **不同供應項目類型的使用量** - Marketplace 資訊目前適用於企業和 Web Direct 客戶。

如需詳細資訊，請參閱 [Marketplace 費用 API](https://docs.microsoft.com/rest/api/consumption/marketplaces) 的技術規格。

## <a name="balances-api"></a>餘額 API

企業客戶可以使用餘額 API 來取得餘額、新購買、Azure Marketplace 服務費用、調整，以及超額部分費用的每月摘要資訊。 您可以取得目前計費期間或任何過去期間的這項資訊。 企業可以使用此資料來執行與手動計算之摘要費用的比較。 此 API 不提供資源特定資訊和成本的彙總檢視。 

此 API 包含︰

-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **僅限企業客戶** - 此 API 僅適用於 EA 客戶。 
    - 客戶必須具有企業系統管理員權限才能呼叫此 API 

如需詳細資訊，請參閱[餘額 API](https://docs.microsoft.com/rest/api/consumption/balances) 的技術規格。

## <a name="budgets-api"></a>預算 API

企業客戶可以使用此 API 來建立資源、資源群組或計費計量的成本或使用量預算。 確定這項資訊之後，即可設定警示，在超過使用者定義的預算閾值時通知。 

此 API 包含︰

-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **僅限企業客戶** - 此 API 僅適用於 EA 客戶。
-   **可設定的通知** - 指定讓使用者在預算有誤時收到通知。
-   **使用量或成本型預算** - 依照您的案例需求，建立依據使用量或成本的預算。
-   **篩選** - 使用下列可設定的篩選，將您的預算篩選為較小的資源子集
    - 資源群組
    - 資源名稱
    - 計量
-   **可設定的預算時段** - 指定預算應重設的頻率，以及預算有效的時間長度。

如需詳細資訊，請參閱[預算 API](https://docs.microsoft.com/rest/api/consumption/budgets) 的技術規格。

## <a name="reservation-recommendations-api"></a>保留建議 API

使用此 API 來取得購買保留的 VM 執行個體建議。 建議的設計目的是為了讓客戶分析預期節省的成本和購買數量。 

此 API 包含︰

-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **篩選** - 使用下列篩選，量身打造您的建議結果：
    - 影響範圍
    - 回顧期間
-   **不同供應項目類型的保留資訊** - 保留資訊目前適用於企業和 Web Direct 客戶。

如需詳細資訊，請參閱[保留建議 API](https://docs.microsoft.com/rest/api/consumption/reservationrecommendations) 的技術規格。

## <a name="reservation-details-api"></a>保留詳細資料 API

使用保留詳細資料 API 來查看先前購買之 VM 保留的相關資訊，例如已保留之使用量與實際使用量的比較。 您可以查看每個 VM 層級的詳細資料。 

此 API 包含︰

-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **篩選** - 使用下列篩選，將您的 API 結果集向下修剪為較小的保留集：
    - 日期範圍
-   **不同供應項目類型的保留資訊** - 保留資訊目前適用於企業和 Web Direct 客戶。

如需詳細資訊，請參閱[保留詳細資料 API](https://docs.microsoft.com/rest/api/consumption/reservationsdetails) 的技術規格。

## <a name="reservation-summaries-api"></a>保留摘要 API

使用此 API 來查看先前購買之 VM 保留的相關彙總資訊，例如已保留之使用量與實際使用量的彙總比較。 

此 API 包含︰

-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **篩選** - 使用下列篩選，量身打造使用每日資料粒度時的結果：
    - 使用日期
-   **不同供應項目類型的保留資訊** - 保留資訊目前適用於企業和 Web Direct 客戶。
-   **每日或每月彙總** - 呼叫端可以指定想要每日或每月資料粒度的保留摘要資料。

如需詳細資訊，請參閱[保留摘要 API](https://docs.microsoft.com/rest/api/consumption/reservationssummaries) 的技術規格。

## <a name="price-sheet-api"></a>價位表 API
企業客戶可以使用此 API 來擷取所有計量的自訂定價。 企業可以使用此 API 搭配使用量詳細資料和 Marketplace 使用量資訊，透過使用量和 Marketplace 資料執行成本計算。 

此 API 包含︰

-   **Azure 角色型存取控制** - 在 [Azure 入口網站](https://portal.azure.com)上或是透過 [Azure CLI](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-cli) 或 [Azure PowerShell Cmdlet](https://docs.microsoft.com/powershell/azure/overview) 設定存取原則，以指定哪些使用者或應用程式可以存取訂用帳戶的使用量資料。 呼叫端必須使用標準的 Azure Active Directory 權杖進行驗證。 將呼叫端新增到「發票讀者」、「讀者」、「擁有者」或「參與者」角色，以存取特定 Azure 訂用帳戶的使用情況資料。 
-   **僅限企業客戶** - 此 API 僅適用於 EA 客戶。 Web Direct 客戶應該使用 RateCard API 來取得定價。 

如需詳細資訊，請參閱[價位表 API](https://docs.microsoft.com/rest/api/consumption/pricesheet) 的技術規格。

## <a name="scenarios"></a>案例

以下是一些可透過使用量 API 進行的案例：

-   **發票對帳** - Microsoft 向我收取的金額是否正確？  我的帳單內容為何，我是否可以自行計算？
-   **交叉收費** - 現在我知道要支付的費用，我的組織中誰需要支付？
-   **成本最佳化** - 我知道已支付了多少費用… 如何才能充分利用花費在 Azure 上的每分每毫？
-   **成本追蹤** - 我想要查看我在一段時間內使用 Azure 所花費的成本。 趨勢為何？ 如何改進？
-   **Azure 當月支出** - 我在本月迄今的支出為何？ 我是否需要對 Azure 費用及/或使用量進行任何調整？ 我的 Azure 使用量在當月何時最高？
-   **設定警示** - 我想要設定資源型使用量或以預算為基礎的貨幣型警示。

## <a name="next-steps"></a>後續步驟

- 如需使用 Azure 計費 API 以程式設計方式取得 Azure 使用量見解的資訊，請參閱 [Azure 計費 API 概觀](billing-usage-rate-card-overview.md)。



