# Deploy Prometheus
GitHub action to deploy Prometheus and Grafana.


Supported Cloud Providers:
- AWS

> **Note:** This action is currently in beta.  Please report any issues you find by creating [an Issue](https://github.com/bitovi/github-actions-deploy-prometheus/issues/new) or a [Pull Requests](https://github.com/bitovi/github-actions-deploy-prometheus/pulls)


![alt](https://bitovi-gha-pixel-tracker-deployment-main.bitovi-sandbox.com/pixel/D21uVdWC6MEN4mIbjB88c)

<!-- ## Getting Started Intro Video
[![Getting Started - Youtube](https://img.youtube.com/vi/oya5LuHUCXc/0.jpg)](https://www.youtube.com/watch?v=oya5LuHUCXc) -->


## Need help or have questions?
This project is supported by [Bitovi, a DevOps Consultancy](https://www.bitovi.com/devops-consulting) and a proud supporter of Open Source software.

You can **get help or ask questions** on our [Discord channel](https://discord.gg/J7ejFsZnJ4)! Come hang out with us; We love discussing solutions!

Or, you can hire us for training, consulting, or development. [Set up a free consultation](https://www.bitovi.com/devops-consulting).


## Requirements

This is a list of requirements you'll need to meet in order to use this action.
1. An AWS account (yep, that's it!)


### 1. An AWS account
You'll need [Access Keys](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html) from an [AWS account](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/)


## Example usage

> **Secrets first!** First, fill out the values in the `.env.example` file. Then, create a secret in your repo called `DOT_ENV` and paste the contents into it. (Do NOT commit any files with your secrets in them!)


Create `.github/workflow/deploy.yaml` with the following to build on push.

### Basic example
```yaml
name: Basic deploy
on:
  push:
    branches: [ main ]

jobs:
  EC2-Deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy
      name: Deploy
      uses: bitovi/github-actions-deploy-prometheus@0.1.0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID}}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY}}
        env_ghs: ${{ secrets.DOT_ENV }}
```

> Once deployed, Visualize the metrics in the Prometheus UI itself (port `:443`) or visualize via Grafana (port `:3000`).

### Advanced example

```yaml
name: Advanced deploy
on:
  push:
    branches: [ main ]

permissions:
  contents: read

jobs:
  EC2-Deploy:
    runs-on: ubuntu-latest
    steps:
    - id: deploy
      name: Deploy
      uses: bitovi/github-actions-deploy-prometheus@0.1.0
      with:
        aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws_session_token: ${{ secrets.AWS_SESSION_TOKEN }}
        aws_default_region: us-east-1
        aws_ec2_instance_type: t2.small
        additional_tags: "{\"key1\": \"value1\",\"key2\": \"value2\"}"

        domain_name: bitovi.com
        sub_domain: app

        dot_env: ${{ secrets.DOT_ENV }}
        ghv_env: ${{ vars.VARS }}

        
        grafana_datasource_dir: sandbox/observability/grafana/datasources
        prometheus_config: sandbox/observability/prometheus/prometheus.yml
```

## Customizing

### Inputs
1. [Action Defaults](#action-defaults-inputs)
2. [AWS Configuration](#aws-configuration-inputs)
3. [Secrets and Environment Variables](#secrets-and-environment-variables-inputs)
4. [EC2](#ec2-inputs)
5. [Prometheus](#prometheus-inputs)
6. [Stack Management](#stack-management)
7. [Domains](#domains)
8. [VPC](#vpc-inputs)


The following inputs can be used as `step.with` keys
<br/>
<br/>

#### **Action defaults Inputs**
| Name             | Type    | Description                        | Required | Default |
|------------------|---------|------------------------------------|----------|---------|
| `checkout` | Boolean | Set to `false` if the code is already checked out. | false | true

#### **AWS Configuration Inputs**
| Name             | Type    | Description                        | Required | Default |
|------------------|---------|------------------------------------|----------|---------|
| `aws_access_key_id` | String | AWS access key ID. | true | |
| `aws_secret_access_key` | String | AWS secret access key. | true | |
| `aws_session_token` | String | AWS session token, if you're using temporary credentials. | false | |
| `aws_default_region` | String | AWS default region. | true | us-east-1 |
| `aws_resource_identifier` | String | Auto-generated by default so it's unique for org/repo/branch. Set to override with custom naming the unique AWS resource identifier for the deployment. Defaults to `${org}-${repo}-${branch}`. | false | `${GITHUB_ORG_NAME}-${GITHUB_REPO_NAME}-${GITHUB_BRANCH_NAME}` |
| `aws_extra_tags` | JSON | A list of additional tags that will be included on created resources. Example: `{"key1": "value1", "key2": "value2"}` | false | {} |

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
| `aws_ec2_instance_type` | String | The AWS EC2 instance type. Default is `t2.medium`. |
| `aws_ec2_instance_profile` | String | The AWS IAM instance profile to use for the EC2 instance. Use if you want to pass an AWS role with specific permissions granted to the instance. |
| `aws_ec2_create_keypair_sm` | Boolean | Creates a Secret in AWS secret manager to store a kypair. Default is `false`. |
| `aws_ec2_instance_vol_size` | String | Root disk size for the EC2 instance. Default is `10`. |
| `aws_ec2_additional_tags` | JSON | A JSON object of additional tags that will be included on created resources. Example: `{"key1": "value1", "key2": "value2"}` |
| `aws_ec2_ami_filter` | String | AMI filter to use when searching for an AMI to use for the EC2 instance. Defaults to `ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*`. |
| `infrastructure_only` | Boolean | Set to `true` to provision infrastructure (with Terraform) but skip the app deployment (with ansible). Default is `false`. |
<hr/>
<br/>

#### **Prometheus Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `grafana_datasource_dir` | String | Path to the grafana datasource directory. Default is `observability/grafana/datasources`. |
| `prometheus_config` | String | Path to the prometheus config file. Default is `observability/prometheus/prometheus.yml`. |
<hr/>
<br/>

#### **Stack Management**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `tf_stack_destroy` | Boolean | Set to `true` to destroy the created AWS infrastructure for this instance. Default is `false`. |
| `tf_state_file_name` | String | Change this to be anything you want to. Carefull to be consistent here. A missing file could trigger recreation, or stepping over destruction of non-defined objects. |
| `tf_state_file_name_append` | String | Append a string to the tf-state-file. Setting this to `unique` will generate `tf-state-aws-unique`. Can co-exist with the tf_state_file_name variable. |
| `tf_state_bucket` | String | AWS S3 bucket to use for Terraform state. Defaults to `${org}-${repo}-{branch}-tf-state-aws`. |
| `tf_state_bucket_destroy` | Boolean | Force purge and deletion of S3 tf_state_bucket defined. Any file contained there will be destroyed. `tf_stack_destroy` must also be `true`. |
<hr/>
<br/>

#### **Domains**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_domain_name` | String | Define the root domain name for the application. e.g. bitovi.com. If empty, ELB URL will be provided. |
| `aws_sub_domain` | String | Define the sub-domain part of the URL. Defaults to `${org}-${repo}-{branch}`. |
| `aws_root_domain` | Boolean | Deploy application to root domain. Will create root and www DNS records. Domain must exist in Route53. |
| `aws_cert_arn` | String | Existing certificate ARN to be used in the ELB. Use if you manage a certificate outside of this action. See https://docs.aws.amazon.com/acm/latest/userguide/gs-acm-list.html for how to find the certificate ARN. |
| `aws_create_root_cert` | Boolean | Generates and manage the root certificate for the application to be used in the ELB. |
| `aws_create_sub_cert` | Boolean | Generates and manage the sub-domain certificate for the application to be used in the ELB. |
| `aws_no_cert` | Boolean | Set this to true if you want not to use a certificate in the ELB. Default is `false`. |
<hr/>
<br/>

#### **VPC Inputs**
| Name             | Type    | Description                        |
|------------------|---------|------------------------------------|
| `aws_vpc_create` | Boolean | Define if a VPC should be created. Default is `true`. |
| `aws_vpc_name` | String | Set a specific name for the VPC. |
| `aws_vpc_cidr_block` | String | Define Base CIDR block which is divided into subnet CIDR blocks. Defaults to `10.0.0.0/16`. |
| `aws_vpc_public_subnets` | String | Comma separated list of public subnets. Defaults to `10.10.110.0/24`. |
| `aws_vpc_private_subnets` | String | Comma separated list of private subnets. If none, none will be created. |
| `aws_vpc_availability_zones` | String | Comma separated list of availability zones. Defaults to `aws_default_region`. |
| `aws_vpc_id` | String | AWS VPC ID. Accepts `vpc-###` values. |
| `aws_vpc_subnet_id` | String | Specify a Subnet to be used with the instance. If none provided, one will be picked. |
| `aws_vpc_additional_tags` | JSON | A JSON object of additional tags that will be included on created resources. Example: `{"key1": "value1", "key2": "value2"}` |

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
