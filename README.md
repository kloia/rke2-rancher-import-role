# RKE2 Cluster Create and Rancher Import Role With AWS SSM

This Ansible role is designed to create RKE2 (Rancher Kubernetes Engine 2) cluster on existing `AWS EC2 Linux Instances` and import into `Rancher`, with `AWS Systems Manager (SSM) integration`.

## Overview

This role automates the process of create RKE2 cluster on existing Linux Instances and import into Rancher manager. It's structured to work with both master (k8sm) and worker (k8sw) nodes, and can be executed via AWS Systems Manager for enhanced security and ease of management.

## Directory Structure

- `inventory/`: Contains the inventory files for defining hosts and groups.
  - `custom_hosts.d/`: Subdirectory for custom host definitions.
    - `100-hosts.yaml`: Likely contains individual host definitions.
    - `110-hosts-groups.yaml`: Defines the host groups structure.
- `tasks/`: Contains the main tasks for the role.
- `templates/`: Stores any Jinja2 templates used by the role.
- `vars/`: Holds variable files for the role.
- `defaults/`: Contains default variables for the role.
  - `main.yaml`: The main file for default variable definitions.
- `README.md`: This file, providing documentation for the role.

The contents of `tasks/`, `templates/`, and `vars/` directories are not specified in the provided information. These directories likely contain additional YAML files for tasks, Jinja2 templates, and variable definitions respectively.

## Inventory

The role uses a custom inventory structure. An example of the inventory grouping is provided in `inventory/custom_hosts.d/110-hosts-groups.yaml`:


## Inventory

The role uses a custom inventory structure. An example of the inventory grouping is provided in `inventory/custom_hosts.d/110-hosts-groups.yaml`:

```yaml
all:
  children:
    linux:
      children:
        k8sm:
        k8sw:
    k8s:
      children:
        k8sm:
        k8sw:
````

This structure defines two main groups:

- k8sm: Kubernetes master nodes

- k8sw: Kubernetes worker nodes

Both are children of the linux and k8s groups.

### Requirements
- Ansible 2.9 or higher

- AWS CLI configured with appropriate permissions

- AWS Systems Manager set up on target instances

### Default Variables
The following variables are defined in defaults/main.yaml:

```yaml
default_rancherimportcluster_api_token: '{{ "sampletoken" }}'
default_rancherimportcluster_rancher_host:  '{{ "samplehost" }}'
default_rancherimportcluster_cluster_name: '{{ "sampleclustername" }}'

default_rancherimportcluster_api_token: '{{ RANCHERIMPORTCLUSTER_API_TOKEN | default(default_rancherimportcluster_api_token) }}'
default_rancherimportcluster_rancher_host: '{{ RANCHERIMPORTCLUSTER_RANCHER_HOST | default(default_rancherimportcluster_rancher_host) }}'
default_rancherimportcluster_cluster_name: '{{ RANCHER_CLUSTER_NAME | default(default_rancherimportcluster_cluster_name) }}'

default_rke2installation_script_url: 'https://get.rke2.io'
default_importrke2_to_rancher: true
default_rke2_os_hardening: true
default_rke2_version: 'v1.30.5+rke2r1'
default_rke2_cni: 'cilium'
default_rke2_token: 'ExAmPlET0kEn'

default_rancherimportcluster_create_yaml:
  apiVersion: provisioning.cattle.io/v1
  kind: Cluster
  metadata:
    name: "{{ default_rancherimportcluster_cluster_name }}"
    annotations: {}
    labels: {}
    namespace: fleet-default
  spec:
  __clone: true
````

These variables control various aspects of the RKE2 installation and Rancher import process. Make sure to set the appropriate values for your environment, especially for sensitive information like API tokens.


### Usage with AWS Systems Manager
#### To use this role with AWS Systems Manager:

- Ensure your EC2 instances are configured as managed instances in AWS Systems Manager.

- Create an SSM document that references this Ansible role. You can use the AWS CLI or AWS Console to create the document.

- Run the SSM document as an Automation, Command, or as part of a Maintenance Window task.

#### Example AWS CLI command to create an SSM document:

```bash
aws ssm create-document \
    --content file://path/to/ssm-playbook.yaml \ [[2]](https://docs.aws.amazon.com/systems-manager/latest/userguide/documents-creating-content.html)
    --name "RKE2-Rancher-Import-Playbook" \
    --document-type "Command" \
    --document-format "YAML"
````

Replace path/to/ssm-playbook.yaml with the actual path to your SSM playbook file.

### Variables
The role uses a combination of default variables (as shown above) and potentially other variables defined in the vars/ directory or passed during execution. Key variables to consider:

```yaml
RANCHERIMPORTCLUSTER_API_TOKEN: Can be set to override the default API token.

RANCHERIMPORTCLUSTER_RANCHER_HOST: Can be set to override the default Rancher host.

RANCHER_CLUSTER_NAME: Can be set to override the default cluster name.
```
When using this role with AWS Systems Manager, ensure that these variables are properly set in your SSM environment or passed as parameters when executing the SSM document.

#### Important Notes:
The default values for sensitive information (like default_rancherimportcluster_api_token and default_rke2_token) are placeholders.

- The default_rke2_version is set to 'v1.30.5+rke2r1'. Ensure this version is compatible with your environment and update as necessary.

- The default_rke2_cni is set to 'cilium'. Make sure this aligns with your network requirements.

- The default_importrke2_to_rancher and default_rke2_os_hardening are boolean flags that control whether to import the RKE2 cluster to Rancher and whether to apply OS hardening, respectively.

- The default_rancherimportcluster_create_yaml provides a template for creating a Rancher cluster. You may need to adjust this based on your specific Rancher setup and requirements.

### Sources
[1] Understanding the configuration profile IAM role - AWS AppConfig
docs.aws.amazon.comappconfiglatestappconfig-creating-configuration-and-profile-iam-role.html.

[2] Creating SSM document content - AWS Systems Manager
docs.aws.amazon.comsystems-managerlatestdocuments-creating-content.html
