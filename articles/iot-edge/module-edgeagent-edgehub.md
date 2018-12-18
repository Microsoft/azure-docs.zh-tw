---
title: Azure IoT EdgeAgent 和 EdgeHub 參考資訊 | Microsoft Docs
description: 檢閱 edgeAgent 和 edgeHub 模組對應項的特定屬性及其值
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 09/21/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 5e358992661f7bcf06121a07c1bafca0850316b2
ms.sourcegitcommit: 42405ab963df3101ee2a9b26e54240ffa689f140
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47423132"
---
# <a name="properties-of-the-edge-agent-and-edge-hub-module-twins"></a>Edge 代理程式和 Edge 中樞模組對應項的屬性

Edge 代理程式和 Edge 中樞是構成 IoT Edge 執行階段的兩個模組。 如需模組各自執行哪些工作的相關資訊，請參閱[了解 Azure IoT Edge 執行階段及其架構](iot-edge-runtime.md)。 

本文提供執行階段模組對應項的所需屬性和報告屬性。 如需如何在 IoT Edge 裝置上部署模組的詳細資訊，請參閱[部署與監視][lnk-deploy]。

## <a name="edgeagent-desired-properties"></a>EdgeAgent 的所需屬性

Edge 代理程式的模組對應項稱為 `$edgeAgent`，並且會協調裝置與 IoT 中樞上執行之 Edge 代理程式之間的通訊。 在單一裝置或規模部署時，在特定裝置上套用部署資訊清單時，會設定預期屬性。 

| 屬性 | 說明 | 必要 |
| -------- | ----------- | -------- |
| schemaVersion | 必須為「1.0」 | 是 |
| runtime.type | 必須為「docker」 | 是 |
| runtime.settings.minDockerVersion | 此部署資訊清單需要設定為最小 Docker 版本 | 是 |
| runtime.settings.loggingOptions | stringified JSON，包含 Edge 代理程式容器的記錄選項。 [Docker 記錄選項][lnk-docker-logging-options] | 否 |
| runtime.settings.registryCredentials<br>.{registryId}.username | 容器登錄的使用者名稱。 Azure Container Registry 的使用者名稱通常是登錄名稱。<br><br> 非公用的任何模組映像都需要登錄認證。 | 否 |
| runtime.settings.registryCredentials<br>.{registryId}.password | 容器登錄的密碼。 | 否 |
| runtime.settings.registryCredentials<br>.{registryId}.address | 容器登錄的位址。 Azure Container Registry 的位址通常是 *{registryname}.azurecr.io*。 | 否 |  
| systemModules.edgeAgent.type | 必須為「docker」 | 是 |
| systemModules.edgeAgent.settings.image | Edge 代理程式映像的 URI。 目前，Edge 代理程式無法自行更新。 | 是 |
| systemModules.edgeAgent.settings<br>.createOptions | stringified JSON，包含 Edge 代理程式容器的建立選項。 [Docker 建立選項][lnk-docker-create-options] | 否 |
| systemModules.edgeAgent.configuration.id | 部署此模組之部署的識別碼。 | 使用部署套用此資訊清單時，此屬性會由 IoT 中樞進行設定。 非部署資訊清單的一部分。 |
| systemModules.edgeHub.type | 必須為「docker」 | 是 |
| systemModules.edgeHub.status | 必須為「執行中」 | 是 |
| systemModules.edgeHub.restartPolicy | 必須為「永遠」 | 是 |
| systemModules.edgeHub.settings.image | Edge 中樞映像的 URI。 | 是 |
| systemModules.edgeHub.settings<br>.createOptions | stringified JSON，包含 Edge 中樞容器的建立選項。 [Docker 建立選項][lnk-docker-create-options] | 否 |
| systemModules.edgeHub.configuration.id | 部署此模組之部署的識別碼。 | 使用部署套用此資訊清單時，此屬性會由 IoT 中樞進行設定。 非部署資訊清單的一部分。 |
| modules.{moduleId}.version | 使用者定義的字串，表示此模組的版本。 | 是 |
| modules.{moduleId}.type | 必須為「docker」 | 是 |
| modules.{moduleId}.restartPolicy | {"never" \| "on-failed" \| "on-unhealthy" \| "always"} | 是 |
| modules.{moduleId}.settings.image | 模組映像的 URI。 | 是 |
| modules.{moduleId}.settings.createOptions | stringified JSON，包含模組容器的建立選項。 [Docker 建立選項][lnk-docker-create-options] | 否 |
| modules.{moduleId}.configuration.id | 部署此模組之部署的識別碼。 | 使用部署套用此資訊清單時，此屬性會由 IoT 中樞進行設定。 非部署資訊清單的一部分。 |

## <a name="edgeagent-reported-properties"></a>EdgeAgent 的報告屬性

Edge 代理程式報告屬性包含三個主要部分資訊：

1. 上次出現預期屬性之應用程式的狀態；
2. 目前在裝置上執行之模組的狀態；由 Edge 代理程式報告；以及
3. 目前在裝置上執行之預期屬性的複本。

萬一最新的預期屬性未由執行階段成功套用，讓裝置仍在執行先前的部署資訊清單，這個最後一項資訊就很有用。

> [!NOTE]
> 可以使用 [IoT 中樞查詢語言][lnk-iothub-query]查詢，以調查大規模部署的狀態時，Edge 代理程式的報告屬性相當有用。 如需如何將 Edge 代理程式屬性用於狀態的詳細資訊，請參閱[了解單一裝置或大規模的 IoT Edge 部署][lnk-deploy]。

下表不包含從預期屬性複製的資訊。

| 屬性 | 說明 |
| -------- | ----------- |
| lastDesiredVersion | 此整數代表 Edge 代理程式處理之所需屬性的最後版本。 |
| lastDesiredStatus.code | 這是狀態代碼，代表 Edge 代理程式看到的最後預期屬性。 允許的值：`200` 成功、`400` 無效的設定、`412` 無效的結構描述版本、`417` 預期屬性是空的、`500` 失敗 |
| lastDesiredStatus.description | 狀態的文字描述 |
| deviceHealth | 如果所有模組的執行階段狀態為 `running`、`stopped` 時，則為 `healthy`，否則為 `unhealthy` |
| configurationHealth.{deploymentId}.health | 如果部署 {deploymentId} 設定之所有模組的執行階段狀態為 `running` 或 `stopped` 時，則為 `healthy`，否則為 `unhealthy` |
| runtime.platform.OS | 報告裝置上執行的作業系統 |
| runtime.platform.architecture | 報告裝置上的 CPU 架構 |
| systemModules.edgeAgent.runtimeStatus | Edge 代理程式的報告狀態：{"running" \| "unhealthy"} |
| systemModules.edgeAgent.statusDescription | Edge 代理程式之報告狀態的文字描述。 |
| systemModules.edgeHub.runtimeStatus | Edge 中樞的目前狀態：{ "running" \| "stopped" \| "failed" \| "backoff" \| "unhealthy" } |
| systemModules.edgeHub.statusDescription | 狀況不良時，Edge 中樞目前狀態的文字描述。 |
| systemModules.edgeHub.exitCode | 如果結束，Edge 中樞容器報告的結束代碼 |
| systemModules.edgeHub.startTimeUtc | Edge 中樞的上次啟動時間 |
| systemModules.edgeHub.lastExitTimeUtc | Edge 中樞的上次結束時間 |
| systemModules.edgeHub.lastRestartTimeUtc | Edge 中樞的上次重新啟動時間 |
| systemModules.edgeHub.restartCount | 此模組重新啟動的次數，作為重新啟動原則的一部分。 |
| modules.{moduleId}.runtimeStatus | 模組的目前狀態：{ "running" \| "stopped" \| "failed" \| "backoff" \| "unhealthy" } |
| modules.{moduleId}.statusDescription | 狀況不良時，模組目前狀態的文字描述。 |
| modules.{moduleId}.exitCode | 如果結束，模組容器報告的結束代碼 |
| modules.{moduleId}.startTimeUtc | 模組的上次啟動時間 |
| modules.{moduleId}.lastExitTimeUtc | 模組的上次結束時間 |
| modules.{moduleId}.lastRestartTimeUtc | 模組的上次重新啟動時間 |
| modules.{moduleId}.restartCount | 此模組重新啟動的次數，作為重新啟動原則的一部分。 |

## <a name="edgehub-desired-properties"></a>EdgeHub 的所需屬性

Edge 中樞的模組對應項稱為 `$edgeHub`，並且會協調裝置與 IoT 中樞上執行之 Edge 中樞之間的通訊。 在單一裝置或規模部署時，在特定裝置上套用部署資訊清單時，會設定預期屬性。 

| 屬性 | 說明 | 部署資訊清單中的必要項目 |
| -------- | ----------- | -------- |
| schemaVersion | 必須為「1.0」 | 是 |
| routes.{routeName} | 字串，表示 Edge 中樞路由。 | `routes` 元素可以存在但為空白。 |
| storeAndForwardConfiguration.timeToLiveSecs | Edge 中樞在與路由端點中斷連線時保留訊息的時間 (以秒為單位)，例如，與 IoT 中樞或本機模組中斷連線 | 是 |

## <a name="edgehub-reported-properties"></a>EdgeHub 的報告屬性

| 屬性 | 說明 |
| -------- | ----------- |
| lastDesiredVersion | 此整數代表 Edge 中樞處理之所需屬性的最後版本。 |
| lastDesiredStatus.code | 這是狀態代碼，代表 Edge 中樞看到的最後預期屬性。 允許的值：`200` 成功、`400` 無效的設定、`500` 失敗 |
| lastDesiredStatus.description | 狀態的文字描述 |
| clients.{device or moduleId}.status | 此裝置或模組的連線狀態。 可能的值 {"connected" \| "disconnected"}。 只有模組身分識別可以處於中斷連線狀態。 連線到 Edge 中樞的下游裝置只會在連線時顯示。 |
| clients.{device or moduleId}.lastConnectTime | 上一次裝置或模組連線時 |
| clients.{device or moduleId}.lastDisconnectTime | 上一次裝置或模組中斷連線時 |

## <a name="next-steps"></a>後續步驟

若要了解如何使用這些屬性建置部署資訊清單，請參閱[了解如何使用、設定以及重複使用 IoT Edge 模組](module-composition.md)。

<!--links -->
[lnk-deploy]: module-deployment-monitoring.md
[lnk-docker-create-options]: https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate
[lnk-docker-logging-options]: https://docs.docker.com/engine/admin/logging/overview/
[lnk-iothub-query]: ../iot-hub/iot-hub-devguide-query-language.md
