---
title: 包含檔案
description: iot edge
author: kgremban
manager: timlt
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 09/18/2018
ms.author: kgremban
ms.openlocfilehash: a8160e677fa99d8cb691db39d7f29ba6eddbd261
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47004674"
---
## <a name="enabling-extended-offline-operation-preview"></a>啟用延伸離線作業 (預覽)
從 Edge 執行階段的 [1.0.2 版](https://aka.ms/edge102)開始，Edge 裝置和連線到它的下游裝置可設定以使用延伸離線作業。 

有了此功能，本機模組或下游裝置即使與 IoT 中樞中斷連線時，仍可視需要向 Edge 裝置重新驗證，並使用訊息和方法互相通訊。 如需詳細資訊和此功能的範圍，請參閱[此部落格文章](https://aka.ms/iot-edge-offline) \(英文\) 與[概念文章](../articles/iot-edge/offline-capabilities.md)。

針對在閘道中啟用延伸離線的案例，請在 Edge 裝置與將連線到它的下游裝置之間建立父子關聯性。

1. 在 IoT 中樞入口網站的 Edge 裝置詳細資料刀鋒視窗中，按一下頂端命令列中的 [管理子裝置 (預覽)] 按鈕。

1. 按一下 [+新增] 按鈕。

1. 從裝置清單中選取子裝置，並使用向右箭號挑選要新增為子系的裝置。

1. 按一下 [ **確定** ] 以確認。

1. 在 Edge 裝置詳細資料的 [設定模組] 畫面中，按一下 [設定進階 Edge 執行階段設定]，然後在 [Edge 中樞] 環境變數下新增項目 **UpstreamProtocol** 與值 **MQTT**。 針對 [Edge 代理程式] 也新增相同的環境變數與值。 

1. 按一下 [儲存]，且務必在按兩次 [下一步] 之後**提交**變更。

Edge 裝置與其子裝置現在已啟用延伸離線作業。  