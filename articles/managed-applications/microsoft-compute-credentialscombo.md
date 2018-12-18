---
title: Azure CredentialsCombo UI 元素 | Microsoft Docs
description: 描述 Azure 入口網站的 Microsoft.Compute.CredentialsCombo UI 元素。
services: managed-applications
documentationcenter: na
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: managed-applications
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/29/2018
ms.author: tomfitz
ms.openlocfilehash: 0412d55fe60524cde404e6a640723d3259e020e1
ms.sourcegitcommit: 5843352f71f756458ba84c31f4b66b6a082e53df
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/01/2018
ms.locfileid: "47586235"
---
# <a name="microsoftcomputecredentialscombo-ui-element"></a>Microsoft.Compute.CredentialsCombo UI 元素
使用 Windows 和 Linux 密碼和 SSH 公開金鑰內建驗證的控制項群組。

## <a name="ui-sample"></a>UI 範例

針對 Windows，使用者會看到：

![Microsoft.Compute.CredentialsCombo Windows](./media/managed-application-elements/microsoft.compute.credentialscombo-windows.png)

針對已選取密碼的 Linux，使用者會看到：

![Microsoft.Compute.CredentialsCombo Linux 密碼](./media/managed-application-elements/microsoft.compute.credentialscombo-linux-password.png)

針對已選取 SSH 公開金鑰的 Linux，使用者會看到：

![Microsoft.Compute.CredentialsCombo Linux 金鑰](./media/managed-application-elements/microsoft.compute.credentialscombo-linux-key.png)

## <a name="schema"></a>結構描述
針對 Windows，請使用下列結構描述：

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "password": "Password",
    "confirmPassword": "Confirm password"
  },
  "toolTip": {
    "password": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must contain at least 12 characters, with at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false
  },
  "osPlatform": "Windows",
  "visible": true
}
```

針對 **Linux**，請使用下列結構描述：

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.CredentialsCombo",
  "label": {
    "authenticationType": "Authentication type",
    "password": "Password",
    "confirmPassword": "Confirm password",
    "sshPublicKey": "SSH public key"
  },
  "toolTip": {
    "authenticationType": "",
    "password": "",
    "sshPublicKey": ""
  },
  "constraints": {
    "required": true,
    "customPasswordRegex": "^(?=.*[A-Za-z])(?=.*\\d)[A-Za-z\\d]{12,}$",
    "customValidationMessage": "The password must contain at least 12 characters, with at least 1 letter and 1 number."
  },
  "options": {
    "hideConfirmation": false,
    "hidePassword": false
  },
  "osPlatform": "Linux",
  "visible": true
}
```

## <a name="remarks"></a>備註
- 必須指定 `osPlatform`，且可以是 **Windows** 或 **Linux**。
- 如果將 `constraints.required` 設為 **true**，則密碼或 SSH 公開金鑰文字方塊必須具有值，才能順利通過驗證。 預設值為 **true**。
- 如果將 `options.hideConfirmation` 設為 **true**，就會將確認使用者密碼的第二個文字方塊加以隱藏。 預設值為 **false**。
- 如果將 `options.hidePassword` 設為 **true**，就會將使用密碼驗證的選項加以隱藏。 只有在 `osPlatform` 是 **Linux** 時才可以使用。 預設值為 **false**。
- 可使用 `customPasswordRegex` 屬性將允許密碼上的其他條件約束加以實作。 密碼無法自訂驗證時，就會顯示 `customValidationMessage` 中的字串。 這兩個屬性的預設值是 **null**。

## <a name="sample-output"></a>範例輸出
如果 `osPlatform` 是 **Windows**，或 `osPlatform` 是 **Linux**，而且使用者提供了密碼而不是 SSH 公開金鑰，則控制項會傳回下列輸出︰

```json
{
  "authenticationType": "password",
  "password": "p4ssw0rddem0",
}
```

如果 `osPlatform` 是 **Linux** 而且使用者提供了 SSH 公開金鑰，則控制項會傳回下列輸出︰

```json
{
  "authenticationType": "sshPublicKey",
  "sshPublicKey": "AAAAB3NzaC1yc2EAAAABIwAAAIEA1on8gxCGJJWSRT4uOrR13mUaUk0hRf4RzxSZ1zRbYYFw8pfGesIFoEuVth4HKyF8k1y4mRUnYHP1XNMNMJl1JcEArC2asV8sHf6zSPVffozZ5TT4SfsUu/iKy9lUcCfXzwre4WWZSXXcPff+EHtWshahu3WzBdnGxm5Xoi89zcE=",
}
```

## <a name="next-steps"></a>後續步驟
* 如需建立 UI 定義的簡介，請參閱[開始使用 CreateUiDefinition](create-uidefinition-overview.md)。
* 如需 UI 元素中通用屬性的說明，請參閱 [CreateUiDefinition 元素](create-uidefinition-elements.md)。
