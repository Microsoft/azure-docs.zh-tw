---
title: 資料管理閘道的版本資訊 | Microsoft Docs
description: 資料管理閘道 tory 版本資訊
services: data-factory
author: nabhishek
manager: craigg
ms.assetid: 14762e82-76d9-41c4-ba9f-14a54da29c36
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: abnarain
robots: noindex
ms.openlocfilehash: ac0e1945e75ee7aea346c103a671b4a47b9e5994
ms.sourcegitcommit: 0c490934b5596204d175be89af6b45aafc7ff730
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/27/2018
ms.locfileid: "37048609"
---
# <a name="release-notes-for-data-management-gateway"></a>資料管理閘道的版本資訊
> [!NOTE]
> 本文適用於 Data Factory 第 1 版。 如果您使用目前版本的 Data Factory 服務，請參閱 [第 2 版中的自我裝載整合執行階段](../create-self-hosted-integration-runtime.md)。

現代資料整合的挑戰之一就是在內部部署和雲端之間來回移動資料。 Data Factory 會透過「資料管理閘道」進行此整合；「資料管理閘道」是一個您可以在內部部署環境安裝來啟用混合式資料移動功能的代理程式。

如需有關「資料管理閘道」的詳細資訊及其使用方式，請參閱下列文章：

*  [資料管理閘道](data-factory-data-management-gateway.md)
*  [在內部部署和雲端之間使用 Azure Data Factory 移動資料](data-factory-move-data-between-onprem-and-cloud.md)


## <a name="current-version"></a>目前版本 
我們不會再於此處維護版本資訊。 請至[此處](https://go.microsoft.com/fwlink/?linkid=853077)取得最新的版本資訊




## <a name="earlier-versions"></a>較早的版本
## <a name="21063477"></a>2.10.6347.7
### <a name="enhancements-"></a>增強功能
- 您可以新增 DNS 項目來將「服務匯流排」加入白名單，而不是將所有 Azure IP 位址都加入防火牆的白名單 (如有需要)。 您可以在 Azure 入口網站上找到各自的 DNS 項目 (Data Factory-> [製作和部署] -> [閘道] -> [serviceUrls] \(在 JSON 中)
- HDFS 連接器現在支援自我簽署的公開憑證，方法是讓您略過 SSL 驗證。
- 已修正︰更新期間的閘道離線問題 (因為時鐘誤差)


## <a name="2963132"></a>2.9.6313.2
### <a name="enhancements-"></a>增強功能
-   您可以新增 DNS 項目來將「服務匯流排」加入白名單，而不是將所有 Azure IP 位址都加入防火牆的白名單 (如有需要)。 以下提供更多詳細資料。
-   您現在可以將資料複製到單一區塊 Blob 或從該 Blob 複製資料，Blob 最大可達 4.75 TB，這是區塊 Blob 的最大支援大小。 (之前的上限為 195 GB)。
-   已修正：在進行複製活動期間將數個較小檔案解壓縮時發生的憶體不足問題。
-   已修正：使用等冪功能從 Document DB 複製到內部部署 SQL Server 時發生的索引超出範圍問題。
-   已修正：來自「複製精靈」的 SQL Server 清除指令碼無法在內部部署 SQL 上運作。
-   已修正：結尾含有空格的資料行名稱在複製活動中無法運作。

## <a name="28662833"></a>2.8.66283.3
### <a name="enhancements-"></a>增強功能
- 已修正：在閘道機器重新啟動時發生遺失認證的問題。
- 已修正：使用備份檔進行閘道還原時發生的註冊問題。


## <a name="2762401"></a>2.7.6240.1
### <a name="enhancements-"></a>增強功能
- 已修正：從作為來源的 Oracle 讀取的十進位 Null 值不正確。

## <a name="2661922"></a>2.6.6192.2
### <a name="whats-new"></a>新功能
- 客戶可以提供關於閘道註冊體驗的意見。
- 支援新的壓縮格式︰ZIP (Deflate)

### <a name="enhancements-"></a>增強功能
- 改善 Oracle 接收、HDFS 來源的效能。
- 針對閘道自動更新、閘道平行處理容量進行錯誤修正。


## <a name="2561641"></a>2.5.6164.1
### <a name="enhancements"></a>增強功能
- 改進而且更穩固的閘道註冊體驗 - 現在您可以在閘道註冊程序期間追蹤進度狀態，讓註冊體驗回應更加靈敏。
- 改善閘道還原程序 - 您仍然可以復原閘道，即使您沒有具備這項更新的閘道備份檔案。 需要您重設入口網站中的連結服務認證。
- 錯誤修正。

## <a name="2461511"></a>2.4.6151.1

### <a name="whats-new"></a>新功能

- 您現在可以在本機上儲存資料來源認證。 認證會加密。 您可以使用可從現有閘道匯出的備份檔案來復原和還原資料來源認證，所有這些操作都在內部部署環境中完成。

### <a name="enhancements-"></a>增強功能

- 改良和更強大的閘道註冊體驗。
- 複製精靈中支援自動偵測文字格式的 QuoteChar 組態並改善整體格式偵測精確度。

## <a name="2361002"></a>2.3.6100.2

- 複製精靈中支援 firstRowAsHeader 和 SkipLineCount 自動偵測內部部署檔案系統和 HDFS 中是否有文字檔案。
- 加強閘道與服務匯流排之間網路連線的穩定性。
- 修正幾個錯誤。


## <a name="2260721"></a>2.2.6072.1

*  支援使用「閘道組態管理員」來設定閘道的 HTTP Proxy。 如果有設定，就會透過 HTTP Proxy 來存取 Azure Blob、「Azure 資料表」、Azure Data Lake 及 DocumentDB。
*  從 Azure Blob、Azure Data Lake Store、內部部署「檔案系統」及內部部署 HDFS 複製資料，或將資料複製到這些位置時，支援處理 TextFormat 的標頭。
*  除了已支援的「區塊 Blob」之外，也支援從「附加 Blob」和「分頁 Blob」複製資料。
*  導入新的閘道狀態「線上 (受限)」 ，此狀態代表除了對「複製精靈」的互動式操作支援之外，閘道的主要功能都可運作。
*  使用註冊金鑰增強閘道註冊的健全度。

## <a name="216040"></a>2.1.6040.

*  DB2 驅動程式現已包含在閘道安裝封裝中。 您不需要另外安裝。
*  DB2 驅動程式現可支援適用於 i (AS/400) 的 z/OS 和 DB2 以及早已支援的平台 (Linux、Unix 和 Windows)。
*  支援使用 Azure Cosmos DB 作為內部部署資料存放區的來源或目的地
*  支援在冷/熱 Blob 儲存體以及早已支援的一般用途儲存體帳戶來回複製資料。
*  可讓您透過閘道以遠端登入權限連線到內部部署 SQL Server。  

## <a name="2060131"></a>2.0.6013.1

*  您可以選取要在手動安裝期間供閘道器使用的語言/文化特性。

*  當閘道未如預期般運作時，您可以選擇將過去 7 天的閘道記錄檔傳送給 Microsoft，以協助進行問題的疑難排解。 如果閘道器未連接到雲端服務，您可以選擇儲存並封存閘道器記錄檔。  

*  閘道器組態管理員的使用者介面增強功能：

    *  讓閘道器狀態在 [常用] 索引標籤上看起來更清楚。

    *  重新組織並簡化控制項。

    *  您可以使用[無程式碼複製工具](data-factory-copy-data-wizard-tutorial.md)儲存體中的資料。 如需此功能的一般詳細資料，請參閱 [分段複製](data-factory-copy-activity-performance.md#staged-copy) 。
*  您可以使用「資料管理閘道」，將資料從內部部署 SQL Server 資料庫直接輸入到 Azure Machine Learning 中。

*  效能改進

    * 在無程式碼複製工具中，改進 SQL Server 檢視結構描述或預覽的效能。

## <a name="11259531"></a>1.12.5953.1

*  錯誤修正

## <a name="11159181"></a>1.11.5918.1

*  閘道事件記錄檔的大小上限已經從 1 MB 增加到 40 MB。

*  如果在閘道自動更新期間必須重新啟動，則會顯示警告對話方塊。 您可以選擇立即重新啟動，或之後再重新啟動。

*  如果自動更新失敗，閘道安裝程式最多會重試自動更新 3 次。

*  效能改進

    * 改善無程式碼複製案例中從內部部署伺服器載入大型資料表的效能。

*  錯誤修正

## <a name="11058921"></a>1.10.5892.1

*  效能改進

*  錯誤修正

## <a name="1958652"></a>1.9.5865.2

*  零接觸自動更新功能
*  閘道狀態指示器的新系統匣圖示
*  能夠從用戶端「立即更新」
*  能夠設定更新排程時間
*  用來開啟/關閉自動更新的 PowerShell 指令碼
*  JSON 格式支援  
*  效能改進
*  錯誤修正

## <a name="1858221"></a>1.8.5822.1

*  改善疑難排解體驗
*  效能改進
*  錯誤修正

### <a name="1757951"></a>1.7.5795.1

*  效能改進
*  錯誤修正

### <a name="1757641"></a>1.7.5764.1

*  效能改進
*  錯誤修正

### <a name="1657351"></a>1.6.5735.1

*  支援內部部署 HDFS 來源/接收器
*  效能改進
*  錯誤修正

### <a name="1656961"></a>1.6.5696.1

*  效能改進
*  錯誤修正

### <a name="1656761"></a>1.6.5676.1

*  在組態管理員上支援診斷工具
*  支援 Azure Data Factory 表格式資料來源的資料表資料行
*  針對 Azure Data Factory 支援 SQL DW
*  針對 Azure Data Factory 支援 BlobSource 和 FileSource 的 Reclusive
*  針對 Azure Data Factory 支援 BlobSink 和 FileSink 中與「二進位複製」相關的 CopyBehavior - MergeFiles、PreserveHierarchy 及 FlattenHierarchy
*  針對 Azure Data Factory 支援複製活動報告進度
*  針對 Azure Data Factory 支援資料來源連線驗證
*  錯誤修正

### <a name="1656721"></a>1.6.5672.1

*  針對 Azure Data Factory 支援 ODBC 資料來源的資料表名稱
*  效能改進
*  錯誤修正

### <a name="1656581"></a>1.6.5658.1

*  針對 Azure Data Factory 支援檔案接收
*  針對 Azure Data Factory 在二進位複製中支援保留階層
*  針對 Azure Data Factory 支援複製活動等冪性
*  錯誤修正

### <a name="1656401"></a>1.6.5640.1

*  針對 Azure Data Factory 另外支援 3 種資料來源 (ODBC、OData、HDFS)
*  針對 Azure Data Factory 在 csv 剖析器中支援引號字元
*  壓縮支援 (BZip2)
*  錯誤修正

### <a name="1556121"></a>1.5.5612.1

*  針對 Azure Data Factory 支援 5 種關聯式資料庫 (MySQL、PostgreSQL、DB2、Teradata 和 Sybase)
*  壓縮支援 (Gzip 和 Deflate)
*  效能改進
*  錯誤修正

### <a name="1455491"></a>1.4.5549.1

*  針對 Azure Data Factory 新增 Oracle 資料來源支援
*  效能改進
*  錯誤修正

### <a name="1454921"></a>1.4.5492.1

*  支援 Microsoft Azure Data Factory 和 Office 365 Power BI 服務的統一二進位檔
*  調整組態 UI 和註冊程序
*  Azure Data Factory - 針對 SQL Server 資料來源的 Azure 輸入和輸出支援

### <a name="1253031"></a>1.2.5303.1

*  修正逾時問題以支援更多耗時的資料來源連線。

### <a name="1155268"></a>1.1.5526.8

*  在設定期間，必要條件是需要 .NET Framework 4.5.1。

### <a name="1051442"></a>1.0.5144.2

*  沒有任何會影響 Azure Data Factory 案例的變更。
