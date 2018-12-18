---
title: Open Source Media Framework 的 Smooth Streaming 外掛程式
description: 了解如何使用 Adobe Open Source Media Framework 的 Azure 媒體服務 Smooth Streaming 外掛程式。
services: media-services
documentationcenter: ''
author: juliako
manager: cfowler
editor: ''
ms.assetid: 6068151f-b6b0-4507-9346-f03416d3d572
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2016
ms.author: juliako
ms.openlocfilehash: 6a4ae0697e59eb00e216fc29b154408e4d7a99b8
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33785917"
---
# <a name="how-to-use-the-microsoft-smooth-streaming-plugin-for-the-adobe-open-source-media-framework"></a>如何使用 Adobe Open Source Media Framework 的 Microsoft Smooth Streaming 外掛程式
## <a name="overview"></a>概觀
Open Source Media Framework 2.0 的 Microsoft Smooth Streaming 外掛程式 (SS for OSMF) 可擴充 OSMF 的預設功能，並可為新的和現有的 OSMF 播放程式新增 Microsoft Smooth Streaming 內容播放功能。 此外掛程式也可為 Strobe Media Playback (SMP) 新增 Smooth Streaming 播放功能。

SS for OSMF 包含兩個外掛程式版本：

* OSMF 的靜態 Smooth Streaming 外掛程式 (.swc)
* OSMF 的動態 Smooth Streaming 外掛程式 (.swf)

本文假設讀者具備使用 OSMF 和 OSMF 外掛程式的一般知識。如需 OSMF 的詳細資訊，請參閱 [OSMF 官方網站](http://osmf.org/)上的文件。

### <a name="smooth-streaming-plugin-for-osmf-20"></a>OSMF 2.0 的 Smooth Streaming 外掛程式
此外掛程式支援以下列功能載入及播放隨選的 Smooth Streaming 內容：

* 隨選 Smooth Streaming 播放 (播放、暫停、搜尋、停止)
* 即時 Smooth Streaming 播放 (播放)
* 即時 DVR 功能 (暫停、搜尋、DVR 播放、移至即時)
* 視訊轉碼器的支援 - H.264
* 音訊轉碼器的支援 - AAC
* 透過 OSMF 內建 API 的多重音訊語言切換
* 透過 OSMF 內建 API 的最高播放品質選取
* 透過 OSMF 字幕外掛程式的 Sidecar 隱藏式輔助字幕
* Adobe&reg; Flash&reg; Player 11.4 或更高版本。
* 此版本僅支援 OSMF 2.0。

## <a name="supported-features-and-known-issues"></a>支援的功能和已知問題
如需支援功能、不支援功能和已知問題的完整清單，請參閱 [這份文件](http://download.microsoft.com/download/3/1/B/31B63D97-574E-4A8D-BF8D-170744181724/Smooth_Streaming_Plugin_for_OSMF.pdf)。

## <a name="loading-the-plugin"></a>載入外掛程式
OSMF 外掛程式可以靜態方式 (在編譯時) 或動態方式 (在執行階段) 載入。 OSMF 的 Smooth Streaming 外掛程式下載，會同時包含動態和靜態版本。

* 靜態載入：若要以靜態方式載入，必須要有靜態程式庫 (SWC) 檔案。 靜態外掛程式會在編譯時新增為專案的參考，並合併到最終的輸出檔內。
* 動態載入：若要以動態方式載入，必須要有預先編譯的 (SWF) 檔案。 動態外掛程式會在執行階段載入，而不會包含在專案輸出中。 (編譯的輸出) 動態外掛程式可使用 HTTP 和 FILE 通訊協定來載入。

如需靜態和動態載入的詳細資訊，請參閱官方 [OSMF 外掛程式頁面](http://osmf.org/dev/osmf/OtherPDFs/osmf_plugin_dev_guide.pdf)。

### <a name="ss-for-osmf-static-loading"></a>SS for OSMF 靜態載入
下方的程式碼片段將說明如何以靜態方式載入 OSMF 的 SS 外掛程式，並使用 OSMF MediaFactory 類別播放基本視訊。 在加入 SS for OSMF 程式碼之前，請確定專案參考包含 "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swc" 靜態外掛程式。

```
package 
{

    import com.microsoft.azure.media.AdaptiveStreamingPluginInfo;

    import flash.display.*;
    import org.osmf.media.*;
    import org.osmf.containers.MediaContainer;
    import org.osmf.events.MediaErrorEvent;
    import org.osmf.events.MediaFactoryEvent;
    import org.osmf.events.MediaPlayerStateChangeEvent;
    import org.osmf.layout.*;



    [SWF(width="1024", height="768", backgroundColor='#405050', frameRate="25")]
    public class TestPlayer extends Sprite
    {        
        public var _container:MediaContainer;
        public var _mediaFactory:DefaultMediaFactory;
        private var _mediaPlayerSprite:MediaPlayerSprite;


        public function TestPlayer( )
        {
            stage.quality = StageQuality.HIGH;

            initMediaPlayer();

        }

        private function initMediaPlayer():void
        {

            // Create the container (sprite) for managing display and layout
            _mediaPlayerSprite = new MediaPlayerSprite();    
            _mediaPlayerSprite.addEventListener(MediaErrorEvent.MEDIA_ERROR, onPlayerFailed);
            _mediaPlayerSprite.addEventListener(MediaPlayerStateChangeEvent.MEDIA_PLAYER_STATE_CHANGE, onPlayerStateChange);
            _mediaPlayerSprite.scaleMode = ScaleMode.NONE;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;
            //Adds the container to the stage
            addChild(_mediaPlayerSprite);

            // Create a mediafactory instance
            _mediaFactory = new DefaultMediaFactory();

            // Add the listeners for PLUGIN_LOADING
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD,onPluginLoaded);
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD_ERROR, onPluginLoadFailed );

            // Load the plugin class 
            loadAdaptiveStreamingPlugin( );  

        }

        private function loadAdaptiveStreamingPlugin( ):void
        {
            var pluginResource:MediaResourceBase;

            pluginResource = new PluginInfoResource(new AdaptiveStreamingPluginInfo( )); 
            _mediaFactory.loadPlugin( pluginResource ); 
        }

        private function onPluginLoaded( event:MediaFactoryEvent ):void
        {
            // The plugin is loaded successfully.
            // Your web server needs to host a valid crossdomain.xml file to allow plugin to download Smooth Streaming files.
        loadMediaSource("http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest")

        }

        private function onPluginLoadFailed( event:MediaFactoryEvent ):void
        {
            // The plugin is failed to load ...
        }


        private function onPlayerStateChange(event:MediaPlayerStateChangeEvent) : void
        {
            var state:String;

            state =  event.state;

            switch (state)
            {
                case MediaPlayerState.LOADING: 

                    // A new source is started to load.

                    break;

                case  MediaPlayerState.READY :   
                    // Add code to deal with Player Ready when it is hit the first load after a source is loaded. 

                    break;

                case MediaPlayerState.BUFFERING :

                    break;

                case  MediaPlayerState.PAUSED :
                    break;      
                // other states ...          
            }
        }

        private function onPlayerFailed(event:MediaErrorEvent) : void
        {
            // Media Player is failed .           
        }

        private function loadMediaSource(sourceURL : String):void 
        {
            // Take an URL of SmoothStreamingSource's manifest and add it to the page.

            var resource:URLResource= new URLResource( sourceURL );

            var element:MediaElement = _mediaFactory.createMediaElement( resource );
            _mediaPlayerSprite.scaleMode = ScaleMode.LETTERBOX;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;

            // Add the media element
            _mediaPlayerSprite.media = element;
        }     

    }
}
```


### <a name="ss-for-osmf-dynamic-loading"></a>SS for OSMF 動態載入
下方的程式碼片段將說明如何以動態方式載入 OSMF 的 SS 外掛程式，並使用 OSMF MediaFactory 類別播放基本視訊。 在加入 SS for OSMF 程式碼之前，請將 "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf" 動態外掛程式複製到專案資料夾 (如果您要使用 FILE 通訊協定進行載入)，或是在 Web 伺服器下複製 (以進行 HTTP 載入)。 您不需要在專案參考中加入 "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swc"。

package {

    import flash.display.*;
    import org.osmf.media.*;
    import org.osmf.containers.MediaContainer;
    import org.osmf.events.MediaErrorEvent;
    import org.osmf.events.MediaFactoryEvent;
    import org.osmf.events.MediaPlayerStateChangeEvent;
    import org.osmf.layout.*;
    import flash.events.Event;
    import flash.system.Capabilities;


    //Sets the size of the SWF

    [SWF(width="1024", height="768", backgroundColor='#405050', frameRate="25")]
    public class TestPlayer extends Sprite
    {        
        public var _container:MediaContainer;
        public var _mediaFactory:DefaultMediaFactory;
        private var _mediaPlayerSprite:MediaPlayerSprite;


        public function TestPlayer( )
        {
            stage.quality = StageQuality.HIGH;
            initMediaPlayer();
        }

        private function initMediaPlayer():void
        {

            // Create the container (sprite) for managing display and layout
            _mediaPlayerSprite = new MediaPlayerSprite();    
            _mediaPlayerSprite.addEventListener(MediaErrorEvent.MEDIA_ERROR, onPlayerFailed);
            _mediaPlayerSprite.addEventListener(MediaPlayerStateChangeEvent.MEDIA_PLAYER_STATE_CHANGE, onPlayerStateChange);

            //Adds the container to the stage
            addChild(_mediaPlayerSprite);

            // Create a mediafactory instance
            _mediaFactory = new DefaultMediaFactory();

            // Add the listeners for PLUGIN_LOADING
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD,onPluginLoaded);
            _mediaFactory.addEventListener(MediaFactoryEvent.PLUGIN_LOAD_ERROR, onPluginLoadFailed );

            // Load the plugin class 
            loadAdaptiveStreamingPlugin( );  

        }

        private function loadAdaptiveStreamingPlugin( ):void
        {
            var pluginResource:MediaResourceBase;
            var adaptiveStreamingPluginUrl:String;

            // Your dynamic plugin web server needs to host a valid crossdomain.xml file to allow loading plugins.

            adaptiveStreamingPluginUrl = "http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf";
            pluginResource = new URLResource(adaptiveStreamingPluginUrl);
            _mediaFactory.loadPlugin( pluginResource ); 

        }

        private function onPluginLoaded( event:MediaFactoryEvent ):void
        {
            // The plugin is loaded successfully.

            // Your web server needs to host a valid crossdomain.xml file to allow plugin to download Smooth Streaming files.

    loadMediaSource("http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest")
        }

        private function onPluginLoadFailed( event:MediaFactoryEvent ):void
        {
            // The plugin is failed to load ...
        }


        private function onPlayerStateChange(event:MediaPlayerStateChangeEvent) : void
        {
            var state:String;

            state =  event.state;

            switch (state)
            {
                case MediaPlayerState.LOADING: 

                    // A new source is started to load.

                    break;

                case  MediaPlayerState.READY :   
                    // Add code to deal with Player Ready when it is hit the first load after a source is loaded. 

                    break;

                case MediaPlayerState.BUFFERING :

                    break;

                case  MediaPlayerState.PAUSED :
                    break;      
                // other states ...          
            }
        }

        private function onPlayerFailed(event:MediaErrorEvent) : void
        {
            // Media Player is failed .           
        }

        private function loadMediaSource(sourceURL : String):void 
        {
            // Take an URL of SmoothStreamingSource's manifest and add it to the page.

            var resource:URLResource= new URLResource( sourceURL );

            var element:MediaElement = _mediaFactory.createMediaElement( resource );
            _mediaPlayerSprite.scaleMode = ScaleMode.LETTERBOX;
            _mediaPlayerSprite.width = stage.stageWidth;
            _mediaPlayerSprite.height = stage.stageHeight;
            // Add the media element
            _mediaPlayerSprite.media = element;
        }     

    }
}

## <a name="strobe-media--playback-with-the-ss-odmf-dynamic-plugin"></a>Strobe Media Playback 與 SS ODMF 動態外掛程式
Smooth Streaming for OSMF 動態外掛程式與 [Strobe Media Playback (SMP)](http://osmf.org/strobe_mediaplayback.html)是相容的。 您可以使用 SS for OSMF 外掛程式，將 Smooth Streaming 內容播放新增至 SMP。 若要這麼做，請使用下列步驟，在 Web 伺服器下複製 "MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf"，以進行 HTTP 載入：

1. 瀏覽 [Strobe Media Playback 設定頁面](http://osmf.org/dev/2.0gm/setup.html)。 
2. 將 src 設為 Smooth Streaming 來源，(例如， http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest) 
3. 進行所需的組態變更，然後按一下 [Preview and Update]。
   
   **注意** 您的內容 Web 伺服器需要有效的 crossdomain.xml。 
4. 使用您慣用的文字編輯器，將程式碼複製並貼至簡單的 HTML 頁面，如下列範例所示：

        <html>
        <body>
        <object width="920" height="640"> 
        <param name="movie" value="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf"></param>
        <param name="flashvars" value="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest &autoPlay=true"></param>
        <param name="allowFullScreen" value="true"></param>
        <param name="allowscriptaccess" value="always"></param>
        <param name="wmode" value="direct"></param>
        <embed src="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf" 
            type="application/x-shockwave-flash" 
            allowscriptaccess="always" 
            allowfullscreen="true" 
            wmode="direct" 
            width="920" 
            height="640" 
            flashvars=" src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true">
        </embed>
        </object>
        </body>
        </html>



1. 將 Smooth Streaming OSMF 外掛程式新增至內嵌程式碼，並加以儲存。
   
        <html>
        <object width="920" height="640"> 
        <param name="movie" value="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf"></param>
        <param name="flashvars" value="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true&plugin_AdaptiveStreamingPlugin=http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf&AdaptiveStreamingPlugin_retryLive=true&AdaptiveStreamingPlugin_retryInterval=10"></param>
        <param name="allowFullScreen" value="true"></param>
        <param name="allowscriptaccess" value="always"></param>
        <param name="wmode" value="direct"></param>
        <embed src="http://osmf.org/dev/2.0gm/StrobeMediaPlayback.swf" 
            type="application/x-shockwave-flash" 
            allowscriptaccess="always" 
            allowfullscreen="true" 
            wmode="direct" 
            width="920" 
            height="640" 
            flashvars="src=http://devplatem.vo.msecnd.net/Sintel/Sintel_H264.ism/manifest&autoPlay=true&plugin_AdaptiveStreamingPlugin=http://yourdomain/MSAdaptiveStreamingPlugin-v1.0.3-osmf2.0.swf&AdaptiveStreamingPlugin_retryLive=true&AdaptiveStreamingPlugin_retryInterval=10">
        </embed>
        </object>
        </html>
2. 儲存您的 HTML 頁面，並發佈至 Web 伺服器。 使用您慣用且具有 Flash&reg; Player 功能的網際網路瀏覽器 (Internet Explorer、Chrome、Firefox 等)，瀏覽至已發佈的網頁。
3. 在 Adobe&reg; Flash&reg; Player 內欣賞 Smooth Streaming 內容。

如需一般 OSMF 開發的詳細資訊，請參閱官方 [OSMF 開發頁面](http://osmf.org/resources.html)。

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="see-also"></a>另請參閱
[OSMF 的 Microsoft 彈性資料流外掛程式更新 (英文)](https://azure.microsoft.com/blog/2014/10/27/microsoft-adaptive-streaming-plugin-for-osmf-update/) 

