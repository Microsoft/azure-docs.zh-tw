---
title: 使用方式和見解報表 |Microsoft Docs
description: Azure Active Directory 入口網站中的使用方式和見解報表簡介
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 3fba300d-18fc-4355-9924-d8662f563a1f
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 05/13/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.openlocfilehash: 54bce5e839786862a6dac9aeb685dd364547a09a
ms.sourcegitcommit: b39cf769ce8e2eb7ea74cfdac6759a17a048b331
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/22/2021
ms.locfileid: "98685018"
---
# <a name="usage-and-insights-report-in-the-azure-active-directory-portal"></a>Azure Active Directory 入口網站中的使用方式和見解報表

使用使用量和見解報表，您可以取得以應用程式為中心的登入資料檢視。 您可以找到下列問題的答案：

*   我的組織中最常使用的應用程式是什麼？
*   哪些應用程式的登入失敗？ 
*   每個應用程式的最常見登入錯誤為何？

## <a name="prerequisites"></a>必要條件 

若要從使用方式和見解報表存取資料，您需要：

* Azure AD 租用戶
* Azure AD premium (P1/P2) 授權來查看登入資料
* 全域管理員、安全性系統管理員、安全性讀取者或報表讀取者角色中的使用者。 此外， (非系統管理員) 的任何使用者都可以存取自己的登入。 

## <a name="access-the-usage-and-insights-report"></a>存取使用量和見解報表

1. 瀏覽至 [Azure 入口網站](https://portal.azure.com)。
2. 選取正確的目錄，然後選取 [ **Azure Active Directory** 並選擇 [ **企業應用程式**]。
3. 從 [ **活動** ] 區段中，選取 [ **使用量 & 見解** ] 以開啟報表。 

![螢幕擷取畫面顯示從活動區段選取的使用量 & 見解。](./media/concept-usage-insights-report/main-menu.png)
                                     

## <a name="use-the-report"></a>使用報表

[使用量和見解] 報表會顯示有一或多個登入嘗試的應用程式清單，並可讓您依成功登入次數、失敗登入次數和成功率進行排序。

按一下清單底部的 [ **載入更多** ] 可以讓您在頁面上查看其他應用程式。 您可以選取日期範圍，以查看已在該範圍內使用的所有應用程式。

![螢幕擷取畫面顯示應用程式活動的使用量 & 見解，您可以在其中為不同的應用程式選取範圍和查看登入活動。](./media/concept-usage-insights-report/usage-and-insights-report.png)

您也可以在特定應用程式上設定焦點。 選取 [ **view 登入] 活動** ，以查看一段時間內的應用程式的登入活動，以及最常見的錯誤。  

當您在應用程式使用量圖表中選取一天時，會取得應用程式的登入活動的詳細清單。  

:::image type="content" source="./media/concept-usage-insights-report/usage-and-insights-application-report.png" alt-text="螢幕擷取畫面顯示特定應用程式的使用量 & 見解，您可以在其中查看登入活動的圖表。":::

## <a name="next-steps"></a>下一步

* [登入報告](concept-sign-ins.md)
