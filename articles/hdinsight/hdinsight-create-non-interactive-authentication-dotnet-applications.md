---
title: 非互動式驗證 .NET 應用程式-Azure HDInsight
description: 了解如何在 Azure HDInsight 中建立非互動式驗證 Microsoft .NET 應用程式。
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive, devx-track-csharp
ms.date: 12/23/2019
ms.openlocfilehash: 121c5850ef47999f54d206b95b69e10775d3e5c9
ms.sourcegitcommit: 2f9f306fa5224595fa5f8ec6af498a0df4de08a8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/28/2021
ms.locfileid: "98946081"
---
# <a name="create-a-non-interactive-authentication-net-hdinsight-application"></a>建立非互動式驗證 .NET HDInsight 應用程式

在應用程式本身的身分識別下執行您的 Microsoft .NET Azure HDInsight 應用程式 (非互動式) ，或在應用程式的已登入使用者身分識別下， (互動式) 。 本文將說明如何建立非互動式驗證 .NET 應用程式，來連線到 Azure 及管理 HDInsight。 如需互動式應用程式的範例，請參閱[連線至 Azure HDInsight](hdinsight-administer-use-dotnet-sdk.md#connect-to-azure-hdinsight)。

從非互動式 .NET 應用程式，您需要︰

* 您的 Azure 訂用帳戶租用戶識別碼 (又稱為目錄識別碼)。 請參閱[取得租用戶識別碼](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in)。
* Azure Active Directory (Azure AD) 應用程式用戶端識別碼。 請參閱 [建立 Azure Active Directory 應用程式](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) 並 [取得應用程式識別碼](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in)。
* Azure AD 應用程式秘密金鑰。 請參閱[取得應用程式驗證金鑰](../active-directory/develop/howto-create-service-principal-portal.md#get-tenant-and-app-id-values-for-signing-in)。

## <a name="prerequisites"></a>先決條件

HDInsight 叢集。 請參閱[入門教學課程](hadoop/apache-hadoop-linux-tutorial-get-started.md)。

## <a name="assign-a-role-to-the-azure-ad-application"></a>將角色新增至 Azure AD 應用程式

為您的 Azure AD 應用程式指派[角色](../role-based-access-control/built-in-roles.md)，以授與執行動作的權限。 您可以針對訂用帳戶、資源群組或資源的層級設定範圍。 較低的範圍層級會繼承較高層級的權限。 例如，將應用程式新增至資源群組的「讀取者」角色，表示應用程式可以讀取資源群組以及其中的任何資源。 在本文中，您會在資源群組層級設定範圍。 如需詳細資訊，請參閱[使用角色指派來管理 Azure 訂用帳戶資源的存取權](../role-based-access-control/role-assignments-portal.md)。

**將擁有者角色新增至 Azure AD 應用程式**

1. 登入 [Azure 入口網站](https://portal.azure.com)。
1. 流覽至您稍後在本文中將用來執行 Hive 查詢的 HDInsight 叢集所在的資源群組。 如果您有大量的資源群組，您可以使用篩選來找出您想要的資源群組。
1. 在資源群組功能表中，選取 [存取控制 (IAM)]。
1. 選取 [角色指派] 索引標籤，以查看目前的角色指派。
1. 從頁面頂端選取 [+ 新增]。
1. 依照指示將擁有者角色新增至您的 Azure AD 應用程式。 成功新增角色之後，應用程式會列在 [擁有者] 角色底下。

## <a name="develop-an-hdinsight-client-application"></a>開發 HDInsight 用戶端應用程式

1. 建立 C# 主控台應用程式。
2. 新增下列 [NuGet](https://www.nuget.org/) 套件：

    * `Install-Package Microsoft.Azure.Common.Authentication -Pre`
    * `Install-Package Microsoft.Azure.Management.HDInsight -Pre`
    * `Install-Package Microsoft.Azure.Management.Resources -Pre`

3. 執行下列程式碼：

    ```csharp
    using System;
    using System.Security;
    using Microsoft.Azure;
    using Microsoft.Azure.Common.Authentication;
    using Microsoft.Azure.Common.Authentication.Factories;
    using Microsoft.Azure.Common.Authentication.Models;
    using Microsoft.Azure.Management.Resources;
    using Microsoft.Azure.Management.HDInsight;
    
    namespace CreateHDICluster
    {
        internal class Program
        {
            private static HDInsightManagementClient _hdiManagementClient;
    
            private static Guid SubscriptionId = new Guid("<Enter your Azure subscription ID>");
            private static string tenantID = "<Enter your tenant ID (also called directory ID)>";
            private static string applicationID = "<Enter your application ID>";
            private static string secretKey = "<Enter the application secret key>";
    
            private static void Main(string[] args)
            {
                var key = new SecureString();
                foreach (char c in secretKey) { key.AppendChar(c); }
    
                var tokenCreds = GetTokenCloudCredentials(tenantID, applicationID, key);
                var subCloudCredentials = GetSubscriptionCloudCredentials(tokenCreds, SubscriptionId);
    
                var resourceManagementClient = new ResourceManagementClient(subCloudCredentials);
                resourceManagementClient.Providers.Register("Microsoft.HDInsight");
    
                _hdiManagementClient = new HDInsightManagementClient(subCloudCredentials);
    
                var results = _hdiManagementClient.Clusters.List();
                foreach (var name in results.Clusters)
                {
                    Console.WriteLine("Cluster Name: " + name.Name);
                    Console.WriteLine("\t Cluster type: " + name.Properties.ClusterDefinition.ClusterType);
                    Console.WriteLine("\t Cluster location: " + name.Location);
                    Console.WriteLine("\t Cluster version: " + name.Properties.ClusterVersion);
                }
                Console.WriteLine("Press Enter to continue");
                Console.ReadLine();
            }
    
            /// Get the access token for a service principal and provided key.          
            public static TokenCloudCredentials GetTokenCloudCredentials(string tenantId, string clientId, SecureString secretKey)
            {
                var authFactory = new AuthenticationFactory();
                var account = new AzureAccount { Type = AzureAccount.AccountType.ServicePrincipal, Id = clientId };
                var env = AzureEnvironment.PublicEnvironments[EnvironmentName.AzureCloud];
                var accessToken =
                    authFactory.Authenticate(account, env, tenantId, secretKey, ShowDialog.Never).AccessToken;
    
                return new TokenCloudCredentials(accessToken);
            }
    
            public static SubscriptionCloudCredentials GetSubscriptionCloudCredentials(SubscriptionCloudCredentials creds, Guid subId)
            {
                return new TokenCloudCredentials(subId.ToString(), ((TokenCloudCredentials)creds).Token);
            }
        }
    }
    ```

## <a name="next-steps"></a>後續步驟

* [在入口網站中建立 Azure Active Directory 應用程式和服務主體](../active-directory/develop/howto-create-service-principal-portal.md)。
* 了解如何[使用 Azure Resource Manager 驗證服務主體](../active-directory/develop/howto-authenticate-service-principal-powershell.md)。
* 瞭解 [ (AZURE RBAC) 的 azure 角色型存取控制 ](../role-based-access-control/role-assignments-portal.md)。
