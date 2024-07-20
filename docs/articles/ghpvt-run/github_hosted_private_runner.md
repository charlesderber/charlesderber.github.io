**Securing GitHub-Hosted Runners with Azure Private Networking: A Step-by-Step Guide**
<br>
**Introduction**

Private networking for GitHub-hosted runners allows organizations to run their CI/CD workflows securely within an Azure virtual network (VNET). This capability provides enhanced security by isolating the runner environment from the public internet, ensuring that sensitive code and data remain protected during the build and deployment processes

<br>

![Architecture Diagram](/docs/articles/ghpvt-run/diagram.png)

<br>

Github-hosted runners execute workflow triggered by events in the GitHub repositories and are run on virtual machines hosted by GitHub in the cloud. A virtual network in Azure where the GitHub-hosted runners will execute the workflow. NetworkSettings resource on Azure is created to associate the Azure subnet with GitHub-hosted runners, enabling private networking. You can additionally configure network security group on the subnet to control the inbound or outbound traffic to secure communication within the virtual network and external services. 

<br>

**Prerequisites**

Before starting, ensure you meet the following pre-requisites: 

- GitHub organization with teams or enterprise plan
- Azure subscription with GitHub provider registration and contributor role on subscription

<br>

**Lets Configure:**

**On Azure Part**
First thing first register the GitHub provider on the subscription where we plan to deploy the GitHub-hosted runner virtual machine. 

![alt text](/docs/articles/ghpvt-run/17.png)

![alt text](/docs/articles/ghpvt-run/18.png)

Enable subnet delegation for the subnet where we plan to deploy the GitHub-hosted runner virtual machine

![alt text](/docs/articles/ghpvt-run/19.png)
![alt text](/docs/articles/ghpvt-run/20.png)

Run the below command to create NETWORK_SETTING_RESOURCE

az resource create --resource-group rg-gh-pvtrunner  --name NETWORK_SETTINGS_RESOURCE --resource-type GitHub.Network/networkSettings --properties "{ \"location\": \"norwayeast\", \"properties\" : {  \"subnetId\": \"/subscriptions/YOUR_SUBCRIPTION_ID/resourceGroups/rg-gh-pvtrunner/providers/Microsoft.Network/virtualNetworks/vnet-ghrunner/subnets/default\", \"businessId\": \"YOUR_ORGANIZATION_ID\" }}" --is-full-object --output table --query "{GitHubId:tags.GitHubId, name:name}" --api-version 2024-04-02

The output of the command should be as below: 

![alt text](/docs/articles/ghpvt-run/21.png)

**Moving to GitHub**

We can restrict the configuration of GitHub hosted compute networking feature to use at enterprise or organization level under enterprise policies, for this article we enable at the organization level. 

![alt text](/docs/articles/ghpvt-run/1.png)

<br>

Go to the organization settings and select Hosted compute networking. 
![alt text](/docs/articles/ghpvt-run/2.png)

<br>

Click on new network configuration and select Azure private network.

![alt text](/docs/articles/ghpvt-run/3.png)

<br>

Put the configuration name "sub-management-ghrunner" and click on Add virtual Network.

![alt text](/docs/articles/ghpvt-run/4.png)

<br>

Click on add Azure virtual network before creating configuration and enter the GitHubId previously generated for the resource NETWORK_SETTINGS_RESOURCE.

![alt text](/docs/articles/ghpvt-run/5.png)

<br>

Verify the new network configuration and click on create configuration.

![alt text](/docs/articles/ghpvt-run/6.png)

<br>

Create a runner group under organization settings -> runner groups. We can select a particular repository in the organization or specific repository to use the runner group, select the network configuration for this runner group which we created above and click on create group.

![alt text](/docs/articles/ghpvt-run/7.png)

<br>

Runner group is now created but without a runner and because there is  no runner created in the runner group it won´t appear in the repository action runner group settings. 

![alt text](/docs/articles/ghpvt-run/8.png)
<br>

![alt text](/docs/articles/ghpvt-run/9.png)

<br>

Go inside the runner group created "demo-privatergh" and create a runner by selecting new GitHub-hosted runner option

![alt text](/docs/articles/ghpvt-run/10.png)

<br>

Review the runner configuration and click on create runner

![alt text](/docs/articles/ghpvt-run/11.png)

<br>

Now the runner is created and ready for the job. Also now it will be visible under the repository setting -> action runner 

![alt text](/docs/articles/ghpvt-run/12.png)

![alt text](/docs/articles/ghpvt-run/13.png)

<br>

For testing purpose lets create a workflow and configure runs: "private-runner01" which we created above. 

![alt text](/docs/articles/ghpvt-run/14.png)

<br>

Under repository action -> job summary we can monitor and validate the workflow job uses the GitHub-hosted private runner.

![alt text](/docs/articles/ghpvt-run/15.png)

<br>

While the job is running we can also validate on the Azure part there are new network interfaces which is created and when the job is finish the NICs will be deleted automatically. 

![alt text](/docs/articles/ghpvt-run/16.png)

<br>

**Conclusion**

Configuring private networking for GitHub-hosted runners with Azure ensures that your CI/CD workflows run securely within a VNET, leveraging Azure's robust networking capabilities. By following the steps outlined in this guide, enterprise owners can effectively set up and manage their private networking configurations, enhancing the security and efficiency of their development processes.