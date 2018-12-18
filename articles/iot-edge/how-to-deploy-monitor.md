---
title: 部署和監視適用於 Azure IoT Edge 的模組 | Microsoft Docs
description: 管理在 Edge 裝置上執行的模組
keywords: ''
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 07/25/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 28aa2904f63a9802305d24fec1650f84e38601ab
ms.sourcegitcommit: c2c64fc9c24a1f7bd7c6c91be4ba9d64b1543231
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/26/2018
ms.locfileid: "39258428"
---
# <a name="deploy-and-monitor-iot-edge-modules-at-scale-using-the-azure-portal"></a>使用 Azure 入口網站大規模部署和監視 IoT Edge 模組

[!INCLUDE [iot-edge-how-to-deploy-monitor-selector](../../includes/iot-edge-how-to-deploy-monitor-selector.md)]

Azure IoT Edge 可讓您將分析移至 Edge，並提供雲端介面，讓您能夠管理和監視 IoT Edge 裝置，而不需實際存取每一個。 隨著物聯網解決方案越來越大且越複雜，從遠端管理裝置的能力就變得越來越重要。 Azure IoT Edge 是設計來支援您的商務目標，而不論您新增了多少個裝置。

您可以管理個別裝置，且一次一個對其部署模組。 不過，如果您想要大規模地變更裝置，可以建立 **IoT Edge 自動部署**，這是 IoT 中樞內裝置自動管理的一部分。 部署是動態程序，可讓您一次將多個模組部署到多個裝置、追蹤模組的狀態和健康情況，並視需要進行變更。 

## <a name="identify-devices-using-tags"></a>使用標記識別裝置

在您可建立部署之前，必須能夠指定您要影響的裝置。 Azure IoT Edge 會使用裝置對應項中的 **tags** 來識別裝置。 每個裝置都可以有多個標記，而您可以利用任何對您解決方案有意義的方式來定義它們。 例如，如果您管理智慧建築的校園，便可以將下列標記新增至裝置：

```json
"tags":{
    "location":{
        "building": "20",
        "floor": "2"
    },
    "roomtype": "conference",
    "environment": "prod"
}
```

如需裝置對應項和標記的詳細資訊，請參閱[了解和使用 IoT 中樞的裝置對應項][lnk-device-twin]。

## <a name="create-a-deployment"></a>建立部署

1. 在 [Azure 入口網站][lnk-portal]中，移至您的 IoT 中樞。 
1. 選取 [IoT Edge]。
1. 選取 [新增 IoT Edge 部署]。

建立部署有五個步驟。 下列各節將逐步解說每一個步驟。 

### <a name="step-1-name-and-label"></a>步驟 1：名稱和標籤

1. 為部署指定唯一的名稱，最長為 128 個小寫字母。 避免空格和下列無效字元：`& ^ [ ] { } \ | " < > /`。
1. 新增標籤，以協助追蹤您的部署。 標籤是成對的「**名稱**, **值**」，可描述您的部署。 例如，`HostPlatform, Linux` 或 `Version, 3.0.1`。
1. 選取 [下一步] 移至步驟二。 

### <a name="step-2-add-modules-optional"></a>步驟 2：新增模組 (選擇性)

您可以新增至部署的模組有兩種類型。 第一種是以 Azure 服務為基礎的模組，例如儲存體帳戶或串流分析。 第二種是以您自己的程式碼為基礎的模組。 您可以將任一種類型的多個模組新增至部署。 

如果您建立不含模組的部署，它會將任何目前的模組從裝置中移除。 

>[!NOTE]
>Azure Machine Learning 和 Azure Functions 尚未支援自動化的 Azure 服務部署。 請使用自訂模組部署，手動將這些服務新增至您的部署。 

若要從 Azure 串流分析新增模組，請遵循下列步驟：
1. 在頁面的 [部署模組] 區段中，按一下 [新增]。
1. 選取 [Azure 串流分析模組]。
1. 從下拉式功能表中選擇 [訂用帳戶]。
1. 從下拉式功能表中選擇 [Edge 作業]。
1. 選取 [儲存]，將您的模組新增至部署。 

若要新增自訂程式碼作為模組，或手動新增 Azure 服務模組，請遵循下列步驟：
1. 在頁面的 [登錄設定] 區段中，為包含模組映像的私人容器登錄提供此部署的名稱和認證。 Edge 代理程式若找不到 Docker 映像的容器登錄認證，將會報告錯誤 500。
1. 在頁面的 [部署模組] 區段中，按一下 [新增]。
1. 選取 [IoT Edge 模組]。
1. 為模組指定**名稱**。
1. 針對 [映像 URI] 欄位，為您的模組輸入容器映像。 
1. 指定任何應傳遞至容器的**容器建立選項**。 如需詳細資訊，請參閱 [docker create][lnk-docker-create]。
1. 使用下拉式功能表來選取**重新啟動原則**。 從下列選項中選擇： 
   * **永遠**：如果模組基於任何原因而關閉，永遠都會重新啟動。
   * **永不**：如果模組基於任何原因而關閉，永遠都不會重新啟動。
   * **處於失敗狀態**：如果模組損毀，就會重新啟動該模組，但如果它完全關閉，就不會重新啟動。 
   * **處於狀況不良狀態**：如果模組損毀或傳回狀況不良的狀態，就會重新啟動。 每個模組必須負責實作健康情況狀態函式。 
1. 使用下拉功能表來選取模組的**預期狀態**。 從下列選項中選擇：
   * **執行中**：這是預設選項。 模組將會在部署後立即開始執行。
   * **已停止**：部署之後，模組將維持閒置狀態，直到您或另一個模組呼叫來啟動為止。
1. 如果您想要將任何標記或所需屬性新增至模組對應項，選取 [啟用]。 
1. 輸入此模組的 [環境變數]。 環境變數可為模組提供有助於進行設定程序的補充資訊。
1. 選取 [儲存]，將您的模組新增至部署。 

設定部署的所有模組之後，選取 [下一步] 移至步驟三。

### <a name="step-3-specify-routes-optional"></a>步驟 3：指定路由 (選擇性)

路由會定義模組在部署內彼此通訊的方式。 根據預設，精靈會提供您名為 **route** 的路由，並且定義為 **FROM /\* INTO $upstream**，這表示任何模組的任何訊息輸出都會傳送到 IoT 中樞。  

請使用[宣告路由](module-composition.md#declare-routes)中的資訊來新增或更新路由，然後選取 [下一步] 繼續檢閱區段。


### <a name="step-4-target-devices"></a>步驟 4：目標裝置

使用裝置的 tags 屬性，將目標設為應該接收此部署的特定裝置。 

由於多個部署可能會以相同裝置為目標，因此，您應該為每個部署提供優先順序號碼。 如果發生衝突，優先順序最高 (值愈高，優先順序就愈高) 的部署將會勝出。 如果兩個部署具有相同的優先順序號碼，則最新建立的部署獲勝。 

1. 為部署**優先順序**輸入一個正整數。 如果兩個以上部署的目標為相同的裝置，則將會套用 [優先順序] 數值最高的部署。
1. 輸入**目標條件**來判斷這個部署會將哪些裝置設為目標。 條件會以裝置對應項標籤或裝置對應項報告屬性為基礎，且應符合運算式格式。 例如，`tags.environment='test'` 或 `properties.reported.devicemodel='4000x'`。 
1. 選取 [下一步] 移到最後一個步驟。

### <a name="step-5-review-template"></a>步驟 5：檢閱範本

檢閱您的部署資訊，然後選取 [提交]。

## <a name="monitor-a-deployment"></a>監視部署

若要檢視部署的詳細資料並監視正在執行該部署的裝置，請使用下列步驟：

1. 登入 [Azure 入口網站][lnk-portal]，然後瀏覽至 IoT 中樞。 
1. 選取 [IoT Edge]。
1. 選取 [IoT Edge 部署]。 

   ![檢視 IoT Edge 部署][1]

1. 檢查部署清單。 針對每個部署，您可以檢視下列詳細資料：
   * **識別碼**：部署的名稱。
   * **目標條件**：用來定義目標裝置的標記。
   * **優先順序**：指派給部署的優先順序號碼。
   * **系統計量** - [設為目標] 會指定 IoT 中樞內符合目標條件的裝置對應項數目，而 [以套用] 則會指定已在 IoT 中樞內將部署內容套用到至模組對應項的裝置數目。 
   * **裝置計量** - 從 IoT Edge 用戶端執行階段報告成功或錯誤部署中的 Edge 裝置數目。
   * **建立時間**：建立部署時的時間戳記。 當兩個部署具有相同的優先順序時，可使用此時間戳記來中斷繫結。 
2. 選取您想要監視的部署。  
3. 檢查部署詳細資料。 您可以使用索引標籤來檢閱部署的詳細資料。

## <a name="modify-a-deployment"></a>修改部署

當您修改部署時，所做的變更會立即複寫到所有目標裝置。 

如果您更新目標條件，就會發生下列更新：
* 如果裝置不符合舊的目標條件，但符合新的目標條件，而且此部署為該裝置的最高優先順序，則會將此部署套用到裝置。 
* 如果目前執行此部署的裝置不再符合目標條件，它會解除安裝此部署，並採用下一個最高優先順序的部署。 
* 如果目前執行此部署的裝置不再符合目標條件，且不符合任何其他部署的目標條件，則不會在裝置上發生任何變更。 裝置會以模組目前的狀態繼續執行它目前的模組，但不再將其當成此部署的一部分來管理。 一旦它符合任何其他部署的目標條件之後，就會解除安裝此部署，並採用新的部署。 

若要修改部署，請使用下列步驟： 

1. 登入 [Azure 入口網站][lnk-portal]，然後瀏覽至 IoT 中樞。 
1. 選取 [IoT Edge]。
1. 選取 [IoT Edge 部署]。 

   ![檢視 IoT Edge 部署][1]

1. 選取您想要修改的部署。 
1. 對下列欄位進行更新： 
   * 目標條件 
   * 標籤 
   * 優先順序 
1. 選取 [ **儲存**]。
1. 依照[監視部署][anchor-monitor]中的步驟來監看所推出的變更。 

## <a name="delete-a-deployment"></a>刪除部署

當您刪除部署時，所有裝置均會採用其下一個最高優先順序的部署。 如果您的裝置不符合任何其他部署的目標條件，則在刪除部署時不會移除模組。 

1. 登入 [Azure 入口網站][lnk-portal]，然後瀏覽至 IoT 中樞。 
1. 選取 [IoT Edge]。
1. 選取 [IoT Edge 部署]。 

   ![檢視 IoT Edge 部署][1]

1. 使用核取方塊來選取您想要刪除的部署。 
1. 選取 [刪除] 。
1. 提示將會通知您，這個動作將刪除此部署，並將所有裝置還原成先前狀態。  這表示將套用優先順序較低的部署。  如果沒有將其他部署設為目標，將不會移除任何模組。 如果您想要從裝置中移除所有模組，請建立不含模組的部署，並將其部署至相同的裝置。 選取 [是] 以繼續進行。 

## <a name="next-steps"></a>後續步驟

深入了解[將模組部署到 Edge 裝置][lnk-deployments]。

<!-- Images -->
[1]: ./media/how-to-deploy-monitor/iot-edge-deployments.png

<!-- Links -->
[lnk-device-twin]: ../iot-hub/iot-hub-devguide-device-twins.md
[lnk-portal]: https://portal.azure.com
[lnk-docker-create]: https://docs.docker.com/engine/reference/commandline/create/
[lnk-deployments]: module-deployment-monitoring.md

<!-- Anchor links -->
[anchor-monitor]: #monitor-a-deployment
