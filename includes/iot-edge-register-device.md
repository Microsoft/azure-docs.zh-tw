---
title: 包含檔案
description: 包含檔案
services: iot-edge
author: kgremban
ms.service: iot-edge
ms.topic: include
ms.date: 06/25/2018
ms.author: kgremban
ms.custom: include file
ms.openlocfilehash: bacafdc8f7fd8e206335f3be0a086df1c54f1081
ms.sourcegitcommit: 0a84b090d4c2fb57af3876c26a1f97aac12015c5
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/11/2018
ms.locfileid: "38746704"
---
建立模擬裝置的裝置身分識別，以便與 IoT 中樞通訊。 由於 IoT Edge 裝置的行為和管理方式不同於典型 IoT 裝置，您從開頭就要將此宣告為 IoT Edge 裝置。 

1. 在 Azure 入口網站中，瀏覽至您的 IoT 中樞。
1. 選取 [IoT Edge]，然後選取 [新增 IoT Edge 裝置]。

   ![加入 IoT Edge 裝置](./media/iot-edge-register-device/add-device.png)

1. 為您的模擬裝置提供唯一的裝置識別碼。
1. 選取 [儲存] 來新增您的裝置。
1. 從裝置清單中選取您的新裝置。
1. 複製 [連接字串 - 主要金鑰] 的值並儲存它。 您將在下一節中使用此值設定 IoT Edge 執行階段。 

