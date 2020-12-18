# [104.59] 使用应用程序网关对 Web 服务流量进行负载均衡
  - [应用程序网关配置概述](https://docs.microsoft.com/zh-cn/azure/application-gateway/configuration-overview)
  - [应用程序网关的工作原理](https://docs.microsoft.com/zh-cn/azure/application-gateway/how-application-gateway-works)
  - 练习 - 创建网站
    ```bash
     rg=rg0902
     az group create --name $rg --location japaneast

     # create vnet vehicleAppVnet
     az network vnet create \
        --resource-group $rg \
        --name vehicleAppVnet \
        --address-prefix 10.0.0.0/16 \
        --subnet-name webServerSubnet \
        --subnet-prefix 10.0.1.0/24
     # create vm
     git clone https://github.com/MicrosoftDocs/mslearn-load-balance-web-traffic-with-application-gateway module-files

     az vm create \
        --resource-group $rg \
        --name webServer1 \
        --image UbuntuLTS \
        --admin-username azureuser \
        --generate-ssh-keys \
        --vnet-name vehicleAppVnet \
        --subnet webServerSubnet \
        --public-ip-address "" \
        --nsg "" \
        --custom-data module-files/scripts/vmconfig.sh \
        --no-wait

     az vm create \
        --resource-group $rg \
        --name webServer2 \
        --image UbuntuLTS \
        --admin-username azureuser \
        --generate-ssh-keys \
        --vnet-name vehicleAppVnet \
        --subnet webServerSubnet \
        --public-ip-address "" \
        --nsg "" \
        --custom-data module-files/scripts/vmconfig.sh
     
     az vm list \
        --resource-group $rg \
        --show-details \
        --output table
     # 创建应用服务和部署驾照更新站点
     az appservice plan create \
        --resource-group $rg \
        --name vehicleAppServicePlan \
        --sku S1
        
     az webapp create \
        --resource-group $rg \
        --name $APPSERVICE \
        --plan vehicleAppServicePlan \
        --deployment-source-url https://github.com/MicrosoftDocs/mslearn-load-balance-web-traffic-with-application-gateway \
        --deployment-source-branch appService
    ```
  - 练习 - 创建和配置应用程序网关 
    ```bash
     # 为应用程序网关配置网络 appGatewaySubnet
     az network vnet subnet create \
        --resource-group $rg \
        --vnet-name vehicleAppVnet  \
        --name appGatewaySubnet \
        --address-prefixes 10.0.0.0/24
     az network public-ip create \
        --resource-group $rg \
        --name appGatewayPublicIp \
        --sku Standard \
        --dns-name vehicleapp${RANDOM}
     
     # 创建应用程序网关
     az network application-gateway create \
        --resource-group $rg \
        --name vehicleAppGateway \
        --sku WAF_v2 \    ★
        --capacity 2 \　　★
        --vnet-name vehicleAppVnet \
        --subnet appGatewaySubnet \
        --public-ip-address appGatewayPublicIp \
        --http-settings-protocol Http \
        --http-settings-port 8080 \
        --frontend-port 8080
    　# 运行以下命令，查找 webServer1 和 webServer2 的专用 IP 地址。 我们将这些地址保存为变量，以便在下一个命令中使用
    　WEBSERVER1IP="$(az vm list-ip-addresses \
        --resource-group $rg \
        --name webServer1 \
        --query [0].virtualMachine.network.privateIpAddresses[0] \
        --output tsv)"

    　WEBSERVER2IP="$(az vm list-ip-addresses \
        --resource-group $rg \
        --name webserver2 \
        --query [0].virtualMachine.network.privateIpAddresses[0] \
        --output tsv)"
     # 为每个网站添加后端池。 首先，为在虚拟机上运行的车辆登记站点创建后端池。 我们将使用上一个命令中每个 VM 的 IP 地址变量。
     az network application-gateway address-pool create \
        --gateway-name vehicleAppGateway \
        --resource-group $rg \
        --name vmPool \
        --servers $WEBSERVER1IP $WEBSERVER2IP

     # 为在应用服务上运行的驾照更新站点创建后端池。
     az network application-gateway address-pool create \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name appServicePool \
        --servers $APPSERVICE.azurewebsites.net
     # 为端口 80 创建一个前端端口。
     az network application-gateway frontend-port create \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name port80 \
        --port 80
     # 创建用于在端口 80 上处理请求的侦听器。
     az network application-gateway http-listener create \
        --resource-group $rg \
        --name vehicleListener \
        --frontend-port port80 \
        --gateway-name vehicleAppGateway
     # ==添加运行状况探测
     # 创建测试 Web 服务器可用性的运行状况探测
     az network application-gateway probe create \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name customProbe \
        --path / \
        --interval 15 \
        --threshold 3 \
        --timeout 10 \
        --protocol Http \
        --host-name-from-http-settings true
     # 为网关创建 HTTP 设置以使用我们创建的运行状况探测。
     az network application-gateway http-settings update \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name appGatewayBackendHttpSettings \
        --host-name-from-backend-pool true \
        --port 80 \
        --probe customProbe
     # == 配置基于路径的路由
     # 为“vmPool”创建路径映射。 *create urlPathMap and first rule for vmPool
     az network application-gateway url-path-map create \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name urlPathMap \
        --paths /VehicleRegistration/* \
        --http-settings appGatewayBackendHttpSettings \
        --address-pool vmPool
     # 为“appServicePool”创建路径映射( path map rule)。 * add a rule to urlPathMap for appServicePool
     az network application-gateway url-path-map rule create \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name appServiceUrlPathMap \
        --paths /LicenseRenewal/* \
        --http-settings appGatewayBackendHttpSettings \
        --address-pool appServicePool \
        --path-map-name urlPathMap
     # 使用我们创建的路径映射创建新的路由规则。
     az network application-gateway rule create \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name appServiceRule \
        --http-listener vehicleListener \
        --rule-type PathBasedRouting \
        --address-pool appServicePool \
        --url-path-map urlPathMap
     # 配置的最后一部分是删除我们最初部署应用程序网关时创建的规则。 有了我们的自定义规则，我们就不再需要它了。
     az network application-gateway rule delete \
        --resource-group $rg \
        --gateway-name vehicleAppGateway \
        --name rule1
    ```
  - 练习 - 测试应用程序网关
    ```bash
     # ==测试车辆登记 Web 应用的负载均衡
     # 在 Cloud Shell 中运行以下命令，生成应用程序网关的根 URL。
     echo http://$(az network public-ip show \
        --resource-group $rg \
        --name appGatewayPublicIp \
        --query dnsSettings.fqdn \
        --output tsv)
     http://vehicleapp31777.japaneast.cloudapp.azure.com/
     # 在 Cloud Shell 中运行以下命令，停止“webServer1”的虚拟机并解除分配：
     az vm deallocate \
        --resource-group $rg \
        --name webServer1
     # 重启“webServer1”实例：
     az vm start \
        --resource-group $rg \
        --name webServer1
     # ==测试基于 URL 路径的路由
     # 访问 http://<vehicleAppGateway>/LicenseRenewal/Create。 你应转到在应用服务上运行的驾照更新页面。 如果 URL 中有“/LicenseRenewal/”，则会路由到运行驾照更新站点的“appServicePool”。
     http://vehicleapp31777.japaneast.cloudapp.azure.com/LicenseRenewal/Create
     #
     #
    ```
# 检查注册状态, 注册
  - 若要检查注册状态，请使用以下命令：  
    az provider show  
    for ex.
    ```
     az provider show -n Microsoft.OperationsManagement -o table
     az provider show -n Microsoft.OperationalInsights -o table
    ```
  - 使用以下命令注册  
    az provide register
    for ex.
    ```
     az provider register --namespace Microsoft.OperationsManagement
     az provider register --namespace Microsoft.OperationalInsights
    ```
    