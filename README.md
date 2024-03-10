# HashiCorp-Cloud-Platform

### Summary 
In this lab, we will explore -
- Self-managed Vault
- HashiCorp Cloud Platform
- Demonstrate HCP Vault

<br>

To explore more about vault & secret engine, check below links. 

<li class="masthead__menu-item">
<a href="https://github.com/myathway-lab/Vault-Installation---Use-Cases">Vault Installation & thier use cases</a>
</li>

<br>
 <li class="masthead__menu-item">
<a href="https://github.com/myathway-lab/Vault-KV-Versions-Diff">Vault-KV-Versions-Diff</a>
</li>


<br>
 <li class="masthead__menu-item">
<a href="https://github.com/myathway-lab/Vault-KV-V2">Vault-KV-V2</a>
</li>


 <br>


**1) Self-managed Vault**

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/c7cb3074-d78f-4ade-9361-519042957308)

Deploy - User needs to install/deploy the vault cluster on VM or K8s or AWS or etc. 

Operate - User needs to manage lifecycle and it is not 1 time step. Users need to make sure vault version up to date, patch if vault has vulnerabilities and so on. 

Adoption - User will use Vault services like secrets management.

*So, user is responsible for all of these operations.*

<br>

**2) HashiCorp Cloud Platform**

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/7ec02fcd-0c3d-4412-ad45-81b786f9dda6)

Rather than working on “Deploy & Operate”, user just want to focus on “Adoption” to get vault capabilities and values.

In HashiCorp cloud platform, HashiCorp focuses on “Deploy & Operate” which provides a fully managed implementation of Vault.  <br>  

User  just needs to manage “Adoption”. 

<br>

**2.1) HashiCorp Virtual Network (HVN)**

- HVN makes HCP networking possible.
- An HVN delegates an IPv4 CIDR range that HCP uses to automatically create resources in the cloud network.

For example, 

- Users want to secure their network. they want to make private network to the services.
- To enable single tendency in that private network, we can use peering connection between cloud provider and a HVN.
- This ensures that only trusted clients (users, applications, containers, etc.) running in the peered public cloud provider connect to Vault and avoid systems outside of the selected network attempting to connect.

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/b800e6e8-07c9-4235-becc-0e68fb24cd7a)

<br>

**3) Demonstrate HCP Vault**

In this HCP lab, we will -

3.1. Create the HCP account.

3.2. Create the new organization.

3.3. Create the service principle.

3.4. Manage an application.

3.5. Manage HCP using Terraform.

3.6. Access Vault Cluster

<br>

**3.1. Create the HCP account.**

- Go to “[Sign in | HashiCorp Cloud Platform](https://portal.cloud.hashicorp.com/sign-in)”
- Create the new account.
- Login to HCP.

<br>

**3.2. Create the new organization.**
    
![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/cb4eb841-7e4c-4e0a-93bf-345d3210ec4f)
    
<br>

**3.3. Create the service principle**

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/4bb59e7d-3301-4bae-a08f-f95eafefcce9)

- Export env variables as below.
- Then login to vault.

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/5c1d1d94-fb03-43e2-8d57-826ddc932124)

<br>

**3.4. Manage the application.**

- Create an application.
- Add new secrets. Let’s add client id & secret id of Org Service principle that we created in Step 3.3.

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/0c6039fc-6874-43b1-9965-31f7be646ca6)

- Once secrets were created, we need to reinitiate the vlt config to get the latest update.
- Then let’s verify the secrets information.


- Read secrets based on the Organization level or Project as below.

```markdown
org name - myathway-lab-org
project - project-default

vlt secrets list
vlt secrets get -organization myathway-lab-org HCP_CLIENT_ID
vlt secrets get -organization myathway-lab-org HCP_CLIENT_SECRET
vlt secrets get --project default-project HCP_CLIENT_ID
vlt secrets get --project default-project HCP_CLIENT_SECRET
vlt secrets get --app-name HELLO-CLOUD-MTLAB HCP_CLIENT_ID
vlt secrets get --app-name HELLO-CLOUD-MTLAB HCP_CLIENT_SECRET
vlt secrets get --plaintext HCP_CLIENT_ID
vlt secrets get --plaintext HCP_CLIENT_SECRET
```

<br>

**3.5. Manage HCP using Terraform.**

- Authenticate HCP using client id and secret id of HCP service principle.
- Create HVN.
- Create Vault Cluster inside HVN.

- versions.tf
    
    ```yaml
    terraform {
      required_providers {
        hcp = {
          source = "hashicorp/hcp"
          version = "0.81.0"
        }
      }
    }
    
    provider "hcp" {
      client_id     = "dvSxxxxxxxxxxxxxxxxxxxxxxxxxfNNs"
      client_secret = "JUG7R-kpxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxrtLLgvtp2E"
    }
    ```
    
- main.tf
    
    ```yaml
    resource "hcp_hvn" "mtlab-hvn" {
      hvn_id         = var.hvn_id
      cloud_provider = var.cloud_provider
      region         = var.region
    }
    
    resource "hcp_vault_cluster" "aws_hcp_vault_cluster" {
      hvn_id     = hcp_hvn.mtlab-hvn.hvn_id
      cluster_id = var.cluster_id
      tier       = var.tier
      public_endpoint = true
    }
    ```
    
- variables.tf
    
    ```yaml
    variable "hvn_id" {
      description = "The ID of the HCP HVN."
      type        = string
      default     = "mtlab-hvn"
    }
    
    variable "cluster_id" {
      description = "The ID of the HCP Vault cluster."
      type        = string
      default     = "mtlab-vault-cluster"
    }
    
    variable "region" {
      description = "The region of the HCP HVN and Vault cluster."
      type        = string
      default     = "us-west-2"
    }
    
    variable "cloud_provider" {
      description = "The cloud provider of the HCP HVN and Vault cluster."
      type        = string
      default     = "aws"
    }
    
    variable "tier" {
      description = "Tier of the HCP Vault cluster. Valid options for tiers."
      type        = string
      default     = "dev"
    }
    ```
    

- Apply terraform to create HVN cluster.

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/eecb939d-a413-448f-8889-20207b1ce675)

- Verify the HVN & Vault cluster are created successfully.

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/0ffcf2ef-cb68-4ffa-8d3e-acfcefeec676)

<br>

**3.6. Access vault cluster.**

- Go inside the “mtlab-vault-cluster”.
- Generate token. Copy the Admin Token.
- Copy the Public url.
- Access the url.

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/56efdb59-7621-4258-871d-501837c9a161)


![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/4ee8d870-3288-4e52-a1be-a96232d0196b)


- Vault login & check vault status & test secrets engines.

```yaml
vagrant@kindcluster-box:~/tf-demo/Vault-Session7-Jan27$ vault status
Key                      Value
---                      -----
Recovery Seal Type       awskms
Initialized              true
Sealed                   false
Total Recovery Shares    1
Threshold                1
Version                  1.15.4+ent
Cluster Name             vault-cluster-71cafbae
Cluster ID               bab461ab-3f60-ed4f-54b4-34e6dc69f94c
HA Enabled               true
HA Cluster               https://172.25.18.51:8201
HA Mode                  active
Last WAL                 341

vagrant@kindcluster-box:~/tf-demo/Vault-Session7-Jan27$ vault secrets enable kubernetes
Success! Enabled the kubernetes secrets engine at: kubernetes/
vagrant@kindcluster-box:~/tf-demo/Vault-Session7-Jan27$
```

![image](https://github.com/myathway-lab/HashiCorp-Cloud-Platform/assets/157335804/38a42620-1d76-462a-b6ff-e427ca0e87a1)
<br>
