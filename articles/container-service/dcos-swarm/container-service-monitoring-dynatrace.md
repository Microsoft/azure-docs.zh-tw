---
title: 監視 Azure DC/OS 叢集 - Dynatrace
description: 使用 Dynatrace 監視 Azure Container Service DC/OS 叢集。 使用 DC/OS 儀表板部署 Dynatrace OneAgent。
services: container-service
author: MartinGoodwell
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 12/13/2016
ms.author: rogardle
ms.custom: mvc
ms.openlocfilehash: 3d1bfc3bb61781d487c40831edd5da6fcb5a7df9
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32162036"
---
# <a name="monitor-an-azure-container-service-dcos-cluster-with-dynatrace-saasmanaged"></a>使用 Dynatrace SaaS/受控監視 Azure Container Service DC/OS 叢集

在本文中，我們會示範如何部署 [Dynatrace](https://www.dynatrace.com/) OneAgent，以監視 Azure Container Service 叢集中的所有代理程式節點。 您需要 Dynatrace SaaS/受控帳戶以進行這項設定。 

## <a name="dynatrace-saasmanaged"></a>Dynatrace SaaS/受控
Dynatrace 是高動態容器和叢集環境適用的雲端原生監視解決方案。 它可讓您使用即時使用量資料，進一步最佳化您的容器部署和記憶體配置。 其藉由提供自動化基準、問題相互關聯和根本原因偵測，就能夠自動查明應用程式和基礎結構問題。

下圖顯示 Dynatrace UI：

![Dynatrace UI](./media/container-service-monitoring-dynatrace/dynatrace.png)

## <a name="prerequisites"></a>先決條件 
[部署](container-service-deployment.md)和[連接](./../container-service-connect.md)至 Azure Container Service 所設定的叢集。 瀏覽 [Marathon UI](container-service-mesos-marathon-ui.md)。 移至 [https://www.dynatrace.com/trial/](https://www.dynatrace.com/trial/) 以設定 Dynatrace SaaS 帳戶。  

## <a name="configure-a-dynatrace-deployment-with-marathon"></a>使用 Marathon 設定 Dynatrace 部署
這些步驟示範如何使用 Marathon 設定 Dynatrace 應用程式並將它部署到您的叢集。

1. 透過 [http://localhost:80/](http://localhost:80/) 存取 DC/OS UI。 在進入 DC/OS UI 後，瀏覽至 **Universe**，然後搜尋 **Dynatrace**。

    ![DC/OS Universe 中的 Dynatrace](./media/container-service-monitoring-dynatrace/dynatrace-universe.png)

2. 若要完成設定，您需要 Dynatrace SaaS 帳戶或免費試用帳戶。 登入 Dynatrace 儀表板後，請選取 [部署 Dynatrace]。

    ![Dynatrace 設定 PaaS 整合](./media/container-service-monitoring-dynatrace/setup-paas.png)

3. 在頁面上，選取 [設定 PaaS 整合]。 

    ![Dynatrace API 權杖](./media/container-service-monitoring-dynatrace/api-token.png) 

4. 將 API 金鑰輸入到 DC/OS Universe 中的 Dynatrace OneAgent 組態。 

    ![DC/OS Universe 中的 Dynatrace OneAgent 組態](./media/container-service-monitoring-dynatrace/dynatrace-config.png)

5. 將執行個體設定為您想要執行的節點數目。 設定較高的數目也可行，但 DC/OS 會繼續嘗試尋找新的執行個體，直到實際達到該數目為止。 如果想要的話，您也可以將此設定為 1000000 之類的值。 在此情況下，每當新節點新增到叢集時，Dynatrace 就會自動將代理程式部署到該新節點，並採用 DC/OS 不斷嘗試部署其他執行個體的價格。

    ![DC/OS Universe 中的 Dynatrace 組態 - 執行個體](./media/container-service-monitoring-dynatrace/dynatrace-config2.png)

## <a name="next-steps"></a>後續步驟

一旦您安裝了套件，請瀏覽回到 Dynatrace 儀表板。 您可以探索您的叢集中各容器的不同使用量計量。 