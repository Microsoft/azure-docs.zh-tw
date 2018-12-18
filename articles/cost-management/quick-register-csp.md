---
title: 使用 CSP 合作夥伴資訊向 Azure中的 Cloudyn 註冊 | Microsoft Docs
description: 使用您的 CSP 合作夥伴資訊向 Cloudyn 註冊。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/18/2018
ms.topic: quickstart
ms.custom: ''
ms.service: cost-management
manager: dougeby
ms.openlocfilehash: 698ddab4f5bca5aacf79cb952a51c385a67eee32
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46985425"
---
# <a name="register-with-the-csp-partner-program-and-view-cost-data"></a>註冊 CSP 合作夥伴計劃並檢視成本資料

如果您是 CSP 合作夥伴，可以向 Cloudyn 註冊。 註冊將可提供 Cloudyn 入口網站的存取權。 本快速入門詳細說明建立 Cloudyn 試用訂用帳戶並登入 Cloudyn 入口網站所需的註冊程序。 它也會示範如何立即開始檢視成本資料。


>[!NOTE]

>只有 CSP 直接協力廠商和 CSP 間接提供者可以完成 Cloudyn 註冊。
>
>驗證和資料存取需要設定合作夥伴中心 API。 需要合作夥伴中心全域系統管理員帳戶，才能佈建 API 存取權。
如需詳細資訊，請參閱[連線到合作夥伴中心 API](https://msdn.microsoft.com/library/partnercenter/mt709136.aspx)。
>
>CSP 間接轉售商向 Cloudyn 註冊其 CSP 間接提供者之後，即可取得 Cloudyn 的存取權。 CSP 間接轉售商可再將 Cloudyn 存取權提供給 Azure 客戶和訂用帳戶。

## <a name="sign-in-to-azure"></a>登入 Azure

- 在 http://portal.azure.com 登入 Azure 入口網站。

## <a name="register-with-cloudyn"></a>向 Cloudyn 註冊

1. 在 Azure 入口網站中，按一下服務清單中的 [成本管理與帳單]。
2. 在 [概觀] 下，按一下 [Cloudyn]  
    ![Cloudyn 頁面](./media/quick-register-csp/cost-mgt-billing-service.png)
3. 在 [Cloudyn] 頁面上，按一下 [前往 Cloudyn] 以在新視窗開啟 Cloudyn 註冊頁面。
4. 在 Cloudyn 入口網站試用註冊頁面上，輸入您的公司名稱，選取 [Microsoft CSP 合作夥伴計劃管理員]，然後按一下 [下一步]。  
5. 輸入 [應用程式識別碼]、[商務識別碼]、[應用程式祕密金鑰 ]，並選取 [預設定價方案]。 如果您無法立即取得該資訊，請使用您的主要系統管理員帳戶從 [https://partnercenter.microsoft.com](https://partnercenter.microsoft.com) 登入合作夥伴中心入口網站，並執行下列步驟：
  1. 移至 [儀表板]，然後依序按一下 [設定] 符號、[合作夥伴設定] 和 [應用程式管理]。
  2. 如果您曾經建立過 Web 應用程式，請略過此步驟。 否則，請按一下 [Web 應用程式] 區段中的 [新增 Web 應用程式]。
  3. 從您的 Web 應用程式複製 [應用程式識別碼] GUID。
  4. 從您的 Web 應用程式複製 [商務識別碼] GUID。
  5. 視需要選取一年或兩年的金鑰有效期限。 選取 [新增金鑰]，然後複製並儲存祕密金鑰的值。  
    ![CSP 合作夥伴中心](./media/quick-register-csp/csp-partner-center.png)
  6. 返回 Cloudyn 註冊頁面並貼上該資訊。  
      ![CSP 帳戶認證](./media/quick-register-csp/csp-reg.png)
6. 同意「使用條款」，然後驗證您的資訊。 按一下 [下一步] 以授權 Cloudyn 收集 Azure 資源資料。 收集的資料包括您訂用帳戶的使用量、效能、計費和標籤資料。  
7. 在 [邀請其他利害關係人] 下，您可以輸入電子郵件地址來新增使用者。 完成時，按一下 [下一步]。 您的所有帳單資料需要大約兩個小時才能新增到 Cloudyn。
8. 按一下 [移至 Cloudyn] 以開啟 Cloudyn 入口網站，然後就會在 [雲端帳戶管理] 頁面上看見已註冊的 CSP 帳戶資訊。

## <a name="configure-indirect-csp-access-in-cloudyn"></a>設定 Cloudyn 中的間接 CSP 存取

根據預設，只有直接 CSP 可以存取夥伴中心 API。 不過，直接 CSP 提供者可以使用 Cloudyn 中的實體群組，為其間接 CSP 客戶或合作夥伴設定存取。

若要啟用間接 CSP 客戶或合作夥伴的存取，依照[建立試用版註冊](#create-a-trial-registration)中的的步驟設定試用版註冊。 接下來，使用 Cloudyn 實體群組，完成下列步驟以分割間接 CSP 的資料。 然後，將適當的使用者權限指派給實體群組。

1. 在[建立實體](tutorial-user-access.md#create-and-manage-entities)提供資訊建立實體群組。
2. 遵循[將訂用帳戶指派給成本實體](https://support.cloudyn.com/hc/articles/115005139425-Video-Assigning-subscriptions-to-Cost-Entities)的步驟。 將間接 CSP 客戶的帳戶及其 Azure 訂用帳戶，與您先前建立的實體產生關聯。
3. 依照[建立具有系統管理員存取權的使用者](tutorial-user-access.md#create-a-user-with-admin-access)的步驟，建立具有系統管理員存取權的使用者帳戶。 接著，確定此使用者帳戶具有特定實體 (您先前為間接帳戶建立) 的系統管理員存取權。

間接 CSP 夥伴使用您為他們建立的帳戶登入 Cloudyn 入口網站。


[!INCLUDE [cost-management-create-account-view-data](../../includes/cost-management-create-account-view-data.md)]

## <a name="next-steps"></a>後續步驟

在此快速入門中，您會使用 CSP 資訊向 Cloudyn 註冊。 您也登入了 Cloudyn 入口網站，並開始檢視成本資料。 若要深入了解 Cloudyn，請繼續 Cloudyn 的教學課程。

> [!div class="nextstepaction"]
> [檢閱使用量和成本](./tutorial-review-usage.md)
