---
title: 利用 Azure 串流分析來建置 IoT 解決方案
description: 串流分析收費亭案例 IoT 解決方案的入門教學課程
services: stream-analytics
author: jasonwhowell
ms.author: jasonh
manager: kfile
ms.reviewer: jasonh, sngun
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 03/21/2018
ms.openlocfilehash: 1610e8173d90be3c0b50f05e64d0e84e1c21ad0e
ms.sourcegitcommit: cb61439cf0ae2a3f4b07a98da4df258bfb479845
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/05/2018
ms.locfileid: "43698038"
---
# <a name="build-an-iot-solution-by-using-stream-analytics"></a>利用串流分析來建置 IoT 解決方案

## <a name="introduction"></a>簡介
在本解決方案中，您將學習如何利用 Azure 串流分析來從您的資料中取得即時見解。 開發人員可以很容易地結合資料串流，例如點選流、記錄和裝置產生的事件，並且包含歷程記錄或參考資料，以衍生商務深入解析。 Azure 串流分析是由 Microsoft Azure 代管且完全受控的即時串流計算服務，它提供內建的備援、低延遲及延展性功能，可讓您在幾分鐘之內就立刻上手。

完成此解決方案之後，您便能夠：

* 熟悉 Azure 串流分析入口網站。
* 設定及部署串流工作。
* 使用串流分析查詢語言來表達真實世界的問題，並加以解決。
* 有自信地使用串流分析來為客戶開發串流解決方案。
* 利用監視和記錄的經驗來排解問題。

## <a name="prerequisites"></a>先決條件
若要完成本解決方案，您需要滿足下列必要條件：
* [Azure 訂用帳戶](https://azure.microsoft.com/pricing/free-trial/)

## <a name="scenario-introduction-hello-toll"></a>案例簡介：「收費站，你好！」
收費站是常見的設施。 您可以在世界各地的許多快速道路、橋樑及隧道中看到它們。 每個收費站都有多個收費亭。 在人工收費亭中，您需要停車來向服務員付費。 在自動收費亭中，位於每個收費亭最上方的感應器會在您通過收費亭時掃描黏貼在您車輛擋風玻璃上的 RFID 卡。 我們可以輕易地把車輛通過這些收費站的情況，想像成其中能執行許多有趣行動的事件串流。

![收費亭中車輛的圖片](media/stream-analytics-build-an-iot-solution-using-stream-analytics/image1.jpg)

## <a name="incoming-data"></a>傳入的資料
本解決方案會搭配兩個資料串流來運作。 安裝在收費亭入口和出口處的感應器會產生第一個串流。 第二個串流是擁有車輛登記資料的靜態查詢資料集。

### <a name="entry-data-stream"></a>入口資料流
入口資料流包含車輛進入收費站的相關資訊。 出口資料事件會從包含在範例應用程式中的 Web 應用程式即時串流到事件中樞佇列。

| TollID | EntryTime | LicensePlate | State | 請確定 | 模型 | VehicleType | VehicleWeight | Toll | Tag |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 |2014-09-10 12:01:00.000 |JNB 7001 |NY |Honda |CRV |1 |0 |7 | |
| 1 |2014-09-10 12:02:00.000 |YXZ 1001 |NY |Toyota |Camry |1 |0 |4 |123456789 |
| 3 |2014-09-10 12:02:00.000 |ABC 1004 |CT |Ford |Taurus |1 |0 |5 |456789123 |
| 2 |2014-09-10 12:03:00.000 |XYZ 1003 |CT |Toyota |Corolla |1 |0 |4 | |
| 1 |2014-09-10 12:03:00.000 |BNJ 1007 |NY |Honda |CRV |1 |0 |5 |789123456 |
| 2 |2014-09-10 12:05:00.000 |CDE 1007 |NJ |Toyota |4x4 |1 |0 |6 |321987654 |

以下是每個資料欄的簡短說明：

| 欄 | 說明 |
| --- | --- |
| TollID |唯一識別收費亭的收費亭識別碼 |
| EntryTime |車輛進入收費亭的日期及時間 (國際標準時間) |
| LicensePlate |車輛的車牌號碼 |
| State |美國的某個洲 |
| 請確定 |車輛的製造商 |
| 模型 |車輛的型號 |
| VehicleType |1 代表載客車或 2 代表商用車 |
| WeightType |車輛的重量，單位為噸；0 代表客車 |
| Toll |通行費，單位為美元 |
| Tag |車輛上可用來自動付費的 e-Tag，空白則代表手動付費 |

### <a name="exit-data-stream"></a>出口資料流
出口資料流包含車輛離開收費站的相關資訊。 出口資料事件會從包含在範例應用程式中的 Web 應用程式即時串流到事件中樞佇列。

| **TollId** | **ExitTime** | **LicensePlate** |
| --- | --- | --- |
| 1 |2014-09-10T12:03:00.0000000Z |JNB 7001 |
| 1 |2014-09-10T12:03:00.0000000Z |YXZ 1001 |
| 3 |2014-09-10T12:04:00.0000000Z |ABC 1004 |
| 2 |2014-09-10T12:07:00.0000000Z |XYZ 1003 |
| 1 |2014-09-10T12:08:00.0000000Z |BNJ 1007 |
| 2 |2014-09-10T12:07:00.0000000Z |CDE 1007 |

以下是每個資料欄的簡短說明：

| 欄 | 說明 |
| --- | --- |
| TollID |唯一識別收費亭的收費亭識別碼 |
| ExitTime |車輛離開收費亭的日期及時間 (國際標準時間) |
| LicensePlate |車輛的車牌號碼 |

### <a name="commercial-vehicle-registration-data"></a>商用車的登記資料
本解決方案將使用商用車登記資料庫的靜態快照集。 這項資料會以 JSON 檔案形式儲存至包含在此範例中的 Azure Blob 儲存體。

| LicensePlate | RegistrationId | 已過期 |
| --- | --- | --- |
| SVT 6023 |285429838 |1 |
| XLZ 3463 |362715656 |0 |
| BAC 1005 |876133137 |1 |
| RIV 8632 |992711956 |0 |
| SNY 7188 |592133890 |0 |
| ELH 9896 |678427724 |1 |

以下是每個資料欄的簡短說明：

| 欄 | 說明 |
| --- | --- |
| LicensePlate |車輛的車牌號碼 |
| RegistrationId |車輛的登記識別碼 |
| 已過期 |車輛的登記狀態：0 代表車輛登記仍有效，1 代表車輛登記已過期 |

## <a name="set-up-the-environment-for-azure-stream-analytics"></a>設定 Azure 串流分析的環境
若要完成此解決方案，您需要 Microsoft Azure 訂用帳戶。 如果您沒有 Azure 帳戶，您可以 [要求獲得免費試用版](http://azure.microsoft.com/pricing/free-trial/)。

請務必依照這篇文章結尾處＜清理您的 Azure 帳戶＞一節中的步驟來進行，讓您能充分利用您的免費 Azure 點數。

## <a name="deploy-the-sample"></a>部署範例 
只要按幾下，就可以輕鬆將數個資源一起部署在資源群組中。 解決方案定義裝載於 github 存放庫，網址為 [https://github.com/Azure/azure-stream-analytics/tree/master/Samples/TollApp](https://github.com/Azure/azure-stream-analytics/tree/master/Samples/TollApp)。

### <a name="deploy-the-tollapp-template-in-the-azure-portal"></a>在 Azure 入口網站中部署 TollApp 範本
1. 若要將 TollApp 環境部署至 Azure，請使用這個連結來[部署 TollApp Azure 範本](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-stream-analytics%2Fmaster%2FSamples%2FTollApp%2FVSProjects%2FTollAppDeployment%2Fazuredeploy.json)。

2. 出現提示時，登入 Azure 入口網站。

3. 選擇各種資源的計費訂用帳戶。

4. 使用唯一的名稱來指定新的資源群組，例如 `MyTollBooth`。 

5. 選取 Azure 位置。

6. 將 [間隔] 指定為秒數。 範例 Web 應用程式中會使用此值來說明將資料傳送到事件中樞的頻率。 

7. [勾選] 以同意條款和條件。

8. 選取 [釘選到儀表板] 以便您稍後可以輕鬆地找到資源。

9. 選取 [購買] 以部署範例範本。

10. 一段時間之後，系統會顯示通知以確認**部署成功**。

### <a name="review-the-azure-stream-analytics-tollapp-resources"></a>檢閱 Azure 串流分析 TollApp 資源
1. 登入 Azure 入口網站

2. 尋找您在上一節中所命名的資源群組。

3. 確認資源群組中列出下列資源：
   - 一個 Cosmos DB 帳戶
   - 一個 Azure 串流分析作業
   - 一個 Azure 儲存體帳戶
   - 一個 Azure 事件中樞
   - 兩個 Web Apps

## <a name="examine-the-sample-tollapp-job"></a>檢查範例 TollApp 作業 
1. 從上一節中的資源群組開始，選取以 **tollapp** 名稱 (名稱會包含隨機字元以保持獨特性) 開頭的串流分析串流作業。

2. 在作業的 [概觀] 頁面中，請注意 [查詢] 方塊，以檢視查詢語法。

   ```sql
   SELECT TollId, System.Timestamp AS WindowEnd, COUNT(*) AS Count
   INTO CosmosDB
   FROM EntryStream TIMESTAMP BY EntryTime
   GROUP BY TUMBLINGWINDOW(minute, 3), TollId
   ```

   為了解釋查詢的意圖，我們假設您需要計算進入某個收費亭的車輛數目。 因為高速公路的收費亭會有川流不息的車輛進入，所以這些進入事件就類似於永不間斷的串流。 若要將串流量化，您必須定義要量測的「期間」。 我們來將問題進一步精簡為「每三分鐘有多少車輛進入收費亭？」 這通常稱為輪轉計數。

   如您所見，Azure 串流分析會使用類似 SQL 的查詢語言，並新增幾個擴充功能來指定和時間有關的查詢層面。  如需詳細資料，請參閱[時間管理](https://msdn.microsoft.com/library/azure/mt582045.aspx)，以及查詢中所用的[時段](https://msdn.microsoft.com/library/azure/dn835019.aspx)建構。

3. 檢查 TollApp 範例作業的輸入。 目前的查詢只會使用 EntryStream 輸入。
   - **EntryStream** 輸入是事件中樞連線，會將代表高速公路上每次有一輛車進入收費亭的資料排入佇列。 屬於範例一部分的 Web 應用程式會建立事件，且該資料會在這個事件中樞內排入佇列。 請注意，此輸入會在串流查詢的 FROM 子句中加以查詢。
   - **ExitStream** 輸入是事件中樞連線，會將代表高速公路上每次有一輛車離開收費亭的資料排入佇列。 此串流輸入會用於稍後的查詢語法變化。
   - **註冊**輸入是 Azure Blob 儲存體連線，會指向可視需要用於查閱的靜態 registration.json 檔案。 此參考資料輸入會用於稍後的查詢語法變化。

4. 檢查 TollApp 範例作業的輸出。
   - **Cosmos DB** 輸出是 Cosmos 資料庫集合，可接收輸出的接收事件。 請注意，此輸出用於串流查詢的 INTO 子句。

## <a name="start-the-tollapp-streaming-job"></a>啟動 TollApp 串流作業
請遵循下列步驟以啟動串流作業：

1. 在作業的 [概觀] 頁面中，選取 [啟動]。

2. 在 [啟動作業] 窗格中，選取 [現在]。

3. 一段時間後，當作業開始執行時，請在串流作業的 [概觀] 頁面中檢視 [監視] 圖形。 圖形應該會顯示數千個輸入事件，以及數十個輸出事件。

## <a name="review-the-cosmosdb-output-data"></a>檢閱 CosmosDB 輸出資料
1. 尋找包含 TollApp 資源的資源群組。

2. 選取名稱模式為 **tollapp<random>-cosmos** 的 Azure Cosmos DB 帳戶。

3. 選取 [資料總管] 標題以開啟 [資料總管] 頁面。

4. 展開 [tollAppDatabase] > [tollAppCollection] > [文件]。

5. Azure 串流分析經悉心設計，能彈性調整以便於處理大量的資料。

6. 選取每個識別碼，以檢閱 JSON 文件。 請注意該視窗中的每個 tollid、windowend 時間以及汽車計數。

7. 再過三分鐘之後，另一組四個文件便可供使用，每一個 tollid 會有一份文件。 


## <a name="report-total-time-for-each-car"></a>報告每輛汽車的總時間
車輛通過收費亭所需的平均時間有助於評估程序效率和客戶經驗。

若要尋找總時間，請聯結 EntryTime 串流和 ExitTime 串流。 在對等相符的 TollId 和 LicencePlate 資料行上，聯結兩個輸入串流。 **JOIN** 運算子需要您指定彈性時間，來說明已聯結事件之間可接受的時間差。 使用 **DATEDIFF** 函式來指定事件的間隔時間不能超過 15 分鐘。 另外也將 **DATEDIFF** 函式套用到入口和出口時間，以便計算車輛經過收費亭的實際時間。 請注意在 **SELECT** 陳述式中 (而非在 **JOIN** 中) 使用 **DATEDIFF** 時，其使用方式的差異。

```sql
SELECT EntryStream.TollId, EntryStream.EntryTime, ExitStream.ExitTime, EntryStream.LicensePlate, DATEDIFF (minute, EntryStream.EntryTime, ExitStream.ExitTime) AS DurationInMinutes
INTO CosmosDB
FROM EntryStream TIMESTAMP BY EntryTime
JOIN ExitStream TIMESTAMP BY ExitTime
ON (EntryStream.TollId= ExitStream.TollId AND EntryStream.LicensePlate = ExitStream.LicensePlate)
AND DATEDIFF (minute, EntryStream, ExitStream ) BETWEEN 0 AND 15
```

### <a name="to-update-the-tollapp-streaming-job-query-syntax"></a>若要更新 TollApp 串流作業查詢語法：

1. 在作業的 [概觀] 頁面中，選取 [停止]。

2. 稍候片刻，便會收到作業已停止的通知。

3. 在 JOB TOPOLOGY 標題下，選取 [< > 查詢]

4. 將已調整的串流 SQL 查詢貼上。

5. 選取 [儲存] 以儲存查詢。 確認 [是] 以儲存變更。

6. 在作業的 [概觀] 頁面中，選取 [啟動]。

7. 在 [啟動作業] 窗格中，選取 [現在]。

### <a name="review-the-total-time-in-the-output"></a>檢閱輸出中的總時間
重複上一節中的步驟，檢閱串流作業中的 CosmosDB 輸出資料。 檢閱最新的 JSON 文件。 

例如，本文件會顯示特定車牌的範例汽車、entrytime 和 exittime，以及 DATEDIFF 導出的 durationinminutes 欄位，顯示收費亭持續時間為兩分鐘： 
```JSON
{
    "tollid": 4,
    "entrytime": "2018-04-05T06:51:39.0491173Z",
    "exittime": "2018-04-05T06:53:09.0491173Z",
    "licenseplate": "JVR 9425",
    "durationinminutes": 2,
    "id": "ff52eb25-d580-7566-2879-1f52bba6601e",
    "_rid": "+8E4AI1DZgBjAAAAAAAAAA==",
    "_self": "dbs/+8E4AA==/colls/+8E4AI1DZgA=/docs/+8E4AI1DZgBjAAAAAAAAAA==/",
    "_etag": "\"ad02f6b8-0000-0000-0000-5ac5c8330000\"",
    "_attachments": "attachments/",
    "_ts": 1522911283
}
```

## <a name="report-vehicles-with-expired-registration"></a>報告車輛登記已過期的車輛
Azure 串流分析可使用靜態的參考資料快照集，來與時間資料流聯結。 若要示範此功能，請使用下面的範例問題。 登記輸入是靜態的 Blob json 檔案，會列出車牌的到期日。 透過聯結車牌，系統會將參考資料與通過收費亭的每輛汽車進行比較。 

如果商用車已經向收費公司登記，就可以直接通過收費亭，不用停車接受檢查。 使用登記查詢資料表，來辨識所有車輛登記已過期的商用車。

```sql
SELECT EntryStream.EntryTime, EntryStream.LicensePlate, EntryStream.TollId, Registration.RegistrationId
INTO CosmosDB
FROM EntryStream TIMESTAMP BY EntryTime
JOIN Registration
ON EntryStream.LicensePlate = Registration.LicensePlate
WHERE Registration.Expired = '1'
```

1. 重複上一節中的步驟，更新 TollApp 串流作業查詢語法。

2. 重複上一節中的步驟，檢閱串流作業中的 CosmosDB 輸出資料。 

範例輸出︰
```json
    {
        "entrytime": "2018-04-05T08:01:28.0252168Z",
        "licenseplate": "GMT 3221",
        "tollid": 1,
        "registrationid": "763220582",
        "id": "47db0535-9716-4eb2-db58-de7886966cbf",
        "_rid": "y+F8AJ9QWACSAQAAAAAAAA==",
        "_self": "dbs/y+F8AA==/colls/y+F8AJ9QWAA=/docs/y+F8AJ9QWACSAQAAAAAAAA==/",
        "_etag": "\"88007d8d-0000-0000-0000-5ac5d7e20000\"",
        "_attachments": "attachments/",
        "_ts": 1522915298
    }
```

## <a name="scale-out-the-job"></a>將作業相應放大
Azure 串流分析是設計成能進行彈性調整，以便於處理大量的資料。 Azure 串流分析查詢可以使用 **PARTITION BY** 子句，來告訴系統此步驟要相應放大。**PartitionId** 是系統新增的特殊資料欄，且它與輸入 (事件中樞) 的分割識別碼相符。

若要將查詢相應放大至分割區，請將查詢語法編輯為下列程式碼：
```sql
SELECT TollId, System.Timestamp AS WindowEnd, COUNT(*)AS Count
INTO CosmosDB
FROM EntryStream 
TIMESTAMP BY EntryTime 
PARTITION BY PartitionId
GROUP BY TUMBLINGWINDOW(minute,3), TollId, PartitionId
```

若要將串流作業相應增加至更多串流單位：

1. **停止**目前的作業。 

2. 更新 [< > 查詢] 頁面中的查詢語法，並儲存變更。

3. 在串流作業的 CONFIGURE 標題下，選取 [縮放]。
   
4. 將 [串流單位] 滑桿從 1 滑動到 6。 串流單位會定義作業所能接收的計算能力量。 選取 [ **儲存**]。

5. **啟動**串流作業以示範額外的縮放。 Azure 串流分析可將工作分送給更多計算資源以改善輸送量，並使用 PARTITION BY 子句中指定的資料行，將工作分割給各個資源。 

## <a name="monitor-the-job"></a>監視作業
[監視] 區域包含執行中作業的相關統計資料。 第一次設定需要使用相同區域中的儲存體帳戶 (如同本文的其餘部分，也就是收費)。   

![監視螢幕擷取畫面](media/stream-analytics-build-an-iot-solution-using-stream-analytics/monitoring.png)

您也可以從作業儀表板的 [設定] 區域存取 [活動記錄]。

## <a name="clean-up-the-tollapp-resources"></a>清除 TollApp 資源
1. 從 Azure 入口網站停止串流分析工作。

2. 尋找資源群組，當中包含八個與 TollApp 範本相關的資源。

3. 選取 [刪除資源群組]。 輸入資源群組名稱以確認刪除。

## <a name="conclusion"></a>結論
本解決方案向您介紹了 Azure 串流分析服務。 課程中示範了如何為串流分析工作設定輸入和輸出。 解決方案中還利用付費資料案例，來解釋在資料空間會不斷變化時所引發的常見問題類型，以及如何在 Azure 串流分析中利用類似 SQL 的簡單查詢來解決這些問題。 解決方案說明了要用來處理時態性資料的 SQL 擴充功能建構。 課程示範了如何聯結不同的資料流，以及如何利用靜態參考資料來豐富資料流的內容，以及如何相應放大查詢來達到更高的輸送量。

雖然本解決方案提供了詳盡的簡介，但也絕對不算是完整的說明。 您可以使用 SAQL 語言在 [一般串流分析使用模式的查詢範例](stream-analytics-stream-analytics-query-patterns.md)中找到更多查詢模式。
