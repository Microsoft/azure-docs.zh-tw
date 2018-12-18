---
title: 使用 Windows VM 系統指派的受控識別來存取 Azure SQL
description: 本教學課程逐步引導您使用 Windows VM 系統指派的受控識別，以存取 Azure SQL。
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: bryanla
ms.service: active-directory
ms.component: msi
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/20/2017
ms.author: daveba
ms.openlocfilehash: 57f9def09f498c3fc644cbee979d5ae552f2206c
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46369336"
---
# <a name="tutorial-use-a-windows-vm-system-assigned-managed-identity-to-access-azure-sql"></a>教學課程：使用 Windows VM 系統指派的受控識別來存取 Azure SQL

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

本教學課程說明如何將系統指派的身分識別用於 Windows 虛擬機器 (VM)，以存取 Azure SQL 伺服器。 受控服務身分識別由 Azure 自動管理，並可讓您驗證支援 Azure AD 驗證的服務，而不需要將認證插入程式碼中。 您會了解如何：

> [!div class="checklist"]
> * 將您的 VM 存取權授與 Azure SQL 伺服器
> * 在 Azure AD 中建立群組，並讓 VM 系統指派的受控識別成為此群組的成員
> * 啟用 SQL Server 的 Azure AD 驗證
> * 在代表 Azure AD 群組的資料庫中建立包含的使用者
> * 使用 VM 身分識別取得存取權杖，並使用它查詢 Azure SQL 伺服器

## <a name="prerequisites"></a>必要條件

[!INCLUDE [msi-qs-configure-prereqs](../../../includes/active-directory-msi-qs-configure-prereqs.md)]

[!INCLUDE [msi-tut-prereqs](../../../includes/active-directory-msi-tut-prereqs.md)]

- [登入 Azure 入口網站](https://portal.azure.com)

- [建立 Windows 虛擬機器](/azure/virtual-machines/windows/quick-create-portal)

- [在虛擬機器上啟用系統指派的受控識別](/azure/active-directory/managed-service-identity/qs-configure-portal-windows-vm#enable-system-assigned-identity-on-an-existing-vm)

## <a name="grant-your-vm-access-to-a-database-in-an-azure-sql-server"></a>將您的 VM 存取權授與 Azure SQL 伺服器中的資料庫

現，在您可以將 VM 存取權授與 Azure SQL 伺服器中的資料庫。  在這個步驟，您可以使用現有的 SQL 伺服器或建立新的。  若要使用 Azure 入口網站建立新的伺服器和資料庫，請遵循此 [Azure SQL 快速入門](https://docs.microsoft.com/azure/sql-database/sql-database-get-started-portal)的作法。 在 [Azure SQL 文件](https://docs.microsoft.com/azure/sql-database/)中也有使用 Azure CLI 和 Azure PowerShell 的快速入門作法。

將 VM 存取權授與資料庫有三個步驟：
1.  在 Azure AD 中建立群組，並讓 VM 系統指派的受控識別成為此群組的成員。
2.  啟用 SQL Server 的 Azure AD 驗證。
3.  在代表 Azure AD 群組的資料庫中建立**包含的使用者**。

> [!NOTE]
> 通常您會建立「包含的使用者」，並將此使用者直接對應到 VM 系統指派的受控識別。  若 Azure AD 服務主體代表 VM 系統指派的受控識別，Azure SQL 目前並不允許此服務主體對應到包含的使用者。  支援的因應措施是，讓 VM 系統指派的受控識別成為 Azure AD 群組的成員，然後在代表該群組的資料庫中建立包含的使用者。


## <a name="create-a-group-in-azure-ad-and-make-the-vms-system-assigned-managed-identity-a-member-of-the-group"></a>在 Azure AD 中建立群組，並讓 VM 系統指派的受控識別成為此群組的成員

您可以使用現有的 Azure AD 群組，或使用 Azure AD PowerShell 建立一個新群組。  

首先，安裝 [Azure AD PowerShell](https://docs.microsoft.com/powershell/azure/active-directory/install-adv2) 模組。 然後使用 `Connect-AzureAD` 登入，執行下列命令來建立群組，並將它儲存在變數中：

```powershell
$Group = New-AzureADGroup -DisplayName "VM managed identity access to SQL" -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
```

輸出看起來像下面這樣，它也會檢查變數的值：

```powershell
$Group = New-AzureADGroup -DisplayName "VM managed identity access to SQL" -MailEnabled $false -SecurityEnabled $true -MailNickName "NotSet"
$Group
ObjectId                             DisplayName          Description
--------                             -----------          -----------
6de75f3c-8b2f-4bf4-b9f8-78cc60a18050 VM managed identity access to SQL
```

接下來，將 VM 系統指派的受控識別新增至群組。  您需要系統所指派受控識別的 **ObjectId**，這可透過 Azure PowerShell 取得。  首先，下載 [Azure Powershell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。 然後使用 `Connect-AzureRmAccount` 登入，執行下列命令來達成這些事項︰
- 確定工作階段內容設為所需的 Azure 訂用帳戶 (如果您有多個訂用帳戶)。
- 列出 Azure 訂用帳戶中的可用資源，確認資源群組和 VM 名稱正確。
- 在 `<RESOURCE-GROUP>` 和 `<VM-NAME>` 中使用適當的值，以取得 VM 系統指派的受控識別屬性。

```powershell
Set-AzureRMContext -subscription "bdc79274-6bb9-48a8-bfd8-00c140fxxxx"
Get-AzureRmResource
$VM = Get-AzureRmVm -ResourceGroup <RESOURCE-GROUP> -Name <VM-NAME>
```

輸出看起來會像下面這樣，其中也會檢查 VM 系統所指派受控識別的服務主體物件識別碼：
```powershell
$VM = Get-AzureRmVm -ResourceGroup DevTestGroup -Name DevTestWinVM
$VM.Identity.PrincipalId
b83305de-f496-49ca-9427-e77512f6cc64
```

現在，將 VM 系統指派的受控識別新增至群組。  使用 Azure AD PowerShell，可以只將一個服務主體新增至群組。  請執行這個命令：
```powershell
Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId $VM.Identity.PrincipalId
```

如果您在之後也檢查群組成員資格，輸出看起來會像這樣：

```powershell
Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId $VM.Identity.PrincipalId
Get-AzureAdGroupMember -ObjectId $Group.ObjectId

ObjectId                             AppId                                DisplayName
--------                             -----                                -----------
b83305de-f496-49ca-9427-e77512f6cc64 0b67a6d6-6090-4ab4-b423-d6edda8e5d9f DevTestWinVM
```

## <a name="enable-azure-ad-authentication-for-the-sql-server"></a>啟用 SQL Server 的 Azure AD 驗證

現在，您已建立群組並將 VM 系統指派的受控識別新增為成員，接著可以使用下列步驟[設定 SQL 伺服器的 Azure AD 驗證](/azure/sql-database/sql-database-aad-authentication-configure#provision-an-azure-active-directory-administrator-for-your-azure-sql-server)：

1.  在 Azure 入口網站中，選取左側導覽中的 [SQL 伺服器]。
2.  按一下要啟用 Azure AD 驗證的 SQL 伺服器。
3.  在刀鋒視窗的 [設定] 區段中，按一下 [Active Directory 系統管理員]。
4.  在命令列中，按一下 [設定系統管理員]。
5.  選取要設為伺服器系統管理員的 Azure AD 使用者帳戶，然後按一下 [選取]。
6.  在命令列中，按一下 [儲存]。

## <a name="create-a-contained-user-in-the-database-that-represents-the-azure-ad-group"></a>在代表 Azure AD 群組的資料庫中建立包含的使用者

在這一個步驟中，您需要 [Microsoft SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)。 在開始之前，先閱讀以下文章了解 Azure AD 整合的背景會很有幫助：

- [SQL Database 和 SQL 資料倉儲的通用驗證 (MFA 的 SSMS 支援)](/azure/sql-database/sql-database-ssms-mfa-authentication)
- [使用 SQL Database 或 SQL 資料倉儲設定和管理 Azure Active Directory 驗證](/azure/sql-database/sql-database-aad-authentication-configure)

1.  啟動 SQL Server Management Studio。
2.  在 [連線到伺服器] 對話方塊中，在 [伺服器名稱] 欄位中輸入您的 SQL 伺服器名稱。
3.  在 [驗證] 欄位中，選取 [具 MFA 支援的 Active Directory - 通用]。
4.  在 [使用者名稱] 欄位中，輸入您設為伺服器系統管理員的 Azure AD 帳戶名稱，例如 helen@woodgroveonline.com。
5.  按一下 [選項] 。
6.  在 [連 線至資料庫] 欄位中，輸入您想要設定的非系統資料庫的名稱。
7.  按一下 [ **連接**]。  完成登入程序。
8.  在 [物件總管] 中展開 [資料庫] 資料夾。
9.  在使用者資料庫上按一下滑鼠右鍵，然後按一下 [新增查詢]。
10.  在查詢視窗中，輸入下列這一行，然後按一下工具列中的 [執行]︰
    
     ```
     CREATE USER [VM managed identity access to SQL] FROM EXTERNAL PROVIDER
     ```
    
     命令應該會順利完成，為群組建立包含的使用者。
11.  清除查詢視窗，輸入下列這一行，然後按一下工具列中的 [執行]︰
     
     ```
     ALTER ROLE db_datareader ADD MEMBER [VM managed identity access to SQL]
     ```

     命令應該會順利完成，將讀取整個資料庫的能力授與包含的使用者。

在 VM 中執行的程式碼現在可以使用其系統指派的受控識別取得權杖，並使用此權杖向 SQL 伺服器進行驗證。

## <a name="get-an-access-token-using-the-vms-system-assigned-managed-identity-and-use-it-to-call-azure-sql"></a>使用 VM 系統指派的受控識別來取得存取權杖，以用來呼叫 Azure SQL 

Azure SQL 原生支援 Azure AD 驗證，因此可直接接受使用適用於 Azure 資源的受控識別所取得的存取權杖。  您使用 **access token** 方法來建立 SQL 連線。  這是 Azure AD 與 Azure SQL 整合的一部分，與在連接字串上提供認證不同。

以下是使用存取權杖開啟 SQL 連線的 .Net 程式碼範例。  此程式碼必須在 VM 上執行，才能夠存取 VM 系統指派的受控識別端點。  需使用 **.Net framework 4.6** 或更高版本，才可使用的 access token 方法。  將 AZURE-SQL-SERVERNAME 和 DATABASE 的值取代為實際值。  請注意，Azure SQL 資源識別碼是「 https://database.windows.net/」。

```csharp
using System.Net;
using System.IO;
using System.Data.SqlClient;
using System.Web.Script.Serialization;

//
// Get an access token for SQL.
//
HttpWebRequest request = (HttpWebRequest)WebRequest.Create("http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://database.windows.net/");
request.Headers["Metadata"] = "true";
request.Method = "GET";
string accessToken = null;

try
{
    // Call managed identities for Azure resources endpoint.
    HttpWebResponse response = (HttpWebResponse)request.GetResponse();

    // Pipe response Stream to a StreamReader and extract access token.
    StreamReader streamResponse = new StreamReader(response.GetResponseStream()); 
    string stringResponse = streamResponse.ReadToEnd();
    JavaScriptSerializer j = new JavaScriptSerializer();
    Dictionary<string, string> list = (Dictionary<string, string>) j.Deserialize(stringResponse, typeof(Dictionary<string, string>));
    accessToken = list["access_token"];
}
catch (Exception e)
{
    string errorText = String.Format("{0} \n\n{1}", e.Message, e.InnerException != null ? e.InnerException.Message : "Acquire token failed");
}

//
// Open a connection to the SQL server using the access token.
//
if (accessToken != null) {
    string connectionString = "Data Source=<AZURE-SQL-SERVERNAME>; Initial Catalog=<DATABASE>;";
    SqlConnection conn = new SqlConnection(connectionString);
    conn.AccessToken = accessToken;
    conn.Open();
}
```

或者，另一個測試端對端安裝程式，而不需要在 VM 上撰寫和部署應用程式的快速方法，是使用 PowerShell。

1.  在入口網站中，瀏覽至 [虛擬機器] 並移至您的 Windows 虛擬機器，在 [概觀] 中按一下 [連線]。 
2.  輸入您建立 Windows VM 時新增的**使用者名稱**和**密碼**。 
3.  現在您已經建立虛擬機器的**遠端桌面連線**，請在遠端工作階段中開啟 **PowerShell**。 
4.  使用 PowerShell 的 `Invoke-WebRequest`，向本機受控識別的端點提出要求，以取得 Azure SQL 的存取權杖。

    ```powershell
       $response = Invoke-WebRequest -Uri 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https%3A%2F%2Fdatabase.windows.net%2F' -Method GET -Headers @{Metadata="true"}
    ```
    
    將來自 JSON 物件的回應轉換為 PowerShell 物件。 
    
    ```powershell
    $content = $response.Content | ConvertFrom-Json
    ```

    擷取回應中的存取權杖。
    
    ```powershell
    $AccessToken = $content.access_token
    ```

5.  開啟 SQL伺服器連線。 記得取代 AZURE-SQL-SERVERNAME 和 DATABASE 的值。
    
    ```powershell
    $SqlConnection = New-Object System.Data.SqlClient.SqlConnection
    $SqlConnection.ConnectionString = "Data Source = <AZURE-SQL-SERVERNAME>; Initial Catalog = <DATABASE>"
    $SqlConnection.AccessToken = $AccessToken
    $SqlConnection.Open()
    ```

    接下來，建立查詢並傳送至伺服器。  記得取代 TABLE 的值。

    ```powershell
    $SqlCmd = New-Object System.Data.SqlClient.SqlCommand
    $SqlCmd.CommandText = "SELECT * from <TABLE>;"
    $SqlCmd.Connection = $SqlConnection
    $SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
    $SqlAdapter.SelectCommand = $SqlCmd
    $DataSet = New-Object System.Data.DataSet
    $SqlAdapter.Fill($DataSet)
    ```

檢查 `$DataSet.Tables[0]` 的值以檢視查詢的結果。

## <a name="next-steps"></a>後續步驟

在本教學課程中，您已了解如何使用系統指派的受控識別，來存取 Azure SQL 伺服器。  若要深入了解 Azure SQL Server，請參閱：

> [!div class="nextstepaction"]
>[Azure SQL Database 服務](/azure/sql-database/sql-database-technical-overview)
