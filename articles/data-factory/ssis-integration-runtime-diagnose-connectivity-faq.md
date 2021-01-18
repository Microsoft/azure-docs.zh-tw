---
title: 使用 SSIS 整合執行時間中的診斷連線能力功能
description: 使用診斷連線功能對 SSIS 整合執行時間中的連接問題進行疑難排解。
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: meiyl
author: meiyl
ms.reviewer: sawinark
manager: yidetu
ms.date: 06/07/2020
ms.openlocfilehash: 698a9c062596a3439d95ac0d586854fc6616fdd6
ms.sourcegitcommit: 6628bce68a5a99f451417a115be4b21d49878bb2
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/18/2021
ms.locfileid: "98556540"
---
# <a name="use-the-diagnose-connectivity-feature-in-the-ssis-integration-runtime"></a>使用 SSIS 整合執行時間中的診斷連線能力功能

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

在 SSIS Integration runtime 中執行 SQL Server Integration Services (SSIS) 套件時，可能會發現連接問題。 如果您的 SSIS 整合執行時間加入 Azure 虛擬網路，就會發生這些問題。

使用 *診斷* 連接功能來測試連線，以針對連線問題進行疑難排解。 這項功能位於 Azure Data Factory 入口網站的 [監視 SSIS 整合執行時間] 頁面上。

 ![監視頁面-診斷連線能力](media/ssis-integration-runtime-diagnose-connectivity-faq/ssis-monitor-diagnose-connectivity.png)

 ![監視頁面-測試連接](media/ssis-integration-runtime-diagnose-connectivity-faq/ssis-monitor-test-connection.png)

使用下列各節，瞭解您在測試連接時所發生的最常見錯誤。 每個章節都會說明：

- 錯誤碼
- 錯誤訊息
- 錯誤的可能原因 (s) 
- 建議的解決方案 (s) 

## <a name="error-code-invalidinput"></a>錯誤碼： InvalidInput

- **錯誤訊息**：「請確認您的輸入正確。」
- **可能的原因**：您的輸入不正確。
- **建議**：檢查您的輸入。

## <a name="error-code-firewallornetworkissue"></a>錯誤碼： FirewallOrNetworkIssue

- **錯誤訊息**：「請確認您的防火牆/伺服器/NSG 上已開啟此埠，且網路穩定。」
- **可能的原因：**
  - 您的伺服器未開啟埠。
  - 您的網路安全性群組拒絕埠上的輸出流量。
  - 您的 NVA/Azure 防火牆/內部部署防火牆未開啟該埠。
- **建議：**
  - 開啟伺服器上的埠。
  - 更新網路安全性群組以允許埠上的輸出流量。
  - 開啟 NVA/Azure 防火牆/內部部署防火牆上的埠。

## <a name="error-code-misconfigureddnssettings"></a>錯誤碼： MisconfiguredDnsSettings

- **錯誤訊息**：「如果您在 Azure-SSIS IR 所聯結的 VNet 中使用自己的 DNS 伺服器，請確認它可以解析您的主機名稱」。
- **可能的原因：**
  -  您的自訂 DNS 有問題。
  -  您未針對私人主機名稱使用 (FQDN) 的完整功能變數名稱。
- **建議：**
  -  請修正您的自訂 DNS 問題，以確定它可以解析主機名稱。
  -  使用 FQDN。 Azure-SSIS IR 不會自動附加您自己的 DNS 尾碼。 例如，使用 **<your_private_server> contoso.com** ，而不是 **<your_private_server**>。

## <a name="error-code-servernotallowremoteconnection"></a>錯誤碼： ServerNotAllowRemoteConnection

- **錯誤訊息**：「請確認您的伺服器允許透過此埠進行遠端 TCP 連接」。
- **可能的原因：**
  -  您的伺服器防火牆不允許遠端 TCP 連接。
  -  您的伺服器不在線上。
- **建議：**
  -  允許伺服器防火牆上的遠端 TCP 連接。
  -  啟動伺服器。
   
## <a name="error-code-misconfigurednsgsettings"></a>錯誤碼： MisconfiguredNsgSettings

- **錯誤訊息**：「請確認您的 VNet NSG 允許透過此埠的輸出流量。 如果您使用 Azure ExpressRoute 和或 UDR，請確認您的防火牆/伺服器上已開啟此埠。
- **可能的原因：**
  -  您的網路安全性群組拒絕埠上的輸出流量。
  -  您的 NVA/Azure 防火牆/內部部署防火牆未開啟該埠。
- **建議：**
  -  更新網路安全性群組以允許埠上的輸出流量。
  -  開啟 NVA/Azure 防火牆/內部部署防火牆上的埠。

## <a name="error-code-genericissues"></a>錯誤碼： GenericIssues

- **錯誤訊息**：「由於一般問題，測試連接失敗。」
- **可能的原因**：測試連接發生一般暫時性問題。
- **建議**：稍後再重試測試連接。 如果重試沒有説明，請聯絡 Azure Data Factory 支援小組。

## <a name="error-code-pspingexecutiontimeout"></a>錯誤碼： PSPingExecutionTimeout

- **錯誤訊息**：「測試連接逾時，請稍後再試一次」。
- **可能的原因**：測試連接逾時。
- **建議**：稍後再重試測試連接。 如果重試沒有説明，請聯絡 Azure Data Factory 支援小組。

## <a name="error-code-networkinstable"></a>錯誤碼： NetworkInstable

- **錯誤訊息**：「因為網路不穩定，導致測試連接不規則地成功。」
- **可能的原因**：暫時性的網路問題。
- **建議**：檢查伺服器或防火牆網路是否穩定。

## <a name="next-steps"></a>後續步驟

- [使用 SSMS 將 SSIS 專案部署至 Azure](/sql/integration-services/ssis-quickstart-deploy-ssms)
- [使用 SSMS 在 Azure 中執行 SSIS 套件](/sql/integration-services/ssis-quickstart-run-ssms)
- [在 Azure 中排程 SSIS 套件](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms)