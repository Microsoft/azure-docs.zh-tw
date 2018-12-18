---
title: 設計您的第一個 Azure SQL Database - C# | Microsoft Docs
description: 了解如何設計您的第一個 Azure SQL Database，並透過 C# 程式使用 ADO.NET 來與它連接。
services: sql-database
author: MightyPen
manager: craigg-msft
ms.reviewer: CarlRabeler
ms.service: sql-database
ms.custom: develop databases, mvc, devcenter
ms.topic: tutorial
ms.author: genemi
ms.date: 06/07/2018
ms.openlocfilehash: 9fdc23eb2e607d1264e1bd2bc9dc85ef8d9be642
ms.sourcegitcommit: 8b694bf803806b2f237494cd3b69f13751de9926
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/20/2018
ms.locfileid: "46498148"
---
# <a name="design-an-azure-sql-database-and-connect-with-cx23-and-adonet"></a>設計 Azure SQL Database 並連接 C&#x23; 和 ADO.NET

Azure SQL Database 是 Microsoft Cloud (Azure) 中的關聯式資料庫即服務 (DBaaS)。 在本教學課程裡，您將了解如何搭配使用 Visual Studio 與 Azure 入口網站和 ADO.NET 執行下列操作： 

> [!div class="checklist"]
> * 在 Azure 入口網站中建立資料庫
> * 在 Azure 入口網站中設定伺服器層級的防火牆規則
> * 使用 ADO.NET 和 Visual Studio 連線到資料庫
> * 使用 ADO.NET 建立資料表
> * 使用 ADO.NET 插入、更新和刪除資料 
> * 使用 ADO.NET 查詢資料

如果您沒有 Azure 訂用帳戶，請在開始之前先[建立免費帳戶](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>必要條件

安裝 [Visual Studio Community 2017、Visual Studio Professional 2017 或 Visual Studio Enterprise 2017](https://www.visualstudio.com/downloads/)。

<!-- The following included .md, sql-database-tutorial-portal-create-firewall-connection-1.md, is long.
And it starts with a ## H2.
-->

[!INCLUDE [sql-database-tutorial-portal-create-firewall-connection-1](../../includes/sql-database-tutorial-portal-create-firewall-connection-1.md)]


<!-- The following included .md, sql-database-csharp-adonet-create-query-2.md, is long.
And it starts with a ## H2.
-->

[!INCLUDE [sql-database-csharp-adonet-create-query-2](../../includes/sql-database-csharp-adonet-create-query-2.md)]


## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解基本的資料庫工作，例如建立資料庫和資料表、載入和查詢資料，以及將資料庫還原至先前的時間點。 您已了解如何︰
> [!div class="checklist"]
> * 建立資料庫
> * 設定防火牆規則
> * 使用 [Visual Studio 和 C#](sql-database-connect-query-dotnet-visual-studio.md) 連線到資料庫
> * 建立資料表
> * 插入、更新和刪除資料
> * 查詢資料

請前進到下一個教學課程，以了解如何移轉資料。

> [!div class="nextstepaction"]
> [將 SQL Server Database 移轉至 Azure SQL Database](sql-database-migrate-your-sql-server-database.md)

