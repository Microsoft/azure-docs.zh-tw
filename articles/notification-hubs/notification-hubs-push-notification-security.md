---
title: 通知中樞的安全性
description: 本主題說明 Azure 通知中樞的安全性。
services: notification-hubs
documentationcenter: .net
author: dimazaid
manager: kpiteira
editor: spelluru
ms.assetid: 6506177c-e25c-4af7-8508-a3ddca9dc07c
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: mobile-multiple
ms.devlang: multiple
ms.topic: article
ms.date: 04/14/2018
ms.author: dimazaid
ms.openlocfilehash: 9f197a85dfad31ce32d0f9c93127b69d8e33c9ee
ms.sourcegitcommit: e221d1a2e0fb245610a6dd886e7e74c362f06467
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 05/07/2018
ms.locfileid: "33778268"
---
# <a name="security"></a>安全性
## <a name="overview"></a>概觀
本主題說明 Azure 通知中樞的安全性模型。 因為通知中樞是服務匯流排實體，所以會實作與服務匯流排相同的安全性模型。 如需詳細資訊，請參閱 [服務匯流排驗證](https://msdn.microsoft.com/library/azure/dn155925.aspx) 的各項主題。

## <a name="shared-access-signature-security-sas"></a>共用存取簽章的安全性 (SAS)
通知中樞會實作實體層級的安全性配置，稱為 SAS (共用存取簽章)。 此配置讓傳訊實體最多可以在其授與該實體權限的描述中宣告 12 項規則。

如＜安全性宣告＞一節所述，每項規則都包含名稱、索引鍵值 (共用密碼) 及一組權限。 建立通知中樞時，會自動建立兩項規則：一項具有接聽權限 (供用戶端應用程式使用)，另一項則具有所有權限 (供應用程式後端使用)。

從用戶端應用程式執行註冊管理時，若透過通知傳送的資訊不是敏感性資訊 (例如氣象更新)，常見存取通知中樞的方法是將只具接聽存取權限之規則的索引鍵值，授與用戶端應用程式，並將該規則之完整存取權限的索引鍵值，授與應用程式後端。

不建議您將索引鍵值內嵌在 Windows 市集用戶端應用程式中。 避免將索引鍵值內嵌在用戶端應用程式中的方法，是讓用戶端應用程式在啟動時，再從應用程式後端擷取此值。

請務必了解，具有接聽權限的索引鍵可以讓用戶端應用程式註冊任何標記。 若您的應用程式必須限制特定用戶端 (例如，當標記代表使用者識別碼時) 的註冊，您的應用程式後端就必須執行該註冊。 如需詳細資訊，請參閱＜註冊管理＞。 請注意，如此一來，用戶端應用程式將無法直接存取通知中樞。

## <a name="security-claims"></a>安全性宣告
與其他實體類似，通知中樞作業也可有接聽、傳送及管理三種安全性宣告。

| 宣告 | 說明 | 允許的作業 |
| --- | --- | --- |
| 接聽 |建立/更新、讀取及刪除單一註冊 |建立/更新註冊<br><br>讀取註冊<br><br>讀取控制代碼的所有註冊<br><br>刪除註冊 |
| 傳送 |將訊息傳送到通知中樞 |傳送訊息 |
| 管理 |對通知中樞執行 CRUD (包含更新 PNS 認證及安全性金鑰) 並依標記讀取的註冊 |建立/更新/讀取/刪除通知中樞<br><br>依標記讀取註冊 |

通知中樞接受授 Microsoft Azure 存取控制權杖所授與的宣告，以及直接在通知中樞設定，由共用金鑰所產生之簽章權杖所授與的宣告。

