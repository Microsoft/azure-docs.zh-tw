---
title: LUIS 應用程式共同作業 - Language Understanding
titleSuffix: Azure Cognitive Services
description: LUIS 應用程式要求只能有一個擁有者，共同作業者則為選擇性。
services: cognitive-services
author: diberry
manager: cgronlun
ms.service: cognitive-services
ms.component: language-understanding
ms.topic: article
ms.date: 09/10/2018
ms.author: diberry
ms.openlocfilehash: 38fc33a6fb823e0435a9c96979c5a9a4539cd6ba
ms.sourcegitcommit: 4ecc62198f299fc215c49e38bca81f7eb62cdef3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47038763"
---
# <a name="collaborating"></a>共同作業

LUIS 提供共同作業，可允許一群人撰寫應用程式。

## <a name="luis-account"></a>LUIS 帳戶
LUIS 帳戶會與單一 [Microsoft Live](https://login.live.com/) 帳戶關聯。 每個 LUIS 帳戶都會獲得一個免費的[撰寫金鑰](luis-concept-keys.md#authoring-key)，可用來撰寫該帳戶可存取的所有 LUIS 應用程式。 

一個 LUIS 帳戶可以有許多 LUIS 應用程式。

若要深入了解 Active Directory 使用者帳戶，請參閱 [Azure Active Directory 租用戶使用者](luis-how-to-collaborate.md#azure-active-directory-tenant-user)。 

## <a name="luis-app-owner"></a>LUIS 應用程式擁有者
建立應用程式的帳戶即為擁有者。 每個應用程式都有單一擁有者。 該擁有者會列在應用程式 [[Settings](luis-how-to-collaborate.md)] \(設定\) 上。 這是可以刪除應用程式的帳戶。 這也是端點配額達到每月限制的 75% 時，會收到電子郵件通知的帳戶。 

## <a name="authorization-roles"></a>授權角色
對於擁有者和共同作業者，LUIS 不支援不同的角色，但有一個例外。 擁有者是可以刪除應用程式的唯一帳戶。

如果您想要控制模型的存取權，請考慮將模型切割成較小的 LUIS 應用程式，其中每個較小的應用程式都有更加受限的共同作業者集合。 使用[分派](https://aka.ms/dispatch-tool)來允許父代 LUIS 應用程式管理父代和子系應用程式之間的協調。

## <a name="transfer-ownership"></a>轉送擁有權
LUIS 不提供擁有權轉移，不過任何共同作業者都可以匯出應用程式，然後再將其匯入來建立應用程式。 請注意，新應用程式會有不同的應用程式識別碼。 您必須將新應用程式定型、發佈，然後使用新端點。

## <a name="luis-app-collaborators"></a>LUIS 應用程式共同作業者
應用程式擁有者可以將共同作業者新增至應用程式。 擁有者必須在應用程式 [[Settings](luis-how-to-collaborate.md)] \(設定\) 上新增共同作業者的電子郵件地址。 共同作業者具備應用程式的完整存取權。 如果共同作業者刪除應用程式，該應用程式會從共同作業者的帳戶中刪除，但會留存在擁有者的帳戶中。 

如果您想要與共同作業者共用多個應用程式，則須為每個應用程式新增共同作業者的電子郵件。 

## <a name="managing-multiple-authors"></a>管理多個作者
[LUIS](luis-reference-regions.md#luis-website) 網站目前不提供交易層級撰寫。 您可以允許作者在來自基底版本的獨立版本上作業。 下列各節將說明兩種不同的方法。

## <a name="manage-multiple-versions-inside-the-same-app"></a>管理相同應用程式內的多個版本
針對每個作者，一開始先從基底版本[複製](luis-how-to-manage-versions.md#clone-a-version)。 

每個作者對自己的應用程式版本進行變更。 在每個作者對模型感到滿意之後，請將新版本匯出成 JSON 檔案。  

匯出的應用程式會是 JSON 格式的檔案，可用來比較變更。 結合檔案以建立新版本的單一 JSON 檔案。 變更 JSON 中的 **versionId** 屬性，以表示新的合併版本。 將該版本匯入至原始應用程式。 

此方法可讓您有一個作用中版本、一個預備版本，以及一個已發佈版本。 您可以在互動式測試窗格中比較這三種版本的結果。

## <a name="manage-multiple-versions-as-apps"></a>以應用程式方式管理多個檔案
[匯出](luis-how-to-manage-versions.md#export-version)基底版本。 每個作者都匯入版本。 匯入應用程式的人即為該版本的擁有者。 當他們修改完應用程式時，請匯出版本。 

匯出的應用程式會是 JSON 格式的檔案，可用來與基底匯出版本比較變更。 結合檔案以建立新版本的單一 JSON 檔案。 變更 JSON 中的 **versionId** 屬性，以表示新的合併版本。 將該版本匯入至原始應用程式。

## <a name="next-steps"></a>後續步驟

了解[版本設定](luis-concept-version.md)概念。 

請參閱[應用程式設定](luis-how-to-collaborate.md)，以了解如何管理您 LUIS 應用程式中的共同作業者。

請參閱如何使用「撰寫 API」[將電子郵件新增至存取清單](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/58fcccdd5aca2f08a4104342)。