---
title: 自訂連線的處理站解決方案 - Azure | Microsoft Docs
description: 說明如何自訂連線處理站解決方案加速器的行為。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.devlang: csharp
ms.topic: conceptual
ms.date: 12/14/2017
ms.author: dobett
ms.openlocfilehash: e91f36c9d6f0cb3195e6903d55cd676f1e9ccf5a
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34626545"
---
# <a name="customize-how-the-connected-factory-solution-displays-data-from-your-opc-ua-servers"></a>自訂連線處理站解決方案顯示 OPC UA 伺服器資料的方式

連線的處理站解決方案會彙總並顯示連線到解決方案的 OPC UA 伺服器資料。 您可以瀏覽並傳送命令至解決方案的 OPC UA 伺服器。 如需 OPC UA 的詳細資訊，請參閱[連線的處理站常見問題集](iot-accelerators-faq-cf.md)。

解決方案中的彙總資料範例包括「整體設備效率 (OEE)」和「關鍵效能指標 (KPI)」，您可以在儀表板的工廠、生產線與工作站層級分別檢視這些資料。 下列螢幕擷取畫面顯示 [Munich (慕尼黑)] 工廠的 [Production line 1 (生產線 1)] 的 [Assembly (組件)] 工作站的 OEE 和 KPI 值：

![解決方案中的 OEE 和 KPI 值範例][img-oee-kpi]

解決方案可讓您檢視 OPC UA 伺服器 (稱為「工作站」) 中特定資料項目的詳細資訊。 下列螢幕擷取畫面顯示特定工作站製造的項目數的繪圖︰

![製造項目數的繪圖][img-manufactured-items]

如果您按一下其中一個圖形，可使用 Time Series Insights (TSI) 進一步探索資料︰

![使用 Time Series Insights 探索資料][img-tsi]

本文章說明：

- 解決方案中各種檢視的資料是如何取得的。
- 如何自訂解決方案顯示資料的方式。

## <a name="data-sources"></a>資料來源

連線的處理站解決方案會顯示連線到解決方案的 OPC UA 伺服器資料。 預設安裝包含數台執行工廠模擬的 OPC UA 伺服器。 您可以將[透過閘道連線][lnk-connect-cf]的自有 OPC UA 伺服器新增到解決方案。

您可以在儀表板中瀏覽連線的 OPC UA 伺服器可傳送至您的解決方案的資料項目︰

1. 選取 [瀏覽器] 以瀏覽至 [選取 OPC UA 伺服器] 檢視︰

    ![瀏覽至 [Select an OPC UA server (選取 OPC UA 伺服器)] 檢視][img-select-server]

1. 選取伺服器，然後按一下 [Connect (連線)]。 出現安全性警告時，按一下 [Proceed (繼續)]。

    > [!NOTE]
    > 這個警告每台伺服器只會出現一次，是建立解決方案儀表板和伺服器之間的信任關係。

1. 您現在可以瀏覽伺服器可傳送至解決方案的資料項目。 正在傳送至解決方案的項目會有核取記號︰

    ![發行的項目][img-published]

1. 如果您是解決方案的「管理員」，則可以選擇發行資料項目，以便在連線的處理站解決方案中加以提供。 身為管理員，您也可以變更 OPC UA 伺服器中的資料項目值及呼叫方法。

## <a name="map-the-data"></a>對應資料

連線的處理站解決方案會將 OPC UA 伺服器中發行的資料項目對應並彙總至解決方案中的各種檢視。 當您佈建連線的處理站解決方案時，此解決方案就會部署到您的 Azure 帳戶。 Visual Studio 連線處理站解決方案中的 JSON 檔案會儲存此對應資訊。 您可以在連線的處理站 Visual Studio 解決方案中檢視及修改這個 JSON 設定檔。 您進行變更之後，可以重新部署解決方案。

您可以使用此組態檔來︰

- 編輯現有的模擬工廠、生產線和工作站。
- 從連接到解決方案的實際 OPC UA 伺服器對應資料。

如需對應和彙總資料以符合特定需求的詳細資訊，請參閱[如何設定連線的處理站解決方案加速器](iot-accelerators-connected-factory-configure.md)。

## <a name="deploy-the-changes"></a>部署變更

完成 **ContosoTopologyDescription.json** 檔案的變更後，您必須將連線的處理站解決方案重新部署到您的 Azure 帳戶。

**azure-iot-connected-factory** 存放庫包含一個 **build.ps1** PowerShell 指令碼，可用於重新建置並部署解決方案。

## <a name="next-steps"></a>後續步驟

閱讀下列文章，深入了解連線的處理站解決方案加速器：

* [連線的處理站解決方案加速器逐步解說][lnk-rm-walkthrough]
* [為連線的處理站部署閘道][lnk-connect-cf]
* [azureiotsuite.com 網站的權限][lnk-permissions]
* [連線的處理站常見問題集](iot-accelerators-faq-cf.md)
* [常見問題集][lnk-faq]


[img-oee-kpi]: ./media/iot-accelerators-connected-factory-customize/oeenadkpi.png
[img-manufactured-items]: ./media/iot-accelerators-connected-factory-customize/manufactured.png
[img-tsi]: ./media/iot-accelerators-connected-factory-customize/tsi.png
[img-select-server]: ./media/iot-accelerators-connected-factory-customize/selectserver.png
[img-published]: ./media/iot-accelerators-connected-factory-customize/published.png


[lnk-rm-walkthrough]:iot-accelerators-connected-factory-sample-walkthrough.md
[lnk-connect-cf]:iot-accelerators-connected-factory-gateway-deployment.md
[lnk-permissions]: iot-accelerators-permissions.md
[lnk-faq]: iot-accelerators-faq.md