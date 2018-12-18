---
title: 在 Azure 資訊安全中心 (預覽) 自訂 OS 安全性設定 | Microsoft Docs
description: 本文示範如何自訂資訊安全中心評估
services: security-center
documentationcenter: na
author: rkarlin
manager: MBaldwin
editor: ''
ms.assetid: ''
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 18/30/2018
ms.author: rkarlin
ms.openlocfilehash: 08174a6781772abdebd9e203a3433a1a4ac82859
ms.sourcegitcommit: 5a9be113868c29ec9e81fd3549c54a71db3cec31
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/11/2018
ms.locfileid: "44378358"
---
# <a name="customize-os-security-configurations-in-azure-security-center-preview"></a>在 Azure 資訊安全中心 (預覽) 自訂 OS 安全性設定

本逐步解說示範如何在 Azure 資訊安全中心自訂 OS 安全性設定評估。

## <a name="what-are-os-security-configurations"></a>什麼是 OS 安全性設定？

Azure 資訊安全中心藉由套用一組[具有超過 150 個建議規則的集合](https://gallery.technet.microsoft.com/Azure-Security-Center-a789e335) \(英文\)，來監視安全性設定以強化 OS，其中包含與防火牆、稽核及密碼原則等相關的規則。 如果發現電腦設定有弱點，資訊安全中心會產生安全性建議。

藉由自訂規則，組織可以控制哪些設定選項更適合其環境。 您可以設定自訂的評估原則，然後將它套用到訂用帳戶中所有適用的電腦上。

> [!NOTE]
> - 目前，OS 安全性設定自訂僅適用於 Windows Server 2008、2008 R2、2012 及 2012 R2 作業系統。
> - 設定會套用到選取的訂用帳戶下連線到所有工作區的所有 VM 和電腦。
> - OS 安全性設定自訂僅適用於資訊安全中心標準層。
>
>

您可以透過啟用和停用特定規則、變更現有規則所需的設定或根據支援的規則類型 (登錄、稽核原則和安全性原則) 加入新規則，以自訂 OS 安全性設定規則。 目前，所需的設定必須是確切值。

新規則和同類型的其他現有規則必須是相同的格式和結構。

> [!NOTE]
> 若要自訂 OS 安全性設定，您必須獲指派「訂用帳戶擁有者」、「訂用帳戶參與者」或「安全性系統管理員」角色。
>
>

## <a name="customize-the-default-os-security-configuration"></a>自訂預設的 OS 安全性設定

若要自訂資訊安全中心的預設 OS 安全性設定，請執行下列作業：

1.  開啟 [資訊安全中心] 儀表板。

2.  在左窗格中，選取 [安全性原則]。      

    ![[安全性原則] 清單](media/security-center-customize-os-security-config/manual-provision.png)

3.  在您要自訂的訂用帳戶資料列中，按一下 [編輯設定]。

4. 選取 [編輯安全性組態]。  
    
    ![[編輯安全性設定] 視窗](media/security-center-customize-os-security-config/blade.png)

5. 依照下列步驟下載、編輯和上傳已修改的檔案。

   > [!NOTE]
   > 根據預設，您下載的設定檔為 *json* 格式。 如需修改此檔案的指示，請移至[自訂設定檔](#customize-the-configuration-file)。
   >

6. 若要認可變更，請選取 [儲存]。 否則不會儲存原則。

    ![[儲存] 按鈕](media/security-center-customize-os-security-config/save-successfully.png)

   成功儲存檔案後，設定就會套用到訂用帳戶下連線到工作區的所有 VM 和電腦。 此程序通常需要數分鐘，但可能會更久，這取決於基礎結構大小。

在任何時間點，您都可以將目前的原則設定重設為其預設狀態。 若要這樣做，在 [編輯 OS 安全性設定規則] 視窗中，選取 [重設]。 在確認快顯視窗中，選取 [是] 以確認此選項。

![[重設確認] 視窗](media/security-center-customize-os-security-config/edit-alert.png)

## <a name="customize-the-configuration-file"></a>自訂設定檔

在自訂檔案中，每個支援的 OS 版本都有一組規則或規則集。 每個規則集都有自己的名稱和唯一識別碼，如下列範例所示：

![規則集名稱和識別碼](media/security-center-customize-os-security-config/custom-file.png)

> [!NOTE]
> 這個範例檔案是在 Visual Studio 中編輯，但如果您有安裝 JSON 檢視器外掛程式，您也可以使用 [記事本]。
>
>

當您編輯自訂檔案時，您可以修改一個規則或所有規則。 每個規則集均包含一個「規則」區段，它分為三個類別：登錄、稽核原則和安全性原則，如下所示：

![三個規則集類別](media/security-center-customize-os-security-config/rules-section.png)

每個類別都有自己的屬性集。 您可以變更下列屬性：

- **expectedValue**：這個屬性的欄位資料類型必須符合每個「規則類型」支援的值，例如：

  - **baselineRegistryRules**：此值應符合該規則中定義的 [regValueType](https://msdn.microsoft.com/library/windows/desktop/ms724884)。

  - **baselineAuditPolicyRules**：使用下列其中一個字串值：

    - 「成功和失敗」

    - 「成功」

  - **baselineSecurityPolicyRules**：使用下列其中一個字串值：

    - 「沒有人」

    - 允許的使用者群組清單，例如：「系統管理員」、「備份操作員」

-   **state**：字串可包含「停用」 或「啟用」選項。 在此私人預覽版本中，字串區分大小寫。

只有這些是可設定的欄位。 如果您違反檔案格式或大小，您將無法儲存變更。 您會收到錯誤，指出您必須上傳有效的 JSON 組態檔。

如需其他可能錯誤的清單，請參閱[錯誤碼](#error-codes)。

下列三個區段包含前述規則的範例。 *expectedValue* 和 *state* 屬性可以變更。

**baselineRegistryRules**
```
    {
    "hive": "LocalMachine",
    "regValueType": "Int",
    "keyPath":
    "System\\\\CurrentControlSet\\\\Services\\\\LanManServer\\\\Parameters",
    "valueName": "restrictnullsessaccess",
    "ruleId": "f9020046-6340-451d-9548-3c45d765d06d",
    "originalId": "0f319931-aa36-4313-9320-86311c0fa623",
    "cceId": "CCE-10940-5",
    "ruleName": "Network access: Restrict anonymous access to Named Pipes and
    Shares",
    "ruleType": "Registry",
    "expectedValue": "1",
    "severity": "Warning",
    "analyzeOperation": "Equals",
    "source": "Microsoft",
    "state": "Disabled"

    }
```

**baselineAuditPolicyRules**
```
    {
    "auditPolicyId": "0cce923a-69ae-11d9-bed3-505054503030",
    "ruleId": "37745508-95fb-44ec-ab0f-644ec0b16995",
    "originalId": "2ea0de1a-c71d-46c8-8350-a7dd4d447895",
    "cceId": "CCE-11001-5",
    "ruleName": "Audit Policy: Account Management: Other Account Management Events",
    "ruleType": "AuditPolicy",
    "expectedValue": "Success and Failure",
    "severity": "Critical",
    "analyzeOperation": "Equals",
    "source": "Microsoft",
    "state": "Enabled"
    }
```

**baselineSecurityPolicyRules**
```
    {
    "sectionName": "Privilege Rights",
    "settingName": "SeIncreaseWorkingSetPrivilege",
    "ruleId": "b0ec9d5e-916f-4356-83aa-c23522102b33",
    "originalId": "b61bd492-74b0-40f3-909d-36b9bf54e94c",
    "cceId": "CCE-10548-6",
    "ruleName": "Increase a process working set",
    "ruleType": "SecurityPolicy",
    "expectedValue": "Administrators, Local Service",
    "severity": "Warning",
    "analyzeOperation": "Equals",
    "source": "Microsoft",
    "state": "Enabled"
    }
```

不同的 OS 類型可能有些規則會重複。 重複的規則有相同的 *originalId* 屬性。

## <a name="create-custom-rules"></a>建立自訂規則

您也可以建立新規則。 建立新規則之前，請記住下列限制：

-   結構描述版本 *baselineId* 和 *baselineName* 無法變更。

-   無法移除規則集。

-   無法加入規則集。

-   允許的規則數目上限 (包括預設規則) 為 1000。

新的自訂規則會標示新的自訂來源 (!= "Microsoft")。 *ruleId* 欄位可以是 Null 或空白。 如果空白，Microsoft 會產生一個。 如果非空白，它必須有一個在所有規則 (預設和自訂) 中是唯一的有效 GUID。 檢閱核心欄位的下列限制：

-   **originalId**：可以是 Null 或空白。 如果 *originalId* 非空白，則應為有效的 GUID。

-   **cceId**：可以是 Null 或空白。 如果 *cceId* 非空白，則必須是唯一的。

-   **ruleType**：(擇一) 登錄、稽核原則和安全性原則。

-   **嚴重性**：(擇一) 不明、重大、警告或資訊。

-   **analyzeOperation**：必須是「等於」。

-   **auditPolicyId**：必須是有效的 GUID。

-   **regValueType**：(擇一) 整數、長、字串或多個字串。

> [!NOTE]
> Hive 必須是 *LocalMachine*。
>
>

新自訂規則的範例：

**登錄**：
```
    {
    "hive": "LocalMachine",
    "regValueType": "Int",
    "keyPath":
    "System\\\\CurrentControlSet\\\\Services\\\\Netlogon\\\\MyKeyName",
    "valueName": "MyValueName",
    "originalId": "",
    "cceId": "",
    "ruleName": "My new registry rule”, "baselineRuleType": "Registry",
    "expectedValue": "123", "severity": "Critical",
    "analyzeOperation": "Equals",
    "source": "MyCustomSource",
    "state": "Enabled"
    }
```
**安全性原則**：
```
   {
   "sectionName": "Privilege Rights",
   "settingName": "SeDenyBatchLogonRight",
   "originalId": "",
   "cceId": "",
   "ruleName": "My new security policy rule", "baselineRuleType":
   "SecurityPolicy",
   "expectedValue": "Guests",
   "severity": "Critical",
   "analyzeOperation": "Equals", "source": " MyCustomSource ",
   "state": "Enabled"
   }
```
**稽核原則**：
```
   {
   "auditPolicyId": "0cce923a-69ae-11d9-bed3-505054503030",
   "originalId": "",
   "cceId": "",
   "ruleName": " My new audit policy rule ", "baselineRuleType": "AuditPolicy",
   "expectedValue": " Failure",
   "severity": "Critical",
   "analyzeOperation": "Equals", "source": " MyCustomSource ",
   "state": "Enabled"
   }
```

## <a name="file-upload-failures"></a>檔案上傳失敗

如果提交的組態檔因為值或格式錯誤而無效，則會顯示失敗錯誤，例如**儲存動作失敗**。 您可以下載詳細的錯誤 .csv 報告來修復並修正錯誤，然後重新提交已更正的設定檔。

錯誤檔案的範例：

![錯誤檔案範例](media/security-center-customize-os-security-config/errors-file.png)

## <a name="error-codes"></a>錯誤碼

下表列出所有可能的錯誤：

| **錯誤**                                | **說明**                                                                                                                              |
|------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| BaselineConfiguratiohSchemaVersionError  | 屬性 *schemaVersion* 無效或空白。 此值必須設定為 *{0}*。                                                         |
| BaselineInvalidStringError               | 屬性 *{0}* 不可包含 *\\n*。                                                                                                         |
| BaselineNullRuleError                    | 基準設定規則清單包含具有值 *null* 的規則。                                                                         |
| BaselineRuleCceIdNotUniqueError          | CCE 識別碼 *{0}* 不是唯一的。                                                                                                                  |
| BaselineRuleEmptyProperty                | 屬性 *{0}* 遺失或無效。                                                                                                       |
| BaselineRuleIdNotInDefault               | 規則具有來源屬性 *Microsoft*，但在 Microsoft 預設規則集中找不到它。                                                   |
| BaselineRuleIdNotUniqueError             | 規則識別碼不是唯一的。                                                                                                                       |
| BaselineRuleInvalidGuid                  | 屬性 *{0}* 無效。 值不是有效的 GUID。                                                                             |
| BaselineRuleInvalidHive                  | Hive 必須是 LocalMachine。                                                                                                                   |
| BaselineRuleNameNotUniqueError           | 規則名稱不是唯一的。                                                                                                                 |
| BaselineRuleNotExistInConfigError        | 在新的設定中找不到規則。 無法刪除規則。                                                                     |
| BaselineRuleNotFoundError                | 在預設基準規則集中找不到規則。                                                                                        |
| BaselineRuleNotInPlace                   | 規則會比對類型為 {0} 的預設規則，並列在 {1} 清單中。                                                                       |
| BaselineRulePropertyTooLong              | 屬性 *{0}* 太長。 允許的最大長度：{1}。                                                                                        |
| BaselineRuleRegTypeInvalidError          | 預期值 *{0}* 不符合定義的登錄值類型。                                                              |
| BaselineRulesetAdded                     | 在預設設定中找不到識別碼為 *{0}* 的規則集。 無法加入規則集。                                               |
| BaselineRulesetIdMustBeUnique            | 指定的基準規則集 *{0}* 必須是唯一的。                                                                                           |
| BaselineRulesetNotFound                  | 在指定的組態中找不到識別碼為 *{0}* 且名稱為 *{1}* 的規則集。 無法刪除規則集。                                |
| BaselineRuleSourceNotMatch               | 已定義識別碼為 *{0}* 的規則。                                                                                                       |
| BaselineRuleTypeDoesntMatch              | 預設規則類型為 *{0}*。                                                                                                              |
| BaselineRuleTypeDoesntMatchError         | 規則的實際類型為 *{0}*，但 *ruleType* 屬性為 *{1}*。                                                                          |
| BaselineRuleUnpermittedChangesError      | 只允許變更 *expectedValue* 和 *state* 屬性。                                                                       |
| BaselineTooManyRules                     | 允許的自訂規則數目上限為 {0} 個規則。 指定的組態包含 {1} 個規則、{2} 個預設規則和 {3} 個自訂規則。 |
| ErrorNoConfigurationStatus               | 找不到設定狀態。 說明所需的設定狀態：「預設」或「自訂」。                                    |
| ErrorNonEmptyRulesetOnDefault            | 設定狀態會設為預設。 *BaselineRulesets* 清單必須是 Null 或空白。                                                          |
| ErrorNullRulesetsPropertyOnCustom        | 指定的設定狀態為「自訂」，但 *baselineRulesets* 屬性為 Null 或空白。                                             |
| ErrorParsingBaselineConfig               | 指定的組態無效。 一或多個定義的值含有 Null 值或無效類型。                                  |
| ErrorParsingIsDefaultProperty            | 指定的 *configurationStatus* 值 *{0}* 無效。 值只可以是「預設」或「自訂」。                                         |
| InCompatibleViewVersion                  | 檢視版本 *{0}* 在此工作區類型中「不」受支援。                                                                                   |
| InvalidBaselineConfigurationGeneralError | 指定的基準設定有一或多個類型驗證錯誤。                                                          |
| ViewConversionError                      | 檢視的版本較工作區支援的版本舊。 檢視轉換失敗：{0}。                                                                 |

如果您沒有足夠權限，您可能會收到一般失敗錯誤，如下所示：

![「儲存動作失敗」錯誤訊息](media/security-center-customize-os-security-config/general-failure-error.png)

## <a name="next-steps"></a>後續步驟
本逐步解說示範如何在資訊安全中心自訂 OS 安全性設定評估。 若要了解設定規則及修復的詳細資訊，請參閱：

- [資訊安全中心一般設定識別碼與基準規則](https://gallery.technet.microsoft.com/Azure-Security-Center-a789e335)。
- 資訊安全中心會使用一般設定列舉 (CCE) 將唯一識別碼指派給設定規則。 如需詳細資訊，請參閱 [CCE](https://nvd.nist.gov/config/cce/index)。
- 如果要解決 OS 設定不符合建議的安全性設定規則時的弱點，請參閱[修復安全性設定](security-center-remediate-os-vulnerabilities.md)。
