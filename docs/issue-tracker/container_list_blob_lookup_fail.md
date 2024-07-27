**Azure Storage Account Connection Issues in GitHub Runner with Terraform (TCP:53)**
<br>
While running the GitHub runner to connect to an Azure Storage Account for Terraform state file management, I encountered the following error:

"Initializing the backend... 17╷ 18│ Error: Failed to get existing workspaces: containers.Client#ListBlobs: Failure sending request: StatusCode=0 -- Original Error: Get ""https://storageaccountname.blob.core.windows.net/statefile?comp=list&prefix=terraform.tfstateenv%3A&restype=container"": ==dial tcp: lookup storageaccountname.blob.core.windows.net on 127.0.0.53:53: no such host== 19│"
<br>

**Background about the setup:**

- GitHub Runner: Running in a virtual network (VNet) subnet within Spoke Subscription A.
- Azure Storage Account: Configured with a private endpoint in a different VNet subnet within Spoke Subscription B.
- Network Configuration:
    - Spoke Subscription B's VNet is peered with Spoke Subscription A's VNet.

The error message indicates a DNS resolution issue where the GitHub runner cannot resolve the storage account’s private endpoint (storageaccountname.blob.core.windows.net).
<br>

**Solution:**

To resolve the issue, I linked the Spoke Subscription B virtual network (where the storage account is configured) to the private DNS zone privatelink.blob.core.windows.net. Additionally, I ensured that the A record for the storage account's private endpoint was correctly configured.

After linking the virtual network and confirming the DNS records, the GitHub runner was able to resolve and connect to the Azure Storage Account successfully. The Terraform initialization error was resolved, and the state file was accessible.

Networking and DNS configurations can often cause connectivity issues, especially in multi-subscription and peered virtual network environments. Ensuring proper DNS resolution through private endpoints and verifying the setup can save a lot of troubleshooting time.