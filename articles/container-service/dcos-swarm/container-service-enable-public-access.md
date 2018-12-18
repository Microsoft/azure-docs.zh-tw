---
title: 啟用對於 Azure DC/OS 容器應用程式的存取
description: 如何在 Azure Container Service 中啟用對 DC/OS 容器的公用存取。
services: container-service
author: sauryadas
manager: madhana
ms.service: container-service
ms.topic: article
ms.date: 08/26/2016
ms.author: saudas
ms.custom: mvc
ms.openlocfilehash: aedc97335a0b9ad00cf653477b62bf530b556900
ms.sourcegitcommit: 5d3e99478a5f26e92d1e7f3cec6b0ff5fbd7cedf
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/06/2017
ms.locfileid: "26332274"
---
# <a name="enable-public-access-to-an-azure-container-service-application"></a>啟用 Azure Container Service 應用程式的公用存取

ACS [公用代理程式集區](container-service-mesos-marathon-ui.md#deploy-a-docker-formatted-container) 中的任何 DC/OS 容器都會自動公開到網際網路。 根據預設，連接埠 **80**、**443**、**8080** 已開啟，在這些連接埠上接聽的任何 (公用) 容器皆可供存取。 本文將說明如何在 Azure Container Service 中開啟更多的連接埠供應用程式使用。

## <a name="open-a-port-portal"></a>開啟連接埠 (入口網站)
首先，我們必須開啟所需的連接埠。

1. 登入入口網站。
2. 尋找 Azure Container Service 所部署到的資源群組。
3. 選取代理程式的負載平衡器 (其名稱類似 **XXXX-agent-lb-XXXX**)。
   
    ![Azure Container Service 的負載平衡器](./media/container-service-enable-public-access/agent-load-balancer.png)
4. 依序按一下 [探查] 和 [新增]。
   
    ![Azure Container Service 的負載平衡器探查](./media/container-service-enable-public-access/add-probe.png)
5. 填寫探查表單，然後按一下 [確定] 。
   
   | 欄位 | 說明 |
   | --- | --- |
   | Name |探查的描述性名稱。 |
   | Port |要測試之容器的連接埠。 |
   | Path |(處於 HTTP 模式時) 探查的相對網站路徑。 不支援 HTTPS。 |
   | 間隔 |探查嘗試間隔的時間量 (秒)。 |
   | 狀況不良臨界值 |在將容器視為狀況不良之前的連續探查嘗試次數。 |
6. 回到代理程式負載平衡器的屬性中，依序按一下 [負載平衡規則] 和 [新增]。
   
    ![Azure Container Service 的負載平衡器規則](./media/container-service-enable-public-access/add-balancer-rule.png)
7. 填寫負載平衡器表單，然後按一下 [確定] 。
   
   | 欄位 | 說明 |
   | --- | --- |
   | Name |負載平衡器的描述性名稱。 |
   | Port |公用的連入通訊埠。 |
   | 後端連接埠 |要將流量路由傳送到之容器的內部對公用連接埠。 |
   | 後端集區 |此集區中的容器將會是此負載平衡器的目標。 |
   | 探查 |用來判斷 **後端集區** 中的目標是否狀況良好的探查。 |
   | 工作階段持續性 |決定針對工作階段的持續時間，應該如何處理來自用戶端的流量。<br><br>**無**：來自相同用戶端的後續要求可以由任何容器處理。<br>**用戶端 IP**：來自相同用戶端 IP 的後續要求會由相同容器處理。<br>**用戶端 IP 和通訊協定**：來自相同用戶端 IP 和通訊協定組合的後續要求會由相同容器處理。 |
   | 閒置逾時 |(僅限 TCP) 讓 TCP/HTTP 用戶端保持開啟，而不依賴「keep-alive」  訊息的時間，以分鐘為單位。 |

## <a name="add-a-security-rule-portal"></a>新增安全性規則 (入口網站)
接下來，我們需要新增會從開啟的連接埠通過防火牆路由傳送流量的安全性規則。

1. 登入入口網站。
2. 尋找 Azure Container Service 所部署到的資源群組。
3. 選取**公用**代理程式網路安全性群組 (其名稱類似 **XXXX-agent-public-nsg-XXXX**)。
   
    ![Azure Container Service 網路安全性群組](./media/container-service-enable-public-access/agent-nsg.png)
4. 依序選取 [輸入安全性規則] 和 [新增]。
   
    ![Azure Container Service 網路安全性群組規則](./media/container-service-enable-public-access/add-firewall-rule.png)
5. 填寫防火牆規則以允許公用連接埠，然後按一下 [確定] 。
   
   | 欄位 | 說明 |
   | --- | --- |
   | Name |防火牆規則的描述性名稱。 |
   | 優先順序 |規則的優先順序排名。 編號愈低，優先順序就愈高。 |
   | 來源 |限制此規則要允許或拒絕的連入 IP 位址範圍。 使用 **任何** 即可不指定限制。 |
   | 服務 |選取一組適用此安全性規則的預先定義服務。 否則，使用 **自訂** 建立自己的服務。 |
   | 通訊協定 |根據 **TCP** 或 **UDP** 限制流量。 使用 **任何** 即可不指定限制。 |
   | 連接埠範圍 |當**服務**是**自訂**時，指定此規則會影響的連接埠範圍。 您可以使用單一連接埠 (例如 **80**) 或 **1024-1500** 之類的範圍。 |
   | 動作 |允許或拒絕符合條件的流量。 |

## <a name="next-steps"></a>後續步驟
了解 [公用和私用 DC/OS 代理程式](container-service-dcos-agents.md)之間的差異。

深入了解 [管理 DC/OS 容器](container-service-mesos-marathon-ui.md)。

