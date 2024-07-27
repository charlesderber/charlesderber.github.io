**Error: Inconsistent dependency lock file while running Terraform Apply**
<br>
While running terraform apply to deploy Azure resources, I encountered the following error:


Run terraform apply -input=false -auto-approve tfplan
/home/runner/work/_temp/03c666f4-4e7b-47b8-a30f-a9fe587f6fa4/terraform-bin apply -input=false -auto-approve tfplan
Acquiring state lock. This may take a few moments...
Releasing state lock. This may take a few moments...
╷
│ Error: Inconsistent dependency lock file
│ 
│ The given plan file was created with a different set of external dependency
│ selections than the current configuration. A saved plan can be applied only
│ to the same configuration it was created from.
│ 
│ Create a new plan from the updated configuration.
<br>


**Background**

I have Terraform resource modules with the version specified as ==required_version = ">= 1.3.0==" in the Terraform version block. However, in the module caller Terraform version file, there was no required_version parameter.
<br>

**My setup includes two GitHub workflows:**

Terraform Plan Workflow: Generates a plan and creates a package for the apply workflow.
Terraform Apply Workflow: Downloads the package from the plan workflow and applies it.
<br>

**Problem Identification**
During the plan workflow, the Terraform initialization used provider versions as follows:

Initializing provider plugins...
- terraform.io/builtin/terraform is built in to Terraform
- Finding integrations/github versions matching "~> 5.45"...
- Finding hashicorp/time versions matching ">= 0.9.1"...
- Finding hashicorp/azurerm versions matching ">= 3.7.0, >= 3.61.0"...
- Finding azure/azapi versions matching ">= 1.11.0"...
- Finding hashicorp/http versions matching ">= 3.4.1"...
- Installing integrations/github v5.45.0...
- Installed integrations/github v5.45.0 (signed by a HashiCorp partner, key ID 38027F80D7FD5FB2)
- Installing hashicorp/time v0.11.2...
- Installed hashicorp/time v0.11.2 (signed by HashiCorp)
- Installing hashicorp/azurerm v3.106.1...
- Installed hashicorp/azurerm v3.106.1 (signed by HashiCorp)

The plan workflow used ==azurerm v3.106.1==. However, during the apply workflow, Terraform initialized with a different version:

Initializing provider plugins...
- terraform.io/builtin/terraform is built in to Terraform
- Finding hashicorp/http versions matching ">= 3.4.1"...
- Finding integrations/github versions matching "~> 5.45"...
- Finding hashicorp/azurerm versions matching ">= 3.7.0, >= 3.61.0"...
- Finding hashicorp/time versions matching ">= 0.9.1"...
- Finding azure/azapi versions matching ">= 1.11.0"...
- Installing hashicorp/azurerm v3.107.0...
- Installed hashicorp/azurerm v3.107.0 (signed by HashiCorp)

The apply workflow used ==azurerm v3.107.0==, leading to a version mismatch and the Inconsistent dependency lock file error.
<br>

**Solution**

To fix the issue, I ensured both Terraform version files specified the same version in the terraform block with the required_version parameter.

Module Caller File:

terraform {
  required_version = ">= 1.3.0"
  required_providers {
    azapi = {
      source  = "azure/azapi"
      version = ">= 1.11.0"
    }
  }
}

By updating the code to include required_version = ">= 1.3.0" in both the resource module and the module caller, the inconsistency was resolved. Both plan and apply workflows executed successfully without further errors.
<br>

**Conclusion**

Ensuring consistent Terraform versions across different workflows and modules is crucial to avoid dependency lock file errors. By aligning the Terraform version configurations, the deployment process becomes smoother and more reliable. If you encounter similar issues, double-check your version specifications to ensure consistency.