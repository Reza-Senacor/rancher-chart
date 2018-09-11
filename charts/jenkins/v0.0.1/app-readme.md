Install ParcelOS Environment
=======

This project contains the playbooks and roles to install the parcelOS system on the different environments.

This Project is build as GIT submodule project so the singles Roles are included as Submodules into this playbook.
 
Inside this project 3 different Playbooks are included.

* installCentral.yml: To install the Central setup.
* installDepot.yml: To install the Depot setup.
* installUniqueTestDB.yml: Just for test reasons to create a dummy uniqueDB

Project Structure
------------

This Project is structured in the following way:


    --ansible-playbook-install-parcelos-env
      |
      |-- ansible-role-create-flyaway-cli
      |-- ansible-role-create-postgresql-db
      |-- ansible-role-flyawy-migrate-db-schema
      |-- ansible-role-oracle-java
      |-- ansible-role-wildfly-standalone-service
      |
      |-- files
      |   |-- applications <-- This folder contains the Application files to deploy (see: ansible-role-flyawy-migrate-db-schema/README.md)
      |   |-- migrations <-- This folder contains the DB Migrations (see: ansible-role-flyawy-migrate-db-schema/README.md)
      |
      |-- inventories
      |   |-- integration
      |       |-- hosts <-- The inventory file for the integration environment.
      |   |-- test
      |       |-- hosts<-- The inventroy file for the test environment.
      |
      |-- installCentral.yml
      |-- installDepot.yml
      |-- installUniqueTestDB.yml

**Roles:**

The roles are independent projects and are just includes as GIT submodules. Pleas have a look at the specific
 README.md files of the projects to learn the specific behavior of the projects.
 
**files:**

The files folder will be process by the roles to load nen static artifacts link applications to deploy and Database 
migrations. Typically these files has to be updates with the newest or target version before running the playbook. 
Because of that they are not committed inside the Ansible roles. 

**inventories**

The inventory folders contains information for the different environment. They will be used to deploy a specific environment.
The Inventory which should be used can be set with the ansible command. E.g.:

     ansible-playbook -i inventories/integration/hosts installDepot.yml 
     
**playbooks**
  
As mentioned above more than one playbook is included into this project. This is because they have nearly the same 
behavior and context.
 
You have to choose the playbook to install in the ansible-playbook command. 

     ansible-playbook -i inventories/integration/hosts installDepot.yml
     

Playbool extra-vars
--------------


```yaml
#The instance variable define which depot will be used. The port offset and sequence offset will be calculated by it.   
instance_number=002

```

Dependencies
------------

To use the playbooks inside this project you need an environment with the following requirements

* Ansible compatible operation system (e.g. Linus, Unix, MacOS) Windows is not valid to run ansible.
* An running GIT installation to checkout this project
* A running Ansible installation to execute this playbook 

SSH Connection
------------

To Access the target System with the Ansible script you nee ssh access from the mashing which run the ansibel script (jenkins) 
and the target test environment. 

To do that you have to create a install user (if not exits), give him sudo rights (visudo) and copy the ssh key of the 
calling machine (jenkins) to it (ssh-copy-id). For example:
 
     ssh-copy-id -i /var/lib/jenkins/.ssh/id_rsa.pub admin@10.4.52.34
     
You can make the new connection available for ansible with the inventories filed.

    [Unique-DB]
    10.4.52.34 ansible_user=admin
    
    [Central-DB]
    10.4.52.34 ansible_user=admin
    
    [Depot-DB]
    10.4.52.34 ansible_user=admin


## Usage


### Step 1: Checkout Project 

Checkout this project as GIT submodule project with

     git clone --recursive ssh://git@collaboration.gls-group.eu:7999/dev/ansible-playbook-install-parcelos-env.git
     
Alternative the this syntax can be used:

     git clone ssh://git@collaboration.gls-group.eu:7999/dev/ansible-playbook-install-parcelos-env.git
     git submodule update --init

### Step 2: Provide current artifacts version

The current version of the arthefacts which should be deployed to the environment has to be added to the project.

Build the parcelOS modules to deploy:

1. Checkout the ParcelOS GIT Submodule Project

       
       #checkout: 
       git clone --recursive ssh://git@collaboration.gls-group.eu:7999/posp/parcelos.git

       #update:
       git submodule foreach git pull origin master


2. Build the project: 


       cd parcelos
       cd  parcelos-fwk
       ./gradlew clean build copyJar
       cd ../parcelos-subsystem-parcel
       ./gradlew clean build copyJar
       cd ..

2. Copy the application from parcelos

      
        cp parcelos/parcelos-subsystem-parcel/parcelos-parcel-service/build/libs/*.war  ansible-playbook-install-parcelos-env/files/applications
        cp parcelos/parcelos-subsystem-parcel/parcelos-parcel-replication/build/libs/*.war  ansible-playbook-install-parcelos-env/files/applications
        

3. Copy the database scripts


        cp parcelos/parcelos-data-model/SQL Scripts/sql/allSchemas/postgresql  ansible-playbook-install-parcelos-env/files/migrations/sql/postgresql
        cp parcelos/parcelos-data-model/SQL Scripts/testdata/allSchemas/postgresql  ansible-playbook-install-parcelos-env/files/migrations/testdata/postgresql
        cp parcelos/parcelos-data-model/SQL Scripts/snapshot/allSchemas/postgresql  ansible-playbook-install-parcelos-env/files/migrations/snapshot/postgresql

3.1 Maybe overwirte the DB Migration if you wish to use a more specific version. 


### Step 3: Edit variables to install the current versions

Open the two playbooks and edit the version of application which should be deployed. 


         vi installDepot.yml
         vi installCentral.yml
         
Set the version of the application to the versions of the arthefacts which has been copyed under files/applications
 
 
### Step 4: Run the installation

1. Install the central DB and applocation on the integration environment
 
 
         ansible-playbook -i inventories/integration/hosts installCentral.yml 
         

2. Install the first and second depot on the integration environment
         
         
         ansible-playbook -i inventories/integration/hosts installDepot.yml -e instance_number="001"
         ansible-playbook -i inventories/integration/hosts installDepot.yml -e instance_number="002"

          



License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
