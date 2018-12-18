---
title: 編碼或解碼一般檔案 - Azure Logic Apps | Microsoft Docs
description: 使用 Azure Logic Apps 與 Enterprise Integration Pack 來編碼或解碼適用於企業整合的一般檔案
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: 82152dab-c7ad-43df-b721-596559703be8
ms.date: 07/08/2016
ms.openlocfilehash: d0ef61b94d7bd604b6c0062341224510f3048c57
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43123923"
---
# <a name="encode-or-decode-flat-files-with-azure-logic-apps-and-enterprise-integration-pack"></a>使用 Azure Logic Apps 與 Enterprise Integration Pack 來編碼或解碼一般檔案

在企業對企業 (B2B) 案例中，您可能會希望先將 XML 內容編碼，然後再傳送給企業夥伴。 在邏輯應用程式中，您可以使用一般檔案編碼連接器來執行此動作。 您所建立的邏輯應用程式可從各種來源取得其 XML 內容，包括 HTTP 要求觸發程序、另一個應用程式，或甚至是這其中任一種 [連接器](../connectors/apis-list.md)。 如需邏輯應用程式的詳細資訊，請參閱[邏輯應用程式文件](logic-apps-overview.md "深入了解 Logic Apps")。  

## <a name="create-the-flat-file-encoding-connector"></a>建立一般檔案編碼連接器
請遵循下列步驟，將一般檔案編碼連接器新增到邏輯應用程式。

1. 建立邏輯應用程式，並[將它連結到您的整合帳戶](logic-apps-enterprise-integration-accounts.md "了解如何將整合帳戶連結到邏輯應用程式")。 此帳戶包含您將用來編碼 XML 資料的結構描述。  
1. 將 [要求 - 收到 HTTP 要求時]  觸發程序新增到您的邏輯應用程式。  
   ![要選取之觸發程序的螢幕擷取畫面](./media/logic-apps-enterprise-integration-b2b/flatfile-1.png)    
1. 新增一般檔案編碼動作，如下所示︰
   
    a. 選取**加號**。
   
    b. 選取 [新增動作] 連結 (會在選取加號之後出現)。
   
    c. 在搜尋方塊中輸入「一般」，篩選所有動作以取得您想要使用的動作。
   
    d. 從清單中選取 [一般檔案編碼] 選項。   
   ![一般檔案編碼選項的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-2.png)   
1. 在 [一般檔案編碼] 對話方塊上選取 [內容] 文字方塊。  
   ![內容文字方塊的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-3.png)  
1. 選取內文標記做為您想要編碼的內容。 內文標記將會填入內容欄位。     
   ![內文標記的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-4.png)  
1. 選取 [結構描述名稱]  清單方塊，然後選擇您想要用來將輸入內容編碼的結構描述。    
   ![結構描述名稱清單方塊的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-5.png)  
1. 儲存您的工作。   
   ![儲存圖示的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-6.png)  

此時，您已完成設定一般檔案編碼連接器。 在真實世界應用程式中，您可能想要在企業營運應用程式 (例如 Salesforce) 中儲存已編碼的資料。 或者，您也可以將該編碼資料傳送給交易夥伴。 您可以使用所提供的任一個其他連接器，輕鬆地新增動作來將編碼動作的輸出傳送到 Salesforce，或傳送給交易夥伴。

您現在可以測試連接器，方法是向 HTTP 端點提出要求，並在要求內文中包含 XML 內容。  

## <a name="create-the-flat-file-decoding-connector"></a>建立一般檔案解碼連接器

> [!NOTE]
> 為了完成這些步驟，您必須已將結構描述檔案上傳到您的整合帳戶。

1. 將 [要求 - 收到 HTTP 要求時]  觸發程序新增到您的邏輯應用程式。  
   ![要選取之觸發程序的螢幕擷取畫面](./media/logic-apps-enterprise-integration-b2b/flatfile-1.png)    
1. 新增一般檔案解碼動作，如下所示︰
   
    a. 選取**加號**。
   
    b. 選取 [新增動作] 連結 (會在選取加號之後出現)。
   
    c. 在搜尋方塊中輸入「一般」，篩選所有動作以取得您想要使用的動作。
   
    d. 從清單中選取 [一般檔案解碼] 選項。   
   ![一般檔案解碼選項的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-2.png)   
1. 選取 [內容]  控制項。 這會從先前步驟中產生一份內容清單，讓您可用來做為要解碼的內容。 請注意，來自傳入 HTTP 要求的「內文」  可用來做為要解碼的內容。 您也可以將要解碼的內容直接輸入於 [內容]  控制項中。     
1. 選取 [內文]  標記。 請注意，內文標記目前位於 [內容]  控制項中。
1. 選取您想要用來將內容解碼的結構描述名稱。 下列螢幕擷取畫面顯示選取的結構描述名稱是「OrderFile」  。 此結構描述名稱先前已上傳到整合帳戶。
   
   ![一般檔案解碼對話方塊的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-decode-1.png)    
1. 儲存您的工作。  
   ![儲存圖示的螢幕擷取畫面](media/logic-apps-enterprise-integration-flatfile/flatfile-6.png)    

此時，您已完成設定一般檔案解碼連接器。 在真實世界應用程式中，您可能想要在企業營運應用程式 (例如 Salesforce) 中儲存已解碼的資料。 您可以輕鬆新增動作，來將解碼動作的輸出傳送到 Salesforce。

您現在可以測試連接器，方法是向 HTTP 端點提出要求，並在要求內文中包含您想要解碼的 XML 內容。  

## <a name="next-steps"></a>後續步驟
* [深入了解企業整合套件](logic-apps-enterprise-integration-overview.md "了解企業整合套件")。  

