---
title: Azure Kubernetes Service 中支援的 Kubernetes 版本
description: 了解 Azure Kubernetes Service (AKS) 中叢集的 Kubernetes 版本支援原則和生命週期
services: container-service
author: sauryadas
ms.service: container-service
ms.topic: article
ms.date: 09/21/2018
ms.author: saudas
ms.openlocfilehash: 6b55825107ae8872b146b3ad4fde0ef4b917b71d
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47045815"
---
# <a name="supported-kubernetes-versions-in-azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS) 中支援的 Kubernetes 版本

Kubernetes 社群大約每隔三個月就會發行次要版本。 這些版本均包含新功能和增強功能。 修補程式版本會更頻繁地發行 (有時每週)，而且僅適用於次要版本中的重要 Bug 修正。 這些修補程式版本包括對於安全性弱點或主要 Bug 的修正，這些安全性弱點或主要 Bug 會影響到根據 Kubernetes 在生產環境中執行的大量客戶和產品。

第一天會在 [acs-engine][acs-engine] 中提供新的 Kubernetes 次要版本。 AKS 服務等級目標 (SLO) 會將目標設定為在 30 天內發行適用於 AKS 叢集的次要版本，具體取決於版本的穩定性。

## <a name="kubernetes-version-support-policy"></a>Kubernetes 版本支援原則

AKS 支援 Kubernetes 的四個次要版本：

- 已發行上游 (n) 的目前次要版本
- 三個先前的次要版本。 每個支援的次要版本也支援兩個穩定的修補程式。

例如，如果 AKS 今天引入 *1.11.x*，也會針對 *1.10.a* + *1.10.b*、*1.9.c*  + *1.9 d*、*1.8.e* + *1.8f* (含有字母的修補程式版本是兩個最新穩定版的組建) 提供支援。

引入新的次要版本時，即會淘汰最舊的次要版本和所支援的修補程式版本。 在發行新次要版本及即將淘汰版本的前 15 天，會透過 Azure 更新通道進行宣告。 在上述發行 *1.11.x* 的範例中，已淘汰的版本為 *1.7.g* + *1.7.h*。

當您在入口網站中或使用 Azure CLI 部署 AKS 叢集時，一律會將叢集設定為 n-1 次要版本和最新修補程式。 例如，如果 AKS 支援 *1.11.x*、*1.10.a* + *1.10.b*、*1.9.c* + *1.9 d*、*1.8.e* + *1.8f*，新叢集的預設版本為 *1.10.b*。

## <a name="faq"></a>常見問題集

**當客戶使用不支援的次要版本升級 Kubernetes 叢集時，會發生什麼事？**

如果您是在 *n-4* 版本中，您就不會在 SLO 中。 如果從版本 n-4 成功升級到 n-3，則您會回到 SLO。 例如︰

- 如果支援的 AKS 版本為 *1.10.a* + *1.10.b*、*1.9.c* + *1.9 d*、*1.8.e* + *1.8f*，而且您在 *1.7.g* 或 *1.7.h* 中，您就不會在 SLO 中。
- 如果從 *1.7.g* 或 *1.7.h* 成功升級到 *1.8.e* 或 *1.8.f*，則您會回到 SLO。

不支援升級到早於 *n-4* 的版本。 在這類情況下，我們建議客戶建立新的 AKS 叢集，並重新部署其工作負載。

**當客戶使用不支援的次要版本調整 Kubernetes 叢集時，會發生什麼事？**

對於 AKS 不支援的次要版本，相應縮小或放大都會繼續運作，而且不會產生任何問題。

**客戶可以永遠停留在某個 Kubernetes 版本嗎？**

是。 不過，如果叢集不在 AKS 所支援的某個版本中，叢集就不會在 AKS SLO 中。 Azure 不會自動升級您的叢集或刪除它。

**如果代理程式叢集不在某個支援的 AKS 版本中，主要叢集會支援哪個版本？**

主要叢集會自動更新為最新支援版本。

## <a name="next-steps"></a>後續步驟

如需如何升級叢集的相關資訊，請參閱[升級 Azure Kubernetes Service (AKS) 叢集][aks-upgrade]。

<!-- LINKS - External -->
[acs-engine]: https://github.com/Azure/acs-engine

<!-- LINKS - Internal -->
[aks-upgrade]: upgrade-cluster.md