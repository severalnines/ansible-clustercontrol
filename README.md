# Ansible Role: ClusterControl (DEVELOPMENT BRANCH)

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

## Usage

1) Get the ClusterControl Ansible role from Ansible Galaxy or Github.

Ansible Galaxy (always stable from master branch):

    ansible-galaxy install severalnines.clustercontrol

Github (master and devel branch):

    git clone -b devel https://github.com/severalnines/ansible-clustercontrol
    cp -rf ansible-clustercontrol /etc/ansible/roles/severalnines.clustercontrol

2) Create a playbook. Examples in the Playbook section.

3) Run the playbook.

    ansible-playbook example-playbook.yml


## Example Playbook

Consider the following inside ``/etc/ansible/hosts``:

```
[clustercontrol]
192.168.55.100

# create galera
[galera]
192.168.55.171
192.168.55.172
192.168.55.173

# create new replication
[mysql-replication]
192.168.55.204
192.168.55.205
```


The following playbook will install ClusterControl on 192.168.55.100, setup passwordless SSH on Galera and MySQL replication nodes, then post create/add job into ClusterControl for the deployment:

```yml
- hosts: clustercontrol
  roles:
  - { role: severalnines.clustercontrol }
  vars:
    cc_admin:
      - email: "acop@email.com"
        password: "test123"
    cc_license:
      - email: "ashraf@severalnines.com"
        company: "Severalnines"
        expired_date: "31/12/2016"
        key: "14359875617895464638"

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
        mysql_password: "kemahiranhidup"
        mysql_port: 3306
        mysql_version: "5.7"
        ssh_keyfile: "/root/.ssh/id_rsa"
        ssh_port: "22"
        ssh_user: "root"
        sudo_password: ""
        type: "mysql"
        vendor: "percona"
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

** Take note that the following tags in role:
 - no tag (default) - Install ClusterControl
 - dbnodes - For all managed nodes to setup passwordless SSH
 - deploy-database - To deploy database after ClusterControl is installed

Variables are mostly similar as the key in JSON job command inside Global Job in ClusterControl. If a key:value is not specified, the default value is used.

If you are running as another user, ensure the user has ability to escalate as super user via sudo. Example playbook for Ubuntu 12.04 with sudo password enabled:

    - hosts: ubuntu@192.168.10.100
      become: yes
      become_user: root
      roles:
        - { role: severalnines.clustercontrol }

Then, execute the command with `--ask-become-pass` flag.

## Role Variables (Outdated. Will update soon.)

Available variables are listed below, along with default values (see `defaults/main.yml`):

    mysql_root_password: password

The MySQL root user account password. ClusterControl will setup the MySQL root user with this password during the installation.

    mysql_root_username: root

The MySQL super user account username. It's recommended to keep the default.

    cmon_mysql_password: cmon

The MySQL password for user 'cmon'. ClusterControl requires this MySQL user to access the CMON database.

    cmon_mysql_port: 3306

ClusterControl will install MySQL/MariaDB server to listen on this port, and ClusterControl applications will be configured accordingly.

    cmon_ssh_user: root

ClusterControl will generate an SSH key for this user.

    cmon_ssh_key_path: /root/.ssh/id_rsa

Location of SSH key file generated for `cmon_ssh_user`. The default value is /root/.ssh/id_rsa corresponds to the default cmon_ssh_user. For non-root user, specify `/home/[user]/.ssh/id_rsa` instead.

## Limitations

This playbook is built on top of Ansible v1.9.4 and has been tested on following platforms:

 - Debian 8.x (jessie)
 - Ubuntu 12.04 LTS (precise)
 - RHEL/CentOS 6/7

## Author Information

This role was created in 2016 by Ashraf Sharif from [Severalnines AB](http://severalnines.com/).
