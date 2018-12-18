---
title: 針對 Azure 基礎結構的合規性自動化使用 InSpec
description: 了解如何使用 InSpec 來偵測 Azure 部署中的問題
keywords: azure, chef, devops, 虛擬機器, 概觀, 自動化, inspce
ms.service: virtual-machines-linux
author: tomarcher
manager: jeconnoc
ms.author: tarcher
ms.date: 05/15/2018
ms.topic: article
ms.openlocfilehash: 4193b7fdb3932cbffa2b56b5d7eee6f3b573bd99
ms.sourcegitcommit: 96089449d17548263691d40e4f1e8f9557561197
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/17/2018
ms.locfileid: "34259532"
---
# <a name="use-inspec-for-compliance-automation-of-your-azure-infrastructure"></a>針對 Azure 基礎結構的合規性自動化使用 InSpec
[InSpec](https://www.chef.io/inspec/) 是免費的開放原始碼架構，可以用來對應用程式和基礎結構進行測試和稽核。 InSpec 的運作方式是將您系統的實際狀態，與您在方便讀取和方便寫入的 InSpec 程式碼中所表示的理想狀態兩者進行比較。 InSpec 會偵測違規，並且以報告形式顯示調查結果，但可讓您控制補救措施。 您可以使用 InSpec 來驗證您在 Azure 中執行的虛擬機器狀態。 您也可以使用 InSpec 來掃描及驗證訂用帳戶內資源和資源群組的狀態。

本文說明使用 InSpec 讓 Azure 上的安全性和合規性更簡單的優點。

## <a name="make-compliance-easy-to-understand-and-assess"></a>讓合規性易於了解和評估
使用 InSpec，您就可以將需求轉換為可建立版本、可執行、人類看得懂的程式碼。 這可讓您將測試組織成可組合的設定檔，您可以在其中視需要定義及自訂例外狀況。

## <a name="detect-fleet-wide-issues-and-prioritize-their-remediation"></a>偵測全面的問題並且對其補救措施設定優先權
InSpec 無代理程式偵測模式可讓您快速大規模評估您的暴露等級。 影響/嚴重性評分的內建中繼資料，可協助判斷要著重哪個區域的補救措施。

## <a name="inspect-machines-data-and-new-saas-apis"></a>檢查電腦、資料和新的 SaaS API
InSpec 雲端 API 合規性功能可讓您針對雲端合規性做出相關的粗略和細微判斷提示，並持續回報。

## <a name="satisfy-audits"></a>滿足稽核要求
有了 InSpec，您就可以隨時回應稽核問題，而不只是以每季或每年等預先決定的間隔來回應。 InSpec 可讓您在知道確切合規性狀態的情況下輸入稽核循環，而不會因為稽核員的調查結果而感到意外。

## <a name="reduce-ambiguity-and-miscommunication-regarding-rules"></a>減少與規則相關的語意模糊和誤解情形
文件中的組態和程序有待解釋。 可執行程式碼能省去應該要評估哪些項目的討論，有利於具有明確意圖的有形測試。

## <a name="keep-up-with-rapidly-changing-threat-and-compliance-landscapes"></a>跟上快速變化的威脅和法務遵循環境
InSpec 可讓您在同一天撰寫及發行偵測程式碼，並且撰寫新的規則以快速回應新的法規。 這表示威脅或法規的變化不再等同於緊急事件。

## <a name="next-steps"></a>後續步驟
* [使用 Chef 在 Azure 上建立 Windows 虛擬機器](/azure/virtual-machines/windows/chef-automation)