---
title: 在 Azure Service Fabric 中重新設定 | Microsoft Docs
description: 了解在 Service Fabric 中重新設定分割區
services: service-fabric
documentationcenter: .net
author: appi101
manager: anuragg
editor: ''
ms.assetid: d5ab75ff-98b9-4573-a2e5-7f5ab288157a
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 01/10/2018
ms.author: aprameyr
ms.openlocfilehash: a24aa6aa1695a3d1166816b7960bdd7b551e1a37
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34212801"
---
# <a name="reconfiguration-in-azure-service-fabric"></a>在 Azure Service Fabric 中重新設定
對於具狀態服務的分割區而言，*設定*定義為複本及其角色。

「重新設定」是指將某個設定移至另一個設定的流程。 這會變更具狀態服務其分割區的複本集。 舊設定稱做「先前設定」(簡稱 PC)，新設定稱做「目前設定」(簡稱 CC)。 Azure Service Fabric 中的重新設定通訊協定在複本集進行任何變更期間，會保持一致性並維持可用性。

容錯移轉管理員會起始重新設定，以回應系統中不同的事件。 例如，如果主要複本失敗，便會起始重新設定，將使用中的次要複本升階為主要複本。 另一個範例是在必須將主要複本移至另一個節點，以升級該節點時，以重新設定回應應用程式升級。

## <a name="reconfiguration-types"></a>重新設定類型
重新設定可分成兩種類型：

- 主要複本變更所在的重新設定：
    - **容錯移轉**：容錯移轉是為了回應執行中主要複本失敗的重新設定。
    - **SwapPrimary**：在交換這種重新設定中，Service Fabric 通常必須將執行中的主要複本從某個節點移到另一節點，以回應負載平衡或升級。

- 主要複本不會變更的重新設定。

## <a name="reconfiguration-phases"></a>重新設定階段
重新設定有數個階段：

- **階段 0**：發生於交換主要複本重新設定中，在此階段，目前主要複本會將其狀態轉移到新的主要複本，並轉換成使用中次要複本。

- **階段 1**：發生於重新設定期間，在此階段，主要複本正在變更中。 此階段進行期間，Service Fabric 會從目前複本中找出正確的主要複本。 在交換主要複本重新設定期間不需要此階段，因為新的主要複本已經選定。 

- **階段 2**：在此階段進行期間，Service Fabric 會確定目前設定其大部分複本中的所有資料皆可使用。

還有其他數個階段僅供內部使用。

## <a name="stuck-reconfigurations"></a>重新設定停滯
重新設定可能會因各種原因而「停滯」。 一些常見原因包括：

- **複本停擺**：某些重新設定階段需要設定中大部分的複本為啟動的狀態。
- **網路或通訊問題**：重新設定需要不同節點之間有網路連線。
- **API 失敗**：重新設定通訊協定需要服務實作完成某些 API。 例如：若不接受取消可靠服務中的權杖，會導致 SwapPrimary 重新設定停滯。

如 System.FM、System.RA、System.RAP 等系統元件的健康情況報告可用於診斷重新設定在哪裡停滯。 [系統健康情況報告頁面](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)會說明這些健康情況報告。

## <a name="next-steps"></a>後續步驟
如需有關 Service Fabric 概念的詳細資訊，請參閱下列文章：

- [Reliable Services 生命週期 - C#](service-fabric-reliable-services-lifecycle.md)
- [系統健康情況報告](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)
- [複本和執行個體](service-fabric-concepts-replica-lifecycle.md)
