---
title: 使用 Grafana 監視 Azure 服務與應用程式
description: 路由 Azure 監視器與 Application Insights 資料，以便使用 Grafana 檢視它們。
services: azure-monitor
keywords: ''
author: rboucher
ms.author: robb
ms.date: 11/06/2017
ms.topic: conceptual
ms.service: azure-monitor
ms.component: ''
ms.openlocfilehash: de2c57949cb2087e41b79a225963225d340f12af
ms.sourcegitcommit: 1b8665f1fff36a13af0cbc4c399c16f62e9884f3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/11/2018
ms.locfileid: "35263077"
---
# <a name="monitor-your-azure-services-in-grafana"></a>在 Grafana 中監視 Azure 服務
您現在也可以從使用 [Azure 監視器資料來源外掛程式](https://grafana.com/plugins/grafana-azure-monitor-datasource)的 [Grafana](https://grafana.com/)，監視 Azure 服務和應用程式。 此外掛程式會蒐集 Application Insights SDK 所收集的應用程式效能資料，以及 Azure 監視器提供的基礎結構資料。 然後，您就可以在 Grafana 儀表板上顯示此資料。

此外掛程式目前為預覽狀態。

使用下列步驟來設定 Azure Marketplace 的 Grafana 伺服器，以及建置 Azure 監視器和 Application Insights 的計量儀表板。

## <a name="set-up-a-grafana-instance"></a>設定 Grafana 執行個體
1. 移至 Azure Marketplace，並依 Grafana 實驗室挑選 Grafana。

2. 填入名稱和詳細資料。 建立新的資源群組。 持續追蹤您選擇的 VM 使用者名稱、VM 密碼和 Grafana 伺服器系統管理員密碼的值。  

3. 選擇 VM 大小和儲存體帳戶。

4. 設定網路組態設定。

5. 檢視摘要，並在接受使用條款之後選取 [建立]。

## <a name="log-in-to-grafana"></a>登入 Grafana
1. 部署完成之後，請選取 [移至資源群組]。 您會看到新建的資源清單。

    ![Grafana 資源群組物件](.\media\monitor-how-to-grafana\grafana1.png)

    如果選取網路安全性群組 (本例中為 *grafana-nsg*)，您會看到使用連接埠 3000 存取 Grafana 伺服器。

2. 請返回資源清單，然後選取 [公用 IP 位址]。 使用這個畫面上找到的值，在瀏覽器中鍵入 *http://<IP address>:3000* 或 *<DNSName>:3000*。 您應該會看到剛才建的 Grafana 伺服器登入頁面。

    ![Grafana 登入畫面](.\media\monitor-how-to-grafana\grafana2.png)

3. 以 *admin* 身分的使用者名稱和您稍早建立的 Grafana 伺服器管理員密碼來登入。

## <a name="configure-data-source-plugin"></a>設定資料來源外掛程式

成功登入之後，您應該會看到已經包含 Azure 監視器資料來源外掛程式。

![Grafana 會顯示 Azure 監視器外掛程式](.\media\monitor-how-to-grafana\grafana3.png)

1. 選取 [新增資料來源] 設定 Azure 監視器和 Application Insights。

2. 選取資料來源的名稱，然後從下拉式清單中選取 [Azure 監視器] 為資料來源。


## <a name="create-a-service-principal"></a>建立服務主體

Grafana 使用 Azure Active Directory 服務主體連接到 Azure 監視器 API，並收集計量資料。 您必須建立服務主體來管理對 Azure 資源的存取。

1. 請參閱[這些指示](../azure-resource-manager/resource-group-create-service-principal-portal.md)來建立服務主體。 複製並儲存您的租用戶識別碼、用戶端識別碼和用戶端密碼。

2. 請參閱[將應用程式指派給角色](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-create-service-principal-portal#assign-application-to-role)，將讀取器角色指派給 Azure Active Directory 應用程式。     

3. 如果您使用 Application Insights，您也可以包含 Application Insights API 和應用程式識別碼，收集 Application Insights 的基礎計量。 如需詳細資訊，請參閱 [Getting your API key and Application ID](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID) (取得 API 金鑰和應用程式識別碼)。

4. 輸入全部資訊後，選取 [儲存]，Grafana 就會測試 API。 您應該會看見類似下方的訊息。  

    ![Grafana 會顯示 Azure 監視器外掛程式](.\media\monitor-how-to-grafana\grafana4-1.png)

> [!NOTE]
> 設定外掛程式時，您可以指示要針對哪個 Azure 雲端 (公用、Azure 美國政府、Azure 德國或Azure 中國) 設定外掛程式。
>
>

## <a name="build-a-grafana-dashboard"></a>建置 Grafana 儀表板

1. 前往首頁，然後選取 [新增儀表板]。

2. 在新的儀表板中選取 [圖形]。 您可以嘗試其他圖表選項，但本文使用 [圖形] 作為範例。

    ![Grafana 新增儀表板](.\media\monitor-how-to-grafana\grafana5.png)

3. 空白的圖形會出現在儀表板中。

4. 在面板標題上按一下，然後選取 [編輯] 輸入您想要在此圖表中繪製的資料詳細資料。

5. 一旦選取所有正確的 VM，您就可以開始檢視儀表板中的計量。

以下是有兩份圖表的簡單儀表板。 左邊的這份顯示兩個 VM 的 CPU 百分比。 右側的圖表依交易 API 類型細分，顯示 Azure 儲存體帳戶中的交易。

![Grafana 雙圖表範例](.\media\monitor-how-to-grafana\grafana6.png)


## <a name="optional-create-dashboard-playlists"></a>選擇性：建立儀表板播放清單

Grafana 有許多實用功能，儀表板播放清單就是其中之一。 您可以建立多個儀表板，並將它們新增至播放清單，設定每個儀表板的顯示間隔。 選取 [播放] 查看循環播放的儀表板。 您可能想要在大型的牆面監視器中顯示它們，為群組提供「狀態面板」。

![Grafana 播放清單範例](.\media\monitor-how-to-grafana\grafana7.png)


## <a name="optional-monitor-your-custom-metrics-in-the-same-grafana-server"></a>選擇性：在相同的 Grafana 伺服器中監視自訂的計量

您也可以安裝 Telegraf 和 InfluxDB，收集自訂和代理程式型的計量，並繪製在相同的 Grafana 執行個體中。 有許多資料來源外掛程式可將這些計量結合在儀表板中。

您也可以重複使用這項設定，以包含來自 Prometheus 伺服器的計量。 使用 Grafana 外掛程式庫中的 Prometheus 資料來源外掛程式。

以下幾篇是非常好的 Telegraf、InfluxDB、Prometheus 和 Docker 使用方式參考文章
 - [How To Monitor System Metrics with the TICK Stack on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-monitor-system-metrics-with-the-tick-stack-on-ubuntu-16-04) (如何在 Ubuntu 16.04 上使用刻度堆疊監視系統計量)

 - [Monitor Docker resource metrics with Grafana, InfluxDB, and Telegraf](https://blog.vpetkov.net/2016/08/04/monitor-docker-resource-metrics-with-grafana-influxdb-and-telegraf/) (使用 Grafana、InfluxDB 及 Telegraf 監視 Docker 資源計量)

 - [A monitoring solution for Docker hosts, containers, and containerized services](https://stefanprodan.com/2016/a-monitoring-solution-for-docker-hosts-containers-and-containerized-services/) (Docker 主機、容器和容器化服務的監控解決方案)

以下是完整的 Grafana 儀表板映像，具有來自 Azure 監視器和 Application Insights 的計量。
![Grafana 範例計量](.\media\monitor-how-to-grafana\grafana8.png)


## <a name="clean-up-resources"></a>清除資源

只要 VM 在執行中，不論您是否使用它們，都會收費。 為避免產生額外費用，請清除依本文建立的資源群組。

1. 從 Azure 入口網站的左側功能表中，依序按一下 [資源群組] 和 [Grafana]。
2. 在資源群組頁面上，按一下 [刪除]，在文字方塊中鍵入 **Grafana**，然後按一下 [刪除]。

## <a name="next-steps"></a>後續步驟
* [Azure 監視器計量概觀](monitoring-overview-metrics.md)
