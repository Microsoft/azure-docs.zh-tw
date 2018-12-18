---
title: 開始使用 Project Acoustics - 認知服務
description: 此快速入門指南會說明如何在您的 Unity 專案中整合外掛程式、製作場景，以及將聲場套用至聲音來源。
services: cognitive-services
author: kegodin
manager: noelc
ms.service: cognitive-services
ms.component: acoustics
ms.topic: article
ms.date: 08/17/2018
ms.author: kegodin
ms.openlocfilehash: b80543b199d4b766c1a8800d2dff4cf5ed81f8cc
ms.sourcegitcommit: 1aedb52f221fb2a6e7ad0b0930b4c74db354a569
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/17/2018
ms.locfileid: "40181925"
---
# <a name="getting-started-with-project-acoustics"></a>開始使用 Project Acoustics
此快速入門指南會說明如何在您的 Unity 專案中整合外掛程式、製作場景，以及將聲場套用至聲音來源。 針對此快速入門，您將必須先建立一個 [Azure Batch 帳戶](create-azure-account.md)。 本指南假設您已熟悉 Unity。

## <a name="download-the-plugin"></a>下載外掛程式
請在[這裡](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRwMoAEhDCLJNqtVIPwQN6rpUOFRZREJRR0NIQllDOTQ1U0JMNVc4OFNFSy4u)註冊以加入「設計人員預覽」。

## <a name="supported-platforms-for-quickstart"></a>支援的快速入門平台
* [Unity 2018.2+](http://www.unity3d.com)
  * 需要將您的專案設定為 **.NET 4.x 對等**指令碼處理執行階段版本 
  * 需要 Windows 型 Unity 編輯器

## <a name="import-the-plugin"></a>匯入外掛程式
將聲場 UnityPackage 匯入到您的專案中。 
* 在 Unity 中，移至 [Assets] \(資產\) > [Import Package] \(匯入套件\) > [Custom Package] \(自訂套件\)

![匯入套件](media/ImportPackage.png)  

* 選擇 [MicrosoftAcoustics.unitypackage]

如果您要將外掛程式匯入到現有的專案中，您的專案可能在專案根目錄中已經有 **mcs.rsp** 檔案，該檔案會指定 C# 編譯器的選項。 您將需要把該檔案的內容與 Project Acoustics 外掛程式隨附的 mcs.rsp 檔案合併。

## <a name="enable-the-plugin"></a>啟用外掛程式
聲場工具組的製作部分需要 .NET 4.x 指令碼處理執行階段版本。 套件匯入將會更新您的 Unity 播放器設定。 請重新啟動 Unity 來讓此設定生效。

![播放器設定](media/PlayerSettings.png)

![.NET 4.5](media/Net45.png)

## <a name="create-a-navigation-mesh"></a>建立導航網格
請使用標準 [Unity 工作流程](https://docs.unity3d.com/Manual/nav-BuildingNavMesh.html)來為您的專案建立導航網格。 如需有關如何使用您自己導航網格的資訊，請參閱[製作 UI 逐步解說](bake-ui-walkthrough.md)。

## <a name="mark-static-meshes-for-acoustics"></a>標示聲場的靜態網格
請使用 Unity 中的 [Window] \(視窗\) > [Acoustics] \(聲場\)  來顯示聲場視窗。 此視窗可以停駐在「偵測器」旁邊。

![開啟 [Acoustics] \(聲場\) 視窗](media/WindowAcoustics.png)

在 Unity 的階層視窗中，將所有已選取的項目取消選取。 在聲場 [Object] \(物件\) 索引標籤中，按一下 [Acoustics Geometry] \(聲場幾何\) 核取方塊，以將您場景中的所有網格和地形標示為聲場幾何。

在 [Materials] \(材質\) 索引標籤上，將聲場材質指派給您場景中所使用的材質。 [Default] \(預設\) 材質的吸收率與混凝土相當。 如需有關指定您自己材質屬性的詳細資訊，請參閱[設計流程頁面](design-process.md)。

![材質索引標籤](media/MaterialsTab.png)

## <a name="preview-the-probes"></a>預覽探測
在 [Probes] \(探測\) 索引標籤上，按一下 [Calculate] \(計算\)。 視場景大小而定，此計算可能需要幾分鐘的時間。 當計算完成時，您會在場景檢視中看到浮動的球體，這標示了聲場模擬的位置 (稱為探查點)。 如果您與場景視窗中的物件靠得夠近，則也可以看到場景體素化。 綠色體素應該會與您標示為幾何的物件對齊。 您可以在場景檢視右上方的 Gizmos 功能表中切換探查點和體素顯示。

![GizmosPreview](media/BakePreviewWithGizmos.png)

## <a name="bake-the-scene"></a>製作場景
在 [Bake] \(製作\) 索引標籤中，輸入您的 Azure 認證，然後按一下 [Bake] \(製作\)。 如果您沒有 Azure Batch 帳戶，請參閱[這個適用於我們所建議之帳戶設定的逐步解說](create-azure-account.md)。
當製作完成時，系統會自動將資料檔下載至您專案中的 **Assets/AcousticsData** 目錄。

## <a name="set-up-audio-runtime-dsp"></a>設定音訊執行階段 DSP
我們在 Unity 的空間定位器架構中內嵌適用於聲場的音訊執行階段 DSP，並將其與 HRTF 型空間定位整合。 若要啟用聲場處理，請移至 [Edit] \(編輯\) > [Project Settings] \(專案設定\) > [Audio] \(音訊\)，然後選取 [Microsoft Acoustics] 作為您專案的 [Spatializer Plugin] \(空間定位器外掛程式\)，來切換至 [Microsoft Acoustics] 空間定位器。 此外，請確定 [DSP Buffer Size] \(DSP 緩衝區大小\) 已設定為 [Best Performance] \(最佳效能\)。

![專案設定](media/ProjectSettings.png)  

![Project Acoustics 空間定位器](media/ChooseSpatializer.png)

開啟 [Audio Mixer] \(音訊混音器\) ([Window] \(視窗\) > [Audio Mixer] \(音訊混音器\))。 請確定您至少有一個含有一個群組的「混音器」。 如果沒有，請按一下 [Mixers] \(混音器\) 右邊的 [+] 按鈕。 在效果區段中聲道控制排的底部上按一下滑鼠右鍵，然後新增 [Microsoft Acoustics Mixer] \(Microsoft Acoustics 混音器\) 效果。 請注意，一次只支援一個「Project Acoustics 混音器」。

![音訊混音器](media/AudioMixer.png)

## <a name="set-up-the-acoustics-lookup-table"></a>設定聲場查閱資料表
將 [Microsoft Acoustics] 預製項目從專案面板拖放到您的場景中：

![聲場預製項目](media/AcousticsPrefab.png)

按一下 [ProjectAcoustics] 遊戲物件以前往其偵測器面板。 透過將製作結果 (**Assets/AcousticsData** 中的 .ACE 檔案) 拖放到 [Acoustics Manager] \(聲場管理員\) 指令碼中，或按一下文字方塊旁邊的圓形按鈕，來指定製作結果的位置。

![聲場管理員](media/AcousticsManager.png)  

## <a name="apply-acoustics-to-sound-sources"></a>將聲場套用至聲音來源
建立音訊來源。 按一下 AudioSource 偵測器面板底部的 [Spatialize] \(空間定位\) 核取方塊。 請確定 [Spatial Blend] \(空間混合\) 已設定為完全 3D。  

![音訊來源](media/AudioSource.png)

## <a name="apply-post-bake-design"></a>套用製作後設計
您可以按一下 [Add Component] \(新增元件\)，然後選擇 [Scripts] \(指令碼\) > [Acoustics Source Customization] \(聲場來源自訂\)，將 **AcousticsSourceCustomization** 指令碼附加至您場景中的聲音來源，以啟用額外的來源設計參數：

![來源自訂](media/SourceCustomization.png)

[Microsoft Acoustics Mixer] \(Microsoft Acoustics 混音器\) 上也有參數。 如需有關製作後設計的詳細資訊，請參閱[設計參數](design-process.md)。

## <a name="next-steps"></a>後續步驟
* 嘗試[範例場景](sample-walkthrough.md)
* 了解整套的[製作功能](bake-ui-walkthrough.md)
* 探索更多詳細的[設計參數](design-process.md)

