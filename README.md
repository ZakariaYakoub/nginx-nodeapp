# Nginx NodeApp Pipeline
This project demonstrates a Jenkins pipeline script that provisions EC2 instances on AWS using Terraform and configures Nginx and a Node.js application on the provisioned instances using Ansible.

## Description
The Nginx NodeApp Pipeline showcases a Jenkins pipeline script that automates the provisioning of EC2 instances, configuration of Nginx, and deployment of a Node.js application. The pipeline is divided into two stages:

Provision EC2 Instances: This stage provisions EC2 instances on AWS using Terraform. It initializes Terraform, applies the configuration, and retrieves the public IP addresses of the provisioned instances.

Manage the Servers: In this stage, the pipeline performs the following actions:

1- Copies necessary configuration files to the Ansible server.
2- Configures Nginx and the Node.js application by running Ansible playbooks on the Ansible server.
