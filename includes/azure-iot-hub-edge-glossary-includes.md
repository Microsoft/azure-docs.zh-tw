## <a name="automatic-device-management"></a>自動裝置管理
Azure IoT 中樞中的自動裝置管理，可將管理大群大型裝置的許多重複且複雜的工作自動化，並且管理範圍橫跨裝置的完整生命週期。 使用自動裝置管理時，您可以根據其屬性以一組裝置為目標，定義所需的設定，並讓 IoT 中樞更新進入範圍的裝置。  包含[自動裝置設定](../articles/iot-hub/iot-hub-auto-device-config.md)和 [IoT Edge 自動部署](../articles/iot-edge/how-to-deploy-monitor.md)。

## <a name="iot-edge"></a>IoT Edge
Azure IoT Edge 能針對內部部署裝置，實現 Azure 服務的雲端驅動部署及解決方案特定程式碼。 IoT Edge 裝置可以在其他裝置的資料被傳送至雲端之前，對這些資料進行彙總來執行計算和分析。 如需詳細資訊，請參閱 [Azure IoT Edge](https://docs.microsoft.com/azure/iot-edge/)。

## <a name="iot-edge-agent"></a>IoT Edge 代理程式
IoT Edge 執行階段負責部署及監視模組的部分。

## <a name="iot-edge-device"></a>IoT Edge 裝置
已安裝 IoT Edge 執行階段，並在裝置詳細資料中標記為 **IoT Edge 裝置**的 IoT Edge 裝置。 了解如何[在 Linux 中的模擬裝置上部署 Azure IoT Edge - 預覽](https://docs.microsoft.com/azure/iot-edge/tutorial-simulate-device-linux)。

## <a name="iot-edge-automatic-deployment"></a>IoT Edge 自動部署
IoT Edge 自動部署會設定一組目標 IoT Edge 裝置，以執行 IoT Edge 模組。 每個部署都會持續確保符合其目標條件的所有裝置都正在執行指定的模組集合，即使在建立新裝置或是將新裝置修改成符合目標條件的情況下也一樣。 每個 IoT Edge 裝置都只會接收符合其條件的最高優先順序部署。 深入了解 [IoT Edge 自動部署](https://docs.microsoft.com/azure/iot-edge/module-deployment-monitoring)。

## <a name="iot-edge-deployment-manifest"></a>IoT Edge 部署資訊清單
JSON 文件，包含要在一個或多個 IoT Edge 裝置之模組對應項中複製的資訊，以部署一組模組、路由及相關模組所需屬性。

## <a name="iot-edge-gateway-device"></a>IoT Edge 閘道裝置
具有下游裝置的 IoT Edge 裝置。 下游裝置可以是 IoT Edge 裝置或不是 IoT Edge 裝置。

## <a name="iot-edge-hub"></a>IoT Edge 中樞
IoT Edge 執行階段負責模組對模組通訊、上游 (朝向 IoT 中樞)，以及下游 (遠離 IoT 中樞) 通訊的部分。 

## <a name="iot-edge-leaf-device"></a>IoT Edge 分葉裝置
不具下游裝置的 IoT Edge 裝置。 

## <a name="iot-edge-module"></a>IoT Edge 模組
IoT Edge 模組是可以部署到 IoT Edge 裝置的 Docker 容器。 它會執行特定的工作，例如從裝置擷取訊息、轉換訊息，或是將訊息傳送至 IoT 中樞。 它會與其他模組通訊，並將資料傳送給 IoT Edge 執行階段。 [了解開發 IoT Edge 模組的需求和工具](https://docs.microsoft.com/azure/iot-edge/module-development)。

## <a name="iot-edge-module-identity"></a>IoT Edge 模組身分識別
位於 IoT 中樞模組身分識別登錄中的記錄，詳述模組向 Edge 中樞或 IoT 中樞驗證時所要使用的存在及安全性認證。

## <a name="iot-edge-module-image"></a>IoT Edge 模組映像
IoT Edge 執行階段用來具現化模組執行個體的 Docker 映像。

## <a name="iot-edge-module-twin"></a>IoT Edge 模組對應項
保存在 IoT 中樞中的 JSON 文件，能夠儲存模組執行個體的狀態資訊。

## <a name="iot-edge-priority"></a>IoT Edge 優先順序
當兩個 IoT Edge 部署都將同一個裝置設為目標時，系統會套用具有較高優先順序的部署。 如果兩個部署的優先順序相同，則系統會套用建立日期較晚的部署。 深入了解[優先順序](https://docs.microsoft.com/azure/iot-edge/module-deployment-monitoring#priority)。

## <a name="iot-edge-runtime"></a>IoT Edge 執行階段
IoT Edge 執行階段包含 Microsoft 散發以安裝於 IoT Edge 裝置上的所有項目。 其中包括 Edge 代理程式、Edge 中樞以及 IoT Edge 安全性精靈。

## <a name="iot-edge-set-modules-to-a-single-device"></a>IoT Edge 將模組設定至單一裝置
複製位於單一裝置模組對應項上之 IoT Edge 資訊清單內容的作業。 基礎 API 是一般的「套用設定」，它會直接將 IoT Edge 資訊清單作為輸入。

## <a name="iot-edge-target-condition"></a>IoT Edge 目標條件
在 IoT Edge 部署中，「目標條件」是裝置對應項的標籤上選取部署目標裝置的任何布林值條件，例如 **tag.environment = prod**。系統會持續評估目標條件以納入任何符合需求的新裝置，或是移除任何不再符合需求的裝置。 深入了解[目標條件](https://docs.microsoft.com/azure/iot-edge/module-deployment-monitoring#target-condition)