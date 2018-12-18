---
title: 轉換 XML 格式 - Azure Logic Apps | Microsoft Docs
description: 在採用 Enterprise Integration Pack 的 Azure Logic Apps 中建立轉換或對應，以轉換 XML 格式
services: logic-apps
ms.service: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, LADocs
ms.topic: article
ms.assetid: add01429-21bc-4bab-8b23-bc76ba7d0bde
ms.date: 07/08/2016
ms.openlocfilehash: 9dd471f70407191734b4c5a3aa84d5365a7beab8
ms.sourcegitcommit: 2ad510772e28f5eddd15ba265746c368356244ae
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/28/2018
ms.locfileid: "43125290"
---
# <a name="create-maps-that-transform-xml-between-formats-in-azure-logic-apps-with-enterprise-integration-pack"></a>在採用 Enterprise Integration Pack 的 Azure Logic Apps 中建立對應，以轉換 XML 格式

企業整合轉換連接器會將資料從某種格式轉換成其他格式。 例如，傳入訊息中目前包含的日期是 YearMonthDay 格式。 您可以使用轉換，將日期重新格式化為 MonthDayYear 格式。

## <a name="what-does-a-transform-do"></a>轉換的作用為何？
轉換 (亦稱為對應) 由來源 XML 結構描述 (輸入) 和目標 XML 結構描述 (輸出) 所組成。 您可以利用不同的內建功能來操控或控制資料，包括字串操作、條件式協議、算術運算式、日期時間格式器，甚至迴圈建構。

## <a name="how-to-create-a-transform"></a>如何建立轉換？
您可以使用 Visual Studio [企業整合 SDK](https://aka.ms/vsmapsandschemas)來建立轉換/對應。 當您完成建立及測試轉換之後，可將轉換上傳到整合帳戶。 

## <a name="how-to-use-a-transform"></a>如何使用轉換
當您將轉換/對應上傳到整合帳戶之後，您可以使用它來建立邏輯應用程式。 每當觸發邏輯應用程式 (而且還有需要轉換的輸入內容) 時，邏輯應用程式接著便會執行您的轉換。

**以下是使用轉換的步驟**：

### <a name="prerequisites"></a>必要條件

* 建立整合帳戶，並加入對應  

既然您已完成必要元件，就可以建立邏輯應用程式：  

1. 建立邏輯應用程式並[將它連結到包含對應的整合帳戶](../logic-apps/logic-apps-enterprise-integration-accounts.md "了解如何將整合帳戶連結到邏輯應用程式")。
2. 將**要求**觸發程序新增至邏輯應用程式  
   ![](./media/logic-apps-enterprise-integration-transforms/transform-1.png)    
3. 先選取 [新增動作] 來新增 [轉換 XML] 動作   
   ![](./media/logic-apps-enterprise-integration-transforms/transform-2.png)   
4. 在搜尋方塊中輸入「轉換」，篩選所有動作以取得您想要使用的動作  
   ![](./media/logic-apps-enterprise-integration-transforms/transform-3.png)  
5. 選取 [轉換 XML] 動作   
6. 新增您將轉換的 XML **內容**。 您可以使用在 HTTP 要求中收到的任何 XML 資料做為 **內容**。 在此範例中，選取觸發邏輯應用程式的 HTTP 要求本文。

   > [!NOTE]
   > 請確定 **轉換 XML** 的內容是 XML。 如果內容不是 XML 或是 base64 編碼，您必須指定可處理內容的運算式。 例如，您可以使用[函式](logic-apps-workflow-definition-language.md#functions)，如使用 ```@base64ToBinary``` 將內容解碼，或使用 ```@xml``` 將內容處理為 XML。
 

7. 選取您想要用來執行轉換的 **對應** 名稱。 對應必須已經位於您的整合帳戶中。 在先前步驟中，您已經為邏輯應用程式提供權限來存取包含對應的整合帳戶。      
   ![](./media/logic-apps-enterprise-integration-transforms/transform-4.png) 
8. 儲存您的工作   
    ![](./media/logic-apps-enterprise-integration-transforms/transform-5.png) 

此時，您已完成設定對應。 在真實世界應用程式中，您可能想要在 LOB 應用程式 (例如 SalesForce) 中儲存已轉換的資料。 您可以輕鬆新增動作，來將轉換的輸出傳送到 Salesforce。 

您現在可以藉由向 HTTP 端點提出要求來測試轉換。  


## <a name="features-and-use-cases"></a>功能和使用案例
* 在對應中建立轉換並不難，例如，只要在不同文件之間複製名稱和位址，即可完成。 或者，您可以使用內建的對應作業，建立更複雜的轉換。  
* 目前有多個對應作業或函數可供使用，包括字串、日期時間函數等等。  
* 您可以在結構描述間執行直接的資料複製。 在 SDK 內含的對應程式中，只要繪製一條線連接來源結構描述中的元素與其目的地結構描述中的對等項目，即可完成此動作。  
* 建立對應時，您可以檢視圖形化對應，其中會顯示您所建立的所有關聯性和連結。
* 使用 [測試對應] 功能，以新增範例 XML 訊息。 只要按一下滑鼠，您即可測試已建立的對應，並檢視產生的輸出。  
* 上傳現有的對應  
* 包括對 XML 格式的支援。

## <a name="advanced-features"></a>進階功能

### <a name="reference-assembly-or-custom-code-from-maps"></a>來自對應的參考組件或自訂程式碼 
轉換動作還支援以外部組件的參考進行對應或轉換。 此功能可讓您直接從 XSLT 對應呼叫自訂 .NET 程式碼。 以下是在對應中使用的組件的必要條件。

* 對應與從對應參考的組件都必須[上傳到整合帳戶](./logic-apps-enterprise-integration-maps.md)。 

  > [!NOTE]
  > 對應和組件都必須依照特定順序上傳。 您必須先上傳組件，接著再上傳參考組件的對應。

* 對應還必須包含以下屬性，以及含有組件程式碼呼叫的 CDATA 區段：

    * **name** 是自訂組件名稱。
    * **namespace** 是組件中包含自訂程式碼的命名空間。

  以下範例展示的對應參考名為 "XslUtilitiesLib" 的組件，並且從組件呼叫 `circumreference` 方法。

  ````xml
  <?xml version="1.0" encoding="UTF-8"?>
  <xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:msxsl="urn:schemas-microsoft-com:xslt" xmlns:user="urn:my-scripts">
  <msxsl:script language="C#" implements-prefix="user">
    <msxsl:assembly name="XsltHelperLib"/>
    <msxsl:using namespace="XsltHelpers"/>
    <![CDATA[public double circumference(int radius){ XsltHelper helper = new XsltHelper(); return helper.circumference(radius); }]]>
  </msxsl:script>
  <xsl:template match="data">
     <circles>
        <xsl:for-each select="circle">
            <circle>
                <xsl:copy-of select="node()"/>
                    <circumference>
                        <xsl:value-of select="user:circumference(radius)"/>
                    </circumference>
            </circle>
        </xsl:for-each>
     </circles>
    </xsl:template>
    </xsl:stylesheet>
  ````


### <a name="byte-order-mark"></a>位元組順序標記
根據預設，轉換將會以位元組順序標記 (BOM) 開頭。 唯有在使用 [程式碼檢視] 編輯器時，您才可以取用這項功能。 若要停用這項功能，請為 `transformOptions` 屬性指定 `disableByteOrderMark`：

````json
"Transform_XML": {
    "inputs": {
        "content": "@{triggerBody()}",
        "integrationAccount": {
            "map": {
                "name": "TestMap"
            }
        },
        "transformOptions": "disableByteOrderMark"
    },
    "runAfter": {},
    "type": "Xslt"
}
````





## <a name="learn-more"></a>深入了解
* [深入了解企業整合套件](../logic-apps/logic-apps-enterprise-integration-overview.md "了解企業整合套件")  
* [深入了解對應](../logic-apps/logic-apps-enterprise-integration-maps.md "了解企業整合對應")  

