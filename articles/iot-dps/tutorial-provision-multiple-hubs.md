---
title: 使用 Azure IoT 中樞裝置佈建服務來跨負載平衡 IoT 中樞佈建裝置 | Microsoft Docs
description: 在 Azure 入口網站中跨負載平衡 IoT 中樞進行 DPS 自動裝置佈建
author: sethmanheim
ms.author: sethm
ms.date: 09/05/2017
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
manager: timlt
ms.custom: mvc
ms.openlocfilehash: d0a3720fe729d5e260bbe5b0902460c8c7cfc7cb
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34629621"
---
# <a name="provision-devices-across-load-balanced-iot-hubs"></a>跨負載平衡 IoT 中樞來佈建裝置

本教學課程示範如何使用裝置佈建服務 (DP) 針對多個負載平衡 IoT 中樞佈建裝置。 在本教學課程中，您了解如何：

> [!div class="checklist"]
> * 使用 Azure 入口網站來將第二個裝置佈建到第二個 IoT 中樞 
> * 在第二個裝置中新增註冊清單項目
> * 將 DPS 配置原則設為**平均分佈**
> * 將新的 IoT 中樞連結至 DPS

如果您沒有 Azure 訂用帳戶，請在開始前建立 [免費帳戶](https://azure.microsoft.com/free/) 。

## <a name="prerequisites"></a>先決條件

本教學課程是以先前的[將裝置佈建到中樞](tutorial-provision-device-to-hub.md)教學課程為基礎所建置。

## <a name="use-the-azure-portal-to-provision-a-second-device-to-a-second-iot-hub"></a>使用 Azure 入口網站來將第二個裝置佈建到第二個 IoT 中樞

請依照[將裝置佈建到中樞](tutorial-provision-device-to-hub.md)教學課程的步驟，將第二個裝置佈建到其他 IoT 中樞。

## <a name="add-an-enrollment-list-entry-to-the-second-device"></a>在第二個裝置中新增註冊清單項目

註冊清單會將它對裝置所使用的證明方法 (用來確認裝置身分識別的方法) 告訴 DPS。 下一個步驟是新增第二個裝置的註冊清單項目。 

1. 在 DPS 的頁面中，按一下 [管理註冊]。 [新增註冊清單項目] 頁面會隨即出現。 
2. 按一下頁面頂端的 [新增]。
2. 填寫欄位，然後按一下 [儲存]。

## <a name="set-the-dps-allocation-policy"></a>設定 DPS 配置原則

配置原則是 DPS 的一項設定，可決定要如何將裝置指派到 IoT 中樞。 支援的配置原則有三個： 

1. **最低延遲**：將裝置佈建到裝置延遲最低的 IoT 中樞。
2. **平均加權分佈** (預設值)：連結的 IoT 中樞有同樣的機率會讓系統在其中佈建裝置。 這是預設設定。 如果您只要將裝置佈建到一個 IoT 中樞，可以保留此設定。 
3. **透過註冊清單進行靜態設定**：您在註冊清單中指定使用的 IoT 中樞，其優先順序高於 DPS 層級的配置原則。

請遵循下列步驟來設定配置原則：

1. 若要設定配置原則，請在 [DPS] 頁面中按一下 [管理配置原則]。
2. 將配置原則設為 [平均加權分佈]。
3. 按一下 [檔案] 。

## <a name="link-the-new-iot-hub-to-dps"></a>將新的 IoT 中樞連結至 DPS

連結 DPS 和 IoT 中樞，讓 DPS 可以向該中樞註冊裝置。

1. 在 [所有資源] 頁面上，按一下您先前建立的 DPS。
2. 在 [DPS] 頁面上，按一下 [連結的 IoT 中樞]。
3. 按一下 [新增] 。
4. 在 [將連結新增至 IoT 中樞] 頁面中，使用選項按鈕來指定連結的 IoT 中樞是位於目前的訂用帳戶中，還是位於不同的訂用帳戶中。 然後，從 [IoT 中樞] 方塊選擇 IoT 中樞的名稱。
5. 按一下 [檔案] 。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何：

> [!div class="checklist"]
> * 使用 Azure 入口網站來將第二個裝置佈建到第二個 IoT 中樞 
> * 在第二個裝置中新增註冊清單項目
> * 將 DPS 配置原則設為**平均分佈**
> * 將新的 IoT 中樞連結至 DPS

<!-- Advance to the next tutorial to learn how to 
 Replace this .md
> [!div class="nextstepaction"]
> [Bind an existing custom SSL certificate to Azure Web Apps](app-service-web-tutorial-custom-ssl.md)
-->
