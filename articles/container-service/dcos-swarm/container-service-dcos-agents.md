---
title: Azure Container Service 的 DC/OS 代理程式集區
description: 公用和私用代理程式集區如何與 Azure Container Service DC/OS 叢集搭配運作
services: container-service
author: iainfoulds
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 01/04/2017
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: 17029f51be9fed8fc36c5f919ece84acbf0461d9
ms.sourcegitcommit: a2ae233e20e670e2f9e6b75e83253bd301f5067c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/13/2018
ms.locfileid: "42145720"
---
# <a name="dcos-agent-pools-for-azure-container-service"></a>Azure Container Service 的 DC/OS 代理程式集區
Azure Container Service 中的 DC/OS 叢集包含兩個集區中的代理程式節點，即公用集區和私用集區。 您可以將應用程式部署到其中任一集區，以影響容器服務中電腦之間的存取性。 電腦可以公開至網際網路 (公用) 或保留在內部 (私用)。 本文簡短概述為什麼會有公用集區和私用集區。


* **私用代理程式**：私用代理程式節點會透過非可路由網路來執行。 只有從系統管理區域或透過公用區域邊緣路由器才可存取此網路。 根據預設，DC/OS 會在私用代理程式節點上啟動應用程式。 

* **公用代理程式**：公用代理程式節點會透過可公開存取的網路執行 DC/OS 應用程式和服務。 

如需有關 DC/OS 網路安全性的詳細資訊，請參閱 [DC/OS 文件](https://dcos.io/docs/1.8/administration/securing-your-cluster/)。

## <a name="deploy-agent-pools"></a>部署代理程式集區

Azure Container Service 中的 DC/OS 代理程式集區會以下列方式建立：

* 「私用集區」包含您[部署 DC/OS 叢集](container-service-deployment.md)時所指定數目的代理程式節點。 

* 「公用集區」一開始會包含所預先決定數目的代理程式節點。 佈建 DC/OS 叢集時會自動新增此集區。

私用集區和公用集區是 Azure 虛擬機器擴展集。 您可以在部署後調整這些集區的大小。

## <a name="use-agent-pools"></a>使用代理程式集區
根據預設，**Marathon** 會將任何新的應用程式部署至*私用*代理程式節點。 您必須在建立應用程式時，明確地將應用程式部署到「公用」節點。 選取 [選用] 索引標籤，然後輸入 **slave_public** 做為 [接受的資源角色] 值。 [這裡](container-service-mesos-marathon-ui.md#deploy-a-docker-formatted-container)和 [DC/OS](https://dcos.io/docs/1.7/administration/installing/custom/create-public-agent/) 文件中皆有此程序的記載。

## <a name="next-steps"></a>後續步驟
* 深入了解如何[管理 DC/OS 容器](container-service-mesos-marathon-ui.md)。

* 了解如何[開啟 Azure 所提供的防火牆](container-service-enable-public-access.md)以允許公開存取您的 DC/OS 容器。

