---
title: 以 CSP 的身分建立和管理 Azure IoT Central 應用程式 | Microsoft Docs
description: 身為 CSP，如何代表您的客戶建立 Azure IoT Central 應用程式。
services: iot-central
ms.service: iot-central
author: tbhagwat3
ms.author: tanmayb
ms.date: 07/09/2018
ms.topic: conceptual
manager: peterpr
ms.openlocfilehash: d32e05e99543b30ee92ea455ae2f800b09d83661
ms.sourcegitcommit: e0a678acb0dc928e5c5edde3ca04e6854eb05ea6
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/13/2018
ms.locfileid: "39012106"
---
# <a name="as-a-csp-create-and-manage-an-azure-iot-central-application-on-behalf-of-your-customer"></a>以 CSP 的身分，代表您的客戶建立和管理 Azure IoT Central 應用程式 

雲端解決方案提供者 (CSP) 方案是 Microsoft 轉銷商方案。 其用意是要為我們的通路合作夥伴提供轉銷所有 Microsoft 商務線上服務的單站式方案。 深入了解[雲端解決方案提供者方案](https://partner.microsoft.com/cloud-solution-provider)。

身為 CSP，您可以透過 [Microsoft 合作夥伴中心](https://partnercenter.microsoft.com/partner/home)代表您的客戶建立和管理 Microsoft Azure IoT Central 應用程式。 當 CSP 代表客戶建立 Azure IoT Central 應用程式時，就像其他 CSP 管理的 Azure 服務一樣，CSP 會管理客戶的帳單。 Azure IoT Central 的費用將會在 Microsoft 合作夥伴中心出現於您的總計費中。

若要開始使用，請在 Microsoft 合作夥伴入口網站上登入您的帳戶，並選取您要為其建立 Azure IoT Central 應用程式的客戶。 從左側導覽區瀏覽至客戶的服務管理。

![Microsoft 夥伴中心客戶、客戶檢視](media\howto-create-application-asCSP\image1.png)

Azure IoT Central 會列為可供系統管理員使用的服務。 按一下頁面上的 Azure IoT Central 連結，可為此客戶建立新的應用程式或管理現有的應用程式。

![可用來管理的 Azure IoT Central](media\howto-create-application-asCSP\image2.png)

您會進入 Azure IoT Central 的 [應用程式管理員] 頁面。 Azure IoT Central 會保留您來自 Microsoft 合作夥伴中心和您前往管理該客戶的內容。 您會在 [應用程式管理員] 頁面的標頭中看到對此內容的確認。 從這裡，您可以瀏覽至您先前為此客戶建立的現有應用程式以進行管理，或為客戶建立新的應用程式。

![建立 CSP 的管理員](media\howto-create-application-asCSP\image3.png)

若要建立 Azure IoT Central 應用程式，請按一下 [新增應用程式] 圖格。 這會載入 [應用程式建立] 頁面。 您必須完成此頁面上的所有欄位，然後選擇 [建立]。 您可以在下方找到更多關於每個欄位的詳細資訊。

![建立 CSP 的應用程式頁面](media\howto-create-application-asCSP\image4-1.png)

![建立 CSP 的應用程式頁面](media\howto-create-application-asCSP\image4-2.png)

## <a name="payment-plan"></a>付款計劃

以 CSP 的身分，您只能建立付費應用程式。 若要為客戶展示 Azure IoT Central，您可以個別建立試用應用程式。 您可以在 [Azure IoT Central 定價頁面](https://azure.microsoft.com/pricing/details/iot-central/)深入了解試用和付費應用程式。

## <a name="application-name"></a>應用程式名稱

您的應用程式名稱會顯示在 [應用程式管理員] 頁面上以及每個 Azure IoT 中心應用程式內。 您可以為 Azure IoT 中心應用程式選擇任何名稱。 選擇對您和組織中其他人有意義的名稱。

## <a name="application-url"></a>應用程式 URL

應用程式的 URL 是您應用程式的 URL。 您可以將其書籤儲存在您的瀏覽器中，或與他人共用 URL。

當您輸入您的應用程式名稱時，就會自動產生您的應用程式 URL。 如果您想要，可以為您的應用程式選擇不同的 URL。 在 Azure IoT Central 中，每個 Azure IoT Central URL 都必須是唯一的。 如果您選擇的 URL 已被採用，則會收到錯誤訊息。

## <a name="directory"></a>目錄

由於 Azure IoT Central 保有您前往管理客戶 (您在 Microsoft 合作夥伴入口網站中所選取) 的內容，因此您在 [目錄] 欄位中只會看到該客戶的 Azure Active Directory 租用戶。 

Azure Active Directory 租用戶包含使用者身分識別、認證和其他組織資訊。 多個 Azure 訂用帳戶可與單一 Azure Active Directory 租用戶產生關聯。

若要深入了解，請參閱 [Azure Active Directory](https://docs.microsoft.com/azure/active-directory/)。

## <a name="azure-subscription"></a>Azure 訂用帳戶

Azure 訂用帳戶可讓您建立 Azure 服務執行個體。 Azure IoT Central 會自動尋找您可以存取的客戶所擁有的所有 Azure 訂用帳戶，並顯示於 [建立應用程式] 頁面的下拉式清單中。 選擇 Azure 訂用帳戶，以建立新的 Azure IoT 中心應用程式。

如果您沒有 Azure 訂用帳戶，您可以在 Microsoft 合作夥伴中心建立一個。 建立 Azure 訂用帳戶之後，請瀏覽回到 [建立應用程式] 頁面。 新的訂用帳戶會出現在 [Azure 訂用帳戶] 下拉式清單中。

若要深入了解，請參閱 [Azure 訂用帳戶](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing)。

## <a name="region"></a>區域

選取您要建立 Azure IoT 中心應用程式的區域。 一般而言，您應選擇實際上最接近您裝置的區域，以取得最佳效能。

若要深入了解，請參閱 [Azure 區域](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#azure-regions)。

您可以在[依區域提供的產品](https://azure.microsoft.com/regions/services/)頁面上，看到可以使用 Azure IoT 中心的區域。

> [!Note]
> 選擇區域後，即無法將應用程式移到不同的區域。

## <a name="application-template"></a>應用程式範本

您可以為新的 Azure IoT 中心應用程式，選擇其中一個可用的應用程式範本。 應用程式範本可包含預先定義的項目 (例如裝置範本和儀表板)，協助您開始使用。

| 應用程式範本 | 說明 |
| -------------------- | ----------- |
| 自訂應用程式   | 為您建立空的應用程式，以填入您自己的裝置範本和裝置。 |
| Contoso 範例       | 建立一個應用程式，其中包含簡單連線裝置的裝置範本。 使用此範本來開始探索 Azure IoT 中心。 |
| Devkits 範例       | 使用您可使用的裝置範本來建立應用程式，以連線 MXChip 或 Raspberry Pi 裝置。 如果您是會在其中一個裝置上試驗程式碼的裝置開發人員，請使用此範本。 |

## <a name="next-steps"></a>後續步驟

您現在已了解如何以 CSP 的身分建立 Azure IoT Central 應用程式，以下是建議的後續步驟：

> [!div class="nextstepaction"]
> [管理您的應用程式](howto-administer.md)