---
title: 快速入門：調查安全性警示
description: 在 IoT 裝置上了解、向下切入和調查「適用於 IoT 的 Defender」的安全性警示。
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/30/2020
ms.author: mlottner
ms.openlocfilehash: 172ae82288c2cb948839b69955b9491715eb4690
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/05/2020
ms.locfileid: "90943614"
---
# <a name="quickstart-investigate-security-alerts"></a>快速入門：調查安全性警示

針對「適用於 IoT 的 Defender」發出的警示排定調查和補救，是確保 IoT 解決方案符合規範且安全的最佳方式。

在本快速入門中，我們將探索每個 IoT 安全性警示中可用的資訊，並說明如何向下切入並使用每個警示和相關裝置的詳細資料，以進行回應和補救。 

讓我們開始這次的教學。 


## <a name="investigate-new-security-alerts"></a>調查新的安全性警示

[IoT 中樞安全性警示] 清單會顯示您 IoT 中樞的所有彙總安全性警示。 

1. 在 Azure 入口網站中，開啟您想要調查其新警示的 **IoT 中樞**。
1. 從 [安全性] 功能表中，選取 [警示]。 IoT 中樞的所有安全性警示都會顯示，而具有**新**旗標的警示代表過去 24 小時內產生的警示。
:::image type="content" source="media/quickstart/investigate-new-security-alerts.png" alt-text="使用新的警示旗標調查新的 IoT 安全性警示":::
1. 從清單中選取並開啟任何警示，即可開啟警示詳細資料並向下切入至警示細節。 

## <a name="security-alert-details"></a>安全性警示詳細資料

開啟每個彙總警示後，即會顯示每個觸發警示的裝置及其詳細警示描述、補救步驟和裝置識別碼，以及警示嚴重性和使用 Log Analytics 進行直接調查的存取。 

1. 從 [IoT 中樞] > [安全性] > [警示] 清單中，選取並開啟任一安全性警示。 
1. 針對彙總期間發出此警示的所有裝置，檢閱警示**描述**、**嚴重性**、**偵測來源**、**裝置詳細資料**。
:::image type="content" source="media/quickstart/drill-down-iot-alert-details.png" alt-text="使用新的警示旗標調查新的 IoT 安全性警示"::: 
1. 檢閱警示細節之後，請使用**手動補救步驟**指示，協助修復及/或解決造成警示的問題。 
:::image type="content" source="media/quickstart/iot-alert-manual-remediation-steps.png" alt-text="使用新的警示旗標調查新的 IoT 安全性警示":::

1. 如果需要進一步調查，請使用連結**來調查 Log Analytics 中的警示**。 
:::image type="content" source="media/quickstart/investigate-iot-alert-log-analytics.png" alt-text="使用新的警示旗標調查新的 IoT 安全性警示":::

## <a name="next-steps"></a>後續步驟

繼續閱讀下一篇文章，以深入了解 Defender 警示類型和可能的自訂內容。

> [!div class="nextstepaction"]
> [了解 IoT 安全性警示](concept-security-alerts.md)
