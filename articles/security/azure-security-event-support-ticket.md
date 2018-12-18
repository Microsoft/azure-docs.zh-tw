---
title: 如何記錄安全性事件支援票證 - Azure | Microsoft Docs
description: 作為 Azure Marketplace 的賣方，在識別出潛在的安全性事件之後，我需要知道如何記錄適當的票證。
services: security
documentationcenter: na
author: DavidBosland
manager: lakoch
editor: v-dabosl
ms.assetid: f1ffde66-98f0-4c3e-ad94-fee1f97cae03
ms.service: marketplace
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/13/2017
ms.author: v-dabosl
ms.openlocfilehash: 753d6abcdd063d5c092b0c90770396d176bc52e8
ms.sourcegitcommit: b854df4fc66c73ba1dd141740a2b348de3e1e028
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 12/04/2017
ms.locfileid: "26129935"
---
# <a name="how-to-log-a-security-event-support-ticket"></a>如何記錄安全性事件支援票證

1. 瀏覽至[發行者支援](https://support.microsoft.com/en-us/getsupport?wf=0&tenant=ClassicCommercial&oaspworkflow=start_1.0.0.0&locale=en-us&supportregion=en-us&pesid=16230&ccsid=636450758943226673) \(英文\)，並以您的 Microsoft 認證登入。
2. 選取 [Security Event] \(安全性事件\) 作為 [Problem Type] \(問題類型\)，並在 [Security Incident] \(安全性事件\) 和 [Vulnerability] \(弱點) 類別之間進行選擇。

    ![事件類型和定義][1]

3. 當您選取 [Problem Type] \(問題類型\) 和 [Category] \(類別\) 之後，按一下 [Start request] \(提出要求\) 按鈕。 提供下列資訊，以協助我們更了解問題。

    i. 問題和/或弱點是什麼？

    ii. 針對弱點，請提供 CVE (mitre.org) 或已填寫的 CVSS3 v3 計算機 (https://www.first.org/cvss/calculator/3.0)。

    iii. 有任何解決方式或緩解功能嗎？ 如果有，則請提供補救步驟。

    iv. 您有想要傳送給客戶的訊息嗎？ 我們將適時協助您建立適當的訊息。

4. 提交確認：一旦您提交問題之後，我們將在一個工作天內確認接收，並指派問題的優先順序和嚴重性。

    - 如果您需要與我們溝通您的問題，請在所有通信聯繫中包括該確認號碼。
    - 您可以隨時檢視您的問題進度。

5. 接下來會發生什麼狀況？ 根據問題和嚴重性，可能會採取下列步驟：

    - 我們將與您溝通我們的評定結果。 我們可能會根據結果，來移除或要求您修改您的供應項目。 在此情況下，我們將與您一起確保會使受影響客戶的中斷情況最小化。
    - 我們將協助您緩解此事件/弱點對於我們共同客戶的影響。


[1]: ./media/azure-security-event-support-ticket/chart.png
