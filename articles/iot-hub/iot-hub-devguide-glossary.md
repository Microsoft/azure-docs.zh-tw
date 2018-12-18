---
title: Azure IoT 中樞術語詞彙 | Microsoft Docs
description: 開發人員指南 - Azure IoT 中樞相關的常用術語詞彙表。
author: dominicbetts
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 01/29/2018
ms.author: dobett
ms.openlocfilehash: 52aabcfd908afeea3bb9bf36eefad706b74d8ba9
ms.sourcegitcommit: ad08b2db50d63c8f550575d2e7bb9a0852efb12f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/26/2018
ms.locfileid: "47227141"
---
# <a name="glossary-of-iot-hub-terms"></a>IoT 中樞術語詞彙
本文會列出 IoT 中樞文章中使用的一些常見術語。

## <a name="advanced-message-queueing-protocol"></a>進階訊息佇列通訊協定
[進階訊息佇列通訊協定 (AMQP)](https://www.amqp.org/) 是 [IoT 中樞](#iot-hub)支援用來與裝置通訊的其中一種傳訊通訊協定。 如需 IoT 中樞支援的傳訊通訊協定詳細資訊，請參閱[使用 IoT 中樞傳送及接收訊息](iot-hub-devguide-messaging.md)。

## <a name="automatic-device-management"></a>自動裝置管理
Azure IoT 中樞中的自動裝置管理，可將管理大群大型裝置的許多重複且複雜的工作自動化，並且管理範圍橫跨裝置的完整生命週期。 使用自動裝置管理時，您可以根據其屬性以一組裝置為目標，定義所需的設定，並讓 IoT 中樞更新進入範圍的裝置。  包含[自動裝置設定](iot-hub-auto-device-config.md)和 [IoT Edge 自動部署](../iot-edge/how-to-deploy-monitor.md)。

## <a name="automatic-device-configuration"></a>自動裝置設定
解決方案後端可以使用[自動裝置設定](iot-hub-auto-device-config.md)來將所需屬性指派給一組[裝置對應項](#device-twin)，以及使用系統計量與自訂計量來報告狀態。 

## <a name="azure-classic-cli"></a>Azure 傳統 CLI
[Azure 傳統 CLI](../cli-install-nodejs.md) 是跨平台、開放原始碼的命令介面式命令工具，用於建立和管理 Microsoft Azure 中的資源。 此版本的 CLI 應僅用於傳統部署。

## <a name="azure-cli"></a>Azure CLI
[Azure CLI](https://docs.microsoft.com/cli/azure/install-az-cli2) 是跨平台、開放原始碼的命令介面式命令工具，用於建立和管理 Microsoft Azure 中的資源。


## <a name="azure-iot-device-sdks"></a>Azure IoT 裝置 SDK
適用於多種語言的_裝置 SDK_ 可讓您建立[裝置應用程式](#device-app)來與 IoT 中樞互動。 IoT 中樞教學課程示範如何使用這些裝置 SDK。 您可以在此 GitHub [儲存機制](https://github.com/Azure/azure-iot-sdks)中找到原始程式碼和進一步的裝置 SDK 資訊。

## <a name="azure-iot-service-sdks"></a>Azure IoT 服務 SDK
適用於多種語言的_服務 SDK_ 可讓您建立[後端應用程式](#back-end-app) 來與 IoT 中樞互動。 IoT 中樞教學課程示範如何使用這些服務 SDK。 您可以在此 GitHub [儲存機制](https://github.com/Azure/azure-iot-sdks)中找到原始程式碼和進一步的服務 SDK 資訊。

## <a name="azure-iot-toolkit"></a>Azure IoT 工具組
[Azure IoT Toolkit](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit) 是跨平台的開放原始碼 Visual Studio Code 延伸模組，可協助您在 VS Code 中管理 Azure IoT 中樞與裝置。 使用 Azure IoT Toolkit 時，IoT 開發人員可以在 VS Code 中輕鬆地開發 IoT 專案。

## <a name="azure-portal"></a>Azure 入口網站
[Microsoft Azure 入口網站](https://portal.azure.com)是一個可以佈建和管理 Azure 資源的中央位置。 它會使用_刀鋒視窗_來組織其內容。

## <a name="azure-powershell"></a>Azure PowerShell
[Azure PowerShell](/powershell/azure/overview) 是 Cmdlet 集合，可用於透過 Windows PowerShell 管理 Azure。 您可以使用 Cmdlet 來建立、測試、部署和管理透過 Azure 平台傳遞的解決方案和服務。

## <a name="azure-resource-manager"></a>Azure Resource Manager
[Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) 可讓您將方案中的資源做為群組使用。 您可以透過單一、協調的作業來部署、更新或刪除方案的資源。

## <a name="azure-service-bus"></a>Azure 服務匯流排
[服務匯流排](../service-bus/index.md)為企業傳訊及轉送通訊提供雲端通訊的能力，讓您可以連接內部部署解決方案與雲端。 有些 IoT 中樞教學課程會利用服務匯流排[佇列](../service-bus-messaging/service-bus-messaging-overview.md)。

## <a name="azure-storage"></a>Azure 儲存體
[Azure 儲存體](../storage/common/storage-introduction.md)是一套雲端儲存體解決方案。 其中包含可用來儲存非結構化物件資料的 Blob 儲存體服務。 有些 IoT 中樞教學課程會使用 Blob 儲存體。

## <a name="back-end-app"></a>後端應用程式
在 [IoT 中樞](#iot-hub)的內容中，後端應用程式是連接到 IoT 中樞上其中一個服務面向端點的應用程式。 例如，後端應用程式可能會擷取[裝置到雲端](#device-to-cloud)訊息或管理[身分識別登錄](#identity-registry)。 後端應用程式通常會在雲端執行，但在許多教學課程中，後端應用程式是在本機開發電腦上執行的主控台應用程式。

## <a name="built-in-endpoints"></a>內建端點
每個 IoT 中樞都包含與事件中樞相容的內建[端點](iot-hub-devguide-endpoints.md)。 您可以使用可搭配「事件中樞」使用的任何機制從此端點讀取裝置到雲端訊息。

## <a name="cloud-gateway"></a>雲端閘道
雲端閘道可讓無法直接連接到 [IoT 中樞](#iot-hub)的裝置能夠連線。 相對於在裝置本機執行的[現場閘道](#field-gateway)，雲端閘道是裝載於雲端。 雲端閘道的典型使用案例是針對您的裝置實作通訊協定轉譯。

## <a name="cloud-to-device"></a>雲端到裝置
意指從 IoT 中樞傳送到連線裝置的訊息。 這些訊息通常是指示裝置採取行動的命令。 如需詳細資訊，請參閱[使用 IoT 中樞傳送及接收訊息](iot-hub-devguide-messaging.md)。

## <a name="configuration"></a>組態
在[自動裝置設定](iot-hub-auto-device-config.md)的內容中，IoT 中樞內的設定會為一組裝置對應項定義所需的設定，並提供一組計量來報告狀態與進度。

## <a name="connection-string"></a>連接字串
您可以在應用程式程式碼中使用連接字串，以封裝連接至端點所需的資訊。 連接字串通常包含端點位址和安全性資訊，但連接字串的格式會因服務而不同。 IoT 中樞服務有兩種相關的連接字串︰
- 「裝置連接字串」可讓裝置連接至 IoT 中樞上的裝置對向端點。
- 「IoT 中樞連接字串」可讓後端應用程式連接至 IoT 中樞上的服務對向端點。

## <a name="custom-endpoints"></a>自訂端點
您可以在 IoT 中樞上建立自訂[端點](iot-hub-devguide-endpoints.md)，以傳遞由[路由規則](#routing-rules)分派的訊息。 自訂端點會直接連線到事件中樞、服務匯流排佇列或服務匯流排主題。

## <a name="custom-gateway"></a>自訂閘道
閘道可讓無法直接連接到 [IoT 中樞](#iot-hub)的裝置能夠連線。 您可以使用 [Azure IoT Edge](#azure-iot-edge) 來建立自訂閘道，以實作自訂邏輯從而處理 Edge 上的訊息、自訂通訊協定轉換和其他處理。

## <a name="data-point-message"></a>資料點訊息
資料點訊息是[裝置到雲端](#device-to-cloud)的訊息，內含風速或溫度等[遙測](#telemetry)資料。

## <a name="desired-configuration"></a>所需組態
在[裝置對應項](iot-hub-devguide-device-twins.md)的內容中，所需組態是指在應該與裝置同步處理的裝置對應項中的一組完整屬性和中繼資料。

## <a name="desired-properties"></a>所需屬性
在[裝置對應項](iot-hub-devguide-device-twins.md)的內容中，所需屬性是裝置對應項的子區段，可搭配[報告屬性](#reported-properties)使用以便同步處理裝置組態或狀況。 所需屬性只能由[後端應用程式](#back-end-app)設定，並由[裝置應用程式](#device-app)觀察。

## <a name="device-to-cloud"></a>裝置到雲端
意指從連接的裝置傳送到 [IoT 中樞](#iot-hub)的訊息。 這些訊息可能是[資料點](#data-point-message)訊息或[互動式](#interactive-message)訊息。 如需詳細資訊，請參閱[使用 IoT 中樞傳送及接收訊息](iot-hub-devguide-messaging.md)。

## <a name="device"></a>裝置
在 IoT 的內容中，裝置通常是小規模的獨立運算裝置，可能會收集資料或控制其他裝置。 比方說，裝置可能是環境監視裝置，或是溫室中灌溉和換氣系統的控制器。 [裝置目錄](https://catalog.azureiotsuite.com/)會提供被認證可使用 [IoT 中樞](#iot-hub)的硬體裝置清單。

## <a name="device-app"></a>裝置應用程式
裝置應用程式是在[裝置](#device)上執行，可處理與 [IoT 中樞](#iot-hub)的通訊。 當您實作裝置應用程式時，您通常會使用其中一個 [Azure IoT 裝置 SDK](#azure-iot-device-sdks)。 在許多 IoT 教學課程中，您可以使用方便的[模擬裝置](#simulated-device)。

## <a name="device-condition"></a>裝置狀況
意指[裝置應用程式](#device-app)所報告的裝置狀態資訊，例如目前使用中的連線方法。 [裝置應用程式](#device-app)也可以報告其功能。 您可以使用裝置對應項來查詢狀況和功能資訊。

## <a name="device-data"></a>裝置資料
裝置資料是指儲存在 IoT 中樞[身分識別登錄](#identity-registry)中的各裝置資料。 您可以匯入和匯出此資料。

## <a name="device-explorer"></a>裝置總管
[裝置總管](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer)是一種在 Windows 上執行的工具，可讓您管理[身分識別登錄](#identity-registry)中的裝置。此工具也可以傳送和接收裝置的訊息。

## <a name="device-identity"></a>裝置身分識別
裝置識別身分是指派給每個在[身分識別登錄](#identity-registry)中註冊之裝置的唯一識別碼。

## <a name="module-identity"></a>模組身分識別
模組身分識別是針對屬於裝置的每個模組所指派的唯一識別碼。 模組身分識別也會登錄在[身分識別登錄](#identity-registry)中。

## <a name="device-management"></a>裝置管理
裝置管理涵蓋與管理 IoT 方案中的裝置相關聯的完整生命週期，包括規劃、佈建、設定、監視及淘汰。

## <a name="device-management-patterns"></a>裝置管理模式
[IoT 中樞](#iot-hub)可啟用一般裝置管理模式，包括重新開機、執行恢復出廠預設值，以及在裝置上執行韌體更新。

## <a name="device-rest-api"></a>裝置 REST API
您可以使用[裝置 REST API](https://docs.microsoft.com/rest/api/iothub/device)，將裝置到雲端訊息從裝置傳送到 IoT 中樞，以及從 IoT 中樞接收[雲端到裝置](#cloud-to-device)訊息。 您通常應該使用如 IoT 中樞教學課程中所示的其中一個較高層級的[裝置 SDK](#azure-iot-device-sdks)。

## <a name="device-provisioning"></a>裝置佈建
裝置佈建是將初始[裝置資料](#device-data)新增至解決方案中存放區的程序。 若要讓新裝置可連接到您的中樞，必須將裝置識別碼和金鑰新增至 IoT 中樞[身分識別登錄](#identity-registry)。 做為佈建程序的一部分，您可能需要初始化其他解決方案存放區中的裝置特定資料。

## <a name="device-twin"></a>裝置對應項
[裝置對應項](iot-hub-devguide-device-twins.md)是存放裝置狀態資訊 (例如中繼資料、組態和狀況) 的 JSON 文件。 [IoT 中樞](#iot-hub)會為您在 IoT 中樞佈建的每個裝置保存裝置對應項。 裝置對應項可讓您同步處理裝置與解決方案後端之間的[裝置狀況](#device-condition)和組態。 您可以查詢裝置對應項，以找出特定裝置和查詢長時間執行作業的狀態。

## <a name="module-twin"></a>模組對應項
和裝置對應項類似，模組對應項是存放模組狀態資訊 (例如中繼資料、組態和狀況) 的 JSON 文件。 IoT 中樞會為您在 IoT 中樞的裝置身分識別下所佈建的每個模組身分識別，都存留一個模組對應項。 模組對應項可讓您同步處理模組與解決方案後端之間的模組條件和設定。 您可以查詢模組對應項，以找出特定模組和查詢長時間執行作業的狀態。

## <a name="twin-queries"></a>對應項查詢
[裝置與模組對應項查詢](iot-hub-devguide-query-language.md)使用類似 SQL 的 IoT 中樞查詢語言，從裝置對應項或模組對應項擷取資訊。 您可以使用相同的 IoT 中樞查詢語言，來對在 IoT 中樞執行的 [](#job)擷取相關資訊。

## <a name="twin-synchronization"></a>對應項同步處理
對應項同步處理會使用裝置對應項或模組對應項中的[所需屬性](#desired-properties)來設定您的裝置或模組，並從其中擷取[報告屬性](#reported-properties)以儲存在對應項中。

## <a name="direct-method"></a>直接方法
[直接方法](iot-hub-devguide-direct-methods.md)可讓您藉由叫用 IoT 中樞上的 API，觸發方法以在裝置上執行。

## <a name="endpoint"></a>端點
IoT 中樞會公開多個[端點](iot-hub-devguide-endpoints.md)，讓您的應用程式連接到 IoT 中樞。 裝置面向端點可讓裝置執行一些作業，例如傳送[裝置到雲端](#device-to-cloud)訊息和接收[雲端到裝置](#cloud-to-device)訊息。 服務面向管理端點可讓[後端應用程式](#back-end-app)執行一些作業，例如[裝置身分識別](#device-identity)管理和裝置對應項管理。 有可用於讀取裝置到雲端訊息的服務面向[內建端點](#built-in-endpoints)。 您可以建立[自訂端點](#custom-endpoints)，以接收由[路由規則](#routing-rules)分派的裝置到雲端訊息。

## <a name="event-hubs-service"></a>事件中樞服務
[事件中樞](../event-hubs/event-hubs-what-is-event-hubs.md)是可高度調整的資料擷取服務，每秒可以擷取數以百萬計的事件。 此服務可讓您處理及分析由連接之裝置與應用程式所產生的大量資料。 如需與 IoT 中樞服務的比較，請參閱 [Azure IoT 中樞與 Azure 事件中樞的比較](iot-hub-compare-event-hubs.md)。

## <a name="event-hub-compatible-endpoint"></a>事件中樞相容端點
若要讀取傳送到 IoT 中樞的[裝置到雲端](#device-to-cloud)訊息，您可以連接到中樞上的端點，並使用任何事件中樞相容方法讀取這些訊息。 事件中樞相容方法包括使用[事件中樞 SDK](../event-hubs/event-hubs-programming-guide.md) 和 [Azure 串流分析](../stream-analytics/stream-analytics-introduction.md)。

## <a name="field-gateway"></a>現場閘道
現場閘道可讓無法直接連線到 [IoT 中樞](#iot-hub)的裝置獲得連線能力，並且通常會與裝置一起部署在本機。 如需詳細資訊，請參閱[何謂 Azure IoT 中樞？](about-iot-hub.md)

## <a name="free-account"></a>免費帳戶
您可以建立[免費的 Azure 帳戶](https://azure.microsoft.com/pricing/free-trial/)來完成 IoT 中樞教學課程，並且試驗 IoT 中樞服務 (和其他 Azure 服務)。

## <a name="gateway"></a>閘道器
閘道可讓無法直接連接到 [IoT 中樞](#iot-hub)的裝置能夠連線。 另請參閱[現場閘道](#field-gateway)、[雲端閘道](#cloud-gateway)和[自訂閘道](#custom-gateway)。

## <a name="identity-registry"></a>身分識別登錄
[身分識別登錄](iot-hub-devguide-identity-registry.md)是內建的 IoT 中樞元件，可存放允許連線到 IoT 中樞的個別裝置相關資訊。

## <a name="interactive-message"></a>互動式訊息
互動式訊息是[雲端到裝置](#cloud-to-device)訊息，可在解決方案後端觸發立即的行動。 例如，裝置可能會傳送有關失敗的警示，並應該自動記錄到 CRM 系統。

[!INCLUDE [azure-iot-hub-edge-glossary-includes](../../includes/azure-iot-hub-edge-glossary-includes.md)]

## <a name="iot-hub"></a>IoT 中樞
IoT 中樞是一項完全受控的 Azure 服務，可在數百萬個裝置和一個解決方案後端之間啟用可靠且安全的雙向通訊。 如需詳細資訊，請參閱[何謂 Azure IoT 中樞？](about-iot-hub.md) 使用 [Azure 訂用帳戶](#subscription)，您可以建立 IoT 中樞來處理 IoT 傳訊工作負載。

## <a name="iot-hub-metrics"></a>IoT 中樞計量
[IoT 中樞計量](iot-hub-metrics.md)可提供 [Azure 訂用帳戶](#subscription)中 IoT 中樞的狀態相關資料。 IoT 中樞度量可讓您評估服務及其連接之裝置的整體健全狀況。 IoT 中樞度量可協助您了解 IoT 中樞的情況並調查根本問題，而不需要連絡 Azure 支援人員。

## <a name="iot-hub-query-language"></a>IoT 中樞查詢語言
[IoT 中樞查詢語言](iot-hub-devguide-query-language.md)是一種類似 SQL 的語言，可讓您查詢 [](#job) 和裝置對應項。

## <a name="iot-hub-resource-rest-api"></a>IoT 中樞資源 REST API
您可以使用 [IoT 中樞資源 REST API](https://docs.microsoft.com/rest/api/iothub/iothubresource)，管理 [Azure 訂用帳戶](#subscription)中執行建立、更新及刪除中樞等作業的 IoT 中樞。

## <a name="iot-solution-accelerators"></a>IoT 解決方案加速器
Azure IoT 解決方案加速器將多個 Azure 服務封裝在一起成為解決方案。 這些解決方案可讓您快速開始使用常見 IoT 案例的端對端實作。 如需詳細資訊，請參閱[什麼是 Azure IoT 解決方案加速器？](../iot-accelerators/about-iot-accelerators.md)

## <a name="the-iot-extension-for-azure-cli"></a>適用於 Azure CLI 的 IoT 擴充功能 
[適用於 Azure CLI 的 IoT 擴充功能](https://github.com/Azure/azure-iot-cli-extension) \(英文\) 是跨平台的命令列工具。 此工具可讓您管理[身分識別登錄](#identity-registry)中的裝置、傳送及接收裝置的訊息和檔案，以及監視 IoT 中樞作業。

## <a name="job"></a>工作 (Job)
解決方案後端可以使用[作業](iot-hub-devguide-jobs.md)來排程和追蹤已向 IoT 中樞註冊之一組裝置的活動。 這些活動包括更新裝置對應項[所需屬性](#desired-properties)、更新裝置對應項[標籤](#tags)，以及叫用[直接方法](#direct-method)。 [IoT 中樞](#iot-hub)也可使用作業從[身分識別登錄](#identity-registry)[匯入和匯出](iot-hub-devguide-identity-registry.md#import-and-export-device-identities)資料。

## <a name="mqtt"></a>MQTT
[MQTT](http://mqtt.org/) 是 [IoT 中樞](#iot-hub)支援用來與裝置通訊的其中一種傳訊通訊協定。 如需 IoT 中樞支援的傳訊通訊協定詳細資訊，請參閱[使用 IoT 中樞傳送及接收訊息](iot-hub-devguide-messaging.md)。

## <a name="operations-monitoring"></a>作業監視
IoT 中樞的[作業監視](iot-hub-operations-monitoring.md)可讓您即時監視其 IoT 中樞上的作業狀態。 [IoT 中樞](#iot-hub)可追蹤橫跨數個作業類別的事件。 您可以選擇將一或多個類別的事件傳送至 IoT 中樞端點進行處理。 您可以監視資料中是否有錯誤，或根據資料模式設定更複雜的處理行為。

## <a name="physical-device"></a>實體裝置
實體裝置是實際的裝置，例如連接到 IoT 中樞的 Raspberry Pi。 為了方便起見，許多 IoT 中樞教學課程都使用[模擬裝置](#simulated-device)，讓您在本機電腦上執行範例。

## <a name="primary-and-secondary-keys"></a>主要和次要金鑰
當您連接到 IoT 中樞上的裝置面向或服務面向端點時，[連接字串](#connection-string)包含可授與存取權的金鑰。 當您將裝置新增到[身分識別登錄](#identity-registry)或將[共用存取原則](#shared-access-policy)新增到中樞時，服務會產生主要和次要金鑰。 擁有兩個金鑰，可讓您在更新金鑰時從一個金鑰轉換至另一個金鑰，而不會中斷對 IoT 中樞的存取。

## <a name="protocol-gateway"></a>通訊協定閘道
通訊協定閘道通常部署在雲端，可為連線到 [IoT 中樞](#iot-hub)的裝置提供通訊協定轉譯服務。 如需詳細資訊，請參閱[何謂 Azure IoT 中樞？](about-iot-hub.md)

## <a name="quotas-and-throttling"></a>配額和節流
您的 [IoT 中樞](#iot-hub)使用可套用各種[配額](iot-hub-devguide-quotas-throttling.md)，其中有許多配額會因 IoT 中樞的層次而異。 [IoT 中樞](#iot-hub)也會將[節流](iot-hub-devguide-quotas-throttling.md)套用至您在執行階段的服務使用。

## <a name="reported-configuration"></a>報告組態
在[裝置對應項](iot-hub-devguide-device-twins.md)的內容中，報告組態是指在應該向解決方案後端報告的裝置對應項中的一組完整屬性和中繼資料。

## <a name="reported-properties"></a>報告屬性
在[裝置對應項](iot-hub-devguide-device-twins.md)的內容中，報告屬性是裝置對應項的子區段，可搭配[所需屬性](#desired-properties)使用以便同步處理裝置組態或狀況。 報告屬性只能由[裝置應用程式](#device-app)設定，並可供[應用程式後端](#back-end-app)讀取和查詢。

## <a name="resource-group"></a>資源群組
[Azure Resource Manager](#azure-resource-manager) 會使用資源群組將相關的資源群組在一起。 您可以使用資源群組，同時對群組中的所有資源執行作業。

## <a name="retry-policy"></a>重試原則
當您連接到雲端服務時，您可以使用重試原則來處理[暫時性錯誤](/azure/architecture/best-practices/transient-faults)。

## <a name="routing-rules"></a>路由規則
您必須在 IoT 中樞設定[路由規則](iot-hub-devguide-messages-read-custom.md)，以將裝置到雲端訊息路由傳送到[內建端點](#built-in-endpoints)或[自訂端點](#custom-endpoints)，以由您的解決方案後端處理。

## <a name="sasl-plain"></a>SASL PLAIN
SASL PLAIN 是 [AMQP](#advanced-message-queue-protocol) 通訊協定用來傳輸安全性權杖的通訊協定。

## <a name="service-rest-api"></a>服務 REST API
您可以從解決方案後端使用[服務 REST API](https://docs.microsoft.com/rest/api/iothub/service) 來管理您的裝置。 該 API 可讓您擷取及更新[裝置對應項](#device-twin)屬性、叫用[直接方法](#direct-method)，以及排程[作業](#job)。 您通常應該使用如 IoT 中樞教學課程中所示的其中一個較高層級的[服務 SDK](#azure-iot-service-sdks)。

## <a name="shared-access-signature"></a>共用存取簽章
共用存取簽章 (SAS) 是以 SHA-256 安全雜湊或 URI 為基礎的驗證機制。 SAS 驗證有兩個元件：「共用存取原則」和「共用存取簽章」 (通常稱為權杖)。 裝置可使用 SAS 向 IoT 中樞進行驗證。 [後端應用程式](#back-end-app)也可使用 SAS 向 IoT 中樞上的服務面向端點進行驗證。 您通常會將 SAS 權杖包含在[連接字串](#connection-string)中，以便應用程式用於建立連往 IoT 中樞的連線。

## <a name="shared-access-policy"></a>共用存取原則
共用存取原則所定義的權限可授與給任何具有與該原則相關聯之有效[主要或次要金鑰](#primary-and-secondary-keys)的人員。 您可以在[入口網站](#azure-portal)中為您的中樞管理共用存取原則和金鑰。

## <a name="simulated-device"></a>模擬裝置
為了方便起見，許多 IoT 中樞教學課程都使用模擬裝置，讓您在本機電腦上執行範例。 相反地，[實體裝置](#physical-device)是實際的裝置，例如連接到 IoT 中樞的 Raspberry Pi。

## <a name="solution"></a>解決方法
_方案_可意指包含一或多個專案的 Visual Studio 方案。 _方案_也可意指包含裝置、[裝置應用程式](#device-app)、IoT 中樞、其他 Azure 服務和[後端應用程式](#back-end-app)等元素的 IoT 方案。

## <a name="subscription"></a>訂用帳戶
Azure 訂用帳戶是發生帳單的地方。 您建立的每個 Azure 資源，或您使用的 Azure 服務會與單一訂用帳戶相關聯。 許多配額也適用於訂用帳戶層級。

## <a name="system-properties"></a>系統屬性
在[裝置對應項](iot-hub-devguide-device-twins.md)的內容中，系統屬性是唯讀屬性，內含關於裝置使用方式的資訊，例如上次活動時間和連線狀態。

## <a name="tags"></a>標記
在[裝置對應項](iot-hub-devguide-device-twins.md)的內容中，標籤是解決方案後端以 JSON 文件形式儲存和擷取的裝置中繼資料。 裝置上的應用程式看不到標籤。

## <a name="telemetry"></a>遙測
裝置可收集遙測資料 (例如風速或溫度)，並使用[資料點訊息](#data-point-messages)將遙測傳送到 IoT 中樞。

## <a name="token-service"></a>權杖服務
您可以使用權杖服務來實作裝置的驗證機制。 建立具備 **DeviceConnect** 權限的 IoT 中樞[共用存取原則](#shared-access-policy)，以建立「裝置範圍」權杖。 這些權杖可讓裝置連接到 IoT 中樞。 裝置可使用自訂驗證機制來向權杖服務進行驗證。 如果裝置驗證成功，則權杖服務會發出 SAS 權杖以供裝置用來存取您的 IoT 中樞。

## <a name="x509-client-certificate"></a>X.509 用戶端憑證
裝置可以使用 X.509 憑證向 [IoT 中樞](#iot-hub)進行驗證。 使用 X.509 憑證是使用 [SAS 權杖](#shared-access-signature)的替代方式。
