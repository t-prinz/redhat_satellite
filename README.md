# Ansible playbooks to manage Red Hat Satellite

These playbooks can be used to install and configure a Satellite server.  If the Satellite server is already setup, the satellite_config.yml playbook can be used to make configuration changes.

### Prerequisites

* These playbooks make use of the Red Hat "redhat.satellite" collection, which is available via the Ansible Automation Platform (which requires a subscription).

    If the playbooks are run via the command line and ansible-galaxy is used to install the collections (as shown in the examples), the ansible-galaxy client must be configured.  The documentation showing how to do this can be found [here](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#configuring-the-ansible-galaxy-client).

    If the playbooks are run via Ansible Tower, an Ansible Galaxy credential must be created and configured for the Organization.  The documentation showing how to do this can be found [here](https://docs.ansible.com/automation-controller/4.2.1/html/userguide/projects.html?extIdCarryOver=true&intcmp=701f2000001OEH1AAO&sc_cid=701f2000000u72fAAA#using-collections-via-hub:~:text=16.8.1.%20Using-,Collections,-via%20Hub).

    *In either case, credentials for Ansible Automation Hub are required; these credentials can be found [here](https://console.redhat.com/ansible/automation-hub/token).*
        
* In a previous version (tag v0.9), the playbooks referenced two variables that, in turn, referenced two files that held the variables controlling the exeuction of the playbooks.  These variables are no longer used; instead, the playbooks expect the variables to be defined as part of the inventory.

* If using Ansible from the command line, install the roles and collections using the following commands.

        ansible-galaxy role install -r roles/requirements.yml
        ansible-galaxy collection install -r collections/requirements.yml

### Using the playbooks

The following workflow shows how these playbooks can be used to set up a Satellite server.  These playbooks make use of the example inventory defined in the directory inventory_example, but the variables can be defined in other ways.

Define some environment variables for convenience

    INVENTORY=inventory_example
    SATELLITE_SERVER=sat.example.com

This playbook will register the system and take care of the prerequisites

    ansible-playbook -i ${INVENTORY} \
                     -l ${SATELLITE_SERVER} \
                     satellite_prep.yml

This playbook will perform the actual Satellite installation

    ansible-playbook -i ${INVENTORY} \
                     -l ${SATELLITE_SERVER} \
                     satellite_install.yml

This playbook will configure the Satellite server

    ansible-playbook -i ${INVENTORY} \
                     -l ${SATELLITE_SERVER} \
                     satellite_config.yml

### Determining the products, repository_sets, and repositories

To list all products:

    hammer product list --organization Acme

Once the product is identified, list all repository_sets with

    hammer repository-set list --product "Red Hat Enterprise Linux Server" --organization Acme

Once the repository_set is identified, list all of the available repositories with

    hammer repository-set available-repositories --product "Red Hat Enterprise Linux Server" --organization Acme --name "Red Hat Enterprise Linux 7 Server (RPMs)"

Take note of the output, since this will determine whether the 'basearch' and/or 'releasever' parameters will need to be specified.  For example, the output of the previous command produces this table


NAME                                     | ARCH   | RELEASE | ENABLED
-----------------------------------------|--------|---------|--------
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7Server | yes    
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.9     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.8     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.7     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.6     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.5     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.4     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.3     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.2     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.1     | no     
Red Hat Enterprise Linux 7 Server (RPMs) | x86_64 | 7.0     | no     


Since both ARCH and RELEASE are defined, they need to be specified in the product definition, for example:

    satellite_products:
      - name: Red Hat Enterprise Linux Server
        repository_sets:
          - name: Red Hat Enterprise Linux 7 Server (RPMs)
            basearch: x86_64
            releasever: "7Server"

For RHEL 8, the output looks as follows:

    hammer repository-set available-repositories --product "Red Hat Enterprise Linux for x86_64" --organization Acme --name "Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)"

    
NAME                                                  | RELEASE | ENABLED
------------------------------------------------------|---------|--------
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8       | yes    
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.8     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.7     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.6     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.5     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.4     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.3     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.2     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.1     | no     
Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs) | 8.0     | no     


In this case, only 'releasever' needs to be specified:

    satellite_products:
      - name: Red Hat Enterprise Linux for x86_64
        repository_sets:
          - name: Red Hat Enterprise Linux 8 for x86_64 - BaseOS (RPMs)
            releasever: 8

Once the repository_set is identified and enabled, list the repositories with

    hammer repository list --product "Red Hat Enterprise Linux Server" --organization Acme

These repository names are used in the Content View definition:

    satellite_content_views:
      - name: RHEL-7-Base-CV
        repositories:
          - name: Red Hat Enterprise Linux 7 Server RPMs x86_64 7Server
            product: Red Hat Enterprise Linux Server
          - name: Red Hat Enterprise Linux 7 Server - Extras RPMs x86_64
            product: Red Hat Enterprise Linux Server
          - name: Red Hat Satellite Client 6 for RHEL 7 Server RPMs x86_64
            product: Red Hat Enterprise Linux Server
