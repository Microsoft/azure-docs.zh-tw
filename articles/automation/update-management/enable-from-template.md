---
title: 使用 Azure Resource Manager 範本啟用更新管理 | Microsoft Docs
description: 此文章說明如何使用 Azure Resource Manager 範本來啟用更新管理。
ms.service: automation
ms.subservice: update-management
ms.topic: conceptual
author: mgoedtel
ms.author: magoedte
ms.date: 09/18/2020
ms.openlocfilehash: e2ebdd3d0f4a4461521ee5f412d5b4c4f872b8a0
ms.sourcegitcommit: 0aec60c088f1dcb0f89eaad5faf5f2c815e53bf8
ms.translationtype: MT
ms.contentlocale: zh-TW
ms.lasthandoff: 01/14/2021
ms.locfileid: "98183229"
---
# <a name="enable-update-management-using-azure-resource-manager-template"></a>使用 Azure Resource Manager 範本啟用更新管理

您可以使用 [Azure Resource Manager 範本](../../azure-resource-manager/templates/template-syntax.md)，來啟用資源群組中的 Azure 自動化更新管理功能。 此文章提供可將下列動作自動化的範例範本：

* 自動化建立 Azure 監視器 Log Analytics 工作區。
* 自動化建立 Azure 自動化帳戶。
* 將自動化帳戶連結至 Log Analytics 工作區。
* 將範例自動化 Runbook 新增至帳戶。
* 啟用更新管理功能。

範本不會自動啟用一或多個 Azure 或非 Azure Vm 上的更新管理。

如果您已在訂用帳戶的支援區域中部署 Log Analytics 工作區和自動化帳戶，則不會與其連結。 使用此範本會成功建立連結並部署更新管理。

>[!NOTE]
>當您使用 ARM 範本時，不支援建立自動化執行身分帳戶。 若要從入口網站或使用 PowerShell 手動建立執行身分帳戶，請參閱[管理執行身分帳戶](../manage-runas-account.md)。

完成這些步驟之後，您需要為您的自動化帳戶[設定診斷設定](../automation-manage-send-joblogs-log-analytics.md)，以將 Runbook 作業狀態和作業串流傳送至連結的 Log Analytics 工作區。

## <a name="api-versions"></a>API 版本

下表列出此範例中使用的資源 API 版本。

| 資源 | 資源類型 | API 版本 |
|:---|:---|:---|
| [工作區](/azure/templates/microsoft.operationalinsights/workspaces) | workspaces | 2020-03-01-preview |
| [自動化帳戶](/azure/templates/microsoft.automation/automationaccounts) | automation | 2020-01-13-preview |
| [工作區連結服務](/azure/templates/microsoft.operationalinsights/workspaces/linkedservices) | workspaces | 2020-03-01-preview |
| [方案](/azure/templates/microsoft.operationsmanagement/solutions) | solutions | 2015-11-01-preview |

## <a name="before-using-the-template"></a>使用範本之前

JSON 範本已設定為提示您輸入：

* 工作區的名稱。
* 要在其中建立工作區的區域。
* 自動化帳戶的名稱。
* 要在其中建立自動化帳戶的區域。

系統會使用 Log Analytics 工作區的預設值來設定範本中的下列參數：

* SKU 預設為在 2018 年 4 月定價模型中發行的每 GB 定價層。
* dataRetention 預設為 30 天。

>[!WARNING]
>如果您想要在選擇加入 2018 年 4 月定價模型的訂用帳戶中建立或設定 Log Analytics 工作區，則唯一有效的 Log Analytics 定價層是 PerGB2018。
>

JSON 範本會針對您的環境中可能用於標準設定的其他參數，指定預設值。 您可以將範本儲存在 Azure 儲存體帳戶中，以在組織內共用存取。 如需有關使用範本的詳細資訊，請參閱[使用 ARM 範本和 Azure CLI 部署資源](../../azure-resource-manager/templates/deploy-cli.md)。

如果您是 Azure 自動化和 Azure 監視器的新手，請務必了解下列設定詳細資料。 當您嘗試建立、設定和使用連結至新自動化帳戶的 Log Analytics 工作區時，這些詳細資料可以協助您避免錯誤。

* 請檢閱[其他詳細資料](../../azure-monitor/samples/resource-manager-workspace.md#create-a-log-analytics-workspace)，以全面了解工作區設定選項，例如，存取控制模式、定價層、保留和容量保留層級。

* 請參閱[工作區對應](../how-to/region-mappings.md)，以指定內嵌或參數檔案中支援的區域。 只有特定區域支援連結 Log Analytics 工作區以及訂用帳戶中的自動化帳戶。

* 如果您是 Azure 監視器記錄的新手，而且尚未部署工作區，您應該參閱[工作區設計指引](../../azure-monitor/platform/design-logs-deployment.md)。 此指引將協助您了解存取控制，並了解我們針對貴組織建議的設計實作策略。

## <a name="deploy-template"></a>部署範本

1. 複製以下 JSON 語法並貼到您的檔案中：

    ```json
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "workspaceName": {
                "type": "string",
                "metadata": {
                    "description": "Workspace name"
                }
            },
            "sku": {
                "type": "string",
                "allowedValues": [
                    "pergb2018",
                    "Free",
                    "Standalone",
                    "PerNode",
                    "Standard",
                    "Premium"
                ],
                "defaultValue": "pergb2018",
                "metadata": {
                    "description": "Pricing tier: perGB2018 or legacy tiers (Free, Standalone, PerNode, Standard or Premium), which are not available to all customers."
                }
            },
            "dataRetention": {
                "type": "int",
                "defaultValue": 30,
                "minValue": 7,
                "maxValue": 730,
                "metadata": {
                    "description": "Number of days to retain data."
                }
            },
            "location": {
                "type": "string",
                "defaultValue": "[resourceGroup().location]",
                "metadata": {
                    "description": "Specifies the location in which to create the workspace."
                }
            },
            "automationAccountName": {
                "type": "string",
                "metadata": {
                    "description": "Automation account name"
                }
            },
            "automationAccountLocation": {
                "type": "string",
                "metadata": {
                    "description": "Specifies the location in which to create the Automation account."
                }
            },
            "sampleGraphicalRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorial"
            },
            "sampleGraphicalRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePowerShellRunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialScript"
            },
            "samplePowerShellRunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "samplePython2RunbookName": {
                "type": "String",
                "defaultValue": "AzureAutomationTutorialPython2"
            },
            "samplePython2RunbookDescription": {
                "type": "String",
                "defaultValue": " An example runbook that gets all the Resource Manager resources by using the Run As account (service principal)."
            },
            "_artifactsLocation": {
                "type": "string",
                "defaultValue": "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-automation/",
                "metadata": {
                    "description": "URI to artifacts location"
                }
            },
            "_artifactsLocationSasToken": {
                "type": "securestring",
                "defaultValue": "",
                "metadata": {
                    "description": "The sasToken required to access _artifactsLocation.  When the template is deployed using the accompanying scripts, a sasToken will be automatically generated"
                }
            }
        },
        "variables": {
        "Updates": {
            "name": "[concat('Updates', '(', parameters('workspaceName'), ')')]",
            "galleryName": "Updates"
          }
        },
        "resources": [
            {
                "type": "Microsoft.OperationalInsights/workspaces",
                "apiVersion": "2020-03-01-preview",
                "name": "[parameters('workspaceName')]",
                "location": "[parameters('location')]",
                "properties": {
                    "sku": {
                        "name": "[parameters('sku')]"
                    },
                    "retentionInDays": "[parameters('dataRetention')]",
                    "features": {
                        "searchVersion": 1,
                        "legacy": 0
                    }
                }
            },
            {
                "apiVersion": "2015-11-01-preview",
                "location": "[parameters('location')]",
                "name": "[variables('Updates').name]",
                "type": "Microsoft.OperationsManagement/solutions",
                "id": "[concat('/subscriptions/', subscription().subscriptionId, '/resourceGroups/', resourceGroup().name, '/providers/Microsoft.OperationsManagement/solutions/', variables('Updates').name)]",
                "dependsOn": [
                    "[concat('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                ],
                "properties": {
                    "workspaceResourceId": "[resourceId('Microsoft.OperationalInsights/workspaces/', parameters('workspaceName'))]"
                },
                "plan": {
                    "name": "[variables('Updates').name]",
                    "publisher": "Microsoft",
                    "promotionCode": "",
                    "product": "[concat('OMSGallery/', variables('Updates').galleryName)]"
                }
            },
            {
                "type": "Microsoft.Automation/automationAccounts",
                "apiVersion": "2020-01-13-preview",
                "name": "[parameters('automationAccountName')]",
                "location": "[parameters('automationAccountLocation')]",
                "dependsOn": [
                    "[parameters('workspaceName')]"
                ],
                "properties": {
                    "sku": {
                        "name": "Basic"
                    }
                },
                "resources": [
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('sampleGraphicalRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "GraphPowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('sampleGraphicalRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.graphrunbook', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePowerShellRunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "PowerShell",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePowerShellRunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorial.ps1', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    },
                    {
                        "type": "runbooks",
                        "apiVersion": "2018-06-30",
                        "name": "[parameters('samplePython2RunbookName')]",
                        "location": "[parameters('automationAccountLocation')]",
                        "dependsOn": [
                            "[parameters('automationAccountName')]"
                        ],
                        "properties": {
                            "runbookType": "Python2",
                            "logProgress": "false",
                            "logVerbose": "false",
                            "description": "[parameters('samplePython2RunbookDescription')]",
                            "publishContentLink": {
                                "uri": "[uri(parameters('_artifactsLocation'), concat('scripts/AzureAutomationTutorialPython2.py', parameters('_artifactsLocationSasToken')))]",
                                "version": "1.0.0.0"
                            }
                        }
                    }
                ]
            },
            {
                "type": "Microsoft.OperationalInsights/workspaces/linkedServices",
                "apiVersion": "2020-03-01-preview",
                "name": "[concat(parameters('workspaceName'), '/' , 'Automation')]",
                "location": "[parameters('location')]",
                "dependsOn": [
                    "[parameters('workspaceName')]",
                    "[parameters('automationAccountName')]"
                ],
                "properties": {
                    "resourceId": "[resourceId('Microsoft.Automation/automationAccounts', parameters('automationAccountName'))]"
                }
            }
        ]
    }
    ```

2. 編輯範本以符合您的需求。 請考慮建立 [Resource Manager 參數檔案](../../azure-resource-manager/templates/parameter-files.md)，而不是以內嵌值傳遞參數。

3. 將此檔案儲存到本機資料夾，例如 **deployUMSolutiontemplate.json**。

4. 您已準備好部署此範本。 您可以使用 PowerShell 或 Azure CLI。 當系統提示您輸入工作區和自動化帳戶名稱時，請提供在所有 Azure 訂用帳戶全域中都是唯一的名稱。

    **PowerShell**

    ```powershell
    New-AzResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateFile deployUMSolutiontemplate.json
    ```

    **Azure CLI**

    ```azurecli
    az deployment group create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deployUMSolutiontemplate.json
    ```

    部署需要幾分鐘的時間才能完成。 完成後，您會看到類似下列包含結果的訊息：

    ![部署完成時的範例結果](media/enable-from-template/template-output.png)

## <a name="review-deployed-resources"></a>檢閱已部署的資源

1. 登入 [Azure 入口網站](https://portal.azure.com)。

2. 在 Azure 入口網站中，開啟您所建立的自動化帳戶。

3. 從左窗格中，選取 [Runbook]。 在 [Runbook] 頁面上，列出三個使用自動化帳戶建立的教學課程 Runbook。

    ![使用自動化帳戶建立的教學課程 Runbook](../media/quickstart-create-automation-account-template/automation-sample-runbooks.png)

4. 從左窗格中，選取 [連結的工作區]。 在 [連結的工作區] 頁面上，其會顯示您稍早指定連結至自動化帳戶的 Log Analytics 工作區。

    ![連結至 Log Analytics 工作區的自動化帳戶](../media/quickstart-create-automation-account-template/automation-account-linked-workspace.png)

5. 從左側窗格中選取 [ **更新管理**]。 在 [ **更新管理** ] 頁面上，它會顯示 [評量] 頁面，而不會顯示任何資訊，因為只會啟用，且不會設定電腦進行管理。

    ![更新管理功能評估視圖](./media/enable-from-template/update-management-assessment-view.png)

## <a name="clean-up-resources"></a>清除資源

當您不再需要它們時，請刪除 Log Analytics 工作區中的 **更新** 解決方案，取消連結工作區中的自動化帳戶，然後刪除自動化帳戶和工作區。

## <a name="next-steps"></a>後續步驟

* 若要使用 Vm 的更新管理，請參閱 [管理 vm 的更新和修補程式](manage-updates-for-vm.md)。

* 如果您不想再使用更新管理而且想要將它移除，請參閱 [移除更新管理功能](remove-feature.md)中的指示。

* 若要從更新管理中刪除 VM，請參閱[從更新管理中移除 VM](remove-vms.md)。