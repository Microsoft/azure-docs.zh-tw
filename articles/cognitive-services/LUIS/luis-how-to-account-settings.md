---
title: 管理您在 LUIS 中的帳戶設定 | Microsoft Docs
description: 使用 LUIS 網站來管理您的帳戶設定。
titleSuffix: Azure
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 07/08/2018
ms.author: diberry
ms.openlocfilehash: f3086f09e29664b816ba709fc5cda75d7b11d1b4
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47035244"
---
# <a name="manage-account-and-authoring-key"></a>管理帳戶和撰寫金鑰
LUIS 帳戶的兩個重要資訊為使用者帳戶和撰寫金鑰。 您的登入資訊是在 [account.microsoft.com](https://account.microsoft.com) 進行管理的。 您的撰寫金鑰是從 [LUIS](luis-reference-regions.md) 網站的 [設定] 頁面進行管理的。 

## <a name="authoring-key"></a>撰寫金鑰

這個單一且區域專屬的撰寫金鑰位於 [設定] 頁面上，可讓您從 [LUIS](luis-reference-regions.md) 網站以及[撰寫 API](https://aka.ms/luis-authoring-api) 撰寫所有應用程式。 為了方便起見，允許撰寫金鑰每個月建立[有限](luis-boundaries.md)數量的端點查詢。 

![LUIS 設定頁面](./media/luis-how-to-account-settings/account-settings.png)

撰寫金鑰是用於您所擁有的任何應用程式，以及任何您被列為共同作業者的應用程式。

## <a name="authoring-key-regions"></a>撰寫金鑰區域
不同的[撰寫區域](luis-reference-regions.md#publishing-regions)有自己特有的撰寫金鑰。 這種金鑰無法在不同的區域中使用。 

## <a name="reset-authoring-key"></a>重設撰寫金鑰
如果您的撰寫金鑰已被盜用，請重設金鑰。 該金鑰會在 [LUIS](luis-reference-regions.md) 網站的所有應用程式上重設。 如果您透過撰寫 API 來撰寫您的應用程式，則需要將 `Ocp-Apim-Subscription-Key` 的值變更成新的金鑰。 

## <a name="delete-account"></a>刪除帳戶
如需刪除帳戶時會刪除哪些資料的相關資訊，請參閱[資料儲存和移除](luis-concept-data-storage.md#accounts)。 

## <a name="next-steps"></a>後續步驟

深入了解您的[撰寫金鑰](luis-concept-keys.md#authoring-key)。 

