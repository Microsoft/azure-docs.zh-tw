---
title: Smooth Streaming Windows 市集應用程式教學課程 | Microsoft Docs
description: 了解如何使用 Azure 媒體服務建立可用 XML MediaElement 控制項來播放 Smooth Streaming 內容的 C# Windows 市集應用程式。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 0fa5d8c5-3d5f-4886-ae55-fb6de4f5256d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/11/2017
ms.author: juliako
ms.openlocfilehash: f55e8aa652d65ea751a77742fa5823b09d9ee87b
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33783807"
---
# <a name="how-to-build-a-smooth-streaming-windows-store-application"></a>如何建置 Smooth Streaming Windows 市集應用程式

Smooth Streaming Client SDK for Windows 8 可讓開發人員建置能夠播放隨選與即時 Smooth Streaming 內容的 Windows 市集應用程式。 除了將 Smooth Streaming 內容進行基本播放，SDK 也提供 Microsoft PlayReady 保護、品質等級限制、Live DVR、音訊資料流切換、接聽狀態更新 (例如品質等級變更) 和錯誤事件等這類豐富的功能。 如需所支援功能的詳細資訊，請參閱 [版本資訊](http://www.iis.net/learn/media/smooth-streaming/smooth-streaming-client-sdk-for-windows-8-release-notes)(英文)。 如需詳細資訊，請參閱 [Player Framework for Windows 8](http://playerframework.codeplex.com/)。 

本教學課程包含四個課程：

1. 建立基本的 Smooth Streaming 市集應用程式
2. 新增滑動軸以控制媒體進度
3. 選取 Smooth Streaming 資料流
4. 選取 Smooth Streaming 曲目

## <a name="prerequisites"></a>先決條件
> [!NOTE]
> Windows 市集 8.1 版和更早版本的專案在 Visual Studio 2017 不受支援。  如需詳細資訊，請參閱 [Visual Studio 2017 平台目標及相容性](https://www.visualstudio.com/en-us/productinfo/vs2017-compatibility-vs)。

* Windows 8 32 位元或 64 位元。
* Visual Studio 2012 到 2015 版。
* [Microsoft Smooth Streaming Client SDK for Windows 8](http://visualstudiogallery.msdn.microsoft.com/04423d13-3b3e-4741-a01c-1ae29e84fea6?SRC=Homehttp://visualstudiogallery.msdn.microsoft.com/04423d13-3b3e-4741-a01c-1ae29e84fea6?SRC=Home)(英文)。

您可以從 MSDN 開發人員程式碼範例 (Code Gallery) 下載每個課程的已完成解答： 

* [課程 1](http://code.msdn.microsoft.com/Smooth-Streaming-Client-0bb1471f) - 簡單 Windows 8 Smooth Streaming Media Player， 
* [課程 2](http://code.msdn.microsoft.com/A-simple-Windows-8-Smooth-ee98f63a) - 具有滑動軸控制項的簡單 Windows 8 Smooth Streaming Media Player， 
* [課程 3](http://code.msdn.microsoft.com/A-Windows-8-Smooth-883c3b44) - 具有串流選擇的 Windows 8 Smooth Streaming Media Player，  
* [課程 4](http://code.msdn.microsoft.com/A-Windows-8-Smooth-aa9e4907) - 具有追蹤選擇的 Windows 8 Smooth Streaming Media Player。

## <a name="lesson-1-create-a-basic-smooth-streaming-store-application"></a>課程 1：建立基本的 Smooth Streaming 市集應用程式

在本課程中，您將建立 Windows 市集應用程式，並使其具有 MediaElement 控制項來播放 Smooth Stream 內容。  執行中的應用程式看起來如下：

![Smooth Streaming Windows Store application example][PlayerApplication]

如需關於開發 Windows 市集應用程式的詳細資訊，請參閱 [開發 Windows 8 適用的好用應用程式](http://msdn.microsoft.com/windows/apps/br229512.aspx)。 本課程包含下列程序：

1. 建立 Windows 市集專案
2. 設計使用者介面 (XAML)
3. 修改程式碼後置檔案
4. 編譯和測試應用程式

**建立 Windows 市集專案**

1. 執行 Visual Studio；支援 2012 到 2015 版。
2. 從 [檔案] 功能表中，按一下 [新增]，再按 [專案]。
3. 從 [新增專案] 對話方塊中，輸入或選取下列值：

| Name | 值 |
| --- | --- |
| 範本群組 |已安裝/範本/Visual C#/Windows 市集 |
| 範本 |空白應用程式 (XAML) |
| Name |SSPlayer |
| 位置 |C:\SSTutorials |
| 方案名稱 |SSPlayer |
| 為方案建立目錄 |(已選取) |

1. 按一下 [確定]。

**新增 Smooth Streaming Client SDK 的參考**

1. 從 [方案總管] 中，在 [SSPlayer] 上按一下滑鼠右鍵，然後按一下 [加入參考]。
2. 輸入或選取下列值：

| Name | 值 |
| --- | --- |
| 參考群組 |Windows/延伸 |
| 參考 |選取 Microsoft Smooth Streaming Client SDK for Windows 8 和 Microsoft Visual C++ Runtime Package |

1. 按一下 [確定]。 

加入參考之後，您必須選取目標平台 (x64 或 x86)，而加入參考在「任何 CPU 平台」組態中將沒有作用。  在方案總管中，您會看到這些加入的參考具有黃色警告標記。

**設計播放程式使用者介面**

1. 從 [方案總管] 中，按兩下 [MainPage.xaml]  ，以設計檢視來開啟它。
2. 在 XAML 檔案中找到 **&lt;Grid&gt;** 和 **&lt;/Grid&gt;** 標籤，然後在這兩個標籤之間貼上下列程式碼：

         <Grid.RowDefinitions>

            <RowDefinition Height="20"/>    <!-- spacer -->
            <RowDefinition Height="50"/>    <!-- media controls -->
            <RowDefinition Height="100*"/>  <!-- media element -->
            <RowDefinition Height="80*"/>   <!-- media stream and track selection -->
            <RowDefinition Height="50"/>    <!-- status bar -->
         </Grid.RowDefinitions>

         <StackPanel Name="spMediaControl" Grid.Row="1" Orientation="Horizontal">
            <TextBlock x:Name="tbSource" Text="Source :  " FontSize="16" FontWeight="Bold" VerticalAlignment="Center" />
            <TextBox x:Name="txtMediaSource" Text="http://ecn.channel9.msdn.com/o9/content/smf/smoothcontent/elephantsdream/Elephants_Dream_1024-h264-st-aac.ism/manifest" FontSize="10" Width="700" Margin="0,4,0,10" />
            <Button x:Name="btnSetSource" Content="Set Source" Width="111" Height="43" Click="btnSetSource_Click"/>
            <Button x:Name="btnPlay" Content="Play" Width="111" Height="43" Click="btnPlay_Click"/>
            <Button x:Name="btnPause" Content="Pause"  Width="111" Height="43" Click="btnPause_Click"/>
            <Button x:Name="btnStop" Content="Stop"  Width="111" Height="43" Click="btnStop_Click"/>
            <CheckBox x:Name="chkAutoPlay" Content="Auto Play" Height="55" Width="Auto" IsChecked="{Binding AutoPlay, ElementName=mediaElement, Mode=TwoWay}"/>
            <CheckBox x:Name="chkMute" Content="Mute" Height="55" Width="67" IsChecked="{Binding IsMuted, ElementName=mediaElement, Mode=TwoWay}"/>
         </StackPanel>

         <StackPanel Name="spMediaElement" Grid.Row="2" Height="435" Width="1072"
                    HorizontalAlignment="Center" VerticalAlignment="Center">
            <MediaElement x:Name="mediaElement" Height="356" Width="924" MinHeight="225"
                          HorizontalAlignment="Center" VerticalAlignment="Center" 
                          AudioCategory="BackgroundCapableMedia" />
            <StackPanel Orientation="Horizontal">
                <Slider x:Name="sliderProgress" Width="924" Height="44"
                        HorizontalAlignment="Center" VerticalAlignment="Center"
                        PointerPressed="sliderProgress_PointerPressed"/>
                <Slider x:Name="sliderVolume" 
                        HorizontalAlignment="Right" VerticalAlignment="Center" Orientation="Vertical" 
                        Height="79" Width="148" Minimum="0" Maximum="1" StepFrequency="0.1" 
                        Value="{Binding Volume, ElementName=mediaElement, Mode=TwoWay}" 
                        ToolTipService.ToolTip="{Binding Value, RelativeSource={RelativeSource Mode=Self}}"/>
            </StackPanel>
         </StackPanel>

         <StackPanel Name="spStatus" Grid.Row="4" Orientation="Horizontal">
            <TextBlock x:Name="tbStatus" Text="Status :  " 
               FontSize="16" FontWeight="Bold" VerticalAlignment="Center" HorizontalAlignment="Center" />
            <TextBox x:Name="txtStatus" FontSize="10" Width="700" VerticalAlignment="Center"/>
         </StackPanel>
   
   MediaElement 控制項會用來播放媒體。 下一個課程將使用名稱為 sliderProgress 的滑動軸控制項來控制媒體進度。
3. 按 **CTRL+S** 儲存檔案。

MediaElement 控制項預設不支援 Smooth Streaming 內容。 若要啟用 Smooth Streaming 支援，您必須依副檔名和 MIME 類型來註冊 Smooth Streaming 位元組資料流處理常式。  若要註冊，請使用 Windows.Media 命名空間的 MediaExtensionManager.RegisterByteStremHandler 方法。

在此 XAML 檔案中，有些事件處理常式會與控制項相關聯。  您必須定義那些事件處理常式。

**修改程式碼後置檔案**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在檔案的頂端，新增下列 using 陳述式：
   
        using Windows.Media;
3. 在 **MainPage** 類別的開頭，新增下列資料成員：
   
         private MediaExtensionManager extensions = new MediaExtensionManager();
4. 在 **MainPage** 建構函式的結尾，新增下列兩行：
   
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "text/xml");
        extensions.RegisterByteStreamHandler("Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", ".ism", "application/vnd.ms-sstr+xml");
5. 在 **MainPage** 類別結尾貼上下列程式碼：
   
         # region UI Button Click Events
         private void btnPlay_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Play();
         txtStatus.Text = "MediaElement is playing ...";
         }
         private void btnPause_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Pause();
         txtStatus.Text = "MediaElement is paused";
         }
         private void btnSetSource_Click(object sender, RoutedEventArgs e)
         {

         sliderProgress.Value = 0;
         mediaElement.Source = new Uri(txtMediaSource.Text);

         if (chkAutoPlay.IsChecked == true)
         {
             txtStatus.Text = "MediaElement is playing ...";
         }
         else
         {
             txtStatus.Text = "Click the Play button to play the media source.";
         }
         }
         private void btnStop_Click(object sender, RoutedEventArgs e)
         {

         mediaElement.Stop();
         txtStatus.Text = "MediaElement is stopped";
         }
         private void sliderProgress_PointerPressed(object sender, PointerRoutedEventArgs e)
         {

         txtStatus.Text = "Seek to position " + sliderProgress.Value;
         mediaElement.Position = new TimeSpan(0, 0, (int)(sliderProgress.Value));
         }
         # endregion

這裡在定義 sliderProgress_PointerPressed 事件處理常式。  還需要進行其他工作，才能讓它運作，在本教學課程的下一個課程中，將會涵蓋這項資訊。
6. 按 **CTRL+S** 儲存檔案。

完成的程式碼後置檔案看起來像這樣：

![Codeview in Visual Studio of Smooth Streaming Windows Store application][CodeViewPic]

**編譯和測試應用程式**

1. 從 [建置] 功能表中，按一下 [組態管理員]。
2. 變更 [使用中的方案平台]  ，以符合您的開發平台。
3. 按 **F6** 鍵編譯專案。 
4. 按 **F5** 鍵執行應用程式。
5. 在應用程式頂端，您可以使用預設 Smooth Streaming URL，或輸入不同的 Smooth Streaming URL。 
6. 按一下 [設定來源] 。 因為預設會啟用 [自動播放]  ，所以應該會自動播放媒體。  您可以使用 [播放]、[暫停] 和 [停止] 按鈕來控制媒體。  您可以使用垂直滑動軸來控制媒體音量。  不過，尚未完整實作用於控制媒體進度的水平滑動軸。 

您已完成課程 1。  在本課程中，您使用 MediaElement 控制項來播放 Smooth Streaming 內容。  在下一個課程中，您將新增滑動軸來控制 Smooth Streaming 內容的進度。

## <a name="lesson-2-add-a-slider-bar-to-control-the-media-progress"></a>課程 2：新增滑動軸以控制媒體進度

在課程 1 中，您已建立 Windows 市集應用程式，並使其具有 MediaElement XAML 控制項來播放 Smooth Streaming 媒體內容。  它包含一些基本媒體功能 (例如啟動、停止和暫停)。  在本課程中，您將在應用程式中新增滑動軸控制項。

在本教學課程中，我們將使用計時器，以根據 MediaElement 控制項的目前位置來更新滑動軸位置。  如果是即時內容，則還需要更新滑動軸開始和結束時間。  這可以透過調適性來源更新事件獲得更恰當的處理。

媒體來源是一種產生媒體資料的物件。  來源解析程式會接受 URL 或位元組資料流，然後為該內容建立適當的媒體來源。  來源解析程式是應用程式建立媒體來源的標準方式。 

本課程包含下列程序：

1. 註冊 Smooth Streaming 處理常式 
2. 新增調適性來源管理員層級事件處理常式
3. 新增調適性來源層級事件處理常式
4. 新增 MediaElement 事件處理常式
5. 新增滑動軸相關程式碼
6. 編譯和測試應用程式

**註冊 Smooth Streaming 位元組資料流處理常式並傳遞屬性集**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在檔案的開頭，新增下列 using 陳述式：

        using Microsoft.Media.AdaptiveStreaming;
3. 在 MainPage 類別的開頭，新增下列資料成員：

         private Windows.Foundation.Collections.PropertySet propertySet = new Windows.Foundation.Collections.PropertySet();             
         private IAdaptiveSourceManager adaptiveSourceManager;
4. 在 **MainPage** 建構函式內，於 **this.Initialize Components();** 行以及上一個課程中所寫的註冊程式碼行的後面新增下列程式碼：

        // Gets the default instance of AdaptiveSourceManager which manages Smooth 
        //Streaming media sources.
        adaptiveSourceManager = AdaptiveSourceManager.GetDefault();
        // Sets property key value to AdaptiveSourceManager default instance.
        // {A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}" must be hardcoded.
        propertySet["{A5CE1DE8-1D00-427B-ACEF-FB9A3C93DE2D}"] = adaptiveSourceManager;
5. 在 **MainPage** 建構函式內，修改兩個 RegisterByteStreamHandler 方法以新增 forth 參數：

         // Registers Smooth Streaming byte-stream handler for ".ism" extension and, 
         // "text/xml" and "application/vnd.ms-ss" mime-types and pass the propertyset. 
         // http://*.ism/manifest URI resources will be resolved by Byte-stream handler.
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "text/xml", 
            propertySet );
         extensions.RegisterByteStreamHandler(

            "Microsoft.Media.AdaptiveStreaming.SmoothByteStreamHandler", 
            ".ism", 
            "application/vnd.ms-sstr+xml", 
         propertySet);
6. 按 **CTRL+S** 儲存檔案。

**新增調適性來源管理員層級事件處理常式**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在 [MainPage]  類別內，新增下列資料成員：
   
     private AdaptiveSource adaptiveSource = null;
3. 在 **MainPage** 類別的結尾，新增下列事件處理常式：
   
         # region Adaptive Source Manager Level Events
         private void mediaElement_AdaptiveSourceOpened(AdaptiveSource sender, AdaptiveSourceOpenedEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
         }

         # endregion Adaptive Source Manager Level Events
4. 在 **MainPage** 建構函式的結尾，新增下行以訂閱調適性來源開放事件：
   
         adaptiveSourceManager.AdaptiveSourceOpenedEvent += 
           new AdaptiveSourceOpenedEventHandler(mediaElement_AdaptiveSourceOpened);
5. 按 **CTRL+S** 儲存檔案。

**新增調適性來源層級事件處理常式**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在 [MainPage]  類別內，新增下列資料成員：
   
     private AdaptiveSourceStatusUpdatedEventArgs adaptiveSourceStatusUpdate;   private Manifest manifestObject;
3. 在 **MainPage** 類別的結尾，新增下列事件處理常式：

         # region Adaptive Source Level Events
         private void mediaElement_ManifestReady(AdaptiveSource sender, ManifestReadyEventArgs args)
         {

            adaptiveSource = args.AdaptiveSource;
            manifestObject = args.AdaptiveSource.Manifest;
         }

         private void mediaElement_AdaptiveSourceStatusUpdated(AdaptiveSource sender, AdaptiveSourceStatusUpdatedEventArgs args)
         {

            adaptiveSourceStatusUpdate = args;
         }

         private void mediaElement_AdaptiveSourceFailed(AdaptiveSource sender, AdaptiveSourceFailedEventArgs args)
         {

            txtStatus.Text = "Error: " + args.HttpResponse;
         }

         # endregion Adaptive Source Level Events
4. 在 **mediaElement AdaptiveSourceOpened** 方法的結尾，新增下列程式碼以訂閱事件：
   
         adaptiveSource.ManifestReadyEvent +=

                    mediaElement_ManifestReady;
         adaptiveSource.AdaptiveSourceStatusUpdatedEvent += 

            mediaElement_AdaptiveSourceStatusUpdated;
         adaptiveSource.AdaptiveSourceFailedEvent += 

            mediaElement_AdaptiveSourceFailed;
5. 按 **CTRL+S** 儲存檔案。

在調適性來源管理員層級也有相同的事件，可用於處理應用程式中所有媒體元素通用的功能。 每個 AdaptiveSource 都包含自己專屬的事件，而且所有 AdaptiveSource 事件都會在 AdaptiveSourceManager 下階層式列出。

**新增媒體元素事件處理常式**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在 **MainPage** 類別的結尾，新增下列事件處理常式：

         # region Media Element Event Handlers
         private void MediaOpened(object sender, RoutedEventArgs e)
         {

            txtStatus.Text = "MediaElement opened";
         }

         private void MediaFailed(object sender, ExceptionRoutedEventArgs e)
         {

            txtStatus.Text= "MediaElement failed: " + e.ErrorMessage;
         }

         private void MediaEnded(object sender, RoutedEventArgs e)
         {

            txtStatus.Text ="MediaElement ended.";
         }

         # endregion Media Element Event Handlers
3. 在 **MainPage** 建構函式的結尾，新增下列程式碼以訂閱事件：

         mediaElement.MediaOpened += MediaOpened;
         mediaElement.MediaEnded += MediaEnded;
         mediaElement.MediaFailed += MediaFailed;
4. 按 **CTRL+S** 儲存檔案。

**新增滑動軸相關程式碼**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在檔案的開頭，新增下列 using 陳述式：
      
        using Windows.UI.Core;
3. 在 [MainPage]  類別內，新增下列資料成員：
   
         public static CoreDispatcher _dispatcher;
         private DispatcherTimer sliderPositionUpdateDispatcher;
4. 在 **MainPage** 建構函式的結尾，新增下列程式碼：
   
         _dispatcher = Window.Current.Dispatcher;
         PointerEventHandler pointerpressedhandler = new PointerEventHandler(sliderProgress_PointerPressed);
         sliderProgress.AddHandler(Control.PointerPressedEvent, pointerpressedhandler, true);    
5. 在 **MainPage** 類別的結尾，新增下列程式碼：

         # region sliderMediaPlayer
         private double SliderFrequency(TimeSpan timevalue)
         {

            long absvalue = 0;
            double stepfrequency = -1;

            if (manifestObject != null)
            {
                absvalue = manifestObject.Duration - (long)manifestObject.StartTime;
            }
            else
            {
                absvalue = mediaElement.NaturalDuration.TimeSpan.Ticks;
            }

            TimeSpan totalDVRDuration = new TimeSpan(absvalue);

            if (totalDVRDuration.TotalMinutes >= 10 && totalDVRDuration.TotalMinutes < 30)
            {
               stepfrequency = 10;
            }
            else if (totalDVRDuration.TotalMinutes >= 30 
                     && totalDVRDuration.TotalMinutes < 60)
            {
                stepfrequency = 30;
            }
            else if (totalDVRDuration.TotalHours >= 1)
            {
                stepfrequency = 60;
            }

            return stepfrequency;
         }

         void updateSliderPositionoNTicks(object sender, object e)
         {

            sliderProgress.Value = mediaElement.Position.TotalSeconds;
         }

         public void setupTimer()
         {

            sliderPositionUpdateDispatcher = new DispatcherTimer();
            sliderPositionUpdateDispatcher.Interval = new TimeSpan(0, 0, 0, 0, 300);
            startTimer();
         }

         public void startTimer()
         {

            sliderPositionUpdateDispatcher.Tick += updateSliderPositionoNTicks;
            sliderPositionUpdateDispatcher.Start();
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderStartTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.StartTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Minimum = absvalue;
            });
         }

         // Slider start and end time must be updated in case of live content
         public async void setSliderEndTime(long startTime)
         {

            await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, () =>
            {
                TimeSpan timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime);
                double absvalue = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero);
                sliderProgress.Maximum = absvalue;
            });
         }

         # endregion sliderMediaPlayer
      
>[!NOTE]
>CoreDispatcher 用來從非 UI 執行緒對 UI 執行緒進行變更。 如果發送器執行緒發生瓶頸，開發人員可以選擇使用自己想要更新之 UI-element 所提供的發送器。  例如︰
   
         await sliderProgress.Dispatcher.RunAsync(CoreDispatcherPriority.Normal, () => { TimeSpan 

         timespan = new TimeSpan(adaptiveSourceStatusUpdate.EndTime); 
         double absvalue  = (int)Math.Round(timespan.TotalSeconds, MidpointRounding.AwayFromZero); 

         sliderProgress.Maximum = absvalue; }); 
6. 在 **mediaElement_AdaptiveSourceStatusUpdated** 方法的結尾，新增下列程式碼：

         setSliderStartTime(args.StartTime);
         setSliderEndTime(args.EndTime);
7. 在 **MediaOpened** 方法的結尾，新增下列程式碼：

         sliderProgress.StepFrequency = SliderFrequency(mediaElement.NaturalDuration.TimeSpan);
         sliderProgress.Width = mediaElement.Width;
         setupTimer();
8. 按 **CTRL+S** 儲存檔案。

**編譯和測試應用程式**

1. 按 **F6** 鍵編譯專案。 
2. 按 **F5** 鍵執行應用程式。
3. 在應用程式頂端，您可以使用預設 Smooth Streaming URL，或輸入不同的 Smooth Streaming URL。 
4. 按一下 [設定來源] 。 
5. 測試滑動軸。

您已完成課程 2。  在本課程中，您將在應用程式中新增滑動軸。 

## <a name="lesson-3-select-smooth-streaming-streams"></a>課程 3：選取 Smooth Streaming 資料流
Smooth Streaming 可以串流含多個曲目可供檢視器選取的內容。  在本課程中，您將讓檢視器選取資料流。 本課程包含下列程序：

1. 修改 XAML 檔案
2. 修改程式碼後置檔案
3. 編譯和測試應用程式

**修改 XAML 檔案**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視設計工具]。
2. 找到 &lt;Grid.RowDefinitions&gt;，並修改 RowDefinitions，讓它們看起來如下：
   
         <Grid.RowDefinitions>            
            <RowDefinition Height="20"/>
            <RowDefinition Height="50"/>
            <RowDefinition Height="100"/>
            <RowDefinition Height="80"/>
            <RowDefinition Height="50"/>
         </Grid.RowDefinitions>
3. 在 &lt;Grid&gt;&lt;/Grid&gt; 標籤內，新增下列程式碼以定義 listbox 控制項，讓使用者可以看到可用資料流清單並選取資料流：

         <Grid Name="gridStreamAndBitrateSelection" Grid.Row="3">
            <Grid.RowDefinitions>
                <RowDefinition Height="300"/>
            </Grid.RowDefinitions>
            <Grid.ColumnDefinitions>
                <ColumnDefinition Width="250*"/>
                <ColumnDefinition Width="250*"/>
            </Grid.ColumnDefinitions>
            <StackPanel Name="spStreamSelection" Grid.Row="1" Grid.Column="0">
                <StackPanel Orientation="Horizontal">
                    <TextBlock Name="tbAvailableStreams" Text="Available Streams:" FontSize="16" VerticalAlignment="Center"></TextBlock>
                    <Button Name="btnChangeStreams" Content="Submit" Click="btnChangeStream_Click"/>
                </StackPanel>
                <ListBox x:Name="lbAvailableStreams" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                    ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
                    <ListBox.ItemTemplate>
                        <DataTemplate>
                            <CheckBox Content="{Binding Name}" IsChecked="{Binding isChecked, Mode=TwoWay}" />
                        </DataTemplate>
                    </ListBox.ItemTemplate>
                </ListBox>
            </StackPanel>
         </Grid>
4. 按 **CTRL+S** 儲存變更。

**修改程式碼後置檔案**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在 SSPlayer 命名空間內，新增類別：
   
        #region class Stream
   
        public class Stream
        {
            private IManifestStream stream;
            public bool isCheckedValue;
            public string name;
   
            public string Name
            {
                get { return name; }
                set { name = value; }
            }
   
            public IManifestStream ManifestStream
            {
                get { return stream; }
                set { stream = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    // mMke the video stream always checked.
                    if (stream.Type == MediaStreamType.Video)
                    {
                        isCheckedValue = true;
                    }
                    else
                    {
                        isCheckedValue = value;
                    }
                }
            }
   
            public Stream(IManifestStream streamIn)
            {
                stream = streamIn;
                name = stream.Name;
            }
        }
        #endregion class Stream
3. 在 MainPage 類別的開頭，新增下列變數定義：
   
         private List<Stream> availableStreams;
         private List<Stream> availableAudioStreams;
         private List<Stream> availableTextStreams;
         private List<Stream> availableVideoStreams;
4. 在 MainPage 類別內，新增下列區域：
   
        #region stream selection
        ///<summary>
        ///Functionality to select streams from IManifestStream available streams
        /// </summary>
   
        // This function is called from the mediaElement_ManifestReady event handler 
        // to retrieve the streams and populate them to the local data members.
        public void getStreams(Manifest manifestObject)
        {
            availableStreams = new List<Stream>();
            availableVideoStreams = new List<Stream>();
            availableAudioStreams = new List<Stream>();
            availableTextStreams = new List<Stream>();
   
            try
            {
                for (int i = 0; i<manifestObject.AvailableStreams.Count; i++)
                {
                    Stream newStream = new Stream(manifestObject.AvailableStreams[i]);
                    newStream.isChecked = false;
   
                    //populate the stream lists based on the types
                    availableStreams.Add(newStream);
   
                    switch (newStream.ManifestStream.Type)
                    {
                        case MediaStreamType.Video:
                            availableVideoStreams.Add(newStream);
                            break;
                        case MediaStreamType.Audio:
                            availableAudioStreams.Add(newStream);
                            break;
                        case MediaStreamType.Text:
                            availableTextStreams.Add(newStream);
                            break;
                    }
   
                    // Select the default selected streams from the manifest.
                    for (int j = 0; j<manifestObject.SelectedStreams.Count; j++)
                    {
                        string selectedStreamName = manifestObject.SelectedStreams[j].Name;
                        if (selectedStreamName.Equals(newStream.Name))
                        {
                            newStream.isChecked = true;
                            break;
                        }
                    }
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function set the list box ItemSource
        private async void refreshAvailableStreamsListBoxItemSource()
        {
            try
            {
                //update the stream check box list on the UI
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableStreams.ItemsSource = availableStreams; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
   
        // This function creates a selected streams list
        private void createSelectedStreamsList(List<IManifestStream> selectedStreams)
        {
            bool isOneVideoSelected = false;
            bool isOneAudioSelected = false;
   
            // Only one video stream can be selected
            for (int j = 0; j<availableVideoStreams.Count; j++)
            {
                if (availableVideoStreams[j].isChecked && (!isOneVideoSelected))
                {
                    selectedStreams.Add(availableVideoStreams[j].ManifestStream);
                    isOneVideoSelected = true;
                }
            }
   
            // Select the frist video stream from the list if no video stream is selected
            if (!isOneVideoSelected)
            {
                availableVideoStreams[0].isChecked = true;
                selectedStreams.Add(availableVideoStreams[0].ManifestStream);
            }
   
            // Only one audio stream can be selected
            for (int j = 0; j<availableAudioStreams.Count; j++)
            {
                if (availableAudioStreams[j].isChecked && (!isOneAudioSelected))
                {
                    selectedStreams.Add(availableAudioStreams[j].ManifestStream);
                    isOneAudioSelected = true;
                    txtStatus.Text = "The audio stream is changed to " + availableAudioStreams[j].ManifestStream.Name;
                }
            }
   
            // Select the frist audio stream from the list if no audio steam is selected.
            if (!isOneAudioSelected)
            {
                availableAudioStreams[0].isChecked = true;
                selectedStreams.Add(availableAudioStreams[0].ManifestStream);
            }
   
            // Multiple text streams are supported.
            for (int j = 0; j < availableTextStreams.Count; j++)
            {
                if (availableTextStreams[j].isChecked)
                {
                    selectedStreams.Add(availableTextStreams[j].ManifestStream);
                }
            }
        }
   
        // Change streams on a smooth streaming presentation with multiple video streams.
        private async void changeStreams(List<IManifestStream> selectStreams)
        {
            try
            {
                IReadOnlyList<IStreamChangedResult> returnArgs =
                    await manifestObject.SelectStreamsAsync(selectStreams);
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }
        }
        #endregion stream selection
5. 找到 mediaElement_ManifestReady 方法，並在函數的結尾附加下列程式碼：
   
        getStreams(manifestObject);
        refreshAvailableStreamsListBoxItemSource();
   
    因此，當 MediaElement 資訊清單就緒時，程式碼會取得可用資料流清單，並將這份清單填入 UI 清單方塊。
6. 在 MainPage 類別中，找到 UI 按鈕並按一下事件區域，再新增下列函式定義：

   
        private void btnChangeStream_Click(object sender, RoutedEventArgs e)
        {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();
   
            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);
   
            // Change streams on the presentation
            changeStreams(selectedStreams);
        }

**編譯和測試應用程式**

1. 按 **F6** 鍵編譯專案。 
2. 按 **F5** 鍵執行應用程式。
3. 在應用程式頂端，您可以使用預設 Smooth Streaming URL，或輸入不同的 Smooth Streaming URL。 
4. 按一下 [設定來源] 。 
5. 預設語言為 audio_eng。 嘗試在 audio_eng 與 audio_es 之間切換。 每次您選取新的資料流時，都必須按一下 [提交] 按鈕。

您已完成課程 3。  在本課程中，您新增了選擇資料流的功能。

## <a name="lesson-4-select-smooth-streaming-tracks"></a>課程 4：選取 Smooth Streaming 曲目
Smooth Streaming 簡報可以包含多個以不同品質等級 (位元速率) 和解析度編碼的視訊檔案。 在本課程中，您將讓使用者選取曲目。 本課程包含下列程序：

1. 修改 XAML 檔案
2. 修改程式碼後置檔案
3. 編譯和測試應用程式

**修改 XAML 檔案**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視設計工具]。
2. 找到名稱為 **gridStreamAndBitrateSelection** 的 &lt;Grid&gt; 標籤，並在標籤的結尾附加下列程式碼：
   
         <StackPanel Name="spBitRateSelection" Grid.Row="1" Grid.Column="1">
         <StackPanel Orientation="Horizontal">
             <TextBlock Name="tbBitRate" Text="Available Bitrates:" FontSize="16" VerticalAlignment="Center"/>
             <Button Name="btnChangeTracks" Content="Submit" Click="btnChangeTrack_Click" />
         </StackPanel>
         <ListBox x:Name="lbAvailableVideoTracks" Height="200" Width="200" Background="Transparent" HorizontalAlignment="Left" 
                  ScrollViewer.VerticalScrollMode="Enabled" ScrollViewer.VerticalScrollBarVisibility="Visible">
             <ListBox.ItemTemplate>
                 <DataTemplate>
                     <CheckBox Content="{Binding Bitrate}" IsChecked="{Binding isChecked, Mode=TwoWay}"/>
                 </DataTemplate>
             </ListBox.ItemTemplate>
         </ListBox>
         </StackPanel>
3. 按 **CTRL+S** 儲存變更。

**修改程式碼後置檔案**

1. 從 [方案總管] 中，在 [MainPage.xaml] 上按一下滑鼠右鍵，然後按一下 [檢視程式碼]。
2. 在 SSPlayer 命名空間內，新增類別：
   
        #region class Track
        public class Track
        {
            private IManifestTrack trackInfo;
            public string _bitrate;
            public bool isCheckedValue;
   
            public IManifestTrack TrackInfo
            {
                get { return trackInfo; }
                set { trackInfo = value; }
            }
   
            public string Bitrate
            {
                get { return _bitrate; }
                set { _bitrate = value; }
            }
   
            public bool isChecked
            {
                get { return isCheckedValue; }
                set
                {
                    isCheckedValue = value;
                }
            }
   
            public Track(IManifestTrack trackInfoIn)
            {
                trackInfo = trackInfoIn;
                _bitrate = trackInfoIn.Bitrate.ToString();
            }
            //public Track() { }
        }
        #endregion class Track
3. 在 MainPage 類別的開頭，新增下列變數定義：
   
        private List<Track> availableTracks;
4. 在 MainPage 類別內，新增下列區域：
   
        #region track selection
        /// <summary>
        /// Functionality to select video streams
        /// </summary>
   
        /// This Function gets the tracks for the selected video stream
        public void getTracks(Manifest manifestObject)
        {
            availableTracks = new List<Track>();
   
            IManifestStream videoStream = getVideoStream();
            IReadOnlyList<IManifestTrack> availableTracksLocal = videoStream.AvailableTracks;
            IReadOnlyList<IManifestTrack> selectedTracksLocal = videoStream.SelectedTracks;
   
            try
            {
                for (int i = 0; i < availableTracksLocal.Count; i++)
                {
                    Track thisTrack = new Track(availableTracksLocal[i]);
                    thisTrack.isChecked = true;
   
                    for (int j = 0; j < selectedTracksLocal.Count; j++)
                    {
                        string selectedTrackName = selectedTracksLocal[j].Bitrate.ToString();
                        if (selectedTrackName.Equals(thisTrack.Bitrate))
                        {
                            thisTrack.isChecked = true;
                            break;
                        }
                    }
                    availableTracks.Add(thisTrack);
                }
            }
            catch (Exception e)
            {
                txtStatus.Text = e.Message;
            }
        }
   
        // This function gets the video stream that is playing
        private IManifestStream getVideoStream()
        {
            IManifestStream videoStream = null;
            for (int i = 0; i < manifestObject.SelectedStreams.Count; i++)
            {
                if (manifestObject.SelectedStreams[i].Type == MediaStreamType.Video)
                {
                    videoStream = manifestObject.SelectedStreams[i];
                    break;
                }
            }
            return videoStream;
        }
   
        // This function set the UI list box control ItemSource
        private async void refreshAvailableTracksListBoxItemSource()
        {
            try
            {
                // Update the track check box list on the UI 
                await _dispatcher.RunAsync(CoreDispatcherPriority.Normal, ()
                    => { lbAvailableVideoTracks.ItemsSource = availableTracks; });
            }
            catch (Exception e)
            {
                txtStatus.Text = "Error: " + e.Message;
            }        
        }
   
        // This function creates a list of the selected tracks.
        private void createSelectedTracksList(List<IManifestTrack> selectedTracks)
        {
            // Create a list of selected tracks
            for (int j = 0; j < availableTracks.Count; j++)
            {
                if (availableTracks[j].isCheckedValue == true)
                {
                    selectedTracks.Add(availableTracks[j].TrackInfo);
                }
            }
        }
   
        // This function selects the tracks based on user selection 
        private void changeTracks(List<IManifestTrack> selectedTracks)
        {
            IManifestStream videoStream = getVideoStream();
            try
            {
                videoStream.SelectTracks(selectedTracks);
            }
            catch (Exception ex)
            {
                txtStatus.Text = ex.Message;
            }
        }
        #endregion track selection
5. 找到 mediaElement_ManifestReady 方法，並在函數的結尾附加下列程式碼：
   
         getTracks(manifestObject);
         refreshAvailableTracksListBoxItemSource();
6. 在 MainPage 類別中，找到 UI 按鈕並按一下事件區域，再新增下列函式定義：

   
         private void btnChangeStream_Click(object sender, RoutedEventArgs e)
         {
            List<IManifestStream> selectedStreams = new List<IManifestStream>();

            // Create a list of the selected streams
            createSelectedStreamsList(selectedStreams);

            // Change streams on the presentation
            changeStreams(selectedStreams);
         }

**編譯和測試應用程式**

1. 按 **F6** 鍵編譯專案。 
2. 按 **F5** 鍵執行應用程式。
3. 在應用程式頂端，您可以使用預設 Smooth Streaming URL，或輸入不同的 Smooth Streaming URL。 
4. 按一下 [設定來源] 。 
5. 預設會選取視訊資料流的所有曲目。 若要試驗位元速率變更，您可以選取可用的最低位元速率，然後選取可用的最高位元速率。 您必須在每次變更之後按一下 [提交]。  您可以看到視訊品質變更。

您已完成課程 4。  在本課程中，您新增了選擇追蹤的功能。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="other-resources"></a>其他資源：
* [如何建置具有進階功能的 Smooth Streaming Windows 8 JavaScript 應用程式 (英文)](http://blogs.iis.net/cenkd/archive/2012/08/10/how-to-build-a-smooth-streaming-windows-8-javascript-application-with-advanced-features.aspx)
* [Smooth Streaming 技術概觀 (英文)](http://www.iis.net/learn/media/on-demand-smooth-streaming/smooth-streaming-technical-overview)

[PlayerApplication]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-1.png
[CodeViewPic]: ./media/media-services-build-smooth-streaming-apps/SSClientWin8-2.png

