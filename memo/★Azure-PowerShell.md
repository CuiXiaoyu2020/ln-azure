- PowerShell 命令称为“cmdlet”（读成“command-let”）。

- Help  
  Get-Help Get-ChildItem -detailed

- 可以使用 Get-Module 命令获取已加载的模块列表  
  Get-Module

- get the command list  
  Get-Command -Name *AzSubscription*


- 安装 Az 模块  
  Install-Module Az -AllowClobber

- 更新模块  
  Update-Module -Name Az

- 加载 Az 的所有 cmdlet  
  Import-Module Az

## Azure
- login  
 Connect-AzAccount

- 指定サブスクリプション
  Select-AzSubscription -Subcription "Azure_Dingyue_01"

- get some information  
  Get-AzSubscription  
  Get-AzResourceGroup  
  Get-AzResourceGroup | Format-Table  
  Get-AzResource  
  Get-AzResource | ft
  * ft is the short write for Format-Table

1. 创建 Azure 虚拟机
    ```
    New-AzVm 
       -ResourceGroupName <resource group name> 
       -Name <machine name> 
       -Credential <credentials object> 
       -Location <location> 
       -Image <image name>
    ```
   1. New-AzVm -ResourceGroupName learn-6b03f398-9506-4c99-9394-2af5924059c4 -Name "testvm-eus-01" -Credential (Get-Credential) -Location "East US" -Image UbuntuLTS -OpenPorts 22
   ```
    PowerShell credential request
    Enter your credentials.
    User: bender
    Password for user bender: ***********

    ResourceGroupName        : learn-6b03f398-9506-4c99-9394-2af5924059c4
    Id                       : /subscriptions/6a6dcbd7-2059-4d69-bf27-265d60290fd6/resourceGroups/learn-6b03f398-9506-4c99-9394-2af5924059c4/providers/Microsoft.Compute/virtualMachines/testvm-eus-01
    VmId                     : 186e6678-08d0-4f61-abb2-28fd34f3a420
    Name                     : testvm-eus-01
    Type                     : Microsoft.Compute/virtualMachines
    Location                 : eastus
    Tags                     : {}
    HardwareProfile          : {VmSize}
    NetworkProfile           : {NetworkInterfaces}
    OSProfile                : {ComputerName, AdminUsername, LinuxConfiguration, Secrets, AllowExtensionOperations,RequireGuestProvisionSignal}
    ProvisioningState        : Succeeded
    StorageProfile           : {ImageReference, OsDisk, DataDisks}
    FullyQualifiedDomainName : testvm-eus-01-b5b67c.East US.cloudapp.azure.com
    ```
   2. $vm = Get-AzVM -Name "testvm-eus-01" -ResourceGroupName learn-6b03f398-9506-4c99-9394-2af5924059c4
    ```
    $vm.HardwareProfile
    $vm.StorageProfile.OsDisk
    $vm | Get-AzPublicIpAddress
    ```
   3. Stop-AzVM -Name $vm.Name -ResourceGroup $vm.ResourceGroupName
   4. Remove-AzVM -Name $vm.Name -ResourceGroup $vm.ResourceGroupName
   5. Some others  
      - Get-AzResource -ResourceName $vm.ResourceGroupName|ft  
      - 删除网络接口  
        $vm | Remove-AzNetworkInterface -Force  
      - 删除托管 OS 磁盘和存储帐户  
      Get-AzDisk -ResourceGroupName $vm.ResourceGroupName -DiskName $vm.StorageProfile.OSDisk.Name | Remove-AzDisk -Force  
      - 删除虚拟网络  
      Get-AzVirtualNetwork -ResourceGroup $vm.ResourceGroupName | Remove-AzVirtualNetwork -Force  
      - 删除网络安全组  
      Get-AzNetworkSecurityGroup -ResourceGroup $vm.ResourceGroupName | Remove-AzNetworkSecurityGroup -Force  
      - 最后，删除公共 IP 地址  
      Get-AzPublicIpAddress -ResourceGroup $vm.ResourceGroupName | Remove-AzPublicIpAddress -Force

2. create vm by script
   - script
    ``` powershell
    param([string]$resourceGroup)

    $adminCredential = Get-Credential -Message "Enter a username and password for the VM administrator."

    For ($i = 1; $i -le 3; $i++)
    {
        $vmName = "ConferenceDemo" + $i
        Write-Host "Creating VM: " $vmName
        New-AzVm -ResourceGroupName $resourceGroup -Name $vmName -Credential $adminCredential -Image UbuntuLTS
    }
    ```
   - Execute it  
     .\ConferenceDailyReset.ps1 learn-6b03f398-9506-4c99-9394-2af5924059c4
   - 確認
   ```
    PS /home/che_bender> Get-AzResource -ResourceType Microsoft.Compute/virtualMachines

    Name              : ConfernceDemo1
    ResourceGroupName : learn-6b03f398-9506-4c99-9394-2af5924059c4
    ResourceType      : Microsoft.Compute/virtualMachines
    Location          : westus
    ResourceId        : /subscriptions/6a6dcbd7-2059-4d69-bf27-265d60290fd6/resourceGroups/learn-6b03f398-9506-4c99-9394-2af5924059c4/providers/Microsoft.Compute/virtualMachines/ConfernceDemo1
    Tags              :

    Name              : ConfernceDemo2
    ResourceGroupName : learn-6b03f398-9506-4c99-9394-2af5924059c4
    ResourceType      : Microsoft.Compute/virtualMachines
    Location          : westus
    ResourceId        : /subscriptions/6a6dcbd7-2059-4d69-bf27-265d60290fd6/resourceGroups/learn-6b03f398-9506-4c99-9394-2af5924059c4/providers/Microsoft.Compute/virtualMachines/ConfernceDemo2
    Tags              :

    Name              : ConfernceDemo3
    ResourceGroupName : learn-6b03f398-9506-4c99-9394-2af5924059c4
    ResourceType      : Microsoft.Compute/virtualMachines
    Location          : westus
    ResourceId        : /subscriptions/6a6dcbd7-2059-4d69-bf27-265d60290fd6/resourceGroups/learn-6b03f398-9506-4c99-9394-2af5924059c4/providers/Microsoft.Compute/virtualMachines/ConfernceDemo3
    Tags              :
    ```

3. Azure Virtual Network
    1. Create virtual network  
    - create it
    ```powershell
    $Location = "EastUS"
    New-AZResourceGroup -Name vm-networks -Location $Location

    $subnet = New-AzVirtualNetworkSubnetConfig -Name default -AddressPrefix 10.0.0.0/24
    New-AzVirtualNetwork -Name myVnet -ResourceGroup vm-networks -Location $Location -AddressPrefix 10.0.0.0/16 -Subnet $subnet

    #delete it
    Remove-AzVirtualNetwork -name myVnet -ResourceGroupName vm-networks
    ```
    - a tip RDP command   
    ```
    mstsc
    ```
    2. Create Azure VPN Gateway
    - pre setting  
    ```
    $VNetName  = "VNetData"
    $FESubName = "FrontEnd"
    $BESubName = "Backend"
    $GWSubName = "GatewaySubnet"
    $VNetPrefix1 = "192.168.0.0/16"
    $VNetPrefix2 = "10.254.0.0/16"
    $FESubPrefix = "192.168.1.0/24"
    $BESubPrefix = "10.254.1.0/24"
    $GWSubPrefix = "192.168.200.0/26"
    $VPNClientAddressPool = "172.16.201.0/24"
    $ResourceGroup = "VpnGatewayDemo"
    $Location = "East US"
    $GWName = "VNetDataGW"
    $GWIPName = "VNetDataGWPIP"
    $GWIPconfName = "gwipconf"
    ```
    - setting virtual network
    ```powershell
    New-AzResourceGroup -Name $ResourceGroup -Location $Location

    $fesub = New-AzVirtualNetworkSubnetConfig -Name $FESubName -AddressPrefix $FESubPrefix
    $besub = New-AzVirtualNetworkSubnetConfig -Name $BESubName -AddressPrefix $BESubPrefix
    $gwsub = New-AzVirtualNetworkSubnetConfig -Name $GWSubName -AddressPrefix $GWSubPrefix

    New-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroup -Location $Location -AddressPrefix $VNetPrefix1,$VNetPrefix2 -Subnet $fesub, $besub, $gwsub -DnsServer 10.2.1.3

    $vnet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroup
    $subnet = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet

    $pip = New-AzPublicIpAddress -Name $GWIPName -ResourceGroupName $ResourceGroup -Location $Location -AllocationMethod Dynamic
    $ipconf = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName -Subnet $subnet -PublicIpAddress $pip
    ```
    - Create VPN Gateway (should cost 45min)
    ```powershell
    New-AzVirtualNetworkGateway -Name $GWName -ResourceGroupName $ResourceGroup `
        -Location $Location -IpConfigurations $ipconf -GatewayType Vpn `
        -VpnType RouteBased -EnableBgp $false -GatewaySku VpnGw1 -VpnClientProtocol "IKEv2"
    ```
    - add clinet pool
    ```powershell
    $Gateway = Get-AzVirtualNetworkGateway -ResourceGroupName $ResourceGroup -Name $GWName
    Set-AzVirtualNetworkGateway -VirtualNetworkGateway $Gateway -VpnClientAddressPool $VPNClientAddressPool
    ```
    - the rest is setting cert for clinet to azure gateway.

4.  Azure Virtual Network exec 104.52 (CLI)
   - create vnet and 3 subnet
      ```bash
        az network vnet create \
            --resource-group learn-b6ec6547-65cc-4897-97af-69210ebd9f7b \
            --name CoreServicesVnet \
            --address-prefix 10.20.0.0/16 \
            --location westus

        az network vnet subnet create \
            --resource-group learn-b6ec6547-65cc-4897-97af-69210ebd9f7b \
            --vnet-name CoreServicesVnet \
            --name GatewaySubnet \
            --address-prefix 10.20.0.0/27
            
        az network vnet subnet create \
            --resource-group learn-b6ec6547-65cc-4897-97af-69210ebd9f7b \
            --vnet-name CoreServicesVnet \
            --name SharedServicesSubnet \
            --address-prefixes 10.20.10.0/24

        az network vnet subnet create \
            --resource-group learn-b6ec6547-65cc-4897-97af-69210ebd9f7b \
            --vnet-name CoreServicesVnet \
            --name DatabaseSubnet \
            --address-prefixes 10.20.20.0/24

        az network vnet subnet create \
            --resource-group learn-b6ec6547-65cc-4897-97af-69210ebd9f7b \
            --vnet-name CoreServicesVnet \
            --name PublicWebServiceSubnet \
            --address-prefixes 10.20.30.0/24

        az network vnet subnet list \
            --resource-name xxx \
            --vnet-name CoreServicesVnet \
            --output table
        >>
        AddressPrefix    Name                    PrivateEndpointNetworkPolicies    PrivateLinkServiceNetworkPolicies    ProvisioningState    ResourceGroup
        ---------------  ----------------------  --------------------------------  -----------------------------------  -------------------  ------------------------------------------
        10.20.0.0/27     GatewaySubnet           Enabled                           Enabled                              Succeeded            learn-b6ec6547-65cc-4897-97af-69210ebd9f7b
        10.20.10.0/24    SharedServicesSubnet    Enabled                           Enabled                              Succeeded            learn-b6ec6547-65cc-4897-97af-69210ebd9f7b
        10.20.20.0/24    DatabaseSubnet          Enabled                           Enabled                              Succeeded            learn-b6ec6547-65cc-4897-97af-69210ebd9f7b
        10.20.30.0/24    PublicWebServiceSubnet  Enabled                           Enabled                              Succeeded            learn-b6ec6547-65cc-4897-97af-69210ebd9f7b

      ```
5. Azure Virtual Network exec 104.55 (CLI)
   ```bash
    = Azure Vnet ======================
    # Create Azure Vnet
    az network vnet create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name Azure-VNet-1 \
    >     --address-prefix 10.0.0.0/16 \
    >     --subnet-name Services \
    >     --subnet-prefix 10.0.0.0/24
    # Create Gateway subnet
    az network vnet subnet create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --vnet-name Azure-VNet-1 \
    >     --address-prefix 10.0.255.0/27 \
    >     --name GatewaySubnet
    # Create local-gateway to HQ
    az network local-gateway create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --gateway-ip-address 94.0.252.160 \
    >     --name LNG-HQ-Network \
    >     --local-address-prefixes 172.16.0.0/16
    {- Finished ..
      "bgpSettings": null,
      "etag": "W/\"09c14cfc-7c2d-43d4-b30c-f58246226028\"",
      "fqdn": null,
      "gatewayIpAddress": "94.0.252.160",
      "id": "/subscriptions/8485da0b-ca20-48ec-a5c6-13784afba88f/resourceGroups/learn-f0e70202-a297-496f-9c74-6ee32b1300f9/providers/Microsoft.Network/localNetworkGateways/LNG-HQ-Network",
      "localNetworkAddressSpace": {
        "addressPrefixes": [
          "172.16.0.0/16"
        ]
      },
      "location": "westus",
      "name": "LNG-HQ-Network",
      "provisioningState": "Succeeded",
      "resourceGroup": "learn-f0e70202-a297-496f-9c74-6ee32b1300f9",
      "resourceGuid": "d3ee8cfb-54a8-48d3-8047-cef49c5cd799",
      "tags": null,
      "type": "Microsoft.Network/localNetworkGateways"
    }

    = HQ ====================
    # Create vnet
    az network vnet create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name HQ-Network \
    >     --address-prefix 172.16.0.0/16 \
    >     --subnet-name Applications \
    >     --subnet-prefix 172.16.0.0/24
    {
      "newVNet": {
        "addressSpace": {
          "addressPrefixes": [
            "172.16.0.0/16"
          ]
        },
        "bgpCommunities": null,
        "ddosProtectionPlan": null,
        "dhcpOptions": {
          "dnsServers": []
        },
        "enableDdosProtection": false,
        "enableVmProtection": false,
        "etag": "W/\"a202b9a3-ceb6-4c76-a12c-ec49bb08311b\"",
        "id": "/subscriptions/8485da0b-ca20-48ec-a5c6-13784afba88f/resourceGroups/learn-f0e70202-a297-496f-9c74-6ee32b1300f9/providers/Microsoft.Network/virtualNetworks/HQ-Network",
        "ipAllocations": null,
        "location": "westus",
        "name": "HQ-Network",
        "provisioningState": "Succeeded",
        "resourceGroup": "learn-f0e70202-a297-496f-9c74-6ee32b1300f9",
        "resourceGuid": "dc7d3f79-3154-49bd-8956-ad6a0d0bf370",
        "subnets": [
          {
            "addressPrefix": "172.16.0.0/24",
            "addressPrefixes": null,
            "delegations": [],
            "etag": "W/\"a202b9a3-ceb6-4c76-a12c-ec49bb08311b\"",
            "id": "/subscriptions/8485da0b-ca20-48ec-a5c6-13784afba88f/resourceGroups/learn-f0e70202-a297-496f-9c74-6ee32b1300f9/providers/Microsoft.Network/virtualNetworks/HQ-Network/subnets/Applications",
            "ipAllocations": null,
            "ipConfigurationProfiles": null,
            "ipConfigurations": null,
            "name": "Applications",
            "natGateway": null,
            "networkSecurityGroup": null,
            "privateEndpointNetworkPolicies": "Enabled",
            "privateEndpoints": null,
            "privateLinkServiceNetworkPolicies": "Enabled",
            "provisioningState": "Succeeded",
            "purpose": null,
            "resourceGroup": "learn-f0e70202-a297-496f-9c74-6ee32b1300f9",
            "resourceNavigationLinks": null,
            "routeTable": null,
            "serviceAssociationLinks": null,
            "serviceEndpointPolicies": null,
            "serviceEndpoints": null,
            "type": "Microsoft.Network/virtualNetworks/subnets"
          }
        ],
        "tags": {},
        "type": "Microsoft.Network/virtualNetworks",
        "virtualNetworkPeerings": []
      }
    }
    # create gatewaysubnet
    az network vnet subnet create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --address-prefix 172.16.255.0/27 \
    >     --name GatewaySubnet \
    >     --vnet-name HQ-Network
    # Cretae local-gateway to aure vnet
    az network local-gateway create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --gateway-ip-address 94.0.252.160 \
    >     --name LNG-Azure-VNet-1 \
    >     --local-address-prefixes 10.0.0.0/16

    =======================list=================
    # list vnet
    az network vnet list --output table

    Name          ResourceGroup                               Location    NumSubnets    Prefixes       DnsServers    DDOSProtection    VMProtection
    ------------  ------------------------------------------  ----------  ------------  -------------  ------------  ----------------  --------------
    Azure-VNet-1  learn-f0e70202-a297-496f-9c74-6ee32b1300f9  westus      2             10.0.0.0/16                  False             False
    HQ-Network    learn-f0e70202-a297-496f-9c74-6ee32b1300f9  westus      2             172.16.0.0/16                False             False
    
    # list local-gateway
    az network local-gateway list \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --output table

    Name              Location    ResourceGroup                               ProvisioningState    GatewayIpAddress    AddressPrefixes
    ----------------  ----------  ------------------------------------------  -------------------  ------------------  -----------------
    LNG-Azure-VNet-1  westus      learn-f0e70202-a297-496f-9c74-6ee32b1300f9  Succeeded            94.0.252.160        10.0.0.0/16
    LNG-HQ-Network    westus      learn-f0e70202-a297-496f-9c74-6ee32b1300f9  Succeeded            94.0.252.160        172.16.0.0/16

    ■ pip,vnet-gateway ■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■■
    # create pip fro azure vng PIP-VNG-Azure-VNet-1
    az network public-ip create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name PIP-VNG-Azure-VNet-1 \
    >     --allocation-method Dynamic
    {
      "publicIp": {
        "ddosSettings": null,
        "dnsSettings": null,
        "etag": "W/\"cc357f1e-88c3-48ce-b529-d319f4ad1854\"",
        "id": "/subscriptions/8485da0b-ca20-48ec-a5c6-13784afba88f/resourceGroups/learn-f0e70202-a297-496f-9c74-6ee32b1300f9/providers/Microsoft.Network/publicIPAddresses/PIP-VNG-Azure-VNet-1",
        "idleTimeoutInMinutes": 4,
        "ipAddress": null,
        "ipConfiguration": null,
        "ipTags": [],
        "location": "westus",
        "name": "PIP-VNG-Azure-VNet-1",
        "provisioningState": "Succeeded",
        "publicIpAddressVersion": "IPv4",
        "publicIpAllocationMethod": "Dynamic",
        "publicIpPrefix": null,
        "resourceGroup": "learn-f0e70202-a297-496f-9c74-6ee32b1300f9",
        "resourceGuid": "f69cc6b9-21a7-4dc4-a8b4-8328cb8e1ab0",
        "sku": {
          "name": "Basic"
        },
        "tags": null,
        "type": "Microsoft.Network/publicIPAddresses",
        "zones": null
      }
    }
    
    # create vnet-gateway in aure vnet
    az network vnet-gateway create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name VNG-Azure-VNet-1 \
    >     --public-ip-address PIP-VNG-Azure-VNet-1 \
    >     --vnet Azure-VNet-1 \
    >     --gateway-type Vpn \
    >     --vpn-type RouteBased \
    >     --sku VpnGw1 \
    >     --no-wait

    # create pip for hq vng PIP-VNG-HQ-Network 
    az network public-ip create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name PIP-VNG-HQ-Network \
    >     --allocation-method Dynamic

    # create vnet-gateway in hq vnet
    az network vnet-gateway create \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name VNG-HQ-Network \
    >     --public-ip-address PIP-VNG-HQ-Network \
    >     --vnet HQ-Network \
    >     --gateway-type Vpn \
    >     --vpn-type RouteBased \
    >     --sku VpnGw1 \
    >     --no-wait

   # watch createing status ====================
    watch -d -n 5 az network vnet-gateway list \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --output table

    ActiveActive    EnableBgp    EnablePrivateIpAddress    GatewayType    Location    Name              ProvisioningState    ResourceGroup                               ResourceGuid                          VpnGatewayGeneration    VpnType
    --------------  -----------  ------------------------  -------------  ----------  ----------------  -------------------  ------------------------------------------  ------------------------------------  ----------------------  ----------
    False           False        False                     Vpn            westus      VNG-Azure-VNet-1  Succeeded            learn-f0e70202-a297-496f-9c74-6ee32b1300f9  f6abdae2-b623-4f8d-a3bb-9ea1c63a1cdb  Generation1             RouteBased
    False           False        False                     Vpn            westus      VNG-HQ-Network    Succeeded            learn-f0e70202-a297-496f-9c74-6ee32b1300f9  19bafc10-4742-4a2f-a822-64f6158e6e9b  Generation1             RouteBased

    # vnet-gateway list
    az network vnet-gateway list \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --query "[?provisioningState=='Succeeded']" \
    >     --output table
    Name              Location    GatewayType    VpnType     VpnGatewayGeneration    EnableBgp    EnablePrivateIpAddress   ActiveActive    ResourceGuid                          ProvisioningState    ResourceGroup
    ----------------  ----------  -------------  ----------  ----------------------  -----------  ------------------------  --------------  ------------------------------------  -------------------  ------------------------------------------
    VNG-Azure-VNet-1  westus      Vpn            RouteBased  Generation1             False        False   False           f6abdae2-b623-4f8d-a3bb-9ea1c63a1cdb  Succeeded            learn-f0e70202-a297-496f-9c74-6ee32b1300f9
    VNG-HQ-Network    westus      Vpn            RouteBased  Generation1             False        False   False           19bafc10-4742-4a2f-a822-64f6158e6e9b  Succeeded            learn-f0e70202-a297-496f-9c74-6ee32b1300f9


    # get vng azure pip value 
    che_bender@Azure:~$ PIPVNGAZUREVNET1=$(az network public-ip show \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name PIP-VNG-Azure-VNet-1 \
    >     --query "[ipAddress]" \
    >     --output tsv)
    che_bender@Azure:~$ echo $PIPVNGAZUREVNET1
    40.112.212.67
    
    # update PIP-VNG-Azure-VNet-1 to LNG-Azure-VNET-1
    che_bender@Azure:~$ az network local-gateway update \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name LNG-Azure-VNet-1 \
    >     --gateway-ip-address $PIPVNGAZUREVNET1
    {
      "bgpSettings": null,
      "etag": "W/\"bfba5b0c-2a26-4438-83eb-f022d59c48cb\"",
      "fqdn": null,
      "gatewayIpAddress": "40.112.212.67",
      "id": "/subscriptions/8485da0b-ca20-48ec-a5c6-13784afba88f/resourceGroups/learn-f0e70202-a297-496f-9c74-6ee32b1300f9/providers/Microsoft.Network/localNetworkGateways/LNG-Azure-VNet-1",
      "localNetworkAddressSpace": {
        "addressPrefixes": [
          "10.0.0.0/16"
        ]
      },
      "location": "westus",
      "name": "LNG-Azure-VNet-1",
      "provisioningState": "Succeeded",
      "resourceGroup": "learn-f0e70202-a297-496f-9c74-6ee32b1300f9",
      "resourceGuid": "5173cf5e-58cf-42c7-b9a2-b5549bba4ad6",
      "tags": null,
      "type": "Microsoft.Network/localNetworkGateways"
    }

    # get vng hq pip value
    che_bender@Azure:~$ PIPVNGHQNETWORK=$(az network public-ip show \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name PIP-VNG-HQ-Network \
    >     --query "[ipAddress]" \
    >     --output tsv)

    # update PIP-VNG-HQ-Network to LNG-HQ-Network 
    che_bender@Azure:~$ az network local-gateway update \
    >     --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
    >     --name LNG-HQ-Network \
    >     --gateway-ip-address $PIPVNGHQNETWORK
    {
      "bgpSettings": null,
      "etag": "W/\"8a041b2a-5960-47f6-904c-f29a7195fdec\"",
      "fqdn": null,
      "gatewayIpAddress": "40.112.209.145",
      "id": "/subscriptions/8485da0b-ca20-48ec-a5c6-13784afba88f/resourceGroups/learn-f0e70202-a297-496f-9c74-6ee32b1300f9/providers/Microsoft.Network/localNetworkGateways/LNG-HQ-Network",
      "localNetworkAddressSpace": {
        "addressPrefixes": [
          "172.16.0.0/16"
        ]
      },
      "location": "westus",
      "name": "LNG-HQ-Network",
      "provisioningState": "Succeeded",
      "resourceGroup": "learn-f0e70202-a297-496f-9c74-6ee32b1300f9",
      "resourceGuid": "d3ee8cfb-54a8-48d3-8047-cef49c5cd799",
      "tags": null,
      "type": "Microsoft.Network/localNetworkGateways"
    }


    # Create connect =======================
    
    #创建用于连接的共享密钥 是一个由可打印 ASCII 字符组成的字符串，长度不得超出 128 个字符
    SHAREDKEY=<shared key>

    az network vpn-connection create \
        --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
        --name Azure-VNet-1-To-HQ-Network \
        --vnet-gateway1 VNG-Azure-VNet-1 \
        --shared-key $SHAREDKEY \
        --local-gateway2 LNG-HQ-Network

    az network vpn-connection create \
        --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
        --name HQ-Network-To-Azure-VNet-1  \
        --vnet-gateway1 VNG-HQ-Network \
        --shared-key $SHAREDKEY \
        --local-gateway2 LNG-Azure-VNet-1


    az network vpn-connection show \
        --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
        --name Azure-VNet-1-To-HQ-Network  \
        --output table \
        --query '{Name:name,ConnectionStatus:connectionStatus}'


    az network vpn-connection show \
        --resource-group learn-f0e70202-a297-496f-9c74-6ee32b1300f9 \
        --name HQ-Network-To-Azure-VNet-1  \
        --output table \
        --query '{Name:name,ConnectionStatus:connectionStatus}'
    Name                        ConnectionStatus
    --------------------------  ------------------
    HQ-Network-To-Azure-VNet-1  Connected
    ```
6. 104.54  virtual network peering  練習  
    ```bash
    # create  salesvnet
    az network vnet create \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --name SalesVNet \
      --address-prefix 10.1.0.0/16 \
      --subnet-name Apps \
      --subnet-prefix 10.1.1.0/24 \
      --location northeurope
    # create MarketingVNet
    az network vnet create \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --name MarketingVNet \
      --address-prefix 10.2.0.0/16 \
      --subnet-name Apps \
      --subnet-prefix 10.2.1.0/24 \
      --location northeurope
    # create ResearchVNet
    az network vnet create \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --name ResearchVNet \
      --address-prefix 10.3.0.0/16 \
      --subnet-name Data \
      --subnet-prefix 10.3.1.0/24 \
      --location westeurope
    # list vnet
    az network vnet list --output table
    Name           ResourceGroup                               Location     NumSubnets   Prefixes     DnsServers    DDOSProtection    VMProtection
    -------------  ------------------------------------------  -----------  ------------ -----------  ------------  ----------------  --------------
    MarketingVNet  learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3  northeurope  1             10.2.0.0/16                False             False
    SalesVNet      learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3  northeurope  1             10.1.0.0/16                False             False
    ResearchVNet   learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3  westeurope   1             10.3.0.0/16                False             False

    # create vm in SalesVNet
    az vm create \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --no-wait \
      --name SalesVM \
      --location northeurope \
      --vnet-name SalesVNet \
      --subnet Apps \
      --image UbuntuLTS \
      --admin-username azureuser \
      --admin-password abcdef123456!
    # create vm in MarketingVNet
    az vm create \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --no-wait \
      --name MarketingVM \
      --location northeurope \
      --vnet-name MarketingVNet \
      --subnet Apps \
      --image UbuntuLTS \
      --admin-username azureuser \
      --admin-password abcdef123456!
    # create vm in ResearchVNet
    az vm create \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --no-wait \
      --name ResearchVM \
      --location westeurope \
      --vnet-name ResearchVNet \
      --subnet Data \
      --image UbuntuLTS \
      --admin-username azureuser \
      --admin-password abcdef123456!
    # 运行以下命令，确认 VM 处于运行状态。 将使用 Linux watch 命令，该命令每五秒刷新一次。
    watch -d -n 5 "az vm list \
    --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
    --show-details \
    --query '[*].{Name:name, ProvisioningState:provisioningState, PowerState:powerState}' \
    --output table"
    
    # ===创建虚拟网络对等互连连接================
    # SalesVNet to MarketingVNet
    az network vnet peering create \
      --name SalesVNet-To-MarketingVNet \
      --remote-vnet MarketingVNet \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --vnet-name SalesVNet \
      --allow-vnet-access
    # MarketingVNet to SalesVNet
    az network vnet peering create \
      --name MarketingVNet-To-SalesVNet \
      --remote-vnet SalesVNet \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --vnet-name MarketingVNet \
      --allow-vnet-access
    
    # MarketingVNet-To-ResearchVNet
    az network vnet peering create \
      --name MarketingVNet-To-ResearchVNet \
      --remote-vnet ResearchVNet \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --vnet-name MarketingVNet \
      --allow-vnet-access
    # ResearchVNet-To-MarketingVNet 
    az network vnet peering create \
      --name ResearchVNet-To-MarketingVNet \
      --remote-vnet MarketingVNet \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --vnet-name ResearchVNet \
      --allow-vnet-access
    
    # check
    az network vnet peering list \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --vnet-name SalesVNet \
      --output table
    AllowForwardedTraffic    AllowGatewayTransit    AllowVirtualNetworkAccess    Name                        PeeringState    ProvisioningState    ResourceGroup                               UseRemoteGateways
    -----------------------  ---------------------  ---------------------------  --------------------------  --------------  ------------------- ------------------------------------------  -------------------
    False                    False                  True                         SalesVNet-To-MarketingVNet  Connected    Succeeded            learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3  False
    
    # 运行以下命令，以查看适用于“SalesVM”网络接口的路由。
    az network nic show-effective-route-table \
    --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
    --name SalesVMVMNic \
    --output table
    
    Source    State    Address Prefix    Next Hop Type    Next Hop IP
    --------  -------  ----------------  ---------------  -------------
    Default   Active   10.1.0.0/16       VnetLocal
    Default   Active   10.2.0.0/16       VNetPeering
    Default   Active   0.0.0.0/0         Internet
    Default   Active   10.0.0.0/8        None
    Default   Active   100.64.0.0/10     None
    Default   Active   192.168.0.0/16    None
    Default   Active   25.33.80.0/20     None
    Default   Active   25.41.3.0/25      None

    # ==== 通过在 Azure 虚拟机之间使用 SSH 验证虚拟网络对等互连 ====

    # list vm
    az vm list \
      --resource-group learn-09eb07fc-85f4-44f2-8cc5-4f2e2a67b5a3 \
      --query "[*].{Name:name, PrivateIP:privateIps, PublicIP:publicIps}" \
      --show-details \
      --output table

    Name         PrivateIP    PublicIP
    -----------  -----------  --------------
    MarketingVM  10.2.1.4     168.61.90.242
    SalesVM      10.1.1.4     65.52.225.217
    ResearchVM   10.3.1.4     51.137.107.112

    # SSH to to test peering 
    # 1. ssh to SalesVM in cloud shell (need public IP)
    che_bender@Azure:~$ ssh -o StrictHostKeyChecking=no azureuser@65.52.225.217
    # 2. ssh to MarketingVM in SalesVM bash (need private IP)
    azureuser@SalesVM:~$ ssh -o StrictHostKeyChecking=no azureuser@10.2.1.4
    
    ```
7. 104.55 Route 練習
    ```bash
    #create route table
    az network route-table create \
    --name publictable \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --disable-bgp-route-propagation false

    #create route
    az network route-table route create \
    --route-table-name publictable \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name productionsubnet \
    --address-prefix 10.0.1.0/24 \
    --next-hop-type VirtualAppliance \
    --next-hop-ip-address 10.0.2.4

    # 创建 vnet 虚拟网络和 publicsubnet 子网。
    az network vnet create \
    --name vnet \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --address-prefix 10.0.0.0/16 \
    --subnet-name publicsubnet \
    --subnet-prefix 10.0.0.0/24

    #创建 privatesubnet 子网。
    az network vnet subnet create \
    --name privatesubnet \
    --vnet-name vnet \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --address-prefix 10.0.1.0/24

    # 创建 dmzsubnet 子网。
    az network vnet subnet create \
    --name dmzsubnet \
    --vnet-name vnet \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --address-prefix 10.0.2.0/24

    # 运行以下命令以显示 vnet 虚拟网络中的所有子网。
    az network vnet subnet list \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --vnet-name vnet \
    --output table

    AddressPrefix    Name           PrivateEndpointNetworkPolicies    PrivateLinkServiceNetworkPolicies    ProvisioningState    ResourceGroup
    ---------------  -------------  --------------------------------  -----------------------------------  -------------------  ------------------------------------------
    10.0.0.0/24      publicsubnet   Enabled                           Enabled                              Succeeded       learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d
    10.0.1.0/24      privatesubnet  Enabled                           Enabled                              Succeeded       learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d
    10.0.2.0/24      dmzsubnet      Enabled                           Enabled                              Succeeded       learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d
    
    # 将路由表与公共子网关联。
    az network vnet subnet update \
    --name publicsubnet \
    --vnet-name vnet \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --route-table publictable

    # ======
    # create vm
    az vm create \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name nva \
    --vnet-name vnet \
    --subnet dmzsubnet \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password <password>

    # 运行以下命令，以检索设备虚拟机的公共 IP 地址。 将地址保存到名为 NVAIP 的变量
    NVAIP="$(az vm list-ip-addresses \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name nva \
    --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
    --output tsv)"

    echo $NVAIP

    # == 启用 Azure 网络接口的 IP 转发
    # 运行以下命令以获取 NVA 网络接口的 ID。
    NICID=$(az vm nic list \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --vm-name nva \
    --query "[].{id:id}" --output tsv)

    echo $NICID

    # 运行以下命令以获取 NVA 网络接口的名称。
    NICNAME=$(az vm nic show \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --vm-name nva \
    --nic $NICID \
    --query "{name:name}" --output tsv)

    echo $NICNAME

    # 运行以下命令以启用网络接口的 IP 转发。
    az network nic update --name $NICNAME \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --ip-forwarding true
    
    # == 在设备中启用 IP 转发
    # 运行以下命令，将 NVA 虚拟机的公共 IP 地址保存到 NVAIP 变量。
    NVAIP="$(az vm list-ip-addresses \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name nva \
    --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
    --output tsv)"

    echo $NVAIP

    # 运行以下命令以启用 NVA 中的 IP 转发
    ssh -t -o StrictHostKeyChecking=no azureuser@$NVAIP 'sudo sysctl -w net.ipv4.ip_forward=1; exit;'

    # === 通过 NVA 路由流量 ==
    # 创建公共和专用虚拟机
    # edit cloud-init.txt
    code cloud-init.txt
      #cloud-config
      package_upgrade: true
      packages:
        - inetutils-traceroute
    
    # 在 Cloud Shell 中运行以下命令以创建公共虚拟机。 将 <password> 替换为 azureuser 帐户的合适密码。
    az vm create \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name public \
    --vnet-name vnet \
    --subnet publicsubnet \
    --image UbuntuLTS \
    --admin-username azureuser \
    --no-wait \
    --custom-data cloud-init.txt \
    --admin-password <password>

    # 运行以下命令以创建专用虚拟机。 将 <password> 替换为合适的密码。
    az vm create \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name private \
    --vnet-name vnet \
    --subnet privatesubnet \
    --image UbuntuLTS \
    --admin-username azureuser \
    --no-wait \
    --custom-data cloud-init.txt \
    --admin-password <password>

    # watch 
    watch -d -n 5 "az vm list \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --show-details \
    --query '[*].{Name:name, ProvisioningState:provisioningState, PowerState:powerState}' \
    --output table"

    # 运行以下命令，将公共虚拟机的公共 IP 地址保存到名为 PUBLICIP 的变量
    PUBLICIP="$(az vm list-ip-addresses \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name public \
    --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
    --output tsv)"

    echo $PUBLICIP

    #运行以下命令，将专用虚拟机的公共 IP 地址保存到名为 PRIVATEIP 的变量。
    PRIVATEIP="$(az vm list-ip-addresses \
    --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d \
    --name private \
    --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
    --output tsv)"

    echo $PRIVATEIP

    # == 通过网络虚拟设备测试流量路由
    # 运行以下命令以跟踪从公共到专用的路由。 在出现提示时输入之前指定的 azureuser 帐户的密码。
    ssh -t -o StrictHostKeyChecking=no azureuser@$PUBLICIP 'traceroute private --type=icmp; exit'
    traceroute to private.lgrtyfvduorudampb0rtsgtvve.dx.internal.cloudapp.net (10.0.1.4), 64 hops max
      1   10.0.2.4  0.654ms  0.256ms  0.235ms
      2   10.0.1.4  1.193ms  2.778ms  1.261ms
    Connection to 13.91.82.23 closed.


    # ### show route-table publictable
    PS D:\box\azure> az network route-table show `
    >> --resource-group learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d `
    >> --name publictable
    {
      "disableBgpRoutePropagation": false,
      "etag": "W/\"b1019297-854d-4a3a-8a96-28ecdf5fa7e6\"",
      "id": "/subscriptions/a1ccbcf7-9d27-4cf3-aeff-4dca9c6f4c01/resourceGroups/learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d/providers/Microsoft.Network/routeTables/publictable",
      "location": "westus",
      "name": "publictable",
      "provisioningState": "Succeeded",
      "resourceGroup": "learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d",
      "routes": [
        {
          "addressPrefix": "10.0.1.0/24",
          "etag": "W/\"b1019297-854d-4a3a-8a96-28ecdf5fa7e6\"",
          "id": "/subscriptions/a1ccbcf7-9d27-4cf3-aeff-4dca9c6f4c01/resourceGroups/learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d/providers/Microsoft.Network/routeTables/publictable/routes/productionsubnet",
          "name": "productionsubnet",
          "nextHopIpAddress": "10.0.2.4",
          "nextHopType": "VirtualAppliance",
          "provisioningState": "Succeeded",
          "resourceGroup": "learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d",
          "type": "Microsoft.Network/routeTables/routes"
        }
      ],
      "subnets": [
        {
          "addressPrefix": null,
          "addressPrefixes": null,
          "delegations": null,
          "etag": null,
          "id": "/subscriptions/a1ccbcf7-9d27-4cf3-aeff-4dca9c6f4c01/resourceGroups/learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d/providers/Microsoft.Network/virtualNetworks/vnet/subnets/publicsubnet",
          "ipConfigurationProfiles": null,
          "ipConfigurations": null,
          "name": null,
          "natGateway": null,
          "networkSecurityGroup": null,
          "privateEndpointNetworkPolicies": null,
          "privateEndpoints": null,
          "privateLinkServiceNetworkPolicies": null,
          "provisioningState": null,
          "purpose": null,
          "resourceGroup": "learn-3f93013c-8cec-411e-8cc8-cb6f0ec4f32d",
          "resourceNavigationLinks": null,
          "routeTable": null,
          "serviceAssociationLinks": null,
          "serviceEndpointPolicies": null,
          "serviceEndpoints": null
        }
      ],
      "tags": null,
      "type": "Microsoft.Network/routeTables"
    }
    PS D:\box\azure>
    ```
8. 練習　104.58　应用程序安全组
   ```bash
    # == 创建虚拟网络和网络安全组 ==
    # create resourcegroup
    rg = rg104-58
    az group create --name $rg --location japaneast

    # create vnet + subnet applications
    az network vnet create \
      --resource-group $rg \
      --name ERP-servers \
      --address-prefix 10.0.0.0/16 \
      --subnet-name Applications \
      --subnet-prefix 10.0.0.0/24
    
    # create subnet databases
    az network vnet subnet create \
      --resource-group $rg \
      --vnet-name ERP-servers \
      --address-prefix 10.0.1.0/24 \
      --name Databases
    
    # CREATE NGS
    az network nsg create \
      --resource-group $rg \
      --name ERP-SERVERS-NSG

    # ==== 
    # create vm AppServer
    wget -N https://raw.githubusercontent.com/MicrosoftDocs/mslearn-secure-and-isolate-with-nsg-and-service-endpoints/master/cloud-init.yml && \
    az vm create \
        --resource-group $rg \
        --name AppServer \
        --vnet-name ERP-servers \
        --subnet Applications \
        --nsg ERP-SERVERS-NSG \
        --image UbuntuLTS \
        --size Standard_DS1_v2 \
        --admin-username azureuser \
        --custom-data cloud-init.yml \
        --no-wait \
        --admin-password <password>
    
    # create vm DataServer
    az vm create \
    --resource-group $rg \
    --name DataServer \
    --vnet-name ERP-servers \
    --subnet Databases \
    --nsg ERP-SERVERS-NSG \
    --size Standard_DS1_v2 \
    --image UbuntuLTS \
    --admin-username azureuser \
    --custom-data cloud-init.yml \
    --admin-password <password>
    
    # query list for vm status
    az vm list \
    --resource-group $rg \
    --show-details \
    --query "[*].{Name:name, Provisioned:provisioningState, Power:powerState}" \
    --output table

    Name        Provisioned    Power
    ----------  -------------  ----------
    AppServer   Succeeded      VM running
    DataServer  Succeeded      VM running

    # == check default link status
    # get public IP ★
    az vm list \
      --resource-group $rg \
      --show-details \
      --query "[*].{Name:name, PrivateIP:privateIps, PublicIP:publicIps}" \
      --output table
    # save to var
    APPSERVERIP="$(az vm list-ip-addresses \
                 --resource-group $rg \
                 --name AppServer \
                 --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
                 --output tsv)"

    DATASERVERIP="$(az vm list-ip-addresses \
                    --resource-group $rg \
                    --name DataServer \
                    --query "[].virtualMachine.network.publicIpAddresses[*].ipAddress" \
                    --output tsv)"
    # check link
    # 将会收到一条 Connection timed out 消息。
    # 原因として、指定したnsgのERP-SERVERS-NSGに、SSH許可ルールがないですから。
    # 普通はVMを作成した時、nsgを指定しないから、デフォルトで作成し、指定されたnsgに、SSH許可は存在する。default-allow-sshとか
    ssh azureuser@$APPSERVERIP -o ConnectTimeout=5
    # the same to APPSERVERIP
    ssh azureuser@$DATASERVERIP -o ConnectTimeout=5

    # == 为 SSH 创建安全规则
    az network nsg rule create \
      --resource-group $rg \
      --nsg-name ERP-SERVERS-NSG \
      --name AllowSSHRule \
      --direction Inbound \
      --priority 100 \
      --source-address-prefixes '*' \
      --source-port-ranges '*' \
      --destination-address-prefixes '*' \
      --destination-port-ranges 22 \
      --access Allow \
      --protocol Tcp \
      --description "Allow inbound SSH"
    # now you can ssh to vm
    ssh azureuser@$APPSERVERIP -o ConnectTimeout=5
    ssh azureuser@$DATASERVERIP -o ConnectTimeout=5

    # ===在 Cloud Shell 中运行以下命令，创建新的入站安全规则来拒绝端口 80 上的 HTTP 访问。
    az network nsg rule create \
      --resource-group $rg \
      --nsg-name ERP-SERVERS-NSG \
      --name httpRule \
      --direction Inbound \
      --priority 150 \
      --source-address-prefixes 10.0.1.4 \
      --source-port-ranges '*' \
      --destination-address-prefixes 10.0.0.4 \
      --destination-port-ranges 80 \
      --access Deny \
      --protocol Tcp \
      --description "Deny from DataServer to AppServer on port 80"
    # should be OK 200
    ssh -t azureuser@$APPSERVERIP 'wget http://10.0.1.4; exit; bash'
    # should be FAILED. becase the nsg rule deny 80 access to 10.0.0.4 from 10.0.1.4
    ssh -t azureuser@$DATASERVERIP 'wget http://10.0.0.4; exit; bash'

    # ===部署应用程序安全组==
    # 新建一个名为“ERP-DB-SERVERS-ASG”的应用程序安全组
    az network asg create \
      --resource-group $rg \
      --name ERP-DB-SERVERS-ASG
    # 将 DataServer 与这个应用程序安全组关联。
    az network nic ip-config update \
      --resource-group $rg \
      --application-security-groups ERP-DB-SERVERS-ASG \
      --name ipconfigDataServer \
      --nic-name DataServerVMNic \
      --vnet-name ERP-servers \
      --subnet Databases
    # 更新 ERP-SERVERS-NSG 网络安全组中的 HTTP 规则。 它应引用 ERP-DB-Servers 应用程序安全组。
    az network nsg rule update \
      --resource-group $rg \
      --nsg-name ERP-SERVERS-NSG \
      --name httpRule \
      --direction Inbound \
      --priority 150 \
      --source-address-prefixes "" \
      --source-port-ranges '*' \
      --source-asgs ERP-DB-SERVERS-ASG \       ★ASG is a source-args
      --destination-address-prefixes 10.0.0.4 \
      --destination-port-ranges 80 \
      --access Deny \
      --protocol Tcp \
      --description "Deny from DataServer to AppServer on port 80 using application security group"
    # the result is the same to before.
  ```

