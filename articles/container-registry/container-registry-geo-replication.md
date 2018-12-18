---
title: 異地複寫 Azure 容器登錄
description: 開始建立及管理異地複寫 Azure 容器登錄。
services: container-registry
author: stevelas
manager: jeconnoc
ms.service: container-registry
ms.topic: overview-article
ms.date: 04/10/2018
ms.author: stevelas
ms.openlocfilehash: e4695428b03961f5e899007609dfb1088dde77a8
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33768204"
---
# <a name="geo-replication-in-azure-container-registry"></a>Azure 容器登錄中的異地複寫

公司想要本機存在或熱備份時，選擇從多個 Azure 區域執行服務。 最佳做法是將容器登錄中放入每個區域，其中執行映像以允許網路關閉作業、啟用快速、可靠的映像圖層傳輸。 異地複寫可讓 Azure 容器登錄成為單一登錄、服務包含多個區域登錄的多重主要區域。

異地複寫登錄提供下列優點：

* 可跨多個區域使用單一登錄/映像/標記名稱
* 從區域部署網路關閉登錄存取
* 沒有其他輸出費用，因為映像是取自容器主機的相同區域中的本機、複寫的登錄
* 跨多個區域單一管理登錄

## <a name="example-use-case"></a>使用案例範例
Contoso 會執行位在美國、加拿大和歐洲的公開金鑰存在網站。 為了在這些市場中提供本機和網路關閉內容，Contoso 在美國西部、美國東部、加拿大中央和西歐執行 [Azure Container Service](/azure/container-service/kubernetes/)(ACS) Kubernetes 叢集。 網站應用程式 (部署為 Docker 映像) 會在所有區域利用相同的程式碼和映像。 從資料庫 (基本在每個區域中佈建) 擷取該區域的本機內容。 每個區域部署都會有其資源的唯一設定，例如本機資料庫。

開發小組位於 Seattle WA，利用美國西部的資料中心。

![推入至多個登錄](media/container-registry-geo-replication/before-geo-replicate.png)<br />*推入至多個登錄*

在使用異地複寫功能前，Contoso 在美國西部具有美國型登錄，在西歐有其他登錄。 若要提供這些不同的區域，開發小組必須將映像發送至兩個不同的登錄。

```bash
docker push contoso.azurecr.io/pubic/products/web:1.2
docker push contosowesteu.azurecr.io/pubic/products/web:1.2
```
![從多個登錄提取](media/container-registry-geo-replication/before-geo-replicate-pull.png)<br />*從多個登錄提取*

多個登錄的常見難題包含：

* 美國東部、美國西部和加拿大中心叢集皆從美國西部登錄擷取，因為其中每個遠端容器主機皆從美國西部資料中心提取資料，因此會產生輸出費用。
* 開發小組必須將映像推送到美國西部和西歐登錄中。
* 開發小組必須設定及維護參考本機登錄之影像名稱所在的每個區域部署。
* 必須為每個區域設定登錄存取。

## <a name="benefits-of-geo-replication"></a>異地複寫的優點

![從異地複寫登錄中提取](media/container-registry-geo-replication/after-geo-replicate-pull.png)

使用 Azure Container Registry 的異地複寫功能來實現這些優點：

* 在所有區域管理單一登錄：`contoso.azurecr.io`
* 管理單一映像部署設定，因為所有區域都使用相同的映像 URL：`contoso.azurecr.io/public/products/web:1.2`
* 推送至單一登錄，同時 ACR 會管理異地複寫，包括本機通知的區域 webhook

## <a name="configure-geo-replication"></a>設定異地複寫
設定異地複寫是簡單的，只要按一下地圖上的區域。

異地複寫是[進階登錄](container-registry-skus.md)的一項功能。 如果您的登錄還不是進階，您可以在 [Azure 入口網站](https://portal.azure.com)中從基本和標準變更為進階：

![在 Azure 入口網站中切換 SKU](media/container-registry-skus/update-registry-sku.png)

若要設定進階登錄的異地複寫，請登入 Azure 入口網站 ( http://portal.azure.com )。

導覽到 Azure 容器登錄，並選取 [複寫]：

![在 Azure 入口網站的容器登錄 UI 中進行複寫](media/container-registry-geo-replication/registry-services.png)

地圖會顯示，包含所有目前的 Azure 區域：

 ![Azure 入口網站的區域圖](media/container-registry-geo-replication/registry-geo-map.png)

* 藍色六邊形代表目前的複本
* 綠色六邊形代表可能的複本區域
* 灰色六邊形代表尚未提供複寫的 Azure 區域

若要設定複本，選取綠色六邊形，然後選取 [建立]：

 ![在 Azure 入口網站中建立複寫 UI](media/container-registry-geo-replication/create-replication.png)

若要設定其他複本，請選取其他地區的綠色六邊形，然後按一下 [建立]。

ACR 會開始同步設定的複本之間的映像。 完成時，入口網站會反映 [準備]。 入口網站中的複本狀態不會自動更新。 使用 [重新整理] 按鈕以查看更新的狀態。

## <a name="geo-replication-pricing"></a>異地複寫價格

異地複寫是 Azure Container Registry 之[進階 SKU](container-registry-skus.md) 的功能。 當您要複寫登錄到您想要的區域時，您會產生每個區域的進階登錄費用。

在上述範例中，Contoso 會將兩個登錄向下合併成一個，並將複本新增至美國東部、加拿大中部和西歐。 Contoso 應支付每月的次進階費用，不含任何額外的設定或管理。 每個區域現在會在本機提取其映像，改善效能和可靠性，而不會衍生從美國西部到加拿大和美國東部的網路輸出費用。

## <a name="summary"></a>總結

利用異地複寫，您可以將區域資料中心作為一個全域雲端管理。 在許多 Azure 服務中使用映像，您可以從單一管理平面獲益，同時維持網路關閉、快速，及可靠提取本機映像。

## <a name="next-steps"></a>後續步驟

簽出三段式教學課程系列，[Azure Container Registry 中的異地複寫](container-registry-tutorial-prepare-registry.md)。 逐步解說建立異地備援登錄、建立容器，然後再使用單一 `docker push` 命令將其部署到多個區域之 適用於容器的 Web Apps 執行個體。

> [!div class="nextstepaction"]
> [Azure 容器登錄中的異地複寫](container-registry-tutorial-prepare-registry.md)