---
title: 針對 Azure 搜尋服務建立、管理及保護系統管理與查詢 API 金鑰 | Microsoft Docs
description: 服務端點的 API 金鑰控制存取權。 系統管理金鑰授與寫入權限。 可針對唯讀存取權建立查詢金鑰。
author: HeidiSteen
manager: cgronlun
tags: azure-portal
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.date: 06/20/2018
ms.author: heidist
ms.openlocfilehash: 2ec720f26cfbadb9963ff3991ad1795c9b30c136
ms.sourcegitcommit: d8ffb4a8cef3c6df8ab049a4540fc5e0fa7476ba
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/20/2018
ms.locfileid: "36284976"
---
# <a name="create-and-manage-api-keys-for-an-azure-search-service"></a>針對 Azure 搜尋服務建立及管理 API 金鑰

搜尋服務的所有要求需要專為您服務而產生的唯讀 API 金鑰。 此 API 金鑰是驗證存取您搜尋服務端點的唯一機制，必須包含在每個要求上。 在[REST 解決方案](search-get-started-nodejs.md#update-the-configjs-with-your-search-service-url-and-api-key)中，API 金鑰通常會要求標頭中指定。 在 [.NET 解決方案](search-howto-dotnet-sdk.md#core-scenarios)中，金鑰通常會指定為組態設定，然後作為 [管理金鑰](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchserviceclient.credentials) 或 [SearchCredentials](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchserviceclient.searchcredentials) (查詢金鑰) 在 [SearchServiceClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchserviceclient) 上傳遞。

金鑰會在服務佈建期間與您的搜尋服務一起建立。 您可以在 [Azure 入口網站](https://portal.azure.com)中檢視及取得金鑰值。

![入口網站頁面 > [設定] > [金鑰] 區段](media/search-manage/azure-search-view-keys.png)

## <a name="what-is-an-api-key"></a>什麼是 API 金鑰

API 金鑰是由隨機產生的數字和字母所組成的字串。 透過[角色型權限](search-security-rbac.md)，您可以刪除或讀取金鑰，但是您無法使用使用者定義的密碼取代金鑰，也無法使用 Active Directory 作為存取搜尋作業的主要驗證方法。 

存取搜尋服務的金鑰有兩種類型：管理 (讀寫) 和查詢 (唯讀)。

|Key|說明|限制|  
|---------|-----------------|------------|  
|Admin|授與所有作業的完整權限，包括能夠管理服務、建立和刪除索引、索引子及資料來源。<br /><br /> 當服務建立時，在入口網站中會產生兩個系統管理金鑰 (稱為「主要」和「次要」金鑰)，而且您可以視需要個別重新產生這些金鑰。 擁有兩個金鑰可讓您在變換一個金鑰時，使用第二個金鑰來繼續存取服務。<br /><br /> 指定管理金鑰時，只能在 HTTP 要求標頭中指定。 您無法將管理 API 金鑰放在 URL 中。|每個服務的上限為 2 個|  
|查詢|授與索引和文件的唯讀存取權，且通常會分派給發出搜尋要求的用戶端應用程式。<br /><br /> 查詢金鑰是視需要建立的。 您可以在入口網站中手動建立這些金鑰，或是透過[管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/) \(英文\) 以程式設計方式建立這些金鑰。<br /><br /> 您可以在 HTTP 要求標頭中指定查詢金鑰，以進行查詢、建議或查閱作業。 或者，您也可以在 URL 上將查詢金鑰當作參數來傳遞。 視您用戶端應用程式制定要求的方式而定，將金鑰當作查詢參數來傳遞可能會較為簡單：<br /><br /> `GET /indexes/hotels/docs?search=*&$orderby=lastRenovationDate desc&api-version=2017-11-11&api-key=[query key]`|每個服務 50 個|  

 管理金鑰或查詢金鑰在外觀上並無差別。 兩種金鑰都是由 32 個隨機產生的英數字元所組成。 如果您忘記在應用程式中指定的是哪種類型的金鑰，您可以[在入口網站中查看金鑰值](https://portal.azure.com)，或使用 [REST API](https://docs.microsoft.com/rest/api/searchmanagement/) 來傳回值和金鑰類型。  

> [!NOTE]  
>  在要求 URI 中傳遞敏感性資料 (例如 `api-key`) 被視為不佳的安全性做法。 基於這個理由，「Azure 搜尋服務」在查詢字串中只接受以查詢金鑰作為 `api-key` ，而且除非您應該公開提供索引內容，否則應避免採用此做法。 一般規則是建議將 `api-key` 當作要求標頭來傳遞。  

## <a name="find-api-keys-for-your-service"></a>尋找您的服務 API 金鑰

您可以在入口網站中或透過[管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/) \(英文\) 取得存取金鑰。 如需詳細資訊，請參閱[管理和查詢 API 金鑰](search-security-api-keys.md)。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
2. 列出您訂用帳戶的[搜尋服務](https://portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices)。
3. 選取服務，然後在服務頁面上尋找 [設定] >[金鑰]，以檢視管理和查詢金鑰。

![入口網站頁面 > [設定] > [金鑰] 區段](media/search-security-overview/settings-keys.png)

## <a name="regenerate-admin-keys"></a>重新產生系統管理金鑰

針對每個服務，會建立兩個系統管理金鑰，讓您可以變換主要金鑰，使用次要金鑰繼續存取。

如果您同時重新產生主要和次要金鑰，使用任一金鑰來存取服務作業的任何應用程式將無法再存取服務。

1. 在 [設定] >[金鑰] 頁面上，複製次要金鑰。
2. 針對所有的應用程式，更新 API 金鑰設定以使用次要金鑰。
3. 重新產生主要金鑰。
4. 更新所有應用程式以使用新的主要金鑰。

## <a name="secure-api-keys"></a>保護 API 金鑰
藉由限制透過入口網站或 Resource Manager 介面 (PowerShell 或命令列介面) 的存取來確保金鑰安全性。 如前所述，訂用帳戶系統管理員可以檢視及重新產生所有的 API 金鑰。 為以防萬一，請檢閱角色指派以了解誰具有管理員金鑰存取權。

+ 在服務儀表板中，按一下 [存取控制 (IAM)] 以檢視您服務的角色指派。

下列角色的成員可以檢視及重新產生金鑰：擁有者、參與者、[搜尋服務參與者](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#search-service-contributor)

> [!Note]
> 針對搜尋結果的身分識別型存取，您可以建立安全性篩選，依身分識別修剪結果、移除要求者不應具備存取權的文件。 如需詳細資訊，請參閱[安全性篩選](search-security-trimming-for-azure-search.md)和[使用 Active Directory 保護安全](search-security-trimming-for-azure-search-with-aad.md)。

## <a name="see-also"></a>另請參閱

+ [Azure 搜尋服務中的角色型存取控制](search-security-rbac.md)
+ [使用 Powershell 管理](search-manage-powershell.md) 
+ [效能與最佳化發行項](search-performance-optimization.md)