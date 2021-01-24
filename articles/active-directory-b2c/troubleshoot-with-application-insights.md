---
title: 使用 Application Insights 針對自訂原則進行疑難排解
titleSuffix: Azure AD B2C
description: 如何設定 Application Insights 以追蹤自訂原則的執行。
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: troubleshooting
ms.date: 10/16/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 33504487b6175023e18893812c533950305cb1d3
ms.sourcegitcommit: 4d48a54d0a3f772c01171719a9b80ee9c41c0c5d
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/24/2021
ms.locfileid: "98745997"
---
# <a name="collect-azure-active-directory-b2c-logs-with-application-insights"></a>使用 Application Insights 收集 Azure Active Directory B2C 記錄

本文提供從 Active Directory B2C (Azure AD B2C) 收集記錄的步驟，讓您可以診斷自訂原則的問題。 Application Insights 提供方法來診斷例外狀況和將應用程式效能問題視覺化。 Azure AD B2C 包含將資料傳送至 Application Insights 的功能。

此處所述的詳細活動記錄應該 **只** 在開發自訂原則期間啟用。

> [!WARNING]
> 請勿 `DeploymentMode` `Development` 在生產環境中將設定為。 記錄會收集所有與身分識別提供者一起傳送的宣告。 身為開發人員，您必須負責 Application Insights 記錄中收集的任何個人資料。 只有當原則是以 **開發人員模式** 放置時，才會收集這些詳細的記錄檔。

## <a name="set-up-application-insights"></a>設定 Application Insights

如果您還沒有帳戶，請在您的訂用帳戶中建立 Application Insights 的實例。

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 在頂端功能表中選取 [ **目錄 + 訂** 用帳戶] 篩選，然後選取包含您的 Azure 訂用帳戶的目錄， (不是您的 Azure AD B2C 目錄) 。
1. 選取左側導覽功能表中的 [ **建立資源** ]。
1. 搜尋並選取 [ **Application Insights**]，然後選取 [ **建立**]。
1. 完成表單，選取 [ **審核 + 建立**]，然後選取 [ **建立**]。
1. 完成部署之後，請選取 [ **移至資源**]。
1. 在 Application Insights] 功能表的 [ **設定** ] 底下，選取 [ **屬性**]。
1. 記錄要在後續步驟中使用的 **檢測金鑰** 。

## <a name="configure-the-custom-policy"></a>設定自訂原則

1. 開啟信賴憑證者 (RP) 檔案，例如 *SignUpOrSignin.xml*。
1. 將下列屬性新增至 `<TrustFrameworkPolicy>` 元素：

   ```xml
   DeploymentMode="Development"
   UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
   ```

1. 如果尚未存在，請將 `<UserJourneyBehaviors>` 子節點加入至 `<RelyingParty>` 節點。 它必須緊接在之後 `<DefaultUserJourney ReferenceId="UserJourney Id" from your extensions policy, or equivalent (for example:SignUpOrSigninWithAAD" />` 。
1. 新增下列節點作為 `<UserJourneyBehaviors>` 元素的子節點。 請務必將取代為 `{Your Application Insights Key}` 您稍早錄製的 Application Insights **檢測金鑰** 。

    ```xml
    <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
    ```

    * `DeveloperMode="true"` 告知 ApplicationInsights 透過處理管線加速遙測。 適用于開發，但限制于大量的磁片區。 在生產環境中，將設定 `DeveloperMode` 為 `false` 。
    * `ClientEnabled="true"` 傳送 ApplicationInsights 用戶端腳本，以追蹤頁面流覽和用戶端錯誤。 您可以在 Application Insights 入口網站的 **browserTimings** 資料表中加以查看。 藉由設定 `ClientEnabled= "true"` ，您可以將 Application Insights 新增至頁面腳本，並取得頁面載入和 ajax 呼叫、計數、瀏覽器例外狀況和 ajax 失敗詳細資料，以及使用者和會話計數的時間。 這個欄位是 **選擇性** 欄位，預設會設定為 `false` 。
    * `ServerEnabled="true"` 會將現有的 UserJourneyRecorder JSON 當作自訂事件傳送至 Application Insights。

    例如：

    ```xml
    <TrustFrameworkPolicy
      ...
      TenantId="fabrikamb2c.onmicrosoft.com"
      PolicyId="SignUpOrSignInWithAAD"
      DeploymentMode="Development"
      UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights"
    >
    ...
    <RelyingParty>
      <DefaultUserJourney ReferenceId="UserJourney ID from your extensions policy, or equivalent (for example: SignUpOrSigninWithAzureAD)" />
      <UserJourneyBehaviors>
        <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="true" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
      </UserJourneyBehaviors>
      ...
    </TrustFrameworkPolicy>
    ```

1. 上傳原則。

## <a name="see-the-logs-in-application-insights"></a>查看 Application Insights 中的記錄

在 Application Insights 中看到新記錄之前，會有短暫的延遲（通常不到五分鐘）。

1. 開啟您在 [Azure 入口網站](https://portal.azure.com)中建立的 Application Insights 資源。
1. 在 [ **總覽** ] 頁面上，選取 [ **記錄**]。
1. 在 Application Insights 入口網站中開啟新的索引標籤。

以下是您可以用來查看記錄的查詢清單：

| 查詢 | 描述 |
|---------------------|--------------------|
`traces` | 查看 Azure AD B2C 產生的所有記錄 |
`traces | where timestamp > ago(1d)` | 查看 Azure AD B2C 在最後一天產生的所有記錄

此項目可能很長。 請將它匯出至 CSV 以仔細查看。

如需查詢的詳細資訊，請參閱 [Azure 監視器中的記錄查詢總覽](../azure-monitor/log-query/log-query-overview.md)。

## <a name="configure-application-insights-in-production"></a>在生產環境中設定 Application Insights

若要改善生產環境的效能和更佳的使用者體驗，請務必設定您的原則，以忽略不重要的訊息。 使用下列設定，只將嚴重的錯誤訊息傳送到您的 Application Insights。 

1. 將 `DeploymentMode` [TrustFrameworkPolicy](trustframeworkpolicy.md) 的屬性設定為 `Production` 。 

   ```xml
   <TrustFrameworkPolicy xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="http://schemas.microsoft.com/online/cpim/schemas/2013/06" PolicySchemaVersion="0.3.0.0"
   TenantId="yourtenant.onmicrosoft.com"
   PolicyId="B2C_1A_signup_signin"
   PublicPolicyUri="http://yourtenant.onmicrosoft.com/B2C_1A_signup_signin"
   DeploymentMode="Production"
   UserJourneyRecorderEndpoint="urn:journeyrecorder:applicationinsights">
   ```

1. 將 `DeveloperMode` [JourneyInsights](relyingparty.md#journeyinsights) 的設定為 `false` 。

   ```xml
   <UserJourneyBehaviors>
     <JourneyInsights TelemetryEngine="ApplicationInsights" InstrumentationKey="{Your Application Insights Key}" DeveloperMode="false" ClientEnabled="false" ServerEnabled="true" TelemetryVersion="1.0.0" />
   </UserJourneyBehaviors>
   ```
   
1. 上傳並測試您的原則。

## <a name="next-steps"></a>後續步驟

該社群已開發了使用者旅程圖檢視器，可協助身分識別開發人員。 它會讀取您的 Application Insights 執行個體，並提供結構良好的使用者旅程圖事件檢視。 請取得原始碼，並將它部署在您自己的解決方案中。

Microsoft 不支援使用者旅程圖播放程式，並以完全相同的方式提供使用。

您可以在下列位置找到從 GitHub 上的 Application Insights 讀取事件的檢視器版本：

[Azure-範例/active directory-b2c-advanced-原則](https://github.com/Azure-Samples/active-directory-b2c-advanced-policies/tree/master/wingtipgamesb2c/src/WingTipUserJourneyPlayerWebApplication)
