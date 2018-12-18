---
title: 什麼是 Azure 雲端服務 | Microsoft Docs
description: 了解什麼是 Azure 雲端服務。
services: cloud-services
documentationcenter: ''
author: jpconnock
manager: timlt
ms.assetid: ed7ad348-6018-41bb-a27d-523accd90305
ms.service: multiple
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/19/2017
ms.author: jeconnoc
ms.openlocfilehash: 6af4533a4ab374071b904cc4b03ca239a3e83364
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39011656"
---
# <a name="overview-of-azure-cloud-services"></a>Azure 雲端服務概觀
Azure 雲端服務是[平台即服務](https://azure.microsoft.com/overview/what-is-paas/) (PaaS) 的一個範例。 這項技術如同 [Azure App Service](../app-service/app-service-web-overview.md)，是專為支援可調整、穩定可靠且操作成本低的應用程式而設計。 如同 App Service 是裝載於虛擬機器 (VM) 上，Azure 雲端服務也是如此。 不過，您可更充分地掌控 VM。 您可以使用 Azure 雲端服務在 VM 上安裝您自己的軟體，並從遠端加以存取。

![Azure 雲端服務圖表](./media/cloud-services-choose-me/diagram.png)

更充分的控制也意味著較低的易用性。 除非您需要額外控制選項，否則與 Azure 雲端服務相較之下，在 App Service 的 Web Apps 功能中通常可更快、更輕易地讓 Web 應用程式上線運作。

Azure 雲端服務角色分為兩種。 這兩者唯一的差別是您的角色在 VM 上裝載的方式：

* **Web 角色**：透過 IIS 自動部署和裝載您的應用程式。

* **背景工作角色**：不使用 IIS，獨立執行您的應用程式。

例如，簡單的應用程式可能只使用單一 Web 角色提供網站服務。 較複雜的應用程式可能使用 Web 角色處理使用者的傳入要求，然後將這些要求傳送給背景工作角色進行處理。 (此通訊可能會使用 [Azure 服務匯流排](../service-bus-messaging/service-bus-fundamentals-hybrid-solutions.md)或 [Azure 佇列儲存體](../storage/common/storage-introduction.md)。)

如上圖所示，單一應用程式中所有的 VM 會在同一個雲端服務中執行。 使用者可以透過單一公用 IP 位址存取應用程式，並且可在應用程式的 VM 之間自動進行要求的負載平衡。 該平台會在 Azure 雲端服務應用程式中[調整和部署](cloud-services-how-to-scale-portal.md) VM，藉此避免發生單一硬體失敗點。

即使應用程式在虛擬機器中執行，也必須了解 Azure 雲端服務是提供 PaaS，而非 IaaS。 以下有一個用來判別的方法。 若使用像是 Azure 虛擬機器等 IaaS，您會先建立及設定應用程式執行的環境。 然後，將應用程式部署至此環境。 您負責管理大部分的環境，處理在各個 VM 中部署作業系統新修補版本等作業。 相反地，在 PaaS 中，環境似乎已經存在。 您只需要部署您的應用程式。 平台的管理會為您處理，包括部署作業系統的新版本。

## <a name="scaling-and-management"></a>調整和管理
藉由 Azure 雲端服務，您不需要建立虛擬機器。 您只需要提供組態檔，讓 Azure 知道您需要多少個執行個體，例如「3 個 Web 角色執行個體」和「2 個背景工作角色執行個體」。 接著，平台會為您建立這些項目。 您仍然可以選擇這些支援 VM 的 [大小](cloud-services-sizes-specs.md) ，但您不需自行建立這些 VM。 如果應用程式需要處理較大的負載，您可以要求更多的 VM，Azure 將建立這些執行個體。 如果負載減少，您可以關閉這些執行個體並停止付費。

Azure 雲端服務應用程式一般透過兩個步驟的程序提供給使用者使用。 開發人員會先將 [應用程式上傳](cloud-services-how-to-create-deploy-portal.md) 到平台的預備區域。 開發人員準備啟動應用程式時，會使用 Azure 入口網站來交換預備與生產環境。 此 [預備與生產之間的切換](cloud-services-how-to-manage-portal.md#swap-deployments-to-promote-a-staged-deployment-to-production) 程序完全不會造成停機，因此執行中的應用程式得以在不干擾使用者的情況下升級至新版。

## <a name="monitoring"></a>監視
Azure 雲端服務也提供監視。 和虛擬機器一樣，它會偵測故障的實體伺服器，並且在新機器上重新啟動原先在該伺服器上執行的 VM。 不過，Azure 雲端服務也會偵測故障的 VM 和應用程式，而不只是硬體故障。 和虛擬機器不同的是，它在各個 Web 角色和背景工作角色中都有代理程式，因此能夠在故障時啟動新的 VM 和應用程式執行個體。

Azure 雲端服務的 PaaS 性質也有其他意涵。 其中一個最重要的意涵是，採用這項技術建立的應用程式應該在任何 Web 角色或背景工作角色執行個體故障時都能正常運作。 為了實現這一點，Azure 雲端服務應用程式不應該在本身 VM 的檔案系統中保持狀態。 不同於透過虛擬機器建立的 VM，對 Azure 雲端服務 VM 的寫入不會持續存在。 沒有類似虛擬機器資料磁碟的項目。 Azure 雲端服務應用程式反而應該將所有狀態明確寫入 Azure SQL 資料庫、Blob、表格或其他一些外部儲存體。 以這種方式建立應用程式使得調整更簡單，而且更能夠因應故障，這是 Azure 雲端服務的兩個重要目標。

## <a name="next-steps"></a>後續步驟
* [在 .NET 中建立雲端服務應用程式](cloud-services-dotnet-get-started.md) 
* [在 Node.js 中建立雲端服務應用程式](cloud-services-nodejs-develop-deploy-app.md) 
* [在 PHP 中建立雲端服務應用程式](../cloud-services-php-create-web-role.md) 
* [在 Python 中建立雲端服務應用程式](cloud-services-python-ptvs.md)



