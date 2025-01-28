# Deploy Prometheus
`bitovi/github-actions-deploy-prometheus` deploys a functional Prometheus and Grafana stack.

This action uses our new GitHub Actions Commons repository, a library that contains multiple Terraform modules, allowing us to condense all of our tools in one repo, hence continuous improvements are made to it. 
![alt](https://bitovi-gha-pixel-tracker-deployment-main.bitovi-sandbox.com/pixel/D21uVdWC6MEN4mIbjB88c)

Supported Cloud Providers:
- AWS

> **Note:** This action is currently in beta.  Please report any issues you find by creating [an Issue](https://github.com/bitovi/github-actions-deploy-prometheus/issues/new) or a [Pull Requests](https://github.com/bitovi/github-actions-deploy-prometheus/pulls)


<!-- ## Getting Started Intro Video
[![Getting Started - Youtube](https://img.youtube.com/vi/oya5LuHUCXc/0.jpg)](https://www.youtube.com/watch?v=oya5LuHUCXc) -->


## Need help or have questions?
This project is supported by [Bitovi, a DevOps Consultancy](https://www.bitovi.com/devops-consulting) and a proud supporter of Open Source software.

You can **get help or ask questions** on our [Discord channel](https://discord.gg/zAHn4JBVcX)! Come hang out with us; We love discussing solutions!

Or, you can hire us for training, consulting, or development. [Set up a free consultation](https://www.bitovi.com/devops-consulting).


## Requirements

This is a list of requirements you'll need to meet in order to use this action.
1. An AWS account (yep, that's it!)


### 1. An AWS account
You'll need [Access Keys](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html) from an [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)


## Example usage

> **Secrets first!** First, fill out the values in the `.env.example` file. Then, create a secret in your repo called `DOT_ENV` and paste the contents into it. (Do NOT commit any files with your secrets in them!)
If not set, Grafana will use the default admin/admin. (You can change this in the first login.)

Create `.github/workflow/deploy.yaml` with the following to build on push.

### Basic example
```yaml
name: Basic deploy
on:
  push:
    branches: [ main ]

jobs:
  Prometheus-Deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy
      name: Deploy
      uses: bitovi/github-actions-deploy-prometheus@v0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        env_ghs: ${{ secrets.DOT_ENV }}
```

> Once deployed, Visualize the metrics in the Prometheus UI itself (port `:9090`) or visualize via Grafana (port `:3000`).

### Advanced example

```yaml
name: Advanced deploy
on:
  push:
    branches: [ main ]

permissions:
  contents: read

jobs:
  Prometheus-Deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy
      name: Deploy
      uses: bitovi/github-actions-deploy-prometheus@v0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_session_token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws_default_region: us-east-1
        aws_ec2_instance_type: t2.small
        aws_additional_tags: "{\"key1\": \"value1\",\"key2\": \"value2\"}"

        aws_r53_domain_name: example.com
        aws_r53_sub_domain_name: app

        env_ghs: ${{ secrets.DOT_ENV }}
        env_ghv: ${{ vars.VARS }}

        
        grafana_datasource_dir: sandbox/observability/grafana/datasources
        prometheus_config: sandbox/observability/prometheus/prometheus.yml
        grafana_scrape_interval: 60m
        prometheus_scrape_interval: 60m
        prometheus_retention_period: 365d
        cadvisor_enable: true
        cadvisor_extra_targets: "target1:8080,target2:8080"
        node_exporter_enable: true
        node_exporter_extra_targets: "target1:9100,target2:9100"
        # Advanced options
        aws_ec2_port_list: "9090,3000,8080,9100"
        aws_elb_create: false
```

## Customizing

### Inputs
1. [Action Defaults](#action-defaults-inputs)
2. [AWS Configuration](#aws-configuration-inputs)
3. [Secrets and Environment Variables](#secrets-and-environment-variables-inputs)
4. [EC2](#ec2-inputs)
5. [Stack](#stack-inputs)
6. [Stack Management](#stack-management)
7. [Domains](#domains)
8. [VPC](#vpc-inputs)
9. [Advanced Options](#advanced-options)

The following inputs can be used as `step.with` keys
<br/>
<br/>

#### **Action defaults Inputs**
| Name             | Type    | Description                        | 
|------------------|---------|------------------------------------|
| `checkout` | Boolean | Set to `false` if the code is already checked out. Default is `true`. |

#### **AWS Configuration Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_access_key_id` | String | AWS access key ID |
| `aws_secret_access_key` | String | AWS secret access key |
| `aws_session_token` | String | AWS session token |
| `aws_default_region` | String | AWS default region. Defaults to `us-east-1` |
| `aws_resource_identifier` | String | Set to override the AWS resource identifier for the deployment. Defaults to `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`. |
| `aws_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to all provisioned resources. |
<hr/>
<br/>

#### **Secrets and Environment Variables Inputs**
| Name             | Type    | Description - Check note about [**environment variables**](#environment-variables). |
|------------------|---------|------------------------------------|
| `env_aws_secret` | String | Secret name to pull env variables from AWS Secret Manager, could be a comma separated list, read in order. Expected JSON content. |
| `env_repo` | String | File containing environment variables to be used with the app. |
| `env_ghs` | String | `.env` file to be used with the app from Github secrets. |
| `env_ghv` | String | `.env` file to be used with the app from Github variables. |
<hr/>
<br/>

#### **EC2 Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_ec2_instance_type` | String | The AWS IAM instance type to use. Default is `t3.medium`. See [this list](https://aws.amazon.com/ec2/instance-types/) for reference. |
| `aws_ec2_ami_filter` | String | AMI filter to use when searching for an AMI to use for the EC2 instance. Defaults to `ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*`. |
| `aws_ec2_iam_instance_profile` | String | The AWS IAM instance profile to use for the EC2 instance. Will create one if none provided with the name `aws_resource_identifier`. |
| `aws_ec2_create_keypair_sm` | Boolean | Generates and manages a secret manager entry that contains the public and private keys created for the ec2 instance. Defaults to `false`. |
| `aws_ec2_instance_root_vol_size` | Integer | Define the volume size (in GiB) for the root volume on the AWS Instance. Defaults to `10`. | 
| `aws_ec2_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to ec2 provisioned resources.|
<hr/>
<br/>

#### **Stack Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `grafana_datasource_dir` | String | Path to the grafana datasource directory. Default is `observability/grafana/datasources`. |
| `prometheus_config` | String | Path to the prometheus config file. Default is `observability/prometheus/prometheus.yml`. |
| `grafana_scrape_interval` | String | Will change the global value of **the default** Prometheus data source. Default is `15s`. |
| `prometheus_scrape_interval` | String | Will set the global value of scrape_interval and evaluation_interval. Won't replace the intervals of scrape_config's if set. Default is `15s`. |
| `prometheus_retention_period`  | String | When to remove old data. Default is `15d`. |
| `cadvisor_enable` | Boolean | Adds a cadvisor entry in the docker-compose file to spin up a cadvisor container in docker. |
| `cadvisor_extra_targets` | String | Add cadvisor targets. Example: `"target1:8080,target2:8080"` |
| `node_exporter_enable` | Boolean | Adds a node-exporter entry in the docker-compose file to spin up a ode-exporter container in docker.  |
| `node_exporter_extra_targets` | String | Add node-exporter targets. Example: `"target1:9100,target2:9100"`|
| `print_yaml_files` | Boolean | Prints resulting docker-compose, prometheus and grafana yaml files. |
<hr/>
<br/>

#### **Stack Management**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `tf_stack_destroy` | Boolean  | Set to `true` to destroy the stack - Will delete the `elb logs bucket` after the destroy action runs. |
| `tf_state_file_name` | String | Change this to be anything you want to. Carefull to be consistent here. A missing file could trigger recreation, or stepping over destruction of non-defined objects. Defaults to `tf-state-aws`. |
| `tf_state_file_name_append` | String | Appends a string to the tf-state-file. Setting this to `unique` will generate `tf-state-aws-unique`. (Can co-exist with `tf_state_file_name`) |
| `tf_state_bucket` | String | AWS S3 bucket name to use for Terraform state. See [note](#s3-buckets-naming) | 
| `tf_state_bucket_destroy` | Boolean | Force purge and deletion of S3 bucket defined. Any file contained there will be destroyed. `tf_stack_destroy` must also be `true`. Default is `false`. |
<hr/>
<br/>

#### **Domains**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_r53_domain_name` | String | Define the root domain name for the application. e.g. bitovi.com'. |
| `aws_r53_sub_domain_name` | String | Define the sub-domain part of the URL. Defaults to `aws_resource_identifier`. |
| `aws_r53_root_domain_deploy` | Boolean | Deploy application to root domain. Will create root and www records. Default is `false`. |
| `aws_r53_enable_cert` | Boolean | Set this to true if you wish to manage certificates through AWS Certificate Manager with Terraform. **See note**. Default is `false`. | 
| `aws_r53_cert_arn` | String | Define the certificate ARN to use for the application. **See note**. |
| `aws_r53_create_root_cert` | Boolean | Generates and manage the root cert for the application. **See note**. Default is `false`. |
| `aws_r53_create_sub_cert` | Boolean | Generates and manage the sub-domain certificate for the application. **See note**. Default is `false`. |
| `aws_r53_additional_tags` | JSON | Add additional tags to the terraform [default tags](https://www.hashicorp.com/blog/default-tags-in-the-terraform-aws-provider), any tags put here will be added to R53 provisioned resources.|
<hr/>
<br/>

#### **VPC Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_vpc_create` | Boolean | Define if a VPC should be created. Default is `false`. |
| `aws_vpc_name` | String | Set a specific name for the VPC. |
| `aws_vpc_cidr_block` | String | Define Base CIDR block which is divided into subnet CIDR blocks. Defaults to `10.0.0.0/16`. |
| `aws_vpc_public_subnets` | String | Comma separated list of public subnets. Defaults to `10.10.110.0/24`. |
| `aws_vpc_private_subnets` | String | Comma separated list of private subnets. If none, none will be created. |
| `aws_vpc_availability_zones` | String | Comma separated list of availability zones. Defaults to `aws_default_region`. |
| `aws_vpc_id` | String | **Existing** AWS VPC ID to use. Accepts `vpc-###` values. |
| `aws_vpc_subnet_id` | String | **Existing** AWS VPC Subnet ID. If none provided, will pick one. (Ideal when there's only one). |
| `aws_vpc_additional_tags` | JSON | A JSON object of additional tags that will be included on created resources. Example: `{"key1": "value1", "key2": "value2"}` |
<hr/>
<br/>

#### **Advanced Options**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `docker_cloudwatch_enable` | Boolean | Toggle cloudwatch creation for Docker. Defaults to `true`. |
| `docker_cloudwatch_skip_destroy` | Boolean | Toggle deletion or not when destroying the stack. Defaults to `false`. |
| `aws_ec2_instance_public_ip` | Boolean | Add a public IP to the instance or not. Defaults to `true`. |
| `aws_ec2_port_list` | String | EC2 Ports to be exposed. Should be set to `9090,3000` if ELB is disabled. |
| `aws_elb_create` | Boolean | Toggles the creation of a load balancer and map ports to the EC2 instance. Defaults to `true`.|
| `aws_elb_app_port` | String | Port in the EC2 instance to be redirected to. Default is `9090,3000`. | 
| `aws_elb_listen_port` | String | Load balancer listening port. Default is `9090,3000`. |
<hr/>
<br/>

`aws_ec2_instance_public_ip` is a must if deployment is done using GitHub runners. Needed to access instance and install Docker. Only set this to `false` if using a self-hosted GitHub runner with access to your private IP.

As a default, Prometheus and Grafana ports (9090,3000) for the EC2 instance are being exposed. (`aws_ec2_port_list`). An ELB will also be created exposing them too. 

The load balancer will listen for outside connections (`aws_elb_listen_port`) and forward this traffic to the defined ports (`aws_elb_app_port`). 
You can change all of this values to expose the services you wish, directly from the EC2 instance or through the ELB. (No need to expose EC2 instance port for ELB ports to work.).
You can even set different listening ports for the ELB. (`aws_elb_listen_port` will map 1 to 1 with `aws_elb_app_port`.).

**Tip**: Setting `aws_elb_listen_port` to `3000` will only expose Grafana. No need to set anything else.

ELB is a **must** if you intend to use DNS and/or certificates. (`aws_elb_create` must be `true`).

#### Default ports are:
| App     | Port | 
|-|-|
| Grafana | 3000 |
| Prometheus | 9090 |
| cadvisor | 8080 |
| node-exporter | 9100 | 

## Environment variables

For envirnoment variables in your app, you can provide:
 - `env_repo` - A file in your repo that contains env vars
 - `env_ghv` - An entry in [Github actions variables](https://docs.github.com/en/actions/learn-github-actions/variables)
 - `env_ghs` - An entry in [Github secrets](https://docs.github.com/es/actions/security-guides/encrypted-secrets)
 - `env_aws_secret` - The path to a JSON format secret in AWS
 

These environment variables are merged (in the following order) to the .env file and provided to both the Prometheus and Grafana services:
 - Terraform passed env vars ( This is not optional nor customizable )
 - Repository checked-in env vars - repo_env file as default. (KEY=VALUE style)
 - Github Secret - Create a secret named DOT_ENV - (KEY=VALUE style)
 - AWS Secret - JSON style like '{"key":"value"}'

## Note about resource identifiers

Most resources will contain the tag `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}`, some of them, even the resource name after. 
We limit this to a 60 characters string because some AWS resources have a length limit and short it if needed.

We use the kubernetes style for this. For example, kubernetes -> k(# of characters)s -> k8s. And so you might see some compressions are made.

For some specific resources, we have a 32 characters limit. If the identifier length exceeds this number after compression, we remove the middle part and replace it for a hash made up from the string itself. 

### S3 buckets naming

Buckets names can be made of up to 63 characters. If the length allows us to add -tf-state, we will do so. If not, a simple -tf will be added.

## CERTIFICATES - Only for AWS Managed domains with Route53

As a default, the application will be deployed and the ELB public URL will be displayed.

If `domain_name` is defined, we will look up for a certificate with the name of that domain (eg. `example.com`). We expect that certificate to contain both `example.com` and `*.example.com`. 

If you wish to set up `domain_name` and disable the certificate lookup, set up `no_cert` to true.

Setting `create_root_cert` to `true` will create this certificate with both `example.com` and `*.example.com` for you, and validate them. (DNS validation).

Setting `create_sub_cert` to `true` will create a certificate **just for the subdomain**, and validate it.

> :warning: Be very careful here! **Created certificates are fully managed by Terraform**. Therefor **they will be destroyed upon stack destruction**.

To change a certificate (root_cert, sub_cert, ARN or pre-existing root cert), you must first set the `no_cert` flag to true, run the action, then set the `no_cert` flag to false, add the desired settings and excecute the action again. (**This will destroy the first certificate.**)

This is necessary due to a limitation that prevents certificates from being changed while in use by certain resources.

## Made with BitOps
[BitOps](https://bitops.sh) allows you to define Infrastructure-as-Code for multiple tools in a central place.  This action uses a BitOps [Operations Repository](https://bitops.sh/operations-repo-structure/) to set up the necessary Terraform and Ansible to create infrastructure and deploy to it.

## Contributing
We would love for you to contribute to [bitovi/github-actions-deploy-prometheus](https://github.com/bitovi/github-actions-deploy-prometheus) and help make it even better than it is today!

Would you like to see additional features?  [Create an issue](https://github.com/bitovi/github-actions-deploy-prometheus/issues/new) or a [Pull Requests](https://github.com/bitovi/github-actions-deploy-prometheus/pulls).

## License
The scripts and documentation in this project are released under the [MIT License](https://github.com/bitovi/github-actions-deploy-prometheus/blob/main/LICENSE).

# Provided by Bitovi
[Bitovi](https://www.bitovi.com/) is a proud supporter of Open Source software.

# We want to hear from you.
Come chat with us about open source in our Bitovi community [Discord](https://discord.gg/J7ejFsZnJ4Z)!