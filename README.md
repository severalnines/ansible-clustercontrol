# Ansible Role: ClusterControl

Installs and configures Severalnines ClusterControl on RHEL/CentOS or Debian/Ubuntu servers. 

## Overview

Installs ClusterControl for your new database node/cluster deployment or on top of your existing database node/cluster. ClusterControl is a management and automation software for database clusters. It helps deploy, monitor, manage and scale your database cluster.

Supported database clusters:

 - Galera Cluster for MySQL
 - Percona XtraDB Cluster
 - MariaDB Galera Cluster
 - MySQL Replication
 - MySQL single instance
 - MySQL Cluster (NDB)
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

1) Clone the repository into the Ansible control host.

    git clone https://github.com/severalnines/ansible

2) Copy the severalnines.clustercontrol directory into `/etc/ansible/roles`.

    cp -rf ansible/severalnines.clustercontrol /etc/ansible/roles

3) Create a playbook. Examples as in the Playbook section.

4) Run the playbook.

    ansible-playbook cc.playbook

5) Once ClusterControl is installed, go to http://[ClusterControl_IP_address]/clustercontrol and create the default admin user/password.

6) On ClusterControl node, setup passwordless SSH key to all target DB nodes. For example, if ClusterControl node is 192.168.0.10 and DB nodes are 192.168.0.11,192.168.0.12, 192.168.0.13:

    ssh-copy-id 192.168.0.11 # DB1
    ssh-copy-id 192.168.0.12 # DB2
    ssh-copy-id 192.168.0.13 # DB3

** Enter the password to complete the passwordless SSH setup.

7) Start to deploy a new database cluster or add an existing one.


## Example Playbook

The simplest playbook would be:

    - hosts: clustercontrol-server
      roles:
        - { role: severalnines.clustercontrol }

If you would like to specify custom configuration values as explained above, create a file called `vars/main.yml` and include it inside the playbook:

    - hosts: 192.168.10.15
      vars:
        - vars/main.yml
      roles:
        - { role: severalnines.clustercontrol }

*Inside `vars/main.yml`*:

    mysql_root_username: admin
    mysql_root_password: super-user-password
    cmon_mysql_password: super-cmon-password
    cmon_mysql_port: 3307

If you are running as another user, ensure the user has ability to escalate as super user via sudo. Example playbook for Ubuntu 12.04 with sudo password enabled:

    - hosts: ubuntu@192.168.10.100
      become: yes
      become_user: root
      roles:
        - { role: severalnines.clustercontrol }

Then, execute the command with `--ask-become-pass` flag.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    mysql_root_password: password

The MySQL root user account password.

    mysql_root_username: root

The MySQL super user account username. It's recommended to keep the default.

    cmon_mysql_password: cmon

The MySQL password for user 'cmon'.

    cmon_mysql_port: 3306

ClusterControl will install MySQL/MariaDB server to listen on this port, and ClusterControl applications will be configured accordingly.

    cmon_ssh_user: root

ClusterControl will generate an SSH key for this user.

    cmon_ssh_key_path: /root/.ssh/id_rsa

Location of SSH key file generated for `cmon_ssh_user`. The value must be corresponding to the user. For non-root user, specify `/home/[user]/.ssh/id_rsa`

## Limitations

This playbook is built on top of Ansible v1.9.4 and has been tested on following platforms:
 - Debian 8.x (jessie)
 - Ubuntu 12.04 LTS (precise)
 - RHEL/CentOS 6/7

## Author Information

This role was created in 2016 by Ashraf Sharif from [Severalnines AB](http://severalnines.com/).
