# Deploying .Net Web App from github using Terraform
The web app gets deployed directly from github repo to web app service which connects to database in the SQL server.

## Resources:
  - .NET Web App
  - Azure SQL server
  - Azure Web App
  - Github Repo
  - Terraform
  <br><br><br>
 
![webapp-arch](https://github.com/salman-cissp/tf-webapp/assets/134168108/4dcdc07a-41aa-431b-a97c-fdc68539f0ad)

The web app gets deployed directly from github repo to web app service which connects to database in the SQL server. 

## Terraform code :

```
terraform {
  required_providers {
    azurerm = {
      source = "hashicorp/azurerm"
      version = "2.92.0"
    }
  }
}

provider "azurerm" {
  
  features {}
}


locals {
  resource_group="app-grp"
  location="North Europe"  
}


resource "azurerm_resource_group" "app_grp"{
  name=local.resource_group
  location=local.location
}

resource "azurerm_app_service_plan" "app_plan1000" {
  name                = "app-plan1000"
  location            = azurerm_resource_group.app_grp.location
  resource_group_name = azurerm_resource_group.app_grp.name

  sku {
    tier = "Basic"
    size = "B1"
  }
}

resource "azurerm_app_service" "webapp" {
  name                = "productappwebapp01062023"
  location            = azurerm_resource_group.app_grp.location
  resource_group_name = azurerm_resource_group.app_grp.name
  app_service_plan_id = azurerm_app_service_plan.app_plan1000.id
     source_control {
    repo_url           = "https://github.com/irtiash/ProductApp"
    branch             = "master"
    manual_integration = true
    use_mercurial      = false
  }
  depends_on=[azurerm_app_service_plan.app_plan1000]
}

resource "azurerm_sql_server" "app_server_6008089" {
  name                         = "appserver6008089"
  resource_group_name          = azurerm_resource_group.app_grp.name
  location                     = "North Europe"  
  version             = "12.0"
  administrator_login          = "sqladmin"
  administrator_login_password = "Azure@123"
}

resource "azurerm_sql_database" "app_db" {
  name                = "appdb"
  resource_group_name = azurerm_resource_group.app_grp.name
  location            = "North Europe"  
  server_name         = azurerm_sql_server.app_server_6008089.name
   depends_on = [
     azurerm_sql_server.app_server_6008089
   ]
}

resource "azurerm_sql_firewall_rule" "app_server_firewall_rule_Azure_services" {
  name                = "app-server-firewall-rule-Allow-Azure-services"
  resource_group_name = azurerm_resource_group.app_grp.name
  server_name         = azurerm_sql_server.app_server_6008089.name
  start_ip_address    = "0.0.0.0"
  end_ip_address      = "0.0.0.0"
  depends_on=[
    azurerm_sql_server.app_server_6008089
  ]
}

resource "azurerm_sql_firewall_rule" "app_server_firewall_rule_Client_IP" {
  name                = "app-server-firewall-rule-Allow-Client-IP"
  resource_group_name = azurerm_resource_group.app_grp.name
  server_name         = azurerm_sql_server.app_server_6008089.name
  start_ip_address    = "82.12.90.169"
  end_ip_address      = "82.12.90.169"
  depends_on=[
    azurerm_sql_server.app_server_6008089
  ]
}

resource "null_resource" "database_setup" {
  provisioner "local-exec" {
      command = "sqlcmd -S appserver6008089.database.windows.net -U sqladmin -P Azure@123 -d appdb -i init.sql"
  }
  depends_on=[
    azurerm_sql_server.app_server_6008089
  ]
}
```

It creates a table with one database entry using sqlcmd <br> <br>
` sqlcmd -S appserver6008089.database.windows.net -U sqladmin -P Azure@123 -d appdb -i init.sql `
