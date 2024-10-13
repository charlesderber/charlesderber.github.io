**Guide on How to Migrate Local Terraform State to Terraform Cloud with the Microsoft Azure Provider**

**Introduction**
Many developers start by managing Terraform state files locally, but as teams grow or infrastructure management becomes more complex, transitioning to Terraform Cloud brings benefits such as remote state management, secure storage, and collaboration features.

In this article, we'll walk through the process of migrating an existing local Terraform state to Terraform Cloud.
<br>

**Scenario**

You have a local state file (terraform.tfstate) that manages an existing Azure virtual machine. Now, you want to migrate that state to Terraform Cloud while ensuring the VM remains untouched and managed seamlessly after migration.

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/1.jpg)

<br>

**Why Migrate to Terraform Cloud?**

- **Centralized state management** : Avoid the risk of losing local state files or overwriting them.
- **Team collaboration** : Multiple team members can work on infrastructure without conflict.
- **Secure storage** : State files are stored securely with encryption and access control.
- **Enhanced workflows** : Features like remote operations, version control, and notifications.

<br>

**Prerequisites**

Before we begin the migration, ensure the following:

- An existing Terraform configuration with a local state file (e.g., terraform.tfstate).
- A Terraform Cloud account: If you don’t have one, you can [Sign Up](https://app.terraform.io/public/signup/account) at Terraform Cloud.
- The Terraform CLI is installed and configured on your local machine. 

<br>

**Create a Workspace in Terraform Cloud**

If you’re using Terraform Cloud for the first time, head over to Terraform Cloud and sign up for an account. Once you have signed up and logged in, you need to create an organization, which serves as the top-level container for workspaces and resources.

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/2.jpg)

HCP Terraform organizes your infrastructure resources by workspaces. A workspace contains infrastructure resources, variables, state data, and run history

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/3.jpg)

<br>

Now that the workspace is created, you'll need to update your main.tf (or similar) file locally to point to Terraform Cloud as the backend.

```
  cloud {
    organization = "org-cd-azure"
    workspaces {
      name = "migrate-tfstate"
    }
  }
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "=4.1.0"
    }
  }
}
```
<br>

When migrating from local state to Terraform Cloud for the first time, you'll need to authenticate from the CLI to store the Terraform Cloud API token locally. This allows Terraform to securely interact with Terraform Cloud.

Before you migrate the state, you need to authenticate Terraform with Terraform Cloud using your API token. Run the following command in your terminal to log in:

```
terraform login
```

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/4.jpg)

<br>

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/5.jpg)

<br>

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/6.jpg)

By the time you are reading the article the token will be expired :)
<br>

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/7.jpg)

You should see the below screenshot if the authentication was successful.
<br>

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/8.jpg)

<br>

**Last thing is to configure Azure Environment Variables in Terraform Cloud.**
<br>

In Terraform Cloud, you'll need to configure environment variables for Azure authentication. Set the following variables in your workspace:

- **ARM_CLIENT_ID**: The client ID of the Azure service principal.
- **ARM_CLIENT_SECRET**: The client secret of the service principal. (secret value)
- **ARM_SUBSCRIPTION_ID**: Your Azure subscription ID.
- **ARM_TENANT_ID**: Your Azure tenant ID.

Go to your workspace in Terraform Cloud. Navigate to the Variables section. Under Environment Variables, click + Add variable. Add the variables listed above, making sure to mark **ARM_CLIENT_SECRET** as Sensitive. In my demo I have marked all the environment variable values as sensitive. 

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/9.jpg)

<br>

**Initialize Terraform and Migrate State**

Now that you've configured the backend by running the below command, reinitialize Terraform to connect to Terraform Cloud:

```
terraform init
```

Terraform will detect the backend change in the terraform block code as we configured the organization and workspaces. It will ask if you want to migrate your local state to the remote backend. Type **yes** to confirm.

**What happens here**: Terraform copies your local state file (terraform.tfstate) to Terraform Cloud. The state migration is non-destructive and does not modify your existing infrastructure (like the VM).
<br>

If your local Terraform version is older than the version required by Terraform Cloud (i.e., **1.9.0 or newer**) or If you can't upgrade your local version immediately, you can bypass the version check using the **-ignore-remote-version** flag. By default the terraform version uses latest under the workspace setting you can adjust based on the local machine terraform version. 

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/16.jpg)
<br>

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/10.jpg)

<br>

For the sake of this article we will ignore by running below command and enter **yes** when prompted for confirming migrating the state to Terraform cloud workspace. 

```
terraform init -ignore-remote-version
```

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/11.jpg)

<br>

You will notice the state is now available on **Terraform cloud workspace** overview page

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/12.jpg)

<br>

Let's go ahead and clean up the Virtual machine by running terraform destroy and you will notice the **CLI** will redirect the process to Terraform Cloud and you can confirm deletion by providing custom message and confirming apply. 

<br>

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/13.jpg)

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/14.jpg)

<br>

You can go ahead and delete the local terraform state file and continue using statefile Terraform cloud workspace.
<br>

![alt text](/docs/articles/tf-remotebackend-az-tfcloud/15.jpg)

