---
title: 使用 IoT 中樞 C SDK 為受限裝置開發 Azure IoT 中樞 | Microsoft Docs
description: 開發人員指南 - 有關如何使用 Azure IoT SDK 開發受限裝置的指引。
author: yzhong94
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/24/2018
ms.author: yizhon
ms.openlocfilehash: 683f3ca88c349fef31f9647566dbed8a840f94dd
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451726"
---
# <a name="develop-for-constrained-devices-using-azure-iot-c-sdk"></a>使用 Azure IoT C SDK 開發受限裝置

Azure IoT 中樞 C SDK 是以 ANSI C (C99) 撰寫，非常適合於操作具有少量磁碟和記憶體使用量的各種平台。 建議的 RAM 至少為 64 KB，但確切的記憶體使用量取決於所用的通訊協定、開啟的連線數目，以及設為目標的平台。

C SDK 是以 apt-get、NuGet 和 MBED 的套件形式提供。 若要以受限裝置為目標，您可以在本機為您的目標平台建置 SDK。 本文件會示範如何使用 [cmake](https://cmake.org/) 來移除特定功能，以縮減 C SDK 的使用量。 此外，本文件會討論適用於受限裝置的最佳做法程式設計模型。

## <a name="building-the-c-sdk-for-constrained-devices"></a>為受限裝置建置 C SDK

為受限裝置建置 C SDK。

### <a name="prerequisites"></a>必要條件

遵循此 [C SDK 設定指南](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md)，準備您的開發環境以供建置 C SDK。 在您開始進行使用 cmake 建置的步驟之前，您可以叫用 cmake 旗標來移除未使用的功能。

### <a name="remove-additional-protocol-libraries"></a>移除額外的通訊協定程式庫

C SDK 目前支援五個通訊協定：MQTT、MQTT over WebSocket、AMQPs、AMQP over WebSocket 和 HTTPS。 大部分的案例都要求有一到兩個通訊協定在用戶端上執行，因此您可以從 SDK 移除您未使用的通訊協定程式庫。 您可以在[選擇 IoT 中樞通訊的通訊協定](iot-hub-devguide-protocols.md)中，找到有關針對您案例選擇適當通訊協定的其他資訊。 例如，MQTT 是輕量型通訊協定，通常比較適合於受限裝置。

您可以使用下列 cmake 命令來移除 AMQP 和 HTTP 程式庫：

```
cmake -Duse_amqp=OFF -Duse_http=OFF <Path_to_cmake>
```

### <a name="remove-sdk-logging-capability"></a>移除 SDK 記錄功能

C SDK 全面提供廣泛記錄來協助進行偵錯。 您可以使用下列 cmake 命令來移除生產裝置的記錄功能：

```
cmake -Dno_logging=OFF <Path_to_cmake>
```

### <a name="remove-upload-to-blob-capability"></a>移除上傳到 blob 功能

您可以使用 SDK 中的內建功能，將大型檔案上傳至 Azure 儲存體。 Azure IoT 中樞可作為相關聯 Azure 儲存體帳戶的發送器。 您可以使用這項功能來傳送媒體檔案、大型遙測批次和記錄。 您可以在[使用 IoT 中樞上傳檔案](iot-hub-devguide-file-upload.md)中取得詳細資訊。 如果您的應用程式不需要這項功能，您可以使用下列 cmake 命令來移除這項功能：

```
cmake -Ddont_use_uploadtoblob=ON <Path_to_cmake>
```

### <a name="running-strip-on-linux-environment"></a>在 Linux 環境上執行 strip

如果您的二進位檔是在 Linux 系統上執行，您可以利用 [strip 命令](https://en.wikipedia.org/wiki/Strip_(Unix))來減少編譯後的最終應用程式大小。

```
strip -s <Path_to_executable>
```

## <a name="programming-models-for-constrained-devices"></a>受限裝置的程式設計模型

接下來，查看受限裝置的程式設計模型。

### <a name="avoid-using-the-serializer"></a>避免使用序列化程式

C SDK 具有選用 [C SDK 序列化程式](https://github.com/Azure/azure-iot-sdk-c/tree/master/serializer)，可讓您使用宣告式對應資料表來定義方法和裝置對應項屬性。 此序列化程式設計用來簡化開發作業，但是會增加額外負荷，所以不是受限裝置的最佳選擇。 在此情況下，請考慮使用基本用戶端 API，並使用 [parson](https://github.com/kgabis/parson) 等輕量型剖析器來剖析 JSON。

### <a name="use-the-lower-layer-ll"></a>使用較低層級 (_LL_)

C SDK 支援兩種程式設計模型。 一組具有 _LL_ 中置詞 (代表較低層級) 的 API。 這組 API 為較輕量級，而且不會啟動背景工作執行緒，這表示使用者必須手動控制排程。 例如，對於裝置用戶端，在此[標頭檔](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/inc/iothub_device_client_ll.h)中可以找到 _LL_ API。 

另一組不具 _LL_ 索引的 API 稱為方便層，背景工作執行緒會在其中自動啟動。 例如，在此 [IoT 裝置用戶端標頭檔](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/inc/iothub_device_client.h)中可以找到裝置用戶端的方便層 API。 對於每個額外執行緒可能都需使用大量系統資源的受限裝置，請考慮使用 _LL_ API。

## <a name="next-steps"></a>後續步驟

若要深入了解 Azure IoT C SDK 架構：
-   [Azure IoT C SDK 原始程式碼](https://github.com/Azure/azure-iot-sdk-c/)
-   [C 適用的 Azure IoT 裝置 SDK 簡介](iot-hub-device-sdk-c-intro.md)
