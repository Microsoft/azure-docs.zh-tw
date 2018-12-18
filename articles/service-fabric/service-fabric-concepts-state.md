---
title: 管理 Azure Service Fabric 服務中的狀態 | Microsoft Docs
description: 了解如何定義和管理 Service Fabric 服務中的服務狀態。
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: ''
ms.assetid: f5e618a5-3ea3-4404-94af-122278f91652
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 48345be959bb9bebf7c30fa71de91b7881863d66
ms.sourcegitcommit: ebd06cee3e78674ba9e6764ddc889fc5948060c4
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/07/2018
ms.locfileid: "44054952"
---
# <a name="service-state"></a>服務狀態
「服務狀態」係指服務正常運作所需的記憶體或磁碟資料。 例如，其包括可以讓服務讀取及寫入，以執行工作的資料結構及成員變數。 根據服務的架構方式而定，它可能還包括儲存在磁碟上的檔案或其他資源。 例如，資料庫要用於儲存資料和交易記錄的檔案。

作為範例服務，讓我們考慮計算機。 基本計算機服務會採用兩個數字並傳回其總和。 執行這項計算不會牽涉任何成員變數或其他資訊。

現在，假設相同的計算機，但包含其他方法可以儲存並傳回上一個計算總和。 此服務現在為具狀態。 具狀態表示其中包含當其計算新的總和時，某些要寫入的狀態，和當您要求其傳回最後一個已計算總和時，要讀取的狀態。

在 Azure Service Fabric 中，第一個服務稱為無狀態服務。 第二個服務稱為可設定狀態的服務。

## <a name="storing-service-state"></a>儲存服務狀態
狀態可進行外部化，也可並存在運作該狀態的程式碼中。 狀態外部化的做法通常是使用外部資料庫，或是透過網路在不同電腦上或在相同電腦上的流程以外執行的其他資料存放區。 在計算機範例中，資料存放區可能是 SQL Database 或 Azure 資料表存放區的執行個體。 計算總和的每個要求都會執行此資料的更新，且要求服務傳回值會造成從存放區擷取目前的值。 

狀態也可以使用操作狀態的程式碼進行共置。 Service Fabric 中的具狀態服務通常是使用此模型來建置。 Service Fabric 提供的基礎結構可確保此狀態是高度可用、一致且持久，且使用這種方式建置的服務可以輕鬆地加以縮放。

## <a name="next-steps"></a>後續步驟
如需有關 Service Fabric 概念的詳細資訊，請參閱下列文章：

* [Service Fabric 服務的可用性](service-fabric-availability-services.md)
* [Service Fabric 服務的延展性](service-fabric-concepts-scalability.md)
* [分割 Service Fabric 服務](service-fabric-concepts-partitioning.md)
* [Service Fabric Reliable Services](service-fabric-reliable-services-introduction.md)
