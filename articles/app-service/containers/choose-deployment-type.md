---
title: Linux 部署上的 Azure App Service - 自訂映像、多容器或內建平台映像？  | Microsoft Docs
description: 如何在自訂 Docker 容器部署、多容器與 Linux 上的 App Service 內建應用程式架構之間做出選擇
keywords: azure app service, web 應用程式, linux, oss
services: app-service
documentationCenter: ''
authors: msangapu
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/04/2018
ms.author: msangapu
ms.openlocfilehash: c619ae164f8f8b6e94d9061c4346de58bd6cb795
ms.sourcegitcommit: 74941e0d60dbfd5ab44395e1867b2171c4944dbe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/15/2018
ms.locfileid: "49319433"
---
# <a name="custom-image-multi-container-or-built-in-platform-image"></a>自訂映像、多容器或內建平台映像？

[Linux 上的 App Service](app-service-linux-intro.md) 提供三種不同的路徑將應用程式發佈到網路上：

- **自訂映像部署**：將您的應用程式 Docker 化成為 Docker 映像，您所有的檔案和相依性都包含在執行就緒的封裝之中。
- **多容器部署**：使用 Docker Compose 或 Kubernetes 組態檔在多個容器中將您的應用程式「Docker 化」。 如需詳細資訊，請參閱[多容器應用程式](#multi-container-apps-supportability)。
- **使用內建平台映像的應用程式部署**：我們的內建平台映像包含常用的 Web 應用程式執行階段和相依性，例如節點和 PHP。 使用任何一種 [Azure App Service 部署方法](../app-service-deploy-local-git.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)，將您的應用程式部署至 Web 應用程式的存放區，然後使用內建的平台映像執行應用程式。

## <a name="which-method-is-right-for-your-app"></a>哪一種方法最適合您的應用程式？ 

需要考量的主要因素如下：

- **Docker 在您的開發工作流程中的可用性**：自訂映像開發需要擁有對 Docker 開發工作流程的基本知識。 將自訂映像部署至 Web 應用程式需要將自訂映像發佈到 Docker Hub 之類的儲存機制主機。 如果您對 Docker 相當熟悉，而且可以將 Docker 工作新增至組建工作流程，或是您已經將應用程式發佈為 Docker 映像，那麼自訂映像可說一定是最佳選擇。
- **多層式架構**： 部署多個容器 (例如 Web 應用程式層和 API 層)，使用多個容器劃分功能。 
- **應用程式效能**：使用 Redis 等快取層級，提升多容器應用程式的效能。 選取多容器可達成此目的。
- **獨特的執行階段需求**：內建平台映像的設計符合大部分的 Web 應用程式的需求，但是自訂能力受到侷限。 您的應用程式在相依性或其他執行階段方面可能有獨特的需求，超出了內建映像的處理能力。
- **組建需求**：藉由[持續部署](../app-service-continuous-deployment.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)，您可以直接從原始程式碼在 Azure 上啟動和執行您的應用程式。 不需要外部的組建或發佈程序。 不過，[Kudu](https://github.com/projectkudu/kudu/wiki)部署引擎內的組建工具在自訂能力和可用性方面受到限制。 您的應用程式可能隨著對於自訂組建邏輯的相依性或需求增加，而超出 Kudu 的功能範圍。
- **磁碟讀取/寫入需求**：所有 Web 應用程式會針對 Web 內容配置存放磁碟區。 此磁碟區獲得 Azure 儲存體支援，掛接於應用程式檔案系統中的 `/home`。 不同於容器檔案系統中的檔案，磁碟區中的檔案可供應用程式所有級別的執行個體存取，而且修改會保存到應用程式重新啟動之後。 不過，內容磁碟區的磁碟延遲較高，延遲的變化也比本機容器檔案系統更多，而且存取可能會受到平台升級、規劃外的停機時間和網路連線問題所影響。 自訂映像部署可能有益於需要對於內容檔案進行大量唯讀存取的應用程式，這類應用程式會將檔案放在映像檔案系統上，而非內容磁碟區上。
- **組建資源使用方式**：若是從來源部署應用程式，Kudu 執行的部署指令碼會使用與執行中的應用程式相同的 App Service 方案之計算和儲存體資源。 大型應用程式部署可能會耗用比所需程度更多的資源或時間。 尤其許多部署工作流程會在應用程式內容磁碟區產生大量磁碟活動，但是應用程式內容磁碟區並非針對這類活動最佳化。 自訂映像不需要額外的檔案傳輸或部署動作，就能夠以單一封裝將所有的應用程式檔案和相依性提供給 Azure。
- **需要快速反覆項目**：Docker 化應用程式需要額外的組建步驟。 若要使變更生效，在每次更新時都必須將新的映像推送到儲存機制。 這些更新接著會被提取到 Azure 環境。 如果其中一個內建容器符合您的應用程式需求，從來源部署可能會加快開發工作流程。

## <a name="multi-container-apps-supportability"></a>多容器應用程式支援能力

### <a name="supported-docker-compose-configuration-options"></a>支援的 Docker Compose 組態選項
- 命令
- entrypoint
- Environment
- 映像
- 連接埠
- restart
- 服務
- 磁碟區

### <a name="unsupported-docker-compose-configuration-options"></a>不支援的 Docker Compose 組態選項
- build (不允許)
- depends_on (已忽略)
- networks (已忽略)
- secrets (已忽略)
- 80 和 8080 以外的連接埠 (已忽略)

> [!NOTE]
> 公開預覽中也會忽略任何未明確標註的其他選項。

### <a name="supported-kubernetes-configuration-options"></a>支援的 Kubernetes 組態選項
- args
- 命令
- containers
- 映像
- name
- 連接埠
- 規格

> [!NOTE]
>公開預覽中不支援任何未明確標註的其他 Kubernetes 選項。
>
