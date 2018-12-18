---
title: Azure App Service 方案概觀 | Microsoft Docs
description: 了解 Azure App Service 之應用程式服務方案的運作方式，以及在管理經驗上帶來的效益。
keywords: App Service, Azure App Service, 級別, 可調整, 延展性, App Service 方案, App Service 成本
services: app-service
documentationcenter: ''
author: cephalin
manager: cfowler
editor: ''
ms.assetid: dea3f41e-cf35-481b-a6bc-33d7fc9d01b1
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/09/2017
ms.author: cephalin
ms.openlocfilehash: 268844eae8dc06937529e79d52515cad2f6da3f4
ms.sourcegitcommit: 7edfa9fbed0f9e274209cec6456bf4a689a4c1a6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/17/2018
ms.locfileid: "27862354"
---
# <a name="azure-app-service-plan-overview"></a>Azure App Service 方案概觀

在 App Service 中，應用程式會在 _App Service 方案_中執行。 App Service 方案會針對要執行的 Web 應用程式定義一組計算資源。 這些計算資源類似於傳統 Web 裝載中的[_伺服器陣列_](https://wikipedia.org/wiki/Server_farm)。 一或多個應用程式可設定為在相同的計算資源上執行 (或在相同的 App Service 方案中執行)。

當您在特定區域 (例如，西歐) 建立 App Service 方案時，會為該區域的方案建立一組計算資源。 無論您將何種應用程式置入此 App Service 方案，都會在該 App Service 方案定義的計算資源上執行。 每個 App Service 方案可定義：

- 區域 (美國西部、美國東部等)
- 虛擬機器執行個體的數目
- 虛擬機器執行個體的大小 (小、中、大)
- 定價層 (免費、共用、基本、標準、進階、PremiumV2、隔離、耗用)

App Service 方案的_定價層_可決定您獲得哪些 App Service 功能，以及為該方案支付多少費用。 定價層有幾個類別：

- **共用計算**：**免費**和**共用**，這兩個基底層會在與其他 App Service 應用程式相同的 Azure VM 上執行應用程式，包括其他客戶的應用程式。 這些層會將 CPU 配額配置到在共用資源上執行的每個應用程式，而且該資源無法向外延展。
- **專用計算**：**基本**、**標準**、**進階**和 **PremiumV2** 層會在專用的 Azure VM 上執行應用程式。 只有位於同個 App Service 方案中的應用程式，才會共用相同的計算資源。 層越高，可用於向外延展的 VM 執行個體就越多。
- **隔離**：這一層會在專用的 Azure 虛擬網路上執行專用的 Azure VM，為您的應用程式提供網路隔離與計算隔離。 它提供了最大的向外延展能力。
- **耗用**：這一層僅適用於[函數應用程式](../azure-functions/functions-overview.md)。 該層會根據工作負載動態延展函數。 如需詳細資訊，請參閱 [Azure Functions 裝載方案比較](../azure-functions/functions-scale.md)。

[!INCLUDE [app-service-dev-test-note](../../includes/app-service-dev-test-note.md)]

每一層也提供 App Service 功能的特定子集。 這些功能包括自訂網域和 SSL 憑證、自動調整、部署位置、備份、流量管理員整合等等。 層越高，可用的功能就越多。 若要了解每個定價層支援的功能，請參閱[App Service 方案詳細資料](https://azure.microsoft.com/pricing/details/app-service/plans/)。

<a name="new-pricing-tier-premiumv2"></a>

> [!NOTE]
> 新的 **PremiumV2** 層提供搭載更快速處理器的 [Dv2 系列 VM](../virtual-machines/windows/sizes-general.md#dv2-series)、SSD 儲存體，以及**標準**層雙倍的記憶體/核心比率。 **PremiumV2** 也支援透過增加執行個體計數來提高規模，同時仍提供標準方案中的所有進階功能。 現有**進階**層中可用的所有功能都包含在 **PremiumV2** 中。
>
> 類似於其他專用層，此層有三個可用的 VM 大小：
>
> - 小 (一個 CPU 核心，3.5 GiB 的記憶體) 
> - 中 (兩個 CPU 核心，7 GiB 的記憶體) 
> - 大 (四個 CPU 核心，14 GiB 的記憶體)  
>
> 如需 **PremiumV2** 價格資訊，請參閱 [App Service 價格](https://azure.microsoft.com/pricing/details/app-service/)。
>
> 若要開始使用新的 **PremiumV2** 定價層，請參閱[設定 App Service 的 PremiumV2 層](app-service-configure-premium-tier.md)。

## <a name="how-does-my-app-run-and-scale"></a>我的應用程式如何執行及調整縮放？

在**免費**和**共用**層，應用程式會收到共用 VM 執行個體上的 CPU 分鐘數，且無法向外延展。在其他層，應用程式的執行與調整方式如下所示。

當您在 App Service 中建立應用程式時，該應用程式會置入 App Service 方案。 當應用程式執行時，會在 App Service 方案中設定的所有 VM 執行個體上執行。 如果有多個應用程式位於相同的 App Service 方案，它們會共用相同的 VM 執行個體。 如果一個應用程式有多個部署位置，所有部署位置也會在相同的 VM 執行個體上執行。 如果您啟用診斷記錄、執行備份，或執行 WebJob，它們也會使用這些 VM 執行個體上的 CPU 週期和記憶體。

如此一來，App Service 方案是 App Service 應用程式的縮放單位。 如果方案設定為執行五個 VM 執行個體，則方案中的所有應用程式會在所有五個執行個體上執行。 如果方案設定為自動調整，則方案中的所有應用程式會根據自動調整設定一起向外延展。

如需有關向外延展應用程式的詳細資訊，請參閱[手動或自動調整執行個體計數](../monitoring-and-diagnostics/insights-how-to-scale.md)。

<a name="cost"></a>

## <a name="how-much-does-my-app-service-plan-cost"></a>我的 App Service 方案費用是多少？

本節描述 App Service 應用程式的計費方式。 如需特定區域價格的詳細資訊，請參閱 [App Service 價格](https://azure.microsoft.com/pricing/details/app-service/)。

除了**免費**層以外，App Service 方案會根據其每小時使用的計算資源計費。

- 在**共用**層，每個應用程式會收到 CPU 分鐘數的配額，因此_每個應用程式_會針對 CPU 配額每小時計費。
- 在專用的計算層 (**基本**、**標準**、**進階**、**PremiumV2**)，App Service 方案會定義調整應用程式的 VM 執行個體數目，因此，App Service 方案中的_每個 VM 執行個體_會每小時計費。 無論有多少個應用程式在 VM 執行個體上執行，這些 VM 執行個體皆採相同收費。 為了避免產生非預期的費用，請參閱[清除 App Service 方案](app-service-plan-manage.md#delete)。
- 在**隔離**層，App Service 環境會定義執行您應用程式的隔離背景工作角色數目，且_每個背景工作角色_會每小時計費。 此外，執行 App Service 環境本身也有每小時的基本費用。 
- (僅限 Azure Functions) **耗用**層會動態配置 VM 執行個體以處理函數應用程式的工作負載，且 Azure 會每秒動態計費。 如需詳細資訊，請參閱 [Azure Functions 價格](https://azure.microsoft.com/pricing/details/functions/)。

使用可用的 App Service 功能 (設定自訂網域、SSL 憑證、部署位置、備份等等) 不會向您收取費用。 例外狀況為：

- App Service 網域 - 當您在 Azure 中購買一個網域且採每年續訂時，即需要付費。
- App Service 憑證 - 當您在 Azure 中購買一個憑證且採每年續訂時，即需要付費。
- 以 IP 為主的 SSL 連線 - 每個以 IP 為主的 SSL 連線會依每小時計費，但是某些**標準**層或更高層級可讓您免費使用以 IP 為主的 SSL 連線。 以 SNI 為主的 SSL 連線是免費的。

> [!NOTE]
> 如果您將 App Service 與另一個 App Service 整合，可能需要考慮來自其他服務的費用。 例如，如果您使用 Azure 流量管理員來調整您的異地應用程式、Azure 流量管理員也會根據您的使用量來收費。 若要預估您在 Azure 中的跨服務成本，請參閱[價格計算器](https://azure.microsoft.com/pricing/calculator/)。 
>
>

## <a name="what-if-my-app-needs-more-capabilities-or-features"></a>如果我的應用程式需要更多的功能？

可以隨時相應增加和相應減少您的 App Service 方案。 這與更改方案的定價層一樣簡單。 一開始，您可以選擇較低的定價層，當您之後需要更多 App Service 功能時，再相應增加。

例如，您可以開始測試**免費** App Service 方案中的 Web 應用程式，無須支付任何費用。 當您想要將[自訂 DNS 名稱](app-service-web-tutorial-custom-domain.md) 加入 Web 應用程式時，只要將您的方案相應增加至**共用**層即可。 稍後，當您想要新增[自訂 SSL 憑證](app-service-web-tutorial-custom-ssl.md)時，將您的方案相應增加至**基本**層。 當您想要有[預備環境](web-sites-staged-publishing.md)時，相應增加至**標準**層。 當您需要更多核心、記憶體或儲存體時，可以在同一層中相應增加到較大的 VM 大小。

反向的運作方式也是一樣。 當您覺得不再需要更高層的功能時，可以相應減少到較低層，這樣可以節省費用。

如需有關相應增加 App Service 方案的詳細資訊，請參閱[在 Azure 中相應增加應用程式](web-sites-scale.md)。

如果您的應用程式與其他應用程式處於相同的 App Service 方案中，您可以藉由隔離計算資源以改善應用程式的效能。 您可以透過將應用程式移到不同的 App Service 方案來實現。 如需詳細資訊，請參閱[將應用程式移至另一個 App Service 方案](app-service-plan-manage.md#move)。

## <a name="should-i-put-an-app-in-a-new-plan-or-an-existing-plan"></a>我應該將應用程式置入新的方案或現有方案？

由於您要針對 App Service 方案配置的計算資源付費 (請參閱[我的 App Service 方案成本是多少？](#cost))，您可以將多個應用程式置入一個 App Service 方案來節省費用。 只要現有方案有足夠資源處理負載，即可繼續將應用程式新增至該方案。 不過，請記住，同個 App Service 方案中的應用程式皆會共用相同的計算資源。 若要判斷新的應用程式是否有所需的資源，您必須了解現有 App Service 方案的容量，以及新應用程式預期的負載。 多載 App Service 方案可能會導致新的和現有的應用程式停機。

如果有下列情況，請將您的應用程式隔離至新的 App Service 方案中：

- 應用程式會耗用大量資源。
- 您希望單獨調整應用程式，不需要現有方案中的其他應用程式。
- 應用程式需要不同地理區域中的資源。

這樣可讓您為應用程式配置一組新的資源，更充分掌控您的應用程式。

## <a name="manage-an-app-service-plan"></a>管理 App Service 方案

> [!div class="nextstepaction"]
> [管理 App Service 方案](app-service-plan-manage.md)
