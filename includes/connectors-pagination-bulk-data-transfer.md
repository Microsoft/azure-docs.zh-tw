---
title: 包含檔案
description: 包含檔案
services: logic-apps
author: ecfan
ms.service: logic-apps
ms.topic: include
ms.date: 05/09/2018
ms.author: estfan
ms.custom: include file
ms.openlocfilehash: 524bc1d3a19ad8ecfc8d0d210e735d6a9ae0058b
ms.sourcegitcommit: 59fffec8043c3da2fcf31ca5036a55bbd62e519c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/04/2018
ms.locfileid: "34678180"
---
### <a name="set-up-pagination"></a>設定分頁

針對某些連接器和其擷取多個項目的動作，您的結果可能會超過連接器的預設頁面大小。 在此情況下，動作只會傳回結果的第一頁。 例如，[SQL Server - 取得資料列] 動作的預設頁面大小為 2048 筆結果，但可能會根據其他設定而有所不同。 若要確保您可取得所有記錄，請開啟該動作的 [分頁] 設定。 此設定會使邏輯應用程式向連接器要求其餘的記錄，但在動作完成時，會以單一訊息傳回所有結果。 

在以下這些連接器中，您可以開啟特定動作的分頁： 

* <a href="https://docs.microsoft.com/connectors/db2/" target="_blank">DB2</a>
* <a href="https://docs.microsoft.com/connectors/dynamicscrmonline/" target="_blank">Dynamics 365 CRM Online</a>
* <a href="https://docs.microsoft.com/connectors/excel/" target="_blank">Excel</a>
* <a href="https://docs.microsoft.com/connectors/oracle/" target="_blank">Oracle Database</a>
* <a href="https://docs.microsoft.com/connectors/sharepointonline/" target="_blank">SharePoint</a>
* <a href="https://docs.microsoft.com/connectors/sql/" target="_blank">SQL Server</a> 

以下是 [取得資料列] 動作的範例：

1. 若要尋找動作是否支援分頁，請開啟動作的 [設定]。 

   ![在動作上開啟 [設定]](./media/connectors-pagination-bulk-data-transfer/sql-action-settings.png)

2. 如果動作支援分頁，請將 [分頁] 設定從 [關閉] 變更為 [開啟]。 若要確定此動作會傳回一組最小數目的結果，請指定 [限制] 值。

   ![指定動作傳回最小數目的結果](./media/connectors-pagination-bulk-data-transfer/sql-action-settings-pagination.png)

3. 準備就緒時，請選擇 [完成]。