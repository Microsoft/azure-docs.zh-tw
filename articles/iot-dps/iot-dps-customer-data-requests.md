---
title: 客戶資料要求功能
author: dominicbetts
ms.author: dobett
manager: timlt
ms.date: 05/16/2018
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.openlocfilehash: d6355926c8fac62b01c36d28265842b1233ce213
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38666933"
---
# <a name="summary-of-customer-data-request-features"></a>客戶資料要求功能的摘要

Azure IoT 中樞裝置佈建服務是以企業客戶為目標的 REST API 型雲端服務，可實現對 Azure IoT 中樞的無縫、全自動裝置佈建，從裝置一直到雲端全都能保有安全性。

[!INCLUDE [gdpr-intro-sentence](../../includes/gdpr-intro-sentence.md)]

個別裝置會由租用戶系統管理員指派註冊識別碼和裝置識別碼。 來自這些裝置的相關資料是根據這些識別碼。 Microsoft 不會保留任何資訊，且無權存取可讓這些裝置與使用者相互關聯的任何資料。

在裝置佈建服務中受控的許多裝置都不是個人裝置，例如辦公室恆溫器或工廠機器人。 但是，客戶可能會認為某些裝置能夠識別個人身分，而自行決定要維護本身的資產或將裝置與個人繫結的庫存追蹤方法。 裝置佈建服務可管理和儲存與裝置相關聯的所有資料，如同處理個人資料一樣。

租用戶系統管理員可以使用 Azure 入口網站或服務的 REST API，藉由匯出或刪除與裝置識別碼或註冊識別碼相關聯的資料，來履行資訊要求。

> [!NOTE]
> 透過裝置佈建服務在 Azure IoT 中樞內佈建的裝置，具有儲存在 Azure IoT 中樞服務的額外資料。 請參閱 [Azure IoT 中樞參考文件](../iot-hub/iot-hub-customer-data-requests.md)，以便完成指定裝置的完整要求。

## <a name="deleting-customer-data"></a>刪除客戶資料

裝置佈建服務會儲存註冊和註冊記錄。 註冊包含可允許裝置佈建的相關資訊，而註冊記錄會顯示哪些裝置已經完成佈建程序。

租用戶系統管理員可以從 Azure 入口網站移除註冊，這樣也會移除任何相關聯的註冊記錄。

如需詳細資訊，請參閱[如何管理裝置註冊](how-to-manage-enrollments.md)。

也可以使用 REST API 執行註冊和註冊記錄的刪除作業：

* 若要刪除單一裝置的註冊資訊，您可以使用[裝置註冊 - 刪除](https://docs.microsoft.com/rest/api/iot-dps/deviceenrollment/delete)。
* 若要刪除一組裝置的註冊資訊，您可以使用[裝置註冊群組 - 刪除](https://docs.microsoft.com/rest/api/iot-dps/deviceenrollmentgroup/delete)。
* 若要刪除已佈建裝置的相關資訊，您可以使用[註冊狀態 - 刪除註冊狀態](https://docs.microsoft.com/rest/api/iot-dps/registrationstate/deleteregistrationstate)。

## <a name="exporting-customer-data"></a>匯出客戶資料

裝置佈建服務會儲存註冊和註冊記錄。 註冊包含可允許裝置佈建的相關資訊，而註冊記錄會顯示哪些裝置已經完成佈建程序。

租用戶系統管理員可以透過 Azure 入口網站檢視註冊和註冊記錄，並且使用複製和貼上將它們匯出。

如需如何管理註冊的詳細資訊，請參閱[如何管理裝置註冊](how-to-manage-enrollments.md)。

也可以使用 REST API 執行註冊和註冊記錄的匯出作業：

* 若要匯出單一裝置的註冊資訊，您可以使用[裝置註冊 - 取得](https://docs.microsoft.com/rest/api/iot-dps/deviceenrollment/get)。
* 若要匯出一組裝置的註冊資訊，您可以使用[裝置註冊群組 - 取得](https://docs.microsoft.com/rest/api/iot-dps/deviceenrollmentgroup/get)。
* 若要匯出已佈建裝置的相關資訊，您可以使用[註冊狀態 - 取得註冊狀態](https://docs.microsoft.com/rest/api/iot-dps/registrationstate/getregistrationstate)。

> [!NOTE]
> 當您使用 Microsoft 的企業服務時，Microsoft 會產生一些資訊，稱為系統所產生的記錄。 租用戶系統管理員無法存取或匯出部分裝置佈建服務系統所產生的記錄。 這些記錄可視為在與個人裝置相關的服務和診斷資料中，所進行的實際動作。

## <a name="links-to-additional-documentation"></a>其他文件的連結

完整的裝置佈建服務 API 文件可於 [https://docs.microsoft.com/rest/api/iot-dps](https://docs.microsoft.com/rest/api/iot-dps) 找到。

Azure IoT 中樞[客戶資料要求功能](../iot-hub/iot-hub-customer-data-requests.md)。