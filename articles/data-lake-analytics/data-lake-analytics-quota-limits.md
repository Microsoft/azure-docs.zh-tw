---
title: 調整 Azure Data Lake Analytics 中的配額和限制
description: 了解如何調整並提高 Azure Data Lake Analytics (ADLA) 帳戶中的配額和限制。
services: data-lake-analytics
ms.service: data-lake-analytics
author: omidm1
ms.author: omidm
ms.reviewer: jasonwhowell
ms.assetid: 49416f38-fcc7-476f-a55e-d67f3f9c1d34
ms.topic: conceptual
ms.date: 03/15/2018
ms.openlocfilehash: 4629b52f3b2c9e351ddc2a68a40c5178a9a73950
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43048250"
---
# <a name="adjust-quotas-and-limits-in-azure-data-lake-analytics"></a>調整 Azure Data Lake Analytics 中的配額和限制

了解如何調整並提高 Azure Data Lake Analytics (ADLA) 帳戶中的配額和限制。 了解這些限制可幫助您了解 U-SQL 作業的行為。 所有配額限制並非強制，因此您可以與 Azure 支援連絡來提高上限。

## <a name="azure-subscriptions-limits"></a>Azure 訂用帳戶限制

**每個訂用帳戶在每個區域的 ADLA 帳戶最大數目**：5

如果您嘗試建立第六個 ADLA 帳戶，會收到「您已達到訂用帳戶名稱下區域中允許的 Data Lake Analytics 帳戶的數目上限 (5)」錯誤。 

如果您想要超過此限制，可嘗試這些選項：
* 選擇另一個區域 (如果適用)
* 透過[開立支援票證](#increase-maximum-quota-limits)連絡 Azure 支援服務，要求增加配額。

## <a name="default-adla-account-limits"></a>預設 ADLA 帳戶限制

**每個帳戶的分析單位 (AU) 數目上限：** 32

這是可以同時在您的帳戶中執行的 AU 的最大數目。 如果您的所有作業加起來的執行中 AU 總數超過此限制，系統會自動將較新的工作排入佇列。 例如︰

* 如果您只有一個作業使用 32 個 AU 在執行，當您提交第二個作業時，在第一個作業完成之前，第二個作業會在作業佇列中等待。
* 如果您已經有四個執行中的作業，而每個都使用 8 AU，當您送出第五個需要 8 AU 的作業時，它會在作業佇列中等到有 8 AU 可使用。

**每個作業的分析單位 (AU) 數目上限：** 32

這是每個個別作業可以在您的帳戶中指派的預設 AU 數目上限。 除非提交者受可提供每項作業更多 AU 的計算原則 (作業提交限制) 影響，否則系統會拒絕已指派超過此限制的作業。 此值的上限是帳戶的 AU 限制。

**每個帳戶的並行 U-SQL 作業最大數目︰** 20

這是可以同時在您的帳戶中執行之工作的最大數目。 超過此值，新的工作會自動排入佇列。

## <a name="adjust-adla-account-limits"></a>調整 ADLA 帳戶限制

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 選擇現有的 ADLA 帳戶。
3. 按一下 [內容] 。
4. 調整 [AU 上限]、[執行中工作上限] 和 [作業提交限制] 的值，以符合您的需求。

## <a name="increase-maximum-quota-limits"></a>增加配額限制上限

您可以在 [Azure 服務特定限制文件](../azure-subscription-service-limits.md#data-lake-analytics-limits)中找到有關 Azure 限制的詳細資訊。

1. 在 Azure 入口網站中開啟支援要求。

    ![Azure Data Lake Analytics 入口網站頁面](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-help-support.png)

    ![Azure Data Lake Analytics 入口網站頁面](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request.png)
2. 選取 [配額] 問題類型。
3. 選取您的 [訂用帳戶] \(請確定它不是「試用」的訂用帳戶\)。
4. 選取 [Data Lake Analytics] 配額類型。

    ![Azure Data Lake Analytics 入口網站頁面](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request-basics.png)

5. 在問題頁面中，說明您所要增加的限制，以及您為何需要這個額外的容量的**詳細資料**。

    ![Azure Data Lake Analytics 入口網站頁面](./media/data-lake-analytics-quota-limits/data-lake-analytics-quota-support-request-details.png)

6. 確認您的連絡資訊，建立支援要求。

Microsoft 會檢閱您的要求，並嘗試盡速符合您的業務需求。

## <a name="next-steps"></a>後續步驟

* [Microsoft Azure Data Lake Analytics 概觀](data-lake-analytics-overview.md)
* [使用 Azure PowerShell 管理 Azure Data Lake Analytics](data-lake-analytics-manage-use-powershell.md)
* [使用 Azure 入口網站監視和疑難排解 Azure Data Lake Analytics 作業](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
