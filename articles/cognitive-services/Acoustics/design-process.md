---
title: 聲場的設計流程概觀 - 認知服務
description: 本文件說明如何在 Project Acoustics 工作流程的所有三個階段中表達您的設計意圖。
services: cognitive-services
author: kegodin
manager: noelc
ms.service: cognitive-services
ms.component: acoustics
ms.topic: article
ms.date: 08/17/2018
ms.author: kegodin
ms.openlocfilehash: 8f594be67c4677fae00cb01598d3899e30dae1e8
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47433219"
---
# <a name="design-process-overview"></a>設計流程概觀
您可以在 Project Acoustics 工作流程的所有三個階段中表達您的設計意圖：製作前場景設定、聲音來源位置，以及製作後設計。 此流程在放置殘響音量方面所需的標記較少，同時又保留了設計人員對場景音效的控制。

## <a name="pre-bake-design"></a>製作前設計
製作前場景設定流程會產生用於聲波模擬的場景和中繼資料，其中包括選取哪些場景元素將參與模擬來提供遮蔽、反射及殘響。 場景的中繼資料是為每個場景元素選取的聲場材質。 聲場材質會控制從每個表面反射回來的聲音能量多寡。

所有表面的預設吸收係數都是 0.04，具有高反射性。 您可以4透過調整整個場景不同材質的吸收係數來達到美學和遊戲效果，當聆聽者聽到從場景的一個區域到另一個區域的轉換時，感受會特別明顯。 例如，從黑暗有殘響的室內轉換到明亮無殘響的室外場景時，可增強轉換的影響。 若要達到此效果，請將室外場景材質上的吸收係數調高。

室內所指定材質的殘響時間與其吸收係數成反比，其中大多數材質的吸收值在 0.01 到 0.20 的範圍。 材質的吸收係數若是在此範圍外，則極具吸收性。

![殘響時間圖](media/ReverbTimeGraph.png)

[製作 UI 逐步解說](bake-ui-walkthrough.md)提供製作前控制項的詳細說明。

## <a name="sound-source-placement"></a>聲音來源位置
針對聲音來源困在體素化幾何內的問題，在執行階段檢視體素和探查點可協助對其進行偵錯。 若要切換體素格線和探查點顯示，請按一下場景檢視右上方 Gizmos 功能表中對應的核取方塊。 如果聲音來源位於牆面體素內，請將其移到空氣體素中。

![Gizmos 功能表](media/GizmosMenu.png)  

體素顯示可協助您判斷遊戲中的視覺元件上是否已套用轉換。 如果已套用轉換，請將相同的轉換套用至裝載 **Acoustics Manager** \(聲場管理員\) 的 GameObject。

## <a name="post-bake-design"></a>製作後設計
製作結果會在 ACE 檔案中儲存成整個場景中所有來源與聆聽者位置之配對的遮蔽和殘響參數。 這個實際上準確的結果可依現況用於您的專案，並且是一個絕佳的設計起點。 製作後設計流程會指定在執行階段轉換製作結果參數的規則。

### <a name="distance-based-attenuation"></a>距離型衰減
**Microsoft Acoustics** Unity 空間定位外掛程式所提供的音訊 DSP 會遵守 Unity 編輯器內建的個別來源距離型衰減。 距離型衰減控制項位於聲音來源之 [Inspector] \(偵測器\) 面板中 [Audio Source] \(音訊來源\) 元件的 [3D Sound Settings] \(3D 聲音設定\) 底下：

![距離衰減](media/distanceattenuation.png)

### <a name="tuning-scene-parameters"></a>調整場景參數
若要調整所有來源的參數，請按一下 Unity [Audio Mixer] \(音訊混音器\) 中的聲道控制排，然後調整 [Acoustics Mixer] \(聲場混音器\) 效果上的參數。

![混音器自訂](media/MixerParameters.png)

### <a name="tuning-source-parameters"></a>調整來源參數
將 **AcousticsSourceCustomization** 指令碼附加至來源可為該來源啟用調整參數。 若要附加指令碼，請按一下 [Inspector] \(偵測器\) 面板底部的 [Add Component] \(新增元件\)，然後瀏覽至 [Scripts] \(指令碼\) > [Acoustics Source Customization] \(聲場來源自訂\)。 此指令碼有三個參數：

![來源自訂](media/SourceCustomization.png)

* **Reverb Power Adjust** \(殘響功率調整\) - 調整殘響功率 (單位為 dB)。 正數值會讓聲音有較多殘響，負數值則會讓聲音較乾淨清楚。
* **Decay Time Scale** \(衰變時間縮放\) - 調整衰變時間的倍數。 例如，如果製作結果指定的衰變時間為 750 毫秒，但此值設定為 1.5，則套用至來源的衰變時間就會是 1,125 毫秒。
* **Enable Acoustics** \(啟用聲場\) - 控制是否要將聲場套用至此來源。 未選取時，會以 HRTF 對來源進行空間定位，但若沒有聲場，即意謂著沒有障礙物、遮蔽物及動態殘響參數 (例如等級和衰變時間)。 系統仍會套用具有固定等級和衰變時間的殘響。

不同來源可能需要不同的設定，才能達到特定的美學或遊戲效果。 對話就是其中一個可能的範例。 人耳較適應語音中有殘響，但對遊戲來說，對話通常必須清晰易懂。 針對這種情況，您只需將殘響功率調低即可，而無須讓對話變成非劇情聲。
