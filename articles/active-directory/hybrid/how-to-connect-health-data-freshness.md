---
title: Azure AD Connect 健康情況 - 健康情況服務的資料並非最新警示 | Microsoft Docs
description: 本文說明「健康情況服務的資料並非最新」警示的成因，以及如何進行疑難排解。
services: active-directory
documentationcenter: ''
author: zhiweiw
manager: maheshu
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/26/2018
ms.author: zhiweiw
ms.openlocfilehash: 430ea5f0a6f737d7632a4352c24d893368b80558
ms.sourcegitcommit: cf606b01726df2c9c1789d851de326c873f4209a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46310377"
---
# <a name="health-service-data-is-not-up-to-date-alert"></a>健康情況服務的資料並非最新警示

## <a name="overview"></a>概觀
<li>當 Azure AD Connect Health 在兩小時內未從伺服器收到所有資料點，就會產生資料更新警示。 此警示的標題為**健康情況服務的資料並非最新**。 </li>
<li>如果 Connect Health 在兩小時內未接收到從伺服器傳送的部分資料元素，就會引發 [警告] 狀態警示。 警告狀態警示不會觸發傳送給租用戶管理員的電子郵件通知。 </li>
<li>如果 Connect Health 在兩小時內未接收到從伺服器傳送的任何資料元素，就會引發 [錯誤] 狀態警示。 錯誤狀態警示會觸發傳送給租用戶管理員的電子郵件通知。 </li>

>[!IMPORTANT] 
> 此警示會遵循 Connect Health 的[資料保留原則](reference-connect-health-user-privacy.md#data-retention-policy)

## <a name="troubleshooting-steps"></a>疑難排解步驟 
* 確實詳讀[需求小節](how-to-connect-health-agent-install.md#requirements)，並符合相關要求。
* 使用[測試連線工具](how-to-connect-health-agent-install.md#test-connectivity-to-azure-ad-connect-health-service)找出連線問題。


## <a name="next-steps"></a>後續步驟
* [Azure AD Connect Health 常見問題集](reference-connect-health-faq.md)
