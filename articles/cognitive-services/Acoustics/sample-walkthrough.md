---
title: 範例：Project Acoustics
titlesuffix: Azure Cognitive Services
description: 此逐步解說會說明 Unity 的 Project Acoustics 範例場景，包括對電腦和 VR 的部署。
services: cognitive-services
author: kegodin
manager: cgronlun
ms.service: cognitive-services
ms.component: acoustics
ms.topic: sample
ms.date: 08/17/2018
ms.author: kegodin
ms.openlocfilehash: f5ea565e68579dfad601d1037daeb4113e3daa43
ms.sourcegitcommit: 7824e973908fa2edd37d666026dd7c03dc0bafd0
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/10/2018
ms.locfileid: "48901153"
---
# <a name="unity-sample-walkthrough"></a>Unity 範例逐步解說
這是 Project Acoustics 範例的逐步解說。 如需有關什麼是 Project Acoustics 的詳細資訊，請參閱 [Project Acoustics 簡介](what-is-acoustics.md)。 如需有關將 Project Acoustics 套件新增至既有 Unity 專案的說明，請使用[使用者入門指南](getting-started.md)。

## <a name="requirements-for-running-the-sample-project"></a>執行範例專案的需求
* Unity 2018.2+ (使用 .NET 4.x 指令碼處理執行階段版本)
* Windows 64 位元 Unity 編輯器
* 此範例支援 Windows 電腦、UWP 與 Android 目標，包括頭戴式顯示器 (HMD)
* 製作流程所需的 Azure Batch 訂用帳戶

## <a name="sample-project-setup"></a>範例專案設定
下載並匯入 **MicrosoftAcoustics.Sample.unitypackage**。 匯入時，會更新專案設定 (包括 [Spatializer] \(空間定位器\) 和 [Scripting Runtime Version] \(指令碼處理執行階段版本\)) 以滿足外掛程式的需求。 此作業完成時，您會在 Unity 主控台中看到來自 **AcousticsGeometry.cs** 的錯誤，此錯誤與將 [Scripting Runtime Version] \(指令碼處理執行階段版本\) 變更為 [.NET 4.x Equivalent] \(.NET 4.x 對等\) 有關。 此設定變更是在匯入套件的過程中進行的，但需要重新啟動 Unity 才能生效。 請立即重新啟動 Unity。

## <a name="running-the-sample"></a>執行範例
此範例包含一個示範場景 **Assets/AcousticsDemo/ProjectAcousticsDemo.unity**。 此場景有三個音效來源。 根據預設，只會播放一個音效來源，其他兩個都會暫停。 這些音效位於 [階層] 中的 [音效來源] 底下。 為了協助建立一般導航指令碼，[Main Camera] \(主要相機\) 會是 CameraHolder 物件的子項。 

![階層檢視](media/SampleHierarchyView.png)

場景已經製作完成，而有一個與 [Hierarchy] \(階層\) 中 [MicrosoftAcoustics] 相關聯的 ACE 檔案。 

請按一下 Unity 編輯器中的播放按鈕來聆聽場景音效。 在桌面上使用 W、A、S、D 與滑鼠即可四處移動。 若要比較場景音效使用原音或不使用原音時的不同，請按下 **R** 按鈕直到重疊文字變成紅色並說出「原音：已停用」。 若要查看鍵盤快速鍵以取得更多控制項，請按下 **F1**。 您可以按一下滑鼠右鍵選取要執行的動作，然後按一下左鍵執行動作來使用所有控制項。

## <a name="targeting-other-platforms"></a>以其他平台為目標
此範例包含可在 Windows 電腦、UWP、Windows Mixed Reality、Android 與 Oculus Go 上執行的設定。 此專案預設是針對 Windows 電腦進行設定的。 若要以 VR 平台為目標，請移至播放器設定 ([Edit] \(編輯\) > [Project Settings] \(專案設定\) > [Player] \(播放器\))，尋找 [XR Settings] \(XR 設定\)，然後按一下 [Virtual Reality Supported] \(支援虛擬實境\) 核取方塊。

![啟用 VR](media/VRSupport.png)  

將 VR 頭戴式裝置連接至您的電腦。 移至 [File] \(檔案\) > [Build Settings] \(建置設定\)，然後按一下 [Build and Run] \(建置並執行\)，將範例部署到您的 VR 頭戴式裝置。 使用您頭戴式裝置的運動控制器，或嘗試使用鍵盤上的 W、A、S、D，來瀏覽整個場景。    
若要以 Android 與 Oculus Go 為目標，請從 [Build Settings] \(建置設定\) 功能表中選擇 [Android]。 按一下 [Switch Target] \(切換目標\)，然後按一下 [Build and Run] \(建置並執行\)。 這會將範例場景部署到已連接的 Android 裝置。 如需有關適用於 Android 的 Unity 開發資訊，請參閱 [Unity 文件](https://docs.unity3d.com/Manual/android-GettingStarted.html)。

![以 Android 為目標](media/TargetAndroid.png)  

## <a name="next-steps"></a>後續步驟
* 為您自己的製作項目[建立 Azure 帳戶](create-azure-account.md)
* 探索[設計流程](design-process.md)

