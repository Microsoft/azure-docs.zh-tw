---
title: 使用 Azure 藍圖針對錯誤進行疑難排解
description: 了解如何針對建立和指派藍圖的問題進行疑難排解
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: dd1163ece225c2e9a9b082f5e8364f34b06a10ae
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46982285"
---
# <a name="troubleshoot-errors-using-azure-blueprints"></a>針對使用 Azure 藍圖發生的錯誤進行疑難排解

您可能會在建立或指派藍圖時遇到錯誤。 此文章說明可能發生的各種錯誤與解決方式。

## <a name="finding-error-details"></a>尋找錯誤詳細資料

許多錯誤都是將藍圖指派給某個範圍而產生的結果。 當指派失敗時，藍圖會提供有關失敗部署的詳細資料。 此資訊將指出問題以便能夠進行修正，而使得後續部署將能成功。

1. 在左窗格中按一下 [所有服務]，然後搜尋並選取 [原則]，以在 Azure 入口網站中啟動 Azure 藍圖服務。 在 [原則] 頁面上，按一下 [藍圖]。

1. 從頁面左邊選取 [指派的藍圖]，然後使用搜尋方塊來篩選藍圖指派以尋找失敗的指派。 您也可以依 [佈建狀態] 欄排序指派表格，以查看所有已群組在一起的失敗指派。

1. 以滑鼠左鍵按一下具有「失敗」狀態的藍圖，或以滑鼠右鍵按一下並選取 [檢視指派詳細資料]。

1. 藍圖指派頁面的最上方是一個代表指派失敗的紅色橫幅警告。 按一下該橫幅上的任一處，以取得更多詳細資料。

錯誤通常是由藍圖中所含的成品 (而非整個藍圖) 造成的。 例如，如果藍圖包含建立 Key Vault 的成品，但 Azure 原則會阻止 Key Vault 建立，則整個指派將會失敗。

## <a name="general-errors"></a>一般錯誤

### <a name="policy-violation"></a>案例：違反原則

#### <a name="issue"></a>問題

範本部署因為原則違規而失敗。

#### <a name="cause"></a>原因

原則可能會因為多種因素而與部署產生衝突：

- 要建立的資源受到原則所限制 (通常是 SKU 或位置限制)
- 部署正在設定原則所設定的欄位 (通常會使用標記)

#### <a name="resolution"></a>解決方案

調整藍圖，使其不會與錯誤資訊中所列的原則產生衝突。 如果這不可行，替代選項就是變更原則指派的範圍，使藍圖不再與原則產生衝突。

## <a name="next-steps"></a>後續步驟

如果您看不到問題或無法解決問題，請瀏覽下列其中一個通道以取得更多支援：

- 透過 [Azure 論壇](https://azure.microsoft.com/support/forums/)獲得由 Azure 專家所提供的解答
- 與 [@AzureSupport](https://twitter.com/azuresupport) 連繫－專為改善客戶體驗而設的官方 Microsoft Azure 帳戶，協助 Azure 社群連接至適當的資源，像是解答、支援及專家等。
- 如果需要更多協助，您可以提出 Azure 支援事件。 請移至 [Azure 支援網站](https://azure.microsoft.com/support/options/)，然後選取 [取得支援]。