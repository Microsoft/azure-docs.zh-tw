---
title: 在 Azure 中為 Cloudyn 設定儲存體帳戶 | Microsoft Docs
description: 本文說明如何為 Cloudyn 設定 Azure 儲存體帳戶和 AWS 儲存體貯體。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 09/18/2018
ms.topic: conceptual
ms.service: cost-management
manager: carmonm
ms.custom: ''
ms.openlocfilehash: 829995d40a73dc181a28a467e4d16fef0bdeb454
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46990649"
---
# <a name="configure-storage-accounts-for-cloudyn"></a>為 Cloudyn 設定儲存體帳戶

<!--- intent: As a Cloudyn user, I want to configure Cloudyn to use my cloud service provider storage account to store my reports. -->

您可以將 Cloudyn 報表儲存在 Cloudyn 入口網站、Azure 儲存體或 AWS 儲存體貯體中。 將報表儲存至 Cloudyn 入口網站中是免費的。 但您也可以選擇將報表儲存至雲端服務提供者的儲存體，只是那會產生額外費用。 本文將協助您設定 Azure 儲存體帳戶和 Amazon Web Services (AWS) 儲存體貯體，以儲存您的報表。

## <a name="prerequisites"></a>先決條件

您必須具有 Azure 儲存體帳戶或 Amazon 儲存體貯體。

如果您沒有 Azure 儲存體帳戶，則需要建立一個。 如需有關建立 Azure 儲存體帳戶的詳細資訊，請參閱[建立儲存體帳戶](../storage/common/storage-quickstart-create-account.md)。

如果您沒有 AWS Simple Storage Service (S3) 貯體，則需要建立一個。 如需有關建立 S3 貯體的詳細資訊，請參閱[建立貯體](https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html)。

## <a name="configure-your-azure-storage-account"></a>設定 Azure 儲存體帳戶

設定 Azure 儲存體以供 Cloudyn 使用相當簡單。 請收集儲存體帳戶的相關詳細資料，然後將它們複製到 Cloudyn 入口網站中。

1. 在 http://portal.azure.com 登入 Azure 入口網站。
2. 按一下 [所有服務]，選取 [儲存體帳戶]，捲動至您想要使用的儲存體帳戶，然後選取該帳戶。
3. 在儲存體帳戶頁面上的 [設定] 底下，按一下 [存取金鑰]。
4. 複製您的 [儲存體帳戶名稱] 和 key1 底下的 [連接字串]。  
![Azure 儲存體存取金鑰](./media/storage-accounts/azure-storage-access-keys.png)  
5. 從 Azure 入口網站中開啟 Cloudyn 入口網站，或瀏覽至 https://azure.cloudyn.com 並登入。
6. 按一下齒輪符號，然後選取 [Reports Storage Management] \(報表儲存體管理\)。
7. 按一下 [Add new +] \(新增 +\)，然後確認已選取 [Microsoft Azure]。 將您的 Azure 儲存體帳戶名稱貼到 [Name] \(名稱\) 區域中。 將您的**連接字串**貼到對應的區域中。 輸入容器名稱，然後按一下 [Save] \(儲存\)。  
![針對 Azure 設定的 Cloudyn 儲存體](./media/storage-accounts/azure-cloudyn-storage.png)

  您的新 Azure 報表儲存體項目會出現在儲存體帳戶清單中。  
    ![清單中的新 Azure 報表儲存體](./media/storage-accounts/azure-storage-entry.png)


您現在已可將報表儲存至 Azure 儲存體。 在任何報表中，按一下 [Actions] \(動作\)，然後選取 [Schedule report] \(排程報表\)。 為報告命名，然後新增您自己的 URL 或使用自動建立的 URL。 選取 [Save to storage] \(儲存至儲存體\)，然後選取儲存體帳戶。 輸入要附加至報告檔案名稱的前置詞。 選取 CSV 或 JSON 檔案格式，然後儲存報表。

## <a name="configure-an-aws-storage-bucket"></a>設定 AWS 儲存體貯體

Cloudyn 會使用現有的 AWS 認證 (使用者或角色) 將報表儲存至您的貯體。 為了測試存取，Cloudyn 會嘗試將一個小文字檔以 _check-bucket-permission.txt_ 檔案名稱儲存至貯體。

您需提供具備您貯體之 PutObject 權限的 Cloudyn 角色或使用者。 然後，使用現有的貯體或建立新貯體來儲存報表。 最後，決定如何管理儲存體類別、設定生命週期規則，或移除任何不必要的檔案。

###  <a name="assign-permissions-to-your-aws-user-or-role"></a>將權限指派給 AWS 使用者或角色

建立新原則時，您需提供將報表儲存至 S3 貯體所需的確切權限。

1. 登入 AWS 主控台，然後選取 [Services] \(服務\)。
2. 從服務清單中，選取 [IAM]。
3. 選取主控台左側的 [Policies] \(原則\)，然後按一下 [Create Policy] \(建立原則\)。
4. 按一下 [JSON] 索引標籤。
5. 下列原則可讓您將報表儲存至 S3 貯體。 複製下列原則範例並貼到 [JSON] 索引標籤中。將 &lt;bucketname&gt; 取代成您的貯體名稱。

  ```
{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid":  "CloudynSaveReport2S3",
        "Effect":      "Allow",
        "Action": [
          "s3:PutObject"
        ],
        "Resource": [
          "arn:aws:s3:::<bucketname>/*"
        ]
      }
    ]
}
```

6. 按一下 [Review policy] \(檢閱原則\)。  
    ![檢閱原則](./media/storage-accounts/aws-policy.png)  
7. 在 [Review policy] \(檢閱原則\) 頁面上，輸入您的原則名稱。 例如 _CloudynSaveReport2S3_。
8. 按一下 [Create policy] \(建立原則\)。

### <a name="attach-the-policy-to-a-cloudyn-role-or-user-in-your-account"></a>將原則連結至您帳戶中的 Cloudyn 角色或使用者

若要連結新原則，您需開啟 AWS 主控台，然後編輯 Cloudyn 角色或使用者。

1. 登入 AWS 主控台，然後選取 [Services] \(服務\)，再從服務清單中選取 [IAM]。
2. 從主控台的左側選取 [Roles] \(角色\) 或 [Users] \(使用者\)。

**針對角色：**

  1. 按一下您的 Cloudyn 角色名稱。
  2. 在 [Permissions] \(權限\) 索引標籤上，按一下 [Attach Policy] \(連結原則\)。
  3. 搜尋您已建立的原則並選取它，然後按一下 [Attach Policy] \(連結原則\)。
    ![AWS - 連結角色的原則](./media/storage-accounts/aws-attach-policy-role.png)

**針對使用者：**

1. 選取 [Cloudyn User] \(Cloudyn 使用者\)。
2. 在 [Permissions] \(權限\) 索引標籤上，按一下 [Add permissions] \(新增權限\)。
3. 在 [Grant Permission] \(授與權限\) 區段中，選取 [Attach existing policies directly] \(直接連結現有的原則\)。
4. 搜尋您已建立的原則並選取它，然後按一下 [Next: Review] \(下一步: 檢閱\)。
5. 在 [Add permissions to role name] \(將權限新增至角色名稱\) 頁面上，按一下 [Add permissions] \(新增權限\)。  
    ![AWS - 連結使用者的原則](./media/storage-accounts/aws-attach-policy-user.png)


### <a name="optional-set-permission-with-bucket-policy"></a>選擇性：使用貯體原則來設定權限

您也可以使用貯體原則來設定權限，以在 S3 貯體上建立報表。 在傳統 S3 檢視中：

1. 建立或選取一個現有的貯體。
2. 選取 [Permissions] \(權限\) 索引標籤，然後按一下 [Bucket policy] \(貯體原則\)。
3. 複製並貼上下列原則範例。 將 &lt;bucket\_name&gt; 和 &lt;Cloudyn\_principle&gt; 取代成您貯體的 ARN。 取代 Cloudyn 所使用角色或使用者的 ARN。

  ```
{
  "Id": "Policy1485775646248",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SaveReport2S3",
      "Action": [
        "s3:PutObject"
      ],
      "Effect": "Allow",
      "Resource": "<bucket_name>/*",
      "Principal": {
        "AWS": [
          "<Cloudyn_principle>"
        ]
      }
    }
  ]
}
```

4. 在貯體原則編輯器中，按一下 [Save] \(儲存\)。

### <a name="add-aws-report-storage-to-cloudyn"></a>將 AWS 報表儲存體新增至 Cloudyn

1. 從 Azure 入口網站中開啟 Cloudyn 入口網站，或瀏覽至 https://azure.cloudyn.com 並登入。
2. 按一下齒輪符號，然後選取 [Reports Storage Management] \(報表儲存體管理\)。
3. 按一下 [Add new +] \(新增 +\)，然後確認已選取 [AWS]。
4. 選取帳戶和儲存體貯體。 系統會自動填入 AWS 儲存體貯體。  
    ![為 AWS 貯體新增報表儲存體](./media/storage-accounts/aws-cloudyn-storage.png)  
5. 按一下 [Save] \(儲存\)，然後按一下 [OK] \(確定\)。

    您的新 AWS 報表儲存體項目會出現在儲存體帳戶清單中。  
    ![清單中的新 AWS 報表儲存體](./media/storage-accounts/aws-storage-entry.png)


您現在已可將報表儲存至 Azure 儲存體。 在任何報表中，按一下 [Actions] \(動作\)，然後選取 [Schedule report] \(排程報表\)。 為報告命名，然後新增您自己的 URL 或使用自動建立的 URL。 選取 [Save to storage] \(儲存至儲存體\)，然後選取儲存體帳戶。 輸入要附加至報告檔案名稱的前置詞。 選取 CSV 或 JSON 檔案格式，然後儲存報表。

## <a name="next-steps"></a>後續步驟

- 檢閱[了解 Cloudyn 報表](understanding-cost-reports.md)，以了解 Cloudyn 報表的基本結構和功能。
