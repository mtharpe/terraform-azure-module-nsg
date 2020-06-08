![Terraform GitHub Actions](https://github.com/mtharpe/terraform-azure-module-nsg/workflows/Terraform%20GitHub%20Actions/badge.svg)

# terraform-azure-module-nsg

## Create a network security group

This Terraform module deploys a Network Security Group (NSG) in Azure and optionally attach it to the specified vnets.

This module is a complement to the [Azure Network](https://registry.terraform.io/modules/Azure/network/azurerm) module. Use the network_security_group_id from the output of this module to apply it to a subnet in the Azure Network module.
NOTE: We are working on adding the support for applying a NSG to a network interface directly as a future enhancement.

This module includes a a set of pre-defined rules for commonly used protocols (for example HTTP or ActiveDirectory) that can be used directly in their corresponding modules or as independent rules.

## Usage with the generic module

The following example demonstrate how to use the network-security-group module with a combination of predefined and custom rules.

```hcl
resource "azurerm_resource_group" "test" {
  name     = "my-resources"
  location = "West Europe"
}

module "network-security-group" {
  source                = "Azure/network-security-group/azurerm"
  resource_group_name   = azurerm_resource_group.test.name
  security_group_name   = "nsg"
  source_address_prefix = ["10.0.3.0/24"]
  predefined_rules = [
    {
      name     = "SSH"
      priority = "500"
    },
    {
      name              = "LDAP"
      source_port_range = "1024-1026"
    }
  ]
  custom_rules = [
    {
      name                   = "myhttp"
      priority               = "200"
      direction              = "Inbound"
      access                 = "Allow"
      protocol               = "tcp"
      destination_port_range = "8080"
      description            = "description-myhttp"
    }
  ]
  tags = {
    environment = "dev"
    costcenter  = "it"
  }
}
```

## Usage with the pre-defined module

The following example demonstrate how to use the pre-defined HTTP module with a custom rule for ssh.

```hcl
resource "azurerm_resource_group" "test" {
  name     = "my-resources"
  location = "West Europe"
}

module "network-security-group" {
  source              = "Azure/network-security-group/azurerm//modules/HTTP"
  resource_group_name = azurerm_resource_group.test.name
  security_group_name = "nsg"
  custom_rules = [
    {
      name                   = "ssh"
      priority               = "200"
      direction              = "Inbound"
      access                 = "Allow"
      protocol               = "tcp"
      destination_port_range = "22"
      source_address_prefix  = "VirtualNetwork"
      description            = "ssh-for-vm-management"
    }
  ]
  tags = {
    environment = "dev"
    costcenter  = "it"
  }
}
```

## License and Maintainer

Maintainer:: HashiCorp (<hello@hashicorp.com>)

Source:: https://github.com/mtharpe/terraform-azure-nsg

Issues:: https://github.com/mtharpe/terraform-azure-nsg/issues

License:: Apache-2.0
