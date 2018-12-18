---
title: 從 Azure 入口網站取得連接字串
description: 從 Azure 入口網站取得連接字串
keywords: sql connection,connection string,sql 連線,連接字串
services: sql-database
author: dalechen
manager: craigg
ms.service: sql-database
ms.custom: develop apps
ms.topic: include
ms.date: 07/13/2018
ms.author: ninarn
ms.openlocfilehash: dab7623c86bea4e562313e618f238b9b33c0fdc5
ms.sourcegitcommit: 7827d434ae8e904af9b573fb7c4f4799137f9d9b
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/18/2018
ms.locfileid: "39117285"
---
### <a name="obtain-the-connection-string-from-the-azure-portal"></a>從 Azure 入口網站取得連接字串
使用 [Azure 入口網站](https://portal.azure.com/)來取得用戶端程式與 Azure SQL Database 進行互動所需的連接字串。

1. 選取 [所有服務] > [SQL 資料庫]。

2. 在 [SQL 資料庫] 刀鋒視窗左上角附近的篩選文字方塊中輸入您的資料庫名稱。

3. 選取資料庫的資料列。

4. 在刀鋒視窗顯示您的資料庫之後，為了閱讀方便，選取 [最小化] 按鈕來摺疊用於瀏覽和資料庫篩選的刀鋒視窗。

5. 在您資料庫的刀鋒視窗上，選取 [顯示資料庫連接字串]。

6. 複製適當的連接字串。 也就是說，如果您想要使用 ADO.NET 連線庫，請從 [ADO.NET] 索引標籤複製適當的字串。

    ![複製資料庫的 ADO 連接字串][20-CopyAdoConnectionString]

7. 視需要編輯該連接字串。 也就是說，將您的密碼插入連接字串，或是在使用者名稱或伺服器名稱太長的情況下，從使用者名稱移除 "@&lt;servername&gt;"。

8. 以其中一種格式，將連接字串資訊貼入您的用戶端程式碼中。

如需詳細資訊，請參閱[連接字串與設定檔](http://msdn.microsoft.com/library/ms254494.aspx)。

<!-- Image references. -->



[20-CopyAdoConnectionString]: ./media/sql-database-include-connection-string-20-portalshots/connqry-connstr-b.png


<!--
These three includes/ files are a sequenced set, but you can pick and choose:

includes/sql-database-include-connection-string-20-portalshots.md
includes/sql-database-include-connection-string-30-compare.md
includes/sql-database-include-connection-string-40-config.md
-->
