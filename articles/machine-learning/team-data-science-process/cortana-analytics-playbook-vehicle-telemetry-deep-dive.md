---
title: 深入探索如何預測車輛健全狀態與駕駛習慣 - Azure | Microsoft Docs
description: 使用 Cortana Intelligence 具備的強大功能，取得關於車輛健全狀態與駕駛習慣的即時預測情資。
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: d8866fa6-aba6-40e5-b3b3-33057393c1a8
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2018
ms.author: deguhath
ms.openlocfilehash: 991e4b86a1d3e75c02e5ed8fe97727c625f174a4
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37055470"
---
# <a name="vehicle-telemetry-analytics-solution-playbook-deep-dive-into-the-solution"></a>車輛遙測分析方案腳本：深入探索方案
此功能表會連結至此腳本的章節： 

[!INCLUDE [cap-vehicle-telemetry-playbook-selector](../../../includes/cap-vehicle-telemetry-playbook-selector.md)]

本文件深入方案架構所述的每個階段。 內含自訂作業的指示和方針。 

## <a name="data-sources"></a>資料來源
此方案使用兩種不同的資料來源：

* 模擬車輛訊號和診斷資料集
* 車輛目錄

此方案包含車輛遠程資訊服務模擬器，如下列螢幕擷取畫面所示。 它會在指定的時間點發出對應於車輛狀態與駕駛模式的診斷資訊和訊號。  車輛目錄包含可將車輛識別號碼 (VIN) 對應至模型的參考資料集。 注意：車輛遠程資訊服務模擬器 Visual Studio 方案資料集已無法再使用。 

![車輛遠程資訊服務模擬器](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig1-vehicle-telematics-simulator.png)


這是 JSON 格式的資料集，其中包含下列結構描述。

| 欄 | 說明 | 值 |
| --- | --- | --- |
| VIN |隨機產生的 VIN |取自於一份含有 10,000 個隨機產生 VIN 的主要清單 |
| Outside temperature |車輛行駛時的外部溫度 |從 0 至 100 隨機產生的數字 |
| Engine temperature |車輛的引擎溫度 |從 0 至 500 隨機產生的數字 |
| 速度 |車輛行駛時的引擎速度 |從 0 至 100 隨機產生的數字 |
| Fuel |車輛的燃油量 |從 0 至 100 隨機產生的數字 (表示燃油量百分比) |
| EngineOil |車輛的機油量 |從 0 至 100 隨機產生的數字 (表示機油量百分比) |
| 胎壓 |車輛胎壓 |從 0 至 50 隨機產生的數字 (表示胎壓位準百分比) |
| Odometer |車輛的里程表讀數 |從 0 至 200,000 隨機產生的數字 |
| Accelerator_pedal_position |車輛的油門踏板位置 |從 0 至 100 隨機產生的數字 (表示油門位準百分比) |
| Parking_brake_status |指出車輛是否已停車 |True 或 False |
| Headlamp_status |指出大燈是否打開 |True 或 False |
| Brake_pedal_status |指出是否踩下剎車踏板 |True 或 False |
| Transmission_gear_position |車輛的傳動齒輪位置 |狀態：first、second、third、fourth、fifth、sixth、seventh、eighth |
| Ignition_status |指出車輛在行駛中或已停車 |True 或 False |
| Windshield_wiper_status |指出擋風玻璃雨刷是否啟動 |True 或 False |
| ABS |指出 ABS 是否發揮作用 |True 或 False |
| Timestamp |建立資料點時的時間戳記 |日期 |
| City |車輛的位置 |此方案中有 4 個城市：Bellevue、Redmond、Sammamish、Seattle |

車輛型號參考資料集會將 VIN 對應至模型。 

| VIN | 模型 |
| --- | --- |
| FHL3O1SA4IEHB4WU1 |轎車 |
| 8J0U8XCPRGW4Z3NQE |混合式 |
| WORG68Z2PLTNZDBI7 |家庭房車 |
| JTHMYHQTEPP4WBMRN |轎車 |
| W9FTHG27LZN1YWO0Y |混合式 |
| MHTP9N792PHK08WJM |家庭房車 |
| EI4QXI2AXVQQING4I |轎車 |
| 5KKR2VB4WHQH97PF8 |混合式 |
| W9NSZ423XZHAONYXB |家庭房車 |
| 26WJSGHX4MA5ROHNL |敞篷車 |
| GHLUB6ONKMOSI7E77 |旅行車 |
| 9C2RHVRVLMEJDBXLP |小型車 |
| BRNHVMZOUJ6EOCP32 |小型 SUV |
| VCYVW0WUZNBTM594J |跑車 |
| HNVCE6YFZSA5M82NY |中型 SUV |
| 4R30FOR7NUOBL05GJ |旅行車 |
| WYNIIY42VKV6OQS1J |大型 SUV |
| 8Y5QKG27QET1RBK7I |大型 SUV |
| DF6OX2WSRA6511BVG |兩人座轎車 |
| Z2EOZWZBXAEW3E60T |轎車 |
| M4TV6IEALD5QDS3IR |混合式 |
| VHRA1Y2TGTA84F00H |家庭房車 |
| R0JAUHT1L1R3BIKI0 |轎車 |
| 9230C202Z60XX84AU |混合式 |
| T8DNDN5UDCWL7M72H |家庭房車 |
| 4WPYRUZII5YV7YA42 |轎車 |
| D1ZVY26UV2BFGHZNO |混合式 |
| XUF99EW9OIQOMV7Q7 |家庭房車 |
| 8OMCL3LGI7XNCC21U |敞篷車 |
| ……. | |

## <a name="ingestion"></a>擷取
利用 Azure 事件中樞、Azure 串流分析和 Azure Data Factory 的組合，以擷取車輛訊號、診斷事件及即時和批次分析。 方案部署時會建立和設定所有這些元件。 

### <a name="real-time-analysis"></a>即時分析
使用事件中樞 SDK 將車輛遠程資訊服務模擬器所產生的事件發佈至事件中樞。  

![事件中樞儀表板](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig4-vehicle-telematics-event-hub-dashboard.png) 

串流分析作業會從事件中樞擷取這些事件，並即時處理資料，以分析車輛健全狀況。

![處理資料的串流分析作業](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig5-vehicle-telematics-stream-analytics-job-processing-data.png) 


串流分析作業：

* 從事件中樞擷取資料。
* 結合參考資料以將車輛 VIN 對應至相符的車型。 
* 將資料保存到 Azure Blob 儲存體以進行充分的批次分析。 

下列串流分析查詢用來將資料保存到 Blob 儲存體： 

![用於擷取資料的串流分析作業查詢](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig6-vehicle-telematics-stream-analytics-job-query-for-data-ingestion.png) 


### <a name="batch-analysis"></a>批次分析
我們也會產生另一批模擬車輛訊號和診斷資料集，以進行更多樣的批次分析。 此額外數量是為了確保有良好的代表性資料量可供批次處理。 基於此目的，在 Data Factory 工作流程中使用 PrepareSampleDataPipeline，以產生一年份的模擬車輛訊號和診斷資料集。 若要下載 Data Factory 自訂 .NET 活動 Visual Studio 方案，以根據您的需求來自訂，請移至 [Data Factory 自訂活動](http://go.microsoft.com/fwlink/?LinkId=717077)網頁。 

此工作流程會顯示為批次處理所準備的範例資料。

![為批次處理工作流程所準備的範例資料](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig7-vehicle-telematics-prepare-sample-data-for-batch-processing.png) 


此管線包含自訂 Data Factory .NET 活動。

![PrepareSampleDataPipeline 活動](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig8-vehicle-telematics-prepare-sample-data-pipeline.png) 

在成功執行管線且 RawCarEventsTable 資料集標示為「就緒」之後，就會產生一年份的模擬車輛訊號和診斷資料。 您在儲存體帳戶的 connectedcar 容器下會看到已建立下列資料夾和檔案：

![PrepareSampleDataPipeline 輸出](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig9-vehicle-telematics-prepare-sample-data-pipeline-output.png) 

## <a name="partition-the-data-set"></a>分割資料集
在資料準備步驟中，原始的半結構化車輛訊號和診斷資料集會分割成 YEAR/MONTH 格式。 此分割能經由切換來提高查詢效率和使用可擴充的長期儲存體。 例如，在第一個 Blob 帳戶填滿時切換到下一個帳戶。 

>[!NOTE] 
>方案中的此步驟僅適用於批次處理。

輸入和輸出資料管理︰

* **輸出資料** (標示為 PartitionedCarEventsTable) 應該在客戶的 Data Lake 中以基本/「最原始」的資料形式長時間保存。 
* 通常會捨棄此管線的**輸入資料**，因為輸出資料會完整無缺地呈現輸入。 這是為了方便後續使用而儲存 (分割)。

分割汽車事件工作流程。

![分割汽車事件工作流程](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig10-vehicle-telematics-partition-car-events-workflow.png)


在 PartitionCarEventsPipeline 中使用 Hive Azure HDInsight 活動來分割原始資料，如下列螢幕擷取畫面所示。 資料準備步驟中所產生的一年範例資料會以 YEAR/MONTH 分割。 資料分割可用來產生一年當中每個月 (總共 12 個磁碟分割) 的車輛訊號和診斷資料。 

![PartitionCarEventsPipeline 活動](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig11-vehicle-telematics-partition-car-events-pipeline.png)


**PartitionConnectedCarEvents Hive 指令碼**

Hive 指令碼 partitioncarevents.hql 可用於分割。 它位於所下載 zip 檔案的 \demo\src\connectedcar\scripts 資料夾中。 
    
    SET hive.exec.dynamic.partition=true;
    SET hive.exec.dynamic.partition.mode = nonstrict;
    set hive.cli.print.header=true;

    DROP TABLE IF EXISTS RawCarEvents; 
    CREATE EXTERNAL TABLE RawCarEvents 
    (
                vin                                string,
                model                            string,
                timestamp                        string,
                outsidetemperature                string,
                enginetemperature                string,
                speed                            string,
                fuel                            string,
                engineoil                        string,
                tirepressure                    string,
                odometer                        string,
                city                            string,
                accelerator_pedal_position        string,
                parking_brake_status            string,
                headlamp_status                    string,
                brake_pedal_status                string,
                transmission_gear_position        string,
                ignition_status                    string,
                windshield_wiper_status            string,
                abs                              string,
                gendate                            string

    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:RAWINPUT}'; 

    DROP TABLE IF EXISTS PartitionedCarEvents; 
    CREATE EXTERNAL TABLE PartitionedCarEvents 
    (
                vin                                string,
                model                            string,
                timestamp                        string,
                outsidetemperature                string,
                enginetemperature                string,
                speed                            string,
                fuel                            string,
                engineoil                        string,
                tirepressure                    string,
                odometer                        string,
                city                            string,
                accelerator_pedal_position        string,
                parking_brake_status            string,
                headlamp_status                    string,
                brake_pedal_status                string,
                transmission_gear_position        string,
                ignition_status                    string,
                windshield_wiper_status            string,
                abs                              string,
                gendate                            string
    ) partitioned by (YearNo int, MonthNo int) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:PARTITIONEDOUTPUT}';

    DROP TABLE IF EXISTS Stage_RawCarEvents; 
    CREATE TABLE IF NOT EXISTS Stage_RawCarEvents 
    (
                vin                                string,
                model                            string,
                timestamp                        string,
                outsidetemperature                string,
                enginetemperature                string,
                speed                            string,
                fuel                            string,
                engineoil                        string,
                tirepressure                    string,
                odometer                        string,
                city                            string,
                accelerator_pedal_position        string,
                parking_brake_status            string,
                headlamp_status                    string,
                brake_pedal_status                string,
                transmission_gear_position        string,
                ignition_status                    string,
                windshield_wiper_status            string,
                abs                              string,
                gendate                            string,
                YearNo                             int,
                MonthNo                         int) 
    ROW FORMAT delimited fields terminated by ',' LINES TERMINATED BY '10';

    INSERT OVERWRITE TABLE Stage_RawCarEvents
    SELECT
        vin,            
        model,
        timestamp,
        outsidetemperature,
        enginetemperature,
        speed,
        fuel,
        engineoil,
        tirepressure,
        odometer,
        city,
        accelerator_pedal_position,
        parking_brake_status,
        headlamp_status,
        brake_pedal_status,
        transmission_gear_position,
        ignition_status,
        windshield_wiper_status,
        abs,
        gendate,
        Year(gendate),
        Month(gendate)

    FROM RawCarEvents WHERE Year(gendate) = ${hiveconf:Year} AND Month(gendate) = ${hiveconf:Month}; 

    INSERT OVERWRITE TABLE PartitionedCarEvents PARTITION(YearNo, MonthNo) 
    SELECT
        vin,            
        model,
        timestamp,
        outsidetemperature,
        enginetemperature,
        speed,
        fuel,
        engineoil,
        tirepressure,
        odometer,
        city,
        accelerator_pedal_position,
        parking_brake_status,
        headlamp_status,
        brake_pedal_status,
        transmission_gear_position,
        ignition_status,
        windshield_wiper_status,
        abs,
        gendate,
        YearNo,
        MonthNo
    FROM Stage_RawCarEvents WHERE YearNo = ${hiveconf:Year} AND MonthNo = ${hiveconf:Month};

成功執行管線之後，您會在儲存體帳戶的 connectedcar 容器下看到已產生下列分割區：

![分割的輸出](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig12-vehicle-telematics-partitioned-output.png)

資料現在已最佳化、更容易管理，可供進一步處理以更深入了解批次。 

## <a name="data-analysis"></a>資料分析
在本節中，您會了解如何結合串流分析、Azure Machine Learning、Data Factory 和 HDInsight，以對車輛健全狀況和駕駛習慣進行充分的進階分析。

### <a name="machine-learning"></a>機器學習服務
這裡的目標是以下列假設為基礎，根據特定的健全狀況統計資料，預測需要維修或召回的車輛：

* 如果下列三個條件之一成立，則車輛需要維修：
  
  * 胎壓偏低。
  * 機油量偏低。
  * 引擎溫度偏高。

* 如果下列其中一個條件成立，則車輛可能有安全性問題而需要召回：
  
  * 引擎溫度偏高，但外部溫度偏低。
  * 引擎溫度偏低，但外部溫度偏高。

根據先前的需求，兩個不同的模型偵測到異常情況。 其中一個模型適用於維修偵測，而另一個模型適用於車輛召回偵測。 在這兩個模型中，使用內建的主成分分析 (PCA) 演算法來偵測異常。 

#### <a name="maintenance-detection-model"></a>**維修偵測模型**

如果三個指標 (胎壓、機油或引擎溫度) 的其中一個符合其各自的條件，維修偵測模型會報告異常。 因此，在建立模型時只需要考慮這三個變數。 在機器學習的實驗中，會使用 [選取資料集中的資料行] 模組來擷取這三個變數。 接下來，使用以 PCA 為基礎的異常偵測模組來建立異常偵測模型。 

PCA 是已建立的機器學習服務技術，可運用於特徵選取、分類及異常偵測。 PCA 會將一組包含可能相關變數的案例，轉換成一組稱為主成分的值。 以 PCA 為基礎的模型化主要概念是將資料投射至較低維度的空間，以便更輕鬆地識別特徵和異常。

針對偵測模型的每個新輸入，異常偵測器會先計算它在特徵向量上的投影。 接著計算正規化重構錯誤。 此正規化錯誤為異常分數：錯誤越高，表示實例越異常。 

在維修偵測問題中，每一筆記錄會被視為由胎壓、機油和引擎溫度座標所定義的三維空間中的一個點。 為了擷取這些異常，PCA 可用於將三維空間中的原始資料投射到二維空間。 因此，PCA 中使用的「成分數目」參數會設為 2。 在套用以 PCA 為基礎的異常偵測時，這個參數扮演重要的角色。 使用 PCA 來投射資料之後，可更輕鬆地識別這些異常。

#### <a name="recall-anomaly-detection-model"></a>**召回異常偵測模型**

在召回異常偵測模型中，[選取資料集中的資料行] 和以 PCA 為基礎的異常偵測模組會以類似的方式使用。 具體來說，先使用 [選取資料集中的資料行] 模組擷取三個變數 - 引擎溫度、外部溫度和速度。 因為引擎溫度通常與速度相互關聯，所以也會包含速度變數。 接下來，以 PCA 為基礎的異常偵測模組可用來將三維空間的資料投射到二維空間。 已滿足召回準則。 當引擎溫度和外部溫度呈現高度負相關時，車輛就需要召回。 在執行 PCA 之後，以 PCA 為基礎的異常偵測演算法可用於擷取異常。 

在訓練其中一種模型時，正常資料可作為輸入資料來訓練以 PCA 為基礎的異常偵測模型。 (正常資料不需要維修或召回。)在評分實驗中，已訓練的異常偵測模型可用來偵測車輛是否需要維修或召回。 

### <a name="real-time-analysis"></a>即時分析
下列串流分析 SQL 查詢可用來取得所有重要車輛參數的平均值。 這些參數包括車輛速度、燃油量、引擎溫度、里程表讀數、胎壓、機油量等等。 平均值可用來偵測異常、發出警示，以及判斷在特定區域行駛的車輛的整體健全狀況。 然後將平均值與人口統計資料相互關聯。 

![用於即時處理的串流分析查詢](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig13-vehicle-telematics-stream-analytics-query-for-real-time-processing.png)

所有平均值都以 3 秒的輪轉時段來計算。 因為需要非重疊的連續時間間隔，所以會使用輪轉時段。 

若要深入了解 Azure 串流分析中的時間範圍功能，請參閱[時間範圍 (Azure 串流分析)](https://msdn.microsoft.com/library/azure/dn835019.aspx)。

#### <a name="real-time-prediction"></a>**即時預測**

此方案包含一個應用程式來即時操作機器學習模型。 在部署方案時會建立和設定 RealTimeDashboardApp 應用程式。 應用程式：

* 接聽由串流分析以某個模式連續發佈事件的事件中樞執行個體。

    ![用於發佈資料的串流分析查詢](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig14-vehicle-telematics-stream-analytics-query-for-publishing.png) 

* 接收事件。 對於此應用程式收到的每個事件： 
   
   * 使用機器學習服務的要求-回應評分 (RRS) 端點來處理資料。 部署時會自動發佈 RRS 端點。
   * RRS 輸出會透過推送 API 發佈至 PowerBI 資料集。

此模式也適用於您想要整合企業營運應用程式與即時分析流程的案例中。 這些案例包括警示、通知和傳訊。

注意：RealtimeDashboardApp Visual Studio 方案的資料已無法再使用。

#### <a name="execute-the-real-time-dashboard-application"></a>**執行即時儀表板應用程式**
1. 擷取 RealtimeDashboardApp，並將它儲存在本機。

    ![RealTimeDashboardApp 資料夾](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig16-vehicle-telematics-realtimedashboardapp-folder.png) 

2. 執行 RealtimeDashboardApp.exe 應用程式。

3. 輸入有效的 Power BI 認證，然後選取 [登入]。  

    ![即時儀表板應用程式登入視窗](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig17a-vehicle-telematics-realtimedashboardapp-sign-in-to-powerbi.png) 
    
4. 選取 [接受]。

    ![即時儀表板應用程式最終登入視窗](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig17b-vehicle-telematics-realtimedashboardapp-sign-in-to-powerbi.png) 

>[!NOTE] 
>如果您想要排清 Power BI 資料集，請使用 "flushdata" 參數來執行 RealtimeDashboardApp。 

    RealtimeDashboardApp.exe -flushdata


### <a name="batch-analysis"></a>批次分析
這裡的目標是顯示 Contoso Motors 如何利用 Azure 計算功能來處理巨量資料。 此資料可讓您深入了解駕駛模式、使用行為和車輛健全狀況。 此資訊可達成下列目標：

* 改善客戶體驗，更輕鬆了解駕駛習慣和省油的駕駛行為。
* 主動了解客戶及其駕駛模式，以制定商務決策並提供最佳的產品與服務。

在此方案中，以達成下列度量為目標：

* **危險駕駛行為**：識別車型、位置、駕駛狀況和該年行駛時間等趨勢，以深入了解危險駕駛模式。 Contoso Motors 可使用這些深入解析來進行行銷活動，引進全新的個人化功能和以使用狀況為基礎的保險。
* **省油駕駛行為**：識別車型、位置、駕駛狀況和該年行駛時間等趨勢，以深入了解省油駕駛模式。 Contoso Motors 可使用這些深入解析來進行行銷活動，引進全新的功能和主動回報駕駛人，進而養成節省成本和符合環保的駕駛習慣。
* **召回車型**：進行異常偵測機器學習服務實驗，以識別需要召回的車型。

讓我們看一看上述每個度量的詳細資料。

#### <a name="aggressive-driving-behavior-patterns"></a>**危險駕駛行為模式**

分割的車輛訊號和診斷資料會在 FuelEfficientDrivingPatternPipeline 中處理，如下列工作流程所示。 使用 Hive 來判斷呈現危險駕駛模式的車型、位置、車輛、駕駛狀況和其他參數。

![危險駕駛模式工作流程](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig18-vehicle-telematics-aggressive-driving-pattern.png) 

危險駕駛模式 Hive 查詢

使用 Hive 指令碼 aggresivedriving.hql 來分析危險駕駛狀況模式。 它位於所下載 zip 檔案的 \demo\src\connectedcar\scripts 資料夾中。 

    DROP TABLE IF EXISTS PartitionedCarEvents; 
    CREATE EXTERNAL TABLE PartitionedCarEvents
    (
                vin                                string,
                model                            string,
                timestamp                        string,
                outsidetemperature                string,
                enginetemperature                string,
                speed                            string,
                fuel                            string,
                engineoil                        string,
                tirepressure                    string,
                odometer                        string,
                city                            string,
                accelerator_pedal_position        string,
                parking_brake_status            string,
                headlamp_status                    string,
                brake_pedal_status                string,
                transmission_gear_position        string,
                ignition_status                    string,
                windshield_wiper_status            string,
                abs                              string,
                gendate                            string

    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:PARTITIONEDINPUT}';

    DROP TABLE IF EXISTS CarEventsAggresive; 
    CREATE EXTERNAL TABLE CarEventsAggresive
    (
                   vin                         string, 
                model                        string,
                timestamp                    string,
                city                        string,
                speed                          string,
                transmission_gear_position    string,
                brake_pedal_status            string,
                Year                        string,
                Month                        string

    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:AGGRESIVEOUTPUT}';



    INSERT OVERWRITE TABLE CarEventsAggresive
    select
    vin,
    model,
    timestamp,
    city,
    speed,
    transmission_gear_position,
    brake_pedal_status,
    "${hiveconf:Year}" as Year,
    "${hiveconf:Month}" as Month
    from PartitionedCarEvents
    where transmission_gear_position IN ('fourth', 'fifth', 'sixth', 'seventh', 'eight') AND brake_pedal_status = '1' AND speed >= '50'


此指令碼使用車輛的傳輸齒輪位置、剎車踏板狀態和速度的組合，根據高速狀態下的剎車模式來偵測魯莽/危險駕駛行為。 

成功執行管線之後，您會在儲存體帳戶的 connectedcar 容器下看到已產生下列分割區：

![AggressiveDrivingPatternPipeline 輸出](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig19-vehicle-telematics-aggressive-driving-pattern-output.png) 


#### <a name="fuel-efficient-driving-behavior-patterns"></a>**省油駕駛行為模式**

分割的車輛訊號和診斷資料會在 FuelEfficientDrivingPatternPipeline 中處理，如下列工作流程所示。 使用 Hive 來判斷呈現省油駕駛模式的車型、位置、車輛、駕駛狀況和其他屬性。

![省油駕駛模式](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig19-vehicle-telematics-fuel-efficient-driving-pattern.png) 

省油駕駛模式 Hive 查詢

使用 Hive 指令碼 fuelefficientdriving.hql 來分析省油駕駛狀況模式。 它位於所下載 zip 檔案的 \demo\src\connectedcar\scripts 資料夾中。 

    DROP TABLE IF EXISTS PartitionedCarEvents; 
    CREATE EXTERNAL TABLE PartitionedCarEvents
    (
                vin                                string,
                model                            string,
                timestamp                        string,
                outsidetemperature                string,
                enginetemperature                string,
                speed                            string,
                fuel                            string,
                engineoil                        string,
                tirepressure                    string,
                odometer                        string,
                city                            string,
                accelerator_pedal_position        string,
                parking_brake_status            string,
                headlamp_status                    string,
                brake_pedal_status                string,
                transmission_gear_position        string,
                ignition_status                    string,
                windshield_wiper_status            string,
                abs                              string,
                gendate                            string

    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:PARTITIONEDINPUT}';

    DROP TABLE IF EXISTS FuelEfficientDriving; 
    CREATE EXTERNAL TABLE FuelEfficientDriving
    (
                   vin                         string, 
                model                        string,
                   city                        string,
                speed                          string,
                transmission_gear_position    string,                
                brake_pedal_status            string,            
                accelerator_pedal_position    string,                             
                Year                        string,
                Month                        string

    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:FUELEFFICIENTOUTPUT}';



    INSERT OVERWRITE TABLE FuelEfficientDriving
    select
    vin,
    model,
    city,
    speed,
    transmission_gear_position,
    brake_pedal_status,
    accelerator_pedal_position,
    "${hiveconf:Year}" as Year,
    "${hiveconf:Month}" as Month
    from PartitionedCarEvents
    where transmission_gear_position IN ('fourth', 'fifth', 'sixth', 'seventh', 'eight') AND parking_brake_status = '0' AND brake_pedal_status = '0' AND speed <= '60' AND accelerator_pedal_position >= '50'


此指令碼使用車輛的傳輸齒輪位置、剎車踏板狀態、速度和油門踏板位置的組合，根據油門、剎車和速度模式來偵測省油駕駛行為。 

成功執行管線之後，您會在儲存體帳戶的 connectedcar 容器下看到已產生下列分割區：

![FuelEfficientDrivingPatternPipeline 輸出](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig20-vehicle-telematics-fuel-efficient-driving-pattern-output.png) 

**召回車型預測**

方案部署時會將機器學習服務實驗佈建並發佈為 Web 服務。 批次評分端點會使用於這個工作流程。 它會註冊為資料處理站連結服務，並使用資料處理站批次評分活動來運作。

![機器學習服務端點](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig21-vehicle-telematics-machine-learning-endpoint.png) 

透過異常偵測模型，在 DetectAnomalyPipeline 中使用已註冊的連結服務來替資料評分。 

![資料處理站中的機器學習服務批次評分活動](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig22-vehicle-telematics-aml-batch-scoring.png)  

在此管線中執行幾個步驟來準備資料，以利用於使用批次評分 Web 服務來操作。 

![用於召回預測的 DetectAnomalyPipeline](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig23-vehicle-telematics-pipeline-predicting-recalls.png)  

異常偵測 Hive 查詢

評分完成之後，HDInsight 活動會處理並彙總由模型歸類為異常的資料。 此模型會使用 0.60 或更高的機率分數。

    DROP TABLE IF EXISTS CarEventsAnomaly; 
    CREATE EXTERNAL TABLE CarEventsAnomaly 
    (
                vin                            string,
                model                        string,
                gendate                        string,
                outsidetemperature            string,
                enginetemperature            string,
                speed                        string,
                fuel                        string,
                engineoil                    string,
                tirepressure                string,
                odometer                    string,
                city                        string,
                accelerator_pedal_position    string,
                parking_brake_status        string,
                headlamp_status                string,
                brake_pedal_status            string,
                transmission_gear_position    string,
                ignition_status                string,
                windshield_wiper_status        string,
                abs                          string,
                maintenanceLabel            string,
                maintenanceProbability        string,
                RecallLabel                    string,
                RecallProbability            string

    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:ANOMALYOUTPUT}';

    DROP TABLE IF EXISTS RecallModel; 
    CREATE EXTERNAL TABLE RecallModel 
    (

                vin                            string,
                model                        string,
                city                        string,
                outsidetemperature            string,
                enginetemperature            string,
                speed                        string,
                Year                        string,
                Month                        string                

    ) ROW FORMAT DELIMITED FIELDS TERMINATED BY ',' LINES TERMINATED BY '10' STORED AS TEXTFILE LOCATION '${hiveconf:RECALLMODELOUTPUT}';

    INSERT OVERWRITE TABLE RecallModel
    select
    vin,
    model,
    city,
    outsidetemperature,
    enginetemperature,
    speed,
    "${hiveconf:Year}" as Year,
    "${hiveconf:Month}" as Month
    from CarEventsAnomaly
    where RecallLabel = '1' AND RecallProbability >= '0.60'


成功執行管線之後，您會在儲存體帳戶的 connectedcar 容器下看到已產生下列分割區：

![DetectAnomalyPipeline 輸出](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig24-vehicle-telematics-detect-anamoly-pipeline-output.png) 

## <a name="publish"></a>發佈

### <a name="real-time-analysis"></a>即時分析
串流分析作業的其中一個查詢會將事件發佈至輸出事件中樞執行個體。 

![已發佈至輸出事件中樞執行個體的串流分析作業](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig25-vehicle-telematics-stream-analytics-job-publishes-output-event-hub.png)

下列串流分析查詢用於發佈至輸出事件中樞執行個體：

![要發佈至輸出事件中樞執行個體的串流分析查詢](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig26-vehicle-telematics-stream-analytics-query-publish-output-event-hub.png)

此方案包含的 RealTimeDashboardApp 會取用此事件串流。 此應用程式會使用機器學習服務的要求-回應 Web 服務進行即時評分。 它會將產生的資料發佈到 PowerBI 資料集以供取用。 

### <a name="batch-analysis"></a>批次分析
批次和即時處理的結果會發佈到 Azure SQL Database 資料表以供取用。 安裝指令碼會自動建立 SQL Server、資料庫和資料表。 

批次處理結果會複製到資料超市工作流程。

![已複製到資料超市工作流程的批次處理結果](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig27-vehicle-telematics-batch-processing-results-copy-to-data-mart.png)

串流分析作業會發佈至資料超市。

![已發佈至資料超市的串流分析作業](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig28-vehicle-telematics-stream-analytics-job-publishes-to-data-mart.png)

資料超市設定位於串流分析作業中。

![串流分析作業中的資料超市設定](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig29-vehicle-telematics-data-mart-setting-in-stream-analytics-job.png)

## <a name="consume"></a>取用
Power BI 給此方案一個豐富的儀表板來提供即時資料和預測性分析視覺效果。 

最終的儀表板如此範例所示：

![Power BI 儀表板](./media/cortana-analytics-playbook-vehicle-telemetry-deep-dive/fig30-vehicle-telematics-powerbi-dashboard.png)

## <a name="summary"></a>總結
本文件包含車輛遙測資料分析方案的詳細探討。 Lambda 架構模式可用於以預測和動作進行即時和批次分析。 此模式適用於需要最忙碌路徑 (即時) 和不忙碌路徑 (批次) 分析的各種使用案例。 

### <a name="references"></a>參考

* [Azure 事件中樞](https://azure.microsoft.com/services/event-hubs/)
* [Azure Data Factory](https://azure.microsoft.com/documentation/learning-paths/data-factory/)
* [適用於串流擷取的 Azure 事件中樞 SDK](../../event-hubs/event-hubs-csharp-ephcs-getstarted.md)
* [Azure Data Factory 資料移動功能](../../data-factory/copy-activity-overview.md)
* [Azure Data Factory .NET 活動](../../data-factory/transform-data-using-dotnet-custom-activity.md)
* [用於準備範例資料的 Azure Data Factory .NET 活動 Visual Studio 方案](http://go.microsoft.com/fwlink/?LinkId=717077) 
