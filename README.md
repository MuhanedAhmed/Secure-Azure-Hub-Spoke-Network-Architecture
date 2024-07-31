# Secure Azure Hub-Spoke Network Architecture Project

## Overview

This project demonstrates the implementation of a Hub-Spoke network topology in Azure using the Azure Portal. It simulates two environments: Production and Testing, with secure network connectivity, application load balancing, firewall security, virtual machine management, and monitoring and backup solutions.

## Table of Contents

- [Secure Azure Hub-Spoke Network Architecture Project](#secure-azure-hub-spoke-network-architecture-project)
  - [Overview](#overview)
  - [Table of Contents](#table-of-contents)
  - [Architecture Diagram](#architecture-diagram)
  - [Project Description](#project-description)
    - [Hub-Vnet Resources:](#hub-vnet-resources)
    - [Production-Vnet Resources:](#production-vnet-resources)
    - [Testing-Vnet Resources:](#testing-vnet-resources)
    - [Additional Services:](#additional-services)
  - [Prerequisites](#prerequisites)
  - [Step-by-Step Implementation](#step-by-step-implementation)
    - [1. Resource Groups Creation](#1-resource-groups-creation)
    - [2. Virtual Networks and Subnets Setup](#2-virtual-networks-and-subnets-setup)
      - [2.1. Creating Peering Connections Between VNets](#21-creating-peering-connections-between-vnets)
    - [3. Deploying Virtual Machines](#3-deploying-virtual-machines)
      - [3.1. Installing Nginx on VMs](#31-installing-nginx-on-vms)
    - [4. Configuring Network Security](#4-configuring-network-security)
    - [5. Implementing Azure Firewall](#5-implementing-azure-firewall)
      - [5.1. Configuring Azure Firewall Policy](#51-configuring-azure-firewall-policy)
    - [6. Setting Up the Application Gateway](#6-setting-up-the-application-gateway)
    - [7. Virtual Network Gateway for VPN](#7-virtual-network-gateway-for-vpn)
      - [7.1 Configuring Point-to-Site (P2S) VPN](#71-configuring-point-to-site-p2s-vpn)
    - [8. Configuring Storage Account and Private Endpoint](#8-configuring-storage-account-and-private-endpoint)
    - [9. Setting Up Azure Bastion](#9-setting-up-azure-bastion)
    - [10. Configuring Monitoring with Azure Monitor and Log Analytics Workspace](#10-configuring-monitoring-with-azure-monitor-and-log-analytics-workspace)
        - [10.1. Enabling Monitoring for VMs with Data Collection Rule](#101-enabling-monitoring-for-vms-with-data-collection-rule)
    - [11. Configuring Backup with Recovery Services Vault](#11-configuring-backup-with-recovery-services-vault)
    - [12. Creating and Configuring Route Tables](#12-creating-and-configuring-route-tables)
      - [12.1. Route Table Creation](#121-route-table-creation)
      - [12.2. Configuring Routes for Production Subnets](#122-configuring-routes-for-production-subnets)
    - [13 Testing the Application Gateway](#13-testing-the-application-gateway)
  - [Conclusion](#conclusion)
  - [Contributors](#contributors)

## Architecture Diagram

![Architecture Diagram](./General%20Architecture.png)

## Project Description

This project consists of three Virtual Networks (Vnets) — Hub-Vnet, Production-Vnet, and Testing-Vnet — each containing specific resources and configurations. The Hub-Vnet serves as the central network hub, while the Production-Vnet and Testing-Vnet simulate two different environments.

![Vnets Overview](./Project%20Screens/Vnets%20and%20Subnets/01%20Vnets%20Overview.png)

### Hub-Vnet Resources:

- **Application Gateway**: Deployed in `AppGatewaySubnet` with Web Application Firewall (WAF) policies applied.
- **Azure Firewall**: Deployed in `AzureFirewallSubnet` to filter traffic between Vnets.
- **Azure Bastion**: Deployed in `AzureBastionSubnet` for secure remote management of VMs.
- **Virtual Network Gateway (Hub-VGW)**: Deployed in `GatewaySubnet` for Point-to-Site (P2S) VPN connections.

![Hub-Vnet](./Project%20Screens/Vnets%20and%20Subnets/02%20Hub%20Vnet%20Subnets.png)

### Production-Vnet Resources:

- **Production-VM01**: Deployed in `Subnet01`.
- **Production-VM02**: Deployed in `Subnet02`.
- **Private Endpoint (SA-PE)**: Connected to a storage account (`gbgproject`) in `Subnet01` for private access.

![Production-Vnet](./Project%20Screens/Vnets%20and%20Subnets/03%20Production%20Vnet%20Subnets.png)

### Testing-Vnet Resources:

- **Testing-VM01**: Deployed in `Subnet01`.

![Testing-Vnet](./Project%20Screens/Vnets%20and%20Subnets/04%20Testing%20Vnet%20Subnets.png)

### Additional Services:

- **Azure Monitor with Log Analytics Workspace**: `Project-LogAnalyticsWorkspace` for collecting logs from all components.
- **Recovery Services Vault**: `Project-RSV` for regular backups of VMs.
- **Storage Account**: FileShare service with a private endpoint.

## Prerequisites

- An active Azure subscription
- Basic knowledge of Azure networking and security services

## Step-by-Step Implementation

### 1. Resource Groups Creation

1. Go to the [Azure Portal](https://portal.azure.com/).
2. In the left-hand menu, click on **Resource groups**.
3. Click on **+ Create**.
4. Fill in the necessary details such as **Subscription**, **Resource group name**, and **Region**.
5. Click **Review + create**, then **Create**.

![Resource Group Creation](./Project%20Screens/Resource%20Group%20Creation.png)

### 2. Virtual Networks and Subnets Setup

1. In the Azure Portal, search for and select **Virtual networks**.
2. Click **+ Create**.
3. Fill in the details for the Hub-Vnet:
   - **Name**: Hub-Vnet
   - **Region**: (Select the desired region)
   - **Resource Group**: Select the previously created resource group
4. Under the **IP Addresses** tab, configure the address space and create subnets like `AppGatewaySubnet`.
5. Repeat this process to create Production-Vnet and Testing-Vnet with their respective subnets.

#### 2.1. Creating Peering Connections Between VNets

1. In the Azure Portal, search for and select **Virtual networks**.
2. Click on the **Hub-Vnet**.
3. Under the **Settings** section, select **Peerings**.
4. Click **+ Add** to create a new peering.
5. Fill in the details for the peering:
   - **Peering link name (to remote VNet)**: Hub-to-Production
   - **Resource group**: Select the existing resource group
   - **Virtual network**: Production-Vnet
   - **Allow virtual network access**: Enabled
   - **Allow forwarded traffic**: Enabled
   - **Allow gateway transit**: Enabled
   - **Use remote gateways**: Disabled
6. Click **Add** to create the peering.

7. Repeat these steps to create peering connections between **Hub-Vnet** and **Testing-Vnet**
 
![VNet Peering](./Project%20Screens/Vnets%20and%20Subnets/05%20Hub%20Peerings.png)

### 3. Deploying Virtual Machines

1. In the Azure Portal, search for and select **Virtual machines**.
2. Click **+ Create** and then **Azure virtual machine**.
3. Choose your **Subscription**, **Resource group**, and provide a **VM name** (e.g., Production-VM01).
4. Select the **Region**, **Availability options**, and **Image** (e.g., Ubuntu Server 20.04 LTS).
5. Configure the **VM size**, **Administrator account** (SSH key or password), and **Inbound port rules**.
6. Under **Networking**, select the **VNet** and **Subnet** where the VM will be deployed (e.g., Subnet01).
7. Click **Review + create**, then **Create**.
8. Repeat this process to create Production-VM02 and Testing-VM01 with their respective Vnets and subnets.

![VMs Creation](./Project%20Screens/VMs%20&%20Backup/01%20VMs.png)

#### 3.1. Installing Nginx on VMs

After deploying the VMs, you need to install nginx on each VM to use as a backend for the Application Gateway.

1. In the Azure Portal, click on the **CloudShell** icon in the top-right corner of the portal (it looks like a >_ symbol).

2. Choose **Bash** in CloudShell.

3. Use the following Azure CLI command to install nginx on your VM. Replace `myResourceGroupAG` with your resource group name and `myVM` with the name of your VM.

```bash
az vm extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vm-name myVM \
  --settings '{ 
      "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"], 
      "commandToExecute": "./install_nginx.sh" 
  }'
```
4. Repeat this process for all the VMs.


### 4. Configuring Network Security

1. In the Azure Portal, search for **Network security groups**.
2. Click **+ Create** and configure the details, such as **Resource group**, **Name**, and **Region**.
3. After creation, click on the NSG and select **Inbound security rules**.
4. Click **+ Add** to create rules like allowing SSH (port 22) only from the subnet where the Azure Bastion will be deployed.
5. Attach this NSG to the three subnets where the VMs are deployed to enable SSH connection only from bastion.

![NSG Creation](./Project%20Screens/NSG%20Creation.png)

### 5. Implementing Azure Firewall

1. In the Azure Portal, search for **Firewall**.
2. Click **+ Create** and fill in the required fields:
   - **Subscription**, **Resource group**, **Firewall name**
   - Select the **VNet** (Hub-Vnet) and **Subnet** (AzureFirewallSubnet).
3. Configure public IP and click **Review + create**, then **Create**.

![Azure Firewall Creation](./Project%20Screens/Azure%20Firewall/01%20Firewall%20Creation.png)

#### 5.1. Configuring Azure Firewall Policy

1. In the Azure Portal, search for **Firewall Manager**.
2. Select your **Azure Firewall** instance, then click on **Firewall Policy** under the **Settings** section.
3. Click on **+ Add** to create a new **Rule Collection**.
4. Fill in the details for the rule collection:
   - **Name**: Project-Network-RuleCollection
   - **Rule collection type**: Network
   - **Priority**: 100 (or as per your policy priority order)
   - **Rule collection action**: Allow
   - **Rule collection group**: DefaultNetworkRuleCollectionGroup
5. Under **Rules**, configure each rule as needed:
   - **Name**: (e.g., Allow-AppGW-to-Backend)
   - **Source type**: IP Address or IP Group
   - **Source**: Specify the IP address or IP range (e.g., 10.10.17.0/24)
   - **Protocol**: Select the protocol (e.g., TCP)
   - **Destination Ports**: Specify the port (e.g., 80)
   - **Destination Type**: IP Address or IP Group
   - **Destination**: Specify the destination IP (e.g., Backend-IP-Group)
6. Once all rules are added, click **Save** to apply the firewall policy.

![Firewall Policy Creation](./Project%20Screens/Azure%20Firewall/02%20Azure%20Firewall%20Policy.png)

### 6. Setting Up the Application Gateway

1. In the Azure Portal, search for **Application gateways**.
2. Click **+ Create** and provide the necessary details, including:
   - **Resource group**, **Application gateway name**, **Region**
   - **VNet** (Hub-Vnet) and **Subnet** (AppGatewaySubnet)

![Application Gateway Creation](./Project%20Screens/Application%20Gateway/01%20App%20Gateway%20Creation.png)

3. Configure the **Frontend IP**, **Backend pools**, **HTTP settings**, and **Routing rules**.

![Frontends Creation](./Project%20Screens/Application%20Gateway/02%20Frontends.png)
![Backends Creation](./Project%20Screens/Application%20Gateway/03%20Backends.png)
![Routing Rules Creation](./Project%20Screens/Application%20Gateway/04%20Routing%20Rule.png)
![Backend Settings Creation](./Project%20Screens/Application%20Gateway/05%20Backend%20settings.png)

4. Under **Web Application Firewall**, select WAF and configure the policy.
5. Review and create the Application Gateway.

![Application Gateway Overview](./Project%20Screens/Application%20Gateway/06%20App%20Gateway%20Overview.png)

### 7. Virtual Network Gateway for VPN

1. In the Azure Portal, search for **Virtual network gateways**.
2. Click **+ Create** and fill in the fields:
   - **Resource group**, **Name**, **Region**, **Gateway type** (VPN), **VPN type** (Route-based)
   - **Virtual network** (Hub-Vnet) and **Public IP address**
3. Click **Review + create**, then **Create**.

![VPN Gateway Creation](./Project%20Screens/Virtual%20Gateway%20&%20VPN/01%20VGW%20Creation.png)

#### 7.1 Configuring Point-to-Site (P2S) VPN

1. In the Azure Portal, go to your **Virtual network gateway**.
2. Click on **Point-to-site configuration** under the **Settings** section.
3. Click **+ Configure now** to begin the configuration.

4. In the **Address pool** section, provide an address pool that will be used for VPN clients. For example:
   - **Address pool**: 10.10.15.0/16

5. In the **Authentication type** section, select **Azure Certificate** or **RADIUS Authentication**. For simplicity, we will use **Azure Certificate**.
   - **Root certificate**: Upload a root certificate that you have generated. This certificate will be used to authenticate the VPN clients.

6. Click **Save** to apply the configuration.

7. To generate and install the VPN client configuration package:
   - Go to the **Point-to-site configuration** page of the virtual network gateway.
   - Click **Download VPN client** to download the VPN client configuration package.
   - Distribute this package to your clients and install it.

![P2S VPN Configuration](./Project%20Screens/Virtual%20Gateway%20&%20VPN/02%20P2S%20Configuration.png)

### 8. Configuring Storage Account and Private Endpoint

1. In the Azure Portal, search for **Storage accounts**.
2. Click **+ Create** and provide the necessary information:
   - **Resource group**, **Storage account name**, **Region**
3. Create a FileShare storage to be attached to the VMs.

![Storage Account](./Project%20Screens/Storage%20Account%20&%20Private%20Endpoint/01%20FileShare.png)

4. Under **Networking**, select **Private endpoint** and associate it with the appropriate VNet (Production-Vnet) and Subnet.

![Storage Account](./Project%20Screens/Storage%20Account%20&%20Private%20Endpoint/02%20Private%20Endpoint%20for%20The%20FileShare.png)

5. Complete the rest of the configuration and click **Review + create**, then **Create**.

### 9. Setting Up Azure Bastion

1. In the Azure Portal, search for **Bastion**.
2. Click **+ Create** and fill in the details:
   - **Resource group**, **Name**, **Region**
   - **Virtual network** (Hub-Vnet) and **Subnet** (AzureBastionSubnet)
3. Configure the **Public IP** and click **Review + create**, then **Create**.

![Azure Bastion Creation](./Project%20Screens/Bastion%20Creation.png)

### 10. Configuring Monitoring with Azure Monitor and Log Analytics Workspace

1. In the Azure Portal, search for **Log Analytics workspaces**.
2. Click **+ Create** and provide the necessary details, including:
   - **Subscription**, **Resource group**, **Workspace name**, **Region**
3. Click **Review + create**, then **Create**.

![Log Analytics Workspace Creation](./Project%20Screens/Log%20Analytics%20Workspace%20&%20DCR/01%20Log%20Analytics%20Workspace%20Overview.png)

##### 10.1. Enabling Monitoring for VMs with Data Collection Rule

1. In the Azure Portal, navigate to **Log Analytics workspaces** and select the workspace you created.
2. Under **Settings**, click on **Data Collection Rules**.
3. Click **+ Create** to start a new data collection rule.
4. Provide the necessary details:
   - **Name**: (e.g., VM-Data-Collection-Rule)
   - **Resource Group**, **Region**
5. Configure the **Data Sources**:
   - **Virtual Machines**: Select the VMs you want to monitor.
6. Under **Rules**, configure the data collection settings:
   - **Collection Settings**: Define metrics and logs to be collected.
7. Click **Review + create**, then **Create** to apply the rule.

![Data Collection Rule Creation](./Project%20Screens/Log%20Analytics%20Workspace%20&%20DCR/02%20DCR%20Resources.png)

### 11. Configuring Backup with Recovery Services Vault

1. In the Azure Portal, search for **Recovery Services vaults**.
2. Click **+ Create** and fill in the required fields:
   - **Subscription**, **Resource group**, **Vault name**, **Region**
3. Click **Review + create**, then **Create**.
4. After creation, go to your **Recovery Services vault**.
5. Click on **Backup** under **Getting Started**.
6. Choose **Azure Virtual Machines** as the workload to backup and follow the prompts to configure backup policies and schedule.

![Backup Configuration](./Project%20Screens/VMs%20&%20Backup/02%20Backup%20Policy.png)
![RSV](./Project%20Screens/VMs%20&%20Backup/03%20RSV%20with%20attached%20VM.png)


### 12. Creating and Configuring Route Tables

#### 12.1. Route Table Creation

1. In the Azure Portal, search for **Route tables**.
2. Click **+ Create** and provide the necessary details:
   - **Subscription**
   - **Resource group**
   - **Route table name**: (e.g., `Production-RouteTable`)
   - **Region**

3. Click **Review + create**, then **Create**.
4. Repeat the process to create route tables for **Application Gateway**, **Virtual Network Gateway** and **Testing-Vnet** 

![Route Table Creation](./Project%20Screens/Route%20Tables/01%20Route%20Tables%20Overview.png)

#### 12.2. Configuring Routes for Production Subnets

After creating the route tables, you need to configure routes to direct traffic through the Azure Firewall.

1. In the Azure Portal, navigate to the **Route table** you just created.
2. Click on **Routes** under the **Settings** section.
3. Click **+ Add** to create a new route. Configure the following routes:

   - **Route to Internet**:
     - **Route name**: `Route-Internet`
     - **Address prefix**: `0.0.0.0/0`
     - **Next hop type**: `Virtual appliance`
     - **Next hop address**: Enter the private IP address of the Azure Firewall
   
   - **Route to Application Gateway**:
     - **Route name**: `Route-AppGateway`
     - **Address prefix**: Enter the address range of the Application Gateway subnet (e.g., `10.0.1.0/24`)
     - **Next hop type**: `Virtual appliance`
     - **Next hop address**: Enter the private IP address of the Azure Firewall

   - **Route to Virtual Network Gateway (VGW)**:
     - **Route name**: `Route-VGW`
     - **Address prefix**: Enter the address range of the VGW subnet (e.g., `10.1.0.0/24`)
     - **Next hop type**: `Virtual appliance`
     - **Next hop address**: Enter the private IP address of the Azure Firewall

   - **Route to Testing VNet**:
     - **Route name**: `Route-TestingVNet`
     - **Address prefix**: Enter the address range of the Testing VNet (e.g., `10.2.0.0/24`)
     - **Next hop type**: `Virtual appliance`
     - **Next hop address**: Enter the private IP address of the Azure Firewall

4. Click **Save** to apply the routes.
5. Associate the route table to its respective subnets.
6. Repeat the process for other route tables.

![Route Configuration](./Project%20Screens/Route%20Tables/03%20Production%20Vnet%20Route%20Table.png)
![Route Configuration](./Project%20Screens/Route%20Tables/02%20Application%20Gateway%20Route%20Table.png)

### 13 Testing the Application Gateway

1. Obtain the public IP address of the Application Gateway.
2. Open a browser and navigate to the public IP address of the Application Gateway.
3. Verify that the backend VMs respond correctly.

![Testing App Gateway](./Project%20Screens/Testing%20App%20gateway%20Public%20IP%2001.png)
![Testing App Gateway](./Project%20Screens/Testing%20App%20gateway%20Public%20IP%2002.png)

## Conclusion

This project showcases a comprehensive implementation of a secure, scalable, and manageable network architecture on Azure using the Azure Portal. The Hub-Spoke topology ensures efficient network traffic management, and the use of Azure services like Firewall, Bastion, and Application Gateway enhances security and availability. Monitoring and backup solutions provide operational resilience, making this architecture suitable for production environments.

## Contributors

- [Muhanned Ahmed](https://github.com/MuhanedAhmed)
