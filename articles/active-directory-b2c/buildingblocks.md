---
title: BuildingBlocks - Azure Active Directory B2C | Microsoft Docs
description: 指定 Azure Active Directory B2C 中自訂原則的 BuildingBlocks 元素。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 85fbbc4e489c7b48f7dc95de1738636b7383de16
ms.sourcegitcommit: 3150596c9d4a53d3650cc9254c107871ae0aab88
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47419401"
---
# <a name="buildingblocks"></a>BuildingBlocks

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

**BuildingBlocks** 元素會新增到 [TrustFrameworkPolicy](trustframeworkpolicy.md) 元素內部。

```XML
<TrustFrameworkPolicy
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:xsd="http://www.w3.org/2001/XMLSchema"
  xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06"
  PolicySchemaVersion="0.3.0.0"
  TenantId="mytenant.onmicrosoft.com"
  PolicyId="B2C_1A_TrustFrameworkBase"
  PublicPolicyUri="http://mytenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase">

  <BuildingBlocks>
    <ClaimsSchema>
      ...
    </ClaimsSchema>
    <Predicates>
    ...
    </Predicates>
    <PredicateValidations>
    ...
    </PredicateValidations>
    <ClaimsTransformations>
      ...
    </ClaimsTransformations>
    <ContentDefinitions>
      ...
    </ContentDefinitions>
    <Localization>
      ...
    </Localization>
 </BuildingBlocks>
```

**BuildingBlocks** 元素包含下列必須以所定義順序來指定的元素：

- [ClaimsSchema](claimsschema.md)：定義可當作原則一部分進行參考的宣告類型。 宣告結構描述是可在其中宣告您的宣告類型的位置。 在許多程式設計語言中，宣告類型會與變數類似。 您可以使用宣告類型來向應用程式的使用者收集資料、接收來自社交識別提供者的宣告、從自訂 REST API 傳送和接收資料，或儲存您自訂原則所使用的任何內部資料。 

- [Predicates 與 PredicateValidationsInput](predicates.md)：可讓您執行驗證程序來確定只能在宣告中輸入正確格式的資料。
 
- [ClaimsTransformations](claimstransformations.md)：包含可在原則中使用的宣告轉換清單。  宣告轉換會將一個宣告轉換成另一個。 在宣告轉換中，您會指定轉換方法，例如： 
    - 將字串宣告的大小寫變更為指定的大小寫。 例如，將字串從小寫變更為大寫。
    - 比較兩個宣告，並使用 True 傳回宣告以指出宣告相符，否則會傳回 False。
    - 從原則中提供的參數建立字串宣告。
    - 使用隨機號碼產生器來建立隨機字串。
    - 根據提供的格式字串來將宣告格式化。 此轉換會使用 C# `String.Format` 方法。

- [ContentDefinitions](contentdefinitions.md)：包含要在使用者旅程圖中使用之 HTML5 範本的 URL。 在自訂原則中，內容定義會定義要在使用者旅程圖中針對指定步驟使用的 HTML5 頁面 URI。 例如，登入或註冊、密碼重設或錯誤網頁。 您可以透過覆寫該 HTML5 檔案的 LoadUri 來修改外觀及操作。 或者，可以根據您的需求來建立新的內容定義。 這個元素可能會使用當地語系化識別碼來包含當地語系化的資源參考。

- [Localization](localization.md)：可讓您支援多種語言。 原則中的當地語系化支援可讓您在原則中設定支援的語言清單，並挑選預設語言。 同時也支援特定語言的字串和集合。


