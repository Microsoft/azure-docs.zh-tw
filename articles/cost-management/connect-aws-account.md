---
title: 將 Amazon Web Services 帳戶連線至 Azure 中的 Cloudyn | Microsoft Docs
description: 連線 Amazon Web Services 帳戶以在 Cloudyn 報告中檢視成本和使用情況資料。
services: cost-management
keywords: ''
author: bandersmsft
ms.author: banders
ms.date: 08/07/2018
ms.topic: conceptual
ms.service: cost-management
manager: dougeby
ms.custom: ''
ms.openlocfilehash: 44bf1d9cd270394720aee71862c1e65118084259
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46978214"
---
# <a name="connect-an-amazon-web-services-account"></a>連線 Amazon Web Services 帳戶

您有兩個選項可將 Amazon Web Services (AWS) 帳戶連線至 Cloudyn。 您可以與 IAM 角色或唯讀 IAM 使用者帳戶連線。 建議項目是 IAM 角色，因為它可以讓您將具有已定義權限的存取權委派給信任的實體。 IAM 角色不需要您共用長期存取金鑰。 將 AWS 帳戶連線至 Cloudyn 之後，Cloudyn 報告便會提供成本和使用情況資料。 這份文件會引導您進行這兩個選項。

如需 AWS IAM 身分識別的詳細資訊，請參閱[身分識別 (使用者、群組和角色)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html)。

此外，您會啟用 AWS 詳細帳單報告，並且將資訊儲存在 AWS 簡單儲存體服務 (S3) 貯體。 詳細帳單報告包含按時數計算的標記和資源資訊費用。 儲存報告可讓 Cloudyn 從您的貯體擷取這些報告，並在其報告中顯示資訊。


## <a name="aws-role-based-access"></a>AWS 角色型存取

下列各節會引導您完成建立唯讀 IAM 角色以提供 Cloudyn 存取權的程序。

### <a name="get-your-cloudyn-account-external-id"></a>取得您的 Cloudyn 帳戶外部 ID

第一個步驟是從 Cloudyn 入口網站取得唯一的連線複雜密碼。 它在 AWS 中用來作為**外部 ID**。

1. 從 Azure 入口網站中開啟 Cloudyn 入口網站，或瀏覽至 [https://azure.cloudyn.com](https://azure.cloudyn.com) 並登入。
2. 按一下齒輪符號，然後選取 [雲端帳戶]。
3. 在 [帳戶管理] 中，選取 [AWS 帳戶] 索引標籤，然後按一下 [新增 +]。
4. 在 [新增 AWS 帳戶] 對話方塊中，複製**外部識別碼**，並且儲存這個值以供下一節的 AWS 角色建立步驟使用。 外部識別碼對於您的帳戶是唯一的。 在下圖中，外部識別碼範例是 _Contoso_加上數字。 您的 ID 不同。  
    ![外部 ID](./media/connect-aws-account/external-id.png)

### <a name="add-aws-read-only-role-based-access"></a>新增 AWS 唯讀角色型存取

1. 登入 AWS 主控台 (位於 https://console.aws.amazon.com/iam/home )，然後選取 [角色]。
2. 按一下 [建立角色]，然後選取 [另一個 AWS 帳戶]。
3. 在 [帳戶識別碼] 方塊中，貼上 `432263259397`。 這個「帳戶識別碼」是 AWS 指派給 Cloudyn 服務的 Cloudyn 資料收集器帳戶。 使用顯示的確切帳戶識別碼。
4. 在 [選項] 旁邊，選取 [需要外部識別碼]。 貼上先前從 Cloudyn 中 [外部識別碼] 欄位複製的唯一值。 然後按 [下一步：權限]。  
    ![建立角色](./media/connect-aws-account/create-role01.png)
5. 在 [附加權限原則] 底下的 [原則類型] 篩選方塊搜尋中，輸入 `ReadOnlyAccess`，選取 [ReadOnlyAccess]，然後按 [下一步：檢閱]。  
    ![唯讀存取](./media/connect-aws-account/readonlyaccess.png)
6. 在 [檢閱] 分頁中，確定您的選擇正確無誤，然後輸入**角色名稱**。 例如，*Azure-Cost-Mgt*。輸入**角色描述**。 例如，_Cloudyn 的角色指派_，然後按一下 [Create role] \(建立角色\)。
7. 在 [角色] 清單中，按一下您建立的角色，然後從 [摘要] 分頁中複製 **角色 ARN** 值。 稍後當您在 Cloudyn 中註冊設定時，請使用 [Role ARN] \(角色 ARN\) (Amazon Resource Name) 值。  
    ![角色 ARN](./media/connect-aws-account/role-arn.png)

### <a name="configure-aws-iam-role-access-in-cloudyn"></a>在 Cloudyn 中設定 AWS IAM 角色存取

1. 從 Azure 入口網站中開啟 Cloudyn 入口網站，或瀏覽至 https://azure.cloudyn.com/ 並登入。
2. 按一下齒輪符號，然後選取 [雲端帳戶]。
3. 在 [帳戶管理] 中，選取 [AWS 帳戶] 索引標籤，然後按一下 [新增 +]。
4. 在 [帳戶名稱] 中，輸入帳戶的名稱。
5. 在 [存取類型] 旁邊，選取 [IAM 角色]。
6. 在 [角色 ARN] 欄位中，貼上您先前複製的值，然後按一下 [儲存]。  
    ![新增 AWS 帳戶方塊](./media/connect-aws-account/add-aws-account-box.png)


您的 AWS 帳戶會出現在帳戶清單中。 列出的**擁有者 ID** 與您的角色 ARN 值相符。 您的 [Account Status] \(帳戶狀態\) 應該會有綠色核取記號符號，表示 Cloudyn 可以存取您的 AWS 帳戶。 在您啟用詳細 AWS 帳單之前，您的彙總狀態會顯示為「獨立」。

![AWS 帳戶狀態](./media/connect-aws-account/aws-account-status01.png)

Cloudyn 會開始收集資料並填入報告。 接下來，[啟用詳細 AWS 帳單](#enable-detailed-aws-billing)。


## <a name="aws-user-based-access"></a>AWS 以使用者為基礎的存取

下列各節會引導您完成建立唯讀使用者以提供 Cloudyn 存取權的程序。

### <a name="add-aws-read-only-user-based-access"></a>新增 AWS 唯讀以使用者為基礎的存取

1. 登入 AWS 主控台 (位於 https://console.aws.amazon.com/iam/home )，然後選取 [使用者]。
2. 按一下 [新增使用者] 。
3. 在 [使用者名稱] 欄位中，輸入使用者名稱。
4. 針對 [存取類型]，選取 [以程式設計方式存取]，然後按 [下一步：權限]。  
    ![新增使用者](./media/connect-aws-account/add-user01.png)
5. 針對權限，選取 [直接附加現有的原則] 按鈕。
6. 在 [附加權限原則] 底下的 [原則類型] 篩選方塊搜尋中，輸入 `ReadOnlyAccess`，選取 [ReadOnlyAccess]，然後按 [下一步：檢閱]。  
    ![設定使用者的權限](./media/connect-aws-account/set-permission-for-user.png)
7. 在 [檢閱] 分頁中，確定您的選擇正確無誤，然後按一下 [建立使用者]。
8. 在 [完成] 分頁中，會顯示您的存取金鑰 ID 和密碼存取金鑰。 您需使用此資訊在 Cloudyn 中設定註冊。
9. 按一下 [下載 .csv] 以將 credentials.csv 檔案儲存到安全的位置。  
    ![下載認證](./media/connect-aws-account/download-csv.png)

### <a name="configure-aws-iam-user-based-access-in-cloudyn"></a>在 Cloudyn 中設定 AWS IAM 使用者型存取

1. 從 Azure 入口網站中開啟 Cloudyn 入口網站，或瀏覽至 https://azure.cloudyn.com/ 並登入。
2. 按一下齒輪符號，然後選取 [雲端帳戶]。
3. 在 [帳戶管理] 中，選取 [AWS 帳戶] 索引標籤，然後按一下 [新增 +]。
4. 針對 [帳戶名稱]，輸入帳戶名稱。
5. 在 [存取類型] 旁邊，選取 [IAM 使用者]。
6. 在 [存取金鑰] 中，貼上來自 credentials.csv 檔案的**存取金鑰 ID** 值。
7. 在 [秘密金鑰] 中，貼上來自 credentials.csv 檔案的**秘密存取金鑰**值，然後按一下 [儲存]。  

您的 AWS 帳戶會出現在帳戶清單中。 您的**帳戶狀態**應該有一個綠色核取符號。

Cloudyn 會開始收集資料並填入報告。 接下來，[啟用詳細 AWS 帳單](#enable-detailed-aws-billing)。

## <a name="enable-detailed-aws-billing"></a>啟用詳細 AWS 帳單

使用下列步驟以取得您的 AWS 角色 ARN。 使用角色 ARN 將讀取權限授與帳單貯體。

1. 登入 AWS 主控台 (位於 https://console.aws.amazon.com )，然後選取 [服務]。
2. 在 [服務搜尋] 方塊中，輸入「IAM」，然後選取該選項。
3. 從左側功能表選取 [角色]。
4. 在 [角色] 清單中，選取您為 Cloudyn 存取所建立的角色。
5. 在 [角色摘要] 頁面上，按一下以複製 [角色 ARN]。 請就近保存 [角色 ARN] 以在後續步驟中使用。

### <a name="create-an-s3-bucket"></a>建立 S3 貯體

建立 S3 貯體以儲存詳細帳單資訊。

1. 登入 AWS 主控台 (位於 https://console.aws.amazon.com )，然後選取 [服務]。
2. 在 [服務搜尋] 方塊中，輸入「S3」，然後選取 [S3]。
3. 在 [Amazon S3] 頁面上，按一下 [建立貯體]。
4. 在建立貯體精靈中，選擇貯體名稱和區域，然後按 [下一步]。  
    ![建立貯體](./media/connect-aws-account/create-bucket.png)
5. 在 [設定屬性] 頁面上，保留預設值，然後按 [下一步]。
6. 在 [檢閱] 頁面上，按一下 [建立貯體]。 貯體清單隨即顯示。
7. 按一下您建立的貯體，選取 [權限] 索引標籤，然後選取 [貯體原則]。 貯體原則編輯器隨即開啟。
8. 複製下列 JSON 範例，並且將它貼至貯體原則編輯器。
  - 將 `<BillingBucketName>` 取代為您的 S3 貯體名稱。
  - 將 `<ReadOnlyUserOrRole>` 取代為您先前複製的角色或使用者 ARN。

  ```
  {
    "Version": "2012-10-17",
    "Id": "Policy1426774604000",
    "Statement": [
        {
            "Sid": "Stmt1426774604000",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::386209384616:root"
            },
            "Action": [
                "s3:GetBucketAcl",
                "s3:GetBucketPolicy"
            ],
            "Resource": "arn:aws:s3:::<BillingBucketName>"
        },
        {
            "Sid": "Stmt1426774604001",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::386209384616:root"
            },
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::<BillingBucketName>/*"
        },
        {
            "Sid": "Stmt1426774604002",
            "Effect": "Allow",
            "Principal": {
                "AWS": "<ReadOnlyUserOrRole>"
            },
            "Action": [
                "s3:List*",
                "s3:Get*"
            ],
            "Resource": "arn:aws:s3:::<BillingBucketName>/*"
        }
    ]
  }
  ```

9. 按一下 [檔案] 。  
    ![貯體原則編輯器](./media/connect-aws-account/bucket-policy-editor.png)


### <a name="enable-aws-billing-reports"></a>啟用 AWS 帳單報告

建立及設定 S3 貯體之後，瀏覽至 AWS 主控台中的 [帳單喜好設定](https://console.aws.amazon.com/billing/home?#/preference)。

1. 在 [喜好設定] 頁面上，選取 [接收帳單報告]。
2. 在 [接收帳單報告] 下，輸入您建立的貯體名稱，然後按一下 [驗證]。  
3. 選取全部 4 個報告細微性選項，然後按一下 [儲存喜好設定]。  
    ![啟用報告](./media/connect-aws-account/enable-reports.png)

Cloudyn 會從您的 S3 貯體擷取詳細帳單資訊，然後在啟用詳細帳單功能之後填入報告。 可能需要多達 24 小時才能讓詳細帳單資料出現在 Cloudyn 主控台。 詳細帳單資料可用時，您的帳戶彙總狀態會顯示為「已彙總」。 帳戶狀態會顯示為「已完成」。

![帳戶彙總狀態](./media/connect-aws-account/consolidated-status.png)

某些最佳化報告可能需要數天的資料，以取得足夠的資料樣本數來提供精確建議。

## <a name="next-steps"></a>後續步驟

- 若要深入了解 Cloudyn，請繼續進行 Cloudyn 的[檢閱使用量和成本](tutorial-review-usage.md)教學課程。
