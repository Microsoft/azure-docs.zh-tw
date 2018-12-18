---
title: Azure 資料目錄術語
description: 本文提供 Azure 資料目錄文件所使用之概念與術語的簡介。
services: data-catalog
author: steelanddata
ms.author: maroche
ms.assetid: 6fec74d9-4a3c-4b4b-88ba-cad5ad143331
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 70772ae07c4a8a6e87b4fa6f119acf2d51a5c23e
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43053111"
---
# <a name="azure-data-catalog-terminology"></a>Azure 資料目錄術語
## <a name="catalog"></a>目錄
Azure 資料目錄是以雲端為基礎的中繼資料儲存機制，其中可以註冊資料來源和資料資產。 此目錄做為中央儲存位置，儲存從資料來源擷取的結構化中繼資料和使用者加入的描述性中繼資料。

## <a name="data-source"></a>資料來源
資料來源是管理資料資產的系統或容器。 範例包括 SQL Server 資料庫、Oracle 資料庫、SQL Server Analysis Services 資料庫 (表格式或多維度) 和 SQL Server Reporting Services 伺服器。

## <a name="data-asset"></a>資料資產
資料資產是資料來源內可向目錄註冊的物件。 範例包括 SQL Server 資料表和檢視、Oracle 資料表和檢視、SQL Server Analysis Services 量值、維度和 KPI，以及 SQL Server Reporting Services 報表。

## <a name="data-asset-location"></a>資料資產位置
目錄儲存資料來源或資料資產的位置，可供用戶端應用程式連接到來源。 位置的格式和詳細資料隨資料來源類型而不同。 比方說，SQL Server 資料表可以透過其四段名稱來識別 – 伺服器名稱、資料庫名稱、結構描述名稱、物件名稱 – 而 SQL Server Reporting Services 報表可以透過其 URL 來識別。

## <a name="structural-metadata"></a>結構化中繼資料
結構化中繼資料是從資料來源擷取的中繼資料，用來描述資料資產的結構。 這包括資產位置、其物件名稱和類型，以及關於類型的其他特性。 例如，資料表和檢視的結構化中繼資料包含物件資料行的名稱和資料類型。

## <a name="descriptive-metadata"></a>描述性中繼資料
描述性中繼資料是描述資料資產的用途或意圖的中繼資料。 描述性中繼資料通常由目錄使用者利用 Azure 資料目錄入口網站來加入，但也可以在註冊期間從資料來源擷取。 例如，Azure 資料目錄註冊工具將會從 SQL Server Analysis Services 和 SQL Server Reporting Services 中的 Description 屬性擷取描述，以及從 SQL Server 資料庫中的 [ms_description 擴充屬性](https://technet.microsoft.com/library/ms190243.aspx)擷取描述 (如果這些屬性已填入值)。

## <a name="request-access"></a>要求存取
資料資產的描述性中繼資料可以包含如何要求存取資料資產或資料來源的相關資訊。 這項資訊會與資料資產位置一併顯示，且可包含一個或多個下列選項：

* 負責授與資料來源存取權之使用者或小組的電子郵件地址。
* 記載程序的 URL，使用者必須遵循才能取得資料來源的存取權。
* 身分識別和存取管理工具 (例如 Microsoft Identity Manager) 的 URL，可以用來取得資料來源的存取權。
* 任意文字項目，用來描述使用者如何取得資料來源的存取權。

## <a name="preview"></a>預覽
Azure 資料目錄中的預覽是最多 20 筆記錄的快照集，可以在註冊期間從資料來源中擷取，並與資料資產中繼資料一起儲存在目錄中。 預覽可協助使用者在探索資料資產時充分了解其功能和用途。 換句話說，查看範例資料比只看到資料行名稱和資料類型更有價值。
僅支援預覽資料表和檢視，而且必須由使用者在註冊期間明確選取。

## <a name="data-profile"></a>資料設定檔
Azure 資料目錄中的資料設定檔是資料表層級的快照集和及資料行層級的中繼資料，已註冊的資料資產在註冊期間可從資料來源中擷取，並與資料資產中繼資料一起儲存在目錄中。 資料設定檔可協助使用者在探索資料資產時充分了解其功能和用途。 和預覽相同，資料設定檔必須是使用者在註冊期間明確選取的資料。

> [!NOTE]
> 為大型資料表和檢視擷取資料設定檔，作業成本會很高，而且可能會大幅增加註冊資料來源所需的時間。
>
>

## <a name="user-perspective"></a>使用者觀點
在 Azure 資料目錄中，任何使用者都可以為已註冊的資料資產提供描述性中繼資料。 每個使用者對資料及其用途各有不同的觀點。 例如，負責伺服器的系統管理員可能提供其服務等級協定 (SLA) 或備份時段的詳細資料。資料負責人可能針對資料所支援的商務程序提供文件連結。分析師可能以最適合的詞彙提供描述給其他分析師，而且對於需要探索及了解資料的使用者來說，可能非常有價值。

每一種觀點基本上都很有價值，透過 Azure 資料目錄，每個使用者可以提供對他們有意義的資訊，而所有使用者都可以使用該資訊來了解資料及其用途。

## <a name="expert"></a>專家
專家是指公認對資料資產有其獨特「專家」觀點的使用者。 任何使用者都可以將自己或其他使用者加入成為資產的專家。 成為專家並不表示在 Azure 資料目錄具有任何額外的權限。它可讓使用者在檢閱資產的描述性中繼資料時，輕鬆找到最可能有用的觀點。

## <a name="owner"></a>擁有者
擁有者是具有額外權限來管理 Azure 資料目錄中的資料資產的使用者。 使用者可以取得已註冊的資料資產的擁有權，而擁有者可以加入其他使用者成為共同擁有者。 如需詳細資訊，請參閱[如何管理資料資產](data-catalog-how-to-manage.md)  

> [!NOTE]
> 只有標準版 Azure 資料目錄中支援擁有權和管理。
>
>

## <a name="registration"></a>註冊
註冊是指從資料來源擷取資料資產中繼資料並複製到 Azure 資料目錄服務的動作。 然後就可以加註和探索已註冊的資料資產。

## <a name="see-also"></a>另請參閱
* [什麼是 Azure 資料目錄？](data-catalog-what-is-data-catalog.md) 〉 - 這篇文章提供 Azure 資料目錄服務的概觀、所提供的價值和所支援的案例。
* [開始使用 Azure 資料目錄](data-catalog-get-started.md) 〉 - 這篇文章提供端對端教學課程，示範如何使用 Azure 資料目錄來探索資料來源。  
