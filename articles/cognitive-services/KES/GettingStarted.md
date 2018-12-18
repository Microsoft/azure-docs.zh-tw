---
title: 範例：開始使用 - 知識探索服務 API
titlesuffix: Azure Cognitive Services
description: 使用知識探索服務 (KES) API 建立跨學術出版物的互動式搜尋體驗的引擎。
services: cognitive-services
author: bojunehsu
manager: cgronlun
ms.service: cognitive-services
ms.component: knowledge-exploration
ms.topic: sample
ms.date: 03/26/2016
ms.author: paulhsu
ms.openlocfilehash: 6cee339793269af0e8060cce56f94fa81db6a6c5
ms.sourcegitcommit: f10653b10c2ad745f446b54a31664b7d9f9253fe
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/18/2018
ms.locfileid: "46124009"
---
# <a name="get-started-with-the-knowledge-exploration-service"></a>開始使用知識探索服務

在此逐步解說中，您要使用知識探索服務 (KES)，針對學術出版物建立互動式搜尋體驗的引擎。 您可以從[知識探索服務 SDK](https://www.microsoft.com/en-us/download/details.aspx?id=51488) 安裝命令列工具 [`kes.exe`](CommandLine.md) 和所有範例檔案。

學術出版物範例中包含由 Microsoft 研究人員所出版的 1000 篇學術論文範例。  每篇論文都會與標題、出版年份、作者和關鍵字相關聯。 每個作者都會以出版時的識別碼、姓名和所屬單位表示。 每個關鍵字都可以與一組同義字相關聯 (例如，關鍵字 "support vector machine" 可以與同義字 "svm" 相關聯)。

## <a name="define-the-schema"></a>定義結構描述

結構描述會描述網域中物件的屬性結構。 它會以 JSON 檔案格式指定每個屬性的名稱和資料類型。 下列範例是檔案 *Academic.schema* 的內容。

```json
{
  "attributes":[
    {"name":"Title", "type":"String"},
    {"name":"Year", "type":"Int32"},
    {"name":"Author", "type":"Composite"},
    {"name":"Author.Id", "type":"Int64", "operations":["equals"]},
    {"name":"Author.Name", "type":"String"},
    {"name":"Author.Affiliation", "type":"String"},
    {"name":"Keyword", "type":"String", "synonyms":"Keyword.syn"}
  ]
}
```

您可以在這裡將 *Title*、*Year* 和 *Keyword* 分別定義為字串、整數和字串屬性。 由於作者是以識別碼、姓名和所屬單位表示，因此您可以利用下列三個子屬性，將 *Author* 定義為複合屬性：*Author.Id*、*Author.Name* 和 *Author.Affiliation*。

根據預設，屬性支援可用於其資料類型的所有作業，包括 *equals*、*starts_with* 和 *is_between*。 作者識別碼僅會在內部當作識別碼使用，因此請覆寫預設值，並將 *equals* 指定為唯一的索引作業。

針對 *Keyword* 屬性，請允許在屬性定義中指定同義字檔案 *Keyword.syn*，以比對同義字和標準關鍵字值。 此檔案包含標準和同義字值組的清單：

```json
...
["support vector machine","support vector machines"]
["support vector machine","support vector networks"]
["support vector machine","support vector regression"]
["support vector machine","support vector"]
["support vector machine","svm machine learning"]
["support vector machine","svm"]
["support vector machine","svms"]
["support vector machine","vector machine"]
...
```

如需有關結構描述定義的其他資訊，請參閱[結構描述格式](SchemaFormat.md)。

## <a name="generate-data"></a>產生資料

資料檔案可描述要編製索引之出版物的清單，其中每行都會以 [JSON 格式](http://json.org/)指定論文的屬性值。  下列範例是資料檔案 *Academic.data* 中的一行，並設定為方便閱讀的格式：

```
...
{
    "logprob": -16.714,
    "Title": "the world wide telescope",
    "Year": 2001,
    "Author": [
        {
            "Id": 717694024,
            "Name": "alexander s szalay",
            "Affiliation": "microsoft"
        },
        {
            "Id": 2131537204,
            "Name": "jim gray",
            "Affiliation": "microsoft"
        }
    ]
}
...
```

在此程式碼片段中，您要將論文的 *Title* 和 *Year* 屬性分別指定為 JSON 字串和數字。 多個值是使用 JSON 陣列表示。 *Author* 是複合屬性，每個值都是使用包含其子屬性的 JSON 物件表示。 缺少值的屬性 (例如此案例中的 *Keyword*) 可以從 JSON 表示法中排除。

若要區分不同論文的可能性，請使用內建的 *logprob* 屬性，指定相對對數概率。 假設 0 和 1 之間的概率為 *p*，您可以使用 log(*p*) 計算對數概率，其中 log() 是自然對數函數。

如需詳細資訊，請參閱[資料格式](DataFormat.md)。

## <a name="build-a-compressed-binary-index"></a>建置壓縮二進位索引

在您有結構描述檔案和資料檔案之後，可以使用 [`kes.exe build_index`](CommandLine.md#build_index-command) 建置資料物件的壓縮二進位索引。 在此範例中，您要從輸入結構描述檔案 *Academic.schema* 和資料檔案 *Academic.data* 建置索引檔案 *Academic.index*。 使用下列命令：

`kes.exe build_index Academic.schema Academic.data Academic.index`

若要在 Azure 外快速進行原型設計，[`kes.exe build_index`](CommandLine.md#build_index-command) 可以從包含高達 10,000 個物件的資料檔案，在本機建置小型索引。 針對較大的資料檔案，您可以從 [Azure 中的 Windows VM](../../../articles/virtual-machines/windows/quick-create-portal.md) 內執行命令，或在 Azure 中執行遠端建置。 如需詳細資訊，請參閱[相應增加](#scaling-up)。

## <a name="use-an-xml-grammar-specification"></a>使用 XML 文法規格

這種文法會指定服務可以解譯的一組自然語言查詢，以及將這些自然語言查詢轉譯為語意查詢運算式的方式。 在此範例中，您要使用 *academic.xml* 中指定的文法：

```xml
<grammar root="GetPapers">

  <!-- Import academic data schema-->
  <import schema="Academic.schema" name="academic"/>

  <!-- Define root rule-->
  <rule id="GetPapers">
    <example>papers about machine learning by michael jordan</example>

    papers
    <tag>
      yearOnce = false;
      isBeyondEndOfQuery = false;
      query = All();
    </tag>

    <item repeat="1-" repeat-logprob="-10">
      <!-- Do not complete additional attributes beyond end of query -->
      <tag>AssertEquals(isBeyondEndOfQuery, false);</tag>

      <one-of>
        <!-- about <keyword> -->
        <item logprob="-0.5">
          about <attrref uri="academic#Keyword" name="keyword"/>
          <tag>query = And(query, keyword);</tag>
        </item>

        <!-- by <authorName> [while at <authorAffiliation>] -->
        <item logprob="-1">
          by <attrref uri="academic#Author.Name" name="authorName"/>
          <tag>authorQuery = authorName;</tag>
          <item repeat="0-1" repeat-logprob="-1.5">
            while at <attrref uri="academic#Author.Affiliation" name="authorAffiliation"/>
            <tag>authorQuery = And(authorQuery, authorAffiliation);</tag>
          </item>
          <tag>
            authorQuery = Composite(authorQuery);
            query = And(query, authorQuery);
          </tag>
        </item>

        <!-- written (in|before|after) <year> -->
        <item logprob="-1.5">
          <!-- Allow this grammar path to be traversed only once -->
          <tag>
            AssertEquals(yearOnce, false);
            yearOnce = true;
          </tag>
          <ruleref uri="#GetPaperYear" name="year"/>
          <tag>query = And(query, year);</tag>
        </item>
      </one-of>

      <!-- Determine if current parse position is beyond end of query -->
      <tag>isBeyondEndOfQuery = GetVariable("IsBeyondEndOfQuery", "system");</tag>
    </item>
    <tag>out = query;</tag>
  </rule>

  <rule id="GetPaperYear">
    <tag>year = All();</tag>
    written
    <one-of>
      <item>
        in <attrref uri="academic#Year" name="year"/>
      </item>
      <item>
        before
        <one-of>
          <item>[year]</item>
          <item>
            <attrref uri="academic#Year" op="lt" name="year"/>
          </item>
        </one-of>
      </item>
      <item>
        after
        <one-of>
          <item>[year]</item>
          <item>
            <attrref uri="academic#Year" op="gt" name="year"/>
          </item>
        </one-of>
      </item>
    </one-of>
    <tag>out = year;</tag>
  </rule>
</grammar>
```

如需有關文法規格語法的詳細資訊，請參閱[文法格式](GrammarFormat.md)。

## <a name="compile-the-grammar"></a>編譯文法

擁有 XML 文法規格之後，您可以使用 [`kes.exe build_grammar`](CommandLine.md#build_grammar-command) 將它編譯為二進位文法。 請注意，如果此文法會匯入結構描述，則結構描述檔案必須位於與文法 XML 相同的路徑中。 在此範例中，您要從輸入 XML 文法檔案 *Academic.xml* 建置二進位文法檔案 *Academic.grammar*。 使用下列命令：

`kes.exe build_grammar Academic.xml Academic.grammar`

## <a name="host-the-grammar-and-index-in-a-web-service"></a>在 Web 服務中裝載文法和索引

若要快速進行原型設計，您可以在本機電腦的 Web 服務中，使用 [`kes.exe host_service`](CommandLine.md#host_service-command) 裝載文法和索引。 接著，您可以透過 [Web API](WebAPI.md) 存取該服務，以驗證資料正確性和文法設計。 在此範例中，您要在 http://localhost:8000/ 裝載文法檔案 *Academic.grammar* 和索引檔案 *Academic.index*。 使用下列命令：

`kes.exe host_service Academic.grammar Academic.index --port 8000`

這樣會起始 Web 服務的本機執行個體。 您可以從瀏覽器瀏覽 `http::localhost:<port>`，以便以互動方式測試服務。 如需詳細資訊，請參閱[測試服務](#testing-service)。

您也可以直接呼叫多種 [Web API](WebAPI.md) 來測試自然語言解譯、查詢完成、結構化查詢評估，以及色階分佈圖計算。 若要停止服務，請在 `kes.exe host_service` 命令提示字元中輸入 "quit"，或按下 Ctrl+C。 這裡有一些範例：

* [http://localhost:8000/interpret?query=papers by susan t dumais](http://localhost:8000/interpret?query=papers%20by%20susan%20t%20dumais)
* [http://localhost:8000/interpret?query=papers by susan t d&complete=1](http://localhost:8000/interpret?query=papers%20by%20susan%20t%20d&complete=1)
* [http://localhost:8000/evaluate?expr=Composite(Author.Name=='susan t dumais')&attributes=Title,Year,Author.Name,Author.Id&count=2](http://localhost:8000/evaluate?expr=Composite%28Author.Name==%27susan%20t%20dumais%27%29&attributes=Title,Year,Author.Name,Author.Id&count=2)
* [http://localhost:8000/calchistogram?expr=And(Composite(Author.Name=='susan t dumais'),Year>=2013)&attributes=Year,Keyword&count=4](http://localhost:8000/calchistogram?expr=And%28Composite%28Author.Name=='susan%20t%20dumais'%29,Year>=2013%29&attributes=Year,Keyword&count=4)

在 Azure 之外，[`kes.exe host_service`](CommandLine.md#host_service-command) 限制為最多 10,000 個物件的索引。 其他限制則包括每秒 10 個要求的 API 速率，以及在處理序自動終止之前，總共 1000 個要求。 若要略過這些限制，請從 [Azure 中的 Windows VM](../../../articles/virtual-machines/windows/quick-create-portal.md) 內執行命令，或使用 [`kes.exe deploy_service`](CommandLine.md#deploy_service-command) 命令部署至 Azure 雲端服務。 如需詳細資訊，請參閱[部署服務](#deploying-service)。

## <a name="scale-up-to-host-larger-indices"></a>相應增加以裝載更大的索引

當您在 Azure 之外執行 `kes.exe` 時，索引限制為 10,000 個物件。 您可以使用 Azure 建置並裝載更大的索引。 註冊[免費試用](https://azure.microsoft.com/pricing/free-trial/)。 或者，如果您訂閱 Visual Studio 或 MSDN，便可以[啟用訂閱者權益](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)。 這些每個月都會提供一些 Azure 點數。

若要允許 `kes.exe` 存取 Azure 帳戶，請從 Azure 入口網站[下載 Azure 發行設定檔案](https://portal.azure.com/#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade)。 如果出現提示，請登入所需的 Azure 帳戶。 將檔案儲存為 `kes.exe` 執行所在工作目錄中的 *AzurePublishSettings.xml*。

建置並裝載大型索引有兩種方法。 第一種方法是在 Azure 中的 Windows VM 中準備結構描述和資料檔案。 接著，執行 [`kes.exe build_index`](#building-index) 以便在 VM 本機建置索引，沒有任何大小限制。 所產生的索引可以在 VM 本機使用 [`kes.exe host_service`](#hosting-service) 裝載以便快速進行原型設計，而且也沒有任何限制。 如需詳細步驟，請參閱 [Azure VM 教學課程](../../../articles/virtual-machines/windows/quick-create-portal.md)。

第二種方法是使用 [`kes.exe build_index`](CommandLine.md#build_index-command) 搭配 `--remote` 參數，執行遠端 Azure 建置。 這樣會指定 Azure VM 大小。 指定 `--remote` 參數時，此命令會建立該大小的臨時 Azure VM。 接著，它會在該 VM 上建置索引、將索引上傳至目標 Blob 儲存體，然後在完成時刪除該 VM。 建置索引時，系統會向您的 Azure 訂用帳戶收取 VM 成本的費用。

這個遠端 Azure 建置功能允許您在任何環境中執行 [`kes.exe build_index`](CommandLine.md#build_index-command)。 當您執行遠端建置時，輸入結構描述和資料引數可以是本機檔案路徑或 [Azure Blob 儲存體](../../storage/blobs/storage-dotnet-how-to-use-blobs.md) URL。 輸出索引引數必須是 Blob 儲存體 URL。 若要建立 Azure 儲存體帳戶，請參閱[關於 Azure 儲存體帳戶](../../storage/common/storage-create-storage-account.md)。 若要有效率地將檔案複製到 Blob 儲存體或從中複製檔案，請使用 [AzCopy](../../storage/common/storage-use-azcopy.md) 公用程式。

在此範例中，您可以假設已經建立下列 Blob 儲存體容器： http://&lt;*account*&gt;.blob.core.windows.net/&lt;*container*&gt;/。 它包含結構描述 *Academic.schema*、參照的同義字檔案 *Keywords.syn*，以及完整規模的資料檔案 *Academic.full.data*。 您可以使用下列命令，從遠端建置完整的索引：

`kes.exe build_index http://<account>.blob.core.windows.net/<container>/Academic.schema http://<account>.blob.core.windows.net/<container>/Academic.full.data http://<account>.blob.core.windows.net/<container>/Academic.full.index --remote <vm_size>`

請注意，佈建臨時 VM 來建置索引可能需要 5-10 分鐘的時間。 若要快速進行原型設計，您可以：
- 使用較小的資料集，在任何電腦本機開發。
- 手動[建立 Azure VM](../../../articles/virtual-machines/windows/quick-create-portal.md)、透過遠端桌面[連線到該 VM](../../../articles/virtual-machines/windows/quick-create-portal.md#connect-to-virtual-machine)、安裝[知識探索服務 SDK](https://www.microsoft.com/en-us/download/details.aspx?id=51488)，然後從 VM 內執行 [`kes.exe`](CommandLine.md)。

分頁會使建置程序變慢。 若要避免分頁，請使用具有輸入資料檔案大小三倍 RAM 數量的 VM 來進行索引建置。 使用 RAM 數量比索引大小多 1 GB 的 VM 來裝載。 如需可用 VM 大小的清單，請參閱[虛擬機器的大小](../../../articles/virtual-machines/virtual-machines-windows-sizes.md)。

## <a name="deploy-the-service"></a>部署服務

擁有文法與索引之後，即可開始將服務部署至 Azure 雲端服務。 若要建立新的 Azure 雲端服務，請參閱[如何建立及部署雲端服務](../../../articles/cloud-services/cloud-services-how-to-create-deploy-portal.md)。 請勿在此時指定部署套件。  

當您建立雲端服務之後，可以使用 [`kes.exe deploy_service`](CommandLine.md#deploy_service-command) 來部署服務。 Azure 雲端服務有兩個部署位置：生產和預備。 針對接收即時使用者流量的服務，您應該一開始就部署到預備位置。 等候服務啟動並初始化其本身。 接著，您可以傳送一些要求來驗證部署，並確認它通過基本測試。

[交換](../../../articles/cloud-services/cloud-services-nodejs-stage-application.md)預備位置與生產位置的內容，如此一來，即時流量現在會導向至新部署的服務。 使用新的資料部署更新的服務版本時，您可以重複此程序。 與其他所有 Azure 雲端服務一樣，您可以選擇性地使用 Azure 入口網站來設定[自動規模調整](../../../articles/cloud-services/cloud-services-how-to-scale-portal.md)。

在此範例中，您要使用 *<vm_size>* VM，將 *Academic* 索引部署至現有雲端服務的預備位置。 使用下列命令：

`kes.exe deploy_service http://<account>.blob.core.windows.net/<container>/Academic.grammar http://<account>.blob.core.windows.net/<container>/Academic.index <serviceName> <vm_size> --slot Staging`

如需可用 VM 大小的清單，請參閱[虛擬機器的大小](../../../articles/virtual-machines/virtual-machines-windows-sizes.md)。

部署服務之後，您可以呼叫多種 [Web API](WebAPI.md) 來測試自然語言解譯、查詢完成、結構化查詢評估，以及色階分佈圖計算。  

## <a name="test-the-service"></a>測試服務

若要針對即時服務進行偵錯，請從網頁瀏覽器瀏覽至主機電腦。 針對透過 [host_service](#hosting-service) 部署的本機服務，請瀏覽 `http://localhost:<port>/`。  針對透過 [deploy_service](#deploying-service) 部署的 Azure 雲端機服務，請瀏覽 `http://<serviceName>.cloudapp.net/`。

此頁面包含基本 API 呼叫統計資料，以及此服務所裝載文法和索引相關資訊的連結。 此頁面也包含可示範如何使用 Web API 的互動式搜尋介面。 若要查看 [interpret](interpretMethod.md)、[evaluate](evaluateMethod.md) 和 [calchistogram](calchistogramMethod.md) API 呼叫的結果，請在 [搜尋] 方塊中輸入查詢。 此頁面的底層 HTML 來源也可作為如何將 Web API 整合到應用程式以建立豐富互動式搜尋體驗的範例。


