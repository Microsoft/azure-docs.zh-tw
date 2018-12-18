---
title: 如何設定 Web 應用程式的 TLS 相互驗證
description: 了解如何設定 Web 應用程式在 TLS 上使用用戶端憑證驗證。
services: app-service
documentationcenter: ''
author: naziml
manager: erikre
editor: jimbe
ms.assetid: cd1d15d3-2d9e-4502-9f11-a306dac4453a
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/08/2016
ms.author: naziml
ms.openlocfilehash: 894a77be05de131ab122f18c62d209e9829357f9
ms.sourcegitcommit: 7208bfe8878f83d5ec92e54e2f1222ffd41bf931
ms.translationtype: HT
ms.contentlocale: zh-TW
ms.lasthandoff: 07/14/2018
ms.locfileid: "39056203"
---
# <a name="how-to-configure-tls-mutual-authentication-for-web-app"></a>如何設定 Web 應用程式的 TLS 相互驗證
## <a name="overview"></a>概觀
為 Azure Web 應用程式啟用不同類型的驗證，即可限制其存取。 這樣做的其中一種方法是要求透過 TLS/SSL 時使用用戶端憑證進行驗證。 這項機制稱為 TLS 相互驗證或用戶端憑證驗證，本文將詳細說明如何設定 Web 應用程式使用用戶端憑證驗證。

> **附註：** 如果您透過 HTTP 存取您的網站，而非 HTTPS，將不會收到任何用戶端憑證。 因此如果您的應用程式需要用戶端憑證，請勿允許透過 HTTP 傳入您應用程式的要求。
> 
> 

[!INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="configure-web-app-for-client-certificate-authentication"></a>設定 Web 應用程式進行用戶端憑證驗證
若要設定 Web 應用程式要求用戶端憑證，則您需要為 Web 應用程式新增 clientCertEnabled 網站設定，並將它設定為 true。 此設定也可以設定於 Azure 入口網站的 [SSL 憑證] 刀鋒視窗中。

您可以使用 [ARMClient 工具](https://github.com/projectkudu/ARMClient) ，輕鬆地製作 REST API 呼叫。 使用此工具登入之後，需要發出下列命令：

    ARMClient PUT subscriptions/{Subscription Id}/resourcegroups/{Resource Group Name}/providers/Microsoft.Web/sites/{Website Name}?api-version=2015-04-01 @enableclientcert.json -verbose

將 {} 中的所有內容取代為您 Web 應用程式的資訊，並使用下列 JSON 內容建立稱為 enableclientcert.json 的檔案：

    {
        "location": "My Web App Location",
        "properties": {
            "clientCertEnabled": true
        }
    }

請務必將 "location" 的值變更為您 Web 應用程式所在的位置 (例如美國中北部或美國西部等)。

您也可以使用 https://resources.azure.com，將 `clientCertEnabled` 屬性翻轉為 `true`。

> **附註︰** 如果您從 Powershell 執行 ARMClient，您必須使用重音符 ` 為 JSON 檔案逸出 \@ 符號。
> 
> 

## <a name="accessing-the-client-certificate-from-your-web-app"></a>從 Web 應用程式存取用戶端憑證
如果您使用 ASP.NET，並將您的應用程式設定為使用用戶端憑證，便可透過 **HttpRequest.ClientCertificate** 屬性取得憑證。 若為其他應用程式堆疊，您則可透過「X-ARR-ClientCert」要求標頭中的 base64 編碼值取得用戶端憑證。 您的應用程式可以從這個值建立憑證，然後將它用於您應用程式中的驗證和授權用途。

## <a name="special-considerations-for-certificate-validation"></a>憑證驗證的特殊考量
Azure Web 應用程式平台不會對傳送給應用程式的用戶端憑證進行任何驗證。 驗證此憑證是 Web 應用程式的責任。 以下是基於驗證而驗證憑證內容的範例 ASP.NET 程式碼。

    using System;
    using System.Collections.Specialized;
    using System.Security.Cryptography.X509Certificates;
    using System.Web;

    namespace ClientCertificateUsageSample
    {
        public partial class cert : System.Web.UI.Page
        {
            public string certHeader = "";
            public string errorString = "";
            private X509Certificate2 certificate = null;
            public string certThumbprint = "";
            public string certSubject = "";
            public string certIssuer = "";
            public string certSignatureAlg = "";
            public string certIssueDate = "";
            public string certExpiryDate = "";
            public bool isValidCert = false;

            //
            // Read the certificate from the header into an X509Certificate2 object
            // Display properties of the certificate on the page
            //
            protected void Page_Load(object sender, EventArgs e)
            {
                NameValueCollection headers = base.Request.Headers;
                certHeader = headers["X-ARR-ClientCert"];
                if (!String.IsNullOrEmpty(certHeader))
                {
                    try
                    {
                        byte[] clientCertBytes = Convert.FromBase64String(certHeader);
                        certificate = new X509Certificate2(clientCertBytes);
                        certSubject = certificate.Subject;
                        certIssuer = certificate.Issuer;
                        certThumbprint = certificate.Thumbprint;
                        certSignatureAlg = certificate.SignatureAlgorithm.FriendlyName;
                        certIssueDate = certificate.NotBefore.ToShortDateString() + " " + certificate.NotBefore.ToShortTimeString();
                        certExpiryDate = certificate.NotAfter.ToShortDateString() + " " + certificate.NotAfter.ToShortTimeString();
                    }
                    catch (Exception ex)
                    {
                        errorString = ex.ToString();
                    }
                    finally 
                    {
                        isValidCert = IsValidClientCertificate();
                        if (!isValidCert) Response.StatusCode = 403;
                        else Response.StatusCode = 200;
                    }
                }
                else
                {
                    certHeader = "";
                }
            }

            //
            // This is a SAMPLE verification routine. Depending on your application logic and security requirements, 
            // you should modify this method
            //
            private bool IsValidClientCertificate()
            {
                // In this example we will only accept the certificate as a valid certificate if all the conditions below are met:
                // 1. The certificate is not expired and is active for the current time on server.
                // 2. The subject name of the certificate has the common name nildevecc
                // 3. The issuer name of the certificate has the common name nildevecc and organization name Microsoft Corp
                // 4. The thumbprint of the certificate is 30757A2E831977D8BD9C8496E4C99AB26CB9622B
                //
                // This example does NOT test that this certificate is chained to a Trusted Root Authority (or revoked) on the server 
                // and it allows for self signed certificates
                //

                if (certificate == null || !String.IsNullOrEmpty(errorString)) return false;

                // 1. Check time validity of certificate
                if (DateTime.Compare(DateTime.Now, certificate.NotBefore) < 0 || DateTime.Compare(DateTime.Now, certificate.NotAfter) > 0) return false;

                // 2. Check subject name of certificate
                bool foundSubject = false;
                string[] certSubjectData = certificate.Subject.Split(new char[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
                foreach (string s in certSubjectData)
                {
                    if (String.Compare(s.Trim(), "CN=nildevecc") == 0)
                    {
                        foundSubject = true;
                        break;
                    }
                }
                if (!foundSubject) return false;

                // 3. Check issuer name of certificate
                bool foundIssuerCN = false, foundIssuerO = false;
                string[] certIssuerData = certificate.Issuer.Split(new char[] { ',' }, StringSplitOptions.RemoveEmptyEntries);
                foreach (string s in certIssuerData)
                {
                    if (String.Compare(s.Trim(), "CN=nildevecc") == 0)
                    {
                        foundIssuerCN = true;
                        if (foundIssuerO) break;
                    }

                    if (String.Compare(s.Trim(), "O=Microsoft Corp") == 0)
                    {
                        foundIssuerO = true;
                        if (foundIssuerCN) break;
                    }
                }

                if (!foundIssuerCN || !foundIssuerO) return false;

                // 4. Check thumprint of certificate
                if (String.Compare(certificate.Thumbprint.Trim().ToUpper(), "30757A2E831977D8BD9C8496E4C99AB26CB9622B") != 0) return false;

                // If you also want to test if the certificate chains to a Trusted Root Authority you can uncomment the code below
                //
                //X509Chain certChain = new X509Chain();
                //certChain.Build(certificate);
                //bool isValidCertChain = true;
                //foreach (X509ChainElement chElement in certChain.ChainElements)
                //{
                //    if (!chElement.Certificate.Verify())
                //    {
                //        isValidCertChain = false;
                //        break;
                //    }
                //}
                //if (!isValidCertChain) return false;

                return true;
            }
        }
    }
