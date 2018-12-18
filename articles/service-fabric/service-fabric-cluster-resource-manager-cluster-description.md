---
title: 叢集資源管理員叢集描述 | Microsoft Docs
description: 將容錯網域、升級網域、節點屬性和節點容量指定給叢集資源管理員，以描述 Service Fabric 叢集。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 55f8ab37-9399-4c9a-9e6c-d2d859de6766
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 13ee238580d645f3e727090bc0e0275b36bdb225
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34208806"
---
# <a name="describing-a-service-fabric-cluster"></a>描述 Service Fabric 叢集
Service Fabric 叢集資源管理員提供數種機制，來描述叢集。 在執行階段，叢集資源管理員會使用此資訊，以確保叢集中執行之服務的高可用性。 在強制執行這些重要規則時，它也會嘗試將叢集的資源消耗量最佳化。

## <a name="key-concepts"></a>重要概念
叢集資源管理員支援數個描述叢集的功能︰

* 容錯網域
* 升級網域
* 節點屬性
* 節點容量

## <a name="fault-domains"></a>容錯網域
容錯網域是任何連鎖故障區域。 單一電腦是一個容錯網域 (因為它本身可能因各種原因而失敗，例如電源供應器故障、磁碟機故障或 NIC 韌體不正確)。 連線至相同乙太網路交換器的電腦位於相同的容錯網域，共用單一位置之單一電源的電腦也是如此。 由於硬體故障天生就會重疊，容錯網域本質上就具備階層性，且在 Service Fabric 中以 URI 表示。

必須正確設定容錯網域，因為 Service Fabric 會使用這項資訊來安全地放置服務。 Service Fabric 放置服務時會注意不要因為遺失容錯網域 (某個元件失敗所造成) 而導致服務中斷。 在 Azure 環境中，Service Fabric 會使用環境所提供的容錯網域資訊，代替您正確地設定叢集中的節點。 對於 Service Fabric Standalone，容錯網域會在叢集設定時定義。 

> [!WARNING]
> 將正確的容錯網域資訊提供給 Service Fabric 相當重要。 例如，假設 Service Fabric 叢集是在 10 部虛擬機器上執行，這些虛擬機器是在五個實體主機上執行。 在此情況下，即使有 10 部虛擬機器，也只有 5 個不同的 (最上層) 容錯網域。 共用相同的實體主機會造成 VM 共用相同的根容錯網域，因為如果其實體主機失敗，VM 會遇到協調失敗。  
>
> Service Fabric 預期節點的容錯網域不會變更。 確保 VM (例如 [HA-VM](https://technet.microsoft.com/library/cc967323.aspx)) 高可用性的其他機制可能會造成與 Service Fabric 的衝突，因為它們是使用從一台主機到另一台主機的 VM 透明移轉。 這些機制不會重新設定或通知在 VM 內的執行中程式碼。 因此，它們**不支援**作為執行中 Service Fabric 叢集的環境。 Service Fabric 應該僅採用高可用性技術。 不需要像是即時 VM 移轉、SAN 或其他項目的機制。 如果與 Service Fabric 搭配使用，這些機制會_減少_應用程式可用性和可靠性，因為它們會導致額外的複雜性、新增集中式失敗來源，並且使用與 Service Fabric 衝突的可靠性和可用性策略。 
>
>

下圖中，我們將所有參與容錯網域的實體塗上顏色，並列出形成的所有不同容錯網域。 在此範例中，我們有資料中心 ("DC")、機架 ("R") 和刀鋒伺服器 ("B")。 如果每個刀鋒伺服器包含一個以上的虛擬機器，可以預料，容錯網域階層中還會有另一層。

<center>
![透過容錯網域組織的節點][Image1]
</center>

在執行階段，Service Fabric 叢集資源管理員會考慮叢集中的容錯網域，並規劃配置。 指定服務的具狀態複本或無狀態執行個體會分散，讓它們位在不同的容錯網域中。 跨容錯網域分散服務，可確保當容錯網域在階層的任何層級失敗時，服務的可用性不會受到危害。

Service Fabric 的叢集資源管理員不在乎容錯網域階層中有多少層。 不過，當階層的任何一部分遺失時，它會嘗試確保這不會影響上方執行的服務。 

在容錯網域階層中每個深度上，最好有相同的節點數目。 如果叢集中的容錯網域「樹狀結構」不平衡，叢集資源管理員會很難找出服務的最佳配置。 不平衡的容錯網域配置表示遺失某些網域時，服務可用性受影響的程度可能大於其他網域。 因此，叢集資源管理員在兩個目標之間掙扎︰希望將服務放在該「重負荷」網域中的電腦上以使用這些電腦，也希望在其他網域放置服務時不要因為遺失網域而造成問題。 

不平衡網域外觀為何？ 下圖中，我們顯示兩個不同的叢集配置。 在第一個範例中，節點平均分散至容錯網域。 在第二個範例中，一個容錯網域比其他容錯網域具有更多節點。 

<center>
![兩個不同的叢集配置][Image2]
</center>

在 Azure 中會替您選擇哪一個容錯網域包含節點。 不過，根據您佈建的節點數目而定，某些容錯網域的節點最後仍可能比其他容錯網域更多。 例如，假設您在叢集中有五個容錯網域，但佈建特定 NodeType 的七個節點。 在此情況下，前兩個容錯網域最後會有較多節點。 如果您繼續部署多個 NodeType，但只有幾個執行個體，則問題會更糟。 基於這個原因，建議每個節點類型中的節點數目是容錯網域數目的倍數。

## <a name="upgrade-domains"></a>升級網域
升級網域是另一項功能，可協助 Service Fabric 叢集資源管理員了解叢集的配置。 升級網域定義一組同時升級的節點。 升級網域可協助叢集資源管理員了解及協調例如升級的管理作業。

升級網域非常類似容錯網域，但是有幾個主要的差異。 首先，協調硬體失敗的區域會定義容錯網域。 另一方面，升級網域會定義原則。 您應該決定想要多少個節點，而不是取決於環境。 您可能會有與節點一樣多的升級網域。 容錯網域和升級網域的另一個差異在於升級網域不是階層式。 相反地，它們更像是簡單的標記。 

下圖顯示三個升級網域等量分佈在三個容錯網域上。 其中也顯示不具狀態服務的三個不同複本有一個可能的位置，而每一個最後都在不同的容錯網域和升級網域。 在服務升級期間，即使遺失容錯網域，這個位置可讓我們仍然保有一份程式碼和資料。  

<center>
![容錯網域和升級網域的位置][Image3]
</center>

大量升級網域有其利弊。 升級網域越多表示每個升級步驟會更輕微，因此受影響的節點或服務較少。 因為必須一次移動的服務較少，對系統的影響較小。 這樣較容易改善可靠性，因為升級期間發生的任何問題會影響較少的服務。 升級網域越多，也表示其他節點上需要較少的可用緩衝區來處理升級的影響。 例如，若您有五個升級網域，則每個網域的節點可處理大約 20% 的流量。 如果您需要關閉升級網域以進行升級，則負載通常必須移至別處。 因為您有四個剩餘的升級網域，每一個都必須有大約總流量 5% 的空間。 多個升級網域表示您在叢集之節點上需要較少的緩衝區。 例如，請考慮如果您擁有 10 個升級網域。 在此情況下，每個 UD 只會處理大約總流量的 10%。 當升級步驟是透過叢集時，每個網域只需要有大約總流量 1.1% 的空間。 多個升級網域通常可讓您在更高的使用率之下執行您的節點，因為您需要較少的保留容量。 同樣的作法也適用於容錯網域。  

擁有許多升級網域的缺點是升級通常需要較長的時間。 Service Fabric 會在升級網域完成之後等候一段時間，並且在開始升級下一個升級網域之前執行檢查。 這些延遲可讓系統在升級繼續之前偵測由升級帶來的問題。 缺點是可接受的，因為它會防止不良的變更一次影響到太多服務。

升級網域太少有許多負面的副作用 - 當個別升級網域關閉來升級時，整體容量會有一大部分無法使用。 例如，若您只有三個升級網域，則一次就關閉大約 1/3 的整體服務或叢集容量。 服務不宜一下子就關閉這麼多，因為叢集的剩餘部分必須有足夠容量來處理工作負載。 維護該緩衝區表示這些節點在一般作業期間的負載比其他期間少。 這會增加執行服務的成本。

環境中的容錯網域或升級網域總數沒有實際限制，它們的重疊方式也不受約束。 話雖如此，有幾個常見模式：

- 1:1 對應的容錯網域和升級網域
- 每個節點 (實體或虛擬作業系統執行個體) 一個升級網域
- 「等量」或「矩陣」模型，其中的容錯網域和升級網域形成一個矩陣，而電腦通常沿著對角線排列。

<center>
![容錯網域和升級網域配置][Image4]
</center>

對於選擇哪個配置並沒有最佳答案，每個答案都各有優缺點。 例如，1FD:1UD 模型相當容易設定。 每個節點 1 個升級網域模型，是使用者最常使用的模型。 在升級期間每個節點會獨立更新。 這類似於以往手動升級小型集合機器的方式。 

最常見的模型是 FD/UD 矩陣，其中 FD 和 UD 形成一個表格，而節點沿著對角線開始放置。 這是在 Azure 中的 Service Fabric 叢集預設使用的模型。 對於具有許多節點的叢集，所有項目最後看起來就像是上述的密集矩陣模式。

## <a name="fault-and-upgrade-domain-constraints-and-resulting-behavior"></a>容錯網域和升級網域的條件約束和導致的行為
### <a name="default-approach"></a>預設方法
根據預設，「叢集資源管理員」會讓「容錯網域」與「升級網域」之間的服務保持平衡。 這會經由模型化成為[條件約束](service-fabric-cluster-resource-manager-management-integration.md)。 「容錯網域」和「升級網域」條件約束陳述：「針對特定的服務分割區，相同階層層級上任兩個網域之間的服務物件 (無狀態服務執行個體或具狀態服務複本) 數目差異應該一律不能大於一」。 假設此條件約束提供「差異上限」保證。 此「容錯網域」和「升級網域」條件約束可防止違反上述規則的某些措施或安排。 

讓我們來看看一個範例。 假設我們的叢集有六個節點，且已設定五個容錯網域和五個升級網域。

|  | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| **UD0** |N1 | | | | |
| **UD1** |N6 |N2 | | | |
| **UD2** | | |N3 | | |
| **UD3** | | | |N4 | |
| **UD4** | | | | |N5 |

設定 1

現在假設我們建立 TargetReplicaSetSize (或者對於無狀態服務是 InstanceCount) 為五的服務。 複本將會落在 N1-N5 上。 事實上，不論建立多少像這樣的服務，都不會用到 N6。 原因為何？ 讓我們看看目前的配置和選擇 N6 時情況如何之間的差異。

以下是我們的配置，以及每個容錯網域和升級網域的複本總數。

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** | |R2 | | | |1 |
| **UD2** | | |R3 | | |1 |
| **UD3** | | | |R4 | |1 |
| **UD4** | | | | |R5 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

配置 1


就每個容錯網域和升級網域的節點數而論，在此配置達到平衡。 在每個容錯網域和升級網域的複本數目方面也達到平衡。 每個網域都擁有相同數量的節點，以及相同數量的複本。

現在，讓我們看看改用 N6 而不使用 N2 的情況。 複本將會如何散佈？

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** |R5 | | | | |1 |
| **UD2** | | |R2 | | |1 |
| **UD3** | | | |R3 | |1 |
| **UD4** | | | | |R4 |1 |
| **FDTotal** |2 |0 |1 |1 |1 |- |

配置 2


此配置違反「容錯網域」條件約束的「差異上限」保證定義。 FD0 有兩個複本，而 FD1 沒有任何複本，所以 FD0 與 FD1 之間的差異總計為 2，大於差異上限 1。 由於違反條件約束，因此「叢集資源管理員」不會允許這個安排。 同樣地，如果挑選 N2 和 N6 (而不是 N1 和 N2)，則會得到：

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** | | | | | |0 |
| **UD1** |R5 |R1 | | | |2 |
| **UD2** | | |R2 | | |1 |
| **UD3** | | | |R3 | |1 |
| **UD4** | | | | |R4 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

配置 3


此配置就容錯網域而言是平衡的。 不過，現在它違反「升級網域」條件約束，因為 UD0 沒有任何複本，而 UD1 有兩個複本。 因此，此配置也無效，而且不會被叢集資源管理員挑選。

這個分配具狀態複本或無狀態執行個體的方法提供了可能的最佳容錯效果。 在一個網域停止運作的情況下，可讓遺失的複本/執行個體數目降到最低。 

另一方面，此方法則可能太過嚴苛，而不允許叢集利用所有資源。 就某些叢集設定而言，會無法使用某些節點。 這可能造成 Service Fabric 無法安置您的服務，而導致產生警告訊息。 在先前的範例中，無法使用某些叢集節點 (所提供範例中的 N6)。 即使您將節點 (N7 – N10) 新增至該叢集，但因為「容錯網域」和「升級網域」條件約束的緣故，複本/執行個體也只會放置在 N1 – N5 上。 

|  | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| **UD0** |N1 | | | |N10 |
| **UD1** |N6 |N2 | | | |
| **UD2** | |N7 |N3 | | |
| **UD3** | | |N8 |N4 | |
| **UD4** | | | |N9 |N5 |

設定 2


### <a name="alternative-approach"></a>替代方法

「叢集資源管理員」支援另一個版本的「容錯網域」和「升級網域」條件約束，此版本既允許安置，同時也保證最低限度的安全性。 替代的「容錯網域」和「升級網域」條件約束可陳述如下：「針對特定的服務分割區，複本在網域的分配方式應該確保分割區不會遭受仲裁遺失」。 假設此條件約束提供「仲裁防護」保證。 

> [!NOTE]
>就具狀態服務而言，我們將「仲裁遺失」定義為一種大多數分割區複本都同時停止運作的情況。 例如，如果 TargetReplicaSetSize 是 5，則由任 3 個複本組成的集合即代表仲裁。 同樣地，如果 TargetReplicaSetSize 是 6，則仲裁必須要有 4 個複本。 在這兩種情況下，如果分割區想要繼續正常運作，就不可以有超過 2 個複本同時停止運作。 就無狀態服務而言，並沒有所謂的「仲裁遺失」情況，因為即使大多數執行個體都同時停止運作，無狀態服務仍然會繼續正常運作。 因此，在剩餘的內容中，我們會將焦點放在具狀態服務。
>

讓我們回到先前的範例。 使用「仲裁防護」版本的條件約束時，所有三個提供的配置都會有效。 這是因為即使第二個配置中的 FD0 或第三個配置中的 UD1 會發生失敗，分割區仍然會有仲裁 (其大多數複本仍然會正常運作)。 使用此版本的條件約束時，幾乎一律會使用 N6。

與「差異上限」方法相比，「仲裁防護」方法提供較多的彈性，因為較容易找出在幾乎任何叢集拓撲中都有效的複本分配方式。 不過，此方法無法保證最佳容錯特性，因為有些錯誤比其他錯誤來得嚴重。 在最糟糕的案例情況中，可能因一個網域再加上一個複本發生錯誤，就遺失大多數複本。 例如，在有 5 個複本或執行個體的情況下，您現在可能只要有 2 個失敗就會失去多數地位，而不是必須要有 3 個失敗才會遺失仲裁。 

### <a name="adaptive-approach"></a>調適型方法
由於這兩種方法都有優缺點，因此我們引進一個結合這兩種策略的調適型方法。

> [!NOTE]
>從 Service Fabric 6.2 版開始，這將會是預設行為。 
>
調適型方法預設會使用「差異上限」邏輯，而只有在必要時，才會切換至「仲裁防護」邏輯。 「叢集資源管理員」會查看叢集和服務的設定方式，來自動查明哪個策略是必要策略。 針對特定的服務：如果 TargetReplicaSetSize 可被「容錯網域」數目和「升級網域」數目整除，**並且**節點數目小於或等於 (容錯網域數目) * (升級網域數目)，「叢集資源管理員」應該就會針對該服務使用「仲裁型」邏輯。 請記住，儘管仲裁遺失與無狀態服務並不相關，但「叢集資源管理員」針對無狀態服務和具狀態服務都會使用此方法。

讓我們回到先前的範例並假設叢集現在有 8 個節點 (叢集仍然是設定為有 5 個「容錯網域」和 5 個「升級網域」，且該叢集上所裝載服務的 TargetReplicaSetSize 仍然是 5)。 

|  | FD0 | FD1 | FD2 | FD3 | FD4 |
| --- |:---:|:---:|:---:|:---:|:---:|
| **UD0** |N1 | | | | |
| **UD1** |N6 |N2 | | | |
| **UD2** | |N7 |N3 | | |
| **UD3** | | |N8 |N4 | |
| **UD4** | | | | |N5 |

設定 3

由於已滿足所有必要條件，因此「叢集資源管理員」將會使用「仲裁型」邏輯來分配服務。 這會允許使用 N6 – N8。 在此情況下，其中一個可能的服務分配方式會看起來像這樣：

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |R1 | | | | |1 |
| **UD1** |R2 | | | | |1 |
| **UD2** | |R3 |R4 | | |2 |
| **UD3** | | | | | |0 |
| **UD4** | | | | |R5 |1 |
| **FDTotal** |2 |1 |1 |0 |1 |- |

配置 4

如果服務的 TargetReplicaSetSize 縮減為 4 (舉例來說)，「叢集資源管理員」將會注意到該變更而又繼續使用「差異上限」邏輯，因為 TargetReplicaSetSize 已不再能夠被 FD 和 UD 整除。 因此，將會進行特定的複本移動以在 N1-N5 節點上分配剩餘的 4 個複本，如此才不會違反「差異上限」版本的「容錯網域」和「升級網域」邏輯。 

回顧第 4 個配置和 TargetReplicaSetSize 為 5 的情況。 如果從叢集中移除 N1，「升級網域」的數目就會變成等於 4。 同樣地，「叢集資源管理員」會開始使用「差異上限」邏輯，因為 UD 的數目已不再能夠整除服務的 TargetReplicaSetSize。 因此，當再次建置複本 R1 時，必須將其放在 N4 上，如此才不會違反「升級網域條件約束」。

|  | FD0 | FD1 | FD2 | FD3 | FD4 | UDTotal |
| --- |:---:|:---:|:---:|:---:|:---:|:---:|
| **UD0** |N/A |N/A |N/A |N/A |N/A |N/A |
| **UD1** |R2 | | | | |1 |
| **UD2** | |R3 |R4 | | |2 |
| **UD3** | | | |R1 | |1 |
| **UD4** | | | | |R5 |1 |
| **FDTotal** |1 |1 |1 |1 |1 |- |

配置 5

## <a name="configuring-fault-and-upgrade-domains"></a>設定容錯網域和升級網域
Azure 託管的 Service Fabric 部署中會自動定義容錯網域和升級網域。 Service Fabric 會從 Azure 中取用環境資訊。

如果想要建立您自己的叢集 (或想要在開發環境中執行特定拓撲)，您可以自行提供容錯網域和升級網域資訊。 在此範例中，我們定義本機開發叢集，具有九個節點，且跨越三個「資料中心」(各有三個機架)。 此叢集也有三個升級網域等量分散於這些三個資料中心。 設定的範例如下： 

ClusterManifest.xml

```xml
  <Infrastructure>
    <!-- IsScaleMin indicates that this cluster runs on one-box /one single server -->
    <WindowsServer IsScaleMin="true">
      <NodeList>
        <Node NodeName="Node01" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType01" FaultDomain="fd:/DC01/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node02" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType02" FaultDomain="fd:/DC01/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node03" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType03" FaultDomain="fd:/DC01/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
        <Node NodeName="Node04" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType04" FaultDomain="fd:/DC02/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node05" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType05" FaultDomain="fd:/DC02/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node06" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType06" FaultDomain="fd:/DC02/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
        <Node NodeName="Node07" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType07" FaultDomain="fd:/DC03/Rack01" UpgradeDomain="UpgradeDomain1" IsSeedNode="true" />
        <Node NodeName="Node08" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType08" FaultDomain="fd:/DC03/Rack02" UpgradeDomain="UpgradeDomain2" IsSeedNode="true" />
        <Node NodeName="Node09" IPAddressOrFQDN="localhost" NodeTypeRef="NodeType09" FaultDomain="fd:/DC03/Rack03" UpgradeDomain="UpgradeDomain3" IsSeedNode="true" />
      </NodeList>
    </WindowsServer>
  </Infrastructure>
```

獨立部署透過 ClusterConfig.json

```json
"nodes": [
  {
    "nodeName": "vm1",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD1"
  },
  {
    "nodeName": "vm2",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD2"
  },
  {
    "nodeName": "vm3",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD3"
  },
  {
    "nodeName": "vm4",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc2/r0",
    "upgradeDomain": "UD1"
  },
  {
    "nodeName": "vm5",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc2/r0",
    "upgradeDomain": "UD2"
  },
  {
    "nodeName": "vm6",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc2/r0",
    "upgradeDomain": "UD3"
  },
  {
    "nodeName": "vm7",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc3/r0",
    "upgradeDomain": "UD1"
  },
  {
    "nodeName": "vm8",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc3/r0",
    "upgradeDomain": "UD2"
  },
  {
    "nodeName": "vm9",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc3/r0",
    "upgradeDomain": "UD3"
  }
],
```

> [!NOTE]
> 透過 Azure Resource Manager 定義叢集時，容錯網域和升級網域會由 Azure 指派。 因此，Azure Resource Manager 範本中的節點類型和虛擬機器擴展集定義不包含容錯網域或升級網域資訊。
>

## <a name="node-properties-and-placement-constraints"></a>節點屬性和放置條件約束
有時候 (事實上是大部分的情況下) 您會想要確保工作負載只在叢集中的特定節點類型上執行。 例如，某些工作負載可能需要 GPU 或 SSD，而有些則不用。 將硬體專用於特定工作負載的最佳例子幾乎都是多層式架構。 特定電腦作為應用程式的前端或 API 供應端，並且公開至用戶端或網際網路。 其他電腦 (通常有不同的硬體資源) 處理計算層或儲存層的工作。 它們通常_不會_直接公開至用戶端或網際網路。 Service Fabric 認為特定的工作負載有時需要在特定硬體設定上執行。 例如︰

* 現有的多層式架構應用程式已「提升並移轉」到 Service Fabric 環境
* 針對效能、級別或安全性隔離原因而想要在特定硬體上執行的工作負載
* 基於原則或資源耗用量的理由，工作負載應該彼此隔離

為了支援這種設定，Service Fabric 有一流的標籤概念可運用在節點上。 這些標記稱為**節點屬性**。 **條件約束**是陳述式，會附加至針對一或多個節點屬性選取的個別服務。 放置條件約束會定義應該執行服務的位置。 條件約束集可延伸 - 任何索引鍵/值組都沒問題。 

<center>
![叢集配置不同工作負載][Image5]
</center>

### <a name="built-in-node-properties"></a>內建節點屬性
Service Fabric 定義一些預設節點屬性，可自動使用，而不需要由使用者定義。 每個節點上定義的預設屬性是 **NodeType** 和 **NodeName**。 例如，您可以將放置條件約束撰寫成 `"(NodeType == NodeType03)"`。 我們大致上發現 NodeType 是其中一個最常用的屬性。 因為它與機器類型的對應是 1:1，所以很有用。 每個機器類型都對應至傳統多層式架構應用程式中的工作負載類型。

<center>
![放置條件約束和節點屬性][Image6]
</center>

## <a name="placement-constraint-and-node-property-syntax"></a>放置條件約束和節點屬性語法 
節點屬性中指定的值可以是字串、bool，或帶正負號長值。 服務的陳述式稱為放置「條件約束」，因為它會限制服務在叢集中可執行的位置。 條件約束可以是任何在叢集中於不同節點上運作的布林值陳述式。 這些布林值陳述式中的有效選取器如下：

1) 建立特定陳述式的條件式檢查

| 陳述式 | 語法 |
| --- |:---:|
| 「等於」 | "==" |
| 「不等於」 | "!=" |
| 「大於」 | ">" |
| 「大於或等於」 | ">=" |
| 「小於」 | "<" |
| 「小於或等於」 | "<=" |

2) 分組和邏輯運算的布林值陳述式

| 陳述式 | 語法 |
| --- |:---:|
| 「和」 | "&&" |
| 「或」 | "&#124;&#124;" |
| 「否」 | "!" |
| 「組成單一陳述式」 | "()" |

以下是基本條件約束陳述式的一些範例。

  * `"Value >= 5"`
  * `"NodeColor != green"`
  * `"((OneProperty < 100) || ((AnotherProperty == false) && (OneProperty >= 100)))"`

服務只能放置在整體放置條件約束陳述式評估為 "True" 的節點上。 未定義屬性的節點不符合包含該屬性的任何放置條件約束。

假設某個節點類型已定義下列節點屬性︰

ClusterManifest.xml

```xml
    <NodeType Name="NodeType01">
      <PlacementProperties>
        <Property Name="HasSSD" Value="true"/>
        <Property Name="NodeColor" Value="green"/>
        <Property Name="SomeProperty" Value="5"/>
      </PlacementProperties>
    </NodeType>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json。 

> [!NOTE]
> 在您的 Azure Resource Manager 範本中，節點類型通常已參數化。 它看起來會是 "[parameters('vmNodeType1Name')]"，而不是 "NodeType01"。
>

```json
"nodeTypes": [
    {
        "name": "NodeType01",
        "placementProperties": {
            "HasSSD": "true",
            "NodeColor": "green",
            "SomeProperty": "5"
        },
    }
],
```

您可以針對服務建立服務放置「條件約束」，如下所示︰

C#

```csharp
FabricClient fabricClient = new FabricClient();
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
serviceDescription.PlacementConstraints = "(HasSSD == true && SomeProperty >= 4)";
// add other required servicedescription fields
//...
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceType -Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton -PlacementConstraint "HasSSD == true && SomeProperty >= 4"
```

如果 NodeType01 的所有節點都是有效的，您也可以使用條件約束 "(NodeType == NodeType01)" 選取該節點類型。

服務放置條件約束其中很棒的一點是它們可以在執行階段動態更新。 所以如果您需要，可以在叢集中移動服務、加入和移除需求等等。Service Fabric 會負責確保服務保持執行且可用，即使進行這類變更。

C#：

```csharp
StatefulServiceUpdateDescription updateDescription = new StatefulServiceUpdateDescription();
updateDescription.PlacementConstraints = "NodeType == NodeType01";
await fabricClient.ServiceManager.UpdateServiceAsync(new Uri("fabric:/app/service"), updateDescription);
```

PowerShell：

```posh
Update-ServiceFabricService -Stateful -ServiceName $serviceName -PlacementConstraints "NodeType == NodeType01"
```

放置條件約束會針對每個不同的具名服務執行個體指定。 更新一律會取代 (覆寫) 先前指定的項目。

叢集定義會定義節點上的屬性。 變更節點的屬性需要叢集設定升級。 升級節點的屬性需要每個受影響的節點重新啟動，以報告其新的屬性。 這些輪流升級是由 Service Fabric 管理。

## <a name="describing-and-managing-cluster-resources"></a>描述與管理叢集資源
任何 Orchestrator 的其中一個最重要的作業是協助管理叢集中的資源耗用量。 管理叢集資源可以表示幾個不同的項目。 首先，確保電腦不會多載。 這表示確定電腦不會執行比它們能夠處理還多的服務。 第二，平衡和最佳化，對於有效率地執行服務很重要。 符合成本效益或效能的敏感服務供應項目不允許某些節點忙碌，而其他節點閒置。 忙碌節點會導致資源爭用和效能不佳，而閒置節點代表浪費資源和增加成本。 

Service Fabric 以 `Metrics` 表示資源。 度量是您想要向 Service Fabric 描述的任何邏輯或實體資源。 度量的範例是例如「WorkQueueDepth」或「MemoryInMb」的項目。 如需 Service Fabric 可以在節點上管理之實體資源的詳細資訊，請參閱[資源管理](service-fabric-resource-governance.md)。 如需設定自訂計量及其用法的相關資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-metrics.md)

計量不同於放置條件約束和節點屬性。 節點屬性是節點本身的靜態描述項。 計量說明節點所擁有的資源，以及在節點上執行時取用的服務。 節點屬性可能是 "HasSSD"，可設為 true 或 false。 該 SSD 上可用 (和服務所耗用) 的空間數量是像 "DriveSpaceInMb" 之類的計量。 

必須注意，就像放置條件約束和節點屬性一樣，Service Fabric 叢集資源管理員也不了解計量名稱的意義。 計量名稱只是字串。 如果您建立的計量名稱可能引起歧義，最好宣告單位。

## <a name="capacity"></a>Capacity
如果您關閉所有資源「平衡」，Service Fabric 的叢集資源管理員仍會確保節點不超出其容量。 除非叢集已滿或工作負載大於任何節點，否則管理容量溢位是可能的。 容量是另一個「條件約束」，可讓叢集資源管理員了解節點使用某項資源的多寡。 另外也會追蹤整個叢集的剩餘容量。 服務層級的容量和耗用量均以度量來表示。 例如，計量可能是 "ClientConnections"，而某個節點的 "ClientConnections" 容量可能是 32768。 其他節點可以有其他限制。在該節點上執行的某些服務可以說它目前耗用 32256 個計量 "ClientConnections"。

在執行階段期間，叢集資源管理員會追蹤叢集中和節點上的剩餘容量。 為了追蹤容量，叢集資源管理員會從服務執行所在的節點容量減去每個服務的使用量。 利用此資訊，Service Fabric 叢集資源管理員即可決定將複本放置或移至何處，不會讓節點超出容量。

<center>
![叢集節點和容量][Image7]
</center>

C#：

```csharp
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
ServiceLoadMetricDescription metric = new ServiceLoadMetricDescription();
metric.Name = "ClientConnections";
metric.PrimaryDefaultLoad = 1024;
metric.SecondaryDefaultLoad = 0;
metric.Weight = ServiceLoadMetricWeight.High;
serviceDescription.Metrics.Add(metric);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

PowerShell：

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 3 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("ClientConnections,High,1024,0)
```

您可以看到叢集資訊清單中定義的容量︰

ClusterManifest.xml

```xml
    <NodeType Name="NodeType03">
      <Capacities>
        <Capacity Name="ClientConnections" Value="65536"/>
      </Capacities>
    </NodeType>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json。 

```json
"nodeTypes": [
    {
        "name": "NodeType03",
        "capacities": {
            "ClientConnections": "65536",
        }
    }
],
```

服務的負載通常會動態變更。 假設複本的負載 "ClientConnections" 從 1024 變成 2048，但它執行所在的節點只剩下該計量的 512 個容量。 現在，該複本或執行個體的放置已無效，因為該節點上沒有足夠的空間。 叢集資源管理員必須介入讓節點回到容量以下。 它可以透過將一或多個複本或執行個體從該節點移至其他節點，來減少超過容量之節點上的負載。 移動複本時，叢集資源管理員會嘗試以最低成本來進行這些移動。 移動成本在[這篇文章](service-fabric-cluster-resource-manager-movement-cost.md)中說明，叢集資源管理員之重新平衡策略和規則的詳細資訊則在[這裡](service-fabric-cluster-resource-manager-metrics.md)說明。

## <a name="cluster-capacity"></a>叢集容量
那麼 Service Fabric 叢集資源管理員如何保持不讓整體叢集太滿？ 使用動態負載的話，它並沒有太多可以執行的工作。 服務的負載可能突然增加，而無視於叢集資源管理員所採取的動作。 因此，您的叢集今天有充足的空餘空間，明天就可能因為出名而後繼無力。 話雖如此，但有一些現成的控制可防止問題。 我們可以做的第一件事是防止建立新的工作負載，該工作負載會導致叢集空間變滿。

假設您建立無狀態服務，而它有一些與它相關聯的負載。 假設服務很注重 "DiskSpaceInMb" 計量。 另外也假設服務的每個執行個體會耗用五個單位的 "DiskSpaceInMb"。 您想要建立服務的三個執行個體。 太棒了！ 這表示叢集中需要有 15 個單位的 "DiskSpaceInMb"，我們才能建立這些服務執行個體。 叢集資源管理員持續計算容量和每個計量的耗用量，讓它可以判斷叢集中的剩餘容量。 如果沒有足夠的空間，叢集資源管理員會拒絕建立服務呼叫。

由於只是要求有 15 個單位可用，有許多不同方式可配置此空間。 例如，15 個不同節點上可能剩餘一個單位的容量，或五個不同節點上剩餘三個單位的容量。 如果叢集資源管理員能夠重新安排讓三個節點上有五個單位可用，就會放置此服務。 重新安排叢集通常都可行，除非叢集幾乎已滿，或因為某些原因致使現有服務無法合併。

## <a name="buffered-capacity"></a>緩衝處理的容量
緩衝處理的容量是叢集資源管理員的另一項功能。 它可以保留整體節點容量的某些部分。 這個容量緩衝區只用來在升級和節點失敗期間放置服務。 緩衝處理的容量是針對所有節點的每個計量進行全域指定。 您挑選的保留容量值取決於叢集中的容錯網域和升級網域數目。 容錯網域和升級網域越多，表示您可以挑選較小的緩衝容量。 如果您有較多網域，則在升級和失敗期間，無法使用的叢集部分當然會比較少。 指定緩衝處理的容量時，必須同時指定計量的節點容量，這樣才有意義。

以下是指定緩衝處理容量的方式：

ClusterManifest.xml

```xml
        <Section Name="NodeBufferPercentage">
            <Parameter Name="SomeMetric" Value="0.15" />
            <Parameter Name="SomeOtherMetric" Value="0.20" />
        </Section>
```

獨立部署透過 ClusterConfig.json，Azure 託管叢集透過 Template.json：

```json
"fabricSettings": [
  {
    "name": "NodeBufferPercentage",
    "parameters": [
      {
          "name": "SomeMetric",
          "value": "0.15"
      },
      {
          "name": "SomeOtherMetric",
          "value": "0.20"
      }
    ]
  }
]
```

當叢集耗盡計量的緩衝容量時，建立新的服務會失敗。 防止建立新服務以保留緩衝區，可確保升級和失敗不會造成節點超出容量。 緩衝容量是選擇性，但建議用在已定義計量容量的任何叢集。

叢集資源管理員會公開此負載資訊。 對於每個計量，這項資訊包括： 
  - 緩衝處理的容量設定
  - 總容量
  - 目前的耗用量
  - 每個計量是否被視為平衡
  - 標準差的統計資料
  - 具有最大和最小負載的節點  
  
我們可以看到該輸出的範例如下︰

```posh
PS C:\Users\user> Get-ServiceFabricClusterLoadInformation
LastBalancingStartTimeUtc : 9/1/2016 12:54:59 AM
LastBalancingEndTimeUtc   : 9/1/2016 12:54:59 AM
LoadMetricInformation     :
                            LoadMetricName        : Metric1
                            IsBalancedBefore      : False
                            IsBalancedAfter       : False
                            DeviationBefore       : 0.192450089729875
                            DeviationAfter        : 0.192450089729875
                            BalancingThreshold    : 1
                            Action                : NoActionNeeded
                            ActivityThreshold     : 0
                            ClusterCapacity       : 189
                            ClusterLoad           : 45
                            ClusterRemainingCapacity : 144
                            NodeBufferPercentage  : 10
                            ClusterBufferedCapacity : 170
                            ClusterRemainingBufferedCapacity : 125
                            ClusterCapacityViolation : False
                            MinNodeLoadValue      : 0
                            MinNodeLoadNodeId     : 3ea71e8e01f4b0999b121abcbf27d74d
                            MaxNodeLoadValue      : 15
                            MaxNodeLoadNodeId     : 2cc648b6770be1bc9824fa995d5b68b1
```

## <a name="next-steps"></a>後續步驟
* 如需叢集資源管理員內的架構和資訊流程的相關資訊，請參閱[這篇文章](service-fabric-cluster-resource-manager-architecture.md)
* 定義重組度量是合併 (而不是擴增) 節點上負載的一種方式。若要了解如何設定重組，請參閱 [這篇文章](service-fabric-cluster-resource-manager-defragmentation-metrics.md)
* 從頭開始，並 [取得 Service Fabric 叢集資源管理員的簡介](service-fabric-cluster-resource-manager-introduction.md)
* 若要了解叢集資源管理員如何管理並平衡叢集中的負載，請查看關於 [平衡負載](service-fabric-cluster-resource-manager-balancing.md)

[Image1]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-domains.png
[Image2]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-uneven-fault-domain-layout.png
[Image3]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domains-with-placement.png
[Image4]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-fault-and-upgrade-domain-layout-strategies.png
[Image5]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-layout-different-workloads.png
[Image6]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-placement-constraints-node-properties.png
[Image7]:./media/service-fabric-cluster-resource-manager-cluster-description/cluster-nodes-and-capacity.png
