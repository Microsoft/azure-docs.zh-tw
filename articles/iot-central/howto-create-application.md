---
title: 建立 Azure IoT Central 應用程式 | Microsoft Docs
description: 了解如何以系統管理員的身分建立 Azure IoT Central 應用程式。
services: iot-central
ms.service: iot-central
author: tbhagwat3
ms.author: tanmayb
ms.date: 07/09/2018
ms.topic: conceptual
manager: peterpr
ms.openlocfilehash: 1fbe3ea142e1dd738cd341f57d2b8f48b539ac75
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39003761"
---
# <a name="create-your-azure-iot-central-application"></a>建立 Azure IoT 中心應用程式

從[建立應用程式](https://apps.microsoftiotcentral.com/create)頁面建立您的 Microsoft Azure IoT 中心應用程式。 若要建立 Azure IoT 中心應用程式，您必須完成這個頁面上的所有欄位，然後選擇 [建立]。 下列包含每個欄位的詳細資訊。

![建立應用程式頁面](media\howto-create-application\image1.png)

## <a name="payment-plan"></a>付款計劃

您可以建立試用或付費應用程式。 在 [Azure IoT Central 定價頁面](https://azure.microsoft.com/pricing/details/iot-central/)深入了解試用和付費應用程式。

## <a name="application-name"></a>應用程式名稱

您的應用程式名稱會顯示在 [應用程式管理員] 頁面上以及每個 Azure IoT 中心應用程式內。 您可以為 Azure IoT 中心應用程式選擇任何名稱。 選擇對您和組織中其他人有意義的名稱。

## <a name="application-url"></a>應用程式 URL

應用程式的 URL 是您應用程式的 URL。 您可以將其書籤儲存在您的瀏覽器中，或與他人共用 URL。

當您輸入您的應用程式名稱時，就會自動產生您的應用程式 URL。 如果您想要，可以為您的應用程式選擇不同的 URL。 在 Azure IoT Central 中，每個 Azure IoT Central URL 都必須是唯一的。 如果您選擇的 URL 已被採用，則會收到錯誤訊息。

## <a name="directory"></a>目錄

僅限於付費應用程式。

選擇 Azure Active Directory 租用戶以建立 Azure IoT 中心應用程式。 Azure Active Directory 租用戶包含使用者身分識別、認證和其他組織資訊。 多個 Azure 訂用帳戶可與單一 Azure Active Directory 租用戶產生關聯。

如果您沒有 Azure Active Directory 租用戶，系統會在您建立 Azure 訂用帳戶時為您建立一個租用戶。

若要深入了解，請參閱 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/)。

## <a name="azure-subscription"></a>Azure 訂用帳戶

Azure 訂用帳戶可讓您建立 Azure 服務執行個體。 Azure IoT 中心會自動尋找您可以存取的所有 Azure 訂用帳戶，並顯示於 [建立應用程式] 頁面的下拉式清單中。 選擇 Azure 訂用帳戶，以建立新的 Azure IoT 中心應用程式。

如果您沒有 Azure 訂用帳戶，您可以在 [Azure 註冊頁面](https://aka.ms/createazuresubscription)上建立一個。 建立 Azure 訂用帳戶之後，請瀏覽回到 [建立應用程式] 頁面。 新的訂用帳戶會出現在 [Azure 訂用帳戶] 下拉式清單中。

若要深入了解，請參閱 [Azure 訂用帳戶](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing)。

## <a name="region"></a>區域

僅限於付費應用程式。

選取您要建立 Azure IoT 中心應用程式的區域。 一般而言，您應選擇實際上最接近您裝置的區域，以取得最佳效能。

若要深入了解，請參閱 [Azure 區域](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#azure-regions)。

您可以在[依區域提供的產品](https://azure.microsoft.com/regions/services/)頁面上，看到可以使用 Azure IoT 中心的區域。

> [!Note]
> 選擇區域後，之後便無法將應用程式移到不同的區域。

## <a name="application-template"></a>應用程式範本

您可以為新的 Azure IoT 中心應用程式，選擇其中一個可用的應用程式範本。 應用程式範本可以包含預先定義的項目 (例如裝置範本和儀表板)，協助您開始使用。

| 應用程式範本 | 說明 |
| -------------------- | ----------- |
| 自訂應用程式   | 為您建立空的應用程式，以填入您自己的裝置範本和裝置。 |
| Contoso 範例       | 建立一個應用程式，其中包含簡單連線裝置的裝置範本。 使用此範本來開始探索 Azure IoT 中心。 |
| Devkits 範例       | 使用您可使用的裝置範本來建立應用程式，以連線 MXChip 或 Raspberry Pi 裝置。 如果您是會在其中一個裝置上試驗程式碼的裝置開發人員，請使用此範本。 |

## <a name="next-steps"></a>後續步驟

您現在已了解如何建立 Azure IoT Central 應用程式，以下是建議的後續步驟：

> [!div class="nextstepaction"]
> [管理您的應用程式](howto-administer.md)