# Ansible Role: ClusterControl

Installs and configures Severalnines ClusterControl on RHEL/CentOS or Debian/Ubuntu servers. It also supports create/add existing database cluster into ClusterControl automatically.

## Overview

Installs ClusterControl for your new database node/cluster deployment or on top of your existing database node/cluster. ClusterControl is a management and automation software for database clusters. It helps deploy, monitor, manage and scale your database cluster.

Supported database clusters:

 - Galera Cluster for MySQL
 - Percona XtraDB Cluster
 - MariaDB Galera Cluster
 - MySQL Replication
 - MySQL single instance
 - MySQL Cluster
 - MongoDB Replica Set
 - MongoDB Sharded Cluster
 - TokuMX Cluster
 - PostgreSQL single instance

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

If you would like to install ClusterControl only, just call the ``severalnines.clustercontrol`` role as the following:

```yml
- hosts: clustercontrol-server
  roles:
    - { role: severalnines.clustercontrol }
```

or, tag it with ``controller``:

```yml
- hosts: clustercontrol-server
  roles:
    - { role: severalnines.clustercontrol, tags: controller }
```

The above is similar to the standard ClusterControl installation using ``install-cc`` script available in our website.

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
  vars:
    cc_admin:
      - email: "admin@email.com"
        password: "test123"
    cc_license:
      - email: "demo@severalnines.com"
        company: "Severalnines"
        expired_date: "31/12/2016"
        key: "XXXXXXXXXXXXXXXXXXXX"

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
        mysql_hostnames:
          - '192.168.55.204'
          - '192.168.55.205'
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
        mysql_hostnames:
          - '192.168.55.171'
          - '192.168.55.172'
          - '192.168.55.173'
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
        mysql_cnf_template: "my.cnf.galera"
        mysql_datadir: "/var/lib/mysql"
        mysql_hostnames:
          - '192.168.55.191'
          - '192.168.55.192'
          - '192.168.55.193'
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
 - dbnodes - For all managed nodes to setup passwordless SSH
 - deploy-database - To deploy database after ClusterControl is installed

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

At the moment, the following options are configurable for ClusterControl. All of them are self-explanatory so we leave it with no description:

Example usage:
```yml
cc_admin:
  - email: "admin@email.com"
    password: "test123"
cc_license:
  - email: "demo@severalnines.com"
    company: "Severalnines"
    expired_date: "31/12/2016"
    key: "XXXXXXXXXXXXXXXXXXXX"
```

### Create new database cluster

Supported create new database cluster:
  - Galera cluster
  - MySQL replication

`deployment: true`

- If true, the role will always send the deployment job to CMON regardless the database cluster is already deployed or not. It's recommended to set it to false once the cluster is successfully created.

`operation: "create"`

- This is compulsory for creating new database cluster.

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
  - Codership and Percona - 5.5 and 5.6.
  - MariaDB - 5.5 and 10.1.
- For MySQL Replication:
  - Oracle - 5.7
  - Percona - 5.7 and 5.6
  - MariaDB - 10.1

`mysql_cnf_template: "my.cnf.galera"`

- MySQL configuration template file under `/usr/share/cmon/templates`. For Galera Cluster, use 'my.cnf.galera'. For MySQL 5.7, use 'my.cnf.repl57'. For MySQL replication 5.6/MariaDB, use 'mysql.cnf.repl'.

`mysql_datadir: "/var/lib/mysql"`

- Location of MySQL data directory.

```yml
mysql_hostnames:
 - '192.168.1.101'
 - '192.168.1.102'
 - '192.168.1.103'
 ```

- List of the MySQL hostnames or IP address for this database cluster. For MySQL Replication, the first node is the master.

`mysql_password: "password"`

- Specify MySQL root password. ClusterControl will configure the same MySQL root password for all instances in the cluster.

`mysql_port: 3306`

- MySQL port for all nodes. Default is 3306.

`mysql_version: "5.6"`

- For Galera cluster:
  - Codership and Percona - 5.5 and 5.6.
  - MariaDB - 5.5 and 10.1.
- For MySQL Replication:
  - Oracle - 5.7
  - Percona - 5.7 and 5.6
  - MariaDB - 10.1

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
  - Galera cluster

`deployment: true`

- If true, the role will always send the deployment job to CMON regardless the database cluster is already deployed or not. It's recommended to set it to false once the cluster is successfully created.

`operation: "add"`

- This is compulsory for add existing cluster.

`api_id: 1`

- API ID for ClusterControl RPC interface. Keep it default is recommended.

`cluster_type: "galera"`

- Cluster type. Supported value is `galera`.

`vendor: "percona"`

- For Galera cluster:
  - percona - Percona XtraDB Cluster (5.5/5.6)
  - codershop - MySQL Galera Cluster (5.5/5.6)
  - mariadb - MariaDB Galera Cluster (5.5/10.x)

`mysql_datadir: "/var/lib/mysql"`

- Location of MySQL data directory.

```yml
mysql_hostnames:
 - '192.168.1.101'
 - '192.168.1.102'
 - '192.168.1.103'
 ```

- List of the MySQL hostnames or IP address for this database cluster.

`mysql_password: "password"`

- Specify MySQL root password. ClusterControl assumes the same MySQL root password for all instances in the cluster.

`mysql_port: 3306`

- MySQL port for all nodes. Default is 3306.

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

This playbook is built on top of Ansible v1.9.4 and has been tested on following platforms:

 - Debian 8.x (jessie)
 - Ubuntu 12.04 LTS (precise)
 - RHEL/CentOS 6/7

## Author Information

This role was created in 2016 by Ashraf Sharif from [Severalnines AB](http://severalnines.com/).
