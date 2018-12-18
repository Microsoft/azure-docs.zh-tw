---
title: Service Fabric 叢集 Resource Manager - 同質性 | Microsoft Docs
description: Service Fabric 服務的設定同質性概觀
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: 678073e1-d08d-46c4-a811-826e70aba6c4
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 57abea79a620aa83e16ad4cc2fd78a4294f2b278
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34204850"
---
# <a name="configuring-and-using-service-affinity-in-service-fabric"></a>在 Service Fabric 中設定並使用服務同質性
同質性是一個控制項，主要提供來協助簡化將較大型的單體式應用程式轉換到雲端和微服務世界的程序。 同質性也可作為最佳化手段來提升服務的效能，不過，這麼做會帶來一些副作用。

假設您正在將較大型的應用程式 (或是在設計時並未考慮到微服務的應用程式) 引進到 Service Fabric (或任何分散式環境)。 這種轉換很常見。 首先要將整個應用程式提升至環境中，封裝應用程式，並確定可以順暢執行。 接著，開始將其分解成不同的較小服務，使其可彼此通訊。

最終，您可能會發現應用程式遇到一些問題。 這些問題通常屬於下列其中一個類別︰

1. 單體式應用程式中的某個元件 X 對元件 Y 具有不明的相依性，但您將這些元件變成個別的服務。 因為現在這些服務是在叢集中不同的節點上執行，它們已中斷。
2. 這些元件透過 (本機具名管道 | 共用記憶體 | 磁碟上的檔案) 來通訊，但基於效能理由，的確需要能夠立即寫入到共用的本機資源。 您稍後或許會移除該硬式相依性。
3. 一切都沒問題，但事實上，這兩個元件具有很多對話/效能敏感的內容。 當我將它們移到個別服務時，整體應用程式效能會遭到重挫或導致延遲增加。 如此一來，整體應用程式就不會符合預期。

在這些情況中，我們既不想失去重構工作，也不想要回到單體模式。 我們甚至可能會希望是最後一個情況，因為要最佳化很簡單。 不過，在我們能夠將元件重新設計的像服務一樣自然運作之前 (或能夠以其他某些方式解決效能期望之前)，我們需要有位置方面的概念。

怎麼辦？ 嗯，您可以嘗試開啟同質性。

## <a name="how-to-configure-affinity"></a>如何設定同質性
若要設定同質性，您可以定義兩個不同服務之間的同質關聯性。 您可以將此同質性想像成在一個服務上「指向」另一個服務，並假設「此服務只有在該服務執行所在的地方才能執行」。 有時我們會將這些同質性稱為父子式關聯性 (您會在父系上指向子系)。 同質性可確保某一個服務的複本或執行個體是放置在與另一個服務的複本或執行個體相同的節點上。

```csharp
ServiceCorrelationDescription affinityDescription = new ServiceCorrelationDescription();
affinityDescription.Scheme = ServiceCorrelationScheme.Affinity;
affinityDescription.ServiceName = new Uri("fabric:/otherApplication/parentService");
serviceDescription.Correlations.Add(affinityDescription);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

> [!NOTE]
> 子服務只能參與一個同質關聯性。 如果您想讓子服務一次與兩個父服務產生同質關聯性，有幾個選項可供選擇：
> - 讓關聯性反轉 (讓 parentService1 和 parentService2 指向目前的子服務)，或是
> - 依照慣例將其中一個父服務指定為中樞，並讓所有服務指向該服務。 
>
> 叢集中所產生的放置行為應該會相同。
>

## <a name="different-affinity-options"></a>各種同質性選項
同質性是透過數種相互關聯的結構描述之一來表示，而且有兩種不同的模式。 同質性的最常見模式是我們稱為 NonAlignedAffinity 的模式。 在 NonAlignedAffinity 中，不同服務的複本或執行個體會放在同一個節點上。 另一個模式是 AlignedAffinity。 對齊的同質性只有在與具狀態服務搭配使用時才有用。 設定兩個具狀態服務來具備對齊的同質性，可以確保那些服務的主要複本都位於和彼此相同的節點上。 它也會讓那些服務的每一對次要複本位於相同的節點上。 它也可能 (但較不常見) 針對具狀態服務設定 NonAlignedAffinity。 對於 NonAlignedAffinity，兩個具狀態服務的不同複本會在相同節點上執行，但它們的主要複本可以在不同節點上結束。

<center>
![同質性模式及其效果][Image1]
</center>

### <a name="best-effort-desired-state"></a>盡力而為的期望狀態
同質關聯性是盡力而為模式。 它所提供的共置或可靠性保證不會和在同一個可執行程序中執行所提供的保證相同。 具有同質關聯性的服務是本質上不同的實體，可能會失敗以及被個別移動。 同質關聯性也可能會中斷，不過這些中斷只是暫時的。 例如，容量限制可能表示同質關聯性中只有部分服務物件可以適用於指定節點。 在這些情況下，即使有同質關聯性，也會因為其他條件約束而無法強制執行。 就算可行，系統之後也會自動更正此違規情形。

### <a name="chains-vs-stars"></a>鏈結與星形的比較
目前，叢集資源管理員無法模擬同質關聯性的鏈結。 這表示是，如果有一個服務是某一個同質關聯性中的子系，則該服務不能是另一個同質關聯性中的父系。 如果您想要建立此類型的關聯性模型，您實際上必須將它建立為星形模型，而不是鏈結模型。 為了從鏈結轉變為星形，最下面的子系會變成第一個子系的父系。 根據服務的排列方式而定，您可能必須這樣做許多次。 如果沒有自然父系服務，您可能必須建立一個作為預留位置。 根據您的需求而定，您也可能考慮[應用程式群組](service-fabric-cluster-resource-manager-application-groups.md)。

<center>
![同質關聯性內容中的鏈結與星形的比較][Image2]
</center>

目前關於同質關聯性的另一個要注意事項是它們是雙向的。 這表示同質性規則只會強制讓子系跟隨父系來放置。 它無法確保父系會與子系位於相同位置。 也請務必注意，同質關聯性不可能完美無缺或立即強制執行，因為不同的服務會有不同的生命週期，因此可以單獨地失敗和移動。 例如，假設父系因為當機而突然容錯移轉至另一個節點。 叢集資源管理員和容錯移轉管理員會先處理容錯移轉，因為其首要任務就是讓服務保持運作、一致性和可用性。 在容錯移轉完成之後，同質關聯性已中斷，但叢集資源管理員會認為一切正常，直到它發現子系並未與父系放在一起。 系統會定期執行這類檢查。 您可以在[這篇文章](service-fabric-cluster-resource-manager-management-integration.md#constraint-types)中找到更多關於叢集資源管理員如何評估條件約束的資訊，[這篇文章](service-fabric-cluster-resource-manager-balancing.md)則會深入說明如何設定這些條件約束的評估頻率。   


### <a name="partitioning-support"></a>支援分割
請注意，關於同質性要注意的最後一點是，分割父系的地方不支援同質關聯性。 我們最終可能會支援分割的父服務，但目前尚未允許這麼做。

## <a name="next-steps"></a>後續步驟
- 如需服務設定的詳細資訊，請[深入了解設定服務](service-fabric-cluster-resource-manager-configure-services.md)
- 若要限制只讓一小組機器使用服務或要彙總服務的負載，請使用[應用程式群組](service-fabric-cluster-resource-manager-application-groups.md)

[Image1]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resrouce-manager-affinity-modes.png
[Image2]:./media/service-fabric-cluster-resource-manager-advanced-placement-rules-affinity/cluster-resource-manager-chains-vs-stars.png