# tf_provision

This role requires [Terraform](https://www.terraform.io/downloads.html) to be installed on the Ansible controller and configured [AWS credentials](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html). Review the resources defined in the templates and verify your AWS credentials have the necessary permissions to request those resources.

Due to the ephemeral nature of execution when using the Red Hat Ansible Automation Platform's Automation Controller (Tower), this role is configured to utilize an S3 bucket as the [backend](https://www.terraform.io/docs/language/settings/backends/s3.html) for Terraform to maintain state.

Ansible will build the necessary .tf files in the `terraform` directory from templates located in the `templates` directory.  There are some opinionated choices there, and some unsafe practices such as the security group configuration located in `templates/common/securitygroups.tf.j2`.  Windows instances provisioned from these templates will have the necessary configuration for Ansible to manage, see the `user_data` definition in `windows.tf.j2`.

The `template` tag can be used to only execute the portion of this role which builds the Terraform resource definition files (.tf) from their Jinja templates.

### Not all of the variables defined as defaults need to be overridden, but these do:

- workspace - Used to set the Terraform [workspace](https://www.terraform.io/docs/language/state/workspaces.html), and also throughout resource definitions as part of their naming convention. (the default will function, but do yourself a favor and set this)

- aws_key - The [SSH key](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html) configured in AWS for accessing the RHEL instances this role builds.

- terraform_s3_bucket - Necessary for using this role with Automation Controller (Tower).  If you are not, the configuration is in `templates/common/main.tf.j2`.

- teardown - Set this to true to delete the resources in your `workspace`.  `workspace` needs to match the one set during provisioning in order to target the correct resources.

Our team uses `ec2_env` as a filter for our dynamic inventory plugin, you may not need this variable at all.

## Example playbook:

This example playbook is the one distributed as part of the collection containing this role, and can be run as-is if you wish.
```yaml
---
- name: do things
  hosts: localhost
  gather_facts: no
  tasks:
    - name: do terraform things
      include_role:
        name: tf_provision
      tags:
        - template
```

## Requirements

- The only Ansible content requirement for execution is the `community.general.terraform` module from the `community.general` [collection](https://galaxy.ansible.com/community/general)
