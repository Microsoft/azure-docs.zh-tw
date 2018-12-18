---
title: Data Lake Storage Gen1 中的存取控制概觀 | Microsoft Docs
description: 了解 Azure Data Lake Storage Gen1 中的存取控制運作方式
services: data-lake-store
documentationcenter: ''
author: nitinme
manager: jhubbard
editor: cgronlun
ms.assetid: d16f8c09-c954-40d3-afab-c86ffa8c353d
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: nitinme
ms.openlocfilehash: fce96cf5be9e70863fd75e5d4b3045bc49f638cf
ms.sourcegitcommit: 7c4fd6fe267f79e760dc9aa8b432caa03d34615d
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/28/2018
ms.locfileid: "47432615"
---
# <a name="access-control-in-azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1 中的存取控制

Azure Data Lake Storage Gen1 實作的存取控制模型衍生自 HDFS，而 HDFS 又衍生自 POSIX 存取控制模型。 此文章摘要說明 Data Lake Storage Gen1 存取控制模型的基本概念。 

## <a name="access-control-lists-on-files-and-folders"></a>檔案和資料夾的存取控制清單

存取控制清單 (ACL) 有兩種類型，**存取 ACL** 和**預設 ACL**。

* **存取 ACL**：這些控制物件的存取權。 檔案和資料夾均有存取 ACL。

* **預設 ACL**：與資料夾相關聯之 ACL 的「範本」，用以判斷再該資料夾下建立的任何子項目的存取 ACL。 檔案沒有預設 ACL。


存取 ACL 和預設 ACL 有相同的結構。



> [!NOTE]
> 變更父代的預設 ACL 並不會影響現存子項目的存取 ACL 或預設 ACL。
>
>

## <a name="permissions"></a>權限

檔案系統物件的權限為 [讀取]、[寫入] 和 [執行]，這些權限可以用於下表所示的檔案和資料夾：

|            |    檔案     |   資料夾 |
|------------|-------------|----------|
| **讀取 (R)** | 可以讀取檔案的內容 | 需要 [讀取] 和 [執行] 才能列出資料夾內容|
| **寫入 (W)** | 可寫入或附加至檔案 | 需要 [寫入] 和 [執行] 才能在資料夾中建立子項目 |
| **執行 (X)** | 不表示 Data Lake Storage Gen1 內容中的任何項目 | 需要周遊資料夾的子項目 |

### <a name="short-forms-for-permissions"></a>權限的簡短形式

**RWX** 用來表示 [讀取 + 寫入 + 執行]。 有更壓縮的數字形式存在，其中 [讀取 = 4]、[寫入 = 2] 和 [執行 = 1]，其總和代表各種權限。 以下有一些範例。

| 數值形式 | 簡短形式 |      意義     |
|--------------|------------|------------------------|
| 7            | `RWX`        | 讀取 + 寫入 + 執行 |
| 5            | `R-X`        | 讀取 + 執行         |
| 4            | `R--`        | 讀取                   |
| 0            | `---`        | 沒有權限         |


### <a name="permissions-do-not-inherit"></a>不會繼承權限

在 Data Lake Storage Gen1 所使用的 POSIX 樣式模型中，項目的權限會儲存在項目本身。 換句話說，無法從父項目繼承項目的權限。

## <a name="common-scenarios-related-to-permissions"></a>權限相關的常見案例

以下是一些常見的案例，可協助您了解在 Data Lake Storage Gen1 帳戶上執行某些作業所需的權限。

|    作業             |    /    | Seattle/ | Portland/ | Data.txt     |
|--------------------------|---------|----------|-----------|--------------|
| Read Data.txt            |   `--X`   |   `--X`    |  `--X`      | `R--`          |
| Append to Data.txt       |   `--X`   |   `--X`    |  `--X`      | `RW-`          |
| Delete Data.txt          |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| Create Data.txt          |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| List /                   |   `R-X`   |   `---`    |  `---`      | `---`          |
| List /Seattle/           |   `--X`   |   `R-X`    |  `---`      | `---`          |
| List /Seattle/Portland/  |   `--X`   |   `--X`    |  `R-X`      | `---`          |


> [!NOTE]
> 只要前面的兩個條件成立，刪除檔案時就不需要其寫入權限。
>
>


## <a name="users-and-identities"></a>使用者和身分識別

每個檔案和資料夾都有這些身分識別的不同權限︰

* 擁有使用者
* 擁有群組
* 具名使用者
* 具名群組
* 所有其他使用者

使用者和群組的身分識別皆為 Azure Active Directory (Azure AD) 身分識別。 因此，除非另有註明，否則 Data Lake Storage Gen1 中的「使用者」可能表示 Azure AD 使用者或 Azure AD 安全性群組。

### <a name="the-super-user"></a>超級使用者

超級使用者具有 Data Lake Storage Gen1 帳戶中所有使用者的大多數權限。 超級使用者：

* 具有**所有**檔案和資料夾的 RWX 權限。
* 可以變更任何檔案或資料夾的權限。
* 可以變更任何檔案或資料夾的擁有使用者或擁有群組。

身為 Data Lake Storage Gen1 帳戶 [擁有者] 角色成員的所有使用者都會自動成為超級使用者。

### <a name="the-owning-user"></a>擁有使用者

建立項目的使用者會自動成為項目的擁有使用者。 擁有使用者可以︰

* 變更所擁有檔案的權限。
* 只要擁有使用者也是目標群組的成員，請變更所擁有檔案的擁有群組。

> [!NOTE]
> 擁有使用者「無法」變更檔案或資料夾的擁有使用者。 只有超級使用者可以變更檔案或資料夾的擁有使用者。
>
>

### <a name="the-owning-group"></a>擁有群組

**背景**

在 POSIX ACL 中，每個使用者都與「主要群組」相關聯。 例如，使用者 "alice" 可能屬於 "finance" 群組。 Alice 也可能屬於多個群組，但一定有一個群組指定為其主要群組。 在 POSIX 中，當 Alice 會建立檔案時，該檔案的擁有群組會設定為她的主要群組，在此案例中為 "finance"。 除此之外，擁有群組的作用類似於指派給其他使用者/群組的權限。

**指派新檔案或資料夾的擁有群組**

* **案例 1**：根資料夾 "/"。 建立 Data Lake Storage Gen1 帳戶時，會建立這個資料夾。 在此情況下，擁有群組會設定為建立帳戶的使用者。
* **案例 2** (其他所有案例)：建立新項目時，會從父資料夾複製擁有群組。

**變更擁有群組**

可以變更擁有群組的對象︰
* 任何超級使用者。
* 擁有使用者，如果擁有使用者也是目標群組的成員。

> [!NOTE]
> 擁有群組「無法」變更檔案或資料夾的 ACL。  如果將擁有群組設定為在上述根資料夾案例 (**案例 1**) 中建立帳戶的使用者，則無法透過擁有群組提供權限給單一使用者帳戶。  您可以將此權限指派給有效的使用者群組 (如果適用的話)。


## <a name="access-check-algorithm"></a>存取檢查演算法

以下的虛擬程式碼代表 Data Lake Storage Gen1 帳戶的存取檢查演算法。

```
def access_check( user, desired_perms, path ) : 
  # access_check returns true if user has the desired permissions on the path, false otherwise
  # user is the identity that wants to perform an operation on path
  # desired_perms is a simple integer with values from 0 to 7 ( R=4, W=2, X=1). User desires these permissions
  # path is the file or folder
  # Note: the "sticky bit" is not illustrated in this algorithm
  
# Handle super users.
  if (is_superuser(user)) :
    return True

  # Handle the owning user. Note that mask IS NOT used.
  entry = get_acl_entry( path, OWNER )
  if (user == entry.identity)
      return ( (desired_perms & e.permissions) == desired_perms )

  # Handle the named users. Note that mask IS used.
  entries = get_acl_entries( path, NAMED_USER )
  for entry in entries:
      if (user == entry.identity ) :
          mask = get_mask( path )
          return ( (desired_perms & entry.permmissions & mask) == desired_perms)

  # Handle named groups and owning group
  member_count = 0
  perms = 0
  entries = get_acl_entries( path, NAMED_GROUP | OWNING_GROUP )
  for entry in entries:
    if (user_is_member_of_group(user, entry.identity)) :
      member_count += 1
      perms | =  entry.permissions
  if (member_count>0) :
    return ((desired_perms & perms & mask ) == desired_perms)
 
  # Handle other
  perms = get_perms_for_other(path)
  mask = get_mask( path )
  return ( (desired_perms & perms & mask ) == desired_perms)
```

### <a name="the-mask"></a>遮罩

如＜存取檢查演算法＞所述，遮罩會限制**具名使用者**、**擁有群組**及**具名群組**的存取權。  

> [!NOTE]
> 對於新的 Data Lake Storage Gen1 帳戶，根資料夾 ("/") 的存取 ACL 遮罩會預設為 RWX。
>
>

### <a name="the-sticky-bit"></a>黏性位元

黏性位元是 POSIX 檔案系統的更進階功能。 在 Data Lake Storage Gen1 的內容中，不太可能需要黏性位元。 總而言之，如果已在資料夾上啟用黏性位元，子項目便只能由子項目的擁有使用者刪除或重新命名。

黏性位元不會顯示在 Azure 入口網站中。

## <a name="default-permissions-on-new-files-and-folders"></a>新檔案和資料夾的預設權限

在現有資料夾之下建立新檔案或資料夾時，父資料夾的預設 ACL 可決定︰

- 子資料夾的預設 ACL 與存取 ACL。
- 子檔案的存取 ACL (檔案沒有預設 ACL)。

### <a name="umask"></a>umask

建立檔案或資料夾時，可使用 umask 來修改子項目上的預設 ACL 設定方式。 umask 是一個在父資料夾上的 9 位元值，其中包含一個用於**擁有使用者**、**擁有群組**及**其他**的 RWX 值。

Azure Data Lake Storage Gen1 的 umask 是一個設定為 007 的常數值。 此值會轉譯成

| umask 元件     | 數值形式 | 簡短形式 | 意義 |
|---------------------|--------------|------------|---------|
| umask.owning_user   |    0         |   `---`      | 針對擁有使用者，將父系的「預設 ACL」複製到子系的「存取 ACL」 | 
| umask.owning_group  |    0         |   `---`      | 針對擁有群組，將父系的「預設 ACL」複製到子系的「存取 ACL」 | 
| umask.other         |    7         |   `RWX`      | 針對其他，移除子系「存取 ACL」上的所有權限 |

Azure Data Lake Storage Gen1 所使用的 umask 值實際上意謂著不論「預設 ACL」的指示為何，預設一律不會在新子系上傳輸用於「其他」的值。 

下列虛擬程式碼說明為子項目建立 ACL 時套用 umask 的方式。

```
def set_default_acls_for_new_child(parent, child):
    child.acls = []
    for entry in parent.acls :
        new_entry = None
        if (entry.type == OWNING_USER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_user))
        elif (entry.type == OWNING_GROUP) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_group))
        elif (entry.type == OTHER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.other))
        else :
            new_entry = entry.clone(perms = entry.perms )
        child_acls.add( new_entry )
```

## <a name="common-questions-about-acls-in-data-lake-storage-gen1"></a>Data Lake Storage Gen1 中 ACL 的相關常見問題

### <a name="do-i-have-to-enable-support-for-acls"></a>我必須啟用 ACL 的支援嗎？

否。 Data Lake Storage Gen1 帳戶一律會啟用透過 ACL 的存取控制。

### <a name="which-permissions-are-required-to-recursively-delete-a-folder-and-its-contents"></a>若要以遞迴方式刪除資料夾與其內容，需要哪些權限？

* 父資料夾必須具有 [寫入 + 執行] 權限。
* 要刪除的資料夾及其中的每個資料夾，都需要 [讀取 + 寫入 + 執行] 權限。

> [!NOTE]
> 您不需要寫入權限即可刪除資料夾中的檔案。 此外，**決不**會刪除根資料夾 "/"。
>
>

### <a name="who-is-the-owner-of-a-file-or-folder"></a>誰是檔案或資料夾的擁有者？

檔案或資料夾的建立者會成為擁有者。

### <a name="which-group-is-set-as-the-owning-group-of-a-file-or-folder-at-creation"></a>在建立檔案或資料夾時，會將哪個群組設定為擁有群組？

擁有群組是從新檔案或資料夾建立所在父資料夾的擁有群組複製而來。

### <a name="i-am-the-owning-user-of-a-file-but-i-dont-have-the-rwx-permissions-i-need-what-do-i-do"></a>我是檔案的擁有使用者，但沒有我需要的 RWX 權限。 該怎麼辦？

擁有使用者可以變更檔案的權限，以取得本身所需的任何 RWX 權限。

### <a name="when-i-look-at-acls-in-the-azure-portal-i-see-user-names-but-through-apis-i-see-guids-why-is-that"></a>當我在 Azure 入口網站中查看 ACL 時，我可看到使用者名稱，但透過 API 卻看到 GUID，為什麼會這樣？

ACL 中的項目會儲存為對應於 Azure AD 中使用者的 GUID。 API 會依現狀傳回 GUID。 Azure 入口網站會儘可能將 GUID 轉譯成好記的名稱，試圖讓 ACL 更容易使用。

### <a name="why-do-i-sometimes-see-guids-in-the-acls-when-im-using-the-azure-portal"></a>當我使用 Azure 入口網站時，為什麼有時候會在 ACL 中看到 GUID？

當使用者已不存在於 Azure AD 時，將會顯示 GUID。 當使用者已離開公司，或已在 Azure AD 中刪除其帳戶時，通常會發生這種情形。

### <a name="does-data-lake-storage-gen1-support-inheritance-of-acls"></a>Data Lake Storage Gen1 是否支援 ACL 的繼承？

否，但預設 ACL 可以用來為父資料夾下新建的子檔案和資料夾設定 ACL。  

### <a name="where-can-i-learn-more-about-posix-access-control-model"></a>何處可以進一步了解 POSIX 存取控制模型？

* [Linux 上的 POSIX 存取控制清單](https://www.linux.com/news/posix-acls-linux)
* [HDFS 權限指南](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html)
* [POSIX 常見問題集](http://www.opengroup.org/austin/papers/posix_faq.html)
* [POSIX 1003.1 2008](http://standards.ieee.org/findstds/standard/1003.1-2008.html)
* [POSIX 1003.1 2013](http://pubs.opengroup.org/onlinepubs/9699919799.2013edition/)
* [POSIX 1003.1 2016](http://pubs.opengroup.org/onlinepubs/9699919799.2016edition/)
* [Ubuntu 上的 POSIX ACL](https://help.ubuntu.com/community/FilePermissionsACLs)
* [Linux 上使用存取控制清單的 ACL](http://bencane.com/2012/05/27/acl-using-access-control-lists-on-linux/)

## <a name="see-also"></a>另請參閱

* [Azure Data Lake Storage Gen1 概觀](data-lake-store-overview.md)
