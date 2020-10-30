# Ansible Role: ClusterControl

Installs and configures Severalnines ClusterControl on RHEL/CentOS or Debian/Ubuntu servers. It also supports deploy a new cluster and import existing cluster into ClusterControl automatically.

## Overview

Installs ClusterControl for your new database node/cluster deployment or on top of your existing database node/cluster. ClusterControl is a management and automation software for database clusters. It helps deploy, monitor, manage and scale your database cluster.

Supported database clusters:

 - MySQL/MariaDB Replication
 - Percona XtraDB Cluster
 - MariaDB Cluster (Galera)
 - MySQL Cluster
 - MongoDB Replica Set
 - MongoDB Sharded Cluster
 - PostgreSQL Streaming Replication
 - TimescaleDB Streaming Replication

More details at [Severalnines](http://www.severalnines.com) website.

## Requirements

Make sure you meet following criteria prior to the deployment:

 - ClusterControl node must run on a clean dedicated host with internet connection.
 - If you are running as non-root user, make sure the user is able to escalate to root with sudo command.

## Installation

1) Get the ClusterControl Ansible role from Ansible Galaxy or Github.

Ansible Galaxy (always stable from master branch):

```bash
ansible-galaxy install severalnines.clustercontrol
```

Github:

*(master branch)*

```bash
git clone https://github.com/severalnines/ansible-clustercontrol
cp -rf ansible-clustercontrol /etc/ansible/roles/severalnines.clustercontrol
```

2) Create playbooks. Refer to the [Example Playbook](#example-deployment-and-playbook) section or [examples](/tree/master/examples) directory.

3) Run the playbook.

```bash
ansible-playbook example-playbook.yml
```


## Usage

The role is capable to perform the following:
  - Install ClusterControl only
  - Install ClusterControl with automatic deployment:
      - Create a new database cluster
      - Add existing database cluster

### Install ClusterControl only

If you would like to install ClusterControl only, just call the ``severalnines.clustercontrol`` role as the following in the playbook:

```yml
- hosts: clustercontrol-server
  roles:
    - { role: severalnines.clustercontrol }
```

Or, tag it with ``controller``:

```yml
- hosts: clustercontrol-server
  roles:
    - { role: severalnines.clustercontrol, tags: controller }
```

For sudo user, use ``become``:

```yml
- hosts: clustercontrol-server
  become: yes
  roles:
    - { role: severalnines.clustercontrol }
```


The above is similar to the standard ClusterControl installation using `install-cc` script available in our website. Once the playbook is executed, open ClusterControl UI at `http://{ClusterControl_host}/clustercontrol` and create the admin user by using email address and password.

#### Using Vault in playbooks

The following example shows the steps to use Ansible Vault to encrypt the ``mysql_root_password`` and ``cmon_mysql_password`` configured on the ClusterControl host.

- ``mysql_root_password``: myrootsecret
- ``cmon_mysql_password``: cmonpassw0rd

1) Create a password file to encrypt variables on the host where the playbook resides, called ``password-file``:

```bash
$ echo -n 'thisismysecrettoencrypt' > password-file
```


2) Generate the Vault value for ``mysql_root_password`` variable using the same password file:

```bash
$ echo -n 'myrootsecret' | ansible-vault encrypt_string --vault-id dev@password-file --stdin-name 'mysql_root_password'
Reading plaintext input from stdin. (ctrl-d to end input)
mysql_root_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          34643531626238366432363932353565336639396661326638393365343031373838666236353661
          3739373864643066363432643035613564653437653935370a396133633630333338333339353531
          39336532633263366437376266636534386533303732336562303733636231373139323533656362
          3765353865356333310a663263613837396362303063656361663166376133386331393265313465
          6562
Encryption successful
```

3) Generate the Vault value for ``cmon_mysql_password`` variable using the same password file:

```bash
$ echo -n 'cmonpassw0rd' | ansible-vault encrypt_string --vault-id dev@password-file --stdin-name 'cmon_mysql_password'
Reading plaintext input from stdin. (ctrl-d to end input)
cmon_mysql_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          33656633633962363964306365393664623533613831626162333262616666626237636439626137
          6263303232666561333534336661336361663635383639300a623937353262656564653031313265
          32653663393534626434633663316635653862366334613234646166666361326363616262663463
          6463666338616165620a663931656131383238643635646437353935326535353766626339613364
          3063
```

4) Then add the vaults variables into the playbook:

```yml

- hosts: clustercontrol
  roles:
  - { role: severalnines.clustercontrol, tags: controller }
  vars:
    mysql_root_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          39353462646431303830643763346462666337393730646361656538383130313164653530333363
          3132343532336537653561386539356136343136663635360a353733653139303364666339663831
          39373732363363333462373539313331613733333739613663303037633963316437336631383566
          3334376330363730360a343364623933626362613362303164373130393431333631363464323935
          3737
    cmon_mysql_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          33656633633962363964306365393664623533613831626162333262616666626237636439626137
          6263303232666561333534336661336361663635383639300a623937353262656564653031313265
          32653663393534626434633663316635653862366334613234646166666361326363616262663463
          6463666338616165620a663931656131383238643635646437353935326535353766626339613364
          3063
```

5) Run the playbook with correct vault ID:

```bash
$ ansible-playbook --vault-id=dev@password-file deploy-cc.yml
```

Variables you may want to encrypt in this role:
1) `mysql_root_password`
2) `cmon_mysql_password`
3) `cc_ldap -> admin_password`
4) `cc_admin -> password`
5) `cc_license -> key`


Example playbook with Ansible Vault on sensitive variables as mentioned above:

```yml

- hosts: clustercontrol
  roles:
  - { role: severalnines.clustercontrol, tags: controller }
  vars:
    mysql_root_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          39353462646431303830643763346462666337393730646361656538383130313164653530333363
          3132343532336537653561386539356136343136663635360a353733653139303364666339663831
          39373732363363333462373539313331613733333739613663303037633963316437336631383566
          3334376330363730360a343364623933626362613362303164373130393431333631363464323935
          3737
    cmon_mysql_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          33656633633962363964306365393664623533613831626162333262616666626237636439626137
          6263303232666561333534336661336361663635383639300a623937353262656564653031313265
          32653663393534626434633663316635653862366334613234646166666361326363616262663463
          6463666338616165620a663931656131383238643635646437353935326535353766626339613364
          3063
    cc_admin:
      - set: true
        email: "my@severalnines.com"
        password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          30219633633962363964306365393664623533613831626162333262616666626237636439626139
          2263303232666561333534336661336361663635383639300a623937353262656564653031313262
          32653663393534626434633663316635653862366334613234646166666361326363616262663461
          6463666338616165620a663931656131383238643635646437353935326535353766626339613341
          9163
    cc_license:
      - set: true
        email: "my@severalnines.com"
        company: "My Company"
        expired_date: "31/12/2020"
        key: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          38373639386536656233656264613164373531613235633037653837653835343466393136336631
          3461663137636135633461366461366565383330353037340a346131343339633239383730346533
          38636331663333613131353635393034613232656230356439393262343662373033343530343632
          6336376138333831300a313938373439656261656264336165323831383363653963636562303837
          3435
    cc_ldap:
      - set: true
        enabled: 1
        host: "ldaps://directory.severalnines.com"
        port: 636
        base_dn: "dc=severalnines,dc=com"
        admin_dn: "cn=Administrator,dc=severalnines,dc=com"
        admin_password: !vault |
          $ANSIBLE_VAULT;1.2;AES256;dev
          37373639386536656233656264613164373531613235633037653837653835343466393136336631
          1461663137636135633461366461366565383330353037340a346131343339633239383730346539
          88636331663333613131353635393034613232656230356439393262343662373033343530343620
          6336376138333831300a313938373439656261656264336165323831383363653963636562303839
          3881
        user_dn: "ou=Users,dc=severalnines,dc=com"
        group_dn: "ou=Group,dc=severalnines,dc=com"
```

Once the playbook is executed, open ClusterControl UI at http://{ClusterControl_host}/clustercontrol and login with the settings defined under `cc_admin` section.

### Install ClusterControl with automatic deployment

The role also supports automatic database deployment by leveraging the CMON RPC interface. This will minimize the deployment time to get your database cluster up and running. Example playbook for automatic deployment in AWS EC2 can be found [here](/tree/master/examples/ec2).

Consider the following inside ``/etc/ansible/hosts``:

```
[clustercontrol]
192.168.55.100

[galera]
192.168.55.171
192.168.55.172
192.168.55.173

[mysql-replication]
192.168.55.204
192.168.55.205
```

The following playbook will install ClusterControl on 192.168.55.100, setup passwordless SSH on Galera and MySQL replication nodes, then post create/add job into ClusterControl for the deployment:

```yml
- hosts: clustercontrol
  roles:
  - { role: severalnines.clustercontrol, tags: controller }

- hosts:
    - mysql-replication
    - galera
  roles:
    - { role: severalnines.clustercontrol, tags: dbnodes }
  vars:
    clustercontrol_ip_address: 192.168.55.100
    ssh_user: root

- hosts: clustercontrol
  roles:
    - { role: severalnines.clustercontrol, tags: deploy-database }
  vars:
    cc_cluster:
      # create new mysql replication. first node is the master
      - deployment: true
        operation: "create"
        cluster_type: "replication"
        nodes:
          - hostname: '192.168.55.204' # the first node is the master
            hostname_data: '192.168.55.204'
            hostname_internal: ""
            port: "3306"
          - hostname: '192.168.55.205'
            hostname_data: '192.168.55.205'
            hostname_internal: ""
            port: "3306"
        mysql_cnf_template: "my.cnf.repl57"
        mysql_datadir: "/var/lib/mysql"
        mysql_password: "password"
        mysql_port: 3306
        mysql_version: "5.7"
        ssh_keyfile: "/root/.ssh/id_rsa"
        ssh_port: "22"
        ssh_user: "root"
        sudo_password: ""
        type: "mysql"
        vendor: "oracle"
      # add existing galera.
      - deployment: true
        operation: "add"
        cluster_type: "galera"
        mysql_password: "password"
        nodes:
          - hostname: '192.168.55.171'
            hostname_data: '192.168.55.171'
            hostname_internal: ""
            port: "3306"
          - hostname: '192.168.55.172'
            hostname_data: '192.168.55.172'
            hostname_internal: ""
            port: "3306"
          - hostname: '192.168.55.173'
            hostname_data: '192.168.55.173'
            hostname_internal: ""
            port: "3306"
        ssh_keyfile: "/root/.ssh/id_rsa"
        ssh_port: 22
        ssh_user: root
        vendor: percona
        sudo_password: ""
        galera_version: "3.x"
        enable_node_autorecovery: true
        enable_cluster_autorecovery: true
      # minimal create new galera
      - deployment: true
        operation: "create"
        cluster_type: "galera"
        mysql_cnf_template: "my.cnf.galera" # Use my.cnf.mdb10x-galera" for MariaDB Galera
        mysql_datadir: "/var/lib/mysql"
        nodes:
          - hostname: '192.168.55.191'
            hostname_data: '192.168.55.191'
            hostname_internal: ""
            port: "3306"
          - hostname: '192.168.55.192'
            hostname_data: '192.168.55.192'
            hostname_internal: ""
            port: "3306"
          - hostname: '192.168.55.193'
            hostname_data: '192.168.55.193'
            hostname_internal: ""
            port: "3306"
        mysql_password: "password"
        mysql_port: 3306
        mysql_version: "5.6"
        ssh_keyfile: "/root/.ssh/id_rsa"
        ssh_user: "root"
        sudo_password: ""
        vendor: "percona"
```

** Take note the following tags in the `role` lines:
 - no tag (default) - Install ClusterControl
 - `dbnode` - For all managed nodes to setup passwordless SSH
 - `deploy-database` - To deploy database after ClusterControl is installed

Variables are mostly similar to keys in JSON job command created in ClusterControl's Cluster Job. If a key:value is not specified, the default value is used.

If you would like to use non-root user to as ClusterControl's SSH user, ensure the user has ability to escalate as super user via sudo. Example playbook for Ubuntu 12.04 with sudo password enabled:

```yml
- hosts: ubuntu@192.168.10.100
  become: yes
  become_user: root
  roles:
    - { role: severalnines.clustercontrol }
```

Then, execute the command with `--ask-become-pass` flag.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### MySQL and CMON on ClusterControl node

`mysql_root_password: password`

- The MySQL root user account password. ClusterControl will setup the MySQL root user with this password during the installation.

`mysql_root_username: root`

- The MySQL super user account username. It's recommended to keep the default.

`cmon_mysql_password: cmon`

- The MySQL password for user 'cmon'. ClusterControl requires this MySQL user to access the CMON database.

`cmon_mysql_port: 3306`

- ClusterControl will install MySQL/MariaDB server to listen on this port, and ClusterControl applications will be configured accordingly.

`cmon_ssh_user: root`

- ClusterControl will generate an SSH key for this user.

`cmon_ssh_key_path: /root/.ssh/id_rsa`

- Location of SSH key file generated for `cmon_ssh_user`. The default value is `/root/.ssh/id_rsa` corresponds to the default cmon_ssh_user. For non-root user, specify `/home/[user]/.ssh/id_rsa` instead.

### Admin Credentials and License

At the moment, the following options are configurable for ClusterControl. All of them are self-explanatory so we leave it with no description (default is `set: false`).

Example usage:

```yml
cc_admin:
  - set: true
    email: "admin@email.com"
    password: "test123"
cc_license:
  - set: true
    email: "demo@severalnines.com"
    company: "Severalnines"
    expired_date: "31/12/2016"
    key: "XXXXXXXXXXXXXXXXXXXX"
```

**It's highly recommended to use Ansible Vault to store confidential information in the playbook. See 'Using Vault in playbooks' section.**

### LDAP Settings

If you would like to integrate LDAP authentication (OpenLDAP, Active Directory, FreeIPA) with ClusterControl, define the settings here with `set: true` and `enabled: 1`:

```yml
cc_ldap:
  - set: true
    enabled: 1
    host: "192.168.1.100"
    port: 389
    base_dn: "dc=mydomain,dc=com"
    admin_dn: "cn=admin,dc=mydomain,dc=com"
    admin_password: "password"
    user_dn: "ou=People,dc=mydomain,dc=com"
    group_dn: "ou=Group,dc=mydomain,dc=com"
```

**It's highly recommended to use Ansible Vault to store confidential information in the playbook. See 'Using Vault in playbooks' section.**

`host: {FQDN or LDAPS URL}`

- The LDAP server hostname or IP address. To use LDAP over SSL/TLS, specify LDAP URI instead, for example `ldaps://LDAP_host`.

`port: 389`

- Default to null. Specify the LDAP server port.

`base_dn: {distinguished name}`

- The root LDAP node under which all other nodes exist in the directory structure.

`admin_dn: {distinguished name}`

- The distinguished name used to bind to the LDAP server. This is often the administrator or manager user. It can also be a dedicated login with minimal access that should be able to return the DN of the authenticating users. ClusterControl must do an LDAP search using this DN before any user can log in. This field is case-sensitive.

`admin_password: {string}`

- The password for the binding user specified in `admin_dn`.

`user_dn: {distinguished name}`

- The user’s relative distinguished name (RDN) used to bind to the LDAP server. For example, if the LDAP user DN is `CN=userA,OU=People,DC=ldap,DC=domain,DC=com`, specify `OU=People,DC=ldap,DC=domain,DC=com`. This field is case-sensitive.

`groupd_dn: {distinguished name}`

- The group’s relative distinguished name (RDN) used to bind to the LDAP server. For example, if the LDAP group DN is `CN=DBA,OU=Group,DC=ldap,DC=domain,DC=com`, specify `OU=Group,DC=ldap,DC=domain,DC=com`. This field is case-sensitive.


### Create new database cluster

Supported create new database cluster:
  - Galera cluster
  - MySQL replication

`deployment: true`

- If true, the role will always send the deployment job to CMON regardless the database cluster is already deployed or not. It's recommended to set it to false once the cluster is successfully created. Default is false.

`operation: "create"`

- This is mandatory for creating new database cluster.

`api_id: 1`

- API ID for ClusterControl RPC interface. Keep it default is recommended.

`cluster_type: "galera"`

- Cluster type. Supported values are: galera, replication.

`create_local_repository: false`

- Create and mirror the current database vendor’s repository and then deploy using the local mirrored repository. This is a preferred option when you have to scale the Galera Cluster in the future, to ensure the newly provisioned node will always have the same version as the rest of the members.

`data_center: 0`

- Exclusive for Galera cluster. Specify the gmcast.segment number to distinguish the geographical location.

`disable_firewall: true`

- Whether to enable or disable firewall. Set to `false` if you have opened required ports for specific cluster.

`disable_selinux: true`

- Whether to enable or disable selinux. Set to `false` if you have added SElinux rules accordingly.

`enable_mysql_uninstall: true`

- Set to `false` if you already have MySQL installed and you don't want ClusterControl to uninstall it during the deployment.

`generate_token: true`

- ClusterControl will generate a new RPC token for the cluster. Keep it default is recommended.

`vendor: "percona"`

- For Galera cluster:
  - Percona XtraDB Cluster - 5.6 and 5.7.
  - MariaDB Galera Cluster - 10.3 and 10.4.
- For MySQL Replication:
  - Oracle - 5.7 and 8.0
  - Percona - 5.6, 5.7 and 8.0
  - MariaDB - 10.3 and 10.4

`mysql_cnf_template: "my.cnf.galera"`

- MySQL configuration template file under `/usr/share/cmon/templates`. Use one of the following values accordingly: `my57.cnf.galera`, `my.cnf.80-pxc`, `my.cnf.galera`, `my.cnf.gtid_replication`, `my.cnf.mdb10x-galera`, `my.cnf.mdb10x-replication`, `my.cnf.mdb55-galera`, `my.cnf.mysqlcluster`, `my.cnf.pxc55`, `my.cnf.repl57`, `my.cnf.repl80`, `my.cnf.replication`.

`mysql_datadir: "/var/lib/mysql"`

- Location of MySQL data directory.

```yml
 nodes:
   - hostname: '192.168.55.191'
     hostname_data: '192.168.55.191'
     hostname_internal: ""
     port: "3306"
   - hostname: '192.168.55.192'
     hostname_data: '192.168.55.192'
     hostname_internal: ""
     port: "3306"
   - hostname: '192.168.55.193'
     hostname_data: '192.168.55.193'
     hostname_internal: ""
     port: "3306"
 ```

- List of the database nodes hostnames or IP address for this database cluster. For MySQL Replication, the first node is the master. Every value in the `nodes` section can have the following fields:
  * `hostname` - Hostname of a node (mandatory).
  * `hostname_data` - Optional hostname of a node to be used for sampling database statistics.
  * `hostname_internal` - Optional hostname of a node to be used for the cluster's internal data communication. ClusterControl should not use this to communicate with any of the nodes.
  * `port` - The database port.

For MySQL Replication, the first node is the master.

`mysql_password: "password"`

- Specify MySQL root password. ClusterControl will configure the same MySQL root password for all instances in the cluster.

`mysql_version: "5.6"`

- For Galera cluster:
  - Percona XtraDB Cluster - 5.6 and 5.7.
  - MariaDB Galera - 10.3 and 10.4.
- For MySQL Replication:
  - Oracle - 5.7 and 8.0
  - Percona - 5.6, 5.7 and 8.0
  - MariaDB - 10.3 and 10.4

`software_package: true`

- Existing MySQL dependencies will be removed. New packages will be installed and existing packages will be uninstalled when provisioning the node with required software.

`ssh_user: "root"`

- SSH user. Default is root.

`ssh_keyfile: "/root/.ssh/id_rsa"`

- The full path of SSH key (the key must exist in ClusterControl node) that will be used by SSH User to perform passwordless SSH.

`ssh_port: 22`

- SSH port for the target DB nodes.

`sudo_password: ""`

- If you use sudo with password, specify it here. Ignore this if SSH User is root or sudoer does not need a sudo password.

`type: "mysql"`

- Database type. Supported value is `mysql`. In the future, we are going to support `mongodb` and `postgresql`.

### Add existing database cluster

Supported add existing database cluster:
  - Galera Cluster

`deployment: true`

- If true, the role will always send the deployment job to CMON regardless the database cluster is already deployed or not. It's recommended to set it to false once the cluster is successfully created. Default is false.

`operation: "add"`

- This is mandatory for add existing cluster.

`api_id: 1`

- API ID for ClusterControl RPC interface. Keep it default is recommended.

`cluster_type: "galera"`

- Cluster type. Supported value is `galera`.

`vendor: "percona"`

- For Galera cluster:
  - percona - Percona XtraDB Cluster (5.6/5.7)
  - codership - MySQL Galera Cluster (5.6/5.7)
  - mariadb - MariaDB Galera Cluster (10.3/10.4)

`mysql_datadir: "/var/lib/mysql"`

- Location of MySQL data directory.

```yml
 nodes:
   - hostname: '192.168.55.191'
     hostname_data: '192.168.55.191'
     hostname_internal: ""
     port: "3306"
   - hostname: '192.168.55.192'
     hostname_data: '192.168.55.192'
     hostname_internal: ""
     port: "3306"
   - hostname: '192.168.55.193'
     hostname_data: '192.168.55.193'
     hostname_internal: ""
     port: "3306"
 ```

- List of the database nodes hostnames or IP address for this database cluster. Every value in the `nodes` section can have the following fields:
  * `hostname` - Hostname of a node (mandatory).
  * `hostname_data` - Optional hostname of a node to be used for sampling database statistics.
  * `hostname_internal` - Optional hostname of a node to be used for the cluster's internal data communication. ClusterControl should not use this to communicate with any of the nodes.
  * `port` - The database port.

`mysql_password: "password"`

- Specify MySQL root password. ClusterControl assumes the same MySQL root password for all instances in the cluster.

`galera_version: "3.x"`

- The Galera replication library version. Supported values are 3.x and 2.x.

`ssh_user: "root"`

- SSH user. Default is root.

`ssh_keyfile: "/root/.ssh/id_rsa"`

- The full path of SSH key (the key must exist in ClusterControl node) that will be used by SSH User to perform passwordless SSH.

`ssh_port: 22`

- SSH port for the target DB nodes.

`sudo_password: ""`

- If you use sudo with password, specify it here. Ignore this if SSH User is root or sudoer does not need a sudo password.

`enable_node_autorecovery: true`

- ClusterControl will perform automatic recovery if it detects any of the nodes in the cluster is down.

`enable_cluster_autorecovery: true`

- ClusterControl will perform automatic recovery if it detects the cluster is down or degraded.

## Limitations

This playbook is built on top of Ansible v2.9.2 and has been tested on the following platforms:

 - Debian 9.x, 10.x
 - Ubuntu 18.04 LTS
 - RHEL/CentOS 7/8

## Author Information

This role was created in 2016 by Ashraf Sharif from [Severalnines AB](http://severalnines.com/).
