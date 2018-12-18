---
title: Azure Resource Graph 概觀
description: Azure Resource Graph 是 Azure 中的一個服務，能夠大規模進行複雜的資源查詢。
services: resource-graph
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: overview
ms.service: resource-graph
manager: carmonm
ms.custom: mvc
ms.openlocfilehash: d68183f4d0a928ac72f3f73ea5225ad174820cb7
ms.sourcegitcommit: 51a1476c85ca518a6d8b4cc35aed7a76b33e130f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47162095"
---
# <a name="what-is-azure-resource-graph"></a>何謂 Azure Resource Graph

Azure Resource Graph 是 Azure 中的一個服務，透過大規模查詢所有訂用帳戶及管理群組的能力，提供兼具效率及效能的資源探索延伸 Azure 資源管理，讓您可以有效的治理環境。 這些查詢提供下列功能：

- 根據資源屬性，使用複雜篩選、群組和排序查詢資源的能力。
- 以治理需求為基礎反覆探索資源，以及將結果運算式轉換成原則定義的能力。
- 評估在廣範圍雲端環境中套用原則所帶來影響的能力。

在此文件中，您會逐一了解每個功能的詳細資料。

> [!NOTE]
> Azure Resource Graph 已用於 Azure 入口網站新瀏覽「所有資源」的體驗。 它的目的旨在協助需要管理大規模環境的客戶。

## <a name="how-does-resource-graph-complement-azure-resource-manager"></a>Resource Graph 輔助 Resource Manager 的方式

Azure Resource Manager 目前會將資料傳送到公開數個資源欄位 (即資源名稱、識別碼、類型、資源群組、訂用帳戶及位置) 的有限資源快取。 現在，若您希望使用更多資源屬性，您必須呼叫每個個別資源提供者，並針對每個資源要求屬性詳細資料。

使用 Azure Resource Graph，您可以存取這些由資源提供者傳回的屬性，而無須對每個資源提供者進行個別呼叫。

## <a name="the-query-language"></a>查詢語言

現在您已更加了解 Azure Resource Graph，讓我們繼續深入了解如何建構查詢。

了解 Azure Resource Graph 的查詢語言是以 [Azure 資料總管查詢語言](../../data-explorer/data-explorer-overview.md)為基礎非常重要。

首先，如需可搭配 Azure Resource Graph 使用的作業及函式詳細資料，請參閱 [Resource Graph 查詢語言](./concepts/query-language.md)。 若要瀏覽資源，請參閱[探索資源](./concepts/explore-resources.md)。

## <a name="permissions-in-azure-resource-graph"></a>Azure Resource Graph 中的權限

若要使用 Resource Graph，您必須先透過[角色型存取控制](../../role-based-access-control/overview.md) (RBAC) 獲得授權，至少取得您欲查詢資源的讀取存取權限。 若您在管理群組、訂用帳戶、資源群組或個別資源上沒有 `read` 權限，它便不會傳回 Resource Graph 查詢的結果。

## <a name="running-your-first-query"></a>執行您的第一個查詢

Resource Graph 支援 Azure CLI 和 Azure PowerShell。 查詢元件的建構方式都是相同的，無論使用的語言為何。 根據預設，任一個 SDK 皆不支援 Azure Resource Graph，所以您必須載入延伸模組或模組才能提供所需的命令。
了解如何在 [Azure CLI](first-query-azurecli.md#add-the-resource-graph-extension) 和 [Azure PowerShell](first-query-powershell.md#add-the-resource-graph-module) 中啟用 Resource Graph。

## <a name="next-steps"></a>後續步驟

- 使用 [Azure CLI](first-query-azurecli.md) 執行第一個查詢
- 使用 [Azure PowerShell](first-query-powershell.md) 執行第一個查詢
- 從[入門查詢](./samples/starter.md)開始
- 使用[進階查詢](./samples/advanced.md)增強您的了解