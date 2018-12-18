---
title: Azure Functions 的外部資料表繫結 (實驗性)
description: 使用 Azure Functions 中的外部資料表繫結
services: functions
documentationcenter: ''
author: alexkarcher-msft
manager: cfowler
editor: ''
ms.assetid: ''
ms.service: functions
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: multiple
ms.topic: article
ms.date: 04/12/2017
ms.author: alkarche
ms.openlocfilehash: 8a4358fa67e45d0b7a2df1519d649099b5ef5850
ms.sourcegitcommit: 1d423a8954731b0f318240f2fa0262934ff04bd9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/05/2018
ms.locfileid: "27613276"
---
# <a name="external-table-binding-for-azure-functions-experimental"></a>Azure Functions 的外部資料表繫結 (實驗性)

本文說明如何在 Azure Functions 中使用 SaaS 提供者上的表格式資料，例如 Sharepoint 和 Dynamics。 Azure Functions 支援外部資料表的輸入和輸出繫結。

> [!IMPORTANT]
> 外部資料表繫結是實驗性質，且可能永遠無法觸達正式上市 (GA) 狀態。 它只包含在 Azure Functions 1.x 中，且並未計劃將它新增至 Azure Functions 2.x。 針對要求存取 SaaS 提供者中資料的情況，請考量使用[呼叫函式的邏輯應用程式](functions-twitter-email.md)。

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="api-connections"></a>API 連線

資料表繫結會運用外部 API 連線以向第三方 SaaS 提供者進行驗證。 

指派繫結時，您可以建立新的 API 連線，或是使用相同資源群組內的現有 API 連線。

### <a name="available-api-connections-tables"></a>可用的 API 連線 (資料表)

|連接器|觸發程序|輸入|輸出|
|:-----|:---:|:---:|:---:|
|[DB2](https://docs.microsoft.com/azure/connectors/connectors-create-api-db2)||x|x
|[Dynamics 365 for Operations (英文)](https://ax.help.dynamics.com/wiki/install-and-configure-dynamics-365-for-operations-warehousing/)||x|x
|[Dynamics 365](https://docs.microsoft.com/azure/connectors/connectors-create-api-crmonline)||x|x
|[Dynamics NAV (英文)](https://msdn.microsoft.com/library/gg481835.aspx)||x|x
|[Google 試算表](https://docs.microsoft.com/azure/connectors/connectors-create-api-googledrive)||x|x
|[Informix](https://docs.microsoft.com/azure/connectors/connectors-create-api-informix)||x|x
|[Dynamics 365 for Financials](https://docs.microsoft.com/azure/connectors/connectors-create-api-crmonline)||x|x
|[MySQL](https://docs.microsoft.com/azure/store-php-create-mysql-database)||x|x
|[Oracle Database](https://docs.microsoft.com/azure/connectors/connectors-create-api-oracledatabase)||x|x
|[Common Data Service (英文)](https://docs.microsoft.com/common-data-service/entity-reference/introduction)||x|x
|[Salesforce](https://docs.microsoft.com/azure/connectors/connectors-create-api-salesforce)||x|x
|[SharePoint](https://docs.microsoft.com/azure/connectors/connectors-create-api-sharepointonline)||x|x
|[SQL Server](https://docs.microsoft.com/azure/connectors/connectors-create-api-sqlazure)||x|x
|[Teradata](http://www.teradata.com/products-and-services/azure/products/)||x|x
|UserVoice||x|x
|Zendesk||x|x

> [!NOTE]
> 外部資料表連線也可以用在 [Azure Logic Apps](https://docs.microsoft.com/azure/connectors/apis-list) 中。

## <a name="creating-an-api-connection-step-by-step"></a>建立 API 連線：逐步解說

1. 在您函式應用程式的 Azure 入口網站頁面上，選取加號 (**+**) 以建立函式。

1. 在 [案例] 方塊中，選取 [實驗性]。

1. 選取 [外部資料表]。

1. 選取語言。

2. 在 [外部資料表連線] 下，選取現有的連線或選取 [新增]。

1. 如需新的連線，請設定設定，然後選取 [授權]。

1. 選取 [建立] 以建立函式。

1. 選取 [整合] > [外部資料表]。

1. 設定連線以使用目標資料表。 這些設定會依 SaaS 提供者不同而有所差異。 下一節中包含範例。

## <a name="example"></a>範例

此範例會連線至名為 "Contact"，並具有 ID, LastName 及 FirstName 資料行的資料表。 程式碼會在資料表中列出 Contact 實體，並記錄名字和姓氏。

以下是 *function.json* 檔案：

```json
{
  "bindings": [
    {
      "type": "manualTrigger",
      "direction": "in",
      "name": "input"
    },
    {
      "type": "apiHubTable",
      "direction": "in",
      "name": "table",
      "connection": "ConnectionAppSettingsKey",
      "dataSetName": "default",
      "tableName": "Contact",
      "entityId": "",
    }
  ],
  "disabled": false
}
```

以下是 C# 指令碼程式碼：

```cs
#r "Microsoft.Azure.ApiHub.Sdk"
#r "Newtonsoft.Json"

using System;
using Microsoft.Azure.ApiHub;

//Variable name must match column type
//Variable type is dynamically bound to the incoming data
public class Contact
{
    public string Id { get; set; }
    public string LastName { get; set; }
    public string FirstName { get; set; }
}

public static async Task Run(string input, ITable<Contact> table, TraceWriter log)
{
    //Iterate over every value in the source table
    ContinuationToken continuationToken = null;
    do
    {   
        //retrieve table values
        var contactsSegment = await table.ListEntitiesAsync(
            continuationToken: continuationToken);

        foreach (var contact in contactsSegment.Items)
        {   
            log.Info(string.Format("{0} {1}", contact.FirstName, contact.LastName));
        }

        continuationToken = contactsSegment.ContinuationToken;
    }
    while (continuationToken != null);
}
```

### <a name="sql-server-data-source"></a>SQL Server 資料來源

若要在 SQL Server 中建立資料表來與此範例搭配使用，請使用以下指令碼。 `dataSetName` 是預設值。

```sql
CREATE TABLE Contact
(
    Id int NOT NULL,
    LastName varchar(20) NOT NULL,
    FirstName varchar(20) NOT NULL,
    CONSTRAINT PK_Contact_Id PRIMARY KEY (Id)
)
GO
INSERT INTO Contact(Id, LastName, FirstName)
     VALUES (1, 'Bitt', 'Prad') 
GO
INSERT INTO Contact(Id, LastName, FirstName)
     VALUES (2, 'Glooney', 'Ceorge') 
GO
```

### <a name="google-sheets-data-source"></a>Google 試算表資料來源

若要建立資料表以在 Google 文件中與這個範例搭配使用，請以名為 `Contact` 的工作表建立試算表。 連接器不能使用試算表顯示名稱。 內部名稱 (以粗體顯示) 必須做為 dataSetName 使用，例如：`docs.google.com/spreadsheets/d/`**`1UIz545JF_cx6Chm_5HpSPVOenU4DZh4bDxbFgJOSMz0`** 將資料行名稱 `Id`, `LastName`, `FirstName` 新增至第一個資料列，然後將資料填入後續資料列。

### <a name="salesforce"></a>Salesforce

若要將此範例與 Salesforce 搭配使用，`dataSetName` 是預設值。

## <a name="configuration"></a>組態

下表說明您在 function.json 檔案中設定的繫結設定屬性。

|function.json 屬性 | 說明|
|---------|----------------------|
|**type** | 必須設為 `apiHubTable`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。|
|**direction** | 必須設為 `in`。 當您在 Azure 入口網站中建立觸發程序時，會自動設定此屬性。 |
|**name** | 代表函式程式碼中事件項目的變數名稱。 | 
|**連接**| 會識別儲存 API 連接字串的應用程式設定。 應用程式設定會在您於整合 UI 中新增 API 連線時自動建立。|
|**dataSetName**|包含要讀取之資料表的資料集名稱。|
|**tableName**|資料表的名稱|
|**entityId**|針對資料表繫結必須為空白。

表格式連接器會提供資料集，每個資料集皆包含資料表。 預設資料集的名稱為 "default"。 以下列出各種 SaaS 提供者的資料集和資料表標題：

|連接器|Dataset|資料表|
|:-----|:---|:---| 
|**SharePoint**|網站|SharePoint 清單
|**SQL**|資料庫|資料表 
|**Google 試算表**|試算表|工作表 
|**Excel**|Excel 檔案|工作表 

## <a name="next-steps"></a>後續步驟

> [!div class="nextstepaction"]
> [深入了解 Azure Functions 觸發程序和繫結](functions-triggers-bindings.md)
