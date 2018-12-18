---
title: 遠端監視存取控制 - Azure | Microsoft Docs
description: 本文提供如何在遠端監視解決方案加速器中設定角色型存取控制 (RBAC) 的相關資訊
author: dominicbetts
manager: timlt
ms.author: dobett
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 08/06/2018
ms.topic: conceptual
ms.openlocfilehash: a56cb92dc8870bf3fff6de0b1d5d907a0898c216
ms.sourcegitcommit: ce526d13cd826b6f3e2d80558ea2e289d034d48f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 09/19/2018
ms.locfileid: "46364290"
---
# <a name="configure-role-based-access-controls-in-the-remote-monitoring-solution-accelerator"></a>在遠端監視解決方案加速器中設定角色型存取控制

本文提供如何在遠端監視解決方案加速器中設定角色型存取控制的相關資訊。 角色型存取控制可讓您將個別使用者的存取權限制為解決方案中的特定功能。

## <a name="default-settings"></a>預設設定

當您第一次部署遠端監視解決方案時，有兩個角色：**管理員**和**唯讀**。

擁有 [管理員] 角色的任何使用者具有解決方案的完整存取權。 擁有 [唯讀] 角色的使用者無法執行下列任何工作：

- 更新警示
- 刪除警示
- 建置裝置
- 更新裝置
- 刪除裝置
- 建立裝置群組
- 更新裝置群組
- 刪除裝置群組
- 建立規則
- 更新規則
- 刪除規則
- 建立作業
- 更新 SIM 管理

部署遠端監視解決方案的人員會自動指派給 [管理員] 角色而且是 Azure Active Directory 應用程式擁有者。 應用程式擁有者可以將角色指派給 Azure 入口網站中的其他使用者。

如果您希望另一位使用者在解決方案中指派角色，他們也必須設定為 Azure 入口網站中的應用程式擁有者。

## <a name="add-or-remove-users"></a>新增或移除使用者

使用 Azure 入口網站在遠端監視解決方案中新增或移除使用者。 下列步驟會使用當您部署遠端監視解決方案時所建立的 [Azure Active Directory 企業應用程式](../active-directory/manage-apps/add-application-portal.md#find-your-azure-ad-tenant-application)。

1. 登入 [Azure 入口網站](https://portal.azure.com)。

1. 請檢查[使用者是否在您所用的目錄中](../active-directory/fundamentals/add-users-azure-active-directory.md)。 當您登入 [Microsoft Azure IoT 解決方案加速器](https://www.azureiotsolutions.com/Accelerators)網站時，您會選擇要使用的目錄。 目錄名稱會顯示在[頁面](https://www.azureiotsolutions.com/Accelerators)的右上角。

1. 在 Azure 入口網站中為您的解決方案尋找**企業應用程式**。 到達該處後，藉由將 [應用程式類型] 設定為 [所有應用程式] 來篩選清單。 依應用程式名稱搜尋應用程式。 應用程式名稱是遠端監視解決方案的名稱。 在下列螢幕擷取畫面中，解決方案和應用程式顯示名稱為 **contoso-rm4**。

    ![企業應用程式](media/iot-accelerators-remote-monitoring-rbac/appregistration.png)

1. 檢查應用程式，然後按一下 [擁有者]，確認您是應用程式的擁有者。 在下列螢幕擷取畫面中，[Contoso 管理員] 是 **contoso-rm4** 應用程式的擁有者：

    ![擁有者](media/iot-accelerators-remote-monitoring-rbac/owners.png)

    如果您不是擁有者，您必須要求現有擁有者將您新增至清單。 只有擁有者可以將應用程式角色 (例如**管理員**或**唯讀**) 指派給其他使用者。

1. 若要查看指派給應用程式中角色的使用者清單，請按一下 [使用者和群組]。

1. 若要新增使用者，請按一下 [+ 新增使用者]，然後按一下 [使用者和群組，未選取任何項目] 以從目錄中選取使用者。

1. 若要將使用者指派給角色，請按一下 [選取角色，未選取任何項目]，並且為使用者選擇 [管理員] 或 [唯讀] 角色。 按一下 [選取]，然後按一下 [指派]。

    ![選取角色](media/iot-accelerators-remote-monitoring-rbac/selectrole.png)

1. 使用者現在可以使用角色所定義的權限來存取遠端監視解決方案。

1. 您可以在入口網站中的 [使用者和群組] 頁面上，從應用程式刪除使用者。

## <a name="create-a-custom-role"></a>建立自訂角色

遠端監視解決方案在第一次部署時包含 [管理員] 和 [唯讀] 角色。 您可以新增具有不同權限集的自訂角色。 若要定義自訂角色，您需要：

- 在 Azure 入口網站中將角色新增至應用程式。
- 在驗證和授權微服務中定義新角色的原則。
- 更新解決方案的 Web UI。

### <a name="define-a-custom-role-in-the-azure-portal"></a>在 Azure 入口網站中定義自訂角色

下列步驟說明如何將角色新增至 Azure Active Directory 中的應用程式。 在此範例中，您可以建立新角色，讓成員能在遠端監視解決方案中建立、更新和刪除裝置。

1. 在 Azure 入口網站中為您的解決方案尋找**應用程式註冊**。 應用程式名稱是遠端監視解決方案的名稱。 在下列螢幕擷取畫面中，解決方案和應用程式顯示名稱為 **contoso-rm4**。

    ![應用程式註冊](media/iot-accelerators-remote-monitoring-rbac/appregistration2.png)

1. 選取您的應用程式，然後按一下 [資訊清單]。 您可以看到兩個針對應用程式定義的現有[應用程式角色](https://docs.microsoft.com/azure/architecture/multitenant-identity/app-roles)：

    ![檢視資訊清單](media/iot-accelerators-remote-monitoring-rbac/viewmanifest.png)

1. 編輯資訊清單，以新增稱為 **ManageDevices** 的角色，如下列程式碼片段所示。 您需要唯一的字串，例如 GUID 作為新的角色識別碼。 您可以使用 [Online GUID Generator](https://www.guidgenerator.com/) 等服務來產生新的 GUID：

    ```json
    "appRoles": [
      {
        "allowedMemberTypes": [
          "User"
        ],
        "displayName": "Admin",
        "id": "a400a00b-f67c-42b7-ba9a-f73d8c67e433",
        "isEnabled": true,
        "description": "Administrator access to the application",
        "value": "Admin"
      },
      {
        "allowedMemberTypes": [
          "User"
        ],
        "displayName": "Read Only",
        "id": "e5bbd0f5-128e-4362-9dd1-8f253c6082d7",
        "isEnabled": true,
        "description": "Read only access to device information",
        "value": "ReadOnly"
      },
      {
        "allowedMemberTypes": [
          "User"
        ],
        "displayName": "ManageDevices",
        "id": "ADD A NEW GUID HERE",
        "isEnabled": true,
        "description": "A new role for the application.",
        "value": "ManageDevices"
      }
    ],
    ```

    儲存變更。

### <a name="define-a-policy-for-the-new-role"></a>定義新角色的原則

若要將角色新增至 Azure 入口網站中的應用程式，您需要針對可指派管理裝置所需權限的角色，在 [roles.json](https://github.com/Azure/remote-monitoring-services-dotnet/blob/master/auth/Services/data/policies/roles.json) 中定義原則。

1. 從 GitHub 將[遠端監視微服務](https://github.com/Azure/remote-monitoring-services-dotnet)存放庫複製到本機電腦。

1. 編輯 **auth/Services/data/policies/roles.json** 檔案，以新增 **ManageDevices** 角色的原則，如下列程式碼片段所示。 [識別碼] 和 [角色] 值必須符合上一節應用程式資訊清單中的角色定義。 允許的動作清單可讓擁有 **ManageDevices** 角色的人員建立、更新和刪除已連線至解決方案的裝置：

    ```json
    {
    "Items": [
      {
        "Id": "a400a00b-f67c-42b7-ba9a-f73d8c67e433",
        "Role": "admin",
        "AllowedActions": [
          "UpdateAlarms",
          "DeleteAlarms",
          "CreateDevices",
          "UpdateDevices",
          "DeleteDevices",
          "CreateDeviceGroups",
          "UpdateDeviceGroups",
          "DeleteDeviceGroups",
          "CreateRules",
          "UpdateRules",
          "DeleteRules",
          "CreateJobs",
          "UpdateSimManagement"
        ]
      },
      {
        "Id": "e5bbd0f5-128e-4362-9dd1-8f253c6082d7",
        "Role": "readOnly",
        "AllowedActions": []
      },
      {
        "Id": "GUID FROM APP MANIFEST",
        "Role": "ManageDevices",
        "AllowedActions": [
          "CreateDevices",
          "UpdateDevices",
          "DeleteDevices"
        ]
      }
    ]
    }
    ```

1. 當您完成 **Services/data/policies/roles.json** 檔案編輯時，請重新建置驗證和授權微服務並和重新部署到您的解決方案加速器。

### <a name="how-the-web-ui-enforces-permissions"></a>Web UI 如何強制執行權限

Web UI 使用[驗證和授權微服務](https://github.com/Azure/remote-monitoring-services-dotnet/tree/master/auth)來判斷使用者得以採取哪些動作，以及 UI 中看得見哪些控制項。 例如，如果您的解決方案稱為 **contoso-rm4**，則 Web UI 會藉由傳送下列要求來擷取目前使用者允許的動作清單：

```http
http://contoso-rm4.azurewebsites.net/v1/users/current
headers:
X-Source: true
Authorization: Bearer <JWT Token from ADAL>
```

對於擁有 **ManageDevices** 角色且稱為**裝置管理員**的使用者，回應本文中會包含下列 JSON：

```json
{
  "Id": "userIdExample",
  "Email": "devicemanager@contoso.com",
  "Name": "Device Manager",
  "AllowedActions": [
    "CreateDevices",
    "UpdateDevices",
    "DeleteDevices"
  ]
}
```

下列程式碼片段來自 [Web UI](https://github.com/Azure/pcs-remote-monitoring-webui/) 中的 [deviceDelete.js](https://github.com/Azure/pcs-remote-monitoring-webui/blob/master/src/components/pages/devices/flyouts/deviceDelete/deviceDelete.js)，可顯示如何以宣告方式強制執行權限：

```json
<FlyoutContent>
  <Protected permission={permissions.deleteDevices}>
    <form className="device-delete-container" onSubmit={this.deleteDevices}>
      ...
    </form>
  </Protected>
</FlyoutContent>
```

如需詳細資訊，請參閱[受保護的元件](https://github.com/Azure/pcs-remote-monitoring-webui/blob/master/src/components/shared/protected/README.md)。 您可以在 [authModel.js](https://github.com/Azure/pcs-remote-monitoring-webui/blob/master/src/services/models/authModels.js) 檔案中定義其他權限。

### <a name="how-the-microservices-enforce-permissions"></a>微服務如何強制執行權限

微服務也會檢查權限以防禦未經授權的 API 要求。 當微服務收到 API 要求時，它會解碼並驗證 JWT 權杖，以取得與使用者角色相關聯的使用者識別碼和權限。

下列程式碼片段來自 [IoTHub 管理員微服務](https://github.com/Azure/remote-monitoring-services-dotnet/tree/master/iothub-manager)中的 [DevicesController.cs](https://github.com/Azure/remote-monitoring-services-dotnet/blob/master/iothub-manager/WebService/v1/Controllers/DevicesController.cs) 檔案，可顯示如何強制執行權限：

```csharp
[HttpDelete("{id}")]
[Authorize("DeleteDevices")]
public async Task DeleteAsync(string id)
{
    await this.devices.DeleteAsync(id);
}
```

## <a name="next-steps"></a>後續步驟

在本文中，您已了解如何在遠端監視解決方案加速器中實作角色型存取控制。

如需有關在遠端監視解決方案加速器中管理時間序列深入解析總管之存取權的資訊，請參閱[設定時間序列深入解析總館的存取控制](iot-accelerators-remote-monitoring-rbac-tsi.md)。

如需關於遠端監視解決方案加速器的詳細概念資訊，請參閱[遠端監視架構](iot-accelerators-remote-monitoring-sample-walkthrough.md)

如需關於自訂遠端監視解決方案的詳細資訊，請參閱[自訂和重新部署微服務](iot-accelerators-microservices-example.md)
<!-- Next tutorials in the sequence -->