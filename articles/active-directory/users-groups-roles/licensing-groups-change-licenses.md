---
title: 如何使用 Azure Active Directory 中的群組型授權在產品授權之間安全地移轉使用者 | Microsoft Docs
description: 說明使用群組型授權在不同產品授權 (例如 Office 365 企業版 E1 和 E3) 之間移轉使用者時的建議程序
services: active-directory
keywords: Azure AD 授權
documentationcenter: ''
author: piotrci
manager: mtillman
editor: ''
ms.assetid: ''
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/07/2018
ms.author: piotrci
ms.openlocfilehash: bed720cf47355887f3bf981f281d82738039270f
ms.sourcegitcommit: ab3b2482704758ed13cccafcf24345e833ceaff3
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/06/2018
ms.locfileid: "37866571"
---
# <a name="how-to-safely-migrate-users-between-product-licenses-by-using-group-based-licensing"></a>如何使用群組型授權安全地在產品授權之間移轉使用者

本文說明使用群組型授權時，在產品授權之間移動使用者的建議方法。 此方法的目標是要確保在移轉期間不會遺失任何服務或資料：使用者應該在產品之間順暢切換。 其中涵蓋兩種移轉程序變化形式：

-   未含有衝突服務方案之產品授權間的簡單移轉，例如在「Office 365 企業版 E3」與「Office 365 企業版 E5」之間移轉。

-   含有部分衝突服務方案之產品間的較複雜移轉，例如在「Office 365 企業版 E1」與「Office 365 企業版 E3」之間移轉。 如需有關衝突的詳細資訊，請參閱[衝突的服務方案](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-group-problem-resolution-azure-portal#conflicting-service-plans)和[無法同時指派的服務方案](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-product-and-service-plan-reference#service-plans-that-cannot-be-assigned-at-the-same-time)。

本文包括範例 PowerShell 程式碼，可用來執行移轉和驗證步驟。 此程式碼對於無法手動執行步驟的大規模作業特別有用。

## <a name="before-you-begin"></a>開始之前
開始進行移轉之前，請務必確認針對所有要移轉之使用者的假設都成立。 如果這些假設不是對所有使用者都成立，部分使用者的移轉可能會失敗。 因此，部分使用者可能就會無法存取服務或資料。 以下是應該確認的假設：

-   使用者擁有以群組型授權指派的「來源授權」。 要作為移轉來源的產品授權是繼承自單一來源群組，而非以直接方式指派。

    >[!NOTE]
    >如果授權也是直接指派的，它們可能會導致無法套用「目標授權」。 深入了解[直接和群組授權指派](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-group-advanced#direct-licenses-coexist-with-group-licenses)。 您可以使用 [PowerShell 指令碼](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-ps-examples#check-if-user-license-is-assigned-directly-or-inherited-from-a-group)來檢查使用者是否擁有直接授權。

-   您擁有目標產品的足夠可用授權。 如果您沒有足夠的授權，部分使用者可能會無法取得「目標授權」。 您可以[查看可用授權的數目](https://portal.azure.com/#blade/Microsoft_AAD_IAM/LicensesMenuBlade/Products)。

-   使用者沒有其他會與「目標授權」衝突或防止移除「來源授權」的已指派產品授權。 例如，與其他產品相依的附加元件產品 (例如「工作場所分析」或 Project Online) 授權。

-   了解您環境中管理群組的方式。 例如，如果您在內部部署環境中管理群組，然後透過 Azure AD Connect 將它們同步至 Azure Active Directory (Azure AD)，則您會使用內部部署系統來新增/移除使用者。 將變更同步至 Azure AD 並被群組型授權套用需要一些時間。 如果您使用 Azure AD 動態群組成員資格，則會改為透過修改使用者的屬性來新增/移除使用者。 不過，整體移轉程序保持不變。 唯一的差異是您為群組成員資格新增/移除使用者的方式。

## <a name="migrate-users-between-products-that-dont-have-conflicting-service-plans"></a>在未含有衝突服務方案的產品之間移轉使用者
移轉目標是要使用群組型授權，將使用者授權從「來源授權」(在此範例中為「Office 365 企業版 E3」) 變更為「目標授權」(在此範例中為「Office 365 企業版 E5」)。 此案例中的這兩個產品並未含有衝突的服務方案，因此可以同時完全指派兩者，而不會發生衝突。 在移轉期間中的任何時間點，使用者都不會無法存取服務或資料。 移轉會以小「批次」的方式執行。 您可以驗證每個批次的結果，並將此程序中可能發生之任何問題的範圍縮減到最小。 整體而言，此程序如下：

1.  使用者是來源群組的成員，並從該群組繼承「來源授權」。

2.  建立具有「目標授權」但沒有任何成員的目標群組。

3.  將一批使用者新增至目標群組。 群組型授權會套用變更並指派「目標授權」。 視批次大小及租用戶中的其他活動而定，此程序可能需要相當長的時間。

4.  確認群組型授權已完全處理該批使用者。 請確認每個使用者都已獲指派「目標授權」。 確認使用者最後並不是處於錯誤狀態，例如與其他產品發生衝突或缺乏足夠的授權。 如需與錯誤相關的詳細資訊，請參閱 [Active Directory 授權群組問題解決方式](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-group-problem-resolution-azure-portal)。

5.  此時，使用者會同時已獲指派「來源授權」和「目標授權」。

6.  從來源群組中移除同一個使用者批次。 群組型授權會回應此變更，而從使用者移除「來源授權」。

7.  針對後續的使用者批次重複此程序。

### <a name="migrate-a-single-user-by-using-the-azure-portal"></a>使用 Azure 入口網站來移轉單一使用者
這是一個說明如何移轉單一使用者的簡單逐步解說。

**步驟 1**：使用者擁有繼承自群組的「來源授權」。 該授權沒有任何直接的指派：

![來源授權繼承自群組的使用者](./media/licensing-groups-change-licenses/UserWithSourceLicenseInherited.png)

**步驟 2**：將使用者新增至目標群組，而由群組型授權處理此變更。 使用者現在同時擁有繼承自群組的「來源授權」和「目標授權」：

![來源授權和目標授權都繼承自群組的使用者](./media/licensing-groups-change-licenses/UserWithBothSourceAndTargetLicense.png)

**步驟 3**：從來源群組中移除使用者，而由群組型授權處理此變更。 使用者現在只有「目標授權」：

![目標授權繼承自群組的使用者](./media/licensing-groups-change-licenses/UserWithTargetLicenseAssigned.png)

### <a name="automate-migration-by-using-azure-powershell"></a>使用 Azure PowerShell 來自動執行移轉
以下程式碼片段說明如何自動執行大規模作業的移轉程序。

> [!NOTE]
> 這個簡單程式碼使用本文[最後一節](#powershell-automation-of-migration-and-verification-steps)中所包含的 PowerShell 函式。

```
# A batch of users that we want to migrate in this iteration.
# The batch can be specified as an array of User Principal Names (string) or ObjectIds (Guid).
# Note: The batch can be loaded from a text file that represents a larger batch of users that we want to migrate.
[string[]]$usersToMigrate = 'MigrationUser@tailspinonline.com','MigrationUser2@tailspinonline.com'

############### NON-CONFLICTING LICENSES SCENARIO ################

# The source group and source license to remove the user from.
$sourceGroupId = [Guid]'b82c04f0-ce30-4ff1-bac7-735d92d83036'
$sourceSkuId = 'TailspinOnline:ENTERPRISEPACK'      # <-- This is the Office 365 Enterprise E3 product.

# The target group and target license to assign to the user.
$targetGroupId = [Guid]'bcf279d1-5ad5-46a5-b469-4b8a552aa2fe'
$targetSkuId = 'TailspinOnline:ENTERPRISEPREMIUM'   # <-- This is the Office 365 Enterprise E5 product.

if(-Not (VerifyAssumptions $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId))
{
    throw "Some users did not pass validation checks. See the output for details. Aborting migration process."
}

Write-Host "STEP 1: Adding users to the target group $targetGroupId. This will assign the target license $targetSkuId to all users"
AddUsersToGroup $usersToMigrate $targetGroupId

# Verify that the target license shows up in the conflict state for each user on the list.
# This step runs in a loop, forever, until all of the users are in the expected state.
# Group-based licensing (GBL) can take some time to reflect the changes on users.
# As a result, the loop should terminate after a period of time that's dependent on the size of the user collection.
# Note: If the loop hasn't terminated after a long period of time, stop the script.
#       Inspect the users that are reported as not yet in the expected state.
#       Verify that the users are not blocked for some other reason.
ExecuteVerificationLoop ${function:VerifySourceandTargetLicensePresent} 'STEP 2: Checking if all users still have the source license and now also have the target license from target group' $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId

# Now it's safe to remove the users from the source group.
Write-Host "STEP 3: Removing users from the source group $sourceGroupId. This will remove the source license $sourceSkuId."
RemoveUsersFromGroup $usersToMigrate $sourceGroupId

# Verify that the target license is now active on each user and the source license is removed.
ExecuteVerificationLoop ${function:VerifySourceLicenseRemovedAndTargetLicenseAssignedFromGroup} 'STEP 4: Checking if all users have source license removed and target license assigned from target group' $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId
```

**範例輸出 (移轉 2 個使用者)**

```
Verifying initial assumptions:
Enough TailspinOnline:ENTERPRISEPREMIUM licenses available (13) for users: 2.
migrationuser@tailspinonline.com                OK
migrationuser2@tailspinonline.com               OK
Checks passed for all 2 users.

STEP 1: Adding users to the target group bcf279d1-5ad5-46a5-b469-4b8a552aa2fe. This will assign the target license TailspinOnline:ENTERPRISEPREMIUM to all users
Adding user MigrationUser@tailspinonline.com to group bcf279d1-5ad5-46a5-b469-4b8a552aa2fe
Adding user MigrationUser2@tailspinonline.com to group bcf279d1-5ad5-46a5-b469-4b8a552aa2fe

STEP 2: Checking if all users still have the source license and now also have the target license from target group. Check iteration: 1

MigrationUser@tailspinonline.com                Expected state: NO
MigrationUser2@tailspinonline.com               Expected state: NO

Total users checked: 2. In expected state: 0. Not yet: 2
Not all users are in expected state. Waiting 60 seconds to try again.

STEP 2: Checking if all users still have the source license and now also have the target license from target group. Check iteration: 2

MigrationUser@tailspinonline.com                Expected state: YES
MigrationUser2@tailspinonline.com               Expected state: YES

Total users checked: 2. In expected state: 2. Not yet: 0
Check passed for all users. Exiting check loop.

STEP 3: Removing users from the source group b82c04f0-ce30-4ff1-bac7-735d92d83036. This will remove the source license TailspinOnline:ENTERPRISEPACK.
Removing user MigrationUser@tailspinonline.com from group b82c04f0-ce30-4ff1-bac7-735d92d83036
Removing user MigrationUser2@tailspinonline.com from group b82c04f0-ce30-4ff1-bac7-735d92d83036

STEP 4: Checking if all users have source license removed and target license assigned from target group. Check iteration: 1

MigrationUser@tailspinonline.com                Expected state: NO
MigrationUser2@tailspinonline.com               Expected state: NO

Total users checked: 2. In expected state: 0. Not yet: 2
Not all users are in expected state. Waiting 60 seconds to try again.

STEP 4: Checking if all users have source license removed and target license assigned from target group. Check iteration: 2

MigrationUser@tailspinonline.com                Expected state: YES
MigrationUser2@tailspinonline.com               Expected state: YES

Total users checked: 2. In expected state: 2. Not yet: 0
Check passed for all users. Exiting check loop.
```

## <a name="migrate-users-between-products-that-have-conflicting-service-plans"></a>在含有衝突服務方案的產品之間移轉使用者
移轉目標是要使用群組型授權，將使用者授權從「來源授權」(在此範例中為「Office 365 企業版 E1」) 變更為「目標授權」(在此範例中為「Office 365 企業版 E3」)。 此案例中的兩個產品含有衝突的服務方案，因此您必須解決衝突，才能順暢地移轉使用者。 如需有關這些衝突的詳細資訊，請參閱 [Active Directory 授權群組問題解決方式：衝突的服務方案](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-group-problem-resolution-azure-portal#conflicting-service-plans)。 在移轉期間中的任何時間點，使用者都不會無法存取服務或資料。 移轉會以小「批次」的方式執行。 您可以驗證每個批次的結果，並將此程序中可能發生之任何問題的範圍縮減到最小。 整體而言，此程序如下：

1.  使用者是來源群組的成員，並從該群組繼承「來源授權」。

2.  建立具有「目標授權」但沒有任何成員的目標群組。

3.  將一批使用者新增至目標群組。 群組型授權會套用變更並嘗試指派「目標授權」。 由於兩個產品中的服務之間有衝突，因此指派會失敗。 群組型授權會在每個使用者上將失敗記錄成錯誤。 視批次大小及租用戶中的其他活動而定，此程序可能需要相當長的時間。

4.  確認群組型授權已完全處理該批使用者。 請確認已為每個使用者記錄衝突錯誤。 確認某些使用者最後並未處於非預期的錯誤狀態。 如需與錯誤相關的詳細資訊，請參閱 [Active Directory 授權群組問題解決方式](https://docs.microsoft.com/azure/active-directory/active-directory-licensing-group-problem-resolution-azure-portal)。

5.  此時，使用者仍擁有「來源授權」，而「目標授權」則有衝突錯誤。 使用者尚未獲指派「目標授權」。

6.  從來源群組中移除同一個使用者批次。 群組型授權會回應此變更，而從每個使用者移除「來源授權」。 同時也會移除衝突錯誤 (當沒有任何其他產品授權造成錯誤時)，然後指派「目標授權」。 此程序可確保在轉換期間不會遺失任何服務或資料。

7.  針對後續的使用者批次重複此程序。

### <a name="migrate-a-single-user-by-using-the-azure-portal"></a>使用 Azure 入口網站來移轉單一使用者
這是一個說明如何移轉單一使用者的簡單逐步解說。

**步驟 1**：使用者擁有繼承自群組的「來源授權」。 該授權沒有任何直接的指派：

![來源授權繼承自群組的使用者](./media/licensing-groups-change-licenses/UserWithSourceLicenseInheritedConflictScenario.png)

**步驟 2**：將使用者新增至目標群組，而由群組型授權處理此變更。 由於使用者仍然擁有「來源授權」，因此「目標授權」會因為衝突而處於錯誤狀態：

![來源授權繼承自群組且目標授權處於錯誤狀態的使用者](./media/licensing-groups-change-licenses/UserWithSourceLicenseAndTargetLicenseInConflict.png)

**步驟 3**：從來源群組中移除使用者，而由群組型授權處理此變更。 「目標授權」會套用至使用者：

![目標授權繼承自群組的使用者](./media/licensing-groups-change-licenses/UserWithTargetLicenseAssignedConflictScenario.png)


### <a name="automate-migration-by-using-azure-powershell"></a>使用 Azure PowerShell 來自動執行移轉
以下程式碼片段說明如何自動執行大規模作業的移轉程序。

> [!NOTE]
> 這個簡單程式碼使用本文[最後一節](#powershell-automation-of-migration-and-verification-steps)中所包含的 PowerShell 函式。

```
# A batch of users that we want to migrate in this iteration.
# The batch can be specified as an array of User Principal Names (string) or ObjectIds (Guid).
# Note: The batch can be loaded from a text file that represents a larger batch of users that we want to migrate.
[string[]]$usersToMigrate = 'MigrationUser@tailspinonline.com', 'MigrationUser2@tailspinonline.com'

############### CONFLICTING LICENSES SCENARIO ################

# The source group and source license to remove the user from.
$sourceGroupId = [Guid]'b82c04f0-ce30-4ff1-bac7-735d92d83036'
$sourceSkuId = 'TailspinOnline:STANDARDPACK'             # <-- This is the Office 365 Enterprise E1 product.

# The target group and target license to assign to the user.
$targetGroupId = [Guid]'bcf279d1-5ad5-46a5-b469-4b8a552aa2fe'
$targetSkuId = 'TailspinOnline:ENTERPRISEPACK'           # <-- This is the Office 365 Enterprise E3 product.

# Assumptions before migration:
# 1. Users are already in the source group and they have the source license assigned from that group.
# 2. Users don't have the same source license assigned from another group at the same time,
#    and they don't have the source license assigned directly.
#    IMPORTANT: If Assumption 2 isn't true, removing users from the source group in STEP 3
#               won't result in the target license being correctly applied.
# 3. There are enough available licenses to assign a target license to all of the users that are being migrated.
if(-Not (VerifyAssumptions $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId))
{
    throw "Some users did not pass validation checks. See the output for details. Aborting migration process."
}

Write-Host "STEP 1: Adding users to the target group $targetGroupId. This will put target license $targetSkuId in conflict state with the source license $sourceSkuId"
AddUsersToGroup $usersToMigrate $targetGroupId

# Verify that the target license shows up in the conflict state for each user on the list.
# This step runs in a loop, forever, until all of the users are in the expected state.
# Group-based licensing (GBL) can take some time to reflect the changes on users.
# As a result, the loop should terminate after a period of time that's dependent on the size of the user collection.
# Note: If the loop hasn't terminated after a long period of time, stop the script.
#       Inspect the users that are reported as not yet in the expected state.
#       Verify that the users are not blocked for some other reason.
ExecuteVerificationLoop ${function:VerifySourceLicensePresentAndTargetLicenseInConflictState} 'STEP 2: Checking if all users still have the source license and are in conflict state for license from target group' $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId

# Now it's safe to remove the users from the source group.
Write-Host "STEP 3: Removing users from the source group $sourceGroupId. This will remove the source license $sourceSkuId and remove the conflict on target license $targetSkuId which will become assigned."
RemoveUsersFromGroup $usersToMigrate $sourceGroupId

# Verify that the target license is now active on each user and the source license is removed.
ExecuteVerificationLoop ${function:VerifySourceLicenseRemovedAndTargetLicenseAssignedFromGroup} 'STEP 4: Checking if all users have source license removed and target license assigned from target group' $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId
```

**範例輸出 (移轉 2 個使用者)**

```
Verifying initial assumptions:
Enough TailspinOnline:ENTERPRISEPACK licenses available (61) for users: 2.
migrationuser@tailspinonline.com                OK
migrationuser2@tailspinonline.com               OK
Checks passed for all 2 users.
STEP 1: Adding users to the target group bcf279d1-5ad5-46a5-b469-4b8a552aa2fe. This will put target license TailspinOnline:ENTERPRISEPACK in conflict state with the source license TailspinOnline:STANDARDPACK
Adding user MigrationUser@tailspinonline.com to group bcf279d1-5ad5-46a5-b469-4b8a552aa2fe
Adding user MigrationUser2@tailspinonline.com to group bcf279d1-5ad5-46a5-b469-4b8a552aa2fe

STEP 2: Checking if all users still have the source license and are in conflict state for license from target group. Check iteration: 1

MigrationUser@tailspinonline.com                Expected state: NO
MigrationUser2@tailspinonline.com               Expected state: NO

Total users checked: 2. In expected state: 0. Not yet: 2
Not all users are in expected state. Waiting 60 seconds to try again.

STEP 2: Checking if all users still have the source license and are in conflict state for license from target group. Check iteration: 3

MigrationUser@tailspinonline.com                Expected state: YES
MigrationUser2@tailspinonline.com               Expected state: YES

Total users checked: 2. In expected state: 2. Not yet: 0
Check passed for all users. Exiting check loop.

STEP 3: Removing users from the source group b82c04f0-ce30-4ff1-bac7-735d92d83036. This will remove the source license TailspinOnline:STANDARDPACK and remove the conflict on target license TailspinOnline:ENTERPRISEPACK which will become assigned.
Removing user MigrationUser@tailspinonline.com from group b82c04f0-ce30-4ff1-bac7-735d92d83036
Removing user MigrationUser2@tailspinonline.com from group b82c04f0-ce30-4ff1-bac7-735d92d83036

STEP 4: Checking if all users have source license removed and target license assigned from target group. Check iteration: 1

MigrationUser@tailspinonline.com                Expected state: NO
MigrationUser2@tailspinonline.com               Expected state: NO

Total users checked: 2. In expected state: 0. Not yet: 2
Not all users are in expected state. Waiting 60 seconds to try again.

STEP 4: Checking if all users have source license removed and target license assigned from target group. Check iteration: 3

MigrationUser@tailspinonline.com                Expected state: YES
MigrationUser2@tailspinonline.com               Expected state: YES

Total users checked: 2. In expected state: 2. Not yet: 0
Check passed for all users. Exiting check loop.
```

<h2 id="powershell-automation-of-migration-and-verification-steps">執行 PowerShell 程式碼以自動執行及確認移轉</h2>

本節包含執行本文所述之指令碼所需的 PowerShell 程式碼。

>[!WARNING]
>此程式碼是基於示範目的而提供的範例。 如果您想要在您的環境中使用它，請考慮先對程式碼進行小規模測試，或在個別的測試租用戶中進行測試。 您可能需要調整程式碼以符合您環境的特定需求。

若要執行此程式碼，請使用 [Azure AD PowerShell v1.0 程式庫](https://docs.microsoft.com/powershell/azure/active-directory/install-msonlinev1?view=azureadps-1.0) \(英文\)。 執行指令碼之前，請先執行 `connect-msolservice` Cmdlet 來登入租用戶。

```
# BEGIN: Helper functions that are used in the scripts.

# GetUserObject function
# Retrieve a user object based on the ObjectId or UserPrincipalName.
function GetUserObject
{
    [OutputType([Microsoft.Online.Administration.User])]
    Param([object]$userId)

    $userIdType = $userId.GetType()

    if($userIdType -eq [Guid])
    {
        return Get-MsolUser -ObjectId $userId
    }
    elseif($userIdType -eq [string])
    {
        return Get-MsolUser -UserPrincipalName $userId
    }
    else
    {
        throw "User Id type must be a Guid or a string (UserPrincipalName). The user id type was: $($userIdType.Name)"
    }
}

# GetGuidUserId function
# Get a Guid objectId for a user, even when a UserPrincipal string is passed in.
# Guid ids are needed for group membership manipulation, where UPNs cannot be used.
function GetGuidUserId
{
    [OutputType([Guid])]
    Param([object]$userId)

    [Guid]$guidId = [Guid]::Empty
    if($userId.GetType() -eq [String])
    {
        $user = GetUserObject $userId
        return $user.ObjectId
    }
    elseif($userId.GetType() -eq [Guid])
    {
        return $userId
    }
    else
    {
        throw "UserId type must be a Guid or a string (UserPrincipalName). The user id type was: $($userId.GetType().Name)"
    }
}

# AddUsersToGroup function
# Add a collection of users to a group.
# Note: This function fails if a user is already a member of the specified group.
function AddUsersToGroup
{
    Param([object[]]$userIds, [Guid]$groupId)

    foreach($userId in $userIds)
    {
        $fetchId = GetGuidUserId $userId
        Write-Host "Adding user $userId to group $groupId"
        Add-MsolGroupMember -GroupObjectId $groupId -GroupMemberType User -GroupMemberObjectId $fetchId
    }
}

# RemoveUsersFromGroup function
# Remove a collection of users from a group.
# Note: This function fails if a user is not a member of the specified group.
function RemoveUsersFromGroup
{
    Param([object[]]$userIds, [Guid]$groupId)

    foreach($userId in $userIds)
    {
        $fetchId = GetGuidUserId $userId
        Write-Host "Removing user $userId from group $groupId"
        Remove-MsolGroupMember -GroupObjectId $groupId -GroupMemberType User -GroupMemberObjectId $fetchId
    }
}

# GetUserLicense function
# Return the license object that corresponds to the skuId.
# Return NULL if no object is found.
function GetUserLicense
{
    [OutputType([Microsoft.Online.Administration.UserLicense])]
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId)

    # Look for the specific license SKU in all of the licenses that are assigned to the user.
    foreach($license in $user.Licenses)
    {
        if ($license.AccountSkuId -ieq $skuId)
        {
            return $license
        }
    }
    return $null
}

# IsLicenseAssignedToUser function
# Check if the specific SKU license is assigned to the user,
#    regardless of how the license is assigned (directly or via GBL).
function IsLicenseAssignedToUser
{
    [OutputType([bool])]
    Param([Microsoft.Online.Administration.User]$user,  [string]$skuId)

    $license = GetUserLicense $user $skuId

    return ($license -ne $null)
}

# GetObjectIdsAssigningLicense function
function GetObjectIdsAssigningLicense
{
    [OutputType([Guid[]])]
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId)

    $license = GetUserLicense $user $skuId

    if ($license -ne $null)
    {
        return [Guid[]]$license.GroupsAssigningLicense
    }
    return [Array]::CreateInstance([Guid],0)
}

# UserHasLicenseAssignedFromThisGroup function
# Return TRUE if the user inherits the license from the specific group.
# Note: This function returns true only if the license is successfully assigned from the group.
#       If the license is in an error state, the function return false.
function UserHasLicenseAssignedFromThisGroup
{
    [OutputType([bool])]
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId, [Guid]$groupId)

    [Guid[]]$objectsAssigningLicense = GetObjectIdsAssigningLicense $user $skuId

    # GroupsAssigningLicense contains a collection of object IDs for assigning the license.
    # This could be a group object or a user object (contrary to what the name suggests).
    foreach ($assignmentSource in $objectsAssigningLicense)
    {
        # If the collection contains at least one ID that doesn't match the user ID, the license is inherited from a group.
        # Note: The license might also be assigned directly, in addition to being inherited.
        if ($assignmentSource -ieq $groupId)
        {
            return $true
        }
    }
    return $false
}

# GetErrorsForLicense function
# Return error objects for a specific license.
function GetErrorsForLicense
{
    [OutputType([Microsoft.Online.Administration.IndirectLicenseError[]])]
    Param([Microsoft.Online.Administration.User]$user, [string]$skuId)

    [Microsoft.Online.Administration.IndirectLicenseError[]] $errorObjects = $user.IndirectLicenseErrors | Where {"$($_.AccountSku.AccountName):$($_.AccountSku.SkuPartNumber)" -ieq $skuId}

    if($errorObjects -eq $null)
    {
        #there are no errors at all
        return [Array]::CreateInstance([Microsoft.Online.Administration.IndirectLicenseError],0)
    }

    return $errorObjects
}

# GetErrorForLicenseFromGroup function
# Return an error label that's associated with a specific license that's inherited from a specific group.
# Return $null if there's no error.
function GetErrorForLicenseFromGroup
{
    [OutputType([string])]
    Param([Microsoft.Online.Administration.User]$user,  [Guid]$groupId, [string]$skuId)


    # There are some errors. Check if any of the errors are associated with the group.
    foreach($licenseError in GetErrorsForLicense $user $skuId)
    {
        if($licenseError.ReferencedObjectId -eq $groupId)
        {
            return $licenseError.Error
        }
    }
    return $null
}

# IsExpectedLicenseStateForGroup function
# Check if the license is in an expected state for a given group.
# If expectedError is set to a value, the function checks if the license is in the specific error state for the group.
# If expectedError is NULL, the function checks if the license is successfully assigned from the group.
function IsExpectedLicenseStateForGroup
{
    [OutputType([bool])]
    Param([Microsoft.Online.Administration.User]$user,  [Guid]$groupId, [string]$skuId, [string]$expectedError)

    # The license is expected to be fully assigned from the group and not in an error state.
    if([string]::IsNullOrEmpty($expectedError))
    {
        # Check if the assigned license is inherted from the expected group and without an error on it.
        return (UserHasLicenseAssignedFromThisGroup $user $skuId $groupId)
    }
    # The license is expected to be in the specific error state on the specific group.
    else
    {
        $error = GetErrorForLicenseFromGroup $user $groupId $skuId
        return ($error -ieq $expectedError)
    }
}

# VerifySourceLicensePresentAndTargetLicenseInConflictState function
# Detect if the licenses are in a specific state where the source license is assigned, but the target license is in a conflict state.
# Note: If the source license is gone, the function throws an exception to abort the script.
#       The conflict state can signify that something went wrong with the migration steps and the user lost access to services.
function VerifySourceLicensePresentAndTargetLicenseInConflictState
{
    [OutputType([bool])]
    Param([Microsoft.Online.Administration.User]$user,  [Guid]$sourceGroupId, [string]$sourceSkuId, [Guid]$targetGroupId, [string]$targetSkuId)

    # Check if the user still has the source license. If not, abort the script because something is seriously wrong.
    if(-Not (UserHasLicenseAssignedFromThisGroup $user $sourceSkuId $sourceGroupId))
    {
        throw "User $($user.UserPrincipalName) ($($user.ObjectId)) does not have the expected license $sourceSkuId from source group $sourceGroupId, which may result in loss of access and data. This is unexpected and should be investigated. Aborting execution."
    }
    # Check if the target license is in conflict, as expected.
    $conflictError = 'MutuallyExclusiveViolation'
    return (IsExpectedLicenseStateForGroup $user $targetGroupId $targetSkuId $conflictError)
}

# VerifySourceLicenseRemovedAndTargetLicenseAssignedFromGroup function
# Detect if the licenses are in a specific state where the source license isn't present,
#    but the target license is correctly assigned.
# Note: If the source license is gone and the target license isn't present,
#       the function throws an exception to abort the script.
#       Something went wrong and the user may have lost access to services.
function VerifySourceLicenseRemovedAndTargetLicenseAssignedFromGroup
{
    [OutputType([bool])]
    Param([Microsoft.Online.Administration.User]$user,  [Guid]$sourceGroupId, [string]$sourceSkuId, [Guid]$targetGroupId, [string]$targetSkuId)

    # Check if user has the source license completely removed, which is a prerequisite to the target license eventually kicking in.
    if(IsLicenseAssignedToUser $user $sourceSkuId)
    {
        return $false
    }

    # Check if the user has the target license at all. If not, abort the script because something is seriously wrong.
    if(-Not (IsLicenseAssignedToUser $user $targetSkuId))
    {
        throw "User $($user.UserPrincipalName) ($($user.ObjectId)) does not have the expected license $targetSkuId assigned, which may result in loss of access and data. This is unexpected and should be investigated. Aborting execution."
    }
    # Check if the target license is assigned from the expected target group, and no longer in an error state.
    return (IsExpectedLicenseStateForGroup $user $targetGroupId $targetSkuId $null)
}

# VerifySourceandTargetLicensePresent function
# Detect if the licenses are in the specific state where the source license is assigned and the target license is also assigned.
# Note: If the source license is gone, the function throws an exception to abort the script.
#       This state can signify that something went wrong with the migration steps and the user lost access to services.
function VerifySourceandTargetLicensePresent
{
    [OutputType([bool])]
    Param([Microsoft.Online.Administration.User]$user,  [Guid]$sourceGroupId, [string]$sourceSkuId, [Guid]$targetGroupId, [string]$targetSkuId)

    #check user still has source license - if not, abort because something is seriously wrong
    if(-Not (UserHasLicenseAssignedFromThisGroup $user $sourceSkuId $sourceGroupId))
    {
        throw "User $($user.UserPrincipalName) ($($user.ObjectId)) does not have the expected license $sourceSkuId from source group $sourceGroupId, which may result in loss of access and data. This is unexpected and should be investigated. Aborting execution."
    }
    #check if the target license is also assigned from the target group
    return (UserHasLicenseAssignedFromThisGroup $user $targetSkuId $targetGroupId)
}

# VerifyAssumptionsForUser function
# Verify basic assumptions that should be true for a user before we execute the migration process.
# The function prints details about the verification steps.
# Return TRUE if all of the assumptions are true.
function VerifyAssumptionsForUser
{
    [OutputType([bool])]
    Param([Microsoft.Online.Administration.User]$user,  [Guid]$sourceGroupId, [string]$sourceSkuId, [Guid]$targetGroupId, [string]$targetSkuId)

    $userName = $user.UserPrincipalName
    # 1. The user has the source license assigned from the source group.
    if(-Not (UserHasLicenseAssignedFromThisGroup $user $sourceSkuId $sourceGroupId))
    {
        Write-Host "$userName does not have source license $sourceSkuId assigned from source group $sourceGroupId."
        return $false
    }

    # 2. The user does't have the same source license assigned from another group at the same time,
    #    and the user doesn't have the source license assigned directly.
    [Guid[]]$otherObjectsAssigningLicense = GetObjectIdsAssigningLicense $user $sourceSkuId | Where {$_ -ne $sourceGroupId}
    foreach($otherObject in $otherObjectsAssigningLicense)
    {
        if($otherObject -eq $user.ObjectId)
        {
            Write-Host "$userName has source license assigned directly to the user."
        }
        else
        {
            Write-Host "$username has source license assigned from an additional unexpected group $otherObject."
        }
    }
    if($otherObjectsAssigningLicense -and $otherObjectsAssigningLicense.Count -gt 0)
    {
        return $false
    }

    # 3. The user doesn't have the target license assigned.
    if(IsLicenseAssignedToUser $user $targetSkuId)
    {
        Write-Host "$userName already has target license assigned."
        return $false
    }

    # 4. The user doesn't have the target license in an error state from some groups.
    [Microsoft.Online.Administration.IndirectLicenseError[]]$licenseErrors = GetErrorsForLicense $user $targetSkuId
    foreach($licenseError in $licenseErrors)
    {
        Write-Host "$userName has target license in error state $($licenseError.Error) from unexpected group $($licenseError.ReferencedObjectId)."
    }
    if($licenseErrors -and $licenseErrors.Count -gt 0)
    {
        return $false
    }

    Write-Host "$userName`t`tOK"
    return $true
}

# VerifyAssumptions function
# Check if all of the users to be migrated are in a correct state.
function VerifyAssumptions
{
    [OutputType([bool])]
    Param([object[]]$userIds, [Guid]$sourceGroupId, [string]$sourceSkuId, [Guid]$targetGroupId, [string]$targetSkuId)

    Write-Host "Verifying initial assumptions:"

    # Check if there are enough target licenses for all of the users.
    $skuState = Get-MsolAccountSku | Where {$_.AccountSkuId -ieq $targetSkuId}

    if($skuState -eq $null)
    {
        Write-Host "Target license SKU $targetSkuId does not exist in the tenant at all."
        return $false
    }

    $availableLicenses = $skuState.ActiveUnits - $skuState.ConsumedUnits

    if($userIds.Count -gt $availableLicenses)
    {
        Write-Host "Not enough licenses for all users. User count: $($userIds.Count), licenses available: $availableLicenses"
        return $false
    }
    else
    {
        Write-Host "Enough $targetSkuId licenses available ($availableLicenses) for users: $($userIds.Count)."
    }

    # Check if each user to be migrated is in an expected state.
    $usersOK = 0
    $usersNotOK = 0
    foreach($userId in $userIds)
    {
        $user = GetUserObject $userId
        if(VerifyAssumptionsForUser $user $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId)
        {
            ++$usersOK
        }
        else
        {
            ++$usersNotOK
        }
    }
    if($usersNotOK -gt 0)
    {
        Write-Host "Checks passed for $usersOK users, but failed for $usersNotOK users."
        return $false
    }
    Write-Host "Checks passed for all $usersOK users."
    return $true
}

# ExecuteVerificationLoop function
# Execute a verification function (passed in as a delegate by using $checkFunction) for each user.
# The function tracks how many users passed/failed verification.
# The function repeats the verification loop until all of the users have passed the check.
#   The loop may never terminate if some users never reach the expected state.
#   If the loop doesn't terminate, you should investigate to determine the cause.
# Note: If the verification function fails with an exception,
#       such as the function detects an unexpected user state,
#       the loop terminates and investigation into the user state is needed.
function ExecuteVerificationLoop
{
    Param([System.Management.Automation.ScriptBlock]$checkFunction, [string]$consoleMessage, [object[]]$userIds,  [Guid]$sourceGroupId, [string]$sourceSkuId, [Guid]$targetGroupId, [string]$targetSkuId)

    # How long to wait until the loop is retried.
    $sleepIntervalSeconds = 60
    $retryIteration = 1

    While($true)
    {
        Write-Host "`n$consoleMessage. Check iteration: $retryIteration`n"

        $usersInExpectedState = 0
        $usersNotYet = 0

        foreach ($userId in $userIds)
        {
            $user = GetUserObject $userId
            if($checkFunction.InvokeReturnAsIs($user, $sourceGroupId, $sourceSkuId, $targetGroupId, $targetSkuId))
            {
                Write-Host "$userId`t`tExpected state: YES"
                ++$usersInExpectedState
            }
            else
            {
                Write-Host "$userId`t`tExpected state: NO"
                ++$usersNotYet
            }
        }

        Write-Host "`nTotal users checked: $($userIds.Count). In expected state: $usersInExpectedState. Not yet: $usersNotYet"

        if($usersNotYet -eq 0)
        {
            Write-Host "Check passed for all users. Exiting check loop."
            return
        }

        ++$retryIteration
        Write-Host "Not all users are in expected state. Waiting $sleepIntervalSeconds seconds to try again."
        Start-Sleep -Seconds $sleepIntervalSeconds
    }
}
# END: Helper functions that are used in the script.

# BEGIN: Execute the script.

# Enable strict execution mode.
Set-StrictMode -Version latest
# Stop the script when the first exception is thrown.
$ErrorActionPreference = "Stop"

# A batch of users that we want to migrate in this iteration.
# The batch can be specified as an array of User Principal Names (string) or ObjectIds (Guid).
# Note: The batch can be loaded from a text file that represents a larger batch of users that we want to migrate.
[string[]]$usersToMigrate = 'MigrationUser@tailspinonline.com', 'MigrationUser2@tailspinonline.com'

############### CONFLICTING LICENSES SCENARIO ################

# The source group and source license to remove the user from.
$sourceGroupId = [Guid]'b82c04f0-ce30-4ff1-bac7-735d92d83036'
$sourceSkuId = 'TailspinOnline:STANDARDPACK'             # <-- This is the Office 365 Enterprise E1 product.

# The target group and target license to assign to the user.
$targetGroupId = [Guid]'bcf279d1-5ad5-46a5-b469-4b8a552aa2fe'
$targetSkuId = 'TailspinOnline:ENTERPRISEPACK'           # <-- This is the Office 365 Enterprise E3 product.

# Assumptions before migration:
# 1. Users are already in the source group and they have the source license assigned from that group.
# 2. Users don't have the same source license assigned from another group at the same time,
#    and they don't have the source license assigned directly.
#    IMPORTANT: If Assumption 2 isn't true, removing users from the source group in STEP 3
#               won't result in the target license being correctly applied.
# 3. There are enough available licenses to assign a target license to all of the users that are being migrated.
if(-Not (VerifyAssumptions $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId))
{
    throw "Some users did not pass validation checks. See the output for details. Aborting migration process."
}

Write-Host "STEP 1: Adding users to the target group $targetGroupId. This will put target license $targetSkuId in conflict state with the source license $sourceSkuId"
AddUsersToGroup $usersToMigrate $targetGroupId

# Verify that the target license shows up in the conflict state for each user on the list.
# This step runs in a loop, forever, until all of the users are in the expected state.
# Group-based licensing (GBL) can take some time to reflect the changes on users.
# As a result, the loop should terminate after a period of time that's dependent on the size of the user collection.
# Note: If the loop hasn't terminated after a long period of time, stop the script.
#       Inspect the users that are reported as not yet in the expected state.
#       Verify that the users are not blocked for some other reason.
ExecuteVerificationLoop ${function:VerifySourceLicensePresentAndTargetLicenseInConflictState} 'STEP 2: Checking if all users still have the source license and are in conflict state for license from target group' $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId

# Now it's safe to remove the users from the source group.
Write-Host "STEP 3: Removing users from the source group $sourceGroupId. This will remove the source license $sourceSkuId and remove the conflict on target license $targetSkuId which will become assigned."
RemoveUsersFromGroup $usersToMigrate $sourceGroupId

# Verify that target license is now active on each user and the source license is removed.
ExecuteVerificationLoop ${function:VerifySourceLicenseRemovedAndTargetLicenseAssignedFromGroup} 'STEP 4: Checking if all users have source license removed and target license assigned from target group' $usersToMigrate $sourceGroupId $sourceSkuId $targetGroupId $targetSkuId

# END: Execute the script.
```

## <a name="next-steps"></a>後續步驟

請閱讀下列文章來深入了解透過群組管理授權的其他案例：

* [識別及解決 Azure Active Directory 中群組的授權指派問題](licensing-groups-resolve-problems.md)
* [Azure Active Directory 群組型授權的 PowerShell 範例](licensing-ps-examples.md)
* [Azure Active Directory 群組型授權其他案例 (英文)](licensing-group-advanced.md)
