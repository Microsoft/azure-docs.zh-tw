---
title: 教學課程：偵測並框出影像中的人臉 - 臉部 API、C#
titleSuffix: Azure Cognitive Services
description: 在本教學課程中，您會建立 Windows 應用程式，以使用 臉部 API 來偵測並框出影像中的人臉。
services: cognitive-services
author: PatrickFarley
manager: cgronlun
ms.service: cognitive-services
ms.component: face-api
ms.topic: tutorial
ms.date: 09/24/2018
ms.author: pafarley
ms.openlocfilehash: ba850c1d4f9d8018f333662eead56f8e91b3fbd7
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/16/2018
ms.locfileid: "49340948"
---
# <a name="tutorial-create-a-wpf-app-to-detect-and-frame-faces-in-an-image"></a>教學課程：建立 WPF 應用程式來偵測並框出影像中的臉部

在本教學課程中，您會建立 Windows Presentation Framework (WPF) 應用程式，以透過其 .NET 用戶端程式庫使用臉部服務。 此應用程式會偵測影像中的臉部、在每張臉的周圍繪出邊框，然後在狀態列上顯示該張臉的描述。 GitHub 上的[偵測 Windows 中的影像並框出人臉](https://github.com/Azure-Samples/Cognitive-Face-CSharp-sample)中會提供完整的程式碼範例。

![顯示已偵測到臉部並使用矩形框出的螢幕擷取畫面](../Images/getting-started-cs-detected.png)

本教學課程說明如何：

> [!div class="checklist"]
> - 建立 WPF 應用程式
> - 安裝臉部服務用戶端程式庫
> - 使用用戶端程式庫來偵測影像中的臉部
> - 在所偵測到的每個臉部周圍繪出邊框
> - 在狀態列上顯示臉部的描述

## <a name="prerequisites"></a>必要條件

- 您需要有訂用帳戶金鑰才能執行範例。 您可以從[試用認知服務](https://azure.microsoft.com/try/cognitive-services/?api=face-api)取得免費的試用訂用帳戶金鑰。
- 任何 [Visual Studio 2015 或 2017](https://www.visualstudio.com/downloads/) 版本。 若為 Visual Studio 2017，則需要有 .NET 桌面應用程式開發工作負載。 本教學課程使用 Visual Studio 2017 Community Edition。
- [Microsoft.Azure.CognitiveServices.Vision.Face 2.2.0-preview](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.2.0-preview) 用戶端程式庫 NuGet 套件。 您不必下載此套件。 下面會提供安裝指示。

## <a name="create-the-visual-studio-solution"></a>建立 Visual Studio 解決方案

請遵循下列步驟以建立 Windows WPF 應用程式專案。

1. 開啟 Visual Studio，然後從 [檔案] 功能表按一下 [新增]，然後按一下 [專案]。
   - 在 Visual Studio 2017 中，依序展開 [已安裝] 和 [其他語言]。 選取 [Visual C#]，然後選取 [WPF 應用程式 (.NET Framework)]。
   - 在 Visual Studio 2015 中，依序展開 [已安裝] 和 [範本]。 選取 [Visual C#]，然後選取 [WPF 應用程式]。
1. 將應用程式命名為 **FaceTutorial**，然後按一下 [確定]。

## <a name="install-the-face-service-client-library"></a>安裝臉部服務用戶端程式庫

請遵循下列指示以安裝用戶端程式庫。

1. 在 [工具] 功能表中，依序選取 [NuGet 套件管理員] 及 [套件管理員主控台]。
1. 在 [套件管理員主控台] 中貼上下列命令，然後按 **Enter**。

    `Install-Package Microsoft.Azure.CognitiveServices.Vision.Face -Version 2.2.0-preview`

## <a name="add-the-initial-code"></a>新增初始程式碼

### <a name="mainwindowxaml"></a>MainWindow.xaml

開啟 MainWindow.xaml (提示：可使用**向上/向下箭號圖示**交換窗格)，然後將其內容取代為下列程式碼。 這個 xaml 程式碼可用來建立 UI 視窗。 請注意 `FacePhoto_MouseMove` 和 `BrowseButton_Click` 事件處理常式。

```xml
<Window x:Class="FaceTutorial.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="MainWindow" Height="700" Width="960">
    <Grid x:Name="BackPanel">
        <Image x:Name="FacePhoto" Stretch="Uniform" Margin="0,0,0,50" MouseMove="FacePhoto_MouseMove" />
        <DockPanel DockPanel.Dock="Bottom">
            <Button x:Name="BrowseButton" Width="72" Height="20" VerticalAlignment="Bottom" HorizontalAlignment="Left"
                    Content="Browse..."
                    Click="BrowseButton_Click" />
            <StatusBar VerticalAlignment="Bottom">
                <StatusBarItem>
                    <TextBlock Name="faceDescriptionStatusBar" />
                </StatusBarItem>
            </StatusBar>
        </DockPanel>
    </Grid>
</Window>
```

### <a name="mainwindowxamlcs"></a>MainWindow.xaml.cs

展開 MainWindow.xaml，然後開啟 MainWindow.xaml.cs，並將其內容取代為下列程式碼。 請忽略紅色鋸齒狀底線；它們會在第一次建置之後消失。

前兩行會匯入用戶端程式庫的命名空間。 接著會建立 `FaceClient`傳入訂用帳戶金鑰，於此同時，會在 `MainWindow` 建構函式中設定 Azure 區域。 `BrowseButton_Click` 和 `FacePhoto_MouseMove` 方法會對應至 MainWindow.xaml 中所宣告的事件處理常式。

`BrowseButton_Click` 會建立 `OpenFileDialog`，讓使用者得以選取 jpg 影像。 主視窗中會讀取並顯示影像。 `BrowseButton_Click` 的其餘程式碼和 `FacePhoto_MouseMove` 的程式碼會在後續步驟中插入。

```csharp
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;

using System;
using System.Collections.Generic;
using System.IO;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;

namespace FaceTutorial
{
    public partial class MainWindow : Window
    {
        // Replace <SubscriptionKey> with your valid subscription key.
        // For example, subscriptionKey = "0123456789abcdef0123456789ABCDEF"
        private const string subscriptionKey = "<SubscriptionKey>";

        // Replace or verify the region.
        //
        // You must use the same region as you used to obtain your subscription
        // keys. For example, if you obtained your subscription keys from the
        // westus region, replace "westcentralus" with "westus".
        //
        // NOTE: Free trial subscription keys are generated in the westcentralus
        // region, so if you are using a free trial subscription key, you should
        // not need to change this region.
        private const string faceEndpoint =
            "https://westcentralus.api.cognitive.microsoft.com";

        private readonly IFaceClient faceClient = new FaceClient(
            new ApiKeyServiceClientCredentials(subscriptionKey),
            new System.Net.Http.DelegatingHandler[] { });

        IList<DetectedFace> faceList;   // The list of detected faces.
        String[] faceDescriptions;      // The list of descriptions for the detected faces.
        double resizeFactor;            // The resize factor for the displayed image.

        public MainWindow()
        {
            InitializeComponent();

            if (Uri.IsWellFormedUriString(faceEndpoint, UriKind.Absolute))
            {
                faceClient.Endpoint = faceEndpoint;
            }
            else
            {
                MessageBox.Show(faceEndpoint,
                    "Invalid URI", MessageBoxButton.OK, MessageBoxImage.Error);
                Environment.Exit(0);
            }
        }

        // Displays the image and calls UploadAndDetectFaces.
        private async void BrowseButton_Click(object sender, RoutedEventArgs e)
        {
            // Get the image file to scan from the user.
            var openDlg = new Microsoft.Win32.OpenFileDialog();

            openDlg.Filter = "JPEG Image(*.jpg)|*.jpg";
            bool? result = openDlg.ShowDialog(this);

            // Return if canceled.
            if (!(bool)result)
            {
                return;
            }

            // Display the image file.
            string filePath = openDlg.FileName;

            Uri fileUri = new Uri(filePath);
            BitmapImage bitmapSource = new BitmapImage();

            bitmapSource.BeginInit();
            bitmapSource.CacheOption = BitmapCacheOption.None;
            bitmapSource.UriSource = fileUri;
            bitmapSource.EndInit();

            FacePhoto.Source = bitmapSource;
        }

        // Displays the face description when the mouse is over a face rectangle.
        private void FacePhoto_MouseMove(object sender, MouseEventArgs e)
        {
        }
    }
}
```

### <a name="insert-your-subscription-key-and-verify-or-change-the-region"></a>插入訂用帳戶金鑰並確認或變更區域

- 在 MainWindow.xaml.cs 中找到下面這一行，並將 `<Subscription Key>` 取代為臉部 API 訂用帳戶金鑰：

    ```csharp
    private const string subscriptionKey = "<SubscriptionKey>";
    ```

- 在 MainWindow.xaml.cs 中找到下面這一行，並取代或確認與訂用帳戶金鑰相關聯的 Azure 區域：

    ```csharp
    private const string Endpoint =
        "https://westcentralus.api.cognitive.microsoft.com";
    ```

    請確定位置與您用來取得訂用帳戶金鑰的位置相同。 例如，如果您是從 **westus** 區域取得訂用帳戶金鑰，請將 `Westcentralus` 取代為 `Westus`。

    如果您是透過免費試用來收到訂用帳戶金鑰，則金鑰的區域會是 **westcentralus**，因此不需要進行任何變更。

### <a name="test-the-app"></a>測試應用程式

按下功能表上的 [啟動] 來測試應用程式。 當視窗開啟時，按一下左下角的 [瀏覽]。 [開啟舊檔] 對話方塊隨即出現，以供您瀏覽並選取相片，然後在視窗中顯示。

![顯示未修改臉部影像的螢幕擷取畫面](../Images/getting-started-cs-ui.png)

## <a name="upload-an-image-to-detect-faces"></a>上傳影像來偵測臉部

若要偵測臉部，最簡單的方式是呼叫 `FaceClient.Face.DetectWithStreamAsync` 方法，這個方法會包裝用於上傳本機影像的[偵測](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) API 方法。

在 `MainWindow` 類別的 `FacePhoto_MouseMove` 方法下方插入下列方法。

系統會建立要分析的臉部屬性清單，並將所提交的影像檔讀取到 `Stream`。 兩者都會傳遞至 `DetectWithStreamAsync` 呼叫。

```csharp
// Uploads the image file and calls DetectWithStreamAsync.
private async Task<IList<DetectedFace>> UploadAndDetectFaces(string imageFilePath)
{
    // The list of Face attributes to return.
    IList<FaceAttributeType> faceAttributes =
        new FaceAttributeType[]
        {
            FaceAttributeType.Gender, FaceAttributeType.Age,
            FaceAttributeType.Smile, FaceAttributeType.Emotion,
            FaceAttributeType.Glasses, FaceAttributeType.Hair
        };

    // Call the Face API.
    try
    {
        using (Stream imageFileStream = File.OpenRead(imageFilePath))
        {
            // The second argument specifies to return the faceId, while
            // the third argument specifies not to return face landmarks.
            IList<DetectedFace> faceList =
                await faceClient.Face.DetectWithStreamAsync(
                    imageFileStream, true, false, faceAttributes);
            return faceList;
        }
    }
    // Catch and display Face API errors.
    catch (APIErrorException f)
    {
        MessageBox.Show(f.Message);
        return new List<DetectedFace>();
    }
    // Catch and display all other errors.
    catch (Exception e)
    {
        MessageBox.Show(e.Message, "Error");
        return new List<DetectedFace>();
    }
}
```

## <a name="draw-rectangles-around-each-face"></a>在每個臉部周圍繪製矩形

新增程式碼以在影像中所偵測到的每個臉部周圍繪製矩形。

在 MainWindow.xaml.cs 中，將 `async` 修飾詞新增至 `BrowseButton_Click` 方法。

```csharp
private async void BrowseButton_Click(object sender, RoutedEventArgs e)
```

在 `BrowseButton_Click` 方法結尾處的 `FacePhoto.Source = bitmapSource` 這一行之後，插入下列程式碼。

`UploadAndDetectFaces` 的呼叫會填入所偵測到臉部的清單。 接下來，每個臉部的周圍都會繪製矩形，且主視窗中會顯示修改後的影像。

```csharp
// Detect any faces in the image.
Title = "Detecting...";
faceList = await UploadAndDetectFaces(filePath);
Title = String.Format(
    "Detection Finished. {0} face(s) detected", faceList.Count);

if (faceList.Count > 0)
{
    // Prepare to draw rectangles around the faces.
    DrawingVisual visual = new DrawingVisual();
    DrawingContext drawingContext = visual.RenderOpen();
    drawingContext.DrawImage(bitmapSource,
        new Rect(0, 0, bitmapSource.Width, bitmapSource.Height));
    double dpi = bitmapSource.DpiX;
    resizeFactor = (dpi > 0) ? 96 / dpi : 1;
    faceDescriptions = new String[faceList.Count];

    for (int i = 0; i < faceList.Count; ++i)
    {
        DetectedFace face = faceList[i];

        // Draw a rectangle on the face.
        drawingContext.DrawRectangle(
            Brushes.Transparent,
            new Pen(Brushes.Red, 2),
            new Rect(
                face.FaceRectangle.Left * resizeFactor,
                face.FaceRectangle.Top * resizeFactor,
                face.FaceRectangle.Width * resizeFactor,
                face.FaceRectangle.Height * resizeFactor
                )
        );

        // Store the face description.
        faceDescriptions[i] = FaceDescription(face);
    }

    drawingContext.Close();

    // Display the image with the rectangle around the face.
    RenderTargetBitmap faceWithRectBitmap = new RenderTargetBitmap(
        (int)(bitmapSource.PixelWidth * resizeFactor),
        (int)(bitmapSource.PixelHeight * resizeFactor),
        96,
        96,
        PixelFormats.Pbgra32);

    faceWithRectBitmap.Render(visual);
    FacePhoto.Source = faceWithRectBitmap;

    // Set the status bar text.
    faceDescriptionStatusBar.Text =
        "Place the mouse pointer over a face to see the face description.";
}
```

## <a name="describe-the-faces-in-the-image"></a>描述影像中的臉部

在 `MainWindow` 類別的 `UploadAndDetectFaces` 方法下方附加下列方法。

此方法會將臉部屬性轉換成用以描述臉部的字串。 當滑鼠指標停留在臉部矩形上時便會顯示此字串。

```csharp
// Creates a string out of the attributes describing the face.
private string FaceDescription(DetectedFace face)
{
    StringBuilder sb = new StringBuilder();

    sb.Append("Face: ");

    // Add the gender, age, and smile.
    sb.Append(face.FaceAttributes.Gender);
    sb.Append(", ");
    sb.Append(face.FaceAttributes.Age);
    sb.Append(", ");
    sb.Append(String.Format("smile {0:F1}%, ", face.FaceAttributes.Smile * 100));

    // Add the emotions. Display all emotions over 10%.
    sb.Append("Emotion: ");
    Emotion emotionScores = face.FaceAttributes.Emotion;
    if (emotionScores.Anger >= 0.1f)
        sb.Append(String.Format("anger {0:F1}%, ", emotionScores.Anger * 100));
    if (emotionScores.Contempt >= 0.1f)
        sb.Append(String.Format("contempt {0:F1}%, ", emotionScores.Contempt * 100));
    if (emotionScores.Disgust >= 0.1f)
        sb.Append(String.Format("disgust {0:F1}%, ", emotionScores.Disgust * 100));
    if (emotionScores.Fear >= 0.1f)
        sb.Append(String.Format("fear {0:F1}%, ", emotionScores.Fear * 100));
    if (emotionScores.Happiness >= 0.1f)
        sb.Append(String.Format("happiness {0:F1}%, ", emotionScores.Happiness * 100));
    if (emotionScores.Neutral >= 0.1f)
        sb.Append(String.Format("neutral {0:F1}%, ", emotionScores.Neutral * 100));
    if (emotionScores.Sadness >= 0.1f)
        sb.Append(String.Format("sadness {0:F1}%, ", emotionScores.Sadness * 100));
    if (emotionScores.Surprise >= 0.1f)
        sb.Append(String.Format("surprise {0:F1}%, ", emotionScores.Surprise * 100));

    // Add glasses.
    sb.Append(face.FaceAttributes.Glasses);
    sb.Append(", ");

    // Add hair.
    sb.Append("Hair: ");

    // Display baldness confidence if over 1%.
    if (face.FaceAttributes.Hair.Bald >= 0.01f)
        sb.Append(String.Format("bald {0:F1}% ", face.FaceAttributes.Hair.Bald * 100));

    // Display all hair color attributes over 10%.
    IList<HairColor> hairColors = face.FaceAttributes.Hair.HairColor;
    foreach (HairColor hairColor in hairColors)
    {
        if (hairColor.Confidence >= 0.1f)
        {
            sb.Append(hairColor.Color.ToString());
            sb.Append(String.Format(" {0:F1}% ", hairColor.Confidence * 100));
        }
    }

    // Return the built string.
    return sb.ToString();
}
```

## <a name="display-the-face-description"></a>顯示臉部描述

以下列程式碼取代 `FacePhoto_MouseMove` 方法。

當滑鼠指標停留在臉部矩形上時，此事件處理常式就會顯示臉部的描述字串。

```csharp
private void FacePhoto_MouseMove(object sender, MouseEventArgs e)
{
    // If the REST call has not completed, return.
    if (faceList == null)
        return;

    // Find the mouse position relative to the image.
    Point mouseXY = e.GetPosition(FacePhoto);

    ImageSource imageSource = FacePhoto.Source;
    BitmapSource bitmapSource = (BitmapSource)imageSource;

    // Scale adjustment between the actual size and displayed size.
    var scale = FacePhoto.ActualWidth / (bitmapSource.PixelWidth / resizeFactor);

    // Check if this mouse position is over a face rectangle.
    bool mouseOverFace = false;

    for (int i = 0; i < faceList.Count; ++i)
    {
        FaceRectangle fr = faceList[i].FaceRectangle;
        double left = fr.Left * scale;
        double top = fr.Top * scale;
        double width = fr.Width * scale;
        double height = fr.Height * scale;

        // Display the face description if the mouse is over this face rectangle.
        if (mouseXY.X >= left && mouseXY.X <= left + width &&
            mouseXY.Y >= top  && mouseXY.Y <= top + height)
        {
            faceDescriptionStatusBar.Text = faceDescriptions[i];
            mouseOverFace = true;
            break;
        }
    }

    // String to display when the mouse is not over a face rectangle.
    if (!mouseOverFace)
        faceDescriptionStatusBar.Text =
            "Place the mouse pointer over a face to see the face description.";
}
```

## <a name="run-the-app"></a>執行應用程式

執行此應用程式，然後瀏覽包含臉部的影像。 請等候幾秒，讓臉部服務回應。 之後，您會看到影像中的臉部上有一個紅色矩形。 只要將滑鼠指標移至臉部矩形上，該臉部的描述就會顯示在狀態列上。

![顯示已偵測到臉部並使用矩形框出的螢幕擷取畫面](../Images/getting-started-cs-detected.png)

## <a name="summary"></a>總結

在本教學課程中，您已了解臉部服務用戶端程式庫的基本使用程序，並建立了應用程式來顯示並框出影像中的臉部。

## <a name="next-steps"></a>後續步驟

了解如何偵測和使用臉部特徵點。

> [!div class="nextstepaction"]
> [如何偵測影像中的人臉](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md)
