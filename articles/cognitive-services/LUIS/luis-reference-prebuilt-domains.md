---
title: 預先建立的定義域參考 - Azure| Microsoft Docs
titleSuffix: Azure
description: 預先建立的定義域參考是從 Language Understanding Intelligent Service (LUIS) 預先建立的意圖和實體集合。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 06/20/2018
ms.author: diberry
ms.openlocfilehash: f424abd339bda6c29079e3d286cbe96593f16f52
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47040035"
---
# <a name="prebuilt-domain-reference"></a>預先建置的網域參考
此參考提供[預先建立的定義域](luis-how-to-use-prebuilt-domains.md)相關資訊，這是 LUIS 提供之預先建立的意圖和實體集合。

相反地，[自訂網域](luis-how-to-start-new-app.md)一開始沒有任何意圖和模型。 您可以將任何預先建立的定義域意圖和實體新增至自訂模型。

## <a name="list-of-prebuilt-domains"></a>預先建立的定義域清單
LUIS 提供 20 個預先建立的定義域。 

| 預建網域 | 說明 | 支援的語言 |
| ---------------- |-----------------------|:------:|
| 行事曆 | Calendar 定義域提供新增、刪除或編輯約會，檢查參與者是否可與會，以及尋找行事曆事件相關資訊的意圖和實體。| zh-TW<br/> zh-CN |
| Camera | Camera 定義域提供拍照、錄製視訊及廣播視訊到應用程式的意圖和實體。| zh-TW |
| 通訊 | 傳送訊息及撥打電話。| zh-TW <br/> zh-CN |
| Entertainment  | 處理與音樂、電影和電視節目相關的查詢。| zh-TW |
| 活動 | 預約演唱會、運動賽事和喜劇表演門票。| zh-TW |
| Fitness | 處理與追蹤健身活動相關的要求。| zh-TW |
| 玩遊戲 | 處理與多玩家遊戲中遊戲陣營相關的要求。| zh-TW |
| HomeAutomation | 控制燈光與家電用品等智慧型家電裝置。| zh-TW<br/> zh-CN |
| MovieTickets | 預約電影院的電影票。| zh-TW |
| 音樂 | 在音樂播放程式上播放音樂。| zh-TW<br/> zh-CN |
| 附註 | Note 定義域提供與建立、編輯及尋找筆記相關的意圖和實體。| zh-TW<br/> zh-CN |
| OnDevice | OnDevice 定義域提供與控制裝置相關的意圖和實體。| zh-TW<br/> zh-CN |
| 地點  | 處理與公司、機構、餐廳、公共空間和地址等地點相關的查詢。| zh-TW<br/> zh-CN |
| 提醒 | 處理與建立、編輯及尋找提醒相關的要求。| zh-TW<br/> zh-CN |
| RestaurantReservation | 處理管理餐廳預約的要求。| zh-TW<br/> zh-CN |
| Taxi | 處理計程車的預約。| zh-TW<br/> zh-CN |
| 翻譯 | 將文字翻譯成目標語言。| zh-TW<br/> zh-CN |
| TV | 控制電視。| zh-TW |
| 公用事業  | 處理許多定義域通用的要求，例如「說明」、「重複」、「重新開始」。| zh-TW |
| Weather | 取得天氣預報。| zh-TW<br/> zh-CN |
| Web | 巡覽至網站。| zh-TW<br/> zh-CN |

如需每個定義域的詳細資料，請參閱下列各節。

## <a name="calendar"></a>行事曆 

Calendar 定義域提供與行事曆項目相關的意圖和實體。 Calendar 意圖包括新增、刪除或編輯約會，檢查是否可與會，以及尋找行事曆項目或約會的相關資訊。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 加 | 新增一次性項目至行事曆。| 建立與 Lisa 在星期日下午 2 點的約會 <br/><br/>我想要排程會議<br/><br/>我需要設定會議|
| CheckAvailability | 在使用者的行事曆或其他人的行事曆上，找到可與會的約會或會議時段。| 何時可以見到 Jim？ <br/><br/>顯示 Carol 明天何時有空<br/><br/>Chris 星期六有空嗎？|
| 刪除 | 要求刪除行事曆項目。| 取消我與 Carol 的約會。 <br/><br/>刪除我上午 9 點的會議<br/>|
| Edit | 要求變更現有的會議或行事曆項目。| 將我上午 9 點的會議移到上午 10 點。<br/><br/>我想要更新排程。<br/><br/>重新排程我與 Ryan 的會議。|
| 尋找 | 顯示我的每週行事曆。| 尋找牙醫回診。 <br/><br/>顯示我的行事曆<br/>|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 位置 | 行事曆項目、會議或約會的位置。 地址、城市和地區都是很好的位置範例。| 209 Nashville Gym <br/><br/>897 Pancake house<br/><br/>車庫|
| 主體 | 會議或約會的標題。| 牙醫門診 <br/><br/>與 Julia 共進午餐<br/><br/>醫生門診|

## <a name="camera"></a>Camera 
Camera 定義域提供與使用相機相關的意圖和實體。 這些意圖涵蓋擷取相片、自拍、螢幕擷取畫面或視訊，以及廣播視訊到應用程式。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| CapturePhoto| 擷取相片。| 拍照<br/><br/>擷取|
| CaptureScreenshot | 擷取螢幕擷取畫面。| 拍下螢幕畫面。<br/><br/>擷取螢幕畫面。|
| CaptureSelfie | 擷取自拍。| 自拍 <br/><br/>拍下我自己的照片 |
| CaptureVideo | 開始錄製視訊。| 開始錄製 <br/><br/>開始錄製|
| StartBroadcasting| 開始廣播視訊。| 開始廣播到 Facebook|
| StopBroadcasting| 停止廣播視訊。| 停止廣播|
| StopVideoRecording| 停止錄製視訊。| 已經夠了<br/><br/>停止錄製|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AppName | 要廣播視訊的目標應用程式名稱。| OneNote<br/><br/>Facebook<br/><br/>Skype|


## <a name="communication"></a>通訊 
Communication 定義域提供與電子郵件、訊息和電話相關的意圖和實體。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AddContact| 新增連絡人至使用者的連絡人清單。|新增連絡人 <br/><br/>儲存此號碼並將名稱設為 Carol|
| AddMore| 新增更多內容至電子郵件或簡訊，作為逐步電子郵件或簡訊撰寫的一部分。|新增更多內容至簡訊 <br/><br/>新增更多內容至電子郵件內文|
| Answer| 接聽來電。|接聽來電 <br/><br/>拿起聽筒|
| AssignContactNickname| 為連絡人指派暱稱。|將 Isaac 變更為爸爸 <br/>編輯 Jim 的暱稱<br/>新增 Patti Owens 的暱稱|
| CallVoiceMail| 連線到使用者的語音信箱。|讓我連線到自己的語音信箱 <br/>語音信箱<br/>撥打語音信箱|
| CheckIMStatus| 檢查 Skype 中的連絡人狀態。|Jim 的連線狀態是否設定為離開？ <br/>Carol 是否有空交談？|
| 確認| 確認動作。|是<br/>確定<br/>好的<br/>我確認要傳送此電子郵件。<br/>|
| 撥號| 撥打電話。|撥打電話給 Jim<br/>請撥打 311<br/>|
| FindContact| 依名稱尋找連絡人資訊。|尋找 Carol 的號碼<br/>顯示 Carol 的號碼<br/>|
| FindSpeedDial| 尋找電話號碼設成的快速撥號，或尋找設為快速撥號的電話號碼。|我的撥號 5 為何？<br/>我是否已設定快速撥號？<br/>941-5555-333 的快速撥號為何？|
| GetForwardingsStatus| 取得來電轉接的目前狀態。|是否啟動我的來電轉接？<br/>告訴我撥號狀態為開啟或關閉<br/>|
| Goback| 返回上一個步驟。|返回 Twitter<br/>返回一個步驟<br/>返回|
| 略過| 略過來電。|不接聽<br/>略過來電|
| IgnoreWithMessage| 略過來電並改以簡訊回覆。|不接聽該通來電但改為傳送訊息。<br/>略過並傳回簡訊。|
| PressKey| 按下按鈕或數字鍵台上的數字。|撥打米字鍵。<br/>按 1 2 3。|
| ReadAloud| 對使用者朗讀訊息或電子郵件。|朗讀簡訊。<br/>她在訊息中說了什麼？|
| TurnForwardingOff| 撥打電話。|<br/><br/>|
| Redial| 重撥或再次撥打某個號碼。|重撥。<br/>重撥我的上一通電話。|
| 拒絕| 拒絕來電。|拒絕來電<br/>目前無法接聽<br/>目前無法接聽，稍後將回電。|
| SendEmail| 傳送電子郵件。 此意圖適用於電子郵件，但不適用於文字簡訊。|傳送電子郵件給 Mike Waters：Mike，上週的晚餐很棒。<br/>傳送電子郵件給 Bob<br/>|
| SendMessage| 傳送文字簡訊或立即訊息。|傳送簡訊給 Chris 和 Carol|
| SetSpeedDial| 設定連絡人電話號碼的快速撥號快速鍵。|將 Carol 的快速撥號設定為 1。<br/>設定媽媽的快速撥號。|
| ShowNext| 查看下一個項目，例如文字簡訊或電子郵件清單中的下一個項目。|顯示下一個。<br/>移至下一頁。|
| ShowPrevious| 查看上一個項目，例如文字簡訊或電子郵件清單中的上一個項目。|顯示上一個。<br/>上一步<br/>移至上一個。|
| StartOver| 重新啟動系統或啟動新的工作階段。|重新開始<br/>新增工作階段<br/>restart|
| TurnForwardingOff| 關閉來電轉接。|停止轉接我的來電<br/>關閉來電轉接|
| TurnForwardingOn| 關閉免持通話器。|將我的來電轉接至 3333<br/>開啟來電轉接至 3333|
| TurnSpeakerOff| 關閉免持聽筒。|結束擴音器。<br/>關閉免持聽筒。<br/>|
| TurnSpeakerOn| 啟動免持聽筒。|免持聽筒模式。<br/>將免持聽筒設定為開啟。<br/>|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AudioDeviceType | 音訊裝置類型 (擴音器、耳機、麥克風等)。| 主講人<br/>免持聽筒<br/>藍牙|
| 類別 | 訊息或電子郵件的類別。| 重要事項<br/>高優先順序|
| ContactAttribute | 使用者查詢的連絡人屬性。| 生日<br/>位址<br/>電話號碼|
| ContactName | 連絡人或訊息收件者的名稱。| Carol<br/>Jim<br/>Chris|
| EmailSubject | 作為電子郵件主旨列的文字。| RE: 有趣的故事|
| 折線圖 | 使用者想要用來撥打電話或傳送簡訊/電子郵件的來源線路。| 工作線路<br/>英國行動電話<br/>Skype|
| 訊息 | 要作為電子郵件或簡訊傳送的訊息。| 今天很高興見到您。 希望很快再次見到您！|
| MessageType | 連絡人或訊息收件者的名稱。| 文字<br/>電子郵件|
| OrderReference | 清單中的序數或相關位置，用於識別要擷取的項目。 例如，「我傳送的最後一則訊息為何？」中的「最後一個」或「最近」| 最後一頁<br/>最近|
| SenderName | 寄件者的名稱。| Patti Owens|

## <a name="entertainment"></a>Entertainment  
Entertainment 定義域提供與搜尋電影、音樂、遊戲和電視節目相關的意圖和實體。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| Search| 搜尋電影、音樂、應用程式、遊戲和電視節目。|搜尋販售 Halo 的網上商店。<br/>搜尋虛擬人偶。|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| ContentRating | 媒體內容分級，例如電影的 G 或 R。|適合兒童的視訊。<br/>分級為 PG。|
| Genre | 電影、遊戲、應用程式或歌曲的內容類型。|喜劇<br/>戲劇<br/>有趣|
| 關鍵字| 一般搜尋關鍵字，指定更明確的媒體位置中不存在的屬性。|原聲帶<br/>Moon River<br/>Amelia Earhart|
| 語言 | 媒體中所使用的語言，例如，電影或歌曲的口語語言。|法文<br/>English<br/>韓文|
| MediaFormat | 用來格式化媒體的其他特殊技術類型。|HD 電影<br/>3D 電影<br/>可下載|
| MediaSource | 取得媒體的網上商店或市集。|Netflix<br/>Prime|
| MediaSubTypes| 小於電影和遊戲的媒體類型。|示範<br/>Dlc<br/>預告片|
| Nationality| 創作電影、節目或歌曲的國家/地區。|法文<br/>德文<br/>韓文|
| Person| 與電影、應用程式、遊戲或電視節目建立關聯的演員、導演、製作人、音樂家或演出者。|Madonna<br/>Stanley Kubrick|
| 角色| 此人在創作媒體時扮演的角色。|唱歌<br/>導演<br/>依據|
| 標題| 電影、應用程式、遊戲、電視節目或歌曲的名稱。|Friends<br/>Minecraft|
| 類型| 電影、應用程式、遊戲、電視節目或歌曲的類型或媒體格式。|音樂<br/>MovieTV <br/>節目|
| UserRating| 使用者星級或拇指評等。|5 顆星<br/>3 顆星<br/>4 顆星|

## <a name="events"></a>活動 
Events 定義域提供與預約演唱會、慶典、運動賽事和喜劇表演等活動門票相關的意圖和實體。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 書籍| 購買活動門票。|我想要購買本週的交響樂門票。|


### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 位址 | 活動位置或地址。 |Palo Alto<br/>300 112th Ave SE <br/> Seattle |
| 名稱 | 活動的標題。|公園莎士比亞表演|
| PlaceName| 活動位置名稱。|羅浮宮<br/>歌劇院<br/>百老匯|
| PlaceType | 活動舉辦位置的類型。|咖啡廳<br/>劇院<br/>程式庫|
| 類型 | 活動的類型。|演唱會<br/>運動賽事|

## <a name="fitness"></a>Fitness 
Fitness 定義域提供與追蹤健身活動相關的意圖和實體。 這些意圖包括儲存筆記、剩餘時間或距離，或是儲存活動結果。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AddNote| 為追蹤的活動新增補充筆記。|此跑步的難度為 6/10<br/>我跑步的地形為柏油路<br/>我使用 3 變速自行車|
|GetRemaining| 取得活動的剩餘時間或距離。|到下一圈還有多少時間？<br/>我的跑步剩下多少英哩？ 劈腿多少時間？|
| LogActivity| 儲存或記錄已完成的活動結果。|儲存我的上一次跑步<br/>記錄我的星期六晨走<br/>儲存我的上一次游泳|
| LogWeight| 儲存或記錄使用者目前的體重。|儲存我目前的體重<br/>記錄我目前的體重<br/>儲存我目前的體重|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| ActivityType | 要追蹤的活動類型。 |執行<br/>步行<br/>游泳<br/>循環 |
| Food | 要在健身應用程式中追蹤的食物類型。 |Banana<br/>鮭魚<br/>高蛋白奶昔|
| MealType| 要在健康或健身應用程式中追蹤的餐飲類型。|早餐<br/>正餐<br/>午餐<br/>晚餐|
| 測量| 用於健身或健康應用程式之時間、距離或體重的度量類型。|公里<br/>英哩<br/>分鐘<br/>公斤|
| 數字 | 用於健身或健康應用程式的數量。|19<br/>three<br/>200<br/>one|
| StatType | 用於健身或健康應用程式之彙總資料的統計資料類型。|總和<br/>平均值<br/>最大值<br/>最小值|

## <a name="gaming"></a>玩遊戲 
Gaming 定義域提供與管理多玩家遊戲中遊戲陣營相關的意圖和實體。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| InviteParty| 邀請連絡人加入遊戲陣營。|邀請此玩家加入我的陣營<br/>來我的陣營<br/>加入我的公會|
|LeaveParty| 取得活動的剩餘時間或距離。|我要離開<br/>我要離開此陣營並加入另一個陣營<br/>我要退出|
| StartParty| 在多玩家遊戲中成立遊戲陣營。|老兄，讓我們成立一個陣營<br/>成立陣營<br/>今晚要成立公會嗎|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 連絡人| 要用於多玩家遊戲的連絡人名稱。|Carol<br/>Jim|


## <a name="homeautomation"></a>HomeAutomation 
HomeAutomation 定義域提供與控制控制燈光與家電用品等智慧型家電裝置相關的意圖和實體。

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| TurnOff| 將裝置關閉電源、關上或解除鎖定。|關燈<br/>停止咖啡機<br/>關上車庫門|
|TurnOn| 啟動裝置，或是將裝置設定為特定設定或模式。|啟動我的咖啡機<br/>您可以啟動我的咖啡機嗎？<br/>將控溫器設定為 72 度。|


### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 裝置 | 可啟動或關閉的裝置類型。|咖啡機<br/>控溫器<br/>燈光|
| 作業 | 要設定的裝置類型。|lock<br/>開啟<br/>on<br/>關|
| 會議室 | 裝置所在位置或房間。|客廳<br/>臥室<br/>廚房|

## <a name="movietickets"></a>MovieTickets 
MovieTickets 定義域提供與預約電影院之電影票相關的意圖和實體。

### <a name="examples"></a>範例
```
Book me two tickets for Captain Omar and the two Musketeers
Cancel tickets
When is Captain Omar showing?
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 書籍 | 購買電影票。|我想要預約 Captain Omar 和 The Two Musketeers 兩張電影票<br/>我想要購買明天的電影票<br/>我想要下週三的 Captian Omar Part 2 電影票|
|GetShowTime| 取得電影放映時間。|Captain Omar 何時放映？|


### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 位址 | 電影院的地址。|Palo Alto<br/>300 112th Ave SE<br/>Seattle|
| MovieTitle | 電影的片名。|Life of Pi<br/>Hunger Games<br/>Inception|
| PlaceName | 電影院的名稱。|Cinema Amir<br/>Alexandria Theatre<br/>New York Theater|
| PlaceType | 電影放映位置的類型。|電影院<br/>劇場<br/>IMAX 電影院|

## <a name="music"></a>Music 
Music 定義域提供與在音樂播放程式上播放音樂相關的意圖和實體。

### <a name="examples"></a>範例
```
play Beethoven
Increase track volume
Skip to the next song
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| DecreaseVolume | 降低裝置音量。|降低曲目音量<br/>降低音量|
| IncreaseVolume | 提高裝置音量。|提高曲目音量<br/>提高音量|
| Mute |將播放的音樂設為靜音。|將歌曲設為靜音<br/>將曲目設為靜音<br/>將音樂設為靜音 |
| 暫停 | 暫停播放音樂。|暫停<br/>暫停音樂<br/>暫停曲目|
| PlayMusic | 在裝置上播放音樂。|播放 Kevin Durant<br/>播放 Coldplay 的 Paradise<br/>播放 Adele 的 Hello|
| Repeat |重複播放音樂。|重複歌曲<br/>重新播放曲目<br/>重複音樂|
| 繼續 | 繼續播放音樂。|繼續歌曲<br/>重新開始音樂<br/>取消暫停|
| SkipBack | 倒轉跳過曲目。|跳到下一首歌<br/>播放下一首歌|
| SkipForward |快轉跳過曲目。|播放上一首歌<br/>返回上一個曲目 |
| Stop | 停止與音樂播放相關的動作。 |停止播放此專輯。|
| Unmute | 取消音樂播放裝置靜音。| 取消靜音。|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| ArtistName | 與要在裝置上播放之媒體建立關聯的演員、導演、製作人、作家、音樂家或演出者。|Elvis Presley<br/>Taylor Swift<br/>Adele<br/>Mozart|
| Genre | 所要求之音樂的內容類型。|鄉村音樂<br/>百老匯經典<br/>播放我的巴洛克時期古典音樂|

## <a name="note"></a>附註 
Note 定義域提供與建立、編輯及尋找筆記相關的意圖和實體。

### <a name="examples"></a>範例
```
Add to my groceries note lettuce tomato bread coffee
Check off bananas from my grocery list
Remove all items from my vacation list
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AddToNote | 將資訊新增至筆記。|將生菜、蕃茄、麵包和咖啡新增至我的雜貨筆記<br/>新增至我的待辦事項清單<br/>將杯子蛋糕新增至我的 Wunderlist|
| CheckOffItem | 從現有筆記中標記項目。|從我的雜貨清單中標記香蕉<br/>將我的假日購物清單上的起士蛋糕標記為完成|
| 清除 | 從現有筆記中清除所有項目。|從我的假期清單中移除所有項目<br/>從我的閱讀清單中清除全部|
| 確認 | 確認與筆記相關的動作。|沒問題<br/>是<br/>我確認保留清單上的所有項目|
| 建立 | 建立新的筆記。 | 建立清單<br/>提醒我 Jason 五月第一週在城鎮中的筆記|
| 刪除 | 刪除整個筆記。 |刪除我的假期清單 <br/>刪除我的雜貨筆記|
| DeleteNoteItem | 從現有筆記中刪除項目。| 從我的雜貨清單中刪除洋芋片<br/>從我的學校購物清單中移除筆|
| ReadAloud | 大聲朗讀清單。|為我朗讀第一個<br/>為我朗讀詳細資料|
| ShowNext | 查看筆記清單中的下一個項目。|顯示下一個<br/>下一頁<br/>下一頁|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AppName | 筆記記錄應用程式名稱。|Wunderlist<br/>OneNote|
| ContactName | 筆記中的連絡人名稱。|Carol<br/>Jim<br/>Chris|
| DataSource | 筆記的位置。|OneDrive<br/>Google 文件<br/>我的電腦|
| DataType | 檔案或文件類型，通常與特定軟體程式建立關聯。|投影片<br/>試算表<br/>工作表|
| 文字 | 筆記或提醒的文字。|步行前先做伸展操<br/>明天的長跑|
| 標題 | 筆記的標題。|雜貨<br/>要致電的人員<br/>待辦事項|

## <a name="ondevice"></a>OnDevice 
OnDevice 定義域提供與控制裝置相關的意圖和實體。

### <a name="examples"></a>範例
```
Close video player
Cancel playback
Can you make the screen brighter?
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AreYouListening | 詢問裝置是否正在接聽。|是否開啟？<br/>您是否正在接聽？|
|CloseApplication|關閉裝置應用程式。|關閉視訊播放程式|
|FileBug|報告裝置上的 Bug。|請報告 Bug<br/>您可以向我報告 Bug 嗎？<br/>讓我報告此 Bug|
|GoBack|要求返回一個步驟或返回上一個步驟。|請返回<br/>移至上一個畫面<br/>返回停止接聽|
|說明| 要求說明。|請說明<br/>您好<br/>您該怎麼辦？<br/>我需要說明| 
|LocateDevice|尋找裝置。|您可以尋找我的手機嗎<br/>尋找 Tom 的 iPhone<br/>尋找我的手機|
|LogIn|使用裝置登入服務。|請登入<br/>Facebook 登入<br/>登入 LinkedIn|
|LogOut|使用裝置登出服務。|登出我的手機<br/>登入 Twitter<br/>登出|
|MainMenu|檢視裝置的主功能表。|檢視功能表。|
|OpenApplication|開啟裝置上的應用程式。|請開啟鬧鐘<br/>開啟相機<br/>啟動行事曆|
|OpenSetting|開啟裝置上的設定。|開啟網路設定。|
|PairDevice|配對裝置。|您可以協助我配對藍牙訊號與手機嗎<br/>開啟藍牙並將它與膝上型電腦配對<br/>配對藍牙訊號與我的膝上型電腦|
|PowerOff | 關閉裝置。|您可以關閉我的電腦嗎<br/>Shutdown<br/>關閉我的手機|
|QueryBattery|取得電池壽命的相關資訊。|顯示電池壽命。<br/>我的電池狀態為何<br/>目前電池剩下多少電量？<br/>顯示電池電量|
|QueryWifi|取得 WiFi 的相關資訊。|取得 WiFi 資訊。|
|重新啟動|重新啟動裝置。|請重新啟動。|
|RingDevice| 要求裝置響鈴，以在遺失時能夠找到它。|讓我的手機響鈴。| 
|SetBrightness|設定裝置亮度。|將亮度設定為中<br/>將亮度設定為高<br/>將亮度設定為低|
|SetupDevice|啟動裝置安裝程式。|我想要安裝作業系統安裝程式<br/>請安裝<br/>請替我安裝|
|ShowAppBar|顯示應用程式列。|顯示應用程式列<br/>請顯示應用程式列<br/>讓我看看應用程式列|
|ShowContextMenu|顯示操作功能表。|讓我看看操作功能表<br/>請顯示操作功能表<br/>您可以顯示操作功能表嗎|
|Sleep|讓裝置進入睡眠狀態。|進入睡眠狀態<br/>睡眠<br/>我的電腦進入睡眠狀態|
|SwitchApplication|切換要在裝置上使用的應用程式。|切換至我的媒體播放程式。|
|TurnDownBrightness|調低裝置亮度。|將螢幕變暗。|
|TurnOffSetting|關閉裝置設定。|停用藍牙<br/>停用資料<br/>中斷藍牙連線|
|TurnOnSetting|開啟裝置設定。|另一 <br/> 關閉|
|TurnUpBrightness|調高裝置亮度。|您可以將螢幕變亮嗎？|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AppName | 裝置上的應用程式名稱。|SoundCloud<br/>YouTube|
| BrightnessLevel | 設定裝置上的亮度。|100%<br/>50<br/>40%|
| ContactName | 裝置上的連絡人名稱。|Paul<br/>Marlen Max|
| DeviceType | 裝置的類型。 |電話<br/>Kindle<br/>膝上型電腦|
| MediaType | 裝置處理的媒體類型。|音樂<br/>電影<br/>電視節目|
| SettingType | 使用者想要編輯的設定或設定面板類型。|WiFi<br/>無線網路<br/>色彩配置<br/>通知中心|

## <a name="places"></a>Places  
Places 定義域提供處理與公司、機構、餐廳、公共空間和地址等地點相關之查詢的意圖。

### <a name="examples"></a>範例
```
Save this location to my favorites
How far away is Holiday Inn?
At what time does Safeway close?
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AddFavoritePlace | 將位置新增至使用者的我的最愛清單。|將此位置儲存至我的最愛<br/>將此地址新增至我的最愛|
|CheckAccident|詢問指定道路上是否有事故。|880 上是否有事故？<br/>顯示事故資訊|
|CheckAreaTraffic|查看一般區域或公路上的交通狀況，而不是指定路線。|西雅圖的交通狀況<br/>西雅圖的交通狀況如何？|
|CheckIntoPlace|使用社交媒體簽入某個地點。|將我簽入 Foursquare<br/>簽入這裡|
|CheckRouteTraffic| 查看使用者指定之特定路線的交通狀況。|到 Mashiko 的交通狀況如何？<br/>顯示到 Kirkland 的交通狀況<br/>到西雅圖的交通狀況如何？| 
|確認|確認與地點相關的動作。|確認我的餐廳預約。|
|結束|結束與地點相關之工作的動作。|請結束<br/>結束提供路線|
|FindPlace|搜尋地點 (公司、機構、餐廳、公共空間、地址)。|最近的圖書館在哪裡？<br/>替我在 Mountain View 找間不錯的義大利餐廳|
|GetAddress| 要求某個地點的地址。|顯示 Robson 街道上 Guu 的地址<br/>最近的 Starbucks 在哪裡？| 
|GetDistance|詢問到特定地點的距離。|Holiday Inn 有多遠？<br/>貝爾維尤廣場距離這裡有多遠<br/>到 Tahoe 的距離為何|
|GetHours|詢問某個地點的營業時間。|Safeway 何時打烊？<br/>Home Depot 的營業時間為何？<br/>Starbucks 還開著嗎？|
|GetMenu|要求餐廳的菜單項目。|Zucca 是否有供應素食？<br/>Sizzler 的菜單內容為何<br/>顯示 Applebee 的菜單|
|GetPhoneNumber| 要求某個地點的電話號碼。|最近的 Starbucks 電話號碼為何？<br/>提供 Home Depot 的號碼| 
|GetPriceRange| 要求某個地點的價位。|Zucca 便宜嗎？<br/>Cineplex 每週三是否半價？<br/>在 Sizzler，全隻龍蝦晚餐價格為何?|
|GetReviews|要求某個地點的評論。|顯示 Cheesecase Factory 的評論<br/>閱讀 Yelp 中的 Cineplex 評論|
|GetRoute|要求到某個地點的路線。|如何步行到貝爾維尤廣場<br/>顯示從這裡到第 8 街和第 59 街的最短路線<br/>取得到 Mountain View CA 的路線|
|GetStarRating|要求某個地點的星級評等。|根據 Yelp，Zucca 的評等為何？<br/>French Laundry 有多少顆星？<br/>蒙特利的水族館是否不錯？|
|GetTransportationSchedule|取得某個地點的公車時程表。|前往市中心的下一班公車時間為何？<br/>顯示 King County 的所有公車|
|GetTravelTime|要求到指定目的地所花的時間。|從這裡到舊金山需要多久的時間<br/>從舊金山開車到丹佛需要多久的時間|
|MakeCall|撥打電話到某個地點。|撥打電話給媽媽<br/>我想要撥打 Skype 電話給 Anna<br/>撥打電話給 Jim|
|MakeReservation|要求預約餐廳或其他營業據點。|預約 Zucca 今晚兩人<br/>預約明天一桌<br/>預約 Palo Alto 8 點 3 人桌|
|MapQuestions|要求路線或指定道路是否前往目的地的相關資訊。|13 是否經過市中心？<br/>我可以搭乘 880 到奧克蘭嗎？|
|Rating|取得餐廳或地點的評等描述。|Contoso Inn 有幾顆星？|
|ReadAloud|大聲朗讀地點清單。|為我朗讀第一個<br/>為我朗讀詳細資料|
|SelectItem|從與一或多個地點相關的選擇清單中選擇一個項目。|選取第二個<br/>選取第一個|
|ShowMap|顯示區域地圖。|顯示第二個的地圖<br/>顯示地圖<br/>在地圖上尋找舊金山|
|ShowNext|顯示序列中的下一個項目。|顯示下一個<br/>移至下一頁|
|ShowPrevious|顯示序列中的上一個項目。|顯示上一個<br/>previous<br/>移至上一個|
|StartOver|重新啟動應用程式或啟動新工作階段。|重新開始<br/>新增工作階段<br/>
restart|
|TakesReservations|詢問某個地點是否接受預約。|此美術館是否接受預約<br/>是否可以預約 Olive Garden

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| AbsoluteLocation | 某個地點的位置或地址。|Palo Alto<br/>300 112th Ave SE<br/>Seattle|
| Amenities | 某個地點的目標特性/優點。|兒童用餐免費<br/>水邊<br/>免費停車|
| Atmosphere | 某個地點的氣氛。|適合兒童<br/>休閒餐廳<br/>運動|
| Cuisine | 某個地點的料理。 |地中海<br/>義大利文<br/>印度|
| DestinationAddress| 目的地位置或地址。|Palo Alto<br/>300 112th Ave SE<br/>Seattle|
| DestinationPlaceName| 公司、餐廳、公共遊樂設施或機構目的地的名稱。|Central Park<br/>Safeway<br/>Walmart|
| DestinationPlaceType | 當地公司、餐廳、公共遊樂設施或機構目的地的類型。 |餐廳<br/>Opera<br/>電影院|
| Distance | 到某個地點的距離。|15 英哩<br/>5 英哩<br/>10 英哩遠|
| MealType | 早餐或午餐等餐飲類型。 |早餐<br/>正餐<br/>午餐<br/>晚餐|
| OpenStatus | 指出某個地點為營業或關店。|開啟<br/>關閉<br/>營業中|
| PlaceName | 地點的名稱。|Cheesecake Factory|
| PlaceType | 地點的類型。|咖啡廳<br/>劇院<br/>程式庫|
| PreferredRoute | 使用者指定的偏好路線。 | 101 <br/>202 <br/>Route 401|
| 產品 | 某個地點提供的產品。 | 衣服<br/>數位 ASR 相機<br/>鮮魚 | 
| PublicTransportationRoute | 使用者要搜尋的大眾運輸路線名稱。 | Northeast Corridor 火車<br/>公車路線 3X |
| Rating | 某個地點的評等。 | 5 顆星<br/>3 顆星<br/>4 顆星|
| RouteAvoidanceCriteria | 避免特定路線 (例如避免事故、施工或通行費) 的準則 | 通行費 <br/>施工<br/>Route 11|
| ServiceProvided | 這是公司或地點提供的服務，例如剪髮、剷雪、造景。 | 剪髮<br/>技師<br/>水管工|
| TransportationCompany | 運輸服務提供者的名稱。|Amtrak<br/>Acela<br/>Greyhound|
| TransportationType | 交通工具類型。|公車<br/>定型<br/>開車|

## <a name="reminder"></a>提醒 
Reminder 定義域提供建立、編輯及尋找提醒的意圖和實體。

### <a name="examples"></a>範例
```
Change my interview to 9 am tomorrow
Remind me to buy milk on my way back home
Can you check if I have a reminder about Christine's birthday?
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 變更| 變更提醒。|變更我在明天上午 9 點的面試<br/>將我的指派提醒移到明天|
| 建立| 建立新提醒。|建立提醒<br/>提醒我買牛奶<br/>我想要記得在家中時要撥打電話給 Rebecca|
| 刪除 | 刪除提醒。|刪除我的圖片提醒<br/>刪除此提醒|
| 尋找 | 尋找提醒。|我是否有週年日的相關提醒？<br/>您可以檢查我是否有 Christine 的生日提醒嗎？|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 文字 | 提醒的文字描述。|拿乾洗衣物<br/>將車子停在維修中心|

## <a name="restaurantreservation"></a>RestaurantReservation 
RestaurantReservation 定義域提供與管理餐廳預留相關的意圖和實體。

### <a name="examples"></a>範例
```
Reserve at Zucca for two for tonight
Book a table at BJ's for tomorrow
Table for 3 in Palo Alto at 7
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| Reserve | 要求預約餐廳。 |預約 Zucca 今晚兩人<br/>預約明天一桌<br/>預約 Palo Alto 7 點 3 人桌|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 位址| 預約的活動位置或地址。|Palo Alto<br/>300 112th Ave SE<br/>Seattle|
| Amenities | 描述某個地點之便利設施的屬性。|海景<br/>非吸煙|
| AppName | 預約的應用程式名稱。|OpenTable<br/>Yelp<br/>TripAdvisor|
| Atmosphere | 餐廳或其他地點的氣氛描述。|浪漫<br/>休閒<br/>適合團體|
| Cuisine | 食物類型、料理或料理國籍。 |中文<br/>義大利文<br/>墨西哥|
| MealType | 與預約建立關聯的飲食類型。|早餐<br/>正餐<br/>午餐<br/>晚餐|
| PlaceName | 當地公司、餐廳、公共遊樂設施或機構的名稱。|IHOP<br/>Cheesecake Factory<br/>羅浮宮|
| PlaceType | 當地公司、餐廳、公共遊樂設施或機構的類型。|餐廳<br/>歌劇<br/>電影院|
| Rating | 地點或餐廳的評等。|5 顆星<br/>3 顆星<br/>4 顆星|

## <a name="taxi"></a>Taxi 
 
Taxi 定義域提供建立及管理計程車預約的意圖和實體。

### <a name="examples"></a>範例
```
Get me a cab at 3 pm
How much longer do I have to wait for my taxi?
Cancel my Uber
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 書籍 | 叫計程車。 |替我叫計程車<br/>尋找計程車<br/>替我預約 uber x|
| 取消 | 取消與預約計程車相關的動作。|取消我的計程車<br/>取消我的 Uber|
| Track | 追蹤計程車路線。|我還要等多久才能坐到計程車？<br/>我的 Uber 在哪裡？|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 位址| 與預約計程車建立關聯的地址。 |Palo Alto<br/>300 112th Ave SE<br/>Seattle|
| DestinationAddress| 目的地位置或地址。 |Palo Alto<br/>300 112th Ave SE<br/>Seattle|
| DestinationPlaceName | 當地公司、餐廳、公共遊樂設施或機構目的地的名稱。 |Central Park<br/>Safeway<br/>Walmart|
| DestinationPlaceType | 當地公司、餐廳、公共遊樂設施或機構目的地的類型。 |餐廳<br/>Opera<br/>電影院|
| PlaceName | 當地公司、餐廳、公共遊樂設施或機構的名稱。 |Central Park<br/>Safeway<br/>Walmart|
| PlaceType| 預約計程車要求中的地點類型。|餐廳<br/>Opera<br/>電影院|
| TransportationCompany | 運輸服務提供者的名稱。|Amtrak<br/>Acela<br/>Greyhound|
| TransportationType | 交通工具類型。|公車<br/>定型<br/>開車|

## <a name="translate"></a>翻譯 
Translate 定義域提供與將文字翻譯成目標語言相關的意圖和實體。

### <a name="examples"></a>範例
```
Translate to French
Translate hello to German
Translate this sentence to English
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 翻譯| 將文字翻譯成其他語言。|翻譯成法文<br/>將 hello 翻譯成德文|


### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| TargetLanguage | 翻譯的目標語言。|法文<br/>德文<br/>韓文|
| 文字 | 要翻譯的文字。|Hello World<br/>Good morning<br/>Good evening|

## <a name="tv"></a>TV 
 
TV 定義域提供控制電視的意圖和實體。

### <a name="examples"></a>範例
```
Switch channel to BBC
Show TV guide
Watch National Geographic
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| ChangeChannel| 變更電視上的頻道。|將頻道變更為 CNN<br/>將頻道切換至 BBC<br/>移至 Channel 4|
| ShowGuide| 顯示電視指南。|顯示電視指南<br/>電影頻道上現在放映什麼？<br/>顯示我的節目清單|
| WatchTV| 要求收看電視頻道。|我想要收看迪士尼頻道<br/>請移至電視節目<br/>收看國家地理頻道|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| ChannelName | 電視頻道的名稱。|CNN<br/>BBC<br/>電影頻道|

## <a name="utilities"></a>公用事業  
Utilities 定義域提供許多工作通用的工作意圖，例如問候語、取消、確認、說明、重說、導航、開始和停止。

### <a name="examples"></a>範例
```
Go back to Twitter
Please help
Repeat last question please
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 取消 | 取消動作。|取消訊息<br/>我不想再傳送電子郵件|
| 確認 | 確認動作。|是的，我確認<br/>好的，我正在確認<br/>沒問題，我正在確認|
| FinishTask | 完成使用者開始的工作。|我已完成<br/>我已完成<br/>完成|
| GoBack | 返回一個步驟或返回上一個步驟。|返回 Twitter<br/>返回一個步驟<br/>返回|
| 說明 | 要求說明。|請說明<br/>開啟說明<br/>help|
| Repeat | 重複動作。|請重複最後一個問題<br/>重複上一首歌|
| ShowNext | 顯示序列中的下一個項目。 |顯示下一個<br/>移至下一頁|
| ShowPrevious | 顯示序列中的上一個項目。|顯示上一個|
| StartOver | 重新啟動應用程式或啟動新工作階段。|重新開始<br/>新增工作階段<br/>restart|
| Stop | 停止動作。| 請停止這麼說<br/>閉嘴<br/>請停止|

## <a name="weather"></a>Weather 
Weather 定義域提供取得天氣預報的意圖和實體。

### <a name="examples"></a>範例
```
weather in London in september
What?s the 10 day forecast?
What's the average temperature in India in september?
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| GetCondition | 取得與天氣相關的過去基本資料。 |倫敦 9 月的天氣<br/>印度 9 月的平均溫度為何？|
| GetForecast | 取得目前天氣與未來幾天的預測。 |今天的天氣如何？<br/>未來 10 天的預測為何？<br/>這個週末的天氣如何？|

### <a name="entities"></a>實體
| 實體名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 位置| 天氣要求的絕對位置。|Seattle<br/>巴黎<br/>Palo Alto|

## <a name="web"></a>Web 
Web 定義域提供巡覽至網站的意圖。

### <a name="examples"></a>範例
```
Navigate to facebook.com
Go to www.twitter.com
Navigate to www.bing.com
```

### <a name="intents"></a>對應方式
| 意圖名稱 | 說明 | 範例 |
| ---------------- |-----------------------|----|
| 瀏覽 | 巡覽至指定網站的要求。 |巡覽至 facebook.com<br/>前往 www.twitter.com|

