---
title: 如何連線到 Azure 資料目錄中的資料來源
description: 操作說明文章著重在說明如何連接到使用 Azure 資料目錄找到的資料來源。
services: data-catalog
author: markingmyname
ms.author: maghan
ms.assetid: 4e6b27a5-cf75-4012-b88c-333c1fe638e8
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 973077b56f1a777d917a94f9c1470e6f8c15a489
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47405049"
---
# <a name="how-to-connect-to-data-sources"></a>如何連接到資料來源
## <a name="introduction"></a>簡介
**Microsoft Azure 資料目錄** 是完全受控的雲端服務，可作為企業資料來源的註冊系統和探索系統。 換句話說，[Azure 資料目錄]  的重點在於協助人們探索、了解，以及使用資料來源，並可協助組織從現有的資料獲得更多價值。 這個背景的重點就在於使用資料，也就是使用者探索資料來源並了解其用途，然後連接到資料來源將使用其資料。

## <a name="data-source-locations"></a>資料來源位置
資料來源在註冊期間， **Azure 資料目錄** 會收到有關資料來源的中繼資料。 此中繼資料包含資料來源位置的詳細資料。 位置的詳細資料會因資料來源而異，但永遠會包含連接所需的資訊。 例如，SQL Server 資料表的位置包含伺服器名稱、資料庫名稱、結構描述名稱和資料表名稱，而 SQL Server Reporting Services 報表的位置包含伺服器名稱和報表的路徑。 其他資料來源類型的位置，則會反映結構與來源系統的功能。

## <a name="integrated-client-tools"></a>整合式用戶端工具
若要連接到資料來源，最簡單的方式就是使用 [開啟於...] 功能表 (位於 **Azure 資料目錄**入口網站)。 這個功能表會顯示一份選項清單，可供您連接到所選取的資料資產。
使用預設的並排顯示檢視時，此功能表位於每個磚上。

 ![從資料資產磚以 Excel 開啟 SQL Server 資料表](./media/data-catalog-how-to-connect/data-catalog-how-to-connect1.png)

使用清單檢視時，此功能表位於入口網站視窗頂端的搜尋列中。

 ![從搜尋列以報告管理員開啟 SQL Server Reporting Services 報表](./media/data-catalog-how-to-connect/data-catalog-how-to-connect2.png)

## <a name="supported-client-applications"></a>支援的用戶端應用程式
在 Azure 資料目錄入口網站中 對資料來源使用 [開啟於...] 功能表時，用戶端電腦上必須已安裝正確的用戶端應用程式。

| 在應用程式中開啟 | 副檔名/通訊協定 | 支援的應用程式版本 |
| --- | --- | --- |
| Excel |.odc |Excel 2010 或更新版本 |
| Excel (前 1000 個) |.odc |Excel 2010 或更新版本 |
| Power Query |.xlsx |安裝了 Power Query for Excel 增益集的 Excel 2016、Excel 2010 或 Excel 2013 |
| Power BI Desktop |.pbix |Power BI Desktop 2016 年 7 月或更新版本 |
| SQL Server Data Tools |vsweb:// |安裝了 SQL Server 工具的 Visual Studio 2013 Update 4 或更新版本 |
| 報表管理員 |http:// |請參閱 [SQL Server Reporting Services 的瀏覽器需求](https://technet.microsoft.com/library/ms156511.aspx) |

## <a name="your-data-your-tools"></a>您的資料與工具
功能表中可用的選項取決於目前選取的資料資產的類型。 當然，並非所有可能的工具都將包含於 [開啟於...] 功能表中，但仍能使用任何用戶端工具輕鬆連接至資料來源。 在 **Azure 資料目錄**入口網站中選取資料資產時，完整的位置會顯示於屬性窗格中。

 ![SQL Server 資料表的連接資訊](./media/data-catalog-how-to-connect/data-catalog-how-to-connect3.png)

連接資訊的詳細資料會因資料來源的類型而異，但是入口網站中所包含的資訊提供您所需的一切，讓您可以使用任何用戶端工具連接到資料來源。 使用者可以複製他們使用 **Azure 資料目錄**所找到之資料來源的連接詳細資料，讓他們能夠使用他們選擇的工具來處理資料。

## <a name="connecting-and-data-source-permissions"></a>連接和資料來源的權限
雖然 **Azure 資料目錄** 可讓資料來源變成可探索，資料本身的存取權仍受到資料來源擁有者或系統管理員的控制。 在 **Azure 資料目錄** 中探索資料來源，並不會提供使用者任何權限以存取資料來源本身。

若要讓使用者更輕鬆地探索資料來源，但不要有存取其資料的權限，使用者可以在加註資料來源時於 [要求存取] 屬性中提供資訊。 這裡所提供的資訊會和入口網站中的資料來源位置資訊一起呈現，包括連結的程序或取得資料來源存取權的聯絡點。

 ![具有所提供之要求存取指示的連接資訊](./media/data-catalog-how-to-connect/data-catalog-how-to-connect4.png)

## <a name="summary"></a>總結
使用 **Azure 資料目錄**註冊資料來源，即可藉由將資料來源的結構性和描述性中繼資料複製到目錄服務內，使其變成可探索的資料。 資料來源一經註冊且變成可探索之後，使用者就能從 **Azure 資料目錄**入口網站的 [開啟於...] 功能表，或使用他們選擇的資料工具，連接到資料來源。

## <a name="see-also"></a>另請參閱
* [開始使用 Azure 資料目錄](data-catalog-get-started.md) 教學課程，取得如何連線至資料來源的逐步詳細資料。
