---
title: Azure IoT 中樞 IP 連線篩選器 | Microsoft Docs
description: 如何使用 IP 篩選來封鎖從特定 IP 位址至 Azure IoT 中樞的連接。 您可以封鎖來自個別 IP 位址或 IP 位址範圍的連接。
author: rezasherafat
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 05/23/2017
ms.author: rezas
ms.openlocfilehash: 864af9cae35912d95f2c0bf0b574a5ca2404a608
ms.sourcegitcommit: 63613e4c7edf1b1875a2974a29ab2a8ce5d90e3b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/29/2018
ms.locfileid: "43190636"
---
# <a name="use-ip-filters"></a>使用 IP 篩選器

安全性是任何以 Azure IoT 中樞為基礎之 IoT 解決方案的重要一環。 有時候您需要在執行安全性設定的程序中明確地指定可連線的裝置 IP 位址。 「IP 篩選器」功能可讓您設定規則，以拒絕或接受來自特定 IPv4 位址的流量。

## <a name="when-to-use"></a>使用時機

有兩個特定使用案例適合封鎖特定 IP 位址的 IoT 中樞端點︰

- 您的 IoT 中樞只應接收來自指定 IP 位址範圍的流量，並拒絕其他所有流量。 例如，您搭配 [Azure Express Route] 使用 IoT 中樞來建立 IoT 中樞與內部部署基礎結構之間的私人連線。
- 您需要拒絕 IoT 中樞系統管理員認為可疑的 IP 位址所傳來的流量。

## <a name="how-filter-rules-are-applied"></a>篩選器規則的套用方式

IP 篩選器規則會套用在 IoT 中樞服務層級。 因此，IP 篩選器規則會套用至來自裝置和後端應用程式的所有連接 (使用任何受支援的通訊協定)。

嘗試建立連線的 IP 位址若符合 IoT 中樞內的拒絕 IP 規則，將會收到未授權 401 狀態碼和描述。 回應訊息則不涉及 IP 規則。

## <a name="default-setting"></a>預設設定

根據預設，IoT 中樞之入口網站中的 **IP 篩選器**方格是空的。 這項預設設定表示您的中樞可接受來自任何 IP 位址的連線。 這項預設設定等同於可接受 0.0.0.0/0 IP 位址範圍的規則。

![IoT 中樞預設 IP 篩選器設定][img-ip-filter-default]

## <a name="add-or-edit-an-ip-filter-rule"></a>新增或編輯 IP 篩選器規則

當您新增 IP 篩選器規則時，系統會提示您輸入下列值︰

- [IP 篩選器規則名稱] 必須是唯一、不區分大小寫的英數字元字串，最長可為 128 個字元。 所能接受的字元只有 ASCII 7 位元英數字元以及 `{'-', ':', '/', '\', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '''}`。
- 選取 [拒絕] 或 [接受] 做為 IP 篩選器規則的 [動作]。
- 提供單一 IPv4 位址或以 CIDR 標記法表示的 IP 位址區塊。 例如，在 CIDR 表示法中，192.168.100.0/22 表示從 192.168.100.0 到 192.168.103.255 的 1024 個 IPv4 位址。

![新增 IP 篩選器規則到 IoT 中樞][img-ip-filter-add-rule]

儲存規則之後，您會看到通知您正在進行更新的警示。

![有關儲存 IP 篩選器規則的通知][img-ip-filter-save-new-rule]

當您達到 10 個 IP 篩選器規則的上限後，[新增] 選項便會停用。

您可以按兩下包含規則的資料列，以編輯現有規則。

> [!NOTE]
> 拒絕 IP 位址可防止其他 Azure 服務 (例如 Azure 串流分析、Azure 虛擬機器，或入口網站中的裝置總管) 與 IoT 中樞互動。

> [!WARNING]
> 如果您使用 Azure 串流分析 (ASA) 從啟用 IP 篩選的 IoT 中樞讀取訊息，請在 ASA 連接字串中使用 IoT 中樞之與事件中樞相容的名稱和端點。

## <a name="delete-an-ip-filter-rule"></a>刪除 IP 篩選器規則

若要刪除 IP 篩選器規則，請選取方格中的一個或多個規則，然後按一下 [刪除]。

![刪除 IoT 中樞 IP 篩選器規則][img-ip-filter-delete-rule]

## <a name="ip-filter-rule-evaluation"></a>IP 篩選器規則評估

IP 篩選器規則會依序套用，第一個符合 IP 位址的規則會決定接受或拒絕動作。

例如，如果您想要接受 192.168.100.0/22 範圍中的位址，並拒絕其他所有位址，則方格中的第一個規則應接受位址範圍 192.168.100.0/22。 下一個規則應使用 0.0.0.0/0 範圍拒絕所有位址。

按一下資料列前端呈垂直方向的三個點並使用拖放功能，即可變更方格中的 IP 篩選器規則順序。

若要儲存新的 IP 篩選器規則順序，請按一下 [儲存]。

![變更 IoT 中樞 IP 篩選器規則的順序][img-ip-filter-rule-order]

## <a name="next-steps"></a>後續步驟

若要進一步探索 IoT 中樞的功能，請參閱︰

- [作業監視][lnk-monitor]
- [IoT 中樞度量][lnk-metrics]

<!-- Images -->
[img-ip-filter-default]: ./media/iot-hub-ip-filtering/ip-filter-default.png
[img-ip-filter-add-rule]: ./media/iot-hub-ip-filtering/ip-filter-add-rule.png
[img-ip-filter-save-new-rule]: ./media/iot-hub-ip-filtering/ip-filter-save-new-rule.png
[img-ip-filter-delete-rule]: ./media/iot-hub-ip-filtering/ip-filter-delete-rule.png
[img-ip-filter-rule-order]: ./media/iot-hub-ip-filtering/ip-filter-rule-order.png


<!-- Links -->

[IoT Hub developer guide]: iot-hub-devguide.md
[Azure Express Route]:  https://azure.microsoft.com/documentation/articles/expressroute-faqs/#supported-services

[lnk-monitor]: iot-hub-operations-monitoring.md
[lnk-metrics]: iot-hub-metrics.md
