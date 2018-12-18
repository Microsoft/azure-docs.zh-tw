---
title: Azure 資料目錄簡介
description: 本文提供 Microsoft Azure 資料目錄的概觀，包括其具備的功能以及所解決的問題。 資料目錄可讓任何使用者註冊、探索、了解及取用資料來源。
services: data-catalog
author: markingmyname
ms.author: maghan
ms.assetid: cc733907-17ec-4153-9f0c-5b3754b2db19
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 3da12ae290730869026de24761b28f332927f01e
ms.sourcegitcommit: b7e5bbbabc21df9fe93b4c18cc825920a0ab6fab
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/27/2018
ms.locfileid: "47408448"
---
# <a name="what-is-azure-data-catalog"></a>什麼是 Azure 資料目錄？
Azure 資料目錄是完全受控的雲端服務，能讓其使用者探索它們需要的資料來源，並了解他們所尋找的資料來源。 同時，資料目錄可協助組織從現有的投資中獲得更多價值。 

任何使用者 (分析師、資料科學家或開發人員) 都能利用資料目錄來探索、了解及取用資料來源。 資料目錄包括中繼資料和附註的 crowdsourcing 模型。 它是單一的中心位置，能讓組織的所有使用者貢獻其專業知識，並建置資料的社群和文化特性。

## <a name="discovery-challenges-for-data-consumers"></a>探索資料取用者面臨的挑戰
傳統上，探索企業資料來源一向都是根據部落知識的一項有機程序。 對於想要充分利用其資訊資產的公司，這個方法會帶來許多挑戰：

* 使用者可能不知道，除非他們在另一個流程當中與資料來源有所接觸，否則資料來源會一直存在。 並沒有任何已註冊資料來源的中央位置。
* 除非使用者知道資料來源的位置，否則它們無法使用用戶端應用程式連線到資料。 資料取用體驗需要使用者知道連接字串或路徑。
* 除非使用者知道資料來源文件的位置，否則無法了解資料的用途。 資料來源和文件可能會存在於許多地方，且可透過各種不同的體驗來取用。
* 如果使用者有關於資訊資產的疑問，必須洽詢負責資料的專家或小組，並讓它們離線參與。 資料和包含關於其使用方式之專家觀點的資料之間，沒有任何明確的連線。
* 除非使用者了解要求存取資料來源的程序，否則，探索資料來源及其文件仍無法協助他們存取資料。

## <a name="discovery-challenges-for-data-producers"></a>探索資料產生者面臨的挑戰
雖然資料取用者面臨先前所述的挑戰，但負責產生和維護的資訊資產的使用者本身也面臨挑戰：

* 將具有描述性中繼資料的資料來源加以註釋，通常是徒勞無功的。 用戶端應用程式通常會忽略儲存在資料來源的描述。
* 建立資料來源的文件，通常是徒勞無功的。 保持文件與資料來源同步處理是一項持續的責任，使用者可能會對於過期的文件缺乏信任。
* 建立和維護資料來源的文件既複雜又耗時。 而要讓使用資料來源的每個使用者可隨時取得文件可能會更為艱鉅。
* 限制對資料來源的存取，並確保資料取用者知道如何要求存取是一項持續的挑戰。

這些挑戰結合起來，會形成更大的障礙，使得公司更難以鼓勵和推動使用企業資料並加以了解。

## <a name="azure-data-catalog-can-help"></a>Azure 資料目錄能提供協助
資料目錄旨在解決這些問題，有助於讓企業能夠充分利用現有的資訊資產。 資料目錄能讓管理資料的使用者輕鬆地探索和了解資料來源。

資料目錄提供雲端型服務，其中可以註冊資料來源。 資料會保留在現有的位置，但其中繼資料的複本會連同資料來源位置的參考，一起新增至資料目錄。 此中繼資料也會編製索引，透過搜尋輕鬆找到每個資料來源，並讓探索資料來源的使用者了解每個資料來源。

註冊資料來源之後，進行註冊的使用者或企業中的其他使用者即可充實其中繼資料。 任何使用者都可以提供描述、標記或其他中繼資料 (例如要求資料來源存取的文件和程序) 來加註資料來源。 此描述性中繼資料可補充資料來源中已註冊的結構化中繼資料 (例如資料行名稱和資料類型)。

註冊來源的主要目的是為了探索和了解資料來源及其用途。 企業使用者可能需要的資料，是需要正確資料的資料商業智慧、應用程式開發、資料科學，或任何其他工作。 他們可以使用資料目錄探索體驗，快速尋找符合其需求的資料、了解資料以評估其適合的用途，並在其選擇的工具中開啟資料來源來取用資料。 

同時，使用者可以參與目錄，方法是將已註冊的資料來源加以標記、記錄及註解。 它們也可以註冊新的資料來源，接著使用者的社群便可加以探索、了解和取用。

![資料目錄功能](./media/data-catalog-what-is-data-catalog/data-catalog-capabilities.png)

## <a name="learn-more-about-data-catalog"></a>深入了解資料目錄
若要深入了解資料目錄的功能，請參閱︰

* [如何註冊資料來源](data-catalog-how-to-register.md)
* [如何探索資料來源](data-catalog-how-to-discover.md)
* [如何註解資料來源](data-catalog-how-to-annotate.md)
* [如何記載資料來源](data-catalog-how-to-documentation.md)
* [如何連接到資料來源](data-catalog-how-to-connect.md)
* [如何使用巨量資料](data-catalog-how-to-big-data.md)
* [如何管理資料資產](data-catalog-how-to-manage.md)
* [如何設定商務詞彙](data-catalog-how-to-business-glossary.md)
* [常見問題集](data-catalog-frequently-asked-questions.md)

## <a name="next-steps"></a>後續步驟
如需開始使用資料目錄，請移至：
* [Microsoft Azure 資料目錄](https://www.azuredatacatalog.com)
* [開始使用 Azure 資料目錄](data-catalog-get-started.md)
