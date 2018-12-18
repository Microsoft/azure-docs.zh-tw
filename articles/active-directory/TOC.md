# [Azure Active Directory 文件](index.md)

# 概觀
## [什麼是 Azure Active Directory？](fundamentals/active-directory-whatis.md)
## [關於 Azure 身分識別管理](fundamentals/identity-fundamentals.md)
## [了解 Azure 身分識別解決方案](fundamentals/understand-azure-identity-solutions.md)
## [選擇混合式身分識別解決方案](choose-hybrid-identity-solution.md)
## [關聯 Azure 訂用帳戶](fundamentals/active-directory-how-subscriptions-associated-directory.md)
## [常駐和資料考量](fundamentals/active-directory-data-storage-eu.md)
## [常見問題集](fundamentals/active-directory-faq.md)
## [新功能](fundamentals/whats-new.md)


# 開始使用
## [註冊 Azure AD Premium](fundamentals/active-directory-get-started-premium.md)
## [新增自訂網域名稱](fundamentals/add-custom-domain.md)
## [設定公司商標](fundamentals/customize-branding.md)
## [在 Azure AD 中新增使用者](fundamentals/add-users-azure-active-directory.md)
## [將授權指派給使用者](fundamentals/license-users-groups.md)
## [啟用自助式密碼重設](authentication/quickstart-sspr.md)
## [在 Azure AD 中新增貴組織的隱私權資訊](active-directory-properties-area.md)
## [存取 Azure Active Directory 以建立新的租用戶](fundamentals/active-directory-access-create-new-tenant.md)


# 作法
## 規劃和設計
### [了解 Azure AD 架構](fundamentals/active-directory-architecture.md)
### [Azure Active Directory 中的宣告對應](develop/active-directory-claims-mapping.md)
### [部署混合式身分識別解決方案](hybrid/plan-hybrid-identity-design-considerations-overview.md)
#### 判斷需求
##### [身分識別](hybrid/plan-hybrid-identity-design-considerations-business-needs.md)
##### [目錄同步](hybrid/plan-hybrid-identity-design-considerations-directory-sync-requirements.md)
##### [多因素驗證](hybrid/plan-hybrid-identity-design-considerations-multifactor-auth-requirements.md)
##### [身分識別生命週期策略](hybrid/plan-hybrid-identity-design-considerations-lifecycle-adoption-strategy.md)
#### [資料安全性的規劃](hybrid/plan-hybrid-identity-design-considerations-data-protection-strategy.md)
##### [資料保護](hybrid/plan-hybrid-identity-design-considerations-dataprotection-requirements.md)
##### [內容管理](hybrid/plan-hybrid-identity-design-considerations-contentmgt-requirements.md)
##### [存取控制](hybrid/plan-hybrid-identity-design-considerations-accesscontrol-requirements.md)
##### [事件回應](hybrid/plan-hybrid-identity-design-considerations-incident-response-requirements.md)
#### 規劃身分識別生命週期
##### [工作](hybrid/plan-hybrid-identity-design-considerations-hybrid-id-management-tasks.md)
##### [採用策略](hybrid/plan-hybrid-identity-design-considerations-identity-adoption-strategy.md)
#### [後續步驟](hybrid/plan-hybrid-identity-design-considerations-nextsteps.md)
#### [工具比較](hybrid/plan-hybrid-identity-design-considerations-tools-comparison.md)

## 管理使用者
### [在 Azure AD 中新增使用者](fundamentals/add-users-azure-active-directory.md)
### [管理使用者設定檔](fundamentals/active-directory-users-profile-azure-portal.md)
### [重設使用者密碼](fundamentals/active-directory-users-reset-password-azure-portal.md)
### [共用帳戶](active-directory-sharing-accounts.md)
### [將使用者指派為管理員角色](fundamentals/active-directory-users-assign-role-azure-portal.md)
### [從另一個目錄 (B2B) 中新增來賓使用者](b2b/what-is-b2b.md)
#### [管理員新增 B2B 使用者](b2b/add-users-administrator.md)
#### [資訊背景工作新增 B2B 使用者](b2b/add-users-information-worker.md)
#### [API 與自訂](b2b/customize-invitation-api.md)
#### [Google 同盟](b2b/google-federation.md)
#### [程式碼和 Azure PowerShell 範例](b2b/code-samples.md)
#### [自助入口網站註冊範例](b2b/self-service-portal.md)
#### [邀請電子郵件](b2b/invitation-email-elements.md)
#### [兌換邀請](b2b/redemption-experience.md)
#### [在沒有邀請的情況下新增 B2B 使用者](b2b/add-user-without-invite.md)
#### [允許或封鎖邀請](b2b/allow-deny-list.md)
#### [B2B 的條件式存取](b2b/conditional-access.md)
#### [B2B 共用原則](b2b/delegate-invitations.md)
#### [將 B2B 使用者新增至角色](b2b/add-guest-to-role.md)
#### [動態群組和 B2B 使用者](b2b/use-dynamic-groups.md)
#### [離開組織](b2b/leave-the-organization.md)
#### [稽核與報告](b2b/auditing-and-reporting.md)
#### [適用於混合式組織的 B2B](b2b/hybrid-organizations.md)
##### [授與 B2B 使用者本機應用程式的存取權限](b2b/hybrid-cloud-to-on-premises.md)
##### [授與本機使用者雲端應用程式的存取權限](b2b/hybrid-on-premises-to-cloud.md)
#### [B2B 和 Office 365 外部共用](b2b/o365-external-user.md)
#### [B2B 授權](b2b/licensing-guidance.md)
#### [目前限制](b2b/current-limitations.md)
#### [常見問題集](b2b/faq.md)
#### [疑難排解 B2B](b2b/troubleshoot.md)
#### [了解 B2B 使用者](b2b/user-properties.md)
#### [B2B 使用者權杖](b2b/user-token.md)
#### [適用於 Azure AD 整合應用程式的 B2B](b2b/configure-saas-apps.md)
#### [B2B 使用者宣告對應](b2b/claims-mapping.md)
#### [比較 B2B 共同作業與 B2C](b2b/compare-with-b2c.md)
#### [取得 B2B 支援](b2b/get-support.md)

## [管理群組和成員](fundamentals/active-directory-manage-groups.md)
### [管理群組](fundamentals/active-directory-groups-create-azure-portal.md)
### [刪除群組和其成員](fundamentals/active-directory-groups-delete-group.md)
### [管理群組設定](fundamentals/active-directory-groups-settings-azure-portal.md)
## [管理報告](reports-monitoring/overview-reports.md)
### [登入活動](reports-monitoring/concept-sign-ins.md)
### [稽核活動](reports-monitoring/concept-audit-logs.md)
### [有風險的使用者](reports-monitoring/concept-user-at-risk.md)
### [有風險的登入](reports-monitoring/concept-risky-sign-ins.md)
### [風險事件](reports-monitoring/concept-risk-events.md)
### [用 Azure 監視器 監視記錄](reports-monitoring/concept-activity-logs-in-azure-monitor.md)
### [常見問題集](reports-monitoring/reports-faq.md)

### 工作
#### [下載登入報告](reports-monitoring/quickstart-download-sign-in-report.md)
#### [下載稽核報告](reports-monitoring/quickstart-download-audit-report.md)
#### [設定具名位置](reports-monitoring/quickstart-configure-named-locations.md)
#### [尋找活動報告](reports-monitoring/howto-find-activity-reports.md)
#### [使用 Azure AD Power BI 內容套件](reports-monitoring/howto-power-bi-content-pack.md)
#### [修復標幟為有風險的使用者](reports-monitoring/howto-remediate-users-flagged-for-risk.md)
#### [將活動記錄路由至 Azure 事件中樞](reports-monitoring/quickstart-azure-monitor-stream-logs-to-event-hub.md)
#### [將活動記錄封存至 Azure 儲存體帳戶](reports-monitoring/quickstart-azure-monitor-route-logs-to-storage-account.md)
#### [使用 Azure 監視器整合活動記錄與 Splunk](reports-monitoring/tutorial-integrate-activity-logs-with-splunk.md)
#### [使用 Azure 監視器整合活動記錄與 SumoLogic](reports-monitoring/howto-integrate-activity-logs-with-sumologic.md)
#### [使用 Azure 監視器整合活動記錄與 Log Analytics](reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

### 參考
#### [保留](reports-monitoring/reference-reports-data-retention.md)
#### [延遲](reports-monitoring/reference-reports-latencies.md)
#### [稽核活動參考](reports-monitoring/reference-audit-activities.md)
#### [登入活動的錯誤代碼](reports-monitoring/reference-sign-ins-error-codes.md)
#### [解譯 Azure 監視器中的稽核記錄結構描述](reports-monitoring/reference-azure-monitor-audit-log-schema.md)
#### [解譯 Azure 監視器中的登入記錄結構描述](reports-monitoring/reference-azure-monitor-sign-ins-log-schema.md)

### 疑難排解
#### [在 Azure AD 活動記錄中遺漏資料](reports-monitoring/troubleshoot-missing-audit-data.md)
#### [下載中缺少的資料](reports-monitoring/troubleshoot-missing-data-download.md)
#### [Azure AD 活動記錄內容套件錯誤](reports-monitoring/troubleshoot-content-pack.md)
#### [Azure AD 報告 API 中的錯誤](reports-monitoring/troubleshoot-graph-api.md)

### [程式設計存取](reports-monitoring/concept-reporting-api.md)
#### [先決條件](reports-monitoring/howto-configure-prerequisites-for-reporting-api.md)
#### [使用憑證](reports-monitoring/tutorial-access-api-with-certificates.md)

## [管理密碼](authentication/concept-sspr-howitworks.md)

## 管理應用程式
### [概觀](manage-apps/what-is-application-management.md)
### [開始使用](manage-apps/plan-an-application-integration.md)
### [SaaS 應用程式整合教學課程](saas-apps/tutorial-list.md)

### [SaaS 應用程式使用者佈建和解除佈建](manage-apps/user-provisioning.md) 
#### [應用程式整合教學課程](saas-apps/tutorial-list.md) 
#### [啟用 SCIM 應用程式的自動化佈建](manage-apps/use-scim-to-provision-users-and-groups.md) 
#### [自訂屬性對應](manage-apps/customize-application-attributes.md) 
#### [撰寫屬性對應的運算式](manage-apps/functions-for-customizing-application-data.md) 
#### [使用範圍篩選器](manage-apps/define-conditional-rules-for-provisioning-user-accounts.md) 
#### [關於使用者自動佈建的報告](manage-apps/check-status-user-account-provisioning.md) 
#### [針對使用者佈建進行疑難排解](active-directory-application-provisioning-content-map.md) 

### [透過 App Proxy 遠端存取應用程式](manage-apps/application-proxy.md)
#### 開始使用
##### [啟用應用程式 Proxy](manage-apps/application-proxy-enable.md)
##### [發佈應用程式](manage-apps/application-proxy-publish-azure-portal.md)
##### [自訂網域](manage-apps/application-proxy-configure-custom-domain.md)
#### [單一登入](manage-apps/application-proxy-single-sign-on.md)
##### [搭配 KCD 的 SSO](manage-apps/application-proxy-configure-single-sign-on-with-kcd.md)
##### [搭配標頭的 SSO](manage-apps/application-proxy-configure-single-sign-on-with-ping-access.md)
##### [搭配密碼保存庫的 SSO](manage-apps/application-proxy-configure-single-sign-on-password-vaulting.md)
#### 概念
##### [連接器](manage-apps/application-proxy-connectors.md)
##### [安全性](manage-apps/application-proxy-security.md)
##### [網路](manage-apps/application-proxy-network-topology.md)


##### [從 TMG 或 UAG 升級](manage-apps/application-proxy-migration.md)

#### 進階組態
##### [在不同網路上發佈](manage-apps/application-proxy-connector-groups.md)
##### [Proxy 伺服器](manage-apps/application-proxy-configure-connectors-with-proxy-servers.md)
##### [宣告感知應用程式](manage-apps/application-proxy-configure-for-claims-aware-applications.md)
##### [原生用戶端應用程式](manage-apps/application-proxy-configure-native-client-application.md)
##### [無訊息安裝](manage-apps/application-proxy-register-connector-powershell.md)
##### [自訂首頁](manage-apps/application-proxy-configure-custom-home-page.md)
##### [翻譯內嵌連結](manage-apps/application-proxy-configure-hard-coded-link-translation.md)
##### [萬用字元](manage-apps/application-proxy-wildcard.md)
##### [移除個人資料](manage-apps/application-proxy-remove-personal-data.md)


#### 發佈逐步解說
##### [遠端桌面](manage-apps/application-proxy-integrate-with-remote-desktop-services.md)
##### [SharePoint](manage-apps/application-proxy-integrate-with-sharepoint-server.md)
##### [Microsoft Teams](manage-apps/application-proxy-integrate-with-teams.md)
##### [Tableau](manage-apps/application-proxy-integrate-with-tableau.md)
##### [Qlik](manage-apps/application-proxy-qlik.md)
#### [PowerShell](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0#application_proxy_application_management) 

#### [疑難排解](manage-apps/application-proxy-troubleshoot.md)

### 管理企業應用程式
#### [新增應用程式](manage-apps/add-application-portal.md)
#### [檢視租用戶應用程式](manage-apps/view-applications-portal.md)
#### [設定單一登入](manage-apps/configure-single-sign-on-portal.md)
#### [指派使用者](manage-apps/assign-user-or-group-access-portal.md)
#### [自訂商標](manage-apps/change-name-or-logo-portal.md)
#### [停用使用者登入](manage-apps/disable-user-sign-in-portal.md)
#### [移除使用者](manage-apps/remove-user-or-group-access-portal.md)

#### [管理使用者帳戶佈建](manage-apps/configure-automatic-user-provisioning-portal.md)

#### [適用於 SAML 應用程式的進階憑證](manage-apps/certificate-signing-options.md)
#### [將應用程式排除在使用者體驗之外](manage-apps/hide-application-from-user-portal.md)
### [使用 HRD 原則設定登入時自動加速](manage-apps/configure-authentication-for-federated-users-portal.md)
### [將 AD FS 應用程式移轉至 Azure AD](manage-apps/migrate-adfs-apps-to-azure.md) 
### [管理應用程式的存取權](manage-apps/what-is-access-management.md)
#### [SSO 存取](manage-apps/what-is-single-sign-on.md)
#### [SSO 憑證](manage-apps/manage-certificates-for-federated-single-sign-on.md)
#### [租用戶限制](manage-apps/tenant-restrictions.md)
#### [使用 SCIM 佈建使用者](manage-apps/use-scim-to-provision-users-and-groups.md)

### [了解 Azure AD 應用程式同意體驗](develop/application-consent-experience.md)

### 疑難排解



#### 存取面板
##### [未顯示的應用程式](manage-apps/access-panel-troubleshoot-application-not-appearing.md)
##### [出現未預期的應用程式](manage-apps/access-panel-troubleshoot-unexpected-application.md)
##### [無法登入](manage-apps/access-panel-troubleshoot-web-sign-in-problem.md)
##### [安裝瀏覽器擴充功能時發生錯誤](manage-apps/access-panel-extension-problem-installing.md)
##### [如何使用自助應用程式存取](manage-apps/access-panel-manage-self-service-access.md)
##### [使用自助應用程式存取時發生錯誤](manage-apps/access-panel-troubleshoot-self-service-access.md)

#### 新增應用程式
##### [選擇應用程式類型](manage-apps/choose-application-type.md)
##### [常見問題 - 資源庫應用程式](manage-apps/adding-gallery-app-common-problems.md)
##### [常見問題 - 非資源庫應用程式](manage-apps/adding-non-gallery-app-common-problems.md)

#### 應用程式 Proxy
##### [顯示應用程式頁面時發生問題](manage-apps/application-proxy-page-appearance-broken-problem.md)
##### [應用程式載入時間過長](manage-apps/application-proxy-page-load-speed-problem.md)
##### [應用程式頁面連結無效](manage-apps/application-proxy-page-links-broken-problem.md)
##### [要開啟哪些適用於應用程式的連接埠](manage-apps/application-proxy-connectivity-ports-how-to.md)
##### [適用於應用程式的連接器群組中沒有運作中的連接器](manage-apps/application-proxy-connectivity-no-working-connector.md)
##### [在管理入口網站中設定](manage-apps/application-proxy-config-how-to.md)
##### [在應用程式中設定單一登入](manage-apps/application-proxy-config-sso-how-to.md)
##### [在管理入口網站中建立應用程式時發生問題](manage-apps/application-proxy-config-problem.md)
##### [設定 Kerberos 限制委派](manage-apps/application-proxy-back-end-kerberos-constrained-delegation-how-to.md)
##### [使用 PingAccess 設定](manage-apps/application-proxy-back-end-ping-access-how-to.md)
##### [「無法存取此公司應用程式」錯誤](manage-apps/application-proxy-sign-in-bad-gateway-timeout-error.md)
##### [安裝應用程式 Proxy 代理程式連接器時發生問題](manage-apps/application-proxy-connector-installation-problem.md)


#### 應用程式註冊
##### [應用程式物件的輸入欄位](develop/registration-config-specific-application-property-how-to.md)
##### [變更權杖存留期預設值](develop/registration-config-change-token-lifetime-how-to.md)

#### 驗證
##### [設定端點](develop/registration-config-how-to.md)

#### 條件式存取
##### [客戶不符合裝置註冊預先要求](active-directory-conditional-access.md)
##### [離線的公司網路規則會在何時生效，以及如何生效？](https://aka.ms/calocation)
##### [如何增加使用者允許在 Azure AD 中註冊的裝置數目？](active-directory-azureadjoin-setup.md)
##### [如何設定 Exchange Online 的條件式存取？](https://aka.ms/csforexchange)
##### [如何設定適用於 Windows 7 裝置的條件式存取？](active-directory-conditional-access.md#device-based-conditional-access)
##### [哪些應用程式支援條件式存取？](active-directory-conditional-access-supported-apps.md)

#### 尋找 API
##### [尋找 API](develop/api-find-an-api-how-to.md)

#### 管理存取
##### [將使用者和群組指派至群組](manage-apps/methods-for-assigning-users-and-groups.md)
##### [移除應用程式中的使用者存取](manage-apps/methods-for-removing-user-access.md)
##### [設定自助式應用程式指派](manage-apps/manage-self-service-access.md)
##### [指派未預期的使用者](manage-apps/ways-users-get-assigned-to-applications.md)
##### [應用程式清單中的未預期應用程式](manage-apps/application-types.md)

#### 多租用戶應用程式
##### [設定新的應用程式](develop/setup-multi-tenant-app.md)
##### [新增至應用程式資源庫](develop/registration-config-multi-tenant-application-add-to-gallery-how-to.md)

#### 權限
##### [選擇適用於 API 的使用權限](develop/perms-for-given-api.md)
##### [將使用權限授與應用程式](develop/registration-config-grant-permissions-how-to.md)
##### [委派與應用程式使用權限](develop/delegated-and-app-perms.md)
##### [應用程式同意](develop/consent-framework.md)

#### 佈建
##### [所需時間](manage-apps/application-provisioning-when-will-provisioning-finish-specific-user.md)
##### [所需時間 - 資源庫應用程式](manage-apps/application-provisioning-when-will-provisioning-finish.md)
##### [設定使用者佈建 - 資源庫應用程式](manage-apps/application-provisioning-config-how-to.md)
##### [設定使用者佈建時發生問題 - 資源庫應用程式](manage-apps/application-provisioning-config-problem.md)
##### [在設定使用者佈建資源庫應用程式的情況下儲存管理員認證時發生問題](manage-apps/application-provisioning-config-problem-storage-limit.md)
##### [未佈建使用者 - 資源庫應用程式](manage-apps/application-provisioning-config-problem-no-users-provisioned.md)
##### [佈建錯誤的使用者 - 資源庫應用程式](manage-apps/application-provisioning-config-problem-wrong-users-provisioned.md)

#### 單一登入
##### [選擇方法](manage-apps/single-sign-on-modes.md)
##### [設定](develop/registration-config-sso-how-to.md)
##### [設定同盟 - 資源庫應用程式](manage-apps/configure-federated-single-sign-on-gallery-applications.md)
##### [設定同盟常見問題 - 資源庫應用程式](manage-apps/configure-federated-single-sign-on-gallery-applications-problems.md)
##### [設定同盟 - 非資源庫應用程式](manage-apps/configure-federated-single-sign-on-non-gallery-applications.md)
##### [設定同盟常見問題 - 非資源庫應用程式](manage-apps/configure-federated-single-sign-on-non-gallery-applications-problems.md)
##### [設定密碼 - 資源庫應用程式](manage-apps/configure-password-single-sign-on-gallery-applications.md)
##### [設定密碼常見問題 - 資源庫應用程式](manage-apps/configure-password-single-sign-on-gallery-applications-problems.md)
##### [設定密碼 - 非資源庫應用程式](manage-apps/configure-password-single-sign-on-non-gallery-applications.md)
##### [設定密碼常見問題 - 非資源庫應用程式](manage-apps/configure-password-single-sign-on-non-gallery-applications-problems.md)

#### 使用者登入問題
##### [未預期的同意提示](manage-apps/application-sign-in-unexpected-user-consent-prompt.md)
##### [使用者同意錯誤](manage-apps/application-sign-in-unexpected-user-consent-error.md)
##### [從自訂入口網站登入時發生問題](manage-apps/application-sign-in-other-problem-deeplink.md)
##### [從存取面板登入時發生問題](manage-apps/application-sign-in-other-problem-access-panel.md)
##### [應用程式登入頁面發生錯誤](manage-apps/application-sign-in-problem-application-error.md)
##### [密碼單一登入問題 - 非資源庫應用程式](manage-apps/application-sign-in-problem-password-sso-non-gallery.md)
##### [密碼單一登入問題 - 資源庫應用程式](manage-apps/application-sign-in-problem-password-sso-gallery.md)
##### [登入 Microsoft 應用程式時發生問題](manage-apps/application-sign-in-problem-first-party-microsoft.md)
##### [同盟單一登入問題 - 非資源庫應用程式](manage-apps/application-sign-in-problem-federated-sso-non-gallery.md)
##### [同盟單一登入問題 - 資源庫應用程式](manage-apps/application-sign-in-problem-federated-sso-gallery.md)
##### [自訂開發應用程式問題](manage-apps/application-sign-in-problem-custom-dev.md)
##### [內部部署應用程式問題 - 應用程式 Proxy](manage-apps/application-sign-in-problem-on-premises-application-proxy.md)

### [開發應用程式](active-directory-applications-guiding-developers-for-lob-applications.md)


## 管理您的目錄
### [Azure AD Connect](hybrid/whatis-hybrid-identity.md)
### 自訂網域名稱
#### [快速入門](fundamentals/add-custom-domain.md)
### [管理目錄](fundamentals/active-directory-administer.md)
### [企業狀態漫遊](active-directory-windows-enterprise-state-roaming-overview.md)
#### [啟用](active-directory-windows-enterprise-state-roaming-enable.md)
#### [群組原則設定](active-directory-windows-enterprise-state-roaming-group-policy-settings.md)
#### [Windows 10 設定](active-directory-windows-enterprise-state-roaming-windows-settings-reference.md)
#### [常見問題集](active-directory-windows-enterprise-state-roaming-faqs.md)
#### [疑難排解](active-directory-windows-enterprise-state-roaming-troubleshooting.md)


### [使用 Azure AD Connect 整合內部部署身分識別](hybrid/whatis-hybrid-identity.md)

### [設定權杖存留期](develop/active-directory-configurable-token-lifetimes.md)

## 保護您的身分識別

### [Privileged Identity Management](privileged-identity-management/pim-configure.md?toc=%2fazure%2factive-directory%2ftoc.json)

## [在 Azure 中部署 AD FS](hybrid/how-to-connect-fed-azure-adfs.md)
### [高可用性](active-directory-adfs-in-azure-with-azure-traffic-manager.md)
### [變更簽章雜湊演算法](active-directory-federation-sha256-guidance.md)

## [疑難排解](fundamentals/active-directory-troubleshooting-support-howto.md)

## 部署 Azure AD 概念證明 (PoC)
### [PoC 腳本：簡介](active-directory-playbook-intro.md)
### [PoC 腳本：因素](active-directory-playbook-ingredients.md)
### [PoC 腳本：實作](active-directory-playbook-implementation.md)
### [PoC 腳本：建置區塊](active-directory-playbook-building-blocks.md)

# 參考
## [程式碼範例](https://azure.microsoft.com/resources/samples/?service=active-directory)
## [Azure PowerShell Cmdlet](/powershell/azure/overview)
## [Java API 參考](/java/api)
## [.NET API](/active-directory/adal/microsoft.identitymodel.clients.activedirectory)

# 相關參考
## [Multi-Factor Authentication](/azure/multi-factor-authentication/)
## [Azure AD Connect](hybrid/whatis-hybrid-identity.md)
## [Azure AD Connect Health](hybrid/whatis-hybrid-identity-health.md)
## [開發人員適用的 Azure AD](./develop/active-directory-how-to-integrate.md)
## [Azure AD Privileged Identity Management](./privileged-identity-management/pim-configure.md)

# 資源
## [Azure AD 部署規劃](./fundamentals/active-directory-deployment-plans.md)
## [Azure 意見反應論壇](https://feedback.azure.com/forums/169401-azure-active-directory)
## [Azure 藍圖](https://azure.microsoft.com/roadmap/?category=security-identity)
## [MSDN 論壇](https://social.msdn.microsoft.com/Forums/azure/en-US/home?forum=WindowsAzureAD)
## [定價](https://azure.microsoft.com/pricing/details/active-directory/)
## [定價計算機](https://azure.microsoft.com/pricing/calculator/)
## [服務更新](https://azure.microsoft.com/updates/?product=active-directory)
## [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-active-directory)
## [影片](https://azure.microsoft.com/documentation/videos/index/?services=active-directory)
