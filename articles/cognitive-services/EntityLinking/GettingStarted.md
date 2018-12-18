---
title: 教學課程：建置實體連結應用程式 - C#
titlesuffix: Azure Cognitive Services
description: 使用實體連結 API 來分析文字，並將具名實體連結到知識庫中的相關項目。
services: cognitive-services
author: DavidLiCIG
manager: cgronlun
ms.service: cognitive-services
ms.component: entity-linking-intelligence
ms.topic: tutorial
ms.date: 07/06/2016
ms.author: davl
ms.openlocfilehash: 907b4cab483f1bf63a864094530784f9c632a1c8
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46365633"
---
# <a name="tutorial-build-an-entity-linking-app-with-c"></a>教學課程：使用 C# 建置實體連結應用程式

Microsoft 實體連結是自然語言處理工具，可用來分析文字並將具名實體連結至知識庫中的相關項目。 

本教學課程會將實體連結用戶端程式庫作為 NuGet 套件使用，藉此探索實體連結。 

### <a name="Prerequisites">先決條件</a>

- Visual Studio 2015
- Microsoft 認知服務 API 金鑰
- 取得用戶端程式庫和範例
- Microsoft 實體連結的 NuGet 套件

您可以透過 [SDK](https://www.github.com/microsoft/cognitive-entitylinking-windows) 下載實體連結智慧服務 API 用戶端程式庫。 下載的 zip 檔案必須解壓縮至您選擇的資料夾，大部分的使用者都是選擇 Visual Studio 2015 資料夾。

### <a name="step-1-subscribe-entity-linking-intelligence-service-and-get-your-own-key">步驟 1：訂閱實體連結智慧服務並取得您的金鑰</a>
在使用實體連結智慧服務之前，您必須註冊 API 金鑰。 請參閱[訂用帳戶](https://www.microsoft.com/cognitive-services/en-us/sign-up)。 本教學課程可以使用主要金鑰或次要金鑰。

### <a name="step-2-create-a-new-project-in-visual-studio">步驟 2：在 Visual Studio 中建立新專案</a>

讓我們從在 Visual Studio 中建立新專案開始吧。 首先，從 [開始] 功能表中啟動 Visual Studio 2015。 然後，選取 [已安裝項目] → [範本] → [Visual C#] → [Windows 通用] → [空白應用程式] 作為專案範本來建立新專案：

 ![建立通用應用程式](./Images/CreateUWP.png)

### <a name="step-3-add-the-entity-linking-nuget-package-to-your-project">步驟 3：將實體連結 NuGet 套件新增至您的專案</a>

認知服務的實體連結會發行為 NuGet.org 套件，而且必須先安裝才能使用。
若要將其新增至您的專案，請移至 [方案總管] 索引標籤並以滑鼠右鍵按一下專案，然後選取 [管理 NuGet 套件]。

首先，在 [NuGet 套件管理員] 視窗中的右上角，選取 [NuGet.org] 作為您的 [套件來源]。 在左上角選取 [瀏覽]，然後在 [搜尋] 方塊中輸入 “ProjectOxford.EntityLinking”。 選取 **Microsoft.ProjectOxford.EntityLinking** NuGet 套件，然後按一下 [安裝]。

接下來，搜尋 Newtonsoft.Json 並安裝。 如果出現要求您檢閱變更的提示，請按一下 [確定]。 如果出現實體連結的授權條款，請按一下 [我接受]。

實體連結現在已安裝為應用程式的一部分。 您可以透過檢查 **Microsoft.ProjectOxford.EntityLinking** 參考是否已顯示為 [方案總管] 中專案的一部分來確認。

 ![專案中已包含 nuget 程式庫](./Images/NugetLibraryInProject.png)
 
### <a name="step-4-add-an-input-and-output-text-block-to-your-apps-xaml">步驟 4：將輸入和輸出文字區塊新增至您應用程式的 XAML</a>
在 [方案總管] 中瀏覽至 **MainPage.xaml**，然後連按兩下此檔案，以在新視窗中開啟。 為方便起見，您可以在 [設計工具] 索引標籤中連按兩下 [XAML] 按鈕，這樣會隱藏 [視覺化設計工具]，並保留所有程式碼檢視空間。

 ![專案中已包含 nuget 程式庫](./Images/UWPMainPage.png)
 
對文字服務而言，將功能視覺化的最佳方式是建立輸入和輸出文字區塊。 若要這樣做，請在 **Grid** 中新增下列 XAML。 此程式碼會新增三個元件：輸入文字方塊、輸出文字區塊及開始按鈕。
 
 ```XAML
 <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
    <Grid.RowDefinitions>
        <RowDefinition Height="*"/>
        <RowDefinition Height="*" />
        <RowDefinition Height="50" />
    </Grid.RowDefinitions>
    <TextBox x:Name="inputBox" Grid.Row="0" TextWrapping="Wrap" Text="Enter a paragraph" Margin="10" AcceptsReturn="True" />
    <TextBlock x:Name="outputBlock" Grid.Row="1" TextWrapping="Wrap" Text="Result will be here" Margin="10" />
    <Button x:Name="button" Grid.Row="2" Content="Get Result" />
</Grid>
 ```
 
### <a name="step-5-proceed-to-add-entity-linking-intelligence-service">步驟 5：開始新增實體連結智慧服務</a>
 
現在，使用者介面已建立。 在使用實體連結服務之前，我們需要新增「按鈕點擊」處理常式。 從 [方案總管] 中開啟 [MainPage.xaml]。 在按鈕結尾加上 button_Click 處理常式。
 
 ```XAML
 <Button x:Name="button" Grid.Row="2" Content="Get Result" Click="button_Click" />
 ```
 
「按鈕點擊」處理常式必須在程式碼中實作。 從 [方案總管] 中開啟 [MainPage.xaml.cs] 以實作按鈕點擊。 EntityLinkingServiceClient 是擷取實體連結回應的包裝函式。 EntityLinkingServiceClient 的建構函式引數是認知服務的訂用帳戶金鑰。 貼上您在**步驟 1**取得的訂用帳戶金鑰，以呼叫實體連結服務。 

以下是範例程式碼，其中已使用實體連結服務將 "wikipediaId" 新增到回應中。 
 
 ```csharp
 private async void button_Click(object sender, RoutedEventArgs e)
{
    var text = this.inputBox.Text;
    var client = new EntityLinkingServiceClient("Your subscription key","https://api.labs.cognitive.microsoft.com");
    var linkResponse = await client.LinkAsync(text);
    var result = string.Join(", ", linkResponse.Select(i => i.WikipediaID).ToList());
    this.outputBlock.Text = result;
}
 ```
 
現在，您可以執行您的第一個自然語言處理實體連結應用程式。 按 [F5 鍵] 以編譯並啟動應用程式。 在輸入方塊中貼上文字片段或段落。 按下 [取得結果] 按鈕，並觀察輸出區塊中已識別的實體。
 
 ![UWP 範例結果](./Images/DemoCodeResult.png)
 
### <a name="summary">總結</a>
 
在本教學課程中，您已了解如何透過幾行 C# 和 XAML 程式碼，來建立可利用實體連結智慧服務用戶端程式庫的應用程式。 

