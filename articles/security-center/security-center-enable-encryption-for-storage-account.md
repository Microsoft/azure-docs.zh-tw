---
title: 在 Azure 資訊安全中心啟用儲存體帳戶的加密 | Microsoft Docs
description: 本文件說明如何實作 Azure 資訊安全中心建議：**啟用 Azure 儲存體帳戶的加密**。
services: security-center
documentationcenter: na
author: TerryLanfear
manager: MBaldwin
editor: ''
ms.assetid: ''
ms.service: security-center
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/20/2016
ms.author: terrylan
ms.openlocfilehash: 82bb201c0b518d0b45e06a1eb25d54f60cb3e028
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 03/28/2018
ms.locfileid: "30235014"
---
# <a name="enable-encryption-for-azure-storage-account-in-azure-security-center"></a>在 Azure 資訊安全中心啟用 Azure 儲存體帳戶的加密
Azure 資訊安全中心可能建議您對待用資料啟用 Azure 儲存體服務加密。

儲存體服務加密 (SSE) 會在資料寫入 Azure 儲存體時加密資料，並於擷取資料之前解密。  SSE 目前僅適用於 Azure Blob 服務，可用於區塊 Blob、分頁 Blob 和附加 Blob。  若要深入了解，請參閱[待用資料的儲存體服務加密](../storage/common/storage-service-encryption.md)。


> [!Note]
> 啟用加密之後，只有新的資料會加密。 儲存體帳戶中任何現有的 blob 保持未加密。 若要加密現有的 blob，請參閱[儲存體服務加密常見問題集](../storage/common/storage-service-encryption.md#faq-for-storage-service-encryption)。
>
>

只有 Resource Manager 儲存體帳戶才支援儲存體服務加密。 目前不支援傳統儲存體帳戶。 若要了解傳統和 Resource Manager 部署模型，請參閱 [Azure 部署模型](../azure-classic-rm.md)。

> [!NOTE]
> 本文件將使用範例部署來介紹服務。  本文件不是一份逐步解說指南。
>
>

## <a name="implement-the-recommendation"></a>實作建議
1. 在 [建議] 刀鋒視窗中，選取 [啟用 Azure 儲存體帳戶的加密]。
   ![啟用儲存體帳戶的加密][1]
2. [啟用儲存體加密] 刀鋒視窗隨即開啟。 此刀鋒視窗會列出已停用儲存體加密的 Azure 儲存體帳戶。 在此範例中，我們選取 [storageacct1]。
   ![啟用儲存體加密][2]
3. **storageacct1** 的 [加密] 刀鋒視窗隨即開啟。 選取 [啟用] 。
   ![加密刀鋒視窗][3]
4. 選取 [ **儲存**]。

您現在已啟用 **storageacct1** 的儲存體加密。


## <a name="see-also"></a>另請參閱
本文件說明如何實作資訊安全中心建議：「啟用 Azure 儲存體帳戶的加密」。 若要深入了解 Azure 儲存體服務加密，請參閱下列主題：

* [待用資料的 Azure 儲存體服務加密](../storage/common/storage-service-encryption.md)

如要深入了解資訊安全中心，請參閱下列主題：

* [在 Azure 資訊安全中心設定安全性原則](security-center-policies.md) - 了解如何為您的 Azure 訂用帳戶及資源群組設定安全性原則。
* [Azure 資訊安全中心的安全性健全狀況監視](security-center-monitoring.md) - 了解如何監視 Azure 資源的健全狀況。
* [在 Azure 資訊安全中心管理與回應安全性警示](security-center-managing-and-responding-alerts.md) - 了解如何管理與回應安全性警示。
* [在 Azure 資訊安全中心管理安全性建議](security-center-recommendations.md) - 了解建議如何協助您保護您的 Azure 資源。
* [Azure 資訊安全中心常見問題集](security-center-faq.md) - 尋找有關使用服務的常見問題。
* [Azure 安全性部落格](http://blogs.msdn.com/b/azuresecurity/) - 尋找有關 Azure 安全性與合規性的部落格文章。

<!--Image references-->
[1]: ./media/security-center-enable-encryption-for-storage-account/enable-encryption-for-storage-account.png
[2]: ./media/security-center-enable-encryption-for-storage-account/enable-storage-encryption.png
[3]: ./media/security-center-enable-encryption-for-storage-account/encryption-blade.png
