Azure & Terraform (Modular Infrastructure)

Welcome to the Azure Terraform Infrastructure Project! This repository demonstrates how to reliably build, operate, and maintain cloud infrastructure in Microsoft Azure using modular, production-ready Terraform code.

It covers: 

Virtual networks & Linux web servers
Automated patch management
Full production parameterization (“one-click” environments)

Table of Contents:

Project Goals
Repository Structure
Assignment 1: Virtual Machines & Networking
Assignment 2: System Patch Management
Assignment 3: Full Composable Environment
Architecture Diagram
Before you begin (Prerequisites)
How to Use This Project
Variable Configuration (terraform.tfvars)
Extending and Troubleshooting
Monitoring Patch Compliance
Cleaning Up



Project Goals:
Real-world maintainability: All infrastructure is built from small, well-encapsulated modules you can easily reuse across many projects.
Security and compliance: VM firewalls are tight by default, servers use SSH keys, and patching is automated.
Observability: Enables true monitoring and compliance with Log Analytics and Azure Automation.
Flexibility by configuration: Everything is parameterized, so you can create many variations of your environment for test, dev, or prod with minimal edits.

Repository Structure:

.
├── main.tf                # Root orchestrator for any assignment/project
├── variables.tf           # Main variable definitions, propagated to modules
├── outputs.tf             # Root outputs for consumption/integration
├── terraform.tfvars       # Safely override key variables and secrets here
├── providers.tf           # Provider configuration for AzureRM
├── cloud-init-nginx.yml   # Cloud-init for VM bootstrapping
├── README.md              # This documentation file
└── modules/               # Reusable infrastructure modules
    ├── vnet/              # VNet and Subnet
    ├── nsg/               # NSG (firewall rules module)
    ├── vm/                # Linux Virtual Machine (cloud-init enabled)
    ├── automation_account/
    ├── log_analytics/
    └── update_management/ # (Assignments 2 & 3)

Assignment 1: Virtual Machines & Networking
Goal: Deploy a secure Linux web server, fully modularized and ready for real production use.

Implementation:
modules/vnet
Provisions a VNet and subnet, which lets you scale out new workloads or segments with no code duplication.

modules/nsg
Built to your policy—includes fine-grained rules for opening minimum network access (SSH/HTTP).

modules/vm
Creates a modern Ubuntu VM with cloud-init so you can inject your NGINX or Apache install via one file, then automate service startup.

Outputs:
VM public IP
Internal subnet ID (useful for chaining/config)
Resource IDs so you can hook in monitoring and patch management without guesswork


Assignment 2: System Patch Management
Goal: Centralize and automate patch compliance & reporting via Azure Automation.

Implementation:
modules/automation_account
Creates a ready-to-go Automation Account; all patching and future runbooks flow from here.

modules/log_analytics
Sends VM and compliance telemetry to a Log Analytics workspace (required for Update Management).

modules/update_management
Centralizes VM onboarding for scheduled patching—works for one or many VMs.

Scenario covered:
Pick schedule in terraform.tfvars (for example, “Sunday at 3am UTC” for patch windows).
VM is enrolled automatically and reports compliance status to Log Analytics.
No need to work with manual agent setups or portal steps—repeatable and scalable.

Assignment 3: Full Composable Environment
Goal: Bundle all foundational modules into a single, parameterizable site. Control everything (region, naming, patch timing, tags) with a single config.

Implementation:
Deploys VNet, NSG, Linux VM, Automation Account, Log Analytics Workspace, AND auto-onboards the VM to weekly/parameterized patch schedule.
Tagging, naming, and scheduling are all 100% variable-driven: you can spin up a prod, dev, or test setup just by copying and editing one tfvars file.

Architecture Diagram

[Resource Group]
├─[Virtual Network + Subnet]
│    └─[Linux VM + NIC + Public IP]
│        └─[NSG (Firewall)]
├─[Automation Account] ──[Update Management Schedule]
└─[Log Analytics Workspace] (for compliance tracking)

Before you begin (Prerequisites)
Terraform: Version 1.4 or greater (https://www.terraform.io/downloads)

Azure CLI: For authentication and resource management (https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

SSH Key Pair: For secure VM access. Generate with:
ssh-keygen -t rsa -b 4096

How to Use This Project:
Clone or copy this repo.

Customize terraform.tfvars for:

Region, resource names, VM size, tags, SSH key path, and scheduling.

Optionally customize cloud-init-nginx.yml if you want to bootstrap Apache or do more than just install NGINX.

Run Terraform workflow:


terraform init
terraform plan
terraform apply
Retrieve outputs: Most critical values (such as VM public IP) will be shown at the end or by running:


terraform output
Access the web server:

ssh azureuser@<public_ip>

http://<public_ip>/ in a browser (for the default NGINX welcome page)

Variable Configuration (terraform.tfvars)
Every assignment here is configurable via a single input file.
Here are the most important values:

region                = "East US"
resource_group_name   = "webdemo-env"
tags = {
  Environment = "Production"
  Owner       = "DevOpsTeam"
}
vnet_name             = "webdemo-vnet"
vnet_address_space    = ["10.30.0.0/16"]
subnet_name           = "webdemo-subnet"
subnet_prefix         = "10.30.1.0/24"
nsg_name              = "webdemo-nsg"
vm_name               = "webdemo-vm"
vm_size               = "Standard_B2s"
admin_username        = "azureuser"
admin_ssh_key_path    = "~/.ssh/id_rsa.pub"

automation_account_name = "webdemo-auto"
workspace_name          = "webdemo-laworkspace"
update_config_name      = "weekly-linux-patch"
schedule_start_time     = "2025-08-26T03:00:00Z"
schedule_week_days      = ["Sunday"]
schedule_name           = "patch-sunday-early"
schedule_time_zone      = "UTC"


Extending and Troubleshooting:
Adding more VMs?
Clone/reuse the vm module as many times as needed, or use count/for_each patterns.

Locked-down inbound rules?
Edit the nsg_rules to only permit your corporate IPs for SSH.

Use custom images, bootstrap scripts, or advanced security?
Just extend the modules—add more input variables as needed, the structure is built for this.

Debugging:
Use terraform plan and terraform state show <resource> often. They’re your best friends!
Azure Portal always reflects live state if you want to check actual deployments.

Monitoring Patch Compliance:
In the Azure Portal:
Browse to your Log Analytics Workspace.
Click Update Management under “Solutions”.
Here you'll see compliance dashboards—number of updated, pending, failed patches per VM.

For custom reporting:
Use Log Analytics Queries, for example:

UpdateSummary
| summarize by Computer, UpdateState
Troubleshooting agents/compliance:

Ensure VM identity/permissions are correct if status is unknown.

Check Automation Account + Log Analytics connections.

Cleaning Up:
When you’re done, destroy resources to avoid unexpected costs:
terraform destroy
