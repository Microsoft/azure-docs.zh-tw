---
title: 快速入門：使用 Azure 入口網站下載稽核報告 | Microsoft Docs
description: 了解如何使用 Azure 入口網站下載稽核報告
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: 4de121ea-f4aa-4c8a-aae4-700c2c5e97a2
ms.service: active-directory
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 09/13/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: d6ecf62b46c25ec7cf4ab80219d1bc220e4eb4cc
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46367401"
---
# <a name="quickstart-download-an-audit-report-using-the-azure-portal"></a>快速入門：使用 Azure 入口網站下載稽核報告

在此快速入門中，您將學到如何下載您租用戶過去 24 小時內的稽核記錄。

## <a name="prerequisites"></a>必要條件

您需要：

* Azure Active Directory 租用戶。 
* 一個使用者，他必須是租用戶中「安全性系統管理員」、「安全性讀取者」或「全域系統管理員」的成員。 此外，租用戶中的任何使用者都可以存取自己的稽核記錄。

## <a name="quickstart-download-an-audit-report"></a>快速入門：下載稽核報告

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。
2. 從左側瀏覽窗格選取 [Azure Active Directory] 並使用 [切換目錄] 按鈕來選取您的 Active Directory。
3. 從儀表板選取 [Azure Active Directory]，然後選取 [稽核記錄]。 
4. 在 [日期範圍] 篩選下拉式清單中選擇 [過去 24 小時]，然後選取 [套用] 以檢視過去 24 小時的稽核記錄。 
5. 選取 [下載] 按鈕以下載包含已篩選之記錄的 CSV 檔案。 

![報告](./media/quickstart-download-audit-report/download-audit-logs.png)

## <a name="next-steps"></a>後續步驟

* [Azure Active Directory 入口網站中的登入活動報告](concept-sign-ins.md)
* [Azure Active Directory 報告保留期](reference-reports-data-retention.md)
* [Azure Active Directory 報告延遲](reference-reports-latencies.md)
