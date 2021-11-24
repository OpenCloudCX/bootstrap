# Full example of OpenCloudCX setup in AWS

This repository contains a framework to use for creation of an OpenCloudCX cluster. After cloning this repository, refer to the below sections for configuration.

# Toolsets

|Toolset|Links|Notes|
|---|---|---|
|Terraform&nbsp;(at least version&nbsp;1.0.8)|[Download](https://releases.hashicorp.com/terraform/1.0.8/) | Terraform is distributed as a single binary. Install Terraform by unzipping it and moving it to a directory included in your system's [PATH](https://superuser.com/questions/284342/what-are-path-and-other-environment-variables-and-how-can-i-set-or-use-them). <br />**This project has been tested with Terraform 1.0.8 -- Will be updated as newer versions are tested.**|
|AWS&nbsp;CLI|[Instructions](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) \|\| [Configuration](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html)|This link provides information for getting started with version 2 of the AWS Command Line Interface (AWS CLI)|
|kubectl|[Instructions](https://kubernetes.io/docs/tasks/tools/#kubectl)|Allows commands to be executed against Kubernetes clusters|
| Git |[Instructions](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)| Need to run this command to avoid a CRLF issues: git config --global core.autocrlf input|

# Setup

Once all toolsets are installed and verified to be operational, configure the cloned bootstrap project.

## AWS S3 State Bucket
If remote state storge is desired, follow the instructions in this section. If not, skip to *Project Variables*. This file is not required for successful environment generation.

OpenCloudCX can use Terraform state buckets to store all infrastructure snapshot information (e.g., S3 buckets, VPC, EC2, EKS). State buckets allow for teams to have a centralized souce of truth for the infrastructure. Per AWS S3 requirements, this bucket name needs to be globally unique. This bucket is not created automatically and needs to be in place before the terraform project is initialized. 

Follows [these]() instructions to create a unique bucket in the account where OpenCloudCX is going to be installed. A good convention for this project is to create and use ```opencloucx-state-bucket-####``` and replace ```####``` with the last 4 digits of the AWS account number. 

Once the bucket has been created, copy ```state.tf.example``` and rename the copy (not the original) to ```state.tf```. Update the requested and save. 

```bash
  backend "s3" {
    key    = "[terraform-state-filename]"
    bucket = "[bucket name]"
    region = "[region]"
  }
```

## Variables

Create a copy of the ```variables.example.tfvars``` file and name it ```variables.auto.tfvars```. If another filename needs to be used, Terraform automatically loads a number of variable definitions files if named the following way:
* Files named exactly ```terraform.tfvars``` or ```terraform.tfvars.json```
* Any files with names ending in ```.auto.tfvars``` or ```.auto.tfvars.json```

Update the variables within the file for any desired configuration changes

<table width=100%>
<tr>
  <th width="15%" style="font-weight:bolder;">Variable</th>
  <th width="35%" style="font-weight:bolder;">Explanation</th>
  <th width="50%" style="font-weight:bolder;">Example</th>
</tr>
<tr>
  <td>eks_map_roles</td>
  <td>Additional IAM roles to add to the aws-auth configmap. 
  </td>
  <td>
  
  <i>Defining Extra Roles</i>
```bash
eks_map_roles = [{
  groups   = ["system:masters"]
  rolearn  = "arn:aws:iam::<account_number>:role/<role name>"
  username = "<username>"
}]  
```
<i>No extra roles</i>
```bash
eks_map_roles = []
```

  </td>
</tr>
<tr>
  <td>eks_map_users</td>
  <td>Additional IAM users to add to the aws-auth configmap</td>
  <td>
  
<i>Defining Extra Users</i>
```bash
eks_map_users = [{
  groups   = ["system:masters"]
  userarn  = "arn:aws:iam::<account number>:user/<user name>"
  username = "<username>"
}]  
```
<i>No extra users</i>
```bash
eks_map_users = []
```

  </td>
</tr>

<tr>
  <td>dns_zone</td>
  <td>To experience the full impact of an OpenCloudCX installation, a valid, publicly accessible DNS zone needs to be supplied within the configuration. The default DNS Zone of ```spinnaker.internal``` can be used for initial prototyping with appropriate local hosts file manipulation.</td>
  <td>

  ```bash
  dns_zone           = "spinnaker.internal"
  ```
</td>
</tr>

</table>

# Modules

TODO: Talk about modules with link to PLUGINS.md

# Environment creation

## Initialize Terraform and Execute

### ```terraform init```

The ```init``` command tells Terraform to initialize the project from the current working directory of terraform configurations. If commands relying on initialization are executed before this step, the command will fail with an error.

From [terraform.io](https://www.terraform.io/docs/cli/init/index.html)

>Initialization performs several tasks to prepare a directory, including accessing state in the configured backend, downloading and installing provider plugins, and downloading modules. Under some conditions (usually when changing from one backend to another), it might ask the user for guidance or confirmation.

### ```terraform apply```

From [terraform.io](https://www.terraform.io/docs/cli/commands/apply.html)

>The terraform apply command performs a plan just like terraform plan does, but then actually carries out the planned changes to each resource using the relevant infrastructure provider's API. It asks for confirmation from the user before making any changes, unless it was explicitly told to skip approval.
### Command Execution

Execute these two commands in succession.

```
$ terraform init
$ terraform apply --auto-approve
```

If you receive the following error, confirm the s3 state bucket referenced above is correct

```bash
Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.
Error refreshing state: AccessDenied: Access Denied
        status code: 403, request id: <string> host id: <string>
```


---
_NOTE: Terraform assumes the current ```[default]``` profile contains the appropriate credentials for environment initialization. If this is not correct, each Terraform command needs to be prefixed with ```AWS_PROFILE=``` and the desired AWS profile to use._
On Linux this can be found in your home directory .aws update both credentials and config file
On Windows C:\Users\[username]\.aws update both credentials and config file
```
$ AWS_PROFILE=<profile name> terraform init
$ AWS_PROFILE=<profile name> terraform apply --auto-approve
```
---

Once Terraform instructions have been applied, the following message will be displayed 

<span style='font-size: 13pt; color: green'>Apply complete! Resources: ### added, 0 changed, 0 destroyed.</span>

# Environment Validation

Once a successful message of completion has been displayed, run the apprioriate script to connect.

<table>
<tr><th style="font-size:14pt">Linux</th><th style="font-size:14pt">Windows</th></tr>
<tr><td>

```bash
$ connect.sh --profile <profile name>
```

</td><td>

```powershell
connect.ps1 -AwsProfile <profile name>
```

</td></tr>
</table>

Output from the above commands:
|Label|Description|
|---|---|
|Cluster name|Name of the Kubernetes cluster for OpenCloudCX. This name will always contain a 4-character randomized string at the end|
|Dashboard&nbsp;token|Token for use when authenticating to the Kubeternetes dashboard (see below)|
|Jenkins PW|Jenkins admin password|
|Grafana PW|Grafana admin password|
<!-- |CodeServer PW|Code Server admin password| -->

Execute following command to list the namespaces in the cluster

```bash
$ kubectl get namespaces -A

NAME                   STATUS   AGE
cert-manager           Active   10m
dashboard              Active   10m
default                Active   10m
develop                Active   10m
ingress-nginx          Active   10m
jenkins                Active   10m
kube-node-lease        Active   10m
kube-public            Active   10m
kube-system            Active   10m
opencloudcx            Active   10m
spinnaker              Active   10m
```

# OpenCloudCX Constituents and Credentials

To access the individual toolsets contained within the OpenCloudCX enclave, use the following URLs, with the appropriate DNS zone from above, paired with the credentials outlined. Each module used may have their own secrets and methods to retrieve in the module documentation

|Name|URL|Username|Password Location|
|---|---|---|---|
|Dashboard| ```https://dashboard.[DNS ZONE]```|None|```connect.sh``` token output|
|Grafana| ```https://grafana.[DNS ZONE]```|admin|AWS Secrets Manager [```grafana```] or ```connect.sh``` token output|
|Jenkins| ```https://jenkins.[DNS ZONE]```|admin|AWS Secrets Manager [```jenkins```] or ```connect.sh``` token output|
|Keycloak| ```https://keycloak.[DNS ZONE]```|user|AWS Secrets Manager [```keycloak-admin```]
|Selenium| ```https://selenium.[DNS ZONE]```|None|None|
|Spinnaker| ```https://spinnaker.[DNS ZONE]```|None|None|

# Environment Destruction

### ```terraform destroy```

From [terraform.io](https://www.terraform.io/docs/cli/commands/destroy.html)

>The terraform destroy command is a convenient way to destroy all remote objects managed by a particular Terraform configuration.

Execute the command.

```
$ terraform destroy --auto-approve
```

If the script terminates with a timout error, re-execute the `destroy` command again. If the script times out again, delete the `spinnaker` namespace

```bash
$ kubectl delete namespace spinnaker

namespace "spinnaker" deleted
```

Once this command completes (it may take a while), re execute the `destroy` command again. 