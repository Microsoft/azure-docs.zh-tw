---
title: Testability：服務通訊 | Microsoft Docs
description: 服務之間的通訊是整合 Service Fabric 應用程式的重要環節。 本文討論設計考量及測試技巧。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''
ms.assetid: 017557df-fb59-4e4a-a65d-2732f29255b8
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/02/2017
ms.author: vturecek
ms.openlocfilehash: e3ea537d310d49c934cf6789184f090791cf16a4
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34211220"
---
# <a name="service-fabric-testability-scenarios-service-communication"></a>Service Fabric Testability 案例：服務通訊
微服務及服務導向的架構樣式會在 Azure Service Fabric 中自然出現。 在這些類型的分散式架構中，元件化的微服務應用程式通常是由需要彼此通訊的多個服務所組成。 即使在最簡單的情況下，您通常至少會有一個無狀態網路服務及一個可設定狀態的資料儲存服務需要相互通訊。

服務之間的通訊是整合應用程式的重要環節，因為各服務會向其他服務公開遠端 API。 與 I/O 相關的一組 API 界限通常需要謹慎處理，且需經過大量測試和驗證。

若要在分散式系統中，將這些服務界限連接起來，有許多方面都需要謹慎考量：

* 傳輸通訊協定。 您要使用 HTTP 以提供更優異的互通性，還是自訂的二進位通訊協定，以應付最大的輸送量？
* 錯誤處理。 如何處理永久性和暫時性錯誤？ 當服務移至另一個節點時，會發生什麼事？
* 逾時與延遲。 在多層式應用程式中，各服務層將如何透過堆疊處理延遲，為使用者順暢提供服務？

不論您使用 Service Fabric 提供的其中一種內建服務通訊元件，或者您選擇自行建立，測試服務之間的互動永遠是確保應用程式復原能力的重要部分。

## <a name="prepare-for-services-to-move"></a>準備讓服務移動
服務執行個體經過一段時間後可能會移動。 尤其在它們設有負載度量，以自訂最佳資源平衡時更是如此。 即使在升級、容錯移轉、向外延展及其他分散式系統存留時間內發生的各種情況期間，Service Fabric 都可以移動您的服務執行個體以最大化其可用性。

由於服務會在叢集中移動，您的用戶端和其他服務應在與服務通訊時準備就緒，以因應兩種案例：

* 在您最後一次與服務執行個體或分割區複本通訊後，其中一項即已移動過。 正常情況下，這是服務生命週期的一部分，而且應該會在應用程式的存留期間發生。
* 服務執行個體或分割區複本正在移動。 在 Service Fabric 中，雖然服務在節點間容錯移轉的速度相當快速，但若服務通訊元件太慢而無法啟動，可能會對可用性造成延遲。

妥善處理這些案例，對於系統流暢運作至關重要。 若要這樣做，請注意下列事項：

* 可連接的各項服務都有「位址」  可接聽 (例如 HTTP 或 WebSockets)。 當服務執行個體或分割移動時，其位址端點會變更。 (它會移至具有不同 IP 位址的不同節點。)如果使用內建的通訊元件，這些元件會為您處理重新解析服務位址。
* 當服務執行個體再次開啟接聽程式時，服務延遲時間可能會短暫增加。 這需取決於服務在服務執行個體移動後開啟接聽程式的速度。
* 必須先關閉任何現有的連線，然後等服務於新的節點上開啟之後再重新開啟。 妥善關閉節點或重新開啟，可讓現有連線擁有足夠時間正常關閉。

### <a name="test-it-move-service-instances"></a>測試：移動服務執行個體
藉由使用 Service Fabric 的 Testability 工具，您可以撰寫測試案例，以不同方式測試這些情況：

1. 移動具狀態服務的主要複本。
   
    具狀態服務分割區的主要複本會因為諸多原因而移動。 以此鎖定特定分割區的主要複本，以便透過高度受控制的方式，查看服務回應移動的情形。
   
    ```powershell
   
    PS > Move-ServiceFabricPrimaryReplica -PartitionId 6faa4ffa-521a-44e9-8351-dfca0f7e0466 -ServiceName fabric:/MyApplication/MyService
   
    ```
2. 停止節點。
   
    節點停止後，Service Fabric 會將該節點上所有服務執行個體或分割區，移動至叢集中其他可用的其中一個節點。 以此測試節點從叢集中遺失，且該節點上的所有服務執行個體及複本都必須移動的情況。
   
    您可以使用 PowerShell **Stop-ServiceFabricNode** Cmdlet 來停止節點：
   
    ```powershell
   
    PS > Stop-ServiceFabricNode -NodeName Node_1
   
    ```

## <a name="maintain-service-availability"></a>維護服務可用性
做為平台，Service Fabric 專為服務提供高可用性。 但是在極端情況下，基礎結構的基礎問題仍可能導致無法使用服務。 請務必也測試這些情況。

具狀態服務會使用仲裁式系統來複寫狀態，藉以達到高可用性。 換句話說，必須要能使用複本仲裁，才能執行寫入作業。 在極罕見情況下，例如大規模的硬體故障，有可能無法使用複本仲裁。 在這些情況下，您將無法執行寫入作業，但仍能執行讀取作業。

### <a name="test-it-write-operation-unavailability"></a>測試：撰寫作業無法使用
藉由使用 Service Fabric 中的 testability 工具，您可以插入引發仲裁遺失的錯誤做為測試。 雖然這樣的案例極為罕見，但仰賴具狀態狀態服務的用戶端和服務務必做好準備，以處理無法要求執行寫入作業的各種情況。 具狀態服務本身也應了解發生這種情況的可能性，並能依正常程序與呼叫者通訊。

您可以藉由使用 **Invoke-ServiceFabricPartitionQuorumLoss PowerShell** Cmdlet 引發仲裁遺失：

```powershell

PS > Invoke-ServiceFabricPartitionQuorumLoss -ServiceName fabric:/Myapplication/MyService -QuorumLossMode QuorumReplicas -QuorumLossDurationInSeconds 20

```

在此範例中，我們將 `QuorumLossMode` 設為 `QuorumReplicas`，以便在引發仲裁遺失時，不會關閉所有複本。 如此才能正常執行讀取作業。 若要測試整個分割區都無法使用的案例，您可將此參數設定為 `AllReplicas`。

## <a name="next-steps"></a>後續步驟
[深入了解 Testability 動作](service-fabric-testability-actions.md)

[深入了解 Testability 案例](service-fabric-testability-scenarios.md)

