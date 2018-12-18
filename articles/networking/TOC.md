# 概觀
## [關於 Azure 網路](networking-overview.md)
## 架構
### [虛擬資料中心](/azure/architecture/vdc/networking-virtual-datacenter)
### [非對稱式路由與多個網路路徑](../expressroute/expressroute-asymmetric-routing.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [保護網路設計](../best-practices-network-security.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [中樞輪輻拓撲](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke)
### [網路安全性的最佳做法](../security/azure-security-network-security-best-practices.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [高可用性的網路虛擬設備](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha )
### [合併負載平衡方法](../traffic-manager/traffic-manager-load-balancing-azure.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [使用 Azure DNS 和流量管理員進行災害復原](disaster-recovery-dns-traffic-manager.md)
## 規劃和設計
### [虛擬網路](../virtual-network/virtual-network-vnet-plan-design-arm.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [跨單位連線 - VPN](../vpn-gateway/vpn-gateway-plan-design.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [跨單位連線 - 專用私人](../expressroute/expressroute-workflows.md?toc=%2fazure%2fnetworking%2ftoc.json)
### 後端連線能力互通性
#### [引用和測試設定](connectivty-interoperability-preface.md?toc=%2fazure%2fnetworking%2ftoc.json)
#### [測試設定組態](connectivty-interoperability-configuration.md?toc=%2fazure%2fnetworking%2ftoc.json)
#### [控制項平面分析](connectivty-interoperability-control-plane.md?toc=%2fazure%2fnetworking%2ftoc.json)
#### [資料平面分析](connectivty-interoperability-data-plane.md?toc=%2fazure%2fnetworking%2ftoc.json)

##  概念
### [虛擬網路](../virtual-network/virtual-networks-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [網路負載平衡](../load-balancer/load-balancer-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [應用程式負載平衡](../application-gateway/overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [DNS](../dns/dns-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [DNS 流量散發](../traffic-manager/traffic-manager-overview.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [連接內部部署 - VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [連線內部部署 - 專用](../expressroute/expressroute-introduction.md?toc=%2fazure%2fnetworking%2ftoc.json)


# 開始使用
## [建立您的第一個虛擬網路](../virtual-network/quick-create-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)

# 作法
## 網際網路連線
### [網路負載平衡公用伺服器](../load-balancer/load-balancer-get-started-internet-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [應用程式負載平衡公用伺服器](../application-gateway/application-gateway-create-gateway-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [保護 Web 應用程式](../application-gateway/application-gateway-web-application-firewall-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [將流量分散到位置](../traffic-manager/traffic-manager-configure-geographic-routing-method.md?toc=%2fazure%2fnetworking%2ftoc.json)
## 內部連線能力
### [網路負載平衡器私用伺服器](../load-balancer/load-balancer-get-started-ilb-arm-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [應用程式負載平衡器私用伺服器](../application-gateway/application-gateway-ilb-arm.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [連線虛擬網路 (相同位置)](../virtual-network/virtual-networks-create-vnetpeering-arm-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [連線虛擬網路 (不同位置)](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
## 跨單位連線
### [建立 S2S VPN 連線 (IPsec/IKE)](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [建立 P2S VPN 連線 (SSTP 與憑證)](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [建立專用私人連線 (ExpressRoute)](../expressroute/expressroute-howto-circuit-portal-resource-manager.md?toc=%2fazure%2fnetworking%2ftoc.json)

## 管理性
### [網路監視概觀](network-monitoring-overview.md)
### [檢查資源用量是否超出 Azure 限制](check-usage-against-limits.md)
### [檢視網路拓撲](../network-watcher/network-watcher-topology-powershell.md?toc=%2fazure%2fnetworking%2ftoc.json)

## 範例指令碼
### [Azure CLI](cli-samples.md)
### [Azure PowerShell](powershell-samples.md)

## 教學課程
### [負載平衡的 VM](../virtual-machines/linux/tutorial-load-balancer.md?toc=%2fazure%2fnetworking%2ftoc.json)
### [將電腦連線到虛擬網路](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fnetworking%2ftoc.json)

# 參考
## [Azure CLI](https://docs.microsoft.com/cli/azure/network)
## [Azure PowerShell](https://docs.microsoft.com/powershell/module/azurerm.network/?view=azurermps-3.8.0)
## [.Net](https://docs.microsoft.com/dotnet/api/microsoft.azure.management.network?view=azuremgmtnetwork-9.1.0-preview)
## [Node.js](https://azure.microsoft.com/develop/nodejs/#azure-sdk)
## [REST](https://msdn.microsoft.com/library/mt163658.aspx)

# 資源
## [作者範本](/azure/azure-resource-manager/resource-group-authoring-templates?toc=%2fazure%2fnetworking%2ftoc.json)
## [Azure 藍圖](https://azure.microsoft.com/roadmap/?category=networking)
## [社群範本](https://azure.microsoft.com/resources/templates/)
## [網路部落格](http://azure.microsoft.com/blog/topics/networking)
## [定價](https://azure.microsoft.com/pricing)
## [定價計算機](https://azure.microsoft.com/pricing/calculator/)
## [區域可用性](https://azure.microsoft.com/regions/services/)
## [Stack Overflow](http://stackoverflow.com/questions/tagged/azure-virtual-network)
## [影片](https://azure.microsoft.com/resources/videos/index/?services=virtual-network)

