---
title: 將 NoSQL 資料庫的文件資料模型化 | Microsoft Docs
description: 了解如何將 NoSQL 資料庫的資料模型化
keywords: 模型化資料
services: cosmos-db
author: aliuy
manager: kfile
ms.service: cosmos-db
ms.devlang: na
ms.topic: conceptual
ms.date: 05/29/2016
ms.author: andrl
ms.openlocfilehash: c577c9734490e3aacc148153f550162371ae482e
ms.sourcegitcommit: 387d7edd387a478db181ca639db8a8e43d0d75f7
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/10/2018
ms.locfileid: "40037995"
---
# <a name="modeling-document-data-for-nosql-databases"></a>將 NoSQL 資料庫的文件資料模型化
雖然無結構描述的資料庫 (像是 Azure Cosmos DB)，讓您極容易運用資料模型變更，但是您仍然應該花一些時間來思考資料。 

要如何儲存資料？ 您的應用程式要如何擷取及查詢資料？ 您的應用程式是大量讀取或大量寫入？ 

閱讀本文後，您將能夠回答下列問題：

* 應如何考慮文件資料庫中的文件？
* 什麼是資料模型化，以及為什麼應該關心？ 
* 文件資料庫中的模型化資料與關聯式資料庫有何不同？
* 如何表達非關聯式資料庫中的資料關聯性？
* 何時內嵌資料，以及何時連結至資料？

## <a name="embedding-data"></a>內嵌資料
當您開始在文件存放區 (例如 Azure Cosmos DB) 中將資料模型化時，請嘗試將您的實體視為 JSON 中呈現的 **獨立式文件**。

在我們更進一步深入探討之前，讓我們往回幾個步驟，看看我們在關聯式資料庫中如何建立某個項目的模型，這是許多人已熟悉的主題。 下列範例示範人員可能如何儲存在關聯式資料庫中。 

![關聯式資料庫模型](./media/sql-api-modeling-data/relational-data-model.png)

使用關聯式資料庫時，我們已多年被告知要進行正規化、正規化、正規化。

將您的資料正規化通常牽涉到取得實體 (例如：個人)，再劃分以顯示各個部分的資料。 在上述範例中，人員可以有多個連絡詳細資料記錄，以及多個地址記錄。 我們甚至透過進一步擷取一般欄位 (像是類別)，更進一步細分連絡詳細資料。 地址也是如此，此處的每個記錄具有類型 (像是 [住家] 或 [商務]) 

將資料正規化的引導前提是在每個記錄資料上 **避免儲存多餘的資料** ，而是參考資料。 在此範例中，若要讀取人員以及其所有連絡詳細資料和地址，您需要使用「聯結」有效地在執行階段彙總資料。

    SELECT p.FirstName, p.LastName, a.City, cd.Detail
    FROM Person p
    JOIN ContactDetail cd ON cd.PersonId = p.Id
    JOIN ContactDetailType on cdt ON cdt.Id = cd.TypeId
    JOIN Address a ON a.PersonId = p.Id

更新單一人員與其連絡詳細資料和地址需要跨許多個別資料表的寫入作業。 

現在讓我們看看如何將相同的資料模型化為文件資料庫中的獨立實體。

    {
        "id": "1",
        "firstName": "Thomas",
        "lastName": "Andersen",
        "addresses": [
            {            
                "line1": "100 Some Street",
                "line2": "Unit 1",
                "city": "Seattle",
                "state": "WA",
                "zip": 98012
            }
        ],
        "contactDetails": [
            {"email": "thomas@andersen.com"},
            {"phone": "+1 555 555-5555", "extension": 5555}
        ] 
    }

使用上述方法，我們現在已將人員記錄**反正規化**，在此**內嵌**與人員相關的所有資訊 (例如他們的連絡詳細資料和地址) 到單一 JSON 文件。
此外，因為我們不受限於固定的結構描述，我們有彈性可進行一些動作，像是讓連絡詳細資料不同的圖形。 

從資料庫擷取完整的人員記錄現在是針對單一集合和單一文件的單一讀取作業。 更新連絡詳細資料和地址等個人記錄，也是針對單一文件的單一寫入作業。

藉由反正規化資料，您的應用程式可能需要發出更少的查詢和更新以完成一般作業。 

### <a name="when-to-embed"></a>內嵌的時機
一般而言，使用內嵌的資料模型的時機為：

* 實體之間有 **包含** 關聯性。
* 實體之間有 **一對一些** 關聯性。
* 內嵌的資料 **不常變更**。
* 內嵌的資料不會 **無限的成長**。
* 內嵌的資料是文件中資料的 **整數** 。

> [!NOTE]
> 通常反正規化的資料模型可提供較佳的 **讀取** 效能。
> 
> 

### <a name="when-not-to-embed"></a>不要內嵌的時機
雖然在文件資料庫中的經驗法則是反正規化所有項目，並內嵌所有資料至單一文件，這可能導致應該避免的某些情況。

取得此 JSON 程式碼片段。

    {
        "id": "1",
        "name": "What's new in the coolest Cloud",
        "summary": "A blog post by someone real famous",
        "comments": [
            {"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
            {"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
            …
            {"id": 100001, "author": "jane", "comment": "and on we go ..."},
            …
            {"id": 1000000001, "author": "angry", "comment": "blah angry blah angry"},
            …
            {"id": ∞ + 1, "author": "bored", "comment": "oh man, will this ever end?"},
        ]
    }

如果我們要模型化一般的部落格或 CMS 系統，這可能是具有內嵌註解的文章實體的外觀。 此範例的問題在於註解陣列是 **unbounded**，表示任何單一文章可以具備的註解數目沒有 (實際) 的限制。 因為文件的大小可能會大幅成長，這會成為問題。

隨著文件大小的增加，透過網路傳輸資料以及大規模讀取和更新文件的能力，將會受到影響。

在此情況下，最好請考慮下列模型。

    Post document:
    {
        "id": "1",
        "name": "What's new in the coolest Cloud",
        "summary": "A blog post by someone real famous",
        "recentComments": [
            {"id": 1, "author": "anon", "comment": "something useful, I'm sure"},
            {"id": 2, "author": "bob", "comment": "wisdom from the interwebs"},
            {"id": 3, "author": "jane", "comment": "....."}
        ]
    }

    Comment documents:
    {
        "postId": "1"
        "comments": [
            {"id": 4, "author": "anon", "comment": "more goodness"},
            {"id": 5, "author": "bob", "comment": "tails from the field"},
            ...
            {"id": 99, "author": "angry", "comment": "blah angry blah angry"}
        ]
    },
    {
        "postId": "1"
        "comments": [
            {"id": 100, "author": "anon", "comment": "yet more"},
            ...
            {"id": 199, "author": "bored", "comment": "will this ever end?"}
        ]
    }

此模型的文章本身內嵌有三個最新註解，也就是目前具有固定繫結的陣列。 其他註解會分組為 100 個註解的批次，並儲存在個別的文件。 因為我們虛構的應用程式可讓使用者一次載入 100 個註解，因此已將批次大小選擇為 100。  

內嵌資料並不是好主意的另一種情況是在內嵌的資料經常跨文件使用，而且會經常變更時。 

取得此 JSON 程式碼片段。

    {
        "id": "1",
        "firstName": "Thomas",
        "lastName": "Andersen",
        "holdings": [
            {
                "numberHeld": 100,
                "stock": { "symbol": "zaza", "open": 1, "high": 2, "low": 0.5 }
            },
            {
                "numberHeld": 50,
                "stock": { "symbol": "xcxc", "open": 89, "high": 93.24, "low": 88.87 }
            }
        ]
    }

這可以代表個人的股票組合。 我們已選擇內嵌股票資料到每個組合文件。 在相關資料經常會變更的環境中，像是股票交易應用程式，內嵌經常變更的資料表示，每次交易股票時您便經常更新每個組合文件。

股票 *zaza* 可能在單一日交易數百次，而上千名使用者的組合上可能有 *zaza*。 有鑑於如上所示的資料模型，我們每天必須更新數千個組合文件許多次，導致系統無法妥善延展。 

## <a id="Refer"></a>參考資料
因此，內嵌資料於許多情況下可適用，但很明顯有時反正規化資料將會造成更多問題，使得適得其反。 那我們現在該怎麼辦？ 

關聯式資料庫不是您可以建立實體之間的關聯性的唯一位置。 在文件資料庫中，您的文件中可以有實際上與其他文件中的資料相關的資訊。 現在，我不是要多用一分鐘來提倡我們建置的系統會更適合 Azure Cosmos DB 中的關聯式資料庫或任何其他文件資料庫，但是簡單的關聯性很好而且可能非常有用。 

在以下的 JSON 中，我們選擇使用先前的股票組合範例，但這次我們參考組合上的股票項目而不是加以內嵌。 如此一來，經常全天變更的股票項目，需要更新的唯一文件是單一股票文件。 

    Person document:
    {
        "id": "1",
        "firstName": "Thomas",
        "lastName": "Andersen",
        "holdings": [
            { "numberHeld":  100, "stockId": 1},
            { "numberHeld":  50, "stockId": 2}
        ]
    }

    Stock documents:
    {
        "id": "1",
        "symbol": "zaza",
        "open": 1,
        "high": 2,
        "low": 0.5,
        "vol": 11970000,
        "mkt-cap": 42000000,
        "pe": 5.89
    },
    {
        "id": "2",
        "symbol": "xcxc",
        "open": 89,
        "high": 93.24,
        "low": 88.87,
        "vol": 2970200,
        "mkt-cap": 1005000,
        "pe": 75.82
    }


不過，這個方法目前的缺點是您的應用程式是否需要在顯示人員的組合時顯示持有的每個股票的相關資訊；在此情況下，您必須對資料庫中進行多個來回行程，才能載入每個股票文件的資訊。 在這裡，我們決定提升全天經常發生之寫入作業的效率，但反而對此特定的系統效能影響較小的讀取作業有害。

> [!NOTE]
> 正規化的資料模型 **可能需要更多來回行程** 到伺服器。
> 
> 

### <a name="what-about-foreign-keys"></a>外部索引鍵呢？
因為目前沒有條件約束、外部索引鍵之類的概念，您在文件中具有的任何文件間關聯性實際上是「弱式連結」，並且將不會由資料庫本身驗證。 如果您想要確定文件所參考的資料真的存在，您就需要在您的應用程式中這麼做，或在 Azure Cosmos DB 上透過使用伺服器端觸發程序或預存程序。

### <a name="when-to-reference"></a>參考時機
一般而言，使用正規化資料模型的時機為：

* 代表 **一對多** 關聯性。
* 代表 **多對多** 關聯性。
* 相關資料 **經常變更**。
* 參考資料可能是 **unbounded**。

> [!NOTE]
> 通常正規化可提供較佳的 **寫入** 效能。
> 
> 

### <a name="where-do-i-put-the-relationship"></a>放置關聯性的位置為何？
關聯性的成長將有助於判斷用來儲存參考的文件。

如果我們看看下面會建立發行者和書籍模型的 JSON。

    Publisher document:
    {
        "id": "mspress",
        "name": "Microsoft Press",
        "books": [ 1, 2, 3, ..., 100, ..., 1000]
    }

    Book documents:
    {"id": "1", "name": "Azure Cosmos DB 101" }
    {"id": "2", "name": "Azure Cosmos DB for RDBMS Users" }
    {"id": "3", "name": "Taking over the world one JSON doc at a time" }
    ...
    {"id": "100", "name": "Learn about Azure Cosmos DB" }
    ...
    {"id": "1000", "name": "Deep Dive in to Azure Cosmos DB" }

如果每個發行者書籍的數量很少而且成長有限，那麼，將書籍參考儲存在發行者文件內可能很有用。 不過，如果每個發行者的書籍數量無限，此資料模型會導致可變動、成長的陣列，如上述的範例發行者文件所示。 

切換項目位元會導致模型仍代表相同的資料，但現在可避免這些大型的可變動集合。

    Publisher document: 
    {
        "id": "mspress",
        "name": "Microsoft Press"
    }

    Book documents: 
    {"id": "1","name": "Azure Cosmos DB 101", "pub-id": "mspress"}
    {"id": "2","name": "Azure Cosmos DB for RDBMS Users", "pub-id": "mspress"}
    {"id": "3","name": "Taking over the world one JSON doc at a time"}
    ...
    {"id": "100","name": "Learn about Azure Cosmos DB", "pub-id": "mspress"}
    ...
    {"id": "1000","name": "Deep Dive in to Azure Cosmos DB", "pub-id": "mspress"}

在上述範例中，我們在發行者文件上捨棄無限制的集合。 我們在每個書籍文件上只有發行者的參考。

### <a name="how-do-i-model-manymany-relationships"></a>如何建立多對多關聯性的模型？
在關聯式資料庫 *多對多* 關聯性中，通常是與聯結資料表模型化，其只是將記錄從其他資料表聯結在一起。 

![聯結資料表](./media/sql-api-modeling-data/join-table.png)

您可能會想要使用文件複寫相同的項目，並產生看起來如下所示的資料模型。

    Author documents: 
    {"id": "a1", "name": "Thomas Andersen" }
    {"id": "a2", "name": "William Wakefield" }

    Book documents:
    {"id": "b1", "name": "Azure Cosmos DB 101" }
    {"id": "b2", "name": "Azure Cosmos DB for RDBMS Users" }
    {"id": "b3", "name": "Taking over the world one JSON doc at a time" }
    {"id": "b4", "name": "Learn about Azure Cosmos DB" }
    {"id": "b5", "name": "Deep Dive in to Azure Cosmos DB" }

    Joining documents: 
    {"authorId": "a1", "bookId": "b1" }
    {"authorId": "a2", "bookId": "b1" }
    {"authorId": "a1", "bookId": "b2" }
    {"authorId": "a1", "bookId": "b3" }

這應該可行。 不過，載入其中一個作者的書籍，或載入書籍的作者，一律需要對資料庫進行至少兩個其他查詢。 一個對聯結文件的查詢，另一個查詢則用來擷取實際聯結的文件。 

如果此聯結資料表的作用完全是在結合兩組資料在一起，那麼為何不完全捨棄它？
請考慮下列。

    Author documents:
    {"id": "a1", "name": "Thomas Andersen", "books": ["b1, "b2", "b3"]}
    {"id": "a2", "name": "William Wakefield", "books": ["b1", "b4"]}

    Book documents: 
    {"id": "b1", "name": "Azure Cosmos DB 101", "authors": ["a1", "a2"]}
    {"id": "b2", "name": "Azure Cosmos DB for RDBMS Users", "authors": ["a1"]}
    {"id": "b3", "name": "Learn about Azure Cosmos DB", "authors": ["a1"]}
    {"id": "b4", "name": "Deep Dive in to Azure Cosmos DB", "authors": ["a2"]}

現在，如果我有作者，我立即會知道他們的書籍，而反之，如果我載入書籍文件，我就知道作者的識別碼。 這樣可以省下對聯結資料表的中繼查詢，減少您的應用程式必須進行的伺服器來回行程數目。 

## <a id="WrapUp"></a>混合式資料模型
我們現在已看過內嵌 (或反正規化) 和參考 (或正規化) 資料，如我們所見，各有其優缺點。 

不必害怕採行不同的方式。 

根據您的應用程式特定的使用模式和工作負載，可能有時候混用內嵌和參考的資料有意義，而可能導致較簡單的應用程式邏輯與較少的伺服器來回行程，同時維持良好的效能等級。

請考慮下列 JSON。 

    Author documents: 
    {
        "id": "a1",
        "firstName": "Thomas",
        "lastName": "Andersen",        
        "countOfBooks": 3,
         "books": ["b1", "b2", "b3"],
        "images": [
            {"thumbnail": "http://....png"}
            {"profile": "http://....png"}
            {"large": "http://....png"}
        ]
    },
    {
        "id": "a2",
        "firstName": "William",
        "lastName": "Wakefield",
        "countOfBooks": 1,
        "books": ["b1"],
        "images": [
            {"thumbnail": "http://....png"}
        ]
    }

    Book documents:
    {
        "id": "b1",
        "name": "Azure Cosmos DB 101",
        "authors": [
            {"id": "a1", "name": "Thomas Andersen", "thumbnailUrl": "http://....png"},
            {"id": "a2", "name": "William Wakefield", "thumbnailUrl": "http://....png"}
        ]
    },
    {
        "id": "b2",
        "name": "Azure Cosmos DB for RDBMS Users",
        "authors": [
            {"id": "a1", "name": "Thomas Andersen", "thumbnailUrl": "http://....png"},
        ]
    }

這裡我們 (差不多) 看完了內嵌的模型，即來自其他實體的資料會內嵌在最上層的文件中，但參考其他資料。 

如果您查看書籍文件，在查看作者陣列時就會看到一些有趣的欄位。 有一個 *id* 欄位，這是我們用來往回參考作者文件的欄位，正規化的模型中的標準作法，然後我們也有 *name* 和 *thumbnailUrl*。 我們可能已受限在 *id* 並讓應用程式使用「連結」向個別作者文件取得其所需的任何其他資訊，但因為我們的應用程式會顯示每個書籍作者的名稱和縮圖圖片，我們可以省下對伺服器就清單中的每個書籍的來回行程由，方法是反正規化 **某些** 作者的資料。

當然，如果作者的名稱變更，或他們想要更新其相片，我們得更新曾經發行的每個書籍，但對我們的應用程式，根據作者不常變更其名稱的假設，這是可接受的設計決策。  

在範例中，有 **預先計算彙總** 值可節省讀取作業費用高昂的處理。 在範例中，作者文件中內嵌的有些資料是在執行階段計算的資料。 每次發行新的書籍時，會建立書籍的文件 **並且** 將 countOfBooks 欄位設定為根據某位特定作者存在的書籍文件數目計算值。 在讀取繁重的系統中 (我們可以負擔執行寫入計算以最佳化讀取)，這項最佳化將很適合。

由於 Azure Cosmos DB 支援**多文件交易**，因此模型現在能夠具有預先計算的欄位。 許多 NoSQL 存放區無法跨文件中執行交易，而因為這項限制而提倡「一律內嵌一切」的設計決策。 藉由 Azure Cosmos DB，您可以使用伺服器端觸發程序或預存程序，插入書籍並更新作者，全都在 ACID 交易內完成。 現在您不 **需** 在一份文件內嵌所有內容，只需要確保您的資料保持一致。

## <a name="NextSteps"></a>後續步驟
從這篇文章獲得的最大的心得是了解在無結構描述的環境中的資料模型化與以往一樣重要。 

正如同沒有單一方法可表示螢幕上的資料片段，沒有單一方法可為您的資料建立模型。 您需要了解您的應用程式，以及它將如何產生、取用及處理資料。 然後，藉由套用一些此處所提供的指導方針，您可以設定相關的建立模型，來處理您的應用程式的立即需求。 當您的應用程式需要進行變更時，您可以利用無結構描述之資料庫的彈性來納入變更，並輕鬆進化您的資料模型。 

若要深入了解 Azure Cosmos DB，請參閱服務的[文件 (英文)](https://azure.microsoft.com/documentation/services/cosmos-db/)頁面。 

若要了解如何跨多個分割將您的資料分區，請參閱[在 Azure Cosmos DB 中分割資料](sql-api-partition-data.md)。 
