---
title: 針對 Azure Stack 驗證即服務進行疑難排解 | Microsoft Docs
description: 針對 Azure Stack 的驗證即服務進行疑難排解。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/24/2018
ms.author: mabrigg
ms.reviewer: johnhas
ms.openlocfilehash: 4372da2f1057c06761dd6abaf18c26df6e33640e
ms.sourcegitcommit: 3f8f973f095f6f878aa3e2383db0d296365a4b18
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/20/2018
ms.locfileid: "40234507"
---
# <a name="troubleshoot-validation-as-a-service"></a>針對驗證即服務進行疑難排解

[!INCLUDE[Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

以下是與軟體版本及其解決方案無關的常見問題。

## <a name="the-portal-shows-local-agent-in-debug-mode"></a>入口網站以偵錯模式顯示本機代理程式

這可能是因為代理程式因網路連線不穩定而無法將活動訊號傳送給服務。 傳送活動訊號時會每隔 5 分鐘傳送一次。 如果服務持續 15 分鐘未收到活動訊號，就會將代理程式視為非使用中，而不會再於該代理程式上排定測試。 請查看位於代理程式啟動目錄中 *Agenthost.log* 檔案內的錯誤訊息。

> [!Note] 
> 任何已經在代理程式上執行的測試都將繼續執行，但如果活動訊號在測試結束前仍未還原，則代理程式將會無法更新測試狀態或上傳記錄。 測試將一律顯示為「正在執行」，而將必須取消。

## <a name="agent-process-on-machine-was-shut-down-while-executing-test-what-to-expect"></a>執行測試時，機器上的代理程式處理序被關閉。 有何影響？

如果代理程式處理序是以非正常方式關閉的 (例如機器被重新啟動、處理序被終止 (在代理程式視窗上按 CTRL+C 會視為正常關閉))，則在代理程式上執行的測試將會繼續顯示為「正在執行」。 如果重新啟動代理程式，代理程式就會將測試的狀態更新成「已取消」。 如果未重新啟動代理程式，測試就會顯示為「正在執行」，您必須手動取消該測試

> [!Note] 
> 工作流程內的測試會排定成依序執行。 「暫止」測試會等到相同工作流程中處於「正在執行」狀態的測試完成後，才會執行。

## <a name="handle-slow-network-connectivity"></a>處理緩慢的網路連線

您可以將 PIR 映像下載到您本機資料中心內的共用中。 接著，您便可以驗證該映像。

<!-- This is from the appendix to the Deploy local agent topic. -->

### <a name="download-pir-image-to-local-share-in-case-of-slow-network-traffic"></a>在網路流量緩慢時將 PIR 映像下載至本機共用

1. 從以下來源下載 AzCopy：[vaasexternaldependencies(AzCopy)](https://vaasexternaldependencies.blob.core.windows.net/prereqcomponents/AzCopy.zip)

2. 將 AzCopy.zip 解壓縮，然後切換至包含 AzCopy.exe 的目錄

3. 從已提升權限的提示字元開啟 Windows PowerShell。 執行下列命令：

```PowerShell  
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Server2016DatacenterFullBYOL.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Server2016DatacenterCoreBYOL.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'WindowsServer2012R2DatacenterBYOL.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Ubuntu1404LTS.vhd' /NC:12 /V:azcopylog.log /Y
    .\azcopy.exe /Source:'https://azurestacktemplate.blob.core.windows.net/azurestacktemplate-public-container' /Dest:'<LocalFileShare>' /Pattern:'Ubuntu1604-20170619.1.vhd' /NC:12 /V:azcopylog.log /Y
```

> [!Note]  
> LocalFileShare 是共用路徑或本機路徑。

### <a name="verifying-pir-image-file-hash-value"></a>驗證 PIR 映像檔雜湊值

您可以使用 **Get-HashFile** Cmdlet 來取得所下載公用映像存放庫映像檔的雜湊值，以檢查映像檔的完整性。

| 檔案名稱 | SHA256 |
|---------------------------------------|------------------------------------------------------------------|
| Server2016DatacenterFullBYOL.vhd | 6ED58DCA666D530811A1EA563BA509BF9C29182B902D18FCA03C7E0868F733E9 |
| WindowsServer2012R2DatacenterBYOL.vhd | 9792CBF742870B1730B9B16EA814C683A8415EFD7601DDB6D5A76D0964767028 |
| Server2016DatacenterCoreBYOL.vhd | 5E80E1A6721A48A10655E6154C1B90E320DF5558487D6A0D7BFC7DCD32C4D9A5 |
| Ubuntu1404LTS.vhd | B24CDD12352AAEBC612A4558AB9E80F031A2190E46DCB459AF736072742E20E0 |
| Ubuntu1604-20170619.1.vhd | C481B88B60A01CBD5119A3F56632A2203EE5795678D3F3B9B764FFCA885E26CB |

## <a name="next-steps"></a>後續步驟

- 深入了解 [Azure Stack 驗證即服務](https://docs.microsoft.com/azure/azure-stack/partner)。
