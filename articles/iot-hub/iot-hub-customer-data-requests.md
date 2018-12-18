---
title: 客戶資料要求功能
description: 客戶資料要求功能的摘要
author: dominicbetts
ms.author: dobett
manager: timlt
ms.date: 05/16/2018
ms.topic: conceptual
ms.service: iot-hub
services: iot-hub
ms.openlocfilehash: 862ed2b38521cac3a4c83a9dc90db78f9f030ff1
ms.sourcegitcommit: f31bfb398430ed7d66a85c7ca1f1cc9943656678
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47451811"
---
# <a name="summary-of-customer-data-request-features"></a>客戶資料要求功能的摘要

Azure IoT 中樞是 REST API 型的雲端服務，目標是企業客戶，它能夠實現數百萬個裝置與資料分割 Azure 服務之間的安全、雙向通訊。

[!INCLUDE [gdpr-related guidance](../../includes/gdpr-intro-sentence.md)]

個別裝置會由租用戶系統管理員指派裝置識別碼 (裝置 ID)。 裝置資料是根據指派的裝置識別碼。 Microsoft 不會保留任何資訊，且無權存取可讓裝置識別碼與使用者相互關聯的任何資料。

在 Azure IoT 中樞中受控的許多裝置都不是個人裝置，例如辦公室恆溫器或工廠機器人。 但是，客戶可能會認為某些裝置能夠識別個人身分，而自行決定要維護本身的資產或將裝置與個人繫結的庫存追蹤方法。 Azure IoT 中樞可管理和儲存與裝置相關聯的所有資料，如同處理個人資料一樣。

租用戶系統管理員可以使用 Azure 入口網站或服務的 REST API，藉由匯出或刪除與裝置識別碼相關聯的資料，來履行資訊要求。

如果您使用 Azure IoT 中樞服務的路由功能，將裝置訊息轉送到其他服務，則必須由租用戶系統管理員為每個路由端點執行資料要求，才能完成指定裝置的完整要求。 如需詳細資訊，請參閱每個端點的參考文件。 如需支援端點的詳細資訊，請參閱[參考 - IoT 中樞端點](iot-hub-devguide-endpoints.md)。

如果您使用 Azure IoT 中樞服務的 Azure 事件方格整合功能，則必須由租用戶系統管理員為這些事件的每位訂閱者執行資料要求。 如需詳細資訊，請參閱[使用事件方格以回應 IoT 中樞事件](iot-hub-event-grid.md)。

如果您使用 Azure IoT 中樞服務的 Azure 監視器整合功能來建立診斷記錄，則必須由租用戶系統管理員針對儲存的記錄執行資料要求。 如需詳細資訊，請參閱[監視 Azure IoT 中樞的健康情況](iot-hub-monitor-resource-health.md)。

## <a name="deleting-customer-data"></a>刪除客戶資料

租用戶系統管理員可以利用 Azure 入口網站中，Azure IoT 中樞擴充功能的 IoT 裝置刀鋒視窗來刪除裝置，這樣會刪除與該裝置相關聯的資料。

也可以使用 REST API 執行裝置的刪除作業。 如需詳細資訊，請參閱[服務 - 刪除裝置](/rest/api/iothub/service/deletedevice)。

## <a name="exporting-customer-data"></a>匯出客戶資料

租用戶系統管理員可以利用 Azure 入口網站中，Azure IoT 中樞擴充功能的 IoT 裝置刀鋒視窗內的複製和貼上，來匯出與裝置相關聯的資料。

也可以使用 REST API 執行裝置的匯出作業。 如需詳細資訊，請參閱[服務 - 取得裝置](/rest/api/iothub/service/getdevice)。

> [!NOTE]
> 當您使用 Microsoft 的企業服務時，Microsoft 會產生一些資訊，稱為系統所產生的記錄。 租用戶系統管理員無法存取或匯出部分 Azure IoT 中樞系統所產生的記錄。 這些記錄可視為在與個人裝置相關的服務和診斷資料中，所進行的實際動作。

## <a name="links-to-additional-documentation"></a>其他文件的連結

Azure IoT 中樞服務 API 的完整文件可於 [IoT 中樞服務 API](https://docs.microsoft.com/rest/api/iothub/service) 中找到。
