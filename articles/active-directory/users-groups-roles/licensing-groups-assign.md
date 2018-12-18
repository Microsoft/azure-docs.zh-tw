---
title: 將授權指派給 Azure Active Directory 中的群組 | Microsoft Docs
description: 如何使用 Azure Active Directory 群組授權的方式將授權指派給使用者
services: active-directory
keywords: Azure AD 授權
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: article
ms.workload: identity
ms.component: users-groups-roles
ms.date: 06/05/2017
ms.author: curtand
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 7a235851d7172d32d62c64b163e0b7635a1a47fd
ms.sourcegitcommit: 0b4da003fc0063c6232f795d6b67fa8101695b61
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/05/2018
ms.locfileid: "37861188"
---
# <a name="assign-licenses-to-users-by-group-membership-in-azure-active-directory"></a>依據 Azure Active Directory 中的群組成員資格將授權指派給使用者

這篇文章會逐步引導您將產品授權指派給 Azure Active Directory (Azure AD) 中的使用者群組，然後確認他們正確獲得授權。

在本範例中，租用戶包含名為「人力資源部門」的安全性群組。 這個群組包括人力資源部門的所有成員 (大約是 1,000 位使用者)。 您想要將 Office 365 Enterprise E3 授權指派給整個部門。 產品中包含的 Yammer Enterprise 服務需要暫時停用，直到部門準備好要開始使用它為止。 您也要將 Enterprise Mobility + Security 授權部署到相同群組的使用者。

> [!NOTE]
> 並非所有位置都可使用某些 Microsoft 服務。 系統管理員必須指定使用者的使用位置屬性，才能將授權指派給使用者。

> 如果是群組授權指派，未指定使用位置的任何使用者會繼承目錄的位置。 如果您在多個位置擁有使用者，我們建議您一律將使用位置設為 Azure AD 中使用者建立流程的一部分 (例如，透過 AAD Connect 設定) - 這可確保授權指派的結果一律是正確的，且使用者不會在不允許的位置收到服務。

## <a name="step-1-assign-the-required-licenses"></a>步驟 1︰指派所需的授權

1. 使用系統管理員帳戶登入 [**Azure 入口網站**](https://portal.azure.com)。 若要管理授權，帳戶必須是全域管理員角色或使用者帳戶管理員角色。

2. 選取左瀏覽窗格中的 [所有服務]，然後選取 [Azure Active Directory]。 您可以將此窗格新增至 [我的最愛]，或將其釘選到入口網站儀表板。

3. 在 [Azure Active Directory] 窗格中，選取 [授權] 以開啟另一個窗格，您可以在其中查看及管理租用戶中的所有可授權產品。

4. 在 [所有產品] 下，選取產品名稱，以選取 Office 365 Enterprise E3 和 Enterprise Mobility + Security。 若要啟動指派，選取窗格頂端的 [指派]。

   ![所有產品、指派授權](./media/licensing-groups-assign/all-products-assign.png)

5. 在 [指派授權] 窗格中，按一下 [使用者和群組] 以開啟 [使用者和群組] 窗格。 搜尋群組名稱人力資源部門、選取群組，然後請務必按一下窗格底部的 [選取] 來確認。

   ![選取群組](./media/licensing-groups-assign/select-a-group.png)

6. 在 [指派授權] 窗格中，按一下 [指派選項 (選用)]，便會顯示我們先前選取的兩個產品中所包含的所有服務方案。 尋找 **Yammer Enterprise** 並將它**關閉**，以從產品授權停用該服務。 按一下 [指派選項] 底部的 [確定] 來確認。

   ![指派選項](./media/licensing-groups-assign/assignment-options.png)

7. 若要完成指派，在 [指派授權] 窗格中，按一下窗格底部的 [指派]。

8. 右上角顯示的通知會顯示程序的狀態和結果。 如果無法完成指派給群組 (例如，因為群組中已存在的授權)，按一下通知以檢視失敗的詳細資料。

我們現在已指定「人力資源部門」群組的授權範本。 在 Azure AD 中的背景處理已開始處理該群組的所有現有成員。 此初始作業可能需要一些時間，依群組的目前大小而定。 下一個步驟將說明如何確認處理程序已完成，並且決定是否需要進一步注意以解決問題。

> [!NOTE]
> 您可以從 Azure AD 中的其他位置︰**使用者和群組**啟動相同的指派。 移至 [Azure Active Directory] > [使用者和群組] > [所有群組]。 然後尋找群組並加以選取，而後移至 [授權] 索引標籤。窗格頂端的 [指派] 按鈕會開啟授權指派窗格。

## <a name="step-2-verify-that-the-initial-assignment-has-finished"></a>步驟 2︰確認已完成初始指派

1. 移至 [Azure Active Directory] > [使用者和群組] > [所有群組]。 然後尋找獲得授權指派的「人力資源部門」 群組。

2. 在 [人力資源部門] 群組窗格中，選取 [授權]。 可讓您快速確認授權是否已完全指派給使用者，以及是否有任何錯誤需要探究。 可用資訊如下：

   - 目前指派給群組的產品授權清單。 選取項目來顯示已啟用且要進行變更的特定服務。

   - 對群組進行的最新授權變更狀態 (變更是否正在處理，或是否已完成所有使用者成員的處理)。

   - 因為無法獲得授權指派而處於錯誤狀態的使用者相關資訊。

   ![指派選項](./media/licensing-groups-assign/assignment-errors.png)

3. 如需授權處理的詳細資訊，請參閱 [Azure Active Directory] > [使用者和群組] > 群組名稱 > [稽核記錄]。 請注意下列活動：

   - 活動︰**開始將群組型授權套用至使用者**。 當我們的系統拾取群組的授權指派變更，並開始將其套用到所有使用者成員時，就會進行記錄。 它包含已進行之變更的相關資訊。

   - 活動︰**完成將群組型授權套用至使用者**。 當系統完成處理群組中的所有使用者時，就會進行記錄。 它包含已成功處理的使用者人數和無法獲得群組授權指派的使用者人數之摘要。

   [閱讀本節](licensing-group-advanced.md#use-audit-logs-to-monitor-group-based-licensing-activity)以深入了解如何使用稽核記錄，以分析群組型授權所做的變更。

## <a name="step-3-check-for-license-problems-and-resolve-them"></a>步驟 3︰檢查授權問題及解決這些問題

1. 移至 [Azure Active Directory] > [使用者和群組] > [所有群組]，然後尋找獲得授權指派的「人力資源部門」群組。
2. 在 [人力資源部門] 群組窗格中，選取 [授權]。 窗格頂端的通知顯示有 10 位使用者無法獲得授權指派。 按一下通知，隨即開啟一份此群組為授權錯誤狀態的所有使用者。
3. [失敗的指派] 資料行告訴我們，兩個產品授權都無法指派給使用者。 [失敗的前幾大原因] 資料行包含失敗的原因。 在此案例中為 [衝突的服務方案]。

   ![失敗的指派](./media/licensing-groups-assign/failed-assignments.png)

4. 選取使用者以開啟 [授權] 窗格。 此窗格會顯示目前指派給此使用者的所有授權。 在此範例中，使用者擁有繼承自 **Kiosk 使用者** 群組的 Office 365 Enterprise E1 授權。 這會與系統嘗試從**人力資源部門**群組套用的 E3 授權衝突。 如此一來，該群組沒有任何授權會指派給使用者。

   ![檢視使用者的授權](./media/licensing-groups-assign/user-license-view.png)

5. 若要解決這個衝突，從 **Kiosk 使用者**群組移除使用者。 在 Azure AD 處理變更之後，會正確指派**人力資源部門**授權。

   ![已正確指派的授權](./media/licensing-groups-assign/license-correctly-assigned.png)

## <a name="next-steps"></a>後續步驟

若要深入了解透過群組管理授權的功能集，請參閱下列文章：

* [什麼是 Azure Active Directory 中以群組為基礎的授權？](../fundamentals/active-directory-licensing-whatis-azure-portal.md)
* [識別及解決 Azure Active Directory 中群組的授權問題](licensing-groups-resolve-problems.md)
* [如何將個別的已授權使用者移轉成 Azure Active Directory 中的群組型授權 (英文)](licensing-groups-migrate-users.md)
* [Azure Active Directory 群組型授權其他案例 (英文)](../active-directory-licensing-group-advanced.md)
