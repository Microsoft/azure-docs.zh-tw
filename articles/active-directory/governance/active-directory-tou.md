---
title: Azure Active Directory 使用規定 | Microsoft Docs
description: Azure AD 使用規定可讓您和貴公司提供 Azure AD 服務的使用規定。
services: active-directory
author: rolyon
manager: mtillman
editor: ''
ms.assetid: d55872ef-7e45-4de5-a9a0-3298e3de3565
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.component: compliance
ms.date: 09/04/2018
ms.author: rolyon
ms.openlocfilehash: b6cc81a49875739a61aa397a65eb70c6b4f082ef
ms.sourcegitcommit: b4a46897fa52b1e04dd31e30677023a29d9ee0d9
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49393978"
---
# <a name="azure-active-directory-terms-of-use-feature"></a>Azure Active Directory 使用規定特性
Azure AD 使用規定提供一種簡單的方法，組織可用來將資訊呈現給終端使用者。 此呈現可確保使用者看到合法或合規性需求的相關免責聲明。 本文將說明如何開始使用 Azure AD 使用規定。

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="what-can-i-do-with-terms-of-use"></a>我可以用使用規定來做什麼？
Azure AD 使用規定可讓您執行下列作業：
- 要求員工或來賓在取得存取權前，先同意您的使用規定。
- 發佈適用於您組織中所有使用者的一般使用規定。
- 發佈以使用者屬性為基礎的特定使用規定 (例如， 醫生與護士或國內員工與國際員工，透過使用[動態群組](../users-groups-roles/groups-dynamic-membership.md)完成)。
- 發佈在存取高商業影響應用程式 (例如 Salesforce) 時的特定使用規定。
- 發佈不同語言的使用規定。
- 列出已同意或未同意使用規定的人員。
- 顯示使用規定活動的稽核記錄。

## <a name="prerequisites"></a>必要條件
若要使用及設定 Azure AD 使用規定，您必須有：

- Azure AD Premium P1、P2、EMS E3 或 EMS E5 訂用帳戶。
    - 如果您沒有其中任何一個訂用帳戶，您可以[取得 Azure AD Premium](../fundamentals/active-directory-get-started-premium.md) 或[啟用 Azure AD Premium 試用版](https://azure.microsoft.com/trial/get-started-active-directory/)。
- 下列其中一個管理帳戶，將用於您要設定的目錄：
    - 全域管理員
    - 安全性系統管理員
    - 條件式存取系統管理員

## <a name="terms-of-use-document"></a>使用規定文件

Azure AD 使用規定使用 PDF 格式來呈現內容。 此 PDF 檔案可以是任何內容 (例如現有的合約文件)，可讓您在使用者登入期間收集終端使用者合約。 PDF 中建議使用的字型大小是 24。

## <a name="add-terms-of-use"></a>新增使用規定
一旦完成了您的使用規定文件，請使用下列程序來新增它。

1. 以全域管理員、安全性系統管理員或條件式存取系統管理員的身分登入 Azure。

1. 瀏覽至 [https://aka.ms/catou](https://aka.ms/catou) 上的 [使用規定]。

    ![[使用規定] 刀鋒視窗](./media/active-directory-tou/tou-blade.png)

1. 按一下 [新增規定]。

    ![新增 TOU](./media/active-directory-tou/new-tou.png)

1. 輸入使用規定的**名稱**

2. 輸入**顯示名稱**。  使用者在登入時會看到此標頭。

3. **瀏覽**至您已完成的使用規定 PDF 並加以選取。

4. **選取**使用規定的語言。  語言選項可讓您上傳多個使用規定，各有不同的語言。  終端使用者會看到的使用規定版本將以其瀏覽器喜好設定為基礎。

5. 針對 [要求使用者展開使用規定]，選擇開啟或關閉。  如果此設定設為開啟，則使用者必須先閱讀使用規定，再接受這些規定。

6. 在 [條件式存取] 之下，您可以選取下拉式清單中的範本或自訂的條件式存取原則，**強制執行**上傳的使用規定。  自訂的條件式存取原則可讓細微使用條款下降到特定雲端應用程式或使用者群組。  如需詳細資訊，請參閱[設定條件式存取原則](../../cognitive-services/qnamaker/concepts/best-practices.md)。

    >[!IMPORTANT]
    >條件式存取原則控制項 (包括使用規定) 不支援服務帳戶的強制執行。  建議您排除條件式存取原則中的所有服務帳戶。

7. 按一下頁面底部的 [新增] 。

8. 如果您已選取自訂的條件式存取範本，則新的畫面會出現，可讓您自訂條件式存取原則。

    您現在應該會看到新的使用規定。

    ![新增 TOU](./media/active-directory-tou/create-tou.png)

## <a name="view-report-of-who-has-accepted-and-declined"></a>檢視已接受和已拒絕的人員報表
[使用規定] 刀鋒視窗會顯示已接受和已拒絕的使用者計數。 這些計數和接受/拒絕人員會加以儲存，直到使用規定的存留期結束。

1. 登入 Azure 並瀏覽至 [https://aka.ms/catou](https://aka.ms/catou) 上的 [使用規定]。

    ![稽核事件](./media/active-directory-tou/view-tou.png)

1. 按一下 [已接受] 或 [已拒絕] 下方的數字，檢視使用者目前的狀態。

    ![稽核事件](./media/active-directory-tou/accepted-tou.png)

## <a name="view-azure-ad-audit-logs"></a>檢視 Azure AD 稽核記錄
如果您想要檢視其他活動，Azure AD 使用規定包含稽核記錄。 每個使用者的同意都會觸發稽核記錄 (儲存 30 天) 中的事件。 您可以在入口網站中檢視這些記錄或將其下載為 .csv 檔案。

若要開始使用 Azure AD 稽核記錄，請使用下列程序：

1. 登入 Azure 並瀏覽至 [https://aka.ms/catou](https://aka.ms/catou) 上的 [使用規定]。

1. 按一下 [檢視稽核記錄]。

    ![稽核事件](./media/active-directory-tou/audit-tou.png)

1. 在 Azure AD 稽核記錄畫面上，您可以使用提供的下拉式清單，將特定的稽核記錄資訊設為目標來篩選資訊。

    ![稽核事件](./media/active-directory-tou/audit-logs-tou.png)

1. 您也可以按一下 [下載]，將資訊下載成 .csv 檔案，以在本機中使用。

## <a name="what-terms-of-use-looks-like-for-users"></a>使用者看到的使用規定
一旦建立並強制執行使用規定，範圍內的使用者將會在登入時看到下列畫面。

![稽核事件](./media/active-directory-tou/user-tou.png)

下列畫面顯示行動裝置上的使用規定。

![稽核事件](./media/active-directory-tou/mobile-tou.png)

使用者只需要接受使用規定一次，後續登入時不會再看到使用規定。

### <a name="how-users-can-review-their-terms-of-use"></a>使用者可檢閱使用規定的方式
使用者可以使用下列程序，檢閱並查看他們已接受的使用規定。

1. 登入 [https://myapps.microsoft.com](https://myapps.microsoft.com)。

1. 在右上角，按一下您的名稱並從下拉式功能表選取 [設定檔]。

    ![設定檔](./media/active-directory-tou/tou14.png)

1. 在您的設定檔頁面上，按一下 [檢閱使用規定]。

    ![稽核事件](./media/active-directory-tou/tou13a.png)

1. 您可以在此檢閱您已接受的使用規定。 

## <a name="edit-terms-of-use-details"></a>編輯使用規定詳細資料
您可以編輯使用規定的一些詳細資料，但無法修改現有的文件。 下列程序說明如何編輯詳細資料。

1. 登入 Azure 並瀏覽至 [https://aka.ms/catou](https://aka.ms/catou) 上的 [使用規定]。

1. 選取您要編輯的使用規定。

1. 按一下 [編輯使用規定]。

1. 在 [編輯使用規定] 窗格中，變更名稱、顯示名稱，或要求使用者展開值。

    ![新增 TOU](./media/active-directory-tou/edit-tou.png)

1. 按一下 [確定] 儲存變更。

    儲存您的變更後，使用者就必須接受新的規定。

## <a name="add-a-terms-of-use-language"></a>新增使用規定語言
下列程序說明如何新增使用規定語言。

1. 登入 Azure 並瀏覽至 [https://aka.ms/catou](https://aka.ms/catou) 上的 [使用規定]。

1. 選取您要編輯的使用規定。

1. 在 [詳細資料] 窗格中，按一下 [語言] 索引標籤。

    ![新增 TOU](./media/active-directory-tou/languages-tou.png)

1. 按一下 [新增語言]。

1. 在 [新增使用規定語言] 窗格中，上傳當地語系化的 PDF 並選取的語言。

    ![新增 TOU](./media/active-directory-tou/language-add-tou.png)

1. 按一下 [新增] 來新增語言。

## <a name="delete-terms-of-use"></a>刪除使用規定
您可以使用下列程序，刪除舊的使用規定。

1. 登入 Azure 並瀏覽至 [https://aka.ms/catou](https://aka.ms/catou) 上的 [使用規定]。

1. 選取您想要移除的使用規定。

1. 按一下 [刪除規定]。

1. 出現詢問您是否要繼續的訊息時，請按一下 [是]。

    ![新增 TOU](./media/active-directory-tou/delete-tou.png)

    您應該再也看不到您的使用規定。

## <a name="deleted-users-and-active-terms-of-use"></a>已刪除的使用者和有效使用規定
根據預設，已刪除的使用者會在 Azure AD 中以已刪除的狀態保留 30 天，在這段期間內，系統管理員可以視需要加以還原。  經過 30 天後，該使用者將永久刪除。  此外，全域管理員可以使用 Azure Active Directory 入口網站，在該時段結束之前明確地[永久刪除最近刪除的使用者](../fundamentals/active-directory-users-restore.md)。  使用者永久刪除後，關於該使用者的資料將會從作用中的使用規定中移除。  與已刪除的使用者有關的稽核資訊仍會保留在稽核記錄中。

## <a name="policy-changes"></a>原則變更
條件式存取原則會立即生效。 當此情況發生時，系統管理員便會開始看到「悲傷雲端」或「Azure AD 權杖問題」。 系統管理員必須登出並再次登入，才能符合新的原則。

>[!IMPORTANT]
> 如果情況如下，範圍中的使用者必須登出並登入，才能符合新的原則：
> - 對使用規定啟用條件式存取原則
> - 或建立第二個使用條款

## <a name="frequently-asked-questions"></a>常見問題集

**問：如何查看使用者何時或是否已接受使用規定？**</br>
答：在 [使用規定] 刀鋒視窗上，按一下 [接受] 底下的數字。 您也可以檢視或搜尋 Azure AD 稽核記錄中的接受活動。 如需詳細資訊，請參閱[檢視已接受和已拒絕的人員報表](#view-who-has-accepted-and-declined)和[檢視 Azure AD 稽核記錄](#view-azure-ad-audit-logs)。
 
**問：資訊會儲存多久時間？**</br>
答：使用規定報表中的使用者計數和已接受/已拒絕的人員都會儲存到使用規定的存留期結束。 Azure AD 稽核記錄會儲存 30 天。

**問：為什麼我在使用規定報表與 Azure AD 稽核記錄中看到同意數目不同？**</br>
答：使用規定報表會儲存到該使用規定的存留期結束，而 Azure AD 稽核記錄會儲存 30 天。 此外，使用規定報表只會顯示使用者目前的同意狀態。 例如，如果使用者先拒絕，然後接受，則使用規定報表將只會顯示該使用者的接受。 如果您需要查看歷程記錄，您可以使用 Azure AD 稽核記錄。

**問：如果我編輯使用規定的詳細資料，使用者是否需要重新接受？**</br>
答：是，如果系統管理員編輯使用規定的詳細資料，使用者就需要重新接受新的規定。

**問：是否可以更新現有的使用規定文件？**</br>
答︰您目前無法更新現有的使用規定文件。 若要變更使用規定文件，您必須建立新的使用規定執行個體。

**問：如果超連結位於使用規定 PDF 文件中，終端使用者將能點選這些超連結嗎？**</br>
答：此 PDF 預設會以 JPEG 形式呈現，因此無法點選超連結。 使用者可以選取**檢視時發生問題嗎？請按一下這裡**的選項，這樣就會以原生方式呈現支援超連結的 PDF。

**問：使用規定是否支援多種語言？**</br>
答： 會。 目前有 108 種語言可供系統管理員為單一使用規定設定。

**問：何時觸發使用規定？**</br>
答：登入體驗期間即會觸發使用規定。

**問：哪些應用程式也可以將使用規定設為目標？**</br>
答：您可以使用新式驗證，在企業應用程式上建立條件式存取原則。  如需詳細資訊，請參閱[企業應用程式](./../manage-apps/view-applications-portal.md)。

**問：是否可以將多個使用規定新增至指定的使用者或應用程式？**</br>
答：是，方法為建立多個條件式存取原則，將群組或應用程式設為目標。 如果使用者落在多個使用規定的範圍內，他們可以逐一同意各項使用規定。
 
**問：如果使用者拒絕使用規定，會發生什麼事？**</br>
答：使用者會遭封鎖而無法存取應用程式。 使用者必須再次登入並同意規定，才能取得存取權。
 
**問：是否可以取消接受先前接受的使用規定？**</br>
答：您可以[檢閱先前接受的使用規定](#how-users-can-review-their-terms-of-use)，但目前沒有辦法取消接受。

## <a name="next-steps"></a>後續步驟

- [Azure Active Directory 中條件式存取的最佳做法](../../active-directory/conditional-access/best-practices.md)
