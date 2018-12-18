---
title: Azure Service Fabric 反向 Proxy 診斷 | Microsoft Docs
description: 了解如何在反向 proxy 監視和診斷要求處理。
services: service-fabric
documentationcenter: .net
author: kavyako
manager: vipulm
ms.assetid: ''
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: required
ms.date: 08/08/2017
ms.author: kavyako
ms.openlocfilehash: 662fc124af71c1ce976037a3544f59e3cea54ef0
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/16/2018
ms.locfileid: "34207626"
---
# <a name="monitor-and-diagnose-request-processing-at-the-reverse-proxy"></a>在反向 proxy 監視和診斷要求處理

從 Service Fabric 5.7 版開始，反向 proxy 事件可供收集。 事件可用於兩個通道，第一個通道只有在反向 proxy 的要求處理失敗相關錯誤事件，而第二個通道包含成功和失敗要求項目的詳細資訊事件。

請參閱[收集反向 proxy 事件](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations)，允許從這些通道收集本機和 Azure Service Fabric 叢集中的事件。

## <a name="troubleshoot-using-diagnostics-logs"></a>使用診斷記錄進行疑難排解
下列範例有關如何解釋可能看見的常見失敗記錄：

1. 反向 proxy 傳回回應狀態碼 504 (逾時)。

    其中一個原因可能是因為服務無法在要求逾時期間內回覆。
下面第一個事件記錄在反向 proxy 收到的要求詳細資料。 第二個事件指出要求在轉寄至服務時失敗，因為「內部錯誤 = ERROR_WINHTTP_TIMEOUT」 

    承載包括︰

    *  **traceId**：此 GUID 可以讓對應至單一要求的所有事件相互關聯。 在以下兩個事件中，traceId = **2f87b722-e254-4ac2-a802-fd315c1a0271**，意味著它們屬於相同的要求。
    *  **requestUrl**：要求所要送往的 URL (反向 proxy URL)。
    *  **指令動詞**︰HTTP 指令動詞。
    *  **remoteAddress**：傳送要求之用戶端的位址。
    *  **resolvedServiceUrl**：傳入要求解析至的服務端點 URL。 
    *  **errorDetails**︰關於失敗的額外資訊。

    ```
    {
      "Timestamp": "2017-07-20T15:57:59.9871163-07:00",
      "ProviderName": "Microsoft-ServiceFabric",
      "Id": 51477,
      "Message": "2f87b722-e254-4ac2-a802-fd315c1a0271 Request url = https://localhost:19081/LocationApp/LocationFEService?zipcode=98052, verb = GET, remote (client) address = ::1, resolved service url = Https://localhost:8491/LocationApp/?zipcode=98052, request processing start time =     15:58:00.074114 (745,608.196 MSec) ",
      "ProcessId": 57696,
      "Level": "Informational",
      "Keywords": "0x1000000000000021",
      "EventName": "ReverseProxy",
      "ActivityID": null,
      "RelatedActivityID": null,
      "Payload": {
        "traceId": "2f87b722-e254-4ac2-a802-fd315c1a0271",
        "requestUrl": "https://localhost:19081/LocationApp/LocationFEService?zipcode=98052",
        "verb": "GET",
        "remoteAddress": "::1",
        "resolvedServiceUrl": "Https://localhost:8491/LocationApp/?zipcode=98052",
        "requestStartTime": "2017-07-20T15:58:00.0741142-07:00"
      }
    }

    {
      "Timestamp": "2017-07-20T16:00:01.3173605-07:00",
      ...
      "Message": "2f87b722-e254-4ac2-a802-fd315c1a0271 Error while forwarding request to service: response status code = 504, description = Reverse proxy Timeout, phase = FinishSendRequest, internal error = ERROR_WINHTTP_TIMEOUT ",
      ...
      "Payload": {
        "traceId": "2f87b722-e254-4ac2-a802-fd315c1a0271",
        "statusCode": 504,
        "description": "Reverse Proxy Timeout",
        "sendRequestPhase": "FinishSendRequest",
        "errorDetails": "internal error = ERROR_WINHTTP_TIMEOUT"
      }
    }
    ```

2. 反向 proxy 傳回回應狀態碼 404 (找不到)。 
    
    在以下範例事件中，反向 proxy 傳回 404，因為它找不到相符的服務端點。
    重要承載項目如下：
    *  **processRequestPhase**：指出發生失敗時的要求處理期間階段，***TryGetEndpoint*** 也就是 嘗試提取要轉送至的服務端點時。 
    *  **errorDetails**：列出端點搜尋準則。 您可以查看指定的 listenerName = **FrontEndListener**，然而複本端點清單只包含名稱為 **OldListener** 的接聽程式。
    
    ```
    {
      ...
      "Message": "c1cca3b7-f85d-4fef-a162-88af23604343 Error while processing request, cannot forward to service: request url = https://localhost:19081/LocationApp/LocationFEService?ListenerName=FrontEndListener&zipcode=98052, verb = GET, remote (client) address = ::1, request processing start time = 16:43:02.686271 (3,448,220.353 MSec), error = FABRIC_E_ENDPOINT_NOT_FOUND, message = , phase = TryGetEndoint, SecureOnlyMode = false, gateway protocol = https, listenerName = FrontEndListener, replica endpoint = {\"Endpoints\":{\"\":\"Https:\/\/localhost:8491\/LocationApp\/\"}} ",
      "ProcessId": 57696,
      "Level": "Warning",
      "EventName": "ReverseProxy",
      "Payload": {
        "traceId": "c1cca3b7-f85d-4fef-a162-88af23604343",
        "requestUrl": "https://localhost:19081/LocationApp/LocationFEService?ListenerName=NewListener&zipcode=98052",
        ...
        "processRequestPhase": "TryGetEndoint",
        "errorDetails": "SecureOnlyMode = false, gateway protocol = https, listenerName = FrontEndListener, replica endpoint = {\"Endpoints\":{\"OldListener\":\"Https:\/\/localhost:8491\/LocationApp\/\"}}"
      }
    }
    ```
    反向 proxy 可能傳回 404 找不到的另一個範例為：ApplicationGateway\Http 組態參數 **SecureOnlyMode** 設為 true，且反向 proxy 在**HTTPS** 接聽，不過所有複本端點都不安全 (在 HTTP 上接聽)。
    反向 proxy 傳回 404，因為它找不到在 HTTPS 上接聽的端點以轉送要求。 在事件承載中分析參數有助於縮小問題範圍：
    
    ```
        "errorDetails": "SecureOnlyMode = true, gateway protocol = https, listenerName = NewListener, replica endpoint = {\"Endpoints\":{\"OldListener\":\"Http:\/\/localhost:8491\/LocationApp\/\", \"NewListener\":\"Http:\/\/localhost:8492\/LocationApp\/\"}}"
    ```

3. 反向 proxy 的要求因為逾時錯誤而失敗。 
    事件記錄包含具有已接收要求詳細資料 (未顯示於此處) 的事件。
    下一個事件顯示服務以 404 狀態碼回應，且反向 proxy 會起始重新解析。 

    ```
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Request to service returned: status code = 404, status description = , Reresolving ",
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "statusCode": 404,
        "statusDescription": ""
      }
    }
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Re-resolved service url = Https://localhost:8491/LocationApp/?zipcode=98052 ",
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "requestUrl": "Https://localhost:8491/LocationApp/?zipcode=98052"
      }
    }
    ```
    收集所有事件時，您會看到一系列顯示每個解析和轉送嘗試的事件。
    序列中的最後一個事件顯示要求處理因為逾時而失敗，以及成功的解析嘗試次數。
    
    > [!NOTE]
    > 建議依預設停用詳細資訊通道事件收集功能，而後根據需求加以啟用以便進行疑難排解。

    ```
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Error while processing request: number of successful resolve attempts = 12, error = FABRIC_E_TIMEOUT, message = , phase = ResolveServicePartition,  ",
      "EventName": "ReverseProxy",
      ...
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "resolveCount": 12,
        "errorval": -2147017729,
        "errorMessage": "",
        "processRequestPhase": "ResolveServicePartition",
        "errorDetails": ""
      }
    }
    ```
    
    如果只針對重大/錯誤事件啟用收集功能，您會看到一個包含逾時相關詳細資料和解析嘗試次數的事件。 
    
    想要回傳 404 狀態碼給使用者的服務，應在回應中新增 X-ServiceFabric 標頭。 將標頭新增到回應後，反向 proxy 就會將狀態碼轉送回用戶端。  

4. 用戶端已中斷要求連線的情況。

    當反向 proxy 將回應轉送至用戶端，但用戶端中斷連線時，就會記錄下列事件：

    ```
    {
      ...
      "Message": "6e2571a3-14a8-4fc7-93bb-c202c23b50b8 Unable to send response to client: phase = SendResponseHeaders, error = -805306367, internal error = ERROR_SUCCESS ",
      "ProcessId": 57696,
      "Level": "Warning",
      ...
      "EventName": "ReverseProxy",
      "Payload": {
        "traceId": "6e2571a3-14a8-4fc7-93bb-c202c23b50b8",
        "sendResponsePhase": "SendResponseHeaders",
        "errorval": -805306367,
        "winHttpError": "ERROR_SUCCESS"
      }
    }
    ```
5. 反向 Proxy 傳回 404 FABRIC_E_SERVICE_DOES_NOT_EXIST

    如果您未在服務資訊清單中指定服務端點的 URI 配置，就會傳回 FABRIC_E_SERVICE_DOES_NOT_EXIST 錯誤。

    ```
    <Endpoint Name="ServiceEndpointHttp" Port="80" Protocol="http" Type="Input"/>
    ```

    若要解決此問題，請在資訊清單中指定 URI 配置。
    ```
    <Endpoint Name="ServiceEndpointHttp" UriScheme="http" Port="80" Protocol="http" Type="Input"/>
    ```

> [!NOTE]
> 目前不記錄 Websocket 要求處理的相關事件。 這將在下一版中新增。

## <a name="next-steps"></a>後續步驟
* [使用 Windows Azure 診斷的事件彙總和收集](service-fabric-diagnostics-event-aggregation-wad.md)，以便在 Azure 叢集中啟用記錄收集。
* 若要在 Visual Studio 中檢視 Service Fabric 事件，請參閱[在本機上監視及診斷](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)。
* 如需 Azure Resource Manager 範本範例，以便使用不同的服務憑證驗證選項來設定安全反向 Proxy，請參閱[設定反向 Proxy 以連接安全的服務](https://github.com/ChackDan/Service-Fabric/tree/master/ARM%20Templates/ReverseProxySecureSample#configure-reverse-proxy-to-connect-to-secure-services)。
* 如需詳細資訊，請讀取 [Service Fabric 反向 proxy](service-fabric-reverseproxy.md)。
