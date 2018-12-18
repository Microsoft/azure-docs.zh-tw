---
title: Azure 身分識別開發人員的支援和協助選項 | Microsoft Docs
description: 了解如何在建立應用程式以與 Microsoft Azure 身分識別 (Azure Active Directory 和 MSA) 整合時，取得開發相關疑難和問題的協助和支援
services: active-directory
documentationcenter: dev-center-name
author: CelesteDG
manager: mtillman
editor: ''
ms.assetid: 820acdb7-d316-4c3b-8de9-79df48ba3b06
ms.service: active-directory
ms.component: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/27/2017
ms.author: celested
ms.reviewer: andret
ms.custom: aaddev
ms.openlocfilehash: 9b2ccce4ef6fad4f7710ba2092d67ac5f4e73b71
ms.sourcegitcommit: 1f0587f29dc1e5aef1502f4f15d5a2079d7683e9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/07/2018
ms.locfileid: "39601262"
---
# <a name="support-and-help-options-for-developers"></a>開發人員的支援和協助選項 

不論您是剛開始整合 Azure Active Directory、Microsoft 身分識別或 Microsoft Graph API，還是在您將新功能實作到應用程式時，您有時會需要從社群取得協助，或是需要了解您身為開發人員所擁有的支援選項。 本文會協助您了解這些選項，以下是本文的摘要：

> [!div class="checklist"]
> * 進行搜尋，以查看社群是否尚未回答您的問題，或您嘗試實作的功能是否已有現存文件
> * 在某些情況下，您只想使用我們的支援工具來協助您進行特定問題的偵錯
> * 如果您找不到所需的解答，就可以在「Stack Overflow」上發問
> * 如果您發現我們的驗證程式庫有問題，請提出 GitHub 問題
> * 最後，如果您需要與人對談，您可以開啟支援要求


## <a name="search"></a>Search

如果您有開發相關問題，您可以在我們的文件、[github 範例](https://github.com/azure-samples)，或 [Stack Overflow](https://www.stackoverflow.com) 問題的解答中，找到您需要的答案。

### <a name="scoped-search"></a>限域搜尋
為了更快獲得結果，請在您[慣用的搜尋引擎](https://bing.com)中使用下列語法，將搜尋範圍限制在 Stack Overflow、我們的文件和我們的程式碼範例：
```
{Your Search Terms} (site:stackoverflow.com OR site:docs.microsoft.com OR site:github.com/azure-samples OR site:cloudidentity.com OR site:developer.microsoft.com/en-us/graph)
```
其中，*{Your Search Terms}* 是您的搜尋關鍵字。
<br/>

## <a name="use-our-development-support-tools"></a>使用我們的開發支援工具

|工具  |說明  |
|---------|---------|
|[jwt.ms](https://jwt.ms)| 貼上識別碼或存取權杖以將宣告名稱和值解碼 |
|[錯誤碼分析器](https://apps.dev.microsoft.com/portal/tools/errors)| 貼上您在登入期間或同意頁面所收到的錯誤碼，以查看可能的原因和補救方法 |
|[Microsoft Graph 總管](https://developer.microsoft.com/graph/graph-explorer)| 此工具可讓您針對 Microsoft Graph API 提出要求並查看回應|

<br/>

[![Stack Overflow](./media/developer-support-help-options/stackoverflow-logo.png)](https://www.stackoverflow.com)
## <a name="post-a-question-to-stack-overflow"></a>在 Stack Overflow 張貼問題

Stack Overflow 是開發相關問題的慣用管道，該社群的成員和 Microsoft 小組成員會直接參與其中，協助您解決問題。

如果您無法透過搜尋找到問題的解答，請在 Stack Overflow 提交新的問題：請在提出問題時使用下列其中一個標籤，以協助社群識別問題，然後及時對您的問題做出解答：

|元件/區域  |標記  |
|---------|---------|
|ADAL 程式庫 |[[adal]](http://stackoverflow.com/questions/tagged/adal)|
|MSAL 程式庫     |[[msal]](http://stackoverflow.com/questions/tagged/msal)|
|OWIN 中介軟體  |[[azure-active-directory]](http://stackoverflow.com/questions/tagged/azure-active-directory)|
|[Azure B2B](https://docs.microsoft.com/azure/active-directory/active-directory-b2b-what-is-azure-ad-b2b)  |[[azure-ad-b2b]](http://stackoverflow.com/questions/tagged/azure-ad-b2b)|
|[Azure B2C](https://azure.microsoft.com/services/active-directory-b2c/)  |[[azure-ad-b2c]](http://stackoverflow.com/questions/tagged/azure-ad-b2b)|
|[Microsoft Graph API](https://developer.microsoft.com/graph/) |[[microsoft-graph]](http://stackoverflow.com/questions/tagged/microsoft-graph)
|與驗證或授權主題相關的其他任何區域 |[[azure-active-directory]](http://stackoverflow.com/questions/tagged/azure-active-directory)
<br/>
> [!TIP]
> 下面的 Stack Overflow 文章包含如何提出問題以及如何新增原始程式碼的訣竅，遵循這些指導方針，會有助於增加社群成員盡速評估問題並做出回應的機率：  
> - [要如何問一個好問題](https://stackoverflow.com/help/how-to-ask)
> - [如何建立最小、完成且可驗證的範例](https://stackoverflow.com/help/mcve)

<br/>


[![Stack Overflow](./media/developer-support-help-options/github-logo.png)](https://www.github.com)
## <a name="create-a-github-issue"></a>建立 GitHub 問題

 如果您發現與我們的程式庫有關的錯誤或問題，請在我們的 GitHub 存放庫上提出問題。 我們的程式庫屬於開放原始碼，因此您也可以自由地提交提取要求。 下面的文章中有一份程式庫及其 GitHub 存放庫的清單：

- [ADAL、MSAL 和 Owin 中介軟體](active-directory-authentication-libraries.md)程式庫以及 GitHub 存放庫

<br/>

## <a name="open-a-support-request"></a>開啟支援要求

如果您需要與人對談，您可以開啟支援要求。 如果您是 Azure 的客戶，則有幾個支援選項可供您使用。 若要比較方案，請參閱[本頁](https://azure.microsoft.com/support/plans/)。 Azure 的客戶也可以使用開發人員支援。 如需如何購買開發人員支援方案的相關資訊，請參閱[本頁](https://azure.microsoft.com/support/plans/developer/)。

- 如果您已經有 Azure 支援方案，請[在此開啟支援要求](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)

- 如果您不是 Azure 的客戶，您也可以透過[我們的商業支援](https://support.microsoft.com/en-us/gp/contactus81?Audience=Commercial)，向 Microsoft 開啟支援要求。

您也可以嘗試[我們的虛擬專員](https://support.microsoft.com/contactus/?ws=support)，以取得支援或提出問題。

### <a name="free-chat-support-for-a-limited-time"></a>限時的免費交談支援

您也可以使用我們的交談支援，Microsoft 合作夥伴可於有限時間內免費使用。 如果貴公司不是 Microsoft 合作夥伴，您可以在[這裡](https://partners.microsoft.com/PartnerProgram/simplifiedenrollment.aspx)免費註冊此功能，並獲得其他好處。

為貴公司註冊後，您就可以在[這裡](https://aka.ms/devchat)啟動交談要求。