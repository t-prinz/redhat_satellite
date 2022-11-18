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
