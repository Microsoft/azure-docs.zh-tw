---
title: Azure Marketplace 的容器供應項目發佈指南
description: 本文說明在 Marketplace 中發佈容器的需求
services: Azure, Marketplace, Compute, Storage, Networking, Blockchain, Security
documentationcenter: ''
author: ellacroi
manager: nunoc
editor: ''
ms.assetid: ''
ms.service: marketplace
ms.workload: ''
ms.tgt_pltfrm: ''
ms.devlang: ''
ms.topic: article
ms.date: 07/09/2018
ms.author: ellacroi
ms.openlocfilehash: 5eb30c65032332825d05097f86d0275b015a8929
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39057580"
---
# <a name="containers-offer-publishing-guide"></a>容器供應項目發佈指南

容器供應項目可協助您將容器映像發佈到 Azure Marketplace。 您可以使用本指南來了解這項供應項目的需求。 

這些交易供應項目是透過 Marketplace 來部署並計費的。 使用者會看到的呼籲行動是「立即取得」。

當您的解決方案是佈建成 Kubernetes 型 Azure 容器服務的 Docker 容器映像時，請使用「容器」供應項目類型。

>[!NOTE]
>例如，Azure Kubernetes Service 或「Azure 容器執行個體」之類的 Kubernetes 型 Azure 容器服務、Azure 客戶選擇的 Kubernetes 型容器執行階段。  

Microsoft 目前支援免費和自備授權 (BYOL) 授權模型。

## <a name="containers-offer"></a>容器供應項目

| 需求 | 詳細資料 |  
|:--- |:--- |  
| 計費和計量 | 支援免費或 BYOL 計費模型。 |  
| 從 Dockerfile 建置的映像 | 容器映像必須以 Docker 映像規格為基礎，而必須從 Dockerfile 建置。<ul> <li>如需建置 Docker 映像的詳細資訊，請瀏覽＜使用方式＞一節 (網址為 [docs.docker.com/engine/reference/builder/#usage](https://docs.docker.com/engine/reference/builder/#usage))。</li> </ul> |  
| 裝載於 ACR 中 | 容器映像必須裝載於 Azure Container Registry (ACR) 存放庫中。<ul> <li>如需使用 ACR 的詳細資訊，請瀏覽＜快速入門：使用 Azure 入口網站建立容器登錄＞頁面 (網址為 [docs.microsoft.com/azure/container-registry/container-registry-get-started-portal](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-portal))。</li> </ul> |  
| 映像標記 | 容器映像必須包含至少 1 個標籤 (最多可容納 16 個標籤)。<ul> <li>如需標記映像的詳細資訊，請瀏覽＜Docker 標籤＞頁面 (網址為 [docs.docker.com/engine/reference/commandline/tag](https://docs.docker.com/engine/reference/commandline/tag))。</li> </ul> |  


## <a name="next-steps"></a>後續步驟

如果您還沒有這麼做，請 

- 在 Marketplace 中[註冊](https://azuremarketplace.microsoft.com/sell)

如果您已註冊，且要建立新的供應項目或使用現有的供應項目，請

- [登入 Cloud Partner 入口網站](https://cloudpartner.azure.com)以建立或完成您的供應項目
