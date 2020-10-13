# kubeflow-sandbox
Reference repository for creating EKS clusters with simple Kubeflow installation

For examining the live environment with the files you can check the `example` branch of that repository
## Preparing workspace
### Install Terrafrom
Please follow to [official Terraform site](https://learn.hashicorp.com/tutorials/terraform/install-cli)
### Create configuration files
Need to create two configuration files
- `backend.hcl`
- `terraform.tfvars`
#### `backend.hcl`
``` hcl
bucket         = "bucket-with-terraform-states"
key            = "some-key/kubeflow-sandbox"
region         = "region-where-bucket-placed"
dynamodb_table = "dynamodb-table-for-locks"
```
#### `terraform.tfvars`
``` hcl
# Main route53 zone id if exist.
mainzoneid = "id-of-route53-zone"

# Name of domains aimed for endpoints
domains = ["sandbox.some.domain.local"]

# ARNs of users who will have admin permissions.
admin_arns = [
  {
    userarn  = "arn:aws:iam::<aws-account-id>:user/<username>"
    username = "<username>"
    groups   = ["system:masters"]
  }
]

# Email that would be used for LetsEncrypt notifications
cert_manager_email = "info@some.domain.local"

```
In most cases, you should also override variables related to the GitHub repository such  a `repository`, `branch` and `owner`
### Initialize Terraform
``` bash
terraform init --backend-config backend.hcl
```
This command will download all remote dependency modules
## Cluster creation
After completing all configuration steps you can execute Terraform:
``` bash
terraform apply
```
This command will create all required AWS resources such a IAM roles, policies, S3 buckets, etc.
Also, a clean EKS would be created, you can access it by updating your local kubefconfig file by the next command:
``` bash 
aws --region <region> eks update-kubeconfig --name <cluster-name>
```
and after that, you can execute `kubectl` commands.
## Post actions and accesses

As part of Terraform execution was generated a few files, by default in the `apps` folder. So, now to start deploying the actual services to EKS need to add these files to Git and push it to the repository.

ArgoCD initially configured to listen to the current repository and when new changes come to `apps` folder they trigger the synchronization process and all objects placed in that folder become created.

By default, would be created two endpoints for accessing services:
- ArgoCD  `https://argocd.some.domain.local`
- Kubeflow  `https://kubeflow.some.domain.local`

For access that URLs need to configure Cognito User Pool with the name which matches with the cluster name.
### Screenshots
![kubeflow](images/kubeflow.png)
![argocd](images/argocd.png)