# 透過 Terraform on Azure 快速建立環境

## 環境與安裝必要條件

- 電腦環境
  - OS : Windows 11 專業版<br>
  - IDE : Visual Studio Code<br>
    https://code.visualstudio.com/download<br>
- 安裝 Azure CLI<br>
  https://docs.microsoft.com/zh-TW/cli/azure/install-azure-cli-windows?tabs=azure-cli<br>
  完成後，可以直接在命令提示字元輸入以下指令來驗證是否安裝成功：<br>
    ```
    az version
    ```
    成功後即可看到以下回覆的目前版本資訊：<br>
    ```
    {
    "azure-cli": "2.36.0",
    "azure-cli-core": "2.36.0",
    "azure-cli-telemetry": "1.0.6",
    "extensions": {}
    }
    ```
- 在 Windows 上安裝 Terraform<br>
  - 下載 Terraform<br>
    https://www.terraform.io/downloads.html<br>
  - 將下載下來的 terraform.exe 放在 C:\terraform 資料夾中，並將此路徑加入至系統環境變數中<br>
    ![GITHUB](https://github.com/BrianHsing/Azure-Bastion/blob/main/images/systempath.png "systempath")<br>
  - 開啟命令提示字元<br>
  - 輸入命令 ` terraform -version`<br>

# 使用 Azure CLI 登入

在這個部分可以持續使用 Terminal 繼續進行操作，但本篇會透過 Vscode 所內建的 Terminal 介面直接進行操作，這部分可以自行選擇要使用哪種方式。<br>

登入之前您必須確認幾件事情：<br>
  1. 確保自己的帳號與訂用帳戶有進行關聯，您必須是`參與者`才能建立 Azure 服務資源<br>
  2. 後續在建立服務主體時確保自己帳號的權限為`擁有者`或`使用者存取系統管理員`的權限，如果您沒有這兩個其中之一的權限，就會收到錯誤訊息<br>
  3. 登入 Azure CLI 後，務必要確認 TenantId、SubscriptionId 是不是您進行部署或管理所提供的<br>

確認後即可開始執行以下操作：<br>

- 首先在 Terminal 介面中執行 `az login`，之後會自動開啟網頁<br>
- 選擇您具有上述權限的帳號進行登入，並且能在您的 Terminal 中看到帳務資訊<br>
  ```
  [
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "您的TenantId",
    "id": "您的訂用帳戶id",
    "isDefault": true,
    "managedByTenants": [],
    "name": "您的訂用帳戶名稱",
    "state": "Enabled",
    "tenantId": "您的目錄ID",
    "user": {
      "name": "isadmin@brianhsing.club",
      "type": "user"
    }
  }
  ]
  ```
# 使用 Azure CLI 建立服務主體

請輸入下方命令列建立服務主體，主要是會在 Azure AD 註冊應用程式，並且給予此應用程式參與者的角色，才能透過 Terraform 進行部署與維運，其中的 `<service_principal_name>` 您可以自訂名稱：<br>
```
az ad sp create-for-rbac --name <service_principal_name> --role Contributor --scopes "/subscriptions/<subscription id>"
```

完成後您將可以得到`appId`、`password`、`tenant`、`displayname`等數值，請務必保留這些數值，稍後會需要用到：<br>
```
{
  "appId": "您的appId",
  "displayName": "您的displayName",
  "password": "您的password",
  "tenant": "您的tenant"
}
```
接下來就是將上述的資訊新增至環境變數，當然如果您只是要簡單進行測試，也可以略過這個步驟，將相關資訊加入到 Terraform 的 mail.tf 中：<br>
```
export ARM_SUBSCRIPTION_ID="<您的訂用帳戶id>"
export ARM_TENANT_ID="<您的tenant>"
export ARM_CLIENT_ID="<您的appId>"
export ARM_CLIENT_SECRET="<您的password>"
```

# 建立 `main.tf` 檔案，並且將以下程式碼貼上

````
terraform {
  required_version = ">=1.1.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>2.99.0"
    }
  }
}
# Define NSG rules
locals {
  bastion-subnet-nsg-rules-1 = {
    AllowBastionHostInBound = {
      key                        = "AllowBastionHostCommunication"
      priority                   = 150
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_ranges    = ["8080", "5701"]
      source_address_prefix      = "VirtualNetwork"
      destination_address_prefix = "VirtualNetwork"
    }
    AllowSshRdpOutBound = {
      key                        = "AllowSshRdpOutBound"
      priority                   = 100
      direction                  = "Outbound"
      access                     = "Allow"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_ranges    = ["22", "3389"]
      source_address_prefix      = "*"
      destination_address_prefix = "VirtualNetwork"
    }
    AllowBastionHostOutBound = {
      key                        = "AllowBastionHostCommunication"
      priority                   = 120
      direction                  = "Outbound"
      access                     = "Allow"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_ranges    = ["8080", "5701"]
      source_address_prefix      = "VirtualNetwork"
      destination_address_prefix = "VirtualNetwork"
    }
  }
  bastion-subnet-nsg-rules-2 = {
    AllowHttpsInbound = {
      key                        = "AllowHttpsInBound"
      priority                   = 120
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "443"
      source_address_prefix      = "Internet"
      destination_address_prefix = "*"
    }
    AllowGatewayManagerInBound = {
      key                        = "AllowGatewayManagerInBound"
      priority                   = 130
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "443"
      source_address_prefix      = "GatewayManager"
      destination_address_prefix = "*"
    }
    AllowAzureLoadBalancerInBound = {
      key                        = "AllowAzureLoadBalancerInBound"
      priority                   = 140
      direction                  = "Inbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "443"
      source_address_prefix      = "AzureLoadBalancer"
      destination_address_prefix = "*"
    }

    AllowAzureCloudOutBound = {
      key                        = "AllowAzureCloudOutBound"
      priority                   = 110
      direction                  = "Outbound"
      access                     = "Allow"
      protocol                   = "Tcp"
      source_port_range          = "*"
      destination_port_range     = "443"
      source_address_prefix      = "*"
      destination_address_prefix = "AzureCloud"
    }

    AllowGetSessionInformation = {
      key                        = "AllowGetSessionInformation"
      priority                   = 130
      direction                  = "Outbound"
      access                     = "Allow"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "80"
      source_address_prefix      = "*"
      destination_address_prefix = "Internet"
    }
    DenyVnetInBount = {
      key                        = "DenyVnetInBount"
      priority                   = 160
      direction                  = "Inbound"
      access                     = "Deny"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "*"
      source_address_prefix      = "VirtualNetwork"
      destination_address_prefix = "VirtualNetwork"
    }
    DenyAzureLoadBalancerInBound = {
      key                        = "DenyAzureLoadBalancerInBound"
      priority                   = 170
      direction                  = "Inbound"
      access                     = "Deny"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "*"
      source_address_prefix      = "AzureLoadBalancer"
      destination_address_prefix = "*"
    }
    DenyVnetOutBound = {
      key                        = "DenyVnetOutBound"
      priority                   = 140
      direction                  = "outbound"
      access                     = "Deny"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "*"
      source_address_prefix      = "VirtualNetwork"
      destination_address_prefix = "VirtualNetwork"
    }

    DenyInternetOutBound = {
      key                        = "DenyInternetOutBound"
      priority                   = 150
      direction                  = "outbound"
      access                     = "Deny"
      protocol                   = "*"
      source_port_range          = "*"
      destination_port_range     = "*"
      source_address_prefix      = "*"
      destination_address_prefix = "Internet"
    }
  }
}
# 用於測試使用，如非測試練習，請不要這些資訊寫在檔案內，否則就會被看光啦
provider "azurerm" {
  features {}
  subscription_id   = "<您的訂用帳戶id>"
  tenant_id         = "<您的tenant>"
  client_id         = "<您的appId>"
  client_secret     = "<您的password>"
}
# 建立一個名叫 bastion-lab 的資源群組，並且位於東亞
resource "azurerm_resource_group" "rg" {
  name     = "bastion-lab"
  location = "eastasia"
}
# Create Network Security Group
resource "azurerm_network_security_group" "hub-subnet-nsg" {
  name                = "hub-subnet-nsg"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}
# Create Network Security Group
resource "azurerm_network_security_group" "bastion-subnet-nsg" {
  name                = "bastion-subnet-nsg"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
}
resource "azurerm_network_security_rule" "bastion-subnet-nsg-rules-1" {
  for_each                    = local.bastion-subnet-nsg-rules-1
  name                        = each.key
  direction                   = each.value.direction
  access                      = each.value.access
  priority                    = each.value.priority
  protocol                    = each.value.protocol
  source_port_range           = each.value.source_port_range
  destination_port_ranges     = each.value.destination_port_ranges
  source_address_prefix       = each.value.source_address_prefix
  destination_address_prefix  = each.value.destination_address_prefix
  resource_group_name         = azurerm_resource_group.rg.name
  network_security_group_name = azurerm_network_security_group.bastion-subnet-nsg.name
}
resource "azurerm_network_security_rule" "bastion-subnet-nsg-rules-2" {
  for_each                    = local.bastion-subnet-nsg-rules-2
  name                        = each.key
  direction                   = each.value.direction
  access                      = each.value.access
  priority                    = each.value.priority
  protocol                    = each.value.protocol
  source_port_range           = each.value.source_port_range
  destination_port_range      = each.value.destination_port_range
  source_address_prefix       = each.value.source_address_prefix
  destination_address_prefix  = each.value.destination_address_prefix
  resource_group_name         = azurerm_resource_group.rg.name
  network_security_group_name = azurerm_network_security_group.bastion-subnet-nsg.name
}
# Create virtual network hub
resource "azurerm_virtual_network" "vnet-hub" {
  name                = "vnet-hub"
  address_space       = ["172.31.10.0/23"]
  resource_group_name = azurerm_resource_group.rg.name
  location            = azurerm_resource_group.rg.location
}
# Create hub subnet
resource "azurerm_subnet" "hub-subnet" {
  name                 = "hub-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet-hub.name
  address_prefixes     = ["172.31.10.0/24"]
}
# Create AzureBastionSubnet
resource "azurerm_subnet" "AzureBastionSubnet" {
  name                 = "AzureBastionSubnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet-hub.name
  address_prefixes     = ["172.31.11.0/26"]
}
# Connect the security group to the hub-subnet
resource "azurerm_subnet_network_security_group_association" "nsg_association_hubsubnet" {
  subnet_id                 = azurerm_subnet.hub-subnet.id
  network_security_group_id = azurerm_network_security_group.hub-subnet-nsg.id
}
# Connect the security group to the AzureBastionSubnet
resource "azurerm_subnet_network_security_group_association" "nsg_association_bastionsubnet" {
  subnet_id                 = azurerm_subnet.AzureBastionSubnet.id
  network_security_group_id = azurerm_network_security_group.bastion-subnet-nsg.id
}
# Create virtual network spoke1
resource "azurerm_virtual_network" "vnet-spoke1" {
  name                = "vnet-spoke1"
  address_space       = ["172.31.20.0/24"]
  resource_group_name = azurerm_resource_group.rg.name
  location            = "southeastasia"
}
# Create spoke1 subnet
resource "azurerm_subnet" "spoke1-subnet" {
  name                 = "spoke1-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet-spoke1.name
  address_prefixes     = ["172.31.20.0/24"]
}
# Create Network Security Group
resource "azurerm_network_security_group" "spoke1-subnet-nsg" {
  name                = "spoke1-subnet-nsg"
  location            = azurerm_virtual_network.vnet-spoke1.location
  resource_group_name = azurerm_resource_group.rg.name
}
# Connect the security group to the spoke1 subnet
resource "azurerm_subnet_network_security_group_association" "nsg_association_vnet" {
  subnet_id                 = azurerm_subnet.spoke1-subnet.id
  network_security_group_id = azurerm_network_security_group.spoke1-subnet-nsg.id
}
# Config Peering hub-spoke1
resource "azurerm_virtual_network_peering" "hub-spoke1" {
  name                         = "hub-spoke1"
  resource_group_name          = azurerm_resource_group.rg.name
  virtual_network_name         = azurerm_virtual_network.vnet-hub.name
  remote_virtual_network_id    = azurerm_virtual_network.vnet-spoke1.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
}
# Config Peering spoke1-hub
resource "azurerm_virtual_network_peering" "spoke1-hub" {
  name                         = "spoke1-hub"
  resource_group_name          = azurerm_resource_group.rg.name
  virtual_network_name         = azurerm_virtual_network.vnet-spoke1.name
  remote_virtual_network_id    = azurerm_virtual_network.vnet-hub.id
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
}
# Create Bastion Public IP
resource "azurerm_public_ip" "basion-pip" {
  name                = "basion-pip"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  allocation_method   = "Static"
  sku                 = "Standard"
}
# Create Bastion
resource "azurerm_bastion_host" "bastion" {
  name                = "bastion"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name
  sku                 = "Standard"

  ip_configuration {
    name                 = "configuration"
    subnet_id            = azurerm_subnet.AzureBastionSubnet.id
    public_ip_address_id = azurerm_public_ip.basion-pip.id
  }
}
# Create Linux Virtual Machine's network interface
resource "azurerm_network_interface" "ubuntu1804-nic" {
  name                = "ubuntu1804-nic"
  location            = azurerm_virtual_network.vnet-spoke1.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "Configuration"
    subnet_id                     = azurerm_subnet.spoke1-subnet.id
    private_ip_address_allocation = "static"
    private_ip_address            = "172.31.20.4"
  }
}
# Create Linux Virtual Machine on Spoke1
resource "azurerm_linux_virtual_machine" "ubuntu1804" {
  name                  = "ubuntu1804"
  location              = azurerm_virtual_network.vnet-spoke1.location
  resource_group_name   = azurerm_resource_group.rg.name
  network_interface_ids = [azurerm_network_interface.ubuntu1804-nic.id]
  size                  = "Standard_D2s_v3"
  os_disk {
    name                 = "myOsDisk"
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "18.04-LTS"
    version   = "latest"
  }
  computer_name                   = "ubuntuvm"
  admin_username                  = "brian"
  admin_password                  = "Brian@20220525"
  disable_password_authentication = false
}
# Create srv01 Virtual Machine's network interface
resource "azurerm_network_interface" "srv01-nic" {
  name                = "srv01-nic"
  location            = azurerm_virtual_network.vnet-spoke1.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "Configuration"
    subnet_id                     = azurerm_subnet.spoke1-subnet.id
    private_ip_address_allocation = "static"
    private_ip_address            = "172.31.20.5"
  }
}
# Create srv01 Virtual Machine's on Spoke1
resource "azurerm_windows_virtual_machine" "srv01" {
  name                  = "srv01"
  resource_group_name   = azurerm_resource_group.rg.name
  location              = azurerm_virtual_network.vnet-spoke1.location
  size                  = "Standard_D2s_v3"
  admin_username        = "brian"
  admin_password        = "Brian@20220525"
  network_interface_ids = [azurerm_network_interface.srv01-nic.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }
}
# Create win2k22-adds network interface
resource "azurerm_network_interface" "win2k22-adds-nic" {
  name                = "win2k22-adds-nic"
  location            = azurerm_resource_group.rg.location
  resource_group_name = azurerm_resource_group.rg.name

  ip_configuration {
    name                          = "Configuration"
    subnet_id                     = azurerm_subnet.hub-subnet.id
    private_ip_address_allocation = "static"
    private_ip_address            = "172.31.10.4"
  }
}
# Create Windows Virtual Machine's on Hub
resource "azurerm_windows_virtual_machine" "win2k22-adds" {
  name                  = "win2k22-adds"
  resource_group_name   = azurerm_resource_group.rg.name
  location              = azurerm_resource_group.rg.location
  size                  = "Standard_D2s_v3"
  admin_username        = "brian"
  admin_password        = "Brian@20220525"
  network_interface_ids = [azurerm_network_interface.win2k22-adds-nic.id]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "MicrosoftWindowsServer"
    offer     = "WindowsServer"
    sku       = "2022-Datacenter"
    version   = "latest"
  }
}

````
- 初始化 Terraform 並且下載建立 Azure 資源群組所需要的模組<br>
  ```
  terraform init
  ```
- 套用 Terraform 在 Azure 環境中部署資源群組<br>
  ```
  terraform apply
  ```