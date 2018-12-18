---
title: 適用於 Azure Stack 整合式系統的 Azure 連線部署決策 | Microsoft Docs
description: 針對多節點 Azure Stack 的 Azure 連線部署，決定部署規劃決策。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/01/2018
ms.author: jeffgilb
ms.reviewer: wfayed
ms.openlocfilehash: d64b834f1c6794976461c93d4ad1d05f8647e986
ms.sourcegitcommit: 96f498de91984321614f09d796ca88887c4bd2fb
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 08/02/2018
ms.locfileid: "39414584"
---
# <a name="azure-connected-deployment-planning-decisions-for-azure-stack-integrated-systems"></a>適用於 Azure Stack 整合式系統的 Azure 連線部署規劃決策
在您決定[如何將 Azure Stack 整合到您的混合式雲端環境](azure-stack-connection-models.md)之後，即可接著完成您的 Azure Stack 部署決策。

部署已連線到 Azure 的 Azure Stack，表示您可使用 Azure Active Directory (Azure AD) 或 Active Directory 同盟服務 (AD FS) 進行身分識別儲存。 您也可以選擇任一種計費模型：使用時付費或以容量為基礎。 已連線的部署是預設選項，因為可讓客戶充分發揮 Azure Stack 的價值，特別適用於牽涉 Azure 與 Azure Stack 的混合式雲端案例。 

## <a name="choose-an-identity-store"></a>選擇身分識別儲存
採用已連線的部署，您可以選擇 Azure AD 或 AD FS 作為您的身分識別儲存。 已中斷連線的部署 (沒有網際網路連線能力) 只能使用 AD FS。

您的身分識別儲存選擇對於租用戶虛擬機器 (VM) 毫無影響。 租用戶 VM 可以根據其設定方式，選擇想要連線的身分識別儲存：Azure AD、已加入網域的 Windows Server Active Directory、工作群組等。這與決定 Azure Stack 識別提供者無關。 

例如，如果您在 Azure Stack 之上部署 IaaS 租用戶 VM，而且想要其加入公司的 Active Directory 網域並從該處使用帳戶，您仍可這麼做。 您不需要對這些帳戶使用您在此選取的 Azure AD 身分識別儲存。

### <a name="azure-ad-identity-store"></a>Azure AD 身分識別儲存
當您使用 Azure AD 作為身分識別儲存時，您需要兩個 Azure AD 的帳戶：全域管理員帳戶和計費帳戶。 這些帳戶可以是相同的帳戶，也可以是不同的帳戶。 如果您的 Azure 帳戶數目有限，使用相同的使用者帳戶雖然比較簡單而且實用，但您的業務需求可能會建議使用兩個帳戶：

1. **全域管理員帳戶** (只有已連線的部署需要)。 這是一個 Azure 帳戶，用於為 Azure Active Directory 中的 Azure Stack 基礎結構服務建立應用程式和服務主體。 此帳戶必須具備要在其中部署 Azure Stack 系統之目錄的目錄管理員權限。 它會變成 Azure AD 租用戶的「雲端操作員」全域管理員，且用途如下： 
    - 用來為需要與 Azure Active Directory 和「圖形 API」互動的所有 Azure Stack 服務佈建及委派應用程式和服務主體。 
    - 作為服務管理員帳戶。 這是預設提供者訂用帳戶 (您可以稍後變更此訂用帳戶) 的擁有者。 您可以使用此帳戶登入 Azure Stack 系統管理入口網站，並可將它用於建立供應項目和方案、設定配額，以及在 Azure Stack 中執行其他管理功能。
2. **計費帳戶** (已連線和已中斷連線的部署都需要)。 此 Azure 帳戶用來建立您的 Azure Stack 整合式系統和 Azure 商務後端之間的計費關係。 這是要支付 Azure Stack 費用的帳戶。 此帳戶也會用於 Marketplace 中的供應項目和其他混合式案例。 

### <a name="ad-fs-identity-store"></a>AD FS 身分識別存放區
如果您要將自己的身分識別儲存 (例如公司的 Active Directory) 用於您的服務管理員帳戶，請選擇此選項。  

## <a name="choose-a-billing-model"></a>選擇計費模型
您可以選擇**使用時付費**或**容量**計費模型。 使用時付費計費模型部署必須能夠至少每 30 天與 Azure 連線一次以回報使用量。 因此使用時付費計費模型僅適用於已連線部署。  

### <a name="pay-as-you-use"></a>使用時付費
若採用使用時付費計費模型，則會向 Azure 訂用帳戶收取使用量的費用。 只有在您使用 Azure Stack 服務時，才需要付費。 如果這是您決定的模型，您就需要 Azure 訂用帳戶以及與該訂用帳戶相關聯的帳戶識別碼 (例如 serviceadmin@contoso.onmicrosoft.com)。 支援 EA、CSP 和 CSL 訂用帳戶。 使用量報告設定於 [Azure Stack 註冊](azure-stack-registration.md)期間。

> [!NOTE]
> 在大部分情況下，企業客戶會使用 EA 訂用帳戶，而服務提供者會使用 CSP 或 CSL 訂用帳戶。

如果您即將使用 CSP 訂用帳戶，請檢閱下表來識別要使用哪個 CSP 訂用帳戶，而正確的方法取決於確切的 CSP 案例：

|案例|網域和訂用帳戶選項|
|-----|-----|
|您是**直接 CSP 夥伴**或**間接 CSP 夥伴**，並且將會操作 Azure Stack|使用 CSL (常見服務層) 訂用帳戶。<br>     或<br>在「合作夥伴中心」中以描述性名稱建立 Azure AD 租用戶。 例如 &lt;您的組織>CSPAdmin，並將 Azure CSP 訂用帳戶與其建立關聯。|
|您是**間接 CSP 轉銷商**，並且將會操作 Azure Stack|要求您的「間接 CSP 提供者」使用「合作夥伴中心」為您的組織建立一個 Azure AD 租用戶，並將 Azure AD 訂用帳戶與其建立關聯。|

### <a name="capacity-based-billing"></a>容量型計費
如果您決定使用容量計費模型，您必須根據您的系統容量，購買 Azure Stack 容量方案 SKU。 您必須知道您 Azure Stack 中的實體核心數目，以購買正確的數量。 

容量計費需有 Enterprise 合約 (EA) Azure 訂用帳戶才能註冊。 原因是註冊會在 Marketplace 中設定項目可用性，而這需要 Azure 訂用帳戶。 此訂用帳戶不會計入 Azure Stack 使用量。

## <a name="learn-more"></a>深入了解
- 如需使用案例、購買、合作夥伴和 OEM 硬體廠商的詳細資訊，請參閱 [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) 產品頁面。
- 如需 Azure Stack 整合系統的藍圖和地理可用性的詳細資訊，請參閱白皮書：[Azure Stack：Azure 的延伸模組](https://azure.microsoft.com/resources/azure-stack-an-extension-of-azure/)。 
- 若要深入了解 Microsoft Azure Stack 套件和定價，請[下載此 .pdf](https://azure.microsoft.com/mediahandler/files/resourcefiles/5bc3f30c-cd57-4513-989e-056325eb95e1/Azure-Stack-packaging-and-pricing-datasheet.pdf)。 

## <a name="next-steps"></a>後續步驟
[資料中心網路整合](azure-stack-network.md)