---
title: 什麼是 Azure IoT 中心 | Microsoft Docs
description: Azure IoT 中心是一個端對端 SaaS 解決方案，您可用來建置及管理自訂 IoT 解決方案。 本文提供 Azure IoT 中心的功能概觀。
author: dominicbetts
ms.author: dobett
ms.date: 11/30/2017
ms.topic: overview
ms.service: iot-central
services: iot-central
ms.custom: mvc
manager: timlt
ms.openlocfilehash: 8c369ab05059e57f2e2a98339052c27292ac7c0d
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34628771"
---
<!---
Purpose of an Overview article: 
1. To give a TECHNICAL overview of a service/product: What is it? Why should I use it? It's a "learn" topic that describes key benefits and our competitive advantage. It's not a "do" topic.
2. To help audiences who are new to service but who may be familiar with related concepts. 
3. To compare the service to another service/product that has some similar functionality, ex. SQL Database / SQL Data Warehouse, if appropriate. This info can be in a short list or table. 
-->

# <a name="what-is-azure-iot-central"></a>何謂 Azure IoT 中心？

Azure IoT 中心是完全受控的 IoT 軟體即服務解決方案，可讓您輕鬆地建立連接實體和數位世界的產品。 您可以將連線的產品願景帶到生活中，方法如下：

- 從連線的裝置衍生新的見解，讓您的客戶擁有更好的產品和體驗。
- 為您的組織創造新商機。

相較於典型的 IoT 專案，Azure IoT 中心藉由下列方法，讓 IoT 解決方案管理變輕鬆：

- 降低管理負擔。
- 降低營運成本和額外負荷。
- 讓應用程式自訂變容易，同時利用：
  - 領先業界的技術，例如 [Azure IoT 中樞](https://azure.microsoft.com/services/iot-hub/) 和 [Azure 時間序列深入解析](https://azure.microsoft.com/services/time-series-insights/)。
  - 企業級安全性功能，例如端對端加密。

下列影片將介紹 Azure IoT 中心的概觀：

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Microsoft-IoT-Central-intro-walkthrough/Player]

這篇文章的其餘部分針對 Azure IoT 中心大略敘述：

- 與專案相關聯的典型角色。
- 如何建立您的應用程式。
- 如何將裝置連線到您的應用程式。
- 如何管理您的應用程式。

## <a name="personas"></a>角色

Azure IoT 中心文件中提及四個會與 Azure IoT 中心應用程式互動的典型角色：

- 「建置者」負責定義連線到應用程式的裝置類型，以及為操作員自訂應用程式。
- 「操作員」管理已連線到應用程式的裝置。
- 「管理員」負責管理各項工作，例如管理應用程式的使用者和角色。
- 「裝置開發人員」建立可在連線到您應用程式的裝置上執行的程式碼。

## <a name="create-your-azure-iot-central-application"></a>建立 Azure IoT 中心應用程式

建置者可使用 Azure IoT 中心，為組織建立自訂、雲端裝載的 IoT 解決方案。 自訂 IoT 解決方案通常包括：

- 雲端式應用程式，可接收來您的裝置的遙測，並可讓您管理這些裝置。
- 多個裝置，這些裝置會執行已連線到雲端式應用程式的自訂程式碼。

您可以快速部署新的 Azure IoT 中心應用程式，然後在瀏覽器中針對您的特定需求加以自訂。 Azure IoT 中心建置者可以使用 Web 架構工具，為連線到您應用程式的裝置建立「裝置範本」。 裝置範本是裝置模型的藍圖，所有裝置都是從裝置範本共用建立而來。 裝置範本可定義某種裝置的特性和行為，例如：

- 它所傳送的遙測。
- 操作員可以修改的業務屬性。
- 由裝置設定而且在應用程式中唯讀的裝置屬性。
- 應用程式回應的閾值。
- 決定裝置行為的設定。

您可以使用 Azure IoT 中心為您產生的模擬資料，立即測試您的裝置範本和應用程式。

建置者也可以為負責應用程式日常使用的操作員，自訂 Azure IoT 中心應用程式 UI。 建置者可以進行的自訂包括：

- 在裝置範本上定義屬性和設定的版面配置。
- 設定自訂儀表板，協助操作員更快發現見解及解決問題。
- 設定自訂分析，以探索來自已連線裝置的時間序列資料。

## <a name="connect-your-devices"></a>連線您的裝置

在建置者定義可連線到應用程式的裝置類型之後，裝置開發人員會建立要在裝置上執行的程式碼。 裝置開發人員可以使用 Microsoft 的開放原始碼 [Azure IoT SDK](https://github.com/Azure/azure-iot-sdks) 來建立裝置程式碼。 這些 SDK 有廣泛的語言、平台和通訊協定支援，符合您將裝置連線到 Azure IoT 中心應用程式的需求。 SDK 可協助您在連線至 Azure IoT 中心的裝置上執行下列工作：

- 建立安全的連線。
- 傳送遙測。
- 報告狀態。
- 接收組態更新。

如需詳細資訊，請參閱部落格文章[使用 Azure IoT SDK 的優點，以及未使用時所要避免的隱藏危險](https://azure.microsoft.com/blog/benefits-of-using-the-azure-iot-sdks-in-your-azure-iot-solution/)。

## <a name="manage-your-application"></a>管理您的應用程式

Azure IoT 中心應用程式完全由 Microsoft 裝載，可降低管理您的應用程式的系統管理負荷。

操作員可以使用 Azure IoT 中心應用程式來管理您的 Azure IoT 中心解決方案中的裝置。 操作員可以執行下列工作：

- 監視已連線到應用程式的裝置。
- 針對裝置問題進行移難排解並修復。
- 佈建新裝置。

建置者可以在裝置範本層級定義可對串流資料操作的自訂規則和動作。 操作員可以在裝置層級啟用或停用這些規則，以控制和自動執行應用程式內的工作。

管理員可以利用[使用者角色和權限](howto-administer.md)，管理您的應用程式存取。

## <a name="next-steps"></a>後續步驟

既然您已大致了解 Azure IoT 中心，以下是建議的後續步驟：

- 了解 [Azure IoT 中心和 Azure IoT 解決方案加速器](overview-iot-options.md)之間的差異。
- 熟悉 [Azure IoT 中心 UI](overview-iot-central-tour.md)。
- 從[建立 Azure IoT 中心應用程式](quick-deploy-iot-central.md)著手。
- 請依照下列一系列的教學課程操作，這些教學課程會告訴您如何：
  - [以建置者身分，建立裝置範本](tutorial-define-device-type.md)
  - [以建置者身分，新增規則以讓您的解決方案自動化](tutorial-configure-rules.md)
  - [以建置者身分，為操作員自訂應用程式](tutorial-customize-operator.md)
  - [以操作員身分，監視您的裝置](tutorial-monitor-devices.md)
  - [以操作員身分，將實際裝置新增至解決方案](tutorial-add-device.md)
  - [以裝置開發人員身分，為您的裝置建立程式碼](tutorial-add-device.md#prepare-the-client-code)
