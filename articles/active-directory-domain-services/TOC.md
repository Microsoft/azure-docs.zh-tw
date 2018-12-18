# [Azure AD Domain Services 文件](index.yml)

# 概觀
## [何謂 Azure AD 網域服務？](active-directory-ds-overview.md)
## 適合您嗎？
### [與 Windows Server AD 比較](active-directory-ds-comparison.md)
### [與 Azure AD 聯結比較](active-directory-ds-compare-with-azure-ad-join.md)
## [有哪些新功能？](https://azure.microsoft.com/updates/?product=active-directory-ds)
## [特性](active-directory-ds-features.md)
## [案例](active-directory-ds-scenarios.md)
## [同步處理如何運作](active-directory-ds-synchronization.md)
## [相容的協力廠商軟體](active-directory-ds-compatible-software.md)

# 開始使用
## [工作 1：設定基本設定](active-directory-ds-getting-started.md)
## [工作 2：設定網路基本設定](active-directory-ds-getting-started-network.md)
## [工作 3：設定系統管理員群組並啟用 Azure AD Domain Services](active-directory-ds-getting-started-admingroup.md)
## [工作 4：更新虛擬網路的 DNS 設定](active-directory-ds-getting-started-dns.md)
## [工作 5：啟用密碼雜湊同步處理](active-directory-ds-getting-started-password-sync.md)

# 作法
## [檢查受控網域健康情況](active-directory-ds-check-health.md)
## [在 Azure CSP 訂用帳戶中使用 Azure AD Domain Services](active-directory-ds-csp.md)
## [使用 PowerShell 啟用 Azure AD Domain Services](active-directory-ds-enable-using-powershell.md)
## [在 Azure AD 中將保留範圍同步處理至受控網域](active-directory-ds-scoped-synchronization.md)
## [保護受控網域](active-directory-ds-secure-your-domain.md)
## 加入受控網域
### [Windows Server VM](active-directory-ds-admin-guide-join-windows-vm-portal.md)
### [來自範本的 Windows Server VM](active-directory-ds-join-windows-vm-template.md)
### [CentOS](active-directory-ds-join-centos-linux-vm.md)
### [CoreOS](active-directory-ds-join-coreos-linux-vm.md)
### [Red Hat Enterprise Linux](active-directory-ds-join-rhel-linux-vm.md)
### [Ubuntu Server](active-directory-ds-join-ubuntu-linux-vm.md)
## 管理受控網域
### [管理受控網域](active-directory-ds-admin-guide-administer-domain.md)
### [管理受控網域上的 DNS](active-directory-ds-admin-guide-administer-dns.md)
### [更新電子郵件通知](active-directory-ds-notifications.md)
### 為受控網域設定安全 LDAP
#### [工作 1 - 取得安全 LDAP 的憑證](active-directory-ds-admin-guide-configure-secure-ldap.md)
#### [工作 2 - 匯出安全 LDAP 憑證](active-directory-ds-admin-guide-configure-secure-ldap-export-pfx.md)
#### [工作 3：使用 Azure 入口網站，為受控網域啟用安全 LDAP](active-directory-ds-admin-guide-configure-secure-ldap-enable-ldaps.md)
#### [工作 4：設定 DNS 以從網際網路存取受控網域](active-directory-ds-ldaps-configure-dns.md)
#### [工作 5：繫結至受控網域，並鎖定安全 LDAP 存取](active-directory-ds-ldaps-bind-lockdown.md)
#### [針對安全 LDAP 進行疑難排解](active-directory-ds-ldaps-troubleshoot.md)

### [在受控網域上建立 OU](active-directory-ds-admin-guide-create-ou.md)
### [在受控網域上建立群組受控服務帳戶](active-directory-ds-create-gmsa.md)
### [受控網域中的管理群組](active-directory-ds-admin-guide-administer-group-policy.md)
### [設定受控網域上的密碼原則](active-directory-ds-password-policy.md)
## [選取虛擬網路](active-directory-ds-networking.md)
## 部署應用程式
### [設定 SharePoint 伺服器設定檔同步處理的支援](active-directory-ds-enable-sharepoint-profile-sync.md)
### [設定 Kerberos 限制委派](active-directory-ds-enable-kcd.md)
### [部署 Azure AD 應用程式 Proxy](active-directory-ds-deploy-azure-app-proxy.md)
## [刪除受控網域](active-directory-ds-disable-aadds.md)
## 疑難排解
### [常見問題集](active-directory-ds-faqs.md)
### [疑難排解指南](active-directory-ds-troubleshooting.md)
### [疑難排解警示](active-directory-ds-troubleshoot-alerts.md)
#### [修正損毀的 NSG 組態](active-directory-ds-troubleshoot-nsg.md)
#### [還原遺失的服務主體](active-directory-ds-troubleshoot-service-principals.md)
#### [安全 LDAP 錯誤](active-directory-ds-troubleshoot-ldaps.md)
### [解析不相符的租用戶錯誤](active-directory-ds-mismatched-tenant-error.md)
### [暫時停權的網域](active-directory-ds-suspension.md)


# 參考
## [程式碼範例](https://azure.microsoft.com/resources/samples/?service=active-directory)

# 相關參考
## [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md)
## [Azure Active Directory B2C](../active-directory-b2c/active-directory-b2c-overview.md)
## [Multi-Factor Authentication](../active-directory/authentication/multi-factor-authentication.md)

# 資源
## [Azure AD 意見反應論壇](https://feedback.azure.com/forums/169401-azure-active-directory)
## [Azure 藍圖](https://azure.microsoft.com/roadmap/?category=security-identity)
## [與我們連絡](active-directory-ds-contact-us.md)
## [定價](https://azure.microsoft.com/pricing/details/active-directory-ds/)
## [定價計算機](https://azure.microsoft.com/pricing/calculator/)
## [服務更新](https://azure.microsoft.com/updates/?product=active-directory-ds)
