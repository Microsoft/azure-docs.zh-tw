---
title: 智慧偵測 - 使用 Azure Application Insights 的智慧偵測套件 | Microsoft Docs
description: 使用 Azure Application Insights 監視應用程式的潛在安全性問題。
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: ea2a28ed-4cd9-4006-bd5a-d4c76f4ec20b
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: conceptual
ms.date: 12/12/2017
ms.author: mbullwin
ms.openlocfilehash: edfe2956da84347abf3e5da7a7d0fd448361c7f5
ms.sourcegitcommit: e8f443ac09eaa6ef1d56a60cd6ac7d351d9271b9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/12/2018
ms.locfileid: "35632542"
---
# <a name="application-security-detection-pack-preview"></a>應用程式安全性偵測套件 (預覽)

Application Insights 會自動分析由您的應用程式所產生的遙測，並且偵測潛在的安全性問題。 這項功能可讓您識別潛在的安全性問題，並且藉由修正應用程式或採取必要的安全性措施來處理它們。

這項功能不需要任何特殊的設定，除了[設定您的應用程式傳送遙測](https://docs.microsoft.com/azure/application-insights/app-insights-usage-overview)以外。

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>何時會取得這個類型的智慧偵測通知？
會偵測三種類型的安全性問題：
1. 不安全的 URL 存取：有人透過 HTTP 和 HTTPS 存取應用程式中的 URL。 一般而言，接受 HTTPS 要求的 URL 不應該接受 HTTP 要求。 這可能表示您的應用程式有錯誤或安全性問題。
2. 不安全的表單：應用程式中的表單 (或其他 "POST" 要求) 使用 HTTP 而非 HTTPS。 使用 HTTP 可能會危及表單傳送的使用者資料。
3. 可疑的使用者活動：同一個使用者在大約相同的時間從多個國家/地區存取應用程式。 例如，同一個使用者在同一小時內從西班牙和美國存取應用程式。 此偵測指出對您的應用程式有潛在的惡意存取嘗試。

## <a name="does-my-app-definitely-have-a-security-issue"></a>我的應用程式一定會有安全性問題嗎？
否，通知並不表示您的應用程式一定有安全性問題。 在許多情況下，上述任何案例的偵測可能代表有安全性問題。 不過，偵測可能只是基於商務理由，您可以忽略。

## <a name="how-do-i-fix-the-insecure-url-access-detection"></a>如何修正「不安全的 URL 存取」偵測？
1. **分級。** 通知會提供存取不安全之 URL 的使用者數目，以及受到不安全的存取影響最深的 URL。 這可協助您將優先順序指派給此問題。
2. **範圍。** 存取不安全之 URL 的使用者的百分比是多少？ 有多少 URL 受到影響？ 可以從通知取得這項資訊。
3. **診斷。** 偵測會提供不安全的要求清單，以及受到影響之 URL 和使用者的清單，協助您進一步診斷問題。

## <a name="how-do-i-fix-the-insecure-form-detection"></a>如何修正「不安全的表單」偵測？
1. **分級。** 通知會提供不安全的表單數目，及其資料可能受到危害的使用者數目。 這可協助您將優先順序指派給此問題。
2. **範圍。** 哪個表單牽涉到最大的不安全傳輸數目，以及一段時間的不安全傳輸散佈如何？ 可以從通知取得這項資訊。
3. **診斷。** 偵測會提供不安全的表單清單以及每個表單之不安全傳輸數目細分，協助您進一步診斷問題。

## <a name="how-do-i-fix-the-suspicious-user-activity-detection"></a>如何修正「可疑的使用者活動」偵測？
1. **分級。** 通知會提供展現可疑行為的不同使用者數目。 這可協助您將優先順序指派給此問題。
2. **範圍。** 可疑要求來自哪個國家/地區？ 哪個使用者最可疑？ 可以從通知取得這項資訊。
3. **診斷。** 偵測會提供可疑使用者清單和每個使用者的國家/地區清單，協助您進一步診斷問題。
