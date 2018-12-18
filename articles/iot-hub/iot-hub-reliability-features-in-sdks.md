---
title: 如何使用 Azure IoT 中樞裝置 SDK 來管理連線能力和可靠傳訊
description: 了解如何在使用 Azure IoT 中樞裝置 SDK 時改進裝置的連線能力和傳訊
services: iot-hub
keywords: ''
author: yzhong94
ms.author: yizhon
ms.date: 07/07/2018
ms.topic: article
ms.service: iot-hub
documentationcenter: ''
manager: timlt
ms.devlang: na
ms.custom: mvc
ms.openlocfilehash: f6cfb7bb988b080189da444cb77e0c258417b1ac
ms.sourcegitcommit: e2ea404126bdd990570b4417794d63367a417856
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/14/2018
ms.locfileid: "45574703"
---
# <a name="how-to-manage-connectivity-and-reliable-messaging-using-azure-iot-hub-device-sdks"></a>如何使用 Azure IoT 中樞裝置 SDK 來管理連線能力和可靠傳訊

本指南藉由利用 Azure IoT 裝置 SDK 中的連線能力和可靠傳訊功能，提供設計彈性裝置應用程式的概略指引。 本文的目標是協助回答問題，並處理這些案例：

- 管理已卸除的網路連線
- 管理在不同網路連線之間的切換
- 管理因為服務暫時性連線錯誤的重新連線

實作的詳細資料可能會因語言而異。如需詳細資訊，請參閱連結的 API 文件或特定 SDK。

- [C/Python/iOS SDK](https://github.com/azure/azure-iot-sdk-c)
- [.NET SDK](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/requirements/retrypolicy.md)
- [Java SDK](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)
- [Node SDK](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)


## <a name="designing-for-resiliency"></a>針對復原而設計

IoT 裝置通常仰賴非連續且 (或) 不穩定的網路連線，例如 GSM 或衛星網路。 此外，當與雲端服務互動時，可能因為暫時的狀況而發生錯誤，例如服務斷斷續續和基礎結構層級的錯誤 (通常稱為「暫時性錯誤」)。 在裝置上執行的應用程式需要管理連線和重新連線機制，以及傳送/接收訊息的重試邏輯。 另外，重試策略需求主要取決於裝置所參與的 IoT 案例，以及裝置的內容和功能。

Azure IoT 中樞裝置 SDK 的目標是藉由提供穩固的連線和全面性的 Azure IoT 中樞傳送/接收訊息方式，以簡化雲端到裝置以及裝置到雲端的連線和通訊。 開發人員也可以修改現有的實作，針對特定案例來開發適合的重試策略。

下列各節涵蓋支援連線能力和可靠傳訊的相關 SDK 功能。

## <a name="connection-and-retry"></a>連線和重試

本節提供管理連線時可用的重新連線和重試模式概觀、在裝置應用程式中使用不同重試邏輯的實作指引，以及裝置 SDK 的相關 API。

### <a name="error-patterns"></a>錯誤模式
連線失敗可能會發生在許多層級中：

-  網路錯誤，例如中斷連線的通訊端和名稱解析錯誤
- HTTP、AMQP 和 MQTT 傳輸的通訊協定層級錯誤，例如連結中斷，或是工作階段過期
- 應用程式層級錯誤，例如可能造成本機錯誤的無效認證，或是超出配額或節流的服務行為

裝置 SDK 可偵測這三種層級中的錯誤。  裝置 SDK 不會偵測和處理與 OS 相關聯的錯誤和硬體錯誤。  設計的基礎是來自 Azure Architecture Center 的[暫時性錯誤處理指引](https://docs.microsoft.com/azure/architecture/best-practices/transient-faults#general-guidelines)。

### <a name="retry-patterns"></a>重試模式

當偵測到連線錯誤時，重試的整體程序是： 
1. SDK 偵測到錯誤以及在網路、通訊協定或應用程式中的相關錯誤。
2. 依據錯誤類型而定，SDK 會使用錯誤篩選來決定是否需要執行重試。  如果 SDK 識別出**無法復原的錯誤**，作業 (連線和傳送/接收) 將會停止，而 SDK 將會通知使用者。 無法復原的錯誤是 SDK 可識別並判斷其無法被復原的錯誤，例如驗證錯誤或不正確的端點錯誤。
3. 如果識別出**可復原的錯誤**，SDK 就會使用指定的重試原則進行重試，直到定義的逾時到期為止。
4. 當定義的逾時到期時，SDK 會停止嘗試連線或傳送，並通知使用者。
5.  SDK 可讓使用者附加回呼，以接收連線狀態變更。 

提供三個重試原則：
- **含隨機跳躍的指數後退 (Exponential back-off with jitter)**：這是套用的預設重試原則。  在一開始時會積極重試，然後變慢，接著達到尚未超過的最大延遲。  設計的基礎是 [Azure Architecture Center 的重試指引](https://docs.microsoft.com/azure/architecture/best-practices/retry-service-specific)。
- **自訂重試**：您可以實作自訂重試原則，並將它插入 RetryPolicy (視選擇的語言而定)。 您可以設計適用於本身案例的重試原則。  這並不適用於 C SDK。
- **不重試**：這是將重試原則設為「不重試」的選項，也就是停用重試邏輯。  假設連線已建立，SDK 會嘗試連線一次，然後傳送訊息一次。 此原則通常適用於有頻寬或成本考量的案例。   如果選擇此選項，無法傳送的訊息就會遺失，並且無法復原。 

### <a name="retry-policy-apis"></a>重試原則 API

   | SDK | SetRetryPolicy 方法 | 原則實作 | 實作指引 |
   |-----|----------------------|--|--|
   |  C/Python/iOS  | [IOTHUB_CLIENT_RESULT IoTHubClient_SetRetryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/2018-05-04/iothub_client/inc/iothub_client.h#L188)        | **預設值**：[IOTHUB_CLIENT_RETRY_EXPONENTIAL_BACKOFF](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)<BR>**自訂：** 使用可用的 [retryPolicy](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)<BR>**不重試：**[IOTHUB_CLIENT_RETRY_NONE](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#connection-retry-policies)  | [C/Python/iOS 實作](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/connection_and_messaging_reliability.md#)  |
   | Java| [SetRetryPolicy](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device._device_client_config.setretrypolicy?view=azure-java-stable)        | **預設值**：[ExponentialBackoffWithJitter 類別](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)<BR>**自訂：** 實作 [RetryPolicy 介面](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/RetryPolicy.java)<BR>**不重試**：[NoRetry 類別](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/src/main/java/com/microsoft/azure/sdk/iot/device/transport/NoRetry.java)  | [Java 實作](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md) |[.NET SDK](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/requirements/retrypolicy.md)
   | .NET| [DeviceClient.SetRetryPolicy](/dotnet/api/microsoft.azure.devices.client.deviceclient.setretrypolicy?view=azure-dotnet#Microsoft_Azure_Devices_Client_DeviceClient_SetRetryPolicy_Microsoft_Azure_Devices_Client_IRetryPolicy) | **預設值**：[ExponentialBackoff 類別](/dotnet/api/microsoft.azure.devices.client.exponentialbackoff?view=azure-dotnet)<BR>**自訂**：實作 [IRetryPolicy 介面](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.iretrypolicy?view=azure-dotnet)<BR>**不重試**：[NoRetry 類別](/dotnet/api/microsoft.azure.devices.client.noretry?view=azure-dotnet) | [C# 實作](https://github.com/Azure/azure-iot-sdk-csharp) |
   | 節點| [setRetryPolicy](/javascript/api/azure-iot-device/client?view=azure-iot-typescript-latest#azure_iot_device_Client_setRetryPolicy) | **預設值**：[ExponentialBackoffWithJitter 類別](/javascript/api/azure-iot-common/exponentialbackoffwithjitter?view=azure-iot-typescript-latest)<BR>**自訂：** 實作 [RetryPolicy 介面](/javascript/api/azure-iot-common/retrypolicy?view=azure-iot-typescript-latest)<BR>**不重試**：[NoRetry 類別](/javascript/api/azure-iot-common/noretry?view=azure-iot-typescript-latest) | [Node 實作](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them) |
   

以下是說明此流程的程式碼範例。 

#### <a name="net-implementation-guidance"></a>.NET 實作指引

下列程式碼示範如何定義和設定預設重試原則：

   ```csharp
   # define/set default retry policy
   RetryPolicy retryPolicy = new ExponentialBackoff(int.MaxValue, TimeSpan.FromMilliseconds(100), TimeSpan.FromSeconds(10), TimeSpan.FromMilliseconds(100));
   SetRetryPolicy(retryPolicy);
   ```

為了避免高 CPU 使用量，如果程式碼立即失敗，便會將重試節流 (例如沒有網路或路由的目的地時)，如此一來執行下一次重試的最小時間為 1 秒。 

如果服務回應了節流錯誤，則重試原則不同，且無法透過公用 API 來變更：

   ```csharp
   # throttled retry policy
   RetryPolicy retryPolicy = new ExponentialBackoff(RetryCount, TimeSpan.FromSeconds(10), TimeSpan.FromSeconds(60), TimeSpan.FromSeconds(5));
   SetRetryPolicy(retryPolicy);
   ```

重試機制會在 `DefaultOperationTimeoutInMilliseconds` 之後停止，其目前設為 4 分鐘。

#### <a name="other-languages-implementation-guidance"></a>其他語言的實作指引
針對其他語言，請檢閱以下的實作文件。  存放庫中提供了重試原則 API 用法的示範範例。
- [C/Python/iOS SDK](https://github.com/azure/azure-iot-sdk-c)
- [.NET SDK](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/devdoc/requirements/retrypolicy.md)
- [Java SDK](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-client/devdoc/requirement_docs/com/microsoft/azure/iothub/retryPolicy.md)
- [Node SDK](https://github.com/Azure/azure-iot-sdk-node/wiki/Connectivity-and-Retries#types-of-errors-and-how-to-detect-them)

## <a name="next-steps"></a>後續步驟
- [使用裝置與服務 SDK](.\iot-hub-devguide-sdks.md)
- [使用適用於 C 的 IoT 裝置 SDK](.\iot-hub-device-sdk-c-intro.md)
- [為受限裝置而開發](.\iot-hub-devguide-develop-for-constrained-devices.md)
- [為行動裝置而開發](.\iot-hub-how-to-develop-for-mobile-devices.md)
- [針對裝置連線中斷進行疑難排解](iot-hub-troubleshoot-connectivity.md)
