# Ansible playbooks to manage Red Hat Satellite

These playbooks can be used to install and configure a Satellite server.  If the Satellite server is already setup, the satellite_config.yml playbook can be used to make configuration changes.

### Prerequisites

* These playbooks make use of the Red Hat "redhat.satellite" collection, which is available via the Ansible Automation Platform (which requires a subscription).

    If the playbooks are run via the command line and ansible-galaxy is used to install the collections (as shown in the examples), the ansible-galaxy client must be configured.  The documentation showing how to do this can be found [here](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#configuring-the-ansible-galaxy-client).

    If the playbooks are run via Ansible Tower, an Ansible Galaxy credential must be created and configured for the Organization.  The documentation showing how to do this can be found [here](https://docs.ansible.com/ansible-tower/latest/html/userguide/projects.html?extIdCarryOver=true&intcmp=701f2000001OEH1AAO&sc_cid=701f2000000u72fAAA#using-collections-in-tower).

    *In either case, credentials for Ansible Automation Hub are required; these credentials can be found [here](https://console.redhat.com/ansible/automation-hub/token).*
        
* Two variables files need to be created; examples of each are given in the vars/ directory.  Note that the playbooks do not automatically search the vars/ directory, so the variables must be specified (see below for an example):

        Credentials file
          Example file                  :  vars/satellite_cred_vars.example.yml
          Variable to specify (required):  satellite_cred_vars

        Configuration file
          Example file                  :  vars/satellite_config_vars.example.yml
          Variable to specify (required):  satellite_config_vars

* If using Ansible from the command line, install the roles and collections using the following commands.

        ansible-galaxy role install -r roles/requirements.yml
        ansible-galaxy collection install -r collections/requirements.yml

### Using the playbooks

The following workflow shows how these playbooks can be used to set up a Satellite server.

Define some environment variables for convenience

    INVENTORY=inventory.yml
    SATELLITE_CRED_VARS=vars/satellite_cred_vars.yml
    SATELLITE_CONFIG_VARS=vars/satellite_config_vars
    SATELLITE_SERVER=sat.example.com

This playbook will register the system and take care of the prerequisites

    ansible-playbook -i ${INVENTORY} \
                     -e satellite_cred_vars=${SATELLITE_CRED_VARS} \
                     -l ${SATELLITE_SERVER} \
                     satellite_prep.yml

This playbook will perform the actual Satellite installation

    ansible-playbook -i ${INVENTORY} \
                     -e satellite_cred_vars=${SATELLITE_CRED_VARS} \
                     -e satellite_config_vars=${SATELLITE_CONFIG_VARS} \
                     -l ${SATELLITE_SERVER} \
                     satellite_install.yml

This playbook will configure the Satellite server

    ansible-playbook -i ${INVENTORY} \
                     -e satellite_cred_vars=${SATELLITE_CRED_VARS} \
                     -e satellite_config_vars=${SATELLITE_CONFIG_VARS} \
                     satellite_config.yml
