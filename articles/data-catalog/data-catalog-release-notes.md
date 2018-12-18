---
title: Azure 資料目錄版本資訊
description: Azure 資料目錄的版本資訊。
services: data-catalog
author: steelanddata
ms.author: maroche
ms.assetid: 3aca9c49-45a4-4352-92e6-bd25ee3eacf7
ms.service: data-catalog
ms.topic: conceptual
ms.date: 01/18/2018
ms.openlocfilehash: 539ef4f591a1ef0c6ab344e1d93a750133d9e089
ms.sourcegitcommit: 161d268ae63c7ace3082fc4fad732af61c55c949
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/27/2018
ms.locfileid: "43052813"
---
# <a name="azure-data-catalog-release-notes"></a>Azure 資料目錄版本資訊
## <a name="notes-for-the-november-20-2015-release-of-azure-data-catalog"></a>Azure 資料目錄 2015 年 11 月 20 日版本的注意事項
### <a name="opening-data-sources-in-power-bi-desktop"></a>在 Power BI Desktop 中開啟資料來源
使用 **Azure 資料目錄** 入口網站的 [在 Power BI Desktop 中開啟資料來源] 選項時，使用者可能會在 Power BI Desktop 應用程式中遇到下列兩個問題之一：

* 會顯示標題為「無法開啟文件」的對話方塊
* Power BI Desktop 應用程式可開啟，但檔案是空的

針對上述每個狀況，只要從 [PowerBI.com](https://powerbi.com)下載並安裝最新版的 Power BI Desktop 即可解決問題。

## <a name="notes-for-the-november-13-2015-release-of-azure-data-catalog"></a>Azure 資料目錄 2015 年 11 月 13 日版本的注意事項
### <a name="registering-and-connecting-to-teradata"></a>註冊並連接至 Teradata
連接到 Teradata 資料來源時，使用者必須已安裝正確的 Teradata ODBC 驅動程式，符合所使用軟體的位元 (32 位元或 64 位元)。

截至本 ADC 發行日為止，最新 [適用於 Windows 的 Teradata ODBC 驅動程式 (15.10 版)](http://downloads.teradata.com/download/connectivity/odbc-driver/windows) 相容於 Office 2013，但不相容於 Office 2016。

## <a name="notes-for-the-july-13-2015-release-of-azure-data-catalog"></a>Azure 資料目錄 2015 年 7 月 13 日版本的注意事項
### <a name="registering-and-connecting-to-oracle-database"></a>註冊並連接至 Oracle 資料庫
連接到 Oracle 資料庫資料來源時，使用者必須已安裝正確的 Oracle 驅動程式，符合所使用軟體的位元 (32 位元或 64 位元)。

* 在執行 32 位元 Windows 的電腦上註冊 Oracle 資料來源時，將使用 32 位元 Oracle 驅動程式
* 在執行 64 位元 Windows 的電腦上註冊 Oracle 資料來源時，將使用 64 位元 Oracle 驅動程式
* 在執行 32 位元版本 Microsoft Office 的電腦上使用 Excel 連接到 Oracle 資料來源時 (包括在 64 位元 Windows 上)，將使用 32 位元 Oracle 驅動程式
* 在執行 64 位元版本 Microsoft Office 的電腦上使用 Excel 連接到 Oracle 資料來源時，將使用 64 位元 Oracle 驅動程式

### <a name="registering-and-connecting-to-sql-server-reporting-services"></a>註冊及連接到 SQL Server Reporting Services
對於 SQL Server Reporting Services (SSRS) 資料來源的支援，僅限於原生模式伺服器。 未來版本將加入在 SharePoint 模式中支援 SSRS。

### <a name="opening-data-assets-in-excel"></a>在 Excel 中開啟資料資產
在 Microsoft Excel 中從 **Azure 資料目錄**入口網站開啟資料資產時，使用者可能會看到 [Microsoft Excel 安全性注意事項] 對話方塊的提示。 這是標準的預期行為，使用者可以選取 [啟用]  以繼續。

如需詳細資訊，請參閱 [啟用或停用可疑網站之連結和檔案的安全性警告](https://support.office.com/article/Enable-or-disable-security-alerts-about-links-and-files-from-suspicious-websites-A1AC6AE9-5C4A-4EB3-B3F8-143336039BBE)。

### <a name="proxy-and-policy-configuration-and-data-source-registration"></a>Proxy 與原則設定及資料來源註冊
使用者可能會遇到一種情況，他們可以登入 Azure 資料目錄入口網站，但在嘗試登入資料來源註冊工具時，卻遇到錯誤訊息，導致無法登入。

此問題行為有兩個可能的原因：

**原因 1：Active Directory Federation Services 設定**。資料來源註冊工具使用「表單驗證」，根據 Active Directory 驗證使用者登入。 Active Directory 系統管理員必須在全域驗證原則中啟用表單驗證，登入才會成功。

在某些情況下，只有當使用者是在公司網路上，或只有當使用者從公司網路外部連接時，才會發生此錯誤行為。 全域驗證原則允許分別為內部網路和外部網路連線啟用驗證方法。 如果使用者連線的網路未啟用表單驗證，可能會發生登入錯誤。

如需詳細資訊，請參閱 [設定驗證原則](https://technet.microsoft.com/library/dn486781.aspx)。

**原因 2：網路 Proxy 設定**。如果公司網路使用 Proxy 伺服器，註冊工具可能無法透過 Proxy 連線到 Azure Active Directory。 使用者可以編輯註冊工具的組態檔，將此區段加入至檔案，以確保註冊工具能夠連線：

      <system.net>
        <defaultProxy useDefaultCredentials="true" enabled="true">
          <proxy usesystemdefault="True"
                          proxyaddress="http://<your corporate network proxy url>"
                          bypassonlocal="True"/>
        </defaultProxy>
      </system.net>


若要找出 RegistrationTool.exe.config 檔案，請啟動註冊工具，然後開啟 Windows 工作管理員公用程式。 在工作管理員的 [詳細資料] 索引標籤，以滑鼠右鍵按一下 RegistrationTool.exe，再從快顯功能表中選擇 [開啟檔案位置]。
