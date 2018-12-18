---
title: 自訂 Azure AD 屬性對應 | Microsoft Docs
description: 了解 Azure Active Directory 中 SaaS 應用程式有哪些屬性對應，以及如何修改屬性對應來應付業務需求。
services: active-directory
documentationcenter: ''
author: barbkess
manager: mtillman
editor: ''
ms.service: active-directory
ms.component: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/09/2018
ms.author: barbkess
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c54df9c4a01dbdb7713d03ff81108f28a60ec727
ms.sourcegitcommit: af9cb4c4d9aaa1fbe4901af4fc3e49ef2c4e8d5e
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44355210"
---
# <a name="customizing-user-provisioning-attribute-mappings-for-saas-applications-in-azure-active-directory"></a>在 Azure Active Directory 中自訂 SaaS 應用程式的使用者佈建屬性對應
Microsoft Azure AD 支援使用者佈建到例如 Salesforce、Google Apps 等等的協力廠商 SaaS 應用程式。 如果您啟用了第三方 SaaS 應用程式的使用者佈建，Azure 入口網站會以「屬性對應」的形式控制其屬性值。

在 Azure AD 使用者物件和每個 SaaS 應用程式的使用者物件之間，有一組預先設定的屬性和屬性對應。 有些應用程式除了使用者以外，還會管理其他類型的物件，例如群組。 <br> 
 您可以根據您的業務需求自訂預設的屬性對應。 這表示您可以變更或刪除現有的屬性對應，或建立新的屬性對應。
 
## <a name="editing-user-attribute-mappings"></a>編輯使用者屬性對應

在 Azure AD 入口網站中，您可以存取這項功能，方法是在 [企業應用程式] 的 [管理] 區段中，按一下 [佈建] 底下的 [對應] 設定。


![Salesforce](./media/customize-application-attributes/21.png) 

按一下 [對應] 組態，可開啟相關的 [屬性對應] 畫面。 SaaS 應用程式需要有幾個屬性對應才能正確運作。 若為必要的屬性，[刪除] 功能就無法使用。


![Salesforce](./media/customize-application-attributes/22.png)

在上述範例中，您可以看到 Salesforce 中受控物件的 **Username** 屬性填入了連結 Azure Active Directory 物件的 **userPrincipalName** 值。

您可以按一下對應來自訂現有的 [屬性對應]。 這會開啟 [編輯屬性] 畫面。

![Salesforce](./media/customize-application-attributes/23.png)


### <a name="understanding-attribute-mapping-types"></a>了解屬性對應類型
透過屬性對應，您將可控制屬性在第三方 SaaS 應用程式中填入的方式。 支援四種不同的對應類型：

* **直接** - 目標屬性會填入 Azure AD 中連結物件的屬性值。
* **常數** - 目標屬性會填入您所指定的特定字串。
* **運算式** - 目標屬性會根據類似指令碼的運算式結果填入。 
  如需詳細資訊，請參閱[在 Azure Active Directory 中撰寫屬性對應的運算式](functions-for-customizing-application-data.md)。
* **無** - 目標屬性保留未修改。 不過，如果目標屬性是空的，就會填入您所指定的預設值。

除了這四個基本類型以外，自訂屬性對應還支援選擇性的**預設**值指派的概念。 預設值指派可確保當 Azure AD 中和目標物件都沒有值時，目標屬性會填入某個值。 最常見的設定是將其保留空白。


### <a name="understanding-attribute-mapping-properties"></a>了解屬性對應屬性

在上一節中，我們已為您介紹屬性對應類型屬性。
除了這個屬性之外，屬性對應也支援下列屬性：

- **來源屬性** - 來源系統的使用者屬性 (例如：Azure Active Directory)。
- **目標屬性** – 目標系統中的使用者屬性 (例如：ServiceNow)。
- **使用此屬性比對物件** – 是否應該將此對應用於唯一識別來源與目標系統之間的使用者。 這通常會在 Azure AD 中的 userPrincipalName 或 mail 屬性上加以設定，通常會對應至目標應用程式中的使用者名稱欄位。
- **比對優先順序** – 您可以設定多個比對屬性。 具有多個屬性時，系統會以此欄位定義的順序進行評估。 只要找到相符項目，便不會評估進一步比對屬性。
- **套用此對應**
    - **一律** – 將此對應套用於使用者建立和更新動作
    - **僅限建立期間** - 僅將此對應套用於使用者建立動作


## <a name="editing-group-attribute-mappings"></a>編輯群組屬性對應

特定應用程式 (例如 ServiceNow、Box 和 Google Apps) 除了使用者物件以外，也支援佈建群組物件的能力。 除了群組成員以外，群組物件也可包含群組屬性，例如顯示名稱和電子郵件別名。

![ServiceNow](./media/customize-application-attributes/24.png)

佈建群組可以選擇性地啟用或停用，方法是在 [對應] 下選取群組對應，然後在 [屬性對應] 畫面中將所需的選項設為 [已啟用]。

佈建為群組物件一部分的屬性，可依照與使用者物件相同的方式進行自訂，如前所述。 

>[!TIP]
>佈建群組物件 (屬性和成員) [與將群組指派](assign-user-or-group-access-portal.md)給應用程式是不同的概念。 您可以將群組指派給應用程式，但只能佈建群組中所包含的使用者物件。 要在指派中使用群組，並不需要佈建整個群組的物件。


## <a name="editing-the-list-of-supported-attributes"></a>編輯支援的屬性清單

系統會預先設定支援指定應用程式的使用者屬性。 大部分應用程式的使用者管理 API 都不支援結構描述探索，因此 Azure AD 佈建服務無法藉由呼叫應用程式動態產生支援的屬性清單。 

不過，某些應用程式可支援自訂屬性。 若要讓 Azure AD 佈建服務能夠讀取和寫入自訂屬性，必須使用 [屬性對應] 畫面底部的 [顯示進階選項] 核取方塊，在 Azure 入口網站中輸入這些屬性的定義。

支援屬性清單自訂的應用程式和系統包括：

* Salesforce
* ServiceNow
* Workday
* Azure Active Directory (支援 [Azure AD Graph API 預設屬性](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#user-entity)和自訂目錄擴充功能)
* 支援 [SCIM 2.0](https://tools.ietf.org/html/rfc7643) 的應用程式，定義於[核心結構描述](https://tools.ietf.org/html/rfc7643)中的屬性必須新增至此處

>[!NOTE]
>我們建議，只有已自訂應用程式和系統的結構描述，並確知其自訂屬性如何定義的系統管理員，才可編輯支援的屬性清單。 為此，有時候必須要熟悉應用程式或系統所提供的 API 和開發人員工具。 

![編輯器](./media/customize-application-attributes/25.png) 

編輯支援的屬性清單時，必須提供下列屬性：

* **名稱** - 屬性的系統名稱，如目標物件的結構描述所定義。 
* **類型** - 屬性儲存的資料類型，如目標物件的結構描述所定義。 這可以是下列其中一項：
   * *二進位* - 屬性包含二進位資料。
   * *布林值* - 屬性包含 True 或 False 值。
   * *日期時間* - 屬性包含日期字串。
   * *整數* - 屬性包含整數。
   * *參考* - 屬性包含對目標應用程式中的另一個資料表所儲存的值進行參考的識別碼。
   * *字串* - 屬性包含文字字串。 
* **是主索引鍵嗎？** - 屬性是否定義為目標物件結構描述中的主索引鍵欄位。
* **必要？** - 屬性是否必須填入目標應用程式或系統中。
* **是多重值嗎？** - 屬性是否支援多個值。
* **大小寫完全相符嗎？** - 是否以區分大小寫的方式評估屬性值。
* **API 運算式** - 不使用，除非特定佈建連接器 (例如 Workday) 的文件另有指示。
* **參考的物件屬性** - 如果這是參考類型屬性，則此功能表可讓您在目標應用程式中選取包含屬性相關值的資料表和屬性。 例如，如果您有名為 "Department" 的屬性，且其儲存值參考了個別 "Departments" 資料表中的物件，則您會選取 "Departments.Name"。 請注意，支援指定應用程式的參考資料表和主要識別碼欄位都是預先設定的，且目前無法使用 Azure 入口網站加以編輯，但可以使用[圖形 API](https://developer.microsoft.com/graph/docs/api-reference/beta/resources/synchronization-configure-with-custom-target-attributes) 來編輯。

若要新增新的屬性，請捲動至支援的屬性清單結尾處，使用提供的輸入填入上方的欄位，然後選取 [新增屬性]。 新增屬性完成後，請選取 [儲存]。 接著，您必須重新載入 [佈建] 索引標籤，使新屬性成為屬性對應編輯器中的可用屬性。

## <a name="restoring-the-default-attributes-and-attribute-mappings"></a>還原預設屬性和屬性對應

如果您要重新開始，並將現有的對應重設為其預設狀態，您可以選取 [還原預設對應] 核取方塊，並儲存設定。 這會比照應用程式已從應用程式庫新增至 Azure AD 租用戶的情形，來設定所有對應。 

選取此選項，可在佈建服務執行期間有效地強制重新同步處理所有的使用者。 

>[!IMPORTANT]
>強烈建議先將 [佈建狀態] 設為 [關閉]，再叫用此選項。


## <a name="what-you-should-know"></a>您應該知道的事情

* Microsoft Azure AD 提供有效率的同步處理程序實作。 在初始化環境中，同步處理期間只會處理需要更新的物件。 

* 更新屬性對應會影響同步處理期間的效能。 更新屬性對應組態需要重新評估所有受控物件。 

* 建議您最好不要經常變更屬性對應。


## <a name="next-steps"></a>後續步驟

* [自動化 SaaS 應用程式使用者佈建/解除佈建](user-provisioning.md)
* [撰寫屬性對應的運算式](functions-for-customizing-application-data.md)
* [適用於使用者佈建的範圍篩選器](define-conditional-rules-for-provisioning-user-accounts.md)
* [使用 SCIM 以啟用從 Azure Active Directory 到應用程式的使用者和群組自動佈建](use-scim-to-provision-users-and-groups.md)
* [如何整合 SaaS 應用程式的教學課程清單](../saas-apps/tutorial-list.md)


