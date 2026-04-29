The PIM step could not be completed because Microsoft Entra Privileged Identity Management requires Microsoft Entra ID P2 or Microsoft Entra ID Governance.
The current tenant uses Microsoft Entra ID Free. Therefore, the permanent role assignment from the previous step is used as a simplified lab variant.

The AzureActivity table works in Log Analytics and shows activity for diagnostic settings.
The role assignment events could be verified directly through Azure Activity Log using Azure CLI, where Microsoft.Authorization/roleAssignments/write was 
shown as Started and Succeeded. However, the events did not appear in Log Analytics within the waiting period, likely due to export or ingestion delay.

Terraform init and validate were executed. Terraform apply returned a conflict because the resources had already been created manually with Azure CLI.
To manage existing resources with Terraform, they need to be imported into the Terraform state, or the environment needs to be created from scratch using only Terraform.

The least privilege test could not be verified correctly because the test was run with the administrator account, not with a separate 
user that is only a member of GRP-AZ-Security-Audit. The az group create command therefore succeeded, which shows that the current account has 
higher permissions than the audit role. The test resource group rg-should-fail was removed after the test.
