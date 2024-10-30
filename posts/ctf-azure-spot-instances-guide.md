# Using Azure Spot Instances for CTF Labs - Student Account

## Overview
This guide addresses a common issue faced by students using Azure for the [Learn To Cloud Linux Challenge](https://github.com/learntocloud/ltc-linux-challenge) and provides a cost-effective solution using Azure Spot Instances.

## The Problem
When following the [Azure setup guide](https://github.com/learntocloud/ltc-linux-challenge/tree/main/aws) and running `terraform apply`, you might encounter this error:
```
Restrictions: 'Standard_B1s' is currently not available in location 'eastus'. 
Please try another size or deploy to a different location or different zone. 
See https://aka.ms/azureskunotavailable for details.
```

This happens because:
1. Student accounts have limited VM size availability
2. Certain regions might have capacity restrictions
3. Standard VMs can be costly for student credits

## The Solution: Azure Spot Instances

Azure Spot Instances provide a budget-friendly alternative with some trade-offs. They can be up to 90% cheaper than standard VMs, making them usable for student accounts.

### Prerequisites
- Azure Student Account
- Terraform installed
- Basic Azure knowledge
- LTC lab files cloned (`git clone https://github.com/learntocloud/ltc-linux-challenge.git`)

### Implementation Steps

1. **Navigate to your lab directory**
   ```bash
   cd ltc-linux-challenge/azure
   ```

2. **Locate the VM Resource**
   Find the "azurerm_linux_virtual_machine" "ctf_vm" resource in your `main.tf`

3. **Replace the VM Configuration**
   Replace the existing VM resource block with this Spot Instance configuration:

```shell
# Create a Linux virtual machine
resource "azurerm_linux_virtual_machine" "ctf_vm" {
  name                = "ctf-vm"
  resource_group_name = azurerm_resource_group.ctf_rg.name
  location            = azurerm_resource_group.ctf_rg.location
  
  # Spot Instance Configuration
  priority            = "Spot"
  eviction_policy     = "Deallocate"
  max_bid_price      = -1  # Pay up to the standard VM price
  
  # VM Configuration
  size                = "Standard_DC1s_v2"
  admin_username      = "azureuser"
  network_interface_ids = [
    azurerm_network_interface.ctf_nic.id,
  ]

  # Authentication
  admin_password                 = "CTFAdminPassword123!"
  disable_password_authentication = false

  # Disk Configuration
  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  # OS Image
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }

  # Custom Setup Script
  custom_data = base64encode(file("ctf_setup.sh"))
}
```

Now run and verify changes:
`terraform plan`

Then run:
`terraform apply`

And continue with the CTF Azure guide!

### Key Parameters Explained

| Parameter | Value | Description |
|-----------|-------|-------------|
| `priority` | "Spot" | Marks this as a Spot Instance |
| `eviction_policy` | "Deallocate" | VM is deallocated (not deleted) if capacity is needed |
| `max_bid_price` | -1 | Uses market price up to standard VM cost |
| `size` | "Standard_DC1s_v2" | A reliable size for student accounts for this project |

## Important Considerations

⚠️ **Key Points to Remember**:
- Spot VMs can be deallocated at any time - save your work often!
- You'll save up to 60-90% on VM costs
- Have a backup plan if your VM gets deallocated

## Troubleshooting Guide

### Common Issues and Solutions

**VM Creation Fails**
   - Try different regions (e.g., eastus2, westus2)
   - Attempt different VM sizes
   - Check your subscription quotas

## Additional Resources

- [LTC Discord](https://discord.gg/dr2kvtA726) - Join the community!
- [Learn To Cloud Main Site](https://learntocloud.guide)
- [Azure Student Account Setup](https://azure.microsoft.com/free/students)
- [Azure Spot VMs Pricing](https://azure.microsoft.com/pricing/details/virtual-machines/spot/)
- [Terraform Installation Guide](https://developer.hashicorp.com/terraform/install)
- [Azure CLI Installation](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

## Contributing

Found this helpful? Consider:
- Sharing your experiences on the [LTC Discord](https://discord.gg/dr2kvtA726)
- Adding more VM sizes that work with student accounts
- Contributing to the posts section of this repo
