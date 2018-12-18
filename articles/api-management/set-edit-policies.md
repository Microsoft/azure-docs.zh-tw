---
title: 如何設定或編輯 Azure API 管理原則 | Microsoft Docs
description: 本主題會示範如何設定或編輯 Azure API 管理原則。
services: api-management
documentationcenter: ''
author: vladvino
manager: cflower
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/27/2017
ms.author: apimpm
ms.openlocfilehash: aaf86a440328e27c8c47b809536951eeaf2104b9
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/10/2018
---
# <a name="how-to-set-or-edit-azure-api-management-policies"></a>如何設定或編輯 Azure API 管理原則

原則定義是一份 XML 文件，描述一連串輸入和輸出陳述式。 可直接在定義視窗中編輯 XML。 您也可以從原則視窗右側所提供的清單中，選取預先定義的原則。 適用於目前範圍的陳述式會是啟用狀態並且醒目顯示。 按一下已啟用的陳述式，定義檢視中的游標位置上就會新增適當的 XML。 

如需有關原則的詳細資訊，請參閱 [Azure API 管理中的原則](api-management-howto-policies.md)。

## <a name="set-or-edit-a-policy"></a>設定或編輯原則

若要設定或編輯原則，請遵循下列步驟：

1. 在 [https://portal.azure.com](https://portal.azure.com) 登入 Azure 入口網站。
2. 瀏覽至您的 APIM 執行個體。
3. 按一下 [API] 索引標籤。
4. 選取其中一個您先前匯入的 API。
5. 選取 [設計] 索引標籤。
6. 選取您要套用原則的作業。 如果您想要將原則套用至所有作業，請選取 [所有作業]。
7. 按一下 [輸入] 或 [輸出] 鉛筆旁的三角形。
8. 選取 [程式碼編輯器] 項目。

    ![編輯原則](./media/set-edit-policies/set-edit-policies01.png)

9. 將需要的原則程式碼貼入其中一個適當的區塊。
         
        <policies>
             <inbound>
                 <base />
             </inbound>
             <backend>
                 <base />
             </backend>
             <outbound>
                 <base />
             </outbound>
             <on-error>
                 <base />
             </on-error>
         </policies>
 
## <a name="configure-scope"></a>設定範圍

原則可以整體設定，或在產品、API 或作業的範圍上設定。 若要開始設定原則，您必須先選取要套用原則的範圍。

系統會以下列順序評估原則範圍：

1. 全域範圍
2. 產品範圍
3. API 範圍
4. 作業範圍

系統會根據 `base` 元素的位置 (若有的話)，評估原則中的陳述式。 全域原則沒有父系原則，在全域原則中使用 `<base>` 元素沒有任何作用。

若要在原則編輯器中查看位於目前範圍的原則，請按一下 [ **重新計算所選取範圍的有效原則**]。

### <a name="global-scope"></a>全域範圍

全域範圍已設定為 APIM 執行個體中的**所有 API**。

1. 登入 [Azure 入口網站](https://portal.azure.com/)，然後瀏覽至您的 APIM 執行個體。
2. 按一下 [全部 API]。

    ![全域範圍](./media/api-management-howto-policies/global-scope.png)

3. 按一下三角形圖示。
4. 選取 [程式碼編輯器]。
5. 新增或編輯原則。
6. 按下 [儲存] 。 

    所做的變更會立即散佈至 API 管理閘道。

### <a name="product-scope"></a>產品範圍

產品範圍已設定為所選的產品。

1. 按一下 [產品]。

    ![產品範圍](./media/api-management-howto-policies/product-scope.png)

2. 選取您要套用原則的產品。
3. 按一下 [原則]。
4. 新增或編輯原則。
5. 按下 [儲存] 。 

### <a name="api-scope"></a>API 範圍

API 範圍已設定為所選 API 的**所有作業**。

1. 選取您要套用原則的 **API**。

    ![API 範圍](./media/api-management-howto-policies/api-scope.png)

2. 選取 [所有作業]
3. 按一下三角形圖示。
4. 選取 [程式碼編輯器]。
5. 新增或編輯原則。
6. 按下 [儲存] 。 

### <a name="operation-scope"></a>作業範圍 

作業範圍已設定為所選的作業。

1. 選取 **API**。
2. 選取您要套用原則的作業。

    ![作業範圍](./media/api-management-howto-policies/operation-scope.png)

3. 按一下三角形圖示。
4. 選取 [程式碼編輯器]。
5. 新增或編輯原則。
6. 按下 [儲存] 。 

## <a name="next-steps"></a>後續步驟

請參閱下列相關主題：

+ [轉換 API](transform-api.md)
+ [原則參考文件](api-management-policy-reference.md)，取得原則陳述式及其設定的完整清單
+ [原則範例](policy-samples.md)