---
title: 在 Windows 中針對 Azure Service Fabric 應用程式進行偵錯 | Microsoft Docs
description: 了解如何監視和診斷在本機開發電腦上使用 Microsoft Azure Service Fabric 所撰寫的服務。
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''
ms.assetid: edcc0631-ed2d-45a3-851d-2c4fa0f4a326
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/15/2017
ms.author: dekapur
ms.openlocfilehash: b4c3bc21591e8472dc8d51309f7431cb5d4421fd
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44054163"
---
# <a name="monitor-and-diagnose-services-in-a-local-machine-development-setup"></a>監視和診斷本機開發設定中的服務
> [!div class="op_single_selector"]
> * [Windows](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)
> * [Linux](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md)
> 
> 

監視、偵測、診斷和疑難排解可讓服務繼續順利執行，盡可能減少服務中斷的使用者經驗。 雖然監視和診斷在實際部署的生產環境中相當重要，但是效率會取決於在服務開發期間採用相似的模型，以確保它們能夠在您移至實際設定時正常運作。 Service Fabric 可讓服務開發人員輕鬆實作診斷，可以在單一電腦本機開發設定和實際生產叢集設定上順暢地工作。

## <a name="event-tracing-for-windows"></a>Windows 的事件追蹤
[Windows 事件追蹤](https://msdn.microsoft.com/library/windows/desktop/bb968803.aspx) (ETW) 是我們建議的技術，可用於追蹤 Service Fabric 中的訊息。 使用 ETW 的部分優點為：

* **ETW 相當快速。** 其是以一種追蹤技術建置而成，並對您程式碼執行時間的影響降到最低。
* **ETW 會在本機開發環境以及實際叢集設定順暢地進行追蹤。** 這表示當您準備好將程式碼部署至實際叢集時，您不需要重寫追蹤程式碼。
* **Service Fabric 系統程式碼也會將 ETW 用於內部追蹤。** 這可讓您檢視與 Service Fabric 系統追蹤交錯的應用程式追蹤。 同時協助您更輕鬆了解在基礎系統中應用程式程式碼與事件之間的序列和相互關係。
* **內建支援的 Service Fabric Visual Studio 工具可供您檢視 ETW 事件。** 一旦使用 Service Fabric 正確設定 Visual Studio 之後，ETW 事件就會出現在 Visual Studio 的 [診斷事件] 檢視中。 

## <a name="view-service-fabric-system-events-in-visual-studio"></a>在 Visual Studio 中檢視 Service Fabric 系統事件
Service Fabric 會發出 ETW 事件，以協助應用程式開發人員了解平台中發生的事情。 如果您還沒有這麼做，請繼續遵循 [在 Visual Studio 中建立第一個應用程式](service-fabric-create-your-first-application-in-visual-studio.md)中的步驟。 此資訊將協助您啟動應用程式，並執行可顯示追蹤訊息的 [診斷事件檢視器]。

1. 若 [診斷事件] 視窗不會自動顯示，請移至 Visual Studio 中的 [檢視] 索引標籤，選擇 [其他視窗]，然後選擇 [診斷事件檢視器]。
2. 每個事件皆有標準的中繼資料資訊，可告訴您事件所來自的節點、應用程式和服務。 您也可以使用事件視窗頂端的 [篩選事件] 方塊來篩選事件清單。 例如，您可以依 [節點名稱] 或 [服務名稱] 進行篩選。 而當您查看事件詳細資料時，您也可以使用事件視窗頂端的 [暫停] 按鈕來暫停並於稍後繼續，而不會遺失任何事件。
   
   ![Visual Studio 診斷事件檢視器](./media/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally/DiagEventsExamples2.png)

## <a name="add-your-own-custom-traces-to-the-application-code"></a>將您自己自訂的追蹤新增至應用程式程式碼
Service Fabric Visual Studio 專案範本包含範例程式碼。 程式碼示範如何新增自訂的應用程式程式碼 ETW 追蹤，其會與來自 Service Fabric 的系統追蹤一併顯示在 Visual Studio ETW 檢視器中。 這個方法的優點是中繼資料會自動新增至追蹤，且 Visual Studio 診斷事件檢視器已設定為顯示追蹤。

針對從「服務範本」(無狀態或可設定狀態) 建立的專案，只要搜尋 `RunAsync` 實作即可：

1. 使用 `ServiceEventSource.Current.ServiceMessage` in the `RunAsync` 時會顯示來自應用程式程式碼的自訂 ETW 追蹤範例。
2. 在 **ServiceEventSource.cs** 檔案中，您會找到 `ServiceEventSource.ServiceMessage` 方法的多載，出於效能因素，應該將其用於高頻率事件。

針對從 **動作項目範本** (無狀態或可設定狀態) 所建立專案：

1. 開啟 **"ProjectName".cs** 檔案，其中 *ProjectName* 是您針對 Visual Studio 專案所選擇的名稱。  
2. 在 *DoWorkAsync* 方法中找出程式碼 `ActorEventSource.Current.ActorMessage(this, "Doing Work");`。  這是來自應用程式程式碼撰寫的自訂 ETW 追蹤範例。  
3. 在 **ActorEventSource.cs** 中，您會找到 `ActorEventSource.ActorMessage` 方法的多載，出於效能因素，應該將其用於高頻率事件。

將自訂 ETW 追蹤新增至服務程式碼之後，您可以再次建置、部署，以及執行應用程式以查看診斷事件檢視器中的事件。 如果您使用 **F5**來偵錯應用程式，診斷事件檢視器將會自動開啟。

## <a name="next-steps"></a>後續步驟
在 Azure 叢集上執行應用程式時，您針對本機診斷在上方應用程式所新增的相同追蹤程式碼，將會使用可以用來檢視這些事件的工具。 請參閱下列文章，其中討論各種適用於工具的選項，並說明如何設定它們。

* [如何利用 Azure 診斷收集記錄檔](service-fabric-diagnostics-how-to-setup-wad.md)
* [使用 EventFlow 的事件彙總與集合](service-fabric-diagnostics-event-aggregation-eventflow.md)

