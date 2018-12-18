---
title: 註冊新的 Azure IoT Edge 裝置 (入口網站) | Microsoft Docs
description: 使用 Azure 入口網站註冊新的 IoT Edge 裝置
author: kgremban
manager: timlt
ms.author: kgremban
ms.date: 06/05/2018
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: b61594469df33e11c23c9cbe0b9542da374fefa3
ms.sourcegitcommit: 150a40d8ba2beaf9e22b6feff414f8298a8ef868
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37034463"
---
# <a name="register-a-new-azure-iot-edge-device-from-the-azure-portal"></a>從 Azure 入口網站註冊新的 Azure IoT Edge 裝置

您必須先向 IoT 中樞註冊 IoT 裝置，才能將 IoT 裝置與 Azure IoT Edge 搭配使用。 裝置註冊好之後，您會收到一個連接字串，可以用來針對 Edge 工作負載設定裝置。 

本文說明如何使用 Azure 入口網站註冊新的 IoT Edge 裝置。

## <a name="prerequisites"></a>先決條件

* Azure 訂用帳戶中的 [IoT 中樞](../iot-hub/iot-hub-create-through-portal.md)。 

## <a name="create-a-device"></a>建立裝置

在 Azure 入口網站中，我們將在連線至 IoT 中樞，但未啟用 Edge 的裝置上分開建立與管理 IoT Edge 裝置。 

1. 登入 [Azure 入口網站](https://portal.azure.com)，然後瀏覽至 IoT 中樞。 
2. 選取功能表中的 [IoT Edge]。
3. 選取 [新增 IoT Edge 裝置]。 
4. 提供描述性裝置識別碼。 
5. 選取 [ **儲存**]。 

## <a name="view-all-devices"></a>檢視所有裝置

所有連線至 IoT 中樞且已啟用 Edge 的裝置都列於 [IoT Edge] 頁面。 

## <a name="retrieve-the-connection-string"></a>擷取連接字串

您準備好開始設定裝置時，需要一個利用在 IoT 中樞中的身分識別連結實體裝置的連接字串。

1. 從入口網站中的 [IoT Edge] 頁面，按一下 Edge 裝置清單中的裝置識別碼。 
2. 複製 [連接字串—主要金鑰] 或 [連接字串—次要金鑰] 的值。 

## <a name="next-steps"></a>後續步驟

了解如何[使用 Azure 入口網站將模組部署至裝置](how-to-deploy-modules-portal.md) (英文)