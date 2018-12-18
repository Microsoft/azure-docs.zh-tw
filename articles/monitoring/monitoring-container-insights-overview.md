---
title: 適用於容器的 Azure 監視器概觀 | Microsoft Docs
description: 本文說明適用於容器的 Azure 監視器，其會監視 AKS 容器深入解析解決方案，以及它藉由監視您 AKS 叢集和 Azure 中容器執行個體的健康情況來提供的值。
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/14/2018
ms.author: magoedte
ms.openlocfilehash: 6819cf96eb968e8faad5441cf3f3dd672653f1cd
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46971428"
---
# <a name="azure-monitor-for-containers-overview"></a>適用於容器的 Azure 監視器概觀

適用於容器的 Azure 監視器是一項功能，其設計訴求是要監視部署至 Azure Kubernetes Service (AKS) 上所裝載受控 Kubernetes 叢集的容器工作負載效能。 監視容器很重要，尤其在您使用多個應用程式大規模執行生產環境叢集時。

適用於容器的 Azure 監視器可藉由透過計量 API 從 Kubernetes 中取得的控制器、節點與容器來收集記憶體與處理器計量，為您提供效能可見度。 容器記錄也會一併收集。  當您從 Kubernetes 叢集啟用監視之後，系統就會透過適用於 Linux 的 Log Analytics 代理程式容器化版本自動收集這些計量和記錄，並儲存在您的 [Log Analytics](../log-analytics/log-analytics-overview.md) 工作區中。 
 
## <a name="what-does-azure-monitor-for-containers-provide"></a>適用於容器的 Azure 監視器提供哪些功能？

適用於容器的 Azure 監視器包含數個預先定義的檢視，這些檢視會顯示所在容器的工作負載，以及影響受監視 Kubernetes 叢集效能健康情況的項目，使您可以：  

* 識別正在節點上執行的 AKS 容器，以及其平均的處理器與記憶體使用率。 此知識可協助您識別資源瓶頸。
* 識別容器在控制器或 Pod 中的所在位置。 此知識可協助您檢視控制器或 Pod 的整體效能。 
* 檢閱在和支援 Pod 的標準程序無關之主機上執行的工作負載的資源使用率。
* 了解叢集在平均負載和最高負載之下的行為。 此知識可協助您識別所需的容量，並判斷叢集可承受的負載上限。 

## <a name="how-do-i-access-this-feature"></a>如何存取此功能？
您有兩種方式可用來存取適用於容器的 Azure 監視器：從 Azure 監視器或直接從所選取的 AKS 叢集。 從 Azure 監視器，您會看見一個包含所有已部署容器 (受監視與不受監視的容器) 的全域檢視方塊，讓您能夠跨訂用帳戶和資源群組進行搜尋和篩選，然後從所選取的容器向下切入到適用於容器的 Azure 監視器。  否則，您只能直接從 AKS 頁面的所選取 AKS 容器中存取此功能。  

![存取適用於容器的 Azure 監視器方法概觀](./media/monitoring-container-insights-overview/azmon-containers-views.png)

如果您想要監視和管理 Docker 與 Windows 容器主機以檢視設定、稽核以及資源使用率，請參閱[容器監視解決方案](../log-analytics/log-analytics-containers.md)。

## <a name="next-steps"></a>後續步驟
若要開始監視您的 AKS 叢集，請檢閱[如何將適用於容器的 Azure 監視器上線](monitoring-container-insights-onboard.md)，以了解啟用監視的需求和可用方法。  