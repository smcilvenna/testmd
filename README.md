
Healthrules Installer and Provisioners
================================

Perforce locations
---------

Location of environment definitions/hosts:
* //CloudOps/environments for OPS prod/non-prod

Installer/Provisioner code:
* //dev/healthrules/installer/tm1 

TODO:
---------

1) Reconcile roles between infra and installer
2) Document VMWare variables
3) Merge in install and upgrade instructions from README_install_upgrade.txt README_onetime_setup.txt

Naming Conventions
---------

#### File Names
1) Use dash (-) in file and directory names, except for group_vars and host_var where it is required by ansible convention.
2) File names should be lower case
3) Directory names should be lower case (abbreviatons/acronyms as well as full names, no camel case)

#### Ansible code
1) Use dash in host group names (e.g. payor-admin)
2) Use dash in role names (e.g. he-os-packages)
3) Use underscore (_) in ansible variable names (ansible/yml requirement)

#### Bash
1) Use underscore in bash variable names


Environments directory
---------

<pre>
environments
  config
    metadata
      customer-acronyms.txt         acronyms for customer names
      env-acronyms.txt              acronyms for environment types 
    vars
      aws-us-east-2.yml
    CMDB
      servers.txt
      
  <b><em>customer</b></em>                      e.g. iuh
    customer.yml                  default customer settings (name, acronym, provisioner, region)
    customer-<b><em>SLA</b></em>.yml            optional SLA-specific customer overrides/additions
    <env>                         e.g. dev
      override                    file with environment specific settings
      group_vars
        all
          000_all                 link to installer/common/group_vars/all (common settings)
        payor-single-server
          000_payor-single-server link to installer/common/group_vars/all (common settings)
      hosts                       (inventory file with generic host group names, check out examples)
    .
    .
    qa
    prod
</pre>

Example of customer file
---------

<pre>
customer_name:          'Maxie'
customer_acronym:       'mcx'
customer_timezone:      'EASTERN'

provisioner:            'aws'
region:                 'us-east-2'

win_dns_domain_name:    'qa.local'
win_domain_ou_path:     'OU=QA-Computers,DC=qa,DC=local'
</pre>


Example of override file:

<pre>
env_type:           dev
env_type_acronym:   dv
env_sla:            np
env_status:         live
</pre>
# TODO: Settings below are for illustration purposes. Please, update versions for this specific environment.
<pre>
customer_software:
  payor:            5.10.2
  connector:        4.10.2
  db:               installed

FOR A FULL TREATMENT on customer and override files, please see README_load-metadata.md in the 'docs' directory.
</pre>


Installer directory
---------
<pre>
installer (tm1, other branches or distributions)
  
  caremanager_scripts
  
  common
    group_vars
    playbooks
    provisioners
    roles
    scripts
    
  docs
  
  internal 
  
  properties
  
  scripts

  third-party
</pre>   
<pre>
Standard host group names are (found in common/group_vars)

- all
- caremanager
- caremanager-apache-efax
- caremanager-integration
- caremanager-oracle
- connector
- oracle-allservers
- payor-allservers
- payor-apache-lb
- payor-iway
- payor-oracle
- payor-dw-oracle
- payor-primary
- payor-secondaries
- payor-single-server
</pre>

Setting up new environment
---------

### One time setup
<pre>
1) Create a directory where your environments and installer directories will reside (on ansible control 
   servers that directory is /home/ansible/he). We will refer to this directory as <b><em>TOPDIR</em></b> below.
2) Check out //CloudOps/environments to <b><em>TOPDIR</em></b>/environments
3) Either check out installer code (if working in development
   mode) or unpack the installer distribution). That will create <b><em>TOPDIR</em></b>/installer for development environments.
   If distribution artifacts are used, convention is <b><em>TOPDIR</em></b>/he-installer-<b><em>VERSION</em></b>.
4) At the top level of the installer distribution you will find file env-setup. This must be bash-sourced:

   cd <b><em>INSTALLER_DIR</em></b>
   source env-setup
   
   For future ease of use you may place the source command (however reference the env-setup file w/ it's full path) in your .bashrc file.
</pre>
### Adding new customer or new environment
<pre>
1) Check <b><em>TOPDIR</em></b>/environments/config/metadata/customer-acronyms.txt and <b><em>TOPDIR</em></b>/environments/config/metadata/env-acronyms.txt 
   to make sure acronyms for your customer and envrionment type exist
</pre> 
2) Initialize Customer
<pre>
    This is only ONCE for each customer, IOW for a customer's first environment
<br>

    cd <b><em>TOPDIR</em></b> environments
    
    init-cust-dir.sh --customer <b><em>NAME</em></b> [--customer-acronym <b><em>CUSTOMER_ACRONYM</em></b>]
<br>

    This will create a directory named after customer, and place a template customer.yml file in this directory.
    (Customer yaml will not be overridden if it already exists).
    The file should be edited at this point to contain the correct values for the customer.
    The customer.yml file must contain at a minimum:
<br>
    customer_name:      'NewCustomer'         # The customer name
    customer_acronym:   'new'                 # Customer acronym (not to exceed three chars)
    customer_timezone:  'EASTERN'             # Customer timezone
    provisioner:        'aws'                 # Customer DEFAULT provisioner and region
    region:             'us-east-2'
    
    # Following two should be set for VMWare customer environments
    solarwinds_network:  'Mgt Services'        # Customer solarwinds network name
    center_network_name: 'MGT_Service'         # Customer vcenter network name
  

    # Customer DEFAULT domain name and ou path.
    win_dns_domain_name:    'qa.local'
    win_domain_ou_path:     'OU=QA-Computers,DC=qa,DC=local'
<br>
    Additional files with the naming convention customer-<b><em>SLA</b></em>.yml may also be created at this time. For example a customer-cnp.yml file could be created directing critical non-prod environments to a different (DR) data center.
</pre>

3) Initialize environment
<pre>
  cd <b><em>TOPDIR</em></b>/environments
  <br>
  init-env-dir.sh --customer <b><em>CUSTOMER_NAME</em></b> --env-type <b><em>ENV_TYPE</em></b> [--env-acronym <b><em>ENV_ACRONYM</b></em>]
<br>
    This will create a directory named <b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> and place a template override file in this directory.
    (Overide file will not be overridden if it already exists).
<br>
    The file should be edited at this point to contain the correct values for the customer.
    The overide file must contain at a minimum:
<br>
    env_type:         dev            # Environment 'type' or name
    env_type_acronym: dv             # Environment acronym not to exceed two chars.
    env_sla:          cnp            # Environment SLA (e.g. np,cnp,prod,dev)
    env_status:       live           # Environment status or version
<br>
    # The software/applications to be installed in the environment:
    customer_software:
      payor:              5.10.2
      connector:          4.10.2
      db:                 installed
      answers:            installed
      axiom-mssql:        installed
<br>
    # env_size IS NOT required, the default is 'small' but that may be overridden here as well
    env_size:         medium
</pre>


4) Check-in baseline files for new environment.
<pre>
   In <b><em>TOPDIR</em></b>/environments/<b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> directory
   p4 submit -d "Added new environment configuration for <customer name>" ./...
</pre>
  
5) Provision servers
<pre>
      In <b><em>TOPDIR</em></b>/environments/<b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> directory
      provision-env.sh
      
      When script completes you should have your environment VMs provisioned and configured.
      ./hosts file is updated and checked-in with host file names of your servers
      <b><em>TOPDIR</em></b>/environments/CMDB/servers.txt is also updated with server metadata
</pre>
<pre>
6) Generated Payor/Connector HR Installer Config
After the VMs are provisioned, the HR Installer configuration are automatically generated in the directory **generated-hr-properties**
Currently, only configuration for Payor/Connector version of 18.1 and beyond are supported. Only the small env_size is currently supported.
</pre>
<pre>
7.) Random Payor and CareManager Oracle Passwords
As part of provisioning, the provisioner creates random passwords for Payor OLTP/DW and the CareManager Oracle schemas.
The passwords are stored in the file ~/secrets/secrets-<b><em>CUSTOMER_NAME</em></b>-<b><em>ENV_TYPE</em></b>.yml  
</pre>
### Updating environment

To update existing environment we can either re-run provisioning from the top or just run configure playbooks. 
Provisioning script runs configure playbook after it creates servers.

1) Updating links to installer directories by re-running init script
<pre>
   In <b><em>TOPDIR</em></b>/environments directory
   init-env-dir.sh --customer <b><em>CUSTOMER_NAME</em></b> --env-type <b><em>ENV_TYPE</em></b>
   
   Init script can be run against existing directories. It will re-create sym links and leave hosts and overrides files
   alone if they already exist.
</pre>

2) Re-run provisioning script

<pre>
   In <b><em>TOPDIR</em></b>/environments/<b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> directory
   provision-env.sh
   
   Provisioning script is indempotent. Be mindful of changes if server naming scheme. If server names or tags change
   new servers will be provisioned and existing servers will be left orphaned.
</pre>
3) Run just configure playbook (uses static inventory hosts file checked-in in the environment directory)
<pre>
   In <b><em>TOPDIR</em></b>/environments/<b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> directory
   configure-env.sh
</pre>

### Terminating environment

Terminate script will destroy all AWS VMs so it should be used with caution.
<pre>
   In <b><em>TOPDIR</em></b>/environments/<b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> directory
   terminate-env.sh
   
   When script completes servers should be terminated in your cloud provider, entries removed from hosts file and CMDB. 
</pre>
### Starting/stopping AWS environment

Start/stop script will start or stop all AWS VMs
<pre>
    To stop AWS instances, in <b><em>TOPDIR</em></b>/environments/<b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> directory
    aws-state-env.sh --state stopped

    To start AWS instances, in <b><em>TOPDIR</em></b>/environments/<b><em>CUSTOMER_NAME</em></b>/<b><em>ENV_TYPE</em></b> directory
    aws-state-env.sh --state running
</pre>

### Use the following properties in the override file to use different DB export dump file. Files are expected to be /opt/media on ansible control server.
<pre>
oltp_dmp_name: "db_demo_load_5.11-12_09_2017_22_25.dmp"
remap_from_schema: payor_oltp


dw_dmp_name:   "dw_demo_load_5.11-12_09_2017_22_25.dmp"
remap_from_schema_dw: payor_dw


remap_data_tbs: data1

remap_index_tbs: indx1
</pre>
##Upgrade ansible used by provisioner to 2.4.3
<pre>
 New location is <b><em>installer branch</b></em>/third-party/ansible-2.4.3;
</pre>
