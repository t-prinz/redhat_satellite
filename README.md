ansible-galaxy role install -r roles/requirements.yml
ansible-galaxy collection install -r collections/requirements.yml

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
