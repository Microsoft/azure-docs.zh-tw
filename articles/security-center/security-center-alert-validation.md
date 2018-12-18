---
title: Azure 資訊安全中心的警示驗證 | Microsoft 文件
description: 本文件可協助您在 Azure 資訊安全中心驗證安全性警示。
services: security-center
documentationcenter: na
author: terrylan
manager: mbaldwin
editor: ''
ms.assetid: f8f17a55-e672-4d86-8ba9-6c3ce2e71a57
ms.service: security-center
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/07/2017
ms.author: yurid
ms.openlocfilehash: 1eb1bc78f233cbe37e8fdb1b05ac3cb90e8574da
ms.sourcegitcommit: ca05dd10784c0651da12c4d58fb9ad40fdcd9b10
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/03/2018
ms.locfileid: "32774840"
---
# <a name="alerts-validation-in-azure-security-center"></a>Azure 資訊安全中心的警示驗證
這份文件可協助您了解如何驗證您的系統是否已針對 Azure 資訊安全中心警示正確設定。

## <a name="what-are-security-alerts"></a>什麼是安全性警示：
資訊安全中心會自動收集、分析及整合您 Azure 資源、網路和已連線合作夥伴解決方案 (例如防火牆和端點保護解決方案) 的記錄資料，以偵測威脅並對您提出警示。 請閱讀[在 Azure 資訊安全中心管理與回應安全性警示](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)以取得安全性警示的詳細資訊，並閱讀[了解 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)以深入了解不同的警示類型。

## <a name="alert-validation"></a>警示驗證
在電腦上安裝資訊安全中心代理程式之後，從您希望警示的受攻擊資源所在的電腦執行下列步驟：

1. 將可執行檔 (例如 calc.exe) 複製到電腦的桌面，或方便您使用的其他目錄。
2. 將這個檔案重新命名為 **ASC_AlertTest_662jfi039N.exe**。
3. 開啟命令提示字元並使用引數 (只是假的引數名稱) 執行此檔案，例如：ASC_AlertTest_662jfi039N.exe -foo
4. 等待 5 到 10 分鐘，然後開啟安全性中心警示。 您應可找到如下所示的警示：

    ![警示驗證](./media/security-center-alert-validation/security-center-alert-validation-fig2.png)

檢閱此警示時，請確定 [啟用引數稽核] 欄位顯示為 true。 如果顯示為 false，您必須啟用命令列引數稽核。 您可以使用下列命令列來啟用這個選項：

*reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system\Audit" /f /v "ProcessCreationIncludeCmdLine_Enabled"*


> [!NOTE]
> 觀看 [Azure 資訊安全中心的警示驗證](https://channel9.msdn.com/Blogs/Azure-Security-Videos/Alert-Validation-in-Azure-Security-Center)影片，以查看這項功能的示範。 

## <a name="see-also"></a>另請參閱
本文介紹警示驗證程序。 現在，您已熟悉此驗證，請嘗試下列文章︰

* [管理及回應 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)。 了解如何在資訊安全中心管理警示，以及回應安全性事件。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md)。 了解如何監視 Azure 資源的健全狀況。
* [了解 Azure 資訊安全中心的安全性警示](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)。 了解不同類型的安全性警示。
* [Azure 資訊安全中心疑難排解指南](https://docs.microsoft.com/azure/security-center/security-center-troubleshooting-guide)。 了解如何針對資訊安全中心的常見問題進行疑難排解。 
* [Azure 資訊安全中心常見問題集](security-center-faq.md)。 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/)。 尋找有關 Azure 安全性與相容性的部落格文章。

