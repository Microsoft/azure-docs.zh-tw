---
title: Azure Key Vault 節流指導方針
description: Key Vault 節流措施會限制並行呼叫數目，以防止過度使用資源。
services: key-vault
documentationcenter: ''
author: bryanla
manager: mbaldwin
tags: ''
ms.assetid: 9b7d065e-1979-4397-8298-eeba3aec4792
ms.service: key-vault
ms.workload: identity
ms.topic: article
ms.date: 05/10/2018
ms.author: bryanla
ms.openlocfilehash: 28756cf28305927246d82f1f006f02b2e9b96469
ms.sourcegitcommit: 0fcd6e1d03e1df505cf6cb9e6069dc674e1de0be
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/14/2018
ms.locfileid: "42144006"
---
# <a name="azure-key-vault-throttling-guidance"></a>Azure Key Vault 節流指導方針

節流是您起始來限制 Azure 服務並行呼叫數目，以避免過度使用資源的程序。 Azure Key Vault (AKV) 被設計來處理大量的要求。 如果產生大量的要求，針對用戶端的要求進行節流有助於維護 AKV 服務的最佳效能和可靠性。

節流限制視情節而異。 例如，如果您正在執行大量的寫入，則節流的可能性要高於僅執行讀取作業的可能性。

## <a name="how-does-key-vault-handle-its-limits"></a>Key Vault 如何處理其限制？

Key Vault 中的服務限制是為了防止濫用資源，並確保所有 Key Vault 用戶端的服務品質。 超過服務閾值時，Key Vault 可以限制一段時間內來自該用戶端的任何進一步要求。 當這種情況發生時，Key Vault 會傳回 HTTP 狀態碼 429 (太多要求)，且要求會失敗。 此外，傳回狀態碼 429 的失敗要求會計入 Key Vault 所追蹤的節流限制中。 

如果您有需要更高節流限制的有效商務案例，請與我們連絡。


## <a name="how-to-throttle-your-app-in-response-to-service-limits"></a>如何為您的應用程式進行節流處理，以回應服務限制

以下是為您應用程式進行節流處理的**最佳做法**：
- 減少每個要求的作業數目。
- 減少要求的頻率。
- 避免立即重試。 
    - 所有要求都會讓您的使用量限制累加。

當您實作應用程式的錯誤處理時，請使用 HTTP 錯誤碼 429 來偵測用戶端節流的需求。 如果要求再次失敗並傳回 HTTP 429 錯誤碼，您仍然遇到 Azure 服務限制。 繼續使用建議的用戶端節流方法，重試要求直到成功為止。

### <a name="recommended-client-side-throttling-method"></a>建議使用的用戶端節流方法

針對 HTTP 錯誤碼 429，使用指數型輪詢方法開始為您的用戶端進行節流處理：

1. 等候 1 秒，重試要求
2. 如果等候 2 秒仍然進行節流處理，重試要求
3. 如果等候 4 秒仍然進行節流處理，重試要求
4. 如果等候 8 秒仍然進行節流處理，重試要求
5. 如果等候 16 秒仍然進行節流處理，重試要求

此時，您應該不會收到 HTTP 429 回應碼。

## <a name="see-also"></a>另請參閱

如需在 Microsoft Cloud 上進行節流處理的詳細資訊，請參閱[節流模式](https://docs.microsoft.com/azure/architecture/patterns/throttling) \(英文\)。

