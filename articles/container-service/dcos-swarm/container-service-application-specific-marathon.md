---
title: 應用程式或使用者特定的 Marathon 服務
description: 建立應用程式或使用者特定的 Marathon 服務
services: container-service
author: rgardler
manager: jeconnoc
ms.service: container-service
ms.topic: article
ms.date: 04/12/2016
ms.author: rogardle
ms.custom: mvc
ms.openlocfilehash: 4ff263fe0ca4f435199127ed64faadee1c2527f9
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 04/28/2018
ms.locfileid: "32164535"
---
# <a name="create-an-application-or-user-specific-marathon-service"></a>建立應用程式或使用者特定的 Marathon 服務

Azure Container Service 提供了一組主要伺服器，供我們在上面預先設定 Apache Mesos 和 Marathon。 這些伺服器可用來協調叢集上的應用程式，但最好不要將主要伺服器用在此目的。 例如，若要調整 Marathon 組態，就必須登入主要伺服器本身並進行變更 -- 這會導致產生與標準組態只有些許不同的獨特主要伺服器，因而需要獨立處理和管理。 此外，某個團隊所需的組態可能不是另一個團隊最適合的組態。

在本文中，我們將說明如何新增應用程式或使用者特定的 Marathon 服務。

由於此服務將隸屬於單一使用者或團隊，因此可以用任何想要的方式加以設定。 此外，Azure 容器服務會確保服務繼續執行。 如果服務失敗，Azure 容器服務會為您重新啟動該服務。 多數時候，您甚至不會發覺它有停機過。

## <a name="prerequisites"></a>先決條件
[部署 Azure Container Service 的執行個體](container-service-deployment.md) (其 Orchestrator 類型為 DCOS)，並[確保您的用戶端可以連線至您的叢集](../container-service-connect.md)。 此外，請執行下列步驟。

[!INCLUDE [install the DC/OS CLI](../../../includes/container-service-install-dcos-cli-include.md)]

## <a name="create-an-application-or-user-specific-marathon-service"></a>建立應用程式或使用者特定的 Marathon 服務
一開始先建立 JSON 組態檔，以定義您想要建立的應用程式服務的名稱。 在此，我們使用 `marathon-alice` 做為架構名稱。 將檔案儲存為類似 `marathon-alice.json`的名稱：

```json
{"marathon": {"framework-name": "marathon-alice" }}
```

接下來，使用 DC/OS CLI，以組態檔中設定的選項安裝 Marathon 執行個體︰

```bash
dcos package install --options=marathon-alice.json marathon
```

現在，您應該會在 DC/OS UI 的 [服務] 索引標籤中看到 `marathon-alice` 服務正在執行。 如果您想要直接存取，此 UI 會是 `http://<hostname>/service/marathon-alice/` 。

## <a name="set-the-dcos-cli-to-access-the-service"></a>設定 DC/OS CLI 來存取服務
您可以藉由將 `marathon.url` 屬性設定為指向 `marathon-alice` 執行個體 (如下所示)，選擇性地設定 DC/OS CLI 來存取這個新服務︰

```bash
dcos config set marathon.url http://<hostname>/service/marathon-alice/
```

您可以使用 `dcos config show` 命令確認 CLI 正在針對哪一個 Marathon 執行個體運作。 您可以使用 `dcos config unset marathon.url`命令還原為使用主要的 Marathon 服務。

