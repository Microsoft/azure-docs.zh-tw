---
title: Service Fabric 應用程式的容量規劃 | Microsoft Docs
description: 描述如何識別 Service Fabric 應用程式所需的計算節點的數目
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: markfuss
editor: ''
ms.assetid: 9fa47be0-50a2-4a51-84a5-20992af94bea
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: ac8abbdbbe9125ea036d837c08e1089aa6d1e55d
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212852"
---
# <a name="capacity-planning-for-service-fabric-applications"></a>Service Fabric 應用程式的容量規劃
本文將說明您如何估計執行 Azure Service Fabric 應用程式所需的資源量 (CPU、RAM、磁碟儲存空間)。 資源需求隨著時間改變是很常見的。 開發/測試您的服務時通常需要少數資源，之後進入生產環境且您的應用程式受歡迎度增長時會需要更多資源。 設計您的應用程式時，請考慮長期需求，並且選擇可讓您調整以符合高度客戶需求的服務。

 建立 Service Fabric 叢集時，您要決定組成叢集的虛擬機器 (VM) 類型。 每個 VM 會以 CPU (核心和速度)、網路頻寬、RAM 和磁碟儲存空間的形式隨附有限數量的資源。 當您的服務隨著時間成長時，可以升級為可提供更大量資源的 VM 和/或加入更多 VM 至您的叢集。 若要執行後者，您必須一開始就架構您的服務，讓服務可以利用新 VM 動態加入叢集的優點。

部分服務本身可管理 VM 上的少數資料或皆不管理。 因此，這些服務的容量規劃應主要著重在效能，這表示要選取適當的 VM CPU (核心和速度)。 此外，您應該考慮網路頻寬，包括發生網路傳輸的頻率特別以及要傳送的資料量。 如果隨著服務使用量增加時，您的服務需要執行良好，您可以加入更多 VM 到叢集，並跨所有 VM 將網路要求負載平衡。

如果是管理 VM 上大量資料的服務，容量規劃應主要著重在大小。 因此，您應該仔細地考量 VM 的 RAM 和磁碟儲存空間容量。 Windows 中的虛擬記憶體管理系統可讓應用程式程式碼將磁碟空間做為 RAM 使用。 此外，Service Fabric 執行階段提供智慧型分頁處理，只將熱資料保留在記憶體，而將冷資料移到磁碟。 因此應用程式就可以使用比 VM 上實際可用還多的記憶體。 有更多 RAM 可完全增加效能，因為 VM 可以在 RAM 中保留更多磁碟儲存空間。 您選取的 VM 要有夠大的磁碟空間來儲存您想要放在 VM 上的資料。 同樣地，VM 也要有足夠的 RAM，提供您想要的效能。 如果您的服務隨著時間成長，您可以加入更多 VM 到叢集，並跨所有 VM 分割資料。

## <a name="determine-how-many-nodes-you-need"></a>決定您需要多少個節點
將您的服務進行分割，可讓您相應放大您的服務資料。 如需有關分割的詳細資訊，請參閱 [分割 Service Fabric](service-fabric-concepts-partitioning.md)。 必須將每個資料分割納入單一 VM，但是可以將多個 (小型) 資料分割放在單一 VM 上。 因此，如果有比較多的小型資料分割，比起有比較少的大型資料分割，可提供您更大的彈性。 缺點是有很多資料分割會增加 Service Fabric 的負擔，並且您無法跨資料分割執行交易的作業。 如果您的服務程式碼經常需要存取位於不同的資料分割的資料片段，也會產生更多潛在的網路流量。 設計您的服務時，應該仔細考慮這些優缺點，以達成有效的資料分割策略。

假設您的應用程式有單一可設定狀態服務，其具有您預期一年會成長到 DB_Size GB 的存放區大小。 您願意在經歷該年的成長以外加入更多應用程式 (與資料分割)。  複寫因素 (RF) 決定影響總計 DB_Size 的服務複本數目。 所有複本的總計 DB_Size 是複寫因素乘以 DB_Size。  Node_Size 表示您想要用於服務的每一節點磁碟空間/RAM。 為了達到最佳效能，DB_Size 應符合整個叢集的記憶體，並且選擇接近 VM RAM 的 Node_Size。 如果配置大於 RAM 容量的 Node_Size，您就會依賴 Service Fabric 執行階段提供的分頁處理。 因此，如果整個資料都被視為熱資料 (因為從那時起資料會進入分頁/移出分頁)，您的效能就可能無法達到最佳。 不過，對只有一部分資料是熱資料的許多服務來說，還是比較具成本效益。

為了獲得最大效能所需的節點數目，可以依以下方式計算：

```
Number of Nodes = (DB_Size * RF)/Node_Size

```


## <a name="account-for-growth"></a>考慮成長
除了開始使用的 DB_Size，您可能想要根據您預期服務成長的 DB_Size 來計算節點數目。 然後，隨著服務的成長而讓節點數目成長，使得您不會過度佈建節點數目 但是，資料分割數目應該基於以最大成長執行您的服務時所需的節點數。

最好有一些額外的電腦隨時可用，以便您可以處理任何預期外的尖峰或失敗 (例如有少數 VM 停機)。  雖然這個額外容量要使用您預期的尖峰來決定，但一開始可以多保留幾個 VM (多 5-10%)。

以上假設使用單一的具狀態服務。 如果您有多個具狀態服務，則必須將與其他服務相關聯的 DB_Size 加入方程式。 或者，您可以分別為每個具狀態服務計算節點數目。  您的服務的複本或資料分割可能不平衡。 請記住，有些資料分割的資料可能比其他資料分割還多。 如需有關分割的詳細資訊，請參閱 [最佳作法的分割文章](service-fabric-concepts-partitioning.md)。 不過，上述方程式不受資料分割或複本影響，因為 Service Fabric 會確保複本以最佳化方式分散在節點之間。

## <a name="use-a-spreadsheet-for-cost-calculation"></a>使用試算表進行成本計算
現在，讓我們在公式中放入一些實際的數字。 [範例試算表](https://servicefabricsdkstorage.blob.core.windows.net/publicrelease/SF%20VM%20Cost%20calculator-NEW.xlsx) 示範如何規劃包含三種資料物件類型的應用程式的容量。 針對每個物件，我們會估計其大小以及我們預計具有的物件數。 我們也選取對每個物件類型我們想要的複本數。 試算表會計算要在叢集中儲存的記憶體數量總計。

然後，我們輸入 VM 大小和每月成本。 根據 VM 大小，試算表會告訴您必須用來將資料分割的最少分割數量，以實際上納入節點。 您可能想要較大量的資料分割以配合您的應用程式特定的計算和網路流量需求。 試算表顯示目前管理使用者設定檔物件的資料分割數目已從 1 增加到 6。

現在，根據這所有資訊，試算表會顯示您可以實際上取得在具有 26 個節點的叢集上所需的資料分割和複本的所有資料。 不過，此叢集會密集壓縮，因此您可能會想要一些其他節點來容納節點失敗和升級。 試算表也顯示具有超過 57 個節點不會提供任何額外的值，因為您會有空白節點。 同樣地，您可能想要有超過 57 節點以容納節點失敗和升級。 您可以調整試算表以符合應用程式的特定需求。   

![成本計算的試算表][Image1]

## <a name="next-steps"></a>後續步驟
查看[分割 Service Fabric 服務][10]來深入了解分割您的服務。

<!--Image references-->
[Image1]: ./media/SF-Cost.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-concepts-partitioning.md
