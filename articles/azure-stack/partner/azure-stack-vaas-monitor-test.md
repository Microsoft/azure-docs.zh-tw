---
title: 監視 Azure Stack 驗證即服務的測試 | Microsoft Docs
description: 監視 Azure Stack 驗證即服務的測試。
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 07/24/2018
ms.author: mabrigg
ms.reviewer: johnhas
ms.openlocfilehash: 553d2a0e4bf6b23f5d8ab200f533d9245bf72d36
ms.sourcegitcommit: f94f84b870035140722e70cab29562e7990d35a3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/30/2018
ms.locfileid: "43286579"
---
# <a name="monitor-a-test-with-azure-stack-validation-as-a-service"></a>監視 Azure Stack 驗證即服務的測試

[!INCLUDE[Azure_Stack_Partner](./includes/azure-stack-partner-appliesto.md)]

您可以對進行中或已完成的測試套件檢視 [作業] 頁面，以監視測試的執行情形。 此頁面會詳細列出測試的狀態及其作業。

## <a name="monitor-a-test"></a>監視測試

1. 選取解決方案。

2. 選取任何工作流程圖格上的 [管理]。

3. 按一下工作流程以開啟其測試摘要頁面。

4. 展開任何測試套件執行個體的內容功能表 **[...]**。

5. 選取 [檢視作業]

![替代文字](media\image4.png)

對於已執行完成的測試，您可以在測試的內容功能表 **[...]** 中按一下 [下載記錄]，從測試摘要頁面中下載記錄。Azure Stack 合作夥伴可以使用這些記錄對失敗的測試進行問題的偵錯。

## <a name="open-the-test-pass-summary"></a>開啟測試進行摘要

1. 開啟入口網站。 
2. 選取先前執行或已排程的測試所屬的現有解決方案名稱。

    ![管理測試進行](media/managetestpasses.png)

3. 選取 [測試進行] 面板中的 [管理]。
4. 選取測試進行以開啟測試進行摘要。 您可以檢閱測試名稱、建立日期、回合、測試耗時和結果 (成功或失敗)。
5. 選取 [**...** ]。

### <a name="test-pass-summary"></a>測試進行摘要

| 欄 | 說明 |
| --- | --- |
| 測試名稱 | 測試的名稱。 參考驗證數字。 |
| 建立時間 | 測試進行的建立時間。 |
| 已啟動 | 測試進行的執行時間。 |
| Duration | 執行測試進行所花費的時間長度。 |
| 狀態 | 測試進行的結果 (成功或失敗)。 |
| 代理程式名稱 | 代理程式的完整網域名稱。 |
| 作業總計 | 在測試進行中嘗試的作業總數。 |
| 通過的作業 | 在測試進行中通過的作業數目。 |
|  失敗的作業 | 失敗的作業數目。 |

### <a name="group-columns-in-the-test-pass-summary"></a>將測試進行摘要中的資料行分組

您可以選取資料行並將其拖曳至標頭中，以建立資料行值的群組。

## <a name="reschedule-a-test"></a>重新排程測試

1. [開啟測試進行摘要](#open-the-test-pass-summary)。
2. 選取 [重新排程]，以重新排程測試進行。
3. 輸入您 Azure Stack 執行個體的雲端管理員密碼。
4. 輸入您在設定帳戶時所定義的診斷儲存體連接字串。
5. 重新排程測試。

## <a name="cancel-a-test"></a>取消測試

1. [開啟測試進行摘要](#open-the-test-pass-summary)。
2. 選取 [取消]。

## <a name="get-test-information"></a>取得測試資訊

1. [開啟測試進行摘要](#open-the-test-pass-summary)。
2. 選取 [檢視資訊] 以重新排程測試進行。

**測試資訊**

| 名稱 | 說明 |
| -- | -- |
| 測試名稱 | 測試的名稱，例如 OEM Update on Azure Stack 1806 RC Validation。 |
| 測試版本 | 測試的版本，例如 5.1.4.0。 |
| 發行者 | 測試發行者，例如 Microsoft。 |
| 類別 | 測試的類別，例如**功能性**或**可靠性**。 |
| 目標服務 | 正在測試的服務，例如 VirtualMachines |
| 說明 | 測試的描述。 |
| 預估持續時間 (分鐘) | 執行測試所花費的時間長度 (以分鐘為單位)。 |
| 連結 | GitHub 問題追蹤程式的連結。 |

## <a name="get-test-parameters"></a>取得測試參數

1. [開啟測試進行摘要](#open-the-test-pass-summary)。
2. 選取 [檢視參數] 以重新排程測試進行。

**參數**

| 名稱 | 說明 |
| -- | -- |
| 測試名稱 | 測試的名稱，例如 oemupdate1806test。 |
| 測試版本 | 測試的版本，例如 5.1.4.0。 |
| 測試執行個體識別碼 | 用來識別特定測試執行個體的 GUID，例如 20b20645-b400-4f0d-bf6f-1264d866ada9。 |
| cloudAdminUser | 作為雲端管理員的帳戶名稱，例如 **cloudadmin**。 |
| DiagnosticsContainerName | 診斷容器的識別碼，例如 04dd3815-5f35-4158-92ea-698027693080。 |

## <a name="get-test-operations"></a>取得測試作業

1. [開啟測試進行摘要](#open-the-test-pass-summary)。
2. 選取 [檢視作業] 以重新排程測試進行。 作業摘要窗格隨即開啟。

## <a name="get-test-logs"></a>取得測試記錄

1. [開啟測試進行摘要](#open-the-test-pass-summary)。
2. 選取 [下載記錄] 以重新排程測試進行。  
    系統會下載名為 ReleaseYYYY-MM-DD.zip 且包含記錄的 Zip 檔案。

## <a name="next-steps"></a>後續步驟

- 深入了解 [Azure Stack 驗證即服務](https://docs.microsoft.com/azure/azure-stack/partner)。
