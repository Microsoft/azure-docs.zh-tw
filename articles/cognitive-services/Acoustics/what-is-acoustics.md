---
title: 何謂 Project Acoustics？
titlesuffix: Azure Cognitive Services
description: Project Acoustics Unity 外掛程式可針對以 VR 和傳統螢幕為目標的專案，提供封閉、迴響和空間處理等音效。
services: cognitive-services
author: kegodin
manager: cgronlun
ms.service: cognitive-services
ms.component: acoustics
ms.topic: overview
ms.date: 08/17/2018
ms.author: kegodin
ms.openlocfilehash: 6a3c97ead40cfb24604edac0624e38a9b0713fc0
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48901019"
---
# <a name="what-is-project-acoustics"></a>何謂 Project Acoustics？
Project Acoustics Unity 外掛程式可針對以 VR 和傳統螢幕為目標的專案，提供封閉、迴響和空間處理等音效。 它提供設計遊戲原音的方法，將設計者的意圖鋪排到物理式波浪模擬上。

## <a name="why-use-acoustics-in-virtual-environments"></a>為何要在虛擬環境中使用原音？
人類透過視聽提示了解其所處的世界。 在虛擬世界中，空間音訊與原音的結合可提高使用者的沉浸體驗。 此處所述的原音工具會分析虛擬世界，以建立真正的原音模擬，並支援模擬後的設計程序。 分析包括虛擬世界中各種表層的幾何構造與組成材質。 模擬則包含到達方向 (傳送)、迴響功率、衰減時間等參數，以及封閉和阻礙效果。

## <a name="how-does-this-approach-to-acoustics-work"></a>此一原音設計方法是如何運作的？
系統須依賴虛擬世界的離線計算，這樣可以支援比在執行階段執行分析更複雜的模擬。 離線計算會產生原音參數的查閱資料表。 設計者可指定在執行階段對參數套用的規則。 調整這些規則可產生更多的背景音效，讓強烈的情緒或擬真的場景達到超逼真的效果。

## <a name="design-process-comparison"></a>設計程序比較
Project Acoustics 外掛程式支援對 Unity 場景中的原音使用新的設計程序。 為了說明這個新的設計程序，我們要將其與現今最廣受使用的方法之一相比較。

### <a name="typical-approach-to-acoustics-today"></a>目前常用的原音設計方法
在目前常用的原音設計方法中，您會繪製迴響音量：

![設計檢視](media/reverbZonesAltSPace2.png)

然後，調整每個區域的參數：

![設計檢視](media/TooManyReverbParameters.png)

最後，加入光跡追蹤邏輯，讓整個場景產生正確的封閉和阻礙過濾，以及傳送的路徑搜尋邏輯。 此程式碼可能會增加執行階段成本。 此外也會有邊角平滑度的問題，且不規則形狀的場景中會有邊緣案例。

### <a name="an-alternative-approach-with-physics-based-design"></a>物理式設計的替代方法
透過 Project Acoustics 的 Unity 外掛程式所提供的方法，您可以提供靜態場景的圖形和材質。 由於場景會體素化，且程序不會使用光跡追蹤，因此您無須提供簡化或水密原音網格。 您也無須在場景中標記迴響音量。 外掛程式會將場景上傳至雲端，而它在該處會使用物理式波浪模擬。 結果會在您的專案中整合為查閱資料表，並可基於美觀或遊戲效果進行修改。

![設計檢視](media/GearsWithVoxels.jpg)

## <a name="requirements"></a>需求
* Unity 2018.2+ (用於原音製作) 和 Unity 5.2+ (用於音效設計和部署)
* Windows 64 位元 Unity 編輯器
* Azure Batch 訂用帳戶 (用於原音製作)
* Unity 指令碼執行階段必須設定為 [.NET 4.x 對等項目]

## <a name="platform-support"></a>平台支援
* Windows 桌面 (x86 和 AMD64)
* Windows UWP (x86、AMD64 和 ARM)
* Android (x86 和 ARM64)

## <a name="download"></a>下載
如果您想要評估聲場外掛程式，請在[這裡](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRwMoAEhDCLJNqtVIPwQN6rpUOFRZREJRR0NIQllDOTQ1U0JMNVc4OFNFSy4u)註冊以加入「設計人員預覽」。

## <a name="next-steps"></a>後續步驟
* 深入了解[設計程序](design-process.md)
* 開始[在 Unity 專案中整合原音](getting-started.md)

