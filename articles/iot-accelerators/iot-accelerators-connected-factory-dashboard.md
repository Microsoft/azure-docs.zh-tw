---
title: 使用連線的處理站儀表板 - Azure | Microsoft Docs
description: 了解如何使用連線處理站儀表板的功能。
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 07/10/2018
ms.author: dobett
ms.openlocfilehash: 4af93b5667181941585cc1ac89f5696379962026
ms.sourcegitcommit: 0b05bdeb22a06c91823bd1933ac65b2e0c2d6553
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/17/2018
ms.locfileid: "39075150"
---
# <a name="use-features-in-the-connected-factory-solution-accelerator-dashboard"></a>使用連線處理站儀表板的功能

[部署雲端式解決方案來管理我的工業 IoT 裝置](quickstart-connected-factory-deploy.md)快速入門會示範如何瀏覽儀表板以及回應警示。 本使用方法指南會示範一些額外的儀表板功能，可用來監視及管理您的工業 IoT 裝置。

## <a name="apply-filters"></a>套用篩選條件

您可以篩選儀表板上顯示的資訊，例如**處理站位置**面板或**警示**面板：

1. 按一下**漏斗**圖示，可在 [處理站位置] 面板或 [警示] 面板中顯示可用的篩選條件清單。

1. 篩選條件面板隨即顯示：

    [![連線的處理站解決方案加速器篩選條件](./media/iot-accelerators-connected-factory-dashboard/filterpanel-inline.png)](./media/iot-accelerators-connected-factory-dashboard/filterpanel-expanded.png#lightbox)

1. 選擇您需要的篩選條件，然後按一下 [套用] 。 您也可以在篩選條件欄位中輸入任意文字。

1. 然後會套用篩選條件。 額外的漏斗圖示表示已套用篩選條件：

    [![連線的處理站解決方案加速器篩選條件已套用](./media/iot-accelerators-connected-factory-dashboard/filterapplied-inline.png)](./media/iot-accelerators-connected-factory-dashboard/filterapplied-expanded.png#lightbox)

    > [!NOTE]
    > 使用中的篩選條件不會影響已顯示的 OEE 和 KPI 值，它只會篩選清單內容。

1. 若要清除篩選條件，請按一下漏斗圖示並按一下篩選內容面板中的 [清除]。

## <a name="browse-an-opc-ua-server"></a>瀏覽 OPC UA 伺服器

當您部署解決方案加速器時，會自動佈建一組可從儀表板瀏覽的模擬 OPC UA 伺服器。 模擬伺服器可讓您輕鬆對解決方案加速器進行實驗，而不需要部署真實的伺服器。 若要將真實 OPC UA 伺服器連線至解決方案，請參閱[將 OPC UA 裝置連線至連線的處理站解決方案加速器](iot-accelerators-connected-factory-gateway-deployment.md)教學課程。

1. 按一下儀表板導覽列中的 [瀏覽器圖示]：

    [![連線的處理站解決方案加速器伺服器瀏覽器](./media/iot-accelerators-connected-factory-dashboard/browser-inline.png)](./media/iot-accelerators-connected-factory-dashboard/browser-expanded.png#lightbox)

1. 在解決方案加速器中，從為您所部署的伺服器清單中，選擇其中一個伺服器：

    [![連線的處理站解決方案加速器伺服器清單](./media/iot-accelerators-connected-factory-dashboard/serverlist-inline.png)](./media/iot-accelerators-connected-factory-dashboard/serverlist-expanded.png#lightbox)

1. 按一下 [連線]，[安全性] 對話方塊隨即顯示。 若要進行模擬，您可以放心按一下 [繼續]。

1. 若要展開伺服器樹狀目錄中的任何節點，請按一下該節點。 正在發佈遙測的節點旁邊有勾號：

    [![連線的處理站解決方案加速器伺服器樹狀目](./media/iot-accelerators-connected-factory-dashboard/servertree-inline.png)](./media/iot-accelerators-connected-factory-dashboard/servertree-expanded.png#lightbox)

1. 以滑鼠右鍵按一下要讀取、寫入、發佈或呼叫該節點的項目。 您可使用的動作取決於您的節點權限和屬性。 讀取選項會顯示一個內容面板，以顯示特定節點的值。 寫入選項會顯示一個內容面板，您可以在此輸入新值。 呼叫選項會顯示一個節點，您可以在此輸入呼叫的參數。

## <a name="publish-a-node"></a>發佈節點

當您瀏覽*模擬的 OPC UA 伺服器*時，也可以選擇發佈新的節點。 您可以分析解決方案中這些節點的遙測資料。 這些「模擬的 OPC UA 伺服器」可讓您輕鬆對解決方案加速器進行實驗，而不需要部署真實的裝置：

1. 瀏覽至 OPC UA 伺服器瀏覽器樹狀目錄中您想要發佈的節點。

1. 以滑鼠右鍵按一下節點。 按一下 [發佈]：

    [![連線的處理站解決方案加速器發佈節點](./media/iot-accelerators-connected-factory-dashboard/publishnode-inline.png)](./media/iot-accelerators-connected-factory-dashboard/publishnode-expanded.png#lightbox)

1. 隨即顯示一個內容面板，告訴您發佈成功。 此節點會出現在站區層級檢視中，且旁邊有核取記號：

    [![連線的處理站解決方案加速器發佈成功](./media/iot-accelerators-connected-factory-dashboard/publishsuccess-inline.png)](./media/iot-accelerators-connected-factory-dashboard/publishsuccess-expanded.png#lightbox)

## <a name="command-and-control"></a>命令與控制

連線的處理站可讓您直接從雲端指揮及控制您的企業裝置。 您可以使用這項功能來回應裝置所產生的警示。 例如，您可以將命令傳送到裝置來打開壓力釋放閥。 您可以在 OPC UA 伺服器瀏覽器樹狀目錄的 **StationCommands** 節點中找到可用的命令。 在此情節中，您會開啟慕尼黑生產線的裝配站上的洩壓閥。 若要使用命令與控制功能，您必須具備解決方案加速器部署的 [系統管理員] 角色：

1. 瀏覽至 OPC UA 瀏覽器樹狀結構中的 **StationCommands** 節點，找出慕尼黑、生產線 0、裝配站。

1. 選擇您想要使用的命令。 在 **OpenPressureReleaseValve** 節點上按滑鼠右鍵。 按一下 [呼叫]：

    [![連線的處理站解決方案加速器呼叫命令](./media/iot-accelerators-connected-factory-dashboard/callcommand-inline.png)](./media/iot-accelerators-connected-factory-dashboard/callcommand-expanded.png#lightbox)

1. 隨即顯示一個內容面板，告知您即將呼叫的方法以及參數詳細資料。 按一下 [呼叫]：

    [![連線的處理站解決方案加速器呼叫參數](./media/iot-accelerators-connected-factory-dashboard/callpanel-inline.png)](./media/iot-accelerators-connected-factory-dashboard/callpanel-expanded.png#lightbox)

1. 內容面板會進行更新，告知您方法呼叫成功。 您可以讀取因為呼叫而更新的壓力節點值，藉此確認呼叫成功。

    [![連線的處理站解決方案加速器呼叫成功](./media/iot-accelerators-connected-factory-dashboard/callsuccess-inline.png)](./media/iot-accelerators-connected-factory-dashboard/callsuccess-expanded.png#lightbox)

## <a name="behind-the-scenes"></a>在幕後

當您部署解決方案加速器時，部署程序會在您選取的 Azure 訂用帳戶中建立多個資源。 您可以在 Azure[入口網站](https://portal.azure.com)中檢視這些資源。 部署程序會建立 **資源群組** ，其名稱是以您為解決方案加速器選擇的名稱為基礎︰

[![連線的處理站解決方案加速器資源群組](./media/iot-accelerators-connected-factory-dashboard/resourcegroup-inline.png)](./media/iot-accelerators-connected-factory-dashboard/resourcegroup-expanded.png#lightbox)

選取資源群組中的資源清單中的資源，即可檢視該資源的設定。

您也可以在 [azure-iot-connected-factory ](https://github.com/Azure/azure-iot-connected-factory) GitHub 存放庫中查看解決方案加速器的原始程式碼。

完成之後，您可以從 [azureiotsolutions.com](https://www.azureiotsolutions.com/Accelerators#dashboard) 網站上的 Azure 訂用帳戶中刪除解決方案加速器。 這個網站可讓您輕鬆刪除在建立解決方案加速器時已佈建的所有資源。

> [!NOTE]
> 若要確保您會刪除與解決方案加速器相關的所有項目，請在 [azureiotsolutions.com](https://www.azureiotsolutions.com/Accelerators#dashboard) 網站上進行刪除。 請勿刪除入口網站中的資源群組。

## <a name="next-steps"></a>後續步驟

您現已部署運作中的解決方案加速器，您可以繼續閱讀下列文章，了解如何開始使用 IoT 解決方案加速器︰

* [連線的處理站解決方案加速器的逐步解說](iot-accelerators-connected-factory-sample-walkthrough.md)
* [將裝置連線至連線的處理站解決方案加速器](iot-accelerators-connected-factory-gateway-deployment.md)
* [azureiotsolutions.com 網站的權限](iot-accelerators-permissions.md)
