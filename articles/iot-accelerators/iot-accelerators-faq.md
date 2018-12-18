---
title: Azure IoT 解決方案加速器的常見問題集 | Microsoft Docs
description: IoT 解決方案加速器的常見問題集
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 02/15/2018
ms.author: dobett
ms.openlocfilehash: 66494ca6bd9b4d8276469e85f4c6bb435ae245f7
ms.sourcegitcommit: c29d7ef9065f960c3079660b139dd6a8348576ce
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "44716395"
---
# <a name="frequently-asked-questions-for-iot-solution-accelerators"></a>IoT 解決方案加速器的常見問題集

另請參閱[連線處理站專屬的常見問題集](iot-accelerators-faq-cf.md)和[遠端監視專屬的常見問題集](iot-accelerators-faq-rm-v2.md)。

### <a name="where-can-i-find-the-source-code-for-the-solution-accelerators"></a>哪裡可以找到解決方案加速器的原始程式碼？

原始程式碼儲存在下列 GitHub 儲存機制中︰

* [遠端監視解決方案加速器 (.NET)](https://github.com/Azure/azure-iot-pcs-remote-monitoring-dotnet) \(英文\)
* [遠端監視解決方案加速器 (Java)](https://github.com/Azure/azure-iot-pcs-remote-monitoring-java) \(英文\)
* [預測性維護解決方案加速器](https://github.com/Azure/azure-iot-predictive-maintenance) \(英文\)
* [連線的處理站解決方案加速器](https://github.com/Azure/azure-iot-connected-factory) \(英文\)

### <a name="what-sdks-can-i-use-to-develop-device-clients-for-the-solution-accelerators"></a>我可以使用哪些 SDK 來為解決方案加速器開發裝置用戶端？

您可以在 [Microsoft Azure IoT SDK](https://github.com/Azure/azure-iot-sdks) GitHub 存放庫中找到不同語言 (C、.NET、Java、Node.js、Python) 的 IoT 裝置 SDK 連結。

如果您使用 DevKit 裝置，您可以在 [IoT DevKit SDK](https://github.com/Microsoft/devkit-sdk) GitHub 存放庫中找到資源和範例。

### <a name="is-the-new-microservices-architecture-available-for-all-the-three-solution-accelerators"></a>新的微服務架構是否適用於這三個解決方案加速器？

目前，只有遠端監視解決方案是使用微服務架構，因為它涵蓋最廣泛的情節。

### <a name="what-advantages-does-the-new-open-sourced-microservices-based-architecture-provide-in-the-new-update"></a>以微服務作為基礎的開放原始碼新基礎架構能在更新中提供哪些優點？

在過去兩年中，雲端架構已大幅發展。 微服務問世以來都是絕佳的模式，可達成縮放及彈性，而不必犧牲開發速度。 此架構模式運用於多項在內部使用的 Microsoft 服務，都獲得絕佳的可靠性與延展性結果。 我們會將這些學習放入實務，以便客戶能夠從中獲益。

### <a name="is-the-new-solution-accelerator-available-in-the-same-geographic-region-as-the-existing-solution"></a>是否可在與現有解決方案相同的地理區域中使用新的解決方案加速器？

是，可在相同的地理區域中使用新的遠端監視。

### <a name="whats-the-difference-between-deleting-a-resource-group-in-the-azure-portal-and-clicking-delete-on-a-solution-accelerator-in-azureiotsuitecom"></a>在 Azure 入口網站中刪除資源群組，以及在 azureiotsuite.com 中對解決方案加速器按一下 [刪除]，兩者間有何差異？

* 如果您在 [azureiotsuite.com](https://www.azureiotsolutions.com/) 中刪除解決方案加速器，就會刪除您在建立該解決方案加速器時所佈建的所有資源。 將額外資源新增到資源群組時，也會一併刪除這些資源。
* 如果您在 [Azure 入口網站](https://portal.azure.com)中刪除資源群組，則只會刪除該資源群組中的資源。 您還必須刪除與解決方案加速器相關聯的 Azure Active Directory 應用程式。

### <a name="can-i-continue-to-leverage-my-existing-investments-in-azure-iot-solution-accelerators"></a>可以繼續利用我在 Azure IoT 解決方案加速器中的現有投資嗎？

是。 目前存在的任何解決方案都會繼續在您的 Azure 訂用帳戶中運作，而且原始碼在 GitHub 中仍保持可用。

### <a name="how-many-iot-hub-instances-can-i-provision-in-a-subscription"></a>我可以在一個訂用帳戶中佈建多少個 IoT 中樞執行個體？

根據預設，您的佈建方式可以是[每個訂用帳戶 10 個 IoT 中樞](../azure-subscription-service-limits.md#iot-hub-limits)。 您可以建立 [Azure 支援票證](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)來提高此限制。 結果就是，由於每個解決方案加速器都會佈建一個新的 IoT 中樞，因此在一個指定的訂用帳戶中，您最多只能佈建 10 個解決方案加速器。

### <a name="how-many-azure-cosmos-db-instances-can-i-provision-in-a-subscription"></a>我可以在一個訂用帳戶中佈建多少個 Azure Cosmos DB 執行個體？

50 個。 您可以建立 [Azure 支援票證](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade)來提高此限制，但每一個訂用帳戶預設只能佈建 50 個 Cosmos DB 執行個體。

### <a name="how-many-free-bing-maps-apis-can-i-provision-in-a-subscription"></a>我可以在一個訂用帳戶中佈建多少個免費 Bing 地圖服務 API？

2 個。 您只可以在 Azure 訂用帳戶中針對 Enterprise 方案建立兩個內部交易層級 1 Bing 地圖。 根據預設，遠端監視方案會隨著內部交易層級 1 方案一起佈建。 因此，您只可以在一個訂用帳戶中最多佈建 2 個遠端監視方案。

### <a name="can-i-create-a-solution-accelerator-if-i-have-microsoft-azure-for-dreamspark"></a>如果我有 Microsoft Azure for DreamSpark，是否可以建立解決方案加速器？

> [!NOTE]
> Microsoft Azure for DreamSpark 現在稱為 Microsoft Imagine 學生版。

您目前無法使用 [Microsoft Azure for DreamSpark](https://azure.microsoft.com/pricing/member-offers/imagine/) 帳戶來建立解決方案加速器。 不過，您只需花幾分鐘即可建立 [Azure 免費試用帳戶](https://azure.microsoft.com/free/)，此帳戶可讓您建立解決方案加速器。

### <a name="how-do-i-delete-an-aad-tenant"></a>如何刪除 AAD 租用戶？

請參閱 Eric Golpe 的部落格文章：[刪除 Azure AD 租用戶的逐步解說](http://blogs.msdn.com/b/ericgolpe/archive/2015/04/30/walkthrough-of-deleting-an-azure-ad-tenant.aspx)。

### <a name="next-steps"></a>後續步驟

您也可以探索 IoT 解決方案加速器的一些其他特性與功能：

* [探索遠端監視解決方案加速器的功能](quickstart-remote-monitoring-deploy.md)
* [預測性維護解決方案加速器概觀](iot-accelerators-predictive-overview.md)
* [部署連線的處理站解決方案加速器](quickstart-connected-factory-deploy.md) \(英文\)
* [從頭建立 IoT 安全性](/azure/iot-fundamentals/iot-security-ground-up)
