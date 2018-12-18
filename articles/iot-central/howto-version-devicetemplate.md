---
title: 了解 Azure IoT 中心應用程式的裝置範本版本設定 | Microsoft Docs
description: 藉由建立新版本來反覆使用裝置範本，而不會影響您的即時連線裝置
author: sandeeppujar
ms.author: sandeepu
ms.date: 01/19/2018
ms.topic: conceptual
ms.service: iot-central
services: iot-central
manager: peterpr
ms.openlocfilehash: b125d822596675b138560c14c76f9a3120ce3424
ms.sourcegitcommit: 266fe4c2216c0420e415d733cd3abbf94994533d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/01/2018
ms.locfileid: "34628822"
---
# <a name="create-a-new-device-template-version"></a>建立新的裝置範本版本

Microsoft Azure IoT 中心可讓您快速開發 IoT 應用程式。 您可藉由新增、編輯或刪除量值、設定或屬性，快速反覆使用您的裝置範本設計。 某些變更可能會干擾目前連線的裝置。 Azure IoT 中心可識別這些重大變更，並提供將這些更新安全地部署到裝置的方法。

您在建立裝置範本時，裝置範本會有版本號碼。 根據預設，版本號碼是 1.0.0。 如果您編輯裝置範本，而且該變更會影響即時連線裝置，則 Azure IoT 中心會提示您建立新的裝置範本版本。

> [!NOTE]
> 若要深入了解如何建立裝置範本，請參閱[設定裝置範本](howto-set-up-template.md)

## <a name="changes-that-prompt-a-version-change"></a>提示版本變更的變更

一般情況下，裝置範本的設定或屬性變更會提示版本變更。

> [!NOTE]
> 沒有任何裝置或最多一個裝置連線時，對裝置範本所做的變更，並不會提示建立新的版本。

下列清單描述可能需要新版本的使用者動作：

* 屬性 (必要)
    * 新增或刪除必要屬性
    * 變更屬性的欄位名稱，這是您的裝置用來傳送訊息的欄位名稱。
*  屬性 (選擇性)
    * 刪除選擇性屬性
    * 變更屬性的欄位名稱，這是您的裝置用來傳送訊息的欄位名稱。
    * 將選擇性屬性變更為必要屬性
*  設定
    * 新增或刪除設定
    * 變更設定的欄位名稱，這是您的裝置用來傳送和接收訊息的欄位名稱。

## <a name="what-happens-on-version-change"></a>版本變更會發生什麼情況？

有版本變更時，規則和裝置儀表板會發生什麼情況？

[規則] 可能包含相依於屬性的條件。 如果您已移除一或多個屬性，則可能會在新的裝置範本版本中違背這些規則。 您可以移至這些特定規則，並更新條件來修正規則。 前一版的規則應有作用，不受影響。

**裝置儀表板**可以包含數種圖格類型。 有些圖格可能包含設定和屬性。 移除圖格中所使用的屬性或設定時，圖格會完全或部分損壞。 您可以移至此圖格，藉由移除圖格或更新圖格的內容來修正問題。

## <a name="migrate-a-device-across-device-template-versions"></a>跨裝置範本版本移轉裝置

您可以建立多個版本的裝置範本。 經過一段時間，您會有使用這些裝置範本的多個連線裝置。 您可以將裝置從一個版本的裝置範本移轉至另一個版本。 下列步驟說明如何移轉裝置：

1. 移至 [總管] 頁面。
1. 選取您需要移轉到另一個版本的裝置。
1. 選擇 [移轉裝置]。
1. 選取您希望裝置移轉至的版本號碼並選擇 [移轉]。

![如何移轉裝置](media\howto-version-devicetemplate\pick-version.png)

## <a name="next-steps"></a>後續步驟

您現在已了解如何在 Azure IoT 中心應用程式中使用裝置範本版本，以下是建議後續的步驟：

> [!div class="nextstepaction"]
> [如何建立遙測規則](howto-create-telemetry-rules.md)