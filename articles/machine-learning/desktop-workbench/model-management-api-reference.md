---
title: 在 Azure Machine Learning 中建立模型管理的 Docker 映像 | Microsoft Docs
description: 本文章說明為您的 Web 服務建立 Docker 映像的步驟。
services: machine-learning
author: chhavib
ms.author: chhavib
manager: hjerez
editor: jasonwhowell
ms.reviewer: jasonwhowell, mldocs
ms.service: machine-learning
ms.component: core
ms.workload: data-services
ms.devlang: na
ms.topic: article
ms.date: 09/20/2017
ROBOTS: NOINDEX
ms.openlocfilehash: 07f8a31244acca48a0bbae8ad26b0c5b509f0c0a
ms.sourcegitcommit: 32d218f5bd74f1cd106f4248115985df631d0a8c
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/24/2018
ms.locfileid: "46959575"
---
# <a name="azure-machine-learning-model-management-account-api-reference"></a>Azure Machine Learning 模型管理帳戶 API 參考

[!INCLUDE [workbench-deprecated](../../../includes/aml-deprecating-preview-2017.md)] 

如需設定部署環境的詳細資訊，請參閱[模型管理帳戶設定](deployment-setup-configuration.md)。

Azure Machine Learning 模型管理帳戶 API 可實作下列作業：

- 註冊模型
- 建立資訊清單
- 建立 Docker 映像
- 建立 Web 服務

您可以使用此映像，在本機或在遠端 Azure Container Service 叢集或您選擇的另一個 Docker 支援的環境中建立 Web 服務。

## <a name="prerequisites"></a>必要條件
確定您已完成[安裝和建立快速入門](quickstart-installation.md)文件中的安裝步驟。

您必須先完成下列，才能繼續：
1. 佈建模型管理帳戶
2. 建立用來部署及管理模型的環境
3. Machine Learning 模型

### <a name="azure-ad-token"></a>Azure AD 權杖
使用 Azure CLI 時，請使用 `az login` 登入。 CLI 會使用 .azure 檔案中您的 Azure Active Directory (Azure AD) 權杖。 如果您希望使用 API，您有下列選項。

##### <a name="acquire-the-azure-ad-token-manually"></a>手動取得 Azure AD 權杖
您可以使用 `az login` 並從您的主目錄中的 .azure 檔案取得最新權杖。

##### <a name="acquire-the-azure-ad-token-programmatically"></a>以程式設計方式取得 Azure AD 權杖
```
az ad sp create-for-rbac --scopes /subscriptions/<SubscriptionId>/resourcegroups/<ResourceGroupName> --role Contributor --years <length of time> --name <MyservicePrincipalContributor>
```
建立服務主體後，請儲存輸出。 您現在可以使用該資訊，從 Azure AD 取得權杖：

```cs
 private static async Task<string> AcquireTokenAsync(string clientId, string password, string authority, string resource)
{
        var creds = new ClientCredential(clientId, password);
        var context = new AuthenticationContext(authority);
        var token = await context.AcquireTokenAsync(resource, creds).ConfigureAwait(false);
        return token.AccessToken;
}
```
權杖會置入授權標頭中以便 API 呼叫。


## <a name="register-a-model"></a>註冊模型

模型註冊步驟會向您建立的 Azure 模型管理帳戶註冊您的 Machine Learning 模型。 此登錄作業能夠追蹤模型及其在註冊期間指派的版本。 使用者會提供模型的名稱。 後續使用相同名稱註冊模型，會產生新的版本和識別碼。

### <a name="request"></a>要求

| 方法 | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models 
 |
### <a name="description"></a>說明
註冊模型。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| model | body | 用來註冊模型的承載。 | 是 | [模型](#model) |


### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 正常。 模型註冊成功。 | [模型](#model) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-models-in-an-account"></a>查詢帳戶中的模型清單
### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models 
 |
### <a name="description"></a>說明
查詢帳戶中的模型清單。 您可以依標籤和名稱來篩選結果清單。 如果未通過任何篩選條件，則查詢會列出帳戶中的所有模型。 傳回的清單會編頁碼，且每頁中的項目計數為選擇性參數。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| name | query | 物件名稱。 | 否 | 字串 |
| tag | query | 模型標籤。 | 否 | 字串 |
| count | query | 要在一個頁面中擷取的項目數目。 | 否 | 字串 |
| $skipToken | query | 要擷取下一頁的接續權杖。 | 否 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedModelList](#paginatedmodellist) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="get-model-details"></a>取得模型詳細資料
### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/models/{id}  
 |

### <a name="description"></a>說明
依識別碼取得模型。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 物件識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [模型](#model) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="register-a-manifest-with-the-registered-model-and-all-dependencies"></a>向已註冊的模型和所有相依性註冊資訊清單

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests | 

### <a name="description"></a>說明
向已註冊的模型及其所有相依性註冊資訊清單。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| manifestRequest | body | 用來註冊資訊清單的承載。 | 是 | [資訊清單](#manifest) |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 資訊清單註冊成功。 | [資訊清單](#manifest) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-manifests-in-an-account"></a>查詢帳戶中資訊清單的清單

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests | 

### <a name="description"></a>說明
查詢帳戶中資訊清單的清單。 您可以依模型識別碼和資訊清單名稱來篩選結果清單。 如果未通過任何篩選條件，則查詢會列出帳戶中的所有資訊清單。 傳回的清單會編頁碼，且每頁中的項目計數為選擇性參數。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| modelId | query | 模型識別碼。 | 否 | 字串 |
| manifestName | query | 資訊清單名稱。 | 否 | 字串 |
| count | query | 要在一個頁面中擷取的項目數目。 | 否 | 字串 |
| $skipToken | query | 要擷取下一頁的接續權杖。 | 否 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedManifestList](#paginatedmanifestlist) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="get-manifest-details"></a>取得資訊清單詳細資料

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/manifests/{id} | 

### <a name="description"></a>說明
依識別碼取得資訊清單。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 物件識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [資訊清單](#manifest) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="create-an-image"></a>建立映像

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images | 

### <a name="description"></a>說明
在 Azure Container Registry 中建立作為 Docker 映像的映像。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| imageRequest | body | 用來建立映像的承載。 | 是 | [ImageRequest](#imagerequest) |

### <a name="responses"></a>回應
| 代碼 | 說明 | headers | 結構描述 |
|--------------------|--------------------|--------------------|--------------------|
| 202 | 非同步作業位置 URL。 GET 呼叫會顯示映像建立工作的狀態。 | Operation-Location |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-images-in-an-account"></a>查詢帳戶中的映像清單

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images | 

### <a name="description"></a>說明
查詢帳戶中的映像清單。 您可以依資訊清單識別碼和名稱來篩選結果清單。 如果未通過任何篩選條件，則查詢會列出帳戶中的所有映像。 傳回的清單會編頁碼，且每頁中的項目計數為選擇性參數。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| manifestId | query | 資訊清單識別碼。 | 否 | 字串 |
| manifestName | query | 資訊清單名稱。 | 否 | 字串 |
| count | query | 要在一個頁面中擷取的項目數目。 | 否 | 字串 |
| $skipToken | query | 要擷取下一頁的接續權杖。 | 否 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedImageList](#paginatedimagelist) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="get-image-details"></a>取得映像詳細資料

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/images/{id} | 

### <a name="description"></a>說明
依識別碼取得映像。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 映像識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [映像](#image) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |


## <a name="create-a-service"></a>建立服務

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services | 

### <a name="description"></a>說明
從映像建立服務。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| serviceRequest | body | 用來建立服務的承載。 | 是 | [ServiceCreateRequest](#servicecreaterequest) |

### <a name="responses"></a>回應
| 代碼 | 說明 | headers | 結構描述 |
|--------------------|--------------------|--------------------|--------------------|
| 202 | 非同步作業位置 URL。 GET 呼叫會顯示服務建立工作的狀態。 | Operation-Location |
| 409 | 已經有指定之名稱的服務。 |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="query-the-list-of-services-in-an-account"></a>查詢帳戶中的服務清單

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services | 

### <a name="description"></a>說明
查詢帳戶中的服務清單。 您可以使用模型名稱/識別碼、資訊清單名稱/識別碼、映像識別碼、服務名稱或 Machine Learning 計算資源識別碼來篩選結果清單。 如果未通過任何篩選條件，則查詢會列出帳戶中的所有服務。 傳回的清單會編頁碼，且每頁中的項目計數為選擇性參數。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| serviceName | query | 服務名稱。 | 否 | 字串 |
| modelId | query | 模型名稱。 | 否 | 字串 |
| modelName | query | 模型識別碼。 | 否 | 字串 |
| manifestId | query | 資訊清單識別碼。 | 否 | 字串 |
| manifestName | query | 資訊清單名稱。 | 否 | 字串 |
| imageId | query | 映像識別碼。 | 否 | 字串 |
| computeResourceId | query | Machine Learning 計算資源識別碼。 | 否 | 字串 |
| count | query | 要在一個頁面中擷取的項目數目。 | 否 | 字串 |
| $skipToken | query | 要擷取下一頁的接續權杖。 | 否 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [PaginatedServiceList](#paginatedservicelist) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse) |

## <a name="get-service-details"></a>取得服務詳細資料

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id} | 

### <a name="description"></a>說明
依識別碼取得服務。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 物件識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [ServiceResponse](#serviceresponse) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)

## <a name="update-a-service"></a>更新服務

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| PUT |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id} | 

### <a name="description"></a>說明
更新現有服務。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 物件識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| serviceUpdateRequest | body | 用來更新現有服務的承載。 | 是 |  [ServiceUpdateRequest](#serviceupdaterequest) |

### <a name="responses"></a>回應
| 代碼 | 說明 | headers | 結構描述 |
|--------------------|--------------------|--------------------|--------------------|
| 202 | 非同步作業位置 URL。 GET 呼叫會顯示更新服務工作的狀態。 | Operation-Location |
| 404 | 具有指定識別碼的服務不存在。 |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)

## <a name="delete-a-service"></a>刪除服務

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| 刪除 |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id} | 

### <a name="description"></a>說明
刪除服務。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 物件識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 |  |
| 204 | 具有指定識別碼的服務不存在。 |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)

## <a name="get-service-keys"></a>取得服務金鑰

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id}/keys | 

### <a name="description"></a>說明
取得服務金鑰。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 服務識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [AuthKeys](#authkeys)
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)

## <a name="regenerate-service-keys"></a>重新產生服務金鑰

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| POST |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/services/{id}/regenerateKeys | 

### <a name="description"></a>說明
產生服務金鑰並將它們傳回。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 服務識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| regenerateKeyRequest | body | 用來更新現有服務的承載。 | 是 | [ServiceRegenerateKeyRequest](#serviceregeneratekeyrequest) |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [AuthKeys](#authkeys)
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)

## <a name="query-the-list-of-deployments-in-an-account"></a>查詢帳戶中的部署清單

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/deployments | 

### <a name="description"></a>說明
查詢帳戶中的部署清單。 您可以使用服務識別碼來篩選結果清單，這只會傳回針對特定服務所建立的部署。 如果未通過任何篩選條件，則查詢會列出帳戶中的所有部署。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |
| serviceId | query | 服務識別碼。 | 否 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [DeploymentList](#deploymentlist) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)

## <a name="get-deployment-details"></a>取得部署詳細資料

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/deployments/{id} | 

### <a name="description"></a>說明
依識別碼取得部署。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 部署識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [部署](#deployment) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)

## <a name="get-operation-details"></a>取得作業詳細資料

### <a name="request"></a>要求
| 方法 | 要求 URI |
|------------|------------|
| GET |  /api/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/accounts/{accountName}/operations/{id} | 

### <a name="description"></a>說明
依作業識別碼取得非同步作業狀態。

### <a name="parameters"></a>參數
| 名稱 | 位於 | 說明 | 必要 | 結構描述
|--------------------|--------------------|--------------------|--------------------|--------------------|
| subscriptionId | path | Azure 訂用帳戶識別碼。 | 是 | 字串 |
| resourceGroupName | path | 模型管理帳戶所在的資源群組名稱。 | 是 | 字串 |
| accountName | path | 模型管理帳戶的名稱。 | 是 | 字串 |
| id | path | 作業識別碼。 | 是 | 字串 |
| api-version | query | 要使用的 Microsoft.Machine.Learning 資源提供者 API 版本。 | 是 | 字串 |
| Authorization | 頁首 | 授權權杖。 它應該類似 "Bearer XXXXXX"。 | 是 | 字串 |

### <a name="responses"></a>回應
| 代碼 | 說明 | 結構描述 |
|--------------------|--------------------|--------------------|
| 200 | 成功。 | [OperationStatus](#asyncoperationstatus) |
| 預設值 | 說明作業為何失敗的錯誤回應。 | [ErrorResponse](#errorresponse)



<a name="definitions"></a>
## <a name="definitions"></a>定義

<a name="asset"></a>
### <a name="asset"></a>資產
在 Docker 映像建立期間所需的資產物件。


|名稱|說明|結構描述|
|---|---|---|
|**id**  <br>*選用*|資產識別碼。|字串|
|**mimeType**  <br>*選用*|模型內容的 MIME 類型。 如需 MIME 類型的詳細資訊，請參閱 [IANA 媒體類型的清單](https://www.iana.org/assignments/media-types/media-types.xhtml)。|字串|
|**unpack**  <br>*選用*|指出我們是否需要在 Docker 映像建立期間將內容解壓縮。|布林值|
|**url**  <br>*選用*|資產位置 URL。|字串|


<a name="asyncoperationstate"></a>
### <a name="asyncoperationstate"></a>AsyncOperationState
非同步作業狀態。

*類型*：enum (NotStarted、Running、Cancelled、Succeeded、Failed)


<a name="asyncoperationstatus"></a>
### <a name="asyncoperationstatus"></a>AsyncOperationStatus
作業狀態。


|名稱|說明|結構描述|
|---|---|---|
|**createdTime**  <br>*選用*  <br>*唯讀*|非同步作業建立時間 (UTC)。|字串 (日期-時間)|
|**endTime**  <br>*選用*  <br>*唯讀*|非同步作業結束時間 (UTC)。|字串 (日期-時間)|
|**error**  <br>*選用*||[ErrorResponse](#errorresponse)|
|**id**  <br>*選用*|非同步作業識別碼。|字串|
|**operationType**  <br>*選用*|非同步作業類型。|enum (Image, Service)|
|**resourceLocation**  <br>*選用*|非同步作業所建立或更新的資源。|字串|
|**state**  <br>*選用*||[AsyncOperationState](#asyncoperationstate)|


<a name="authkeys"></a>
### <a name="authkeys"></a>AuthKeys
服務的驗證金鑰。


|名稱|說明|結構描述|
|---|---|---|
|**primaryKey**  <br>*選用*|主索引鍵。|字串|
|**secondaryKey**  <br>*選用*|次要索引鍵。|字串|


<a name="autoscaler"></a>
### <a name="autoscaler"></a>AutoScaler
Autoscaler 的設定。


|名稱|說明|結構描述|
|---|---|---|
|**autoscaleEnabled**  <br>*選用*|啟用或停用 Auto Scaler。|布林值|
|**maxReplicas**  <br>*選用*|要相應增加至的 Pod 複本數目上限。  <br>**最小值**：`1`|integer|
|**minReplicas**  <br>*選用*|要相應減少至的 Pod 複本數目下限。  <br>**最小值**：`0`|integer|
|**refreshPeriodInSeconds**  <br>*選用*|重新整理自動調整觸發程序的時間。  <br>**最小值**：`1`|integer|
|**targetUtilization**  <br>*選用*|觸發自動調整的使用率百分比。  <br>**最小值**：`0`  <br>**最大值**：`100`|integer|


<a name="computeresource"></a>
### <a name="computeresource"></a>ComputeResource
Machine Learning 計算資源。


|名稱|說明|結構描述|
|---|---|---|
|**id**  <br>*選用*|資源識別碼。|字串|
|**type**  <br>*選用*|資源類型。|enum (Cluster)|


<a name="containerresourcereservation"></a>
### <a name="containerresourcereservation"></a>ContainerResourceReservation
要為叢集中的容器保留資源的組態。


|名稱|說明|結構描述|
|---|---|---|
|**cpu**  <br>*選用*|指定 CPU 保留。 Kubernetes 的格式：請參閱 [CPU 的意義](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-cpu)。|字串|
|**memory**  <br>*選用*|指定記憶體保留。 Kubernetes 的格式：請參閱[記憶體的意義](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-memory)。|字串|


<a name="deployment"></a>
### <a name="deployment"></a>部署
Azure Machine Learning 部署的執行個體。


|名稱|說明|結構描述|
|---|---|---|
|**createdAt**  <br>*選用*  <br>*唯讀*|部署建立時間 (UTC)。|字串 (日期-時間)|
|**expiredAt**  <br>*選用*  <br>*唯讀*|部署到期時間 (UTC)。|字串 (日期-時間)|
|**id**  <br>*選用*|部署識別碼。|字串|
|**imageId**  <br>*選用*|與此部署相關聯的映像識別碼。|字串|
|**serviceName**  <br>*選用*|服務名稱。|字串|
|**狀態**  <br>*選用*|目前部署狀態。|字串|


<a name="deploymentlist"></a>
### <a name="deploymentlist"></a>DeploymentList
部署物件的陣列。

*類型*：<[Deployment](#deployment)> 陣列


<a name="errordetail"></a>
### <a name="errordetail"></a>ErrorDetail
模型管理服務錯誤詳細資料。


|名稱|說明|結構描述|
|---|---|---|
|**code**  <br>*必要*|錯誤碼。|字串|
|**message**  <br>*必要*|錯誤訊息。|字串|


<a name="errorresponse"></a>
### <a name="errorresponse"></a>ErrorResponse
模型管理服務錯誤物件。


|名稱|說明|結構描述|
|---|---|---|
|**code**  <br>*必要*|錯誤碼。|字串|
|**details**  <br>*選用*|錯誤詳細資料物件的陣列。|<[ErrorDetail](#errordetail)> 陣列|
|**message**  <br>*必要*|錯誤訊息。|字串|
|**statusCode**  <br>*選用*|HTTP 狀態碼。|integer|


<a name="image"></a>
### <a name="image"></a>映像
Azure Machine Learning 映像。


|名稱|說明|結構描述|
|---|---|---|
|**computeResourceId**  <br>*選用*|在 Machine Learning 計算資源中建立之環境的識別碼。|字串|
|**createdTime**  <br>*選用*|映像建立時間 (UTC)。|字串 (日期-時間)|
|**creationState**  <br>*選用*||[AsyncOperationState](#asyncoperationstate)|
|**description**  <br>*選用*|映像描述文字。|字串|
|**error**  <br>*選用*||[ErrorResponse](#errorresponse)|
|**id**  <br>*選用*|映像識別碼。|字串|
|**imageBuildLogUri**  <br>*選用*|從映像組建上傳之記錄的 URI。|字串|
|**imageLocation**  <br>*選用*|已建立映像的 Azure Container Registry 位置字串。|字串|
|**imageType**  <br>*選用*||[ImageType](#imagetype)|
|**manifest**  <br>*選用*||[資訊清單](#manifest)|
|**name**  <br>*選用*|映像名稱。|字串|
|**version**  <br>*選用*|模型管理服務所設定的映像版本。|integer|


<a name="imagerequest"></a>
### <a name="imagerequest"></a>ImageRequest
要求建立 Azure Machine Learning 映像。


|名稱|說明|結構描述|
|---|---|---|
|**computeResourceId**  <br>*必要*|在 Machine Learning 計算資源中建立之環境的識別碼。|字串|
|**description**  <br>*選用*|映像描述文字。|字串|
|**imageType**  <br>*必要*||[ImageType](#imagetype)|
|**manifestId**  <br>*必要*|將從中建立映像的資訊清單識別碼。|字串|
|**name**  <br>*必要*|映像名稱。|字串|


<a name="imagetype"></a>
### <a name="imagetype"></a>ImageType
指定映像的類型。

*類型*：enum (Docker)


<a name="manifest"></a>
### <a name="manifest"></a>資訊清單
Azure Machine Learning 資訊清單。


|名稱|說明|結構描述|
|---|---|---|
|**assets**  <br>*必要*|資產清單。|<[Asset](#asset)> 陣列|
|**createdTime**  <br>*選用*  <br>*唯讀*|資訊清單建立時間 (UTC)。|字串 (日期-時間)|
|**description**  <br>*選用*|資訊清單描述文字。|字串|
|**driverProgram**  <br>*必要*|資訊清單的驅動程式。|字串|
|**id**  <br>*選用*|資訊清單識別碼。|字串|
|**modelIds**  <br>*選用*|已註冊模型的模型識別碼清單。 如果尚未註冊任何已包含的模型，則要求會失敗。|<string> 陣列|
|**modelType**  <br>*選用*|指明已經向模型管理服務註冊模型。|enum (Registered)|
|**name**  <br>*必要*|資訊清單名稱。|字串|
|**targetRuntime**  <br>*必要*||[TargetRuntime](#targetruntime)|
|**version**  <br>*選用*  <br>*唯讀*|模型管理服務所指派的資訊清單版本。|integer|
|**webserviceType**  <br>*選用*|指定將從資訊清單建立的所需 Web 服務類型。|enum (Realtime)|


<a name="model"></a>
### <a name="model"></a>模型
Azure Machine Learning 模型的執行個體。


|名稱|說明|結構描述|
|---|---|---|
|**createdAt**  <br>*選用*  <br>*唯讀*|模型建立時間 (UTC)。|字串 (日期-時間)|
|**description**  <br>*選用*|模型描述文字。|字串|
|**id**  <br>*選用*  <br>*唯讀*|模型識別碼。|字串|
|**mimeType**  <br>*必要*|模型內容的 MIME 類型。 如需 MIME 類型的詳細資訊，請參閱 [IANA 媒體類型的清單](https://www.iana.org/assignments/media-types/media-types.xhtml)。|字串|
|**name**  <br>*必要*|模型名稱。|字串|
|**「標記」**  <br>*選用*|模型標籤清單。|<string> 陣列|
|**unpack**  <br>*選用*|指出我們是否需要在 Docker 映像建立期間將模型解壓縮。|布林值|
|**url**  <br>*必要*|模型的 URL。 通常我們會將共用存取簽章 URL 放在此處。|字串|
|**version**  <br>*選用*  <br>*唯讀*|模型管理服務所指派的模型版本。|integer|


<a name="modeldatacollection"></a>
### <a name="modeldatacollection"></a>ModelDataCollection
模型資料收集資訊。


|名稱|說明|結構描述|
|---|---|---|
|**eventHubEnabled**  <br>*選用*|啟用服務的事件中樞。|布林值|
|**storageEnabled**  <br>*選用*|啟用服務的儲存體。|布林值|


<a name="paginatedimagelist"></a>
### <a name="paginatedimagelist"></a>PaginatedImageList
已編頁的映像清單。


|名稱|說明|結構描述|
|---|---|---|
|**nextLink**  <br>*選用*|清單中下一個結果頁面的接續連結 (絕對 URI)。|字串|
|**value**  <br>*選用*|模型物件的陣列。|<[Image](#image)> 陣列|


<a name="paginatedmanifestlist"></a>
### <a name="paginatedmanifestlist"></a>PaginatedManifestList
已編頁之資訊清單的清單。


|名稱|說明|結構描述|
|---|---|---|
|**nextLink**  <br>*選用*|清單中下一個結果頁面的接續連結 (絕對 URI)。|字串|
|**value**  <br>*選用*|資訊清單物件的陣列。|<[Manifest](#manifest)> 陣列|


<a name="paginatedmodellist"></a>
### <a name="paginatedmodellist"></a>PaginatedModelList
已編頁的模型清單。


|名稱|說明|結構描述|
|---|---|---|
|**nextLink**  <br>*選用*|清單中下一個結果頁面的接續連結 (絕對 URI)。|字串|
|**value**  <br>*選用*|模型物件的陣列。|<[Model](#model)> 陣列|


<a name="paginatedservicelist"></a>
### <a name="paginatedservicelist"></a>PaginatedServiceList
已編頁的服務清單。


|名稱|說明|結構描述|
|---|---|---|
|**nextLink**  <br>*選用*|清單中下一個結果頁面的接續連結 (絕對 URI)。|字串|
|**value**  <br>*選用*|服務物件的陣列。|<[ServiceResponse](#serviceresponse)> 陣列|


<a name="servicecreaterequest"></a>
### <a name="servicecreaterequest"></a>ServiceCreateRequest
要求建立服務。


|名稱|說明|結構描述|
|---|---|---|
|**appInsightsEnabled**  <br>*選用*|為服務啟用 Application Insights。|布林值|
|**autoScaler**  <br>*選用*||[AutoScaler](#autoscaler)|
|**computeResource**  <br>*必要*||[ComputeResource](#computeresource)|
|**containerResourceReservation**  <br>*選用*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*選用*||[ModelDataCollection](#modeldatacollection)|
|**imageId**  <br>*必要*|用來建立服務的映像。|字串|
|**maxConcurrentRequestsPerContainer**  <br>*選用*|並行要求數上限。  <br>**最小值**：`1`|integer|
|**name**  <br>*必要*|服務名稱。|字串|
|**numReplicas**  <br>*選用*|在任何時間執行的 pod 複本數目。 如果已啟用 Autoscaler，則不能指定。  <br>**最小值**：`0`|integer|


<a name="serviceregeneratekeyrequest"></a>
### <a name="serviceregeneratekeyrequest"></a>ServiceRegenerateKeyRequest
要求重新產生服務的金鑰。


|名稱|說明|結構描述|
|---|---|---|
|**keyType**  <br>*選用*|指定要產生哪種金鑰。|enum (Primary, Secondary)|


<a name="serviceresponse"></a>
### <a name="serviceresponse"></a>ServiceResponse
服務的詳細狀態。


|名稱|說明|結構描述|
|---|---|---|
|**createdAt**  <br>*選用*|服務建立時間 (UTC)。|字串 (日期-時間)|
|**識別碼**  <br>*選用*|服務識別碼。|字串|
|**映像**  <br>*選用*||[映像](#image)|
|**manifest**  <br>*選用*||[資訊清單](#manifest)|
|**models**  <br>*選用*|模型的清單。|<[Model](#model)> 陣列|
|**name**  <br>*選用*|服務名稱。|字串|
|**scoringUri**  <br>*選用*|用於服務評分的 URI。|字串|
|**state**  <br>*選用*||[AsyncOperationState](#asyncoperationstate)|
|**updatedAt**  <br>*選用*|上次日期時間 (UTC)。|字串 (日期-時間)|
|**appInsightsEnabled**  <br>*選用*|為服務啟用 Application Insights。|布林值|
|**autoScaler**  <br>*選用*||[AutoScaler](#autoscaler)|
|**computeResource**  <br>*必要*||[ComputeResource](#computeresource)|
|**containerResourceReservation**  <br>*選用*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*選用*||[ModelDataCollection](#modeldatacollection)|
|**maxConcurrentRequestsPerContainer**  <br>*選用*|並行要求數上限。  <br>**最小值**：`1`|integer|
|**numReplicas**  <br>*選用*|在任何時間執行的 pod 複本數目。 如果已啟用 Autoscaler，則不能指定。  <br>**最小值**：`0`|integer|
|**error**  <br>*選用*||[ErrorResponse](#errorresponse)|


<a name="serviceupdaterequest"></a>
### <a name="serviceupdaterequest"></a>ServiceUpdateRequest
要求更新服務。


|名稱|說明|結構描述|
|---|---|---|
|**appInsightsEnabled**  <br>*選用*|為服務啟用 Application Insights。|布林值|
|**autoScaler**  <br>*選用*||[AutoScaler](#autoscaler)|
|**containerResourceReservation**  <br>*選用*||[ContainerResourceReservation](#containerresourcereservation)|
|**dataCollection**  <br>*選用*||[ModelDataCollection](#modeldatacollection)|
|**imageId**  <br>*選用*|用來建立服務的映像。|字串|
|**maxConcurrentRequestsPerContainer**  <br>*選用*|並行要求數上限。  <br>**最小值**：`1`|integer|
|**numReplicas**  <br>*選用*|在任何時間執行的 pod 複本數目。 如果已啟用 Autoscaler，則不能指定。  <br>**最小值**：`0`|integer|


<a name="targetruntime"></a>
### <a name="targetruntime"></a>TargetRuntime
目標執行階段的類型。


|名稱|說明|結構描述|
|---|---|---|
|**properties**  <br>*必要*||<string, string> 對應|
|**runtimeType**  <br>*必要*|指定執行階段。|enum (SparkPython, Python)|

