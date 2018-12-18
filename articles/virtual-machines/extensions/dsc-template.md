---
title: 採用 Azure Resource Manager 範本的預期狀態設定延伸模組
description: 了解 Azure 中適用於「預期狀態設定」(DSC) 延伸模組的 Resource Manager 範本定義。
services: virtual-machines-windows
documentationcenter: ''
author: DCtheGeek
manager: carmonm
editor: ''
tags: azure-resource-manager
keywords: dsc
ms.assetid: b5402e5a-1768-4075-8c19-b7f7402687af
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: na
ms.date: 05/02/2018
ms.author: dacoulte
ms.openlocfilehash: 1dcbc8e0221689a6ece7e061d4b1a2632986ae84
ms.sourcegitcommit: 44fa77f66fb68e084d7175a3f07d269dcc04016f
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/24/2018
ms.locfileid: "39224369"
---
# <a name="desired-state-configuration-extension-with-azure-resource-manager-templates"></a>採用 Azure Resource Manager 範本的預期狀態設定延伸模組

本文說明適用於[預期狀態設定 (DSC) 延伸模組處理常式](dsc-overview.md)的 Azure Resource Manager 範本。

> [!NOTE]
> 您可能會遇到略為不同的結構描述範例。 結構描述變更出現在 2016 年 10 月的版本中。 如需詳細資料，請參閱[從先前的格式更新](#update-from-a-previous-format)。

## <a name="template-example-for-a-windows-vm"></a>Windows VM 的範本範例

下列程式碼片段會進入範本的 **Resource** 區段。
DSC 延伸模組會繼承預設的延伸模組屬性。
如需詳細資訊，請參閱 [VirtualMachineExtension 類別](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.compute.models.virtualmachineextension?view=azure-dotnet.)(英文\)。

```json
{
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "[concat(parameters('VMName'),'/Microsoft.Powershell.DSC')]",
    "apiVersion": "2017-12-01",
    "location": "[resourceGroup().location]",
    "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', parameters('VMName'))]"
    ],
    "properties": {
        "publisher": "Microsoft.Powershell",
        "type": "DSC",
        "typeHandlerVersion": "2.76",
        "autoUpgradeMinorVersion": true,
        "settings": {
            "protectedSettings": {
                "Items": {
                    "registrationKeyPrivate": "registrationKey"
                }
            },
            "publicSettings": {
                "configurationArguments": [{
                        "Name": "RegistrationKey",
                        "Value": {
                            "UserName": "PLACEHOLDER_DONOTUSE",
                            "Password": "PrivateSettingsRef:registrationKeyPrivate"
                        }
                    },
                    {
                        "RegistrationUrl": "registrationUrl"
                    },
                    {
                        "NodeConfigurationName": "nodeConfigurationName"
                    }
                ]
            }
        }
    }
}
```

## <a name="template-example-for-windows-virtual-machine-scale-sets"></a>Windows 虛擬機器擴展集的範本範例

虛擬機器擴展集節點具有 **properties** 區段，其中含有 **VirtualMachineProfile, extensionProfile** 屬性。
在 **extensions** 下方新增 DSC 擴充功能的詳細資料。

DSC 延伸模組會繼承預設的延伸模組屬性。
如需詳細資訊，請參閱 [VirtualMachineScaleSetExtension 類別](/dotnet/api/microsoft.azure.management.compute.models.virtualmachinescalesetextension?view=azure-dotnet)(英文\)。

```json
"extensionProfile": {
    "extensions": [{
        "type": "Microsoft.Compute/virtualMachines/extensions",
        "name": "[concat(parameters('VMName'),'/Microsoft.Powershell.DSC')]",
        "apiVersion": "2017-12-01",
        "location": "[resourceGroup().location]",
        "dependsOn": [
            "[concat('Microsoft.Compute/virtualMachines/', parameters('VMName'))]"
        ],
        "properties": {
            "publisher": "Microsoft.Powershell",
            "type": "DSC",
            "typeHandlerVersion": "2.76",
            "autoUpgradeMinorVersion": true,
            "settings": {
                "protectedSettings": {
                    "Items": {
                        "registrationKeyPrivate": "registrationKey"
                    }
                },
                "publicSettings": {
                    "configurationArguments": [{
                            "Name": "RegistrationKey",
                            "Value": {
                                "UserName": "PLACEHOLDER_DONOTUSE",
                                "Password": "PrivateSettingsRef:registrationKeyPrivate"
                            },
                        },
                        {
                            "RegistrationUrl": "registrationUrl"
                        },
                        {
                            "NodeConfigurationName": "nodeConfigurationName"
                        }
                    ]
                }
            },
        }
    }]
}
```

## <a name="detailed-settings-information"></a>詳細的設定資訊

請在 Resource Manager 範本中 Azure DSC 延伸模組的 **settings** 區段中使用下列結構描述。

如需預設設定指令碼可用的引數清單，請參閱[預設設定指令碼](#default-configuration-script)。

```json
"settings": {
    "wmfVersion": "latest",
    "configuration": {
        "url": "http://validURLToConfigLocation",
        "script": "ConfigurationScript.ps1",
        "function": "ConfigurationFunction"
    },
    "configurationArguments": {
        "argument1": "Value1",
        "argument2": "Value2"
    },
    "configurationData": {
        "url": "https://foo.psd1"
    },
    "privacy": {
        "dataCollection": "enable"
    },
    "advancedOptions": {
        "downloadMappings": {
            "customWmfLocation": "http://myWMFlocation"
        }
    }
},
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
            "userName": "UsernameValue1",
            "password": "PasswordValue1"
        },
        "parameterOfTypePSCredential2": {
            "userName": "UsernameValue2",
            "password": "PasswordValue2"
        }
    },
    "configurationUrlSasToken": "?g!bber1sht0k3n",
    "configurationDataUrlSasToken": "?dataAcC355T0k3N"
}
```

## <a name="details"></a>詳細資料

| 屬性名稱 | 類型 | 說明 |
| --- | --- | --- |
| settings.wmfVersion |字串 |指定應該安裝在您 VM 上的 Windows Management Framework (WMF) 版本。 將此屬性設定為 **latest** 會安裝最新版的 WMF。 此屬性目前只有下列可能值：**4.0****5.0****5.0PP** 及 **latest**。 這些可能的值可能會更新。 預設值為 **latest**。 |
| settings.configuration.url |字串 |指定要從中下載 DSC 設定 .zip 檔案的 URL 位置。 如果所提供的 URL 需要 SAS 權杖才能存取，請將 **protectedSettings.configurationUrlSasToken** 屬性設定為您 SAS 權杖的值。 如果已定義 **settings.configuration.script** 或 **settings.configuration.function**，就需要這個屬性。 如果沒有為這些屬性指定值，延伸模組就會呼叫預設設定指令碼來設定「位置設定管理員」(LCM) 中繼資料，而應該提供引數。 |
| settings.configuration.script |字串 |指定指令碼的檔案名稱，其中包含 DSC 組態的定義。 此指令碼必須位於從 **configuration.url**屬性所指定 URL 下載之 zip 檔案的根資料夾中。 如果已定義 **settings.configuration.url** 或 **settings.configuration.script**，就需要這個屬性。 如果沒有為這些屬性指定值，延伸模組就會呼叫預設設定指令碼來設定 LCM 中繼資料，而應該提供引數。 |
| settings.configuration.function |字串 |指定 DSC 組態的名稱。 所命名的設定必須包含在 **configuration.script** 所定義的指令碼中。 如果已定義 **settings.configuration.url** 或 **settings.configuration.function**，就需要這個屬性。 如果沒有為這些屬性指定值，延伸模組就會呼叫預設設定指令碼來設定 LCM 中繼資料，而應該提供引數。 |
| settings.configurationArguments |集合 |定義任何您想要傳遞給 DSC 設定的參數。 這個屬性並未加密。 |
| settings.configurationData.url |字串 |指定 URL，從中下載您的組態資料 (.psd1) 檔案以做為 DSC 組態的輸入。 如果所提供的 URL 需要 SAS 權杖才能存取，請將 **protectedSettings.configurationDataUrlSasToken** 屬性設定為您 SAS 權杖的值。 |
| settings.privacy.dataEnabled |字串 |啟用或停用遙測收集。 此屬性只有下列可能值：**Enable****Disable****''** 或 **$null**。 將此屬性保持空白或 null 即可啟用遙測。 預設值為 **''**。 如需詳細資訊，請參閱 [Azure DSC 延伸模組集合](https://blogs.msdn.microsoft.com/powershell/2016/02/02/azure-dsc-extension-data-collection-2/) \(英文\)。 |
| settings.advancedOptions.downloadMappings |集合 |定義要從中下載 WMF 的替代位置。 如需詳細資訊，請參閱 [Azure DSC 延伸模組 2.8 及如何將延伸模組相依性的下載對應至您自己的位置](http://blogs.msdn.com/b/powershell/archive/2015/10/21/azure-dsc-extension-2-2-amp-how-to-map-downloads-of-the-extension-dependencies-to-your-own-location.aspx)(英文\)。 |
| protectedSettings.configurationArguments |集合 |定義任何您想要傳遞給 DSC 設定的參數。 這個屬性已加密。 |
| protectedSettings.configurationUrlSasToken |字串 |指定用來存取 **configuration.url** 所定義 URL 的 SAS 權杖。 這個屬性已加密。 |
| protectedSettings.configurationDataUrlSasToken |字串 |指定用來存取 **configurationData.url**所定義 URL 的 SAS 權杖。 這個屬性已加密。 |

## <a name="default-configuration-script"></a>預設設定指令碼

如需有關下列值的詳細資訊，請參閱[本機設定管理員基本設定](/powershell/dsc/metaconfig#basic-settings)。
您可以使用 DSC 延伸模組預設設定指令碼來設定的 LCM 屬性僅限下表所列的屬性。

| 屬性名稱 | 類型 | 說明 |
| --- | --- | --- |
| settings.configurationArguments.RegistrationKey |securestring |必要屬性。 指定節點向「Azure 自動化」服務註冊時，用來作為 PowerShell 認證物件密碼的金鑰。 針對「自動化」帳戶使用 **listkeys** 方法，即可自動探索此值。 此值應該作為一項受保護的設定來受到保護。 |
| settings.configurationArguments.RegistrationUrl |字串 |必要屬性。 指定節點嘗試進行註冊之「自動化」端點的 URL。 針對「自動化」帳戶使用**reference** 方法，即可自動探索此值。 |
| settings.configurationArguments.NodeConfigurationName |字串 |必要屬性。 指定「自動化」帳戶中要指派給節點的 節點設定。 |
| settings.configurationArguments.ConfigurationMode |字串 |指定 LCM 的模式。 有效的選項包括 **ApplyOnly****ApplyandMonitor**, 及 **ApplyandAutoCorrect**。  預設值為 **ApplyandMonitor**。 |
| settings.configurationArguments.RefreshFrequencyMins | uint32 | 指定 LCM 嘗試檢查「自動化」帳戶是否有更新的頻率。  預設值為 **30**。  最小值為 **15**。 |
| settings.configurationArguments.ConfigurationModeFrequencyMins | uint32 | 指定 LCM 驗證目前設定的頻率。 預設值為 **15**。 最小值為 **15**。 |
| settings.configurationArguments.RebootNodeIfNeeded | 布林值 | 指定節點是否能在 DSC 作業要求時自動重新開機。 預設值為 **false**。 |
| settings.configurationArguments.ActionAfterReboot | 字串 | 指定套用設定時，在重新開機後會發生什麼事。 有效的選項為 **ContinueConfiguration** 和 **StopConfiguration**。 預設值為 **ContinueConfiguration**。 |
| settings.configurationArguments.AllowModuleOverwrite | 布林值 | 指定 LCM 是否會覆寫節點上現有的模組。 預設值為 **false**。 |

## <a name="settings-vs-protectedsettings"></a>Settings 與ProtectedSettings

所有設定都會儲存在 VM 上的 settings 文字檔中。
列在 **settings** 底下的屬性是公用屬性。
公用屬性在設定文字檔中不會加密。
列在 **protectedSettings** 底下的屬性會以憑證加密，而不會在 VM 上的設定檔案中以純文字顯示。

如果設定需要認證，您可以將認證包含在 **protectedSettings**中：

```json
"protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
            "userName": "UsernameValue1",
            "password": "PasswordValue1"
        }
    }
}
```

## <a name="example-configuration-script"></a>範例設定指令碼

下列範例說明 DSC 延伸模組的預設行為，也就是向 LCM 提供中繼資料設定，並向 Automation DSC 服務註冊。
設定引數為必要引數。
設定引數會傳遞給預設設定指令碼來設定 LCM 中繼資料。

```json
"settings": {
    "configurationArguments": {
        {
            "Name": "RegistrationKey",
            "Value": {
                "UserName": "PLACEHOLDER_DONOTUSE",
                "Password": "PrivateSettingsRef:registrationKeyPrivate"
            },
        },
        "RegistrationUrl": "[parameters('registrationUrl1')]",
        "NodeConfigurationName": "nodeConfigurationNameValue1"
    }
},
"protectedSettings": {
    "Items": {
        "registrationKeyPrivate": "[parameters('registrationKey1')]"
    }
}
```

## <a name="example-using-the-configuration-script-in-azure-storage"></a>在 Azure 儲存體中使用設定指令碼的範例

下列範例來自 [DSC 延伸模組處理常式概觀](dsc-overview.md)。
此範例使用 Resource Manager 範本 (而不是Cmdlet) 來部署擴充功能。
請儲存 IisInstall.ps1 設定、將它放在 .zip 檔案中，然後以可存取的 URL 上傳此檔案。
此範例會使用 Azure Blob 儲存體，但您可以從任意位置下載 .zip 檔案。

在 Resource Manager 範本中，下列程式碼會指示 VM 下載正確的檔案，然後執行適當的 PowerShell 函式：

```json
"settings": {
    "configuration": {
        "url": "https://demo.blob.core.windows.net/",
        "script": "IisInstall.ps1",
        "function": "IISInstall"
    }
},
"protectedSettings": {
    "configurationUrlSasToken": "odLPL/U1p9lvcnp..."
}
```

## <a name="update-from-a-previous-format"></a>從先前的格式更新

採用先前延伸模組格式 (且包含 **ModulesUrl****ConfigurationFunction****SasToken**或 **Properties** 等公用屬性) 的任何設定，都會自動調整成目前的延伸模組格式。
其執行方式會與之前相同。

下列結構描述顯示先前 settings 結構描述看起來的樣子︰

```json
"settings": {
    "WMFVersion": "latest",
    "ModulesUrl": "https://UrlToZipContainingConfigurationScript.ps1.zip",
    "SasToken": "SAS Token if ModulesUrl points to private Azure Blob Storage",
    "ConfigurationFunction": "ConfigurationScript.ps1\\ConfigurationFunction",
    "Properties": {
        "ParameterToConfigurationFunction1": "Value1",
        "ParameterToConfigurationFunction2": "Value2",
        "ParameterOfTypePSCredential1": {
            "UserName": "UsernameValue1",
            "Password": "PrivateSettingsRef:Key1"
        },
        "ParameterOfTypePSCredential2": {
            "UserName": "UsernameValue2",
            "Password": "PrivateSettingsRef:Key2"
        }
    }
},
"protectedSettings": {
    "Items": {
        "Key1": "PasswordValue1",
        "Key2": "PasswordValue2"
    },
    "DataBlobUri": "https://UrlToConfigurationDataWithOptionalSasToken.psd1"
}
```

以下是先前的格式如何調整成目前的格式︰

| 屬性名稱 | 先前結構描述的對等項目 |
| --- | --- |
| settings.wmfVersion |settings.wmfVersion |
| settings.configuration.url |settings.ModulesUrl |
| settings.configuration.script |settings.ConfigurationFunction 的第一個部分 (在 \\\\ 之前) |
| settings.configuration.function |settings.ConfigurationFunction 的第二個部分 (在 \\\\ 之後) |
| settings.configurationArguments |settings.Properties |
| settings.configurationData.url |protectedSettings.DataBlobUri (不含 SAS 權杖) |
| settings.privacy.dataEnabled |settings.privacy.dataEnabled |
| settings.advancedOptions.downloadMappings |settings.advancedOptions.downloadMappings |
| protectedSettings.configurationArguments |protectedSettings.Properties |
| protectedSettings.configurationUrlSasToken |settings.SasToken |
| protectedSettings.configurationDataUrlSasToken |SAS token from protectedSettings.DataBlobUri |

## <a name="troubleshooting---error-code-1100"></a>疑難排解 - 錯誤碼 1100

錯誤碼 1100 表示與 DSC 延伸模組的使用者輸入有關的問題。
這些錯誤的文字並非固定，可能會有變更。
以下是一些您可能遇到的錯誤及其修正方式。

### <a name="invalid-values"></a>無效的值

「Privacy.dataCollection 為 '{0}'。
僅有的可能值為 ''、'Enable' 和 'Disable'"。
「WmfVersion 為 '{0}'。
可能的值為 … 和 'latest'"。

**問題**︰所提供的值不是允許的值。

**解決方式**︰將無效的值變更為有效的值。
如需詳細資訊，請參閱[詳細資料](#details)中的表格。

### <a name="invalid-url"></a>無效的 URL

「ConfigurationData.url 為 '{0}'。 這不是有效的 URL」「DataBlobUri 為 '{0}'。 這不是有效的 URL」「Configuration.url 為 '{0}'。 這不是有效的 URL」

**問題**︰所提供的 URL 無效。

**解決方式**︰檢查您提供的所有 URL。
請確定所有 URL 都會解析成延伸模組可在遠端電腦上存取的有效位置。

### <a name="invalid-configurationargument-type"></a>無效的 ConfigurationArgument 類型

「無效的 configurationArguments 類型 {0}」

**問題**︰*ConfigurationArguments* 屬性無法解析成 **Hashtable**物件。

**解決方式**︰將 *ConfigurationArguments* 屬性設定成**雜湊表**。
請依照上述範例中提供的格式。 請留意引號、逗號及大括號。

### <a name="duplicate-configurationarguments"></a>重複的 ConfigurationArguments

「在公用和受保護的 configurationArguments 中找到重複的引數 '{0}'」

**問題**︰公用設定中的 *ConfigurationArguments* 和受保護設定中的 *ConfigurationArguments* 具有相同名稱的屬性。

**解決方式**︰移除其中一個重複的屬性。

### <a name="missing-properties"></a>遺漏屬性

「Configuration.function 要求指定 configuration.url 或 configuration.module」

「Configuration.url 要求指定 configuration.script」

「Configuration.script 要求指定 configuration.url」

「Configuration.url 要求指定 configuration.function」

「ConfigurationUrlSasToken 要求指定 configuration.url」

「ConfigurationDataUrlSasToken 要求指定 configurationData.url」

**問題**︰已定義的屬性需要另一個屬性，但遺漏該屬性。

**解決方式**：

- 提供遺漏的屬性。
- 移除需要遺漏屬性的屬性。

## <a name="next-steps"></a>後續步驟

- 了解[搭配 Azure DSC 延伸模組使用虛擬機器擴展集](../../virtual-machine-scale-sets/virtual-machine-scale-sets-dsc.md)。
- 深入了解 [DSC 的安全認證管理](dsc-credentials.md)。
- 取得 [Azure DSC 延伸模組處理常式簡介](dsc-overview.md)。
- 如需有關 PowerShell DSC 的詳細資訊，請移至 [PowerShell 文件中心](/powershell/dsc/overview)。
