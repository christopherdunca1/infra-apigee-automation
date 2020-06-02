All Credit for this Automation Configuration goes to Carlos Frias. Myself and my team worked hard to integrate his outstanding work with Ansible into our own Apigee environment in hopes that Automation would be our main course in the future. 

# What is it?

A set of lightweight roles and playbooks to manage the installation and 
Uninstallation of Apigee Edge for a Single Datacenter setup.
This README will guide you through the basics of configuring an Ansible
environment and running the Apigee Edge Install playbook for a Single Datacenter
setup.

# Alternative tools

These roles and playbooks provide the simplest possible interface to install an
Apigee Single Datacenter planet using Ansible. They drive the installation 
using ideal defaults and require just a few simple steps to perform a 
full installation. They do not
cover additional operational tasks such as
infrastructure creation, OS management, Apigee planet expansion, and other
tasks. If you are looking for an Ansible-managed solution for those tasks, see
the expanded Ansible roles offered at
<https://github.com/apigee/playbook-setup-ansible>.

# Prerequisites
1.) Ensure that the connectivity exists from Ansible Control Node to Remote 
nodes using Password less SSH communication.

2.) Validate the Configurations and Pre-requisites from 
<https://docs.apigee.com/private-cloud/latest/installation-requirements> 

3.) Ensure that the EPEL Repo is enabled and Java is installed on Remote nodes.
By default, the playbook would Install Java-8, if Java is not already <br /> installed.

4.) Disable the SELinux on remote nodes.

5.) TCP Wrappers can block communication of some ports and can affect OpenLDAP,
Postgres, and Cassandra installation. On those nodes, check `/etc/hosts.allow` 
and `/etc/hosts.deny` to ensure that there are no port restrictions on the 
required OpenLDAP, Postgres, and Cassandra ports.

6.) On the nodes where Qpidd/Qpid-Server is to be installed, remove the existing 
'qpid-proton' package already installed. This is because the installation on Qpid
might fail due to this.
Ex: To remove 'qpid-proton-c-0.16.0-12.el7sat.x86_64' on a Qpid node, execute:

    `$ yum remove -y qpid-proton-c-0.16.0-12.el7sat.x86_64`
    
Also refer the article: <https://community.apigee.com/articles/52998/qpid-installation-fails-due-qpid-proton-dependency.html>

7.) DO NOT the change the variable names in vars/secure.yaml and vars/extra_vars.yml file.<br />
    Only change the Values for these variables as per the requirement.


# Quickstart

## Install Ansible

The Ansible installation method depends on the distribution.

To install on Debian or Ubuntu from the distributor repository:
`sudo apt install ansible`

To install on Red Hat or CentOS from the distributor repository:
`sudo yum -y install ansible`

To install from pip:
`sudo pip install ansible`

Full installation details including packages for the latest Ansible release
are available at <https://docs.ansible.com/ansible/latest/intro_installation.html>.

## Populate your Ansible inventory

The hosts file contains the IP Addresses of the remote nodes under different host
groups as defined below. The different groups are:

[ds] - Add the nodes on which Zookeeper and Cassandra has to be installed.<br />
[ms] - Add the node on which Management-Server (which includes openLDAP and UI) has to be installed.<br />
[rmp] - Add the nodes on which Routers and Messsage-Processors to be installed.<br />
[ps] - Add the nodes on which Postgres for Analytics to be installed.<br />
[qs] - Add the nodes on which Qpid-Servers/Qpidd to be installed.<br />
[pdbdp] - Add the nodes on which Drupal Dev Portal including its PSQL Database to be installed.

Sample Inventory host file with the remote node details  can be found at :
<https://git.delta.com/apicoe/infra-apigee-automation/blob/INSTALL/inventory/hosts>

Inventory details are at <https://docs.ansible.com/ansible/latest/intro_inventory.html>.

To execute the main playbook, update the IPs with appropriate node details in 
the Inventory host file.

DO NOT CHANGE OR UPDATE THE GROUP NAME (like [ds] to [cszk] etc.) in the 
Inventory Host file for any of the groups as this might cause the Playbook to
break and hence may cause unexpected outcome.

## Configure SSH access and test Ansible

In order for Ansible to function you will need direct SSH access to the hosts
you wish to control. Ansible uses the standard OpenSSH client to establish
connections, so any settings in your local OpenSSH configuration file, such as
custom usernames or private keys, are respected. You can also override SSH
configuration settings in Ansible playbooks or the command line.

The simplest test to verify Ansible connectivity is:

`ansible all -m ping`

This command uses the inventory hosts file in inventory directoty , 
targets `all` host groups as defined in inventory, and executes the 
`ping` module on the remote hosts. Assuming all goes well, you will receive 
output similar to this (truncated for brevity) for all the groups defined in 
Inventory File:

```
host1 | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
host2 | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
...
```

You can change the remote SSH user:

`ansible all -m ping -u otheruser`

Prompt for an SSH password:

`ansible all -m ping -k`

Elevate to root through sudo:

`ansible all -m ping --become`

Prompt for a sudo password:

`ansible all -m ping --become -K`

Many other options are available; see <https://docs.ansible.com/ansible/latest/intro_adhoc.html>
or `man ansible` for a full description.

## Edit the build playbook to Install Apigee in a Single-Datacenter

The  `install_apigee_planet.yml` is the main playbook which needs to be invoked 
to install Apigee on Remote nodes as defined in Inventory Hosts file.

This playbook runs on all the host groups of Inventory Hosts File. It contains
different roles which executes as per the condition defined for a task in that particular 
role.

For ex: `apigee-organization` and `apigee-validate` role will only execute on 
Management-Server node under [ms] group as it is designed to run on 
only Management-Server node.Similarly, `apigee-dev-portal` will only execute 
for Dev-Portal Remote node defined under [pdbdp] group in inventory host file.


## Providing Extra Variables during Playbook Execution "vars/secure.yaml"

There are additional variables which needs to be defined at the run-time.
Few of the sensitive variables are encrypted using Vault Feature of ansible using 'ansible-vault'
command.
The sensitive variables are encrypted in file; vars/secure.yaml.

This variable contains the value of variables: 

- `apigee_repository_username`: The username provided by Apigee to access the
repository at software.apigee.com and the file transfer service at sftp.apigee.com.
- `apigee_repository_password`: The password provided by Apigee to access the
repository at software.apigee.com and the file transfer service at sftp.apigee.com.
- `apigee_admin_email`: The email of the Apigee Edge admin user.
- `apigee_admin_password`: The password of the Apigee Edge admin user.
- `apigee_ldap_password`: The LDAP Password for Apigee Setup.
- `apigee_postgresql_password`: The Apigee Analytics PostgreSQL Password.
- `apigee_cassandra_password`: The Apigee Cassandra Password. Do NOT change this value.


The default values for these variables are already defined in secure.yaml file.
However, if the values for these variables need to be changed execute the below ansible command
to update these values.<br />

`$ ansible-vault edit vars/secure.yaml`<br />

Enter the Ansible-Vault Password when prompted. Edit the values, save the file and exit.

DO NOT CHANGE the name of the variables in the file . Only change the values for these variables


## Providing Extra Variables during Playbook Execution "vars/extra_vars.yml"

Apart from the variables encrypted in "secure.yaml" file, there are additional variables which need
to be supplied during the playbook execution.
These variables are defined in vars/extra_vars.yml file.
This file can be found at : <https://git.delta.com/apicoe/infra-apigee-automation/blob/INSTALL/vars/extra_vars.yml>

The following variables are defined in the file:

- `apigee_release`: The version of Apigee that needs to be installed. Ex: 4.18.05
- `apigee_mp_pod`: The MP_POD for Apigee Setup. The usual default value is 'gateway'
- `apigee_installation_path`: The default installation Path for Apigee Setup. The usual default value is '/opt'
- `path`: The Symlink for /opt/apigee . Ex: "prd/api", "int/api". DO NOT include trailing '/' before 'int' or 'prd'
- `apigee_license_path`: Apigee License Path where the license file is kept. Do NOT change the value as by default the license is already present in home directory of playbooks.
- `apigee_smtp_host`: The Apigee SMTP Host value. The default is 'smtp.delta.com'
- `apigee_smtp_mail_from`: The default value is 'apisysadmin@delta.com'
- `apigee_organization_name`: The name of the organization to be created.
- `apigee_environment_name`: The name of the environment to be created.
- `apigee_virtual_host_name`: The Name of VHOST to be created. The default value is 'default'.
- `apigee_virtual_host_port`: The VHOST Port for the VHOST created.
- `apigee_virtual_host_aliases`: A list of VHOST Aliases to be created for the environment. Provide the value as per the syntax defined ion 'extra_vars.yml' file.
- `apigee_clean_validate_org`: Provide the valus as 'true' if 'VALIDATE' Organization needs to be removed. The default value is false.
- `apigee_dp_admin_firstname`: The Dev Portal Drupal Admin First Name.
- `apigee_dp_admin_lastname`: The Dev Portal Drupal Admin Last Name.
- `apigee_dp_admin_username`: The Dev Portal Drupal Admin User Name.
- `apigee_dp_admin_email`: The Dev Portal Drupal Admin Email.
- `apigee_dp_admin_password`: The Dev Portal Drupal Admin Password.
- `apigee_dp_smtp_host`: The Dev Portal SMTP Host. The default value to be provided is 'smtp.delta.com'


The Value for ALL the above variables must be updated with the actual Setup Requirement. 
Do NOT Change or Modify the Name of the variable defined. Just update the Values for these variables.



## Run the Install playbook

With Ansible configured and 'install_apigee_planet.yml' updated, you can run the
playbook to build your planet:

`$ ansible-playbook  install_apigee_planet.yml --ask-vault-pass`

Provide the Ansible Vault Password which was used to encrypt the variables in 'secure.yml' file.


If playbook execution fails, the output of `ansible-playbook` may be sufficient to
diagnose. If not, the `/tmp/setup-root.log` file on the remote host will provide
additional insight into the cause of the failure. Since the Apigee installer is
idempotent, it is safe to re-run `ansible-playbook` once the cause of the failure
is fixed. The only exception is if incorrect profiles are applied to some hosts
in the cluster. If you accidentally apply incorrect profiles (such as putting an
rmp profile on an ms host), you can clean up by removing all packages and the
installation directory on all Apigee hosts:

## Execute the Uninstall Apigee Playbook to Uninstall on Remote nodes.

When the above 'install_apigee_planet.yml' playbook fails in execution, correct the issue with the logs and try to run the 'Install' Playbook again.
If the remote nodes need a cleanup for Apigee with the values updated in the variable files, execute the uninstall playbook as :<br />

`$ ansible-playbook  uninstall_apigee_singleDC.yml --ask-vault-pass`

Enter the Vault Password when prompted.

once the cleanup happens, execute the Anisble-Install playbook again.


# Known Limitations

1) Multi-DC Setup Playbook to be added yet.

2) Multi-DC Uninstall Playbook to be added yet
