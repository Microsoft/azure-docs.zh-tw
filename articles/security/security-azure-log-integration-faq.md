---
title: Azure 記錄整合常見問題集 | Microsoft Docs
description: 本文提供 Azure 記錄整合的相關問題解答。
services: security
documentationcenter: na
author: TomShinder
manager: MBaldwin
editor: TerryLanfear
ms.assetid: d06d1ac5-5c3b-49de-800e-4d54b3064c64
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload8: na
ms.date: 06/07/2018
ms.author: barclayn
ms.custom: azlog
ms.openlocfilehash: bec62b8c6b70706fa6519cbc2fd59bf69f119e9d
ms.sourcegitcommit: 4e36ef0edff463c1edc51bce7832e75760248f82
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 06/08/2018
ms.locfileid: "35236258"
---
# <a name="azure-log-integration-faq"></a>Azure 記錄整合常見問題集

本文提供 Azure 記錄整合的常見問題集 (FAQ) 解答。

>[!IMPORTANT]
> Azure 記錄整合功能將於 2019/06/01 淘汰。 AzLog 下載將於 2018 年 6 月 27 日停用。 如需繼續進行的指導，請檢閱 [Use Azure monitor to integrate with SIEM tools](https://azure.microsoft.com/blog/use-azure-monitor-to-integrate-with-siem-tools/) (使用 Azure 監視器與 SIEM 工具整合) 一文 

Azure 記錄整合是 Windows 作業系統服務，您可以使用此服務將來自 Azure 資源的未經處理記錄，整合到內部部署安全性資訊及事件管理 (SIEM) 系統內。 這項整合為您在內部部署或雲端中的所有資產，提供一個整合儀表板。 您可以接著彙總、相互關聯、分析與應用程式建立關聯的安全性事件，並發出警示。

整合 Azure 記錄檔的慣用方法是使用 SIEM 廠商的 Azure 監視器連接器，並遵循這些[指示](../monitoring-and-diagnostics/monitor-stream-monitoring-data-event-hubs.md)。 不過，如果您的 SIEM 廠商沒有提供 Azure 監視器連接器，在這類連接器可供使用之前，您可以使用 Azure 記錄整合作為暫時解決方案 (如果 Azure 記錄整合支援您的 SIEM)。

## <a name="is-the-azure-log-integration-software-free"></a>Azure 記錄整合軟體是否為免費？

是。 Azure 記錄整合軟體不需任何費用。

## <a name="where-is-azure-log-integration-available"></a>哪裡有提供 Azure 記錄整合？

它目前於 Azure Commercial 和 Azure Government 中提供，且無法在中國或德國使用。

## <a name="how-can-i-see-the-storage-accounts-from-which-azure-log-integration-is-pulling-azure-vm-logs"></a>如何查看 Azure 記錄整合從中提取 Azure VM 記錄的儲存體帳戶？

執行 **AzLog source list** 命令。

## <a name="how-can-i-tell-which-subscription-the-azure-log-integration-logs-are-from"></a>如何分辨 Azure 記錄整合記錄來自哪一個訂用帳戶？

在稽核記錄位於 **AzureResourcemanagerJson** 目錄的案例中，訂用帳戶識別碼是在記錄檔名稱中。 這也適用於 **AzureSecurityCenterJson** 資料夾中的記錄。 例如︰

20170407T070805_2768037.0000000023.**1111e5ee-1111-111b-a11e-1e111e1111dc**.json

Azure Active Directory 稽核記錄包含租用戶識別碼，作為名稱的一部分。

讀取自事件中樞的診斷記錄不會在名稱中包含訂用帳戶識別碼。 相反地，這些診斷記錄會包含建立事件中樞來源時所指定的易記名稱。 

## <a name="how-can-i-update-the-proxy-configuration"></a>如何更新 Proxy 組態？

如果您的 Proxy 設定不允許直接存取 Azure 儲存體，請開啟 **c:\Program Files\Microsoft Azure Log Integration** 中的 **AZLOG.EXE.CONFIG** 檔案。 更新檔案，以便將組織的 Proxy 位址納入 **defaultProxy** 區段。 更新完成之後，使用 **net stop AzLog** 和 **net start AzLog** 命令停止並啟動服務。

    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <system.net>
        <connectionManagement>
          <add address="*" maxconnection="400" />
        </connectionManagement>
        <defaultProxy>
          <proxy usesystemdefault="true"
          proxyaddress="http://127.0.0.1:8888"
          bypassonlocal="true" />
        </defaultProxy>
      </system.net>
      <system.diagnostics>
        <performanceCounters filemappingsize="20971520" />
      </system.diagnostics>   

## <a name="how-can-i-see-the-subscription-information-in-windows-events"></a>如何查看 Windows 事件中的訂用帳戶資訊？

在新增來源時將訂用帳戶識別碼附加到易記名稱的後面：

    Azlog source add <sourcefriendlyname>.<subscription id> <StorageName> <StorageKey>  
事件 XML 具有下列中繼資料，包括訂用帳戶識別碼：

![事件 XML][1]

## <a name="error-messages"></a>錯誤訊息
### <a name="when-i-run-the-command-azlog-createazureid-why-do-i-get-the-following-error"></a>執行 ```AzLog createazureid``` 命令時，為什麼收到下列錯誤訊息？

Error:

  *無法建立 AAD 應用程式 - 租用戶 72f988bf-86f1-41af-91ab-2d7cd011db37 - 原因 = 「禁止」 - 訊息 = 「權限不足以完成作業。」*

**azlog createazureid** 命令嘗試在 Azure 登入具有存取權之訂用帳戶的所有 Azure AD 租用戶中建立服務主體。 如果您的 Azure 登入在該 Azure AD 租用戶中只是來賓使用者，則命令會失敗，並出現「權限不足，無法完成此作業」。 要求租用戶系統管理員將您的帳戶新增為租用戶中的使用者。

### <a name="when-i-run-the-command-azlog-authorize-why-do-i-get-the-following-error"></a>當我執行 **azlog authorize** 命令時，為什麼收到下列錯誤？

Error:

  *建立角色指派警告 - AuthorizationFailed：具有物件識別碼 'fe9e03e4-4dad-4328-910f-fd24a9660bd2' 的用戶端 janedo@microsoft.com' 沒有在 '/subscriptions/70d95299-d689-4c97-b971-0d8ff0000000' 範圍內執行動作 'Microsoft.Authorization/roleAssignments/write' 的權限。*

**azlog authorize** 命令會將 Azure AD 服務主體讀取者的角色 (以 **azlog createazureid** 建立) 指派給所提供的訂用帳戶。 如果 Azure 登入不是訂用帳戶的共同管理員或擁有者，命令會失敗，並出現「授權失敗」錯誤訊息。 需要共同管理員或擁有者的 Azure 角色型存取控制 (RBAC) 才能完成此動作。

## <a name="where-can-i-find-the-definition-of-the-properties-in-the-audit-log"></a>哪裡可以找到稽核記錄中屬性的定義？

請參閱：

* [使用 Azure Resource Manager 來稽核作業](../azure-resource-manager/resource-group-audit.md)
* [在 Azure 監視器 REST API 中列出訂用帳戶中的管理事件](https://msdn.microsoft.com/library/azure/dn931934.aspx)

## <a name="where-can-i-find-details-on-azure-security-center-alerts"></a>哪裡可以找到 Azure 資訊安全中心警示的詳細資訊？

請參閱 [管理及回應 Azure 資訊安全中心的安全性警示](../security-center/security-center-managing-and-responding-alerts.md)。

## <a name="how-can-i-modify-what-is-collected-with-vm-diagnostics"></a>如何修改 VM 診斷會收集什麼？

如需如何取得、修改和設定 Azure 診斷設定的詳細資訊，請參閱[使用 PowerShell 在執行 Windows 的虛擬機器中啟用 Azure 診斷](../virtual-machines/windows/ps-extensions-diagnostics.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)。 

下列範例會取得 Azure 診斷設定：

    -AzureRmVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient
    $publicsettings = (Get-AzureRmVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient).PublicSettings
    $encodedconfig = (ConvertFrom-Json -InputObject $publicsettings).xmlCfg
    $xmlconfig = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($encodedconfig))
    Write-Host $xmlconfig

    $xmlconfig | Out-File -Encoding utf8 -FilePath "d:\WADConfig.xml"

下列範例會修改 Azure 診斷設定。 在此設定中，只會從安全性事件記錄檔收集事件識別碼 4624 和事件識別碼 4625。 Microsoft Antimalware for Azure 事件則會從系統事件記錄檔收集。 如需使用 XPath 運算式的詳細資訊，請參閱 [Consuming Events](https://msdn.microsoft.com/library/windows/desktop/dd996910(v=vs.85)) (取用事件)。

    <WindowsEventLog scheduledTransferPeriod="PT1M">
        <DataSource name="Security!*[System[(EventID=4624 or EventID=4625)]]" />
        <DataSource name="System!*[System[Provider[@Name='Microsoft Antimalware']]]"/>
    </WindowsEventLog>

下列範例會設定 Azure 診斷設定：

    $diagnosticsconfig_path = "d:\WADConfig.xml"
    Set-AzureRmVMDiagnosticsExtension -ResourceGroupName AzLog-Integration -VMName AzlogClient -DiagnosticsConfigurationPath $diagnosticsconfig_path -StorageAccountName log3121 -StorageAccountKey <storage key>

完成變更之後，請檢查儲存體帳戶，以確保會收集正確的事件。

如果您在安裝和設定期間有任何問題，請開啟[支援要求](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request)。 選取 [記錄整合] 作為您要求支援的服務。

## <a name="can-i-use-azure-log-integration-to-integrate-network-watcher-logs-into-my-siem"></a>可以使用 Azure 記錄整合將網路監看員記錄整合到 SIEM 嗎？

Azure 網路監看員會產生大量的記錄資訊。 這些記錄不一定會傳送到 SIEM。 網路監看員記錄的唯一支援目的地是儲存體帳戶。 Azure 記錄整合不支援讀取這些記錄，並且讓它們可供 SIEM 使用。

<!--Image references-->
[1]: ./media/security-azure-log-integration-faq/event-xml.png
