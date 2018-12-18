---
title: 如何將 Azure 訂用帳戶連結到 Azure Active Directory B2C | Microsoft Docs
description: 啟用 Azure 訂用帳戶內 Azure AD B2C 租用戶計費的逐步指南。
services: active-directory-b2c
author: davidmu1
manager: mtillman
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.date: 12/05/2017
ms.author: davidmu
ms.component: B2C
ms.openlocfilehash: 80ba42d7eab1726c7add6c4c9426b7dde3b55480
ms.sourcegitcommit: 86cb3855e1368e5a74f21fdd71684c78a1f907ac
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/03/2018
ms.locfileid: "37445924"
---
# <a name="linking-an-azure-subscription-to-an-azure-ad-b2c-tenant"></a>將 Azure 訂用帳戶連結至 Azure AD B2C 租用戶

> [!IMPORTANT]
> Azure AD B2C 的最新使用量計費和價格資訊位於下列網頁︰[Azure AD B2C 價格](https://azure.microsoft.com/pricing/details/active-directory-b2c/)

Azure AD B2C 的使用費用會計入 Azure 訂用帳戶。 建立 Azure AD B2C 租用戶後，租用戶系統管理員必須明確地將 Azure AD B2C 租用戶連結至 Azure 訂用帳戶。 本文將說明如何做到。

> [!NOTE]
> 連結至 Azure AD B2C 租用戶的訂用帳戶只能用於 Azure AD B2C 使用量計費。 此訂用帳戶無法用來*在 Azure AD B2C 租用戶內*新增其他 Azure 服務或 Office 365 授權。

 在目標 Azure 訂用帳戶內建立 Azure AD B2C「資源」，即可達成此訂用帳戶連結。 在單一 Azure 訂用帳戶內可以建立許多 Azure AD B2C「資源」，以及其他 Azure 資源 (例如，VM、資料儲存體、LogicApps)。 移至與訂用帳戶相關聯的 Azure AD 租用戶，您即可看到訂用帳戶內的所有資源。

需要有效的 Azure 訂用帳戶才能繼續執行。

## <a name="create-an-azure-ad-b2c-tenant"></a>建立 Azure AD B2C 租用戶

您必須先[建立 Azure AD B2C 租用戶](active-directory-b2c-get-started.md)，以便與訂用帳戶連結。 如果您已經建立 Azure AD B2C 租用戶，請略過此步驟。

## <a name="open-azure-portal-in-the-azure-ad-tenant-that-shows-your-azure-subscription"></a>在 Azure AD 租用戶中開啟 Azure 入口網站，以顯示您的 Azure 訂用帳戶

瀏覽至 Azure AD 租用戶，以顯示您的 Azure 訂用帳戶。 開啟 [Azure 入口網站](https://portal.azure.com)，並切換至 Azure AD 租用戶，以顯示您想要使用的 Azure 訂用帳戶。

![切換至 Azure AD 租用戶](./media/active-directory-b2c-how-to-enable-billing/SelectAzureADTenant.png)

## <a name="find-azure-ad-b2c-in-the-azure-marketplace"></a>在 Azure Marketplace 中尋找 Azure AD B2C

按一下 [建立資源] 按鈕。 在 [搜尋 Marketplace] 欄位中，輸入 `B2C`。

![在 [搜尋 Marketplace] 欄位中新增醒目提示的按鈕和 Azure AD B2C 文字](../../includes/media/active-directory-b2c-create-tenant/find-azure-ad-b2c.png)

在結果清單中選取 [Azure AD B2C]。

![結果清單中選取的 Azure AD B2C](../../includes/media/active-directory-b2c-create-tenant/find-azure-ad-b2c-result.png)

系統會顯示有關 Azure AD B2C 的詳細資訊。 若要開始設定新的 Azure Active Directory B2C 租用戶，請按一下 [建立] 按鈕。

在資源建立畫面中，選取 [將現有的 Azure AD B2C 租用戶連結至我的 Azure 訂用帳戶]。

## <a name="create-an-azure-ad-b2c-resource-within-the-azure-subscription"></a>在 Azure 訂用帳戶內建立 Azure AD B2C 資源

在資源建立對話方塊中，從下拉式清單中選取 Azure AD B2C 租用戶。 您會看到您身為全域管理員之下的所有租用戶，以及尚未連結至訂用帳戶的租用戶。

系統將會預先選取 Azure AD B2C 資源名稱，以符合 Azure AD B2C 租用戶的網域名稱。

針對 [訂用帳戶]，選取您身為管理員的作用中 Azure 訂用帳戶。

選取資源群組和資源群組位置。 這裡的選取項目對於 AD B2C 租用戶位置、效能或計費狀態沒有任何影響。

![建立 B2C 資源](./media/active-directory-b2c-how-to-enable-billing/createresourceb2c.png)

## <a name="manage-your-azure-ad-b2c-tenant-resources"></a>管理 Azure AD B2C 租用戶資源

在 Azure 訂用帳戶內成功建立 Azure AD B2C 資源後，您應會看到「B2C 租用戶」類型的新資源隨著其他 Azure 資源一起新增。

您可以使用此資源來：

- 瀏覽至訂用帳戶，以檢閱計費資訊。
- 移至您的 Azure AD B2C 租用戶
- 提交支援要求
- 將 Azure AD B2C 租用戶資源移至另一個 Azure 訂用帳戶，或另一個資源群組。

![B2C 資源設定](./media/active-directory-b2c-how-to-enable-billing/b2cresourcesettings.png)

## <a name="known-issues"></a>已知問題

### <a name="csp-subscriptions"></a>CSP 訂用帳戶

目前，Azure AD B2C 租用戶**無法**連結至 CSP 訂用帳戶。

### <a name="self-imposed-restrictions"></a>自行加入的限制

使用者可能針對建立 Azure 資源建立了區域限制。 此限制可能會導致 Azure AD B2C 資源無法建立。 若要減輕此問題的影響，請放寬這項限制。

## <a name="next-steps"></a>後續步驟

為每個 Azure AD B2C 租用戶完成這些步驟後，您的 Azure 訂用帳戶就會根據 Azure Direct 或 Enterprise 合約詳細資料計費。

您可以在選取的 Azure 訂用帳戶中，檢閱使用量和計費詳細資料。 您也可以使用[使用量報告 API](active-directory-b2c-reference-usage-reporting-api.md) 來檢閱詳細的每天使用量報告。
