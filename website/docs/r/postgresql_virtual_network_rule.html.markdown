---
layout: "azurerm"
page_title: "Azure Resource Manager: azurerm_postgresql_virtual_network_rule"
sidebar_current: "docs-azurerm-resource-database-sql-virtual-network-rule"
description: |-
  Manages a PostgreSQL Virtual Network Rule.
---

# azurerm_postgresql_virtual_network_rule

Allows you to add, update, or remove an Azure PostgreSQL server to a subnet of a virtual network.

## Example Usage

```hcl
resource "azurerm_resource_group" "example" {
  name     = "example-postgresql-server-vnet-rule"
  location = "West US"
}

resource "azurerm_virtual_network" "vnet" {
  name                = "example-vnet"
  address_space       = ["10.7.29.0/29"]
  location            = "${azurerm_resource_group.example.location}"
  resource_group_name = "${azurerm_resource_group.example.name}"
}

resource "azurerm_subnet" "subnet" {
  name                 = "example-subnet"
  resource_group_name  = "${azurerm_resource_group.example.name}"
  virtual_network_name = "${azurerm_virtual_network.vnet.name}"
  address_prefix       = "10.7.29.0/29"
  service_endpoints    = ["Microsoft.Sql"]
}

resource "azurerm_postgresql_server" "postgresql_server" {
  name                = "postgresql-server-1"
  location            = "${azurerm_resource_group.test.location}"
  resource_group_name = "${azurerm_resource_group.test.name}"

  sku {
    name = "B_Gen4_2"
    capacity = 2
    tier = "Basic"
    family = "Gen4"
  }

  storage_profile {
    storage_mb = 5120
    backup_retention_days = 7
    geo_redundant_backup = "Disabled"
  }

  administrator_login = "psqladminun"
  administrator_login_password = "H@Sh1CoR3!"
  version = "9.5"
  ssl_enforcement = "Enabled"
}

resource "azurerm_postgresql_virtual_network_rule" "postgresqlsql_virtual_network_rule" {
  name                = "postgresql-vnet-rule"
  resource_group_name = "${azurerm_resource_group.example.name}"
  server_name         = "${azurerm_postgresql_server.postgresql_server.name}"
  subnet_id           = "${azurerm_subnet.subnet.id}"
}
```

## Argument Reference

The following arguments are supported:

* `name` - (Required) The name of the SQL virtual network rule. Cannot be empty and must only contain alphanumeric characters and hyphens. Cannot start with a number, and cannot start or end with a hyphen. Changing this forces a new resource to be created.

~> **NOTE:** `name` must be between 1-128 characters long and must satisfy all of the requirements below:
1. Contains only alphanumeric and hyphen characters
2. Cannot start with a number or hyphen
3. Cannot end with a hyphen

* `resource_group_name` - (Required) The name of the resource group where the PostgreSQL server resides. Changing this forces a new resource to be created.

* `server_name` - (Required) The name of the SQL Server to which this PostgreSQL virtual network rule will be applied to. Changing this forces a new resource to be created.

* `subnet_id` - (Required) The ID of the subnet that the PostgreSQL server will be connected to.

~> **NOTE:** Due to [a bug in the Azure API](https://github.com/Azure/azure-rest-api-specs/issues/3719) this resource currently doesn't expose the `ignore_missing_vnet_service_endpoint` field and defaults this to `false`. Terraform will check during the provisioning of the Virtual Network Rule that the Subnet contains the Service Rule to verify that the Virtual Network Rule can be created.

## Attributes Reference

The following attributes are exported:

* `id` - The ID of the PostgreSQL Virtual Network Rule.

## Import

PostgreSQL Virtual Network Rules can be imported using the `resource id`, e.g.

```shell
terraform import azurerm_postgresql_virtual_network_rule.rule1 /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/servers/myserver/virtualNetworkRules/vnetrulename
```
