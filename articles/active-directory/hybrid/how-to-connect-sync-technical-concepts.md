---
title: Azure AD Connect 同步：技術概念 | Microsoft Docs
description: 說明 Azure AD Connect 同步的技術概念。
services: active-directory
documentationcenter: ''
author: billmath
manager: mtillman
editor: ''
ms.assetid: 731cfeb3-beaf-4d02-aef4-b02a8f99fd11
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/15/2018
ms.component: hybrid
ms.author: billmath
ms.openlocfilehash: df3f244b70b52b6145b664c4e575c66542ada5b9
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46309577"
---
# <a name="azure-ad-connect-sync-technical-concepts"></a>Azure AD Connect 同步處理：技術概念
本文是 [了解架構](how-to-connect-sync-technical-concepts.md)主題的摘要。

Azure AD Connect 同步建置在一個穩固的中繼目錄同步處理平台上。
下列各節介紹中繼目錄同步處理的概念。
建置於 MIIS、ILM 以及 FIM 之上，Azure Active Directory 同步服務提供連接資料來源、在資料來源間同步處理資料，以及佈建和解除佈建身份識別的下一代平台。

![技術概念](./media/how-to-connect-sync-technical-concepts/scenario.png)

下列章節提供有關 FIM 同步處理服務下列層面的詳細資訊：

* 連接器
* 屬性流程
* 連接器空間
* Metaverse
* 佈建

## <a name="connector"></a>連接器
用來與連接的目錄通訊之程式碼模組，稱為連接器 (之前稱為管理代理程式 (MA))。

這些模組是安裝在執行 Azure AD Connect 同步的電腦上。連接器使用遠端系統通訊協定，替代依賴特定代理程式的部署，提供無代理程式能力的通訊。 這表示降低了風險與部署時間，特別是在處理重要的應用程式與系統時。

在上圖中，連接器與連接器空間是同義詞，但包含了所有與外部系統的通訊。

連接器負責系統的所有匯入及匯出功能，並且讓開發人員不需要了解如何以原生方式連接到每個系統，即可使用宣告式佈建來自訂資料轉換。

匯入與匯出作業只會在排程時進行，以進一步與系統中發生的變更隔絕，因為變更不會自動傳播至連接的資料來源。 此外，開發人員也可以建立自己的連接器，幾乎可連接至任何一個資料來源。

## <a name="attribute-flow"></a>屬性流程
Metaverse 是鄰近連接器空間中所有聯結的身份識別的合併檢視。 在上圖中，屬性流程是利用具有箭頭的線條來繪製輸入和輸出流程。 屬性流程是將資料從一個系統複製或轉換到另一個系統以及所有屬性流程 (輸入或輸出) 的程序。

當同步處理 (完整或差異) 作業已安排要執行時，屬性流程會在連接器空間與 Metaverse 的雙向通訊之間進行。

屬性流程只會在執行這些同步處理時進行。 屬性流動則是在同步化規則中定義。 這些可以是輸入 (如上圖之 ISR) 或輸出 (如上圖之 OSR)。

## <a name="connected-system"></a>連接的系統
連接的系統 (也稱為連接的目錄) 是指 Azure AD Connect 同步處理已連接且來回讀取和寫入身分識別資料的遠端系統。

## <a name="connector-space"></a>連接器空間
每個已連接的資料來源都會以連接器空間中物件和屬性的已篩選子集的方式表示。
這可讓同步服務在本機作業，而不需要在同步處理物件時連接遠端系統並將互動限制為僅匯入及匯出。

當資料來源及連接器具有提供變更清單 (差異匯入) 的能力時，因為只在最後一次輪詢循環交換之後變更，作業效率會大幅提升。 連接器空間可透過要求連接器排程匯入及匯出，讓已連接的資料來源不會受到自動傳播變更影響。 這項新增的預防措施讓您在測試、預覽，或確認下次更新時可以更加放心。

## <a name="metaverse"></a>Metaverse
Metaverse 是鄰近連接器空間中所有聯結的身份識別的合併檢視。

由於身份識別連結在一起，且授權透過匯入流程對應指派給各種屬性，中心 Metaverse 物件要從多個系統開始彙總資訊。 從此物件屬性流程，對應包含輸出系統的資訊。

當授權系統投影物件到 Metaverse 時會建立物件。 在所有的連接移除後，就會刪除 Metaverse 物件。

您無法直接編輯 Metaverse 中的物件。 物件中的所有資料都必須透過屬性流程來提供。 Metaverse 會利用每個連接器空間來維護永續性連接器。 這些連接器不需要在每次執行每次同步處理進行重新評估。 這表示 Azure AD Connect 同步處理不需每次尋找相符的遠端物件。 這樣就不需要使用昂貴的代理程式，來防止對通常負責建立物件之間關聯的屬性進行變更。

探索可能具有既存需管理之物件的新資料來源 時，Azure AD Connect 同步會使用名為「聯結規則」的程序，評估建立連結的潛在候選項目。
連結一旦建立，此評估就不會重新執行，而一般屬性流程則可在遠端連結的資料來源和 Metaverse 間進行。

## <a name="provisioning"></a>佈建
當授權來源投影一個新物件至 Metaverse　時，會在另一個連接器中建立新的連接器空間物件，代表一個下游的已連接資料來源。

這本身就會建立連結，且屬性流程可以雙向進行。

每當規則判斷需要建立新的連接器空間物件，此即稱為佈建。 不過，因為此作業只會在連接器空間中進行，所以在執行匯出之前不會影響已連接的資料來源。

## <a name="additional-resources"></a>其他資源
* [Azure AD Connect 同步處理：自訂同步處理選項](how-to-connect-sync-whatis.md)
* [整合內部部署身分識別與 Azure Active Directory](whatis-hybrid-identity.md)

<!--Image references-->
[1]: ./media/active-directory-aadsync-technical-concepts/ic750598.png
