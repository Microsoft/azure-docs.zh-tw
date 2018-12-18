---
title: 教學課程：使用 Azure BizTalk 服務處理 EDIFACT 發票 | Microsoft Docs
description: 如何建立並設定 Box 連接器或 API 應用程式，並在 Azure App Service 的邏輯應用程式中使用它
services: biztalk-services
documentationcenter: .net,nodejs,java
author: msftman
manager: erikre
editor: ''
ms.assetid: 7e0b93fa-3e2b-4a9c-89ef-abf1d3aa8fa9
ms.service: biztalk-services
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 05/31/2016
ms.author: deonhe
ms.openlocfilehash: 2ebd6a8cb70f218c3b56bc78c9b853dbf51ab468
ms.sourcegitcommit: e266df9f97d04acfc4a843770fadfd8edf4fa2b7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/11/2017
ms.locfileid: "26737545"
---
# <a name="tutorial-process-edifact-invoices-using-azure-biztalk-services"></a>教學課程：使用 Azure BizTalk 服務處理 EDIFACT 發票

> [!INCLUDE [BizTalk Services is being retired, and replaced with Azure Logic Apps](../../includes/biztalk-services-retirement.md)]

您可以使用 BizTalk 服務入口網站設定和部署 X12 與 EDIFACT 協議。 在本教學課程中，我們將說明如何建立 EDIFACT 協議以讓交易夥伴彼此交換發票。 本教學課程描述端對端商務方案，內容涉及兩個交易夥伴，分別是 Northwind 和 Contoso，他們會交換 EDIFACT 訊息。  

## <a name="sample-based-on-this-tutorial"></a>根據本教學課程所建立的範例
本教學課程描述 **使用 BizTalk 服務傳送 EDIFACT 發票**的範例，您可以從 [MSDN 程式碼資源庫](http://go.microsoft.com/fwlink/?LinkId=401005)下載此範例。 您可以使用此範例來完成本教學課程，以了解此範例的建置方式。 或者，您可以使用本教學課程從頭建立自己的方案。 本教學課程鎖定第二種方法，以便您能夠了解此方案的建置方式。 此外，本教學課程會盡可能與範例保持一致，並對構件使用範例中所使用的相同名稱 (例如，結構描述、轉換)。  

> [!NOTE]
> 此方案牽涉到從 EAI 橋接器傳送訊息至 EDI 橋接器，因此會重複使用 [BizTalk 服務橋接器鏈結範例](http://code.msdn.microsoft.com/BizTalk-Bridge-chaining-2246b104) 的範例。  
> 
> 

## <a name="what-does-the-solution-do"></a>此方案有哪些功能？
在此方案中，Northwind 會從 Contoso 收到 EDIFACT 發票。 這些發票未採用標準 EDI 格式。 因此在將發票傳送至 Northwind 之前，必須先將它轉換為 EDIFACT 發票 (也稱為 INVOIC) 文件。 在收到時，Northwind 必須處理 EDIFACT 發票，並回傳控制訊息 (也稱為 CONTRL) 給 Contoso。

![][1]  

為了達成此商務案例，Contoso 使用 Microsoft Azure BizTalk 服務所提供的功能。

* Contoso 使用 EAI 橋接器將原始發票轉換為 EDIFACT INVOIC。
* EAI 橋接器接著傳送訊息至部署為 BizTalk 服務入口網站中所設定之協議一部分的 EDI 傳送橋接器。
* EDI 傳送橋接器處理 EDIFACT INVOIC 並將其路由傳送至 Northwind。
* 接收到發票之後，Northwind 回傳 CONTRL 訊息給部署為協議一部分的 EDI 接收橋接器。  

> [!NOTE]
> (選擇性) 此方案也會示範如何使用批次功能來批次傳送發票，而不是個別傳送每張發票。  
> 
> 

為了完成案例，我們使用服務匯流排佇列將發票從 Contoso 傳送至 Northwind，或從 Northwind 接收通知。 這些佇列可以透過用戶端應用程式 (可經由下載取得) 來予以建立，並且會包含在本教學課程所提供的範例封裝中。  

## <a name="prerequisites"></a>先決條件
* 您必須具有服務匯流排命名空間。 如需建立命名空間的指示，請參閱 [作法：建立或修改服務匯流排服務命名空間](https://msdn.microsoft.com/library/azure/hh674478.aspx)。 讓我們假設您已佈建服務匯流排命名空間，且其名稱為 **edifactbts**。
* 您必須擁有 BizTalk 服務訂用帳戶。 在本教學課程中，讓我們假設您擁有 BizTalk 服務訂用帳戶，且其名稱為 **contosowabs**。
* 在 BizTalk 服務入口網站註冊 BizTalk 服務訂用帳戶。 如需相關指示，請參閱 [在 BizTalk 服務入口網站註冊 BizTalk 服務部署](https://msdn.microsoft.com/library/hh689837.aspx)
* 您必須安裝 Visual Studio。
* 您必須安裝 BizTalk 服務 SDK。 您可以從 [http://go.microsoft.com/fwlink/?LinkId=235057](http://go.microsoft.com/fwlink/?LinkId=235057)  

## <a name="step-1-create-the-service-bus-queues"></a>步驟 1：建立服務匯流排佇列
此方案使用服務匯流排佇列以讓交易夥伴彼此交換訊息。 Contoso 和 Northwind 會傳送訊息至 EAI 和 (或) EDI 橋接器從中取用訊息的佇列。 在此方案中，您需要三個服務匯流排佇列：

* **northwindreceive** – Northwind 透過此佇列從 Contoso 接收發票。
* **contosoreceive** – Contoso 透過此佇列從 Northwind 接收通知。
* **suspended** – 所有擱置的訊息會路由傳送至此佇列。 如果訊息在處理期間失敗，則會加以擱置。

您可以使用範例封裝中包含的用戶端應用程式建立這些服務匯流排佇列。  

1. 從您下載範例的位置，開啟 **使用 BizTalk 服務 EDI Bridges.sln 傳送發票的教學課程**。
2. 按 **F5** 以建置並啟動 **Tutorial Client** 應用程式。
3. 在畫面中輸入服務匯流排 ACS 命名空間、簽發者名稱和簽發者金鑰。
   
   ![][2]  
4. 隨即出現訊息方塊提示，指出將會在服務匯流排命名空間中建立三個佇列。 按一下 [SERVICEPRINCIPAL] 。
5. 讓 Tutorial Client 保持執行狀態。 開啟，按一下 [服務匯流排] > 您的服務匯流排命名空間 > [佇列]，然後驗證是否已建立這三個佇列。  

## <a name="step-2-create-and-deploy-trading-partner-agreement"></a>步驟 2：建立和部署交易夥伴協議
建立 Contoso 與 Northwind 之間的交易夥伴協議。 交易夥伴協議定義兩個商務夥伴之間的交易協議，例如要使用的訊息結構描述、要使用的訊息通訊協定等等。交易夥伴協議包含兩個 EDI 橋接器，一個用來傳送訊息給交易夥伴 (稱為 **EDI 傳送橋接器**)，另一個接收來自交易夥伴的訊息 (稱為 **EDI 接收橋接器**)。

在此方案的內容中，EDI 傳送橋接器對應到協議的傳送端，並且會用來將 EDIFACT 發票從 Contoso 傳送至 Northwind。 同樣地，EDI 接收橋接器則對應到協議的接收端，並且會用來從 Northwind 接收通知。  

### <a name="create-the-trading-partners"></a>建立交易夥伴
若要開始，請建立 Contoso 和 Northwind 的交易夥伴。  

1. 在 BizTalk 服務入口網站的 [夥伴] 索引標籤上，按一下 [新增]。
2. 在 [新增夥伴] 頁面上，輸入 **Contoso** 當做夥伴名稱，然後按一下 [儲存]。
3. 重複此步驟來建立第二個夥伴 ( **Northwind**)。  

### <a name="create-the-agreement"></a>建立協議
交易夥伴的商務設定檔之間會建立交易夥伴協議。 此方案會使用我們建立夥伴時自動建立的預設夥伴設定檔。  

1. 在 BizTalk 服務入口網站中，按一下 [協議] > [新增]。
2. 在新協議的 [一般設定] 頁面上指定值 (如下圖所示)，然後按一下 [繼續]。
   
   ![][3]  
   
   按一下 [繼續] 之後，[接收設定] 和 [傳送設定] 這兩個索引標籤會變成可用狀態。
3. 建立 Contoso 與 Northwind 之間的傳送協議。 此協議控管 Contoso 如何將 EDIFACT 發票傳送至 Northwind。
   
   1. 按一下 [傳送設定] 。
   2. 保留 [輸入 URL]、[轉換] 和 [批次處理] 索引標籤上的預設值。
   3. 在 [通訊協定] 索引標籤的 [結構描述] 區段底下，上傳 **EFACT_D93A_INVOIC.xsd** 結構描述。 此結構描述可在範例封裝內取得。
      
      ![][4]  
   4. 在 [傳輸] 索引標籤上，指定服務匯流排佇列的詳細資料。 對於傳送端協議，我們使用 **northwindreceive** 佇列來將 EDIFACT 發票傳送至 Northwind，並使用 **suspended** 佇列來路由傳送任何在處理期間失敗且因而擱置的訊息。 您已在**步驟 1：建立服務匯流排佇列** (在本主題) 中建立這些佇列。
      
      ![][5]  
      
      在 [傳輸設定] > [傳輸類型] 和 [訊息擱置設定] > [傳輸類型] 底下，選取 [Azure 服務匯流排] 並提供值 (如上圖所示)。
4. 建立 Contoso 與 Northwind 之間的接收協議。 此協議控管 Contoso 如何從 Northwind 接收通知。
   
   1. 按一下 [接收設定] 。
   2. 保留 [傳輸] 和 [轉換] 索引標籤上的預設值。
   3. 在 [通訊協定] 索引標籤的 [結構描述] 區段底下，上傳 **EFACT_4.1_CONTRL.xsd** 結構描述。 此結構描述可在範例封裝內取得。
   4. 在 [路由]  索引標籤上，建立篩選器以確保只有來自 Northwind 的通知會路由傳送至 Contoso。 在 [路由設定] 底下，按一下 [新增] 以建立路由篩選條件。
      
      ![][6]  
      
      1. 提供 [規則名稱]、[路由規則] 和 [路由目的地] 的值 (如上圖所示)。
      2. 按一下 [檔案] 。
   5. 再次在 [路由] 索引標籤中，指定要將擱置的通知 (在處理期間失敗的通知) 傳送到哪裡。 將傳輸類型設為 [Azure 服務匯流排]、將路由目的地類型設為 [佇列]、將驗證類型設為 [共用存取簽章] \(SAS)、提供服務匯流排命名空間的 SAS 連接字串，然後在佇列名稱中輸入 **suspended**。
5. 最後，按一下 [部署]  來部署協議。 請記下傳送和接收協議部署所在的端點。
   
   * 在 [傳送設定] 索引標籤的 [輸入 URL] 底下，記下該端點。 若要使用 EDI 傳送橋接器將訊息從 Contoso 傳送至 Northwind，您必須傳送訊息給此端點。
   * 在 [接收設定] 索引標籤的 [傳輸] 底下，記下該端點。 若要使用 EDI 接收橋接器將訊息從 Northwind 傳送至 Contoso，您必須傳送訊息給此端點。  

## <a name="step-3-create-and-deploy-the-biztalk-services-project"></a>步驟 3：建立和部署 BizTalk 服務專案
在上一個步驟中，您部署了 EDI 傳送和接收協議，以處理 EDIFACT 發票和通知。 這些協議只可以處理符合標準 EDIFACT 訊息結構描述的訊息。 不過，根據此方案的案例，Contoso 會以內部專屬的結構描述傳送發票給 Northwind。 因此，訊息在傳送給 EDI 傳送橋接器之前，必須先從內部結構描述轉換為標準 EDIFACT 發票結構描述。 BizTalk 服務 EAI 專案會負責執行此工作。

負責轉換訊息的 BizTalk 服務專案 **InvoiceProcessingBridge**也包含在您下載的範例中。 此專案包括下列構件：

* **INHOUSEINVOICE.XSD** – 傳送至 Northwind 之內部發票的結構描述。
* **EFACT_D93A_INVOIC.XSD** – 標準 EDIFACT 發票的結構描述。
* **EFACT_4.1_CONTRL.XSD** – Northwind 傳送給 Contoso 之 EDIFACT 通知的結構描述。
* **INHOUSEINVOICE_to_D93AINVOIC.TRFM** – 將內部發票結構描述對應至標準 EDIFACT 發票結構描述的轉換。  

### <a name="create-the-biztalk-services-project"></a>建立 BizTalk 服務專案
1. 在 Visual Studio 方案中，展開 InvoiceProcessingBridge 專案，然後開啟 **MessageFlowItinerary.bcs** 檔案。
2. 在畫布上任何位置按一下，然後在屬性方塊中設定 [BizTalk 服務 URL]  ，以指定 BizTalk 服務訂用帳戶名稱。 例如， `https://contosowabs.biztalk.windows.net`。
   
   ![][7]  
3. 從工具箱拖曳 **Xml 單向橋接器** 到畫布上。 將橋接器的 [實體名稱] 和 [相對位址] 屬性設定為 [ProcessInvoiceBridge]。 按兩下 [ProcessInvoiceBridge]  以開啟橋接器設定介面。
4. 在 [訊息類型] 方塊中，按一下加號 (**+**) 按鈕，以指定傳入訊息的結構描述。 因為 EAI 橋接器的傳入訊息一律是內部發票，因此請將此項目設為 [INHOUSEINVOICE] 。
   
   ![][8]  
5. 按一下 [Xml 轉換] 圖形，並在屬性方塊中，針對 [對應] 屬性按一下省略符號 (**...**) 按鈕。 在 [選取對應] 對話方塊中，選取 **INHOUSEINVOICE_to_D93AINVOIC** 轉換檔案，然後按一下 [確定]。
   
   ![][9]  
6. 返回 **MessageFlowItinerary.bcs**，然後將 [雙向外部服務端點] 從工具箱拖曳至 **ProcessInvoiceBridge** 右邊。 將其 [實體名稱] 屬性設為 [EDIBridge]。
7. 在 [方案總管] 中，展開 **MessageFlowItinerary.bcs**，然後按兩下 **EDIBridge.config** 檔案。 將 **EDIBridge.config** 的內容更換為下列程式碼。
   
   > [!NOTE]
   > 為何需要編輯 .config 檔案？ 我們新增至橋接器設計工具畫布上的外部服務端點代表我們稍早部署的 EDI 橋接器。 EDI 橋接器是具有傳送和接收端的雙向橋接器。 不過，我們新增至橋接器設計工具的 EAI 橋接器是單向橋接器。 因此，為了處理兩個橋接器的不同訊息交換模式，我們使用自訂的橋接器行為，將其設定納入 .config 檔案中。 此外，自訂行為也會處理 EDI 傳送橋接器端點的驗證。這個自訂行為是可從 [BizTalk 服務橋接器鏈結範例 - EAI 到 EDI](http://code.msdn.microsoft.com/BizTalk-Bridge-chaining-2246b104) 中取得的另一個範例。 此方案重複使用該範例。  
   > 
   > 
   
   ```
   <?xml version="1.0" encoding="utf-8"?>
   <configuration>
     <system.serviceModel>
       <extensions>
         <behaviorExtensions>
           <add name="BridgeAuthentication" 
                 type="Microsoft.BizTalk.Bridge.Behaviour.BridgeBehaviorElement, Microsoft.BizTalk.Bridge.Behaviour, Version=1.0.0.0, Culture=neutral, PublicKeyToken=ae58f69b69495c05" />
         </behaviorExtensions>
       </extensions>
       <behaviors>
         <endpointBehaviors>
           <behavior name="BridgeAuthenticationConfiguration">
             <!-- Enter the ACS namespace, issuer name and issuer secret of the BizTalk Services deployment -->
             <BridgeAuthentication acsnamespace="[YOUR ACS NAMESPACE]" 
                                   issuername="owner" 
                                   issuersecret="[YOUR ACS SECRET]" />
             <webHttp />
           </behavior>
         </endpointBehaviors>
       </behaviors>
       <bindings>
         <webHttpBinding>
           <binding name="BridgeBindingConfiguration">
             <security mode="Transport" />
           </binding>
         </webHttpBinding>
       </bindings>
       <client>
         <clear />
         <!--
           Go BizTalk Portal > Agreement > Send Settings > Inbound URL
           Copy the Endpoint URL and paste it in the below address field
         -->
         <endpoint name="TwoWayExternalServiceEndpointReference1" 
                   address="[YOUR EDI BRIDGE SEND URI]" 
                   behaviorConfiguration="BridgeAuthenticationConfiguration" 
                   binding="webHttpBinding" 
                   bindingConfiguration="BridgeBindingConfiguration" 
                   contract="System.ServiceModel.Routing.IRequestReplyRouter" />
       </client>
     </system.serviceModel>
   </configuration>
   
   ```
8. 更新 EDIBridge.config 檔案以納入設定詳細資料
   
   * 在 [傳輸設定] &gt; [傳輸類型] *<behaviors>* 下，提供與 BizTalk 服務訂用帳戶相關聯的 ACS 命名空間和金鑰。
   * 在 [傳輸設定] &gt; [傳輸類型] *<client>* 下，提供 EDI 傳送協議部署所在的端點。
   
   儲存變更並關閉設定檔。
9. 從 [工具箱] 中按一下 [連接器]，然後聯結 [ProcessInvoiceBridge] 和 [EDIBridge] 元件。 選取連接器，然後在 [屬性] 方塊中將 [篩選條件] 設定為 [全部符合]。 這可確保 EAI 橋接器處理的所有訊息都會路由傳送至 EDI 橋接器。
   
   ![][10]  
10. 儲存方案的變更。  

### <a name="deploy-the-project"></a>部署專案
1. 在您建立 BizTalk 服務專案的電腦上，下載並安裝您 BizTalk 服務訂用帳戶的 SSL 憑證。 從 [BizTalk 服務] 底下按一下 [儀表板]，然後按一下 [下載 SSL 憑證]。 按兩下憑證，並依照提示來完成安裝。 確定您是在 [受信任的根憑證授權單位] 憑證存放區底下安裝憑證。
2. 在 Visual Studio 的 [方案總管] 中，以滑鼠右鍵按一下 **InvoiceProcessingBridge** 專案，然後按一下 [部署]。
3. 提供如下圖所示的值，然後按一下 [部署] 。 您可以從 BizTalk 服務儀表板按一下 [連接資訊]  ，以取得 BizTalk 服務的 ACS 認證。
   
   ![][11]  
   
   從輸出窗格中，複製 EAI 橋接器部署所在的端點，比方說 `https://contosowabs.biztalk.windows.net/default/ProcessInvoiceBridge`。 您稍後將需要此端點 URL。  

## <a name="step-4-test-the-solution"></a>步驟 4：測試方案
在本主題中，我們將說明如何使用範例中提供的 **Tutorial Client** 應用程式來測試方案。  

1. 在 Visual Studio 中，按 F5 以啟動 **Tutorial Client**。
2. 畫面上必須已預先填入我們用來建立服務匯流排佇列之步驟所指定的值。 按 [下一步] 。
3. 在下一個視窗中，提供 BizTalk 服務訂用帳戶的 ACS 認證，以及 EAI 和 EDI (接收) 橋接器部署所在的端點。
   
   您已在上一個步驟中複製 EAI 橋接器端點。 至於 EDI 接收橋接器端點，請在 BizTalk 服務入口網站中，移至 [協議] > [接收設定] > [傳輸] > [端點]。
   
   ![][12]  
4. 在下一個視窗中，於 Contoso 底下按一下 [傳送內部發票]  按鈕。 在 [檔案開啟] 對話方塊中，開啟 INHOUSEINVOICE.txt 檔案。 檢查檔案的內容，然後按一下 [確定]  以傳送發票。
   
   ![][13]  
5. 幾秒鐘內 Northwind 就會收到發票。 按一下 [檢視訊息]  連結以查看 Northwind 收到的發票。 請注意在 Contoso 所傳送的發票是內部結構描述的情況下，Northwind 所收到的發票是如何變成標準 EDIFACT 結構描述的。
   
   ![][14]  
6. 選取發票，然後按一下 [傳送通知] 。 在彈出的對話方塊中，請注意所收到的發票和所傳送的通知中，有相同的交換 ID。 在 [傳送通知]  對話方塊中按一下 [確定]。
   
   ![][15]  
7. 幾秒後，Contoso 就會成功收到通知。
   
   ![][16]  

## <a name="step-5-optional-send-edifact-invoice-in-batches"></a>步驟 5 (選用)：批次傳送 EDIFACT 發票
BizTalk 服務 EDI 橋接器也支援批次處理傳出訊息。 對於想要接收一批訊息 (符合特定條件) 而非個別訊息的接收夥伴，這項功能很實用。

在處理批次時最重要的一點是批次的實際釋放，也稱為釋放準則。 釋放準則可以是依據接收夥伴想要接收訊息的方式。 如果啟用批次處理，則要等到符合釋放準則後，EDI 橋接器才會傳送傳出訊息給接收夥伴。 例如，依據訊息大小的批次準則只會在批次處理 'n' 則訊息時才分派批次。 批次準則也可以是依據時間，以便在每天固定時間傳送批次。 在此方案中，我們嘗試以訊息大小為基礎的準則。

1. 在 BizTalk 服務入口網站中，按一下您稍早建立的協議。 按一下 [傳送設定] > [批次處理] > [新增批次]。
2. 對批次名稱輸入 **InvoiceBatch**、提供描述，然後按 [下一步]。
3. 指定定義必須批次處理哪些訊息的批次準則。 在此方案中，我們會批次處理所有訊息。 因此，請選取 [使用進階定義] 選項，然後輸入 **1 = 1**。 這個條件永遠會成立，因此會批次處理所有訊息。 按 [下一步] 。
   
   ![][17]  
4. 指定批次釋放準則。 從下拉式方塊中，選取 [MessageCountBased]，然後在 [計數] 中指定 **3**。 這表示每批次會傳送三個訊息至 Northwind。 按 [下一步] 。
   
   ![][18]  
5. 檢閱摘要，然後按一下 [儲存] 。 按一下 [部署]  來重新部署協議。
6. 返回 **Tutorial Client**，按一下 [傳送內部發票]，並遵循提示以傳送發票。 您會發現 Northwind 未收到任何發票，因為批次大小不符。 再重複此步驟兩次，以便將三則發票訊息傳送至 Northwind。 這就符合 3 則訊息的批次釋放準則，您現在應該會在 Northwind 看到發票。

<!--Image references-->
[1]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-1.PNG  
[2]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-2.PNG  
[3]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-3.PNG
[4]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-4.PNG  
[5]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-5.PNG  
[6]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-6.PNG  
[7]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-7.PNG  
[8]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-8.PNG
[9]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-9.PNG  
[10]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-10.PNG  
[11]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-11.PNG  
[12]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-12.PNG  
[13]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-13.PNG
[14]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-14.PNG  
[15]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-15.PNG  
[16]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-16.PNG  
[17]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-17.PNG  
[18]: ./media/biztalk-process-edifact-invoice/process-edifact-invoices-with-auzure-bts-18.PNG

