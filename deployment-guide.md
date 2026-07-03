## Deploy Virtual Network base infrastructure
1. Launch Azure Cloud Shell from the top right corner
    1. Select **Bash**
    2. Selec the Azure subscription
    3. Do not select to create a storage account
2. Verify that you can see the Resource Group in Azure CLI by running `az group list`
3. Set your resource group name as an input variable: 
    1. `rgName="<provided-resource-group-name>"`, or;
    2. `rgName="$(az group list --output json | jq '.[0].name' -r)"`
4. Change to the **objective-01** directory: `cd objective-01`
5. Deploy the ARM template which will provision all the base infrastructure, including Virtual Network, Network Security Groups, Log Analytics Workspace and Storage Account: `az deployment group create -g $rgName -n "objective-01" -f azureDeploy.json`
6. Validate in the Azure Portal:
    1. **Resource grou**p > **Deployments**: latest deployment succeeded.
    2. Check the virtual networks: **vnet1**, **vnet2**, and **vnet3** exist, each with **subnet1**, **subnet2**, and **GatewaySubnet** subnets
    3. Network security groups: **nsgmin001**, **nsgmin002**, and **nsgmin003** exist and are associated to **subnet1** and **subnet2**.
    4. Storage accounts: a storage account named **`st<unique>`** exists (generated).
    5. Log Analytics workspaces: a workspace named **`log-<unique>`** exists (generated).

## Create and configure Azure VPN Gateways
1. Change to the **objective-02** directory: `cd ../objective-02`
2. Deploy the ARM template which will provision the Virtual Network (VPN) Gateways as well as the associated Public IP Addresses: `az deployment group create -g $rgName -n "objective-02" -f azureDeploy.json`
3. Validate in Portal
    1. **Virtual network gateways**: **vnet1-vng**, **vnet2-vng**, **vnet3-vng** show **Provisioning state: Succeeded** and a public IP address exists for each gateway: **vnet1-vng-pip**, **vnet2-vng-pip**, and **vnet3-vng-pip**.
    2. **Azure Monitor** > **Settings** > **Diagnostics settings**
    3. For any VNet Gateway: confirm diagnostic settings to the storage account and Log Analytics workspace

## Establish VNet-to-VNet VPN connectivity
1. Change to the **objective-03** directory: `cd ../objective-03`
2. Deploy the ARM template which will establish VNet-to-VNet VPN connectivity between **vnet1** and **vnet2**, and **vnet2** and **vnet3**: `az deployment group create -g $rgName -n "objective-03" -f azureDeploy.json`
3. Validate in Portal
    1. **Virtual network gateways** > select **vnet1-vng** or **vnet2-vng** > **Connections**
    2. See connections to the paired gateways with **Status: Connected** (after a few minutes)

## Enable Border Gateway Protocol (BGP) peering
1. Change to the **objective-04** directory: `cd ../objective-04`
2. Deploy the ARM template which will enable and configure BPG on each of the gateways, as well as on all the VPN connections: `az deployment group create -g $rgName -n "objective-04" -f azureDeploy.json`
3. Validate in Portal
    1. **Virtual network gateways** > each gateway > **Configuration**
    2. BGP shows a unique Autonomous System Number (ASN) and BGP peering address; provisioning state is **Succeeded**
    3. **Connections** tab on a gateway: connections show **BGP Enabled** and **Status: Connected**