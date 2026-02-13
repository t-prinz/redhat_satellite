# Ansible playbooks to manage Red Hat Satellite

These playbooks can be used to install and configure a Satellite server.  If the Satellite server is already setup, the satellite_config.yml playbook can be used to make configuration changes.

### Prerequisites

* These playbooks make use of the Red Hat "redhat.satellite" collection, which is available via the Ansible Automation Platform subscription.

* The Satellite server must be registered to a content source and have access to the Satellite repositories.

### Using the playbooks

The following workflow shows how these playbooks can be used to set up a Satellite server.  These playbooks make use of the example inventory defined in the directory inventory_example, but the variables can be defined in other ways.

Define some environment variables for convenience

    INVENTORY=inventory_example
    SATELLITE_SERVER=sat.example.com

This playbook will take care of the prerequisites and install the Satellite server

    ansible-playbook -i ${INVENTORY} \
                     -l ${SATELLITE_SERVER} \
                     satellite_prep_install.yml

This playbook will configure the Satellite server

    ansible-playbook -i ${INVENTORY} \
                     -l ${SATELLITE_SERVER} \
                     satellite_config.yml

### Determining the products, repository_sets, and repositories

The playbook satellite_update_cv_errata.yml and the associated survey file (survey_specs/satellite_update_cv_errata.json) illustrate how an Ansible Automation Platform Job Template Survey can be used to update and publish a Content View.  A template like this could be run on a regular basis to update a Content View with the latest errata.

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

### Version information

* In version tag v0.9, the playbooks referenced two variables that, in turn, referenced two files that held the variables controlling the exeuction of the playbooks.  These variables are no longer used; instead, the playbooks expect the variables to be defined as part of the inventory.

* In version tag v2.0, the playbooks started making use of version 4.x of the redhat.satellite collection.  This version introduced changes to the way Content View filters are managed.

* In version tag v2.1, the playbooks satellite_prep.yml and satellite_install.yml were merged into a single playbook for both preparation and installation.

* In version tag v3.0, the tasks that dealt with registering the Satellite server for content access were removed.  Consequently, the Satellite server must now be registered and have access to the Satellite repositories prior to using these playbooks.  The playbooks take care of enabling the proper repositories.
