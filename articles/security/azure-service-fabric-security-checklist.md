---
title: Azure Service Fabric 安全性檢查清單 | Microsoft Docs
description: 本文提供一組 Azure 網狀架構安全性的檢查清單。
services: security
documentationcenter: na
author: unifycloud
manager: mbaldwin
editor: tomsh
ms.assetid: ''
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/04/2017
ms.author: tomsh
ms.openlocfilehash: d0826d60d766874ddd186bcf406ea987134dab16
ms.sourcegitcommit: f20e43e436bfeafd333da75754cd32d405903b07
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 10/17/2018
ms.locfileid: "49385850"
---
# <a name="azure-service-fabric-security-checklist"></a>Azure Service Fabric 安全性檢查清單
本文提供便於使用的檢查清單，以協助您保護 Azure Service Fabric 環境。

## <a name="introduction"></a>簡介
Azure Service Fabric 是分散式系統平台，可讓您輕鬆封裝、部署及管理可調整和可信賴的微服務。 Service Fabric 也可解決開發與管理雲端應用程式時遭遇的重大挑戰。 開發人員與管理員能夠避免複雜的基礎結構問題，專注於實作關鍵且嚴格要求之可調整、可信賴且可管理的工作負載。

## <a name="checklist"></a>檢查清單
您可以使用下列檢查清單，確保未忽略安全 Azure Service Fabric 解決方案管理和設定中的任何重要問題。


|檢查清單類別| 說明 |
| ------------ | -------- |
|[角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security-roles) | <ul><li>存取控制可讓叢集系統管理員針對不同的使用者群組限制特定叢集作業的存取權，讓叢集更加安全。</li><li>系統管理員可以完整存取管理功能 (包括讀取/寫入功能)。 </li><li> 使用者預設只具有管理功能的讀取存取權 (例如查詢功能)，以及解析應用程式和服務的能力。</li></ul>|
|[X.509 憑證和 Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security) | <ul><li>執行生產環境工作負載之叢集中所使用的[憑證](https://docs.microsoft.com/dotnet/framework/wcf/feature-details/working-with-certificates)應該是使用已正確設定的 Windows Server 憑證服務來建立，或是從已核准的[憑證授權單位 (CA)](https://en.wikipedia.org/wiki/Certificate_authority) 取得。</li><li>絕對不要在生產環境中使用以 [MakeCert.exe](https://msdn.microsoft.com/library/windows/desktop/aa386968.aspx) 這類工具建立的[暫時或測試憑證](https://docs.microsoft.com/dotnet/framework/wcf/feature-details/how-to-create-temporary-certificates-for-use-during-development)。 </li><li>您可以使用[自我簽署憑證](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-x509-security)，但應該只用於測試叢集，不應該在生產環境中使用。</li></ul>|
|[叢集安全性](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security) | <ul><li>叢集安全性案例包含節點對節點安全性、用戶端對節點安全性和[角色型存取控制 (RBAC)](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-security-roles)。</li></ul>|
|[叢集驗證](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm) | <ul><li>驗證叢集同盟的[節點對節點通訊](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/service-fabric/service-fabric-cluster-security.md)。 </li></ul>|
|[伺服器驗證](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm) | <ul><li>驗證管理用戶端的[叢集管理端點](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-portal)。</li></ul>|
|[應用程式安全性](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-creation-via-arm)| <ul><li>將應用程式組態值加密和解密。</li><li>   在複寫期間將資料跨節點加密。</li></ul>|
|[叢集憑證](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-x509-security) | <ul><li>需有此憑證，才能保護叢集上節點之間的通訊。</li><li>    在 Thumbprint 區段中設定主要憑證的指紋，以及在 ThumbprintSecondary 變數中設定次要憑證的指紋。</li></ul>|
|[ServerCertificate](https://docs.microsoft.com/azure/service-fabric/service-fabric-windows-cluster-x509-security)| <ul><li>用戶端嘗試連線到此叢集時，會向用戶端此憑證顯示此憑證。 您可以使用兩個不同的伺服器憑證 (主要和次要) 進行更新。</li></ul>|
|ClientCertificateThumbprints| <ul><li>這是您想在經過驗證的用戶端上安裝的一組憑證。 </li></ul>|
|ClientCertificateCommonNames| <ul><li>針對 CertificateCommonName 設定第一個用戶端憑證的一般名稱。 CertificateIssuerThumbprint 是此憑證的簽發者指紋。 </li></ul>|
|ReverseProxyCertificate| <ul><li>如果您想要保護[反向 Proxy](https://docs.microsoft.com/azure/service-fabric/service-fabric-reverseproxy)，可以指定此選擇性憑證。 </li></ul>|
|Key Vault| <ul><li>用來管理 Azure 中 Service Fabric 叢集的憑證。  </li></ul>|


## <a name="next-steps"></a>後續步驟
- [Service Fabric 叢集升級程序與您的期望](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-upgrade)
- [在 Visual Studio 中管理 Service Fabric 應用程式](https://docs.microsoft.com/azure/service-fabric/service-fabric-manage-application-in-visual-studio)。
- [Service Fabric 健康狀態模型簡介](https://docs.microsoft.com/azure/service-fabric/service-fabric-health-introduction)。
