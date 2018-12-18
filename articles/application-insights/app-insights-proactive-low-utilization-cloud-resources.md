---
title: 智慧偵測 - Azure Application Insights 偵測到雲端資源的低使用率 | Microsoft Docs
description: 使用 Azure Application Insights 監視應用程式是否有雲端資源低使用率。
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
ms.openlocfilehash: ca4f944f605db96a2cedf2682f3ff4c811007ffb
ms.sourcegitcommit: cc4fdd6f0f12b44c244abc7f6bc4b181a2d05302
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/25/2018
ms.locfileid: "47094907"
---
# <a name="low-cpu-utilization-in-cloud-resources-preview"></a>雲端資源的低 CPU 使用率 (預覽)

Application Insights 會自動分析應用程式中每個角色執行個體的 CPU 消耗，並且偵測低 CPU 使用率的執行個體。 這項偵測可讓您減少您的 Azure 資源並降低成本，方法是降低每個角色利用的角色執行個體數目，或是降低角色數目。

這項功能不需要任何特殊的設定，除了為您的應用程式[設定效能計數器](https://docs.microsoft.com/azure/application-insights/app-insights-performance-counters)以外。 當您的應用程式產生足夠的 CPU 效能計數器遙測 (處理器時間百分比) 時，它是有效的。

## <a name="when-would-i-get-this-type-of-smart-detection-notification"></a>何時會取得這個類型的智慧偵測通知？
許多 Web/背景工作角色執行個體呈現低 CPU 使用率時會發生典型通知。

## <a name="does-my-app-definitely-consume-too-many-resources"></a>我的應用程式一定是耗用太多資源嗎？
否，通知並不表示您的應用程式一定是耗用太多資源。 雖然這類模式的低 CPU 使用率通常表示可以降低資源耗用量，但是這種行為對於您的特定角色可能很常見，或者可能有業務上的正當理由，可以忽略。 例如，可能是其他資源需要多個執行個體，例如記憶體/網路，而不是 CPU。

## <a name="how-do-i-fix-it"></a>如何修正問題？
通知包括要在診斷程序中支援的診斷資訊：
1. **分級。** 通知會顯示應用程式中呈現低 CPU 使用率的角色。 這可協助您將優先順序指派給此問題。
2. **範圍。** 多少角色呈現低 CPU 使用率，以及每個角色中有多少執行個體利用低 CPU？ 可以從通知取得這項資訊。
3. **診斷。** 偵測包含利用的 CPU 百分比，顯示經過一段時間之每個執行個體的 CPU 使用率。 您也可以使用連結到支援資訊的相關項目和報表，例如 CPU 使用率的百分位數，協助您進一步診斷問題。
