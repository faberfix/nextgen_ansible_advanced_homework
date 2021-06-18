# Advanced Deployment with Red Hat Ansible Automation - Final Lab

## About

This assignment involves deploying the following with [Ansible](https://ansible.com/):

- A 3 tier application (Database, 2 App servers, Load balancer)
- Provisioning QA infrastructure for the app in OpenStack
- Provisioning Prod infrastructure in AWS
- Deploying an Ansible Tower project with the necessary job and workflow templates to manage the above

## Deployment

To deploy this, please follow these steps.

### Create Lab Environment

- Provision the following services from the [OPENTLC lab portal](https://labs.opentlc.com/):
	- OPENTLC Automation / Ansible Advanced - Homework
	- OPENTLC Automation / Ansible Advanced NG - OpenStack
- Once provisioned, connect to the **Homework** lab's `control` host:
	- `ssh <user-id>@control.<guid>.example.opentlc.com`
- Switch to root: `sudo -i`
- Copy your OPENTLC private key (for the public key in [account.opentlc.com](https://account.opentlc.com/account/)) to `/root/.ssh/mykey.pem`, and set permissions to 0400.
- Fork this git repo: `git clone https://github.com/clifford2/nextgen_ansible_advanced_homework.git && cd nextgen_ansible_advanced_homework`
- Copy the `labrc` file from the cloned repo to your home directory, and edit it to fill in the required *sensitive* information
- Set the environment variables from this file: `source ~/labrc`
- Provision the OSP network, flavor, security groups, and SSH keys, and set up the OSP `workstation` host as an Tower isolated node, by running the following (supplying the OSP workstation SSH password from email when prompted, and the vault password `ansible`):
	```
	mv ~/ansible-tower-setup-*/ ~/ansible-tower-setup-latest
	cp /etc/ansible/hosts ~/ansible-tower-setup-latest/inventory
	chmod 0400 /root/.ssh/openstack.pem
	ansible-playbook site-setup-prereqs.yaml --ask-pass --ask-vault-pass
	```
- Verify a successful connection between the `control` and `workstation` hosts:
	```
	ssh -i /root/.ssh/openstack.pem cloud-user@workstation-${OSP_GUID}.${OSP_DOMAIN}
	```
- From your web browser, connect to `tower1.${TOWER_GUID}.example.opentlc.com` to verify that the isolated node is operational, using `admin` for the username and password as provided in email.
- In the web UI, verify that the `osp` instance group was created
- In the web UI, change the `admin` password to `r3dh4t1!`
- Back on the `control` host, also change the password in `~/.tower_cli.cfg`
- Create the Tower project, job templates & workflow template by running the following command (vault password is `ansible`):
	- `ansible-playbook site-config-tower.yml -e tower_GUID=${TOWER_GUID} -e osp_GUID=${OSP_GUID} -e osp_DOMAIN=${OSP_DOMAIN} -e opentlc_login=${OPENTLC_ID} -e path_to_opentlc_key=/root/.ssh/mykey.pem -e param_repo_base=${JQ_REPO_BASE} -e opentlc_password="${OPENTLC_PASSWORD}" -e REGION_NAME=${REGION} -e EMAIL=${MAIL_ID} -e github_repo=${GITHUB_REPO} --ask-vault-pass`

### Deploy the application

The ` cicd_workflow` template should now exist in Tower, and can be launched from there.
When prompted, please provide the SSH password for the AWS bastion host (from the email you received from OPENTLC, as your SSH key isn't always deployed with the lab).

## List of Playbooks

| Playbook | Description | Roles | Description |
| -------- | ----------- | ----- | ----------- |
| `site-setup-prereqs.yaml` | Setup OSP workstation & core resources, and install Tower isolated node (by importing `site-install-isolated-node.yaml` playbook) | 1. `setup-workstation` | Setup workstation with SSH keypair & required software |
| | | 2. `osp-setup` | Provision OSP network, security groups, keypair, image, flavors |
| `site-install-isolated-node.yaml` | Install Ansible Tower on isolated node (imported into `site-setup-prereqs.yaml`) | `install-isolated-node` | Install Ansible Tower |
| `site-config-tower.yml` | Configure Ansible Tower | `config-tower` | Configure Tower project, inventories, job templates and workflow |
| `site-osp-instances.yml` | Provision OSP instances | `osp-servers` | Create OSP server instances |
| `aws_provision.yml` | Provision AWS environment | `aws-provision` | Use `order_svc.sh` script to provision env |
| `aws_creds.yml` | Create machine credential to connect to AWS instances | `aws-creds` | Fetch GUIDkey.pem from bastion of Three tier application env and create machine credential to connect to AWS instances |
| `aws_status_check.yml` | Check aws instances are up or not | `aws-status` | Try and connect using SSH to test if instances are up |
| `site-3tier-app.yml` | Deploy three tier app | 1. `osp-facts` | Genrate in-memory inventory for OSP instances |
| | | 2. `base-config` | Initial, common, system setup steps for all servers |
| | | 3. `lb-tier` | Install & configure haproxy load balancer |
| | | 4. `app-tier` | Install & configure Apache Tomcat application server |
| | | 5. `db-tier` | Install & Configure Postgres database |
| `site-smoke-osp.yml` | test three tier app on OSP | None |
| `site-smoketest-aws.yml` | test three tier app on AWS | None |
| `grading-script.yml` | Self grading script | None |
| `site-osp-delete.yml` | Delete OSP instances | `osp-instance-delete` | Delete all OSP server instances |
| `site-setup-workstation.yml` | Unused - see `site-setup-prereqs.yaml` instead | setup-workstation | See above |
