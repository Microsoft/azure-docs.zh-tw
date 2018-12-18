---
title: ActiveDirectoryInteractive 連線至 SQL | Microsoft Docs
description: C# 程式碼範例 (具有說明)，使用 SqlAuthenticationMethod.ActiveDirectoryInteractive 模式來連線到 Azure SQL Database。
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: active directory
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: MirekS
ms.reviewer: GeneMi
ms.date: 04/06/2018
manager: craigg
ms.openlocfilehash: 80944e73f21d75943d4fa71c7ac9500e47bab250
ms.sourcegitcommit: 715813af8cde40407bd3332dd922a918de46a91a
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "47055521"
---
# <a name="use-activedirectoryinteractive-mode-to-connect-to-azure-sql-database"></a>使用 ActiveDirectoryInteractive 模式以連線到 Azure SQL Database

本文提供可執行的 C# 程式碼範例，可以連線到您的 Microsoft Azure SQL Database。 C# 程式會使用互動式驗證模式，其支援 Azure AD 多重要素驗證 (MFA)。 例如，連線嘗試可能包含正傳送到您行動電話的驗證碼。

如需適用於 SQL 工具之 MFA 支援的詳細資訊，請參閱 [SQL Server Data Tools (SSDT) 中的 Azure Active Directory 支援](https://docs.microsoft.com/sql/ssdt/azure-active-directory)。




## <a name="sqlauthenticationmethod-activedirectoryinteractive-enum-value"></a>SqlAuthenticationMethod .ActiveDirectoryInteractive 列舉值

從 .NET Framework 4.7.2 版開始，列舉 [**SqlAuthenticationMethod**](https://docs.microsoft.com/dotnet/api/system.data.sqlclient.sqlauthenticationmethod) 有新的值 **.ActiveDirectoryInteractive**。 用戶端 C# 程式使用這個列舉值時，該值會將系統導向為使用 Azure AD 互動模式，該模式支援使用 MFA 對 Azure SQL Database 進行驗證。 執行程式的使用者隨後會看到下列對話方塊：

1. 對話方塊會顯示 Azure AD 使用者名稱，並且要求 Azure AD 使用者的密碼。
    - 如果不需要密碼，就不會顯示此對話方塊。 如果使用者的網域已與 Azure AD 同盟，就不需要任何密碼。

    如果在 Azure AD 中設定的原則會對該使用者強制執行 MFA ，則接下來會顯示下列對話方塊。

2. 只有在使用者第一次體驗 MFA 案例時，系統才會顯示額外對話方塊。 對話方塊會詢問要將簡訊傳送到哪個行動電話號碼。 每個訊息會提供*驗證碼*，使用者必須在下一個對話方塊中輸入該驗證碼。

3. 另一個對話方塊會詢問 MFA 驗證碼，系統已將該驗證碼傳送到行動電話。

如需如何設定 Azure AD 以要求 MFA 的詳細資訊，請參閱[開始在雲端中使用 Azure Multi-Factor Authentication](https://docs.microsoft.com/azure/multi-factor-authentication/multi-factor-authentication-get-started-cloud)。

如需這些對話方塊的螢幕擷取畫面，請參閱[設定適用於 SQL Server Management Studio 和 Azure AD 的多重要素驗證](sql-database-ssms-mfa-authentication-configure.md)。

> [!TIP]
> 我們所有類型 .NET Framework API 的一般搜尋頁面，可以在以下的便利 **.NET API 瀏覽器** 工具的連結中取得：
>
> [https://docs.microsoft.com/dotnet/api/](https://docs.microsoft.com/dotnet/api/)
>
> 藉由將類型名稱新增至選擇性附加 **?term=** 參數，搜尋頁面會準備好結果，等待我們取用：
>
> [https://docs.microsoft.com/dotnet/api/?term=SqlAuthenticationMethod](https://docs.microsoft.com/dotnet/api/?term=SqlAuthenticationMethod)


## <a name="preparations-for-c-by-using-the-azure-portal"></a>使用 Azure 入口網站準備 C#

我們假設您已經[建立 Azure SQL Database 伺服器](sql-database-get-started-portal.md)而且已可供使用。


### <a name="a-create-an-app-registration"></a>A. 建立應用程式註冊

若要使用 Azure AD 驗證，您的 C# 用戶端程式必須提供 GUID，作為當您的程式嘗試連線時的 clientId。 完成應用程式註冊時，會在 Azure 入口網站中產生及顯示 GUID，並標示為**應用程式識別碼**。 瀏覽步驟如下所示：

1. Azure 入口網站 &gt; **Azure Active Directory** &gt; **應用程式註冊**

    ![應用程式註冊](media\active-directory-interactive-connect-azure-sql-db\sshot-create-app-registration-b20.png)

2. 系統會產生及顯示**應用程式識別碼**值。

    ![顯示的應用程式識別碼](media\active-directory-interactive-connect-azure-sql-db\sshot-application-id-app-regis-mk49.png)

3. **已註冊的應用程式** &gt; **設定** &gt; **必要權限** &gt; **新增**

    ![已註冊應用程式的權限設定](media\active-directory-interactive-connect-azure-sql-db\sshot-registered-app-settings-required-permissions-add-api-access-c32.png)

4. **必要權限** &gt; **新增 API 存取權** &gt; **選取 API** &gt; **Azure SQL Database**

    ![將存取權新增至 Azure SQL Database 的 API](media\active-directory-interactive-connect-azure-sql-db\sshot-registered-app-settings-required-permissions-add-api-access-Azure-sql-db-d11.png)

5. **API 存取權** &gt; **選取權限** &gt; **委派的權限**

    ![將權限委派給 Azure SQL Database 的 API](media\active-directory-interactive-connect-azure-sql-db\sshot-add-api-access-azure-sql-db-delegated-permissions-checkbox-e14.png)


### <a name="b-set-azure-ad-admin-on-your-sql-database-server"></a>B. 在您的 SQL Database 伺服器上設定 Azure AD 管理員

每個 Azure SQL Database 伺服器都有它自己的 Azure AD SQL 邏輯伺服器。 針對我們的 C# 案例，您必須為您的 Azure SQL 伺服器設定 Azure AD 系統管理員。

1. **SQL Server** &gt; **Active Directory 管理員** &gt; **設定管理員**

    - 如需 Azure SQL Database 的 Azure AD 管理員和使用者的詳細資訊，請參閱[設定及管理對 SQL Database 的 Azure Active Directory 驗證](sql-database-aad-authentication-configure.md#provision-an-azure-active-directory-administrator-for-your-azure-sql-database-server)，在**為您的 Azure SQL Database 伺服器佈建 Azure Active Directory 系統管理員**一節中的螢幕擷取畫面。


### <a name="c-prepare-an-azure-ad-user-to-connect-to-a-specific-database"></a>C. 準備 Azure AD 使用者以連線到特定資料庫

在 Azure SQL Database 伺服器特定的 Azure AD 中，您可以新增使用者，該使用者具有特定資料庫的存取權。

如需詳細資訊，請參閱[使用 Azure Active Directory 驗證向 SQL Database、受控執行個體或 SQL 資料倉儲進行驗證](sql-database-aad-authentication.md)。


### <a name="d-add-a-non-admin-user-to-azure-ad"></a>D. 將非管理員使用者新增至 Azure AD

SQL Database 伺服器的 Azure AD 管理員可以用來連線到您的 SQL Database 伺服器。 不過，更常見的情形是將非管理員使用者新增至 Azure AD。 當使用非管理員來連線時，如果 Azure AD 對此使用者強制執行 MFA，則會叫用 MFA 序列。




## <a name="azure-active-directory-authentication-library-adal"></a>Azure Active Directory Authentication Library (ADAL)

C# 程式會依賴命名空間 **Microsoft.IdentityModel.Clients.ActiveDirectory**。 此命名空間的類別是在相同名稱的組件中。

- 使用 NuGet 以下載及安裝 ADAL 組件。
    - [https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)

- 將參考新增至組件，以支援 C# 程式的編譯。




## <a name="sqlauthenticationmethod-enum"></a>SqlAuthenticationMethod 列舉

C# 範例所依賴的一個命名空間是 **System.Data.SqlClient**。 特別引人興趣的是列舉 **SqlAuthenticationMethod**。 這個列舉具有下列值：

- **SqlAuthenticationMethod.ActiveDirectory *互動式***：&nbsp;  與 Azure AD 使用者名稱搭配使用，以達成多重要素驗證 MFA。
    - 這個值是目前文章的焦點。 它會透過顯示對話方塊來要求輸入使用者密碼，接著若強制對此使用者執行 MFA，則會要求進行 MFA 驗證，來產生互動式體驗。
    - 此值從 .NET Framework 4.7.2 版開始可供使用。

- **SqlAuthenticationMethod.ActiveDirectory *整合式***：&nbsp;  針對同盟帳戶使用此設定。 針對同盟帳戶，Windows 網域已知使用者名稱。 這個方法不支援 MFA。

- **SqlAuthenticationMethod.ActiveDirectory *密碼***：&nbsp;  針對需要 Azure AD 使用者和使用者密碼的驗證使用此設定。 Azure SQL Database 會執行驗證。 這個方法不支援 MFA。




## <a name="prepare-c-parameter-values-from-the-azure-portal"></a>從 Azure 入口網站準備 C# 參數值

為了成功執行 C# 程式，您必須將適當的值指派給下列靜態欄位。 這些靜態欄位就像是程式中的參數。 欄位會顯示在這裡，具有假設值。 同時會顯示您可以在 Azure 入口網站中取得適當值的位置：


| 靜態欄位名稱 | 假設值 | 在 Azure 入口網站中的位置 |
| :---------------- | :------------ | :-------------------- |
| Az_SQLDB_svrName | "my-favorite-sqldb-svr.database.windows.net" | **SQL Server** &gt; **依名稱篩選** |
| AzureAD_UserID | "user9@abc.onmicrosoft.com" | **Azure Active Directory** &gt;  &gt; **新的來賓使用者** |
| Initial_DatabaseName | "master" | **SQL Server** &gt; **SQL Database** |
| ClientApplicationID | "a94f9c62-97fe-4d19-b06d-111111111111" | **Azure Active Directory** &gt; **應用程式註冊**<br /> &nbsp; &nbsp; &gt; **依名稱搜尋** &gt; **應用程式識別碼** |
| RedirectUri | new Uri( "https://bing.com/") | **Azure Active Directory** &gt; **應用程式註冊**<br /> &nbsp; &nbsp; &gt; **依名稱搜尋** &gt; *[Your-App-regis]* &gt;<br /> &nbsp; &nbsp; **設定** &gt; **RedirectURIs**<br /><br />針對這篇文章，任何有效值都適用於 RedirectUri。 該值實際上不會在我們的案例中使用。 |
| &nbsp; | &nbsp; | &nbsp; |


根據您的特定案例，您也許不需要上述表格中的所有參數值。




## <a name="run-ssms-to-verify"></a>執行 SSMS 以驗證

在執行 C# 程式前執行 SQL Server Management Studio (SSMS) 很有幫助。 SSMS 執行會驗證各種組態是否正確。 然後，可以將 C# 程式的任何錯誤範圍縮減為僅其原始程式碼。


#### <a name="verify-sql-database-firewall-ip-addresses"></a>驗證 SQL Database 防火牆 IP 位址

在您稍後執行 C# 程式的相同組建中，從相同的電腦執行 SSMS。 您可以使用您覺得最簡單的**驗證**模式。 如果有任何指示表示資料庫伺服器防火牆不接受您的 IP 位址，您可加以修正，如同 [Azure SQL Database 伺服器層級和資料庫層級防火牆規則](sql-database-firewall-configure.md)中所示。


#### <a name="verify-multi-factor-authentication-mfa-for-azure-ad"></a>驗證 Azure AD 的多重要素驗證 (MFA)

再次執行 SSMS，但是這次將**驗證**設為 **Active Directory - 通用 (具有 MFA 支援)**。 針對此選項，您必須具備 SSMS 17.5 版或更新版本。

如需詳細資訊，請參閱[設定 SSMS 和 Azure AD 的多重要素驗證](sql-database-ssms-mfa-authentication-configure.md)。




## <a name="c-code-example"></a>C# 程式碼範例

若要編譯這個 C# 範例，您必須將參考新增至名為 **Microsoft.IdentityModel.Clients.ActiveDirectory** 的 DLL 組件。


#### <a name="reference-documentation"></a>參考文件

- **System.Data.SqlClient** 命名空間：
    - 搜尋：&nbsp; [https://docs.microsoft.com/dotnet/api/?term=System.Data.SqlClient](https://docs.microsoft.com/dotnet/api/?term=System.Data.SqlClient)
    - 直接：&nbsp; [System.Data.Client](https://docs.microsoft.com/dotnet/api/system.data.sqlclient)

- **Microsoft.IdentityModel.Clients.ActiveDirectory** 命名空間：
    - 搜尋：&nbsp; [https://docs.microsoft.com/dotnet/api/?term=Microsoft.IdentityModel.Clients.ActiveDirectory](https://docs.microsoft.com/dotnet/api/?term=Microsoft.IdentityModel.Clients.ActiveDirectory)
    - 直接：&nbsp; [Microsoft.IdentityModel.Clients.ActiveDirectory](https://docs.microsoft.com/dotnet/api/microsoft.identitymodel.clients.activedirectory)


#### <a name="c-source-code-in-two-parts"></a>C# 原始程式碼，分成兩個部分

&nbsp;

```csharp

using System;    // C# ,  part 1 of 2.

// Add a reference to assembly:  Microsoft.IdentityModel.Clients.ActiveDirectory.DLL
using Microsoft.IdentityModel.Clients.ActiveDirectory;

using DA = System.Data;
using SC = System.Data.SqlClient;
using AD = Microsoft.IdentityModel.Clients.ActiveDirectory;
using TX = System.Text;
using TT = System.Threading.Tasks;

namespace ADInteractive5
{
    class Program
    {
        // ASSIGN YOUR VALUES TO THESE STATIC FIELDS !!
        static public string Az_SQLDB_svrName = "<YOUR VALUE HERE>";
        static public string AzureAD_UserID = "<YOUR VALUE HERE>";
        static public string Initial_DatabaseName = "master";
        // Some scenarios do not need values for the following two fields:
        static public readonly string ClientApplicationID = "<YOUR VALUE HERE>";
        static public readonly Uri RedirectUri = new Uri("<YOUR VALUE HERE>");

        public static void Main(string[] args)
        {
            var provider = new ActiveDirectoryAuthProvider();

            SC.SqlAuthenticationProvider.SetProvider(
                SC.SqlAuthenticationMethod.ActiveDirectoryInteractive,
                //SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated,  // Alternatives.
                //SC.SqlAuthenticationMethod.ActiveDirectoryPassword,
                provider);

            Program.Connection();
        }

        public static void Connection()
        {
            SC.SqlConnectionStringBuilder builder = new SC.SqlConnectionStringBuilder();

            // Program._  static values that you set earlier.
            builder["Data Source"] = Program.Az_SQLDB_svrName;
            builder.UserID = Program.AzureAD_UserID;
            builder["Initial Catalog"] = Program.Initial_DatabaseName;

            // This "Password" is not used with .ActiveDirectoryInteractive.
            //builder["Password"] = "<YOUR PASSWORD HERE>";

            builder["Connect Timeout"] = 15;
            builder["TrustServerCertificate"] = true;
            builder.Pooling = false;

            // Assigned enum value must match the enum given to .SetProvider().
            builder.Authentication = SC.SqlAuthenticationMethod.ActiveDirectoryInteractive;
            SC.SqlConnection sqlConnection = new SC.SqlConnection(builder.ConnectionString);

            SC.SqlCommand cmd = new SC.SqlCommand(
                "SELECT '******** MY QUERY RAN SUCCESSFULLY!! ********';",
                sqlConnection);

            try
            {
                sqlConnection.Open();
                if (sqlConnection.State == DA.ConnectionState.Open)
                {
                    var rdr = cmd.ExecuteReader();
                    var msg = new TX.StringBuilder();
                    while (rdr.Read())
                    {
                        msg.AppendLine(rdr.GetString(0));
                    }
                    Console.WriteLine(msg.ToString());
                    Console.WriteLine(":Success");
                }
                else
                {
                    Console.WriteLine(":Failed");
                }
                sqlConnection.Close();
            }
            catch (Exception ex)
            {
                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine("Connection failed with the following exception...");
                Console.WriteLine(ex.ToString());
                Console.ResetColor();
            }
        }
    } // EOClass Program .

```

&nbsp;

#### <a name="second-half-of-the-c-program"></a>第二個部分的 C# 程式

為了更佳的視覺顯示，C# 程式會分割成兩個程式碼區塊。 若要執行程式，請將兩個程式碼區塊貼在一起。

&nbsp;

```csharp

    // C# ,  part 2 of 2 ,  to concatenate below part 1.

    /// <summary>
    /// SqlAuthenticationProvider - Is a public class that defines 3 different Azure AD
    /// authentication methods.  The methods are supported in the new .NET 4.7.2 .
    ///  . 
    /// 1. Interactive,  2. Integrated,  3. Password
    ///  . 
    /// All 3 authentication methods are based on the Azure
    /// Active Directory Authentication Library (ADAL) managed library.
    /// </summary>
    public class ActiveDirectoryAuthProvider : SC.SqlAuthenticationProvider
    {
        // Program._ more static values that you set!
        private readonly string _clientId = Program.ClientApplicationID;
        private readonly Uri _redirectUri = Program.RedirectUri;

        public override async TT.Task<SC.SqlAuthenticationToken>
            AcquireTokenAsync(SC.SqlAuthenticationParameters parameters)
        {
            AD.AuthenticationContext authContext =
                new AD.AuthenticationContext(parameters.Authority);
            authContext.CorrelationId = parameters.ConnectionId;
            AD.AuthenticationResult result;

            switch (parameters.AuthenticationMethod)
            {
                case SC.SqlAuthenticationMethod.ActiveDirectoryInteractive:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,  // "https://database.windows.net/"
                        _clientId,
                        _redirectUri,
                        new AD.PlatformParameters(AD.PromptBehavior.Auto),
                        new AD.UserIdentifier(
                            parameters.UserId,
                            AD.UserIdentifierType.RequiredDisplayableId));
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_1 == '.ActiveDirectoryIntegrated'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserCredential());
                    break;

                case SC.SqlAuthenticationMethod.ActiveDirectoryPassword:
                    Console.WriteLine("In method 'AcquireTokenAsync', case_2 == '.ActiveDirectoryPassword'.");

                    result = await authContext.AcquireTokenAsync(
                        parameters.Resource,
                        _clientId,
                        new AD.UserPasswordCredential(
                            parameters.UserId,
                            parameters.Password));
                    break;

                default: throw new InvalidOperationException();
            }
            return new SC.SqlAuthenticationToken(result.AccessToken, result.ExpiresOn);
        }

        public override bool IsSupported(SC.SqlAuthenticationMethod authenticationMethod)
        {
            return authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryIntegrated
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryInteractive
                || authenticationMethod == SC.SqlAuthenticationMethod.ActiveDirectoryPassword;
        }
    } // EOClass ActiveDirectoryAuthProvider .
} // EONamespace.  End of entire program source code.

```

&nbsp;

#### <a name="actual-test-output-from-c"></a>C# 的實際測試輸出

```
[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>> ADInteractive5.exe
In method 'AcquireTokenAsync', case_0 == '.ActiveDirectoryInteractive'.
******** MY QUERY RAN SUCCESSFULLY!! ********

:Success

[C:\Test\VSProj\ADInteractive5\ADInteractive5\bin\Debug\]
>>
```

&nbsp;


## <a name="next-steps"></a>後續步驟

- [Get-AzureRmSqlServerActiveDirectoryAdministrator](https://docs.microsoft.com/powershell/module/azurerm.sql/get-azurermsqlserveractivedirectoryadministrator)

